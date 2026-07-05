---
title: "RedGridLink"
url: https://github.com/RedGridTactical/RedGridLink
date_fetched: 2026-05-14
section: "Random"
---

# RedGridLink: Offline Team Navigation

Flutter-based mobile app for small teams (2-8 people) to coordinate positions without cellular infrastructure. BLE proximity sync with encrypted data exchange.

Navigation: MGRS coordinates with meter-level precision, offline maps (OpenStreetMap/OpenTopoMap MBTiles), 11 tactical nav tools including dead reckoning, resection, pace counting, bearing calculations, celestial navigation.

Field Link Team Sync: Zero-config peer-to-peer via BLE. ~100-300m standard range, 400m-1km with BLE Long Range. Delta payloads under 200 bytes. Three security tiers: Open (training), PIN (AES-256-GCM), QR (ECDH P-256 ephemeral key exchange).

Four mission modes: Search & Rescue, Backcountry, Hunting, Training.

Battery: Expedition Mode <3%/hr, Ultra Expedition <2%/hr (30-60 second update intervals, BLE-only).

Team features (v1.3): Role-based management with callsigns (Lead, Scout, Medic, Comms), shared waypoints, collaborative annotations, boundary geofences with crossing alerts, NATO phonetic voice callouts.

Privacy: No external servers, device-to-device encryption, no analytics tracking, no advertising networks.

iOS production-ready. Android targeting Q3 2026. Pricing: Free (2-device), Pro ($3.99/mo), Pro+Link ($5.99/mo), Team ($199.99/yr), Lifetime ($149.99).

Future: ATAK/CoT interop (V2.0), GPX recording (V2.1), cloud relay (V3.0), Garmin inReach (V3.1).
