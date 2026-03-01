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

- AIN1: All F.Cu, 0.3mm traces
- AIN2-4: F.Cu stubs + B.Cu hops (via pairs) to avoid crossing AIN1 vertical
- +5VA: F.Cu 0.5mm power trace
- GND: SMD pads get stub traces to vias connecting to B.Cu ground zone; TH pads connect through zone directly

## Generator

`gen_daughter_output.py` — generates both schematic and PCB programmatically.

## Status

Work in progress. Schematic and footprint placement complete. Trace routing implemented but has 2 remaining DRC shorting errors where AIN1/AIN2 traces pass through jack S pads. See `journal.md` for details.
