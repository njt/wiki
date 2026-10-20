---
url: https://netflixtechblog.com/ontology-driven-observability-building-an-end-to-end-knowledge-graph-at-netflix-scale-421392cf5adf
title: "Ontology-Driven Observability: Building an End-to-End Knowledge Graph at Netflix Scale"
author: Prasanna Vijayanathan, Renzo Sanchez-Silva (Netflix Technology Blog)
date_fetched: 2026-10-20
date_published: 2026-10-17
topics:
  - guardrails-and-feedback-loops
  - agent-architecture
---

Netflix's Observability Ontology is a knowledge graph — RDF/OWL/SHACL/SPARQL, persisted to their homegrown RDF store QuipuDB (built on redb) — that unifies MELT telemetry, incidents, deployments, alerts and incident-chat reasoning into typed triples across twelve namespaces. The thesis: end-to-end incident investigation is slow not for lack of data but because each tool (Atlas, Radar, PagerDuty, Spinnaker, incident.io) is internally coherent but externally isolated, disagreeing on names and types. The ontology is the machine-readable agreement that fixes that.

The striking move is converting stochastic inputs into deterministic output. LLMs summarise incident Slack channels, fuzzy matchers resolve service names, a probabilistic correlation engine proposes causal chains — but nothing enters the graph that violates the type system. An "entropy ladder" of five layers, and dozens of sequential best-effort enrichers that each emit typed triples and read what earlier enrichers wrote, turn non-deterministic processes into a graph that answers a SPARQL query the same way every time. Model identity is enforced hard: one authoritative catalog (~49,000 entries) gates every ops:Application; unvalidated strings never become entities; dependency edges are capped both directions.

On top sits AutoRCA, an agentic system of specialist agents whose shared interface is the ontology itself — each specialist writes and reads triples about ops:Application without knowing the others exist. The claimed win is honest: the alert still fires, the engineer still fixes; what the graph absorbs is the hour or two of context assembly. The "knowledge flywheel" (Observe → Enrich → Infer → Adapt) aims to turn discovered causal chains into detection rules, so the graph "gradually stops being a schema and starts being a memory." Future work: integration with Upper, the domain-modeling language of Netflix's Unified Data Architecture.
