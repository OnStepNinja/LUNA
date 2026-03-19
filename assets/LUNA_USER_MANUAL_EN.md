# LUNA User Manual
# Complete Reference Guide

**Brand Name**: LUNA (AiBridgeMCP v3.9.1)
**Audience**: Users and engineers who want to control RS-232C devices with AI
**Revision**: r1.0
**Last Updated**: 2026-03-19
**Created**: 2026-03-19

---

> **[Important Notice]** This software is distributed in **binary form** under the MIT License (free of charge, commercial use permitted). Source code is not publicly available.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Initial Setup (AP Mode)](#2-initial-setup-ap-mode)
3. [Hardware Configuration](#3-hardware-configuration)
4. [Claude Desktop Configuration](#4-claude-desktop-configuration)
5. [MCP Tool Reference](#5-mcp-tool-reference)
   - 5.1 Serial Bridge Tools (Device Control)
   - 5.2 Lua Autonomous Control Tools
   - 5.3 Notification Tools
   - 5.4 System Information Tools
   - 5.5 File System Tools (SPIFFS)
6. [Lua Autonomous Control](#6-lua-autonomous-control)
   - 6.1 Lua API Reference
   - 6.2 Script Chaining (luna.run)
7. [Reset Modes](#7-reset-modes)
8. [Troubleshooting](#8-troubleshooting)
9. [Specifications](#9-specifications)

---

## 1. System Overview

### What is LUNA?

LUNA is a bridge system that uses an ESP32 microcontroller to control any RS-232C device with Claude AI.
It also features an autonomous gateway capability — Lua scripts can continue running independently even when Claude is offline.

```
[Direct Control Mode]
Claude Desktop ── WiFi/MCP ── LUNA(ESP32) ── RS-232C ── Device
                                                         (instruments, industrial equipment, astronomy gear, etc.)

[Lua Autonomous Control Mode]
Claude Desktop ── Designs and saves Lua scripts
                        ↓
                  LUNA runs on its own (Claude not required)
                        ↓
                  Notifies Claude only on completion or error
```

### What You Can Do

| Feature | Description |
|---------|-------------|
| **Send commands to devices** | `serial_query` sends a command and retrieves the response in one step |
| **Receive data from devices** | `serial_read` receives data that devices send spontaneously |
| **Execute Lua scripts** | Run Lua 5.1 scripts asynchronously on the ESP32 |
| **Persist scripts** | `lua_load` saves scripts to SPIFFS — survives power loss |
| **Autonomous script chaining** | `luna.run()` chains multiple scripts without Claude |
| **Async notification to Claude** | `notify.set()` lets scripts alert Claude from within |
| **Auto-delivery of usage guide** | On connection, `prompts/get luna_guide` runs automatically |

### Compatible Devices (Examples)

In principle, any device with an RS-232C port can be controlled.

| Field | Example Devices |
|-------|----------------|
| **Industrial / Manufacturing** | NC machine tools, PLCs, inverters, robot controllers |
| **Measurement / Research** | Multimeters, oscilloscopes, spectrum analyzers, power supplies |
| **Astronomy** | Takahashi Temma2, Celestron, and other telescope mounts |
| **Broadcast / AV** | Switchers, VTR controllers |

---

## 2. Initial Setup (AP Mode)

### Why AP Mode Exists

After purchase or factory reset, LUNA **always boots in AP Mode**.
AP Mode means the device itself becomes a WiFi access point.

1. **To perform initial WiFi configuration**
2. **To check the DHCP-assigned IP address**
3. **To allow reconfiguration when the router changes**

The AP is always active, so you can always connect to the device regardless of its current state.

### First-Time Setup Procedure

#### ① Connect to the AP

In your WiFi settings, select:

```
AiBridgeMCP_Ai-{LocationName}
```

Then open in a browser:

```
http://192.168.4.1
```

*Note: Internet access is unavailable while connected to the AP.*

#### ② Configure Your Home WiFi

1. Open File Manager
2. Enter your router's SSID and password
3. Press Save
4. Restart the device

#### ③ Find the DHCP Address

After reboot, the AP remains active. Reconnect to the AP and open `http://192.168.4.1`:

```
STA Address: 192.168.x.xxx (DHCP)
```

This is the IP address you will register in Claude Desktop.

### How to Open File Manager

| Method | Operation | Active Duration |
|--------|-----------|----------------|
| **At power-on** | Hold GPIO39 LOW while powering on | 30 minutes |
| **During operation** | Briefly pull GPIO39 LOW | 10 minutes |

---

## 3. Hardware Configuration

### Required Components

| Item | Specification | Notes |
|------|---------------|-------|
| ESP32 development board | ESP32-WROOM-32 recommended | LUNA firmware pre-flashed |
| RS-232C level converter | MAX3232 or equivalent | 3.3V ↔ RS-232C conversion (required) |
| RS-232C cable | D-sub 9-pin or 25-pin | Choose based on target device |
| WiFi router | 2.4GHz compatible | With internet connection |
| PC (for Claude Desktop) | Windows / Mac | Claude Desktop installed |

### GPIO Pin Assignment

| GPIO | Function | Notes |
|------|----------|-------|
| GPIO 16 | UART2 RX | Device → ESP32 (via RS-232C) |
| GPIO 17 | UART2 TX | ESP32 → Device (via RS-232C) |
| GPIO 27 | LED | Operation indicator LED |
| GPIO 34 | Reset Mode Select | HIGH = Soft reset / LOW = Hard reset |
| GPIO 39 | File Manager Enable | LOW opens File Manager |
| GPIO 25 | File Manager LED | Indicator while File Manager is active |

### Wiring

```
[LUNA]                      [RS-232C Device]

  TX2 (GPIO17) ──────────────► RXD
  RX2 (GPIO16) ◄────────────── TXD
  GND          ──────────────── GND

  * Connect via a MAX3232 or equivalent RS-232C level converter circuit
  * For devices that require flow control, also connect RTS/CTS
```

### Baud Rate Configuration

| Edition | Baud Rate |
|---------|-----------|
| **LUNA (Free)** | 9600 bps fixed (8N1) |
| **SerialBridge Pro (v4.2.0)** | Configurable via `serial_config` tool |

If your device requires a baud rate other than 9600 bps, consider the Pro edition.

### Level Conversion Circuit (MAX3232)

ESP32 GPIO operates at 3.3V; RS-232C uses ±12V signals. Level conversion is mandatory.

```
[Recommended ICs]
  MAX3232 (3.3V operation, ±15kV ESD protection)
  MAX232  (5V operation, requires 4 external capacitors)

[Basic Connections]
  ESP32 TX2(GPIO17) → MAX3232 T1IN → T1OUT → Device RXD
  ESP32 RX2(GPIO16) → MAX3232 R1OUT ← R1IN ← Device TXD
  ESP32 GND         → MAX3232 GND   ────────── Device GND
```

---

## 4. Claude Desktop Configuration

### Registering the MCP Server

Add the following to Claude Desktop's configuration file (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "LUNA-Lab": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "http://192.168.x.xxx/mcp"
      ]
    }
  }
}
```

Replace `192.168.x.xxx` with the IP address you found in step ②.

> **Note**: The legacy `/mcp/sse` endpoint is not used. LUNA uses `/mcp` (Streamable HTTP).

### Verifying the Connection

Launch Claude Desktop. If LUNA tools (such as `get_system_info`) appear, the connection is successful.

On connection, LUNA automatically runs `prompts/get luna_guide`, so Claude automatically learns how to use LUNA.

### Providing the Device Manual to Claude

Paste the device manual into the Claude Desktop chat window:

```
I will control the following device via RS-232C through LUNA.

[Device Info]
  Device: [Manufacturer] [Model]
  Communication: 9600bps, 8N1
  Command set: SCPI-compliant (or proprietary)

[Manual]
  [Attach PDF, or paste the command list as text]

Please read and understand the manual above, then control the device according to my instructions.
```

**Tip**: Including the command list, response format, and initialization procedure is most effective.

### If the IP Address Changes

In a DHCP environment, the IP address may change.

1. Connect to AP mode (`http://192.168.4.1`) to find the new IP address
2. Update the IP in `claude_desktop_config.json` and save
3. Restart Claude Desktop

**Permanent fix**: Use your router's "DHCP reservation (static assignment)" feature to fix LUNA's IP address.

---

## 5. MCP Tool Reference

A complete list of tools available from Claude Desktop to LUNA.

---

### 5.1 Serial Bridge Tools (Primary Device Control)

#### `serial_query` (Primary Tool)

Sends a command, waits for the specified time, then receives the response.

| Item | Details |
|------|---------|
| **Parameters** | `content` (required): Command string to send |
| | `timeout_ms` (optional): Response wait time in milliseconds. Default: 500ms |
| **Returns** | Response string from the device |
| **When to use** | Typical RS-232C communication: send a command and receive a response |

```
Examples:
  serial_query("MEAS:VOLT:DC?\r\n")
    → returns "+1.2345E+00\r\n"

  serial_query("*IDN?\r\n", timeout_ms=1000)
    → returns "HEWLETT-PACKARD,34401A,..."
```

**Tip**: Match the line ending (`\r\n` or `\r`) to your device's specification.

---

#### `serial_write`

**Sends** data to the serial port only — no response expected.

| Item | Details |
|------|---------|
| **Parameters** | `content` (required): String to send |
| **Returns** | Number of bytes sent |
| **When to use** | One-way transmission: configuration commands, data upload, etc. |

---

#### `serial_read`

**Reads** whatever data has accumulated in the serial receive buffer.

| Item | Details |
|------|---------|
| **Parameters** | None |
| **Returns** | Contents of the receive buffer (empty string if no data) |
| **When to use** | When the device sends data spontaneously |

**Note**: In most cases, `serial_query` is simpler. Use `serial_read` only when needed.

---

#### `serial_config` (SerialBridge Pro only)

Changes the serial baud rate and parity.
**Not available in LUNA Free edition (returns an error).**

| Item | Details |
|------|---------|
| **Parameters** | `baud` (required): Baud rate |
| | `parity` (optional): `N` = None (default), `E` = Even, `O` = Odd |
| **Supported baud rates** | 300 / 600 / 1200 / 2400 / 4800 / 9600 / 19200 / 38400 / 115200 |

---

### 5.2 Lua Autonomous Control Tools (LUNA New Features)

> **📖 See Companion Guide**: For detailed Lua programming (API reference, sample code, language guide),
> refer to **`LUNA_Lua_Guide.md`** (Japanese) or **`LUNA_Lua_Guide_EN.md`** (English).

#### `lua_exec`

Executes Lua code immediately, in place. Use for testing and one-time operations.

| Item | Details |
|------|---------|
| **Parameters** | `code` (required): Lua 5.1 source code |
| **Returns** | `{"status":"started"}` (asynchronous — returns immediately) |
| **When to use** | Verification, testing, simple one-time execution |

```lua
-- Example: get device ID
lua_exec('local r = serial.query("*IDN?\\r\\n", 1000); log.write(r)')
```

Retrieve results with `lua_status` or `lua_log`.

---

#### `lua_load`

Saves a Lua script to SPIFFS. Persists across power loss.

| Item | Details |
|------|---------|
| **Parameters** | `name` (required): Script name (alphanumeric, `_`, `-` only; max 22 chars) |
| | `code` (required): Lua 5.1 source code |
| **Saved as** | `/lua_{name}.lua` |

---

#### `lua_run`

Runs a previously saved script.

| Item | Details |
|------|---------|
| **Parameters** | `name` (required): Script name saved with `lua_load` |
| **Returns** | `{"status":"started"}` (asynchronous — returns immediately) |
| **When to use** | Scheduled runs, production operation |

---

#### `lua_status`

Checks the Lua VM execution state.

| Item | Details |
|------|---------|
| **Returns** | `state` (running/idle), `last_result`, `free_heap`, `uptime` |
| | `chain_index`, `chain_script` (progress of `luna.run()` chain) |

---

#### `lua_stop`

Forcibly stops a running Lua task.

---

#### `lua_log`

Retrieves logs written by `log.write()` in scripts (ring buffer of last 20 lines).

---

#### `lua_list`

Lists all Lua scripts saved in SPIFFS.

---

### 5.3 Notification Tools

#### `check_notify`

Checks the notification flag set by `notify.set()` in a Lua script.

| Item | Details |
|------|---------|
| **Parameters** | None |
| **Returns** | `{"flag":0}` or `{"flag":1, "message":"..."}` |

**Usage**: When a Lua script calls `notify.set("message")`, Claude can detect it with `check_notify`. The flag resets automatically after being read.

---

### 5.4 System Information Tools

| Tool | Function | Parameters |
|------|----------|------------|
| `get_system_info` | ESP32 system info (location, chip model, heap, SPIFFS usage, uptime) | None |
| `get_network_status` | WiFi / network status (location, AP SSID, STA IP, RSSI) | None |
| `get_license_info` | License, edition, and author info | None |
| `hard_reset` | Restart ESP32 (`ESP.restart()` after 1 second) | None |

---

### 5.5 File System Tools (SPIFFS)

| Tool | Function | Parameters |
|------|----------|------------|
| `fs_list` | List files in SPIFFS | None |
| `fs_read` | Read a file (max 4KB) | `path` (required) |
| `fs_write` | Write a file (max 1KB, overwrites) | `path`, `content` (required) |
| `fs_append` | Append to a file (max 1KB) | `path`, `content` (required) |
| `fs_delete` | Delete a file | `path` (required) |

> **Note**: `fs_*` tools are called by Claude via MCP. They cannot be called from within Lua scripts.
> To write to SPIFFS from Lua, use `log.save()`.

---

### Tool Selection Cheat Sheet

```
[Send a command to the device and receive a response]
  → serial_query  (adjust timeout_ms to match your device)

[Push a program or configuration one-way to the device]
  → serial_write  (repeat line by line)

[Receive data that the device sends spontaneously]
  → serial_read

[Test a Lua script]
  → lua_exec

[Save and run a Lua script in production]
  → lua_load → lua_run

[Check Lua script execution status]
  → lua_status

[Check Lua script log output]
  → lua_log

[Stop a Lua script mid-execution]
  → lua_stop

[List saved scripts]
  → lua_list

[Notify Claude of completion or error from a Lua script]
  → call notify.set() in script → Claude polls with check_notify

[Verify LUNA is operating normally]
  → get_system_info or get_network_status

[Check data saved by log.save() from Claude]
  → fs_read("/luna_log.txt")
```

---

## 6. Lua Autonomous Control

> **📖 See Companion Guide**: For detailed Lua programming (API reference, sample code, language guide),
> refer to **`LUNA_Lua_Guide.md`** (Japanese) or **`LUNA_Lua_Guide_EN.md`** (English).

This is the autonomous control feature that keeps the ESP32 running on its own without Claude.

### Basic Workflow

```
① Tell Claude the script specification
  "Write a script that measures voltage every 30 seconds
   and sends a notification if it goes below 1.0V or above 1.5V"

② Claude generates, saves, and runs the Lua script
  lua_load("voltage_monitor", <script>) → saved to SPIFFS
  lua_run("voltage_monitor")            → execution starts

③ Claude can disconnect (ESP32 continues running on its own)

④ On error
  notify.set() fires
  Claude detects it with check_notify
  Reports "Voltage anomaly: 0.8234V"
```

### Lua Script Example (Voltage Monitor)

```lua
-- Voltage monitoring script (voltage_monitor)
for i = 1, 120 do
  local res = serial.query("MEAS:VOLT:DC?\r\n", 1000)
  local v = tonumber(res)
  if v then
    log.write(string.format("%.4fV", v))
    if v < 1.0 or v > 1.5 then
      notify.set("Voltage anomaly: " .. string.format("%.4f", v) .. "V")
      break
    end
  end
  hw.delay(30000)  -- wait 30 seconds
end
```

---

### 6.1 Lua API Reference

APIs that give Lua scripts access to ESP32 hardware features.

#### serial module

| API | Description |
|-----|-------------|
| `serial.query(cmd, timeout_ms)` | Send command → wait → receive. Primary API |
| `serial.write(data)` | Send only |
| `serial.read()` | Read receive buffer |

**Note**: The serial module and serial MCP tools (`serial_query`, etc.) share the same serial port. Avoid using both simultaneously while a Lua script is running.

#### log module

| API | Description |
|-----|-------------|
| `log.write(msg)` | Write to the log buffer (Claude reads with `lua_log`) |
| `log.read([n])` | Read from the log buffer (for inter-script data passing) |
| `log.save()` | Save log to `/luna_log.txt` in SPIFFS (**do not call inside a loop**) |
| `log.load()` | Load from SPIFFS into buffer (clears buffer first) |

**Two roles of the log module**:

| Purpose | Operation |
|---------|-----------|
| Script → Claude | Write with `log.write()` → Claude reads with `lua_log` |
| Script → Script | `log.write()` + `log.save()` → next script reads with `log.load()` |

#### hw module

| API | Description |
|-----|-------------|
| `hw.delay(ms)` | Wait for the specified number of milliseconds |
| `hw.millis()` | Elapsed time since boot (ms) |
| `hw.reset()` | Restart the ESP32 |

#### notify module

| API | Description |
|-----|-------------|
| `notify.set(msg)` | Set a notification message for Claude. Detected via `check_notify` |

#### luna module

| API | Description |
|-----|-------------|
| `luna.run(name)` | Queue the next script and exit the current one (chain max: 10) |

---

### 6.2 Script Chaining (luna.run)

Using `luna.run(name)`, a script can autonomously launch the next script.
This allows multi-step sequences like "initialize → measure → report" to complete entirely without Claude.

```lua
-- Script 1: init (initialization)
serial.query("*RST\r\n", 2000)
serial.query("CONF:VOLT:DC\r\n", 500)
log.write("init_ok")
luna.run("measure")   -- queue next script and exit
```

```lua
-- Script 2: measure (measurement)
local results = {}
for i = 1, 10 do
  local r = serial.query("MEAS:VOLT:DC?\r\n", 1000)
  table.insert(results, r)
  hw.delay(1000)
end
log.write(table.concat(results, ","))
log.save()            -- save to SPIFFS (pass to next script)
luna.run("report")    -- proceed to next script
```

```lua
-- Script 3: report (aggregation and notification)
log.load()            -- receive measurement data (clears buffer then loads)
local data = log.read()
-- (aggregation processing)
notify.set("Measurement complete: " .. data)
```

**Chain Specifications**:

| Item | Specification |
|------|---------------|
| Maximum chain length | 10 scripts |
| Method | "Queue and exit" (next script starts after current one ends) |
| Progress check | `lua_status` → `chain_index` / `chain_script` |

---

## 7. Reset Modes

### Two Types of Reset

#### Soft Reset

> **"Restart only the MCP connection — ESP32 keeps running"**

- ESP32 continues operating
- WiFi connection is maintained
- SPIFFS data is preserved
- MCP connection drops, and Claude Desktop reconnects automatically

**When it triggers**: Automatically fires when no AI communication occurs for 5 minutes (300 seconds) AND **GPIO34 is HIGH**.

#### Hard Reset

> **"Fully restart the ESP32 — equivalent to cycling the power"**

- WiFi reconnects (SSID / password reloaded from SPIFFS)
- SPIFFS data (WiFi config, Lua scripts, etc.) is preserved
- RAM contents (running Lua tasks, etc.) are all cleared
- MCP server restarts automatically after reboot

**Three triggers for Hard Reset**:

| Method | Operation | Delay |
|--------|-----------|-------|
| MCP tool `hard_reset` | Called by Claude | 1 second |
| SSE timeout (GPIO34 = LOW) | Triggers automatically | 1 second |

### Reset Mode Selection via GPIO34

Selects behavior when AI communication is lost for 5 minutes.

| GPIO34 State | Reset Type |
|-------------|-----------|
| HIGH (pull-up / open) | Soft Reset (default) |
| LOW (connected to GND) | Hard Reset |

### Soft Reset vs. Hard Reset Comparison

| Item | Soft Reset | Hard Reset |
|------|-----------|-----------|
| ESP32 restart | No | Yes |
| WiFi connection | Maintained | Reconnects (3–10 seconds) |
| Running Lua tasks | Continue | **Stopped** |
| SPIFFS data | Preserved | Preserved |
| MCP connection | Drops → auto-reconnects | Drops → reconnects after reboot |

### Recommended Settings for Long-Term Unattended Operation

For observatories, factory floors, or any installation running for extended periods without human intervention:

```
GPIO34 = LOW (Hard Reset Mode) — recommended

Reason:
  Communication lost for 5 minutes (network blip / AI session timeout)
        ↓
  Hard Reset triggers (ESP32 fully restarts)
        ↓
  WiFi reconnects → MCP server restarts
        ↓
  Automatic recovery

Memory leaks and stack corruption are also cleared on restart,
improving long-term operational stability.
```

---

## 8. Troubleshooting

### Cannot Communicate with Device

```
Checklist:
  □ Does the baud rate match the device? (LUNA Free is fixed at 9600)
  □ Is the crossover / straight-through cable choice correct?
  □ Is GND connected?
  □ Is the level converter (MAX3232, etc.) wired and working correctly?
  □ Does the device require flow control (RTS/CTS)?
```

### Command Sent but No Response

```
Checklist:
  □ Is the line ending correct? (CR / LF / CR+LF — check device spec)
  □ What is the device's echo-back setting?
  □ Is the wait time sufficient? (try increasing timeout_ms)
  □ Does the device need an initialization command sent first?
```

### Lua Script Not Running

```
Checklist:
  □ Check lua_status for state (running / idle / error)
  □ Check lua_log for log.write() output from the script
  □ Is the timeout in serial.query() too short?
  □ Does the luna.run() chain exceed LUA_MAX_CHAIN (10)?
  □ Is log.save() being called inside a loop? (risk to flash lifespan)
```

### Unstable WiFi Connection

```
Checklist:
  □ Distance and obstacles between ESP32 and router
  □ Radio interference on the 2.4GHz band
  □ GPIO34 setting (Hard Reset Mode recommended for long-term use)
```

### MCP Not Connecting

```
Checklist:
  □ Has the IP address changed? (check via AP mode)
  □ Is the URL in claude_desktop_config.json correct? (use /mcp)
  □ Restart Claude Desktop
  □ Is LUNA powered on?
```

### ESP32 Becomes Unresponsive

```
Steps:
  1. Call the hard_reset tool from Claude
  2. Unplug and replug the USB cable to power-cycle
  3. Set GPIO34 LOW (Hard Reset Mode) and power on
```

---

## 9. Specifications

### Firmware Specifications

| Item | Specification |
|------|--------------|
| Brand Name | LUNA |
| Version | v3.9.1 |
| Platform | ESP32 (Arduino framework) |
| Protocol | MCP (Model Context Protocol) Streamable HTTP |
| Web Server | Port 80 |
| MCP Endpoint | `/mcp` |
| Lua Engine | Lua 5.1 |
| Lua RAM Usage | ~25KB |
| Lua Startup Time | ~9ms |
| Max Script Chain Length | 10 |
| Log Buffer | Last 20 lines (ring buffer) |
| SPIFFS | WiFi config, Lua scripts, logs |

### Serial Communication Specifications (Free Edition)

| Item | Value |
|------|-------|
| Baud Rate | **9600 bps** |
| Data Bits | **8 bits** |
| Stop Bits | **1 bit** |
| Parity | **None (N)** |
| Flow Control | **None** |

### MCP Tool Summary

| Category | Tools |
|----------|-------|
| **Serial Bridge** | serial_query / serial_write / serial_read / serial_config (Pro only) |
| **Lua Autonomous Control** | lua_exec / lua_load / lua_run / lua_status / lua_stop / lua_log / lua_list |
| **Notification** | check_notify |
| **System Information** | get_system_info / get_network_status / get_license_info / hard_reset |
| **File System** | fs_list / fs_read / fs_write / fs_append / fs_delete |

### Document List

| File | Purpose |
|------|---------|
| `LUNA_README.md` | Quick start and connection procedure |
| `LUNA_USER_MANUAL_EN.md` | This file (complete English manual) |
| `LUNA_Lua_Guide_EN.md` | RS-232C device control concepts, Lua usage guide (English) |
| `LUNA_SERIAL_BRIDGE_GUIDE_EN.md` | Serial bridge detailed guide (English) |
| `Temma_Protocol_Spec.md` | Takahashi Temma2 protocol specification |

---

*LUNA (AiBridgeMCP v3.9.1) / MIT License*
*(C) 2026 Nishioka Sadahiko*
*Blog   : https://nskikaku.blog.fc2.com/*
*GitHub : https://github.com/OnStepNinja/LUNA*

---

Disclaimer:
This system (LUNA / AiBridgeMCP) is provided "as is" under the MIT License.

Use at your own risk: All use of this device and its accompanying software is entirely at the user's own risk.

No liability for damages: The author assumes no liability whatsoever for any direct or indirect damages arising from the use or inability to use this device — including but not limited to device failure, circuit damage, data loss, or any other loss — regardless of the cause.

Acknowledgment: By connecting this device to RS-232C equipment, the user is deemed to have fully understood the associated electrical risks and to have agreed to these terms.

---

Legal Disclaimer:
This product is an independently developed hardware by OnStepNinja and is not affiliated with, endorsed by, or sponsored by Anthropic, Takahashi Seisakusho, or any other company.
"Temma", "MCP" and other product names are trademarks of their respective owners.
Use of these names is solely for the purpose of describing compatibility (Nominative Fair Use).

本プロジェクトは各社とは無関係の個人によるプロジェクトです。
