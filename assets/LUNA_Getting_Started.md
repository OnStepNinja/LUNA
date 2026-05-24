# LUNA — Getting Started

> **No coding. No manuals. Just talk to AI.**

---

> ## 📌 Before you read anything
>
> This repository contains many technical documents marked with `[AI]`.  
> **These are written for AI assistants and specialist developers — not for general users.**  
> You do not need to open, read, or understand any of them.
>
> **Wondering what LUNA can do, or want to try something specific?**  
> Just ask NotebookLM — it has read all the documentation and will explain everything in plain language.  
> 👉 https://notebooklm.google.com/notebook/f2ce997f-18c2-44c4-8738-973b204c190c
>
> *This guide is the only document a general user needs to read.*

---

## What is LUNA?

LUNA is an ESP32-based device that connects AI assistants (Claude, Gemini) to real-world hardware — telescopes, sensors, lab instruments, and more.

You describe what you want in plain language. The AI does the rest.

---

## What can LUNA do?

### Astronomy
- "Point the telescope to M42"
- "Take 10 exposures of 30 seconds each with NINA"
- "Show the current telescope position in Stellarium"
- "Connect my Vixen mount via WiFi and get the current RA/Dec"
- "Set up a SkySafari connection"

### Wireless Sensor Nodes (LUNA Node — BLE)
- "Read the temperature and humidity from the remote sensor node"
- "Blink the Node's LED green when temperature exceeds 30°C"
- "Get the latest log from the outdoor sensor"
- "Deploy a new monitoring script to the Node"

### Any RS-232C Device
- Connect any device that has a serial port
- Hand the device manual to Claude — it learns instantly
- CNC machines, lab instruments, AV equipment, PLCs, power supplies...

### Autonomous Operation
- LUNA can run Lua scripts on its own — no AI needed after setup
- Scheduled measurements, automated logging, real-time monitoring

---

## AI Assistants that work with LUNA

| AI | Connection | Cost |
|----|-----------|------|
| **Claude Desktop** | MCP (recommended) | Free plan available |
| **Gemini CLI** | MCP | Very large free tier |

Both work the same way — connect LUNA, then just talk.

---

## Getting Started — 3 Steps

### Step 1 — Flash the firmware
Follow **LUNA_Firmware_Flashing_Guide.md** to upload LUNA firmware to your ESP32.  
*(One-time setup. Takes about 10 minutes.)*

### Step 2 — Connect your AI assistant

Recommended AI assistants are **Claude Desktop** and **Gemini CLI** — both can be installed on your PC.  
**Antigravity** and **Cursor** have also been confirmed to work with LUNA.  
The connector file (`aibridge-connector-v2.exe`) is required — place it anywhere on your PC and note the full path.

> 📦 **Connector (free download):** https://onstepninja.booth.pm/items/8237213

---

**The easiest way to complete this step: ask the AI itself.**

If you are unsure how to edit config files or set up the connection,  
just tell Claude or Gemini your LUNA's IP address and ask for help:

> *"Help me connect LUNA (MCP server). The IP address is 192.168.3.xxx and the connector is at C:\LUNA\connector\aibridge-connector-v2.exe"*

The AI will walk you through the entire setup — or even do it automatically.

---

**Claude Desktop** — requires editing `claude_desktop_config.json` with your LUNA IP.  
Ask Claude for step-by-step instructions if needed.

**Gemini CLI** — place the connector file, then tell Gemini your IP address.  
Gemini can automatically register the MCP server and complete the connection setup on its own.  
*(Confirmed working: Gemini configured the connection automatically when given the IP address.)*

> 💡 **Not sure which AI to start with?**  
> Gemini CLI has a very large free tier — great for trying LUNA at no cost.  
> Claude Desktop offers the most natural conversation experience.

### Step 3 — Talk to your AI
```
"Tell me the status of the LUNA device."
"What can you do?"
```

That's it. The AI will guide you from here.

---

## Need help?

📚 LUNA Docs / AiBridgeMCP — Start here for setup guides, technical references, and FAQ.
Ask questions directly inside NotebookLM and get answers from the official documentation.
👉 https://notebooklm.google.com/notebook/f2ce997f-18c2-44c4-8738-973b204c190c

💬 AiBridgeMCP Community — Join our Facebook group to ask questions, share your results, and connect with other users. The developer is also a member. Everyone is welcome.
👉 https://www.facebook.com/groups/1230935959149731

---

## About the technical documents　 (`[AI]` prefix)

**You do not need to read them.**

They exist for the AI — Claude and Gemini read them automatically to understand  
LUNA's full capabilities. Developers and specialists can refer to them directly.

---

*LUNA — solo indie project by Nishioka Sadahiko*  