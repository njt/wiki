# Code Field

A prompting technique from BLUECOW009 (NeoVertex1) that uses inhibition rather than instruction to improve LLM code generation. The prompt is all negations -- no instructions on what to do, only what not to do. Tested across 18 experiments: assumption-stating went from 0% to 100%, bug detection from 39% to 89%, and severity recognition from 0% to 100%.

---

## Key Quotes

> "Code is frozen thought. The bugs live where the thinking stopped too soon."

> "The finding: inhibition shapes LLM behavior more reliably than instruction."

> "The tests you didn't write are the bugs you'll ship. The assumptions you didn't state are the docs you'll need. The edge cases you didn't name are the incidents you'll debug."

> "The question is not 'Does this work?' but 'Under what conditions does this work, and what happens outside them?'"

> "Let edge cases surface before you handle them. Let the failure modes exist in your mind before you prevent them. Let the code be smaller than your first instinct."

## Key Themes

#agentic-coding #prompting #mindfulness #simplicity #inhibition

The key finding is the strongest claim: **inhibition shapes LLM behavior more reliably than instruction.** The 4-line atomic prompt -- nothing but negations and a question -- outperformed positive instructions in both code generation and code review tasks. This is counterintuitive in a field where the standard advice is "give more context, write longer specs, pack the prompt."

The full prompt works at two levels. The philosophical framing ("code is frozen thought," "notice the completion reflex") primes the LLM to slow down and reflect before generating. The operational rules ("do not write code before stating assumptions," "do not handle the happy path and gesture at the rest") constrain the output format. The combination forces the LLM out of its default pattern-matching mode and into something closer to deliberate reasoning.

The "completion reflex" list is particularly well-observed:
- The urge to produce something that runs
- The pattern-match to similar problems you've seen
- The assumption that compiling is correctness
- The satisfaction of "it works" before "it works in all cases"

These describe exactly how LLMs (and humans) take shortcuts. Naming the failure modes makes them easier to avoid.

This connects to [[AI Zealotry]]'s "take long walks" advice and to the planning emphasis in [[Addy Osmani's Workflow]]. All three are getting at the same thing: the bottleneck has shifted from typing to thinking, so invest in the thinking. The Code Field prompt is a concrete tool for forcing that investment at the prompt level.

The full research is published at https://github.com/NeoVertex1/context-field/blob/main/code_field.md.

## Critical Analysis

The test results are impressive but the sample size (18 tests) is small and the methodology isn't peer-reviewed. The baseline comparison (standard prompts vs. Code Field) may not account for prompt length effects -- the full prompt is substantially longer than a typical instruction, and length alone can improve output quality by giving the model more time to "think."

That said, the core insight is genuine and practically useful. The atomic prompt (4 lines, all negations) is the real test, and the fact that it moved assumption-stating from 0% to 100% suggests that LLMs have latent capability that negative constraints unlock more effectively than positive instructions. This matches the general pattern that LLMs are better at following prohibitions than prescriptions -- telling a model what not to do narrows the action space more precisely than telling it what to do.

The "write what you can defend" framing is the actionable takeaway. It shifts the success criterion from "does this work?" to "can I justify every choice?" -- which is a higher bar that naturally produces more robust code.

The poetic register ("code is frozen thought," "let edge cases surface") is unusual for a prompting technique but may be functional: it engages the model's attention in a way that dry instruction lists don't. Whether that's replicable or specific to this prompt's particular phrasing is an open question.

---
*Sources: [[summary/code-field]]*
*Last updated: 2026-05-14*
