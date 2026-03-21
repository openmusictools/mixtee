# PHONEE — Connections

*← Back to [README](README.md)*

This document defines all connector interfaces on the PHONEE headphone output module.

------

## Input Cable from Board 1-top (4-pin JST-PH)

| Pin | Signal | Notes |
|-----|--------|-------|
| 1 | Master L | Post-TS5A3159 mute, post-line driver (from Board 1-top) |
| 2 | Master R | Post-TS5A3159 mute, post-line driver |
| 3 | 5V_ISO | Isolated power (from Board 1-top) |
| 4 | GND_ISO | Isolated ground |

**Connector:** 4-pin JST-PH (B4B-PH-K-S), 2.0mm pitch. Cable ~40–60mm from Board 1-top.

------

## Headphone Jack (1/4" TRS, panel-mount)

| Contact | Signal | Notes |
|---------|--------|-------|
| Tip | HP Left | Post-volume-pot |
| Ring | HP Right | Post-volume-pot |
| Sleeve | GND_ISO | Isolated ground |
| Detect switch | HP_DETECT | Normally closed to GND_ISO; open when plug inserted |

**Connector:** Switchcraft 35RASMT2BHNTRX or equivalent, 1/4" TRS stereo with detect switch. Panel-mount via jack nut (top panel, left zone).

**Headphone detect routing:** The detect switch signal routes back to Board 1-top via the 4-pin cable (piggybacked on a spare wire or via a 5th pin if the connector is upgraded). Alternatively, the detect signal can be routed through one of the spare FFC pins. Currently planned: detect switch connects to MCP23008 GP6 on Board 1-top — Teensy reads it via I2C polling.

------

## Volume Potentiometer

- 10kΩ log taper (A10K)
- PCB-mount, between TPA6132A2 output and headphone jack
- Protrudes through enclosure panel (in MIXTEE: top panel, left zone, labeled "PHONES" / "VOL")

------

[Back to README](README.md)
