# Input Mother Board

**Dimensions:** ~80 x 40 mm | **Layers:** 4 | **Orientation:** Vertical, mounted to back panel | **Instances:** 2

Board 1-top handles channels 1–8 (TDM1), Board 2-top handles channels 9–16 (TDM2). Same PCB design for both — I2C address selection via solder jumpers. Board 1-top also carries 4× output reconstruction filters + line drivers; Board 2-top leaves output section unpopulated.

## Key ICs

- 2× AK4619VN codec (4-in/4-out each, 8 ADC channels per board)
- 8× OPA1678 input buffer op-amps + 8× Sallen-Key anti-alias filters
- 8× ESD clamp diodes
- ADP7118 LDO (5V → 3.3V_A, per-board analog supply)

## See Also

- [`connections.md`](connections.md) — FFC, JST, jack pinouts (interface contract)
- [`architecture.md`](architecture.md) — analog stage design, LDO, I2C addressing, BOM variant
- [`ak4619-wiring.md`](ak4619-wiring.md) — AK4619VN codec reference (pin table, registers, TDM)

## Status

Routed via FreeRouting (4 unrouted, 5 DRC unconnected — plane fills resolve most). Gerbers exported.
