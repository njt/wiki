# SuperStation One

An affordable FPGA gaming console from Retro Remake that recreates the original PlayStation in open-source hardware. Built on the Cyclone V FPGA (the MiSTer platform's chip), it plays PS1 games natively while supporting the full MiSTer ecosystem for other retro systems. The bet is that FPGA preservation hardware doesn't need to cost what Analogue charges.

---

## Key Quotes

> "The world's first affordable FPGA gaming console. A recreation of the best console of the 90s. Open source from day 1."

The "affordable" claim is the entire strategy. FPGA consoles have been boutique luxury items — Analogue's products start around $200 and climb. Retro Remake is positioning as the commodity alternative, and "open source from day 1" is the trust signal that distinguishes them from competitors who use open-source FPGA cores but keep their own work proprietary.

> "We don't believe in locking down current hardware to sell *future* hardware."

A direct shot at Analogue's business model of releasing incremental hardware revisions. This is the "right to repair" argument applied to FPGA consoles — the hardware you buy shouldn't be artificially capped to create demand for the next version.

## Specifications

| Component | Detail |
|-----------|--------|
| FPGA | Cyclone V |
| RAM | 128MB BGA SDRAM |
| Video DAC | 24-Bit ADV7125 |
| Video Out | HDMI (up to 1536p/1440p), VGA, DIN10, Composite, Component |
| Audio Out | 3.5mm analog, TOSLINK digital |
| Connectivity | Wi-Fi, Bluetooth, Ethernet |
| Ports | USB-C power, 3x USB-A, TF Card, Dual PS1 SNAC, IO Expansion |
| Extra | NFC reader with Zaparoo support |

The Cyclone V is the same FPGA family as the Terasic DE10-Nano board that the MiSTer project is built on. This is not cutting-edge silicon — it's proven, widely available, and has a mature open-source toolchain. That's the point.

## Key Themes

- #hardware #tool — FPGA-based game preservation as consumer product
- #pattern — Open-source hardware as competitive strategy against proprietary incumbents
- #concept — SNAC (Serial Native Accessory Converter) ports for original controller latency — the kind of detail that separates preservation projects from emulation boxes

## Critical Analysis

This is a product page, not a review, so grain of salt. But the strategy is legible: Retro Remake is doing to Analogue what Analogue did to software emulation — offering hardware accuracy at a lower price point by betting on open source.

The Cyclone V choice is both strength and limitation. It's mature, well-supported, and cheap. But it's also the chip MiSTer has been using for nearly a decade. The PS1 core is well-established on this platform, so native PS1 support is credible. The question is whether "affordable" means "cheap enough to matter" or just "cheaper than Analogue." Without a published price, it's impossible to know which one they mean.

The placeholder image carousels on the product page ("Tell your brand's story through images") suggest this is still in the pre-order hype phase rather than a shipping product. The SuperDock add-on (disk support, more IO) implies a product family strategy, but launching with an expansion ecosystem before shipping the base unit is ambitious.

The open-source commitment is the right call technically and strategically. MiSTer's community is large and productive — being compatible with that ecosystem rather than forking it is the difference between a product and a paperweight. But "open source from day 1" is a promise, not a track record. The proof will be whether core definitions, PCB layouts, and firmware actually land in public repos.

---

*Source: [[summary/retroremake-superstation-one]]*
*Last updated: 2026-05-22*
