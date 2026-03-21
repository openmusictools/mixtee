# Keys4x4 PCB — Connections

## JST-PH to Main Board (6-pin)

| Pin | Signal | Teensy Pin | Notes |
|-----|--------|------------|-------|
| 1 | NeoPixel DIN | 6 | Data line from Teensy (300–500Ω series resistor on Main Board) |
| 2 | I2C SDA | 18 | Shared Wire bus (Teensy pin 18) |
| 3 | I2C SCL | 19 | Shared Wire bus (Teensy pin 19) |
| 4 | MCP23017 INT | 22 | Optional interrupt to Teensy |
| 5 | 5V | — | NeoPixel + MCP23017 power |
| 6 | GND | — | Common ground |

Connector: 6-pin JST-PH (B6B-PH-K-S), 2.0mm pitch, ~30–40mm cable.

## NeoPixel Chain

16× WS2812B-2020 daisy-chained. DIN enters via JST pin 1. Each pixel has 100nF decoupling cap. Data flows: JST DIN → pixel 0 DOUT → pixel 1 DIN → ... → pixel 15 DOUT (unused).

---
Back to [README](README.md)
