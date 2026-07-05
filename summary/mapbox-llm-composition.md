---
url: https://ridgetext.com/blog/mapbox-llm-composition
title: "Mapping with In-Memory Layers to Reduce LLM Overload"
author: Allan Bogh
date_fetched: 2026-07-05
date_published: 2026-06-18
---

# Mapping with In-Memory Layers to Reduce LLM Overload

**Author:** Allan Bogh
**Publication:** RidgeText Blog (RidgeText SMS AI)
**Date:** June 18, 2026

## Summary

The article describes how RidgeText, an SMS-based orchestration layer on top of an LLM, built a Mapbox-compatible map compositor that keeps GeoJSON data out of the LLM's context window.

## The Core Problem

Passing large datasets through tool calls is untenable. A modest wildfire dataset in GeoJSON can be "50–500KB of raw GeoJSON," which at ~4 bytes per token translates to roughly 125,000 tokens — exceeding many context windows and creating high costs. The LLM becomes "a pipe for data it cannot reason about."

## The Layer-First Pattern (Solution)

Instead of returning GeoJSON to the LLM, each data-fetching tool stores results server-side and returns a lightweight acknowledgment (~50 bytes). The sequence:

1. `retrieve_wildfire_layer` → returns `{ status: "queued", layerId: "wildfires-0", featureCount: 847 }`
2. `retrieve_trail_layer` → returns `{ status: "queued", layerId: "trail-1", featureCount: 1 }`
3. `generate_map` → returns `{ mapUrl: "https://storage.../map-abc123.jpg" }`

The LLM sees only ~150 tokens in tool results rather than 125,000+.

## Architecture

Each `retrieve_*` call appends to an ordered layer array held in request context. The `generate_map` function renders layers in insertion order — "exactly like Mapbox's layer stack." Implementation uses an in-process `Map` keyed by session ID with a 30-minute TTL for automatic eviction.

The render pipeline fetches a Mapbox Static API base image (terrain, roads, labels), then composites data layers on top using `sharp`. The renderer is swappable — a headless Mapbox GL JS instance running in Playwright could replace static tiles "without any changes to the tools or the LLM's interface."

## Tradeoffs

**Gains:**
- Context window stays small regardless of dataset size
- Deterministic, independently testable render pipeline
- New layer types don't require LLM interface changes

**Losses:**
- The LLM "can't reason about the underlying geometry" of queued layers
- The layer queue is ephemeral, so multi-turn map refinement requires re-fetching (author suggests persisting layers to a database table as a fix)

## Applicability Beyond Maps

The author identifies a general pattern: when "Tool A fetches data → LLM receives it → LLM passes it directly to Tool B," the LLM is misused as a data pipe. Three examples:

1. **Multi-source data enrichment** — compositors merge datasets server-side rather than passing them through the LLM
2. **Log analysis with multiple passes** — fetch logs once, let multiple analytical tools read from the stored result
3. **Any ETL pipeline** where "the output is what matters" — the LLM should orchestrate and describe, not participate in merging

## Underlying Philosophy

> "If the LLM isn't making a decision based on the content, it shouldn't be holding the content at all."

Tool design should shape what the LLM sees so that "the range of reasonable responses all lead to correct outcomes."
