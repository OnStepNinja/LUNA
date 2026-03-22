# LUNA Observatory Resources Guide
**Version:** v5.9.1
**Audience:** LUNA Observatory Users
**Last Updated:** 2026-03-22
**Author:** Nishioka Sadahiko

---

> **[Important Notice]** This software is distributed in **binary form** under the MIT License (free of charge, commercial use permitted). Source code is not publicly available.

---

## 1. What Are Resources?

LUNA Observatory implements three capabilities of MCP (Model Context Protocol):

| Capability | Role | Analogy |
|-----------|------|---------|
| **Tools** | Claude **operates** a device | Pressing a button |
| **Prompts** | Claude **reads** a guide | Consulting a manual |
| **Resources** | Claude **reads** device state | Reading a gauge |

**Resources are "gauges."**
They allow Claude to naturally read real-time information — telescope position, dome status, weather data — during conversation.

### Difference from Tools

**With Tools only (before v5.9.1):**
```
User: "Where is the telescope pointing?"
Claude: calls serial_query(":GR#") → receives result
Claude: calls serial_query(":GD#") → receives result
→ Two tool calls required
```

**With Resources (v5.9.1):**
```
User: "Where is the telescope pointing?"
Claude: reads luna://lua/mount → answers immediately
→ One step, natural in conversation
```

---

## 2. How It Works

LUNA Observatory Resources are **defined by Lua scripts**.

```
SPIFFS (ESP32 internal storage)
    └── lua_resource_mount.lua    → luna://lua/mount
    └── lua_resource_dome.lua     → luna://lua/dome
    └── lua_resource_weather.lua  → luna://lua/weather
```

LUNA automatically scans SPIFFS for `lua_resource_*.lua` files and publishes them as Resources.
**Add a script and a Resource appears. Delete it and the Resource disappears.** No firmware changes required.

### Processing Flow

```
① Claude requests resources/read "luna://lua/mount"
      ↓
② LUNA reads /lua_resource_mount.lua from SPIFFS
      ↓
③ LUNA executes the Lua script
      ↓
④ Script sends commands to telescope and retrieves RA/Dec
      ↓
⑤ Script returns a JSON string via return
      ↓
⑥ LUNA returns it to Claude as an MCP response
      ↓
⑦ Claude receives RA/Dec and answers the user
```

---

## 3. Writing Resource Scripts

### Naming Convention

| Item | Rule |
|------|------|
| File name | `lua_resource_{name}.lua` |
| Allowed characters | Alphanumeric, underscore, hyphen |
| URI | `luna://lua/{name}` |
| Return value | JSON string (returned via `return`) |

### Basic Template

```lua
-- lua_resource_{name}.lua
-- Retrieve data from device
local value = serial.query("command", 100)

-- Error check
if value == "" then
  return '{"error":"no response"}'
end

-- Return JSON string via return
return string.format('{"key":"%s"}', value)
```

### Example ① Equatorial Mount (LX200-compatible)

```lua
-- lua_resource_mount.lua
local ra  = serial.query(":GR#", 100)
local dec = serial.query(":GD#", 100)
if ra == "" or dec == "" then
  return '{"error":"no response from mount"}'
end
return string.format('{"ra":"%s","dec":"%s"}', ra, dec)
```

### Example ② Dome

```lua
-- lua_resource_dome.lua
local status = serial.query("DOME_STATUS\r", 300)
if status == "" then
  return '{"error":"no response from dome"}'
end
return '{"status":"' .. status .. '"}'
```

### Example ③ Multiple Values Combined

```lua
-- lua_resource_mount.lua (detailed version)
local ra     = serial.query(":GR#", 100)
local dec    = serial.query(":GD#", 100)
local status = serial.query(":GW#", 100)
if ra == "" then
  return '{"error":"no response from mount"}'
end
return string.format(
  '{"ra":"%s","dec":"%s","status":"%s"}',
  ra, dec, status
)
```

### Example ④ Publish Log Data

```lua
-- lua_resource_log.lua
log.load()
local lines = log.read(5)
return '{"recent_log":"' .. lines .. '"}'
```

### Convention Summary

| Item | Rule |
|------|------|
| File name | `lua_resource_{name}.lua` |
| Return value | Must return a JSON string via `return` |
| On error | Return `{"error":"message"}` |
| Execution time | Keep short (100–200ms recommended for serial.query) |
| `log.save()` | Do **not** call inside resource scripts (flash lifetime) |
| Side effects | Read-only recommended — avoid commands that move the device |

---

## 4. Saving Scripts (lua_save)

You can save scripts directly from Claude.

### Example Claude Instruction

```
"Save the following Lua script with the name resource_mount"

local ra  = serial.query(":GR#", 100)
local dec = serial.query(":GD#", 100)
if ra == "" or dec == "" then
  return '{"error":"no response from mount"}'
end
return string.format('{"ra":"%s","dec":"%s"}', ra, dec)
```

### lua_save Tool

```
lua_save("resource_mount", "script content")
→ saved to SPIFFS as /lua_resource_mount.lua
→ luna://lua/mount automatically appears in Resources
```

---

## 5. Verifying Resources

### 5-1. resources/list — Check Available Resources

Claude Desktop references this automatically.
To check manually, use curl:

```bash
curl -X POST http://192.168.3.154/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"resources/list","params":{}}'
```

**Example response:**
```json
{
  "resources": [
    {
      "uri": "luna://lua/mount",
      "name": "mount",
      "description": "Resource: /lua_resource_mount.lua",
      "mimeType": "application/json"
    }
  ]
}
```

### 5-2. resources/read — Read a Resource Value

```bash
curl -X POST http://192.168.3.154/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":2,"method":"resources/read","params":{"uri":"luna://lua/mount"}}'
```

**Example response:**
```json
{
  "contents": [
    {
      "uri": "luna://lua/mount",
      "mimeType": "application/json",
      "text": "{\"ra\":\"20:41.4#\",\"dec\":\"+45:16#\"}"
    }
  ]
}
```

### 5-3. Using from Claude Desktop

With LUNA Observatory connected to Claude Desktop, Resources are available through natural conversation:

```
User: "Where is the telescope pointing?"
Claude: reads luna://lua/mount → answers with RA/Dec

User: "Can we observe tonight?"
Claude: reads luna://lua/mount and luna://lua/weather → gives a combined assessment
```

---

## 6. LUNA Observatory Architecture

One LUNA handles one device. Claude Desktop connects to multiple LUNAs simultaneously to control the entire observatory.

```
Claude Desktop (AI brain)
    │
    ├── LUNA #1 (Serial mount)          192.168.x.x
    │     Serial → NS-5000 / Temma2 (LX200 / Temma protocol)
    │     luna://lua/mount
    │
    ├── LUNA #2 (Dome)                  192.168.x.x
    │     luna://lua/dome
    │
    ├── LUNA #3 (Weather sensor)        192.168.x.x
    │     luna://lua/weather
    │
    └── LUNA #4 (AiBridge gateway)      192.168.x.x
          http.get/put → AiBridge v7.14 (Alpaca REST + LX200 API)
          Resources:
            luna://lua/mount_aibridge  ← OnStep mount (/api/status)
            luna://lua/weather         ← Weather sensor (DHT22)
            luna://lua/switch          ← Relay 4 ports
```

### Identifying Multiple LUNAs

Change the `location` name on each LUNA to distinguish them.

| Item | How to configure |
|------|-----------------|
| **location name** | Set each device to `Ai-LUNA_1`, `Ai-LUNA_2`, etc. |
| **IP address** | Auto-assigned by DHCP, or set as static |
| **Claude Desktop config** | Register server names and IPs in `claude_desktop_config.json` |

```json
{
  "mcpServers": {
    "luna-mount":   { "args": ["http://192.168.3.154/mcp"] },
    "luna-dome":    { "args": ["http://192.168.3.155/mcp"] },
    "luna-weather": { "args": ["http://192.168.3.156/mcp"] }
  }
}
```

Claude distinguishes each device using the `location` field from `get_network_status`.

### LUNA vs LUNA Observatory

LUNA Observatory is a **fully backward-compatible superset** of LUNA.

| Feature | LUNA v3.9.1 | LUNA Observatory v5.9.1 |
|---------|:-----------:|:-----------------------:|
| All Tools | ✅ | ✅ |
| Prompts (luna_guide) | ✅ | ✅ |
| Lua engine + luna.run() | ✅ | ✅ |
| notify.set() / log module | ✅ | ✅ |
| **Resources** | ❌ | ✅ |
| **http module** | ❌ | ✅ |

LUNA Observatory works as a standalone single-device setup.
Its full potential is realized when integrating multiple devices.

---

## 7. Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Not appearing in resources/list | Wrong file name | Check that it starts with `lua_resource_` |
| Returns `{"error":"no response"}` | Device not responding | Increase serial.query timeout |
| resources/read returns error | Lua task is running | Check lua_status for idle, then retry |
| JSON is malformed | Return string contains `"` | Escape with `string.format` |

---

## 8. http Module — AiBridge / Alpaca Integration (v5.9.1)

LUNA v5.9.1 allows Lua scripts to call HTTP devices on the LAN directly.
This lets you publish any **AiBridge (ASCOM/Alpaca compatible)** device as a Resource.

### API

```lua
http.get(url, timeout_ms)           -- HTTP GET. Returns response body
http.put(url, body, timeout_ms)     -- HTTP PUT. Returns response body
-- timeout_ms default: 2000ms
-- Returns empty string "" on error or timeout
-- HTTPS not supported (LAN HTTP only)
```

### AiBridge Alpaca Integration Examples

#### Weather Sensor (observingconditions)

```lua
-- lua_resource_weather.lua
local temp = http.get("http://192.168.3.7/api/v1/observingconditions/0/temperature", 1000)
local hum  = http.get("http://192.168.3.7/api/v1/observingconditions/0/humidity",    1000)
if temp == "" then return '{"error":"no response from AiBridge"}' end
return string.format('{"temp":%s,"humidity":%s}', temp, hum)
```

#### Switch Port Status (switch)

```lua
-- lua_resource_switch.lua
local s0 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=0", 1000)
local s1 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=1", 1000)
local s2 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=2", 1000)
local s3 = http.get("http://192.168.3.7/api/v1/switch/0/getswitch?Id=3", 1000)
return string.format('{"port0":%s,"port1":%s,"port2":%s,"port3":%s}', s0, s1, s2, s3)
```

#### Switch Control (PUT)

```lua
-- Turn on heater (call from lua_exec)
local r = http.put("http://192.168.3.7/api/v1/switch/0/setswitch", "Id=0&State=true", 1000)
return r
```

### Recommended Timeout Values

| Use case | Recommended timeout |
|----------|-------------------|
| Alpaca GET (sensor / status read) | 1000ms |
| Alpaca PUT (command send) | 2000ms |
| `/api/external/discover` | 4000ms (longer due to UDP wait) |

### Hardware Test Results (2026-03-21)

| Test | Result | Response time |
|------|--------|--------------|
| `http.get("http://192.168.4.1/")` | AiBridgeMCP Console response | 35ms ✅ |
| `http.get("http://192.168.3.7/")` | AiBridge Console response | 153ms ✅ |
| `switch/0/maxswitch` | Value:4 (4 ports confirmed) | ✅ |
| `switch/0/getswitch?Id=0–3` | All ports false (OFF) | 315ms ✅ |
| `observingconditions/0/temperature` | Value:-999.0 (no sensor) | ✅ |
| `/api/external/discover` | Discovery complete (data:[]) | 3386ms ✅ |

### LUNA + AiBridge Integrated Architecture

```
Claude Desktop (AI brain)
    │
    ├── LUNA #1 (Mount)
    │     Serial → LX200 / Temma2
    │     Resources: luna://lua/mount
    │
    └── LUNA #2 (AiBridge gateway)
          http.get/put → AiBridge → Alpaca REST
          Resources:
            luna://lua/weather   ← Temperature, humidity, cloud cover
            luna://lua/switch    ← Output port status
            luna://lua/focuser   ← Focuser position
```

**No AiBridge firmware changes required.** It continues to operate as an Alpaca REST server.
Any Alpaca device can be published as a LUNA Resource with just a few lines of Lua.

---

## 8.1 AiBridge LX200 REST API — Controlling OnStep Mounts (v5.9.1)

In addition to Alpaca REST, AiBridge provides a **REST API for sending and receiving LX200 commands over HTTP**.
By calling this API from LUNA's http module, you can control an OnStep telescope connected to AiBridge.

### Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /api/status` | Retrieve RA / Dec / tracking state as JSON |
| `GET /api/command?cmd=XX` | Send any LX200 command (no `:` or `#` needed), returns JSON |
| `GET /api/goto?ra=HH:MM:SS&dec=±DD:MM:SS` | GoTo specified coordinates |

### Lua Script Examples

#### Get Mount Status via AiBridge

```lua
-- lua_resource_mount_aibridge.lua
-- Retrieve RA/Dec/state of OnStep telescope connected to AiBridge (192.168.3.7)
local resp = http.get("http://192.168.3.7/api/status", 2000)
if resp == "" then return '{"error":"no response from AiBridge"}' end
return resp
-- → {"status":"success","data":{"ra":"20:41.4","dec":"+45:16","state":"Tracking",...}}
```

#### Send Arbitrary LX200 Commands

```lua
-- Example from lua_exec (GoTo: set target → execute GoTo)
local r1 = http.get("http://192.168.3.7/api/command?cmd=SrHH:MM:SS", 1000)  -- Set RA
local r2 = http.get("http://192.168.3.7/api/command?cmd=SdDD:MM:SS", 1000)  -- Set Dec
local r3 = http.get("http://192.168.3.7/api/command?cmd=MS", 1000)           -- GoTo
return r3
```

### Hardware Test Results (2026-03-21)

| Test | Result | Response time |
|------|--------|--------------|
| `GET /api/status` | `{"state":"Tracking","connected":true,...}` | 113ms ✅ |
| `GET /api/command?cmd=GR` | `{"status":"success","message":"Command executed"}` | 206ms ✅ |

*Note: No telescope connected during testing — RA/Dec were empty. API operation confirmed.*

### Comparison: Direct Serial vs AiBridge LX200 API

| | LUNA direct serial | LUNA + AiBridge LX200 API |
|--|-------------------|--------------------------|
| Connection | Cable required | Over LAN (WiFi) |
| Command format | `serial.query(":GR#", 100)` | `http.get(".../api/command?cmd=GR", 1000)` |
| Response | Raw string `"20:41.4#"` | JSON `{"data":"20:41.4#"}` |
| Target devices | Legacy serial devices | OnStep mounts (via AiBridge) |

---

## 9. Verified Hardware Environments

| Device | Connection | Resource Script | Status |
|--------|-----------|----------------|--------|
| NS-5000 (LX200-compatible) | Serial 9600bps | lua_resource_mount.lua | ✅ Verified on hardware |
| AiBridge v7.14 switch | http.get/put | lua_resource_switch.lua | ✅ Verified on hardware |
| AiBridge v7.14 observingconditions | http.get | lua_resource_weather.lua | ✅ Verified on hardware |
| AiBridge v7.14 LX200 REST API | http.get `/api/status` `/api/command` | lua_resource_mount_aibridge.lua | ✅ Verified on hardware |
| Takahashi Temma2 | Serial 19200bps | lua_resource_mount.lua | 📋 Planned |

---

## 10. Expanding to LUNA Factory

The LUNA Observatory architecture applies beyond astronomy.

**LUNA Factory** is a concept that applies the same architecture to industrial and factory environments.

| | LUNA Observatory | LUNA Factory |
|--|-----------------|-------------|
| Devices | Telescope, dome, weather | CNC, PLC, sensors |
| Protocols | LX200, Temma | Industrial serial |
| Automation | Observation sequences | Production processes |
| AI decisions | Weather, star positions | Quality, anomalies, efficiency |
| Target market | Free, astronomy community | Paid, enterprise |

**Almost no firmware changes required.** Just define your devices with Lua scripts.

---

*AiBridgeMCP LUNA Observatory v5.9.1 — Bridge Across Decades © 2026 Nishioka Sadahiko*
