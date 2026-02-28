# MIXTEE Enclosure

## Back Panel — Inputs + Outputs + Power

All 24 audio jacks (12 stereo pairs) on the back panel, evenly spaced in 2 rows (L top, R bottom). No physical gaps between groups — separation is by labeling only.

**PWR — USB-C (power only):** Located on the back panel (right side, looking at back). 5V 3A, no data lines. Plug-and-forget connection to dedicated power supply.

### Jack order (left to right, looking at back)

1. AUX1, 2. AUX2, 3. AUX3, 4. Master, 5. 15/16, 6. 13/14, 7. 11/12, 8. 9/10, 9. 7/8, 10. 5/6, 11. 3/4, 12. 1/2

Jack centers at x = 20, 40, 60, 80, 100, 120, 140, 160, 180, 200, 220, 240 mm from left panel edge.

### Spacing

- Jack center-to-center: **20 mm** (uniform, all 12 pairs)
- 12 pairs → 11 gaps → 11 × 20 = **220 mm** span
- End margins: **20 mm** each side
- Total width: 220 + 20 + 20 = **260 mm**

### Width conclusion

Enclosure: **260 × 84.6 × 50 mm** (W × D × H).

## Top Panel

Dimensions: **260 mm** wide × **84.6 mm** deep (front to back). Outputs moved to back panel — top panel is now entirely controls.

### Left zone — Display + Controls

Left zone splits into two columns: encoders on the far left, display to their right.

**Encoder section** (far left strip):
- Nav encoder (top)
- Edit encoder (below Nav)
- Stacked vertically as a pair

**Display** (to the right of encoders):
- TFT 93 × 56 mm visible area

### Right zone — Keys + Monitoring + Connectivity

**3×4 CHOC key grid** (left portion of right zone):
- Row 1 (channel controls): Mute, Solo, Rec, (spare)
- Row 2: (spare), (spare), (spare), (spare)
- Row 3 (navigation): Home, Back, Page, Shift
- Custom PCB with CHOC hotswap sockets + WS2812B-2020 per key + 100nF decoupling caps

**SD / Vol / Phones column** (to the right of keys, vertical stack):
- Full-size SD card slot (aligned with key row 1, breakout from Teensy 4.1 SDIO lines)
- Volume pot (aligned with key row 2)
- Headphone 1/4″ TRS jack (aligned with key row 3)

**Top-right corner** (connectivity + controls):
- Power button (soft, momentary push, triggers TPS22918 load switch latch)
- PC — USB-C (data only: USB Audio 2-in/2-out + USB MIDI composite device)
- MIDI HOST — Dual USB-A (stacked, Pi-style double port)
- MIDI IN (5-pin DIN)

Power-on: press button → load switch latches on → Teensy boots.
Power-off: press button → firmware saves state to SD → load switch releases.

## Keys — Custom PCB

All 12 CHOC key switches mount on custom PCB(s) instead of individual NeoKey breakout boards.

Per switch position:
- Kailh CHOC hotswap socket
- WS2812B-2020 NeoPixel (daisy-chained DOUT→DIN, single data pin)
- 100nF ceramic decoupling cap

All 12 LEDs share one data line. Switches wire to direct GPIO (Teensy has plenty of spare pins for 12 keys — no scan matrix needed).
