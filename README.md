# LUNA
ESP32 MCP Server + Lua scripting engine. Control any serial device with Claude AI.

> ⚠️ **Work in Progress** — This repository is under preparation. Firmware binaries and full documentation are not yet available. Star or Watch this repo to be notified when released.

---

# LUNA — AI-Driven Telescope Controller

**MCP Server + Lua Scripting Engine on ESP32**

> *Ask Claude AI to slew your telescope. Write Lua scripts that run autonomously on a $5 chip.*

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Firmware](https://img.shields.io/badge/Firmware-v3.9.1-blue.svg)]()
[![Platform](https://img.shields.io/badge/Platform-ESP32-red.svg)]()
[![MCP](https://img.shields.io/badge/MCP-Streamable%20HTTP-purple.svg)]()

---

## What is LUNA?

LUNA is an **ESP32-based MCP (Model Context Protocol) Server with an embedded Lua scripting engine**.

It acts as a bridge between **Claude AI** and your telescope (or any serial device), enabling:

- **Direct AI control** — Claude sends commands directly to your mount
- **Autonomous Lua scripts** — scripts run on the ESP32 independently, without Claude present
- **Script chaining** — `luna.run()` lets scripts trigger the next script automatically
- **Script-to-script data passing** — automated observation sequences across multiple scripts

```
Claude AI  ──MCP──▶  LUNA (ESP32)  ──Serial──▶  Telescope / Device
                        │
                    Lua Engine
                  (autonomous control)
```

---

## Supported Devices

| Device | Protocol | Baud Rate | Status |
|--------|----------|-----------|--------|
| **Takahashi Temma2** | Temma | 19200 bps | ✅ Supported |
| **NS-5000** | LX200 | 9600 bps | ✅ Supported |
| **OnStep** | LX200 | 9600 bps | ✅ Supported |
| **LX200-compatible mounts** | LX200 | 9600 bps | ✅ Supported |
| Any RS-232C serial device | Custom Lua | Configurable | ✅ Via Lua scripting |

---

## Key Features

### For Telescope Users
- **"Slew to M42"** — just tell Claude the target, it calculates coordinates and sends the goto command
- **AI-assisted alignment** — Claude reads the mount's response and guides you through the process
- **Protocol exploration** — Claude can probe an unknown device and figure out its command set

### For Developers
- **Lua 5.1 scripting** — full language support (functions, tables, coroutines)
- **`serial.*` API** — send/receive/query over Serial2
- **`log.*` API** — script-to-Claude and script-to-script data passing
- **`hw.*` API** — GPIO, LED, timing
- **`notify.*` API** — signal Claude when a long-running script completes
- **SPIFFS storage** — save and reload scripts across power cycles
- **`luna.*` API** — chain scripts autonomously (max 10 in sequence)

### Architecture
- **FreeRTOS dual-core** — Core 0: WiFi/MCP communication, Core 1: Lua execution (no interference)
- **Stateless Lua execution** — each script runs in a fresh Lua state (memory leak-free)
- **Script state persistence** — `log.save()` / `log.load()` pass data between scripts via SPIFFS

---

## Getting Started

### Step 1 — Flash the Firmware

Download the firmware binary for your device:

| File | Target |
|------|--------|
| `LUNA-LX200-v3.9.1.bin` | LX200 / OnStep (9600 bps) |
| `LUNA-Temma2-v3.9.1.bin` | Takahashi Temma2 (19200 bps) |

Flash the binary to your ESP32, then configure your WiFi credentials.

📄 **For full flashing and WiFi setup instructions, see the [Setup Guide](https://github.com/OnStepNinja/AiBridgeMCP).**

### Step 2 — Connect Claude Desktop to LUNA

Configure Claude Desktop to connect to your LUNA board over your local network.

📄 **For full Claude Desktop configuration instructions, see the [Setup Guide](https://github.com/OnStepNinja/AiBridgeMCP).**

Once connected, you can talk to your telescope through Claude.

```
You:    "Slew to M42"
Claude: [calls lua_exec, sends goto command, confirms slew started]

You:    "What is the current RA/Dec?"
Claude: [calls serial_query with Q\r, parses and displays coordinates]
```

---

## Lua Scripting Example

### Slew to a target (Temma2)

```lua
-- Send current position query
local pos = serial.query("Q\r", 2000)
log.write("Current: " .. (pos or "no response"))

-- Goto M42: RA 5h35m17.3s, Dec -5°23'28"
local resp = serial.query("G0535173-052328\r", 3000)
log.write("Goto response: " .. (resp or "none"))

notify.set()
return {current = pos, response = resp}
```

### Autonomous multi-script sequence with `luna.run()`

Claude calls `lua_run("calibrate")` **once**. Scripts chain automatically — no further Claude intervention needed.

**Script A — Calibration** (`lua_load("calibrate", ...)`)
```lua
local cal = 1.0023
log.write("CAL=" .. cal)
log.save()              -- persist to SPIFFS
luna.run("measure")     -- automatically start next script
notify.set()
```

**Script B — Measurement** (`lua_load("measure", ...)`)
```lua
log.load()       -- restore Script A's result
local cal = tonumber((log.read() or ""):match("CAL=(.+)")) or 1.0
local raw = tonumber(serial.query(":MEAS?\r\n", 500)) or 0.0
log.write(string.format("RESULT=%.4f", raw * cal))
notify.set()     -- signal Claude: all done
```

---

## MCP Tools

| Tool | Description |
|------|-------------|
| `lua_exec(code)` | Execute Lua code immediately |
| `lua_status()` | Poll running script state and result |
| `lua_stop()` | Abort a running script |
| `lua_log()` | Read the script log buffer (last 20 lines) |
| `lua_load(name, code)` | Save a script to SPIFFS |
| `lua_run(name)` | Run a saved script asynchronously |
| `lua_list()` | List saved scripts |
| `serial_query(cmd, ms)` | Send command, receive response |
| `serial_write(data)` | Send data to serial port |
| `serial_read()` | Read available serial data |
| `fs_read(path)` | Read a file from SPIFFS |
| `fs_write(path, data)` | Write a file to SPIFFS |

---

## Documentation

- **[LUNA Guide](AiBridgeMCP_LUNA_Guide.md)** — complete reference with examples (Japanese / 日本語)

---

## Hardware Board

> **Coming Soon** — Pre-configured LUNA boards are under preparation.

| Board | Baud Rate | Target Device |
|-------|-----------|---------------|
| LUNA-Temma2 | 19200 bps | Takahashi Temma2 |
| LUNA-LX200 | 9600 bps | OnStep / LX200-compatible |

Ready to use — no wiring or configuration required.
Watch this repository for availability updates.

---

## About

LUNA is part of the **AiBridgeMCP** project — connecting Claude AI to legacy devices via ESP32.

- **Author**: Nishioka Sadahiko ([@OnStepNinja](https://github.com/OnStepNinja))
- **License**: MIT
- **Firmware**: AiBridgeMCP v3.9.1 LUNA Edition

> LUNA works with any RS-232C serial device — not just telescopes.

---

## License

**Freeware — Free to use for personal and commercial purposes.**

The LUNA firmware binary is released under the [MIT License](LICENSE).
Source code is proprietary and not published.

### Third-party Licenses

LUNA firmware incorporates **Lua 5.1**, which is distributed under the MIT License:

```
Copyright © 1994–2012 Lua.org, PUC-Rio.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
```
