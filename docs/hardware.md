# MIXTEE: Hardware

*← Back to [README](../README.md) | See also: [Features](features.md) · [Firmware](firmware.md) · [UI Architecture](ui-architecture.md)*

------

## Audio Codec Architecture

- **Codec:** 4× AK4619VN (4-in / 4-out per chip, unified vendor)
  - Total capacity: 16 ADC channels + 16 DAC channels (8 DAC outputs used)
  - Single vendor simplifies TDM formatting, clock distribution, and register configuration
  - TDM/I2S serial audio interface
  - I2C control interface (4 unique addresses via pin strapping)
  - 48 kHz / 24-bit target
- **Digital Interface:** TDM over I2S to Teensy 4.1

## TDM Bus Assignment

- **TDM1 (SAI1):** Codecs U1 + U2 (ADC channels 1-8, DAC channels 1-8)
  - U1: TDM slots 0-3 (inputs 1-4), slots 0-3 (outputs 1-4: Main L/R + Aux1 L/R)
  - U2: TDM slots 4-7 (inputs 5-8), slots 4-7 (outputs 5-8: Aux2 L/R + Aux3 L/R)
- **TDM2 (SAI2):** Codecs U3 + U4 (ADC channels 9-16, DAC unused)
  - U3: TDM slots 0-3 (inputs 9-12)
  - U4: TDM slots 4-7 (inputs 13-16)
  - DAC sections on U3/U4 unused (available for future expansion)
- **Clock:** Teensy generates MCLK (256fs = 12.288 MHz), BCLK, and LRCLK as TDM master
  - All four codecs slave to shared clock tree
  - BCLK = 48 kHz × 16 slots × 32 bits = 24.576 MHz per TDM bus
- **Note:** Map and verify slot assignments during Phase 1 breadboard bringup before PCB layout

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
- **Pop suppression:** Mute relay or analog switch on outputs, controlled by firmware during boot/shutdown ramp

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

- **PWR — USB-C (power only):**
  - Adafruit Sunken USB Type-C Breakout Board (product 6050)
  - 5.1k CC resistors for 5V sink negotiation
  - Accepts 5V @ up to 3A from compliant USB-C power supplies
  - VBUS and GND connected; D+/D- not routed — power only
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
- **NeoPixels (12 keys):** ~240 mA typical, 720 mA worst-case (capped in firmware)
- **TFT display (4.3" RA8875):** ~250-350 mA (backlight dependent)
- **Teensy + logic:** ~200 mA
- **Audio analog stages:** ~200-300 mA
- **Total target:** 5V @ 3A supply

### Power Distribution

**5V Rail Partitioning:**

- 5V_DIG: USB hub VBUS + NeoPixels + TFT backlight (noisy loads)
- 5V_A: Dedicated low-noise LDO for audio analog stages

**Audio Analog Power:**

- **LDO regulator:** ADP7118 (or TPS7A47) — 5V input → 3.3V_A clean analog rail
  - Ultra-low noise: <10 µV RMS (vs. ferrite bead approach which only filters HF)
  - Separate LDO for audio section, not shared with digital 3.3V
  - Provides clean supply for op-amps and codec analog sections
  - If ±5V analog rails needed (for wider headroom op-amp stages): use a charge pump inverter (e.g., LM2776 or TPS60403) to generate -5V from +5V, or run op-amps single-supply on the 5V_A rail with virtual ground biasing
- **Ferrite bead** still used between 5V_DIG and 5V input to LDO (belt-and-suspenders)

**Soft-Start Circuit:**

- **P-FET load switch** (e.g., TPS22918 or equivalent) on main 5V input after polyfuse
  - Controlled slew rate limits inrush current to <1A during power-on
  - Prevents voltage sag on cheap USB-C supplies from bulk cap charging
  - Enable signal can be tied to RC delay or Teensy GPIO for sequenced startup

**Protection:**

- Input polyfuse (1.5A hold / 3A trip) on USB-C VBUS
- Soft-start load switch (see above)
- Per-port current limiting for USB host (TPS2051-class power switches or hub-integrated)
- Bulk capacitor (500-1000 µF) near NeoPixel power entry
- Local decoupling at every subsystem

**Current Measurement Test Point:**

- Low-side shunt resistor (10 mΩ) on main 5V rail with test points
- Allows builders to verify supply current with a multimeter during bringup

### USB Host Hub

**Internal powered USB 2.0 hub module:**

- 1 upstream to Teensy USB host
- 2 downstream to panel-mount USB-A sockets
- Self-powered design (500 mA per port, per USB 2.0 spec)
- Overcurrent protection per port (prevents brownout from misbehaving controllers)

------

## Noise Mitigation Strategy

### Power Integrity

1. **Bulk capacitor (500-1000 µF)** across 5V/GND near NeoPixels (buffers current spikes)
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

- **Cap NeoPixel global brightness** (10-20% default reduces noise and power)
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
| Rotary encoder with push      | 2        | Quadrature, interrupt-capable pins            |
| Custom key PCB                | 1–2     | CHOC hotswap sockets + WS2812B-2020 + 100nF caps |
| Kailh CHOC hotswap sockets    | 12       | Soldered to custom PCB                        |
| WS2812B-2020 NeoPixels        | 12       | Daisy-chained, single data pin                |
| CHOC v2 key switches          | 12       | User-supplied or included in BOM              |

### Power & Connectivity

| Part                            | Quantity | Notes                             |
| ------------------------------- | -------- | --------------------------------- |
| Adafruit USB-C Breakout (6050)  | 1        | PWR port — 5V sink, power only    |
| USB-C receptacle (PCB-mount)    | 1        | PC port — data only (audio+MIDI)  |
| USB 2.0 hub module/PCB          | 1        | 2+ downstream ports, self-powered |
| TPS2051 (or equiv) power switch | 2        | USB host port current limiting    |
| USB-A panel-mount socket        | 2        | Host ports for MIDI controllers   |

### Audio I/O & Analog

| Part                          | Quantity | Notes                                          |
| ----------------------------- | -------- | ---------------------------------------------- |
| 1/4" TS panel jacks           | 24       | 16 inputs + 8 outputs                          |
| 1/4" TRS panel jack           | 1        | Headphone output (stereo), top panel            |
| Potentiometer (10 kΩ log)     | 1        | Headphone volume, dedicated analog pot, top panel |
| OPA1678 (dual op-amp, SOIC-8) | 16       | Input buffers (8), anti-alias (8), output (4), reconstruction (4) — estimate, refine in schematic |
| NJM4580 (alternate)           | 16       | Budget-friendly alternate to OPA1678            |
| Analog switches (optional)    | TBD      | If implementing assignable I/O later            |

### Power Regulation

| Part                           | Quantity | Notes                                    |
| ------------------------------ | -------- | ---------------------------------------- |
| ADP7118 (or TPS7A47) LDO      | 1        | 5V → 3.3V_A ultra-low-noise analog rail  |
| TPS22918 (or equiv) load switch | 1       | Soft-start / inrush limiting             |
| 10 mΩ shunt resistor           | 1        | Current measurement test point           |

### Passives & Protection

| Part                     | Quantity | Notes                                      |
| ------------------------ | -------- | ------------------------------------------ |
| 500-1000 µF electrolytic | 2+       | NeoPixel bulk, power entry                 |
| 0.1 µF ceramic caps      | 60+      | Local decoupling everywhere (incl. op-amps)|
| Ferrite beads            | 2-3      | 5V_DIG to LDO input isolation              |
| Resistors (300-500 Ω)    | 1        | NeoPixel data series resistor              |
| Polyfuse (1.5A/3A)       | 1        | USB-C input protection                     |
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

- **Dimensions:** 260×84.6×50 mm (W×D×H) — compact desktop format
- **Weight:** ~1-1.5 kg (depending on enclosure material)
- **Power Consumption:** 5V @ 2-3A typical, 3A peak

### Panel Layout (v2.0)

**Enclosure:** 260 × 84.6 × 50 mm (W × D × H). See [enclosure.md](enclosure.md) for full detail.

**Top panel (260 × 84.6 mm — all controls and connectivity):**

Left zone (0–120 mm):
- 2× rotary encoders (Nav + Edit): stacked vertically on far left
- 1× TFT display (4.3" RA8875, ~93×56 mm visible area): right of encoders

Right zone (120–260 mm):
- 12× CHOC key switches (3×4 grid): left portion of right zone, top-aligned with display
- SD card / Volume pot / Headphone jack: vertical column to right of keys
- Top-right corner cluster: Power button, PC (USB-C, data only), MIDI HOST (dual USB-A stacked), MIDI IN (5-pin DIN)

**Back panel (260 × 50 mm — all audio I/O):**

- 24× 1/4" TS jacks: 12 evenly spaced stereo pairs (L top row, R bottom row), 20 mm center-to-center
- Order left to right (looking at back): AUX1, AUX2, AUX3, Master, 15/16, 13/14, 11/12, 9/10, 7/8, 5/6, 3/4, 1/2
- No physical gaps between output and input groups — separation by labeling only
- PWR USB-C (power only, 5V 3A): right side of back panel

**No front, left, or right side panels with connectors.** All I/O accessible from top and back.

**Validate** with 1:1 panel template printout during Phase 1 before committing to enclosure fabrication. See [`mixtee-layout.svg`](../hardware/mixtee-layout.svg) for detailed panel drawings.

### User Experience

- **Boot Time:** <3 seconds to ready
- **UI Responsiveness:** <50 ms encoder/button to visual feedback
- **MIDI Latency:** <10 ms CC message to parameter change
- **Hot-plug:** MIDI controllers safe to connect/disconnect during operation
