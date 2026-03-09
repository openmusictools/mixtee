# USB Audio Interface (Removed)

*← Back to [README](../README.md) | See also: [Network Connectivity](network-connectivity.md) · [Hardware](hardware.md)*

------

## Status: XMOS XU216 Removed

The XMOS XU216 USB audio bridge has been removed from the MIXTEE design. DAW connectivity now uses **AES67 network audio over Ethernet** (16-in / 8-out via the Teensy 4.1's built-in DP83825I PHY).

This simplifies the Main Board design by eliminating:

- XMOS XU216-256-TQ128-C20 (128-pin TQFP)
- W25Q32 QSPI flash (firmware storage)
- AP2112K-1.0 LDO (1.0V core supply)
- 24 MHz + 24.576 MHz crystals
- USBLC6-2 ESD protection
- PC USB-C receptacle
- SPI0 control bus (pins 11–13 now spare; pins 9–10 reassigned to ESP32 boot control)
- 9-signal TDM passive tap

**Freed Teensy pins:** 9, 10, 11, 12, 13 (5 pins returned to spare pool). Pins 9–10 subsequently assigned to ESP32_EN and ESP32_GPIO0 for SD card update reflash — see [pin-mapping.md](pin-mapping.md). Pins 11–13 remain spare.

## DAW Connectivity via Ethernet

See [network-connectivity.md §9](network-connectivity.md) for:

- Stream layout (2× TX streams + 1× RX stream)
- Full AES67 compliance (SDP, SAP, PTP, RTP)
- Recommended virtual soundcards per OS
- Firmware impact summary

## Legacy / Fallback

If a future revision requires USB audio, the Teensy's native USB can provide:

- **UAC1 stereo:** 2-in / 2-out, 16-bit 44.1 kHz (zero cost, firmware only)
- **UAC2 firmware (alex6679):** Up to 8 channels on macOS/Linux; Windows capped at 8ch without commercial driver

See [alex6679/teensy-4-usbAudio](https://github.com/alex6679/teensy-4-usbAudio) for the community UAC2 implementation.
