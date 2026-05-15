# World Monitor

A real-time global intelligence dashboard with AI-powered news aggregation, geopolitical monitoring, and infrastructure tracking. 500+ curated feeds, 65+ data sources, 45 geospatial layers, 21 languages, local AI via Ollama.

---

## Key Quotes

> "500+ curated news feeds across 15 categories, AI-synthesized into briefs"

## Key Themes

#tool #geopolitics #intelligence #dashboard #osint

This is an absurdly ambitious project -- a unified situational awareness interface that aggregates news, financial data, geopolitical signals, climate data, aviation tracking, and cyber intelligence into a single dashboard. The dual mapping engine (3D globe via globe.gl, flat WebGL via deck.gl) with 45 geospatial layers gives it the feel of a military command center you can run on your laptop.

The Country Intelligence Index with composite risk scoring across 12 signal categories and cross-stream correlation (linking military, economic, disaster, and escalation signals) goes beyond simple aggregation into analysis. The local AI via Ollama means the analysis runs without sending your queries to an external API.

54.1k stars suggests this hit a nerve. Five site variants from a single codebase (world, tech, finance, commodity, "happy") show the architecture is flexible enough to serve different audiences.

## Critical Analysis

The scope is the weakness. Maintaining 500+ news feeds and 65+ data sources is an operational nightmare. Feeds break, APIs change, sources go down. The question is whether the community behind those 54k stars actually maintains the data sources or whether they star and move on.

The AGPL-3.0 license with a commercial license requirement is a pragmatic choice for sustainability but limits adoption by companies who might otherwise deploy it internally.

The real competition is Bloomberg Terminal for finance, Palantir for intelligence, and plain RSS readers for news. World Monitor tries to be all three, which means it's probably worse than each specialist at their core job. But if you can't afford Bloomberg and don't have a Palantir contract, this is a remarkable free alternative.

---
*Sources: [[raw/worldmonitor]]*
*Last updated: 2026-05-14*
