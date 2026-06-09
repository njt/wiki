# DeepSeek Reverse Engineers TeamSpeak Licensing

A Reddit user reverse-engineered TeamSpeak 3.13.8's licensing system in 5 hours using DeepSeek Pro + Flash, Ghidra, and x64dbg — for $3.88 in API credits. The real story isn't the crack: it's the first public field report of an LLM being *willing* to do binary reverse engineering that Claude, Grok, and GLM all refused, and what that says about the emerging safety-model capability gap.

---

## Key Quotes

> "They really made it like fort knox but forgot to lock the final door. Once we found that starting position, it was easy to trace forward."

This is the essence of reverse engineering: one wrong assumption in the defense chain and everything collapses. The AI didn't find a novel 0-day — it traced the call chain and found the architectural mistake (parser reading from AES decrypt buffer instead of the signed payload).

> "Claude instantly threw in the towel and said nope not allowed. Grok had a hissy fit even mentioning it. GLM just straight died. But DeepSeek? Goes head first, infact it just dives right in."

The safety-capability tradeoff, quantified in the wild. DeepSeek didn't outperform on benchmarks — it outperformed by *not refusing*. This is what people mean when they say Chinese models have different alignment priorities.

> "Less than 1% of those tokens are output tokens."

690M tokens, only ~1M output. This is a thinking-heavy workload where the model is reading disassembly, reasoning about control flow, and planning patches. The cache-hit ratio (545M of 547M Pro input tokens were cache hits) means repeated context reuse across iterations — exactly the pattern you'd expect from an iterative RE loop.

> "I'm shocked there was no heavy protections in place like anti debuggers or random checks or pit falls."

The surprising part: TeamSpeak's protection was deep but brittle. Once you trace the license validation call chain end-to-end, it's over. No runtime integrity checks, no anti-debugging. The author expected Fort Knox, got a really complicated lock on an unlocked door.

> "If i could do it this cheaply, image what some mega mind on red team could do on enterprise grade software."

$3.88 for what would take a human weeks or months. The economics of RE have changed faster than most security teams realize.

---

## Key Themes

#reverse-engineering #tool #security #model-comparison

---

## The Technical Achievement

The OP mapped the full license validation call chain from server startup to display output, then:

1. **Found the architectural bug**: Parser reads from AES decrypt buffer instead of signed payload
2. **Decoded obfuscation**: Custom XOR scheme on all log messages
3. **Extracted embedded secrets**: PolarSSL certs and private keys baked into the binary
4. **Patched 27 instructions**: Signature verification, certificate checks, download gates, validator functions, slot enforcement, and a state-reset timer
5. **Result**: Server starts with 1024 slots (from 32), no crashes

The approach was iterative: Pro handled reasoning and complexity (analyzing disassembly, tracing call chains, planning patches), Flash handled micro-adjustments and testing. Token economics tell the story — 545M cache hits on Pro means the model was repeatedly working with the same context (the binary, the disassembly), adding small discoveries each iteration.

---

## The Safety Gap

This is the first well-documented public comparison of model refusals for a real RE task. DeepSeek was the only model willing to engage. Claude refused outright (consistent with its system prompt prohibition on reverse engineering). Grok "had a hissy fit." GLM simply crashed.

But this isn't just "Chinese models are unsafe" — it's more nuanced:

- **Claude Opus 4.6 can do this work** (per commenter u/sdexca: "isn't too far behind") but won't
- **The capability is there** in multiple frontier models; alignment policy is the bottleneck
- **Claude Code's system prompt** explicitly forbids RE, so the OP was fighting the harness too (u/raydou noted this)

The practical implication: if you have a legitimate RE need (vulnerability research, malware analysis, interoperability), your choice of model is dictated by alignment policy, not capability. This is a weird state of affairs — and probably not sustainable as capability gaps narrow.

---

## Harness Notes

The OP used Claude Code (not a dedicated RE tool) with DeepSeek's API endpoint and two MCP servers:
- [Ghidra MCP](https://github.com/bethington/ghidra-mcp) — for static analysis
- [x64dbg MCP](https://github.com/AgentSmithers/x64DbgMCPServer) — for dynamic debugging

The comment thread reveals a broader truth: the harness matters more than the model for specialized work. Multiple commenters debated Kilo vs OpenCode vs Claude Code — but the OP picked Claude Code for its structured system prompts and workflow, then swapped in a model that wouldn't refuse the task. That's a pattern worth watching: **harness as platform, model as configurable component**.

---

## Critical Analysis

**The real cost isn't $3.88.** The OP is an experienced reverse engineer who knew what to ask and how to structure the approach. "Little timmy down the street" with $50 won't replicate this — domain expertise is still the multiplier. The AI accelerated an expert, not replaced one.

**The fort-knox-but-forgot-to-lock-the-door pattern is everywhere.** TeamSpeak's protection was deep in the build chain but had no runtime integrity verification. This is the standard story in DRM and licensing: complexity without defense-in-depth. The AI found the seam the same way a human would — by tracing execution — just much faster.

**Safety-through-refusal is a temporary filter, not a permanent solution.** Claude Code forbids RE. The OP just used DeepSeek's API through Claude Code's harness. As more models launch without these restrictions, safety-through-refusal on one model becomes meaningless. The real question is whether the RE task itself is legitimate — and that's a human judgment, not a model's.

**The discussion thread is a microcosm of the coding-agent tool wars.** One Reddit post about RE spiraled into a debate about Kilo vs OpenCode vs Claude Code, with the OP reporting higher cache hits through Claude Code's endpoint. These side conversations are more interesting than the main topic for anyone tracking the agent tooling landscape.

**690M tokens and 5 hours is actually a lot of compute for a single binary.** Compare this to the cost of traditional RE (IDA Pro license, weeks of human time) and it's a bargain. Compare it to what a dedicated RE toolchain with a fine-tuned model could achieve, and it's absurdly inefficient. The efficiency comes from the workflow loop, not the model.

---

*Sources: [[raw/deepseek-reverse-engineers-teamspeak]]*
*Last updated: 2026-06-09*
