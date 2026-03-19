# Takahashi Temma / Temma2 Serial Communication Protocol Specification

**Created**: 2026-03-12
**Author**: Nishioka Sadahiko (AiBridgeMCP Project)
**Target Models**: Temma PC / Temma2 / Temma2 Jr. / EM-10/200/400/500 Temma2 series

---

## Sources and Reliability

| Information | Source | Reliability |
|-------------|--------|-------------|
| Connector spec / cable wiring | Takahashi official cable diagram (ASCOM Temma Driver PDF p.34) | ✅ Primary source |
| Coordinate format (ASCII format) | ASCOM Temma V2 Driver Help — Troubleshooting section | ✅ Primary source |
| Temma and Temma2 use identical protocol | Documented by multiple ASCOM/INDI driver developers | ✅ Multiple primary sources |
| Command set (Q/G/I/T/E) | Prior session research; implemented in v2.1.0 | ⚠️ Not yet tested on actual hardware |
| Baud rate (Temma2 = 19200) | Prior session research | ⚠️ Not yet tested on actual hardware |

> ⚠️ Items marked ⚠️ require final verification with actual hardware.

---

## 1. Communication Specifications

| Item | Temma (original / PC) | Temma2 / Temma2 Jr. |
|------|-----------------------|----------------------|
| Baud Rate | **9600 bps** ⚠️ | **19200 bps** ⚠️ |
| Data Bits | 8 | 8 |
| Parity | None (N) | None (N) |
| Stop Bits | 1 | 1 |
| Electrical Standard | RS-232C (±12V) | RS-232C (±12V) |
| Connector (mount side) | Mini-DIN 4-pin ✅ | Mini-DIN 4-pin ✅ |
| Command Terminator | CR (0x0D) | CR (0x0D) |

---

## 2. Connector and Cable Specifications

### Mini-DIN 4-Pin Assignment (Mount side — male connector, front view)

```
[3] [4]
[1] [2]
```

| Pin | Signal | Direction | Notes |
|-----|--------|-----------|-------|
| 1 | TXD | Mount → PC | Mount transmit |
| 2 | RXD | PC → Mount | Mount receive |
| 3 | GND | — | Ground |
| 4 | Control signal | — | Normally NC |

> **Source**: Takahashi official cable diagram (ASCOM Temma Driver PDF p.34)
> Mini-DIN 4P Male → D-sub 9P Female, cross-wired

### Takahashi OEM Cable Wiring (from ASCOM Driver PDF)

```
Mini-DIN 4P (Male)          D-sub 9P (Female)
     Pin 1  ────────────────  Pin 2  (RXD: PC receive = Mount transmit)
     Pin 2  ────────────────  Pin 3  (TXD: PC transmit = Mount receive)
     Pin 3  ────────────────  Pin 5  (GND)
     Pin 4  ────────────────  Pin 6  (DSR)
                              Pin 7  ─┐
                              Pin 8  ─┘ (RTS-CTS loopback)
```

### ESP32 Connection (AiBridgeMCP v2.1.0 Design)

```
Temma2 Mini-DIN4     MAX3232 Module          ESP32
  Pin 3 (GND) ──────── GND ─────────────── GND
  Pin 1 (TXD) ──────── R1IN → R1OUT ────── GPIO16 (RX2)
  Pin 2 (RXD) ──────── T1IN ← T1OUT ────── GPIO17 (TX2)
  Pin 4 (NC)  ─ Not connected
```

> ⚠️ A MAX3232 or equivalent is required for RS-232C (±12V) ↔ TTL 3.3V conversion.
> ⚠️ Pin numbering may vary depending on how the connector is oriented.
>    Always verify TXD/RXD continuity when making the physical connection.

---

## 3. Coordinate Format

**Source**: ASCOM Temma V2 Driver Help — Troubleshooting section (original text):
> *"The Temma protocol reports coordinates to the nearest second in right ascension (15 arcseconds at 0 degrees declination) and to the nearest 6 arcseconds in declination."*
> *"The Temma's control system accepts declination coordinates to the nearest 12 arcseconds."*

This confirms that coordinates are transmitted in **ASCII decimal format**.

### Right Ascension (RA) Format: `HHMMSSs` (7 digits)

| Field | Digits | Content | Example |
|-------|--------|---------|---------|
| HH | 2 | Hours (00–23) | `10` |
| MM | 2 | Minutes (00–59) | `30` |
| SSs | 3 | Seconds × 10 (000–599) | `456` = 45.6 sec |

Example: `1030456` = 10h 30m 45.6s

### Declination (Dec) Format: `SDDMMSS` (7 digits)

| Field | Digits | Content | Example |
|-------|--------|---------|---------|
| S | 1 | Sign (`+` or `-`) | `-` |
| DD | 2 | Degrees (00–90) | `32` |
| MM | 2 | Minutes (00–59) | `10` |
| SS | 2 | Seconds (00–59) | `30` |

Example: `-321030` = −32° 10′ 30″

---

## 4. Command Reference

> ⚠️ The commands below have been researched and implemented, but require final verification on actual hardware.
> Append **CR (0x0D)** to the end of every command.

### 4.1 Get Current Position

```
Send:    Q\r
Receive: <HHMMSSs><SDDMMSS>  (14 characters + terminator)
```

Example:
```
Send:    Q
Receive: 1030456-321030
         ↑       ↑
         RA      Dec
```

### 4.2 GoTo (Slew to Target)

```
Send:    G<HHMMSSs><SDDMMSS>\r
Receive: >  (ready prompt, or no response)
```

Example:
```
Send: G1030456-321030\r
       = Slew to RA 10h 30m 45.6s, Dec −32° 10′ 30″
```

### 4.3 Initialization Command Sequence

Before issuing the first GoTo after startup, the following initialization sequence is required:

```
1. Set latitude:  I+<DDMMSS>\r  or  I-<DDMMSS>\r
                  Example: I+354600\r = North latitude 35°46'00"

2. Set time:      T<HHMMSS>\r
                  Example: T203045\r = 20:30:45 (JST)

3. Confirm:       E\r
```

> ⚠️ The exact response format for each command must be confirmed with actual hardware.

---

## 5. Operational Notes

### Computer Standby Switch (Important)

The **Computer Standby switch on the Temma2 must be set to ON**; otherwise RS-232C communication is disabled.
If this switch is OFF, the mount will not respond to any commands.

### Power-On Sequence

To avoid garbled characters at startup, the following order is recommended:

```
1. Power on ESP32 (AiBridgeMCP) → wait for WiFi connection to complete
2. Power on Temma2
```

This is a workaround for the issue where the ESP32's startup welcome message is sent to the Temma2.

### Protocol Consistency

Multiple independent driver developers (ASCOM and INDI) have confirmed:
**"The protocol is unchanged between Temma PC and Temma2."**
(Same driver and same cable work for both.)

---

## 6. Accuracy Specifications (Documented Values)

| Item | Accuracy | Notes |
|------|----------|-------|
| RA reporting resolution | 1 second (= 15 arcsec at Dec = 0°) | Temma protocol spec |
| Dec reporting resolution | 6 arcsec | Temma protocol spec |
| Dec command resolution | 12 arcsec | Temma control system spec |

> Source: ASCOM Temma V2 Driver Help p.33

---

## 7. Items to Verify with Actual Hardware

After connecting to actual Temma2 hardware, verify the following in priority order:

| Priority | Item to Verify | How to Verify |
|----------|----------------|---------------|
| **Highest** | Whether `Q\r` receives a response | Send `serial_query("Q\r", 1000)` |
| **Highest** | Response format of Q command | Record the response string |
| **Highest** | Whether communication works at 19200 bps | Confirm connection can be established |
| **High** | Response to `G` command (whether `>` is returned) | GoTo test |
| **High** | Whether GoTo works without initialization | Send G command without I/T/E first |
| Medium | Exact TXD/RXD pin correspondence | Swap TXD↔RXD and test |

---

## 8. Rejected Protocols (Records)

The following protocol specifications were encountered during research but **could not be corroborated and were not adopted**:

- Three-step GoTo via `X<hhhhhhh>` / `Y<hhhhhhh>` / `S`
- 24-bit hexadecimal coordinates (7-digit hex)
- `Z` status (bit flags) / `H` hard stop

None of the above commands appear in any publicly available primary source for ASCOM or INDI drivers, and they have been excluded from this specification.

---

*This specification will be updated after actual hardware testing is completed.*
*Test results from actual hardware will be recorded in `handover_YYYYMMDD.md`.*
