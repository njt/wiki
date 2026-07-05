# Radical Accountability

Wes McKinney's argument that AI has eliminated the excuse of insufficient engineering time. If you can have exactly what you want at near-zero cost, mediocre software reflects bad taste or bad judgment, not resource constraints. The era of "we'd love to fix that but we don't have the bandwidth" is over.

---

## Key Quotes

> "AI is going to create... radical accountability for creators of software projects."

> "Right now you can have exactly what you want. You no longer have to make compromises."

> "The only excuse is that you don't know what the right thing to do is or you have bad taste."

> "2026 is going to be about... taking everything bad and mediocre, burning it to the ground."

## Key Themes

#agentic-coding #management #simplicity

McKinney traces the argument through his own history: pandas was born from direct user feedback during the 2008 financial crisis. Now he champions individual creators building bespoke solutions (Spicy Takes, Money Flow, MSGVault, Roborev) at near-zero cost.

The radical accountability thesis: when building is cheap, quality evaluation shifts from features to credibility. Can you trust the creator? Do they understand the domain? Do they have good taste? The sales pitch changes from "look at our feature list" to "look at who built this and why."

McKinney notes that database systems and data engineering remain AI-resistant -- the Beaver benchmark shows frontier models struggling with complex real-world SQL. This suggests radical accountability hits application-layer software first and infrastructure later.

Connects to [[The Claude C Compiler]] (Lattner's "deciding what should be built" as the scarce resource), [[Zero Alignment]] (Appleton's "opportunity cost becomes the real cost"), and [[Simplicity in the Age of AI-Assisted]] (if building is cheap, why tolerate inherited complexity?). McKinney is also the creator of [[Kata]], the local-first issue tracker -- he's building the tools for the world he's describing.

## Critical Analysis

The thesis is provocative and largely correct at the individual level. Solo developers with good taste can now build alternatives to mediocre enterprise software at remarkable speed.

The weakness: organizations aren't solo developers. The "radical accountability" frame works for indie tools and personal software but doesn't address the coordination problems that [[Zero Alignment]] identifies. A team of ten with AI doesn't automatically produce better software than a team of one -- it often produces worse software, faster. The accountability McKinney describes requires both taste and the authority to act on it, which most engineers in most organizations don't have.

---
*Sources: [[summary/radical-accountability]]*
*Last updated: 2026-05-14*