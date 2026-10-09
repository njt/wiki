---
url: https://blog.fizzbee.ai/formal-analysis-in-requirements-specification/
title: "Your .md File Is Not a Specification: Using Formal Analysis to Find Requirements Gaps"
author: FizzBee team
date_fetched: 2026-10-09
date_published: undated
topics:
  - specifications-as-the-product
  - guardrails-and-feedback-loops
---

A FizzBee blog post arguing that plain-English (or even EARS-formatted) requirements hide gaps that only a formal, executable specification exposes. It walks a salon appointment-booking example through EARS notation, then Dynamic Logic ([A]p / <A>p modalities), then a FizzBee model checked by the FizzBee model checker.

The worked example is the payload. Three requirements ("stylists set schedules; customers book appointments; all appointments are within the schedule") look complete but the model checker immediately finds two underspecified behaviors: booking without a schedule, and a stylist changing their schedule after a booking exists — which forces a genuine product decision (block the change, cascade-cancel, or flag for manual rescheduling) that no amount of prose polishing would surface. Adding cancellation then reveals a third gap (one customer overwriting another's booking) that even the formal invariant missed, prompting the honest caveat that formal methods only find what your assertions ask about.

The post closes by connecting the executable spec to three uses beyond gap-finding: stakeholder exploration of the state space, model-based testing of the implementation (refinement/bisimulation against the spec, without re-asserting the invariant), and agent-assisted spec authoring — FizzBee ships skills for Claude Code, Cursor and Gemini CLI, positioning formal specification as a natural fit for specification-driven development.
