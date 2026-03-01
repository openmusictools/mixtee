# Key PCB

**Dimensions:** ~72 x 72 mm (4x4 grid at 18mm pitch)
**Layers:** 2
**Orientation:** Horizontal, mounted through top panel
**Instances:** 1

## Components

- 16x Kailh CHOC hotswap sockets
- 16x WS2812B-2020 NeoPixels (daisy-chained)
- 16x 100nF ceramic decoupling caps
- 16x 1N4148 signal diodes (anti-ghosting, cathode toward row)
- 1x MCP23017 I2C GPIO expander (0x20, handles 4x4 key scan matrix)
- 1x 6-pin JST-PH connector to main board

## Connector pinout (to Main Board)

| Pin | Signal |
|-----|--------|
| 1 | NeoPixel DIN (Teensy pin 6) |
| 2 | I2C SDA (Teensy pin 18) |
| 3 | I2C SCL (Teensy pin 19) |
| 4 | MCP23017 INT (Teensy pin 22) |
| 5 | 5V |
| 6 | GND |

## Generator

`gen_key_pcb.py` — generates schematic and PCB (68 components).

## Status

Schematic and PCB generated. Needs routing and DRC validation.
