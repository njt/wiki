---
url: https://github.com/reticlehq/reticle
title: "Reticle — the proof layer for AI agents"
author: reticlehq
date_fetched: 2026-11-13
date_published: 2026 (repo, v3.9.0)
topics:
  - guardrails-and-feedback-loops
  - coding-agents-and-frameworks
---

Reticle is an open-source (Apache-2.0) pnpm/Turborepo monorepo that verifies what AI coding agents claim about web apps. A dev-only SDK embedded in the running app instruments the DOM, network, console, routing and framework state; a local Node server bridges that SDK to the agent over MCP (`reticle_look`, `reticle_act`, `reticle_assert`, `reticle_act_and_wait`); the verdict — `yes · no · unknown` — comes back with evidence and the `file:line` to fix. The tagline: "Your AI agent says 'done.' Reticle finds out if that is true."

The key positioning is against screenshots: a screenshot is ~1,365 image tokens per look, slow, non-deterministic, and blind to non-visual failure (the failed request, the console error, the route that didn't change). Reticle reads the accessibility tree and network log as compact structured data instead — an order of magnitude cheaper and it sees bugs pixels can't.

Three ideas carry the design. **Assert the consequence, not the appearance**: assertions are graded in tiers — an app-emitted signal is strongest, network/route/state is next, DOM presence is the weak fallback the tool nudges away from — and the verdict reports which tier it used. **Record once, replay deterministically**: a flow is recorded with the agent, then replayed in CI with no model at all, re-resolving durable element anchors and re-asserting the declared consequence at ~0% flake; self-healing rebinding fires only if the consequence still verifies. **Independence of evidence**: the bundled Open Verification Protocol (OVP) formalises whether an observation could have been influenced by the action it is evidence for — the app's own report of its own success is not proof.

The repo is unusually disciplined for its size (~4,100 files): hard import boundaries between packages (the browser SDK may never import Node APIs, the server never DOM APIs), a zod-parsed wire contract in `@reticlehq/core`, a rules engine (`@reticlehq/engine`, ~16.6k lines) that is deliberately free of browser and daemon code, an extensive benchmark harness with false-green and honesty scorecards, and per-framework `init` detection covering Next, Vite, Angular, Nuxt, Django, Streamlit, Electron and Tauri.
