# USB Type-C and Power Delivery Architecture

Texas Instruments' definitive 72-page e-book (SLYY228, November 2024) on the engineering architecture of USB Type-C and USB Power Delivery — the protocol stack, electrical design, and system integration that makes a single reversible connector carry 240W of power and 80Gbps of data simultaneously. Written by TI applications engineers, it functions as both a tutorial and a component selection guide, walking from connector basics through USB4, eUSB2, and Extended Power Range with block diagrams for every common end-equipment configuration.

---

## Key Quotes

> "USB Type-C® (USB-C®) is an industry-standard connector that enables the transmission of both data and power on a single interface. With USB Power Delivery, you can now transmit up to 240W of power and up to 80Gbps of data at the same time."

This is the opening sentence and it sets the scope: USB-C is not a cable, it's a convergence play. Power and data were separate interfaces for decades (barrel jack + VGA/DVI/HDMI + USB-A). USB-C collapses them into one connector. The engineering achievement here is underappreciated — this isn't just a new plug shape, it's a real-time negotiation protocol running over the same pins that carry the negotiated power.

> "The USB-C specification allows shorting of the D+ and D– lines together (D+ to D+ and D– to D–) on the receptacle side. Regardless of cable orientation, the physical layer (PHY) will always see the cable's D+ and D– pair."

This is the kind of detail that separates a standard from a spec. USB 2.0 speeds (480Mbps) are slow enough that a stub from shorting is acceptable; USB 3.1 speeds (10Gbps) are not — SuperSpeed requires a multiplexer rather than a passive short. The document repeatedly surfaces these engineering trade-offs: passive where possible, active where necessary. This is hardware design as constraint satisfaction, and the document makes the constraints legible.

> "To enter this state, you need either a data-role swap or a power-role swap."

The docking station example is the killer app for role swaps: the dock provides power (source) but receives data (UFP). Without role swaps, you'd need two cables — one for power, one for data. With USB PD, a single cable handles both, and the roles are negotiated dynamically. This is the moment USB-C stops being "a better USB plug" and starts being a systems architecture decision.

> "The purpose of the new handshake guarantees that a single mistake in the handshake cannot lead to an EPR contract (in order to satisfy safety regulators)."

The EPR safety handshake is the document's most interesting protocol design detail. 240W at 48V is enough to start a fire. The protocol was designed so that no single message error can accidentally negotiate a voltage above 20V — the sink must explicitly request EPR entry, the source verifies both cable and sink are EPR-capable, and the sink must send keep-alive messages every ~500ms or the source hard-resets to 5V. This is safety engineering as protocol design, and TI had functional safety experts independently audit the 240W functionality. It's a case study in making a standard that regulators will accept.

> "With a highly integrated design, TI PD controllers eliminate the need for firmware development or an external micro-controller."

The final chapter is unapologetically a sales pitch, but it's a useful one: it draws a concrete architectural contrast between TI's fully integrated approach (internal FETs, dead-battery LDO, 26V-tolerant CC pins, no coding required) and "typical" PD controllers that require external power paths, external microcontrollers, and manual firmware. Even as marketing, this is genuine systems engineering content — the comparison diagrams make the integration trade-offs visible.

## Key Themes

#hardware #protocol #standard #power #reference

**The connector IS the protocol.** The physical reversibility of USB-C is not a passive feature — it requires active electronics (CC detection, SuperSpeed multiplexing, orientation-aware PHY routing). This is a recurring theme: the connector shape and the protocol are co-designed. You can't understand USB-C without understanding both.

**Negotiation as the core primitive.** USB PD's entire value proposition is negotiation: power levels, data roles, alternate modes, cable capabilities. The document walks through the exact message sequences (Source_Capabilities → Request → Accept → PS_RDY) with timing diagrams. This is a state machine, not a static connection. The EPR handshake adds a second layer: SPR contract first, then EPR entry with keep-alive, then higher-voltage contract. Two nested negotiation layers, each with failure modes that default to safe states.

**Backward compatibility as a design tax.** Every chapter pays the backward-compatibility tax. USB4 must fall back to USB 2.0 if no PD controller is present. eUSB2 repeaters exist solely to bridge the voltage gap between sub-7nm chips and legacy 3.3V USB 2.0 devices. DisplayPort Alternate Mode pin assignments have three variants (C/D/E) to handle different cable configurations. The spec accretes complexity because it refuses to break existing devices.

**The EU mandate as market force.** The document mentions the EU's movement to universalize USB-C across small electronic devices. This is the regulatory tailwind: USB-C isn't winning on technical merit alone, it's winning because governments are mandating it. For engineers, this means USB-C design knowledge stops being optional and starts being table stakes.

**Integration as moat.** TI's pitch is integration: put the FETs, LDO, power paths, and protections inside one IC so the engineer doesn't have to. The "typical PD controller" comparison diagrams are devastating — external FETs everywhere, separate load switches, no dead-battery path, manual I2C register configuration. Whether you buy TI or not, the document makes a persuasive case that PD controller selection is a make-or-break system design decision, not a commodity part pick.

## Critical Analysis

**What the document does well:** It's genuinely educational. Chapter 3 (USB-C and USB PD Specifications) is worth the read alone — it walks through the exact message catalog (control, data, extended), the power negotiation sequence with analyzer traces, and the role-swap state machines with specification diagrams. The block diagrams in chapters 9-10 are immediately useful: you can look at the laptop diagram and understand why there are two separate power paths (one sink for charging, one source for peripherals) and where the multiplexer sits. This is reference-quality documentation.

**What it omits:** The document never mentions competing standards (Apple's Lightning, proprietary charging protocols) or the historical chaos of USB-C cable compatibility. Every USB-C user has experienced the "this cable charges but doesn't do video" frustration — the e-marker system that *should* prevent this is described technically but the user-facing failure modes are absent. The document also skips the counterfeit cable problem entirely, which is a real safety concern at 240W. These omissions are understandable for an applications note but worth noting — this is how TI wants USB-C to work, not necessarily how it works in the wild.

**The TI bias is structural, not textual.** The technical content in chapters 1-9 is largely vendor-neutral — it's explaining the USB-IF specifications. But the choice of what to explain and what to skip reflects TI's product line. The extended treatment of eUSB2 (chapter 7) and the detailed signal integrity discussion (chapters 4-5) map directly to TI's component catalog (eUSB2 repeaters, USB redrivers, DisplayPort multiplexers). This isn't deceptive — it's an applications note from a component manufacturer — but a reader should understand that the document's scope is shaped by what TI sells.

**The unspoken story is regulatory-driven commoditization.** The EU mandate is mentioned in passing, but it's the most important structural force in the document. When governments require USB-C on all small electronics, the connector becomes a commodity and the value shifts to the silicon inside it. TI's e-book is, at one level, a bet on this future: if every device has a USB-C port, every device needs a PD controller, and TI wants engineers to reach for theirs.

**Connections:** This sits at the intersection of hardware abstraction and protocol design — the same design instincts that show up in [[Smart Models Dumb Pipes]] (smart protocols, dumb transport) and [[The Lindy Effect in Software]] (USB as a 30-year standard that keeps absorbing new capabilities). The EPR safety handshake is a case study in [[Guardrails and Feedback Loops]] applied to power electronics. [[Muxcard]] wrestles with the physical constraints of USB-C at the 1mm thickness scale.

---
*Sources: [[raw/ti-usb-c-pd-ebook]]*
*Last updated: 2026-07-25*
