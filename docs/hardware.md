# MIXTEE: Hardware

*← Back to [README](../README.md) | See also: [Features](features.md) · [Firmware](firmware.md) · [UI Architecture](ui-architecture.md)*

------

## Audio Codec Architecture

- **Codec:** 4× AK4619VN (4-in / 4-out per chip, unified vendor)
  - Total capacity: 16 ADC channels + 16 DAC channels (8 DAC outputs used)
  - Single vendor simplifies TDM formatting, clock distribution, and register configuration
  - TDM/I2S serial audio interface
  - I2C control interface (2 addresses via single CAD pin; TCA9548A I2C mux on main board isolates each codec board — see [AK4619VN Wiring](ak4619-wiring.md))
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

------

## Analog Stage Design

### Input Stage (per channel)

- **Input impedance:** 10 kΩ (standard for line-level synth/electronic instrument outputs)
- **Input voltage range:** ±5V peak (accommodates Eurorack, +4 dBu pro gear, and consumer -10 dBV)
- **Topology:** Non-inverting buffer with adjustable attenuation
  - Op-amp: OPA1678 (4.5 nV/√Hz, rail-to-rail output, low distortion)
  - Input resistor divider or trim to scale hot signals into ADC full-scale range
  - AC-coupled input (10 µF + 10 kΩ gives ~1.6 Hz HPF, blocks DC offsets from synths)
- **Anti-alias filter:** 2nd-order Sallen-Key low-pass, fc ≈ 22 kHz, Butterworth alignment
  - Placed between input buffer output and ADC input pin
  - Component values: ~3.3 nF + ~6.8 nF, ~1 kΩ + ~1 kΩ (adjust for exact fc)
  - Provides ~40 dB/decade rolloff above Nyquist at 48 kHz
- **ESD protection:** Schottky clamp diodes to rails on all input jacks

### Output Stage (per channel)

- **Topology:** Active reconstruction filter + line driver
  - Op-amp: OPA1678 (same as input, keeps BOM simple)
  - 2nd-order Sallen-Key low-pass, fc ≈ 22 kHz (reconstruction / smoothing filter)
- **Output impedance:** ~100 Ω series resistor (cable drive, short-circuit protection)
- **Output level:** 0 dBu nominal (+4 dBu max before clipping)
- **DC blocking:** AC coupling cap on output (47 µF, keeps DC offset from reaching downstream gear)
- **Pop suppression:** TS5A3159 analog switch per output pair (SPST, SOT-23-5)
  - GPIO-controlled by Teensy — firmware opens switches during boot/shutdown ramp for silent transitions
  - 4× switches total (Main L/R, AUX1 L/R, AUX2 L/R, AUX3 L/R — one IC per stereo pair)
  - Low Ron (~1Ω), 5V tolerant, SOT-23-5 footprint

### Op-Amp Selection Rationale

| Parameter      | TL072 (original) | OPA1678 (recommended) | NJM4580 (alternate) |
| -------------- | ----------------- | --------------------- | ------------------- |
| Noise density  | 18 nV/√Hz        | 4.5 nV/√Hz           | 8 nV/√Hz           |
| THD+N          | 0.003%            | 0.00005%              | 0.001%              |
| GBW            | 3 MHz             | 24 MHz                | 15 MHz              |
| Supply range   | ±3.5–18V          | 4.5–36V               | ±2–18V              |
| Package        | DIP-8 / SOIC-8    | SOIC-8 / TSSOP-8      | DIP-8 / SOIC-8      |
| Cost (approx)  | $0.50             | $1.20                 | $0.80               |

- OPA1678 recommended for all stages (input buffer, anti-alias, reconstruction, output driver)
- NJM4580 acceptable as lower-cost alternate for DIY builders on a budget
- TL072 not recommended: noise floor incompatible with >100 dB dynamic range target

------

## Power System

### Power Input

**Dual USB-C (separate power and data):**

- **PWR — USB-C (power only, USB PD 5V/5A):**
  - PCB-mount USB-C receptacle (mid-mount or through-hole)
  - STUSB4500 USB PD sink controller — negotiates 5V @ 5A from PD-capable supplies
  - Fallback: 5.1kΩ CC resistors still present for non-PD supplies (defaults to 5V/3A)
  - VBUS and GND connected; D+/D- not routed — power only
  - STUSB4500 configured via I2C or NVM to request 5V PDO only (no higher voltages)
  - Labeled "PWR" on back panel (right side)
- **PC — USB-C (data only):**
  - Second USB-C breakout or PCB-mount receptacle
  - D+/D- routed to Teensy 4.1 native USB device port
  - VBUS not used for system power (only for USB signaling / pull-ups as needed)
  - Carries USB Audio (2-in/2-out UAC1) + USB MIDI (composite device)
  - Labeled "PC" on top panel
  - Ground connected to system GND through ferrite bead to reduce computer-injected noise

### Power Budget (5V rail)

- **USB host ports:** 2× 500 mA = 1.0 A
- **NeoPixels (16 keys):** ~320 mA typical (at 30% cap), 960 mA worst-case (uncapped)
- **TFT display (4.3" RA8875):** ~250-350 mA (backlight dependent)
- **Teensy + logic:** ~200 mA
- **Audio analog stages:** ~200-300 mA
- **Worst-case total:** ~2.81 A (with uncapped NeoPixels)
- **With 20% reserve:** ~3.37 A
- **Supply target:** 5V @ 5A via USB PD (headroom for uncapped NeoPixels + builder modifications)

### Power Distribution

**5V Rail Partitioning:**

- 5V_DIG: USB hub VBUS + NeoPixels + TFT backlight (noisy loads)
- 5V_A: Dedicated low-noise LDO for audio analog stages

**Audio Analog Power:**

- **LDO regulator:** ADP7118 — 5V input → 3.3V_A clean analog rail (3 instances total)
  - Ultra-low noise: <10 µV RMS (vs. ferrite bead approach which only filters HF)
  - Each Input Mother Board has its own ADP7118, powered from FFC pin 9 (5V raw)
  - Main Board has a third ADP7118 for the headphone amplifier and virtual ground buffer
  - Separate LDO per board, not shared with digital 3.3V
  - Provides clean supply for op-amps and codec analog sections
  - **Op-amps run single-supply on 5V** with 2.5V virtual ground biasing (no charge pump needed)
  - Virtual ground generated by precision resistor divider (2× 10kΩ 0.1%) buffered by one section of an OPA1678
  - This provides a stable mid-rail reference for AC-coupled signal paths
  - OPA1678 rail-to-rail output gives ~3.5Vpp usable swing — sufficient for +4 dBu (≈3.47Vpp)
  - Simpler power design, no charge pump switching noise to contaminate audio
- **Ferrite bead** still used between 5V_DIG and 5V input to LDO (belt-and-suspenders)

**Soft-Start Circuit:**

- **P-FET load switch** (TPS22965, 5A continuous) on main 5V input after polyfuse
  - Controlled slew rate limits inrush current during power-on
  - Prevents voltage sag on USB-C supplies from bulk cap charging
  - Enable signal can be tied to RC delay or Teensy GPIO for sequenced startup

**Protection:**

- Input polyfuse (2.5A hold / 5A trip) on USB-C VBUS
- Soft-start load switch (see above)
- Per-port current limiting for USB host (TPS2051-class power switches or hub-integrated)
- Bulk capacitor (1000-2200 µF) near NeoPixel power entry
- Local decoupling at every subsystem

**Current Measurement Test Point:**

- Low-side shunt resistor (10 mΩ) on main 5V rail with test points
- Allows builders to verify supply current with a multimeter during bringup

### USB Host Hub

**FE1.1s discrete USB 2.0 hub IC** on main board (~$1.50):

- 1 upstream port connected to Teensy 4.1 USB host pins (D+/D-)
- 2 downstream ports routed to panel-mount USB-A sockets (MIDI host)
- External 12 MHz crystal + 2× 15 pF load capacitors
- Per-port VBUS power switching via TPS2051 load switches (500 mA per port)
- Self-powered configuration (VBUS from 5V_DIG rail)
- Minimal external components: crystal, caps, pull-up/pull-down resistors per datasheet
- Overcurrent protection per port (TPS2051 fault output routed to Teensy GPIO for firmware notification)

### Headphone Amplifier

**TPA6132A2** dedicated headphone driver IC:

- Ground-referenced output — no AC coupling capacitors required on headphone jack
- 25 mW into 32Ω load, 0.01% THD+N typical
- Single 3.3–5V supply (powered from 5V_A rail)
- Stereo differential input from DAC outputs (Master L/R from U1 DAC)
- Output routed through 10kΩ log potentiometer (volume control) to 1/4" TRS headphone jack
- Headphone detect switch on TRS jack wired to Teensy GPIO (allows firmware to mute main outputs when headphones inserted)

### MIDI IN Circuit

**3.5mm TRS Type A** input with **6N138** optocoupler isolation:

- Input side: 220Ω series resistor + 1N4148 protection diode across optocoupler LED
- TRS wiring (Type A standard): Tip = current sink (pin 5), Ring = current source (pin 4), Sleeve = shield
- Optocoupler output: open-collector, pulled up to 3.3V via 470Ω resistor
- Output connects to Teensy UART RX pin for serial MIDI at 31.25 kbaud
- 6N138 is industry standard, proven at MIDI baud rate, universal reference schematics available
- 3.5mm TRS is compact and modern; legacy 5-pin DIN gear connects via TRS-to-DIN adapter cable

### MIDI OUT Circuit

**3.5mm TRS Type A** output (same connector type as MIDI IN):

- TRS wiring (Type A standard): Tip = current sink (pin 5), Ring = current source (pin 4), Sleeve = shield
- 3.3V → 10Ω → TRS Ring (source); TRS Tip → 33Ω → Serial4 TX (pin 17)
- Standard MIDI current-loop output at 31.25 kbaud
- No optocoupler needed on output side (optocoupler is on the receiving device)
- Software handles pass-through of MIDI IN messages and/or Teensy-generated MIDI output

### Soft-Latch Power Circuit

**SR latch** from discrete components + Teensy GPIO override:

- **Power on:** Momentary button press sets latch → P-FET load switch (TPS22965) enables 5V rail → Teensy boots
- **Power off (clean):** Button press detected by Teensy GPIO → firmware saves state to SD → Teensy asserts shutdown GPIO → latch resets → power off
- **Power off (hard):** Long-press (>4 seconds) triggers RC timeout that resets latch directly, bypassing firmware — emergency fallback if firmware hangs
- **Implementation:** 74LVC1G00 NAND gate (SOT-23-5) wired as SR latch, RC network for long-press timeout, debounce cap on button input
- Teensy KEEP_ALIVE GPIO holds latch set during normal operation; releasing it allows clean shutdown

------

## PCB Layer Stackup

| Board | Layers | Stackup | Rationale |
|-------|--------|---------|-----------|
| Main Board | **4-layer** | Sig / GND / PWR / Sig | TDM clock integrity (24.576 MHz BCLK), mixed-signal ground plane, power distribution |
| Input Mother Board | **4-layer** | Sig / GND / PWR / Sig | AK4619VN codecs + analog input stages + TDM signals need solid ground reference |
| Input Daughter Board | **2-layer** | Sig / GND | Simple board: jacks + ESD diodes + connector, no high-speed signals |
| Output Board | **2-layer** | Sig / GND | Simple board: jacks + ESD diodes + connector, no high-speed signals |
| Key PCB | **2-layer** | Sig / GND | Switches + LEDs only, no high-speed signals |

------

## Noise Mitigation Strategy

### Power Integrity

1. **Bulk capacitor (1000-2200 µF)** across 5V/GND near NeoPixels (buffers current spikes from up to 960 mA worst-case draw)
2. **Ferrite bead + decoupling** between 5V_DIG and 5V_A (isolates audio analog)
3. **Local decoupling** at every IC and subsystem boundary (0.1 µF ceramic + bulk)
4. **NeoPixel series resistor (300-500 Ω)** in data line near first pixel (reduces ringing)

### Grounding

- **Single continuous ground plane** on PCB (don't hard-split analog/digital grounds)
- **Star topology** for high-current returns (USB, NeoPixels, TFT backlight) near power entry
- **Keep audio ground references tight** around codec and output jacks
- **Separate return paths** for noisy digital and sensitive analog currents
- **Component placement strategy:** Power entry + USB hub on one end of board → Teensy + UI (display, encoders, keys) in center → audio codecs + analog stages + jacks on opposite end. This ensures high-current digital return paths don't cross under the audio section of the ground plane

### Layout

- **Physical separation:** USB hub/switching away from audio codecs and outputs
- **Short audio traces:** minimize analog signal path length
- **Keep digital noise away:** TFT, USB, NeoPixel data lines routed away from audio
- **Shielding (optional):** ground pour between noisy and analog sections

### Firmware-Based Noise Mitigation

- **Cap NeoPixel global brightness** (30% default recommended — reduces noise and power; hardware is sized for uncapped operation in case builders remove the limit)
- **Smooth parameter changes** (no abrupt steps in gain/pan)
- **USB host power management** (ability to power-cycle ports via software)

------

## Bill of Materials (Key Components)

### Core Processing

| Part                  | Quantity | Notes                                          |
| --------------------- | -------- | ---------------------------------------------- |
| Teensy 4.1            | 1        | ARM Cortex-M7, USB host, TDM audio, SD slot    |
| AK4619VN              | 4        | 4-in/4-out codec; U1-U2 full, U3-U4 ADC only  |
| PSRAM (8 MB, QSPI)   | 1        | IPS6404LSQ or APS6404L; solder to Teensy bottom pads |

### UI Components

| Part                          | Quantity | Notes                                        |
| ----------------------------- | -------- | -------------------------------------------- |
| 4.3" 480×272 TFT w/ RA8875   | 1        | SPI, hardware drawing engine, PJRC RA8875_t4 |
| Rotary encoder with push      | 3        | Quadrature, interrupt-capable pins; NavX + NavY + Edit |
| Custom key PCB                | 1–2     | CHOC hotswap sockets + WS2812B-2020 + MCP23017 + 100nF caps; 4×4 grid |
| Kailh CHOC hotswap sockets    | 16       | Soldered to custom PCB                        |
| WS2812B-2020 NeoPixels        | 16       | Daisy-chained, single data pin                |
| MCP23017 I2C GPIO expander    | 1        | 4×4 key scan matrix on Key PCB; I2C address 0x20 |
| CHOC v2 key switches          | 16       | User-supplied or included in BOM              |

### Power & Connectivity

| Part                            | Quantity | Notes                             |
| ------------------------------- | -------- | --------------------------------- |
| TCA9548A I2C mux                | 1        | I2C bus switch on main board; isolates codec boards; address 0x70 |
| USB-C receptacle (PCB-mount)    | 1        | PWR port — power only, back panel             |
| STUSB4500 USB PD sink controller | 1       | Negotiates 5V/5A; fallback 5V/3A via CC resistors |
| USB-C receptacle (PCB-mount)    | 1        | PC port — data only (audio+MIDI)  |
| FE1.1s USB 2.0 hub IC           | 1        | Discrete IC, 12 MHz crystal + caps, per-port VBUS via TPS2051 |
| 12 MHz crystal                  | 1        | FE1.1s clock source, 15 pF load caps             |
| 6N138 optocoupler               | 1        | MIDI IN galvanic isolation, 31.25 kbaud           |
| 1N4148 signal diode             | 1        | MIDI IN optocoupler LED protection                |
| TPS2051 power switch             | 2        | USB host port current limiting    |
| USB-A panel-mount socket        | 2        | Host ports for MIDI controllers   |
| 3.5mm TRS jack (MIDI OUT)       | 1        | MIDI OUT Type A; top panel        |
| Resistor 33 ohm (MIDI OUT)      | 1        | MIDI OUT series resistor (Tip to Serial4 TX) |
| Resistor 10 ohm (MIDI OUT)      | 1        | MIDI OUT source resistor (3.3V to Ring)      |

### Audio I/O & Analog

| Part                          | Quantity | Notes                                          |
| ----------------------------- | -------- | ---------------------------------------------- |
| 1/4" TS panel jacks           | 24       | 16 inputs + 8 outputs                          |
| 1/4" TRS panel jack           | 1        | Headphone output (stereo), top panel            |
| Potentiometer (10 kΩ log)     | 1        | Headphone volume, dedicated analog pot, top panel |
| OPA1678 (dual op-amp, SOIC-8) | 16       | Input buffers (8), anti-alias (8), output (4), reconstruction (4) — estimate, refine in schematic |
| NJM4580 (alternate)           | 16       | Budget-friendly alternate to OPA1678            |
| TS5A3159 analog switch (SOT-23-5) | 4    | Pop suppression — 1 per output stereo pair (Main, AUX1-3) |
| TPA6132A2 headphone amplifier | 1        | Ground-referenced stereo HP driver, 25mW/32Ω   |

### Power Regulation

| Part                           | Quantity | Notes                                    |
| ------------------------------ | -------- | ---------------------------------------- |
| ADP7118 LDO                    | 3        | 5V → 3.3V_A ultra-low-noise analog rail; 1 per Input Mother Board (2) + 1 on Main Board (HP amp + virtual ground) |
| TPS22965 load switch             | 1       | Soft-start / inrush limiting, 5A continuous |
| 10 mΩ shunt resistor           | 1        | Current measurement test point           |

### Passives & Protection

| Part                     | Quantity | Notes                                      |
| ------------------------ | -------- | ------------------------------------------ |
| 1000-2200 µF electrolytic | 2+       | NeoPixel bulk, power entry                 |
| 0.1 µF ceramic caps      | 60+      | Local decoupling everywhere (incl. op-amps)|
| Ferrite beads            | 2-3      | 5V_DIG to LDO input isolation              |
| Resistors (300-500 Ω)    | 1        | NeoPixel data series resistor              |
| Polyfuse (2.5A/5A)       | 1        | USB-C input protection                     |
| 1N4148 signal diode (key matrix) | 16 | Anti-ghosting diodes for 4×4 key matrix on Key PCB |
| Schottky clamp diodes    | 25       | ESD protection on all audio jacks (incl. headphone) |
| Film caps (3.3nF, 6.8nF) | 48+     | Anti-alias & reconstruction filter caps    |
| Resistors (1 kΩ, misc)   | 60+     | Filter networks, input dividers            |

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

Left zone:
- 1× TFT display (4.3" RA8875, ~93×56 mm visible area): upper left
- 3× rotary encoders (NavX + NavY + Edit): horizontal row below display

Right zone:
- 16× CHOC key switches (4×4 grid): left portion of right zone, top-aligned with display
- Right-side 2-wide column: Vol + Power / Phones + PC USB-C / SD card + MIDI HOST / MIDI TH. + MIDI IN

**Back panel (260 × 50 mm — all audio I/O):**

- 24× 1/4" TS jacks: 12 evenly spaced stereo pairs (L top row, R bottom row), 20 mm center-to-center
- Order left to right (looking at back): Master, AUX1, AUX2, AUX3, 15/16, 13/14, 11/12, 9/10, 7/8, 5/6, 3/4, 1/2
- No physical gaps between output and input groups — separation by labeling only
- PWR USB-C (power only, 5V/5A USB PD): right side of back panel

**No front, left, or right side panels with connectors.** All I/O accessible from top and back.

**Validate** with 1:1 panel template printout during Phase 1 before committing to enclosure fabrication. See [`mixtee-layout.svg`](../hardware/mixtee-layout.svg) for detailed panel drawings.

### User Experience

- **Boot Time:** <3 seconds to ready
- **UI Responsiveness:** <50 ms encoder/button to visual feedback
- **MIDI Latency:** <10 ms CC message to parameter change
- **Hot-plug:** MIDI controllers safe to connect/disconnect during operation
