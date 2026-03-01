# MIXTEE: PCB Architecture

*← Back to [README](../README.md) | See also: [Hardware](hardware.md) · [Enclosure](enclosure.md)*

------

## Overview

MIXTEE uses a modular multi-board design. The main board sits horizontally under the top panel. Six back-panel boards (three mother+daughter pairs) carry all audio I/O. A key PCB handles the illuminated switch grid.

Total: **5 unique PCB designs**, 9 physical boards.

------

## Board Definitions

### Main Board

- **Dimensions:** ~260 × 85 mm (matches top panel)
- **Orientation:** Horizontal, under top panel
- **Instances:** 1

**Components:**

- Teensy 4.1 on socket headers (+ PSRAM soldered to Teensy underside)
- TCA9548A I2C mux (address 0x70) — isolates codec I2C buses per Input Mother Board via FFC
- Power management: TPS22965 load switch, polyfuse, ferrite beads, ADP7118 analog LDO (Main Board instance — HP amp + virtual ground)
- PWR USB-C receptacle (routes to back panel cutout, power only)
- PC USB-C receptacle (top panel, data only)
- FE1.1s USB 2.0 hub IC (1 upstream to Teensy USB host, 2 downstream) + 12 MHz crystal
- Dual stacked USB-A receptacle (MIDI host, top panel)
- MIDI IN 3.5mm TRS Type A jack + 6N138 optocoupler circuit (top panel)
- MIDI OUT 3.5mm TRS Type A jack + series/source resistors (top panel)
- Power button (momentary, top panel)
- 3× rotary encoder footprints (top panel, through-hole; NavX + NavY + Edit)
- TFT display connector (to RA8875 module via ribbon/header)
- Full-size SD card socket (Molex 472192001, top panel, SDIO routed from Teensy bottom pads 42–47; built-in micro-SD slot unused)
- Volume pot 10 kΩ log (top panel)
- TPA6132A2 headphone amplifier IC (ground-referenced stereo output)
- Headphone 1/4" TRS jack + volume pot → TPA6132A2 (top panel)
- 4× TS5A3159 analog switch ICs (pop suppression, SOT-23-5, GPIO-controlled)
- Soft-latch power circuit (74LVC1G00 NAND gate SR latch, SOT-23-5, + RC timeout)
- 2.5V virtual ground buffer (1× OPA1678 section + precision resistor divider)
- Bulk caps, decoupling, test points

**Panel-mount components** protrude through top panel cutouts. PWR USB-C protrudes through a back panel cutout.

### Input Mother Board

- **Dimensions:** ~80 × 30 mm
- **Orientation:** Vertical, mounted to back panel
- **Instances:** 2 (Board 1-top for channels 1–8, Board 2-top for channels 9–16)

**Components:**

- ADP7118 LDO (5V → 3.3V_A, per-board analog power supply)
- 4× 1/4" TS jacks (L channels — odd numbered: 1,3,5,7 or 9,11,13,15)
- 2× AK4619VN codec (4-in/4-out each, 8 ADC channels total per board)
- 8× input buffer op-amps (OPA1678)
- 8× 2nd-order Sallen-Key anti-alias filters
- 8× ESD clamp diodes
- Decoupling caps (100nF per IC + bulk)
- Vertical board-to-board connector (to daughter board below)
- FFC connector (to main board)

**Board 1-top** also carries 4× output reconstruction filters + line drivers for the 4 L output channels (Master L, AUX1-3 L). DAC outputs from U1+U2 feed directly into the output analog stages on the same board, then line-level signals route to Board O-top via cable.

**Board 2-top** is ADC-only (U3+U4 DAC sections unused). Same PCB design as Board 1-top — output analog section left unpopulated or omitted via jumper/BOM variant.

**Design note:** Both instances can share one PCB design. I2C address selection via solder jumpers. TDM bus routed to the same FFC pinout — the main board maps them to SAI1 or SAI2. Output analog section populated only on Board 1-top.

### Input Daughter Board

- **Dimensions:** ~80 × 20 mm
- **Orientation:** Vertical, below mother board
- **Instances:** 2 (Board 1-bot for channels 2,4,6,8; Board 2-bot for channels 10,12,14,16)

**Components:**

- 4× 1/4" TS jacks (R channels — even numbered: 2,4,6,8 or 10,12,14,16)
- 4× ESD clamp diodes
- Vertical board-to-board connector (mates with mother board above)

Analog signals route up through the board-to-board connector to the codec on the mother board. No active components besides ESD protection.

### Output Board

- **Dimensions:** ~80 × 20 mm
- **Orientation:** Vertical, mounted to back panel
- **Instances:** 2 (Board O-top for L outputs, Board O-bot for R outputs)

**Components (O-top):**

- 4× 1/4" TS jacks (L outputs: Master, AUX1, AUX2, AUX3)
- 4× ESD clamp diodes
- Vertical board-to-board connector (to O-bot below)
- Cable connector (receives line-level signals from Board 1-top)

**Components (O-bot):**

- 4× 1/4" TS jacks (R outputs)
- 4× ESD clamp diodes
- Vertical board-to-board connector (mates with O-top)

Both output boards are passive — jacks + ESD only. The output analog stages (reconstruction filters, line drivers) live on Board 1-top, and post-driver line-level signals travel via a short cable to O-top. O-bot routes R signals up through the board-to-board connector to O-top, then down the same cable.

**Design note:** O-top, O-bot, and the input daughter boards are all very similar (4× TS jacks + ESD + connector). It may be possible to use the same PCB for all five "dumb" boards if jack spacing and connector placement align. Investigate during schematic phase.

### Key PCB

- **Dimensions:** ~72 × 72 mm (4 columns × 4 rows at 18 mm pitch)
- **Orientation:** Horizontal, mounted through top panel
- **Instances:** 1

**Components:**

- 16× Kailh CHOC hotswap sockets
- 16× WS2812B-2020 NeoPixels (daisy-chained, single data pin)
- 16× 100nF ceramic decoupling caps
- MCP23017 I2C GPIO expander (address 0x20) — handles 4×4 key scan matrix
- 16× 1N4148 signal diodes (anti-ghosting, cathode toward row)
- 6-pin JST-PH connector to main board (NeoPixel DIN, I2C SDA, SCL, MCP23017 INT, 5V, GND)

------

## Board Summary

| ID | Name | Unique design | Instances | Layers | Active components |
|----|------|--------------|-----------|--------|-------------------|
| M | Main Board | Yes | 1 | 4 | Teensy, power mgmt, FE1.1s hub, TPA6132A2 HP amp, TS5A3159 mute, 6N138 MIDI, 74LVC1G00 soft-latch, ADP7118 LDO |
| 1-top | Input Mother (TDM1) | Shared w/ 2-top | 1 | 4 | ADP7118 LDO, 2× AK4619VN, 8× input analog, 4× output analog |
| 2-top | Input Mother (TDM2) | Shared w/ 1-top | 1 | 4 | ADP7118 LDO, 2× AK4619VN, 8× input analog |
| 1-bot | Input Daughter (TDM1) | Shared w/ all daughters | 1 | 2 | ESD diodes only |
| 2-bot | Input Daughter (TDM2) | Shared | 1 | 2 | ESD diodes only |
| O-top | Output Top | Shared (or near-identical) | 1 | 2 | ESD diodes only |
| O-bot | Output Bottom | Shared | 1 | 2 | ESD diodes only |
| K | Key PCB | Yes | 1 | 2 | 16× NeoPixel, 16× CHOC socket, MCP23017 |

**Unique PCB designs:** 5 (Main, Input Mother, Daughter/Output, Key, plus possibly a separate Output Top if connector differs)

------

## Interconnects

### Board-to-Board (vertical, mother ↔ daughter)

Used between each mother+daughter pair. Short vertical connection (~15 mm standoff matching the spacing between top and bottom jack rows).

| Link | Signals | Pin count |
|------|---------|-----------|
| 1-top ↔ 1-bot | 4× analog (R input ch 2,4,6,8) + 5V_A + GND | 6 |
| 2-top ↔ 2-bot | 4× analog (R input ch 10,12,14,16) + 5V_A + GND | 6 |
| O-top ↔ O-bot | 4× analog (R output) + GND | 5 |

**Connector:** 6-pin JST-PH wire harness (2.0 mm pitch). Flexible cable accommodates both boards hanging off the same back panel jacks — no rigid alignment required. Cheap, polarized, pre-crimped harnesses readily available. Both boards are mechanically supported by their panel-mount jack nuts, so no structural rigidity needed from the connector.

### Main Board ↔ Input Mother Boards (FFC cable)

| Link | Signal | Pin |
|------|--------|-----|
| Main ↔ 1-top | MCLK | 1 |
| | TDM1 BCLK | 2 |
| | TDM1 LRCLK | 3 |
| | TDM DATA IN — Codec 1 SDOUT (codec → Teensy RX_DATA0) | 4 |
| | TDM DATA OUT (Teensy → codec SDIN1) | 5 |
| | I2C SDA | 6 |
| | I2C SCL | 7 |
| | 5V_DIG | 8 |
| | 5V (raw, LDO input on codec board) | 9 |
| | GND | 10–12 |
| | TDM DATA IN — Codec 2 SDOUT (codec → Teensy RX_DATA1) | 13 |
| | (spare) | 14–16 |

**Same pinout for Main ↔ 2-top**, substituting TDM2 signals. MCLK shared (active on both cables from Teensy). Each codec gets its own SDOUT line to a separate SAI RX data pin, avoiding TDM bus contention.

**Connector:** 16-pin 1.0 mm pitch FFC, ZIF socket on each board. Cable length ~50–60 mm (top panel to back panel within the 100 mm enclosure depth). Spare pins available for codec reset, interrupt, or future signals.

**Why FFC:** Flat, low-profile, consistent impedance for TDM clocks at 24.576 MHz, easy to route inside a compact enclosure. No crimping tools needed — pre-made cables available in standard lengths.

### Board 1-top → Board O-top (output analog cable)

| Signal | Pin |
|--------|-----|
| Master L (line level) | 1 |
| AUX1 L | 2 |
| AUX2 L | 3 |
| AUX3 L | 4 |
| Master R (from O-bot via O-top) | 5 |
| AUX1 R | 6 |
| AUX2 R | 7 |
| AUX3 R | 8 |
| GND | 9–10 |

**Connector:** 10-pin JST-PH (2.0 mm pitch) or 10-pin 1.0 mm FFC. Short cable (~80 mm, spanning from the input section to the output section along the back panel). Line-level signals are low impedance post-driver, so connector quality is not critical.

**Alternative:** If Board O-top and Board 1-top are adjacent on the back panel, a short rigid board-to-board link could replace the cable entirely.

### Main Board ↔ Key PCB

| Pin | Signal | Notes |
|-----|--------|-------|
| 1 | NeoPixel DIN | Data line from Teensy pin 6 |
| 2 | I2C SDA | Shared Wire bus (Teensy pin 18) |
| 3 | I2C SCL | Shared Wire bus (Teensy pin 19) |
| 4 | MCP23017 INT | Optional interrupt to Teensy (pin 22) |
| 5 | 5V | NeoPixel + MCP23017 power |
| 6 | GND | Common ground |

**Connector:** 6-pin JST-PH (2.0 mm pitch), ~30–40 mm cable. The MCP23017 on the Key PCB handles the 4×4 key scan matrix over I2C, eliminating the need for individual switch GPIO lines.

### Main Board ↔ Display

| Signal | Pin |
|--------|-----|
| SPI MOSI | 1 |
| SPI MISO | 2 |
| SPI SCK | 3 |
| CS | 4 |
| INT | 5 |
| RESET | 6 |
| 3.3V | 7 |
| GND | 8 |
| (backlight PWM, optional) | 9 |

**Connector:** Match whatever header the RA8875 TFT module ships with. Typically a 2×5 or 1×10 pin header, or an FPC ribbon. The display sits flat against the top panel — a short (~20 mm) ribbon or header-to-header cable connects it to the main board below.

------

## Connector Summary

| Connection | Type | Pitch | Pins | Cable length |
|------------|------|-------|------|-------------|
| Mother ↔ Daughter (×3) | JST-PH wire harness | 2.0 mm | 6 | ~15–20 mm |
| Main ↔ Input Mother (×2) | FFC + ZIF | 1.0 mm | 16 | ~40–50 mm |
| 1-top → O-top | JST-PH or FFC | 2.0 / 1.0 mm | 10 | ~80 mm |
| Main ↔ Key PCB | JST-PH | 2.0 mm | 6 | ~30–40 mm |
| Main ↔ Display | Per module | Varies | 8–10 | ~20 mm |

------

## Back Panel Board Arrangement

Looking at the back panel (260 mm wide × 50 mm tall):

```
┌─────────────────────────────────────────────────────────────────────┐
│  O-top   │         1-top          │         2-top          │ [PWR] │
│  4 jacks │        4 jacks         │        4 jacks         │ USB-C │
│(Mst,A1-3)│    (L in 1,3,5,7)      │   (L in 9,11,13,15)   │       │
├──────────┼────────────────────────┼────────────────────────┤       │
│  O-bot   │         1-bot          │         2-bot          │       │
│  4 jacks │        4 jacks         │        4 jacks         │       │
│ (R outs) │    (R in 2,4,6,8)      │   (R in 10,12,14,16)  │       │
└─────────────────────────────────────────────────────────────────────┘
              ← outputs                    inputs →
```

PWR USB-C mounts on the main board and protrudes through a cutout on the far right of the back panel, above the jack boards. The three mother+daughter pairs tile across the remaining width.

------

## Design Reuse Opportunities

1. **Input mother boards (1-top, 2-top):** Same PCB. Codec I2C addresses set by solder jumpers. Output analog section on Board 1-top populated; on Board 2-top left empty.
2. **All daughter/output boards (1-bot, 2-bot, O-top, O-bot):** Potentially same PCB if connector placement and jack spacing match. All are 4× TS jacks + ESD + one connector. Worth investigating during schematic phase — could reduce unique designs from 5 to 3 (Main, Input Mother, Universal Daughter, Key).
3. **Key PCB** is standalone with no reuse opportunities.

------

## Mechanical Mounting

- **Main Board:** Screwed to top panel via standoffs (M3 or M2.5). Board hangs below top panel, components protrude through panel cutouts.
- **I/O Boards (Mother, Daughter, Output):** Mechanically held by panel-mount jack nuts — the 1/4" TS jacks thread through back panel holes and their nuts clamp the boards to the panel. No additional standoffs needed.
- **Key PCB:** Mounted to top panel via standoffs or snap-fit clips. CHOC switches protrude through top panel cutouts, keycaps sit flush with panel surface.
