# AiBridge API Reference v7.15

**AiBridge** is an ESP32-based WiFi/Web server that bridges OnStep and NS-5000 telescope mounts
to AI agents (Claude, etc.) and astronomy applications (NINA / Stellarium / SkySafari / PHD2).

- **Author**: Nishioka Sadahiko  
- **License**: MIT  
- **GitHub**: https://github.com/OnStepNinja

---

## IP Address

| Mode | IP | Notes |
|------|----|-------|
| AP mode (default) | `192.168.4.1` | SSID: "AiBridgeXXXX" (last 4 digits of MAC) |
| STA mode | DHCP assigned | Check the console after WiFi setup |

---

## API Groups

1. [OnStep Direct Control API](#1-onstep-direct-control-api)
2. [Dynamic UI API (LUNA Integration)](#2-dynamic-ui-api-luna-integration)
3. [External Alpaca Proxy API](#3-external-alpaca-proxy-api)
4. [ASCOM/Alpaca Telescope API](#4-ascomalpaca-telescope-api)
5. [ASCOM/Alpaca ObservingConditions API](#5-ascomalpaca-observingconditions-api)
6. [ASCOM/Alpaca Switch API](#6-ascomalpaca-switch-api)
7. [Alpaca Management API](#7-alpaca-management-api)
8. [System Management API](#8-system-management-api)
9. [TCP LX200 Ports (SkySafari, etc.)](#9-tcp-lx200-ports-skysafari-etc)
10. [Web UI](#10-web-ui)

---

## 1. OnStep Direct Control API

Directly controls the OnStep / NS-5000 mount connected to AiBridge. Coordinates use LX200 format (HMS/DMS).

### GET /api/status — Get telescope status

```
GET /api/status
```

Response example:
```json
{
  "status": "success",
  "message": "Status retrieved",
  "data": {
    "ra": "12:30:45",
    "dec": "+45:30:00",
    "state": "Tracking",
    "speedRatio": 1.0,
    "raSpeed": 15.041,
    "decSpeed": 0.0,
    "connected": true
  }
}
```

- `state`: `Stopped` / `Tracking` / `Guiding` / `Slewing`
- `speedRatio`: speed ratio relative to sidereal rate (1.0 = sidereal tracking)
- `raSpeed` / `decSpeed`: speed in arcsec/second

---

### GET /api/goto — Slew to coordinates

```
GET /api/goto?ra=12:30:45&dec=+45:30:00
```

| Parameter | Format | Example |
|-----------|--------|---------|
| `ra` | HH:MM:SS | `12:30:45` |
| `dec` | ±DD:MM:SS | `+45:30:00` |

Response example:
```json
{"status": "success", "message": "GOTO command sent", "data": "RA:12:30:45 DEC:+45:30:00"}
```

---

### GET /api/command — Send LX200 command

```
GET /api/command?cmd=GR
```

- `cmd`: LX200 command (without `:` prefix or `#` suffix)

Common commands:

| Command | Action | Response example |
|---------|--------|-----------------|
| `GR` | Get Right Ascension | `12:30:45#` |
| `GD` | Get Declination | `+45:30:00#` |
| `Sr12:30:45` | Set target RA | `1` |
| `Sd+45:30:00` | Set target DEC | `1` |
| `MS` | Execute GOTO | `0` |
| `Q` | Stop all motion | `` |
| `Mn` / `Ms` / `Me` / `Mw` | Manual move (N/S/E/W) | `` |
| `RS` / `RM` / `RC` | Speed: slew / find / center | `` |
| `SB` | Buzzer | `` |
| `GVP` | Get product name | `OnStep#` |
| `GVN` | Get firmware version | `4.24k#` |

Response example:
```json
{"status": "success", "message": "Command executed", "data": "12:30:45#"}
```

---

### GET /api/stop — Emergency stop

```
GET /api/stop
```

```json
{"status": "success", "message": "Emergency stop executed"}
```

---

### GET /api/buzzer — Buzzer

```
GET /api/buzzer
```

---

## 2. Dynamic UI API (LUNA Integration)

Allows LUNA to generate and serve HTML dashboards, and receive browser button events. (Added in v7.15)

### POST /api/ui/upload — Save HTML to SPIFFS

```
POST /api/ui/upload?filename=index2.html
Content-Type: text/html

<html>...</html>
```

- Default filename if omitted: `/index2.html`
- LUNA Lua example: `http.post("http://192.168.3.7/api/ui/upload?filename=myapp.html", html_content, 3000)`

---

### POST /api/ui/action — Button notification from browser

Called from browser-side JavaScript:
```javascript
fetch('/api/ui/action?action=next', {method: 'POST'})
// or
fetch('/api/ui/action', {method: 'POST', body: JSON.stringify({action: 'start'})})
```

---

### GET /api/ui/action — LUNA polls for button events

```
GET /api/ui/action
```

Response when a button was pressed:
```json
{"status": "success", "action": "next", "pending": true}
```

Response when no button pressed:
```json
{"status": "success", "action": "", "pending": false}
```

- The queue is cleared after retrieval.

---

### GET /api/ui/state — Check dashboard state

```
GET /api/ui/state
```

- Returns current state without clearing the queue.

---

## 3. External Alpaca Proxy API

Controls other ASCOM/Alpaca devices on the network (other mounts, focusers, etc.) through AiBridge.

### GET /api/external/discover — Auto-discover devices

```
GET /api/external/discover
```

Broadcasts on UDP port 32227 and returns a list of devices found.

Response example:
```json
{
  "status": "success",
  "devices": [
    {"ip": "192.168.3.120", "port": 11111, "deviceType": "telescope", "deviceNumber": 0, "deviceName": "ASCOM Remote Telescope"}
  ]
}
```

---

### GET /api/external/alpaca — Universal Alpaca proxy (GET)

```
GET /api/external/alpaca?target=192.168.3.120&device=telescope&property=rightascension
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `target` | ✅ | IP address of the target device |
| `device` | ✅ | Device type (telescope / focuser / filterwheel / etc.) |
| `property` | ✅ | Alpaca property name (lowercase) |
| `number` | - | Device number (default: 0) |
| `port` | - | Alpaca server port (default: 80) |

Response example:
```json
{
  "status": "success",
  "data": {"Value": 12.5125, "ErrorNumber": 0, "ErrorMessage": "", "ClientTransactionID": 1, "ServerTransactionID": 1}
}
```

- The value is in `data.Value` — no JSON.parse needed.
- `data.ErrorNumber === 0` indicates success.

---

### PUT /api/external/alpaca — Universal Alpaca proxy (PUT/POST)

```
GET /api/external/alpaca?target=192.168.3.120&device=telescope&property=slewtocoordinatesasync&method=PUT&params=RightAscension%3D12.5125%26Declination%3D45.5
```

| Parameter | Description |
|-----------|-------------|
| `method` | `PUT` or `POST` |
| `params` | URL-encoded parameter string |

**Coordinate conversion (Alpaca requires decimal format)**:
- RA: `HH:MM:SS` → decimal hours, e.g. `12:30:45` → `12.5125`
- DEC: `±DD:MM:SS` → decimal degrees, e.g. `+45:30:00` → `45.5`
- Formula: `degrees + minutes/60 + seconds/3600`

---

### GET /api/external/status — Get external telescope status (simplified)

```
GET /api/external/status?target=192.168.3.120
```

---

### GET /api/external/goto — GOTO on external telescope

```
GET /api/external/goto?target=192.168.3.120&ra=12.5125&dec=45.5
```

---

### GET /api/external/stop — Stop external telescope

```
GET /api/external/stop?target=192.168.3.120
```

---

## 4. ASCOM/Alpaca Telescope API

Astronomy software such as NINA, Stellarium, and PHD2 connect to AiBridge directly as an Alpaca server.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/telescope/0/connected` | GET / PUT | Connection status (always true) |
| `/api/v1/telescope/0/interfaceversion` | GET | Interface version (3) |
| `/api/v1/telescope/0/name` | GET | Telescope name ("Ai-{Location} Telescope") |
| `/api/v1/telescope/0/description` | GET | Description string |
| `/api/v1/telescope/0/driverinfo` | GET | Driver info |
| `/api/v1/telescope/0/driverversion` | GET | Driver version |
| `/api/v1/telescope/0/supportedactions` | GET | Supported actions (empty array) |
| `/api/v1/telescope/0/atpark` | GET | Park state (always false) |
| `/api/v1/telescope/0/rightascension` | GET | Current RA (decimal hours) |
| `/api/v1/telescope/0/declination` | GET | Current DEC (decimal degrees) |
| `/api/v1/telescope/0/tracking` | GET / PUT | Tracking ON/OFF |
| `/api/v1/telescope/0/slewing` | GET | Whether the mount is moving |
| `/api/v1/telescope/0/canslew` | GET | GOTO capable (true) |
| `/api/v1/telescope/0/slewtocoordinatesasync` | PUT | Async GOTO (decimal coordinates) |
| `/api/v1/telescope/0/abortslew` | PUT | Abort slew |
| `/api/v1/telescope/0/canpulseguide` | GET | PulseGuide capable (true) |
| `/api/v1/telescope/0/pulseguide` | PUT | Execute pulse guide (PHD2) |
| `/api/v1/telescope/0/ispulseguiding` | GET | Whether pulse guiding is active |

**slewtocoordinatesasync PUT parameters**:
```
RightAscension=12.5125&Declination=45.5
```

**pulseguide PUT parameters**:
```
Direction=0&Duration=500
```
Direction: 0=North / 1=South / 2=East / 3=West  
Duration: milliseconds

---

## 5. ASCOM/Alpaca ObservingConditions API

Returns weather data from a DHT22 sensor connected to GPIO5.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/observingconditions/0/connected` | GET / PUT | Connection state |
| `/api/v1/observingconditions/0/temperature` | GET | Temperature (°C), -999.0 on error |
| `/api/v1/observingconditions/0/humidity` | GET | Humidity (%), -999.0 on error |
| `/api/v1/observingconditions/0/timesincelastupdate` | GET | Seconds since last update |

- Update interval: 60 seconds
- DHT11 also supported

---

## 6. ASCOM/Alpaca Switch API

4-channel relay control.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/switch/0/connected` | GET / PUT | Connection state |
| `/api/v1/switch/0/maxswitch` | GET | Number of switches (4) |
| `/api/v1/switch/0/getswitch?Id=0` | GET | Get switch state (Id: 0–3) |
| `/api/v1/switch/0/setswitch` | PUT | Set switch ON/OFF (Id and State parameters) |
| `/api/v1/switch/0/getswitchname?Id=0` | GET | Switch name ("Output 1", etc.) |
| `/api/v1/switch/0/getswitchdescription?Id=0` | GET | Switch description |

GPIO pin mapping:

| Id | GPIO |
|----|------|
| 0 | GPIO27 |
| 1 | GPIO14 |
| 2 | GPIO12 |
| 3 | GPIO13 |

**setswitch PUT parameters**:
```
Id=0&State=true
```

---

## 7. Alpaca Management API

| Endpoint | Description |
|----------|-------------|
| `GET /management/apiversions` | Supported Alpaca API versions ([1]) |
| `GET /management/v1/description` | Server information |
| `GET /management/v1/configureddevices` | List of registered devices (Telescope / ObservingConditions / Switch) |

---

## 8. System Management API

### WiFi Configuration

```
GET  /api/wifi/config       — Get current WiFi settings
POST /api/wifi/config       — Update WiFi settings (SSID / Password / Location / etc.)
```

### File Management (SPIFFS)

```
GET    /api/files/list                        — List files
POST   /api/files/upload                      — Upload file
DELETE /api/files/delete?filename=xxx         — Delete file
GET    /api/system/format_spiffs              — Check SPIFFS status (capacity info)
POST   /api/system/format_spiffs?confirm=YES  — Format SPIFFS
```

### Documentation

```
GET /api/spec       — OpenAPI JSON spec (alias for openapi.json)
GET /openapi.json   — OpenAPI JSON spec
GET /api/license    — License info (JSON)
GET /license        — License page (HTML)
```

---

## 9. TCP LX200 Ports (SkySafari, etc.)

Direct LX200 protocol connections from astronomy apps.

| Port | Purpose |
|------|---------|
| **9999** | LX200 TCP primary (SkySafari, etc.) |
| **9998** | LX200 TCP secondary (simultaneous 2nd connection) |

Stability features (added in v7.14):
- `WiFi.setSleep(false)` — Disables modem sleep for iOS compatibility
- `setNoDelay(true)` — Disables Nagle algorithm (immediate small packet delivery)
- 30-second idle timeout — Automatically closes zombie connections

---

## 10. Web UI

| URL | Description |
|-----|-------------|
| `/` | Root (index.html or AiBridge Console) |
| `/control` | AiBridge Console (telescope control & status) |
| `/files` | File Manager (SPIFFS files & WiFi settings) |

- When GPIO33 = LOW at boot, `/` serves `index2.html` (custom UI) first
- GPIO32 switches SerialOnStep between UART0 (HIGH) and UART2 (LOW)

---

## Alpaca Error Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1024 | Not implemented |
| 1025 | Invalid value |
| 1027 | Not connected |
| 1030 | Invalid coordinates (unreachable target) |

---

## Hardware Pin Reference

| Function | GPIO |
|----------|------|
| DHT22 sensor | GPIO5 |
| Relay Ch0 | GPIO27 |
| Relay Ch1 | GPIO14 |
| Relay Ch2 | GPIO12 |
| Relay Ch3 | GPIO13 |
| Custom UI toggle | GPIO33 |
| SerialOnStep select | GPIO32 |
| UART0 (TX/RX) | GPIO1 / GPIO3 |
| UART2 (TX/RX) | GPIO17 / GPIO16 |
| Debug UART1 (TX/RX) | GPIO19 / GPIO18 |

---

*AiBridge v7.15 — Copyright (c) 2025-2026 Nishioka Sadahiko / MIT License*
