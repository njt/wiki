# Loop Engineering

Addy Osmani's 2026 piece naming and taxonomizing a practice already spreading through the agent-coding community: designing systems that prompt agents, rather than prompting agents yourself. The frame is a maturity leap — from "I tell the agent what to do" to "I design the harness that tells the agent what to do, on a schedule, with guardrails, across worktrees and sub-agents, and I review what emerges."

---

## Key Quotes

> "Loop engineering is replacing yourself as the person who prompts the agent."

The one-sentence thesis. It's catchy and correct, though "replacing" overstates it — you're still designing what gets prompted, just at one layer of abstraction up.

> "I don't prompt Claude anymore. I have loops running that prompt Claude" — Boris Cherny

The most provocative quote in the piece, and also the most misleading taken alone. Cherny *does* prompt Claude — he just does it through skills, hooks, and automations he designed. The loop didn't design itself. The labor shifted, it didn't vanish.

> "The model that wrote the code is way too nice grading its own homework."

On why sub-agent verification matters. This is a concise statement of [[Guardrails and Feedback Loops]]'s core principle: verification must be independent of generation. A second agent with different instructions (and sometimes a different model) is the minimum viable independence.

> "Build the loop. But build it like someone who intends to stay the engineer, not just the person who presses go."

The closer — and the most important sentence in the piece. This is where Osmani separates his position from the "dark factory" maximalists. The loop is a tool for engineers who want to scale their judgment, not a replacement for judgment itself. This aligns with his other writing ([[Addy Osmani's Workflow]]) where "the human engineer remains the director of the show."

---

## The Taxonomy

Osmani organizes loop infrastructure into five components plus state:

### Automations
Scheduled triggers — `/loop`, `/goal`, cron, hooks — that initiate agent work without human prompting. The `/goal` primitive (run until a verifiable condition is met) is the shared concept across Claude Code and Codex. This is the entry point: instead of "I'll run that agent now," the system decides when to run.

### Worktrees
Isolated git worktrees so parallel agents don't collide. Both platforms now support this natively. [[DeltaDB]] and [[ctx – Agentic Development Environment]] are pushing this further — but Osmani is describing the current state, not the horizon.

### Skills
The `SKILL.md` format as codified project memory. Without skills, "an agent starts every session cold" and fills gaps with confident guesses. This is [[Claude Code Mastery]]'s central claim as well: skills are compounding infrastructure. Osmani's contribution is positioning skills *within* the loop — they're not just for interactive sessions, they're what automations and sub-agents reference.

### Plugins and Connectors
MCP-based bridges to external systems. The shift from "agent says here's the fix" to "loop opens the PR, links the ticket, pings the channel." This is the [[Agent Orchestration]] dimension — the loop isn't just generating code, it's participating in the engineering workflow.

### Sub-agents
Writer/reviewer separation as the minimum viable independent verification. Different agent, different instructions, sometimes different model. This patterns shows up everywhere: [[OpenCodeReview]]'s per-file concurrent subagents, [[The Advisor Strategy]]'s advisor-executor pattern, [[MiMo Code]]'s independent memory writer subagent. Osmani is naming what practitioners keep rediscovering.

### State/Memory
"Memory has to be on disk" because the model forgets everything between runs. This is where [[Coding Agents Continuity Not Memory]] and [[napkin]] converge — the repo as the natural home for operational state. Osmani's treatment is the shallowest part of the piece (a markdown file or Linear board), but the placement is correct: state is the sixth component that makes the other five work across sessions.

---

## Key Themes

#concept #agentic-loop #automation #subagents #verification

- **#concept — Loop Engineering**: The meta-skill of designing systems that prompt agents. A layer above prompt engineering, a layer below "dark factory." The term is new; the practice is already widespread.
- **#pattern — The Five-Component Loop**: Automations (triggers) + worktrees (isolation) + skills (context) + connectors (integration) + sub-agents (verification). Plus state to persist across runs.
- **#pattern — Writer/Reviewer Separation**: The simplest and most reliable verification pattern. Different agent, different instructions. This is the anti-vibes guardrail.
- **#concept — Comprehension Debt**: The faster loops ship code, the wider the gap between what exists and what the engineer understands. Osmani names this as a real cost, not a theoretical concern.
- **#person — Peter Steinberger**: Creator of the "you shouldn't be prompting coding agents anymore" line. Credits Osmani for popularizing the loop engineering frame.
- **#person — Boris Cherny**: Creator of Claude Code. "I don't prompt Claude anymore." The practitioner whose workflow the piece is describing.

---

## Critical Analysis

**The term "loop engineering" is doing real work.** It names something that practitioners were already doing but didn't have language for. Before this piece, you'd describe it with a paragraph: "I set up a cron job that runs Claude Code on a worktree, reads the CI failures, and files PRs." Now: "loop engineering." The compression matters — it lets the community discuss the practice as a thing, not a workflow description.

**But the piece is more taxonomy than invention.** Every component Osmani describes (automations, worktrees, skills, sub-agents, connectors) already existed. His contribution is naming the integration as a discipline and organizing it into a teachable frame. This is valuable — naming is infrastructure too — but it's worth being clear about what's novel (the frame) and what's not (the components).

**The Boris Cherny quote is doing too much rhetorical work.** "I don't prompt Claude anymore" is the headline that'll get shared, but it's false in the way that matters. Someone designed the loops. Someone wrote the skills. Someone chose the verification criteria. The loop didn't emerge from the ether — it was engineered, and that engineering *is* prompting, just at a different level of abstraction. The risk is that "I don't prompt Claude anymore" becomes an aspiration divorced from the judgment that makes it work.

**The three "problems that sharpen" are the best part.** Verification debt, comprehension debt, and cognitive surrender. These are the actual costs of loop engineering, and Osmani doesn't soft-pedal them. "Designing the loop is the cure when you do it with judgement and the accelerant when you do it to avoid thinking" — this should be the pull quote, not the Boris Cherny line.

**Osmani is straddling two audiences.** The piece reads as both a practitioner's guide and a platform comparison (Claude Code vs. Codex). The platform comparison sections dilute the argument — the specific feature comparisons (Codex has an Automations tab, Claude Code has `/loop`) will date quickly, while the conceptual frame won't.

**The missing component: feedback from production.** The loop architecture describes generating and reviewing code, but says nothing about what happens after merge. Does the code work in production? Are users happy? Did the fix introduce regressions? This is where [[Harness Engineering]] and [[Guardrails and Feedback Loops]] go further — verification doesn't end at the PR. A loop that can't learn from production outcomes is a loop that can't improve. This connects to [[Compound Engineering]]'s thesis: the fourth step (Compound) is what separates productive teams from fast ones.

**The training-problem critique:** [[Harness Engineering is not Enough]] argues that even perfect loop engineering can't solve what is fundamentally a model training issue: RL rewards test-passing correctness, not maintainability, and the reward signal for good architecture arrives months too late. This doesn't invalidate loop engineering—Horthy's own proposed workflow uses AI-assisted loops for planning—but it bounds it: loops can raise the floor, not change the ceiling set by what was reinforced during training.

**Comparison with [[Don't Fear the Dark Factory]]:** Matt Wynne's piece describes the same thing — simple loop + validation harness — but from the perspective of personal conversion. Osmani is writing the architecture manual; Wynne wrote the testimony. Both are correct; they serve different readers. Wynne's version is more honest about the emotional experience; Osmani's is more useful as a reference.

**Comparison with [[From AI Studio to AI Forge]]:** McCormick's five-plane stack is the industrial-grade version of what Osmani describes. Where Osmani says "skills + automations + sub-agents," McCormick says "five planes with human changing altitude as the control mechanism." Osmani's version is the one you'd actually implement this week; McCormick's is the one you'd architect toward.

**The honest tension**: Osmani's previous piece ([[Addy Osmani's Workflow]]) was all about the human staying in the loop — review every line, treat the agent like a junior dev. This piece describes removing the human from the prompt loop while keeping them in the review loop. That's a step toward [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] Level 3-4, and it's in tension with his earlier advice. He doesn't acknowledge this tension directly, but the closer ("build it like someone who intends to stay the engineer") is trying to hold both positions.

**Production-scale validation: [[AI Code Migration with Claude Code]]** is Loop Engineering at 1M lines. Anthropic's Bun migration instantiates every component of Osmani's taxonomy: skills (the rulebook), sub-agents (implementation fan-out + adversarial review), automations (the mechanical work queue driven by compiler output), and state (the filesystem as the kanban board). The article's thesis — "fix the process that produced the code" — is Osmani's meta-skill stated as a migration principle rather than a development one.

---

*Sources: [[summary/loop-engineering]]*
*Last updated: 2026-07-25*
