# Display Engine Architecture

**Date:** 2026-03-09

## Summary

Designed and documented the display engine architecture for MIXTEE. The ESP32-S3 becomes a **device-agnostic LVGL widget rendering engine** with no device-specific knowledge. The Teensy is the brain: reads UI definitions from SD card, streams binary widget commands to the display over UART, and handles all firmware updates (including reflashing the ESP32 itself).

## Key Decisions

| Decision | Rationale |
|----------|-----------|
| Device-agnostic display engine | Same ESP32 firmware reusable across MIXTEE, SYNTEE, and future devices |
| COBS-encoded binary protocol at 921600 baud | Unambiguous framing, 15x headroom over meter data rate |
| All widgets created at boot, switched via SET_VISIBLE | Avoids LVGL memory fragmentation |
| Flat widget table (array indexed by ID, max ~128) | O(1) lookup for 30 Hz meter updates |
| ui.json on SD card parsed by Teensy | Decouples UI layout from firmware; UI-only updates without reflash |
| Single SD card update mechanism (FlasherX + esp-serial-flasher) | Non-technical users (musicians) can update by copying files to SD |
| Pins 9/10 for ESP32 EN/GPIO0 bootloader control | Enables Teensy-controlled ESP32 reflash without external tools |
| 6-pin display header (was 4-pin) | Added ESP32_EN + GPIO0 for reflash; spare pins drop 11 → 9 |

## Pin Changes

| Pin | Old | New |
|-----|-----|-----|
| 9 | Spare | ESP32_EN (display reset) |
| 10 | Spare | ESP32_GPIO0 (display boot mode) |

Spare pins: 11 → 9 (pins 11, 12, 13, 14, 30, 35, 37, 38, 39).

## Protocol Summary

| Range | Category | Key commands |
|-------|----------|-------------|
| 0x01-0x07 | Lifecycle | HANDSHAKE, READY, CLEAR_ALL, BOOT_COMPLETE, HEARTBEAT, ACK/NACK |
| 0x10-0x16 | Widget creation | CREATE_WIDGET, DELETE_WIDGET, SET_PROPERTY, SET_VISIBLE, CREATE_PAGE, SWITCH_PAGE |
| 0x20-0x24 | Runtime data | METER_BATCH, SET_VALUE, SET_TEXT, SET_STATE, SET_COLOR |
| 0x30-0x32 | Touch (ESP32 to Teensy) | TOUCH_DOWN, TOUCH_UP, TOUCH_DRAG |
| 0x40-0x41 | Update | ENTER_BOOTLOADER, UPDATE_PROGRESS |

METER_BATCH: 24 meters x 6 bytes = 144 bytes/frame at 30 Hz = ~4.6 KB/s.

## SD Card Update Flow

1. User copies `/UPDATE/` folder (manifest.json, teensy.hex, display.bin, ui.json) to SD card
2. Teensy boots, detects manifest, verifies device ID + checksums
3. Teensy self-updates via FlasherX (writes upper flash first, vector table last)
4. Teensy reflashes ESP32 via esp-serial-flasher (GPIO0 LOW + EN pulse + UART stream, ~16s)
5. Copies ui.json to `/SYSTEM/ui.json`, renames `/UPDATE/` to `/UPDATE_DONE/`
6. Normal boot resumes

## Files Changed

| File | Change |
|------|--------|
| `docs/pin-mapping.md` | Pins 9/10 assigned, spare count 11 to 9 |
| `hardware/pcbs/main/connections.md` | Display header 4-pin to 6-pin |
| `hardware/pcbs/main/architecture.md` | Added ESP32 reflash circuit section, updated GPIO table |
| `docs/firmware.md` | Added FlasherX, esp-serial-flasher, display engine protocol, SD update boot sequence |
| `docs/ui-architecture.md` | Added display engine architecture, widget types, ui.json format |
| `docs/features.md` | Added SD card update feature, updated display section |
| `docs/hardware.md` | Updated display module BOM entry (6-pin header) |
| `docs/system-topology.md` | Updated connector summary (display 4 to 6 pin) |
| `docs/usb-audio.md` | Updated stale pin 9/10 spare references |
| `CLAUDE.md` | Updated architecture summary, libraries, cross-references |

## New Files

| File | Content |
|------|---------|
| `docs/display-engine.md` | Full protocol spec: COBS framing, command set, widget types, property keys, error handling |
| `docs/sd-update.md` | Update mechanism: package format, manifest, boot sequence, FlasherX, esp-serial-flasher, error handling |

## Risks

| Risk | Mitigation |
|------|-----------|
| ESP32 module doesn't expose EN/GPIO0 | Verify before module selection (Waveshare 4.3" does expose these) |
| Power loss during FlasherX | Upper-flash-first write + HalfKay ROM = always USB-recoverable |
| ESP32 reflash too slow | 921600 baud = ~16s for 1.5 MB; NeoPixel progress indicator |
| ui.json parse slows boot | ~5 ms parse + ~200 ms UART stream = <500 ms total |
