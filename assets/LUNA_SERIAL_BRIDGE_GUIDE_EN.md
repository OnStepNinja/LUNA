# LUNA — AI Serial Bridge Guide
## A New Paradigm for Autonomous AI Control of RS-232C Devices

**Audience**: Engineers and technical users
**Version**: LUNA (AiBridgeMCP v3.9.1)
**Revision**: r1.0
**Last Updated**: 2026-03-19
**Created**: 2026-03-19

---

> **[Important Notice]** This software is distributed in **binary form** under the MIT License (free of charge, commercial use permitted). Source code is not publicly available.

---

## Introduction

LUNA was developed as a bridge connecting ESP32 and Claude AI.
But its true potential goes much further.

> **Any device with an RS-232C port can be directly controlled by AI.**
> **Furthermore, Lua scripts keep running autonomously even when Claude is offline.**

Factory production equipment, laboratory instruments, medical devices, astronomical equipment —
everything that previously required dedicated software and specialized knowledge
**becomes an AI system simply by handing over the manual.**

---

---

# Part One: Why This Is Revolutionary

---

## 1. The World Before and After

### The Traditional Approach

```
[Getting a new device up and running]

  Purchase device
      ↓
  Expert reads the manual              ← days to weeks
      ↓
  Learn the command set                ← more weeks
      ↓
  Write a control program              ← weeks to months
      ↓
  Test and debug                       ← more weeks
      ↓
  Finally "ready to use"

  Total: months to years, millions of yen in costs
  Problem: when the engineer leaves, nobody can use it
```

### The LUNA + AI Approach

```
[Getting a new device up and running]

  Purchase device
      ↓
  Connect to LUNA via RS-232C cable       ← minutes
      ↓
  Hand the manual (PDF, etc.) to AI       ← minutes
      ↓
  Say "do ○○ with this device"
  or write a Lua script for autonomous    ← runs even without Claude
  execution

  Total: tens of minutes, near-zero additional development cost
  Advantage: anyone can use it, anytime, as long as the manual exists
```

### The Essence of the Paradigm Shift

Until now, **humans had to learn the machine's language**.

- Memorize assembly language
- Learn command sets
- Implement protocols

LUNA **reverses that relationship**.

```
Before: humans learn the machine's language
         ↓
New way: AI learns the machine's language, humans speak naturally
         Furthermore: Lua scripts keep controlling devices autonomously
```

**AI leaps over 40 years of "machine language barriers" in an instant.**

---

## 2. The Impact of "Just Hand Over the Manual"

### Why a Manual Is All It Takes

AI has vast linguistic knowledge.
RS-232C command systems, serial communication protocols, device control methods —
all of this can be described as text.

So when you give the device manual to AI:

```
What the manual says:
  "MEAS:VOLT:DC?" → command to measure DC voltage
  Response format: "+1.2345E+00\r\n"

What AI understands:
  "To measure DC voltage, send MEAS:VOLT:DC?"
  "The response comes back as a number in scientific notation"

What the user says:
  "Measure the DC voltage"

What AI does:
  Sends MEAS:VOLT:DC? to serial port → receives response → interprets and reports result
```

### Months of Development Becomes Minutes

| | Traditional | LUNA |
|---|---|---|
| Setup time | Weeks to months | Minutes (just hand over the manual) |
| Required skills | Programming, protocol knowledge | Ability to speak naturally |
| When changing devices | New development required | Just hand over the new manual |
| When expert is absent | Nobody can use it | Anyone can use it if the manual exists |
| Cost | High (development cost) | Low (hardware only) |
| **Unattended/overnight operation** | **Requires separate system** | **Autonomous execution via Lua scripts** |

### Solving the Knowledge Silo Problem

There is a serious problem in the technical world.

```
[A common tragedy]

  Veteran engineer A:
    "With this instrument, you first send *ESE,
     then use WAI to synchronize..."

  Engineer A retires
      ↓
  Nobody can operate it anymore
      ↓
  Expensive instrument goes to storage
      ↓
  Scrapped
```

With LUNA:

```
  As long as the manual survives
      ↓
  AI understands all the procedures
      ↓
  The device can be used forever
```

**What has been lost because "the person who knew how" is gone — can now be rescued.**

---

## 3. System Architecture

### Overall System Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        User Environment                      │
│                                                             │
│  ┌──────────────────┐          ┌──────────────────────────┐ │
│  │  Claude Desktop  │          │   Device Manual (PDF etc) │ │
│  │                  │◄────────►│   ← into AI context      │ │
│  │  MCP Client      │          └──────────────────────────┘ │
│  └────────┬─────────┘                                       │
│           │ MCP / HTTP (WiFi LAN)  ← Streamable HTTP        │
└───────────┼─────────────────────────────────────────────────┘
            │
┌───────────┼─────────────────────────────────────────────────┐
│           ▼          LUNA (ESP32)                           │
│  ┌─────────────────┐                                        │
│  │  MCP Server     │  ← MCP protocol handler (Streamable HTTP) │
│  │  (port 80 /mcp) │                                        │
│  │                 │                                        │
│  │  MCP Tools      │  ← serial_query / serial_write         │
│  │                 │     serial_read / serial_config        │
│  │                 │     get_system_info / hard_reset       │
│  │                 │     fs_list / fs_read / fs_write etc   │
│  │                 │                                        │
│  │  ┌───────────┐  │  ← Lua VM (v3.9.1 new feature)        │
│  │  │ Lua 5.1  │  │     lua_exec / lua_load / lua_run      │
│  │  │  Engine  │  │     lua_status / lua_log               │
│  │  │          │  │     luna.run() script chaining         │
│  │  └───────────┘  │                                        │
│  │                 │                                        │
│  │  Serial Bridge  │  ← RS-232C send/receive               │
│  └────────┬────────┘                                        │
└───────────┼─────────────────────────────────────────────────┘
            │ RS-232C (serial cable)
            │
┌───────────┼─────────────────────────────────────────────────┐
│           ▼          Controlled Devices                      │
│  ┌─────────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │  Instruments    │  │  Factory    │  │  Astronomical   │  │
│  │  (GPIB→RS232)   │  │  Equip(PLC) │  │  (Temma2, etc.) │  │
│  └─────────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Role of Each Component

| Component | Role |
|---|---|
| **Claude Desktop** | AI brain. Reads manuals and determines control strategy |
| **MCP (Model Context Protocol)** | Standard communication protocol between AI and LUNA (Streamable HTTP) |
| **ESP32 (LUNA)** | WiFi ↔ RS-232C conversion bridge + Lua autonomous execution engine |
| **Lua VM** | Autonomous control scripting engine that runs without Claude (v3.9.1 new feature) |
| **RS-232C** | Physical communication path to devices (40+ year standard) |
| **Device Manual** | The "instruction manual" for AI. With this, control is possible |

### Why ESP32?

```
Why we chose ESP32:
  ✅ Built-in WiFi      → no cables needed, install anywhere
  ✅ Inexpensive (~$3)  → low cost to place one per device
  ✅ RS-232C support    → easily connected via MAX3232, etc.
  ✅ Built-in SPIFFS    → persistent storage for config, data, Lua scripts
  ✅ Widely available   → purchasable worldwide
  ✅ Proven stability   → suitable for industrial use
  ✅ Lua VM capable     → Lua 5.1 runs in 25KB RAM
```

### Why MCP?

```
Why we chose MCP:
  ✅ Official standard protocol from Anthropic
  ✅ Native integration with Claude Desktop
  ✅ Can support other AIs in the future
  ✅ Extensible with Tools
  ✅ Real-time response via Streamable HTTP (v3.9.1)
  ✅ prompts/list and prompts/get auto-notify LUNA usage to Claude
```

---

## 4. Compatible Devices

Any device with an RS-232C port can in principle be controlled.

### Industrial / Manufacturing

| Device Category | Examples | Use Scenarios |
|---|---|---|
| **CNC Machine Tools** | Fanuc, Mitsubishi CNC | Program transfer, machining status monitoring |
| **PLCs** | OMRON, Mitsubishi PLCs | Sequence control, status verification |
| **Inverters** | Various motor drivers | Speed control, fault diagnosis |
| **Robot Controllers** | Industrial robot controllers | Motion program transfer, status monitoring |

### Measurement / Research

| Device Category | Examples | Use Scenarios |
|---|---|---|
| **Multimeters** | Hewlett-Packard, Keithley | Automated measurement, data collection |
| **Oscilloscopes** | Tektronix, Yokogawa | Waveform capture, automated analysis |
| **Spectrum Analyzers** | Anritsu, Rohde & Schwarz | Frequency analysis, automated scanning |
| **Power Supplies** | Various programmable supplies | Automated voltage/current control |
| **Temperature Controllers** | Thermal chambers, furnaces | Temperature profile control |

### Astronomy / Specialty

| Device Category | Examples | Use Scenarios |
|---|---|---|
| **Telescope Mounts** | Takahashi Temma2, Celestron, etc. | GoTo, auto-tracking control |
| **Spectrographs** | Astronomical spectroscopes | Spectrum acquisition and analysis |
| **Weather Instruments** | Various sensors and loggers | Data collection and analysis |

### Legacy Computers / AV

| Device Category | Examples | Use Scenarios |
|---|---|---|
| **Retro PCs** | MSX, PC-8801, PC-9801 | Program transfer, AI chat |
| **Broadcast Equipment** | Switchers, VTR controllers | Automated control, logging |
| **Printers / Plotters** | Legacy large-format printers | Data transfer and print control |

---

---

# Part Two: Practical Use

---

## 5. Hardware Connection

### What You Need

```
① LUNA (ESP32 board)                        ← this product
② RS-232C cable (crossover or straight, depending on device)
③ Power supply (USB 5V)
④ WiFi router (existing one is fine)
⑤ PC with Claude Desktop installed
⑥ Manual for the device you want to control (PDF, etc.)
```

### Wiring

```
[LUNA]                      [RS-232C Device]

  TX2 (GPIO17) ─────────────► RXD
  RX2 (GPIO16) ◄───────────── TXD
  GND          ─────────────── GND

  ※ Connect via RS-232C level converter circuit (MAX3232, etc.)
  ※ Connect RTS/CTS if flow control is required
```

### Baud Rate Settings

| Edition | Setting |
|---|---|
| **LUNA (Free)** | Fixed at 9600 bps (8N1) |
| **SerialBridge Pro (v4.2.0)** | Configurable via serial_config tool |

If your device requires a different baud rate, consider the Pro edition.

---

## 6. Step-by-Step: AI Device Control

### Step 1: Initial LUNA Setup

Power on LUNA and configure WiFi and Location Name.
See **LUNA_README.md** for details.

```
After setup, verify:
  WiFi SSID: AiBridgeMCP_Ai-{LocationName}
  MCP URL:   http://{IPAddress}/mcp
```

### Step 2: Register MCP Server in Claude Desktop

Add to Claude Desktop's configuration file (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "LUNA-Lab": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "http://192.168.1.100/mcp"
      ]
    }
  }
}
```

> **Note**: The `/mcp/sse` endpoint from v1.8.8 and earlier is the old format. LUNA uses `/mcp` (Streamable HTTP).

### Step 3: Hand the Manual to AI

In the Claude Desktop chat window, give it the device manual.

```
Method A: Drag & drop the PDF
  → Hand the manual (PDF) directly to Claude Desktop

Method B: Paste as text
  → Copy and paste the command list section from the manual

Method C: Describe key points as text
  → "This device connects via RS-232C, the commands are..."
```

**Key point**: Most effective when the command list, response format, and initialization procedure are included.

> **With LUNA**: On connection, `prompts/get luna_guide` is automatically executed, so Claude automatically understands how to use LUNA (serial.* tools, Lua scripts, etc.).

### Step 4: Issue Instructions in Natural Language

From here, just give AI instructions in normal conversation.

```
Examples:
  User: "Measure the DC voltage"
  AI:   Sends MEAS:VOLT:DC?
         → Receives response → "The DC voltage is 1.2345V"

  User: "Measure voltage 10 times at 1-second intervals and calculate the average"
  AI:   10 consecutive measurements → collects data → calculates and reports average

  User: "Let me know if the voltage is out of spec (below 1.0V or above 1.5V)"
  AI:   Measures → evaluates → alerts only on anomaly
```

### Step 5 (LUNA New Feature): Autonomous Control with Lua Scripts

Without relying on Claude, Lua scripts on ESP32 keep running independently.

```
Example:
  User: "Create a script that measures voltage every minute
         and notifies me if there's an anomaly"
  AI:   Generates Lua script → saves with lua_load → starts with lua_run
        → From here, ESP32 monitors autonomously even without Claude
        → On anomaly, notify.set() notifies Claude
```

---

## 7. Practical Examples

### Example A: Automated Multimeter Measurement (Claude Direct Control)

**Scenario**: Automated voltage measurement with a Hewlett-Packard multimeter (SCPI-compatible)

**Example instruction to Claude**:
```
Here is the manual for this multimeter. [attach manual]

It is connected via RS-232C through LUNA.
Please do the following:
1. Initialize the device
2. Set to DC voltage measurement mode
3. Measure voltage every 5 seconds, recording 10 data points
4. Report the maximum, minimum, and average values
```

**AI operation**:
```
[Command sequence AI executes automatically]

*RST          ← initialize
CONF:VOLT:DC  ← set DC voltage measurement mode
MEAS:VOLT:DC? ← measure (repeat 10 times)
(5 second wait)
MEAS:VOLT:DC?
... (repeat)

[Claude's report]
Measurement complete.
  Maximum: 1.3456 V (3rd measurement)
  Minimum: 1.2201 V (7th measurement)
  Average: 1.2890 V
  Std dev: 0.0312 V
```

---

### Example B: Autonomous Monitoring with Lua Script (LUNA v3.9.1 New Feature)

**Scenario**: Periodic voltage monitoring without Claude, notification on anomaly

**Example instruction to Claude**:
```
Write a Lua script to LUNA that measures voltage every 30 seconds
and notifies me if it goes below 1.0V or above 1.5V.
```

**Lua script generated and saved by AI**:
```lua
-- Voltage monitoring script (voltage_monitor)
for i = 1, 20 do
  local res = serial.query("MEAS:VOLT:DC?\r\n", 1000)
  local v = tonumber(res)
  if v then
    log.write(string.format("%.4fV", v))
    if v < 1.0 or v > 1.5 then
      notify.set("Voltage anomaly: " .. string.format("%.4f", v) .. "V")
      break
    end
  end
  hw.delay(30000)
end
```

**Operation flow**:
```
lua_load("voltage_monitor", <script>) → saved to SPIFFS
lua_run("voltage_monitor")            → execution started
                                      → Claude can step away
(on anomaly)
notify.set() fires
  → Claude detects with check_notify
  → Reports "Voltage anomaly: 0.8234V"
```

---

### Example C: Script Chaining with luna.run() (LUNA v3.9.1 New Feature)

**Scenario**: Autonomous execution of initialize → measure → report across 3 scripts

```lua
-- Script 1: init (initialization)
serial.query("*RST\r\n", 2000)
serial.query("CONF:VOLT:DC\r\n", 500)
log.write("init_ok")
luna.run("measure")   -- auto-launch next script and exit
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
log.save()            -- save to SPIFFS (hand off to report script)
luna.run("report")    -- proceed to next script
```

```lua
-- Script 3: report (aggregation and notification)
log.load()            -- receive measurement data
local sum = 0
local n = 0
-- (aggregation processing)
notify.set(string.format("Measurement complete: avg %.4fV", sum/n))
```

---

### Example D: Program Transfer to Retro PC (MSX)

**Scenario**: AI creates an MSX BASIC program and transfers it to the MSX

**Example instruction to Claude**:
```
Create a program in MSX BASIC to blink an LED,
and send it to the MSX via LUNA's serial bridge.
```

**AI operation**:
```
[Generate MSX BASIC program]
10 OUT &HFF,1
20 FOR I=1 TO 1000:NEXT I
30 OUT &HFF,0
40 FOR I=1 TO 1000:NEXT I
50 GOTO 10

[Send line by line via serial bridge]
→ Send newline code after each line
→ MSX BASIC interpreter receives and inputs
→ Start execution with RUN command
```

---

### Example E: Coordinated Control of Multiple Devices

Using multiple LUNA units, you can control multiple devices simultaneously.

```
[Example: Automated Environmental Testing]

  LUNA-Chamber  → temperature control for thermal chamber
  LUNA-Meter    → voltage and current measurement
  LUNA-Logger   → data logger

[Instruction to Claude]
"Vary the temperature from -20°C to +85°C in 10°C steps,
 stabilize at each temperature for 5 minutes,
 then measure voltage and current,
 and record in the data logger"

→ Claude coordinates 3 LUNA units to run automated testing
```

---

## 8. Direct Control vs Lua Autonomous Control

LUNA has two control modes.

### Direct Control (AI-driven direct control)

```
[Use Case]
  Cases where AI takes the lead in controlling devices
  → Automated measurement instrument control
  → Industrial equipment automation
  → Sequential execution per manual

[Flow]
  Hand manual to Claude Desktop
      ↓
  Issue natural language instruction "please do ○○"
      ↓
  AI sends commands via MCP tools
      ↓
  Device responds → AI interprets and reports

[MCP tools used]
  serial_query  ← send → wait → receive in one call (primary tool)
  serial_write  ← send only (when no response needed)
  serial_read   ← receive only (for data spontaneously sent by device)
```

### Lua Autonomous Control (v3.9.1 New Feature)

```
[Use Case]
  Cases where ESP32 runs independently without Claude
  → Periodic monitoring / log collection
  → Overnight / unattended operation
  → Condition-triggered actions

[Flow]
  Claude designs and saves Lua script
      ↓
  Launch script with lua_run
      ↓
  ESP32 runs independently (no Claude needed)
      ↓
  On completion or anomaly, notify Claude with notify.set()

[MCP tools used]
  lua_load    ← save script to SPIFFS
  lua_run     ← execute saved script
  lua_exec    ← execute immediately on the spot (for testing)
  lua_status  ← check execution state and chain progress
  lua_log     ← read log written by script
  lua_stop    ← force stop running script
```

### When to Use Which

| | Direct Control | **Lua Autonomous Control** |
|---|---|---|
| **Driven by** | AI (Claude) | ESP32 (Lua VM) |
| **Typical use** | Instruments, industrial control | **Periodic monitoring, unattended** |
| **Claude not needed** | ✗ | **✅** |
| **Complex sequences** | ✅ | ✅ |
| **Real-time performance** | High | **Highest (local execution)** |
| **Script chaining** | ✗ | **✅ luna.run()** |

---

## 9. Troubleshooting

### Cannot Communicate

```
Check:
  □ Does baud rate match the device? (LUNA Free is fixed at 9600)
  □ Is crossover vs straight cable selection correct?
  □ Is GND connected?
  □ Is the level converter circuit (MAX3232, etc.) working correctly?
  □ Does the device require flow control (RTS/CTS)?
```

### Command Sent but No Response

```
Check:
  □ Is the line terminator at the end of the command correct? (CR/LF/CR+LF)
  □ What is the device's echo-back setting?
  □ Is the wait time after sending sufficient? (increase timeout_ms)
  □ Does an initialization command need to be sent first?
```

### Timeouts Occurring

```
Solutions:
  □ Check device response time and adjust timeout_ms
  □ Tell AI "wait ○ seconds for response before sending next command"
  □ Test device response speed before complex processing
```

### WiFi Connection Unstable

```
Check:
  □ Distance and obstacles between ESP32 and router
  □ MCP connection timeout settings
  □ GPIO34 configuration (soft reset vs hard reset)
```

### Lua Script Not Working

```
Check:
  □ Check state with lua_status (running / idle / error)
  □ Check log.write() output in script with lua_log
  □ Is serial.query timeout (ms) too short?
  □ Is luna.run() chain exceeding LUA_MAX_CHAIN (10)?
  □ Is log.save() being called inside a loop? (watch flash lifetime)
```

---

---

# Appendix

---

## Appendix A: Complete MCP Tool Reference

Complete list of tools available from Claude Desktop to LUNA.

---

### 🔌 Serial Bridge Tools (Primary tools for device control)

#### `serial_query` (Most Important Tool)

Sends a command, waits for the specified time, then receives the response.

| Item | Details |
|------|---------|
| **Parameters** | `content` (required): command string to send |
| | `timeout_ms` (optional): response wait time in milliseconds. Default 500ms |
| **Return** | Response string from device |
| **Use when** | Typical RS-232C communication: send command, device responds |

```
Usage examples:
  serial_query("MEAS:VOLT:DC?\r\n")
    → returns "+1.2345E+00\r\n"

  serial_query("*IDN?\r\n", timeout_ms=1000)
    → returns "HEWLETT-PACKARD,34401A,..."
```

**Key point**: Match the line terminator at the end of the command (`\r\n` or `\r`) to the device specification.

---

#### `serial_write`

Tool that **only sends** data to the serial port.

| Item | Details |
|------|---------|
| **Parameters** | `content` (required): string to send |
| **Return** | Number of bytes sent |
| **Use when** | One-way send with no response needed (program transfer, configuration commands, etc.) |

---

#### `serial_read`

Tool that **only reads** data accumulated in the serial port receive buffer.

| Item | Details |
|------|---------|
| **Parameters** | None |
| **Return** | Receive buffer contents (empty string if no data) |
| **Use when** | Receiving data spontaneously sent by the device |

**Note**: Using `serial_query` is simpler in most cases.

---

#### `serial_config` (SerialBridge Pro edition only)

Changes serial communication baud rate and parity.
**Not available in LUNA Free edition (returns error).**

| Item | Details |
|------|---------|
| **Parameters** | `baud` (required): baud rate |
| | `parity` (optional): `N`=none (default), `E`=even, `O`=odd |
| **Supported baud rates** | 300 / 600 / 1200 / 2400 / 4800 / 9600 / 19200 / 38400 / 115200 |

---

### 🤖 Lua Autonomous Control Tools (v3.9.1 LUNA New Features)

#### `lua_exec`

Executes Lua code immediately on the spot. Use for testing and one-off processing.

| Item | Details |
|------|---------|
| **Parameters** | `code` (required): Lua 5.1 source code |
| **Return** | Execution start confirmation (get result with lua_status) |
| **Use when** | Verification, testing, simple one-time execution |

```lua
-- Usage example
lua_exec("serial.query('*IDN?\\r\\n', 1000)")
```

---

#### `lua_load`

Saves a Lua script to SPIFFS. Persists after power-off.

| Item | Details |
|------|---------|
| **Parameters** | `name` (required): script name (alphanumeric, _, - only, max 22 chars) |
| | `code` (required): Lua 5.1 source code |
| **Save path** | `/lua_{name}.lua` |

---

#### `lua_run`

Executes a saved script.

| Item | Details |
|------|---------|
| **Parameters** | `name` (required): script name saved with lua_load |
| **Use when** | Regular execution, production operation |

---

#### `lua_status`

Checks the Lua VM execution state.

| Item | Details |
|------|---------|
| **Return** | state (running/idle), last_result, free_heap, uptime |
| | chain_index, chain_script (luna.run() chain progress) |

---

#### `lua_stop`

Force stops the running Lua task.

---

#### `lua_log`

Retrieves the log written by `log.write()` in scripts (ring buffer of latest 20 lines).

---

#### `lua_list`

Displays a list of Lua scripts saved in SPIFFS.

---

### 📊 System Information Tools

| Tool | Function | Parameters |
|---|---|---|
| `get_system_info` | Get ESP32 system information | None |
| `get_network_status` | Get WiFi / network status | None |
| `get_license_info` | Get license, edition, author info | None |
| `hard_reset` | Restart ESP32 (ESP.restart() after 1 second) | None |

---

### 📁 File System Tools (SPIFFS)

| Tool | Function | Parameters |
|---|---|---|
| `fs_list` | List SPIFFS files | None |
| `fs_read` | Read file (max 4KB) | `path` (required) |
| `fs_write` | Write file (max 1KB, overwrites) | `path` (required), `content` (required) |
| `fs_append` | Append to file (max 1KB) | `path` (required), `content` (required) |
| `fs_delete` | Delete file | `path` (required) |

> **Note**: fs_* tools are called from Claude (via MCP). They cannot be called from within Lua scripts.
> To write to SPIFFS from Lua, use `log.save()`.

---

### 🔔 Notification Tools

| Tool | Function | Parameters |
|---|---|---|
| `check_notify` | Check notification flag set by `notify.set()` in Lua scripts | None |

> **Usage**: When a Lua script calls `notify.set("message")`, Claude can detect it with `check_notify`.

---

> **Note on Board A/B**
> `board_read` / `board_write` / `board_list` are implemented in the firmware,
> but since LUNA does not have a legacy PC chat feature, **they are not normally used**.
> These are inherited from v2.0.1 Nano Edition (for MSX/CP-M).

---

### Tool Selection Cheat Sheet

```
[Send a command to a device and receive a response]
  → serial_query (adjust timeout_ms to device)

[Push a program or settings one-way to the device]
  → serial_write (repeat line by line)

[Receive data spontaneously sent by the device]
  → serial_read

[Check if LUNA is operating normally]
  → get_system_info or get_network_status

[Test a Lua script]
  → lua_exec

[Save and run a Lua script for production]
  → lua_load → lua_run

[Check Lua script execution status]
  → lua_status

[Check Lua script log]
  → lua_log

[Notify Claude of completion or anomaly from Lua script]
  → call notify.set() in script → Claude detects with check_notify
```

---

## Appendix B: Lua API Reference (LUNA v3.9.1)

API for using ESP32 features from Lua scripts.

### serial module

| API | Description |
|-----|-------------|
| `serial.query(cmd, timeout_ms)` | Send command → wait → receive. Primary API |
| `serial.write(data)` | Send only |
| `serial.read()` | Read receive buffer |

### log module

| API | Description |
|-----|-------------|
| `log.write(msg)` | Write to log buffer (Claude reads with lua_log) |
| `log.read([n])` | Read log buffer (inter-script data passing) |
| `log.save()` | Save log to `/luna_log.txt` in SPIFFS (do not call in loop) |
| `log.load()` | Load from SPIFFS into buffer (clears buffer on call) |

### hw module

| API | Description |
|-----|-------------|
| `hw.delay(ms)` | Wait specified milliseconds |
| `hw.millis()` | Elapsed time since boot (ms) |
| `hw.reset()` | Restart ESP32 |

### notify module

| API | Description |
|-----|-------------|
| `notify.set(msg)` | Set notification message to Claude. Detected by check_notify |

### luna module

| API | Description |
|-----|-------------|
| `luna.run(name)` | Queue next script and exit current script (max 10 in chain) |

---

## Appendix C: Example Prompts for AI

### Recommended Format for Handing Over a Device Manual

```
I will control the following device via RS-232C through LUNA.

[Device Information]
  Device name: ○○ Model by ○○ Company
  Communication settings: 9600bps, 8N1
  Command system: SCPI-compliant (or proprietary commands)

[Manual (attached or as text)]
  [paste manual content here or attach PDF]

Please understand the above manual and control the device according to my instructions.
When sending commands, tell me which command you are sending before executing.
```

### Example Lua Script Creation Request

```
Please create a Lua script with the following specification,
save it to LUNA, and execute it:

Script name: voltage_monitor
Operation: Measure voltage every 30 seconds,
           call notify.set() if below 1.0V or above 1.5V
Measurement command: MEAS:VOLT:DC?\r\n (response timeout: 1000ms)
Repeat count: 120 times (approx. 1 hour)
```

### Example of Coordinated Multi-Device Control

```
Please coordinate the devices connected to these 2 LUNA units:
· LUNA-Power: programmable power supply
· LUNA-Meter: digital multimeter

Vary the power supply from 0V to 10V in 1V steps,
compare with multimeter readings at each voltage,
and record the error.
```

---

## Appendix D: Supported Serial Settings

### LUNA (Free Edition — Fixed)

```
Baud rate:   9600 bps
Data bits:   8
Parity:      None (N)
Stop bits:   1
Flow control: None
```

### SerialBridge Pro Edition (v4.2.0 — Configurable)

```
Baud rate:   300 / 600 / 1200 / 2400 / 4800 / 9600 / 19200 /
             38400 / 115200 bps
Data bits:   8
Parity:      None / Odd / Even
Stop bits:   1
Flow control: None
```

---

## Appendix E: RS-232C Level Conversion Circuit

ESP32 GPIOs operate at 3.3V; RS-232C uses ±12V. A level converter is always required.

```
[Recommended ICs]
  MAX3232 (3.3V operation, +/-15kV ESD protection)
  MAX232  (5V operation, requires 4 external capacitors)

[Basic Connection]
  ESP32 TX2(GPIO17) → MAX3232 T1IN → T1OUT → Device RXD
  ESP32 RX2(GPIO16) → MAX3232 R1OUT ← R1IN ← Device TXD
  ESP32 GND         → MAX3232 GND   ────────── Device GND
```

---

## Closing

LUNA may look like a simple piece of hardware —
an "ESP32 serial bridge."

But its true nature is:

> **"A universal platform that turns any RS-232C device into an AI-controlled system"**
> **"And an autonomous gateway where Lua scripts keep running through the night on their own"**

With just a manual, any device instantly becomes an AI system.
Devices that have been sleeping in storage for 40 years return to active service.
Even when "the person who knew how" is gone, the device can still be used.
And while Claude is away, LUNA quietly keeps working.

**This is a new paradigm that spans industry, research, and cultural preservation.**

---

*LUNA (AiBridgeMCP v3.9.1) / MIT License*
*(C) 2026 Nishioka Sadahiko*
*Blog   : https://nskikaku.blog.fc2.com/*
*GitHub : https://github.com/OnStepNinja/LUNA*

---

**Disclaimer**
This system (LUNA / AiBridgeMCP) is provided "as is" under the MIT License.

**Use at your own risk**: Use of this product and the provided software is entirely at the user's own risk.

**No warranty for damages**: The author accepts no responsibility whatsoever for any direct or indirect damages arising from the use or inability to use this product (including device failure, circuit damage, data loss, or any other disadvantage), regardless of the cause.

**Understanding and consent**: By connecting this product to RS-232C equipment, users are deemed to have fully understood the electrical risks involved and to have agreed to these terms.

---

**Legal Disclaimer:**
This product is an independently developed hardware by OnStepNinja and is not affiliated with, endorsed by, or sponsored by Anthropic, Takahashi Seisakusho, or any other company.
"Temma", "MCP" and other product names are trademarks of their respective owners.
Use of these names is solely for the purpose of describing compatibility (Nominative Fair Use).

CP/M is a registered trademark of Digital Research (or its successors).
MSX is a registered trademark of MSX Licensing Corporation.
PC-8001/PC-88/PC-98 are trademarks of NEC Corporation.

This project is an independent project by an individual unaffiliated with any of the above companies. It is free software provided in binary form under the MIT License (free of charge, commercial use permitted); source code is not publicly available.
