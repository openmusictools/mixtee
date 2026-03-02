# Input Mother Board — Architecture

*← Back to [README](README.md)*

This document covers the detailed analog circuit design, power regulation, I2C addressing, and BOM variant strategy for the Input Mother Board.

------

## Analog Input Stage (per channel)

- **Input impedance:** 10 kOhm
- **Input voltage range:** +/-5V peak (Eurorack, +4 dBu, -10 dBV)
- **Topology:** Non-inverting buffer with adjustable attenuation
  - Op-amp: OPA1678 (4.5 nV/sqrt(Hz), rail-to-rail output)
  - AC-coupled input (10uF + 10kOhm -> ~1.6 Hz HPF)
- **Anti-alias filter:** 2nd-order Sallen-Key LP, fc ~ 22 kHz, Butterworth
  - Component values: ~3.3 nF + ~6.8 nF, ~1 kOhm + ~1 kOhm
  - ~40 dB/decade rolloff above Nyquist at 48 kHz
- **ESD protection:** Schottky clamp diodes to rails on all input jacks

------

## Output Analog Stage (Board 1-top only, per channel)

- **Topology:** Active reconstruction filter + line driver
  - Op-amp: OPA1678
  - 2nd-order Sallen-Key LP, fc ~ 22 kHz
- **Output impedance:** ~100 Ohm series resistor
- **Output level:** 0 dBu nominal (+4 dBu max)
- **DC blocking:** 47uF AC coupling cap
- **Pop suppression:** TS5A3159 on Main Board (not on this board)

------

## Op-Amp Selection

| Parameter | OPA1678 (recommended) | NJM4580 (alternate) |
|-----------|----------------------|---------------------|
| Noise density | 4.5 nV/sqrt(Hz) | 8 nV/sqrt(Hz) |
| THD+N | 0.00005% | 0.001% |
| GBW | 24 MHz | 15 MHz |
| Supply range | 4.5–36V | +/-2–18V |
| Package | SOIC-8 / TSSOP-8 | DIP-8 / SOIC-8 |
| Cost | $1.20 | $0.80 |

Op-amps run single-supply on 5V with 2.5V virtual ground biasing (generated on Main Board). OPA1678 rail-to-rail output gives ~3.5Vpp usable swing — sufficient for +4 dBu.

------

## Power: ADP7118 LDO

- Each Input Mother Board has its own ADP7118 (5V -> 3.3V_A)
- Ultra-low noise: <10 uV RMS
- Powers both AK4619VN codec AVDD/TVDD pins
- 5V raw supply arrives from FFC pin 9
- Ferrite bead between 5V_DIG (FFC pin 8) and 5V raw input

------

## I2C Address Selection

- Solder jumpers select TCA9548A mux channel (Ch 0 = Board 1-top, Ch 1 = Board 2-top)
- U1/U3: CAD pin low -> 0x10; U2/U4: CAD pin high -> 0x11
- Same PCB design for both instances

------

## BOM Variant

- **Board 1-top:** fully populated (8x input buffers, 8x anti-alias filters, 4x output reconstruction filters, 4x line drivers)
- **Board 2-top:** output section left unpopulated (DAC sections of U3/U4 unused)
