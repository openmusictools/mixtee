# Key PCB — Architecture

## MCP23017 Key Matrix

- I2C GPIO expander at address 0x20
- Port A (GPA0–3): 4 column inputs with internal pull-ups
- Port B (GPB0–3): 4 row outputs (active-low scan)
- 16× 1N4148 diodes per switch (cathode toward row) prevent ghosting
- Scan rate: polled via I2C at ~1 kHz, or interrupt-driven via INTA/INTB pin

### Wiring Diagram

```
             GPA0          GPA1          GPA2          GPA3
         (COL 0, pull-up) (COL 1)      (COL 2)      (COL 3)
              │              │              │              │
GPB0 (ROW 0)─┼──[SW1]──|>──┼──[SW2]──|>──┼──[SW3]──|>──┼──[SW4]──|>──
              │              │              │              │
GPB1 (ROW 1)─┼──[SW5]──|>──┼──[SW6]──|>──┼──[SW7]──|>──┼──[SW8]──|>──
              │              │              │              │
GPB2 (ROW 2)─┼──[SW9]──|>──┼──[SW10]─|>──┼──[SW11]─|>──┼──[SW12]─|>──
              │              │              │              │
GPB3 (ROW 3)─┼──[SW13]─|>──┼──[SW14]─|>──┼──[SW15]─|>──┼──[SW16]─|>──
              │              │              │              │
```

`|>` = 1N4148 diode (cathode toward row pin). All matrix wiring local to Key PCB.

## Key-Switch Mapping

| Matrix Position | Key Function | NeoPixel Index |
|----------------|-------------|----------------|
| ROW0 × COL0 (SW1) | Mute | 0 |
| ROW0 × COL1 (SW2) | Solo | 1 |
| ROW0 × COL2 (SW3) | Rec | 2 |
| ROW0 × COL3 (SW4) | (assignable) | 3 |
| ROW1 × COL0 (SW5) | (assignable) | 4 |
| ROW1 × COL1 (SW6) | (assignable) | 5 |
| ROW1 × COL2 (SW7) | (assignable) | 6 |
| ROW1 × COL3 (SW8) | (assignable) | 7 |
| ROW2 × COL0 (SW9) | (assignable) | 8 |
| ROW2 × COL1 (SW10) | (assignable) | 9 |
| ROW2 × COL2 (SW11) | (assignable) | 10 |
| ROW2 × COL3 (SW12) | (assignable) | 11 |
| ROW3 × COL0 (SW13) | Home | 12 |
| ROW3 × COL1 (SW14) | Back | 13 |
| ROW3 × COL2 (SW15) | Page | 14 |
| ROW3 × COL3 (SW16) | Shift | 15 |

## NeoPixel Daisy-Chain

- 16× WS2812B-2020 addressable LEDs
- Single data line from Teensy pin 6 (via JST)
- 300–500Ω series resistor on Main Board (near first pixel data entry)
- 100nF decoupling cap per pixel
- Bulk capacitor (1000–2200 uF) near 5V entry on Key PCB recommended
- Default firmware brightness cap: 30% (reduces noise and power draw)

---
Back to [README](README.md)
