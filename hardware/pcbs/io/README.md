# IO Board

**Dimensions:** ~50 x 80 mm
**Layers:** 2
**Orientation:** Horizontal, under top panel (left side)
**Instances:** 1

## Key ICs

- FE1.1s USB 2.0 hub (SSOP-28) + 12 MHz crystal + 2x 15pF load caps
- 2x TPS2051 USB host power switches (SOT-23-5)
- 6N138 optocoupler (DIP-8) — MIDI IN galvanic isolation

## Connectors

| Connector | Type | Pins | Destination |
|-----------|------|------|-------------|
| FFC x1 | ZIF 1.0mm | 12 | Main Board (Ethernet + USB + MIDI + power) |
| 6-pin header | 2.54mm | 6 | Ethernet ribbon cable from Main Board |
| 4-pin header | 2.54mm | 4 | HP amp breakout module (HP_L, HP_R, 5V_A, GND) |
| USB-A dual stacked | Amphenol 67298-4090 | — | MIDI HOST (2x USB-A, top panel) |
| RJ45 MagJack | Through-hole | — | Ethernet (top panel) |
| 3.5mm TRS x2 | CUI SJ-3523-SMT | — | MIDI IN + MIDI OUT (top panel) |
| Volume pot | 10kΩ log, through-hole | 3 | Headphone volume (top panel) |

## Panel-mount components (left column, top to bottom)

1. MIDI HOST — dual USB-A (stacked)
2. ETH — RJ45 MagJack
3. MIDI IN — 3.5mm TRS Type A
4. MIDI OUT — 3.5mm TRS Type A
5. Headphone output (from nearby HP amp breakout module)
6. PHONES label + VOL pot

## Functional zones

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

## Ethernet section

- RJ45 MagJack with integrated magnetics (through-hole, panel-mount)
- 0.1µF coupling cap between Teensy PHY signals and MagJack transformer
- Signals arrive via 6-pin ribbon cable from Main Board (ETH TX+, TX-, RX+, RX-, LED, GND)
- Post-PHY analog — no impedance-controlled routing required

## Design rules

Per `docs/pcb-design-rules.md`:
- 2-layer stackup: L1 signal + components, L2 GND + some signal
- Default trace width: 0.25mm, Power: 0.5mm, USB: 0.2mm
- Clearance: 0.15mm minimum
- Via: 0.3mm drill signal, 0.4mm drill power
- GND zone on B.Cu
- 1mm corner radius on board outline

## Status

Stage 1 spec complete. Proceeding to SKiDL netlist (Stage 2).
