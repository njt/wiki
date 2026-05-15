# How Intercom Uses Claude Code

Intercom's internal Claude Code deployment is the most comprehensive enterprise case study published to date: 13 plugins, 100+ skills, hooks enforcing workflow discipline, and full OpenTelemetry observability -- all built by the engineering team, not bought. The system has crossed the line from "developer tool" to "company-wide platform" with non-engineers among its heaviest users.

---

## Precis

Brian Scanlan describes how Intercom transformed Claude Code from an individual coding assistant into a full-stack engineering platform through plugins, skills, and hooks. The standout capabilities: a read-only production Rails console via MCP (with Okta auth and DynamoDB audit trails), a 14-event-type observability pipeline flowing to Honeycomb, a forensic flaky test fixer with a 20-category taxonomy, and a permission analyzer that mines 14 days of session transcripts to auto-generate safety-classified allowlists. The data team independently built "Claude4Data" with 30+ analytics skills spanning Snowflake, Gong, and finance. Design managers, support engineers, and product leaders -- not just developers -- are among the top users.

## Key Quotes

> "It is either the worst thing in the world that will ruin Intercom, or complete genius. It is used a lot. No issues so far."

This quote captures the honest uncertainty of early enterprise AI adoption. Nobody is sure this is safe, but it's already load-bearing.

## Key Themes

#claude-code #enterprise #skills #hooks #observability #MCP #production-access #case-study

**Hooks as enforcement, not suggestion.** PreToolUse hooks intercept raw `gh pr create` commands, forcing developers through the create-pr skill first. Hooks block modifications to merged PR branches. This is [[claude-ctrl]]'s thesis -- "an instruction in context is not a constraint" -- implemented at enterprise scale.

**Observability as first-class infrastructure.** 14 session event types instrumented via OpenTelemetry to Honeycomb. Session transcripts sync to S3. On session end, Claude Haiku auto-classifies improvement opportunities (missing_skill, missing_tool, repeated_failure, wrong_info) and posts to Slack with pre-filled GitHub issue links. This closes the [[Feedback Loop is All You Need]] loop at the organizational level: the system watches itself fail and generates its own improvement tickets.

**Permission management from evidence, not guesswork.** After 5 permission prompts, the system suggests scanning 14 days of session transcripts. Commands get GREEN/YELLOW/RED classification. Safe commands auto-write to settings. This is the first published example of data-driven permission tuning.

**Non-engineers as power users.** The production console's top users are design managers, support engineers, and product leaders querying live data through Claude. This confirms [[Two Kinds of User Are Emerging]] -- the power users are often not the people you'd expect.

**The flaky test fixer is surprisingly rigorous.** 9 steps, 20-category taxonomy, hard rules against skipping specs or guessing without CI data, plus a sweep for sibling instances. This is [[Compound Engineering]] applied to a specific problem: each fix strengthens the taxonomy, not just the test suite.

**Claude4Data as a shadow platform.** The data team independently built 30+ analytics skills. Users span sales, product, and data science. This organic growth pattern suggests that once the plugin/skill infrastructure exists, domain teams will self-serve without central coordination.

## Critical Analysis

This is the strongest evidence yet that Claude Code's plugin/skill/hook architecture can scale to a real enterprise. Most published Claude Code workflows are individual-developer setups; Intercom is running it as organizational infrastructure with observability, audit trails, and cross-functional users.

The production console is the boldest move. Read-only access with Okta auth and DynamoDB audit trails is a reasonable safety posture, but "no issues so far" is not the same as "we have proven this is safe." The quote about it being "either the worst thing in the world or complete genius" is refreshingly honest -- and the honest answer is that nobody knows yet.

The observability pipeline is the real competitive advantage. Most teams using Claude Code are flying blind on how the tool is actually being used. Intercom can see failure patterns, auto-generate improvement tickets, and measure adoption across roles. This is the [[Harness Engineering]] feedback loop operating at the organizational level.

What's missing: cost data. 100+ skills, S3 transcript storage, Honeycomb telemetry, Haiku analysis on every session -- this is not cheap. The article reads like a showcase, not a cost-benefit analysis.

Also missing: failure stories. The flaky test fixer is described in detail, but what about skills that didn't work? With 100+ skills, some must have flopped. The absence of failure data weakens the credibility of the success claims.

The "skill-ify everything" conclusion feels inevitable but under-examined. If every workflow becomes a skill, who maintains 100+ skills? Who tests them? The [[Woodshed]] eval framework exists for exactly this problem, but Intercom doesn't mention anything equivalent.

## Cross-Links

- [[claude-ctrl]] — Same thesis (hooks over prompts), smaller scale. Intercom validates the approach at enterprise level
- [[Compound Engineering]] — The compounding loop applied to an entire engineering org, not just a solo developer
- [[Feedback Loop is All You Need]] — Intercom's observability pipeline is the organizational version of "linters beat prompts"
- [[Harness Engineering]] — Böckeler's framework in practice: computational feedback (hooks, telemetry) plus inferential feedback (Haiku session analysis)
- [[Two Kinds of User Are Emerging]] — Non-engineers as power users confirmed
- [[How Boris Uses Claude Code]] — Creator's individual workflow vs. Intercom's institutional deployment
- [[MinMax Skills]] — Another large skills library, but framework-focused rather than enterprise-workflow-focused
- [[Woodshed]] — The eval framework Intercom probably needs but doesn't mention
- [[Pre-Commit Lint Checks]] — Intercom's hook enforcement is this idea extended beyond linting
- [[Building Agents for Production Systems with MCP]] — Anthropic's MCP guide; Intercom's production console is an advanced implementation
- [[Security and Sandboxing]] — Read-only replicas, blocked tables, and Okta auth as the production safety model
- [[Awesome Agentic Patterns]] — Intercom's 100+ skills as a proprietary instance of the pattern catalogue
- [[CLAUDE.md (Universal)]] — Intercom auto-updates CLAUDE.md files via weekly GitHub Actions

---
*Sources: [[raw/how-we-use-claude-code-today-at-intercom]]*
*Last updated: 2026-05-14*
