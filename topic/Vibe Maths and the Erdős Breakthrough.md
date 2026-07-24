# Vibe Maths and the Erdős Breakthrough

A 23-year-old amateur armed with ChatGPT solved a 60-year-old Erdős conjecture that had stumped professional mathematicians. The AI found a formula from a related domain that nobody had thought to apply — not because it was obscure, but because the entire field had "collectively made a slight wrong turn at move one" (Tao). The raw AI output was poor; it took a Fields Medalist and a Stanford mathematician to extract and validate the insight. This is the most interesting AI-assisted discovery story yet: not because AI was smarter than humans, but because it wasn't constrained by human disciplinary habits.

---

## Key Quotes

> "The humans that looked at it just collectively made a slight wrong turn at move one."

Terence Tao's diagnosis of why a 60-year-old problem fell to an amateur with ChatGPT. The AI wasn't smarter — it just didn't inherit the field's shared assumption about which approaches were worth trying. This is the anti-expertise argument made concrete: sometimes the expert blindspot IS the barrier.

> "What's beginning to emerge is that the problem was maybe easier than expected, and it was like there was some kind of mental block."

Tao again, reframing the discovery. The problem wasn't intrinsically hard; the field had developed a collective blind spot. The AI's value wasn't raw intelligence but freedom from disciplinary groupthink — it tried something obvious to an outsider that insiders had tacitly ruled out.

> "The raw output of ChatGPT's proof was actually quite poor. So it required an expert to kind of sift through."

The anti-"AI replacing experts" quote. The AI produced a diamond in the rough — actually, mostly rough with a diamond buried inside. Human expertise didn't become obsolete; it pivoted from doing the work to recognizing when the work was onto something. Verification eats generation.

> "We have discovered a new way to think about large numbers and their anatomy."

Tao on the broader significance. The AI didn't just solve one problem — it opened a new approach that may apply to other open questions. This is the difference between a one-off lucky guess and a genuine methodological contribution.

## Key Themes

#vibe-maths #amateur #LLM #mathematics #discovery #mental-block #verification

**Vibe maths as the mathematical analogue of vibe coding.** Price had no formal training. He was "just doing Erdős problems as I do sometimes, giving them to the AI." The methodology was identical to vibe coding: throw the problem at the AI, see what comes back, iterate. The difference? His output got reviewed by Terence Tao before anyone deployed it to production.

**The mental block as institutionalized cognition.** Every discipline develops shared assumptions about what will and won't work. These aren't wrong — they're usually right, which is why they persist. But when an entire field converges on the same wrong turn, the barrier isn't intelligence; it's that nobody is willing to try the "obviously wrong" approach. AI has no such compunction. It will try the stupid thing, and occasionally the stupid thing works.

**Verification as the enduring human role.** Price prompted. ChatGPT generated. Barreto recognized. Tao verified. Lichtman refined. Five humans in the loop, each performing a different cognitive function that the AI couldn't: taste, judgment, validation, curation. The human role didn't shrink; it migrated up the stack.

**The economics are inverted.** $20/month subscription + an idle Monday = a 60-year-old conjecture solved. The cost structure of mathematical research just got renegotiated. This doesn't mean professional mathematicians are obsolete — the verification step still requires deep expertise — but it does mean the bottleneck has shifted from "can we generate candidate solutions" to "can we tell which candidates are real."

## Critical Analysis

This story is being framed as "AI solves math" but that's exactly wrong. The AI proposed. Humans verified, interpreted, and refined. The breakthrough wasn't the AI's raw output — it was the socio-technical pipeline: amateur + AI + collaborator + elite mathematicians. Take out any link and it fails. Price without Barreto doesn't know he's found something. The AI without Tao produces an unpublishable mess. Tao without Price never thinks to try the "wrong" approach.

The "vibe maths" label is clever but dangerous. It invites the same [[Vibe Coding and the Maker Movement]] critique: democratized production without quality control produces slop. But here the quality control was built in — the Erdős Problems website, the collaborator network, the expert verification. The lesson isn't "vibe maths works." It's "vibe maths works when embedded in a verification ecosystem." Most vibe coding has no such ecosystem.

The "mental block" framing is the most transferable insight. Every field has its version of move-one-wrong-turn — the shared assumption so deep nobody questions it. AI's real superpower may not be reasoning but innocence: it doesn't know what it's not supposed to try. This connects to [[A Non-Anthropomorphized View of LLMs]] — the AI as strange attractor exploring paths through solution space that human cognition has walled off.

Price's amateur status mattered more than his prompting skill. He had enough math literacy to recognize an Erdős problem and enough AI literacy to frame it for ChatGPT. But he didn't have enough training to inherit the field's blind spots. This is the [[Things You're Allowed to Do]] dynamic: sometimes the expert's greatest liability is knowing what's impossible.

The uncomfortable question this raises: how many other 60-year-old problems are vulnerable to "what if we just tried the obvious thing the field decided was wrong 40 years ago?" And what happens when someone automates the Price-Barreto-Tao pipeline — when an AI can both generate the candidate solution AND simulate expert verification? That's not here yet, but this story is a proof of concept for a much more disruptive future.

A parallel case worth comparing: [[Poolside Laguna S 2.1]] independently rediscovered a proof to Erdős problem #397 — a different problem, also open for 50+ years — working fully autonomously for 68 minutes with no human in the loop. It found a structurally different solution from the known proof using Perl for brute-force prime factorizations (Python wasn't available), and its November 2025 knowledge cutoff predates the January 2026 published solution, ruling out memorization. Where the vibe maths story required five humans in a verification pipeline, Laguna's case study suggests a model with behavioral RL training (persistence, verification, not declaring victory early) can sometimes run the full loop autonomously. The difference between "AI proposes, humans verify" and "AI verifies its own work" is the line these Erdős stories are drawing in real time.

---

*Sources: [[summary/amateur-chatgpt-vibe-maths-erdos]]*
*Last updated: 2026-05-15*
