# MIXTEE: Connector Part Numbers

*← Back to [README](../README.md) | See also: [PCB Architecture](pcb-architecture.md) · [Hardware](hardware.md) · [Enclosure](enclosure.md)*

------

## Purpose

Every connector needs an exact manufacturer part number (MPN) for KiCad footprint selection and BOM generation. This document tracks connector selections across all MIXTEE PCBs.

------

## Connector Selections

### USB Connectors

| Connector | MPN | Manufacturer | Package | Board | Notes |
|-----------|-----|-------------|---------|-------|-------|
| USB-C receptacle (PWR) | — (on breakout module) | — | Module-included | Power Module (off-the-shelf) | Power only; STUSB4500 breakout handles USB PD; back panel |
| USB-C receptacle (PC) | USB4105-GF-A | GCT | Mid-mount SMD | Main Board | Data only (D+/D- to Teensy USB device); top panel; same part for BOM consolidation |
| USB-A dual stacked | 67298-4090 | Amphenol | Through-hole | IO Board | 2× USB-A host ports for MIDI controllers; top panel |

### FFC / FPC Connectors

| Connector | MPN | Manufacturer | Package | Board | Notes |
|-----------|-----|-------------|---------|-------|-------|
| FFC 16-pin ZIF (bottom contact) | 5025861690 | Molex | SMD, 1.0mm pitch | Main Board (×2), Input Mother (×2) | ZIF latch, bottom-contact, for 16-pin 1.0mm FFC cable |
| FFC 12-pin ZIF (bottom contact) | TBD (Molex 502586 series) | Molex | SMD, 1.0mm pitch | Main Board (×1), IO Board (×1) | ZIF latch, bottom-contact, for 12-pin 1.0mm Main↔IO FFC cable |

### JST-PH Connectors

| Connector | MPN | Manufacturer | Package | Board | Notes |
|-----------|-----|-------------|---------|-------|-------|
| JST-PH 6-pin header | B6B-PH-K-S | JST | Through-hole, 2.0mm pitch | Main Board, Key PCB, Mother/Daughter boards | Key PCB cable + mother↔daughter interconnects |
| JST-PH 10-pin header | B10B-PH-K-S | JST | Through-hole, 2.0mm pitch | Board 1-top, Board O-top | Output analog cable (8 signals + 2 GND) |

### Audio Connectors

| Connector | MPN | Manufacturer | Package | Board | Notes |
|-----------|-----|-------------|---------|-------|-------|
| 1/4" TS jack (panel mount) | 112BPC | Switchcraft | PCB-mount through-hole | Input Mother, Input Daughter, Output boards | 24× total (16 input + 8 output); 12.7mm mounting hole |
| 1/4" TRS jack (headphone) | 35RASMT2BHNTRX | Switchcraft | PCB-mount through-hole | IO Board | Stereo headphone output with detect switch; top panel |
| 3.5mm TRS jack (MIDI IN) | SJ-3523-SMT | CUI Devices | SMD | IO Board | MIDI IN Type A; top panel |
| 3.5mm TRS jack (MIDI OUT) | SJ-3523-SMT | CUI Devices | SMD | IO Board | MIDI OUT Type A; top panel; same part as MIDI IN |

### SD Card

| Connector | MPN | Manufacturer | Package | Board | Notes |
|-----------|-----|-------------|---------|-------|-------|
| Full-size SD card socket | 472192001 | Molex | Through-hole / SMD | Main Board | Panel-mount; SDIO routed from Teensy bottom pads 42-47 |

### User Interface

| Connector | MPN | Manufacturer | Package | Board | Notes |
|-----------|-----|-------------|---------|-------|-------|
| Rotary encoder (with push) | PEC11R-4215F-S0024 | Bourns | Through-hole | Main Board (×3) | 24 detents, quadrature, push switch; NavX + NavY + Edit |
| Power button (momentary) | — | TBD | Panel mount | Main Board | Momentary push button for soft-latch; select during enclosure design |

### Key PCB

| Connector | MPN | Manufacturer | Package | Board | Notes |
|-----------|-----|-------------|---------|-------|-------|
| Kailh CHOC hotswap socket | CPG135001S30 | Kailh | SMD | Key PCB (×16) | CHOC v1/v2 compatible |

------

## Cable Assemblies

| Cable | Type | Length | Pins | Notes |
|-------|------|--------|------|-------|
| Main ↔ Input Mother (×2) | FFC 1.0mm pitch | 40-50mm | 16 | Pre-made, bottom-contact |
| Main ↔ IO Board | FFC 1.0mm pitch | 30-40mm | 12 | Pre-made, bottom-contact |
| Mother ↔ Daughter (×3) | JST-PH wire harness | 15-20mm | 6 | Pre-crimped |
| Board 1-top → Board O-top | JST-PH wire harness | ~80mm | 10 | Output analog signals |
| Main ↔ Key PCB | JST-PH wire harness | 30-40mm | 6 | NeoPixel + I2C + INT + power |

------

## KiCad Footprint Status

| Part | KiCad Library Footprint | Status |
|------|------------------------|--------|
| USB4105-GF-A | Connector_USB:USB_C_Receptacle_GCT_USB4105 | Verify in KiCad 9 library |
| 67298-4090 | Connector_USB:USB_A_Amphenol_67298-4090_Dual | Verify or create |
| 5025861690 | Connector_FFC-FPC:Molex_5025861690 | Verify or create |
| TBD (12-pin ZIF) | Connector_FFC-FPC:Molex_502586_12pin | Select MPN + verify or create |
| B6B-PH-K-S | Connector_JST:JST_PH_B6B-PH-K-S_1x06_P2.00mm_Vertical | Available in KiCad |
| B10B-PH-K-S | Connector_JST:JST_PH_B10B-PH-K-S_1x10_P2.00mm_Vertical | Available in KiCad |
| 112BPC | Custom — Switchcraft 112BPC | Must create |
| SJ-3523-SMT | Connector_Audio:CUI_SJ-3523-SMT | Verify in KiCad 9 library |
| 472192001 | Connector_Card:SD_Molex_472192001 | Verify or create |
| PEC11R-4215F-S0024 | Connector:Bourns_PEC11R | Verify or create |
| CPG135001S30 | Custom — Kailh CHOC hotswap | Community library available |
