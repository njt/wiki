# Zeroclaw

A Rust-based agent runtime distributed as a single binary, designed around trait-based interfaces so every component is swappable. 31.3k stars. Handles 30+ communication channels (Discord, Telegram, Matrix, email, webhooks, CLI), supports ~20 LLM providers with fallback chains, implements OS-level sandboxing (Landlock, Bubblewrap, Seatbelt, Docker), and even supports hardware peripherals (GPIO, I2C, SPI) for embedded deployment.

---

## Key Quotes

> "You own the agent. You own the data. You own the machine it runs on."

## Key Themes

#rust #agent-runtime #self-hosted #security #hardware #trait-based

The interface-based architecture is the real contribution. Everything is a Rust trait: channels, providers, tools, memory systems. You can drop in a new LLM provider, messaging platform, or security policy by implementing the right trait. This makes Zeroclaw less of a product and more of an agent construction kit.

The security architecture is serious: supervised autonomy (medium-risk = approval required, high-risk = blocked), workspace boundaries, command policies, OS-level sandboxes, and cryptographic tool receipts documenting every action. This connects directly to [[yolo-cage]]'s philosophy of moving security to the infrastructure layer, but Zeroclaw bakes it into the runtime rather than wrapping an existing agent.

The hardware integration (Raspberry Pi, STM32, Arduino, ESP32 via Peripheral trait) puts it in conversation with [[MimiClaw]], but at a very different level -- MimiClaw runs the whole agent on the ESP32, while Zeroclaw uses peripherals as tools controlled by a full agent runtime on more capable hardware.

## Critical Analysis

Strong: the trait-based architecture is the right design for a runtime that needs to outlive any single LLM provider, messaging platform, or deployment model. The 31.3k stars suggest a real community. The security model is the most comprehensive in this batch -- tool receipts with cryptographic signing is enterprise-grade auditability.

Weak: Rust's compile times and learning curve make community contribution harder than Go or TypeScript alternatives ([[Serf]], [[Dorothy]]). The feature surface (30+ channels, 20+ providers, hardware, SOPs, web dashboard, ACP) is enormous, raising maintenance questions. The "everything is swappable" philosophy can lead to the framework trap where the system is more about composability than about doing any one thing well.

The 31.3k stars vs. MimiClaw's 5.4k stars tells you something about market positioning: Zeroclaw targets serious builders who want ownership; MimiClaw targets tinkerers who want delight.

---
*Sources: [[summary/zeroclaw]]*
*Last updated: 2026-05-14*
