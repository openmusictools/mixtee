# 028 — PHONEE and DESPEE listed as external modules in BOM

**Date:** 2026-03-22
**Scope:** `hardware/bom.csv`

## Summary

Replaced individual PHONEE component lines (TRS jack, pot, TPA6132A2) and the DESPEE encoder stub with single module-level entries in the MIXTEE BOM. Each module's own repo now owns its component-level BOM. Schottky diode count dropped from 25 to 24 (headphone jack ESD moved to PHONEE BOM). Fabrication line updated to exclude PHONEE PCB.

## What changed

| Change | Detail |
|--------|--------|
| PHONEE in bom.csv | 3 component lines → 1 module line (`openaudiotools/phonee`) |
| DESPEE in bom.csv | Merged display line + removed stale encoder qty-0 line → 1 module line (`openaudiotools/despee`) |
| Schottky diodes | 25 → 24 (HP jack ESD is PHONEE's responsibility) |
| Fabrication line | Removed PHONEE (2L) from PCB fab list; noted as external alongside DESPEE and power module |

## Context

PHONEE and DESPEE are reusable modules with their own repositories. Listing their internal components in MIXTEE's BOM created dual-maintenance and drift risk. MIXTEE's BOM now treats them the same way it treats the off-the-shelf STUSB4500 power module — as a purchased/assembled unit with a cost estimate and a pointer to the canonical source.
