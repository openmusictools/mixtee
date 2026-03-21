# 022 — State synchronization strategy for Teensy↔ESP32

**Date:** 2026-03-13
**Scope:** `despee/docs/protocol.md`, `despee/docs/integration-guide.md`, `docs/display/protocol.md`, `docs/firmware.md`, `docs/ui-architecture.md`

## Summary

Documented the state synchronization strategy between Teensy (source of truth for DSP parameters) and ESP32 (LVGL display engine). Chose a "continuous push + periodic full sync" model that keeps all widget values current regardless of which page is visible, eliminating stale-value problems on page switch.

## What changed

| File | Change |
|------|--------|
| `despee/docs/protocol.md` | Added "State Synchronization Model" section: continuous push, periodic sync, bandwidth budget table, encoder echo rules, PAGE_CHANGED handling |
| `despee/docs/integration-guide.md` | Added "State Synchronization" section with code examples for continuous push, encoder value clamping, and periodic sync_tick() |
| `docs/display/protocol.md` | Mirrored state sync section from DESPEE canonical with Teensy-specific language |
| `docs/firmware.md` | Expanded UI Update Loop with continuous push and periodic sync bullet points |
| `docs/ui-architecture.md` | Added "State replication" row to Rendering Split table |

## Context

The problem: when state changes on the Teensy side (MIDI CC, state restore, etc.) while the ESP32 shows a different page, widget values go stale. Four options were analyzed:

- **A. Pull model** — rejected (adds request-response complexity to fire-and-forget protocol)
- **B. Push-on-navigate** — rejected (visible pop-in, doesn't handle gradual drift)
- **C. Continuous replication** — chosen (simple, zero page-switch latency, trivial bandwidth cost)
- **D. Periodic full sync** — chosen as insurance layer on top of C

Key design decisions:
- Encoder-originated changes are NOT echoed back (ESP32 already shows correct value), unless Teensy clamps/quantizes
- Periodic sync spreads one widget per 50 ms (~20/sec), completing full UI cycle every ~6s
- Total bandwidth: ~5.6% of UART capacity (including meters + sync + sporadic param changes)
- No new protocol commands needed — uses existing SET_VALUE/SET_TEXT/SET_STATE
