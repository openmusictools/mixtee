# Restructure Docs into Layered Agent Context

Reorganized the flat `docs/` directory into a layered documentation architecture so AI agents load only board-relevant context instead of the entire ~150KB doc set.

## What Changed

### Per-board directories (`hardware/pcbs/{board}/`)

Each board now has:

| File | Purpose |
|------|---------|
| `README.md` | Trimmed to essentials — dimensions, layers, key ICs, status |
| `connections.md` | Interface contract — connector pinouts, signal tables |
| `architecture.md` | Deep implementation — circuits, register configs, analog design |
| `CLAUDE.md` | Agent context guide — what to read, what to skip |
| `designs/` | All KiCad files, scripts, gerbers (moved from board root) |

Boards: input-mother, io, main, key, daughter-output, power.

### Design files moved to `designs/`

For daughter-output, input-mother, io, key: all KiCad files (`*.kicad_pcb`, `*.kicad_pro`, `*.dsn`, `*.ses`, `gen_pcb.py`, `fp-lib-table`, `gerbers/`, etc.) moved into `designs/` subdirectory. Updated `fp-lib-table` and `gen_pcb.py` relative paths (`../../lib/` → `../../../lib/`).

### Shared docs trimmed

| File | Change |
|------|--------|
| `pcb-architecture.md` | Renamed → `system-topology.md`, trimmed from 355→~85 lines |
| `hardware.md` | Removed board-specific sections (analog stages, USB hub, MIDI, soft-latch, detailed power), added redirects. ~422→~287 lines |
| `pin-mapping.md` | Removed FFC pinouts, key matrix wiring, power sequencing, added redirects. ~380→~275 lines |
| `ak4619-wiring.md` | Replaced with 5-line redirect stub → `input-mother/ak4619-wiring.md` |
| `connector-parts.md` | Replaced with compact MPN index table |

### Root files updated

- `CLAUDE.md` — rewritten with new structure, Documentation Loading Guide, updated cross-references
- `README.md` — updated repo structure tree and documentation tables

### Global link audit

All references to `pcb-architecture.md` updated to `system-topology.md`. All `ak4619-wiring.md` links updated to new location. Journal entries left as historical records.

## Context Size Impact

| Agent task | Before | After |
|-----------|--------|-------|
| IO Board PCB work | ~69KB | ~18KB |
| Input Mother PCB work | ~90KB | ~30KB |
| Key PCB work | ~64KB | ~13KB |
| Cross-board interface check | ~20KB | ~2KB |

## Files Created/Modified

- 24 new files across 6 board directories (connections.md, architecture.md, CLAUDE.md, plus ak4619-wiring.md copy for input-mother)
- 4 boards' design files moved to `designs/` subdirectories
- 4 fp-lib-table + 4 gen_pcb.py path fixes
- 6 board READMEs trimmed
- 5 shared docs trimmed/renamed/replaced
- 2 root files (CLAUDE.md, README.md) updated
- Multiple cross-reference link fixes across docs
