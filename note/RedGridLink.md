# RedGridLink

Offline MGRS navigation plus BLE proximity team sync for 2-8 people. No cell service needed. A Flutter app that turns phones into tactical navigation devices with encrypted peer-to-peer position sharing over Bluetooth Low Energy.

---

## Key Themes

#hardware #offline #navigation #ble #tactical

This sits at the intersection of outdoor navigation and mesh networking. MGRS (Military Grid Reference System) coordinates with meter-level precision, offline maps via MBTiles, 11 tactical navigation tools (dead reckoning, resection, pace counting, celestial navigation, etc.), and BLE-based team position sharing with sub-200-byte delta payloads.

The security model is thoughtful: three tiers from open (training) through PIN-protected (AES-256-GCM) to QR-authenticated (ECDH P-256 ephemeral key exchange). No external servers touch position data. Device-to-device encryption only.

Four mission modes (Search & Rescue, Backcountry, Hunting, Training) adapt the interface to context. Battery optimization is impressive: Expedition Mode claims less than 3% per hour, Ultra Expedition less than 2%, using 30-60 second update intervals over BLE-only.

The v1.3 role system (Lead, Scout, Medic, Comms, custom callsigns) with shared waypoints, collaborative annotations, and NATO phonetic voice callouts shows someone who actually uses this in the field.

## Critical Analysis

This is genuinely useful for anyone who operates in areas without cell service -- SAR teams, backcountry groups, hunters. The BLE range limitation (100-300m standard, up to 1km with Long Range) constrains the use case to groups that stay relatively close together.

iOS-only for now, with Android targeting Q3 2026. The pricing model (free for 2-device, paid for more) is reasonable. The ATAK/CoT military interoperability planned for V2.0 would open this to a much larger user base.

The privacy stance (no analytics, no advertising, no device identifiers collected) is refreshing for a mobile app. See also [[AI Brain for Flipper]] for another piece of hardware-adjacent tooling with security considerations.

---
*Sources: [[summary/redgridlink]]*
*Last updated: 2026-05-14*
