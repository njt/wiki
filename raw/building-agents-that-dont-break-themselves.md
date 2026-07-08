---
url: https://fly.io/blog/building-agents-that-dont-break-themselves/
title: Building Agents that Don't Break Themselves
author: Daniel Botha
date_fetched: 2026-07-08
date_published: 2026-06-08
site: Fly.io Blog
---

# Building Agents that Don't Break Themselves

**Author:** Daniel Botha
**Last updated:** June 8, 2026
**Reading time:** 7 minutes

*Image: Annie Ruygt*

## Overview

The article addresses a core problem in AI agent development: agents that break themselves by running destructive commands in their own execution environment. Botha argues that "where your agent lives and where it runs code are two entirely separate considerations."

## Brains vs Hands

The piece distinguishes between the agent's reasoning loop (the "brain") and its shell execution (the "hands"). The agent process — described as "a loop" that calls a model, reads responses, and picks tools — can live comfortably on a durable, long-lived machine. Shell commands, however, should run in what Botha calls a "padded room," isolated from the agent itself.

## Two Case Studies

**SpriteDoc** (by Henrique): A multi-user troubleshooting agent where each user session spins up its own ephemeral sandbox called a Sprite. "When it's done, the Sprite goes with it." A notable design choice: user authentication tokens are injected into the environment per-command and never stored at rest, meaning "no long-lived secret ever lands on shared disk."

**Hermes Agent** (by Kyle, for Nous Research): Takes the opposite lifecycle approach, keeping "one Sprite per task and resumes it next time" so installations persist across sessions. Because commands run in a true sandbox, Hermes skips confirmation prompts on dangerous operations — "the sandbox is the security boundary now."

## Defense in Depth

Botha notes that even an agent running inside a sandbox should dispatch its commands to a *separate* sandbox. Kyle tested Hermes running within a Sprite that sent commands to another Sprite, and they returned "with a different id and a different boot." The takeaway: "The agent's home can be durable and comfortable. The place it runs untrusted strings should still be somewhere you would be happy to set on fire."

## The Undo Button

The article illustrates recovery power with a concrete example: after an agent runs `rm -rf /root/app /usr/bin/python3 /usr/bin/git` in response to a cleanup prompt, checkpointing lets you restore everything — files and tooling — "in about nine seconds." The restore uses copy-on-write, so "checkpointing before every risky step is cheap enough to be a reflex."

## Key Philosophy

Botha closes with a direct aphorism: "Telling your agent to be careful is silly. Just make it do things somewhere it doesn't have to be."
