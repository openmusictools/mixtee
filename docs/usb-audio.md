# USB Audio Interface — Optional Upgrades

MIXTEE ships with basic 2-in/2-out USB audio over the Teensy 4.1's built-in USB (UAC1, 16-bit 44.1kHz). This document covers optional paths to expand that capability.

## Current: Stereo USB Audio (no extra hardware)

Teensy 4.1 presents as a USB Audio Class 1 device over the dedicated PC USB-C port (data only). Firmware routes any stereo pair (typically master bus) to/from the DAW. The PC sees MIXTEE as a 2-in/2-out sound card. Works alongside USB MIDI on the same connection (composite device). Power is supplied separately via the PWR USB-C port, isolating the mixer from computer power rail noise.

## Option 1: USB Audio Class 2 (firmware only)

Community-maintained UAC2 implementations for Teensy 4.x support up to 8–16 input channels over USB. No extra hardware — just firmware complexity. macOS and Linux handle UAC2 natively; Windows needs ASIO4ALL or a vendor driver. Best suited for multitrack input only (sending individual channels to the DAW).

## Option 2: XMOS Companion Chip

A dedicated XMOS chip (e.g. XU208) sits between the Teensy's TDM bus and the USB port, handling class-compliant multi-channel USB audio. This is what most commercial interfaces use. Adds ~$10–15 in parts. Gives rock-solid 16-in class-compliant USB audio with no driver headaches. The Teensy handles all mixing/DSP; the XMOS acts purely as a USB audio bridge.

## Option 3: Hybrid (Stereo USB + ADAT)

Stereo USB audio for the master mix over the existing USB-C connection, plus an ADAT optical output for multitrack. The PC side uses any ADAT-capable interface (Behringer ADA8200, Focusrite with ADAT, etc.) to receive all channels individually. Requires adding a TOSLINK optical output and ADAT transmitter circuitry (~$5–8 in parts). Both paths can run simultaneously.

## Option 4: Network Audio

Stream audio over Ethernet using the Teensy 4.1's built-in Ethernet port. Lightweight protocols exist but DAW support is limited without commercial Dante licensing. Overkill for this project unless targeting a specific networked audio workflow.

## Recommendation

Start with the built-in stereo USB audio (current plan). If multitrack into DAW becomes a priority, Option 1 (UAC2 firmware, input-only) is the lowest-cost path. Option 2 (XMOS) is the most robust if budget allows.
