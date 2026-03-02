# Power Board — Architecture

This is an **off-the-shelf module** — no custom PCB design.

## Module Selection

Recommended modules:
- SparkFun Power Delivery Board (USB-C, Qwiic)
- STEVAL-ISC005V1 (ST evaluation board)
- Generic STUSB4500 breakout (AliExpress/Tindie)

## Integration Notes

- Mount to back panel via USB-C panel-mount nut or adhesive standoff
- USB-C receptacle protrudes through panel cutout
- 2-pin wire (5V + GND) from module output to Main Board TPS22965 load switch input
- STUSB4500 negotiates 5V @ 5A from PD-capable supplies
- Fallback: 5.1k CC resistors default to 5V/3A for non-PD sources
- Module has on-board decoupling and protection

## Power Path

```
USB-C (PD supply) → STUSB4500 module → 2-pin cable → Main Board TPS22965 → system 5V rail
```

---
Back to [README](README.md)
