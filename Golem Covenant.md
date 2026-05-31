# Golem Covenant

A v0.1 draft specification framework for building AI agents that are bounded, answerable, and revocable. Instead of the usual permission model, it uses the golem from Jewish folklore as a central metaphor: clay animated by command, useful while bounded, dangerous when command outruns judgment. Its core rule: no golem without a soul, no soul without declared organs, no organs without limits, no limits without tested revocation.

---

## Architecture

The Golem Covenant is a **document-based specification framework**, not a runtime library. It has four tiers:

**Normative tier** — what MUST be true:
- `golem.md` — human-readable covenant and spec
- `schema/golem.schema.json` — JSON Schema (Draft 2020-12) for machine validation of agent manifests
- `docs/conformance.md` — RFC 2119 conformance language (MUST/SHOULD/MAY)

**Declaration tier** — what a specific agent declares:
- `golem.yml` — reference manifest declaring which organs are enabled, rest policy, emergency scope, revocation controls, and audit config

**Template tier** — fill-in forms for implementers:
- `templates/SOUL.first-person.md` — the agent's own covenant, written in first person ("I am a golem: clay plus command")
- `templates/CAPABILITIES.md` — organ declaration worksheet with concrete fields per organ
- `templates/SHABBAT.md` — rest mode policy with explicit allowed/forbidden lists
- `templates/INCIDENT.md` — return-to-dust protocol with trigger conditions and immediate actions
- `templates/RETURN_TO_DUST_TEST.md` — pre-launch test checklist requiring evidence for each step
- `templates/AUDIT.md` — reviewer assignments, review cadence, required logs
- `templates/MEMORY.md` — what the agent may remember, must forget, and the dignity-based memory test

**Justification tier**:
- `SOURCES.md` — citation map where every source is tagged with what it supports AND what it must NOT be used to claim
- `docs/abrahamic-lenses.md` — commentary through Jewish, Christian, and Islamic perspectives with an explicit non-flattening principle

The project is also bot-friendly by design: `llms.txt`, `.well-known/golem.json`, and proper `rel="alternate"` links mean agents and crawlers can discover the spec without scraping HTML.

## Key Techniques

### The five-organ taxonomy

Instead of "permissions" or "scopes," the Covenant names five **organs** — a visceral, pre-technical vocabulary organized around human harms, not API endpoints:

| Organ | What it means | Default |
|-------|--------------|---------|
| **Mouth** | Speak publicly, privately, legally, commercially | denied |
| **Purse** | Spend, sell, trade, transfer value | denied |
| **Seal** | Approve, sign, merge, deploy, publish, bind | denied |
| **Key** | Access secrets, credentials, personal data | denied |
| **Sword** | Cause bodily, legal, financial, reputational harm | denied |

All organs are denied by default. Mouth + Purse + Seal together requires "extraordinary constraint, logging, review, and revocation."

### Dual-track validation

Two parallel conformance checks, not one:

1. **Machine validation**: The manifest validates against the JSON Schema — it can verify that an organ has a non-empty `limits` array, but not whether those limits are meaningful.
2. **Human conformance**: The RFC 2119 spec requires human judgment — e.g., "Emergency authority MUST NOT be used for lost revenue, reputation management, ordinary customer escalation, growth, or optimization."

The schema catches structural errors; the human review catches intent errors. Neither alone is sufficient.

### Citation map with claim boundaries

Each source in `SOURCES.md` is annotated with explicit boundaries: "Used for: X. Not used for: Y." For example, citing Sanhedrin 65b supports the golem motif of creation and return to dust, but must not be used to claim a "complete theology of artificial agency." This prevents the usual problem of source lists being used to claim authority they don't provide.

### Tested revocation is mandatory

The `RETURN_TO_DUST_TEST.md` requires that revocation be *tested*, not just documented. Each of seven shutdown steps needs evidence: process stopping, queue pausing, egress blocking, credential revocation, log preservation. Both keeper and second reviewer must sign off. The kill switch must be retested every 30 days.

This is a notable departure from most security frameworks, which are satisfied with documentation.

### Shabbat as a control-surface test

Rest mode isn't an optional feature — it's a diagnostic. If the agent can't stop during Shabbat/quiet hours, the keeper doesn't actually control it. The test: "A rest-safe golem is mute, purseless, sealless, bounded, and boring." During rest, the agent may compute silently but may not speak in the keeper's name, spend, sign, deploy, publish, or summon humans.

### Emergency scoped by negative list

Emergency authority has an explicit forbidden list: lost revenue, reputation management, ordinary customer escalation, growth, optimization. The bucket-and-bell metaphor constrains emergency to the narrowest containment actions: ring the bell, carry the bucket, close the gate, revoke the key, wake the keeper. This prevents the classic failure mode where "emergency" becomes a backdoor for business continuity.

## Design Decisions

**Declaration over enforcement.** The Covenant is primarily a documentation standard, not a runtime enforcement library. A malicious implementer could fill out all templates and still build an unbounded agent. The bet is that forced declaration creates accountability even without technical enforcement. Runtime enforcement is listed as a needed review area for future versions.

**Jewish vocabulary, not generic spirituality.** The core metaphor (golem, Shabbat, Babel, Bezalel) is deeply particular. This gives the framework moral depth but risks alienating implementers who don't share that tradition. The Abrahamic lenses document attempts to broaden without flattening, but the vocabulary is unavoidably specific. The explicit position: "Do not say all Abrahamic traditions agree. Say instead they converge around certain anxieties."

**Moral seriousness over engineering convenience.** The Covenant deliberately uses weighty language — dignity, harm, dust, holy time — to signal that agent safety is a moral problem, not just a deployment checklist. This will feel overwrought to engineers who want a simple compliance standard. That discomfort is the point.

**What to declare, not what limits to set.** The Covenant requires declaration of limits but doesn't prescribe them. Two conforming agents could have wildly different safety postures. The assumption is that forced declaration leads to better decisions than prescribed limits that may not fit every deployment.

## Comparison Notes

Unlike [[Security and Sandboxing]] tools like [[yolo-cage]] and [[Stockyard]] which focus on technical isolation (Vagrant boxes, Firecracker VMs, egress proxies), the Golem Covenant operates at the *declaration and accountability* layer. It doesn't sandbox the agent — it demands the keeper declare what they're building and prove they can stop it.

Unlike [[agentsh]] which enforces security at the execution layer (FUSE+eBPF+seccomp, redirect-don't-deny), the Covenant is mostly pre-launch: fill out the templates, validate the manifest, test the kill switch, then deploy. The two approaches are complementary — agentsh could be the runtime enforcement for a Covenant-declared agent.

Unlike [[Elements of Agentic Systems Design]] which provides a taxonomy of what to think about, the Covenant provides a specific operational standard with machine-validatable conformance. It's prescriptive where Elements is descriptive.

The SOUL.md concept originally comes from the OpenClaw ecosystem's identity/continuity file — but the Covenant deliberately repurposes it from a personality file ("make the agent sound like me") to a covenant of restraint ("make the agent know its bounds"). This is a sharp departure from the [[Agent Identity]] pattern of identity-as-personality toward identity-as-constraint.

Unlike [[Memory Is a Mistake]] which argues most AI products shouldn't ship memory, the Covenant's `MEMORY.md` template takes a middle position: memory is permitted but must pass dignity-based tests ("Does this memory protect dignity? Would the human expect this to be remembered?").

Tags: #project #specification #agents #security #ethics

---

*Sources: [[raw/golem-covenant]]*
*Last updated: 2026-05-31*
