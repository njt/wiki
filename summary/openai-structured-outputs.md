---
url: https://developers.openai.com/api/docs/guides/structured-outputs
title: Structured Model Outputs — OpenAI API Documentation
author: OpenAI
date_fetched: 2026-05-14
date_published: 2024-08-06
---

# Structured Model Outputs

OpenAI API feature that guarantees model responses conform to a user-supplied JSON Schema. Available starting with `gpt-4o-2024-08-06` and `gpt-4o-mini`.

## Two Forms

1. **Function calling** — for connecting models to tools, functions, or system data
2. **`response_format` / `text.format` with `json_schema`** — for structuring user-facing output

## Structured Outputs vs JSON Mode

JSON mode only guarantees valid JSON. Structured Outputs additionally guarantees schema adherence (with supported schemas). JSON mode works with older models (`gpt-3.5-turbo`, `gpt-4-*`); Structured Outputs requires newer ones.

## SDK Approaches

**Python**: Uses Pydantic `BaseModel` with `client.chat.completions.parse()` or `client.responses.parse()`. Response available via `completion.choices[0].message.parsed`.

**JavaScript**: Uses Zod schemas with `zodResponseFormat()` (Chat Completions) or `zodTextFormat()` (Responses API).

**Raw cURL**: Manual JSON Schema with `"strict": true`, `"additionalProperties": false`, all properties in `"required"`.

## Schema Rules for Strict Mode

- Top-level must be `"type": "object"`
- Every property listed in `"required"`
- `"additionalProperties": false` on all objects
- Enums via `"enum": [...]` arrays
- Nullable fields via `"type": ["string", "null"]`

## Edge Cases

1. Incomplete responses — check `finish_reason === "length"` or `response.status === "incomplete"`
2. Refusals — check `refusal` property on message for safety-based refusals
3. Content filter — may block certain outputs entirely

## Limitations

- Not all JSON Schema features supported (performance/technical reasons)
- Fine-tuned models: first request with any schema incurs extra latency; subsequent requests with same schema skip this
- Recursive schemas supported via `z.lazy()` (Zod) or forward-referenced types with `model_rebuild()` (Pydantic)

## Four Use Cases Documented

1. **Chain of Thought** — math tutoring with step-by-step reasoning and final answer
2. **Structured Data Extraction** — pulling title, authors, abstract, keywords from research papers
3. **UI Generation** — recursive schemas for nested UI component trees
4. **Moderation / Content Compliance** — classification with violation flags and category enums
