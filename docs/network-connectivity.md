# MIXTEE: Network Connectivity Plan (v1)

*← Back to [README](../README.md) | See also: [Hardware](hardware.md) · [Firmware](firmware.md) · [System Topology](system-topology.md)*

------

## 1. Common network stack

- **Hardware**
  - Audio-capable boxes (synth, mixer, hub): Teensy 4.1 with native 100 M Ethernet (RMII PHY).
  - Control-only boxes (motorized controllers): cheaper Teensy + external MAC/PHY, same UDP/IP stack, no audio.
- **Network layer**
  - IPv4 only, single LAN, no routing.
  - UDP for audio, MIDI, discovery; TCP optional for config UI.

---

## 2. Addressing and naming

- **IP**
  - Use DHCP if available.
  - Fallback: IPv4 link-local (`169.254.x.x`) AutoIP when no DHCP.
- **Hostnames (via mDNS)**
  - `device-type-XXXX.local`, where `XXXX` is a short unique suffix (e.g. last 4 MAC hex digits).
  - Examples:
    - `synth-a3f2.local`
    - `mixer-01b7.local`
    - `ctrl-92cc.local`

---

## 3. Discovery (mDNS + DNS-SD)

### Service types

- Audio endpoints:
  - `_jfa-audio._udp.local`
- MIDI 2.0 endpoints:
  - `_jfa-midi2._udp.local`

### Service instance naming

- `Synth Out 1-2._jfa-audio._udp.local`
- `Mixer Main._jfa-audio._udp.local`
- `Main Controller._jfa-midi2._udp.local`

### TXT fields: `_jfa-audio._udp`

- `role=`: `synth` | `mixer` | `hub`
- `dir=`: `tx` | `rx` | `txrx`
- `ch=`: channel count (e.g. `2`, `8`, `16`)
- `sr=`: sample rate (e.g. `48000`)
- `fmt=`: `pcm24`
- `pkt=`: packet time in ms (e.g. `1`)
- `stream=`: short stream ID (`main`, `aux1`, `busA`, ...)

### TXT fields: `_jfa-midi2._udp`

- `dir=`: `in` | `out` | `inout`
- `ump=`: UMP version (e.g. `2.0`)
- `ep=`: endpoint role (`synth` | `mixer` | `ctrl` | `hub`)
- `ch=`: logical channel count or groups

Each Teensy:

- Registers its services at boot (mDNS/DNS-SD).
- Answers queries for its host/service names.
- Optionally browses for peers (for auto-pairing).

---

## 4. Time sync and audio clocking (audio devices)

- **PTP (IEEE 1588v2) on Teensy 4.1**
  - Use i.MX RT hardware timestamping.
  - One device (default: mixer) is PTP **grandmaster**.
  - Other audio devices are PTP slaves.
- **Audio profile**
  - 48 kHz, 24-bit PCM.
  - 1 ms packet time (48 samples/channel/packet).
  - RTP timestamps derived from PTP clock.

Control-only controllers: no PTP required.

---

## 5. Media transports

### Audio (Teensy 4.1 synth/mixer/hub)

- RTP over UDP, AES67-style fixed profile:
  - 48 kHz, 24-bit, 1 ms packets.
  - Fixed channel counts per stream (2 / 8 / 16).
- UDP ports:
  - Either static per-device (e.g. 50000 + stream index), or
  - Advertised in DNS-SD TXT or SRV records.
- Subscriptions:
  - Receiver uses sender IP + port + stream ID.
  - Multicast or unicast depending on design.

### MIDI 2.0 (all devices)

- Network MIDI 2.0 (UDP + UMP):
  - One UDP port per MIDI endpoint.
  - Endpoint port and name advertised via `_jfa-midi2._udp`.
  - Simple session model: once discovered, endpoints handshake and start UMP exchange.

---

## 6. Roles per device

### Virtual synth (Teensy)

- Publishes 1-N `_jfa-audio._udp` TX streams (main/aux outs).
- Publishes one `_jfa-midi2._udp` endpoint:
  - `dir=inout`, `ep=synth`.
- Optionally browses for controller `_jfa-midi2._udp` services.

### Digital mixer (Teensy 4.1)

- PTP grandmaster.
- Publishes multiple `_jfa-audio._udp`:
  - RX (inputs) and TX (buses/mains/auxes).
- Publishes `_jfa-midi2._udp` (IN/OUT) for full control.
- Optional small HTTP server at `http://mixer-XXXX.local/` for UI.

### MIDI/audio hub/interface

- Subscribes to selected audio streams from synth/mixer.
- Publishes `_jfa-audio._udp` TX if it exposes its own stream(s).
- Bridges Network MIDI 2.0 <-> DIN/USB, publishing `_jfa-midi2._udp` endpoints.

### Motorized-fader controller

- No audio, no PTP.
- Publishes one `_jfa-midi2._udp` endpoint (`ep=ctrl`, `dir=inout`).
- Browses for mixer `_jfa-midi2._udp` and pairs automatically or on user action.

---

## 7. Patchbay "brain"

- Runs on mixer or hub.
- Periodically browses `_jfa-audio._udp` and `_jfa-midi2._udp`.
- Builds in-RAM graph of devices and ports from TXT data.
- Exposes:
  - Simple web UI (HTTP/JSON + minimal HTML/JS), or
  - Simple OSC/JSON API.
- Applies routes by:
  - Opening/closing RTP sockets for audio subscriptions.
  - Initiating Network MIDI 2.0 sessions between chosen endpoints.

---

## 8. Security / isolation (v1)

- Assume trusted studio LAN.
- No auth, no encryption in v1.
- Keep all custom UDP ports in a documented, non-conflicting range (e.g. 50000-50100).
- Use a clear prefix for all custom service types (`_jfa-*`) to avoid clashes.

------

## Feasibility Assessment

*Based on analysis of MIXTEE hardware resources (Teensy 4.1 + DP83825I PHY).*

### Verdict: Fully feasible

The plan is well-matched to the Teensy 4.1 hardware. All proposed network services fit comfortably within available CPU, memory, and bandwidth.

### CPU Budget

| Task | CPU % |
|------|-------|
| Audio DSP (existing) | ~30% |
| RTP audio encode/decode | 1-3% |
| PTP software timestamping | 2-5% |
| mDNS/DNS-SD | <0.5% |
| MIDI 2.0 over UDP | <1% |
| UDP/IP stack (QNEthernet) | 2-3% |
| **Total** | **~40%** |

**~60% CPU headroom** remains after all network services.

### Bandwidth

- 16ch x 48 kHz x 24-bit = **18.4 Mbps** uncompressed RTP audio
- 100 Mbps Ethernet link provides ample headroom for audio + mDNS + MIDI + PTP
- 1 ms packet time (48 samples) matches AES67 standard

### Memory

- 8 MB PSRAM total, ~2 MB used for recording buffers
- **6 MB available** for RTP ring buffers, PTP logs, mDNS cache

### Network Audio Readiness

| Feature | Status | Notes |
|---------|--------|-------|
| Ethernet PHY | Ready | DP83825I on Teensy 4.1; 100 Mbps full-duplex |
| RTP Audio (AES67-style) | Ready | 18.4 Mbps << 100 Mbps link capacity |
| PTP Timestamping | Software-feasible | GPT @ 1 MHz (1 us); ~10-100 us accuracy; nanosecond PTP needs PHY with hardware support (not available on DP83825I) |
| mDNS / DNS-SD | Ready | Lightweight; ~0.5% CPU |
| MIDI 2.0 over RTP | Ready | AppleMIDI or custom RTP-MIDI; ~0.5-1% CPU |

### Caveats

1. **PTP accuracy is software-limited.** The DP83825I PHY has no hardware PTP timestamping. Achievable accuracy is ~10-100 us via Teensy's GPT timer in the RX ISR — sufficient for studio use with 1 ms packet times (48-sample buffer absorbs jitter), but not nanosecond-grade like a proper AES67 endpoint with a PTP-capable PHY (e.g., DP83640).

2. **Control-only boxes on cheaper Teensy + external MAC/PHY** — the library situation for SPI Ethernet (W5500, ENC28J60) is less mature for mDNS/PTP. Consider whether Teensy 4.1 across the board (~$30 each) simplifies development vs. mixed hardware.

3. **Multicast vs. unicast for RTP audio** — on a small studio LAN with <10 devices, unicast is simpler and avoids IGMP snooping headaches on consumer switches. Multicast becomes worthwhile only with multiple receivers per stream.

### Integration Architecture

```
Audio Callback (interrupt, ~2.67 ms):
  +-- TDM RX (SAI1/SAI2) -- Codecs -> memory
  +-- ChannelStrip DSP (16 ch inline)
  +-- Mixer matrix (16->8) -- bus summing
  +-- TDM TX (SAI1/SAI2) -- memory -> Codecs
  +-- [RTP encode] -- if audio streaming enabled
  +-- [PTP timestamp capture] -- if Ethernet RX coincides

Main Loop (low priority):
  +-- Ethernet RX ISR (DMA-driven, ~1-2% CPU)
  +-- UDP/TCP socket operations (QNEthernet event loop)
  +-- RTP/PTP message processing (async)
  +-- mDNS daemon (polled, ~100 ms interval)
  +-- MIDI parsing (USB host ISR)
  +-- SD card writes (ring buffer -> SDIO DMA)
  +-- UI updates (encoder/button debounce, display UART)
  +-- NeoPixel updates (if needed)
```

------

## 9. DAW Connectivity

With the XMOS XU216 USB audio bridge removed, Ethernet becomes the sole digital audio path between MIXTEE and a DAW. MIXTEE speaks standard AES67, so any compliant virtual soundcard on the host computer can receive and send audio — no companion app required.

### 9.1 DAW Usage Scenarios

| Scenario | Direction | Latency | Priority |
|----------|-----------|---------|----------|
| Multitrack recording | MIXTEE → DAW (16 ch) | One-way ~5–10 ms | Core |
| Stem/playback return | DAW → MIXTEE (8 ch) | One-way ~5–10 ms | Core |
| Live monitoring | Hardware path (ADC → DSP → DAC) | N/A | Already solved |
| Plugin insert (send/return) | Bidirectional | Round-trip ~10–20 ms | Stretch goal |

Live monitoring never touches the network — it stays on the hardware audio path through the codecs and Teensy DSP, so latency is sub-millisecond regardless of network configuration.

### 9.2 Network Stream Layout (16-in / 8-out)

**MIXTEE TX (to DAW) — 16 channels in 2 streams:**

| Stream ID | Channels | Content |
|-----------|----------|---------|
| `adc-1-8` | 8 | Pre-mix ADC inputs 1–8 (codecs U1 + U2, SAI1) |
| `adc-9-16` | 8 | Pre-mix ADC inputs 9–16 (codecs U3 + U4, SAI2) |

**MIXTEE RX (from DAW) — 8 channels in 1 stream:**

| Stream ID | Channels | Content |
|-----------|----------|---------|
| `return-1-8` | 8 | DAW playback returns routed into mixer |

**Total bandwidth:**

- TX: 16 ch × 48 kHz × 24 bit × 1.1 overhead ≈ 20.3 Mbps
- RX: 8 ch × 48 kHz × 24 bit × 1.1 overhead ≈ 10.1 Mbps
- **Bidirectional total: ~30.4 Mbps** (well within 100 Mbps full-duplex)

Two separate TX streams let a laptop subscribe to only one 8-channel group if full 16-channel capture isn't needed, halving host-side bandwidth and buffer requirements.

### 9.3 Full AES67 Compliance

The existing network plan (§5) describes "AES67-style RTP." For DAW interoperability with standard virtual soundcards, MIXTEE must implement the full AES67 discovery and session stack:

#### SDP (Session Description Protocol, RFC 4566)

Each stream is described by a text SDP blob. Required fields:

```
v=0
o=- <session-id> <version> IN IP4 <device-ip>
s=MIXTEE adc-1-8
c=IN IP4 <multicast-group-or-unicast-ip>
t=0 0
m=audio <port> RTP/AVP 96
a=rtpmap:96 L24/48000/8
a=ptime:1
a=ts-refclk:ptp=IEEE1588-2008:<ptp-grandmaster-id>
a=mediaclk:direct=0
```

Key points:
- Payload type `L24` (linear 24-bit PCM), dynamic PT 96–127 assigned via SDP
- `ptime:1` — 1 ms packet time (48 samples per channel)
- `ts-refclk` references the PTP grandmaster clock
- One SDP per stream (3 total: `adc-1-8`, `adc-9-16`, `return-1-8`)

#### SAP (Session Announcement Protocol, RFC 2974)

SAP is the primary discovery mechanism for AES67 devices:

- MIXTEE multicasts its SDP blobs to **239.255.255.255:9875** (SAP well-known address)
- Announcement interval: every 30 seconds per stream (configurable)
- Each SAP packet: 8-byte header + SDP payload (~200–300 bytes per stream)
- Virtual soundcards on the DAW host listen on this multicast group and auto-discover MIXTEE streams

#### PTP IEEE 1588v2 (Media Profile for AES67)

- PTP domain: **0** (AES67 default)
- Announce interval: 1 s (log₂ = 0)
- Sync interval: 125 ms (log₂ = −3)
- Delay request interval: 125 ms (log₂ = −3)
- MIXTEE is PTP grandmaster on the studio LAN (consistent with §4)

**PTP accuracy caveat:** DP83825I has no hardware timestamping. Software PTP via Teensy GPT timer gives ~10–100 µs accuracy — sufficient for 1 ms packet jitter buffers but not nanosecond-grade AES67. For studio use with a single MIXTEE on a quiet LAN, this is acceptable. Professional AES67 endpoints with hardware PTP will still sync, but may report higher offset than usual.

#### RTP Payload Format

- Payload type: L24 (24-bit linear PCM, network byte order)
- Max payload: 1460 bytes (fits standard 1500 MTU Ethernet frame)
- Packet time: 1 ms = 48 samples/channel
- 8-channel stream: 48 × 8 × 3 bytes = **1152 bytes/packet** (fits in one Ethernet frame)
- SSRC: unique per stream, stable across session lifetime
- Sequence numbers: monotonically increasing, wrapping at 2¹⁶

#### Discovery Mechanism Summary

| Mechanism | Purpose | Scope |
|-----------|---------|-------|
| SAP/SDP | AES67 stream discovery for DAW virtual soundcards | DAW interop |
| mDNS/DNS-SD (`_jfa-*`) | Device-to-device discovery within JFA ecosystem | JFA ecosystem |

Both mechanisms run simultaneously. SAP/SDP ensures any AES67-compliant virtual soundcard can find MIXTEE streams without custom configuration.

### 9.4 Recommended Virtual Soundcards

**No companion app will be built.** MIXTEE speaks standard AES67, so users install an existing virtual soundcard on their DAW host.

#### Linux — PipeWire (built-in, recommended)

PipeWire 1.1+ has native AES67 support via `pipewire-module-rtp-source` and `pipewire-module-rtp-sink`:

- Auto-discovers MIXTEE streams via SAP — zero configuration
- Creates virtual ALSA / JACK / PulseAudio devices; any DAW (Ardour, Reaper, Bitwig) sees them natively
- Verified interop with Dante and RAVENNA devices
- Ships with most modern Linux distributions (Fedora, Ubuntu 24.04+, Arch)

**Fallback:** [aes67-linux-daemon](https://github.com/bondagit/aes67-linux-daemon) — GPL-licensed, provides a web UI and REST API, based on the Merging Technologies RAVENNA/AES67 ALSA driver. Good option for distributions still on PulseAudio.

#### macOS — AES67 macOS Driver (open-source, beta)

[AES67_macos_Driver](https://github.com/maxajbarlow/AES67_macos_Driver) — Core Audio HAL plugin:

- Up to 128 channels, Apple Silicon native
- Installs to `/Library/Audio/Plug-Ins/HAL/`, no kernel extension needed (AudioServerPlugIn architecture)
- ~95% complete as of 2025; limited real-world validation — test before relying on it for production sessions

**Commercial fallback:** Merging RAVENNA Virtual Audio Device, Lawo R3LAY VSC.

#### Windows — Limited open-source options

- [DIGISYN VSC](https://github.com/Digisynthetic/Digisyn-Link-VSC) — open-source AES67 virtual soundcard, WDM + ASIO support, early-stage development
- **Commercial options:** DIGISYN VSC (commercial release), Lawo JADE VSC (64 ch, WDM + ASIO), Merging MAD, h7r AES67 VSD
- No mature open-source solution exists yet for Windows — commercial options are recommended for reliable DAW use

#### Cross-platform utility — GStreamer

GStreamer has full RTP/PTP infrastructure built in and can receive/send AES67 streams. Useful for testing and bridging (e.g., piping MIXTEE audio into a recording pipeline), but it does not create a virtual soundcard device visible to DAWs.

### 9.5 Firmware Impact Summary

| Component | What it does | CPU impact |
|-----------|-------------|------------|
| RTP packetizer | Copies post-ADC samples into 2 TX ring buffers per audio callback | ~1% |
| RTP depacketizer | Reads Ethernet RX into return ring buffer in main loop | ~1% |
| SAP/SDP announcer | Periodic multicast of SDP blobs (~200 bytes/stream every 30 s) | <0.5% |
| Return audio path | Ethernet RX → ring buffer → DSP memory (no SAI data line needed) | Negligible |

**Total CPU addition: ~1–3%**, within the existing 40% total budget from §Feasibility Assessment.

The return audio path is pure memory — DAW playback arrives via Ethernet, gets depacketized into a ring buffer, and the DSP reads from that buffer during the audio callback. No codec SAI slot is consumed for return channels.
