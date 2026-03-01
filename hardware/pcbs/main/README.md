# Main Board

**Dimensions:** ~260 x 85 mm (matches top panel)
**Layers:** 4
**Orientation:** Horizontal, under top panel
**Instances:** 1

## Key ICs

- Teensy 4.1 on socket headers (+ PSRAM)
- TCA9548A I2C mux (0x70) — isolates codec buses via FFC
- FE1.1s USB 2.0 hub + 12 MHz crystal
- TPA6132A2 headphone amp (ground-referenced stereo)
- 4x TS5A3159 analog mute switches (pop suppression)
- 74LVC1G00 NAND soft-latch power circuit
- ADP7118 LDO (HP amp + virtual ground)
- 6N138 optocoupler (MIDI IN)
- OPA1678 (2.5V virtual ground buffer)

## Connectors

| Connector | Type | Pins | Destination |
|-----------|------|------|-------------|
| FFC x2 | ZIF 1.0mm | 16 | Input Mother Boards |
| JST-PH | 2.0mm | 6 | Key PCB |
| Header/ribbon | per module | 8-10 | RA8875 TFT display |
| USB-C (PWR) | panel-mount | — | Back panel cutout |
| USB-C (PC) | panel-mount | — | Top panel |
| USB-A dual | panel-mount | — | Top panel (MIDI host) |
| 3.5mm TRS x2 | panel-mount | — | MIDI IN/OUT |
| 1/4" TRS | panel-mount | — | Headphone out |

## Panel-mount components

Encoders (NavX, NavY, Edit), volume pot, SD card socket, and all connectors above protrude through top panel cutouts. PWR USB-C protrudes through back panel.

## Status

Not started. Waiting on finalized pinout and input mother board validation.
