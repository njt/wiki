# AI Brains Landscape (secondcoffee)

An audited October 2026 market map of 53 "AI brain" tools — products and infrastructure that hold memory for people, companies, agents and documents — grouped by job-to-be-done rather than category, with a thesis that the winning product needs both shared company memory and agent-specific adaptive memory, plus governance.

---

## What it is

consulting.secondcoffee.ai's landscape page is the vendor-comparison counterpart to the wiki's memory-architecture sources. It audits 53 entries across four tiers — products you buy, memory infrastructure, build substrates, personal or emerging tools — and re-verified every entry against vendor sites, docs, repos and package registries in mid-October 2026 (251 source URLs, 29 GitHub projects, all unarchived). Its discipline is unusually high for a marketing-adjacent page: every MCP server claim was independently re-checked against npm, PyPI and vendor docs, and one claimed licence was disproven and recorded as a finding.

The page's most useful contribution is not the table but its conceptual split:

> "**Shared company memory** answers: what does the organization know across docs, customers, workflows, decisions, tools, and people? … **Agent-specific adaptive memory** answers: what has this AI worker learned from doing its job for this customer, team, or process?"

This is a buyer's version of the taxonomy problem [[Agent Memory]] solves at the architecture level — Jones's seven types are memory *kinds*; this is memory *ownership*, and the two compose rather than compete.

> "The serious product will need both, plus governance. Otherwise you either get a smart search box that never learns from work, or a learning agent with no trustworthy company context."

The dichotomy sharpens the positioning of [[OzBrain]]: OzBrain bets on one shared brain every agent reads and writes, which is exactly the "shared company memory" pole — the landscape's thesis implies that alone is not enough, because the adaptive, per-agent learning layer is where work actually compounds.

> "**Transactive memory** answers: who or what knows this, where is the authoritative version, and how should the right context be retrieved before work starts?"

The strongest idea on the page, borrowed from organizational psychology (who-knows-what maps): a company brain's job is not to store everything but to route to authoritative sources — which is the same insight [[A Home for Personal Context (Duncan Davidson)]] makes from the individual's side, that the question is a trust boundary and a pointer, not a copy.

## Key themes

- #concept — shared vs. adaptive memory, and transactive memory as the routing layer between them
- #tool — 53-vendor market map, 34 shipping first-party MCP servers
- #pattern — "use cases first, labels second"; tiers separating buyer-facing products from plumbing
- #concept — governance (provenance, retention, memory-poisoning defenses) as a pre-rollout gate

## Analysis

This page is a service to buyers, and it shows: the audit rigor (independently verified MCP claims, open row citations, an honest "this link disproved our claim" finding) is the real differentiator in a category drowning in vendor noise. Its MCP column is quietly the most consequential artifact — by 2026, "does it ship a first-party MCP server" has become the de facto interoperability test for memory products, and the landscape treats it that way rather than as a checkbox.

The thesis is right but undersold: the shared/adaptive split maps onto a governance distinction the page gestures at but doesn't drive home. Shared company memory needs *permission* governance (who may read what); adaptive agent memory needs *truth* governance (what happens when the learned lesson is wrong — the memory-poisoning problem). These fail differently, and a product claiming to do both needs two different control systems. The page lists "memory-poisoning defenses" as one governance item among six, when it is arguably the load-bearing one for the adaptive half.

Where it connects: [[Munder Difflin — Clones of You, Not a Shared Bot]] takes the opposite pole (per-person clones, no shared bot), and this landscape's two-memory thesis suggests both poles are half-right — clones for adaptive memory, a shared substrate for company knowledge. [[Agent Memory]] supplies the type taxonomy the landscape's "check the memory model" advice presupposes.

---
*Sources: [[raw/ai-brains]], [[summary/ai-brains]]*
*Last updated: 2026-10-24*
