# Inside the AI Workflows of Every's Six Engineers

Rhea Purohit profiles six engineers at Every — all building the same suite of AI products — and finds six radically different toolchains, converging on a few non-obvious truths: plan before you prompt, run multiple models side by side, and build guardrails against AI's tendency to derail you.

---

## Key Quotes

> "The problem with CLIs is it's easy to get derailed and lose focus."

Yash Poojary on why he structures his day around one big task. He's right in a way that goes deeper than he says: CLIs are seductive because every new prompt feels like progress, even when it's thrashing. The [[acceleration-flow]] dopamine loop.

> "If it's not in Linear, it doesn't exist."

Naveen Naidu's project management absolutism. Every ticket links back to its original source. This is the human-side complement to [[Specifications as the Product]] — durable intent tracking, not just durable specs. Linear is his externalized [[Agent Memory and Context|memory]].

> "I realize that I've placed too much of my trust in Anthropic, which leaves me vulnerable."

Nityesh Agarwal, after Claude Code glitched for two days and no alternative matched. The most honest line in the piece. Single-model dependency is a business continuity risk. [[On a Year of Multi-Model Development|Hoffman's multi-model setup]] isn't just about task fit — it's insurance.

> "I applaud the people at OpenAI for becoming a real menace to Anthropic's code generation reign."

Andrey Galko, who switched from Cursor to Codex and isn't looking back. The competition dynamic matters: the "reign" he references was real, and its erosion changes tool recommendations monthly.

> "I don't use Cursor anymore. I haven't opened it in months."

Danny Aziz, who does ~70% of his work in Droid. Tool churn among power users is accelerating. Today's essential tool is tomorrow's "haven't opened it in months."

---

## Key Themes

#agentic-coding #multi-model #workflow #planning #tool-comparison #practitioner-profile

**Planning-first is universal.** Every single engineer emphasizes planning before letting AI write code. Kieran generates plans in Claude Code with custom agents. Danny talks through second- and third-order consequences with GPT-5 Codex before coding. Nityesh spends hours researching and sketching plans with Claude. Naveen generates draft PRs purely for exploration. This isn't a preference — it's a pattern that maps directly onto [[The Plan Is the Program]].

**Multi-model is the norm, not the exception.** Five of six engineers run both Claude and OpenAI models. Yash runs identical prompts through Claude Code and Codex simultaneously. Danny uses GPT-5 Codex for big builds, Anthropic for refinement. Kieran is Claude Code-primary but reaches for Codex or Amp for "nerdier" features. This is [[On a Year of Multi-Model Development|Hoffman's construction-trade taxonomy]] in the wild: different models for different task shapes, routed by human judgment.

**The setups are wildly divergent despite identical context.** Same company, same tech stack, same product suite — and the engineers use different terminals (Warp, Ghostty), different IDEs (Zed, Xcode, Cursor, none), different primary agents (Claude Code, Codex, Droid), and different physical setups (single laptop to dual-machine desktop). There is no "right" stack.

**Guardrails are personal and structural, not technical.** Yash splits his day into build vs. explore. Nityesh watches Claude "like a hawk" with his finger on Escape. Naveen uses Codex Cloud for exploration and Codex CLI for real builds — two-track execution as a guardrail against confusing brainstorming with building. These are human process guardrails, not [[Guardrails and Feedback Loops|deterministic enforcement]]. They're fragile but they work for solo practitioners.

**Personal tool-building is a meta-layer.** Yash built AgentWatch to ping him when sessions finish. Naveen built Monologue for speech-to-text prompting. These aren't the products they're paid to build — they're infrastructure for using AI tools, built with AI tools. The self-devouring loop.

**Code review patterns are converging.** Kieran has a review command that loops Claude + Cursor + Charlie until ship-ready. Naveen uses Codex's `/review` then manual side-by-side comparison then Sentry error log verification. The Cora team (Kieran and Nityesh) uses GitHub as the human-agent interface: humans leave PR comments, Claude Code fetches and fixes. This is [[Compound Engineering]] in practice — every review cycle tightens the loop.

**GPT-5 shifted competitive dynamics.** Andrey's claim that GPT-5-Codex "finally got good at the user interface, too" is the most specific model-capability delta in the article. Naveen migrated from Claude Code to Codex entirely. Danny uses GPT-5 Codex for big features. The "Claude rules code" consensus is fragmenting.

---

## Critical Analysis

The article's value is as a dataset, not as journalism. Purohit reports faithfully but doesn't synthesize. The real insights emerge from reading all six profiles side by side and noticing what repeats. The fact that Every's engineers converged on planning-first, multi-model, guardrail-heavy workflows *independently* is a stronger signal than any single engineer's preferences.

The most important finding — planning-first is universal — is buried. Purohit doesn't call it out. Every engineer describes a planning step, but the article presents it as background rather than the headline. The headline should be: "Six engineers who ship AI products daily all agree: plan before you prompt." That's a finding with prescriptive force.

Nityesh's profile is the most interesting because it's the most constrained. Single terminal, single model, "100 percent attention," finger on Escape. He's getting value through depth and discipline rather than breadth and parallelism. This is the opposite of Yash's multi-machine, multi-model, parallel-session approach. Both engineers are productive. The article doesn't explore why — what task shapes or personality traits map to which style — and that's the missing analysis.

The tool churn should make infrastructure investors nervous. Danny abandoned Cursor. Andrey abandoned Cursor. Naveen abandoned Claude Code for Codex. These migrations happened within months. Any tool that's essential today could be "haven't opened it in months" next quarter. The durable investment isn't in any specific agent — it's in [[Specifications as the Product|the specs]], the [[Guardrails and Feedback Loops|deterministic enforcement]], and the human judgment about what to build.

Andrey's pricing complaint about Cursor (hitting monthly limits in a week) is a reminder that power users hit ceilings that casual users never see. The tools that serve the top 1% of users are different from the tools that serve the mass market. This echoes [[Two Kinds of User Are Emerging]] — these six engineers are all power users, and their tool requirements are unrecognizable to someone using Copilot in VS Code.

What's conspicuously absent: failure stories. Every engineer describes what works. Nobody describes a project where their AI workflow produced worse results than manual coding. Nobody quantifies the rework rate. The article would be stronger with one honest post-mortem. The wiki's [[Agent Coding Workflow]] synthesis page notes the same gap: "Failure case studies are almost entirely absent."

The physical setup details are quietly revealing. Nityesh on a MacBook Air M1. Danny on a single monitor or just a laptop. Yash on a Mac Studio + laptop. The hardware spread correlates loosely with parallelism (Yash runs concurrent sessions across machines) but not obviously with output quality. Compute is not the bottleneck.

---

## Cross-Links

- [[On a Year of Multi-Model Development]] — Hoffman's field report on running Claude+Codex+Gemini side by side; the construction-trade taxonomy maps directly onto how Every's engineers route tasks
- [[Agent Coding Workflow]] — this article is six case studies for the maturity spectrum that synthesis describes
- [[The Plan Is the Program]] — planning-first is the article's buried headline; every engineer practices what Angert named
- [[How Boris Uses Claude Code]] — the creator's workflow for comparison: parallel sessions, Plan mode, verification as force multiplier
- [[Addy Osmani's Workflow]] — the responsible professional's approach; closest analogue to Nityesh's disciplined single-model style
- [[Slowing the Fuck Down]] — Nityesh's "finger on Escape" is Zechner's deliberate friction made physical
- [[acceleration-flow]] — Yash's warning about CLI derailment is the practitioner experiencing what Marc theorized
- [[Guardrails and Feedback Loops]] — the engineers' guardrails are human process, not deterministic enforcement; the gap between the two is instructive
- [[Compound Engineering]] — Kieran's review loop and Naveen's three-stage review process are compound engineering in practice
- [[Cognitive Debt]] — the guardrails against drift are implicitly guards against cognitive debt
- [[How Intercom Uses Claude Code]] — the enterprise-scale contrast: 13 plugins, 100+ skills, vs. six individual stacks
- [[Specifications as the Product]] — the planning-first consensus is this thesis confirmed by practitioners
- [[Two Kinds of User Are Emerging]] — all six engineers are power users; their tool requirements are unrecognizable to casual users
- [[AI-Driven Development Life Cycle]] — AWS's methodology mirrors the plan→execute→review loops described
- [[Radical Accountability]] — taste and judgment are what remain; every engineer here is exercising taste, not typing
- [[Managing Agents via Kanban Boards]] — Kieran's GitHub work-command pattern is a kanban board by another name
- [[Scaling Long-Running Agents]] — Yash's parallel sessions + AgentWatch is a one-person implementation of multi-agent coordination
- [[Claude Code on the Go]] — Naveen's Codex Cloud from iOS is the same mobile-supervision pattern
- [[Code Field]] — Nityesh's shortened leash (interrupting, asking for explanations) is inhibition over instruction
- [[Pencil]] — MCP-native design tool; connects to the Figma MCP usage by Yash
- [[Figma MCP]], [[Context 7]], [[Sentry]] — MCP integrations used in production

---

*Sources: [[raw/inside-the-ai-workflows-of-every-s-six-engineers]]*
*Last updated: 2026-05-14*
