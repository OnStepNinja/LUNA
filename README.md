> 🚀 **v5.9.1b Released** — Each board gets a unique Wi-Fi name automatically. Recommended over all previous versions.

---

# LUNA — AiBridgeMCP + Lua Scripting Engine

> *The next evolution of AiBridgeMCP — now with autonomous scripting on the ESP32.*

**[AiBridgeMCP](https://github.com/OnStepNinja/AiBridgeMCP)** proved that a $5 ESP32 chip could bridge Claude AI and any legacy serial device.
**LUNA** takes it further: Claude no longer has to stay connected.

LUNA runs the observation sequence autonomously — slewing, measuring, logging, and notifying Claude only when the job is done.

Lua scripts can also control **any ASCOM/Alpaca-compatible device** — focusers, filter wheels,
rotators, weather stations — directly over HTTP, without a Windows PC or ASCOM Platform.

Write a Lua script, deploy it to the ESP32, and walk away.
LUNA runs the observation sequence autonomously — slewing, measuring, logging, and notifying Claude only when the job is done.

| Feature | AiBridgeMCP | LUNA |
|---------|:-----------:|:----:|
| Claude controls device directly | ✅ | ✅ |
| Lua scripts run on ESP32 | — | ✅ |
| Autonomous operation (no Claude needed) | — | ✅ |
| Script chaining (`luna.run()`) | — | ✅ |
| HTTP access to Alpaca REST / local devices | — | ✅ |
| Async notification to Claude (`notify.set()`) | — | ✅ |
| MCP Resources (live sensor data) | — | ✅ |
| Unique Wi-Fi name per board (MAC-based) | — | ✅ |

LUNA is designed for astronomical instrument control:
**Takahashi Temma2**, **NS-5000**, and **OnStep NS-3000**.

> Lua 5.1 is the scripting language running on the ESP32.
> For the full API reference, see the [LUNA Guide](AiBridgeMCP_LUNA_Guide.md).

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Firmware](https://img.shields.io/badge/Firmware-v5.9.1b-blue.svg)]()
[![Platform](https://img.shields.io/badge/Platform-ESP32-red.svg)]()
[![MCP](https://img.shields.io/badge/MCP-Streamable%20HTTP-purple.svg)]()

---

## Supported Devices

| Device | Protocol | Baud Rate |
|--------|----------|-----------|
| Takahashi Temma2 | Temma | 19200 bps |
| NS-5000 | LX200 | 9600 bps |
| OnStep NS-3000 | LX200 | 9600 bps |
| Any RS-232C serial device | Custom Lua | Configurable |

> **Note:** For OnStep, only the NS-3000 is supported. 

---

## Getting Started

### 1. Flash the Firmware

| File | Target |
|------|--------|
| `LUNA OnStepNinjaV2 9600 v5.9.1b.zip` | NS-5000 / OnStep NS-3000 (9600 bps) |
| `LUNA OnStepNinjaV2 19200 v5.9.1b.zip` | Takahashi Temma2 (19200 bps) |

Flash the binary to your ESP32 using the included flasher tool.

### 2. Connect to LUNA

On first boot, LUNA starts a Wi-Fi access point (`LUNA_xxxxxx`, no password).
Open **http://192.168.4.1** in your browser to configure your home Wi-Fi.

### 3. Connect Claude Desktop

Add LUNA to your `claude_desktop_config.json` and restart Claude Desktop.

📄 Full instructions: [Quick Setup Guide](https://notebooklm.google.com/notebook/f2ce997f-18c2-44c4-8738-973b204c190c)

---

## Usage Example

```text
You:    "Slew to M42"
Claude: [sends goto command, confirms slew started]

You:    "Run the observation sequence"
Claude: [calls lua_run("observe") — scripts chain autonomously from here]


## MCP Tools

| Tool | Description |
|------|-------------|
| `lua_exec(code)` | Execute Lua code immediately |
| `lua_run(name)` | Run a saved script asynchronously |
| `lua_save(name, code)` | Save a script to SPIFFS |
| `lua_list()` | List saved scripts |
| `lua_status()` | Poll running script state |
| `lua_stop()` | Abort a running script |
| `lua_log()` | Read the script log (last 20 lines) |
| `serial_query(cmd, ms)` | Send command, receive response |
| `serial_write(data)` | Send data to serial port |
| `serial_read()` | Read available serial data |
| `fs_read(path)` | Read a file from SPIFFS |
| `fs_write(path, data)` | Write a file to SPIFFS |
| `get_system_info()` | ESP32 system status |
| `get_network_status()` | Wi-Fi / network information |

---

## Support

📚 **Documentation** — Setup guides, technical references, and FAQ.
Ask questions directly inside NotebookLM and get answers from the official documentation.
👉 https://notebooklm.google.com/notebook/f2ce997f-18c2-44c4-8738-973b204c190c

💬 **Community** — Join our Facebook group to ask questions, share your results, and connect with other users. The developer is also a member.
👉 https://www.facebook.com/groups/1230935959149731

---

## About

LUNA is part of the **AiBridgeMCP** project — connecting Claude AI to legacy devices via ESP32.

- **Author**: Nishioka Sadahiko ([@OnStepNinja](https://github.com/OnStepNinja))
- **Firmware**: AiBridgeMCP v5.9.1b LUNA Edition
- **License**: MIT

> LUNA works with any RS-232C serial device — not just telescopes.

---

## License

**Freeware — Free to use for personal and commercial purposes.**

The LUNA firmware binary is released under the [MIT License](LICENSE).
Source code is proprietary and not published.

### Third-party Licenses

LUNA firmware incorporates **Lua 5.1**, distributed under the MIT License:

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
*AiBridgeMCP — Bridge Across Decades © 2026 Nishioka Sadahiko*
