# AiBridge USER MANUAL v7.15

**Version**: v7.15
**Author**: Nishioka Sadahiko
**License**: MIT License (free to use, commercial use permitted, modification permitted, redistribution permitted)
**Last Updated**: 2026-04-05

---

> **Important Notice**
>
> AiBridge is distributed as a compiled binary (`.bin` file) only.
> **Source code is not publicly available.**
> Arduino IDE and library installation are not required.
> Use `esptool.py` or a GUI tool (ESP32 Flash Download Tool) to flash the firmware.
> This software is free to use and redistribute (binary form, MIT License).

---

## Table of Contents

**Part 1 — Overview & Specifications**
1. [What is AiBridge?](#1-what-is-aibridge)
2. [Product Information](#2-product-information)
3. [System Architecture](#3-system-architecture)

**Part 2 — Hardware**
4. [Requirements](#4-requirements)
5. [GPIO Pin Assignment](#5-gpio-pin-assignment)
6. [Serial Port (UART) Configuration](#6-serial-port-uart-configuration)
7. [Hardware Wiring](#7-hardware-wiring)

**Part 3 — Installation & Setup**
8. [Flashing the Firmware](#8-flashing-the-firmware)
9. [Initial Setup](#9-initial-setup)
10. [Using the File Manager](#10-using-the-file-manager)

**Part 4 — Operation & Connections**
11. [Using the AiBridge Console](#11-using-the-aibridge-console)
12. [Connecting Astronomy Apps via TCP (SkySafari, etc.)](#12-connecting-astronomy-apps-via-tcp-skysafari-etc)
13. [ASCOM Alpaca Support](#13-ascom-alpaca-support)

**Part 5 — API Reference**
14. [Network Configuration & Protocols](#14-network-configuration--protocols)
15. [API Reference](#15-api-reference)
16. [Web UI Pages](#16-web-ui-pages)

**Part 6 — LUNA Integration & Observatory Control**
17. [Dynamic Dashboard (LUNA Integration)](#17-dynamic-dashboard-luna-integration)
18. [Integration with LUNA AI Gateway](#18-integration-with-luna-ai-gateway)
19. [Observatory Integration Vision (Sorami System)](#19-observatory-integration-vision-sorami-system)

**Part 7 — Appendix**
20. [Troubleshooting](#20-troubleshooting)
21. [Technical Specifications](#21-technical-specifications)
22. [Version History](#22-version-history)
23. [Boot Log Example](#23-boot-log-example)

---

# Part 1 — Overview & Specifications

## 1. What is AiBridge?

AiBridge is a compact ESP32-based telescope control interface designed to bridge the gap between various control protocols and hardware in astronomical observation. It integrates both the traditional LX200 protocol and the modern ASCOM Alpaca standard, acting as a strategically important bridge that allows modern control applications such as SkySafari, NINA, and Stellarium to seamlessly operate legacy telescope mounts.

```
Smartphone / PC / AI (Claude)
        │
        │ Wi-Fi (TCP / HTTP / Alpaca)
        │
   AiBridge (ESP32)
        │
        │ Serial (LX200 Protocol / 9600bps)
        │
   Telescope Mount (OnStep / NS-5000, etc.)
```

### Features

| Feature | Description |
|---------|-------------|
| **Web Console** | Control telescope from a browser — GoTo, status monitoring |
| **SkySafari Connection** | Direct TCP control from iOS / Android astronomy apps |
| **ASCOM Alpaca** | Integration with Windows astronomy software (NINA, Sequence Generator, etc.) |
| **PHD2 Autoguiding** | PulseGuide support |
| **Weather Sensor** | Provides temperature and humidity via Alpaca using DHT22 |
| **Switch Control** | 4-channel GPIO for ON/OFF control of external devices |
| **Dynamic Dashboard** | Custom display screens in collaboration with LUNA AI Gateway *(v7.15)* |
| **AP + STA Simultaneous** | Acts as an access point simultaneously — ready to use without infrastructure changes |
| **MAC Auto SSID** | Automatically generates a unique AP SSID per unit *(v7.15)* |

---

## 2. Product Information

| Item | Details |
|------|---------|
| Product Name | AiBridge - AI-Enabled Telescope Interface |
| Version | 7.15 |
| Copyright | Copyright (c) 2026 Nishioka Sadahiko / 西岡定彦 |
| License | MIT License |
| Distribution | Binary only (source code not publicly available) |
| Contact | nishioka.sst@gmail.com |
| Official Site | https://nskikaku.blog.fc2.com/ |
| GitHub | https://github.com/OnStepNinja |

---

## 3. System Architecture

AiBridge acts as a central control hub, accepting connections from various clients over Wi-Fi and converting them into serial commands that the telescope mount can interpret.

### Key Components

1. **Client Applications**: SkySafari, NINA, Stellarium, Claude AI (via LUNA Observatory), etc. connect over Wi-Fi using LX200 (TCP) or ASCOM Alpaca (HTTP).

2. **AiBridge Unit**: ESP32 module. A Web server (HTTP), TCP server (LX200), and UDP listener (Alpaca Discovery) run continuously, interpreting and converting commands.

3. **Telescope Mount**: An equatorial or alt-azimuth mount with serial communication support (OnStep, etc.). Serial commands are sent via UART.

4. **External Devices**: Proxy control of other Alpaca-compatible devices on the same network (focuser, camera, dome, etc.) is also possible.

5. **LUNA Observatory (v7.15+)**: LUNA AI Gateway connects via HTTP to execute dynamic dashboards and AI-driven observation workflows.

```
 Client Apps                      LUNA Observatory (AI)
(SkySafari / NINA / ─────────────── ↕ ─────────────────
 Stellarium / Browser)                │
         │                            │
         │   Wi-Fi (TCP / HTTP / Alpaca)
         │                            │
         └──────── AiBridge (ESP32) ──┘
                          │
                   UART (LX200 Serial)
                          │
                  Telescope Mount (OnStep)
```

---

# Part 2 — Hardware

## 4. Requirements

### Hardware

| Item | Specification | Notes |
|------|---------------|-------|
| ESP32 Development Board | WROOM-32 or compatible | 4MB flash or more |
| LX200-compatible Telescope Mount | OnStep / NS-5000, etc. | RS-232C or TTL serial |
| USB-RS232C Adapter (if needed) | 3.3V TTL compatible | OnStep supports direct TTL connection |
| DHT22 Temperature/Humidity Sensor (optional) | Connect to GPIO5 | DHT11 also compatible |
| Small Tactile Switch (optional) | Connect to GPIO39 | For enabling File Manager |
| LED (optional) | Connect to GPIO25 | File Manager status indicator |
| Relay Module (optional) | 4-channel | Controlled via GPIO27/14/12/13 |

### Software (Required for flashing only)

| Software | Purpose |
|----------|---------|
| esptool.py or ESP32 Flash Download Tool | Firmware flashing (first time only) |
| SkySafari / NINA, etc. (optional) | Astronomy control apps |

---

## 5. GPIO Pin Assignment

Pins with `INPUT_PULLUP` (GPIO39, 33, 32) have internal pull-up resistors enabled at firmware level — no external pull-up resistors are needed.

| Constant Name | GPIO | Function | Direction |
|---------------|------|----------|-----------|
| `FILE_MANAGER_LED_PIN` | 25 | File Manager active LED | OUTPUT |
| `FILE_MANAGER_ENABLE_PIN` | 39 | File Manager enable button | INPUT_PULLUP |
| `CUSTOM_UI_SELECT_PIN` | 33 | Custom UI selection | INPUT_PULLUP |
| `SERIALONSTEP_SELECT_PIN` | 32 | OnStep UART selection | INPUT_PULLUP |
| `DHT_PIN` | 5 | DHT22/11 temperature/humidity sensor | INPUT |
| `SWITCH_0_PIN` | 27 | Alpaca switch output 1 | OUTPUT |
| `SWITCH_1_PIN` | 14 | Alpaca switch output 2 | OUTPUT |
| `SWITCH_2_PIN` | 12 | Alpaca switch output 3 | OUTPUT |
| `SWITCH_3_PIN` | 13 | Alpaca switch output 4 | OUTPUT |
| `DEBUG_TX` | 19 | Debug serial TX | OUTPUT |
| `DEBUG_RX` | 18 | Debug serial RX | INPUT |

---

## 6. Serial Port (UART) Configuration

### OnStep Connection Port (SerialOnStep)

The UART used is automatically selected based on the input level of GPIO32 at boot.

| GPIO32 State | UART Used | TX Pin | RX Pin |
|-------------|-----------|--------|--------|
| HIGH (open or 3.3V) | UART0 | GPIO1 | GPIO3 |
| LOW (connected to GND) | UART2 | GPIO17 | GPIO16 |

- **Baud Rate**: 9600 baud (fixed)
- **Format**: 8N1

### Debug Port (SerialDebug)

- **UART**: UART1 (TX: GPIO19, RX: GPIO18)
- **Baud Rate**: 115200 baud

---

## 7. Hardware Wiring

### 7-1. Telescope Connection (Serial)

```
[GPIO32 = HIGH (open or connected to 3.3V) → Use UART0]
  GPIO1 (TX) ─── Telescope RX
  GPIO3 (RX) ─── Telescope TX

[GPIO32 = LOW (connected to GND) → Use UART2]
  GPIO17 (TX) ─── Telescope RX
  GPIO16 (RX) ─── Telescope TX
```

> **For OnStep users**: OnStep supports direct TTL-level serial connection.
> **For RS-232C devices (NS-5000, etc.)**: An RS-232C ↔ TTL level converter adapter is required.

### 7-2. DHT22 Sensor Wiring

```
DHT22                    ESP32
VCC  ─────────────────── 3.3V
GND  ─────────────────── GND
DATA ─── 10kΩ → 3.3V
     └───────────────── GPIO5
```

### 7-3. File Manager Switch & LED (Optional)

```
[Tactile Switch (GPIO39)]
  One end ─── GPIO39
  Other end ─── GND
  (Uses internal pull-up. Pressing pulls LOW, enabling File Manager)

[LED (Status Indicator GPIO25)]
  Anode  ─── Resistor (220Ω) ─── GPIO25
  Cathode ─── GND
  (Lit while File Manager is active)
```

### 7-4. Custom UI Selection (GPIO33)

```
GPIO33 connected to GND: Shows index2.html (custom UI) at boot
GPIO33 open (HIGH): Shows standard AiBridge Console
```

---

# Part 3 — Installation & Setup

## 8. Flashing the Firmware

AiBridge distributes a compiled binary (`.bin` file). **Arduino IDE and source code are not required.**

### 8-1. What You Need

| Item | Source |
|------|--------|
| `AiBridge_v7.15_Standard.bin` | Download from distribution page |
| USB cable (data transfer capable) | — |
| esptool.py | `pip install esptool` |

### 8-2. Flashing with esptool.py

```bash
pip install esptool

esptool.py --chip esp32 --port COMx --baud 921600 write_flash -z 0x0 AiBridge_v7.15_Standard.bin
```

| OS | Port Example |
|----|-------------|
| Windows | `COM3` (check in Device Manager) |
| macOS | `/dev/cu.SLAB_USBtoUART` |
| Linux | `/dev/ttyUSB0` |

### 8-3. GUI Tool (Windows)

[ESP32 Flash Download Tool](https://www.espressif.com/en/support/download/other-tools) (official Espressif tool) is also available.

| Setting | Value |
|---------|-------|
| Address | `0x0` |
| File | `AiBridge_v7.15_Standard.bin` |
| Mode | DIO |
| Baud Rate | 921600 |

Click the START button to begin flashing.

> ⚠️ **After flashing, be sure to format SPIFFS before configuring WiFi.**
> See [Section 10-3](#10-3-spiffs-format) for instructions.

---

## 9. Initial Setup

### Why AP Mode Exists

After powering on, AiBridge **always starts in AP mode**. AP mode means the device itself becomes a WiFi access point.

1. **To perform initial WiFi configuration**
2. **To check the DHCP-assigned IP address**
3. **To allow reconfiguration when the WiFi router changes**

AP is always active, so you can always connect to the device regardless of its state.

---

> ### ⚠️ Important: Format SPIFFS Before WiFi Configuration
>
> After flashing new firmware, always format SPIFFS before performing WiFi configuration for the first time.
> Without formatting, old data may remain, causing WiFi settings to not be saved correctly.
>
> **Steps**: Connect to AP (`AiBridgeXXXX`) → `http://192.168.4.1` → File Manager → **Format SPIFFS** → Confirm
>
> The device will restart after formatting. Then proceed from step ① below.

---

### ① Connect to the AP

Select the following SSID in your WiFi settings:

```
SSID    : AiBridgeXXXX  (XXXX = last 4 digits of MAC address — e.g., AiBridgeA1B2)
Password: None (open network)
```

> **The AP SSID is unique per device.** Automatically generated from the MAC address, so multiple units will never have conflicting SSIDs.

Open the following in your browser:

```
http://192.168.4.1
```

### ② Configure Home WiFi (STA)

1. Click the **"File Manager"** button in the AiBridge Console
2. Press the GPIO39 button to enable File Manager (confirm LED turns on)
3. Enter in the **WiFi Configuration** section:
   - **SSID**: Your home WiFi network name
   - **Password**: Your WiFi password
   - **Location Name**: Identifier for this device (e.g., `Observatory`)
4. Click **"Save WiFi Config"** → Device will restart

### ③ Check the STA IP Address

After restarting, reconnect to the AP (`AiBridgeXXXX`) and open `http://192.168.4.1`:

```
STA Address: 192.168.x.xxx (DHCP assigned)
```

This is the IP address you will use during normal operation.

> **💡 Tip**: After setup is complete, switch your phone/PC back to your home WiFi.
> Normal operation is done via home WiFi. The AP remains always active as an emergency access method.

### About Location Name

Location Name is the name used to identify the device during Alpaca Discovery.

| Input Value | Display in Alpaca Discovery |
|-------------|----------------------------|
| Bridge (default) | Ai-Bridge |
| Observatory | Ai-Observatory |
| MainScope | Ai-MainScope |
| GuideScope | Ai-GuideScope |

> **For multiple units**: Always set different Location Names (e.g., `Mount1`, `Mount2`).

---

## 10. Using the File Manager

File Manager is a screen for managing SPIFFS (internal file system). For security, activation via the GPIO39 button is required.

### 10-1. Enabling File Manager

| Method | Operation | Active Duration |
|--------|-----------|-----------------|
| **At boot** | Set GPIO39 LOW and power on | 30 minutes |
| **During operation** | Click "File Manager" button in Console, then press GPIO39 | 10 minutes |

The GPIO25 LED lights up while File Manager is active.

### 10-2. WiFi Configuration

The **WiFi Configuration** section in File Manager:

| Field | Description |
|-------|-------------|
| **SSID** | Home WiFi network name |
| **Password** | WiFi password |
| **Location Name** | Device identifier (used for Alpaca Discovery) |

After entering the values, click **"Save WiFi Config"** to save to `/aibridge_net_cfg.dat` and restart.

### 10-3. SPIFFS Format

The **"Format SPIFFS"** button initializes the internal file system.

> ⚠️ Formatting deletes **all saved files** (WiFi settings, custom HTML, etc.).
> Run this only once after the initial firmware flash — not during normal use.

### 10-4. File Management

The File Manager screen allows you to view and manage files in SPIFFS:

- View file list and check storage usage
- Upload files from PC (custom HTML, etc.)
- Delete files

### 10-5. Custom UI (index2.html)

Booting with GPIO33 LOW displays `/index2.html` instead of the standard Console.
You can save a custom dashboard HTML delivered from LUNA here for use.

---

# Part 4 — Operation & Connections

## 11. Using the AiBridge Console

Access `http://<IP>/` in a browser to open the AiBridge Console.

### 11-1. Network Information

An information panel visible immediately after boot.

| Display Item | Content |
|-------------|---------|
| **Location Name** | Alpaca Discovery identifier (`Ai-` + location name) |
| **AP SSID** | This device's AP name (auto-generated from MAC) |
| **AP Address** | IP address in AP mode (192.168.4.1) |
| **STA Address** | IP when connected to home WiFi + connection status badge |
| **Command Ports** | TCP command ports (9999 / 9998) |

Connection status badges:
- 🟢 **Connected** — Connected to home WiFi
- 🔴 **Not Connected** — Not connected (AP mode only)

### 11-2. ASCOM / Alpaca Device Discovery

Automatically discovers Alpaca-compatible devices on the network.

1. Click the **"Discover ASCOM / Alpaca Telescopes"** button
2. Wait a few seconds for detected devices to appear in the list
   - Example: `📡 Simulator Telescope #0 (192.168.3.100:80)`

> AiBridge itself is not shown in the list (self-discovery excluded).

### 11-3. Telescope Status

| Display Item | Description |
|-------------|-------------|
| **RA** | Current Right Ascension (HH:MM:SS) |
| **DEC** | Current Declination (±DD:MM:SS) |
| **Tracking** | Tracking state (On / Off) |
| **Slewing** | Whether currently slewing |
| **Pier Side** | Pier side (East / West / Unknown) |

### 11-4. GoTo Command

Directly enter RA and Dec to slew the telescope to the specified coordinates.

1. Enter Right Ascension in the **RA** field (e.g., `05:34:32`)
2. Enter Declination in the **Dec** field (e.g., `+22:00:52`)
3. Click the **"GoTo"** button

> Uses the LX200 `:MS#` command.

### 11-5. LX200 Command Transmission

Send arbitrary LX200 commands directly.

1. Enter the command in the **Command** field (e.g., `GR`, `GD`, `Q`)
2. Click the **"Send"** button
3. The telescope's response is displayed

> The `:` prefix and `#` suffix are added automatically. Entering `GR` will send `:GR#`.

### 11-6. Tracking Control

| Button | Function |
|--------|---------|
| **Tracking On** | Start sidereal tracking |
| **Tracking Off** | Stop tracking |
| **Stop** | Immediate slew stop (`:Q#`) |

### 11-7. Switch Control

Turns ON/OFF relays and other devices connected to GPIOs.

| Switch | GPIO |
|--------|------|
| Switch 1 | GPIO27 |
| Switch 2 | GPIO14 |
| Switch 3 | GPIO12 |
| Switch 4 | GPIO13 |

### 11-8. Weather

When a DHT22 sensor is connected, displays temperature and humidity in real time.

- **Temperature**: Air temperature (°C)
- **Humidity**: Humidity (%)

Displays `N/A` when no sensor is connected.

---

## 12. Connecting Astronomy Apps via TCP (SkySafari, etc.)

AiBridge listens on 2 TCP ports simultaneously for the LX200 protocol.

| Port | Purpose |
|------|---------|
| **9999** | Main port (SkySafari, etc.) |
| **9998** | Sub port (for simultaneous second client connection) |

### 12-1. SkySafari Configuration

1. SkySafari → **Settings → Telescope → Setup**
2. Configure the following:

| Item | Value |
|------|-------|
| Scope Type | Meade LX-200 GPS (or compatible) |
| Mount | Select according to your hardware |
| Connection Method | **WiFi** |
| IP Address | AiBridge STA address (e.g., `192.168.3.154`) |
| Port Number | **9999** |

3. Tap **"Connect to Telescope"**

### 12-2. Notes for iOS (SkySafari) Users

For WiFi connection stability on iOS, please note:

- Verify AiBridge and your iOS device are connected to the **same WiFi network**
- If SkySafari disconnects, restart the app and reconnect
- v7.14+: iOS connection stability implemented via `setNoDelay(true)` and `WiFi.setSleep(false)`
- Idle connections time out after 30 seconds (auto-disconnect)

### 12-3. Using Both Ports Simultaneously

SkySafari (port 9999) and another app (port 9998) can be connected at the same time.

```
SkySafari    ─── TCP:9999 ─── AiBridge ─── LX200 ─── Telescope
PHD2, etc.   ─── TCP:9998 ─┘
```

### 12-4. iOS TCP Stability Measures (v7.14+)

The iOS stability improvements implemented in v7.14 are carried over in v7.15.

| Measure | Purpose |
|---------|---------|
| `WiFi.setSleep(false)` | Disables Modem-Sleep. Prevents iOS TCP timeouts during beacon intervals |
| `setNoDelay(true)` | Disables Nagle algorithm. Sends short LX200 commands immediately |
| `client.stop()` on disconnect | Releases socket resources. Prevents socket exhaustion on iOS reconnect |
| Idle timeout 30 seconds | Detects iOS "silent disconnect" |
| Command receive timeout 100ms | Handles packet delays due to iOS TCP power saving |

---

## 13. ASCOM Alpaca Support

AiBridge supports the ASCOM Alpaca v1 protocol. Direct control from Windows astronomy software such as NINA and Sequence Generator Pro is available.

### 13-1. Alpaca Discovery

Automatically responds to UDP Discovery (port 32227) from Alpaca-compatible software.
Running "Search for Devices" in NINA or similar will automatically detect AiBridge.

Display name when detected: `Ai-` + Location Name (e.g., `Ai-Observatory`)

### 13-2. Supported Devices

| Device Type | Device Number | Content |
|-------------|--------------|---------|
| **Telescope** | 0 | LX200-compatible telescope control |
| **ObservingConditions** | 0 | DHT22 temperature and humidity |
| **Switch** | 0 | 4-channel GPIO ON/OFF |

### 13-3. Telescope Device (Key Properties)

| Property | Description |
|----------|-------------|
| `RightAscension` | Current RA (decimal hours) |
| `Declination` | Current Dec (decimal degrees) |
| `Tracking` | Tracking state (true/false) |
| `Slewing` | Whether slewing (true/false) |
| `SlewToCoordinatesAsync` | Coordinate-specified slew |
| `AbortSlew` | Abort slew |
| `PulseGuide` | Pulse guide (PHD2 compatible) |
| `CanPulseGuide` | true (pulse guide supported) |

### 13-4. Using with NINA

1. NINA → **Equipment → Telescope → Alpaca**
2. Select AiBridge when it is auto-detected and connect
3. After connecting, GoTo, tracking control, and pulse guide are available from NINA

### 13-5. Telescope State Monitoring (Auto Classification)

AiBridge reads coordinates every 3 seconds and automatically determines the state from movement speed.

| State | Speed Ratio (sidereal rate multiple) | Description |
|-------|--------------------------------------|-------------|
| `Tracking` | Less than ×0.4 | Normal tracking |
| `Guiding` | ×0.4 to 5.0 | Autoguide correction in progress |
| `Slewing` | ×5.0 or more | GoTo slew in progress |

---

# Part 5 — API Reference

## 14. Network Configuration & Protocols

### 14-1. Wi-Fi Mode

AiBridge operates in `WIFI_AP_STA` mode, providing both AP and STA functionality simultaneously.

#### Access Point (AP) Mode

| Item | Value |
|------|-------|
| **SSID** | `AiBridgeXXXX` (auto-generated from last 4 digits of MAC) |
| **Password** | None (open network) |
| **IP Address** | 192.168.4.1 (fixed) |

#### MAC Address Auto SSID Generation *(New in v7.15)*

```
MAC Address: AA:BB:CC:DD:A1:B2
  → AP SSID: "AiBridgeA1B2"
```

- Uses `ESP.getEfuseMac()` (`WiFi.softAPmacAddress()`, which returns zero before initialization, is not used)
- No SSID conflicts even when operating multiple units simultaneously

#### Station (STA) Mode

| Item | Value |
|------|-------|
| **IP Address** | Automatically assigned by DHCP |
| **Configuration** | Edit `aibridge_net_cfg.dat` via File Manager |

### 14-2. Supported Protocols

| Protocol | Port | Description |
|----------|------|-------------|
| LX200 TCP | 9999, 9998 | Supports multiple simultaneous client connections on 2 ports |
| ASCOM Alpaca HTTP | 80 (RESTful API) | NINA, Stellarium, etc. connect directly |
| Alpaca Discovery UDP | 32227 | Automatic device discovery |

---

## 15. API Reference

**Base URL**: `http://<AiBridge IP>/`

---

### 15-1. OnStep Direct Control API

Coordinates use HMS/DMS format (`HH:MM:SS`, `±DD:MM:SS`).

#### GET /api/status

Returns the overall system status.

**Response Example:**
```json
{
  "version": "7.15",
  "location": "Bridge",
  "ra": "05:34:32",
  "dec": "+22:00:52",
  "tracking": true,
  "slewing": false,
  "temperature": 18.5,
  "humidity": 62.3,
  "wifi_ssid": "MyHomeNetwork",
  "wifi_ip": "192.168.3.154",
  "ap_ssid": "AiBridgeA1B2",
  "ap_ip": "192.168.4.1",
  "uptime": 3600
}
```

#### GET /api/ra / GET /api/dec

Returns current Right Ascension / Declination as plain text.

```
Response: 05:34:32
```

#### GET /api/command

Sends an arbitrary LX200 command.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `cmd` | string | ✅ | LX200 command (without the `:` and `#`) |

**Examples:**
```
GET /api/command?cmd=GR   → Sends :GR# (get RA)
GET /api/command?cmd=Q    → Sends :Q# (stop)
```

**Response Example:**
```json
{
  "status": "success",
  "command": "GR",
  "response": "05:34:32#"
}
```

> **v7.15 Change**: Timeout extended from 40ms to **100ms**.

#### GET /api/goto

Slews to the specified coordinates.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `ra` | string | ✅ | Right Ascension (HH:MM:SS format) |
| `dec` | string | ✅ | Declination (±DD:MM:SS format) |

**Example:** `GET /api/goto?ra=05:34:32&dec=+22:00:52`

**Response Example:**
```json
{
  "status": "success",
  "ra": "05:34:32",
  "dec": "+22:00:52"
}
```

#### GET /api/sync

Syncs the current position to the specified coordinates (`:CM#`).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `ra` | string | ✅ | Right Ascension (HH:MM:SS format) |
| `dec` | string | ✅ | Declination (±DD:MM:SS format) |

#### GET /api/tracking

Controls tracking ON/OFF.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `state` | string | ✅ | `on` or `off` |

#### GET /api/stop

Immediately stops slewing (`:Q#`).

**Response Example:** `{"status": "success"}`

#### GET /api/switch

Turns a GPIO switch ON or OFF.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | integer | ✅ | Switch number (0–3) |
| `state` | string | ✅ | `on` or `off` |

**Examples:**
```
GET /api/switch?id=0&state=on    → Set GPIO27 HIGH
GET /api/switch?id=2&state=off   → Set GPIO12 LOW
```

**Response Example:**
```json
{"status": "success", "switch": 0, "state": "on"}
```

#### GET /api/weather

Returns DHT22 sensor readings.

**Response Example:**
```json
{"temperature": 18.5, "humidity": 62.3, "sensor": "DHT22"}
```

When no sensor is connected:
```json
{"temperature": null, "humidity": null, "sensor": "N/A"}
```

---

### 15-2. Dynamic Dashboard UI API *(New in v7.15)*

| Endpoint | Method | Function |
|----------|--------|---------|
| `/api/ui/upload` | POST | Saves HTML to SPIFFS as `/index2.html` |
| `/api/ui/action` | POST | Browser notifies of button press (`?action=xxx`) |
| `/api/ui/action` | GET | LUNA polls for button notifications (clears after retrieval) |
| `/api/ui/state` | GET | Check current notification state (does not clear) |

**POST /api/ui/upload Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `filename` | string | Filename to save (default: `index2.html`) |

**Response Example:**
```json
{"status": "success", "file": "/index2.html", "size": 1024}
```

**GET /api/ui/action Response Example:**
```json
// Notification present
{"status": "success", "action": "goto_vega", "pending": true}
// No notification
{"status": "success", "action": "", "pending": false}
```

---

### 15-3. File Management API

> ⚠️ File Manager API requires activation via the GPIO39 button.

| Endpoint | Method | Function |
|----------|--------|---------|
| `/files` | GET | Display File Manager Web UI |
| `/list` | GET | Return SPIFFS file list (JSON) |
| `/upload` | POST | Upload file (`multipart/form-data`) |
| `/delete` | DELETE | Delete file (`?path=/filename`) |
| `/format` | POST | Format SPIFFS (deletes all files) |

**GET /list Response Example:**
```json
[
  {"name": "/aibridge_net_cfg.dat", "size": 48},
  {"name": "/index2.html", "size": 2048}
]
```

---

### 15-4. ASCOM Alpaca Device API

> **Note**: Coordinate format follows Alpaca standard — decimal numbers (RA: hours, Dec: degrees).

#### Management API

| Endpoint | Method | Function |
|----------|--------|---------|
| `/management/apiversions` | GET | Supported API versions |
| `/management/v1/description` | GET | Device description and manufacturer info |
| `/management/v1/configureddevices` | GET | Configured device list (Telescope / ObservingConditions / Switch) |

**Discovery:** Auto-responds on UDP port 32227

#### Telescope Device (`/api/v1/telescope/0/`)

| Endpoint | Method | Function |
|----------|--------|---------|
| `connected` | GET/PUT | Connection state |
| `rightascension` | GET | Current RA (hour angle) |
| `declination` | GET | Current Dec (degrees) |
| `tracking` | GET/PUT | Tracking state |
| `slewing` | GET | Slewing state |
| `atpark` | GET | Park state |
| `slewtocoordinatesasync` | PUT/GET | Async GoTo (RA/Dec specified) |
| `slewtocoordinates` | PUT | Sync GoTo (waits for completion) |
| `synctocoordinates` | PUT | Coordinate sync (`:CM#`) |
| `abortslew` | PUT/GET | Abort slew |
| `canpulseguide` | GET | Pulse guide support (true) |
| `pulseguide` | PUT | Send pulse guide (PHD2 compatible) |
| `ispulseguiding` | GET | Whether pulse guiding is active |
| `siderealtime` | GET | Local sidereal time |
| `siteelevation` | GET/PUT | Site elevation |
| `sitelatitude` | GET/PUT | Site latitude |
| `sitelongitude` | GET/PUT | Site longitude |

> `slewtocoordinatesasync` and `abortslew` support both PUT and GET for broader client compatibility. PUT is the Alpaca standard.

#### ObservingConditions Device (`/api/v1/observingconditions/0/`)

| Endpoint | Method | Function |
|----------|--------|---------|
| `connected` | GET/PUT | Connection state |
| `temperature` | GET | Air temperature (°C) |
| `humidity` | GET | Humidity (%) |
| `refresh` | PUT | Update sensor values |

> Returns `-999.0` when no sensor is connected or reading fails.

#### Switch Device (`/api/v1/switch/0/`)

| Endpoint | Method | Function |
|----------|--------|---------|
| `connected` | GET/PUT | Connection state |
| `maxswitch` | GET | Total number of switches (4) |
| `getswitch` | GET | Get ON/OFF state (`?Id=0-3`) |
| `setswitch` | PUT | Set state (`Id`, `State`) |
| `getswitchname` | GET | Get switch name |
| `getswitchdescription` | GET | Get switch description |

**Switch ↔ GPIO Mapping**

| Switch | GPIO |
|--------|------|
| Switch 0 | GPIO27 |
| Switch 1 | GPIO14 |
| Switch 2 | GPIO12 |
| Switch 3 | GPIO13 |

---

### 15-5. External Alpaca Device Proxy API

| Endpoint | Method | Function |
|----------|--------|---------|
| `/api/external/discover` | GET | Discovers all Alpaca devices via UDP broadcast (IP, port, model, type) |
| `/api/external/alpaca` | GET | Proxies any property of any Alpaca device (`target`, `device`, `property` specified) |

---

### 15-6. Error Response Format

**Common error for custom API:**
```json
{
  "status": "error",
  "message": "Error description"
}
```

**Alpaca API error (Alpaca spec compliant):**
```json
{
  "Value": null,
  "ErrorNumber": 1024,
  "ErrorMessage": "Not connected"
}
```

---

## 16. Web UI Pages

| URL | Description |
|-----|-------------|
| `http://<IP>/` | AiBridge Console (main control screen) |
| `http://<IP>/index2.html` | Custom dashboard (auto-displayed when GPIO33 is LOW) |
| `http://<IP>/files` | File Manager (requires GPIO39 activation) |
| `http://<IP>/openapi.json` | OpenAPI specification (JSON) |
| `http://<IP>/alpaca-api-spec.json` | Alpaca API specification (JSON) |

---

# Part 6 — LUNA Integration & Observatory Control

## 17. Dynamic Dashboard (LUNA Integration)

In collaboration with LUNA AI Gateway, the AiBridge browser screen can be dynamically changed.

### 17-1. How It Works

```
LUNA (Lua Script)
    │
    ├── POST /api/ui/upload  → Save custom HTML to AiBridge SPIFFS
    │
    └── GET  /api/ui/action  → Poll for browser button operations
              ↑
        Browser displays /index2.html
        User presses button → POST /api/ui/action
```

### 17-2. GET `/api/ui/action` Response Example

```json
// Button pressed
{"status": "success", "action": "goto_vega", "pending": true}

// No notification
{"status": "success", "action": "", "pending": false}
```

### 17-3. LUNA Lua Script Example

```lua
-- Deliver custom HTML to AiBridge
local html = [[
<!DOCTYPE html><html><body>
<h2>Observatory Control</h2>
<button onclick="fetch('/api/ui/action?action=goto_vega',{method:'POST'})">
  Slew to Vega
</button>
</body></html>
]]
http.post("http://192.168.3.154/api/ui/upload", html, 3000)

-- Poll for browser button operations
while true do
  local res = http.get("http://192.168.3.154/api/ui/action", 500)
  if res and res ~= "" then
    local data = json.parse(res)
    if data and data.action == "goto_vega" then
      -- GoTo Vega process
      serial.write(":SrXX:XX:XX#:SdXX:XX:XX#:MS#")
    end
  end
  hw.delay(500)
end
```

### 17-4. Displaying the Custom UI

1. Send HTML from LUNA to `/api/ui/upload`
2. Open `http://<AiBridge IP>/index2.html` in a browser
   (Or boot with GPIO33 pulled LOW for automatic display)

---

## 18. Integration with LUNA AI Gateway

### 18-1. System Configuration

```
Claude Desktop
    │ MCP Protocol
    └── LUNA (ESP32)
            │ HTTP (http.get / http.post)
            └── AiBridge (ESP32)
                    │ LX200 Serial (9600bps)
                    └── Telescope Mount
```

### 18-2. Controlling AiBridge from LUNA

You can call the AiBridge REST API from LUNA Lua scripts:

```lua
-- Get RA/Dec
local ra  = http.get("http://192.168.3.154/api/ra", 500)
local dec = http.get("http://192.168.3.154/api/dec", 500)
log.write("RA=" .. ra .. " Dec=" .. dec)

-- Send LX200 command
http.get("http://192.168.3.154/api/command?cmd=Q", 500)  -- Stop

-- Execute GoTo
http.get("http://192.168.3.154/api/goto?ra=18:36:56&dec=+38:47:01", 2000)
```

### 18-3. Publishing as MCP Resource

Using LUNA's MCP Resources feature, the current telescope position can be made directly readable by Claude:

```lua
-- /luna_resource_aibridge.lua
local ra  = http.get("http://192.168.3.154/api/ra",  500)
local dec = http.get("http://192.168.3.154/api/dec", 500)
return '{"ra":"' .. (ra or "N/A") .. '","dec":"' .. (dec or "N/A") .. '"}'
```

---

## 19. Observatory Integration Vision (Sorami System)

### 19-1. Concept

v7.15 is the first step of a platform where **Claude AI becomes the orchestrator of the entire observatory, providing a new astronomical experience for general users**.

```
[Natural visitor queries]
"What star are we looking at?"
"I want to see M31."
"What objects are recommended tonight?"
            ↓
[Claude AI (Orchestrator)]
· Verify and calculate celestial information
· Optimize equipment settings
· Manage observation sequence execution
· Report progress and results
            ↓
[LUNA Observatory + AiBridge]
Integrated control of telescope, camera, dome, weather sensors, etc.
            ↓
[Dynamic Dashboard (Browser)]
Real-time status, celestial images, interaction
```

### 19-2. Visitor Guide System Example

**Scenario:**

```
Visitor: "What star are we looking at?"
        ↓ Sent from browser button
Claude (via LUNA):
  1. Get current RA/Dec from AiBridge
  2. Identify the object in the star catalog
  3. Compile information about the object and respond
  4. Display star chart and explanation on dashboard
        ↓
"Aldebaran in Taurus. A red supergiant..." displayed and narrated
```

### 19-3. Implementation Roadmap

| Stage | Feature | Technology | Status |
|-------|---------|------------|--------|
| **v7.15** | Text Q&A + telescope real-time dashboard | Dynamic Dashboard UI API | ✅ Complete |
| **v7.16+** | Voice narration output | OpenAI TTS MCP (Claude Desktop side) | 📋 Next |
| **v7.17+** | Immersive experience with BGM and sound effects | Audio Playback MCP + Epidemic Sound MCP | 📋 Planned |
| **v8.0+** | Image/video/audio multimedia integration | File server + LUNA multimodal | 📋 Future |

### 19-4. Component Roles

| Component | Role |
|-----------|------|
| **AiBridge** | Provides telescope position and sensor information |
| **LUNA Observatory** | Integrates multiple devices (camera, dome, etc.) |
| **Claude AI** | Answers and makes decisions using astronomical knowledge + real-time data |
| **Dynamic Dashboard** | Displays text, images, and buttons in browser |
| **Audio Output (v7.16+)** | Narration via OpenAI TTS MCP |

---

# Part 7 — Appendix

## 20. Troubleshooting

| Symptom | Cause | Solution |
|---------|-------|---------|
| `AiBridgeXXXX` SSID not visible | Power not on / booting | Check LED. Verify USB power |
| Cannot connect to AP | Password set | v7.15 default is open (no password) |
| `http://192.168.4.1` does not open | Not connected to AP | Select `AiBridgeXXXX` in WiFi settings |
| STA address not displayed | WiFi not configured / SSID or password incorrect | Check WiFi settings in File Manager |
| SkySafari cannot connect | Wrong IP address or port | Check STA address and port 9999 |
| SkySafari RA/Dec not updating | Telescope serial not connected | Check wiring and baud rate (9600bps) |
| Not shown in Alpaca Discovery | Different network | Verify PC and AiBridge are on same WiFi |
| File Manager does not open | GPIO39 button not pressed | Press the button, then open File Manager |
| WiFi settings not saved | SPIFFS not formatted | Run Format SPIFFS (first time only) |
| `/api/command` response slow or empty | Timeout too short | Extended to 100ms in v7.15. Check OnStep response speed |
| iOS SkySafari disconnects frequently | WiFi sleep | Improved in v7.14+ (`WiFi.setSleep(false)`) |
| Custom UI not displayed | GPIO33 or index2.html not configured | Connect GPIO33 to GND / upload HTML in File Manager |

---

## 21. Technical Specifications

| Item | Specification |
|------|--------------|
| **MCU** | ESP32-WROOM-32 (Xtensa LX6 Dual-Core 240MHz) |
| **RAM** | 520KB SRAM |
| **Flash** | 4MB (recommended) |
| **Partition** | Huge APP 3MB / SPIFFS 1MB |
| **WiFi** | 802.11 b/g/n (2.4GHz) AP + STA simultaneous |
| **AP IP** | 192.168.4.1 (fixed) |
| **AP SSID** | `AiBridgeXXXX` (last 4 digits of MAC, auto-generated) |
| **AP Password** | None (open) |
| **STA IP** | DHCP auto-assigned |
| **TCP Ports** | 9999 / 9998 (LX200) |
| **HTTP Port** | 80 |
| **Alpaca Discovery** | UDP port 32227 |
| **Serial (OnStep)** | UART0 or UART2 (selected by GPIO32) 9600bps 8N1 |
| **Serial (Debug)** | UART1 (GPIO19/18) 115200bps |
| **DHT Sensor** | GPIO5 (DHT22 / DHT11) |
| **Switch Output** | GPIO27 / 14 / 12 / 13 (4 channels) |
| **File Manager LED** | GPIO25 |
| **File Manager Button** | GPIO39 (internal pull-up) |
| **Custom UI Select** | GPIO33 (LOW: index2.html, HIGH: standard Console) |
| **Serial Select** | GPIO32 (HIGH: UART0, LOW: UART2) |
| **Operating Voltage** | 3.3V logic / 5V USB power |

---

## 22. Version History

| Version | Date | Key Changes |
|---------|------|-------------|
| v7.13 | 2025-10 | Base version — LX200 + Alpaca + File Manager |
| v7.14 | 2026-03 | iOS TCP stability improvements (Nagle disabled, socket release, idle timeout, Modem-Sleep disabled) |
| **v7.15** | **2026-04** | **MAC auto SSID, Dynamic Dashboard UI API, /api/command timeout 100ms** |

### v7.15 Change Summary

| Change | Details |
|--------|---------|
| AP SSID auto-generation | Generates `AiBridgeXXXX` from last 4 digits of MAC. Unique per unit |
| AP password default | Changed to open (no password). Easier initial connection |
| Dynamic Dashboard UI API | 4 new endpoints: `/api/ui/upload`, `/api/ui/action`, `/api/ui/state` |
| `/api/command` timeout | Extended 40ms → **100ms** to reliably capture OnStep responses |
| Observatory integration vision | Formally adopts LUNA Observatory integrated architecture |

---

## 23. Boot Log Example

The following log is output to the serial monitor (115200bps) when AiBridge boots:

```
AiBridge v7.15 starting...
AP SSID: AiBridgeA1B2
AP IP: 192.168.4.1
Connecting to WiFi: MyHomeNetwork
WiFi connected. STA IP: 192.168.3.154
TCP server 1 started on port 9999
TCP server 2 started on port 9998
HTTP server started
Alpaca Discovery listening on UDP 32227
Ready.
```

---

*AiBridge Complete Manual v7.15*
*Copyright (c) 2026 Nishioka Sadahiko / 西岡定彦*
*MIT License — Binary distribution only · Source code not publicly available*
