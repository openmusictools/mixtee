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
- **Encoders:** PJRC Encoder Library (quadrature decoding)
- **Buttons:** PJRC Bounce Library (debouncing)
- **Display:** RA8875_t4 (PJRC Teensy 4.x optimized RA8875 library, hardware-accelerated drawing)
- **NeoPixels:** Adafruit NeoPixel or FastLED (level-shifted data output)
- **MIDI:** USBHost_t36 (Teensy USB Host library)
- **SD Card:** SdFat (Bill Greiman) with SDIO support (built into Teensyduino)
- **PSRAM:** extmem allocation (Teensy core EXTMEM keyword for PSRAM-backed buffers)

------

## Audio DSP Chain

**Per-channel signal path (fixed at compile time):**

Input (TDM ADC) → Gain → Pan/Balance → EQ (3× biquad) → Send taps (Aux1/2/3) → Level → Mute → Main bus summing

**EQ implementation:**

- 3-band fixed EQ per input channel: Low shelf @ 200 Hz, Mid bell @ 1 kHz (Q ≈ 0.7–1.0), High shelf @ 5 kHz
- 48 total biquad filter instances (3 bands × 16 channels)
- Biquad coefficients pre-computed at fixed frequencies; only gain param changes at runtime
- Coefficient recalculation on gain change: ~1–2 µs per band (sin/cos for shelf/bell formula), negligible vs. 2.67 ms block time
- Spread coefficient updates across successive audio blocks if multiple channels change simultaneously (e.g., MIDI CC burst)

### CPU Budget Estimate

| Subsystem                          | Estimated CPU |
| ---------------------------------- | ------------- |
| TDM receive (16 ch)               | ~2–3%         |
| TDM transmit (8 ch)               | ~1–2%         |
| Mixer matrix (16→8 crosspoints)   | ~5–8%         |
| Peak/RMS metering (24 ch)         | ~2–3%         |
| Recording interleave to PSRAM     | ~1–2%         |
| EQ (48 biquads)                   | ~4–6%         |
| **Total audio DSP**               | **~15–24%**   |

Headroom: ~75% CPU available for non-audio tasks (display, MIDI, SD writes, UI logic).

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
- **Meter analysis:** Every audio block (peak + RMS calculation)
- **UI refresh:** 20-30 Hz (independent task, non-blocking)
- **Input polling:** Encoders (interrupt-driven), buttons (Bounce debounce), MIDI (event-driven)

### Priority Model

The PJRC Audio Library runs on a timer interrupt and preempts all other code. The firmware must be structured so that no non-audio task holds shared resources (SPI bus, memory) long enough to cause audio buffer underruns. Key considerations:

- **Display rendering** is the most CPU-intensive non-audio task. The RA8875 controller offloads rectangle fills, line drawing, and text rendering to hardware, reducing Teensy CPU load. Use partial screen updates (only redraw changed meters/parameters). SPI clock limited to ≤22 MHz for RA8875 stability. Dedicate a separate SPI bus to the display if possible (Teensy 4.1 has multiple SPI peripherals).
- **SD card writes** use SDIO DMA and run from the main loop at lowest priority. The PSRAM ring buffer decouples recording from real-time audio.
- **NeoPixel updates** are fast (8 pixels, microseconds) and non-blocking.
- **MIDI parsing** is event-driven via USBHost_t36 callbacks — lightweight and non-blocking. The MIDI handler resolves `MIDI channel → channel group` (Ch 1: inputs 1–8, Ch 2: inputs 9–16, Ch 3: outputs), then `CC number → parameter + channel offset`. Uniform parsing, no special cases per group.

### Noise Mitigation (Firmware)

- **Cap NeoPixel global brightness** (30% default recommended — reduces noise and power; hardware is sized for uncapped operation)
- **Smooth parameter changes** (no abrupt steps in gain/pan — use ramping over 1-2 audio blocks)
- **USB host power management** (ability to power-cycle ports via software)
- **Pop suppression sequencing:** On boot, hold output mute relay/switch closed until DSP is initialized and stable (~500 ms), then ramp up. On shutdown (voltage drop detected), ramp down and mute before power loss.
