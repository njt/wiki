# Running an AI-Native Engineering Org

Fiona Fung's field report from leading the Claude Code and Claude Cowork engineering teams. The core insight: when coding throughput stops being the bottleneck, the entire engineering process needs to be rethought — not optimized, but redesigned. This is the most concrete, battle-tested description of what actually changes when an engineering org goes AI-native, written by someone doing it.

## Key Quotes

> "Writing code, writing tests, and refactoring rarely slows us down anymore."

The bottleneck moved from *generation* to *verification*. This isn't just faster coding — it's a fundamental shift in what engineering work *is*. When typing stops being the constraint, code review, security, and domain judgment become the scarce resources. Every process built around "engineering bandwidth is expensive" becomes suspect.

> "Claude handles all the style and linting, PR feedback requests" — human review reserved for legal, trust boundaries, security-sensitive code, and product/design judgment.

This is the right split: AI for style/bugs/tests (patterns it can learn from code), humans for domain expertise (judgment that requires context outside the codebase). Fung is honest that this balance "will shift as models improve" — she's not claiming a permanent boundary, just describing today's.

> "I haven't seen a non-Claude-assisted commit in the last four months."

This is the real adoption metric. Not "users who tried it once," not "GitHub stars." Every commit, every engineer, for four months. When 100% of your output flows through the tool, your processes either adapt or break.

> "Pick your noisiest workflow" — the most expensive or dreaded one — and ask whether it still serves its purpose.

The most actionable advice in the piece. Fung describes canceling a weekly review where "everyone was on their laptops except during status reports." The meeting existed because the process existed — no one had asked "why are we having this meeting?" in years.

## Key Themes

- **#pattern JIT Planning**: Fung draws a direct analogy to JIT compiling. Six-month roadmaps made sense when engineering capacity was the constraint. When AI removes that constraint, planning needs to be as responsive as the build pipeline. The team now prototypes first, plans through PR discussions, and iterates on internal feedback.

- **#pattern Bottleneck Migration**: The core architectural insight — optimizing the old bottleneck (coding speed) only exposes the next one (verification, review, security). AI-native engineering isn't about going faster at the same things; it's about recognizing that the constraint moved and reorganizing around the new one.

- **#pattern Dogfooding as Culture**: "Relentlessly dogfood your product" is listed as non-negotiable #1. Not "use it sometimes" — every team member uses Claude Code and Cowork. This isn't just QA; it's how the team discovers what their own tool can and can't do, which directly shapes both product and process.

- **#pattern Flat Teams with Agency**: Managers must start as ICs first, pods stay small and agile, and team members have "explicit permission to question and remove obsolete processes." The org structure mirrors the technical architecture: distributed agency, minimal hierarchy, permission to kill dead code.

- **#concept Throughput vs. Success**: Fung's warning — "don't confuse throughput with success" — is the most important caveat in the piece. PR cycle time dropping is only meaningful if the PRs are worth writing. Speed without direction is just faster waste.

## Critical Analysis

**What's genuinely new here:** Fung describes *process ossification* as the hidden cost of pre-AI engineering. Processes (roadmaps, review rituals, status meetings) were designed to coordinate scarce human coding bandwidth. When that scarcity evaporates, the processes don't automatically disappear — they become zombie overhead. The JIT planning analogy is sharp: traditional planning is like AOT compilation, optimized for slow build cycles; AI-native planning is JIT, optimized for fast iteration. This reframes "agile" not as a methodology but as an architectural property of the org.

**What's undersold:** The role transition from "I write code" to "I direct Claude writing code" gets only a brief mention (PMs coding, blurred roles). But this is the psychological chasm every engineer faces. Fung describes the *organizational* answer (hire for product sense + systems expertise, deprioritize raw throughput) but not the *individual* one. How do you retain engineering identity when you're not the one typing? [[Claude Code Mastery]] covers this from the practitioner side; Fung covers it from the hiring side. The gap between them is where most teams will struggle.

**The hidden tension:** Fung's three non-negotiables (dogfood, flat teams, kill processes) only work because the team builds the tool they're dogfooding. A team using Claude Code but building something unrelated (most teams) can dogfood the tool but can't dogfood the *integration* of tool and product. The advice generalizes, but the intensity of the feedback loop doesn't.

**What to copy vs. adapt:** JIT planning and "ask Claude first" are universally applicable. The specific hiring profiles (creative builders + systems experts) are worth adopting. The flat-team structure is contingent on team size and mandate. The "kill processes" norm requires psychological safety that doesn't exist everywhere — Fung's explicit permission-giving is a deliberate culture intervention, not a default.

**Compare to:** [[Compound Engineering]] (building compound workflows atop AI), [[Agent Coding Workflow]] (the practitioner's loop), [[Writing Code vs. Shipping Code]] (the attenuation from commit to release), [[How Intercom Uses Claude Code]] (another team's Claude Code adoption pattern), [[10 Principles for Agent-Native CLIs]] (parallel thinking for tool design), [[Organizational Intelligence Systems]] (the same bottleneck-migration and MCP+skills architecture applied to organizational decision-making rather than code generation).

---

*Source: [Running an AI-native engineering org](https://claude.com/blog/running-an-ai-native-engineering-org), Fiona Fung, claude.com/blog, 2026-06-03. Fetched 2026-06-15.*
