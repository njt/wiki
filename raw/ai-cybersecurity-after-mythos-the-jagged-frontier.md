---
url: https://aisle.com/blog/ai-cybersecurity-after-mythos-the-jagged-frontier
title: "AI Cybersecurity After Mythos: The Jagged Frontier"
author: Stanislav Fort, Founder and Chief Scientist at AISLE
date_fetched: 2026-05-15
date_published: 2026-04-07
---

# AI Cybersecurity After Mythos: The Jagged Frontier

Stanislav Fort argues that while Anthropic's Mythos announcement is significant, the framing that such cybersecurity work "fundamentally depends on a restricted, unreleased frontier model" is overstated. His central claim: **"the moat in AI cybersecurity is the system, not the model."**

## Why the Moat is the System, Not the Model

Fort introduces the "jagged frontier" concept -- AI cybersecurity capability "doesn't scale smoothly with model size." He tested the specific vulnerabilities Anthropic showcased on small, cheap, open-weights models, and they "recovered much of the same analysis."

## Context

Anthropic announced Claude Mythos Preview and Project Glasswing on April 7, 2026, describing "Mythos autonomously finding thousands of zero-day vulnerabilities across every major operating system and web browser." The post detailed a 27-year-old OpenBSD bug, a 16-year-old FFmpeg bug, and multi-vulnerability exploit chains.

AISLE's track record includes: 15 CVEs in OpenSSL (including "12 out of 12 in a single security release" with a CVSS 9.8 Critical), 5 CVEs in curl, and over 180 externally validated CVEs across 30+ projects. Their security analyzer now runs on OpenSSL, curl, and OpenClaw pull requests.

## Decomposing the Pipeline

Fort breaks AI cybersecurity into five modular tasks:
1. Broad-spectrum scanning
2. Vulnerability detection
3. Triage/verification
4. Patch generation
5. Exploit construction

These have "vastly different scaling properties." Blending them into one narrative creates the false impression "that all of them require frontier-scale intelligence."

## The Evidence: Three Tests

### Test 1 -- OWASP False-Positive Discrimination

Models were given a Java snippet that looked like SQL injection but wasn't -- after a `remove(0)` operation, `get(1)` returned a constant string, not user input. Results showed "something close to inverse scaling: small, cheap models outperform large frontier ones." DeepSeek R1, GPT-OSS-20b (3.6B params), and OpenAI o3 got it right. Every GPT-4.1 model, every GPT-5.4 model except o3 and pro, and most Anthropic models (Sonnet 4.5, Opus 4.5) failed. Sonnet 4.5 confidently mistraced: "Index 1: param → this is returned!"

### Test 2 -- FreeBSD NFS Detection (CVE-2026-4747)

Eight out of eight models detected the stack buffer overflow in `svc_rpc_gss_validate`, including a 3.6B-parameter model at "$0.11 per million tokens." All models correctly identified exploitation constraints (no stack canary, KASLR disabled, ROP as right technique). GPT-OSS-120b produced a gadget sequence matching the actual exploit. Kimi K2 independently noted the vulnerability is "wormable" -- a detail Anthropic's post didn't highlight.

When given the payload-size constraint (304 bytes for a 1000+ byte chain), none arrived at Mythos's multi-round RPC approach, but several proposed alternatives. DeepSeek R1 concluded "304 bytes is plenty" for a minimal privilege escalation chain via `prepare_kernel_cred(0)`/`commit_creds`.

### Test 3 -- OpenBSD SACK Bug

The most technically subtle test -- reasoning about signed integer overflow in TCP SACK handling across a 27-year-old codebase. GPT-OSS-120b (5.1B active params) "recovered the core public chain in a single call" and proposed the correct mitigation. Qwen3 32B -- which scored a perfect CVSS 9.8 on the FreeBSD test -- confidently declared "The code is robust to such scenarios."

### April 9 Update: Sensitivity vs. Specificity

A reader noted GPT-OSS-20b reported the *patched* FreeBSD code as still vulnerable. Fort re-ran all models on both versions three times each. Every model found the bug in unpatched code (100% sensitivity). But on patched code, only GPT-OSS-120b was perfectly reliable (3/3 correctly calling it safe). Others false-positived, fabricating arguments about signed-integer bypasses -- wrong because `oa_length` is `u_int` (unsigned). Fort argues this "is exactly why the scaffold and triage layer are essential."

## What About Exploitation?

Fort distinguishes "can reason about exploitation" from "can independently conceive a novel constrained-delivery mechanism." The creative engineering insight -- "treating the bug as a reusable building block" across multiple requests -- is "where Mythos-class capability genuinely separates." But he notes this wasn't tested with agentic infrastructure, and "with actual tool access, the gap would likely narrow further."

## The Bigger Picture

Fort calls the Mythos announcement "very good news" that "validates the category." But he warns that a framing concentrating capability "behind a single API" could discourage organizations from adopting AI security tools today. He argues discovery-grade capabilities are "broadly accessible with current models, including cheap open-weights alternatives."

Because small, fast models suffice for much detection work, defenders can deploy cheap models broadly and "compensate for lower per-token intelligence with sheer coverage." This compares to "a thousand adequate detectives searching everywhere" finding more bugs than one brilliant detective guessing where to look.

## Caveats

Fort acknowledges the tests gave models the vulnerable function directly with contextual hints, making this "an upper bound" on autonomous performance. No agentic testing was done. He is "not claiming Mythos is not capable" -- rather that "the framing overstates how exclusive these capabilities are."

## Key Data Point

"Eight out of eight models detected Mythos's flagship FreeBSD exploit, including one with only 3.6 billion active parameters."

## Closing Message

Fort urges defenders to start building now: "the scaffolds, the pipelines, the maintainer relationships, the integration into development workflows. The models are ready."
