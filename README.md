# LUNA — AI-Driven Telescope & Observatory Controller

**MCP Server + Lua Scripting Engine on ESP32**

> Let Claude AI control your telescope, camera, and entire observatory — in plain language.

![Version](https://img.shields.io/badge/version-v5.9.3-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-ESP32-orange)

---

## ⚠️ Version Notice

| Version | Lua API Guide | Vixen Protocol | Status |
|---------|---------------|---|--------|
| v5.9.6+ | ✅ Automatic | ✅ Automatic | Recommended |
| v5.9.5  | ⚠️ Manual load required | ❌ Not available | Legacy |

For v5.9.5 users: Please load [LUNA_Lua_Guide](docs/LUNA_Lua_Guide.md) into Claude or upgrade to v5.9.6+.

---

## What is LUNA?

LUNA is a firmware for ESP32 that turns it into an **MCP (Model Context Protocol) server with a built-in Lua scripting engine**.

Connect LUNA to your telescope or observatory equipment, then control everything through Claude AI — in plain language.

```
You ──→ Claude AI ──→ LUNA (ESP32) ──→ Telescope / Serial Device
              "Go to M42"    serial command    🔭 slews

You ──→ Claude AI ──→ LUNA (ESP32) ──→ NINA (PC) ──→ Camera / Focuser / Guider
         "Image M42 tonight"   HTTP API       🌟 full automation
```

LUNA runs **Lua scripts autonomously** on the ESP32. Claude writes the scripts, deploys them, and monitors results — all through the MCP protocol.

![LUNA Observatory System v5.9.6](assets/LUNA%20Observatory%20System%20v5.9.6.png)

---

## ★ New in v5.9.3 — Vixen Mount Support (UDP)

LUNA now supports **Vixen Wireless Unit** mounts (AXD, SXP, SXD2, AP, SX2, AXJ, SXP2) via UDP, and **Vixen STAR BOOK TEN** via HTTP — all through the new `udp.*` Lua module.

| Mount | Protocol | Connection |
|-------|----------|------------|
| Vixen Wireless Unit (AXD/SXP/SXD2/AP/SX2/AXJ/SXP2) | UDP | 192.168.6.1 : 60023 |
| Vixen STAR BOOK TEN | HTTP GET | 169.254.1.1 : 80 |

```lua
-- Example: slew Vixen Wireless Unit to a target
udp.begin(50000)
udp.connect("192.168.6.1", 60023)
udp.write("$MTTGT=1,83.82,−5.39,Orion Nebula\r\n")
udp.write("$MTMV2\r\n")
local resp = udp.read(2000)
udp.close()
log.write("GoTo response: " .. (resp or "timeout"))
```

> Claude knows the full Vixen command set internally via `vixen_guide` — just tell it where to point.

---

## ★ New in v5.9.2 — NINA Astrophotography Integration

LUNA now controls **NINA (Nighttime Imaging 'N' Astronomy)** — the world's most popular free astrophotography software — directly via its HTTP API.

### What this means

| Without LUNA | With LUNA + NINA |
|-------------|-----------------|
| Build sequences manually (30–50 settings) | Just tell Claude what you want to image |
| Start imaging manually | "Image M42 at 3 min, 20 frames" — done |
| Configure each device separately | Claude coordinates telescope, camera, focuser, guider, dome, and weather sensor together |
| Troubleshoot errors yourself | Claude explains what went wrong and suggests fixes |

### Verified working (2026-03-27)

```
✅ Camera connection and status
✅ Capture start / stop
✅ Autofocus
✅ Filter wheel control
✅ Sequence start / stop
✅ Guider start / stop
✅ ASCOM / Alpaca device control (switch, weather sensor)
✅ Stellarium integration
```

### How it works

```
"Image M42 tonight"

  ① Check weather sensors → safe to observe?     (LUNA + Alpaca)
  ② Slew telescope to M42                         (Alpaca)
  ③ Run autofocus                                 (NINA HTTP API)
  ④ Test exposure → plate solve → correct pointing (NINA HTTP API)
  ⑤ Start main imaging sequence                   (NINA HTTP API)
  ⑥ Re-focus periodically as temperature drops    (NINA HTTP API)
  ⑦ Auto-abort if weather deteriorates            (LUNA monitoring)
```

> **NINA is the "hands", Claude is the "brain", LUNA is the "nervous system."**

### Requirements for NINA integration

- NINA v3.x — [nighttime-imaging.eu](https://nighttime-imaging.eu/) (free)
- NINA Advanced API plugin (free, installed from within NINA)
- Port: **1888** (default), API Enabled: **ON**
- LUNA and the NINA PC on the same WiFi network

---

## Key Features

- **MCP Server** — connects directly to Claude via the Model Context Protocol
- **Lua 5.1 scripting engine** — write, save, and run scripts on the ESP32
- **Autonomous operation** — scripts run independently; Claude monitors and intervenes as needed
- **Vixen mount support** — Wireless Unit (UDP) and STAR BOOK TEN (HTTP) via new `udp.*` module (v5.9.3)
- **NINA integration** — control astrophotography software via HTTP POST (v5.9.2)
- **http module** — `http.get` / `http.put` / `http.post` for ASCOM/Alpaca, NINA Advanced API, and Vixen STAR BOOK TEN
- **udp module** — `udp.begin` / `udp.connect` / `udp.write` / `udp.read` / `udp.close` for Vixen Wireless Unit (v5.9.3)
- **MCP Resources** — Claude reads device state naturally, like reading a gauge
- **Script-to-script data passing** — `log.save()` / `log.load()` enable stateful workflows without Claude
- **Serial device control** — full RS-232C control via `serial.query()`, `serial.write()`, `serial.read()`
- **FreeRTOS dual-core** — MCP server and Lua engine run on separate cores; no blocking

---

## Supported Devices

| Variant | Baud Rate | Target Devices |
|---------|-----------|----------------|
| **LUNA Standard** | 9,600 bps | Meade LX200, OnStep, LX200-compatible mounts |
| **LUNA Temma2** | 19,200 bps | Takahashi Temma2 |

**PC Software Integration (v5.9.2+)**

| Software | Protocol | Port |
|----------|----------|------|
| NINA Advanced API | HTTP POST/GET | 1888 |
| Stellarium Remote Control | HTTP POST/GET | 8090 |
| ASCOM / Alpaca devices | HTTP GET/PUT | varies |

**Vixen Mount Integration (v5.9.3)**

| Mount | Protocol | Address |
|-------|----------|---------|
| Vixen Wireless Unit (AXD/SXP/SXD2/AP/SX2/AXJ/SXP2) | UDP | 192.168.6.1 : 60023 |
| Vixen STAR BOOK TEN | HTTP GET | 169.254.1.1 : 80 |

> LUNA works with **any RS-232C serial device** and any **ASCOM/Alpaca-compatible device** on your LAN.

---

## Requirements

### Hardware
- **ESP32 development board** (ESP32-DevKitC or compatible)
- RS-232C interface circuit (MAX3232 or equivalent)
- Power: **5V via micro USB** (power bank or USB charger)

> **Ready-to-use LUNA boards** (LUNA OnStepNinja V2 — pre-wired, pre-flashed) are available from the author.

### Software / Service
- [Claude Desktop](https://claude.ai/download) (free)
- MCP connector (included in the GitHub Releases download package)

---

## Getting Started

### Step 1 — Flash the Firmware

Download the firmware binary from [GitHub Releases](https://github.com/OnStepNinja/LUNA/releases):

| File | Target |
|------|--------|
| `LUNA_v5.9.3_Standard.bin` | LX200 / OnStep (9600 bps) |
| `LUNA_v5.9.3_Temma2.bin` | Takahashi Temma2 (19200 bps) |

Flash to your ESP32 using esptool or ESP32 Flash Download Tool.

📄 **See [LUNA_Quick_Setup_Guide.md](docs/LUNA_Quick_Setup_Guide.md) for full instructions.**

### Step 2 — Connect Claude Desktop to LUNA

Configure Claude Desktop using the MCP connector included in the download package.

📄 **See [LUNA_Quick_Setup_Guide.md](docs/LUNA_Quick_Setup_Guide.md) for the 3-step setup.**

### Step 3 — Start talking to your telescope

```
You:    "Go to M42"
Claude: → slews telescope to M42

You:    "Image M42 tonight, 3 minutes per frame, 20 frames"
Claude: → autofocuses, plate solves, starts NINA sequence
```

---

## Example: Full Observatory Automation

```lua
-- LUNA Lua script: check weather, then start NINA sequence
local resp, code = http.get(
  "http://192.168.x.x/api/v1/observingconditions/0/humidity", 2000)

if code == 200 then
  local humidity = -- parse resp
  if humidity < 80 then
    http.post("http://192.168.x.x:1888/v2/api/sequence/start", "{}",
              "application/json", 3000)
    log.write("Sequence started. Humidity: " .. humidity .. "%")
  else
    notify.set("Humidity too high (" .. humidity .. "%). Imaging skipped.")
  end
end
```

---

## MCP Tools Available to Claude

| Tool | Description |
|------|-------------|
| `lua_exec` | Execute a Lua script immediately |
| `lua_save` | Save a script to the ESP32 (SPIFFS) |
| `lua_run` | Run a saved script asynchronously |
| `lua_status` | Check execution state and results |
| `lua_log` | Read the script log buffer |
| `lua_stop` | Stop a running script |
| `lua_list` | List saved scripts |
| `serial_query` | Send a command and read response |
| `serial_write` | Send a raw serial command |
| `serial_read` | Read serial data |
| `fs_read` | Read a file from ESP32 storage |
| `fs_write` | Write a file to ESP32 storage |
| `check_notify` | Receive async notifications from Lua scripts |

---

## Lua API (inside scripts)

```lua
-- Serial control
serial.write(cmd)
serial.read(timeout_ms)
serial.query(cmd, timeout_ms)

-- HTTP (LAN only — ASCOM/Alpaca/NINA/Stellarium/Vixen STAR BOOK TEN)
http.get(url, timeout_ms)
http.put(url, body, content_type, timeout_ms)
http.post(url, body, content_type, timeout_ms)  -- v5.9.2

-- UDP (Vixen Wireless Unit)              -- v5.9.3
udp.begin(local_port)
udp.connect(host, port)
udp.write(data)
udp.read([timeout_ms])
udp.close()

-- Logging & data passing
log.write(message)
log.read([n])       -- n omitted = latest line
log.save()          -- persist to flash
log.load()          -- restore from flash

-- Hardware
hw.delay(ms)
hw.millis()
hw.free_heap()
hw.led(0 or 1)

-- Notify Claude
notify.set("message")

-- Script chaining
luna.run("next_script_name")
```

---

## Hardware

**LUNA OnStepNinja V2** — pre-built board available from the author:

- Pre-flashed with LUNA v5.9.3
- RS-232C connector (D-sub 9-pin) included
- Power: 5V micro USB
- Compatible with NS-5000, OnStep, and LX200-compatible mounts

> ⚠️ **Cable note:** Use a **straight RS-232C cable** between LUNA and a PC.
> Use a **cross cable** between LUNA and NS-5000 / OnStep.

Contact: [GitHub Issues](https://github.com/OnStepNinja/LUNA/issues)

---

## Documentation

| Document | Description |
|----------|-------------|
| [LUNA_Quick_Setup_Guide.md](docs/LUNA_Quick_Setup_Guide.md) | First-time setup (3 steps) |
| [LUNA_USER_MANUAL.md](docs/LUNA_USER_MANUAL.md) | Full reference manual |
| [LUNA_Lua_Guide.md](docs/LUNA_Lua_Guide.md) | Lua scripting guide |

📚 **[LUNA Docs on NotebookLM](https://notebooklm.google.com/notebook/f2ce997f-18c2-44c4-8738-973b204c190c)** — Ask questions about LUNA directly.

---

## License

The LUNA firmware binary is distributed under the [MIT License](LICENSE).
Source code is proprietary and not published.

Copyright © 2026 Nishioka Sadahiko

---

### Third-Party Licenses

LUNA incorporates **Lua 5.1**, distributed under the MIT License:

> Copyright © 1994–2012 Lua.org, PUC-Rio.
> Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions: The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software. THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.

See also: https://www.lua.org/license.html

---

## Author

**Nishioka Sadahiko**
GitHub: [@OnStepNinja](https://github.com/OnStepNinja)
