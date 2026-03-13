# Architecture Diagram Review & Corrections

**Date:** 2026-03-06
**Scope:** `docs/architecture-diagram.md`
**Source docs cross-referenced:** All board `connections.md` files, `main/connections.md` (XMOS section)

## Correctness Fixes

| # | Issue | Before | After |
|---|-------|--------|-------|
| 1 | HP Board connection source | `MCP23008 -- "HP audio" --> HP_BOARD` | `TS5A -- "4-pin JST-PH" --> HP_BOARD` |
| 2 | TS5A3159 missing as node | Audio routed through MCP23008 (I2C expander) | Separate `TS5A` node; MCP23008 sends GPIO control, TS5A carries audio |
| 3 | 10-pin output cable unlabeled | `MCP23008 -- "unmuted out" --> D_OUT` | `TS5A -- "10-pin JST-PH\n8ch post-mute" --> D_OUT` |
| 4 | Key PCB cable split into two lines | Separate `I2C 0x20` and `NeoPixel pin 6` lines | Single `6-pin JST-PH\nI2C + NeoPixel + INT + 5V` line |
| 5 | IO Board cable topology | Individual signal lines (ETH, USB, MIDI) direct to components | `12-pin FFC` to IO subgraph + separate `6-pin ribbon` for ETH diff pairs |
| 6 | HP detect return missing | No return path shown | `HP_BOARD -. "HP detect\nvia 4-pin cable" .-> MCP23008` |

## Completeness Improvements

| # | Addition |
|---|---------|
| 7 | MCP23008 address label: `0x21` (was embedded in mute text) |
| 8 | Board 2-top labeled `(input only, output DNP)` |
| 9 | TCA9548A channel labels: Ch 0 / Ch 1 (already present, kept) |
| 10 | TDM tap label: `9-signal TDM tap\n(pre-isolator, high-Z)` |
| 11 | XMOS return audio: `Return audio (8ch)\nTBD: pin 9 SAI1_RX_DATA2?` |
| 12 | BCLK frequency on TDM lines: `BCLK 24.576 MHz` |
| 13 | D_IN/D_OUT clarified: `Input Daughters` / `Output Boards` |

## Legend Update

- Dotted line meaning expanded: "Power distribution, passive connections, detect signals"
- MCP23008 GPIO mute control now uses dotted line (control, not audio)
