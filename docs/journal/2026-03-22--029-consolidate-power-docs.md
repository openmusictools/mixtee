# 029 — Consolidate power documentation into docs/power.md

**Date:** 2026-03-22
**Scope:** `docs/power.md`, `docs/hardware.md`, `hardware/pcbs/main/architecture.md`, `hardware/pcbs/power/`, `docs/system-topology.md`, `CLAUDE.md`

## Summary

Consolidated all power-related documentation from 6+ locations into a single `docs/power.md`. Deleted the `hardware/pcbs/power/` directory (4 files) since the power board is an off-the-shelf STUSB4500 breakout with no custom PCB. Also fixed two open issues: reconciled power budget math (openaudiotools/about#18) and resolved polyfuse board location conflict (openaudiotools/about#17).

## What changed

| Change | Detail |
|--------|--------|
| Created `docs/power.md` | Consolidated power reference: USB PD input, power path diagram, budget table, rail distribution, protection, soft-latch circuit, power button wiring |
| Added "Why 5V?" info block | GitHub `> [!NOTE]` admonition explaining why higher PD voltage offers no benefit |
| Slimmed `docs/hardware.md` | Replaced full Power System section (~45 lines) with 2-line summary + link to `docs/power.md` |
| Slimmed `main/architecture.md` | Removed soft-latch, power distribution, protection, and power sequencing sections; replaced with link to `docs/power.md` |
| Deleted `hardware/pcbs/power/` | Removed README.md, architecture.md, connections.md, CLAUDE.md (content absorbed into `docs/power.md`) |
| Updated `system-topology.md` | Power Module row and connector summary now link to `docs/power.md` |
| Updated `CLAUDE.md` | Board table, documentation loading guide, and cross-reference table updated for new power doc location |
| Fixed power budget (issue #18) | Worst-case: 2.67A → 2.91A; reserve: 3.20A → 3.49A; target specs: 3.4A → 2.91A |
| Fixed polyfuse location (issue #17) | Documented on Main Board (matches BOM); was incorrectly listed as Power Board in `main/architecture.md` |

## Context

Power documentation was scattered across `hardware/pcbs/power/` (4 lightweight files for an off-the-shelf module), `docs/hardware.md` (budget, distribution, input), and `hardware/pcbs/main/architecture.md` (soft-latch, TPS22965, protection, sequencing). This made it hard to get a complete picture without reading 3+ files. Since the power board has no custom PCB, a dedicated board directory was unnecessary. The new `docs/power.md` is the single canonical reference for all power system documentation.
