# Input Mother Board

**Dimensions:** ~80 x 30 mm
**Layers:** 4
**Orientation:** Vertical, mounted to back panel
**Instances:** 2 (Board 1-top channels 1-8, Board 2-top channels 9-16)

## Key ICs

- 2x AK4619VN codec (4-in/4-out each, 8 ADC channels per board)
- 8x OPA1678 input buffer op-amps
- 8x 2nd-order Sallen-Key anti-alias filters
- 8x ESD clamp diodes
- ADP7118 LDO (5V -> 3.3V_A, per-board analog supply)

## Connectors

| Connector | Type | Pins | Destination |
|-----------|------|------|-------------|
| FFC | ZIF 1.0mm | 16 | Main Board |
| JST-PH | 2.0mm | 6 | Daughter Board (below) |
| 1/4" TS x4 | panel-mount | — | L channels (odd: 1,3,5,7 or 9,11,13,15) |

## Design notes

- Same PCB for both instances. I2C address selection via solder jumpers.
- Board 1-top also carries 4x output reconstruction filters + line drivers for Master L, AUX1-3 L. Board 2-top leaves output section unpopulated.
- TDM bus routed to same FFC pinout; main board maps to SAI1 or SAI2.

## Status

Not started.
