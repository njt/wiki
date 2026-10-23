# Who Maintains the Code Agents Write (arXiv Empirical Study)

An empirical study of what happens to AI-generated code *after* it merges: across 508 AI-generated files and 508 matched human files from 100 popular repos (AIDev dataset), agent-created files receive about half the maintenance commits of human files, those commits are dominated by feature extensions rather than bug fixes, and humans perform 83% of the work. The study's real contribution is refusing the obvious reading — it explicitly entertains the possibility that "less maintenance" means "nobody wants to touch it" rather than "it doesn't need touching."

---

## Key quotes

> "the most frequent modifications to AI code are feature extensions, whereas human updates focus on bug fixes"

The most interesting asymmetry in the paper. AI code isn't accumulating fix commits — it's accumulating *feat* commits. That reads as requirements under-coverage at generation time: the agent produced something plausible but incomplete, and humans build on it rather than repair it. Defects may be lurking unfixed precisely because nobody is patching.

> "human developers perform approximately 83% of maintenance on AI-generated files, with AI agents contributing only 17%"

The headline number, and the one that should kill any fantasy of fire-and-forget agents. Note the secondary figure: agents do maintain their *own* output (17%) far more than human code (7%) — but "more often" here means "still almost never."

> "Practitioners should therefore not assume that low maintenance frequency alone indicates high code quality."

The paper's most honest move. The authors list the confounds themselves: developers may avoid modifying AI code due to comprehension difficulty, or the files may be dead code that never runs in production. Both explanations would make low churn a *bad* sign.

> "establish review processes that track how generated files evolve after merging, rather than treating the initial generation as a finished product"

The practical takeaway. The unit of accountability doesn't end at merge.

## Themes

#concept #pattern #research

## Analysis

This study slots into the growing empirical literature that says: generation-time quality metrics are the wrong lens for agentic coding. The costs show up on a different timescale — a theme echoed by the studies the paper itself cites (He et al.'s 30% rise in static-analysis warnings, 41% rise in complexity; 63% more code smells). What's new here is the *who*: maintenance remains a human activity even on files the agent authored, which means the comprehension burden lands on developers modifying code they didn't write and may not understand.

Two caveats worth pressing. First, six months is short, and the authors admit early-adopter populations over-index on feature velocity; the converging gap from month four could go either way. Second, the "less maintenance = fine" reading is genuinely under-determined — this paper can't distinguish "stable" from "unowned" or "unused." That's exactly the comprehension-debt problem other pages in this wiki describe: the code is average, and nobody knows anything anymore.

The most forward-looking finding is the framing of a missing product category: maintenance-oriented agents. Current agents are generation machines; the 17% number is the gap between what agents do and what agents could own. A pipeline that predicts which AI-authored files will need sustained attention, then delegates the upkeep, is a plausible roadmap — and it aligns with the "own the outer loop" school: humans currently *are* the outer loop for agent-written code.

## Related

- Strengthens [[Agents and Acquiring Debt]] with hard numbers: 83% human maintenance of agent-authored files is the quantitative face of the comprehension debt that essay theorises.
- Complicates [[The Problem is not the AI Code, but Nobody Knows Anything Anymore]] — the study confirms the maintenance burden falls on humans, but its feature-extension finding suggests the mechanism is under-specification of requirements, not just lost architectural knowledge.
- Nuances [[Architectural Guardrails for AI-Generated Code]]: guardrails at generation time don't address the post-merge lifecycle; this paper argues the review process must track evolution after merging.
- Adds empirical weight to [[Five Studies That Are Changing How I Think About AI in Software Engineering]] — this is exactly the genre of long-horizon empirical work that essay argues for.

---
*Sources: [[raw/2605-06464v2]], [[summary/2605-06464v2]]*
*Last updated: 2026-10-24*
