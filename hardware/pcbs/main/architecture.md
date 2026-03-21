# Main Board — Architecture

---

## Power

Soft-latch circuit, load switch, rail distribution (5V_DIG / 5V_ISO), protection, and power sequencing are documented in **[Power System](../../docs/power.md)**.

---

## TCA9548A I2C Mux

- Address **0x70** on main I2C bus (Wire, pins 18/19)
- Isolates each Input Mother Board onto its own channel:
  - **Ch 0** → FFC to Board 1-top (U1 @ 0x10, U2 @ 0x11)
  - **Ch 1** → FFC to Board 2-top (U3 @ 0x10, U4 @ 0x11)
- Resolves AK4619VN 2-address limitation
- MCP23017 (0x20, Keys4x4 PCB) sits upstream — always accessible
- External pull-ups: 4.7k ohm to 3.3V upstream of mux

---

## Galvanic Isolation (Digital ↔ Analog Domain Boundary)

The FFC cables to the Input Mother Boards are the **galvanic isolation boundary**. No shared copper exists between the digital domain (Main Board) and the analog domain (Mother Boards, Daughter/Output Boards, PHONEE). All isolation components sit on the Main Board side, near each FFC connector.

### Per-FFC Isolation Set (×2 — one per TDM bus)

| Component | Function | Key Spec |
|-----------|----------|----------|
| **Murata MEJ2S0505SC** | Isolated DC-DC 5V→5V_ISO | 2W (400 mA), 5.2 kV isolation, SIP-7 TH |
| **Si8662BB-B-IS1** | 6-channel digital isolator (TDM) | 150 Mbps, 4 forward + 2 reverse, SOIC-16W |
| **ISO1541DR** | Isolated bidirectional I2C | 1 MHz, SOIC-8 |

### Si8662BB Channel Assignment

| Channel | Direction | Signal |
|---------|-----------|--------|
| A (fwd) | Main→Analog | MCLK |
| B (fwd) | Main→Analog | BCLK |
| C (fwd) | Main→Analog | LRCLK |
| D (fwd) | Main→Analog | TDM TX_DATA (Teensy→codec SDIN) |
| E (rev) | Analog→Main | TDM RX_DATA0 (codec 1 SDOUT) |
| F (rev) | Analog→Main | TDM RX_DATA1 (codec 2 SDOUT) |

### Timing

BCLK = 24.576 MHz → 40.7 ns period, 20.3 ns half-period. Si8662BB propagation = ~7.5 ns typical. Round-trip (BCLK out + SDOUT back) = ~15 ns + codec internal delay (~10 ns) = ~25 ns. Valid within the 40.7 ns BCLK period. The BB (high-speed) grade is mandatory.

### ISO1541 I2C Isolation

Bidirectional I2C isolator on SDA/SCL lines. Connects between TCA9548A mux output (per-channel) and the FFC. Supports 1 MHz — adequate for AK4619VN and MCP23008 I2C traffic.

### MEJ2S0505SC Isolated Power

5V system rail → 5V_ISO (isolated output). 400 mA capacity per converter. Board 1-top fully populated draws ~200 mA (2× codecs 40 mA + 12× op-amps 60 mA + ADP7118 + MCP23008 + 4× TS5A3159 + HP amp ~50 mA) — provides 2× margin.

### GND_ISO

GND_ISO exists on the Main Board only as small copper islands around isolator Side 2 output pins and FFC pads. **≥1 mm clearance** between GND and GND_ISO copper on all layers. Keep MEJ2S0505SC (switching DC-DC) physically separated from Si8662BB signal pins to minimize switching noise coupling.


---

## ESP32-S3 Reflash Circuit

Teensy pins 9 and 10 provide hardware control over the ESP32-S3 display module's boot mode, enabling firmware updates from SD card without user intervention.

### Circuit

| Teensy Pin | Signal | Connection | Notes |
|-----------|--------|------------|-------|
| 9 | ESP32_EN | Module EN pin | Active-high enable (module has internal pull-up). Teensy drives LOW to hold reset, releases (HIGH-Z or HIGH) to boot. |
| 10 | ESP32_GPIO0 | Module GPIO0/BOOT pin | Drive LOW before EN release → UART bootloader mode. Leave HIGH/float → normal app boot. |

### Reflash Sequence (triggered by SD card update)

1. Teensy asserts ESP32_GPIO0 LOW (pin 10)
2. Teensy pulses ESP32_EN LOW for ~100 ms, then releases (pin 9)
3. ESP32-S3 boots into UART download mode
4. Teensy streams `display.bin` via Serial1 at 921600 baud using `esp-serial-flasher` library
5. Flash time: ~16 seconds for 1.5 MB binary
6. Teensy releases ESP32_GPIO0 (HIGH-Z), pulses ESP32_EN to reboot into normal app mode
7. Normal display engine handshake resumes

### Header Change

Display header expanded from 4-pin to 6-pin JST-PH to carry ESP32_EN and ESP32_GPIO0 signals. See [connections.md](connections.md#esp32-s3-display-header-6-pin).

---

## TS5A3159 Pop Suppression — Moved to Input Mother Board

The 4× TS5A3159 analog mute switches have been **moved to Board 1-top** (Input Mother Board, TDM1) to keep the entire analog output signal path within the isolated domain. They are controlled via a **MCP23008 I2C GPIO expander** (address 0x21) on Board 1-top, accessed through the already-isolated I2C bus. Board 2-top (input-only) does not populate the MCP23008/TS5A3159 section.

**Freed Teensy GPIO pins:** 30, 35, 37, 38, 39 (5 pins returned to spare pool — 4 mute + 1 headphone detect via MCP23008).

See [Input Mother Board architecture](../input-mother/architecture.md) for TS5A3159 and MCP23008 details.

---

## Teensy Pin Assignments

### Peripheral pins (fixed)

- **SAI1 (TDM1):** pins 7, 8, 20, 21, 23, 32
- **SAI2 (TDM2):** pins 2, 3, 4, 5, 33 (bottom), 34 (bottom)
- **SPI0:** pins 11, 12, 13 *(spare — XMOS removed)*; pin 10 reassigned to ESP32_GPIO0
- **I2C (Wire):** pins 18, 19
- **Serial3 RX (MIDI IN):** pin 15
- **Serial4 TX (MIDI OUT):** pin 17
- **SDIO (SD card):** bottom pads 42–47
- **USB Host:** bottom pads (D+, D-, VBUS, GND)
- **Ethernet:** bottom pads (TX+/-, RX+/-, LED, GND)

### GPIO assignments

| Pin | Function |
|-----|----------|
| 0, 1 | Serial1 — ESP32-S3 display UART |
| 6 | NeoPixel data out |
| 9 | ESP32_EN (display reset) |
| 10 | ESP32_GPIO0 (display boot mode) |
| 14 | *(spare — was RA8875 RESET)* |
| 16 | *(spare — was Encoder 3 Edit push, moved to DESPEE)* |
| 22 | MCP23017 INT |
| 24, 25, 28 | *(spare — was Encoder 1 NavX A/B/push, moved to DESPEE)* |
| 26, 27 | *(spare — was Encoder 3 Edit A/B, moved to DESPEE)* |
| 29, 31, 36 | *(spare — was Encoder 2 NavY A/B/push, moved to DESPEE)* |
| 30 | *(spare — was TS5A3159 mute, moved to isolated domain)* |
| 35 | *(spare — was TS5A3159 mute AUX1, bottom pad)* |
| 37 | *(spare — was TS5A3159 mute AUX2)* |
| 38 | *(spare — was TS5A3159 mute AUX3)* |
| 39 | *(spare — was headphone detect, now via MCP23008 on Board 1-top)* |
| 40 | Power button sense |
| 41 | KEEP_ALIVE |

**Total edge pins:** 42. **Consumed by peripherals:** 18 (SAI ×12, I2C ×2, Serial1/ESP32 ×2, Serial3 RX ×1, Serial4 TX ×1). **GPIO:** 6 used (4 original + 2 ESP32 boot control), 18 spare (pins 11, 12, 13, 14, 16, 24, 25, 26, 27, 28, 29, 30, 31, 35, 36, 37, 38, 39). Encoders moved to DESPEE display module — ESP32-S3 reads them locally via GPIO.

---

[Back to README](README.md)
