---
url: https://www.ti.com/lit/eb/slyy228/slyy228.pdf
title: "USB Type-C and Power Delivery Application Note (SLYY228)"
author: Adam McGaffin, Nate Enos, Brian Gosselin, Taylor Vogt, Eric Beljaars, Ghouse Mohiuddin, Undrea Fields, David Liu, Mike Campbell, Nicholaus Malone, Joe Li
date_fetched: 2026-07-25
date_published: 2024-11
topics:
  - ai-infrastructure-and-hardware
---

A 72-page Texas Instruments e-book (SLYY228) that serves as a comprehensive
introduction to USB Type-C and USB Power Delivery. It spans connector
basics through to practical end-equipment design with block diagrams.

The book walks through the USB-C connector (24-pin, reversible, cold-socket),
the CC-line negotiation protocol (300kbps BMC signaling, Rp/Rd resistors, three
message types), and the full power negotiation sequence. It covers the
distinction between USB-C the connector standard and USB PD the power/data
protocol — a common point of confusion.

Power delivery is a major focus: the evolution from USB PD 1.0 through 3.1
Extended Power Range (EPR), which enables 240W at 48V/5A with adjustable
voltage supply in 100mV steps. The book details the safety implications above
100W and the sink-driven EPR entry mechanism with keep-alive messages.

Data coverage includes USB 2.0 through USB4 Gen 4 (80Gbps symmetric, 120Gbps
asymmetric), signal integrity challenges and conditioning (retimers vs. linear
redrivers), alternate modes (DisplayPort pin assignments C/D/E), and eUSB2 for
sub-7nm process nodes. Practical design chapters show block diagrams for
laptops, docking stations, Bluetooth speakers, power tools, and more.

The final chapter is TI promotional material, but it usefully contrasts TI's
integrated PD controller approach (on-chip FETs, dead-battery LDO, GUI config
tools) against typical discrete designs. Engineers selecting components will
find the architectural comparison concrete rather than purely marketing fluff.
