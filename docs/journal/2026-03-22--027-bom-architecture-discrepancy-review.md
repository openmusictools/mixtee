# 027 — BOM architecture discrepancy review

**Date:** 2026-03-22
**Scope:** `hardware/bom.csv`, `docs/hardware.md`, board architecture docs

## Summary

Cross-referenced `hardware/bom.csv`, the markdown BOM tables in `docs/hardware.md`, and all per-board `architecture.md` files. Found 10 discrepancies — fixed 5 in code, deferred 2 to GitHub issues, resolved 3 by removing redundant tables. Key design change: moved the 2.5V virtual ground buffer from the Main Board to the Input Mother Boards, since it can't cross the galvanic isolation boundary.

## What changed

| Change | Files |
|--------|-------|
| Move virtual ground buffer to Input Mother Boards; remove Main Board ADP7118 LDO | `hardware/bom.csv`, `docs/hardware.md`, `hardware/pcbs/main/architecture.md`, `hardware/pcbs/main/README.md`, `hardware/pcbs/input-mother/architecture.md`, `hardware/pcbs/input-mother/ak4619-wiring.md`, `docs/system-topology.md`, `docs/architecture-diagram.md` |
| Correct OPA1678 count: 17/16 → 26 (Board 1-top: 17, Board 2-top: 9) | `hardware/bom.csv`, `docs/hardware.md` |
| ADP7118 count: 3 → 2; precision resistors: 2 → 4 | `hardware/bom.csv`, `docs/hardware.md` |
| Add missing MIDI IN jack | `docs/hardware.md` |
| Remove Ethernet coupling caps (MagJack has integrated magnetics) | `hardware/bom.csv`, `docs/hardware.md`, `hardware/pcbs/io/architecture.md`, `hardware/pcbs/io/connections.md` |
| Split Power & Connectivity into separate sections | `docs/hardware.md` |
| Replace duplicate BOM tables with pointer to `hardware/bom.csv` | `docs/hardware.md` |
| List PHONEE and DESPEE as external modules, not individual components | `hardware/bom.csv` |
| Schottky diodes 25 → 24 (HP jack ESD now in PHONEE BOM) | `hardware/bom.csv` |

## Deferred

- **Polyfuse location** (Main Board vs Power Board): [openaudiotools/about#17](https://github.com/openaudiotools/about/issues/17)
- **Power budget math** (line items don't sum to stated totals): [openaudiotools/about#18](https://github.com/openaudiotools/about/issues/18)

## Context

The virtual ground buffer was documented on the Main Board so it could serve both Input Mother Boards from one location. However, the mother boards are in the galvanically isolated analog domain — no analog signal can cross the FFC isolation boundary. Each mother board now generates its own 2.5V reference locally using a precision resistor divider + one OPA1678 section, powered by its existing ADP7118 LDO. This eliminated the Main Board's ADP7118 instance entirely.

The duplicate BOM tables in `docs/hardware.md` were a recurring source of drift (counts, categories, missing items). Replaced with a single pointer to `hardware/bom.csv` as the canonical source, with per-board details in each board's `architecture.md`.
