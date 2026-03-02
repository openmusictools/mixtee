# Main Board — Architecture

---

## Soft-Latch Power Circuit

- **74LVC1G00** NAND gate (SOT-23-5) wired as SR latch
- **Power on:** button press sets latch → TPS22965 EN → 5V rail on → Teensy boots
- **Power off (clean):** Teensy detects button (pin 40) → saves state → releases KEEP_ALIVE (pin 41) → latch resets → TPS22965 off
- **Power off (hard):** long-press >4s → RC timeout resets latch directly (hardware failsafe)
- Debounce cap on button input
- Teensy KEEP_ALIVE GPIO holds latch set during normal operation

---

## Power Distribution

### 5V Rail Partitioning

- **5V_DIG:** USB hub VBUS + NeoPixels + TFT backlight (noisy loads)
- **5V_A:** Dedicated low-noise LDO for audio analog stages

### TPS22965 Load Switch

- 5A continuous, soft-start / inrush limiting
- Input from Power Board (post-polyfuse)
- Controlled slew rate limits inrush from bulk cap charging

### Protection

- Input polyfuse (2.5A hold / 5A trip) on Power Board
- Per-port current limiting for USB host (TPS2051 on IO Board)
- Bulk capacitor (1000–2200 uF) near NeoPixel power entry
- 10 mohm shunt resistor test point for current measurement

### ADP7118 LDO (Main Board instance)

- 5V → 3.3V_A for virtual ground buffer
- 5V_A also powers HP amp breakout module via short wire

### 2.5V Virtual Ground

- Precision resistor divider (2x 10k ohm 0.1%) buffered by one OPA1678 section
- Provides stable mid-rail reference for AC-coupled signal paths
- OPA1678 rail-to-rail output gives ~3.5Vpp usable swing

### PC USB-C Ground

- GND connected to system GND through ferrite bead to reduce computer-injected noise

---

## TCA9548A I2C Mux

- Address **0x70** on main I2C bus (Wire, pins 18/19)
- Isolates each Input Mother Board onto its own channel:
  - **Ch 0** → FFC to Board 1-top (U1 @ 0x10, U2 @ 0x11)
  - **Ch 1** → FFC to Board 2-top (U3 @ 0x10, U4 @ 0x11)
- Resolves AK4619VN 2-address limitation
- MCP23017 (0x20, Key PCB) sits upstream — always accessible
- External pull-ups: 4.7k ohm to 3.3V upstream of mux

---

## TS5A3159 Pop Suppression

- 4x analog switch ICs (SOT-23-5), one per output stereo pair
- GPIO-controlled by Teensy: high = unmuted
- Firmware opens switches during boot/shutdown ramp
- Low Ron (~1 ohm), 5V tolerant
- Pin assignments: Main L/R = pin 30, AUX1 = pin 35, AUX2 = pin 37, AUX3 = pin 38

---

## Teensy Pin Assignments

### Peripheral pins (fixed)

- **SAI1 (TDM1):** pins 7, 8, 20, 21, 23, 32
- **SAI2 (TDM2):** pins 2, 3, 4, 5, 33 (bottom), 34 (bottom)
- **SPI0 (display):** pins 10, 11, 12, 13
- **I2C (Wire):** pins 18, 19
- **Serial3 RX (MIDI IN):** pin 15
- **Serial4 TX (MIDI OUT):** pin 17
- **SDIO (SD card):** bottom pads 42–47
- **USB Host:** bottom pads (D+, D-, VBUS, GND)
- **Ethernet:** bottom pads (TX+/-, RX+/-, LED, GND)

### GPIO assignments

| Pin | Function |
|-----|----------|
| 0, 1 | spare (Serial1 debug) |
| 6 | NeoPixel data out |
| 9 | RA8875 INT |
| 14 | RA8875 RESET (reclaimed from Serial3 TX) |
| 16 | Encoder 3 (Edit) push |
| 22 | MCP23017 INT |
| 24, 25, 28 | Encoder 1 (NavX) A, B, push |
| 26, 27 | Encoder 3 (Edit) A, B |
| 29, 31, 36 | Encoder 2 (NavY) A, B, push |
| 30 | TS5A3159 mute Main L/R |
| 35 | TS5A3159 mute AUX1 (bottom pad) |
| 37 | TS5A3159 mute AUX2 |
| 38 | TS5A3159 mute AUX3 |
| 39 | Headphone detect |
| 40 | Power button sense |
| 41 | KEEP_ALIVE |

**Total edge pins:** 42. **Consumed by peripherals:** 20. **GPIO:** 22 used, 0 spare (pins 0/1 reserved debug).

---

## Power Sequencing

```
Button press → SR latch SET → TPS22965 EN → 5V rail on → Teensy boots
                                                            ↓
                                            Teensy asserts KEEP_ALIVE (pin 41)
                                            Teensy reads button via pin 40

Button press → Teensy detects → saves state → releases KEEP_ALIVE
                                                            ↓
                                            SR latch RESET → TPS22965 off
Long-press (>4s) → RC timeout resets latch directly (hardware failsafe)
```

---

[Back to README](README.md)
