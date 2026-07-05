---
title: AI Pricing
url: https://ai-pricing.fyi/
fetched: 2026-05-14
description: Live, queryable per-token pricing for AI model APIs across major providers.
doc_version: "1.0"
last_updated: "2026-05-07"
---

# AI Pricing

AI Pricing tracks current and historical API pricing for AI model providers,
including OpenAI, Anthropic, Google, DeepSeek, GLM, Qwen, MiniMax, Mistral,
xAI, Groq, Fireworks, Together, DeepInfra, Cloudflare Workers AI, Novita,
Cerebras, Nscale, DigitalOcean, and Kimi.

## What This Site Provides

- Public JSON endpoints for providers, models, current prices, price filters,
  recent changes, and GPU placeholder data.
- A React web interface for browsing providers, models, and prices.
- Markdown API documentation for agents and command-line clients.
- Per-token price rows with provider, model, metric, billing tier, unit,
  currency, and latest observation timestamp.

## API

- API overview: /api.md
- OpenAPI schema: /openapi.json
- JSON API root: https://ai-pricing.fyi/v1
- Markdown docs: https://ai-pricing.fyi/api.md

## Terminology

- **Provider:** An inference platform that publishes or hosts model pricing.
- **Serving provider:** The platform that serves an offer, such as `cloudflare`;
  distinct from the canonical model vendor.
- **Canonical vendor:** The creator/vendor credited on the canonical model row,
  such as `anthropic` for Claude models.
- **Canonical model:** A normalized model identity used across providers.
- **Offer:** A provider-specific product row for a model and billing mode.
- **Price dimension:** A metric such as `input_token`, `cached_input_token`,
  or `output_token`, priced per unit.
- **Current price:** The latest observed snapshot for one price dimension.

## API Endpoints

### GET /v1/providers
Lists every tracked inference platform. Parameters: active (boolean), limit (default 50), offset (default 0).

### GET /v1/providers/:slug
Details for one provider by slug.

### GET /v1/models
Lists canonical models. Filters: vendor, provider, deployment_mode, family, open_weights, model_type, input_modality, output_modality, capability, model_group. Limit default 50, max 1000.

### GET /v1/models/:key
One model by canonical slug or model_group_key. Follows 301 redirects for group keys.

### GET /v1/prices/current
Latest numeric price for every active priced offer. Filters: provider, model, family, q (search), model_type, input_modality, output_modality, capability, model_group, metric, billing_basis, billing_tier, region. Limit default 100.

Metrics: input_token, cached_input_token, output_token, cache_write_5m_token, cache_write_1h_token.
Billing tiers: standard, batch, flex, priority, turbo.

### GET /v1/prices/filters
Discovers available filter values. Useful for building dropdowns.

## Rate Limits

60 requests/60s on /v1/prices/* endpoints; 120 requests/60s on others.

## Key Data Points

19 providers tracked. No authentication required. Read-only API. JSON responses. OpenAPI schema available. Markdown docs served via Accept header.

Example: Claude Opus 4.7 input tokens at $5/1M tokens (standard tier, Anthropic direct).
