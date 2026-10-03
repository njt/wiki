# Your App's Frontend UI Is Now Optional

Richard Seroter's essay argues that AI agents make static frontends optional: users need an app's data and function, not its bespoke interface. He prescribes agent-facing surfaces — CLIs, APIs, skills, MCP servers, and dynamically rendered A2UI/MCP Apps components — and three usage patterns, closing with the warning that agent visitors will soon outnumber human ones.

---

Seroter opens with an inventory everyone recognises — 334 apps on his phone, each demanding its own interface for tasks done three times a year. The thesis: "I *still* need the data or function from that app. But I never want to 'see' it again." Drop the UI. Go headless.

## What to build

**Agent-facing tools.** Computer use exists, but "spelunking your DOM wastes tokens and time" — the economic argument for structured access. He points to Salesforce's headless partnership with Anthropic, Box, HubSpot, and Google's Android CLI, 150 agent skills, and managed MCP servers as evidence that SaaS is already moving. Toolkits now exist to generate MCP servers, skills, and CLIs, so the marginal cost of shipping an agent surface is collapsing.

**Dynamic UI components.** A2UI lets agents send component descriptions that render safely client-side (Angular, React, Flutter) rather than shipping arbitrary HTML. MCP Apps return interactive HTML inside chat surfaces. The point is not to abolish UI but to relocate it: the agent composes it, per user, per context.

## What to do with them

Three patterns: retrieve or act from wherever you already are (Cloud Run's MCP server inside a CLI; a hotel concierge agent inside Gemini Enterprise); build disposable, personalized pages by mashing up components — one adaptive page instead of dozens of static ones; and build wherever-you-want experiences, like a Chrome plugin driving Google Cloud through a remote gcloud MCP server, spinning up a Pub/Sub topic while reading.

## Key quotes

> "I *still* need the data or function from that app. But I never want to 'see' it again. Drop the user interface. Go headless."

The cleanest formulation of the agent-era inversion: the app's value was never its chrome, it was the capability behind the login. This reframes every "engagement" metric a product team tracks.

> "Spelunking your DOM wastes tokens and time."

A one-line restatement of the token-economics case for structured interfaces — the same gap quantified in [[Computer Use is 45x More Expensive Than Structured APIs]].

> "Many websites and mobile apps will have significantly more agent visitors than human ones. Build accordingly!"

The strategic punchline, hedged honestly ("static frontends won't become truly optional for a while"). When agents are the majority traffic, analytics, pricing, and even rate-limiting must be redesigned around non-human users.

## Key themes

#concept (headless-first product design) #tool (MCP servers, A2UI, MCP Apps) #pattern (agent traffic exceeding human traffic) #person (Richard Seroter, Google's headless developer advocacy)

## Analysis

This is a vendor essay wearing an evangelist's hat — Seroter works at Google, and every example is Google Cloud, Gemini Enterprise, or Antigravity. Treat the certainty with a discount. But the core claim is structural, not promotional, and it lands: the app-as-destination competes with the capability-as-tool, and the tool is cheaper to build, cheaper to consume, and composable across apps.

The interesting tension he underplays: headless access undermines the attention business model. If nobody "sees" your app, who sees your ads, upsells, or dark patterns? Headless is only rational for SaaS whose revenue isn't attention-mediated — which is exactly why the examples are enterprise vendors. And "one adaptive page instead of dozens of static ones" quietly relocates QA burden from tested pages to untestable dynamic composition — a governance problem disguised as a UX win.

It also complicates the interface-convergence story told elsewhere in this wiki: rather than one super-app swallowing everything, Seroter's vision is a *component market* — A2UI widgets and MCP Apps as portable UI atoms any surface can render. That's a different (and more interesting) endgame than monopoly chat.

## Related pages

This source strengthens [[How AI Coding Agents Actually Use Your Technology]] — Mastykarz's AX cascade describes how agents discover and invoke tools; Seroter's essay is the vendor-side manual for being discoverable in that cascade, and both converge on "structured access beats DOM spelunking." It gives concrete protocol names (A2UI, MCP Apps) to [[Smart Models Dumb Pipes]]'s end-to-end argument — judgment in the model, execution in dumb pipes — by specifying what the pipes should expose. It nuances [[When Your Buyer Is an AI Agent]] from the buy side to the product surface: if agents outnumber human visitors, the storefront argument extends to *consumer* apps too. And it puts a vendor's thumb on the scale of [[Computer Use is 45x More Expensive Than Structured APIs]] — Seroter concedes computer use exists but costs tokens, the cost side of that quantified gap.

---
*Sources: [[raw/your-apps-frontend-ui-is-now-optional]], [[summary/your-apps-frontend-ui-is-now-optional]]*
*Last updated: 2026-10-03*
