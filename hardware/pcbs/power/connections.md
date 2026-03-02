# Power Board — Connections

## 2-pin Output to Main Board

| Wire | Signal | Notes |
|------|--------|-------|
| + | 5V (VBUS) | From STUSB4500 output, post-module protection |
| - | GND | Power return |

Connector: 2-pin JST-PH or screw terminal, ~60–80mm cable, 22 AWG minimum. Carries full system current (up to 5A).

## Power Button Wires (2-pin)

Momentary push button (screw-collar, back panel) wired to Main Board soft-latch sense/keepalive pins (Teensy pins 40/41).

## NVM Configuration

STUSB4500 NVM pre-configured for 5V PDO only (no higher voltages) via I2C before installation. 5.1k CC resistors on module provide fallback 5V/3A for non-PD supplies.

## USB-C Receptacle (panel-mount)

On the breakout module itself. Protrudes through back panel cutout (right side). Labeled "PWR". Power only — D+/D- not routed.

---
Back to [README](README.md)
