# Everything I Own, Owned

A security professional spends two weeks of evenings pointing Claude Opus 5 at five peripherals on their desk — a webcam, monitor, microphone, capture card, and key light — and comes away with a plaintext shell in the microphone, a killable recording LED on the webcam, and a memory-write primitive on a WiFi light. The report is equal parts love letter to agentic reverse engineering (13 hours of "churn," 98 prompts, hardware "almost universally open for tinkering") and warning: assume every attached device may already carry a malicious firmware implant, and the per-model labor that once made that a state-actor activity has just been handed to an agent.

---

## Key Quotes

> "Peripherals have proven to be an ideal target for agentic RE - they're tiny computers attached to my computer, with a data connection to the host and usually a firmware update mechanism, so an agent has something to iterate against."

The thesis in one line. A peripheral is a self-contained machine with a USB/wireless interface, an update path, and live hardware to test against — a closed loop. Compare [[Giving Your Agent Eyes with Game Boy Hacking]], where the feedback loop was an *emulator*; here it's the real device on the desk. The author noticed what Langworth noticed, then pointed it at hardware the reader actually owns.

> "`su sup` just works, and the top tier can disable the touch panel so you can't mute at the device, and drive the mute LED independently of whether the microphone is actually muted. It's the webcam LED trick again, on a microphone."

The single most damning sentence in the piece. A four-tier privilege system on a Shure MV7 whose *entire authentication is a string comparison against the name of the tier you asked for*. The mute LED can be driven independently of actual mute state — meaning an implant could make a live microphone display as muted. This is the concrete consequence of the abstract fear: the trust indicator is part of the attack surface.

> "This means that a single HTTP POST of `ATSE=0200ED94,0E001009` turns the signature check into a no-op, and we can freely update to a firmware image without a legitimate signature."

The one device with real protection — Ed25519 over SHA-512 on the Elgato Key Light Mini — turns out to protect the firmware at exactly one point in time. The signature check happens during update only, not at boot, and the updater runs while the rest of the device is live. So an HTTP POST that pokes a memory address disables the check. This is the recurring lesson of the whole piece, the same one [[DeepSeek Reverse Engineers TeamSpeak Licensing]] found in software: deep protection at one layer, nothing behind it.

> "I would work from the operating assumption that any device attached to a computer could have had a malicious firmware implant performed, where previously that required significant per-model investment and was stereotyped as a 'state actor' kind of activity."

The threat-model shift in one line. The cost curve that used to confine firmware implants to nation-states has collapsed, because the two inputs — per-model reverse engineering and hardware-in-hand validation — have both been de-scarce-ified: the first by agents, the second by the fact that malware already sits on the infected host. This is the same economics argument as [[Web Application and API Protection (WAAP)]]'s "the barrier didn't just lower — it collapsed," applied to silicon rather than binaries.

---

## Key Themes

#reverse-engineering #agentic-development #hardware-security #firmware #embedded #supply-chain #threat-model #webhid #webusb

---

## Critical Analysis

**The empirical payload is the point.** The author front-loads numbers — 13 hours of churn, 98 prompts, per-device breakdowns — and links each device to a GitHub repo of "generated-slop docs and scripts, most of which have been validated live against real hardware." The self-deprecation is doing defensive work: unlike a hot take about agentic RE, this is reproducible. The distinction between "churn" (Claude actually working) and wall-clock time is a small methodological honesty that most AI field reports skip.

**The vulnerability findings are banal — and that's the finding.** No novel exploitation techniques here. MD5 integrity hashes, no signature validation, I2C-over-USB, a string-compare auth. The shocks are (a) that this is *universal* across five devices from four different vendors, and (b) that the only vendor who did it right (Elgato, Ed25519) still got the security model wrong by checking only at update time. The piece is stronger as a *survey* of how uniformly bad peripheral security is than as any single technical breakthrough.

**The refusal question hangs over this and the author doesn't address it.** [[DeepSeek Reverse Engineers TeamSpeak Licensing]] documented Claude refusing binary RE on safety grounds, forcing a model swap. Here Claude Opus 5 cheerfully disables a webcam's recording LED and writes firmware to a mic — because the goals were framed as interoperability and security research on the author's own property. Nobody draws the line, but the contrast is instructive: the same capability, compliant or refused depending on framing and ownership. "It's my own hardware" is doing a lot of unexamined moral work in this essay.

**The worm speculation is the weakest part and the author knows it.** The closing riff — an "AI-equipped automatically-reverse-engineering worm" that probes its environment and pushes into adjacent accessories — is flagged as "only a tiny leap" but the leap is doing real work. A worm that ships its own LLM-driven RE loop to every victim is not a one-line change from "an agent RE'd my webcam for me"; it's a completely different engineering problem (model size, compute, reliability on hostile hosts). But the *structural* point underneath it is sound and worth taking seriously: the two scarce inputs to hardware implants have both been de-scarce-ified, and nobody has a defense story for "your microphone is now a keyboard." That's the essay's real thesis, and it survives the purple prose.

**It slots cleanly into a lineage this wiki already tracks.** [[Giving Your Agent Eyes with Game Boy Hacking]] found the feedback loop; [[DeepSeek Reverse Engineers TeamSpeak Licensing]] found the economics; [[Kuna — Agent-First Decompiler]] found the tool-building. This piece is the security corollary — what the same loop does when pointed at the hardware you own, and what it means that it's now *cheap* for everyone, not just the curious. The author is the first in this lineage to ask the question the others skip: what happens when the attacker is the one running the loop.

---

## Connections

- [[Giving Your Agent Eyes with Game Boy Hacking]] — the direct ancestor: same feedback-loop insight, but pointed at live attached hardware instead of an emulator, with a security conclusion Langworth never draws.
- [[DeepSeek Reverse Engineers TeamSpeak Licensing]] — the economic collapse of RE, and the unaddressed refusal contrast (Claude refused there, cooperated here).
- [[Web Application and API Protection (WAAP)]] — the "client is its own attack surface" argument, extended from binaries to firmware.
- [[Transsion Telemetry — Embedded Mobile Surveillance]] — the "your device is not your own" theme at industrial scale; this piece shows the same weakness is now trivially exploitable by one person in an evening.
- [[AI Brain for Flipper]] — the other side of agentic hardware hacking: lowering the barrier for legitimate research vs. misuse, with a responsible-design answer this piece doesn't attempt.

---

*Sources: [[raw/everything-i-own-owned]], [[summary/everything-i-own-owned]]*
*Last updated: 2026-08-25*
