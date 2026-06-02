# Muxcard

A fully functional computer literally the size of a credit card — ~1mm thick, built around an ESP32-C3, flexible e-paper display, and NFC reader/writer. Not a concept render: working prototype with professionally manufactured revisions in testing. The core achievement is mechanical, not computational: packing a programmable computer with display, wireless, NFC, IMU, and battery into a package you can carry in a wallet without noticing.

---

## Architecture

The system is a single-board embedded design with five functional blocks, documented as separate schematic PDFs in `hardware/schematics/`:

- **MCU** — ESP32-C3 (RISC-V, WiFi/BLE, 0.85mm nominal height). nRF52832 considered as lower-power alternative with a 0.4mm BGA variant that would ease the thickness constraint.
- **Display** — 1.54" 200×200 flexible e-paper driven over SPI via the GxEPD2 library. The flexible variant is essential; rigid e-paper would crack under wallet bending.
- **NFC** — RC522 active reader/writer (not a passive dynamic tag). Supports ISO/IEC 14443 Type A. Enables the card to read and emulate NFC credentials rather than just being one.
- **IMU** — LIS2DW12 3-axis accelerometer (0.7mm). Intentionally accelerometer-only: a gyro would add wake-trigger power consumption without proportional benefit. Used for tap-to-wake from deep sleep.
- **Power** — Ultra-thin LiPo (23×23×1mm, 30mAh in current prototype; planned 0.5mm variant with larger area for 30–50mAh). Charging via magnetic pogo pins; caseless USB-C being tested.

The firmware (`firmware/typewriter`, 120 lines of Arduino C++) is a single-file demo: SPI initialization on pins 6/7/8, GxEPD2 display driver, word-by-word partial refresh animation, then `display.hibernate()` followed by empty loop. The device wakes, renders, and goes back to sleep — ~8µA draw in deep sleep with RTC active.

The PCB is a single-layer flex circuit. The first prototype was home-etched using a 3D printer as a lithography machine (UVTools software, kapton/copper/photoresist substrate), with a solder stencil made from stacked photoresist film. Professional manufactured flex PCBs are in progress for the next revision.

## Key Techniques

**Strain isolation via deliberate weak points.** The most interesting engineering decision: instead of trying to resist bending forces, the board creates "islands" around large ICs and introduces tactical weak points between them. Forces route around sensitive components rather than through them. This only works because components have margin between upper and lower layers — they float slightly rather than being rigidly encapsulated. It's a counterintuitive approach: make the board deliberately fragile in specific places to protect what matters.

**Additive trace routing for flex durability.** Long straight traces on the home-etched flex PCB ripped apart under bending. The fix: more turns — deliberately non-minimal routing that distributes strain across multiple direction changes rather than concentrating it on a single straight segment. The opposite of conventional PCB design where trace length minimization is the goal.

**Active NFC over passive tag.** Most smart-card concepts use a dynamic NFC tag (NTAG-style) that phones read passively. Muxcard uses a full NFC reader/writer (RC522), which means it can read and emulate cards — the card interrogates other devices rather than being interrogated. This is what makes pentesting and credential-wallet use cases possible.

**Partial e-paper refresh for animation.** The typewriter demo uses `display.setPartialWindow(0, 0, 200, 200)` to update the e-paper word by word without a full-screen flash. E-paper partial refresh is finicky — ghosting, incomplete clears, timing sensitivity. The demo proves it works on this hardware stack.

**3D printer as lithography machine.** The DIY flex PCB fabrication used a resin 3D printer's UV display panel as a photolithography exposure unit, controlled via UVTools — software designed for PCB lithography specifically. This produces detail comparable to professional PCBs (fine-pitch traces) without a CNC or photoplotter. The solder stencil was also DIY: stacked layers of photoresist film, UV-exposed and developed into a one-use stencil.

## Design Decisions

**Optimized for thinness above everything else.** Every component choice is filtered through height: the nRF52 BGA variant is preferred over ESP32-C3 despite worse developer experience because it's 0.4mm vs 0.85mm. The accelerometer-only IMU choice (no gyro) saves both height and power. The battery strategy plans a shift to thinner cells with larger area to maintain capacity. This isn't about being small — it's about fitting within the 0.76–1mm envelope of a real ISO 7816 smartcard, where every 0.1mm matters.

**DIY fabrication as speed play, not cost play.** PCBs are cheap. The home-etch process was about iteration speed — avoiding multi-week international shipping and customs delays for flex PCB fabrication. The author explicitly says they were "too impatient for a fab." This is a common pattern in hardware prototyping: the constraint isn't money, it's feedback loop latency.

**Magnetic pins over USB-C for primary interface.** The main charging/data connection is magnetic pogo pins. USB-C is being tested as a backup, but even the slimmest USB-C connector strains the thickness budget. This is the right call for daily use — aligning pins magnetically is more practical than plugging in a cable on a 1mm card.

**Trade-off: power vs capability.** The ESP32-C3 is only "acceptable" on power consumption — the nRF52 would be dramatically better but requires more complex firmware development (no Arduino abstraction). The current prototype accepts higher sleep current for faster development. This is the classic embedded trade-off: developer ergonomics vs battery life.

## Comparison Notes

**vs. MimiClaw** — [[MimiClaw]] puts an AI agent on a $5 ESP32-S3, but it's a dev board, not a wearable form factor. Muxcard is the opposite bet: same MCU family, but optimized for physical form factor over computational capability. MimiClaw proves the ESP32 can run agent loops; Muxcard could theoretically run the same firmware in a wallet-sized package. The 0.5W power draw is the main obstacle — MimiClaw is USB-powered, Muxcard runs on a 30mAh battery.

**vs. Resident** — [[Resident — ESP32 Lua Sandbox with Agent Skills]] provides a sandboxed Lua runtime for ESP32 devices with hot-reload and Claude Code integration. Muxcard is a hardware platform that could host Resident — the combination would let agents write and deploy apps to a credit card in your pocket. Resident's 10 FPS Lua tick and PSRAM allocator would run on Muxcard's ESP32-C3 without modification.

**vs. Claude Lamp** — [[Claude Lamp]] uses physical computing as ambient agent-status display. Muxcard's e-paper display could serve the same purpose (peripheral awareness of agent state) but adds mobility, NFC, and a touch interface. The e-paper's zero-power image retention makes it ideal for status displays that update infrequently.

**vs. Flipper Zero** — The Flipper Zero is the closest commercial product in spirit (multi-protocol NFC/RF/IR pentesting tool), but it's 20× thicker and pocket-sized rather than wallet-sized. Muxcard targets the same use cases in a radically thinner form factor, trading protocol diversity for wearability.

---

#hardware #embedded #esp32 #project

*Sources: [[raw/muxcard]]*
*Last updated: 2026-06-02*
