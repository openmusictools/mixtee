# Input Mother Board — Connections

*← Back to [README](README.md)*

This document defines all connector interfaces on the Input Mother Board, including pinouts, directions, and mating connectors.

------

## FFC to Main Board (16-pin 1.0mm ZIF)

| FFC Pin | Signal | Direction |
|---------|--------|-----------|
| 1 | MCLK | In (from Teensy) |
| 2 | BCLK | In |
| 3 | LRCLK | In |
| 4 | TDM DATA IN (codec 1 SDOUT → Teensy) | Out |
| 5 | TDM DATA OUT (Teensy → codec SDIN1) | In |
| 6 | I2C SDA | Bidirectional |
| 7 | I2C SCL | In |
| 8 | 5V_DIG | Power in |
| 9 | 5V (raw, LDO input) | Power in |
| 10–12 | GND | Ground |
| 13 | TDM DATA IN (codec 2 SDOUT → Teensy) | Out |
| 14–16 | (spare) | — |

Same pinout for both TDM1 and TDM2 instances, substituting bus signals.

**Connector:** 16-pin 1.0mm pitch FFC, ZIF socket (Molex 5025861690). Cable length ~40–50mm.

------

## Mother ↔ Daughter Board (6-pin JST-PH)

| Pin | Signal | Notes |
|-----|--------|-------|
| 1–4 | AIN1–4 (analog R channels) | From daughter board jacks |
| 5 | +5V_A | Analog power to daughter |
| 6 | GND | Ground |

**Connector:** 6-pin JST-PH wire harness (B6B-PH-K-S), 2.0mm pitch, ~15–20mm cable.

------

## Output Analog Cable (10-pin JST-PH, Board 1-top only)

| Pin | Signal |
|-----|--------|
| 1 | Master L (line level) |
| 2 | AUX1 L |
| 3 | AUX2 L |
| 4 | AUX3 L |
| 5 | Master R (from O-bot via O-top) |
| 6 | AUX1 R |
| 7 | AUX2 R |
| 8 | AUX3 R |
| 9–10 | GND |

**Connector:** 10-pin JST-PH (B10B-PH-K-S), 2.0mm pitch, ~80mm cable. Board 1-top only — carries post-driver line-level signals to Output Board O-top.

------

## 1/4" TS Jacks (panel-mount)

4x Switchcraft 112BPC on each instance. Carry L channels (odd: 1,3,5,7 on Board 1-top; 9,11,13,15 on Board 2-top).
