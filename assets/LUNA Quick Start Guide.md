# LUNA Quick Start Guide

**Target Version**: v5.9.6 (ESP32) / v6.9.3 (ESP32-S3)
**Updated**: 2026-04-21

---

> 💡 **No need to read the manual.**
> Just connect LUNA and ask Claude — it will guide you through everything.

---

## What You Need

| Item | Where to Get It | Cost |
|------|----------------|------|
| **Claude Desktop** | [https://claude.ai/download](https://claude.ai/download) | Free |
| **LUNA Firmware** | [GitHub Releases](https://github.com/OnStepNinja/LUNA/releases) | Free |
| **Windows PC** | Connected to the same Wi-Fi as LUNA | — |

> 💡 **Claude Code is NOT required.** Claude Desktop (free) is all you need to use every feature of LUNA.
> If you get stuck at any point, just ask Claude — it can walk you through the setup.

---

## Devices You Can Control with LUNA

Any device with an RS-232C port can be controlled. LUNA acts as an AI-powered bridge.

| Category | Examples |
|----------|---------|
| **Industrial / Manufacturing** | CNC machines, PLCs, inverters, robot controllers |
| **Measurement / Research** | Multimeters, oscilloscopes, power supplies, spectrum analyzers |
| **Astronomy (hardware)** | Takahashi Temma2, Celestron mounts, and more |
| **Astronomy (software)** | NINA, Stellarium, ASCOM/Alpaca compatible devices |
| **Astronomy (INDI)** | All INDI-compatible devices (telescope, camera, focuser) — v6.9.3 only |
| **Broadcast / AV** | Video switchers, VTR controllers |

> Just hand Claude the device manual (PDF, etc.) and you can start controlling it immediately.

---

## Need Help?

> 🔗 **[Open LUNA Docs (NotebookLM)](https://notebooklm.google.com/notebook/f2ce997f-18c2-44c4-8738-973b204c190c)**
>
> Ask any question about LUNA — setup, settings, Lua scripts, and more.
> A Google account is all you need.

---

## Setup (3 Steps)

### 1. Place the Connector Folder

Download the `connector` folder for free:

> 📦 **BOOTH**: [https://onstepninja.booth.pm/items/8237213](https://onstepninja.booth.pm/items/8237213)

Extract the ZIP and place the `connector` folder somewhere on your PC (e.g., `C:\LUNA\`).
**Do not move it after placing** — the connection will break if the path changes.

---

### 2. Find LUNA's IP Address

With LUNA connected to your home router (STA mode), find its assigned IP address.

How to check:
- Look for **LUNA** in your router's DHCP client list
- Or open the LUNA Web Console at `http://192.168.4.1` (AP mode) → **Network Information** → **STA IP**

You will use this **STA IP address** (e.g., `192.168.3.50`) in the next step.

---

### 3. Edit the Claude Desktop Configuration

#### ① Open the Config File

Press **Windows key + R** on your keyboard.
When the "Run" dialog opens, copy and paste the following, then press **Enter**:

```
%APPDATA%\Claude
```

A folder will open. Inside, find **`claude_desktop_config.json`**.
(If it does not exist, create a new text file with that name.)

Open the file with **Notepad** or any text editor.

> 💡 **If you're unsure what to do, just ask Claude — it's the most reliable way to get help.**

---

#### ② Edit and Save

Copy the following and **replace the entire contents** of the file, then save:

```json
{
  "mcpServers": {
    "LUNA": {
      "command": "C:\\LUNA\\connector\\aibridge-connector-v2.exe",
      "args": ["192.168.3.xxx"]
    }
  }
}
```

**There is only one thing to change:**

`"192.168.3.xxx"` → Replace with the STA IP address you found in Step 2
(e.g., `"192.168.3.50"`)

> ⚠️ **Note**: The `\\` in the `command` path is a double backslash — this is required.
> If you placed the `connector` folder in a different location, update the path accordingly.
> (e.g., on the Desktop → `"C:\\Users\\YourName\\Desktop\\connector\\aibridge-connector-v2.exe"`)

---

#### ③ Fully Quit Claude Desktop

After saving, completely quit Claude Desktop.
The most reliable method is to use **Task Manager**:

1. Press **Ctrl + Shift + Esc** to open Task Manager
2. Find **Claude** in the list of processes
3. Right-click → **"End Task"**

> 💡 **Not sure how to do this? Ask Claude before you close it — it's the most reliable way.**
> Try: *"How do I end a task in Task Manager?"*

---

#### ④ Verify the Connection

Launch Claude Desktop and click the **"+"** icon at the bottom left of the chat input.
Select **"Connectors"** — if **LUNA** shows a 🔵 blue toggle (ON), you're connected!

> 💡 **If anything is unclear, ask Claude AI — it can help you solve any issue.**

---

### Connection Test

Say this to Claude:

> *"Tell me the status of the LUNA device."*

If Claude responds with the IP address, firmware version, and free memory — everything is working correctly.

---

## What LUNA Can Do — The Serial Bridge Revolution

LUNA lets Claude AI directly control **any device** with an RS-232C port.

```
Traditional approach:
  Buy device → Expert reads manual → Write control software → Test
  Total: months to years, potentially huge development cost
  Problem: when the expert leaves, nobody can use it anymore

LUNA + AI approach:
  Buy device → Connect via RS-232C cable to LUNA (minutes)
             → Hand the manual to Claude (minutes)
             → Just say "Do X with this device"
  Total: tens of minutes, near-zero development cost
  Benefit: anyone can use it, anytime, as long as the manual exists
```

Factory equipment, lab instruments, astronomy gear — **just hand Claude the manual and it becomes an AI system.**

With Lua scripts, the ESP32 can also run autonomously — even when Claude is not connected.

---

## Talking to Claude

Once connected, just speak naturally:

```
"What can you do?"

"Tell me the current RA/Dec of the telescope (NS-5000)"

"Set up a connection for SkySafari"

"Take a 30-second exposure with NINA"

"Show the current telescope position in Stellarium"

"Write a Lua script that logs RA every 10 seconds"
```

---

## Key Things to Tell Claude First

### 📖 get_lua_guide — Always Start Here

> *"Please fetch get_lua_guide."*

LUNA has a **Lua scripting** system that lets the ESP32 run autonomously — controlling telescopes, running scheduled tasks, and more — even without Claude connected. For Claude to use this effectively, it needs to read LUNA's design reference first.

`get_lua_guide` delivers that reference to Claude. **Before asking for complex scripts, always fetch this first — it leads to more accurate results.**

---

### 🔭 Vixen Support — Connect Vixen Mounts (v5.9.6 / v6.9.3)

> *"Create a Lua script to connect my Vixen mount via WiFi."*

LUNA supports the **Vixen Wireless Unit** and **STAR BOOK TEN** protocols. If you have a Vixen mount, you can control it directly from Claude — no dedicated controller needed.

Supported models: AXD / SXP / SXD2 / AP / SX2 / AXJ, and more.

**`get_vixen_guide` contains all Vixen-specific usage instructions.** Tell Claude:

> *"Please fetch get_vixen_guide."*

> ⚠️ **The Vixen protocol information is confidential.**
> The protocol specification used in this implementation is proprietary. Please do not share or republish it.

---

### 📷 NINA Integration — Automate Astrophotography (v5.9.6 / v6.9.3)

> *"Take a 30-second exposure using NINA."*

**NINA** (Nighttime Imaging 'N' Astronomy) is a Windows-based astrophotography application. Through NINA's **Advanced API plugin**, LUNA lets you control the camera, start exposures, and confirm completion — all from a Claude conversation.

What you can do:
- Check camera connection status, temperature, and gain
- Start an exposure with specified duration and gain
- Automatically wait for exposure completion and receive a notification

**get_lua_guide includes instructions for the NINA Advanced API.** Fetch `get_lua_guide` before making requests.

---

### 🔌 ASCOM / Alpaca — Windows Standard Device Control (v5.9.6 / v6.9.3)

> *"Get telescope information via Alpaca."*

**ASCOM** is the standard protocol for astronomy device control on Windows. **Alpaca** is its WiFi variant, allowing HTTP-based control of telescopes, cameras, focusers, and more. LUNA uses the Lua `http` module to access Alpaca devices directly.

What you can do:
- Get current telescope position (RA/Dec)
- Send GoTo commands
- Control any Alpaca-compatible device

**get_lua_guide includes Alpaca integration instructions.** Fetch `get_lua_guide` before making requests.

---

### 📡 Resources — Claude Reads Live Status Automatically (v5.9.6 / v6.9.3)

LUNA has a **Resources** system. By registering real-time data sources — telescope position, weather data, camera status — as "instruments," Claude can reference them automatically during conversation.

```
Without Resources: "Where is the telescope pointing?"
  → Claude sends a command and waits for a response each time

With Resources: "Where is the telescope pointing?"
  → Claude answers instantly
```

Simply save a script named `lua_resource_mount.lua` to SPIFFS and it becomes active. Ask Claude to create the script for you:

> *"Create a Resource script that automatically reads telescope RA/Dec."*

---

### 🌐 INDI Server Integration — Full Observatory AI Control (v6.9.3 only)

> *"Connect to the INDI server and point the telescope to RA=18.6, DEC=38.8."*

**INDI** (Instrument-Neutral Distributed Interface) is the standard astronomy control protocol for Linux (Raspberry Pi, etc.). It integrates with KStars / Ekos / PHD2 to manage telescopes, cameras, focusers, and more.

With v6.9.3 (ESP32-S3), LUNA connects directly to an INDI server. Combined with a Raspberry Pi, this enables a low-cost DIY observatory automation system — controlled entirely through Claude.

INDI has dedicated MCP tools that Claude uses directly:

| Tool | Function |
|------|----------|
| `indi_connect` | Connect to INDI server |
| `indi_devices` | List connected devices |
| `indi_get` | Get device value (RA/Dec, etc.) |
| `indi_set_number` | Set numeric value (GoTo coordinates, etc.) |
| `indi_set_switch` | Toggle switch (tracking ON/OFF, etc.) |

Just tell Claude:

> *"Connect to the INDI server at 192.168.x.x using indi_connect."*

---

## Using Multiple LUNA Devices

The names `"LUNA-mount"` and `"LUNA-aux"` are the **MCP server names** — you can choose any names you like.
Claude will use these names to distinguish between devices (e.g., *"Get RA from the mount"*).

```json
{
  "mcpServers": {
    "LUNA-mount": {
      "command": "C:\\LUNA\\connector\\aibridge-connector-v2.exe",
      "args": ["192.168.3.50"]
    },
    "LUNA-aux": {
      "command": "C:\\LUNA\\connector\\aibridge-connector-v2.exe",
      "args": ["192.168.3.154"]
    }
  }
}
```

---

## Troubleshooting

| Symptom | Solution |
|---------|---------|
| LUNA not showing in Connectors | Check JSON for syntax errors (commas, brackets). Restart Claude Desktop. |
| File error | Confirm you are using `aibridge-connector-v2.exe` (with `-v2`). |
| Communication error | Ensure the ESP32 and PC are on the **same Wi-Fi network**. Check the IP address. |
| IP address changes after restart | Set a static IP for LUNA in your router (DHCP reservation). |

---

## Differences Between v6.9.3 (ESP32-S3) and v5.9.6 (ESP32)

| Feature | v5.9.6 | v6.9.3 |
|---------|--------|--------|
| Serial, Lua, UI, TCP, HTTP | ✅ | ✅ |
| INDI server integration | ❌ | ✅ |
| RGB LED status (NeoPixel) | ❌ | ✅ |

All other features are identical between versions.

---

*LUNA — Bridge Across Decades © 2026 Nishioka Sadahiko*
*Support: PayPal → nishioka.sst@gmail.com / Buy a board → GitHub Issues*
