---
url: https://seroter.com/2026/09/30/your-apps-frontend-ui-is-now-optional/
title: "Your App's Frontend UI Is Now Optional"
author: Richard Seroter
date_fetched: 2026-10-03
date_published: 2026-09-30
topics:
  - mcp-and-tool-protocols
  - ai-product-and-business
---

Richard Seroter (Google) argues that AI agents are making static frontends optional: users still need an app's data and function, but never need to *see* the app. The essay is a developer call-to-action — stop building bespoke interfaces, start exposing capability through agent-facing surfaces.

He splits the advice into two things to build and three things to do with them. Build: (1) CLIs, APIs, skills, and MCP servers so agents get "anywhere access" to data and functionality — he cites Salesforce/Anthropic's headless partnership, Box, HubSpot, and Google's Android CLI, 150 agent skills, and managed MCP servers; (2) A2UI and MCP Apps components for dynamic, safe client-side rendering of agent-composed interfaces. Then do: retrieve data or trigger action from wherever you already are (Cloud Run via MCP inside the Antigravity CLI, a hotel concierge agent inside Gemini Enterprise); build on-the-fly personalized visualizers from A2UI components — one adaptive page instead of dozens of static ones; and build wherever-you-want experiences, e.g. a Chrome plugin driving Google Cloud through a remote gcloud MCP server.

His closing hedge is the honest part: static frontends won't become truly optional for a while, but agent traffic will soon exceed human traffic, so "build accordingly."
