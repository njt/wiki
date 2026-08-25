---
url: https://schlarp.com/posts/everything-i-own-owned/
title: "Everything I own, owned"
author: schlarp
date_fetched: 2026-08-25
date_published: unknown
site: schlarp.com
---

## What this is

A security professional's field report on two weeks of agent-driven reverse engineering of five peripherals within arm's reach — an Insta360 Link webcam, an ASUS ROG Swift PG42UQ monitor, a Shure MV7 microphone, an Elgato Cam Link 4K capture device, and an Elgato Key Light Mini. Using Claude Opus 5 and a standard prompt template ("reverse engineer the update format and protocol, determine security properties, enumerate all functionality, find hidden/debug features"), schlarp turned each device into a full teardown plus working tooling in roughly 13 hours of total "churn" across 98 prompts.

## The five devices

- **Insta360 Link webcam** — runs a full ThreadX RTOS hosting vision models for face tracking and gesture detection. Two update paths: a mass-storage mode requiring a replug, and an XU command over the USB vendor class that allows arbitrary file read/write and reboot — full flash with no user interaction. The only integrity protection is an appended MD5 hash. Claude patched the LED "pattern" table to disable the recording indicator.
- **ASUS ROG Swift PG42UQ monitor** — no protection beyond a two-slot A/B scheme and a simple checksum; firmware updates over I2C bridged over USB. Started from annoyance at the un-disableable pixel-cleaning overlay. Also mapped the DDC/CI control channel into a shell script for crosshair/zoom/FPS-counter toggles.
- **Shure MV7 microphone** — firmware hidden inside the Windows-only MOTIV Mix software (extracted via Wine). The update protocol turned out to be a USB HID vendor protocol implementing a *full plaintext command shell* with 48 commands: a dozen DSP knobs, arbitrary memory read/write, LED control, and a 4-tier privilege system whose entire authentication is a string comparison — `su sup` just works. Reachable from a webpage via WebHID.
- **Elgato Cam Link 4K** — ran fully unattended overnight; MCU image plus an FPGA bitstream, EDID data extracted, and a vendor HID protocol tunneling the internal I2C bus. No protection on the update path.
- **Elgato Key Light Mini** — the only one with real integrity protection: Ed25519 over SHA-512 on firmware updates. But it protects only at update time, not at boot, and the updater runs while everything else is live — so a single HTTP POST of `ATSE=0200ED94,0E001009` pokes memory to no-op the signature check. Verified with a patch that renamed the device.

## The argument

Two conclusions. For interoperability, this is wonderful — hardware is "almost universally open for tinkering" with a couple of hours of machine-driven labor, and schlarp looks forward to modifying webcam firmware as easily as Linux software. As a security professional, it's terrifying: assume any attached device could carry a malicious firmware implant (previously a "state actor" activity), operating systems can't stop a microphone from becoming a keyboard, and WebUSB/WebHID/WebBluetooth mean a single permissions prompt could permanently backdoor a device. Network-connected gear is "near universally fucked." The closing speculation is an AI-equipped, automatically-reverse-engineering worm that probes its environment and pushes itself into adjacent accessories — the per-model RE labor was just handed to an agent, and the hardware-in-hand validation is free to malware already on an infected host.
