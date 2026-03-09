# 019 — DESPEE naming across docs

**Date:** 2026-03-10
**Scope:** Documentation naming cleanup

## What changed

Named DESPEE explicitly everywhere the custom ESP32-S3 display PCB is referenced. Previously, docs described it generically as "ESP32-S3 custom display PCB" or "ESP32-S3 integrated display module" without naming the standalone project.

## Files modified

| File | Change |
|------|--------|
| `hardware/bom.csv` | Display entry → "DESPEE display module", Custom PCB, updated price to $15–20 |
| `docs/features.md` | Added DESPEE name and link |
| `docs/firmware.md` | Added DESPEE reference for display offloading |
| `docs/hardware.md` | Power budget label → "DESPEE display module" |
| `docs/display/rationale.md` | Added DESPEE naming at Recommended Hardware heading |
| `docs/ui-architecture.md` | Added DESPEE name/link to display engine description |
| `docs/sd-update.md` | "ESP32-S3 display engine firmware" → "DESPEE display engine firmware" |
| `hardware/pcbs/main/connections.md` | Section heading → "DESPEE Display Header" |
| `CLAUDE.md` | Updated Architecture at a Glance, Key ICs, Key Libraries |

## Context

DESPEE (openaudiotools/despee) is the standalone display module project — same hardware/firmware across all Open Audio Tools devices. This change is part of a cross-repo effort to establish DESPEE as the standard display component (MIXTEE, SYNTEE, ABOUT repos updated together).
