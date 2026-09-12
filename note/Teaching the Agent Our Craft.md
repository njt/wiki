# Teaching the Agent Our Craft

A field report from 8th Light's Alex Haldeman on applying disciplined software engineering — TDD, clean architecture, explicit conventions — to Claude Code agentic development on a real healthcare platform. The article is the most concrete published example of the full [[Steering Claude Code]] taxonomy in production: CLAUDE.md as knowledge root, path-scoped rules for domain-specific conventions, skills for structured workflows (`/story-writer` and `/tdd-build`), subagents with `PreToolUse` hook enforcement, and MCP integration connecting Linear and Figma directly into the agent's context. Two developers carried what would normally require a larger team by encoding their craft into the harness rather than repeating it in code review.

---

## Key Quotes

> "Getting this root right matters more than any single rule beneath it."

On CLAUDE.md as the knowledge root. This is the same argument [[Writing a Good CLAUDE.md]] makes about the instruction budget — every line in CLAUDE.md cascades across every session — but Haldeman frames it as leverage rather than constraint. The four critical rules he lists (no speculative code, tests before implementation, reuse before new, comments explain WHY) are not novel; what's novel is encoding them once and having the agent follow them consistently, freeing code review for what actually requires human judgment.

> "A test written with the implementation already in view tends to describe that implementation, not the behavior we actually care about."

The justification for the isolated `test-writer` subagent. This is the sharpest TDD insight in the article and one that transfers immediately to any team. By dispatching a subagent with a fresh context window that has never seen the implementation, the test is forced to describe behavior from the spec alone. The RED gate — confirming the test fails for the right reason — becomes the contract between specification and implementation. This inverts the common failure mode where AI-generated tests are little more than implementation recapitulation.

> "The prompt states the intent, and the hook is what actually enforces it."

The operational thesis of the entire harness. Haldeman's `restrict_test_writer_paths.py` is the practical embodiment of what [[Guardrails and Feedback Loops]] calls deterministic enforcement: the prompt says "only write test files," but the `PreToolUse` hook *prevents* writing anywhere else. Instructions are probabilistic; hooks are not. This one sentence captures why the [[Steering Claude Code]] enforcement hierarchy (prompt < hook < managed settings) matters in production.

> "This approach was an accelerant to our team's abilities, not a replacement."

The article's most important hedge, and the one that separates it from dark-factory maximalism. Engineers still scaffolded when needed, then folded lessons back into the rules. The feedback loop — engineers improve the harness, harness lets them cover more ground — is the same flywheel [[I Stopped Coding and Started Architecting Agents (And Why You Should Too)]] describes. The agent didn't replace craft; it became the primary medium for expressing it.

> "That is usually what separates genuine progress from debt you only discover later."

On scope control. The structure that kept the agent honest — research before planning, failing test before implementation — also kept the codebase coherent under schedule pressure. This is the unglamorous payoff of methodology: not speed, but the absence of hidden regressions.

## Key Themes

#concept #pattern #tool #tdd #harness-engineering

- **Harness as encoded craft**: The team didn't tell the agent what to do in every prompt; they encoded their conventions into rules, skills, and hooks so the agent *defaulted* to the right behavior. This is [[Loop Engineering]] in practice — design the system that prompts the agent, not the prompts themselves.
- **Isolated test-writing subagent**: The `test-writer` subagent with a fresh context window is a genuinely novel pattern. It solves the implementation-contamination problem that makes most AI-generated tests vacuous. The `PreToolUse` hook restricting writes to test directories is the enforcement mechanism — prompt states intent, hook enforces it.
- **RPI pipelined through skills**: Tyler Burleigh's Research-Plan-Implement model structured into two Claude Code skills (`/story-writer` and `/tdd-build`) with human review gates between phases. The [[SDDW (Spec-Driven Development Workflow)]] uses a similar pipeline but with specs as the durable artifact; Haldeman's variant makes failing tests the contract.
- **MCP as collaboration infrastructure**: Linear for backlog, Figma for design — not copied into prompts but read live by the agent. This "collapsed the lag between a design decision, a product priority, and the code that implemented it." The [[Agentic Product Standard v2.0]] calls this the tool-design layer of the harness.
- **Small-team force multiplication**: Two developers carrying a multi-tenant platform, AI coaching companion, and secure data architecture. The story isn't that AI wrote more code — it's that the harness made a small team *coherent* at a scale that usually requires more hands.

## Critical Analysis

The article is **the best public field report on structured agentic development** as of mid-2026, but it has three significant gaps.

**First, it's a success story with no failure modes documented.** The `test-writer` subagent pattern is elegant, but what happens when the behavioral spec is underspecified and the test-writer produces a test that passes for the wrong reason? What happens when the RED gate passes but the test is testing something trivial? Haldeman doesn't say. Every pattern in the article has a failure mode, and the silence on them makes the approach look more foolproof than it is. Compare [[Swarm Skill]], which documents fourteen hard rules learned from concrete failures — that's the standard for operational honesty.

**Second, the MCP integration story is underspecified.** The `.mcp.json` shows Linear, Figma, EHR, and CMS endpoints, but the article never explains how the EHR and CMS MCP servers were built or what their tool surface looks like. That's where the real engineering challenge lives — building MCP servers that expose the right abstractions to agents — and it's the part the article skips. The Figma-to-design-tokens story is tantalizing but thin on implementation detail. Compare [[How AI Coding Agents Actually Use Your Technology]], which traces the entire discovery-to-invocation cascade for agent tools.

**Third, the article is a consulting case study that doubles as a sales pitch.** The closing CTA ("Have an idea, workflow, or challenge that has been a blocker for your business? Let's talk") is fine — 8th Light is a consultancy — but it means the article selects for what worked and omits what didn't. The real test of this approach isn't whether it worked once on a greenfield healthcare platform with a motivated team; it's whether it survives the second project, the third team, the inherited codebase. The framework's transferability is asserted, not demonstrated.

**What the article gets right, and why it matters**: The RPI → skills pipeline is the most concrete published workflow for keeping product and design involved in agentic development. Most agentic workflows optimize for developer throughput and treat product/design as inputs to be consumed. Haldeman's framework structures review gates so PMs and designers stay in the loop at phase transitions. That's the genuinely hard problem — not making agents faster, but keeping humans meaningfully involved — and the article is the best published answer to it so far.

---
*Sources: [[raw/teaching-the-agent-our-craft-structured-agentic-development-on-a-real-codebase]]*
*Last updated: 2026-07-18*
