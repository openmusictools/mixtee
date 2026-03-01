# Main Board

**Dimensions:** ~260 x 85 mm (matches top panel)
**Layers:** 4
**Orientation:** Horizontal, under top panel
**Instances:** 1

## Key ICs

- Teensy 4.1 on socket headers (+ PSRAM)
- TCA9548A I2C mux (0x70) — isolates codec buses via FFC
- 4x TS5A3159 analog mute switches (pop suppression)
- 74LVC1G00 NAND soft-latch power circuit
- ADP7118 LDO (virtual ground buffer)
- OPA1678 (2.5V virtual ground buffer)

## Connectors

| Connector | Type | Pins | Destination |
|-----------|------|------|-------------|
| FFC x2 | ZIF 1.0mm | 16 | Input Mother Boards |
| FFC x1 | ZIF 1.0mm | 12 | IO Board |
| JST-PH | 2.0mm | 6 | Key PCB |
| JST-PH | 2.0mm | 2 | Power Board (5V + GND in) |
| Header/ribbon | per module | 8-10 | RA8875 TFT display |
| USB-C (PC) | panel-mount | — | Top panel (data only) |

## Panel-mount components

Encoders (NavX, NavY, Edit), SD card socket, and PC USB-C protrude through top panel cutouts. Power arrives from the back-panel Power Board via 2-pin cable.

## Status

Not started. Waiting on finalized pinout and input mother board validation.
