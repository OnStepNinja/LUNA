--------------------------------------------------------------------------------
LUNA Quick Start Guide
Target Versions: v5.9.6 (ESP32) / v6.9.3 (ESP32-S3) Last Updated: 2026-04-21
--------------------------------------------------------------------------------
💡 No manual required. Just connect the device and talk to Claude. Claude will guide you through everything [1].
--------------------------------------------------------------------------------
What You Need
Requirement
Source
Cost
Claude Desktop
https://claude.ai/download
Free
LUNA Firmware
GitHub Releases
Free
Windows PC
Connected to the same Wi-Fi as LUNA
—
💡 Claude Code is not required. You can access all LUNA features using only the free Claude Desktop. If you get stuck with settings or operations, you can consult Claude directly within the app [1, 2].
--------------------------------------------------------------------------------
Devices You Can Control
In principle, any device with an RS-232C port can be controlled [3, 4].
Industrial/Manufacturing: NC machine tools, PLCs, inverters, robot controllers.
Measurement/Research: Multimeters, oscilloscopes, power supplies, spectrum analyzers.
Astronomy (Hardware): Takahashi Temma2, Celestron, and other telescope mounts.
Astronomy (Software): NINA, Stellarium, ASCOM/Alpaca-compatible devices.
Astronomy (INDI): All INDI-compatible devices (v6.9.3 only).
Broadcast/AV: Switchers, VTR controllers.
Simply hand over the device's manual (PDF, etc.) to Claude, and you can start controlling it immediately [3, 5].
--------------------------------------------------------------------------------
Setup (3 Simple Steps)
1. Place the Connector Folder
Download the connector folder from the link below: 📦 BOOTH: https://onstepninja.booth.pm/items/8237213 Extract the ZIP and place the connector folder anywhere on your PC (e.g., C:\LUNA\). Do not move it once placed, as the connection will break if the path changes [6, 7].
2. Check LUNA's IP Address
With LUNA connected to your home router (STA Mode), identify its assigned IP address [6, 8]. How to check:
Check the STA IP in the LUNA Web Console at http://192.168.4.1 (AP Mode) under Network Information [9, 10].
Alternatively, check the DHCP client list in your router's settings [9]. You will use this STA IP address (e.g., 192.168.3.50) in the next step.
3. Configure Claude Desktop
Open the config file: Press Windows Key + R, paste %AppData%\Roaming\Claude and hit Enter. Open claude_desktop_config.json with a text editor [9, 11].
Edit and Save: Copy the following and replace all content in the file.
Change "192.168.3.xxx" to your LUNA's STA IP address [12].
Note: Use double backslashes \\ for the path to aibridge-connector-v2.exe [12].
Restart Claude: Completely exit Claude Desktop (using Task Manager is recommended) and then relaunch it [12, 13].
Verify Connection: Click the "+" (Add Content) icon in the bottom-left of the chat and select "Connectors". If LUNA is 🔵 blue (ON), the connection is successful [13, 14].
--------------------------------------------------------------------------------
Connection Test
Ask Claude:
> "Tell me the status of the LUNA device."
If it returns the IP address, firmware version, and free memory, it is working correctly [13].
--------------------------------------------------------------------------------
Essential Tips for Talking to Claude
📖 get_lua_guide — The First Step
> "Get the lua guide."
LUNA uses Lua scripts for autonomous control. To write accurate scripts, Claude needs to read LUNA's design specifications first. Always run this tool before requesting complex operations [5, 15].
🔭 Vixen Support (v5.9.6 / v6.9.3)
> "Get the vixen guide."
LUNA supports Vixen Wireless Unit and STAR BOOK TEN protocols. You can control Vixen mounts directly without a dedicated controller [16].
📷 NINA Integration (v5.9.6 / v6.9.3)
LUNA can control the NINA astrophotography software via its Advanced API. You can check camera status or start exposures directly through conversation with Claude [17, 18].
📡 Resources — Real-time Status Monitoring
LUNA can publish device states (like telescope position or weather data) as Resources. Claude can "read these gauges" naturally during conversation without manual tool calls [19, 20].
--------------------------------------------------------------------------------
Troubleshooting
Symptom
Solution
LUNA not in Connectors
Check for syntax errors (commas/brackets) in the JSON file [21].
Communication error
Ensure the ESP32 and PC are on the same Wi-Fi network [21].
IP address changes
Set a DHCP reservation (static IP) in your router [9, 22].
--------------------------------------------------------------------------------
LUNA — Bridge Across Decades © 2026 Nishioka Sadahiko [21, 23]