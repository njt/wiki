---
url: https://www.ti.com/lit/eb/slyy228/slyy228.pdf
title: "USB Type-C and Power Delivery Application Note (SLYY228)"
author: Adam McGaffin, Nate Enos, Brian Gosselin, Taylor Vogt, Eric Beljaars, Ghouse Mohiuddin, Undrea Fields, David Liu, Mike Campbell, Nicholaus Malone, Joe Li
date_fetched: 2026-07-25
date_published: 2024-11
---

# USB Type-C® and USB Power Delivery Application Note (SLYY228)

Texas Instruments e-book providing a comprehensive introduction to USB Type-C and USB Power Delivery, covering connector basics, protocol specifications, signal integrity, multiplexing, USB4, eUSB2, Extended Power Range (EPR), and end-equipment design with block diagrams.

## Introduction

USB Type-C® (USB-C®) is an industry-standard connector that enables the transmission of both data and power on a single interface. USB Power Delivery (PD) is a standard using the USB-C connector to increase the capabilities and features of the USB-C interface. With USB Power Delivery, you can now transmit up to 240W of power and up to 80Gbps of data at the same time. You can also implement predefined alternate modes such as DisplayPort™ and Thunderbolt to support video and other advanced features.

## Contents

The e-book covers 10 chapters across 72 pages:

1. **Basics of USB Type-C** — connector, data speeds, power levels, data/power roles (DFP, UFP, DRD, source, sink, DRP), pinout, reversibility, cable detection and orientation via CC lines, when a PD controller is needed
2. **History of USB Type-C** — connector evolution from Type-A/B to Type-C, USB PD protocol history from 1.0 to 3.1, USB-C vs. USB PD distinction, EU universalization movement
3. **Introduction and Overview of USB-C and USB PD Specifications** — connections via Rp/Rd resistors, VCONN for active cables, three messaging types (SOP/SOP'/SOP"), control/data/extended message catalog, power negotiation sequence, data-role and power-role swaps, alternate mode introduction, EPR introduction
4. **USB Signals over USB-C** — USB 2.0 signaling (low/full/high speed), SuperSpeed signaling, LFPS/LBPM speed negotiation, signal integrity challenges and conditioners
5. **Signal Multiplexing for USB Type-C** — USB 2.0 and USB 3.0 multiplexing, DisplayPort Alternate Mode pin assignments (C, D, E for both DFP_D and UFP_D)
6. **USB4** — overview, discovery and entry process requiring USB PD, system architecture (host/hub/device/peripheral routers), sideband communication via 1Mbps UART over SBU pins, lane and data rate configurations (Gen2/3/4 up to 120Gbps asymmetric), insertion loss budgets, retimer and linear redriver signal conditioning, DisplayPort and USB4 sideband multiplexing
7. **Introduction to eUSB2** — embedded USB 2.0 for sub-7nm process nodes, native mode vs. repeater mode, host/peripheral/back-to-back repeater configurations, Register Access Protocol (RAP) replacing I2C
8. **Extended Power Range (EPR)** — 240W at 48V/5A, technical specifications, safety implications above 100W, sink-driven EPR entry with keep-alive messages, adjustable voltage supply (AVS) in 100mV steps
9. **USB Type-C and USB PD Common Use Cases and Block Diagrams** — source-only, sink-only, DRP at 5V and 20V with USB PD, DisplayPort Alternate Mode, battery charger integration, with detailed block diagrams for each configuration
10. **End Equipment-Specific Block Diagrams** — laptops/industrial PCs, docking stations, Bluetooth speakers, Wi-Fi routers/smart speakers, power tools
11. **Benefits of a TI PD Controller** — integrated power paths (OCP, OVP, RCP), GUI/web-based configuration tools, USB-IF certification and validation, reference designs, customer support

## Key Specifications

- **USB PD 3.1 EPR**: 48V, 5A, 240W maximum
- **USB4 Gen 4**: 80Gbps symmetric, 120Gbps asymmetric
- **USB-C connector**: 24 pins, reversible, cold socket (0V on VBUS when disconnected)
- **CC communication**: 300kbps ±10% biphase mark code (BMC) signaling
- **eUSB2**: 1.2V/1.0V signaling for sub-7nm process nodes, backward compatible via repeaters
- **Cable requirements for EPR**: visually marked cables, higher-voltage-rated bypass capacitors, snubber circuits; 5A cables must be EPR-capable going forward

## TI-Specific Content

The final chapter is explicitly promotional, detailing TI PD controller advantages: fully integrated power paths with FETs, dead battery LDO, 26V-tolerant CC pins, GUI and web-based configuration tools (no coding required), USB-IF board membership and certification, reference designs, and E2E forum support. This is marketing, but it's also genuinely useful information for engineers selecting components — the comparison between TI's integrated approach and "typical PD controller" designs with external FETs and discrete power paths is concrete and architectural.
