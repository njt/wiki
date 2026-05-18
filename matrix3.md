# matrix³

Tavis Ormandy's experimental Chrome extension that resurrects the spirit of uMatrix under Manifest V3's constraints, using declarativeNetRequest and CSP directives instead of the now-banned webRequest blocking API. A prototype that proves you can still build meaningful content control in the post-MV3 browser — just differently.

---

## Key Quotes

> matrix³ is an experimental content policy manager, inspired by umatrix, but built on declarativeNetRequest.

Ormandy is direct about the lineage. uMatrix was the gold standard for per-site resource control; MV3 killed the API it relied on. Rather than complaining, he built the adaptation.

> This extension basically just provides an interface to Content-Security-Policy, if you're familiar with the CSP3 specification you'll be familiar with this extension.

The framing is humble but sharp: the extension isn't adding a new security layer — it's giving users direct control over a mechanism the platform already ships. CSP is built into every browser. matrix³ just makes it wieldable.

> Any subresource that was denied by this extension is highlighted in orange. You can continue to enable things until the website works, then click Reload to refresh the tab. When you're happy with your settings, clicking Commit will make them persistent.

The workflow is the insight: **observe what breaks, selectively unbreak, commit**. This is the same iterative-tightening loop that makes linter ratchets work — start restrictive, relax only what you need, and make the relaxation explicit and persistent.

> A group is a named bundle of origins you frequently want to trust together.

The group abstraction solves the combinatorial explosion problem. Instead of toggling 15 CDN origins per site, you define a "CDNs" group once and Trust/Untrust it as a unit. This is the same pattern as skills bundling tools for agents — one decision, not fifteen.

## Key Themes

- #tool — browser extension for per-site content policy management
- #concept — declarative enforcement beats imperative checking (the CSP/declarativeNetRequest version of "linters beat prompts")
- #pattern — iterative tightening: start strict, observe breakage, selectively relax, commit
- #person — Tavis Ormandy, security researcher behind many high-impact vulnerability discoveries

## Critical Analysis

**The tool-as-spec-exploration.** matrix³ is interesting less as a product and more as a proof that MV3's constraints aren't the end of user-controlled content policy. Ormandy is mapping the terrain: what CAN you still do with declarativeNetRequest + CSP? The answer turns out to be "quite a lot." The extension is explicitly a prototype, and that's the right posture — it's an exploration of the design space, not a product launch.

**Declarative rules are the pattern that keeps winning.** The wiki is full of this insight in other domains ([[Feedback Loop is All You Need]], [[claude-ctrl]], [[Ratchets in Software Development]]), but here it plays out in browser security architecture. webRequest let extensions run arbitrary JS on every network request — powerful, flexible, and a privacy nightmare. declarativeNetRequest replaces that with a static rule engine: you declare what to block, the browser enforces it without seeing your code. Less expressive, more trustworthy. The same tradeoff as lint rules vs. code review checklists.

**The group abstraction is underrated.** Defining named bundles of origins and applying them as a unit sounds trivial, but it's the difference between "I configured this once" and "I'm constantly fiddling with per-site settings." It's the configuration equivalent of Don't Repeat Yourself — factor out the common decisions, apply them as a named thing.

**The "ignore" group is the sharpest UX decision.** Most content blockers show you everything they blocked. matrix³ lets you hide origins you don't care about. This is the same insight as filtering noise from a log stream — the useful information isn't "everything," it's "what's surprising."

**What's missing:** No packaged release, no Chrome Web Store listing, no Firefox port (which would need a different approach since Firefox still supports webRequest blocking). This is a researcher's prototype, not a consumer product. The question is whether someone picks it up and ships it, or whether it remains an existence proof.

---

*Sources: [[raw/matrix3]]*
*Last updated: 2026-05-18*
