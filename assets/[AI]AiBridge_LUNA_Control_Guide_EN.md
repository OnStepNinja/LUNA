# Controlling AiBridge from LUNA

A practical guide to operating AiBridge from LUNA Lua scripts using `http.get` and `http.post`.

**Prerequisites**:
- Set the AiBridge IP address in the `BRIDGE` variable
- AiBridge must be connected to an OnStep or NS-5000 mount

---

## Basic Pattern

```lua
local BRIDGE = "http://192.168.3.7"   -- Change to your AiBridge IP

-- GET request
local res = http.get(BRIDGE .. "/api/status", 2000)

-- POST request
local res = http.post(BRIDGE .. "/api/ui/upload?filename=myapp.html", html_body, 3000)
```

Responses are JSON strings. Parse them with `json.parse()`:

```lua
local data = json.parse(res)
if data and data.status == "success" then
    -- handle success
end
```

---

## 1. Get telescope status

```lua
local BRIDGE = "http://192.168.3.7"

local res = http.get(BRIDGE .. "/api/status", 2000)
local data = json.parse(res)

if data and data.status == "success" then
    local d = data.data
    log.write("RA: " .. d.ra)
    log.write("DEC: " .. d.dec)
    log.write("State: " .. d.state)
    log.write("Speed Ratio: " .. tostring(d.speedRatio))
end
```

`data.state` is one of `"Stopped"` / `"Tracking"` / `"Guiding"` / `"Slewing"`.

---

## 2. Slew to coordinates (GOTO)

```lua
local BRIDGE = "http://192.168.3.7"

-- Andromeda Galaxy M31 (example)
local ra  = "00:42:44"
local dec = "+41:16:09"

local url = BRIDGE .. "/api/goto?ra=" .. ra .. "&dec=" .. dec
local res = http.get(url, 5000)
local data = json.parse(res)

if data and data.status == "success" then
    log.write("GOTO sent: " .. data.data)
end
```

---

## 3. GOTO and wait for completion

```lua
local BRIDGE = "http://192.168.3.7"

-- Send GOTO
http.get(BRIDGE .. "/api/goto?ra=00:42:44&dec=+41:16:09", 5000)

-- Wait until state returns to Tracking
for i = 1, 60 do
    hw.delay(3000)
    local res = http.get(BRIDGE .. "/api/status", 2000)
    local data = json.parse(res)
    if data and data.data and data.data.state == "Tracking" then
        log.write("GOTO complete")
        break
    end
    log.write("Slewing... " .. (data and data.data and data.data.state or "?"))
end
```

---

## 4. Send raw LX200 commands

```lua
local BRIDGE = "http://192.168.3.7"

-- Get Right Ascension
local res = http.get(BRIDGE .. "/api/command?cmd=GR", 2000)
local data = json.parse(res)
-- data.data contains a string like "12:30:45#"
log.write("RA raw: " .. (data and data.data or "error"))

-- Manually nudge north, then stop
http.get(BRIDGE .. "/api/command?cmd=Mn", 1000)
hw.delay(500)
http.get(BRIDGE .. "/api/command?cmd=Q", 1000)

-- Set slew speed to maximum
http.get(BRIDGE .. "/api/command?cmd=RS", 1000)
```

---

## 5. Emergency stop

```lua
local BRIDGE = "http://192.168.3.7"

http.get(BRIDGE .. "/api/stop", 2000)
log.write("Stopped")
```

---

## 6. Discover and control a network Alpaca device

Controls an external Alpaca device (another mount, etc.) on the network.

```lua
local BRIDGE = "http://192.168.3.7"

-- Discover devices (takes ~3 seconds)
local res = http.get(BRIDGE .. "/api/external/discover", 5000)
local data = json.parse(res)

if not data or data.status ~= "success" then
    log.write("Discovery failed")
    return
end

local devices = data.devices
if #devices == 0 then
    log.write("No devices found")
    return
end

local target_ip = devices[1].ip
log.write("Found: " .. target_ip)

-- Get RA from the external telescope (Alpaca returns decimal)
local ra_res = http.get(BRIDGE .. "/api/external/alpaca?target=" .. target_ip .. "&device=telescope&property=rightascension", 2000)
local ra_data = json.parse(ra_res)
if ra_data and ra_data.data then
    log.write("External RA: " .. tostring(ra_data.data.Value))
end
```

---

## 7. GOTO on an external Alpaca telescope

Alpaca GOTO requires **decimal coordinates** (not HMS/DMS strings).

```lua
local BRIDGE = "http://192.168.3.7"
local TARGET = "192.168.3.120"   -- external telescope IP

-- M31 coordinates (decimal): RA=0.7122 hours, DEC=41.2692 degrees
local ra  = "0.7122"
local dec = "41.2692"

-- URL-encode: RightAscension=0.7122&Declination=41.2692
local params = "RightAscension%3D" .. ra .. "%26Declination%3D" .. dec
local url = BRIDGE .. "/api/external/alpaca?target=" .. TARGET ..
            "&device=telescope&property=slewtocoordinatesasync&method=PUT&params=" .. params

local res = http.post(url, "", 5000)
log.write("GOTO: " .. (res or "error"))

-- Wait for slew to complete
for i = 1, 30 do
    hw.delay(2000)
    local s_res = http.get(BRIDGE .. "/api/external/alpaca?target=" .. TARGET .. "&device=telescope&property=slewing", 2000)
    local s_data = json.parse(s_res)
    if s_data and s_data.data and s_data.data.Value == false then
        log.write("GOTO complete")
        break
    end
end
```

**HMS/DMS to decimal conversion**:
```lua
-- Example: "12:30:45" → 12.5125
local function hms_to_decimal(hms)
    local h, m, s = hms:match("(%d+):(%d+):(%d+)")
    return tonumber(h) + tonumber(m)/60 + tonumber(s)/3600
end
```

---

## 8. Control relay switches

```lua
local BRIDGE = "http://192.168.3.7"

-- Turn Switch 0 (GPIO27) ON
local url = BRIDGE .. "/api/v1/switch/0/setswitch?Id=0&State=true"
http.get(url, 2000)

-- Turn Switch 1 OFF
local url2 = BRIDGE .. "/api/v1/switch/0/setswitch?Id=1&State=false"
http.get(url2, 2000)

-- Read current state of Switch 0
local status_res = http.get(BRIDGE .. "/api/v1/switch/0/getswitch?Id=0", 2000)
local status_data = json.parse(status_res)
if status_data then
    log.write("Switch 0: " .. tostring(status_data.Value))
end
```

---

## 9. Read weather data (DHT22)

```lua
local BRIDGE = "http://192.168.3.7"

local temp_res = http.get(BRIDGE .. "/api/v1/observingconditions/0/temperature", 2000)
local hum_res  = http.get(BRIDGE .. "/api/v1/observingconditions/0/humidity",    2000)

local temp_data = json.parse(temp_res)
local hum_data  = json.parse(hum_res)

if temp_data and temp_data.Value ~= -999 then
    log.write("Temp: " .. tostring(temp_data.Value) .. " C")
    log.write("Humidity: " .. tostring(hum_data.Value) .. " %")
else
    log.write("Sensor not connected")
end
```

---

## 10. Generate and serve a dashboard UI (v7.15)

LUNA generates HTML, uploads it to AiBridge, and then listens for button events from the browser.

```lua
local BRIDGE = "http://192.168.3.7"

-- Generate HTML
local html = [[<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>Telescope Control</title></head>
<body>
<h2>Telescope Control</h2>
<button onclick="fetch('/api/ui/action?action=goto_m31',{method:'POST'})">Go to M31</button>
<button onclick="fetch('/api/ui/action?action=stop',{method:'POST'})">Stop</button>
</body>
</html>]]

-- Upload to AiBridge SPIFFS
local res = http.post(BRIDGE .. "/api/ui/upload?filename=mycontrol.html", html, 5000)
log.write("Upload: " .. (res or "error"))

-- Now accessible at http://192.168.3.7/mycontrol.html

-- Poll for button events
while true do
    local action_res = http.get(BRIDGE .. "/api/ui/action", 1000)
    local action_data = json.parse(action_res)

    if action_data and action_data.pending then
        local action = action_data.action
        log.write("Button: " .. action)

        if action == "goto_m31" then
            http.get(BRIDGE .. "/api/goto?ra=00:42:44&dec=+41:16:09", 5000)
        elseif action == "stop" then
            http.get(BRIDGE .. "/api/stop", 2000)
        end
    end

    hw.delay(500)
end
```

---

## Error Handling Pattern

```lua
local function safe_get(url, timeout)
    local res = http.get(url, timeout or 2000)
    if not res or res == "" then
        log.write("HTTP error: " .. url)
        return nil
    end
    local data = json.parse(res)
    if not data then
        log.write("JSON parse error")
        return nil
    end
    return data
end

-- Usage
local status = safe_get(BRIDGE .. "/api/status")
if status and status.status == "success" then
    log.write("State: " .. status.data.state)
end
```

---

## Notes

- Set `http.get` timeout to 2000ms or higher for OnStep (responses can take up to 100ms)
- Poll status every 3 seconds after sending GOTO
- Alpaca GOTO always requires decimal coordinates — HMS/DMS strings are not accepted
- Sound the buzzer to signal operation completion: `http.get(BRIDGE .. "/api/buzzer", 1000)`

---

*AiBridge v7.15 / LUNA Control Guide — Copyright (c) 2025-2026 Nishioka Sadahiko / MIT License*
