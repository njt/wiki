# OpenRewrite Supported Languages

Reference catalog of OpenRewrite's language, data format, build tool, and framework support as of January 2026. Five JVM/JS languages, seven data formats, three build tools, and four frameworks — with Moderne's commercial platform filling the gaps (C#, Python, Ruby, COBOL).

---

## Key Quotes

> "Moderne offers support for additional languages and frameworks (such as JavaScript, C#, Python, Ruby, COBOL, etc.)."

The commercial/OSS split is the real story here. OpenRewrite covers the JVM ecosystem as open source. Everything else — the languages enterprises actually have sprawling legacy in — requires Moderne's paid platform. This is a clean business model: the OSS project proves the approach on Java; the company monetizes the hard polyglot reality.

> "Migration recipes are developed through collaboration between the OpenRewrite team, the original framework authors, and the wider OSS community."

This collaboration model is underappreciated. Framework authors (Spring, Quarkus, Micronaut, Jakarta) co-author the migration recipes because it's in their interest to reduce upgrade friction. The recipe catalog is effectively a collectively-maintained upgrade path — a social contract between framework maintainers and users, encoded as deterministic AST transforms.

## Key Themes

- #tool — OpenRewrite is a code transformation engine, not just a linter. It modifies ASTs with guaranteed correctness for defined recipes
- #pattern — The OSS/commercial split (JVM free, everything else paid) is a recurring open-core model that works when the free tier is genuinely useful on its own
- #concept — "Recipes" as deterministic, authored code transforms are a different category from AI-generated refactors. They're feedforward enforcement ([[Harness Engineering]]) — prevention, not detection

## Critical Analysis

**The page tells you what's supported, not what you can do with it.** This is a capability catalog, not a guide. Without version ranges or per-language recipe counts, you can't assess whether "supports Kotlin" means "has 200 migration recipes" or "can parse Kotlin ASTs and you're on your own." The former is a product; the latter is a platform. The page doesn't distinguish.

**The Moderne upsell is honest but creates a hierarchy.** Java gets the full OSS treatment. Kotlin and Groovy are JVM-adjacent so they ride the same infrastructure. JavaScript/TypeScript support exists but the interesting polyglot stuff — Python, C#, COBOL — is paywalled. For brownfield modernization ([[A Practical Guide to Brownfield AI Development]]), the languages you most need help with are the ones that cost money.

**The framework collaboration model is the most interesting detail and gets the least ink.** Spring, Quarkus, Micronaut, and Jakarta maintainers co-author migration recipes. This means framework upgrades aren't just documented — they're automated. That's a genuine structural advantage over ecosystems where "read the migration guide and grep for deprecated APIs" is the state of the art.

**What's missing:** No mention of custom recipe authoring difficulty. No indication of how many recipes exist per language. No performance characteristics for large codebases. The page is a menu, not a review — useful for checking whether your stack is covered, useless for deciding whether OpenRewrite is worth adopting.

---

*Sources: [[raw/openrewrite-supported-languages]]*
*Last updated: 2026-05-22*
