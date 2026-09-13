# Kage (Design-to-Prompt Gallery)

Kage (kage.design) is a commercial design-inspiration gallery whose entire pitch is the last mile between taste and agent: browse landing pages taken from real products, and "turn any design into a prompt for Claude Code, Codex or Cursor." The homepage counts 240 designs decomposed into 1,265 components across 135 products, plus 17 "skills" and 7 "tools" of its own. The source is a product homepage, not an essay — the interesting content is what the positioning implies.

---

## Key Quotes

> "Design inspiration, ready to build."

The H1 does a lot of work in four words. "Ready to build" claims the gap between *seeing* a design and *having an agent implement it* has collapsed to a button. Inspiration galleries used to end at the human designer's eye; this one ends at a prompt.

> "Browse beautiful interfaces from real products and turn any design into a prompt for Claude Code, Codex or Cursor."

The value proposition names its addressee, and it isn't a human designer — it's a coding-agent harness. Note the three targets: the generalist agents people already run, not a proprietary builder. Where v0 folded design reference *inside* a closed product, Kage sells the reference layer to everyone else's agents.

> 240 designs · 1,265 components · 135 products · 17 skills · 7 tools

The decomposition is the product. A gallery of screenshots would be a mood board; 1,265 components implies the reference is granular enough to reassemble, and "skills" implies Kage ships agent-side assets, not just a website to look at.

## Key Themes

**#tool — Kage.** The gallery itself: real-product landing pages tagged by category (fintech dominates — ynab, wealthfront, venmo, rocketmoney, Monarch, Lemonade; plus Raycast 2.0, phantom.com, ledger.com, krea.ai, descript.com, sweetgreen.com), converted on demand into prompts for Claude Code, Codex, or Cursor.

**#concept — Reference-as-prompt.** You don't describe the design you want; you cite one. Taste is outsourced to selection rather than articulated in words — the prompt becomes a pointer, and the gallery exists to make pointers plentiful.

**#pattern — Prompt supply chain.** An upstream input vendor for coding agents. As generation gets cheap, the scarce inputs (context, verification, taste) get productized — this is taste, packaged.

**#concept — Aesthetic convergence risk.** A shared reference corpus is a shared aesthetic. If enough agents are prompted from the same 240 designs, the distinctiveness the gallery sells is what it erodes.

## Critical Analysis

**The catalog composition is the tell.** It's overwhelmingly polished consumer landing pages — eight-plus fintech entries, crypto wallets, AI tool sites. Landing pages are the ideal substrate for design-to-prompt: bounded scope, high polish, almost no logic to wire. The implicit concession is that agent-built UIs today are mostly the shallow end — marketing surfaces — and "ready to build" is easiest precisely where there's nothing behind the pixels.

**The commoditization move is clever and fragile.** Selling the reference layer to *every* harness (Claude Code, Codex, Cursor) rather than building the closed agent is the right arbitrage — the reference corpus, not the model, is the asset. But nothing here is defensible: it's one native "design from screenshot" feature, or harnesses indexing the web's designs themselves, away from being absorbed. A gallery is a feature wearing a business model.

**The rights question is invisible on the page.** Turning ynab.com's landing page into a prompt for your own product is design copying with extra steps. Designers have always borrowed, but "turn any design into a prompt" industrializes the borrow — and the page says nothing about licensing, consent, or where inspired-by becomes copied-from. That silence is the product's load-bearing wall.

**Homogenization is the long-term failure mode.** A single curated corpus funnels thousands of builders toward the same layouts, the same spacing, the same hero sections. The gallery's pitch is distinction; its mechanics produce sameness. The web it helps build will need somewhere else to go for inspiration.

**Honest caveat on the evidence.** This is marketing copy scraped from a homepage: no author, no pricing, no methodology for how a design becomes a prompt, no way to verify the counts. The analysis above reads the *positioning*; the product underneath may be better or worse than it claims.

## Related Pages

- [[Lessons from Building Vercel v0 and the d0 Agent]] — Kage is that page's Tailwind-prompt-hack moment, externalized: where v0 folded design reference inside a closed product agent, Kage sells the same input as an open catalog to any harness, confirming design reference as the valuable layer of the generation pipeline.
- [[Writing Style Guides for Better UIs]] — strengthens that page's reference-as-agent-context pattern while complicating it: Langworth feeds *rules* (a style guide the agent applies), Kage feeds *examples* (a design the agent transplants) — rules compress taste, examples copy it, and only the second raises the copying question.
- [[Make Pages Interactive]] — nuances the show-don't-tell loop by moving it upstream: Kage shows the agent a design *before* generation where Paras Chopra's skill collects visual comments *after*; both raise the bandwidth of visual context, and neither closes the verification gap on what the agent actually built.
- [[uiui — Dense Console UI Kit]] — the two opposite directions of design-for-agents: uiui is opinionated (one design language, plus a skill.md so agents "mean the same thing"), Kage is agnostic (any design, transplanted wholesale); the tension between teaching agents taste and lending them references is exactly the space between these pages.

---

*Sources: [[raw/kage-design]], [[summary/kage-design]]*
*Last updated: 2026-09-13*
