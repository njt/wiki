# TEHAS — A Pi Software Factory

TEHAS is a live control-room website for a self-hosted software factory running on the Pi coding agent and pi-rukas: GitHub issues enter one end, reviewed pull requests come out the other, everything builds in a sandbox on self-hosted models, and a person does the merge. The page itself is a JS-rendered dashboard rather than an essay, so what the wiki ingests is the factory's visible shape — its kanban line, its production console, its energy metering — not an argument about the pattern.

---

## What the page shows

The site describes itself in one sentence, worth quoting whole because it is the entire thesis:

> "The floor of a software factory powered by the Pi coding agent and pi-rukas. GitHub issues go in and reviewed pull requests come out, built in a sandbox on self-hosted models. A person merges."

Four stages make up the line: **In Line / Next Up**, **Building / In Work**, **Review / Needs Review**, **Shipped / Latest** — a straightforward kanban over an autonomous build-and-review pipeline. Around it sit the operational instruments: a Protolab, a Production Console "looking for a running cycle", and an **Energy Use** panel reporting instantaneous watts plus 24-hour and 30-day kWh "serving the factory", with tables of hosts (host, source, RAM) and models (model, hosts, engines, busy, queued).

The touch I find genuinely telling is the **Breakroom**: a Breakout-style game called Buzzword Buster whose bricks are "the commonest words in the ticket titles, shuffled every round". The site is careful to add: "Optional. The score changes nothing on the line." Someone built an idle-time ornament out of their own factory's ticket stream and then explicitly fenced it off from the production semantics.

## Themes

- #tool — a concrete, running instance of the software-factory pattern, on Pi
- #concept — metering the factory's physical footprint (watts, kWh) as first-class UI
- #pattern — human merge as the only stated human gate; sandbox builds; self-hosted models

## Analysis

There is almost no prose here to argue with, which is itself informative: TEHAS is a factory that presents itself as a *place* — a floor, a console, an energy meter — rather than as a product or a blog post. The interface choices encode a philosophy. Energy consumption given equal billing to build status says the operator cares about what the factory *costs*, not just what it ships. Hosting it publicly, with live ticket data, makes the whole loop legible to outsiders — a observability posture most factories in the wiki's literature only describe in essays.

The pipeline shape is the standard one: issues → sandboxed builds → review → human merge. Nothing in the static text suggests multi-agent orchestration beyond what the "hosts / engines / busy / queued" model table implies, so the interesting differentiator versus other factories is locality: self-hosted models, metered energy, one person merging. It reads like a hobby-scale answer to the enterprise factory writing — the same loop, run on a home rack, with the costs displayed on the wall.

## Relation to the wiki

- This is a live, instrumented instance of the pattern described in [[AI Software Factory (Lieben)]] — where that essay theorises stages and self-improvement, TEHAS just runs a line and shows it, and its energy/queue telemetry is exactly the observability that essay says separates buy-scale from build-scale.
- It is also a working sibling to [[Self-Hosted Sandboxed Agentic Software Factory]] — both put sandboxed builds on self-hosted models, so TEHAS strengthens that page's claim that the fully-local factory is practical, and complicates it by showing how much UI plumbing (console, host tables, energy metering) running one actually takes.
- The single stated human gate — "a person merges" — is a data point for [[Human Judgment Doesnt Leave the Software Factory — It Relocates]]: review work sits on the line, but final authority stays with a named human, matching that argument exactly.
- The breakroom game built from ticket titles nuances [[Inside a Software Factory]], which treats the factory as all pipeline; TEHAS adds the sociological touch — the factory's vocabulary becomes the game, and the boundary between factory and breaktime is explicitly marked as load-bearing ("the score changes nothing on the line").

---
*Sources: [[raw/tehas-app]], [[summary/tehas-app]]*
*Last updated: 2026-10-11*
