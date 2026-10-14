# Spec Kit's Enterprise Extension Model

Microsoft's follow-up on GitHub Spec Kit sketches how a shared spec-driven workflow survives contact with a real organization: four extension layers (presets, extensions, bundles, workflows), a two-tier governed catalog, and a layered constitution — with "stable core, governed building blocks" as the operating model.

---

## What the article argues

The problem statement is organizational, not technical: teams differ in domain, stack, and compliance controls, and "forking the toolkit for every team would fragment the experience and make upgrades costly." The proposed shape is a stable core with explicit extension points, split into four complementary layers:

- **Presets** — stackable, priority-ordered overrides for templates, commands, prompts, and terminology. A security preset strengthens threat-modeling prompts; a domain preset injects industry terminology and mandatory acceptance criteria. Customization without forking.
- **Extensions** — new capabilities beyond the built-in lifecycle: a security-review command, an accessibility validation gate, an internal work-management integration. Modular, versionable, independently maintainable.
- **Bundles** — presets plus extensions packaged as one versioned, installable unit with pinned versions and provenance tracking. For platform teams, the way to kill setup variability; for product teams, "one governed entry point."
- **Workflows** — orchestration of commands, prompts, scripts, and human checkpoints into repeatable processes (bug remediation, release readiness, incident follow-up), with conditional paths, loops, and pause/resume at review gates.

The governance story is a two-tier catalog: the **organization catalog** is reviewed, installable, and gets highest resolution priority; the **community catalog** is discovery-only by default, because "a listing confirms format and metadata — not a security review or endorsement of its code." That separation is called out explicitly as a security boundary, with the org catalog enforcing contribution standards, provenance, and prompt-injection safeguards.

The same layering is applied to the constitution itself: an organization baseline of non-negotiables, a domain/technology overlay, and narrowly scoped team exceptions — giving agents "clearer and more durable guardrails than ad hoc instructions repeated in individual prompts."

## Key quotes

> "A listing confirms format and metadata – not a security review or endorsement of its code."

The cleanest line in the piece, and the one doing real security work: it refuses the trust-transfer that plagues every package registry, where the existence of a package in a curated list is read as a vetting signal. The two-catalog split operationalizes the lesson.

> "The goal is not to remove human judgment – it is to make the expected sequence, artifacts, and checkpoints explicit."

Workflows are pitched as evidence-producing scaffolding rather than automation-for-its-own-sake. Whether that survives contact with efficiency pressure — the same pressure that hollowed out code review — is the open question.

> "Keep customization modular and the core untouched."

The upgradeability argument, and the real thesis: an enterprise SDD program lives or dies on whether local improvements become long-lived forks. Everything else (bundles, catalogs, pinned versions) is infrastructure for making the approved path the easiest path.

## Themes

#concept #tool #specifications #governance

## Critical take

This is a vendor's ecosystem-growth essay, and it reads like one: the extension taxonomy is neat to the point of being suspiciously complete, and there is no data anywhere — no adoption numbers, no failure stories from pilot orgs. The token-usage example is the closest thing to a concrete artifact, and even it is described in one paragraph.

That said, the architecture matches what's actually in the Spec Kit codebase (see [[Spec Kit]]): the extension/preset/workflow commands, the template resolution stack, and the catalog hardening in `_download_security.py` all predate or accompany this framing, so the four layers are retroactive naming for a real design rather than vaporware.

The genuinely interesting claim is the layered constitution. It takes the pattern every platform team eventually invents for lint rules and applies it to the *instructions* agents read: non-negotiables at the org level, overlays by domain, narrow exceptions per team. This is the right answer to the steering-file sprawl problem — but it quietly assumes organizations can actually write their "non-negotiable engineering expectations" down, which is exactly where most constitution documents rot.

The other load-bearing assumption: that "presets for policy, extensions for capabilities" stays a clean line. In practice a "security preset" that strengthens threat-modeling prompts is making an extension-shaped promise (a behavior change) through a preset-shaped mechanism (a template override), and the boundary will blur as soon as someone needs a preset to add a checkpoint.

## Relation to the wiki

- [[Spec Kit]] — this article is the product-level articulation of what that note found in the source tree: the extension commands, template resolution stack, workflow engine with gates, and catalog security work are all there in code; the essay adds only the enterprise vocabulary and the catalog governance story.
- [[When Spec-Driven Development Pays Off]] — that note found SDD's replicated benefit is attribution, not detection; the layered constitution and evidence-producing workflows here are attribution machinery raised to org scale, which strengthens the case that this is where SDD's value concentrates.
- [[The Clanker Constitution]] — Kenn's seven-clause behavioral baseline is a single-file, portable constitution; Spec Kit's layered version is the same idea institutionalized into org-baseline-plus-overlays, complicating the "one small file" ethos with an inheritance hierarchy.
- [[The Enterprise Gap from Vibe Coding]] — both pieces identify the same chasm between individual practice and organizational adoption; this one responds with extension points and governed catalogs rather than process mandate.

---
*Sources: [[raw/from-spec-first-to-enterprise-ready-extending-github-spec-kit]], [[summary/from-spec-first-to-enterprise-ready-extending-github-spec-kit]]*
*Last updated: 2026-10-14*
