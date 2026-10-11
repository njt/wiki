---
url: https://www.lukew.com/ff/2164/design-systems-for-ai-agents
date_fetched: 2026-10-11
---

For as long as I've been working on design systems (20+ years, oof!), I've seen the same pattern play out... design teams invest a lot of time in creating a system, then struggle to keep it up to date as the actual product evolves. Inevitably the design system ends up describing a product that no longer exists.

The most common way to address this was to throw more resources at the problem: design system teams and/or centralized design reviews. But now, AI agents give design systems a different path forward. When agents write most of the code, the design system can become part of what I've called the AI steering layer: the context that keeps every website update aligned with brand, design, and development guidelines. Put simply, a design system for agents is a steering layer for a site's design and front-end code.

With AI agents, anyone on a team can update a website. Marketing adds landing pages, engineering adds docs, a PM tweaks the pricing page. Each of their agents will happily implement its own tone and voice, colors, spacing, etc. unless something steers them toward a unified whole.

That's the case for collaborative steering. A design lead and front-end lead define the grid, fonts, colors, spacing, and components once, and everyone's agents are "snapped to" it. The team keeps shipping and the site keeps its integrity. So what does that look like in practice? Here are a few examples from our recent projects.

On the Intent website, an AGENTS.md file tells AI agents to reuse the theme classes and CSS variables in the site's global stylesheet instead of duplicating color, typography, or grid values. That stylesheet defines colors, fonts, type sizes, the responsive grid, and light and dark themes as design tokens (named values the whole site shares). A design system page shows it all as live examples agents can inspect.

When someone asks an agent to add a new button, it reuses the existing Button component and its styling, which also adapts to dark mode automatically. And no human has to look anything up.

Intent's design system started from Figma specs that were automatically translated into code using a Figma MCP connection. After that, the code itself became the source of truth and was updated with a new body font, responsive layouts, dark mode, and more as the site evolved. We've even had agents write changes back to the original Figma files. But since the code is the source of truth, that hasn't been necessary in practice.

Sol and Aria's websites follow the same approach (links go to their design system pages), each with design tokens, agent instructions, and a design system page that stays current because it's built from the same code as the site.

Design teams have spent years trying to get people to actually read their design system documentation. **Agents read it every single time.**
