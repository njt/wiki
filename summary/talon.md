---
title: "Talon"
url: https://talonvoice.com/
author: Ryan Hileman (lunixbochs)
date_fetched: 2026-05-31
section: "Local & Personal Computing"
topics:
  - developer-tools
---

# Talon: Hands-Free Computer Control

Talon is a multimodal, hands-free computer input system enabling users to control their computer through voice commands, mouth-sound clicks ("noise control"), eye tracking, and customizable Python scripts. Created by Ryan Hileman (lunixbochs), it runs on macOS, Linux (X11), and Windows.

## How It Works

Four input modalities combine to replace keyboard and mouse:

1. **Voice Control** — Speak commands to control the computer. Goes beyond dictation: "delete line," "open Chrome," "scroll down" are native commands.
2. **Noise Control** — Mouth sounds (pops, clucks, hisses) mapped to mouse clicks and other actions. A back-beat pop becomes a click.
3. **Eye Tracking** — Gaze-based cursor control. Look at a screen region, make a noise or say a command to interact with it.
4. **Python Scripts** — Full extensibility. Users write Python scripts to define custom commands, grammars, and integrations with any application.

## Pricing Model

Talon operates on a Patreon patronage model (patreon.com/join/lunixbochs), not a fixed-price product. Supporters get early feature access and high-priority support. This is unusual for a productivity tool — it's community-funded shareware in the age of SaaS subscriptions.

## Platforms

- macOS (.dmg)
- Linux X11 (.tar.xz)
- Windows (.exe installer + portable .zip)

## Community

- Documentation at talonvoice.com/docs
- Slack workspace at talonvoice.com/chat
- YouTube playlist for demos
- Twitter: @lunixbochs

## Architecture Notes

Talon is proprietary (EULA at talonvoice.com/EULA.txt). The Python scripting layer sits on top of a native recognition engine. The eye tracking integration supports hardware like Tobii eye trackers. The noise recognition system uses distinctive mouth-sound patterns that can be reliably distinguished from speech — this is the most technically novel piece: using non-verbal vocal input as a discrete control channel alongside continuous speech recognition.

Unlike cloud-based voice assistants, Talon processes speech locally. This is critical for latency (sub-100ms command recognition) and privacy.

## Key Differentiators

- **Not a dictation tool.** Talon is a computer control system. The voice commands are structured grammars, not free-form transcription.
- **Multi-modal from the ground up.** Voice + noise + eyes + scripts aren't bolted on — they're co-designed input channels.
- **User-programmable.** The Python scripting layer means users define their own interaction models. This creates a community of shared command sets for specific applications (VSCode, terminal, browsers, etc.).
- **Patronage-funded.** No VC, no SaaS pricing — a single developer funded by the community.

## Source Content

The landing page at talonvoice.com is sparse — it's a download portal and link hub. The real documentation lives at talonvoice.com/docs and the community distributes knowledge through Slack and YouTube. For a full technical understanding, the documentation site and community command sets would need to be examined.
