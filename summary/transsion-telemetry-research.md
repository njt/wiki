---
url: https://www.nowsecure.com/blog/2026/07/08/what-the-transsion-telemetry-research-means-for-mobile-security/
title: "What the Transsion Telemetry Research Means for Mobile Security"
author: Buchodi and David Weinstein
date_fetched: 2026-07-11
date_published: 2026-07-08
---

Transsion — the world's fourth-largest smartphone maker (TECNO, Infinix, itel) — ships all devices with a first-party Android telemetry framework consisting of **Athena** (event collection) and **oneID** (cross-app tracking), both phoning home to `*.shalltry.com`.

Researchers decrypted the traffic by extracting AES-CBC encryption keys from the client binary via runtime analysis (Frida hook on a non-rooted device). The key table turned out to be a global constant stable across devices and SDK versions — the encryption was obfuscation, not genuine confidentiality.

Decrypted payloads reveal granular device-wide surveillance: GPS coordinates with cell tower IDs, foreground app tracking with ambient light levels, camera usage logging, per-app network consumption for every installed app (including TikTok, Facebook, WhatsApp, and M-Pesa), and battery/charge sessions. Each event is bound to roughly 14 persistent identifiers (gaid, oneid, device_id, chipid, etc.), none user-resettable.

The SDK ships as a system component on Transsion hardware, preinstalled and unremovable. It also appears inside popular third-party Play Store apps — Boomplay (100M+ downloads), AHA Games (500M+), Hola Browser (500M+), and others — extending the collection framework beyond Transsion's own devices.

Uploads go to Alibaba Cloud infrastructure (eu-central-1) with CloudFront CDN and GSLB-based hostname rotation. Downstream references point to Taboola and a ByteDance analytics pipeline. The recommended defense is a DNS-level wildcard block of `*.shalltry.com`, since static host blocklists fail against the rotating sink hostnames.

The piece argues that runtime analysis is essential for uncovering hidden data flows that source-code review and vendor documentation miss.
