# 014 - Mermaid Hardware Architecture Diagram

**Date:** 2026-03-06

## Summary

Created a comprehensive Mermaid flowchart diagram at `docs/architecture-diagram.md` showing the full MIXTEE hardware architecture. Consolidated nodes to reduce horizontal spread.

## What was done

- Single Mermaid `graph TB` diagram covering all 7 board types + off-the-shelf modules
- Two top-level subgraphs: Digital Domain (GND) and Analog Domain (GND_ISO)
- Galvanic isolation boundary clearly shown via red-bordered Si8662BB, ISO1541, MEJ2S0505SC nodes
- Signal paths differentiated by line style: thick (TDM audio), solid (control/data), dotted (power)
- Node color-coding: blue (digital), green (analog), red (isolation), yellow (power)
- Signal path summary section for quick reference

### Node consolidation (width fix)

Merged related items into multi-line nodes to reduce horizontal pressure:

| Area | Before | After |
|------|--------|-------|
| IO Board | 6 nodes (FE1.1s, USB-A, RJ45, 6N138, TPS2051, MIDI jacks) | 3 nodes (USB_HOST, MIDI, RJ45) |
| Key PCB | 3 nodes (MCP23017, NeoPixels, CHOC) | 1 node (KEY_CONTENTS) |
| Daughters | 4 nodes (1-bot, 2-bot, O-top, O-bot) | 2 nodes (D_IN, D_OUT) |
| HP Board | 3 nodes (amp, pot, jack) | 1 node (HP_BOARD) |
| Main Board | 8 nodes + ISO block | 5 nodes + ISO block (SD/LDO/Latch/USB-C merged into MAIN_IO) |
| Mother boards | MCP23008 + TS5A3159 separate | merged into single MCP23008 node |
| Loose nodes | ESP32, encoders, PWR_BTN floating | grouped in PERIPHERALS subgraph |

Total nodes reduced from ~37 to ~24.

## File modified

- `docs/architecture-diagram.md`
