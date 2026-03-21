# 024 — HP Board becomes PHONEE reusable headphone module

**Date:** 2026-03-21
**Scope:** `hardware/pcbs/hp/`, all cross-referencing docs, `hardware/bom.csv`

## Summary

Redesigned the HP Board as **PHONEE** — a reusable headphone output module (like DESPEE for displays). Replaces the off-the-shelf TPA6132/MAX97220 breakout approach with a custom PCB carrying TPA6132A2 IC, PCB-mount 10kΩ log pot, and 1/4" TRS jack on a single board. PHONEE will live in its own repo at [openaudiotools/phonee](https://github.com/openaudiotools/phonee).

## What changed

| File(s) | Change |
|---------|--------|
| `hardware/pcbs/hp/{README,architecture,connections,CLAUDE}.md` | Renamed to PHONEE; replaced breakout module with TPA6132A2 IC; updated pot to PCB-mount; added BOM table |
| `CLAUDE.md` | Updated board table, key hardware ICs, module descriptions |
| `docs/system-topology.md` | Board summary, connector summary, reuse section, mounting section |
| `docs/hardware.md` | Stackup table, noise mitigation, grounding, BOM tables, panel layout |
| `docs/architecture-diagram.md` | Mermaid subgraph label, signal path summary |
| `docs/enclosure.md` | Section heading and module description |
| `docs/connector-parts.md` | Cable table |
| `docs/pcb-design-rules.md` | Analog domain grounding section |
| `hardware/bom.csv` | Replaced breakout module line with TPA6132A2 IC; updated pot and jack notes; updated fabrication line |
| `hardware/pcbs/io/{README,architecture,connections,CLAUDE}.md` | Updated HP Board references to PHONEE |
| `hardware/pcbs/input-mother/{connections,architecture}.md` | Updated HP Board references to PHONEE |
| `hardware/pcbs/main/architecture.md` | Updated HP Board references to PHONEE |

## Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Module vs in-repo | Own project (like DESPEE) | Reusable across devices; same pattern as display module |
| Amp IC | TPA6132A2 | Best balance: capless output, SOT-23-8 (smallest), ~$1.70 total amp BOM, fewest components |
| Volume pot | PCB-mount | Single-board solution; board positions at panel via jack nut + pot nut |
| Name | PHONEE | Follows DESPEE naming convention |

## Context

No off-the-shelf module combines headphone amp + volume pot + 1/4" TRS jack on one small PCB. The previous plan used a breakout module mounted on a carrier board, but this was neither compact nor reusable. PHONEE fills the gap as a drop-in headphone output module for any audio device.
