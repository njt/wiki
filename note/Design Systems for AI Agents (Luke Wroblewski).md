# Design Systems for AI Agents (Luke Wroblewski)

Luke Wroblewski argues that the oldest failure mode of design systems — drifting out of sync with the product until they describe something that no longer exists — is solved not by more design-system staff but by repositioning the design system as part of the AI steering layer: context every agent on the team reads before it ships a pixel.

---

Wroblewski has watched the same pattern for twenty years: a team invests in a design system, then can't keep it updated, and eventually "the design system ends up describing a product that no longer exists." The traditional fix — dedicated design-system teams, centralized design review — treated the symptom. With agents writing most of the code, he argues, the system becomes load-bearing in a new way: it is the context that keeps "every website update aligned with brand, design, and development guidelines."

The real stakes are that *everyone* now ships UI. Marketing adds landing pages, PMs tweak pricing, engineering adds docs — and "each of their agents will happily implement its own tone and voice, colors, spacing, etc. unless something steers them toward a unified whole." His prescription is collaborative steering: a design lead and a front-end lead define grid, fonts, colors, and components once, "and everyone's agents are 'snapped to' it."

The implementation is deliberately mundane: an AGENTS.md telling agents to reuse theme classes and CSS variables, design tokens in a global stylesheet, and a design system page generated from the same code as the site so it stays current by construction. On Intent, Figma specs seeded the tokens via Figma MCP, after which the code became the source of truth — writing changes back to Figma turned out to be unnecessary.

## Key quotes

> "the design system ends up describing a product that no longer exists."

The two-decade diagnosis. Documentation-as-afterthought fails because humans stop reading it the moment it stales.

> "each of their agents will happily implement its own tone and voice, colors, spacing, etc. unless something steers them toward a unified whole."

The genuinely new problem: the audience of a design system has multiplied from designers to every agent on the team, including ones owned by people who never learned what a token is.

> "Design teams have spent years trying to get people to actually read their design system documentation. **Agents read it every single time.**"

The essay's thesis in one line — and the deepest structural change in it. Compliance stops being a cultural problem and becomes a default behavior.

## Themes

#concept — the design system as *steering layer*: context infrastructure, not documentation
#pattern — code-as-source-of-truth; docs generated from the artifact agents edit
#tool — AGENTS.md, design tokens, Figma MCP as the concrete mechanism

## Analysis

The strongest idea here is the inversion of the documentation problem. A design system that lives as instructions-and-tokens in the agent's context is *used* deterministically, whereas human-read documentation is used aspirationally. "Agents read it every single time" is a genuinely new property, and it means design-system investment finally compounds instead of decaying.

Two caveats. First, the essay is thin on enforcement: what happens when an agent *ignores* the steering layer, or when a landing page legitimately needs a new token? Wroblewski says "snapped to," but snapping implies a check, and checks live elsewhere in this wiki ([[Code Standards That Survive the Next Agent]] puts CI on the enforcing side of exactly this line). Second, "the code is the source of truth" quietly demotes designers from authors to bootstrappers — the Figma-to-code pipeline runs one way in practice, which is a real org change presented as a footnote.

Still, this is a crisp articulation of an idea the wiki keeps circling: steering context is now the product surface where team standards live or die. It strengthens [[Steering Claude Code]] with a concrete non-engineering-domain instance; it extends [[Writing Style Guides for Better UIs]] from prose to pixels; and it gives [[Design Language in Three Days (curiosity.ai)]] a maintenance-side counterpart — that source shows agents generating brand from scratch, this one shows agents *keeping* it coherent after everyone else arrives.

---
*Sources: [[raw/design-systems-for-ai-agents]], [[summary/design-systems-for-ai-agents]]*
*Last updated: 2026-10-11*
