# 023 — Rename Key PCB to Keys4x4

**Date:** 2026-03-21
**Scope:** `hardware/pcbs/key/` → `hardware/pcbs/keys4x4/`, all cross-referencing docs

## Summary

Renamed the Key PCB board directory and all references across the codebase from "Key PCB" / `pcbs/key/` to "Keys4x4 PCB" / `pcbs/keys4x4/`. The rename avoids ambiguity between the board name and generic uses of "key" (e.g., key switches, keycaps). Journal entries were left unchanged as historical records.

## What changed

| File(s) | Change |
|---------|--------|
| `hardware/pcbs/key/` → `hardware/pcbs/keys4x4/` | Directory renamed via `git mv` |
| `hardware/pcbs/keys4x4/{README,architecture,connections,CLAUDE}.md` | Board title updated from "Key PCB" to "Keys4x4 PCB" |
| `hardware/pcbs/keys4x4/designs/gen_pcb.py` | Docstring and silk screen text updated |
| `CLAUDE.md` | Board table path, architecture summary, cross-ref table |
| `README.md` | Repo structure tree |
| `docs/system-topology.md` | Board summary table, connector summary, reuse section, mounting section |
| `docs/pin-mapping.md` | I2C section, GPIO key matrix section, architecture link, FFC link |
| `docs/architecture-diagram.md` | Mermaid subgraph label and comment |
| `docs/enclosure.md` | Section heading and scanning description |
| `docs/hardware.md` | Stackup table, noise mitigation, BOM tables (MCP23017, 1N4148) |
| `docs/features.md` | Key scanning reference |
| `docs/pcb-design-rules.md` | 2-layer stackup list, digital domain grounding |
| `docs/connector-parts.md` | MPN table (JST-PH, CHOC), cable table |
| `docs/pcbs-workflow.md` | PDF example path |
| `hardware/bom.csv` | MCP23017 and diode notes, fabrication line |
| `hardware/pcbs/main/{connections,architecture,CLAUDE}.md` | JST-PH section, I2C mux note, do-not-read path |
| `hardware/pcbs/{daughter-output,io,hp,input-mother,power}/CLAUDE.md` | Do-not-read path references |

## Context

The name "Key PCB" was ambiguous — "key" could refer to switches, keycaps, or the board itself. "Keys4x4" explicitly describes the 4×4 key matrix grid, making it immediately clear what the board does. This is a naming-only change with no functional impact.
