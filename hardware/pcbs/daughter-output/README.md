# Daughter / Output Board (Universal)

**Dimensions:** 80 x 20 mm | **Layers:** 2 | **Orientation:** Vertical, below mother/output board | **Instances:** 5

Simplest board in the system — same PCB used for all 5 instances (2× input daughter, 2× output top, 1× output bottom). Jacks + ESD diodes + connector, no active signal processing.

## Components

- 4× 1/4" TS jacks (Switchcraft 112BPC)
- 4× BAT54 ESD clamp diodes (SOD-323)
- 1× 100nF decoupling cap (0603)
- 1× 6-pin JST-PH connector (to mother/output board above)

## See Also

- [`connections.md`](connections.md) — JST-PH pinout, jack nets
- [`architecture.md`](architecture.md) — design reuse, signal flow, routing summary

## Status

**Routing complete.** DRC 0 errors, 0 unconnected. Gerbers exported.
