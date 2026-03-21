# 026 — PHONEE integration guide simplified

**Date:** 2026-03-21
**Scope:** `d:\projects\openaudiotools\phonee\docs\integration-guide.md`

## Summary

Simplified the PHONEE integration guide from a tutorial-style document with three grounding strategy subsections and three detect routing options down to a concise reference. The guide now describes what the connector exposes, what the board expects, and recommends galvanic isolation since PHONEE is an analog signal output stage where ground noise is directly audible. Also removed the `hardware/` directory from the phonee repo (no PCB files yet — docs only for now).

## What changed

| File | Change |
|------|--------|
| `phonee/docs/integration-guide.md` | Replaced ~150-line tutorial with ~60-line reference: input connector table, detect switch behavior, isolation recommendation, mechanical notes, MIXTEE example |
| `phonee/hardware/` | Removed (premature — no KiCad files yet; BOM table already in `docs/architecture.md`) |
| `phonee/README.md` | Removed BOM link (pointed to deleted `hardware/bom.csv`) |

## Context

The original guide enumerated grounding strategies (shared, star, isolated) and detect routing options (GPIO, I2C expander, ignore) as if teaching PCB design. The user's feedback was clear: just describe the interface and recommend isolation. Host-specific integration details belong in the host project's docs (e.g., MIXTEE's `hardware/pcbs/hp/`), not in PHONEE's canonical repo.
