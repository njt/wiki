# Uno Platform's Two MCP Servers

Uno Platform's engineering team describes how they gave AI agents both grounding and verification for cross-platform .NET development: two MCP servers split by the lifetime of what they know, a Skills library for procedure, and a browser-hosted Roslyn pipeline that compiles and hot-reloads agent-written code live. It is one of the most concrete public write-ups of MCP server design decisions in production.

---

## The argument in one paragraph

Uno Platform claims that the bottleneck in agentic UI development is no longer generation but verification — an agent can write cross-platform .NET UI code faster than any human team yet cannot tell whether what it wrote renders and behaves correctly — and that this is fixable not with better prompts or more context, but by giving the agent live eyes and hands on the running app through a purpose-built MCP server, paired with a separately hosted docs server for grounding. The claim is falsifiable: if agents with screenshot, visual-tree, and automation-peer access to a running app still ship broken UI at roughly the same rate as agents with docs alone, the verification bottleneck thesis is wrong, and if the two-server split by "lifetime" adds operational cost without measurably improving answer freshness or correctness, the architecture is over-engineered.

## Key quotes

> Documentation is large, the useful slice is small and query-dependent, and no amount of prompt real estate substitutes for the agent being able to look something up at the moment it has the question.

A crisp dismissal of context stuffing that aligns with the retrieval-over-stuffing consensus — but note it is also self-serving, since Uno hosts the docs server and benefits from being the lookup target.

> Pixels are for detection, structure is for diagnosis, and an agent needs both.

The best sentence in the post. It explains precisely why screenshots alone are a weak verification signal and why the XML visual-tree snapshot is the tool that "earns its keep" — a distinction most UI-automation-for-agents discussions blur.

> A tool the model never selects may as well not exist, and the only lever you have over selection is the wording.

Tool descriptions as prompts, not documentation. The concrete example — `uno_app_pointer_click`'s description steering the model toward automation peers — is prompt engineering embedded in a tool schema, which is a genuinely underappreciated surface.

> Generation got cheap. Verification did not.

The economic core of the whole argument, and the same asymmetry driving the review-at-scale and agentic-testing literature. Everything else in the post is infrastructure built to attack this one sentence.

> Split your servers by lifetime, not by feature – knowledge that changes when you ship does not belong in the same process as state that changes when the app runs.

The architectural takeaway, and a genuinely transferable design rule: partition MCP servers by how often their truth changes, not by domain boundaries. It also quietly justifies their hosting decision — doc corrections propagate to every agent on next call rather than waiting for a NuGet release.

## Critical analysis

The non-obvious contribution here is the lifetime-split heuristic. Most "should I build an MCP server" write-ups agonize over tool surface or framework choice; Uno's constraint-driven answer — hosted multi-tenant with OAuth gets HTTP, single-machine child process gets stdio, and the topology decides so there is nothing to agonize over — is refreshingly deterministic. The token-cost disclosure is also unusually honest: 6.4k tokens for the docs server, 1.5k for the app server, benchmarked against GitHub's MCP server at 5.2k. Publishing those numbers invites comparison and gives other teams a baseline for the "permanent tax" every tool definition levies on the context window.

The weak point is that the post is vendor advocacy wearing an engineering-blog costume. Every claim about the verification loop working — the agent "decides for itself whether the change did what was asked" and "fixes it before handing anything back" — is asserted, never measured. There are no numbers on how often the agent's self-verification catches real defects, how often the visual tree misleads it, or how the automation-peer path fails. Compare Slack's agentic-testing study, which ran 200+ executions and reported failure rates; Uno offers conviction. Given the post's own thesis that verification is the hard problem, the absence of verification data about the verification system is a real irony.

What is also left out: cost and latency of the loop. Screenshot-plus-visual-tree-plus-click-through per change is many round trips per edit, and the post is silent on what that does to iteration time or token spend. The security story for the hosted docs server (OAuth is mentioned once, in passing) gets one clause. And the browser-based Studio 3.0 section — Roslyn compiling agent output in-browser with hot reload — is arguably the most technically interesting thing in the post, yet it gets the thinnest treatment, with no discussion of what the "specialized agent orchestrated by Microsoft Agent Framework" actually does differently from a plain coding agent.

Still, as a design reference for anyone building MCP servers around a framework or SDK, this is one of the better sources available: the transport-from-topology rule, the tool-description-as-prompt discipline, and the lifetime split are all directly reusable.

## Related

- [[How AI Coding Agents Actually Use Your Technology]] — Mastykarz argues most agent failures are invisible tool-selection or tool-content failures; Uno's tool-description-as-prompt discipline and published token costs are a concrete practitioner response to exactly that diagnosis, strengthening it with production numbers.
- [[Agentic Manual Testing]] — Willison's case that agents must exercise running code because tests pass while servers crash is the same verification asymmetry Uno names; this source strengthens it by shipping the missing Playwright-equivalent for native .NET apps rather than just advocating the practice.
- [[Agentic Testing]] — Slack's empirical finding that the MCP tool surface quality mattered as much as the model (near-zero failures for persistent DOM view versus 12–20% for CLI snapshots) complicates Uno's unmeasured claims: it suggests the visual-tree-over-pixels bet is plausible but demands the failure-rate data Uno never provides.
- [[Stateless MCP]] — Willison's push toward stateless single-call MCP tools nuances Uno's split: the docs server fits the stateless mold exactly, while the app server is deliberately stateful and session-bound, showing the stateless/spec debate is really about which lifetime of state you are serving.

---
*Sources: [[raw/how-uno-platform-uses-dotnet-mcp-ai-to-build-high-quality-apps]], [[summary/how-uno-platform-uses-dotnet-mcp-ai-to-build-high-quality-apps]]*
*Last updated: 2026-09-15*
