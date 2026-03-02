# IO Board

**Dimensions:** ~50 x 80 mm | **Layers:** 2 | **Orientation:** Horizontal, under top panel (left side) | **Instances:** 1

Handles Ethernet, USB MIDI host, MIDI IN/OUT, and provides a mounting area for an off-the-shelf headphone amp breakout module. All panel-mount components in the left column of the top panel.

## Key ICs

- FE1.1s USB 2.0 hub (SSOP-28) + 12 MHz crystal
- 2× TPS2051 USB host power switches (SOT-23-5)
- 6N138 optocoupler (DIP-8) — MIDI IN galvanic isolation

## Functional Zones

```
┌──────────────────────────────────────────────────────────────┐
│  USB-A (panel)   RJ45 (panel)                               │ y=0 (panel edge)
│                                                              │
│  FE1.1s + xtal   TPS2051x2   6N138                         │
│  15pF  15pF       caps         MIDI passives                │
│                                                              │
│  MIDI_IN (panel)  MIDI_OUT (panel)                          │
│                                                              │
│  VOL pot (panel)  HP breakout header                        │
│                                                              │
│  [FFC ZIF 12-pin]  [6-pin ETH header]                      │ y=80 (interior edge)
└──────────────────────────────────────────────────────────────┘
 x=0                                                     x=50
```

## See Also

- [`connections.md`](connections.md) — FFC, Ethernet ribbon, HP header, panel-mount connector pinouts
- [`architecture.md`](architecture.md) — USB hub, MIDI, Ethernet, HP amp circuit details

## Status

Fully routed via FreeRouting. Gerbers exported.
