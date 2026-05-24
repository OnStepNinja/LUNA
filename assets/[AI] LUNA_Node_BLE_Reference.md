# [AI] LUNA Node BLE Reference

> Target: LUNA Bridge v8.9.7 / LUNA Node v1.0.4  
> Updated: 2026-05-16

---

> 💡 **Need help?**  
> - Ask **Claude** (AI): "I want to do [X] with LUNA BLE"  
> - Load this guide into **NotebookLM** and ask questions

---

## Table of Contents

1. [What is LUNA BLE?](#1-what-is-luna-ble)
2. [Controlling the Node from the Bridge](#2-controlling-the-node-from-the-bridge)
3. [Node Network Management (multiple devices & pairing)](#3-node-network-management-multiple-devices--pairing)
4. [What the Node can do — Sensors, GPIO, ADC](#4-what-the-node-can-do--sensors-gpio-adc)
5. [Troubleshooting](#5-troubleshooting)
6. [Current Limitations and Roadmap](#6-current-limitations-and-roadmap)
7. [UUID Cheat Sheet](#7-uuid-cheat-sheet)

---

## 1. What is LUNA BLE?

### In one sentence

> **A wireless control system for Node devices (compact sensor units) — no WiFi required.**

```
You / Claude
    ↕ WiFi · MCP
 LUNA Bridge (Main)   ← Controlled from PC (Claude / Gemini via MCP)
    ↕ BLE (Bluetooth)
 LUNA Node            ← Deployed on-site. No WiFi needed
    ↓
  Sensors · GPIO · ADC
```

### How is this different from regular Bluetooth?

| Common misconception | Reality |
|----------------------|---------|
| You need to pair in the OS Bluetooth settings | **Not required** (do not use the system settings screen) |
| You need a PIN code to connect | **Not required** |
| The device remembers the connection | LUNA uses "connect each time from the app" |

**LUNA Node is not a "pair-in-settings" device like headphones.**  
The Bridge connects programmatically via BLE GATT — no OS pairing screen or smartphone needed.

---

## 2. Controlling the Node from the Bridge

### Basic flow

When you ask Claude to control the Node, the sequence is:

```
Your instruction to Claude
  → Lua script runs on the Bridge
    → Command sent to Node over BLE
      → Node executes the script
        → Result returned to Bridge
```

### How to ask Claude (examples)

```
"Start the sensor measurement script (myapp) on the Node"
"Stop the script running on the Node"
"Deploy a new script to the Node and run it"
"Get the logs from the Node"
```

### Basic Lua pattern (Bridge side)

```lua
local SUB  = "ac:a7:04:fd:16:71"   -- Node BLE address (change to match your device)
local SVC  = "ae9a2e94-5878-408b-882d-3c8ea1ed7169"
local CMD  = "60a99508-94f7-4592-8f4d-8a2a5561a500"
local RSP  = "f04de0cc-38e7-4bb5-8439-b7362539669b"

-- Prepare to receive responses (sub.decode converts hex → string)
local rsp = ""
ble.on("notify", function(evt) rsp = sub.decode(evt) end)

-- 1. Connect
ble.connect(SUB)

-- 2. Enable response notifications
ble.notify(SUB, SVC, RSP)
hw.delay(300)

-- 3. Send command (sub.send handles hex encoding internally)
sub.send(SUB, SVC, CMD, "return hw.free_heap()")
hw.delay(500)
log.write("RSP: " .. rsp)   -- → "OK:165680"

-- 4. Start / stop scripts
sub.lua_run(SUB, SVC, CMD, "myapp")
hw.delay(3000)
sub.lua_stop(SUB, SVC, CMD)
hw.delay(500)

-- 5. Disconnect
ble.disconnect(SUB)
```

> **Important**: Do not pass text directly to `ble.write()` — it will fail.  
> Always use `sub.send()`, which handles hex encoding internally.

---

### Retrieve Node logs (sub.get_log)

Fetches all lines written by `log.write()` on the Node in one call.

```lua
ble.connect(SUB)
ble.notify(SUB, SVC, RSP)
hw.delay(300)

-- !! Always reset to no-op BEFORE get_log (skipping this causes an infinite hang)
ble.on("notify", function() end)

local logs = sub.get_log(SUB, SVC, CMD, 8000)
log.write("Node logs:\n" .. (logs or "failed"))
ble.disconnect(SUB)
```

---

### Read a file from the Node (sub.read_file)

Transfers a file from Node SPIFFS to the Bridge.

```lua
ble.connect(SUB)
ble.notify(SUB, SVC, RSP)
hw.delay(300)

-- !! Reset to no-op before read_file as well
ble.on("notify", function() end)

local content, err = sub.read_file(SUB, SVC, CMD, "/luna_log.txt", 10000)
if content then
  log.write("File content:\n" .. content)
else
  log.write("Error: " .. tostring(err))
end
ble.disconnect(SUB)
```

---

### Check Node status via Beacon scan (no connection needed)

The Node constantly broadcasts its status over BLE advertising (ADV Beacon).  
You can check it without connecting.

```lua
ble.on("scan", function(dev)
  local b = ble.parse_luna_beacon(dev.data)
  if b then
    log.write(string.format(
      "Node: heap=%dKB lua=%s paired=%s log_count=%d",
      b.heap_kb, tostring(b.lua_running), tostring(b.paired), b.log_count))
  end
end)
ble.scan({ duration=3000, active=true })  -- active=true is mandatory
hw.delay(3500)
```

> **`active=true` is required.** Without it, Beacon data in the Scan Response is not received.

| Field | Meaning |
|-------|---------|
| `heap_kb` | Node free memory (KB) |
| `lua_running` | Whether a script is currently running |
| `paired` | Whether the Node is paired with a Bridge |
| `fw_version` | Node firmware version |
| `log_count` | Unread log lines (0 = no connection needed) |

---

### Power-efficient log retrieval using log_count

Connect only when `log_count > 0` to avoid unnecessary BLE connections.

```lua
local SVC = "ae9a2e94-5878-408b-882d-3c8ea1ed7169"
local CMD = "60a99508-94f7-4592-8f4d-8a2a5561a500"
local RSP = "f04de0cc-38e7-4bb5-8439-b7362539669b"

ble.on("scan", function(dev)
  local b = ble.parse_luna_beacon(dev.data)
  if b and b.log_count > 0 then
    ble.scan_stop()
    ble.connect(dev.addr)
    ble.notify(dev.addr, SVC, RSP)
    hw.delay(300)
    ble.on("notify", function() end)  -- no-op required
    local logs = sub.get_log(dev.addr, SVC, CMD, 8000)
    log.write(logs or "")
    ble.disconnect(dev.addr)
  end
end)
ble.scan({ duration=5000, active=true })
```

---

## 3. Node Network Management (multiple devices & pairing)

From v8.9.7, multiple Nodes can be managed as a "network."  
The Bridge stores `/sub_network.json` on SPIFFS with each Node's MAC address.  
**Once paired and registered, you can connect instantly without scanning.**

---

### /sub_network.json structure

```json
{
  "subs": [
    {"name": "sensor1", "addr": "ac:a7:04:fd:16:71", "fw": "1.0.4", "note": "temp/humidity"},
    {"name": "relay1",  "addr": "aa:bb:cc:dd:ee:ff", "fw": "1.0.4", "note": "relay control"}
  ]
}
```

---

### Node network management API

| Function | Description |
|----------|-------------|
| `sub.net_load()` | Load `/sub_network.json` and return as JSON string (returns `{"subs":[]}` if not found) |
| `sub.net_save(json)` | Save JSON string to `/sub_network.json` |
| `sub.pair(addr, svc, pair_uuid)` | Send "ENTER" to PAIR characteristic to execute pairing |
| `sub.forget(addr, svc, pair_uuid)` | Send "FORGET" to PAIR characteristic to remove pairing |

**PAIR UUID**: `c0b11fea-1607-4952-b598-bb57e9ef4453`

---

### First-time pairing + DB registration (run once)

```lua
local SVC  = "ae9a2e94-5878-408b-882d-3c8ea1ed7169"
local PAIR = "c0b11fea-1607-4952-b598-bb57e9ef4453"

-- Step 1: Scan for an unpaired Node
local found_addr = nil
ble.on("scan", function(dev)
  local b = ble.parse_luna_beacon(dev.data)
  if b and not b.paired then
    found_addr = dev.addr   -- use dev.addr (not dev.address)
    ble.scan_stop()
  end
end)
ble.scan({ duration=5000, active=true })
hw.delay(3500)

-- Step 2: Connect, pair, and register
if found_addr then
  ble.connect(found_addr)
  hw.delay(500)
  local rsp = sub.pair(found_addr, SVC, PAIR)
  -- → "OK:PAIRED rssi=-29" (success) / "ERR:TOO_FAR rssi=N" (too far)
  log.write("pair: " .. tostring(rsp))

  if rsp and rsp:sub(1,3) == "OK:" then
    local db = json.parse(sub.net_load()) or {subs={}}
    table.insert(db.subs, {name="node1", addr=found_addr, fw="1.0.4", note=""})
    sub.net_save(json.encode(db))
    log.write("Registered: " .. found_addr)
  end
  ble.disconnect(found_addr)
end
```

> **Place the Node within 50 cm of the Bridge when pairing.**  
> If you get `ERR:TOO_FAR`, move closer and retry.

---

### Control all registered Nodes (no scan needed)

```lua
local SVC = "ae9a2e94-5878-408b-882d-3c8ea1ed7169"
local CMD = "60a99508-94f7-4592-8f4d-8a2a5561a500"
local RSP = "f04de0cc-38e7-4bb5-8439-b7362539669b"

local db = json.parse(sub.net_load())
for _, s in ipairs(db.subs) do
  log.write("Connecting: " .. s.name .. " (" .. s.addr .. ")")
  ble.connect(s.addr)
  ble.notify(s.addr, SVC, RSP)
  hw.delay(300)
  ble.on("notify", function() end)  -- no-op required
  local logs = sub.get_log(s.addr, SVC, CMD, 5000)
  if logs then log.write(s.name .. ": " .. logs) end
  ble.disconnect(s.addr)
  hw.delay(300)
end
```

---

### Remove pairing

```lua
local SVC  = "ae9a2e94-5878-408b-882d-3c8ea1ed7169"
local PAIR = "c0b11fea-1607-4952-b598-bb57e9ef4453"
local target = "ac:a7:04:fd:16:71"

ble.connect(target)
hw.delay(500)
local rsp = sub.forget(target, SVC, PAIR)
log.write("forget: " .. tostring(rsp))  -- → "OK:FORGOTTEN"
ble.disconnect(target)
```

---

## 4. What the Node can do — Sensors, GPIO, ADC

The Node is a compact ESP32-S3 microcontroller. The following hardware can be controlled via Lua scripts.

### GPIO (digital I/O)

```lua
-- Output: configure and toggle
gpio.output_pin(1)         -- configure output slot 1 (returns GPIO pin number)
gpio.write(38, 1)          -- GPIO38 HIGH (ON)
gpio.write(38, 0)          -- GPIO38 LOW (OFF)

-- Input: configure and read
gpio.mode(39, 0)           -- set GPIO39 as input (0=INPUT, 1=OUTPUT, 2=INPUT_PULLUP)
local val = gpio.read(39)  -- returns 0 or 1
```

**Use cases**: relay control, LED on/off, switch reading, motor direction

> **Note**: `gpio.input_pin()` does not exist. Use `gpio.mode(pin, 0)` + `gpio.read(pin)` for input.

---

### ADC (analog input — read voltage as a number)

```lua
adc.atten(4, 3)              -- set GPIO4 to 0–3.1 V range (3 = 11 dB)
local mv = adc.read_mv(4)    -- read voltage in millivolts (e.g. 1650 = 1.65 V)
local raw = adc.read(4)      -- raw ADC value (0–4095)
log.write("Voltage: " .. mv .. " mV")
```

**Supported pins**: GPIO1–10 (ESP32-S3 ADC1)  
**Use cases**: light sensors, distance sensors, voltage monitoring, potentiometers

> **Note**: ADC2 (GPIO11+) conflicts with WiFi and cannot be used.  
> DAC (analog output) is not available on ESP32-S3. Use `gpio.pwm()` as a pseudo-analog substitute.

---

### PWM (pseudo-analog output / motor speed control)

```lua
gpio.pwm(38, 1000, 128)  -- GPIO38: 1 kHz, 50% duty cycle (0–255)
gpio.pwm(38, 1000, 0)    -- duty 0% (off)
gpio.pwm_stop(38)        -- stop PWM and release channel
```

**Use cases**: LED dimming, DC motor speed, servo control

---

### Temperature & Humidity Sensor (DHT22)

```lua
local temp, humi = sensor.dht22()   -- default pin: GPIO5
if temp ~= -999.0 then
  log.write(string.format("Temp: %.1f C  Humidity: %.0f%%", temp, humi))
else
  log.write("Sensor read error")
end
```

**Wiring**: connect DHT22 to GPIO5. Returns `-999.0` on error — always check.

---

### NeoPixel RGB LED (GPIO48)

```lua
hw.rgb(255, 0, 0)                  -- red
hw.rgb(0, 255, 0)                  -- green
hw.rgb(0, 0, 0)                    -- off
hw.rgb_blink(255, 200, 0, 1000)    -- yellow blink for 1 s (blocking)
hw.rgb_flag(1, 0, 255, 0)          -- green non-blocking blink (start)
hw.rgb_flag(0)                     -- stop blink
```

---

### Firmware info & Device ID

```lua
hw.version()    -- returns firmware version string (e.g. "1.0.4")
hw.license()    -- returns MIT license info with author and GitHub URL
hw.device_id()  -- returns EFuse MAC as 12-char HEX (e.g. "ACA704FD1671")
```

Example — check from the Bridge:

```lua
sub.send(SUB, SVC, CMD, "return hw.version()")   -- → OK:1.0.4
sub.send(SUB, SVC, CMD, "return hw.license()")   -- → OK:LUNA Node v1.0.4 / License: MIT ...
sub.send(SUB, SVC, CMD, "return hw.device_id()") -- → OK:ACA704FD1671
```

---

### WiFi configuration (for WIFI_DEFAULT_ON=0 environments)

The Node starts with **WiFi OFF** in production mode. You can connect on demand.

```lua
-- Save credentials only (no connection, non-blocking)
wifi.set_config("your_ssid", "your_password")

-- Connect using saved credentials (blocks up to 15 s)
wifi.on()

-- Disconnect
wifi.off()

-- Check connection status
local s = wifi.status()
-- s.connected = true/false,  s.ip = IP address string
```

> **Initial setup**: call `wifi.set_config()` to save → `wifi.on()` to verify → thereafter `wifi.on()` alone is sufficient.

---

### SPIFFS format (initialize after first flash)

Use this when SPIFFS is uninitialized on a fresh ESP32, or to wipe all files cleanly.

```lua
fs.format()   -- erase entire SPIFFS and remount → returns OK: on success
```

> ⚠️ **All files are deleted**, including `/luna_pair.txt`.  
> **Re-pairing is required** after `fs.format()`.  
> Not needed during normal operation.

---

### Sending Node sensor data to the Bridge

Write results with `log.write()` in a Node Lua script, then retrieve with `sub.get_log()` from the Bridge.

```lua
-- Node script (e.g. /lua_sensor.lua)
local temp, humi = sensor.dht22()
if temp ~= -999.0 then
  log.write(string.format("T=%.1f H=%.0f", temp, humi))
end
-- log.save() is not needed — log.write() alone updates Beacon log_count
```

```lua
-- Bridge side: retrieve logs
ble.on("notify", function() end)  -- no-op required
local logs = sub.get_log(SUB, SVC, CMD, 8000)
print(logs)  -- "T=25.5 H=60" etc.
```

---

## 5. Troubleshooting

### Node not found

| Check | Action |
|-------|--------|
| Is the Node powered on? | Check LED status |
| Is the Bridge close enough? | Move within 1 m |
| Is another device connected? | Node allows only 1 connection at a time — disconnect other clients first |
| Is `active=true` specified in scan? | Beacon data requires `active=true` |

### Commands sent but no response

| Check | Action |
|-------|--------|
| Is a script currently running? | Send `STOP` first, then `RUN:name` |
| Is the Node paired? | If `ERR:NOT_PAIRED` is returned, perform pairing first |

### Notify not arriving / get_log hanging on Bridge Lua

| Check | Action |
|-------|--------|
| Did you call `ble.notify()`? | `ble.on("notify", cb)` alone is not enough — you must also call `ble.notify(addr, svc, RSP)` |
| Did you add a delay after notify? | Add `hw.delay(300)` after `ble.notify()` |
| Did you set no-op before `sub.get_log` / `sub.read_file`? | Forgetting this causes the callback to consume queue entries, resulting in an infinite hang. Always call `ble.on("notify", function() end)` immediately before. |

### Cannot reconnect after lua_stop

`sub.lua_stop()` stops the Lua task but does **not** disconnect BLE.  
A subsequent `ble.connect()` to the same address will hang.

**Fix**: call `ble.disconnect()` before `sub.lua_stop()`.

---

## 6. Current Limitations and Roadmap

### Implemented ✅ (confirmed working in v8.9.7 / v1.0.4)

| Feature | Status |
|---------|--------|
| Bridge → Node script deploy (send_file) | ✅ |
| Bridge → Node script start (lua_run) | ✅ |
| Bridge → Node script stop (lua_stop) | ✅ |
| Node → Bridge log retrieval (get_log / GETLOG) | ✅ |
| Node → Bridge file transfer (read_file / READFILE) | ✅ |
| Node print() → Bridge real-time streaming (OUT: Notify) | ✅ |
| Node status check (Beacon scan / parse_luna_beacon) | ✅ |
| Beacon log_count (check log availability without connecting) | ✅ |
| Node network management (net_load / net_save / pair / forget) | ✅ |
| json.encode nested table support | ✅ |
| hw.version / hw.license / hw.device_id | ✅ |
| wifi.set_config (save credentials, non-blocking) | ✅ |
| fs.format (SPIFFS full erase and remount) | ✅ |

---

### Known limitations

#### 🟡 Medium priority — needed for stable production use

| Missing feature | Description |
|----------------|-------------|
| **Auto-reconnect** | No built-in mechanism to reconnect automatically when BLE drops |
| **Connection timeout detection** | Timeout handling when Node is unresponsive depends on the Lua script |
| **Auto-disconnect after lua_stop** | `lua_stop` stops the Lua task but BLE connection remains open (manual `ble.disconnect()` required) |

#### 🟢 Low priority — nice to have

| Missing feature | Description |
|----------------|-------------|
| **BLE security** | Communication is not encrypted. Any device that knows the UUIDs can connect. |
| **Node reset command** | A `RESET` BLE command to reboot the Node remotely |
| **Beacon Byte 19** | Notify type / payload field is Reserved (future expansion) |

---

## 7. UUID Cheat Sheet

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  LUNA Node v1.0.4 — BLE UUIDs
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Device name: LUNA_Sub

  Service:
  ae9a2e94-5878-408b-882d-3c8ea1ed7169

  CMD (Write — send commands):
  60a99508-94f7-4592-8f4d-8a2a5561a500

  RSP (Notify — receive responses):
  f04de0cc-38e7-4bb5-8439-b7362539669b

  INFO (Read — device information):
  55b29db3-c1aa-408b-86de-b07e489ddeec

  PAIR (Write+Notify — pairing control):
  c0b11fea-1607-4952-b598-bb57e9ef4453

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Text commands (send via sub.send())

  STOP              → OK:stopped
  RUN:name          → OK:started:name / ERR:...
  GETLOG            → LOG:line × N → LOG:END
  READFILE:path     → FSTART:size / FDATA:... / FEND:
  (anything else)   → executed as inline Lua → OK:result / ERR:msg

  PAIR Characteristic
  ENTER             → OK:PAIRED rssi=N / ERR:TOO_FAR rssi=N
  FORGET            → OK:FORGOTTEN / ERR:NOT_PAIRED

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  RSP Notify prefixes

  OK:result         success (stopped / started:name / exec result)
  ERR:msg           error (e.g. ERR:NOT_PAIRED)
  BUSY              Lua is running, command rejected
  OUT:text          print() streaming (inline Lua only)
  LOG:line          one line from GETLOG
  LOG:END           GETLOG terminator (log buffer cleared)
  FSTART:size       READFILE start
  FDATA:...         READFILE chunk
  FEND:             READFILE complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

*LUNA project reference document — for questions, ask Claude or NotebookLM with this guide loaded.*
