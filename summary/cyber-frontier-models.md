---
url: https://blog.cloudflare.com/cyber-frontier-models/
title: "Project Glasswing: What Mythos Showed Us"
author: Grant Bourzikas
date_fetched: 2026-05-18
date_published: 2026-05-18
---

# Project Glasswing: What Mythos Showed Us

**Author:** Grant Bourzikas
**Date Published:** May 18, 2026
**Reading Time:** 9 minutes

## Overview

Cloudflare spent months testing security-focused LLMs against their own infrastructure, with particular focus on Anthropic's **Mythos Preview** model. They pointed it at over 50 of their own repositories as part of Project Glasswing, an Anthropic initiative.

## What Changed with Mythos Preview

The author states plainly that Mythos Preview represents "a real step forward" and characterizes the jump from prior frontier models as "not just a refinement of what came before" but "a different kind of tool doing a different kind of work."

Two standout capabilities are highlighted:

- **Exploit chain construction:** Rather than finding isolated bugs, Mythos Preview can reason about combining several low-severity primitives into a working exploit chain. The author notes the reasoning process "looks like the work of a senior researcher rather than the output of an automated scanner."

- **Proof generation:** The model writes triggering code, compiles it, runs it, and iterates on failures. The ability to close the loop from suspicion to working proof is what differentiates it, because "a suspected flaw without a working proof is speculation."

Other frontier models found similar underlying bugs and, in some cases, "got further than we expected on the reasoning side too." Their shortcoming was at the stitching stage — they'd identify a bug, describe why it mattered, and stop, leaving exploitability unconfirmed. Mythos Preview now chains those low-severity bugs (which "would traditionally sit invisible in a backlog") into single, more severe exploits.

## Model Refusals in Legitimate Vulnerability Research

The version of Mythos Preview used in Project Glasswing lacked the additional safeguards present in generally available models like Opus 4.7 or GPT-5.5. Nevertheless, the model showed "emergent guardrails" that sometimes caused it to push back on legitimate security research requests. However, these organic refusals proved inconsistent: the same task framed differently could produce opposite outcomes, and even identical requests could yield different results across runs due to the model's probabilistic nature. The post argues this inconsistency means emergent guardrails alone "aren't consistent enough to serve as a complete safety boundary on their own," and any future generally available cyber frontier model must include additional safeguards on top.

## The Signal-to-Noise Problem

Two factors dominate noise in AI vulnerability scanning:

- **Programming language:** C and C++ projects produced consistently more false positives than memory-safe languages like Rust.
- **Model bias:** Models lack the confidence calibration of human researchers. Findings come back hedged with words like "possibly" and "potentially," with hedged results vastly outnumbering solid ones. The author calls this "a reasonable bias for an exploratory tool" but "a ruinous one for a triage queue."

Mythos Preview represented "a clear improvement" here, especially in chaining primitives with working PoCs. At triage time, its output showed "fewer hedged findings, clearer reproduction steps, and less work to reach a fix-or-dismiss decision."

## Why Pointing a Generic Coding Agent at a Repo Doesn't Work

Two reasons are given:

- **Context:** Coding agents hold a single hypothesis and iterate — the wrong shape for vulnerability research, which is "narrow and parallel by nature." A single agent against a large repository covers perhaps "a tenth of a percent of the surface" before context compaction discards earlier findings.
- **Throughput:** Real codebases need many concurrent hypotheses. Pushing a single agent harder eventually hits limits imposed by interaction shape, not model capability. The model is fine as "a second pair of eyes" for a researcher with a lead, but it's "the wrong tool for achieving high coverage."

## What a Harness Actually Fixes

Four lessons from running the work at scale:

1. **Narrow scope produces better findings** — Broad prompts make the model wander; specific scope hints produce researcher-like behavior.
2. **Adversarial review reduces noise** — A second agent with a different prompt and model, incapable of generating its own findings, catches noise a self-reviewing agent would miss. Putting "two agents in deliberate disagreement" proved far more effective than telling one agent to be careful.
3. **Splitting the chain across agents produces better reasoning** — Asking "Is this code buggy?" and "Can an attacker reach this bug?" are separate questions best answered separately.
4. **Parallel narrow tasks beat one exhaustive agent** — Coverage improves with many agents on tightly scoped questions plus deduplication.

## The Vulnerability Discovery Harness

The harness operates across Cloudflare's runtime, edge data path, protocol stack, control plane, and open-source dependencies through nine stages:

1. **Recon** — Reads the repository, fans out to subagents, produces an architecture document covering build commands, trust boundaries, entry points, and attack surface, and generates an initial task queue.
2. **Hunt** — Each task pairs one attack class with a scope hint. About 50 hunters run concurrently, each fanning out to exploration subagents with tools to compile and run PoC code.
3. **Validate** — An independent agent tries to disprove the original finding, using a different prompt with no ability to emit new findings.
4. **Gapfill** — Under-covered areas get re-queued for another pass, counteracting "the model's tendency to drift toward attack classes it has already had success with."
5. **Dedupe** — Findings sharing root cause collapse into a single record; variant analysis is treated as a feature, not an inflation mechanism.
6. **Trace** — For confirmed findings in shared libraries, a tracer fans out per consumer repository, uses a cross-repo symbol index, and determines whether attacker-controlled input actually reaches the bug from outside. This stage "matters most," turning "there is a flaw" into "there is a reachable vulnerability."
7. **Feedback** — Reachable traces become new hunt tasks in consumer repositories where the bug is actually exposed.
8. **Report** — A structured report is written, validated against a schema, and submitted to an ingest API so output is "queryable data, not free-form prose."

## What This Means for Security Teams

The author pushes back on the instinct to simply accelerate patching. Many teams now operate under two-hour SLAs from CVE to patch. The warning: "Patching faster does not change the shape of the pipeline that produces the patch." If regression testing takes a day, a two-hour SLA requires skipping it, and "the bugs you ship when you skip regression testing tend to be worse than the bugs you were trying to patch." Cloudflare learned this firsthand after letting the model write patches that "fixed the original bug while quietly breaking something else."

The harder question is architectural: making exploitation harder even when bugs exist, so the disclosure-to-patch gap matters less. That means front-end defenses, application design containing flaws, and the ability to roll out fixes everywhere simultaneously without waiting on individual teams.

The post also acknowledges the dual-use concern: the same capabilities that found Cloudflare's bugs "will, in the wrong hands, accelerate the attack side against every application on the Internet." Cloudflare's position in front of millions of applications means the architectural principles described mirror those their products are built to deliver for customers.

**Contact:** security-ai-research@cloudflare.com

**Credits:** Albert Pedersen, Craig Strubhart, Dan Jones, Irtefa Fairuz, Martin Schwarzl, and Rohit Chenna Reddy.

**Tags:** Security, AI, Agents, Threat Intelligence, LLM, Risk Management, Threat Operations, Automation, Engineering
