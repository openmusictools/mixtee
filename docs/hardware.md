# MIXTEE: Hardware

*← Back to [README](../README.md) | See also: [Features](features.md) · [Firmware](firmware.md) · [UI Architecture](ui-architecture.md) · [System Topology](system-topology.md)*

------

## Audio Codec Architecture

- **Codec:** 4× AK4619VN (4-in / 4-out per chip, unified vendor)
  - Total capacity: 16 ADC channels + 16 DAC channels (8 DAC outputs used)
  - Single vendor simplifies TDM formatting, clock distribution, and register configuration
  - TDM/I2S serial audio interface
  - I2C control interface (2 addresses via single CAD pin; TCA9548A I2C mux on main board isolates each codec board — see [AK4619VN Wiring](../hardware/pcbs/input-mother/ak4619-wiring.md))
  - 48 kHz / 24-bit target
- **Digital Interface:** TDM over I2S to Teensy 4.1

## TDM Bus Assignment

- **TDM1 (SAI1):** Codecs U1 + U2 (ADC channels 1-8, DAC channels 1-8)
  - U1: TDM slots 0-3 on SAI1_RX_DATA0 (pin 8) — inputs 1-4; slots 0-3 TX — outputs 1-4 (Main L/R + Aux1 L/R)
  - U2: TDM slots 0-3 on SAI1_RX_DATA1 (pin 32) — inputs 5-8; slots 0-3 TX — outputs 5-8 (Aux2 L/R + Aux3 L/R)
  - Each codec transmits on its own SDOUT line — no bus contention
- **TDM2 (SAI2):** Codecs U3 + U4 (ADC channels 9-16, DAC unused)
  - U3: TDM slots 0-3 on SAI2_RX_DATA0 (pin 5) — inputs 9-12
  - U4: TDM slots 0-3 on SAI2_RX_DATA1 (pin 34, bottom pad) — inputs 13-16
  - DAC sections on U3/U4 unused (available for future expansion)
- **Clock:** Teensy generates MCLK (256fs = 12.288 MHz), BCLK, and LRCLK as TDM master
  - All four codecs slave to shared clock tree
  - BCLK = 48 kHz × 16 slots × 32 bits = 24.576 MHz per TDM bus
- **Audio Library modification required:** Enable SAI multi-data-line receive via SAI_RCR3 register to read both RX_DATA0 and RX_DATA1 per bus

## MCU / DSP Core

- **Teensy 4.1** (ARM Cortex-M7 @ 600 MHz)
  - 2× I2S/TDM interfaces for 16-in/8-out audio
  - USB host capability for MIDI controllers
  - Extensive GPIO for encoders/buttons/display
  - PJRC Audio Library ecosystem (mixer objects, effects, routing)
- **DAW connectivity:** Via Ethernet (AES67) — 16-in / 8-out network audio using virtual soundcards on the host PC. See [network-connectivity.md](network-connectivity.md) §9 for DAW integration details.

------

*Analog stage design (input stage, output stage, op-amp selection): see [Input Mother Board architecture](../hardware/pcbs/input-mother/architecture.md).*

------

## Power System

### Power Input

**Dual USB-C (separate power and data):**

- **PWR — USB-C (power only, USB PD 5V/5A) — off-the-shelf STUSB4500 breakout (back panel):**
  - Purchased breakout module (SparkFun Power Delivery Board, STEVAL-ISC005V1, or generic)
  - STUSB4500 USB PD sink controller — negotiates 5V @ 5A from PD-capable supplies
  - Fallback: 5.1kΩ CC resistors on module for non-PD supplies (defaults to 5V/3A)
  - NVM pre-configured to request 5V PDO only (no higher voltages)
  - 2-pin wire (5V + GND) from module output to Main Board TPS22965 input
  - Labeled "PWR" on back panel (right side)
- **DAW connectivity via Ethernet:**
  - 16-in / 8-out AES67 network audio via RJ45 MagJack on IO Board
  - No USB audio bridge — DAW connectivity uses standard AES67 virtual soundcards on host PC
  - See [network-connectivity.md](network-connectivity.md) §9 for stream layout and recommended software

### Power Budget (5V rail)

- **USB host ports:** 2× 500 mA = 1.0 A
- **NeoPixels (16 keys):** ~320 mA typical (at 30% cap), 960 mA worst-case (uncapped)
- **DESPEE display module:** Self-powered from 5V_DIG (~250-350 mA including backlight)
- **Teensy + logic:** ~200 mA
- **Isolated analog domain (via 2× MEJ2S0505SC):** ~400 mA total (2× codecs, op-amps, ADP7118 LDOs, TS5A3159, MCP23008, HP amp)
- **Worst-case total:** ~2.67 A (with uncapped NeoPixels)
- **With 20% reserve:** ~3.20 A
- **Supply target:** 5V @ 5A via USB PD (headroom for uncapped NeoPixels + builder modifications)

### Power Distribution

5V rail is split into 5V_DIG (noisy digital loads: USB, NeoPixels, TFT) and 5V_ISO (galvanically isolated analog domain via MEJ2S0505SC DC-DC converters). TPS22965 load switch provides soft-start. ADP7118 LDOs (2 instances — 1 per Input Mother Board) generate clean 3.3V_A rails. See [Main Board architecture](../hardware/pcbs/main/architecture.md) for detailed power distribution and galvanic isolation design.

*USB host hub details: see [IO Board architecture](../hardware/pcbs/io/architecture.md).*

*Headphone amplifier details: see [PHONEE architecture](../hardware/pcbs/hp/architecture.md).*

*Ethernet details: see [IO Board architecture](../hardware/pcbs/io/architecture.md).*

*MIDI IN circuit details: see [IO Board architecture](../hardware/pcbs/io/architecture.md).*

*MIDI OUT circuit details: see [IO Board architecture](../hardware/pcbs/io/architecture.md).*

*Soft-latch power circuit details: see [Main Board architecture](../hardware/pcbs/main/architecture.md).*

*Galvanic isolation details: see [Main Board architecture](../hardware/pcbs/main/architecture.md#galvanic-isolation).*

------

## PCB Layer Stackup

| Board | Layers | Stackup | Rationale |
|-------|--------|---------|-----------|
| Main Board | **4-layer** | Sig / GND / PWR / Sig | TDM clock integrity (24.576 MHz BCLK), galvanic isolation boundary (GND + GND_ISO islands), power distribution |
| IO Board | **2-layer** | Sig / GND | USB Full-Speed (12 Mbps), post-PHY Ethernet analog, MIDI — no high-speed digital |
| Power Module | Off-the-shelf | — | STUSB4500 USB PD breakout (purchased module, no custom PCB) |
| Input Mother Board | **4-layer** | Sig / GND_ISO / PWR_ISO / Sig | AK4619VN codecs + analog input stages + TDM signals; entirely on GND_ISO (isolated domain) |
| Input Daughter Board | **2-layer** | Sig / GND_ISO | Simple board: jacks + ESD diodes + connector; inherits isolation from mother board |
| Output Board | **2-layer** | Sig / GND_ISO | Simple board: jacks + ESD diodes + connector; inherits isolation from mother board |
| PHONEE | **2-layer** | Sig / GND_ISO | TPA6132A2 headphone amp + PCB-mount volume pot + TRS jack; entirely on GND_ISO; reusable external module ([openaudiotools/phonee](https://github.com/openaudiotools/phonee)) |
| Keys4x4 PCB | **2-layer** | Sig / GND | Switches + LEDs only, no high-speed signals |

------

## Noise Mitigation Strategy

### Galvanic Isolation (Primary)

The primary noise mitigation is **galvanic isolation** between the digital domain (Main Board, IO Board, Keys4x4 PCB) and the analog domain (Input Mother Boards, Daughter/Output Boards, PHONEE). No shared copper between domains — all signals and power cross the boundary through isolators:

- **Si8662BB-B-IS1** digital isolators (×2) for TDM signals (150 Mbps, 6-channel)
- **ISO1541DR** isolated I2C (×2) for codec/MCP23008 control
- **MEJ2S0505SC** isolated DC-DC converters (×2) for analog domain power

This eliminates USB ground loops, switching noise, and NeoPixel current spikes from the analog signal path entirely — not just attenuated, but physically absent.

### Power Integrity

1. **Bulk capacitor (1000–2200 µF)** across 5V/GND near NeoPixels (buffers current spikes from up to 960 mA worst-case draw)
2. **Isolated DC-DC (MEJ2S0505SC)** provides galvanic separation between digital and analog power — replaces ferrite bead approach
3. **Post-filter on analog boards:** ferrite bead (600Ω @ 100 MHz) + 10 µF ceramic after 5V_ISO, before ADP7118 LDO input. Combined PSRR >100 dB at DC-DC switching frequency
4. **Local decoupling** at every IC and subsystem boundary (0.1 µF ceramic + bulk)
5. **NeoPixel series resistor (300–500 Ω)** in data line near first pixel (reduces ringing)

### Grounding

- **Digital domain (Main Board):** Single continuous GND plane with star topology for high-current returns (USB, NeoPixels, TFT backlight)
- **Analog domain (Mother Boards, PHONEE):** Continuous GND_ISO plane, entirely isolated from digital GND
- **Main Board isolation boundary:** GND_ISO exists only as small copper islands around isolator Side 2 pins and FFC pads. **≥1 mm clearance** between GND and GND_ISO copper on all layers
- **No ground connection** between domains — isolation integrity must be maintained through layout

### Layout

- **Physical separation:** USB hub/switching away from audio codecs and outputs (on separate boards)
- **Isolation component placement:** MEJ2S0505SC, Si8662BB, ISO1541 placed near each FFC connector on Main Board
- **Keep DC-DC away from signal isolators:** MEJ2S0505SC switching noise could couple into Si8662BB signal pins
- **Short audio traces:** minimize analog signal path length on Mother Boards

### Firmware-Based Noise Mitigation

- **Cap NeoPixel global brightness** (30% default recommended — reduces noise and power; hardware is sized for uncapped operation in case builders remove the limit)
- **Smooth parameter changes** (no abrupt steps in gain/pan)
- **USB host power management** (ability to power-cycle ports via software)
- **I2C mute sequencing:** MCP23008 mute outputs held low during boot; high after DSP stabilizes (~500 ms)

------

## Bill of Materials

The canonical BOM is [`hardware/bom.csv`](../hardware/bom.csv). Per-board component details are in each board's `architecture.md`.

------

## Target Specifications

### Audio Performance (Target Goals)

- **Sample Rate:** 48 kHz
- **Bit Depth:** 24-bit
- **Latency:** <5 ms round-trip (ADC → processing → DAC)
- **THD+N:** <0.01% (codec-dependent, AK4619VN: 0.001% typ at 1 kHz)
- **Dynamic Range:** >100 dB (codec-dependent)
- **Crosstalk:** <-80 dB @ 1 kHz

### Recording Performance (Target Goals)

- **Channels:** 16 simultaneous (post-ADC, pre-mixer)
- **Format:** 48 kHz / 24-bit WAV (RF64 for files >4 GB)
- **Data Rate:** 2.3 MB/s sustained
- **Max Session Length:** Limited by SD card capacity (~3.5 hrs on 32 GB, ~7 hrs on 64 GB)
- **Buffer Tolerance:** >3 seconds of SD write stall without dropout (8 MB PSRAM)
- **Zero-dropout target:** No missed samples during normal operation with recommended SD card class

### Physical (Estimated)

- **Dimensions:** 260×100×50 mm (W×D×H) — compact desktop format
- **Weight:** ~1-1.5 kg (depending on enclosure material)
- **Power Consumption:** 5V @ 2-3A typical, 3.4A worst-case (uncapped NeoPixels); 5A supply via USB PD

### Panel Layout (v2.0)

**Enclosure:** 260 × 100 × 50 mm (W × D × H). See [enclosure.md](enclosure.md) for full detail.

**Top panel (260 × 84.6 mm — all controls and connectivity):**

Left zone (Main Board):
- Full-size SD card slot (left of display, vertically aligned with bottom edge of screen, slot opens upward)
- 1× Custom display PCB (ESP32-S3-WROOM-1-N16R8 + 4.3" 800×480 LCD panel; designed to MIXTEE enclosure dimensions)
- 3× rotary encoders (NavX + NavY + Edit): horizontal row below display, mounted on DESPEE display PCB (not wired to Main Board)

Center:
- 16× CHOC key switches (4×4 grid): top-aligned with display

Left column (IO Board + PHONEE):
- MIDI HOST dual USB-A (stacked)
- ETH RJ45 (Ethernet)
- MIDI IN + MIDI OUT (3.5mm TRS Type A)
- Headphone output (from PHONEE module, isolated analog domain)
- PHONES label + VOL pot (PCB-mount on PHONEE)

**Back panel (260 × 50 mm — all audio I/O + USB-C + power):**

- 24× 1/4" TS jacks: 12 evenly spaced stereo pairs (L top row, R bottom row), 20 mm center-to-center
- Order left to right (looking at back): Master, AUX1, AUX2, AUX3, 15/16, 13/14, 11/12, 9/10, 7/8, 5/6, 3/4, 1/2
- No physical gaps between output and input groups — separation by labeling only
- PWR USB-C (power only, 5V/5A USB PD): right side of back panel, on dedicated Power Board
- POWER button (momentary, screw-collar): next to PWR USB-C on back panel, wired to Main Board soft-latch

**No front, left, or right side panels with connectors.** All I/O accessible from top and back.

**Validate** with 1:1 panel template printout during Phase 1 before committing to enclosure fabrication. See [`mixtee-layout.svg`](../hardware/mixtee-layout.svg) for detailed panel drawings.

### User Experience

- **Boot Time:** <3 seconds to ready
- **UI Responsiveness:** <50 ms encoder/button to visual feedback (encoders are local to DESPEE — sub-frame LVGL latency for navigation; semantic events forwarded to Teensy over UART)
- **MIDI Latency:** <10 ms CC message to parameter change
- **Hot-plug:** MIDI controllers safe to connect/disconnect during operation
