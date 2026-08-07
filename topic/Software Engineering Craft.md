# Software Engineering Craft

The fundamentals don't change even as agents rewrite the tooling layer. Error handling still matters. API design still matters. Operations still matters. Project management still matters. If anything, these fundamentals matter more when code generation is cheap -- because the cost of bad architecture gets amplified rather than amortized. When an LLM can generate code 10x faster, it generates bad code 10x faster too. The craft of software engineering is now less about writing code and more about the judgment calls that shape what gets written: what to build, how to handle failure, when to simplify, and how to keep systems running once they're deployed.

---

## The Landscape

### Error Handling and API Design

[[Better Error Messages]] documents Wix's "Errorgate 2021": an audit of 7,643 error messages that revealed the company was "more like that friend who loves to gossip but doesn't pick up the phone when life gets hard." The five rules (say what happened, say why, reassure, give a way out, help fix it) are timeless and apply to CLI error messages, API responses, and agent failure states equally.

[[Designing a Passively Safe API]] goes deeper: after any failure, the system either completes exactly once or lands in a visible terminal state. No duplicate charges, no orphaned side effects. The engineering is concrete: idempotency keys, transactional outbox/inbox, recovery-point checkpointing, explicit `is_transient` booleans in error responses. This is the best single-article treatment of API idempotency available.

[[Good API Design]] complements both from the strategic level: Goedecke argues that boring, immutable APIs win -- product value matters more than interface elegance, cursor pagination beats offsets, and GraphQL is a complexity tax. Where Albaugh gives the mechanical engineering, Goedecke gives the design philosophy.

Both become more important in an agent world. Agents generate API calls at scale, retry without understanding failure semantics, and can't tell the difference between transient and permanent errors unless the API explicitly communicates it.

### Simplicity and Comprehensibility

[[Elements of Code]] stakes the position: "Writing comprehensible code is what allows us to be wrong in correctable ways." The goal isn't perfection; it's correctability. This is the antidote to [[Cognitive Debt]]'s diagnosis: if velocity exceeds comprehension, invest in comprehensibility.

[[Simplicity in the Age of AI-Assisted]] argues that LLMs are more valuable as demolition tools than construction tools -- the real unlock is cheap rebuilds without inherited complexity. But [[Systems Ideas That Sound Good]] warns about eight engineering patterns that sound like simplification and actually shift complexity to where it's less visible: pluggability, over-abstracting, DIY async, deferred security, data sync, cross-platform, escape to native. Sinofsky's catalogue is essential reading for anyone tempted to "simplify" with a new abstraction layer.

[[Nobody Knows How Large Software Projects Work]] names the uncomfortable truth: complexity at scale is inherent, not a staffing or process failure. The system is unknowable not because of poor engineering but because of successful product development. Documentation fails because the system changes faster than anyone can write it down. The answer from other pages: observability ([[The Future of Software Engineering is SRE]]), spec-first development ([[Spec-Driven Development]]), and radical simplification.

### Operations as the Differentiator

[[The Future of Software Engineering is SRE]] makes the sharpest argument: when code generation is trivially easy, keeping software running reliably is the scarce skill. "The first 90% to get a working demo is easy. It's the other 190% that matters." Uptime guarantees, defect identification, proactive issue detection, secure data handling, failure recovery -- these are what separate toys from services.

[[Anomaly Detection]] provides a beautiful example of operational tooling done right: Welford's algorithm for running mean and variance in constant memory, hourly bucketing in a KV store, 2-sigma threshold. No ML, no config, no training phase. Just math. This is the right level of sophistication for most monitoring needs.

[[Before Reading Code]] shows git archaeology as a substitute for institutional knowledge: churn hotspots predict defects better than complexity metrics, contributor ranking reveals bus factor, velocity trending shows organizational health. Minutes of work that save hours of misdirected reading.

### Project Management in the Agent Era

[[How I've Run Major Projects]] (Ben Kuhn) offers six principles: focus (6+ hours/day), detailed planning for victory, fast OODA loops, overcommunication, break off subprojects, and have fun. The key insight: "projects are information-constrained, not execution-constrained." Information-processing is the work, not a tax on it.

[[When the Target Keeps Moving]] extends this to AI-accelerated projects: LLMs get you to the hard part sooner, and the hard part is learning what you don't know yet. The discovery/delivery ratio is the metric that matters. In a 32-day sprint, the project grew from 254 tasks to ~1,400 -- for every 100 completed, ~109 new ones appeared. Track whether you're converging or diverging.

[[14 More lessons from 14 years at Google]] adds the organizational layer: "approve, choose, unblock, or inform" for meetings; reliability as a product feature; recurring heroism as a failure mode. The anti-hero-culture argument: sustainable teams design normal operations that don't require exceptional effort.

[[Lovelace]] takes a different angle on the same problem: instead of managing projects in a separate SaaS tool, it puts tickets, docs, ADRs, and agent session records directly in the repo as Markdown+YAML files, where coding agents already live. The tooling (Tauri app + Claude Code MCP server) is secondary to the files themselves.

### The C#/.NET Corner

Four pages form a thin but notable cluster. [[Dapper Performance Trap]] documents the NVARCHAR vs VARCHAR implicit conversion that defeats indexes -- 176x slower on a million-row table, completely invisible in the C# code. [[Installing VS Compilers From Commandline]] solves the "skip Visual Studio, install just the compiler" problem. [[dotnet Slopwatch]] catches agent-specific shortcuts in .NET code: disabled tests, suppressed warnings, empty catches. [[C# DateTimeOffset Format Selection]] is a field guide to picking timestamp formats at API boundaries, organized around the only criterion that matters: what survives the round trip. Like [[Span-First CSharp — Designing Around SpanT]], it's a single-concept design guide that earns its keep by telling you not just how but when — and when not to. Its core insight (RFC 3339 is the narrow, interoperable profile of ISO 8601 that most APIs actually mean) applies far beyond C#.

This is an area to grow. The wiki has 197 pages and only 4 cover C#/.NET specifically. Given that .NET is a major production ecosystem and the agent tooling for it is thinner than Python or TypeScript, there's a gap here worth filling.

## Key Tensions

**Simplicity vs. useful complexity.** [[Simplicity in the Age of AI-Assisted]] says rebuild simpler. [[Systems Ideas That Sound Good]] says most "simplification" is complexity displacement. [[Nobody Knows How Large Software Projects Work]] says complexity at scale is inherent. The resolution: simplify where you can, accept what you must, and invest in observability for everything else.

**Code ownership vs. disposability.** [[Write Only Code]] says nobody reads agent-generated code. [[Elements of Code]] says comprehensibility is the primary virtue. If code is write-only, who cares about comprehensibility? The answer: someone has to debug it when it breaks. The code may be write-only during generation, but it's read during incidents. That's when comprehensibility pays off.

**Operational investment vs. feature velocity.** [[The Future of Software Engineering is SRE]] says operations is the differentiator. But organizations reward feature delivery, not operational reliability. The accountant who builds spreadsheet automation saves hours weekly, then spends every vacation debugging it. The incentive structure punishes operational investment until something breaks.

**Discovery vs. delivery.** [[When the Target Keeps Moving]] says cheap delivery means you should invest in discovery. [[How I've Run Major Projects]] says focus on execution. Both are right for different project phases: deliberate discovery early, focused delivery late. The three-phase model (first useful thing, deliberate discovery, denominator stabilization) resolves this.

## What's Missing

**Agent-era incident response.** How do you do incident response when nobody wrote the code? How do you debug systems where the implementation was generated by three different models across six months? [[The Future of Software Engineering is SRE]] raises the question; nobody answers it.

**C#/.NET depth.** Three pages is thin for a major ecosystem. Agent-specific tooling for .NET (beyond [[dotnet Slopwatch]]) is particularly sparse.

**Operational patterns for agent-generated systems.** We have operational patterns for human-written systems (SRE, chaos engineering, observability). We don't have patterns for systems where the code changes faster than anyone can understand it.

**Quantitative craft metrics.** [[Before Reading Code]] uses git data for code health. But we don't have metrics for "is this codebase getting more or less comprehensible over time?" [[Cognitive Debt]] names the problem; nobody measures it.

## Key Themes

#software-craft #error-handling #api-design #sre #simplicity #project-management

## Pages

- [[14 More lessons from 14 years at Google]] — Osmani on organizational dynamics: meetings, reliability as product, team interfaces
- [[Assorted less(1) Tips]] — Tim Chase's 17 less tricks plus HN's crowd-sourced addendum: a masterclass in deep tool knowledge, security footguns, and the pager as interactive programming environment
- [[Better Error Messages]] — Say what happened, say why, reassure, give a way out, help fix it
- [[Git Rebase for the Terrified]] — The rebase fear is irrational: your local clone is disposable, and the nuclear option costs nothing
- [[Designing a Passively Safe API]] — After any failure: complete exactly once, or land in a visible terminal state
- [[Idempotency Is Easy Until the Second Request Is Different]] — The hard cases: concurrent retries, partial failures, key reuse, recovery. 409 Conflict on same-key-different-command
- [[Cashpoints Partners API]] — NZ loyalty-points POS API: two-phase commit via lock-then-commit, field warnings as scar tissue, and the refund clawback problem
- [[Good API Design]] — Goedecke's practitioner's guide: boring over clever, immutability over versioning, product over interface
- [[Having a Creative Practice as a Programmer]] — Programming as artistic practice: the parallel track of daily creative work that sustains craft over a career, separate from productive output
- [[What You NEED to Know Before Touching a Video File]] — Video encoding craft guide: quality as fidelity-to-source, remuxing vs. reencoding, sharp opinions earned through mechanism understanding
- [[Elements of Code]] — Rules for comprehensible software. "Wrong in correctable ways"
- [[Patterns.dev]] — The definitive modern reference for web design, rendering, and performance patterns across vanilla JS, React, and Vue
- [[Igor Schwarzmann Design Systems]] — A dead reference site, worth remembering for who built it: design systems as organizational strategy, not component libraries
- [[The Future of Software Engineering is SRE]] — AI makes code trivial; operations becomes the differentiator
- [[Platform Engineering End-to-End]] — Cavallin's full-lifecycle field guide: team composition, product management, operations, migrations, stakeholder politics, and the priority order for starting from zero
- [[Spec-First Development at Benchling]] — Define each object once; let platform capabilities consume the schema
- [[The Coming Need for Formal Specification]] — AI makes code cheap, review lags, and formal methods become the systematic answer to the mismatch
- [[Systems Ideas That Sound Good]] — Sinofsky's eight engineering patterns that fail 9 out of 10 times
- [[Microservices for the Benefits, Not the Hustle]] — Microservices are about changeability, not scalability; hard repo boundaries enforce cohesion that monoliths allow to erode
- [[Nobody Knows How Large Software Projects Work]] — Complexity is inherent at scale; the team's value is answering questions
- [[Capturing Why Engineering Decisions]] — HN thread on documenting decision rationale: docs survive next to code, ADRs as point-in-time RFCs, LLMs invert the economics of documentation
- [[How I've Run Major Projects]] — Ben Kuhn: focus, detailed planning, fast OODA loops, overcommunication
- [[When the Target Keeps Moving]] — Track discovery-to-delivery ratio to know if you're converging or diverging
- [[Before Reading Code]] — Five git commands to diagnose codebase health before reading a single line
- [[Common Diagram Mistakes]] — Seven anti-patterns in architecture diagrams. Most diagram failures are communication failures
- [[Make the Easy Change Hard]] — Invert Beck's maxim: refactor the architecture first, then the easy feature writes itself. Async Rust war story
- [[WebRTC Is the Problem]] — Ex-Twitch/Discord WebRTC engineer: the protocol is wrong for voice AI. QUIC fixes this. Eight RTTs to one, port-binding to connection migration, Redis-backed load balancers to stateless routing
- [[Frozen Test Fixtures]] — Test the property, not the data: assertion patterns that survive fixture evolution
- [[How HTML Changes in ePub]] — ePub is XHTML, not HTML5. Unlearn your web habits
- [[Correct by Construction]] — Data quality as a whitelist: anchors, attributes, links, no NULLs
- [[Anomaly Detection]] — Welford's algorithm + KV store. No ML, no config, just math
- [[The Pragmatic Summit]] — Gergely Orosz's inaugural curated conference: 28 practitioner speakers, 3 tracks (Build/Frame/Lead), Thomas Dohmke's Entire.io reveal
- [[A Field Guide to Bugs]] — Stephen Diehl's poetic taxonomy of 30+ bug species from Bohrbug to Omega Bug: half CS folklore, half literary performance, and the sharpest diagnosis of LLM-era failure modes in print
- [[Book OCR Project Report — Structured Workflow Runtime and Manual PDF Repair]] — Manuel's full-arc project report: 202-page scanned book OCR, custom Go workflow runtime, structured JSON boundaries, and a day-long manual PDF repair loop that found five distinct failure classes. A masterclass in model-output engineering
- [[The Lindy Effect]] — Technology-choice heuristic: bet on things that have already survived. The probabilistic argument for craft over novelty.
- [[Software Engineering Practice Atlas]] — 4,654-entry AI-generated reference map across five practice areas and 25 domain guides; the most ambitious attempt to catalog software engineering craft as a navigable map rather than a linear text
