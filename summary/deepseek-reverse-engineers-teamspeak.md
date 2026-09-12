---
url: https://www.reddit.com/r/DeepSeek/comments/1txcfrh/with_388_690003591_tokens_and_5_hours_deepseek/
title: "With $3.88 & 690,003,591 tokens and 5 hours, DeepSeek Pro & Flash combined, managed to reverse engineer TeamSpeak's Licensing System for 3.13.8"
author: u/cyb3rofficial
date_fetched: 2026-06-09
date_published: 2026-06-05
platform: Reddit (r/DeepSeek)
topics:
  - ai-research-and-models
---

## Original Post

With $3.88 & 690,003,591 tokens and 5 hours, Deepseek Pro & Flash combined, managed to reverse engineer Teamspeak's Licensing System for 3.13.8 (latest at time of post).

No I will not release it, so don't ask, but Deepseek is very powerful if given the proper tools and if you know what you are doing. In 5 hours of trial and error, debugging with Ghidra and x64dbg, the models are really good with IoT hacking and reverse engineering. We mapped the full license validation call chain from server startup through to the display output. Found that the parser reads from an AES decrypt buffer instead of the signed payload (easy fix once you know), decoded a custom XOR obfuscation scheme for all log messages, extracted the embedded PolarSSL certs and private keys, and patched 27 instructions across the binary to bypass signature verification, certificate checks, download gate checks, validator functions, slot enforcement, and a state reset timer callback that kept overwriting our values. They really made it like fort knox but forgot to lock the final door. Once we found that starting position, it was easy to trace forward. I'm shocked there was no heavy protections in place like anti debuggers or random checks or pit falls. For something they heavily sell on, sure was left wide open once the path was found. The server now starts with 1024 slots instead of 32, enforcement is bypassed so the API accepts the servercreate command with the slots, and there are no crashes. Total cost: $3.88 in API credits. 690 million tokens. 5 hours. Really not bad for what would take a human weeks if not months. If i could do it this cheaply, image what some mega mind on red team could do on enterprise grade software.

## Token Breakdown (OP comment)

**DeepSeek Pro ($3.25):**
- 545,976,064 Cache Hit input tokens
- 1,390,426 Cache Miss input tokens
- 783,398 Output tokens

**DeepSeek Flash ($0.68):**
- 157,831,040 Input (Cache hit)
- 1,337,769 Input (Cache miss)
- 239,799 Output

Pro was mostly used for reasoning and complexity, flash was used for final end goals, testing and micro adjustments. Less than 1% of total tokens were output tokens.

## OP's Harness Setup

Uses Claude Code with DeepSeek's API endpoint (Claude Code API compatibility). Tools: Ghidra MCP server (https://github.com/bethington/ghidra-mcp) and x64dbg MCP server (https://github.com/AgentSmithers/x64DbgMCPServer). Reports higher cache hits than most other tools used.

## Model Safety Comparison (OP comment)

OP's experience testing models on reverse engineering task:
- **DeepSeek**: "Goes head first, infact it just dives right in"
- **Claude**: "Instantly threw in the towel and said nope not allowed"
- **Grok**: "Had a hissy fit even mentioning it"
- **GLM**: "Just straight died when even mentioning reverse engineering and poc idea"

OP: "Making explosives = bad, cracking software? Goes head first."

OP notes he's already experienced in reverse engineering, so had the advantage of knowing what to do and how to structure goals and prompts. "Pretty sure little timmy down the street could do it with $50."

## Selected Comments

**u/--Spaci--**: "Less than 1% of those tokens are output tokens"

**u/sdexca** (Top 1% Commenter): "Yeah deepseek is the best in this regards, it's the only one which is willing to crack software. Claude Opus 4.6 isn't too far behind but won't crack software. I didn't realize this was the latest version, I thought it was legacy version."

**u/raydou**: "Maybe if you used another harness than Claude code it would have been easier for you. Claude code's system prompt forbid reverse engineering and even white hat hacking. How much time have you spent reworking your prompts?"

**u/Tarul-etek**: "I am more interested in how you got it to do it rather than the crack itself. I know you can tune your request so its palatable but sometimes it's very stubborn, even for legitimate requests."

**u/lab34fr**: Asked about Ghidra MCP server and harness.

Side discussion about coding agent tools: u/Choice-Principle9947 asked about using DeepSeek API with Antigravity-like coding tools. u/ImagineEyes recommended opencode (free tier of v4 flash). u/VehiculeUtilitaire recommended kilo code for VS Code. u/ClearRabbit605 noted that "the real advantages come from how the tool is structured and the system prompts are made. Claude code is really great at that."
