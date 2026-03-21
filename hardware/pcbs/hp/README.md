# PHONEE — Headphone Output Module

> **Note:** The canonical PHONEE specification lives in
> [openaudiotools/phonee](https://github.com/openaudiotools/phonee).
> This directory contains MIXTEE-specific integration context.
> Design changes should be made in PHONEE first and synced here.

**Dimensions:** ~30 × 20 mm | **Layers:** 2 | **Orientation:** Horizontal, under top panel (left zone) | **Instances:** 1

PHONEE is a **reusable headphone output module** — a standalone custom PCB designed for use across multiple audio devices (similar to [DESPEE](https://github.com/openaudiotools/despee) for displays). It carries a TPA6132A2 headphone amp, PCB-mount volume pot, and 1/4" TRS jack on a single board.

In MIXTEE, PHONEE sits in the **galvanically isolated analog domain**. It receives Master L/R audio and isolated power from Board 1-top (Input Mother TDM1) via a 4-pin JST-PH cable. Entire board runs on GND_ISO — no connection to system GND.

## Key Components

- TPA6132A2 headphone amplifier IC (SOT-23-8, ground-referenced, capless output)
- 10kΩ log potentiometer — PCB-mount (A10K)
- 1/4" TRS headphone jack (panel-mount, with detect switch)
- 4-pin JST-PH connector (signal + power input)

## Why a Reusable Module

- Keeps headphone output fully in the isolated analog domain without complicating the IO Board with split ground planes
- Single-board solution (amp + pot + jack) that can be dropped into any audio device needing a headphone output
- No off-the-shelf module combines all three components — this fills that gap
- Canonical source: [openaudiotools/phonee](https://github.com/openaudiotools/phonee)

## See Also

- [`connections.md`](connections.md) — connector pinouts (4-pin input cable, headphone jack)
- [`architecture.md`](architecture.md) — amp circuit, power, headphone detect routing
- [`../input-mother/connections.md`](../input-mother/connections.md) — Board 1-top HP cable pinout (source)

## Status

Not started. Design phase — will be developed in [openaudiotools/phonee](https://github.com/openaudiotools/phonee).
