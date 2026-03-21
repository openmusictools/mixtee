# Keys4x4 PCB

**Dimensions:** ~72 x 80 mm | **Layers:** 2 | **Orientation:** Horizontal, mounted through top panel | **Instances:** 1

4×4 illuminated key grid with I2C-based matrix scanning. CHOC switches protrude through top panel cutouts, keycaps sit flush with panel surface.

## Components

- 16× Kailh CHOC hotswap sockets
- 16× WS2812B-2020 NeoPixels (daisy-chained)
- 16× 100nF ceramic decoupling caps
- 16× 1N4148 signal diodes (anti-ghosting)
- 1× MCP23017 I2C GPIO expander (0x20)
- 1× 6-pin JST-PH connector to Main Board

## See Also

- [`connections.md`](connections.md) — JST-PH pinout, NeoPixel chain
- [`architecture.md`](architecture.md) — MCP23017 key matrix wiring, key-switch mapping, NeoPixel daisy-chain

## Status

Fully routed via FreeRouting (139/139 nets). Gerbers exported.
