# Daughter / Output Board (Universal)

**Dimensions:** 80 x 20 mm
**Layers:** 2 (F.Cu signal + B.Cu GND zone)
**Orientation:** Vertical, below mother/output board
**Instances:** 5 (2x input daughter, 2x output, 1x output bottom)

## Components

- 4x 1/4" TS jacks (Switchcraft 112BPC)
- 4x BAT54 ESD clamp diodes (SOD-323)
- 1x 100nF decoupling cap (0603)
- 1x 6-pin JST-PH connector (to mother board above)

## Nets

| Net | Description |
|-----|-------------|
| AIN1-4 | Analog signals (jack tip -> connector) |
| +5VA | Analog power from mother board |
| GND | Ground (B.Cu zone) |

## Routing

Autorouted via FreeRouting (Specctra DSN/SES round-trip). 44 trace segments, 2 layers.

- **AIN1-4:** 0.3mm traces (Audio_Analog net class), F.Cu + B.Cu with vias
- **+5VA:** 0.5mm traces (Power net class), F.Cu
- **GND:** B.Cu ground zone, F.Cu stubs to vias for SMD pads; TH pads connect through zone directly

## Generator

`gen_daughter_output.py` — generates both schematic and PCB programmatically.

## Status

**Routing complete.** DRC passes with 0 errors, 0 unconnected items. 26 cosmetic warnings (silkscreen overlap/edge, library mismatch from generated footprints). Gerbers exported to `gerbers/`.
