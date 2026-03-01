# MIXTEE: PCB Architecture

*← Back to [README](../README.md) | See also: [Hardware](hardware.md) · [Enclosure](enclosure.md)*

------

## Overview

MIXTEE uses a modular multi-board design. The main board sits horizontally under the top panel. Six back-panel boards (three mother+daughter pairs) carry all audio I/O. A dedicated IO board on the left side of the top panel handles Ethernet, USB MIDI host, MIDI IN/OUT, and provides a mounting area for an off-the-shelf headphone amp breakout module. A key PCB handles the illuminated switch grid. An off-the-shelf STUSB4500 breakout module on the back panel provides USB PD power input.

Total: **6 unique PCB designs**, 10 physical boards + 1 off-the-shelf power module.

------

## Board Definitions

### Main Board

- **Dimensions:** ~260 × 85 mm (matches top panel)
- **Orientation:** Horizontal, under top panel
- **Instances:** 1

**Components:**

- Teensy 4.1 on socket headers (+ PSRAM soldered to Teensy underside)
- TCA9548A I2C mux (address 0x70) — isolates codec I2C buses per Input Mother Board via FFC
- Power management: TPS22965 load switch, polyfuse, ferrite beads, ADP7118 analog LDO (Main Board instance — virtual ground buffer)
- PC USB-C receptacle (top panel, data only — USB Audio + MIDI composite device)
- 3× rotary encoder footprints (top panel, through-hole; NavX + NavY + Edit)
- TFT display connector (to RA8875 module via ribbon/header)
- Full-size SD card socket (Molex 472192001, top panel left zone — left of display, vertically aligned with bottom edge of screen; SDIO routed from Teensy bottom pads 42–47; built-in micro-SD slot unused)
- 4× TS5A3159 analog switch ICs (pop suppression, SOT-23-5, GPIO-controlled)
- Soft-latch power circuit (74LVC1G00 NAND gate SR latch, SOT-23-5, + RC timeout)
- 2.5V virtual ground buffer (1× OPA1678 section + precision resistor divider)
- FFC connector for IO Board (12-pin 1.0 mm ZIF)
- Bulk caps, decoupling, test points

- 6-pin header (Ethernet ribbon cable from Teensy bottom pads to IO Board)

**Panel-mount components** protrude through top panel cutouts. PC USB-C protrudes through a top panel cutout (left zone, near SD card slot). The physical power button has moved to the back panel (next to PWR USB-C); the soft-latch circuit remains on the Main Board.

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

### Power Module (Off-the-Shelf)

- **Type:** STUSB4500 USB PD breakout module (purchased, not custom PCB)
- **Orientation:** Vertical, mounted to back panel
- **Instances:** 1

**Module provides:**

- USB-C receptacle (power only — VBUS + GND, D+/D- not routed)
- STUSB4500 USB PD sink controller (negotiates 5V/5A; fallback 5V/3A via CC resistors)
- On-board decoupling and protection

**Panel-mount:** USB-C receptacle protrudes through a back panel cutout (right side). Labeled "PWR" on back panel. Module secured by USB-C panel-mount nut or adhesive standoff. An off-the-shelf screw-collar momentary push button mounts next to the PWR USB-C on the back panel, wired to the Main Board soft-latch circuit.

**Integration:** 2-pin wire (5V + GND) from module screw terminal / solder pads to Main Board TPS22965 load switch input. NVM pre-configured for 5V PDO only. Power button wires (2-pin) route to Main Board soft-latch sense/keepalive pins (40/41).

**Recommended modules:** SparkFun Power Delivery Board (USB-C, Qwiic), STEVAL-ISC005V1 (ST eval board), or generic STUSB4500 breakout.

### IO Board

- **Dimensions:** ~50 × 80 mm
- **Orientation:** Horizontal, under top panel (left side)
- **Instances:** 1

**Components:**

- FE1.1s USB 2.0 hub IC (1 upstream via FFC from Teensy USB host, 2 downstream) + 12 MHz crystal
- 2× TPS2051 USB host port power switches (500 mA per port)
- Dual stacked USB-A receptacle (MIDI HOST, top panel)
- RJ45 MagJack with integrated magnetics (Ethernet, top panel) + 0.1µF coupling cap
- 6N138 optocoupler + MIDI IN 3.5mm TRS Type A jack (top panel)
- MIDI OUT 3.5mm TRS Type A jack + series/source resistors (top panel)
- Potentiometer 10 kΩ log (headphone volume, top panel)
- Header for off-the-shelf headphone amp breakout module (4-pin: HP_L, HP_R, 5V_A, GND — receives signals from Main Board)
- FFC connector to Main Board (12-pin 1.0 mm ZIF)
- 6-pin header (Ethernet ribbon cable from Teensy bottom pads via Main Board)
- Decoupling caps

**Panel-mount components** (USB-A, RJ45, MIDI jacks, headphone output, volume pot) protrude through top panel cutouts in the left column.

**Headphone amp:** An off-the-shelf TPA6132 or MAX97220 breakout module (~$2–5) mounts near the IO Board. HP_L and HP_R signals route from the Main Board (codec DAC outputs) directly to the breakout module, not via the Main↔IO FFC. The breakout module's output feeds the volume pot and headphone jack on the IO Board.

**Ethernet:** The Teensy 4.1 has a native Ethernet PHY (DP83825I) with differential TX/RX pairs accessible on bottom pads. A 6-pin ribbon cable routes these signals from the Main Board to the IO Board, where they connect through 0.1µF coupling caps to the RJ45 MagJack's integrated transformer. Post-PHY analog signals are cable-tolerant — no impedance-controlled routing required.

**Design note:** 2-layer PCB sufficient — only USB Full-Speed (12 Mbps for MIDI host hub) and post-PHY Ethernet analog signals. No high-speed digital traces on this board.

------

## Board Summary

| ID | Name | Unique design | Instances | Layers | Active components |
|----|------|--------------|-----------|--------|-------------------|
| M | Main Board | Yes | 1 | 4 | Teensy, power mgmt, TS5A3159 mute, 74LVC1G00 soft-latch, ADP7118 LDO |
| IO | IO Board | Yes | 1 | 2 | FE1.1s hub, 6N138 MIDI, 2× TPS2051, RJ45 MagJack |
| P | Power Module | Off-the-shelf | 1 | — | STUSB4500 USB PD breakout (purchased) |
| 1-top | Input Mother (TDM1) | Shared w/ 2-top | 1 | 4 | ADP7118 LDO, 2× AK4619VN, 8× input analog, 4× output analog |
| 2-top | Input Mother (TDM2) | Shared w/ 1-top | 1 | 4 | ADP7118 LDO, 2× AK4619VN, 8× input analog |
| 1-bot | Input Daughter (TDM1) | Shared w/ all daughters | 1 | 2 | ESD diodes only |
| 2-bot | Input Daughter (TDM2) | Shared | 1 | 2 | ESD diodes only |
| O-top | Output Top | Shared (or near-identical) | 1 | 2 | ESD diodes only |
| O-bot | Output Bottom | Shared | 1 | 2 | ESD diodes only |
| K | Key PCB | Yes | 1 | 2 | 16× NeoPixel, 16× CHOC socket, MCP23017 |

**Unique PCB designs:** 6 (Main, IO, Input Mother, Daughter/Output, Key, plus possibly a separate Output Top if connector differs) + 1 off-the-shelf power module

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

### Power Board → Main Board (power cable)

| Pin | Signal | Notes |
|-----|--------|-------|
| 1 | 5V (VBUS) | From STUSB4500 output, post-polyfuse |
| 2 | GND | Power return |

**Connector:** 2-pin JST-PH (2.0 mm pitch) or screw terminal, ~60–80 mm cable. Carries the full system current (up to 5A) from the Power Board's USB-C input to the Main Board's TPS22965 load switch. Use thick gauge wire (22 AWG or heavier) to minimize voltage drop.

### Main Board ↔ IO Board (FFC cable)

| Pin | Signal | Notes |
|-----|--------|-------|
| 1 | ETH_TX+ | Differential pair, from Teensy bottom 6-pin header |
| 2 | ETH_TX- | Differential pair |
| 3 | GND (guard) | Shield between Ethernet pairs |
| 4 | ETH_RX+ | Differential pair |
| 5 | ETH_RX- | Differential pair |
| 6 | GND (guard) | Shield between Ethernet/USB |
| 7 | USB_HOST_D+ | FE1.1s upstream, USB FS 12 Mbps |
| 8 | USB_HOST_D- | FE1.1s upstream |
| 9 | MIDI_RX | Serial3 pin 15, 31.25 kbaud |
| 10 | MIDI_TX | Serial4 pin 17, 31.25 kbaud |
| 11 | 5V_DIG | USB hub, MIDI, Ethernet circuits |
| 12 | GND | Main return |

**Connector:** 12-pin 1.0 mm pitch FFC, ZIF socket on each board. Cable length ~100–120 mm (IO Board on left side, Main Board center/right). Ethernet signals are post-PHY analog — cable-tolerant at these lengths. Only USB Full-Speed (12 Mbps for MIDI host hub) — no HS signals.

**Ethernet ribbon cable (separate):** A 6-pin ribbon cable carries Ethernet TX+/TX-/RX+/RX- from the Teensy bottom pads through a header on the Main Board to a header on the IO Board. From the IO Board header, signals route through 0.1µF coupling caps to the RJ45 MagJack. This is separate from the FFC and carries only the Ethernet differential pairs + GND.

**Headphone signals:** HP_L and HP_R from the codec DAC route from the Main Board directly to the off-the-shelf headphone amp breakout module via short wires, not through the Main↔IO FFC.

**Why separate IO board:** Simplifies main board layout, shortens panel-mount wiring for left-column connectors, and allows independent revision of the IO section without re-spinning the main board.

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
| Main ↔ IO Board | FFC + ZIF | 1.0 mm | 12 | ~100–120 mm |
| Power Board → Main | JST-PH or screw terminal | 2.0 mm | 2 | ~60–80 mm |
| 1-top → O-top | JST-PH or FFC | 2.0 / 1.0 mm | 10 | ~80 mm |
| Main ↔ Key PCB | JST-PH | 2.0 mm | 6 | ~30–40 mm |
| Main ↔ Display | Per module | Varies | 8–10 | ~20 mm |

------

## Back Panel Board Arrangement

Looking at the back panel (260 mm wide × 50 mm tall):

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  O-top   │         1-top          │         2-top          │ [PWR]  [PWR]  │
│  4 jacks │        4 jacks         │        4 jacks         │ USB-C  BUTTON │
│(Mst,A1-3)│    (L in 1,3,5,7)      │   (L in 9,11,13,15)   │(Pwr Brd)      │
├──────────┼────────────────────────┼────────────────────────┤               │
│  O-bot   │         1-bot          │         2-bot          │               │
│  4 jacks │        4 jacks         │        4 jacks         │               │
│ (R outs) │    (R in 2,4,6,8)      │   (R in 10,12,14,16)  │               │
└─────────────────────────────────────────────────────────────────────────────┘
              ← outputs                    inputs →
```

The PWR USB-C receptacle mounts on the **Power Board** — a small dedicated PCB on the far right of the back panel. Labeled "PWR" (power only, 5V/5A PD). A 2-pin cable carries 5V + GND from the Power Board to the Main Board. An **off-the-shelf screw-collar momentary push button** ("POWER") mounts next to the PWR USB-C on the back panel, wired to the Main Board soft-latch circuit. The **PC USB-C** (data only, USB Audio + MIDI composite) is on the **top panel** (left zone, Main Board mount). The three mother+daughter pairs tile across the remaining back panel width.

------

## Design Reuse Opportunities

1. **Input mother boards (1-top, 2-top):** Same PCB. Codec I2C addresses set by solder jumpers. Output analog section on Board 1-top populated; on Board 2-top left empty.
2. **All daughter/output boards (1-bot, 2-bot, O-top, O-bot):** Potentially same PCB if connector placement and jack spacing match. All are 4× TS jacks + ESD + one connector. Worth investigating during schematic phase — could reduce unique designs from 6 to 4 (Main, IO, Input Mother, Universal Daughter, Key).
3. **Key PCB** and **IO Board** are standalone custom designs with no reuse opportunities. The **Power Module** is an off-the-shelf STUSB4500 breakout (no custom PCB). The **headphone amp** is an off-the-shelf TPA6132 or MAX97220 breakout module.

------

## Mechanical Mounting

- **Main Board:** Screwed to top panel via standoffs (M3 or M2.5). Board hangs below top panel, components protrude through panel cutouts.
- **IO Board:** Screwed to top panel via standoffs (M3 or M2.5), left side. Panel-mount components (USB-A, RJ45, MIDI jacks, headphone output, volume pot) protrude through top panel cutouts. Connected to main board via 12-pin FFC + 6-pin Ethernet ribbon cable.
- **I/O Boards (Mother, Daughter, Output):** Mechanically held by panel-mount jack nuts — the 1/4" TS jacks thread through back panel holes and their nuts clamp the boards to the panel. No additional standoffs needed.
- **Power Module:** Off-the-shelf STUSB4500 breakout, secured to back panel by USB-C jack nut or adhesive standoff.
- **Key PCB:** Mounted to top panel via standoffs or snap-fit clips. CHOC switches protrude through top panel cutouts, keycaps sit flush with panel surface.
