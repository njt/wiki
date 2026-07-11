---
url: https://www.nowsecure.com/blog/2026/07/08/what-the-transsion-telemetry-research-means-for-mobile-security/
title: What the Transsion Telemetry Research Means for Mobile Security
author: Buchodi and David Weinstein
date_fetched: 2026-07-11
date_published: 2026-07-08
site: NowSecure Blog
---

# What the Transsion Telemetry Research Means for Mobile Security

**Authors:** Buchodi (security researcher) and David Weinstein (CPTO, NowSecure)
**Published:** July 8, 2026
**Source:** NowSecure Blog

---

## Overview

Transsion — the world's fourth-largest smartphone manufacturer, whose brands include TECNO, Infinix, and itel — ships all devices with a first-party Android telemetry framework consisting of **Athena** (event collection) and **oneID** (cross-app tracking). Both systems communicate with `*.shalltry.com`. Researchers decrypted the traffic by extracting encryption keys from the client binary using runtime analysis, revealing the full scope of data collection.

## How the Encryption Was Broken

The telemetry endpoint uses AES-CBC with a fixed IV ("abcdefghijk1mnop") and a 64-key lookup table that turned out to be a **global constant** stable across devices and SDK versions (2.3.x through 3.1.x). The researchers recovered the key table via a Frida hook on a non-rooted device, reading the in-memory key list after repackaging the Weather app with the Frida gadget.

> "The encryption here is obfuscation of collection, not confidentiality against anyone holding the client binary."

## What Data Is Collected

Decrypted payloads reveal granular device-wide surveillance:

- **GPS coordinates** (latitude/longitude), geohash, and surrounding cell tower IDs with signal levels — emitted by `com.android.settings` and the AI voice assistant
- **Foreground app tracking** — `com.android.systemui` reports which app is active moment-to-moment with timestamps and ambient light level
- **Camera usage** — `com.transsion.trancare` logs which app opened the camera
- **Per-app network usage** — `com.hoffnung` reports data consumption for every installed app (including TikTok, Facebook, WhatsApp, M-Pesa, betting apps) along with install source and connection type
- **Battery/charge sessions** and device serial

Each event is bound to roughly **14 persistent identifiers** (gaid, oneid, device_id, vaid, chipid, etc.), none user-resettable, making the data joinable across all apps and time.

## Privileged System Access

The SDK ships in two forms: a sandboxed app-level version, and a **system-component (-sys) build** embedded in system partitions. The central hub is `com.hoffnung` (labeled "TPMS"), whose manifest declares extensive permissions including `PACKAGE_USAGE_STATS, QUERY_ALL_PACKAGES, READ_CLIPBOARD_IN_BACKGROUND`.

> "The same SDK, granted system rights, stops being 'an app reporting itself' and becomes a device-wide agent."

On Transsion hardware it is **preinstalled and unremovable** without breaking the device.

## Third-Party App Distribution

Beyond Transsion phones, the Athena SDK appears inside popular third-party apps on Google Play — including **Boomplay** (100M+ downloads, owned by Transsion), **AHA Games** (500M+), **Hola Browser** (500M+), **Supermom** (100M+), **StarTimes** (10M+), and **Orange Max it** (5M+) — meaning "the collection framework is not confined to the OS layer of Transsion phones."

## Data Destination

Uploads go to `*.shalltry.com` infrastructure hosted on **Alibaba Cloud (eu-central-1)** with CloudFront CDN. A GSLB resolver distributes live sink hostnames at runtime. Downstream references point to **Taboola** (lockscreen content) and a **ByteDance "volcano" analytics pipeline**.

## Mitigation

The article advises that client-side controls are insufficient on Transsion hardware. The recommended defense is:

> "A DNS-level wildcard block of \*.shalltry.com" because "simple blocklists that rely on static leaf hosts will fail" given the GSLB-based hostname rotation.

## Key Conclusions

> "Mobile apps increasingly rely on third-party SDKs, preinstalled services and other software components that organizations don't build and often can't fully evaluate."

The research highlights the value of **runtime analysis** for uncovering hidden data flows that source code review and vendor documentation miss. The piece was produced in partnership with NowSecure's Mobile App Risk Intelligence (MARI) service.
