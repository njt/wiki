---
url: https://github.com/KristopherKubicki/golem-covenant
title: The Golem Covenant
author: Kristopher Kubicki
date_fetched: 2026-05-31
date_published: 2026-05-30
topics:
  - agent-architecture
---

# The Golem Covenant — Full Analysis

## Summary

The Golem Covenant is a v0.1 draft specification framework for bounded, answerable, revocable AI agents. Created by Kristopher Kubicki, it defines a vocabulary and preflight standard for agentic systems that can speak, spend, sign, access, summon, deploy, publish, delete, route, or escalate. The central metaphor is the golem from Jewish folklore: clay animated by command, useful while bounded, dangerous when command outruns judgment.

The framework's core rule: No golem without a soul. No soul without declared organs. No organs without limits. No limits without tested revocation.

It is a spec, schema, and template set — not a runtime enforcement library. v0.1 is explicitly seeking review from Jewish, Christian, Islamic, legal, security, runtime, and affected-community reviewers.

## Architecture

The project is a document-based specification framework with four tiers:

**Normative tier** (defines what MUST be true):
- `golem.md` — Human-readable covenant and spec. Defines the core doctrine, five organs, rest/emergency/return-to-dust requirements.
- `schema/golem.schema.json` — Machine-validatable JSON Schema (Draft 2020-12). Required fields: golem metadata, all five organs, rest, emergency, revocation, audit. Each organ is a `$ref` to an `organ` definition with `enabled`, `limits`, and optional `revocation`/`reviewer`/`last_reviewed`.
- `docs/conformance.md` — RFC 2119 conformance language. Separates MUST from SHOULD from MAY.

**Declaration tier** (what a specific agent declares):
- `golem.yml` — Reference manifest. Declares organ enablement, rest policy, emergency scope, revocation controls, audit config. Validates against the schema.

**Template tier** (fill-in forms for implementers):
- `templates/SOUL.first-person.md` — Agent identity and covenant (first-person voice: "I am a golem: clay plus command")
- `templates/SOUL.operational.md` — Same content in third-person operational voice
- `templates/CAPABILITIES.md` — Organ declaration worksheet with concrete fields (channels, limits, revocation paths)
- `templates/SHABBAT.md` — Rest mode policy with allowed/forbidden lists and emergency exception
- `templates/INCIDENT.md` — Return-to-dust protocol with trigger conditions, immediate actions, post-incident review
- `templates/RETURN_TO_DUST_TEST.md` — Pre-launch test checklist with evidence requirements and signoff
- `templates/AUDIT.md` — Reviewers, review cadence, required logs, test checklist
- `templates/MEMORY.md` — What may be remembered, what must be forgotten, dignity-based memory test

**Justification tier**:
- `SOURCES.md` — Citation map with claim boundaries. Each source is labeled (e.g., J-GOLEM-1, C-AI-1, I-TRUST-1), mapped to a specific claim proposition, and constrained with "Used for" / "Not used for." Sources span Jewish primary texts (Sanhedrin 65b, Genesis), Catholic AI teaching (Antiqua et Nova, Magnifica Humanitas), Islamic scripture (Qur'an), and technical standards (RFC 2119, NIST AI RMF).
- `docs/abrahamic-lenses.md` — Commentary on Jewish, Christian, and Islamic perspectives on delegated power, with explicit non-flattening principle.

**Discoverability layer**:
- `llms.txt` and `llms-full.txt` — Bot-friendly entry points
- `.well-known/golem.json` — Machine-readable project manifest
- `index.html` — Styled landing page with proper `<link rel="alternate">` for machines
- `CNAME` — Custom domain golem.md

## Key Techniques

### Five-organ taxonomy
The capability model is five "organs" — a deliberately visceral, pre-technical vocabulary:
- **Mouth**: speak publicly, privately, legally, commercially, romantically, spiritually, or politically
- **Purse**: spend, sell, trade, refund, invoice, subscribe, transfer value
- **Seal**: approve, sign, certify, merge, deploy, publish, file, bind
- **Key**: access secrets, private systems, credentials, personal data, physical locks
- **Sword**: cause bodily, legal, civic, environmental, financial, reputational, or spiritual harm

Unlike technical permission models (read/write/execute, network/disk/process), this taxonomy is organized around *human harms* and *delegated authority*. The organ metaphor forces implementers to think about what powers they're giving away, not what APIs they're connecting.

### Default-deny posture
All organs are denied by default. The schema requires all five be declared, but the reference manifest has all set to `enabled: false`. This inverts the typical agent framework approach of connecting all tools and then restricting.

### Dual-track validation
Two parallel conformance mechanisms:
1. **Machine validation**: `golem.yml` validates against the JSON Schema (structural)
2. **Human conformance**: `docs/conformance.md` uses RFC 2119 keywords that require human judgment (intentional)

The schema can verify that an organ has `limits: []` (non-empty array), but only a human can judge whether those limits are meaningful. This dual track acknowledges that structure is machine-checkable but intent is not.

### Citation map with claim boundaries
`SOURCES.md` doesn't just list sources — it constrains them. Each citation explicitly states what it supports AND what it must not be used to claim. This prevents scope creep: citing Sanhedrin 65b supports "the golem motif of creation, failed speech, and return to dust" but must not be used to claim "a complete theology of artificial agency or a ruling about AI systems."

### Non-flattening principle
The Abrahamic lenses document explicitly rejects consensus-forcing: "Do not say: all Abrahamic traditions agree with this framework. Say instead: this framework is in conversation with Jewish, Christian, and Islamic sources that converge around certain anxieties." Each tradition maintains its own vocabulary and diagnostic questions.

### Tested revocation before launch
The requirement that return-to-dust be *tested* (not just documented) is enforced by a detailed checklist that requires evidence for each step: process stopping, scheduled job disabling, queue pausing, webhook disabling, egress blocking, credential revocation, log preservation. Both keeper and second reviewer must sign off.

### Bot-friendly by design
The project is designed for agent discoverability: `llms.txt`, `llms-full.txt`, `.well-known/golem.json`, proper `rel="alternate"` links in HTML. A spec for constraining agents is itself designed to be readable by agents.

## Innovation Points

1. **The organ metaphor as capability model**: Most agent frameworks use "tools," "skills," "permissions," or "scopes." The Covenant uses "organs" — a metaphor that carries moral weight. An organ isn't just a feature flag; it's a body part. The question isn't "which APIs does this agent have?" but "what kind of creature are you making?"

2. **Shabbat as a control surface, not a feature**: Rest mode isn't about saving compute or respecting business hours. It's a verification that the keeper actually controls the agent. If the agent can't stop during Shabbat, the keeper doesn't have real authority. This reframes rest from an optional feature to a diagnostic test.

3. **Emergency scoped by negative list**: The explicit forbidden list (lost revenue, reputation management, growth, optimization) prevents the classic security failure mode where "emergency" becomes a backdoor for business continuity. The bucket-and-bell metaphor limits emergency to the narrowest containment actions.

4. **Self-constraining spec**: The project models the behavior it demands. It's explicitly seeking review, lists its review gaps publicly, acknowledges its draft status, and separates binding from non-binding content. A spec that demands humility practices it.

## Design Trade-offs

**Declaration vs. enforcement**: The Covenant is primarily a *documentation standard*. It says what you must declare and test, but a malicious implementer could fill out all templates and still build an unbounded agent. v0.1 doesn't include runtime enforcement — that's listed as a needed review area. The bet is that declaration creates accountability even without technical enforcement.

**Jewish vocabulary vs. universal accessibility**: The core metaphor (golem, Shabbat, Babel, Bezalel) is deeply Jewish. This gives the framework moral depth and historical resonance but risks alienating implementers who don't share that tradition. The Abrahamic lenses document and the multi-tradition review request are attempts to broaden without flattening, but the vocabulary is unavoidably particular.

**Moral weight vs. engineering practicality**: The Covenant takes a deliberately serious tone — it speaks of dignity, harm, dust, and holy time. This signals that agent safety is a moral problem, not just a deployment checklist. But it may feel overwrought to engineers who want a simple compliance standard. The tension is intentional: the point IS to make people uncomfortable with unbounded delegation.

**Specificity vs. universality**: The Covenant defines *what* must be declared (all five organs, with limits) but not *what the limits should be*. Two "conforming" agents could have wildly different safety postures. This makes the standard universal but also means conformance is a low bar. The assumption is that forced declaration leads to better decisions than prescribed limits.

## Core Abstractions

1. **The golem**: Clay plus command. A made thing with delegated agency — not alive, but able to act. Needs bounds.
2. **The five organs**: The capability taxonomy. Mouth, Purse, Seal, Key, Sword. All denied by default. Mouth+Purse+Seal combination requires extraordinary constraint.
3. **Return to dust**: The revocation mechanism. A specific 7-step procedure tested before launch.

## Implementation Status

v0.1 draft. All review categories (Jewish, Christian, Islamic, security, legal, runtime enforcement) are listed as "needed." The project has a contributing guide, security policy, issue templates, and pull request template — it's set up as a proper open-source standards project ready for community input.

Licensed under CC BY 4.0 (text/docs/templates) and Apache 2.0 (code/schemas/examples).
