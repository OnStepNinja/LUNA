---

# LUNA AI Integration Auto-Setup Tool Instruction Manual 2026-05-25

This tool is an automation script designed to connect "LUNA" — an astronomy control and physical computing platform — with AI agents (Gemini CLI and Claude Desktop) in just one click.

It completely automates the process of manually editing complex configuration files (JSON) and writing paths via the command prompt, which was previously required.

---

## 📦 Included Files

When you extract the distributed ZIP file, you will find the following files. These **must always be kept within the same folder**.

1. `aibridge-connector-v2.exe` (LUNA Connector Core)
2. `setup_luna.bat` (Auto-Setup Batch File)
3. `README.md` (This Instruction Manual)

---

## 📂 1. Folder Installation Location

You can place this tool (the entire extracted folder) **anywhere you like on your computer** (e.g., Desktop, directly under C drive, Documents folder, etc.).

*Note: When you run the setup, Windows automatically detects the current location (path) of this tool and registers it into the AI's configuration files. Therefore, **do not move the folder after the setup is complete**. (If you do move it, simply run the batch file again in its new location to reconfigure it).*

---

## ⚙️ 2. LUNA IP Address Configuration (How to Change)

By default, the LUNA IP address in this tool is set to **`192.168.3.50`**.
To change the IP address to match your local network environment, please follow these steps **before running the tool**:

### 【IP Address Modification Steps】

1. Right-click `setup_luna.bat` and select **"Edit"** (or "Open with" > "Notepad").
2. Once Notepad opens, look toward the bottom of the file to find the following lines (there are 2 occurrences):
```bat
"args": ["192.168.3.50"]

```


3. Change `192.168.3.50` to your LUNA's actual IP address (e.g., `192.168.1.100`).
```bat
"args": ["192.168.1.100"]

```


*Note: Make sure to update the IP address in both locations.*
4. From the Notepad menu, select **"Save"** and close Notepad.

---

## 🚀 3. How to Use (Setup Procedure)

1. Turn on LUNA and ensure it is connected to the same network as your computer.
2. Confirm that the "IP Address Configuration" mentioned above is completed.
3. **Double-click `setup_luna.bat` to run it.**
4. A black screen (Command Prompt) will launch, and the configurations will be written automatically.
5. When the screen displays **`[Success] All setup procedures have been successfully completed!`**, press any key to close the window.

That's it! All configurations are now complete.

---

## 🟢 4. Verifying the Connection

After completing the setup, you can check whether LUNA is properly recognized by each AI tool.

### ■ For Gemini CLI

1. Open the Command Prompt, type `gemini`, and press Enter to launch it.
2. In the chat interface, type `/mcp list` and press Enter.
3. If **`🟢 luna - Ready`** is displayed on the screen, the connection is successful.

### ■ For Claude Desktop

1. Launch (or restart) the Claude Desktop application.
2. Click the **plug/outlet icon (MCP plugin mark)** located at the bottom-right (or bottom-left) of the chat input field.
3. If the **`luna`** features (such as Lua script transmission or status retrieval commands) appear in the list of available tools, the connection is successful.

---

## 🛠️ Troubleshooting

* **Error: "aibridge-connector-v2.exe not found"**
👉 Please make sure that the batch file and the connector executable (`.exe`) are located in the exact same folder. This error occurs if you run the batch file directly from inside the ZIP file without extracting it. Ensure you completely extract the ZIP archive before running.
* **The AI does not recognize LUNA / There is no response**
👉 Double-check if the specified IP address is correct and ensure that both LUNA and your PC are connected to the exact same Wi-Fi/local network.

---
