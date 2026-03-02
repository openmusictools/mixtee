# Main Board — Connections

Interface contract for the Main Board — the central hub connecting all other boards. Every inter-board cable terminates here. This document defines connector types, pinouts, and cable specs for each interface.

---

## FFC to Input Mother Boards (16-pin 1.0mm ZIF, ×2)

See [`../input-mother/connections.md`](../input-mother/connections.md) for the full 16-pin FFC pinout. Summary:

- Pins 1–3: TDM clocks (MCLK, BCLK, LRCLK)
- Pins 4, 13: TDM data in (one per codec)
- Pin 5: TDM data out
- Pins 6–7: I2C (via TCA9548A mux channels)
- Pin 8: 5V_DIG, Pin 9: 5V raw
- Pins 10–12: GND
- Pins 14–16: spare

**Connector:** Molex 5025861690 (16-pin 1.0mm ZIF).
**Cable:** ~40–50mm FFC.
**Instances:** Two — one per Input Mother Board (TDM1 vs TDM2).

---

## FFC to IO Board (12-pin 1.0mm ZIF)

See [`../io/connections.md`](../io/connections.md) for the full 12-pin FFC pinout. Summary:

- Pins 1–6: Ethernet TX/RX differential pairs + GND guards
- Pins 7–8: USB Host D+/D-
- Pins 9–10: MIDI RX/TX
- Pin 11: 5V_DIG, Pin 12: GND

**Connector:** Molex 502586 series (12-pin 1.0mm ZIF).
**Cable:** ~100–120mm FFC.

---

## JST-PH to Key PCB (6-pin)

| Pin | Signal | Teensy Pin | Notes |
|-----|--------|------------|-------|
| 1 | NeoPixel DIN | 6 | Data line, 300–500 ohm series resistor |
| 2 | I2C SDA | 18 | Shared Wire bus |
| 3 | I2C SCL | 19 | Shared Wire bus |
| 4 | MCP23017 INT | 22 | Optional interrupt |
| 5 | 5V | — | NeoPixel + MCP23017 power |
| 6 | GND | — | Common ground |

**Connector:** 6-pin JST-PH (B6B-PH-K-S), 2.0mm pitch.
**Cable:** ~30–40mm.

---

## JST-PH from Power Board (2-pin)

| Pin | Signal | Notes |
|-----|--------|-------|
| 1 | 5V (VBUS) | From STUSB4500 output, up to 5A |
| 2 | GND | Power return |

**Connector:** 2-pin JST-PH or screw terminal.
**Cable:** ~60–80mm, 22 AWG. Carries full system current.

---

## Display Header (8–10 pin)

| Pin | Signal |
|-----|--------|
| 1 | SPI MOSI |
| 2 | SPI MISO |
| 3 | SPI SCK |
| 4 | CS |
| 5 | INT |
| 6 | RESET |
| 7 | 3.3V |
| 8 | GND |
| 9 | (backlight PWM, optional) |

**Connector:** Per RA8875 TFT module (2x5 or 1x10 pin header).
**Cable:** ~20mm.

---

## Ethernet Ribbon (6-pin header)

6-pin 2.54mm pitch header carries ETH TX+/TX-/RX+/RX-/LED/GND from Teensy bottom pads to IO Board. ~100mm ribbon cable (separate from FFC).

---

## PC USB-C (panel-mount)

USB4105-GF-A (GCT), mid-mount SMD. Data only — D+/D- to Teensy native USB device. Top panel, left zone.

---

## SD Card Socket

Molex 472192001, full-size SD. SDIO from Teensy bottom pads 42–47. Top panel, left of display.

---

[Back to README](README.md)
