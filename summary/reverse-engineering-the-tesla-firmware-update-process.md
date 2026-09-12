---
url: https://www.pentestpartners.com/security-blog/reverse-engineering-the-tesla-firmware-update-process/
title: "Reverse Engineering the Tesla Firmware Update Process"
author: Pen Test Partners
date_fetched: 2026-09-13
date_published: unknown
site: pentestpartners.com
topics:
  - security-and-sandboxing
  - software-engineering-craft
---

## What this is

Pen Test Partners' security researchers spent a couple of weeks reverse-engineering how a real Tesla Model S updates its own firmware — Part 2 of a hardware series covering the CID (center touchscreen) and IC (instrument cluster), published as "the process has recently changed with the Model 3." Internal evidence (extracted VPN keys that expired 31 May 2018; Tegra public-key boot "not in place before 2015") dates the architecture to roughly 2012–2015.

## What they found

The CID is a Tegra SoC running full Ubuntu, booting through a ROM BPMP → ~56 KB "QUICKBOOT" stage1 → custom stage2 chain, with dual-bank layout everywhere: mirrored stage1/stage2 primary+recovery partitions, kernel_a/kernel_b, and online/offline read-only squashfs `/usr` partitions. Updates — applied via legacy shell scripts, a monolithic multi-personality "updater" binary (ic-updater/cid-updater/gwxfer/sm-updater/…, one binary with per-name personalities), full tar.gz packages, bsdiff40 diffs, or Redbend deltas — are patched onto the offline bank at raw-flash level, signature-checked, then redeployed. That A/B machinery is why the car still "(mostly) worked" after two weeks of researchers taking it apart.

## The security story

The one genuinely strong boundary is a per-vehicle OpenVPN tunnel (certificates keyed to the VIN, outbound from the car). Inside it, the integrity story is soft: handshakes ride plain HTTP (the updater "has no TLS functionality at all"); firmware downloads over the public Internet rely on Salsa20 encryption plus HMAC'd, expiring URLs; the legacy shell-script updater executes `install.sh` as root from a trivially re-creatable package format; MD5s and CRC32s stand in where signatures are needed (Redbend literally labels a CRC32 a "signature"); the updater's self-integrity SHA-512 is truncated to 8 of its 64 bytes; and requests could be made for any VIN using another car's VPN. ECU updates go over UDS Security Access — some ECUs answer with fixed seeds, and there is little evidence the ECUs verify firmware signatures themselves. Crucially, because Tesla updates remotely, the "programming device" with all seed/key algorithms ships inside every vehicle.

## Why it matters

A pre-agent baseline for what vehicle firmware RE used to cost — expert weeks, one vehicle, "our biggest obstacle is the lack of spare parts" — and a snapshot of 2012-era connected-car thinking: strong transport security wrapped around a trusted interior. Also documents remote feature enablement (enable-autopilot-after-purchase.sh updating gateway internal.dat), a daily security token gating diagnostics and root SSH, and update-retry escalation strategies named from "INDIFFERENT" to "SUICIDE_BOMBER."
