# MIXTEE: Teensy 4.1 Pin Mapping

*← Back to [README](../README.md) | See also: [Hardware](hardware.md) · [PCB Architecture](pcb-architecture.md) · [AK4619VN Wiring](ak4619-wiring.md)*

------

## Pin Reference Source

All assignments verified against the official [PJRC Teensy 4.1 reference cards](https://www.pjrc.com/store/teensy41.html) (card11a/card11b rev4) and the PJRC Audio Library source code (output_tdm.cpp, input_tdm.cpp, output_tdm2.cpp, input_tdm2.cpp).

------

## Peripheral Pin Assignments (Fixed)

These pins are consumed by dedicated hardware peripherals and cannot be reassigned.

### SAI1 — TDM Bus 1 (Codecs U1 + U2)

| Teensy Pin | Function | Card Label | iMXRT1062 Pad | Direction |
|-----------|----------|------------|---------------|-----------|
| **23** | SAI1 MCLK | MCLK1 | AD_B1_09 | Output (master clock) |
| **21** | SAI1 BCLK | BCLK1 | AD_B1_11 | Output (bit clock) |
| **20** | SAI1 LRCLK | LRCLK1 | AD_B1_10 | Output (frame sync) |
| **7** | SAI1 TX_DATA0 | OUT1A | B1_00 | Output (Teensy → codec) |
| **8** | SAI1 RX_DATA0 | IN1 | B1_01 | Input (U1 SDOUT → Teensy) |
| **32** | SAI1 RX_DATA1 | OUT1B | B0_10 | Input (U2 SDOUT → Teensy) |

BCLK = 48 kHz × 16 slots × 32 bits = **24.576 MHz**. MCLK = 256×fs = **12.288 MHz**.

**Note:** Pin 23 is also the built-in LED — LED unavailable when audio is active.

### SAI2 — TDM Bus 2 (Codecs U3 + U4)

| Teensy Pin | Function | Card Label | iMXRT1062 Pad | Direction |
|-----------|----------|------------|---------------|-----------|
| **33** | SAI2 MCLK | MCLK2 | EMC_07 | Output (master clock) |
| **4** | SAI2 BCLK | BCLK2 | EMC_06 | Output (bit clock) |
| **3** | SAI2 LRCLK | LRCLK2 | EMC_05 | Output (frame sync) |
| **2** | SAI2 TX_DATA0 | OUT2 | EMC_04 | Output (Teensy → codec) |
| **5** | SAI2 RX_DATA0 | IN2 | EMC_08 | Input (U3 SDOUT → Teensy) |
| **34** | SAI2 RX_DATA1 | — | B0_12 | Input (U4 SDOUT → Teensy), **bottom pad** |

**Critical:** Pin 33 is a **bottom pad** — not accessible from the breadboard edge. On the custom PCB, route this from a via or bottom-side pad to the FFC connector for TDM2.

### I2C — Codec Control Bus

| Teensy Pin | Function | Card Label | Bus |
|-----------|----------|------------|-----|
| **18** | SDA | SDA | Wire (I2C0) |
| **19** | SCL | SCL | Wire (I2C0) |

All four AK4619VN codecs share one I2C bus via a **TCA9548A I2C mux** (address 0x70) on the main board. The mux isolates each Input Mother Board onto its own channel (Ch 0 for U1/U2, Ch 1 for U3/U4), resolving the AK4619VN 2-address limitation. A **MCP23017 I2C GPIO expander** (address 0x20) on the Key PCB handles the 4×4 key scan matrix over the same bus. SDA and SCL routed on both FFC cables and the Key PCB cable. External pull-ups: 4.7kΩ to 3.3V on the main board (upstream of mux). Each Input Mother Board has its own downstream pull-ups. See [AK4619VN Wiring](ak4619-wiring.md) for addressing details.

### SPI0 — RA8875 TFT Display

| Teensy Pin | Function | Card Label |
|-----------|----------|------------|
| **10** | CS | CS |
| **11** | MOSI | MOSI |
| **12** | MISO | MISO |
| **13** | SCK | SCK |

Standard SPI0 bus. Display is the only SPI0 device — no shared bus contention.

### Serial3 — MIDI IN (6N138 Output)

| Teensy Pin | Function | Card Label |
|-----------|----------|------------|
| **15** | RX (MIDI data in) | S/PDIF IN, RX3 |
| (14) | TX (unused, see note) | S/PDIF OUT, TX3 |

MIDI IN is receive-only at 31,250 baud. Pin 15 configured as Serial3 RX.

**Pin 14 note:** `Serial3.begin(31250)` claims both TX (pin 14) and RX (pin 15). Pin 14 is reclaimed as GPIO in firmware by overriding the IOMUX after Serial3 initialization. Used for RA8875 RESET (see GPIO section).

### Serial4 — MIDI OUT

| Teensy Pin | Function | Card Label |
|-----------|----------|------------|
| **17** | TX (MIDI data out) | SDA1, TX4 |

MIDI OUT is transmit-only at 31,250 baud. Pin 17 configured as Serial4 TX. Standard MIDI current-loop output via 3.5mm TRS Type A jack. Software handles pass-through of MIDI IN messages and/or Teensy-generated MIDI output.

### USB Host (Bottom Pads)

| Pad | Signal |
|-----|--------|
| +5V | VBUS out (to FE1.1s upstream) |
| D- | USB data minus |
| D+ | USB data plus |
| GND | Ground (×2 pads) |

These are dedicated USB2 PHY pads on the bottom of the Teensy 4.1 — not GPIO pins. Connected to the FE1.1s USB hub IC upstream port. Use `USBHost_t36` library for USB MIDI host.

### SDIO — External Full-Size SD Card Socket

| Teensy Bottom Pad | Signal | Connection |
|-------------------|--------|------------|
| 42 | SD_DAT3 | Routed to external full-size SD socket on Main Board |
| 43 | SD_DAT2 | Routed to external full-size SD socket on Main Board |
| 44 | SD_DAT1 | Routed to external full-size SD socket on Main Board |
| 45 | SD_DAT0 | Routed to external full-size SD socket on Main Board |
| 46 | SD_CLK | Routed to external full-size SD socket on Main Board |
| 47 | SD_CMD | Routed to external full-size SD socket on Main Board |

The Teensy 4.1's SDIO bottom pads (pins 42–47) are routed from the Teensy underside directly to a panel-mount full-size SD card socket on the Main Board. The Teensy's built-in micro-SD slot is left physically empty. Access via `SD.begin(BUILTIN_SDCARD)` — the SDIO peripheral is the same whether using the onboard slot or external routing. Native 4-bit SDIO provides 20–30 MB/s sustained write — well above the 2.3 MB/s needed for 16-channel recording.

### QSPI — PSRAM (Bottom Pads)

Pins 49, 50, 52, 53, 54 (plus CS on 48 or 51) are consumed by the 8 MB PSRAM soldered to the bottom QSPI pads. Unavailable for GPIO.

------

## Peripheral Pin Summary

| Pin | Peripheral | Unavailable alternates |
|-----|-----------|----------------------|
| 2 | SAI2 TX_DATA0 | CRX2, CTX2 |
| 3 | SAI2 TX_SYNC | — |
| 4 | SAI2 TX_BCLK | — |
| 5 | SAI2 RX_DATA0 | — |
| 7 | SAI1 TX_DATA0 | RX2 (Serial2 blocked) |
| 8 | SAI1 RX_DATA0 | TX2 (Serial2 blocked) |
| 10 | SPI0 CS | — |
| 11 | SPI0 MOSI | CTX1 |
| 12 | SPI0 MISO | — |
| 13 | SPI0 SCK | (LED) |
| 15 | Serial3 RX (MIDI IN) | S/PDIF IN, RX4 |
| 17 | Serial4 TX (MIDI OUT) | SDA1 |
| 18 | Wire SDA | A4 |
| 19 | Wire SCL | A5 |
| 20 | SAI1 TX_SYNC | TX5, RX5 (Serial5 blocked) |
| 21 | SAI1 TX_BCLK | TX5, RX5 (Serial5 blocked) |
| 23 | SAI1 MCLK | CRX1, CTX1, A9, (LED) |
| 32 | SAI1 RX_DATA1 | OUT1B (SAI1_DATA1) |
| 33 | SAI2 MCLK | Bottom pad |
| 34 | SAI2 RX_DATA1 | Bottom pad |

**Total edge pins consumed: 20** (plus bottom pads for USB Host, SDIO, QSPI)

------

## GPIO Budget

Total Teensy 4.1 edge pins: **42** (pins 0–41)
Consumed by peripherals: **20** (17 original + pin 32 SAI1_RX_DATA1 + pin 34 SAI2_RX_DATA1 + pin 17 MIDI OUT)
Available for GPIO: **22**

### GPIO Requirements

| Function | Pins needed | Notes |
|----------|------------|-------|
| Encoder 1 (NavX) | **3** | A, B, push switch — horizontal navigation |
| Encoder 2 (NavY) | **3** | A, B, push switch — vertical navigation |
| Encoder 3 (Edit) | **3** | A, B, push switch — value editing |
| NeoPixel data | **1** | Single data line to 16× WS2812B |
| RA8875 INT | **1** | Active-low interrupt |
| RA8875 RESET | **1** | Active-low reset (reclaimed from Serial3 TX) |
| TS5A3159 mute control | **4** | 1 per output stereo pair |
| Headphone detect | **1** | TRS jack switch input |
| Power button sense | **1** | Soft-latch button state |
| KEEP_ALIVE | **1** | Hold soft-latch set during operation |
| MCP23017 INT (optional) | **1** | Interrupt-driven key scan; can be omitted if polling |
| MIDI OUT | **1** | Serial4 TX (pin 17) |
| **Total** | **21** | |

**Remaining spare: 2 pins** (pins 0 and 1 reserved for Serial1 debug)

### Key Matrix via MCP23017

The 4×4 key matrix (16 keys) is handled entirely by a **MCP23017 I2C GPIO expander** (address 0x20) on the Key PCB. This frees 7 Teensy GPIO pins compared to a direct-wired matrix:

- MCP23017 Port A (GPA0–3): 4 column inputs with internal pull-ups
- MCP23017 Port B (GPB0–3): 4 row outputs (active-low scan)
- 16× 1N4148 diodes per switch (cathode toward row) prevent ghosting
- Scan rate: polled via I2C at ~1 kHz, or interrupt-driven via INTA/INTB pin
- Only I2C (SDA/SCL) + optional INT needed in the Key↔Main cable (6 pins total)

------

## GPIO Pin Assignments

| Teensy Pin | Assignment | Direction | Notes |
|-----------|-----------|-----------|-------|
| **0** | *(spare — Serial1 RX)* | — | Reserved for debug UART |
| **1** | *(spare — Serial1 TX)* | — | Reserved for debug UART |
| **6** | NeoPixel data out | Output | 300–500Ω series resistor; also has OUT1D (SAI1_DATA3) alternate — not used |
| **9** | RA8875 display INT | Input | Active-low interrupt; also has OUT1C (SAI1_DATA2) alternate — not used |
| **14** | RA8875 display RESET | Output | Active-low; IOMUX reclaimed from Serial3 TX after `Serial3.begin()` |
| **16** | Encoder 3 (Edit) — Push | Input (pull-up) | Also SCL1/RX4 alternate |
| **17** | MIDI OUT (Serial4 TX) | Output | 31.25 kbaud MIDI output; also SDA1/TX4 alternate |
| **22** | MCP23017 INT (optional) | Input (pull-up) | Key scan interrupt from Key PCB; also A8 |
| **24** | Encoder 1 (NavX) — A | Input (pull-up) | Interrupt-capable; horizontal navigation; also SCL2/A10 |
| **25** | Encoder 1 (NavX) — B | Input (pull-up) | Interrupt-capable; also SDA2/A11 |
| **26** | Encoder 3 (Edit) — A | Input (pull-up) | Interrupt-capable; also A12 |
| **27** | Encoder 3 (Edit) — B | Input (pull-up) | Interrupt-capable; also A13/SCK1 |
| **28** | Encoder 1 (NavX) — Push | Input (pull-up) | Also RX7 |
| **29** | Encoder 2 (NavY) — A | Input (pull-up) | Interrupt-capable; vertical navigation; also CRX3 |
| **30** | TS5A3159 mute — Main L/R | Output | Relocated from pin 34 (now SAI2_RX_DATA1); high = unmuted |
| **31** | Encoder 2 (NavY) — B | Input (pull-up) | Relocated from pin 32 (now SAI1_RX_DATA1); interrupt-capable |
| **35** | TS5A3159 mute — AUX1 L/R | Output | **Bottom pad**; also TX8/RX8 |
| **36** | Encoder 2 (NavY) — Push | Input (pull-up) | |
| **37** | TS5A3159 mute — AUX2 L/R | Output | |
| **38** | TS5A3159 mute — AUX3 L/R | Output | Also A14 |
| **39** | Headphone detect | Input (pull-up) | TRS jack switch; also A15 |
| **40** | Power button sense | Input (pull-up) | Reads soft-latch button state; also A16 |
| **41** | KEEP_ALIVE | Output | Holds soft-latch set; release to shut down; also A17 |

### Key Matrix Wiring (on Key PCB, via MCP23017)

```
             GPA0          GPA1          GPA2          GPA3
         (COL 0, pull-up) (COL 1)      (COL 2)      (COL 3)
              │              │              │              │
GPB0 (ROW 0)─┼──[SW1]──|>──┼──[SW2]──|>──┼──[SW3]──|>──┼──[SW4]──|>──
              │              │              │              │
GPB1 (ROW 1)─┼──[SW5]──|>──┼──[SW6]──|>──┼──[SW7]──|>──┼──[SW8]──|>──
              │              │              │              │
GPB2 (ROW 2)─┼──[SW9]──|>──┼──[SW10]─|>──┼──[SW11]─|>──┼──[SW12]─|>──
              │              │              │              │
GPB3 (ROW 3)─┼──[SW13]─|>──┼──[SW14]─|>──┼──[SW15]─|>──┼──[SW16]─|>──
              │              │              │              │
```

`|>` = 1N4148 diode (cathode toward row pin). Column pins use MCP23017 internal pull-ups. All matrix wiring is local to the Key PCB — only I2C + INT + power travel in the 6-pin cable to the main board.

### Key ↔ Switch Mapping

| Matrix Position | Key Function | NeoPixel Index |
|----------------|-------------|----------------|
| ROW0 × COL0 (SW1) | Mute | 0 |
| ROW0 × COL1 (SW2) | Solo | 1 |
| ROW0 × COL2 (SW3) | Rec | 2 |
| ROW0 × COL3 (SW4) | (assignable) | 3 |
| ROW1 × COL0 (SW5) | (assignable) | 4 |
| ROW1 × COL1 (SW6) | (assignable) | 5 |
| ROW1 × COL2 (SW7) | (assignable) | 6 |
| ROW1 × COL3 (SW8) | (assignable) | 7 |
| ROW2 × COL0 (SW9) | (assignable) | 8 |
| ROW2 × COL1 (SW10) | (assignable) | 9 |
| ROW2 × COL2 (SW11) | (assignable) | 10 |
| ROW2 × COL3 (SW12) | (assignable) | 11 |
| ROW3 × COL0 (SW13) | Home | 12 |
| ROW3 × COL1 (SW14) | Back | 13 |
| ROW3 × COL2 (SW15) | Page | 14 |
| ROW3 × COL3 (SW16) | Shift | 15 |

------

## Alternate Functions Lost

Using certain pins as GPIO makes their alternate peripheral functions unavailable:

| Pins | Lost alternate | Impact |
|------|---------------|--------|
| 24, 25 | Wire2 (I2C2), Serial6 | Not needed |
| 28 | Serial7 RX | Not needed — MIDI uses Serial3 |
| 29 | CAN3 RX | Not needed |
| 0, 1 | SPI1 CS/MISO, CAN2 | Kept as Serial1 debug; SPI1/CAN2 not needed |

**Note:** Pin 16 (Wire1 SCL / Serial4 RX) remains spare. Pin 17 is used as Serial4 TX for MIDI OUT — its Wire1 SDA alternate is unavailable.

------

## Conflict Warnings

1. **Serial2 (pins 7/8) is BLOCKED** — these pins are SAI1 audio data lines
2. **Serial5 (pins 20/21) is BLOCKED** — these pins are SAI1 BCLK/LRCLK
3. **Pin 23 = LED** — built-in LED unavailable during audio playback
4. **Pin 14 dual-use** — Serial3 TX alternate; must override IOMUX in firmware to use as GPIO
5. **Pins 34, 35 are bottom pads** — require PCB traces from Teensy underside; not breadboard accessible
6. **Pin 33 is a bottom pad** — SAI2 MCLK must be routed from Teensy underside to FFC connector
7. **Pin 32 is now SAI1_RX_DATA1** — no longer available as GPIO (was Encoder 2 B, moved to pin 31)
8. **Pin 34 is now SAI2_RX_DATA1** — no longer available as GPIO (was TS5A3159 mute Main, moved to pin 30)
9. **Pins 6, 9 have SAI1 DATA2-3 alternates** — used as GPIO; safe because the Audio Library multi-data-line mode only uses DATA0 + DATA1

------

## FFC Cable Pin Mapping

### Main ↔ Input Mother Board (16-pin 1.0mm FFC)

| FFC Pin | Signal | Teensy Pin | Direction |
|---------|--------|-----------|-----------|
| 1 | MCLK | 23 (TDM1) or 33 (TDM2) | Out |
| 2 | BCLK | 21 (TDM1) or 4 (TDM2) | Out |
| 3 | LRCLK | 20 (TDM1) or 3 (TDM2) | Out |
| 4 | TDM DATA IN (codec → Teensy) | 8 (TDM1) or 5 (TDM2) | In |
| 5 | TDM DATA OUT (Teensy → codec) | 7 (TDM1) or 2 (TDM2) | Out |
| 6 | I2C SDA | 18 | Bidirectional |
| 7 | I2C SCL | 19 | Out |
| 8 | 5V_DIG | — | Power |
| 9 | 5V_A (to codec board LDO input) | — | Power |
| 10 | GND | — | Ground |
| 11 | GND | — | Ground |
| 12 | GND | — | Ground |
| 13 | TDM DATA IN 2 (codec 2 SDOUT → Teensy) | 32 (TDM1) or 34 (TDM2) | In |
| 14–16 | (spare) | — | Future: codec PDN, interrupt |

### Main ↔ Key PCB Cable

With the MCP23017 handling the key matrix on-board, the cable only carries NeoPixel data, I2C, interrupt, and power:

| Pin | Signal | Teensy Pin |
|-----|--------|-----------|
| 1 | NeoPixel DIN | 6 |
| 2 | I2C SDA | 18 |
| 3 | I2C SCL | 19 |
| 4 | MCP23017 INT | 22 (optional) |
| 5 | 5V | — |
| 6 | GND | — |

**Connector:** 6-pin JST-PH (2.0mm pitch), ~30–40mm cable.

------

## Power Sequencing (Soft-Latch)

```
Button press → SR latch SET → TPS22965 EN → 5V rail on → Teensy boots
                                                            ↓
                                            Teensy asserts KEEP_ALIVE (pin 41)
                                            Teensy reads button via pin 40

Button press → Teensy detects (pin 40) → saves state → releases KEEP_ALIVE (pin 41)
                                                            ↓
                                            SR latch RESET → TPS22965 off → power down

Long-press (>4s) → RC timeout resets latch directly (hardware failsafe)
```

------

## Verification Checklist

- [ ] Confirm SAI1/SAI2 pin assignments match PJRC Audio Library `output_tdm.cpp` / `input_tdm2.cpp`
- [ ] Verify SAI1_RX_DATA1 (pin 32) and SAI2_RX_DATA1 (pin 34) work with modified Audio Library for multi-data-line RX
- [ ] Verify pin 33 (bottom pad) can be routed on custom PCB to FFC connector
- [ ] Confirm pins 6, 9 are not claimed by Audio Library in TDM mode (DATA2/DATA3 alternates unused)
- [ ] Test Serial3 RX-only operation with pin 14 reclaimed as GPIO
- [ ] Verify MCP23017 I2C key matrix scan works at 0x20 (no bus conflicts with TCA9548A at 0x70)
- [ ] Check encoder interrupt latency on pins 24/25/26/27/29/31 (all interrupt-capable ✓)
- [ ] Validate FE1.1s upstream D+/D- connects to Teensy USB Host bottom pads (not USB Device)
- [ ] Print 1:1 Teensy 4.1 footprint, verify bottom pad access for pins 33, 34, 35, 42–47
- [ ] Verify external SD card socket SDIO routing from Teensy bottom pads 42–47
