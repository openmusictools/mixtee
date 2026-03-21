# MIXTEE: Firmware

*← Back to [README](../README.md) | See also: [Features](features.md) · [Hardware](hardware.md) · [UI Architecture](ui-architecture.md)*

------

## Firmware Framework

- **Platform:** Arduino/Teensyduino (PJRC Audio Library)
- **Audio Processing:** Block-based (128 samples @ 48 kHz = 2.67 ms latency)
- **Routing Engine:** 16→8 matrix mixer with per-crosspoint gain

------

## Key Libraries

- **Audio:** PJRC Audio Library (AudioInputTDM, AudioOutputTDM, AudioMixer4)
- **Encoders:** Handled by DESPEE display module (ESP32-S3 reads encoders via GPIO, drives LVGL focus groups natively, forwards semantic events over UART)
- **Buttons:** PJRC Bounce Library (debouncing)
- **Display:** Offloaded to [DESPEE](https://github.com/openaudiotools/despee) display module (ESP32-S3 custom PCB) (LVGL display engine); Teensy streams binary widget commands over Serial1 UART (pins 0/1) at 921600 baud; see [Display Protocol](display/protocol.md)
- **NeoPixels:** Adafruit NeoPixel or FastLED (level-shifted data output)
- **Network Audio:** AES67 over Ethernet (16-in / 8-out to DAW); see [network-connectivity.md](network-connectivity.md) §9
- **MIDI:** USBHost_t36 (Teensy USB Host library)
- **SD Card:** SdFat (Bill Greiman) with SDIO support (built into Teensyduino)
- **PSRAM:** extmem allocation (Teensy core EXTMEM keyword for PSRAM-backed buffers)
- **FlasherX** (MIT): Teensy 4.x self-programming from SD card — reads `.hex` file, writes to flash, reboots
- **esp-serial-flasher** (Espressif, Apache 2.0): Embedded host library for flashing ESP32 targets over UART — Teensy drives ESP32_EN/GPIO0 pins to enter bootloader, streams `display.bin` at 921600 baud
- **ArduinoJson:** Streaming JSON parser for `ui.json` (UI layout definitions loaded from SD card)

------

## Audio DSP Chain

### Monolithic ChannelStrip Architecture

Each input channel uses a single `AudioChannelStrip` object that combines the entire per-channel signal path into one `update()` call:

**Per-channel signal path (fixed at compile time):**

Input (TDM ADC) → Gain → Pan/Balance → EQ (3× biquad) → Send taps (Aux1/2/3) → Level → Mute → Main bus summing → Peak/RMS metering

Each `AudioChannelStrip` processes one 128-sample block per channel per audio callback. Gain, pan, 3-band EQ (biquad filters), send taps, level, mute, and peak/RMS metering are all computed inline within a single object — no per-stage AudioStream connections or separate AudioAnalyzePeak/AudioAnalyzeRMS objects.

**Why monolithic:** The PJRC Audio Library dispatches each AudioStream object individually (~270 objects in a naive implementation). Per-object overhead (virtual dispatch, block allocation/release, connection traversal) dominates at this scale. Combining per-channel DSP into ~16 monolithic objects eliminates ~250 object dispatches per audio block.

**Object count:** ~80 total (16× AudioChannelStrip + 40× AudioMixer4 for bus summing + 2× AudioInputTDM + 2× AudioOutputTDM + 16× AudioRecordQueue + misc)

**EQ implementation:**

- 3-band fixed EQ per input channel: Low shelf @ 200 Hz, Mid bell @ 1 kHz (Q ≈ 0.7–1.0), High shelf @ 5 kHz
- 48 total biquad filter instances (3 bands × 16 channels), computed inline within ChannelStrip
- Biquad coefficients pre-computed at fixed frequencies; only gain param changes at runtime
- Coefficient recalculation on gain change: ~1–2 µs per band (sin/cos for shelf/bell formula), negligible vs. 2.67 ms block time
- Spread coefficient updates across successive audio blocks if multiple channels change simultaneously (e.g., MIDI CC burst)

**Metering:** Peak and RMS values computed inline during the ChannelStrip processing loop — no separate analysis objects. The ChannelStrip exposes `peak()` and `rms()` methods that return the most recent block's values.

### CPU Budget Estimate

| Subsystem                          | Estimated CPU |
| ---------------------------------- | ------------- |
| TDM receive (16 ch)               | ~2–3%         |
| TDM transmit (8 ch)               | ~1–2%         |
| ChannelStrip (16 ch, inline DSP)  | ~8–12%        |
| Mixer matrix (16→8 crosspoints)   | ~5–8%         |
| Recording interleave to PSRAM     | ~1–2%         |
| Object dispatch overhead           | ~2–3%         |
| **Total audio DSP**               | **~20–30%**   |

Headroom: ~70% CPU available for non-audio tasks (UART display link, MIDI, SD writes, UI logic). Object dispatch overhead drops from ~50–100% of block time (with ~270 individual objects) to ~2–3% (with ~80 monolithic objects).

Profile with `AudioProcessorUsage` and `AudioProcessorUsageMax` during Phase 1 breadboard bringup to validate estimates.

------

## State Management

- Channel state: gain, pan, mute, solo, sends, EQ band levels, record arm (per channel)
- Pair state: stereo link mode, enabled/disabled (per pair)
- Global state: UI focus (selected channel, nav depth, current view/page/module/component), meter hold
- Recording state: idle/armed/recording, armed channels bitmask, elapsed time, file size, buffer fill level
- Selected channel: global variable set at Page level, persists across nav depth changes; Mute/Solo/Rec buttons always resolve to this channel
- MIDI state: 14-bit mode enable flag (per channel group or per parameter type, default: off/7-bit)

------

## UI Update Loop

- **Audio callback:** 44.1/48 kHz, real-time priority
- **Meter analysis:** Every audio block (peak + RMS computed inline in ChannelStrip)
- **UART meter transmission:** Teensy sends METER_BATCH commands (24 channels × 6 bytes = 144 bytes/frame) to ESP32-S3 at 30 Hz (~4.6 KB/s with COBS framing)
- **Continuous parameter push:** Every parameter change (MIDI CC, state restore, automation) sends SET_VALUE/SET_TEXT/SET_STATE to the corresponding widget immediately, regardless of which page is visible — LVGL widgets on hidden pages update in memory and display correctly on page switch with zero latency. Encoder-originated changes are not echoed back (ESP32 already shows the correct value) unless the Teensy clamps/quantizes the value.
- **Periodic full sync:** The main loop cycles through all parameter widgets at ~50 ms intervals (~20 widgets/sec), re-sending current values to catch any dropped UART frames. Full UI (~120 widgets) syncs every ~6 seconds at ~240 bytes/s (0.3% of UART capacity).
- **Input polling:** UI state events from DESPEE (UART, event-driven: SELECTION_CHANGED/VALUE_CHANGED/PAGE_CHANGED), buttons (Bounce debounce via MCP23017 I2C), MIDI (event-driven)

### Priority Model

The PJRC Audio Library runs on a timer interrupt and preempts all other code. The firmware must be structured so that no non-audio task holds shared resources (memory) long enough to cause audio buffer underruns. Key considerations:

- **Display communication** is handled via Serial1 UART (pins 0/1) to the ESP32-S3 display engine at 921600 baud — lightweight, non-blocking, DMA-capable. No SPI bus contention with display rendering. The Teensy streams binary widget commands (COBS-encoded frames with CRC16); the ESP32-S3 is a generic LVGL rendering engine with no device-specific knowledge. See [Display Protocol](display/protocol.md) for protocol details.
- **SD card writes** use SDIO DMA and run from the main loop at lowest priority. The PSRAM ring buffer decouples recording from real-time audio.
- **NeoPixel updates** are fast (8 pixels, microseconds) and non-blocking.
- **MIDI parsing** is event-driven via USBHost_t36 callbacks (for USB MIDI controllers) — lightweight and non-blocking. The MIDI handler resolves `MIDI channel → channel group` (Ch 1: inputs 1–8, Ch 2: inputs 9–16, Ch 3: outputs), then `CC number → parameter + channel offset`. Uniform parsing, no special cases per group.
- **Network audio (AES67)** runs in the main loop — RTP encode/decode, SAP/SDP announcements, and PTP synchronization are handled via QNEthernet. CPU impact ~1–3%. See [network-connectivity.md](network-connectivity.md) §9 for details.

### Noise Mitigation (Firmware)

- **Cap NeoPixel global brightness** (30% default recommended — reduces noise and power; hardware is sized for uncapped operation)
- **Smooth parameter changes** (no abrupt steps in gain/pan — use ramping over 1-2 audio blocks)
- **USB host power management** (ability to power-cycle ports via software)
- **Pop suppression sequencing:** On boot, MCP23008 (address 0x21, on Board 1-top, accessed via TCA9548A Ch 0 → ISO1541) mute outputs default low (muted). After DSP is initialized and stable (~500 ms), firmware sets GP0–GP3 high to unmute. On shutdown, set GP0–GP3 low before power loss. MCP23008 also provides codec PDN control (GP4–GP5) and headphone detect input (GP6).

------

## SD Card Update Mechanism

MIXTEE uses a single SD card update mechanism for all firmware components — no separate flashing tools required. See [SD Update](sd-update.md) for the full specification.

### Boot Sequence

```
Power on → Teensy boots
  ↓
Check SD for /UPDATE/manifest.json
  ↓ (found)                              ↓ (not found)
Verify device ID + checksums             Normal boot
  ↓
Teensy self-update (FlasherX)
  → reads teensy.hex from SD
  → writes to flash (upper first, vector table last)
  → reboots
  ↓
ESP32 reflash (esp-serial-flasher)
  → assert GPIO0 LOW (pin 10)
  → pulse EN LOW/HIGH (pin 9)
  → stream display.bin over Serial1 @ 921600 baud
  → ~16 seconds for 1.5 MB binary
  → reboot ESP32 into normal mode
  ↓
Copy ui.json → /SYSTEM/ui.json
  ↓
Rename /UPDATE/ → /UPDATE_DONE/
  ↓
Normal boot: load ui.json → stream widgets to display → run
```

### Version Tracking

Teensy EEPROM stores current versions (teensy firmware, display firmware, ui layout). Only components with version mismatches are updated. The `/UPDATE/manifest.json` file contains target versions and SHA-256 checksums for each component.

### Display Engine Boot

After normal boot (or post-update reboot):

1. Teensy sends HANDSHAKE command over Serial1
2. ESP32 responds with READY + screen capabilities (width, height, color depth, touch support)
3. Teensy parses `/SYSTEM/ui.json` from SD card (~5 ms with ArduinoJson streaming parser)
4. Teensy translates JSON widget definitions to binary CREATE_WIDGET commands → streams to ESP32 (~200 ms)
5. Teensy sends focus group definitions (CREATE_FOCUS_GROUP, ADD_TO_GROUP, SET_ENCODER_CONFIG) → configures LVGL encoder groups
6. ESP32 builds LVGL widget tree; all pages created at startup, switched via SET_VISIBLE
7. Teensy enters normal operation: METER_BATCH at 30 Hz, SET_VALUE/SET_TEXT on parameter change
8. ESP32 reads on-board encoders, drives LVGL focus navigation and value editing locally, sends data updates (SELECTION_CHANGED, VALUE_CHANGED, PAGE_CHANGED) to Teensy
9. ESP32 forwards touch events (TOUCH_DOWN/UP/DRAG) back to Teensy as coordinate-based events

Total boot-to-display time: <500 ms after Teensy boot completes.
