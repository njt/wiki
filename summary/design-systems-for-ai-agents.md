---
url: https://www.lukew.com/ff/2164/design-systems-for-ai-agents
title: "Design Systems for AI Agents"
author: Luke Wroblewski
date_fetched: 2026-10-11
date_published: 2026-10-11
topics:
  - agent-coding-workflow
  - agent-architecture
---

Luke Wroblewski (20+ years in design systems) argues that the perennial failure of design systems — they drift out of sync with the product and end up "describing a product that no longer exists" — has a new solution now that AI agents write most code. Rather than throwing more design-system-team resources at the drift, make the design system part of the AI steering layer: the context that keeps every site change aligned with brand and front-end guidelines.

The argument turns on who writes the site. When marketing, engineering, and PMs each ship pages through their own agents, each agent will happily invent its own colors, spacing, and tone unless something steers it toward a unified whole. A design lead and front-end lead define the tokens once, and everyone's agents are "snapped to" them.

In practice this means AGENTS.md instructions to reuse theme classes and CSS variables, design tokens in a global stylesheet (colors, type, grid, dark mode), and a design system page rendered from the same code as the site — so documentation stays current by construction. Wroblewski's punchline: "Design teams have spent years trying to get people to actually read their design system documentation. Agents read it every single time."

The piece draws on real projects (the Intent website, Sol, Aria), including a Figma MCP bootstrapping step after which the code itself became the source of truth.
