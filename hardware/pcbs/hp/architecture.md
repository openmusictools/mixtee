# PHONEE — Architecture

*← Back to [README](README.md)*

------

## Headphone Amplifier

- **TPA6132A2** (TI, SOT-23-8) — ground-referenced stereo headphone driver
- Capless output — no AC coupling caps needed on jack
- 25 mW into 32 Ω, 0.01% THD+N typical
- Single 3.3–5V supply (from 5V_ISO via input cable)
- Output through PCB-mount 10kΩ log pot (volume) to headphone TRS jack

### BOM (amp section)

| Part | Quantity | Package | Notes |
|------|----------|---------|-------|
| TPA6132A2 | 1 | SOT-23-8 | ~$1.50 |
| 10kΩ log pot (A10K) | 1 | PCB-mount | Volume control |
| 1/4" TRS jack (w/ detect) | 1 | Panel-mount TH | Switchcraft 35RASMT2BHNTRX or equiv |
| 0.1µF ceramic cap | 2 | 0402/0603 | Decoupling |
| 1µF ceramic cap | 2 | 0402/0603 | Input coupling |
| 10µF ceramic cap | 1 | 0805 | Bulk decoupling |

------

## Power

- **Input:** 5V_ISO via 4-pin JST-PH cable pin 3
- **Ground:** GND_ISO — entire board on isolated ground, no connection to system GND
- **Consumption:** ~30 mA (TPA6132A2 + minimal passives)
- Decoupling: 10 µF + 0.1 µF ceramic near IC power input

------

## Headphone Detect

The TRS jack includes a detect switch (normally closed to GND_ISO, open when plug inserted). In MIXTEE, the detect signal is routed back to Board 1-top where it connects to **MCP23008 GP6** (I2C GPIO expander at address 0x21). Teensy reads the detect state via I2C polling (~100 Hz).

This eliminates the need for a dedicated Teensy GPIO pin (pin 39 freed) and avoids routing a detect signal across the isolation barrier. Other devices using PHONEE can route the detect signal as needed.

------

## Design Notes

- 2-layer PCB, minimal routing
- Entire ground plane is GND_ISO (in MIXTEE; other devices may use system GND)
- No high-speed digital signals on this board
- Mounting: panel-mount via headphone jack nut + pot nut (both protrude through enclosure panel)
- Keep analog signal traces short — TPA6132A2 placed near input connector

------

[Back to README](README.md)
