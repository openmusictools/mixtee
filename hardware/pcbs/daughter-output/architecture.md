# Daughter / Output Board — Architecture

## Design Reuse

This is the simplest board in the system. The same PCB design is used for all 5 instances:
- 2× Input Daughter (1-bot, 2-bot): carry R input channels from jacks to mother board
- 2× Output (O-top, O-bot): carry output channels
- Jacks + ESD diodes + connector — no active signal processing

## Signal Flow

- **Input instances**: jack tip → BAT54 ESD diode to rails → JST pin → mother board input buffer
- **Output O-top**: receives line-level signals from Board 1-top via 10-pin cable, routes to jacks. Also receives R signals from O-bot via JST.
- **Output O-bot**: routes R output signals up through JST to O-top

## Routing Summary

- Autorouted via FreeRouting (Specctra DSN/SES round-trip)
- 44 trace segments, 2 layers
- AIN1–4: 0.3mm traces (Audio_Analog net class), F.Cu + B.Cu with vias
- +5VA: 0.5mm traces (Power net class), F.Cu
- GND: B.Cu ground zone, F.Cu stubs to vias for SMD pads
- DRC: 0 errors, 0 unconnected. 26 cosmetic warnings (silkscreen overlap).

---
Back to [README](README.md)
