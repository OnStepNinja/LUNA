--------------------------------------------------------------------------------
LUNA Firmware Flashing Guide
Target Versions: v5.x.x (ESP32) / v6.x.x (ESP32-S3) Last Updated: 2026-04-21 [1]
--------------------------------------------------------------------------------
Purchase a Pre-flashed Board (Recommended)
For a plug-and-play experience, you can purchase the pre-flashed LUNA OnStepNinja V2 board. [1] 🛒 BOOTH: https://onstepninja.booth.pm/ If you have purchased a pre-flashed board, this flashing procedure is not required. Please proceed directly to 01_QuickStart.md. [1]
--------------------------------------------------------------------------------
Manual Flashing Procedure
1. Download Firmware
Download the firmware for your specific hardware from the GitHub release page: [1, 2] 📦 GitHub Releases: https://github.com/OnStepNinja/LUNA/releases
2. Included Files
The downloaded package typically contains: [3]
LUNA_vX.x.x.bootloader.bin
LUNA_vX.x.x.partitions.bin
LUNA_vX.x.x.bin
flash_command.txt — Contains the pre-verified flashing command.
--------------------------------------------------------------------------------
Step 1: Install Python and esptool
Download and install Python from Python.org.
CRITICAL: Ensure "Add Python to PATH" is checked during installation. [3]
Install esptool by running the following in your terminal: pip install esptool
Step 2: Connect ESP32 to PC
Connect your board using a USB cable or RS-232C cable (depending on your hardware). [3]
Identify the COM port: [3]
Right-click the Start button → Device Manager → "Ports (COM & LPT)".
Step 3: Enter Flash Mode (If required)
If the flashing process does not start automatically: [4]
Press and hold both BOOT and RESET buttons.
Release the RESET button first, then release the BOOT button.
Step 4: Execute Flashing Command
Open the folder containing the firmware files.
Shift + Right-click in the folder and select "Open PowerShell window here". [4]
Open flash_command.txt, replace COM3 with your actual port number, then copy and paste the command into PowerShell. [4]
Press Enter to execute.
Once successful, press the RESET button once to boot the firmware. [4]
--------------------------------------------------------------------------------
Step 5: Initial WiFi Configuration
Upon the first boot, LUNA starts in AP Mode. [5, 6]
Open WiFi settings on your PC or smartphone.
Connect to the network named LUNA_XXXX (where XXXX is a unique ID). [5, 7]
Open a browser and navigate to http://192.168.4.1. [5, 7]
Open the File Manager. [5, 7]
IMPORTANT (First time only): Perform Format SPIFFS. This is required for the v5.9.4+ partition scheme. [5-7]
Go to WiFi Configuration, click Load Config, enter your home router's SSID and Password, then click Save Config. [5, 7]
Press the RESET button to reboot. LUNA will now connect to your home network. [5]
Verify the assigned IP address in your router's DHCP list or the LUNA Web Console. [5]
Tip: It is highly recommended to set a DHCP reservation (Static IP) for LUNA in your router settings for stable operation. [5, 8]
After WiFi setup is complete, please proceed to 01_QuickStart.md. [5]
--------------------------------------------------------------------------------
Troubleshooting
Symptom
Solution
'python' not found
Try using python3 command instead. [9]
Connection Failed
Repeat the button sequence in Step 3. Ensure you are not using a "charge-only" USB cable. [9]
LUNA_XXXX not visible
Press the RESET button once to reboot the device. [9]
Cannot connect after WiFi setup
Verify SSID/Password (case-sensitive). Note: 5GHz WiFi is not supported; use 2.4GHz. [9]
--------------------------------------------------------------------------------
LUNA — Bridge Across Decades © 2026 Nishioka Sadahiko [9]