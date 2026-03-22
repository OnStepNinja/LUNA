# LUNA Lua Guide

**Target Firmware**: AiBridgeMCP v3.9.1 LUNA / v5.9.1 LUNA Observatory
**Document Revision**: r1.8
**Last Updated**: 2026-03-21
**Author**: Nishioka Sadahiko
**Audience**: Users who want to control external devices via Lua scripts on ESP32 through Claude AI

---

> **[Important Notice]** This software is distributed in **binary form** under the MIT License (free of charge, commercial use permitted). Source code is not publicly available.

---

## Revision History

| Rev | Date | Changes |
|-----|------|---------|
| r1.0 | 2026-03-17 | Initial release (Phase 0–5 complete, all MCP tools verified on hardware) |
| r1.1 | 2026-03-18 | Added log.read / log.save / log.load (Phase 6/6b) — inter-script data passing |
| r1.2 | 2026-03-18 | Enhanced explanation of log module's two roles; clarified log.load() behavior; added fs_read verification method |
| r1.3 | 2026-03-18 | Confirmed LUNA system brand name; added introductory description; introduced guide versioning |
| r1.4 | 2026-03-18 | Added explanation of two serial communication paths, conflict rules, and direct Claude serial control |
| r1.5 | 2026-03-18 | Added luna module (v3.9.1), script chain execution, new Section 2-5 |
| r1.6 | 2026-03-20 | Fixed notify.set() message argument (bug fix); updated all code examples to pass message string; added polling instructions (Desktop / Claude Code) |
| r1.7 | 2026-03-20 | Renamed lua_load → lua_save (all code examples and API reference updated); converted TOC links to plain text |
| r1.8 | 2026-03-21 | v5.9.1 support: added http module (http.get / http.put), AiBridge/Alpaca integration, new Section 2-6; existing 2-6–2-9 renumbered to 2-7–2-10 |
| r1.9 | 2026-03-22 | Added Section 3-8: Writing MCP Resource Scripts (v5.9.1) |

---

## Table of Contents

- What is the LUNA System?
- Introduction
- Chapter 1 — Quick Start
  - 1-1. What You Need
  - 1-2. Running Your First Script
  - 1-3. Basic Workflow
  - 1-4. Saving and Reusing Scripts
- Chapter 2 — Lua API Reference
  - 2-1. serial module — Serial Communication
  - 2-2. log module — Logging & Inter-Script Data Passing
  - 2-3. hw module — Hardware Control
  - 2-4. notify module — Completion Notification
  - 2-5. luna module — Script Chain Execution (v3.9.1)
  - 2-6. http module — HTTP Communication & Alpaca Integration (v5.9.1)
  - 2-7. Standard Lua 5.1 Libraries
  - 2-8. Numbers, Hex, and Bitwise Operations (Hardware Verified)
  - 2-9. Unsupported Features
  - 2-10. Quick Reference
- Chapter 3 — Practical Script Collection
  - 3-1. Device Communication Check
  - 3-2. Periodic Measurement (Multiple Times)
  - 3-3. Conditional Control
  - 3-4. Measurement with Error Handling
  - 3-5. LED as Activity Indicator
  - 3-6. Sequential Multi-Command Execution
  - 3-7. Pseudo-Concurrent Processing with Coroutines
  - 3-8. MCP Resources — Writing Resource Scripts (v5.9.1)
- Chapter 4 — Tips and Precautions
  - 4-1. Basic Principles for Working with Claude
  - 4-2. Tips for log.write
  - 4-3. Timeout Guidelines
  - 4-4. Common Issues and Solutions
  - 4-5. Script Naming Rules
  - 4-6. Concurrent Execution
- Chapter 5 — Interactive Programming with Claude
  - 5-1. What Is This?
  - 5-2. Basic Workflow
  - 5-3. Create Scripts by Handing Over the Manual
  - 5-4. Tips for Success
  - 5-5. Comparison with Traditional Programming
  - 5-6. Summary of What LUNA Can Do
- Appendix A — MCP Tool Reference (Tools Used by Claude)
- Appendix B — Lua Language Minimal Reference
- Appendix C — Passing Data from Lua to Claude (Hardware Verified)
- Appendix D — Hardware Sample: Takahashi Temma2 Telescope Control

---

> ⚠️ **[Preliminary Release]**
> This document is currently under development. Content may change without notice.
> Please use it as a reference until the official version is released.

---

## What is the LUNA System?

**LUNA** is an integrated system combining an **MCP server + Lua scripting engine running on ESP32**.

```
┌─────────────────────────────────────────┐
│              LUNA System                │
│                                         │
│   MCP Server          Lua Engine        │
│   ───────────   +   ───────────────     │
│   Communicates        Executes          │
│   with Claude         scripts on ESP32  │
│                                         │
│              ESP32                      │
└─────────────────────────────────────────┘
        ↕ WiFi (MCP)          ↕ Serial
     Claude (AI)          External devices / sensors
```

Claude simply **hands a script** to LUNA.
LUNA then **autonomously** controls the device and notifies Claude upon completion.

| Item | Details |
|------|---------|
| Official Name | LUNA System (AiBridgeMCP v3.9.1 LUNA Edition) |
| Platform | ESP32 (microcontroller, approx. $3–5) |
| Scripting Language | Lua 5.1 |
| Communication Protocol | MCP (Model Context Protocol) over HTTP |
| License | MIT (free, including commercial use) |

---

## Introduction

With LUNA, you can run **Lua scripts** inside the ESP32.

In previous versions, Claude sent commands one by one directly.
With LUNA, Claude can hand a script to the ESP32 and let it **execute autonomously**.

```
[Before] Claude → send command → device → Claude → next command… (repeated round-trips)

[LUNA]   Claude → hand script → ESP32 executes autonomously → notifies completion → Claude checks result
```

Claude can generate scripts for you.
Even without knowledge of Lua, you can use it by just describing what you want in plain English.

---

## Chapter 1 — Quick Start

### 1-1. What You Need

- ESP32 (with AiBridgeMCP v3.9.0 firmware flashed)
- Claude or another MCP-compatible AI client
- An external device to control (RS-232C connection)

### 1-2. Running Your First Script

Just tell Claude what you want:

> "Blink the LED 5 times"

Claude will generate and execute a script like this:

```lua
for i = 1, 5 do
    hw.led(1)
    hw.delay(200)
    hw.led(0)
    hw.delay(200)
end
log.write("Blinking complete")
notify.set("done")
```

### 1-3. Basic Workflow

```
① Tell Claude what you want to do
      ↓
② Claude generates a Lua script
      ↓
③ Execute with lua_exec or lua_save + lua_run
      ↓
④ Check completion with lua_status
      ↓
⑤ Check results with lua_log
```

### 1-4. Saving and Reusing Scripts

Once a script works, you can save it with a name.

```
Tell Claude: "Save this script with the name 'volt_check'"

Next time: Just say "Run volt_check"
```

Saved scripts persist even when power is off.

---

## Chapter 2 — Lua API Reference

### 2-1. serial module — Serial Communication

#### Two Communication Paths

In the LUNA system, there are **two independent paths** for serial communication with external devices.

```
[Path ①] Claude controls directly (MCP tools)

  Claude
    ↓ serial_write / serial_read / serial_query (MCP tools)
    ↓
  ESP32 C++ layer (executeSerialQuery, etc.)
    ↓ serialMutex exclusive control
  External device (Serial2)


[Path ②] Lua script controls (Lua bindings)

  Claude
    ↓ lua_exec / lua_run
  Lua script
    ↓ serial.write / serial.read / serial.query (Lua API)
    ↓
  ESP32 C++ layer (same functions as above)
    ↓ serialMutex exclusive control
  External device (Serial2)
```

**Both paths share the same C++ functions internally, protected by `serialMutex`.**

---

#### Usage Guidelines

| Use Case | Recommended Path | Reason |
|----------|-----------------|--------|
| Verification, debugging, single command | **① Claude direct** | Try immediately, no Lua needed |
| Manual device operation while reading manual | **① Claude direct** | Interactive workflow |
| Repeated measurements, periodic execution | **② Lua** | Autonomous operation, no Claude needed |
| Multi-step protocols | **② Lua** | State can be maintained between steps |
| Processing that persists after power-off | **② Lua** + log.save | State can be persisted to SPIFFS |

---

#### Conflict Handling

`serialMutex` **prevents physical data corruption (byte errors, etc.)**.
However, **logical conflicts** require attention.

```
Bad example: Claude interrupts while a Lua script is running

  Lua script  → "INIT\r"     → device
  Claude      → "STATUS?\r"  → device  ← interruption!
  device      → "OK"         → unclear which one this responds to
  Lua script  → "MEAS?\r"    → device
  device      → data         → Lua cannot receive the correct response
```

**Rule: Do not use `serial_query` while a Lua script is running (`lua_status` is `running`).**

---

#### Controlling Devices Directly with Claude (without Lua)

Without writing Lua scripts, Claude can directly control external devices using MCP tools.
**Best for prototyping, verification, and one-off operations.**

**Available MCP tools (called directly by Claude):**

| Tool | Action |
|------|--------|
| `serial_write` | Send a command (does not wait for response) |
| `serial_read` | Read the receive buffer |
| `serial_query` | Send and receive in one operation (★recommended) |

**Usage examples:**

```
User: "Send *IDN? to the device and tell me the model name"

Claude:
  Runs serial_query("*IDN?\r\n", 1000)
  → Receives "KEITHLEY INSTRUMENTS,MODEL 2110,..."
  → "It's a Keithley 2110 Digital Multimeter"
```

```
User: "What is the current position of Temma2?"

Claude:
  Runs serial_query("Q\r", 2000)
  → Receives "0535173-052328"
  → "Current position: RA 5h35m17.3s / Dec -5°23'28""
```

**Comparison with Lua:**

| Item | Claude direct (MCP) | Lua script |
|------|---------------------|------------|
| Setup | None (use immediately) | Script creation required |
| Repetition | Claude calls each time | Autonomous loop possible |
| Complex processing | Claude decides and proceeds | Written in script, runs autonomously |
| Operation without Claude | Not possible | Possible (lua_run) |
| Best for | Debugging, interactive, exploration | Periodic measurement, autonomous control |

> 💡 **Typical workflow**: First verify operation with Claude direct → then graduate to a Lua script.

---

#### `serial.query(data, timeout_ms)` ★Recommended

Sends data and receives a response.

| Argument | Type | Description |
|----------|------|-------------|
| data | string | String to send (including terminator) |
| timeout_ms | number | Receive timeout in milliseconds |

| Return | Type | Description |
|--------|------|-------------|
| response | string | Received response. Empty string `""` on timeout |

```lua
-- Basic usage
local resp = serial.query("*IDN?\r\n", 1000)
if resp ~= "" then
    log.write("Response: " .. resp)
else
    log.write("Timeout")
end

-- Receive as number
local raw = serial.query(":MEAS:VOLT:DC?\r\n", 2000)
local val = tonumber(raw)
if val then
    log.write(string.format("Voltage: %.4f V", val))
end
```

---

#### `serial.write(data)`

Sends data without waiting for a response.

| Argument | Type | Description |
|----------|------|-------------|
| data | string | String to send |

```lua
serial.write("START\r\n")
hw.delay(100)
local resp = serial.read()
```

---

#### `serial.read()`

Reads data from the receive buffer.

| Return | Type | Description |
|--------|------|-------------|
| data | string | Received data (empty string if buffer is empty) |

> **Note**: Timing-dependent. Use `serial.query()` when a reliable response is required.

---

### 2-2. log module — Logging & Inter-Script Data Passing

#### Two Roles of the log Module

The log module is the **information hub** of the LUNA system. It's not just a debug log — it has two independent purposes.

```
[Role ①] Script → Claude data transfer

  Lua script
    log.write("Measurement: 1.234")
         ↓  Ring buffer in RAM (latest 20 lines)
  Claude
    lua_log tool → retrieves all 20 lines at once
    → uses results for confirmation and next instructions

[Role ②] Script → Script data passing (zero Claude involvement)

  Script A
    log.write("CAL=1.0023")
    log.save()  → persists to SPIFFS /luna_log.txt
         ↓  Survives power-off, different sessions, time gaps
  Script B
    log.load()  → reads /luna_log.txt
    log.read()  → retrieves "CAL=1.0023" for processing
```

| Use Case | Write | Read | Persistence |
|----------|-------|------|-------------|
| Script → Claude | `log.write()` | `lua_log` (MCP tool) | RAM only (volatile) |
| Script → Script (same session) | `log.write()` | `log.read()` | RAM only (volatile) |
| Script → Script (across power-off) | `log.write()` + `log.save()` | `log.load()` + `log.read()` | SPIFFS (persistent) |
| Claude directly checks SPIFFS | `log.save()` | `fs_read("/luna_log.txt")` | SPIFFS (persistent) |

> 💡 **Practical value**: Using `log.save()` / `log.load()`, calibration scripts, measurement scripts, and analysis scripts can work together autonomously without Claude. Periodic scripts can also resume from the previous state.

---

#### `log.write(message)`

Records a message in the log buffer. Claude retrieves this content with the `lua_log` tool.

| Argument | Type | Description |
|----------|------|-------------|
| message | string | Message to record |

```lua
log.write("Process started")
log.write(string.format("Measurement: %.3f V", value))
log.write("Error: no response")
```

> **Limit**: Ring buffer retains the **latest 20 lines**.
> Lines beyond 21 overwrite the oldest entries.

---

#### `log.read([n])` — Read log lines (Phase 6 / 6b)

Returns one line from the ring buffer. Without arguments, returns the latest line (backward compatible). An index argument provides access to any specific line.

| Argument | Type | Description |
|----------|------|-------------|
| n | number (optional) | Omit/-1=latest, 0=oldest, 1,2,...=from oldest, -2,-3,...=from latest going back |

| Return | Type | Description |
|--------|------|-------------|
| line | string / nil | The line. `nil` if buffer is empty or index out of range |

```lua
log.read()      -- latest line (no argument, same as before)
log.read(-1)    -- latest line (same)
log.read(0)     -- oldest line
log.read(1)     -- second oldest
log.read(-2)    -- second from latest
log.read(99)    -- out of range → nil
```

> ✅ **Verified on hardware (2026-03-18)**
> ```
> -- After log.write("LINE0") through log.write("LINE3")
> read()=LINE3 | read(0)=LINE0 | read(1)=LINE1 | read(-1)=LINE3 | read(-2)=LINE2 | read(99)=nil
> ```

#### Example ① Use only the latest line (simplest)

Write the value to pass as the **last `log.write()`**, then retrieve it with `log.read()` (no argument).

```lua
-- Script A: write the value to pass at the end
log.write("TS=" .. hw.millis())     -- additional info
log.write("CAL=1.0023")            -- ← value to pass, written last
log.save()
notify.set("done")

-- Script B: retrieve the latest line with log.read() (zero Claude involvement)
log.load()
local cal = tonumber((log.read() or ""):match("CAL=(.+)")) or 1.0
local v = tonumber(serial.query(":MEAS?\r\n", 1000)) * cal
return string.format("%.4f", v)
```

#### Example ② Scan all lines to find a specific key

Search for a value by key name without worrying about write order.

```lua
log.load()
local function find(key)
    for i = 0, 19 do
        local line = log.read(i)
        if not line then break end
        local v = line:match(key .. "=(.+)")
        if v then return v end
    end
    return nil
end

local cal    = tonumber(find("CAL"))    or 1.0
local offset = tonumber(find("OFFSET")) or 0.0
log.write(string.format("cal=%.4f offset=%.4f", cal, offset))
```

#### Example ③ Extract multiple values from the last two lines

```lua
log.load()
local line1 = log.read(-1)   -- latest
local line2 = log.read(-2)   -- previous
-- Example: "VOLT=1.234" and "TEMP=25.6" stored
local volt = tonumber((line1 or ""):match("VOLT=(.+)"))
local temp = tonumber((line2 or ""):match("TEMP=(.+)"))
return string.format("V=%.3f T=%.1f", volt or 0, temp or 0)
```

---

#### `log.save()` — Persist to SPIFFS (Phase 6)

Writes the ring buffer contents to `/luna_log.txt` in SPIFFS. Use for state persistence across power-off.

| Return | Type | Description |
|--------|------|-------------|
| result | boolean | `true` on success, `false` on failure |

```lua
log.write("calibration=1.0023")
local ok = log.save()
if ok then
    log.write("Saved successfully")
end
```

> ⚠️ **Flash lifetime warning**: Design `log.save()` to be called **only once at script end**.
> Calling it repeatedly inside a loop will drastically increase SPIFFS write cycles, shortening flash lifetime.

---

#### `log.load()` — Restore state from SPIFFS (Phase 6)

Reads `/luna_log.txt` contents into the ring buffer. Restores the state saved by the previous script.

| Return | Type | Description |
|--------|------|-------------|
| result | boolean | `true` on success, `false` if file not found or failed |

> ⚠️ **Important**: `log.load()` **clears the ring buffer first**, then reads the file. Any content written to the buffer before calling `log.load()` will be lost. Always call it at the **beginning of the script**.

```lua
-- Correct pattern: call load() at the start of the script
local ok = log.load()
if ok then
    local last = log.read()
    log.write("Previous state: " .. tostring(last))
else
    log.write("First run (no saved data)")
end
-- Write your processing code after this

-- Wrong pattern (do not do this)
log.write("Process started")   -- this line will be erased by load()
local ok = log.load()          -- ← buffer is cleared here
```

#### Checking /luna_log.txt Directly from Claude

Files saved with `log.save()` can be read directly by Claude using the `fs_read` tool. Useful for reviewing results from autonomously executed scripts.

```
Claude operation:
  fs_read("/luna_log.txt")
  → retrieves all lines saved by the script
```

This enables an asynchronous workflow where "scripts autonomously accumulate data, and Claude retrieves it later."

---

### 2-3. hw module — Hardware Control

#### `hw.delay(ms)`

Stops processing for the specified time.

| Argument | Type | Description |
|----------|------|-------------|
| ms | number | Wait time in milliseconds |

```lua
hw.delay(500)     -- wait 0.5 seconds
hw.delay(1000)    -- wait 1 second
hw.delay(60000)   -- wait 1 minute
```

> **Important characteristic**: `hw.delay()` only pauses the Lua task.
> The ESP32 HTTP server is not blocked — Claude's operations are still accepted during the wait.

---

#### `hw.millis()`

Returns elapsed time in milliseconds since ESP32 boot.

| Return | Type | Description |
|--------|------|-------------|
| ms | number | Milliseconds since boot |

```lua
local start = hw.millis()
-- processing
local elapsed = hw.millis() - start
log.write(string.format("Processing time: %d ms", elapsed))
```

---

#### `hw.free_heap()`

Returns current free heap memory in bytes.

| Return | Type | Description |
|--------|------|-------------|
| bytes | number | Free heap size in bytes |

```lua
log.write("Free heap: " .. hw.free_heap() .. " bytes")

-- Memory shortage check
if hw.free_heap() < 10000 then
    log.write("Warning: low memory")
    notify.set("low memory")
    return
end
```

---

#### `hw.led(state)`

Controls the ESP32's onboard LED.

| Argument | Type | Description |
|----------|------|-------------|
| state | number | `1` to turn on, `0` to turn off |

```lua
hw.led(1)        -- on
hw.delay(200)
hw.led(0)        -- off
```

```lua
-- Blinking example
for i = 1, 5 do
    hw.led(1)
    hw.delay(100)
    hw.led(0)
    hw.delay(100)
end
```

> ⚠️ **Note**: `hw.led(true)` / `hw.led(false)` (boolean) will cause an error.
> Always use numeric `0` or `1`.

---

### 2-4. notify module — Completion Notification

#### `notify.set()`

Sets a flag to notify Claude that the script has completed.

```lua
notify.set()
```

> **Recommended**: Always include this at the end of long-running scripts.
> When Claude detects this flag, it automatically retrieves `lua_log` to check the results.

```lua
-- Typical script structure
log.write("Process started")

-- Main processing
local last_val = ""
for i = 1, 10 do
    local v = serial.query(":MEAS:VOLT?\r\n", 1000)
    log.write(string.format("%d: %s", i, v))
    last_val = v
    hw.delay(1000)
end

log.write("Process complete")
notify.set("measurement complete: last value " .. last_val)    -- ← pass a message
```

#### How to check notifications (polling)

LUNA notifications use a polling model. Claude receives the message when it calls `check_notify`.

**Claude Desktop (standard use)**:

After running a script, just tell Claude in plain language:

```
"check notify"
```

**Claude Code (advanced users)**:

Use the `/loop` skill for automatic periodic checking:

```
/loop 1m check_notify を呼んで flag=1 なら報告して
```

→ Checks every minute and reports immediately if a notification is found.

---

### 2-5. luna module — Script Chain Execution (v3.9.1)

#### `luna.run(name)` — Auto-launch the next script

Automatically starts the specified script after the current script finishes.
Allows multiple scripts to run sequentially **without Claude's involvement**.

| Argument | Type | Description |
|----------|------|-------------|
| name | string | Name of the next script to run (must be saved with `lua_save`) |

| Return | Type | Description |
|--------|------|-------------|
| result | boolean | `true` on success, `false` if chain limit exceeded |

```lua
-- End of script "init"
log.write("Initialization complete")
log.save()
luna.run("measure")   -- auto-launch "measure" after this ends
notify.set("done")
```

#### How It Works: "Queue and Exit" Design

Calling `luna.run()` **does not stop the current script immediately**. After the script finishes completely, the ESP32 automatically loads and starts the next script.

```
Claude
  ↓ calls lua_run("init") just once
ESP32 (autonomous operation)
  init runs → luna.run("measure") → init ends
  measure runs → luna.run("analyze") → measure ends
  analyze runs → notify.set("done") → ends
  ↓
Notification to Claude (entire sequence complete)
```

#### Chain Limit: Maximum 10 Scripts

To prevent infinite loops, chains are limited to a maximum of 10 scripts (the first + 9 more).

```lua
-- Bad example (infinite loop)
-- Script "a": luna.run("b")
-- Script "b": luna.run("a")  ← automatically stops at 10th
-- "chain limit reached" is recorded in log
```

When the limit is reached:
- `luna.run()` returns `false`
- `"luna.run: chain limit (10) reached"` is recorded in log
- The current script continues (no crash)

#### Practical Example: Observation Sequence

```lua
-- Saved with lua_save("obs_init", ...)
log.write("obs: initialization started")
-- Device initialization processing
log.write("obs: initialization complete")
log.save()
luna.run("obs_measure")   -- proceed to measurement script
notify.set("done")

-- Saved with lua_save("obs_measure", ...)
log.load()
log.write("obs: measurement started")
for i = 1, 5 do
    local v = serial.query(":MEAS?\r\n", 1000) or "err"
    log.write(string.format("obs: %d=%s", i, v))
    hw.delay(10000)
end
log.save()
luna.run("obs_finish")    -- proceed to finalization script
notify.set("done")

-- Saved with lua_save("obs_finish", ...)
log.load()
log.write("obs: sequence complete")
notify.set("done")   -- notify Claude of final completion
```

> ⚠️ **Note**: Scripts launched with `luna.run()` must be saved in advance with `lua_save`. If a non-existent script name is specified, an error is recorded in log and the chain terminates.

---

### 2-6. http module — HTTP Communication & Alpaca Integration (v5.9.1)

> **New in v5.9.1** — Added in LUNA Observatory.

Sends HTTP requests to devices on the LAN.
Ideal for integrating with **AiBridge (ASCOM/Alpaca compatible)** and any Alpaca REST device.

#### `http.get(url, timeout_ms)` ★ Recommended (status/read)

```lua
local body = http.get(url, timeout_ms)
```

| Parameter | Description |
|-----------|-------------|
| `url` | Target URL (HTTP only — HTTPS not supported) |
| `timeout_ms` | Timeout in ms (default 2000ms if omitted) |
| Return value | Response body string. Returns `""` on error or timeout |

```lua
-- Read temperature from Alpaca observingconditions (AiBridge)
local t = http.get("http://192.168.3.7/api/v1/observingconditions/0/temperature", 1000)
if t == "" then return '{"error":"no response"}' end
return '{"temp":' .. t .. '}'
```

#### `http.put(url, body, timeout_ms)` — Send command

```lua
local resp = http.put(url, body, timeout_ms)
```

| Parameter | Description |
|-----------|-------------|
| `url` | Target URL |
| `body` | Request body (`application/x-www-form-urlencoded` format) |
| `timeout_ms` | Timeout in ms (default 2000ms if omitted) |
| Return value | Response body string. Returns `""` on error |

```lua
-- Turn on Alpaca switch port 0 (AiBridge switch)
local r = http.put(
  "http://192.168.3.7/api/v1/switch/0/setswitch",
  "Id=0&State=true",
  1000
)
return r ~= "" and "OK" or "error"
```

#### AiBridge Alpaca Integration Examples

**Expose weather sensor as a Resource (lua_resource_weather.lua)**

```lua
local temp = http.get("http://192.168.3.7/api/v1/observingconditions/0/temperature", 1000)
local hum  = http.get("http://192.168.3.7/api/v1/observingconditions/0/humidity",    1000)
if temp == "" then return '{"error":"no response from AiBridge"}' end
return string.format('{"temp":%s,"humidity":%s}', temp, hum)
```

**Expose switch port states as a Resource (lua_resource_switch.lua)**

```lua
local s0 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=0", 1000)
local s1 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=1", 1000)
local s2 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=2", 1000)
local s3 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=3", 1000)
return string.format('{"port0":%s,"port1":%s,"port2":%s,"port3":%s}', s0, s1, s2, s3)
```

#### Recommended Timeout Values

| Use case | Recommended timeout |
|----------|-------------------|
| Alpaca GET (sensor / status read) | 1000ms |
| Alpaca PUT (command send) | 2000ms |
| AiBridge `/api/external/discover` | 4000ms (longer due to UDP wait) |

#### Notes

- **HTTPS not supported** (LAN HTTP only)
- **Blocking operation** — the Lua VM pauses during the request. Keep timeouts short
- Returns `""` on error or timeout — always check the return value
- Like `log.save()`, avoid calling repeatedly in tight loops (causes memory fragmentation)
- No AiBridge firmware changes needed — it works as an Alpaca REST server as-is

#### Hardware Verified (2026-03-21)

| Test | Response time |
|------|--------------|
| AiBridgeMCP Console (192.168.4.1) | 35ms ✅ |
| AiBridge Console (192.168.3.7) | 153ms ✅ |
| switch/maxswitch → Value:4 | ✅ |
| switch getswitch Id=0–3 → all false | 315ms ✅ |
| observingconditions/temperature → -999.0 | ✅ |

---

### 2-7. Standard Lua 5.1 Libraries

The following standard libraries are available.

| Library | Main Functions |
|---------|---------------|
| `string.*` | format, find, sub, len, rep, upper, lower |
| `math.*` | floor, ceil, sqrt, abs, max, min, pi |
| `table.*` | insert, remove, concat, sort |
| `coroutine.*` | create, resume, yield, status, wrap |
| Basic functions | tonumber, tostring, type, print, pcall, ipairs, pairs |

```lua
-- Common combination
local resp = serial.query(":MEAS:VOLT?\r\n", 1000)
local val = tonumber(resp)
if val then
    log.write(string.format("%.4f V", val))
else
    log.write("Conversion failed: [" .. tostring(resp) .. "]")
end
```

---

### 2-8. Numbers, Hex, and Bitwise Operations (Hardware Verified)

#### Number Types

All numbers in Lua 5.1 are stored as **double (double-precision floating point)**.
There is no distinction between integers and reals — both are the same `number` type.

```lua
local a = 42          -- used as integer
local b = -7
local f = 3.14159     -- real number

log.write(a + b)      -- → 35
log.write(a * b)      -- → -294
log.write(math.floor(a / 5))  -- → 8 (use math.floor for integer division)
```

#### Hexadecimal (Hardware Verified ✅)

```lua
-- Hex literals (0x prefix)
local h1 = 0xFF       -- → 255
local h2 = 0x1A       -- → 26
local h3 = 0xDEAD     -- → 57005

-- Decimal → hex string
string.format("0x%X",   255)   -- → "0xFF"
string.format("0x%04X", 1234)  -- → "0x04D2" (4 digits zero-padded)
string.format("%02X",   10)    -- → "0A"

-- Hex string → number
tonumber("FF",   16)   -- → 255
tonumber("1A2B", 16)   -- → 6699
tonumber("dead", 16)   -- → 57005 (lowercase accepted)
```

#### Bitwise Operations (Hardware Verified ❌)

The `bit` library is **not available** in this version.
Use the following pure Lua implementations as alternatives.

```lua
-- AND alternative
local function band(a, b)
    local result = 0
    local bit = 1
    while a > 0 and b > 0 do
        if a % 2 == 1 and b % 2 == 1 then result = result + bit end
        a = math.floor(a / 2)
        b = math.floor(b / 2)
        bit = bit * 2
    end
    return result
end

-- OR alternative
local function bor(a, b)
    local result = 0
    local bit = 1
    while a > 0 or b > 0 do
        if a % 2 == 1 or b % 2 == 1 then result = result + bit end
        a = math.floor(a / 2)
        b = math.floor(b / 2)
        bit = bit * 2
    end
    return result
end

-- Left shift alternative
local function lshift(a, n)
    return a * (2 ^ n)
end

-- Right shift alternative
local function rshift(a, n)
    return math.floor(a / (2 ^ n))
end

-- Usage examples
log.write(band(0xFF, 0x0F))    -- → 15
log.write(bor(0xF0, 0x0F))     -- → 255
log.write(lshift(1, 4))        -- → 16
log.write(rshift(0xFF, 4))     -- → 15
```

> 💡 **Tip**: If a device responds in hex format, use `tonumber(resp, 16)` to convert to a number.
> For use cases with heavy bitwise operations (binary protocols, etc.), ask Claude to include the alternative functions in the script.

---

### 2-9. Unsupported Features

| Feature | Result | Reason |
|---------|--------|--------|
| `io.*` (file I/O) | ❌ | SPIFFS access is done via MCP tools |
| `os.*` (OS operations) | ❌ | Not supported in embedded environment |
| `require()` (external modules) | ❌ | Not supported in single-script environment |
| Network communication (HTTP, TCP, etc.) | ❌ | Not implemented in this version |
| `bit.*` (bitwise library) | ❌ | Not implemented (pure Lua alternative available) |
| Multiple simultaneous scripts | ❌ | One script at a time (coroutines can simulate concurrency) |

---

### 2-10. Quick Reference

```lua
-- Serial communication
serial.write(data)                  -- send only
serial.read()                       -- receive only
serial.query(data, timeout_ms)      -- send → receive (recommended)

-- Logging
log.write(message)                  -- record log (retains latest 20 lines)
log.read([n])                       -- read line (omit/-1=latest, 0=oldest, negative=from end)
log.save()                          -- ring buffer → SPIFFS /luna_log.txt (once at end only)
log.load()                          -- SPIFFS /luna_log.txt → ring buffer

-- Hardware
hw.delay(ms)                        -- wait (does not block HTTP server)
hw.millis()                         -- milliseconds since boot
hw.free_heap()                      -- free heap in bytes
hw.led(1 or 0)                      -- onboard LED control (number only)

-- Notification
notify.set("done")                  -- notify Claude of completion

-- Script chain (v3.9.1)
luna.run(name)                      -- auto-launch name after current script ends (max 10)

-- HTTP communication / Alpaca integration (v5.9.1)
http.get(url, timeout_ms)           -- HTTP GET. Returns response body ("" on error)
http.put(url, body, timeout_ms)     -- HTTP PUT. Returns response body ("" on error)
```

---

## Chapter 3 — Practical Script Collection

### 3-1. Device Communication Check

```lua
-- Check if the device responds
local resp = serial.query("*IDN?\r\n", 2000)
if resp ~= "" then
    log.write("Device OK: " .. resp)
else
    log.write("Error: no response (check cable and baud rate)")
end
notify.set("done")
```

---

### 3-2. Periodic Measurement (Multiple Times)

```lua
-- Measure 10 times at 1-second intervals and log results
local count = 10
log.write(string.format("Measurement started (%d times)", count))

local sum = 0
for i = 1, count do
    local raw = serial.query(":MEAS:VOLT:DC?\r\n", 2000)
    local val = tonumber(raw) or 0
    sum = sum + val
    log.write(string.format("[%d/%d] %.6f V", i, count, val))
    if i < count then hw.delay(1000) end
end

local avg = sum / count
log.write(string.format("Average: %.6f V", avg))
log.write(string.format("Elapsed: %d ms", hw.millis()))
notify.set("measurement complete")
```

---

### 3-3. Conditional Control

```lua
-- Change next operation based on response content
local status = serial.query(":STAT?\r\n", 1000)
log.write("Status: " .. status)

if string.find(status, "READY") then
    log.write("Ready → starting measurement")
    local val = serial.query(":MEAS?\r\n", 2000)
    log.write("Value: " .. val)
elseif string.find(status, "BUSY") then
    log.write("Busy → retrying in 2 seconds")
    hw.delay(2000)
    local val = serial.query(":MEAS?\r\n", 2000)
    log.write("Value (retry): " .. val)
else
    log.write("Unknown status: " .. status)
end

notify.set("done")
```

---

### 3-4. Measurement with Error Handling

```lua
-- Use pcall to catch errors and execute safely
local ok, err = pcall(function()
    for i = 1, 5 do
        local resp = serial.query(":MEAS:VOLT?\r\n", 1500)
        if resp == "" then
            error("Timeout (attempt " .. i .. ")")
        end
        local val = tonumber(resp)
        if not val then
            error("Number conversion failed: [" .. resp .. "]")
        end
        log.write(string.format("%d: %.4f V", i, val))
        hw.delay(1000)
    end
end)

if ok then
    log.write("All measurements complete")
else
    log.write("Error occurred: " .. tostring(err))
end
notify.set("done")
```

---

### 3-5. LED as Activity Indicator

```lua
-- Blink LED during processing, fast blink 3x on completion
log.write("Process started")

-- Processing indicator (5 blinks while measuring)
for i = 1, 5 do
    hw.led(1)
    local resp = serial.query(":MEAS?\r\n", 1000)
    log.write(string.format("%d: %s", i, resp))
    hw.led(0)
    hw.delay(500)
end

-- Completion signal (3 fast blinks)
for i = 1, 3 do
    hw.led(1)
    hw.delay(100)
    hw.led(0)
    hw.delay(100)
end

log.write("Process complete")
notify.set("done")
```

---

### 3-6. Sequential Multi-Command Execution

```lua
-- Automate: initialize → configure → measure → shutdown
local function send(cmd, wait_ms)
    wait_ms = wait_ms or 500
    local resp = serial.query(cmd, wait_ms)
    log.write(cmd:gsub("\r\n","") .. " → " .. (resp ~= "" and resp or "OK"))
    return resp
end

log.write("=== Initialization sequence started ===")
send("*RST\r\n", 2000)        -- reset
hw.delay(1000)
send(":CONF:VOLT:DC\r\n")     -- set DC voltage mode
send(":VOLT:RANG 10\r\n")     -- set range
send(":TRIG:SOUR IMM\r\n")    -- set trigger
hw.delay(500)

log.write("=== Measurement started ===")
for i = 1, 3 do
    local val = send(":READ?\r\n", 2000)
    hw.delay(1000)
end

log.write("=== Complete ===")
notify.set("done")
```

---

### 3-7. Pseudo-Concurrent Processing with Coroutines

Only one script can run at a time in v3.9.0, but **Lua coroutines** allow a single script to simulate concurrent processing by alternating between multiple tasks.

#### What are Coroutines?

A mechanism that pauses processing midway (`yield`) and switches to another task.
Not true parallel execution, but effective for "advancing two tasks alternately."

```
Task A: step 1 → yield → step 2 → yield → step 3 → done
Task B:           step 1 → yield → step 2 → yield → step 3 → done
```

#### Basic Usage (Hardware Verified)

```lua
-- Example: two tasks running alternately
local function task_a()
    for i = 1, 3 do
        log.write("Task A: step " .. i)
        coroutine.yield()       -- pause here, return control
    end
    log.write("Task A: done")
end

local function task_b()
    for i = 1, 3 do
        log.write("Task B: step " .. i)
        coroutine.yield()
    end
    log.write("Task B: done")
end

-- Create coroutines
local co_a = coroutine.create(task_a)
local co_b = coroutine.create(task_b)

-- Scheduler: run alternately
while true do
    local a_alive = coroutine.status(co_a) ~= "dead"
    local b_alive = coroutine.status(co_b) ~= "dead"
    if not a_alive and not b_alive then break end
    if a_alive then coroutine.resume(co_a) end
    if b_alive then coroutine.resume(co_b) end
end

log.write("All tasks complete")
notify.set("done")
```

**Output:**
```
Task A: step 1
Task B: step 1
Task A: step 2
Task B: step 2
Task A: step 3
Task B: step 3
Task A: done
Task B: done
All tasks complete
```

#### Practical Example: Simultaneous Measurement and LED Blinking

```lua
-- Blink LED while measuring
local function led_blinker(count)
    for i = 1, count do
        hw.led(1)
        coroutine.yield()
        hw.led(0)
        coroutine.yield()
    end
end

local function measure(count)
    local results = {}
    for i = 1, count do
        local v = serial.query(":MEAS:VOLT?\r\n", 1000)
        log.write(string.format("[%d] %s V", i, v))
        coroutine.yield()
        hw.delay(500)
        coroutine.yield()
    end
    log.write("Measurement complete")
end

local co_led  = coroutine.create(function() led_blinker(10) end)
local co_meas = coroutine.create(function() measure(5) end)

while true do
    local l = coroutine.status(co_led)  ~= "dead"
    local m = coroutine.status(co_meas) ~= "dead"
    if not l and not m then break end
    if l then coroutine.resume(co_led)  end
    if m then coroutine.resume(co_meas) end
end

notify.set("measurement complete")
```

#### Coroutine Function Reference

| Function | Description |
|----------|-------------|
| `coroutine.create(f)` | Create a coroutine from function f |
| `coroutine.resume(co)` | Start or resume a coroutine |
| `coroutine.yield()` | Pause and return control to caller |
| `coroutine.status(co)` | Return status (`"running"` / `"suspended"` / `"dead"`) |
| `coroutine.wrap(f)` | Return a wrapper that calls resume as a function |

> **Note**: Calling `hw.delay()` inside a coroutine also pauses all other coroutines.
> The key to pseudo-concurrency is finding the right combination of `yield` and `delay`.

---

### 3-8. MCP Resources — Writing Resource Scripts (v5.9.1)

> **📖 Details**: See `LUNA_Observatory_Resources_Guide_EN.md`.

#### What Are Resources?

LUNA Observatory (v5.9.1) lets you publish Lua scripts as "gauges" that Claude can read naturally.
When a user asks "Where is the telescope pointing?", instead of calling `serial_query` twice, Claude simply reads `luna://lua/mount` once and answers immediately.

```
[Scripts in 3-1 to 3-7]   Run with lua_exec / lua_run → perform an action
[Resource scripts in 3-8]  Save once → auto-published → Claude reads them
```

#### Naming Convention

| File name | URI (the name Claude uses) |
|-----------|--------------------------|
| `lua_resource_mount.lua` | `luna://lua/mount` |
| `lua_resource_weather.lua` | `luna://lua/weather` |
| `lua_resource_switch.lua` | `luna://lua/switch` |

Any file saved to SPIFFS with the `lua_resource_` prefix is **automatically published as a Resource**. No firmware changes required.

#### Basic Template

```lua
-- lua_resource_{name}.lua
-- ① Retrieve data from device
local value = serial.query("command", 100)

-- ② Error check (required)
if value == "" then
  return '{"error":"no response"}'
end

-- ③ Return a JSON string via return (required)
return string.format('{"key":"%s"}', value)
```

**Key conventions:**

| Item | Rule |
|------|------|
| Return value | Must return a JSON string via `return` |
| On error | Return `'{"error":"message"}'` |
| Side effects | Read-only recommended — avoid commands that move the device |
| Execution time | Keep short (100–200ms recommended for `serial.query`) |
| `log.save()` | Do **not** call inside resource scripts (flash lifetime) |

#### Example ① RS-232C Device (LX200-compatible mount)

```lua
-- lua_resource_mount.lua
local ra  = serial.query(":GR#", 100)
local dec = serial.query(":GD#", 100)
if ra == "" or dec == "" then
  return '{"error":"no response from mount"}'
end
return string.format('{"ra":"%s","dec":"%s"}', ra, dec)
```

#### Example ② AiBridge (Alpaca weather sensor)

```lua
-- lua_resource_weather.lua (uses http module)
local temp = http.get("http://192.168.3.7/api/v1/observingconditions/0/temperature", 1000)
local hum  = http.get("http://192.168.3.7/api/v1/observingconditions/0/humidity",    1000)
if temp == "" then return '{"error":"no response from AiBridge"}' end
return string.format('{"temp":%s,"humidity":%s}', temp, hum)
```

#### Example ③ AiBridge (Switch port status)

```lua
-- lua_resource_switch.lua
local s0 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=0", 1000)
local s1 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=1", 1000)
local s2 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=2", 1000)
local s3 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=3", 1000)
return string.format('{"port0":%s,"port1":%s,"port2":%s,"port3":%s}', s0, s1, s2, s3)
```

#### How to Save

Just tell Claude to save the script:

```
"Save the following Lua script with the name resource_mount"
(paste the script content)
```

→ `lua_save("resource_mount", ...)` runs and saves `/lua_resource_mount.lua`
→ `luna://lua/mount` automatically appears in Resources

#### Using from Claude Desktop

Once saved, Resources work naturally in conversation:

```
User: "Where is the telescope pointing?"
Claude: reads luna://lua/mount → answers with RA/Dec immediately

User: "Can we observe tonight?"
Claude: reads luna://lua/weather too → gives a combined assessment
         including temperature and humidity
```

---

## Chapter 4 — Tips and Precautions

### 4-1. Basic Principles for Working with Claude

**① Start with verification**

```
Tell Claude: "Check if the device responds with *IDN?"
→ Claude verifies directly with serial_query
→ Once response is confirmed, proceed to Lua scripts
```

**② Test small, then expand**

```
Bad approach: build a complex script all at once
Good approach: test 1 command → loop 3 times → add conditionals — build step by step
```

**③ Give Claude the manual and it will find the commands**

```
Tell Claude:
"Read this manual (PDF) and confirm the voltage measurement command,
 then create a script that measures 5 times at 1-second intervals"
```

---

### 4-2. Tips for log.write

Make the most of the 20-line limit:

```lua
-- Bad: writing every iteration overflows during large-scale measurement
for i = 1, 100 do
    log.write("Measurement: " .. serial.query(":MEAS?\r\n", 1000))
end

-- Good: aggregate every 10 iterations
local sum = 0
for i = 1, 100 do
    sum = sum + (tonumber(serial.query(":MEAS?\r\n", 1000)) or 0)
    if i % 10 == 0 then
        log.write(string.format("[%d/100] subtotal avg: %.4f", i, sum/i))
    end
end
log.write(string.format("Final average: %.4f", sum/100))
```

---

### 4-3. Timeout Guidelines

| Device Type | Recommended Timeout |
|-------------|---------------------|
| Fast-responding devices (instruments, etc.) | 500–1000 ms |
| Devices with physical motion (telescopes, etc.) | 2000–5000 ms |
| Initialization / reset commands | 2000–5000 ms |
| Unknown devices | Start with 2000 ms |

---

### 4-4. Common Issues and Solutions

| Symptom | Cause | Solution |
|---------|-------|----------|
| `serial.query` returns empty string | Timeout or wrong terminator | Increase timeout / change `\r\n` to `\r` |
| `tonumber()` returns nil | Extra whitespace or newlines in response | Remove with `string.gsub(resp, "%s+", "")` |
| Script hangs | Infinite loop or long `hw.delay` | Force stop with `lua_stop` tool |
| Concurrent execution error | Another script already running | Check with `lua_status` → wait for completion |
| Script not found | Wrong name or not saved | Check saved list with `lua_list` |

---

### 4-5. Script Naming Rules

| Rule | Details |
|------|---------|
| Allowed characters | Alphanumeric, underscore `_`, hyphen `-` |
| Maximum length | **22 characters** (SPIFFS constraint) |
| Case sensitivity | Case-sensitive |
| Subdirectories | **Not allowed** (slash `/` cannot be used) |

```
Good: volt_check, led_test, temp_monitor, my-script-01
Bad:  volt check (space not allowed), lua/test (slash not allowed)
```

> **⚠️ Important: Subdirectories cannot be created**
>
> SPIFFS is a flat filesystem — there is no directory concept.
> All scripts are saved at the same level.
>
> ```
> /lua_led_test.lua       ← all flat
> /lua_volt_avg.lua
> /lua_measure01.lua
> ```

#### Recommended Naming Convention for Categories

As the number of scripts grows, use **prefixes to indicate categories**:

```
serial_check      ← serial communication verification
serial_volt       ← serial communication
measure_temp      ← measurement
measure_volt_avg  ← measurement
led_blink         ← LED control
led_test          ← LED control
scope_trigger     ← oscilloscope
temma_goto        ← telescope control
```

When displayed with `lua_list`, scripts will group naturally by category.

---

### 4-6. Concurrent Execution

Only **one script** can run at a time in v3.9.0.

```lua
-- For multiple operations, combine them in one script
-- Operation A
local v = serial.query(":MEAS:VOLT?\r\n", 1000)
log.write("Voltage: " .. v)

-- Operation B
local t = serial.query(":MEAS:TEMP?\r\n", 1000)
log.write("Temperature: " .. t)

notify.set("done")
```

---

## Chapter 5 — Interactive Programming with Claude

### 5-1. What Is This?

AiBridgeMCP v3.9.0 is more than just a control tool.
It's an **interactive programming environment for the AI era** — where you can have a conversation with Claude, watch results on screen, and build, run, and refine scripts on the spot.

You don't need to know Lua. **Just describe what you want** and you can use it.

---

### 5-2. Basic Workflow

#### Step ① Talk to the device first

Don't overthink it — start by asking Claude to verify.

```
User:    "Check if the connected device responds.
          The command is *IDN? with CR+LF terminator."

Claude:  (verifies with serial_query)
          → Response: KEITHLEY INSTRUMENTS, MODEL 2110
          → Communication is working normally.
```

> Key point: test just the first command. If it works, move forward.

---

#### Step ② Describe what you want

Once communication is verified, describe what you want to do.

```
User:    "Measure voltage 5 times at 1-second intervals and show the results"

Claude:  I'll run the following script:

         for i = 1, 5 do
             local v = serial.query(":MEAS:VOLT:DC?\r\n", 2000)
             log.write(string.format("[%d] %.4f V", i, tonumber(v) or 0))
             hw.delay(1000)
         end
         notify.set("done")

         → Running... (5 seconds later)
         → [1] 1.2345 V
         → [2] 1.2347 V
         → ...
```

---

#### Step ③ Review results and request changes

If you see something to adjust, just describe it.

```
User:    "Increase to 10 times and also show the average at the end"

Claude:  I'll modify and re-run it.
         (generates modified version → executes immediately)

User:    "Timeouts occur occasionally"

Claude:  I'll change timeout from 1000ms → 2000ms
         and add error handling.
         (generates modified version → executes immediately)
```

> Key point: Don't worry about errors. Claude will explain the cause and fix it.

---

#### Step ④ Save when it works

Once you're satisfied with the results, save it with a name.

```
User:    "Save this as 'volt_check'"

Claude:  lua_save("volt_check", ...) → saved
         Next time, just say "run volt_check"
         and it will execute immediately.
```

---

#### Step ⑤ Call it by name next time

Saved scripts persist even when power is off.

```
User:    "Run volt_check"

Claude:  lua_run("volt_check") → executing
         → showing results
```

---

### 5-3. Create Scripts by Handing Over the Manual

Even without knowing device commands, just give the manual to Claude.

```
User:    "Here is the manual for this instrument (PDF).
          Create a script that alternately measures temperature and voltage
          5 times each and logs the results in CSV format"

Claude:  I've reviewed the manual.
         Temperature is :MEAS:TEMP?, voltage is :MEAS:VOLT:DC?.
         How about this script:

         log.write("time_ms,type,value")
         for i = 1, 5 do
             local t = serial.query(":MEAS:TEMP?\r\n", 2000)
             local v = serial.query(":MEAS:VOLT:DC?\r\n", 2000)
             local ms = hw.millis()
             log.write(ms .. ",TEMP," .. (t or "ERR"))
             log.write(ms .. ",VOLT," .. (v or "ERR"))
             hw.delay(2000)
         end
         notify.set("done")

         Shall I test it?
```

> **This is the greatest strength of this system.**
> Users don't need to memorize commands.
> Just hand over the manual and describe what you want —
> Claude generates, executes, and revises the script for you.

---

### 5-4. Tips for Success

#### ① Don't try to do everything at once

```
Bad:  "Initialize, configure, measure 100 times, do statistics,
       detect anomalies, and trigger an alarm"
      → If built all at once, it's hard to tell where it fails

Good: Start with "can it measure once?"
      → Then "measure 10 times"
        → Then "add statistics"
          → Build incrementally
```

#### ② Errors are opportunities

```
When an error occurs:
"This error appeared. Tell me the cause and fix it"

Claude will:
  - Explain the cause in plain language
  - Generate a fixed version
  - Re-execute
```

Even without programming knowledge, Claude handles debugging.

#### ③ Save as soon as it works

```
Save even when it "kind of works"
→ Later you can say "restore the script from that time"
```

#### ④ Log is the "window" to results

`log.write()` is the means of sharing information with Claude.
By leaving records at key points in the script,
Claude can accurately track how far execution has progressed.

```lua
log.write("=== Started ===")         -- record start
log.write("Initialization complete")
log.write("Value: " .. val)          -- record important values
log.write("=== Complete ===")        -- record end
notify.set("done")
```

#### ⑤ Use LED to confirm "it's alive"

For long-running scripts, blinking the LED lets you know
whether it's running or stopped — without looking at the screen.

```lua
-- Light the LED on each iteration
for i = 1, 10 do
    hw.led(1)
    local v = serial.query(":MEAS?\r\n", 1000)
    log.write(v)
    hw.led(0)
    hw.delay(1000)
end
```

---

### 5-5. Comparison with Traditional Programming

| Item | Traditional Programming | LUNA System |
|------|------------------------|-------------|
| Writing code | Write it yourself | Claude generates it |
| Execution environment | IDE / compiler required | Just talk to Claude |
| Error handling | Debug yourself | Claude explains and fixes |
| Device spec research | Read manual yourself | Hand manual to Claude |
| Checking results | Terminal / screen | Check instantly with `lua_log` |
| Save and reuse | File management | Save with a name in one sentence |
| Required knowledge | Must know Lua syntax and API | **Just describe what you want** |

---

### 5-6. Summary of What LUNA Can Do

```
Talk to Claude  →  Claude writes the script
                        ↓
                   Execute on ESP32
                        ↓
                   Check results with lua_log
                        ↓
Talk to Claude  →  Claude modifies and improves
                        ↓
                   Save with a name when satisfied
                        ↓
                   Call with one word next time
```

> **"Even without programming knowledge,**
> **if you can describe what you want, you can automate device control."**
>
> This is what the LUNA system aims to be.

---

## Appendix A — MCP Tool Reference (Tools Used by Claude)

| Tool | Description |
|------|-------------|
| `lua_exec` | Execute Lua code directly (for testing) |
| `lua_status` | Check the state of the running script |
| `lua_stop` | Force stop the running script |
| `lua_log` | Retrieve log buffer contents |
| `lua_save` | Save a script to SPIFFS with a name |
| `lua_run` | Execute a saved script by name |
| `lua_list` | List all saved scripts |
| `serial_query` | Claude sends and receives serial commands directly |

---

## Appendix B — Lua Language Minimal Reference

> This reference is a minimal Lua 5.1 language guide focused on what's needed for AiBridgeMCP.
> The goal is to read Claude-generated scripts and understand them, or to make small changes yourself.

---

### B-1. Variables and Types

Declare variables with `local`. No type declaration needed.

```lua
local a = 42          -- number (integer)
local b = 3.14        -- number (decimal)
local s = "hello"     -- string
local s2 = 'world'    -- string (single quotes also work)
local flag = true     -- boolean (true / false)
local nothing = nil   -- no value (undefined / deleted)
```

**Type checking:**
```lua
type(42)        -- → "number"
type("hello")   -- → "string"
type(true)      -- → "boolean"
type(nil)       -- → "nil"
```

---

### B-2. Operators

**Arithmetic operators:**

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `+` | addition | `3 + 2` | `5` |
| `-` | subtraction | `3 - 2` | `1` |
| `*` | multiplication | `3 * 2` | `6` |
| `/` | division | `7 / 2` | `3.5` |
| `%` | remainder | `7 % 2` | `1` |
| `^` | exponent | `2 ^ 8` | `256` |

```lua
local x = math.floor(7 / 2)   -- integer division → 3
```

**Comparison operators:**

| Operator | Meaning |
|----------|---------|
| `==` | equal |
| `~=` | not equal (use `~=`, not `!=`) |
| `<` / `>` | less than / greater than |
| `<=` / `>=` | less than or equal / greater than or equal |

```lua
10 == 10    -- → true
10 ~= 5     -- → true (inequality uses ~=)
```

**Logical operators:**

| Operator | Meaning |
|----------|---------|
| `and` | logical AND |
| `or` | logical OR |
| `not` | logical NOT |

```lua
true and false   -- → false
true or false    -- → true
not true         -- → false
```

**String concatenation:**

```lua
"Hello" .. " " .. "World"   -- → "Hello World"
"Value: " .. 42             -- → "Value: 42"
```

> ⚠️ Numbers are auto-converted when concatenated with strings,
> but use `tostring(42)` to be safe.

---

### B-3. Conditionals (if)

```lua
if condition then
    -- when condition is true
elseif another_condition then
    -- when another_condition is true
else
    -- when none match
end
```

**Example:**
```lua
local val = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000))

if val == nil then
    log.write("Error: number conversion failed")
elseif val > 5.0 then
    log.write("High voltage: " .. val .. " V")
elseif val > 1.0 then
    log.write("Normal: " .. val .. " V")
else
    log.write("Low voltage: " .. val .. " V")
end
```

**Common patterns:**
```lua
-- nil check
if val then
    log.write("Has value: " .. val)
end

-- check if string contains something
if string.find(resp, "OK") then
    log.write("Response OK")
end
```

---

### B-4. Loops (for / while)

**Numeric for (most common):**

```lua
for variable = start, end do
    -- processing
end

for variable = start, end, step do
    -- processing
end
```

```lua
-- 1 to 5
for i = 1, 5 do
    log.write("i = " .. i)
end

-- 10 down to 1 (reverse)
for i = 10, 1, -1 do
    log.write(i)
end

-- every 2
for i = 0, 10, 2 do
    log.write(i)    -- 0, 2, 4, 6, 8, 10
end
```

**while (repeat while condition is true):**

```lua
while condition do
    -- processing
end
```

```lua
local count = 0
while count < 5 do
    log.write("count = " .. count)
    count = count + 1
end
```

**break (exit loop early):**

```lua
for i = 1, 100 do
    local v = tonumber(serial.query(":MEAS?\r\n", 1000)) or 0
    log.write(v)
    if v > 10.0 then
        log.write("Threshold exceeded → stopping")
        break
    end
end
```

---

### B-5. Functions

```lua
local function functionName(arg1, arg2)
    -- processing
    return returnValue
end
```

```lua
-- Basic function
local function add(a, b)
    return a + b
end
log.write(add(3, 5))    -- → 8

-- Default argument values
local function measure(cmd, timeout)
    timeout = timeout or 1000    -- default 1000 if omitted
    return serial.query(cmd, timeout)
end

-- Multiple return values
local function min_max(t)
    local mn, mx = t[1], t[1]
    for _, v in ipairs(t) do
        if v < mn then mn = v end
        if v > mx then mx = v end
    end
    return mn, mx
end

local lo, hi = min_max({3, 1, 4, 1, 5, 9, 2, 6})
log.write("Min: " .. lo .. " Max: " .. hi)
```

---

### B-6. Tables (Used as Arrays)

Lua tables can be used as both arrays and dictionaries.
Here we cover the most common use case: arrays.

```lua
-- Create array (index starts at 1)
local data = {}
local data2 = {10, 20, 30}   -- with initial values

-- Add values
table.insert(data, 1.234)
table.insert(data, 5.678)

-- Get values
log.write(data[1])    -- → 1.234
log.write(data[2])    -- → 5.678

-- Number of elements
log.write(#data)      -- → 2

-- Process all elements
for i, v in ipairs(data) do
    log.write(string.format("[%d] %.3f", i, v))
end
```

**Practical example: accumulate measurements and calculate average**
```lua
local values = {}

for i = 1, 10 do
    local v = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000)) or 0
    table.insert(values, v)
    hw.delay(500)
end

-- Calculate average
local sum = 0
for _, v in ipairs(values) do
    sum = sum + v
end
log.write(string.format("Average: %.4f V", sum / #values))
```

---

### B-7. String Operations (Commonly Used)

```lua
-- Format (printf style)
string.format("%.4f", 3.14159)       -- → "3.1416"
string.format("%d", 255)             -- → "255"
string.format("0x%X", 255)           -- → "0xFF"
string.format("[%d/%d]", 3, 10)      -- → "[3/10]"

-- Number conversion
tonumber("3.14")                     -- → 3.14
tonumber("FF", 16)                   -- → 255 (hex)
tonumber("abc")                      -- → nil (failed)

-- Safe number conversion
local val = tonumber(resp) or 0      -- use 0 on failure

-- String search
string.find("READY OK", "OK")        -- → 7, 8 (returns position)
string.find("ERROR", "OK")           -- → nil (not found)

-- String length
string.len("hello")                  -- → 5
#"hello"                             -- → 5 (same)

-- Remove whitespace / newlines
string.gsub(resp, "%s+", "")         -- remove all whitespace
string.gsub(resp, "[\r\n]", "")      -- remove newlines

-- Case conversion
string.upper("hello")                -- → "HELLO"
string.lower("HELLO")                -- → "hello"
```

---

### B-8. Error Handling (pcall)

Use when you want to continue executing a script even if an error occurs.

```lua
local ok, err = pcall(function()
    -- code that might error
    local v = serial.query(":MEAS?\r\n", 1000)
    if v == "" then error("Timeout") end
    log.write("Value: " .. v)
end)

if ok then
    log.write("Success")
else
    log.write("Error: " .. tostring(err))
end
```

**Retry example:**
```lua
local function safe_measure(retries)
    for i = 1, retries do
        local ok, result = pcall(function()
            local v = serial.query(":MEAS?\r\n", 1000)
            if v == "" then error("Timeout") end
            return v
        end)
        if ok then return result end
        log.write("Retry " .. i .. "/" .. retries)
        hw.delay(500)
    end
    return nil
end

local val = safe_measure(3)
if val then
    log.write("Measurement: " .. val)
else
    log.write("Measurement failed (3 retries)")
end
notify.set("done")
```

---

### B-9. Common Patterns

```lua
-- ① Safely convert measurement to number
local raw = serial.query(":MEAS?\r\n", 1000)
local val = tonumber(raw) or 0

-- ② Check if response contains a specific string
if string.find(resp, "ERROR") then
    log.write("Error response: " .. resp)
end

-- ③ Repeated measurement with logging
for i = 1, 10 do
    local v = serial.query(":MEAS?\r\n", 1000)
    log.write(string.format("[%02d] %s", i, v))
    hw.delay(1000)
end

-- ④ Stop when threshold is exceeded
for i = 1, 100 do
    local v = tonumber(serial.query(":MEAS?\r\n", 1000)) or 0
    if v > 5.0 then
        log.write("Threshold exceeded: " .. v)
        break
    end
    hw.delay(500)
end

-- ⑤ Record multiple values together
local t = serial.query(":MEAS:TEMP?\r\n", 1000)
local v = serial.query(":MEAS:VOLT?\r\n", 1000)
log.write(string.format("%d,%.3f,%.3f", hw.millis(),
          tonumber(t) or 0, tonumber(v) or 0))
```

---

## Appendix C — Passing Data from Lua to Claude (Hardware Verified)

> Explains how a Lua script that has received data from an external device can pass that data to Claude.

---

### ⚠️ Important: Global Variables Are NOT Carried Between Scripts

The Lua execution environment is reset with each `lua_exec` / `lua_run` call.
Variables defined in a previous script are not visible in the next script.

```lua
-- Script ① (executed with lua_exec)
x = 42          -- assigned to global variable...

-- Script ② (executed with another lua_exec)
log.write(x)    -- → error! x is nil (previous value is gone)
```

> **Reason**: To prevent memory leaks, the Lua internal state is created and destroyed on each script execution.

#### Methods to Pass Data Between Scripts

Since v3.9.0 (Phase 6), using `log.read()` / `log.save()` / `log.load()` allows **passing data between scripts without Claude's involvement**.

| Method | Claude Needed | Survives Power-off | Use Case |
|--------|:-------------:|:------------------:|----------|
| **`log.write()` → `log.read()`** | No | ✗ (RAM only) | Pass data between scripts in same session |
| **`log.save()` → `log.load()`** | No | ✅ (SPIFFS) | Save/restore state across power-off |
| **`return` → Claude embeds in next script** | Yes | ✗ | Same session (Claude as intermediary) |
| **`return` → Claude `fs_write` → `fs_read`** | Yes | ✅ | Cross-session (Claude as intermediary) |

```
[No Claude needed — using log.save / load]

Script A (calibration script)
  local cal = ...measure...
  log.write("CAL=" .. cal)
  log.save()        ← saved to SPIFFS /luna_log.txt
  notify.set("done")
      ↓ (power-off, different session, time gap — all OK)
Script B (measurement script)
  log.load()        ← restored from /luna_log.txt
  local cal = tonumber(log.read():match("CAL=(.+)")) or 1.0
  local v = tonumber(serial.query(":MEAS?\r\n", 1000)) * cal
  return v
```

---

### C-1. Choosing the Right Data Passing Method

| Method | Capacity | Use Case | Characteristics |
|--------|----------|----------|----------------|
| **`log.write()` + `log.read()`** | Latest 20 lines | Pass values between scripts | **No Claude** — immediately available |
| **`log.save()` + `log.load()`** | Latest 20 lines | State persistence across power-off | **No Claude** — SPIFFS persistence |
| **`return`** | Unlimited (large data) | Measurements, arrays, CSV | Claude retrieves after execution |
| **`log.write()` + `return` combined** | — | Full record of long processes | Most reliable method |

---

### C-2. Passing All Measurement Data: Using `return` (★Recommended)

Writing `return` at the end of a script puts the value in `lua_status`'s `last_result.return`.
**No line limit** — large amounts of data can be passed.

#### Return a simple value

```lua
local v = serial.query(":MEAS:VOLT?\r\n", 2000)
return "Voltage: " .. v .. " V"
```

`lua_status` result:
```json
{"last_result": {"status":"ok", "return":"Voltage: 1.2345 V"}}
```

#### Return an array as a string (Hardware Verified ✅)

```lua
local results = {}
for i = 1, 5 do
    local v = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000)) or 0
    table.insert(results, v)
    hw.delay(1000)
end

local out = ""
for i, v in ipairs(results) do
    out = out .. string.format("[%d]=%.4f ", i, v)
end
return out
```

`return` content:
```
[1]=1.2355 [2]=1.2365 [3]=1.2375 [4]=1.2385 [5]=1.2395
```

#### Return in CSV format (Hardware Verified ✅)

```lua
local csv = "index,voltage,timestamp\n"
for i = 1, 10 do
    local v = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000)) or 0
    csv = csv .. string.format("%d,%.4f,%d\n", i, v, hw.millis())
    hw.delay(1000)
end
return csv
```

`return` content:
```
index,voltage,timestamp
1,1.2355,14783456
2,1.2365,14783556
...
```

> Claude can use this CSV directly for analysis, aggregation, or chart instructions.

---

### C-3. Inspect Variable Contents On the Spot

To check the state of specific variables during or after script execution,
output them explicitly with `return` or `log.write()`.

```lua
-- Return all variable states at once (Hardware Verified ✅)
local x = 42
local name = "Instrument A"
local data = {1.234, 5.678, 9.012}
local flag = true

return string.format(
    "x=%d, name=%s, flag=%s, data=[%.3f, %.3f, %.3f], heap=%d",
    x, name, tostring(flag),
    data[1], data[2], data[3],
    hw.free_heap()
)
```

`return` content:
```
x=42, name=Instrument A, flag=true, data=[1.234, 5.678, 9.012], heap=201800
```

> **Example instruction to Claude**:
> "Run the script and check the contents of the data array"
> → Claude reads the variables from the `return` content and explains them.

---

### C-4. Long-Running Processes: `log` (progress) + `return` (all data)

```lua
-- Recommended pattern: log for progress, return for final result
log.write("=== Measurement started ===")

local csv = "index,voltage\n"
for i = 1, 20 do
    local v = tonumber(serial.query(":MEAS:VOLT?\r\n", 1000)) or 0
    csv = csv .. string.format("%d,%.4f\n", i, v)

    -- log progress every 5 iterations
    if i % 5 == 0 then
        log.write(string.format("Progress: %d/20 complete", i))
    end
    hw.delay(1000)
end

log.write("=== Measurement complete ===")
notify.set("measurement complete")
return csv    -- all data via return
```

| Output | Content |
|--------|---------|
| `lua_log` | Progress reports (every 5 iterations) |
| `return` | All 20 CSV data entries |

---

### C-5. Choosing the Best Method by Use Case

```
[Small data / progress reports]
  → log.write() (latest 20 lines, can write during execution)

[Measurement results / arrays / large data]
  → return (no line limit, CSV format supported)

[Inspect variables / array contents]
  → return string.format(...) for explicit output
  → Claude reads values from lua_status

[Complete record of long processes]
  → Combine log (progress) + return (all data)
```

---

## Appendix D — Hardware Sample: Takahashi Temma2 Telescope Control

Sample scripts for controlling Takahashi's **Temma2** telescope mount with the LUNA system.
Retrieves current position and performs GoTo via RS-232C serial communication.

> **Compatible models**: Temma PC / Temma2 / Temma2 Jr. / EM-10/200/400/500 Temma2 series

---

### D-1. Connection Setup

Temma2 uses RS-232C (±12V), so a **MAX3232 level converter module** is required between Temma2 (RS-232C) and ESP32 (3.3V TTL).

```
Temma2                  MAX3232              ESP32
Mini-DIN4               module
  Pin 1 (TXD) ────── R1IN → R1OUT ──── GPIO16 (RX2)
  Pin 2 (RXD) ────── T1IN ← T1OUT ──── GPIO17 (TX2)
  Pin 3 (GND) ────── GND  ──────────── GND
  Pin 4       ─ not connected
```

| Setting | Value |
|---------|-------|
| Baud rate | **19200 bps** |
| Data bits | 8 |
| Parity | None (N) |
| Stop bits | 1 |
| Command terminator | CR (`\r`) |

> ⚠️ The **Computer Standby switch on the Temma2 body must be ON** to enable RS-232C communication.

---

### D-2. Coordinate Format

Temma2 uses a proprietary coordinate format in ASCII decimal.

#### Right Ascension (RA): `HHMMSSs` (7 digits)

| Field | Digits | Content |
|-------|--------|---------|
| HH | 2 | Hours (00–23) |
| MM | 2 | Minutes (00–59) |
| SSs | 3 | Seconds × 10 (17.3s → `173`) |

Example: `0535173` = 05h 35m 17.3s

#### Declination (Dec): `SDDMMSS` (7 digits)

| Field | Digits | Content |
|-------|--------|---------|
| S | 1 | Sign (`+` or `-`) |
| DD | 2 | Degrees (00–90) |
| MM | 2 | Minutes (00–59) |
| SS | 2 | Seconds (00–59) |

Example: `-052328` = -05° 23' 28"

---

### D-3. Script ① temma_q — Get Current Position (Communication Check)

Run this first for connection verification. Gets the current position with a single `Q\r` command.

```lua
-- temma_q: Temma2 current position (communication check)
-- Q\r → <HHMMSSs><SDDMMSS> 14 characters
local resp = serial.query("Q\r", 2000)

if resp and #resp >= 14 then
  local ra_h  = resp:sub(1, 2)
  local ra_m  = resp:sub(3, 4)
  local ra_ss = tonumber(resp:sub(5, 7)) or 0   -- seconds × 10
  local dec   = resp:sub(8, 14)

  local ra_s  = ra_ss / 10.0
  log.write(string.format("RA  = %sh %sm %.1fs", ra_h, ra_m, ra_s))
  log.write(string.format("Dec = %s", dec))
  log.write("RAW = " .. resp)
else
  log.write("ERROR: " .. (resp or "no response"))
end

notify.set("done")
return resp
```

**Run and verify:**
```
Claude operations:
  lua_save("temma_q", <above code>)
  lua_run("temma_q")
  lua_log
    → RA  = 10h 30m 45.6s
    → Dec = -321030
    → RAW = 1030456-321030
```

---

### D-4. Script ② temma_goto — GoTo (Automatic Slewing)

Slews the mount to the specified coordinates. Change `ra_target` and `dec_target` before use.

```lua
-- temma_goto: Temma2 GoTo script (LUNA v3.9.0)
--
-- Change the target coordinates before use.
-- Example: M42 (Orion Nebula) RA 05h35m17.3s  Dec -05°23'28"
--
-- RA  format: HHMMSSs (to 1/10 second. e.g. 17.3s → 173)
-- Dec format: SDDMMSS (S = sign + or -)
--
-- ⚠️ First time: make sure Computer Standby SW on mount body is ON

local ra_target  = "0535173"   -- 05h 35m 17.3s  ← change this
local dec_target = "-052328"   -- -05° 23' 28"   ← change this

-- Step 1: Get current position (communication check)
log.write("=== temma_goto start ===")
local pos = serial.query("Q\r", 2000)
if pos and #pos >= 14 then
  log.write("Current position RAW = " .. pos)
else
  log.write("WARNING: Q command no response -> " .. tostring(pos))
end

-- Step 2: Send GoTo command
-- G<HHMMSSs><SDDMMSS>\r
local cmd = "G" .. ra_target .. dec_target .. "\r"
log.write("Goto cmd = G" .. ra_target .. dec_target)
local resp = serial.query(cmd, 3000)

-- Response is ">" or no response (both may be normal)
if resp and #resp > 0 then
  log.write("Response = " .. resp)
else
  log.write("No response (slew may have started)")
end

log.write("=== temma_goto end ===")
notify.set("done")
return {current=pos, goto_resp=resp}
```

**Run and verify:**
```
Claude operations:
  lua_save("temma_goto", <above code>)  ← change ra_target / dec_target first
  lua_run("temma_goto")
  lua_log
    → === temma_goto start ===
    → Current position RAW = 1030456-321030
    → Goto cmd = G0535173-052328
    → No response (slew may have started)
    → === temma_goto end ===
```

---

### D-4a. Minimal GoTo Script — Direct RA/Dec String Input

Minimal version without communication check (Q command). Best first step for learning the protocol.

```lua
-- Temma2 minimal GoTo script
-- Specify RA/Dec directly as Temma format strings
--
-- RA  format: HHMMSSs  (seconds × 10. e.g. 45.6s → 456)
-- Dec format: SDDMMSS  (S = sign + or -)

local ra_str  = "1030456"   -- 10h 30m 45.6s
local dec_str = "-321030"   -- -32° 10' 30"

-- Send GoTo command: G<HHMMSSs><SDDMMSS>\r
local cmd = "G" .. ra_str .. dec_str .. "\r"
log.write("Send: " .. cmd:gsub("\r", "\\r"))

local resp = serial.query(cmd, 5000)   -- 5 second wait
if resp and resp ~= "" then
    log.write("Resp: " .. resp)
else
    log.write("GoTo sent (no response is normal in some cases)")
end

notify.set("done")
```

> 💡 **About Temma2 responses**: Some implementations return no response or `>` to the G command. Both are normal. Confirm slew start by the sound of the mount moving, not by the response.

---

### D-4b. GoTo Script with Coordinate Conversion — Input in Hours/Degrees

Specify RA in hours (e.g. 10.5127h) and Dec in degrees (e.g. -32.175°), and the script converts to Temma format before GoTo. You can copy coordinates directly from astronomy software or Stellarium.

```lua
-- Temma2 GoTo script with coordinate conversion
-- ra_hours : RA in hours  (e.g. 10h30m45.6s ≈ 10.512666h)
-- dec_deg  : Dec in degrees (e.g. -32°10'30" ≈ -32.175°)

-- RA (hours) → Temma HHMMSSs format
-- Integer arithmetic to avoid floating point error
local function ra_to_temma(ra_hours)
    local s10_total = math.floor(ra_hours * 36000 + 0.5)  -- integer in 0.1s units
    if s10_total < 0       then s10_total = 0 end
    if s10_total >= 864000 then s10_total = 863999 end
    local h   = math.floor(s10_total / 36000)
    local m   = math.floor((s10_total - h * 36000) / 600)
    local s10 = s10_total - h * 36000 - m * 600
    return string.format("%02d%02d%03d", h, m, s10)
end

-- Dec (degrees) → Temma SDDMMSS format
-- Integer arithmetic to avoid floating point error
local function dec_to_temma(dec_deg)
    local sign = "+"
    if dec_deg < 0 then sign = "-"; dec_deg = -dec_deg end
    if dec_deg > 90.0 then dec_deg = 90.0 end
    local arcsec = math.floor(dec_deg * 3600 + 0.5)  -- integer in arc-second units
    local d = math.floor(arcsec / 3600)
    local m = math.floor((arcsec - d * 3600) / 60)
    local s = arcsec - d * 3600 - m * 60
    return string.format("%s%02d%02d%02d", sign, d, m, s)
end

-- ★ User changes these values ★
local ra_hours = 10.512666   -- e.g. 10h 30m 45.6s
local dec_deg  = -32.175     -- e.g. -32° 10' 30"

-- Convert
local ra_str  = ra_to_temma(ra_hours)
local dec_str = dec_to_temma(dec_deg)

log.write("RA  (Temma): " .. ra_str)
log.write("Dec (Temma): " .. dec_str)

-- Send GoTo command
local cmd = "G" .. ra_str .. dec_str .. "\r"
log.write("Send: " .. cmd:gsub("\r", "\\r"))

local resp = serial.query(cmd, 5000)
if resp and resp ~= "" then
    log.write("Resp: " .. resp)
else
    log.write("GoTo sent (no response is normal in some cases)")
end

notify.set("done")
return {ra=ra_str, dec=dec_str, resp=resp}
```

**Conversion examples:**

| Input | Converted |
|-------|-----------|
| `ra_hours = 10.512666` | `"1030456"` (10h 30m 45.6s) |
| `dec_deg  = -32.175` | `"-321030"` (-32° 10′ 30″) |
| `ra_hours = 5.588139` | `"0535173"` (M42: 5h 35m 17.3s) |
| `dec_deg  = -5.391` | `"-052328"` (M42: -5° 23′ 28″) |

> 💡 **Using Claude**: Tell Claude "slew to M42" and it will look up the coordinates, calculate `ra_hours` / `dec_deg`, and execute this script with `lua_save` + `lua_run`.

---

### D-5. Quick Reference — Common Celestial Objects

Use `ra_target` / `dec_target` directly as-is.

| Object | `ra_target` | `dec_target` | Note |
|--------|-------------|--------------|------|
| M42 Orion Nebula | `"0535173"` | `"-052328"` | Winter favorite |
| M45 Pleiades | `"0347000"` | `"+240600"` | Winter favorite |
| M31 Andromeda Galaxy | `"0042443"` | `"+411609"` | Autumn favorite |
| M13 Hercules Globular | `"1641420"` | `"+362755"` | Summer favorite |
| M57 Ring Nebula (Lyra) | `"1853570"` | `"+330200"` | Summer favorite |
| Polaris (North Star) | `"0231480"` | `"+892542"` | Polar alignment check |

> ⚠️ Planets and the Moon change coordinates with time. Get current coordinates from astronomy software or Stellarium.

---

### D-6. Initialization Commands (When Needed)

Initial setup or after power cycling may require initialization if GoTo accuracy is poor.
Tell Claude "initialize Temma2", or send the following commands in order with `serial_query`.

```
1. Set latitude:  I+354600\r   (e.g. 35°46'00" N)
2. Set time:      T203045\r    (e.g. 20:30:45 JST)
3. Confirm:       E\r
```

> ⚠️ Exact response format for each command may vary by hardware.

---

### D-7. Troubleshooting

| Symptom | Cause | Solution |
|---------|-------|----------|
| `temma_q` shows "no response" | Computer Standby SW is OFF | Turn switch ON on mount body |
| Garbled response | Baud rate mismatch | LUNA firmware is fixed at 19200bps. Check no other device is connected |
| `temma_q` responds but `temma_goto` doesn't slew | Initialization not done | Run initialization commands in D-6 |
| Garbled characters at startup | ESP32 boot messages flow into Temma2 | Power on ESP32 first, then power on Temma2 |

> **Note**: Changing baud rate with `serial_config` tool is a Pro Edition feature. LUNA (free edition) is **fixed at 19200bps** — no firmware configuration changes needed.

---

### D-8. Autonomous Observation Sequence with luna.run() (v3.9.1)

Using `luna.run()`, scripts can automatically launch the next script without Claude's involvement.
The following is an example of a fully autonomous 3-stage sequence: **communication check → GoTo → position confirmation**.

```
Claude calls lua_run("temma_init") just once
  ↓
temma_init → temma_goto → temma_confirm
              (automatic)   (automatic)
  ↓
notify.set("done") after completion → notification to Claude
```

#### Script ① temma_init (communication check + parameter setup)

```lua
-- temma_init: communication check + save target coordinates to log
local resp = serial.query("Q\r", 2000)
if not resp or resp == "" then
    log.write("ERROR: Temma2 no response")
    log.write("CHECK: Turn Computer Standby SW ON")
    notify.set("error detected")
    return
end
log.write("OK: Temma2 response confirmed")
log.write("POS=" .. resp)

-- Set target coordinates (change here)
log.write("TARGET_RA=0535173")    -- M42: 05h 35m 17.3s
log.write("TARGET_DEC=-052328")   -- M42: -05° 23′ 28″

log.save()                         -- save to SPIFFS
luna.run("temma_goto")             -- auto-launch next script
```

#### Script ② temma_goto (automatic slewing)

```lua
-- temma_goto: read coordinates from log and execute GoTo
log.load()

-- find TARGET_RA / TARGET_DEC from log
local ra, dec
for i = 0, 19 do
    local line = log.read(i)
    if not line then break end
    ra  = ra  or line:match("^TARGET_RA=(.+)")
    dec = dec or line:match("^TARGET_DEC=(.+)")
end

if not ra or not dec then
    log.write("ERROR: coordinates not found")
    notify.set("error detected")
    return
end

local cmd = "G" .. ra .. dec .. "\r"
log.write("GOTO: " .. cmd:gsub("\r","\\r"))
local resp = serial.query(cmd, 5000)
log.write("RESP=" .. (resp or "none"))
log.save()

luna.run("temma_confirm")          -- proceed to confirmation script
```

#### Script ③ temma_confirm (position confirmation)

```lua
-- temma_confirm: position check after slew completes
hw.delay(3000)                     -- wait briefly for slew to complete

local pos = serial.query("Q\r", 2000)
if pos and pos ~= "" then
    log.write("ARRIVED=" .. pos)
else
    log.write("WARN: could not confirm position after slew")
end
log.save()
notify.set("done")                 -- notify Claude of completion
```

#### How to Use

```
1. Save the 3 scripts with lua_save:
   lua_save("temma_init",    <script ① above>)
   lua_save("temma_goto",    <script ② above>)
   lua_save("temma_confirm", <script ③ above>)

2. Launch the first script from Claude:
   lua_run("temma_init")

3. ESP32 runs autonomously. Check results with lua_log after completion.
```

> 💡 **Chain limit**: `luna.run()` supports up to 10 chained scripts (`LUA_MAX_CHAIN=10`).
> When the limit is reached, `"chain limit reached"` is recorded in log and no further auto-launches occur.

---

## License

The LUNA System (AiBridgeMCP v3.9.1 LUNA Edition) is released under the **MIT License**.
Free to use for both commercial and non-commercial purposes.

---

*AiBridgeMCP Project: https://github.com/OnStepNinja/AiBridgeMCP*
*Author: Nishioka Sadahiko*
