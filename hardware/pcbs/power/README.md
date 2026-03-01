# Power Board (Off-the-Shelf Module)

**No custom PCB required.** Uses an off-the-shelf STUSB4500 USB PD breakout module.

## Recommended Modules

- **SparkFun Power Delivery Board - USB-C (Qwiic)** — STUSB4500-based, USB-C in, screw terminal / Qwiic out
- **STEVAL-ISC005V1** — ST official evaluation board
- **Generic STUSB4500 breakout** — widely available on AliExpress/Tindie

## Integration

- Mount module to back panel (USB-C receptacle protrudes through panel cutout)
- Configure NVM for 5V PDO only (no higher voltages) via I2C before installation
- Wire 5V + GND output to Main Board TPS22965 input via 2-pin JST-PH cable (~60-80 mm, 22 AWG)
- Label "PWR" on back panel

## Pin Interface to Main Board

| Wire | Signal | Destination |
|------|--------|-------------|
| + | 5V (VBUS post-module) | Main Board power input (TPS22965 load switch) |
| - | GND | Main Board ground |

## Status

Off-the-shelf solution selected. No gen_pcb.py needed.
