# Auditing Legacy Rails Codebases

Ally Piechowski's compact diagnostic for auditing legacy Rails applications — nine questions organized by audience (developers, CTOs, stakeholders) that surface the friction points no one volunteers in status meetings. Simon Willison surfaced it on his linkblog in March 2026.

---

## Key Quotes

Piechowski's nine diagnostic questions, tiered by who needs to answer them:

> "What's the one area you're afraid to touch?"
> "When's the last time you deployed on a Friday?"
> "What broke in production in the last 90 days that wasn't caught by tests?"

These three developer-facing questions are the sharpest of the set. The first is a fear thermometer — every legacy codebase has a haunted room, and asking directly surfaces it faster than any static analysis tool. The second is a deployment-confidence proxy: Friday deploys aren't inherently bad, but *flinch at the question* tells you everything. The third is a test-suite honesty check — if your tests aren't catching production failures, they're cosplay.

> "What feature has been blocked for over a year?"
> "Do you have real-time error visibility right now?"
> "What was the last feature that took significantly longer than estimated?"

The management-tier questions target organizational scar tissue. "Blocked for over a year" identifies the dependency or architectural bottleneck everyone's learned to route around. "Real-time error visibility right now" is the question that separates teams with production discipline from teams running on vibes. The estimation question surfaces where the codebase's complexity has outrun the team's mental model.

> "Are there features that got quietly turned off and never came back?"
> "Are there things you've stopped promising customers?"

The stakeholder questions are the ones engineers rarely think to ask. Quietly killed features are technical debt in product form — dead code with a customer-facing tombstone. What you've stopped promising is the gap between the product you market and the product you can actually ship.

---

## Key Themes

- **#pattern** — Diagnostic questions as lightweight codebase audit: nine targeted questions can surface more than a week of code review
- **#concept** — Fear as signal: the code you're afraid to touch IS the technical debt that matters — everything else is cosmetic
- **#concept** — Audience-tiered diagnosis: developers, managers, and stakeholders see different symptoms of the same disease; a good audit samples all three
- **#pattern** — Production reality over test coverage: the gap between what tests catch and what breaks in production is the only metric that counts

---

## Critical Analysis

The elegance here is that none of these questions require looking at code. That's not laziness — it's methodology. Codebase quality has two dimensions: the code itself (cyclomatic complexity, test coverage, dependency freshness) and the *organizational relationship* to the code (fear, avoidance, learned helplessness). Static analysis answers the first; Piechowski's questions answer the second. The second is almost always the binding constraint.

The developer questions are the strongest because they're falsifiable. "What are you afraid to touch?" — the answer is a specific file or module you can go inspect. "When did you last deploy on Friday?" — git log has the answer. "What broke in production?" — incident reports exist or they don't. The CTO questions drift toward subjective self-assessment, and the stakeholder questions rely on institutional memory that may have already walked out the door. If you only ask three, pick the developer set.

What's missing: there's no question about *why* the code got this way. A Rails app with haunted rooms and killed features didn't rot by accident — there were deadlines, turnover, architectural bets that didn't pay off. Understanding the *how we got here* matters because it tells you whether the rot is accelerating or stable. An audit that diagnoses symptoms without understanding causes produces a cleanup plan, not a prevention plan.

The Rails framing is incidental — these questions work for any legacy web application. The "deploy on Friday" question might be the only Rails-specific tell (Rails shops have a strong cultural norm against it). Everything else is portable.

This pairs well with [[A Practical Guide to Brownfield AI Development]] — Pupius tackles the structural work of making a legacy codebase agent-modifiable, while Piechowski diagnoses the organizational relationship to the code that determines whether anyone will bother. See also [[Discovery Debt]] for the product-side cousin of technical debt, [[Frozen Test Fixtures]] for a concrete testing pathology these questions would catch, and [[Ratchets in Software Development]] for the cheapest possible enforcement mechanism once you've identified what needs to change.

---

*Sources: [[raw/ally-piechowski]]*
*Curated by: [[Simon Willison — Engineering Practices That Make Coding Agents Work|Simon Willison]]*
*Last updated: 2026-07-11*
