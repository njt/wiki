---
url: https://github.com/krauseler/muxcard
title: Muxcard
author: krauseler
date_fetched: 2026-06-02
date_published: 2026-03-31
topics:
  - developer-tools
---

# Muxcard — A Fully Working Computer the Size of a Credit Card

Source: GitHub repository at https://github.com/krauseler/muxcard

License: Creative Commons BY-NC-SA 4.0

## Project Overview

Muxcard is a credit card-sized computer built around an ESP32-C3, flexible e-paper display, and NFC reader/writer. The target is ~1mm total thickness — matching actual credit card dimensions including thickness, not just footprint. The project is in prototype stage (V0.1), with professionally manufactured prototypes being tested.

## Repository Structure

```
firmware/typewriter          — Single Arduino C++ source file (120 lines) — e-paper demo
hardware/schematics/Display.pdf
hardware/schematics/MCU.pdf
hardware/schematics/NFC.pdf
hardware/schematics/Periphirals.pdf
hardware/schematics/Power.pdf
hardware/pcb/Layout.pdf
README.md                    — Extensive project documentation
LICENSE                      — CC BY-NC-SA 4.0
```

## Firmware Analysis

The only source file is `firmware/typewriter` (120 lines of Arduino C++):

```cpp
#include <GxEPD2_BW.h>
#include <Fonts/FreeMonoBold12pt7b.h>

#define EPD_CS    8
#define EPD_DC    5
#define EPD_RST   4
#define EPD_BUSY  3

GxEPD2_BW<GxEPD2_154_D67, GxEPD2_154_D67::HEIGHT> display(
  GxEPD2_154_D67(EPD_CS, EPD_DC, EPD_RST, EPD_BUSY)
);
```

Key firmware details:
- Uses GxEPD2 library for the 1.54" 200x200 e-paper display (GxEPD2_154_D67 model)
- SPI communication: `SPI.begin(6, -1, 7, EPD_CS)` — MOSI on pin 6, no MISO, SCK on pin 7
- Display rotation set to 3 (landscape)
- Font: FreeMonoBold12pt7b (embedded bitmap font)
- Implements a **typewriter animation**: word-by-word partial refresh using `display.setPartialWindow()`
- Uses `display.hibernate()` after animation completes (e-paper retains image without power)
- Line-wrapping algorithm: measures text width with `getTextBounds()`, breaks when exceeding 180px max width
- Margin: 20px, line height: 24px, starting Y: 50px
- Hardcoded demo text: "The future belongs to those who build it."
- After display: `loop() {}` is empty — the device enters deep sleep after rendering

The firmware demonstrates that the e-paper partial refresh works and that the MCU can control the display over SPI. It's a proof-of-concept demo, not production firmware.

## Hardware Architecture

### MCU: ESP32-C3 (current) or nRF52832 (considered)

ESP32-C3 advantages:
- Beginner-friendly, good Arduino integration
- WiFi for occasional API calls
- ~8uA in deep sleep with RTC on
- Nominal height: 0.85mm

nRF52832 advantages (not yet used):
- Much lower power consumption (active BLE < ESP32 idle)
- BGA variant at 0.4mm thin
- Better for the 1mm thickness constraint

### Display: 1.54" 200x200px flexible e-paper

- Flexible variant is essential — the rigid version would crack under wallet/pocket bending
- Supports partial updates (enables the typewriter animation)
- Connection is the hardest engineering challenge: 0.5mm pitch with 0.2mm gaps, one-sided plating, thick stiffener that insulates from heat
- Hot bar soldering doesn't work on this connector as-designed
- First prototype: hand-soldered individual wires ("top 3 most annoying soldering sessions of my life")

### IMU: LIS2DW12 accelerometer

- Low power, good feature set, 0.7mm height
- BMA530 considered (0.55mm) but accuracy sensitive to flexing
- 3-axis accelerometer only (no gyro) — intentionally: gyro consumes significantly more power, accelerometer sufficient for wake triggers
- Used as a wake-up source (tap/gesture detection while MCU sleeps)

### NFC: RC522 reader/writer

- Active NFC reader/writer, not just a passive (dynamic) tag
- Supports ISO/IEC 14443 Type A and other common standards
- Possible replacement: smaller chips being evaluated
- Potential 125kHz MIFARE support in future chip revision

### Battery: Ultra-thin LiPo

- Current: 23x23x1mm, 30mAh — requires full cutout in frame, no physical protection
- Planned: ~0.5mm thin, larger surface area for 30-50mAh capacity
- Protection plan: thin stainless steel layers (similar to PCB stencils) on both sides
- Sourcing problem: no European or US suppliers found for ultra-thin LiPos

### USB-C: Caseless, minimal port

- Main port is magnetic pins (pogo pins) — no physical connector protruding
- USB-C being tested for a caseless, minimal implementation

### PCB: Home-etched flex PCB

First prototype was DIY-etched:
- 3D printer used as lithography machine (software: UVTools)
- Substrate: kapton tape with copper foil, thin photoresist layer
- Single layer, no solder mask, no vias
- Solder paste stencil made from stacked photoresist film
- Problem: long thin traces ripped apart when bending — solved by adding more turns on longer nets to avoid strain accumulation
- Professional flex PCBs being manufactured for next revision

## Key Mechanical Innovation: Strain Isolation Islands

The most interesting engineering insight is the **strain isolation** approach:

Instead of trying to make components and traces survive bending stress, the design creates "islands" — mechanically isolated zones where larger ICs sit. Weak points are deliberately introduced between islands so bending forces route around sensitive areas, which barely experience meaningful strain.

This works only because components have margin between upper and lower layers to move slightly, rather than being completely fixed. It adds a new layer of design complexity but solves most mechanical fatigue issues.

## Potential Use Cases (from README)

- Minimalist wallet for QR codes, NFC keys, tickets, boarding passes
- Pentesting & ethical hacking (similar to Flipper Zero)
- Smart-home dashboard & control
- Offline password, 2FA, crypto wallet
- Business card with near-certain memorability
- Micro SD card slot for storage expansion

## License

CC BY-NC-SA 4.0 — non-commercial use encouraged, commercial licensing available separately.
