# IO Board — Architecture

## USB Host Hub

- FE1.1s discrete USB 2.0 hub IC (~$1.50, SSOP-28)
- 1 upstream port from Teensy USB host pins (D+/D-) via Main-IO FFC
- 2 downstream ports to panel-mount USB-A sockets
- External 12 MHz crystal + 2x 15 pF load capacitors
- Self-powered configuration (VBUS from 5V_DIG via FFC)
- Per-port VBUS power switching: 2x TPS2051 load switches (500 mA each)
- TPS2051 fault output routed to Teensy GPIO for firmware OC notification
- Minimal external components: crystal, caps, pull-up/pull-down resistors per datasheet

## MIDI IN Circuit

- 3.5mm TRS Type A input with 6N138 optocoupler (DIP-8)
- Input side: 220-ohm series resistor + 1N4148 protection diode across LED
- TRS wiring (Type A): Tip = current sink (pin 5), Ring = current source (pin 4), Sleeve = shield
- Optocoupler output: open-collector, pulled up to 3.3V via 470-ohm
- Output routes via FFC pin 9 (MIDI_RX) to Teensy Serial3 RX (pin 15) at 31.25 kbaud
- 3.5mm TRS compact and modern; legacy 5-pin DIN connects via adapter cable

## MIDI OUT Circuit

- 3.5mm TRS Type A output
- TRS wiring (Type A): Tip = current sink (pin 5), Ring = current source (pin 4), Sleeve = shield
- MIDI_TX arrives from Teensy Serial4 TX (pin 17) via FFC pin 10
- Resistor network: 3.3V -> 10-ohm -> TRS Ring (source); TRS Tip -> 33-ohm -> MIDI_TX
- Standard MIDI current-loop output at 31.25 kbaud
- No optocoupler on output side (receiver's responsibility)

## Ethernet Section

- RJ45 MagJack with integrated magnetics + optional activity LEDs
- 0.1uF coupling caps between Teensy PHY signals and MagJack transformer
- Signals arrive via 6-pin ribbon cable (NOT the FFC)
- Post-PHY analog signals -- no impedance-controlled routing required
- Native Teensy 4.1 Ethernet (DP83825I PHY on board), 10/100 Mbps

## Headphone Amplifier Integration

- Off-the-shelf TPA6132 or MAX97220 breakout module (~$2-5)
- Ground-referenced output (no AC coupling caps needed on jack)
- 25 mW into 32-ohm, 0.01% THD+N typical
- Single 3.3-5V supply (from 5V_A via Main Board)
- HP_L/HP_R arrive via short wires from Main Board, NOT via FFC
- Output through 10k-ohm log pot (volume) to headphone TRS jack
- Headphone detect switch wired to Teensy GPIO pin 39

## Design Notes

- 2-layer PCB sufficient -- only USB Full-Speed (12 Mbps) and post-PHY Ethernet analog
- No high-speed digital traces on this board
- Functional zones: USB section (top), MIDI section (center), volume/headphone (bottom), FFC/header (interior edge)

---
Back to [README.md](README.md)
