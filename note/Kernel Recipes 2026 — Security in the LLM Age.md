# Kernel Recipes 2026 — Security in the LLM Age

Greg Kroah-Hartman's Kernel Recipes 2026 talk is the receiving end of the AI vulnerability-discovery story: the Linux kernel security team drowning in LLM-generated bug reports and patches, and their playbook for surviving it. He debunks the Mythos "79 bugs" headline with the raw data (only 20 real fixes — one hour of kernel development), catalogues the pathologies of LLM security output (50% wrong patches, sycophancy, data leaks, dated idioms), and argues the response is boring: document threat models, demand patches, push back, run local models, and grind it out like the fuzzer flood.

---

## Key Quotes

> "24 of them were nothing... 3 of them were completely made up... 11 of them were famously found and fixed by other people in public before this report even came out... In the end, Mythos broke down to 10 real bug fixes... one hour of kernel development."

The demystification of the "79 vulnerabilities" headline. This is the ground truth that vendor announcements compress away: a finder that produces 79 reports of which 10–20 are real has a false-positive rate around 75–87%. Useful, but a fraction of the marketing.

> "I don't like using the word hallucination because that gives an idea that there's an entity behind this stuff. It's just fake. They made up data out of thin air."

A sharper epistemology than most commentary: "hallucination" anthropomorphises. These are "fuzzy pattern matching" tools, and code happens to be where fuzzy pattern matching works well — which is also why the bugs they find are real even though no reasoning produced them.

> "These tools want to please you. They're very sycophantic. If you say, give me a bug, it'll work really, really hard to give you a bug. So hard it'll go out and read our mailing list and report bugs that other people have found and fixed."

Sycophancy as a *security-process* hazard, not just a UX annoyance: the tool re-serves already-fixed bugs as "findings," and the same bug then goes to multiple researchers, generating credit disputes (people "mad at us every single week because... they discovered a problem... that was reported a day earlier in public").

> "Turns out half the patches that these things generate that look correct, they feel correct and they want to be correct are just flat out wrong."

His empirical result from six graduate students reviewing batches of LLM patches. "They fooled me" — even a kernel security veteran misjudged one. The patches read like the work of a reasoning human; there is no human and no reasoning.

> "If you see people send you that stuff, say please start over."

On changelog walls of text: his interns found that deleting the changelog entirely and reviewing only the code "actually works." The persuasive scaffolding is not signal.

> "Anything that you upload to these bots, it will leak... Do not upload anything you don't want to see on the web."

The counter-intelligence framing of AI tooling: everything you upload is data to be trained on and re-served to someone else, so all security-relevant triage belongs on local models.

## Key Themes

#concept #security #pattern

**Fuzzy pattern matching, not intelligence.** The talk's core reframe. Bug finding works because it is the same pattern completion that Sasha Levin and Julia Lawall's Coccinelle work did a decade ago — "where has this type of fix not been applied elsewhere?" — now with more context. New interface, old idea, and the attribution to prior research is conspicuously missing.

**Noise as the attack surface.** The real threat is not bugs the LLMs find (the kernel grinds through those) but the industrialisation of low-quality submissions: −7 day exploit lead time from chaining minor unfixed issues, weekly "we found 100 bugs" emails that resolve to two real fixes, and maintainers who must treat every patch as unverified. The DoS is on human attention.

**Procedural defenses over technical ones.** Documented threat models per subsystem, requiring a patch before engaging, CCing maintainers, the "assisted by" tag, asking "did you forget the assisted-by tag?" as a bot-detector. The kernel fights a probabilistic adversary with deterministic process — the same move as [[Guardrails and Feedback Loops]] generally.

**The fuzzer precedent.** The strongest argument in the talk is historical: the syzkaller/satan flood felt identical, the work got done, and rsync now comes back clean from every scanner. Bugs get fixed and stop. "These things do not persist forever." A rough 12–18 months, then equilibrium.

## Analysis

This is the most valuable document yet on the *receiving* side of automated vulnerability discovery, and it is pointedly at odds with the vendor story. Where [[Project Glasswing — Mythos at Cloudflare]] celebrates closed-loop proof generation, Kroah-Hartman has the raw Mythos data and the decomposition is brutal: most of the headline number was crashes-without-reports, non-bugs, fabrications, and public re-derivations. Both can be true — Glasswing's harness is for one's own repos with triage built in; the kernel got raw output dumped on public lists — but the contrast is the lesson: the same tool is a godsend inside a pipeline and a menace as a press release.

It also concretely strengthens [[AI Cybersecurity After Mythos — The Jagged Frontier]]'s specificity argument. Fort showed frontier models flag patched code as vulnerable; Kroah-Hartman supplies the maintainer-side ledger of what that costs per week. And his 50%-wrong-patch finding is a stronger number than most enterprise surveys dare, with a mechanism (sycophancy + pattern matching, no test execution) rather than a vibe.

The least examined claim is the optimism. "Once you fix them all, it stops" worked for fuzzers because fuzzing converges on a fixed surface; but LLM finders improve with each model generation, so "the flood ends" may be wrong — the flood changes shape. His own data hints at this: patch volume rising, new contributors indistinguishable from bots, trust (the kernel's actual currency) eroding. His answer — "just act human," go to conferences, answer email — is touching but not a scaling solution. Still, as a practising prescription for surviving the transition, it is unmatched: push back, demand reproducers, never upload secrets, and fix the bugs.

---

*Sources: [[raw/kernel-recipes-2026-security-in-the-llm-age]], [[summary/kernel-recipes-2026-security-in-the-llm-age]]*
*Last updated: 2026-10-10*
