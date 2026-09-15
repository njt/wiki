---
url: https://github.com/dbmcco/agent-designer
date_fetched: 2026-09-15
---

# Agent Designer

A Pi skill that designs and deploys expert ensembles: casts of domain-expert
personas coordinated through a fixed, inherited methodology.

One skill, two protocols, one craft layer:

- **Panel protocol** — creates a standing panel (moderator, specialist cast,
  sequencing machinery, transcript validation) in the repository where the work
  happens. For deep, repeatable analysis that is re-entered and refined over time.
  A calibration gate runs first: the user confirms the panel's goal and roster
  before any persona is drafted.
- **Runtime protocol** — casts a small ensemble (two to five experts) live for an
  immediate question. Fast, session-scoped execution with transcripts and a
  promotable cast left behind on disk.

The two protocols share everything that makes the personas good: the persona
doctrine (the template rules and completeness gate) and the methodology kernel
(the orchestration canon). They differ in depth, machinery, and durability.

## Why the personas work

A persona is a retrieval key into the training distribution. Each template
element — a domain-common human name, a title and institutional signature that
looks like the bottom of an expert's email, a CV written in the field's actual
technical language, a formative experience, declared proclivities and blind
spots, a textured communication style, and an expert query vocabulary — is a
pointer into expert-text territory. A complete, coherent set of pointers lands
the model in that territory, where it continues convincingly. Generic pointers
keep it in generic-assistant space no matter how hard the prompt tries.

The doctrine enforces completeness. A draft that "reads like a job posting" or
"has no vocabulary of its own" fails the gate and is re-drafted before it can
join a roster.

## Status

Implemented — v0.3.0. Design specification:
[`docs/specs/2026-08-24-agent-designer-skill-design.md`](docs/specs/2026-08-24-agent-designer-skill-design.md).
Implementation plan:
[`docs/plans/2026-08-24-agent-designer-skill-implementation.md`](docs/plans/2026-08-24-agent-designer-skill-implementation.md).
Verify the bundle anytime with `scripts/verify-bundle.sh`.

## Layout

```
agent-designer/                  # repo root — the skill bundle
├── SKILL.md                     # dispatcher: purpose triage, protocol routing
├── doctrine/
│   └── persona.md               # template rules, completeness gate, failure signatures
├── methodologies/
│   ├── kernel.md                # the canon (versioned; inherited whole)
│   └── overlays/
│       ├── scenario-planning.md
│       ├── terrain-mapping.md
│       ├── root-cause.md
│       └── assignment-decomposition.md  # opt-in pilot: pre-roster assignment stage
├── reference/
│   ├── roster-heuristics.md     # casting rules: sizing, adversarial seat, support seats
│   └── contribution-review.md   # post-run review: per-seat contribution, counterfactual
├── scripts/
│   ├── validate-persona.sh      # structural persona pre-pass
│   ├── validate-synthesis.sh    # required Dissent section check (kernel rule 3)
│   └── diversity.py             # advisory divergence signal across seat outputs
├── templates/
│   ├── panel.json.md            # panel manifest shape
│   └── panel-readme.md          # panel README starter shape
└── docs/                        # build history: specs, plans, implementation notes
```

## Verification and review (v0.3.0)

Two additions, adapted from Noah Raford's
[agent-studio](https://github.com/nraford7/agent-studio) (MIT) and narrowed to
what fits this skill's doctrine:

- **Output-stage verification.** Every synthesis must carry a labeled Dissent
  section (kernel rule 3); `scripts/validate-synthesis.sh` fails the file
  without one, so consensus collapse cannot pass silently.
  `scripts/diversity.py` gives the facilitator an advisory divergence read
  across isolated seat outputs.
- **Post-run contribution review** (`reference/contribution-review.md`). After
  each run the facilitator records what each seat contributed, what the
  synthesis kept or discarded, and — when stakes justify — runs a single
  counterfactual synthesis without one seat. Causal-lift claims require the
  counterfactual; otherwise the record states traceable contribution only.

Deliberately not adopted from agent-studio: its deterministic evidence gate
(its corpus studies thin label-personas, not doctrine-built ones) and its
famous-exemplar casting. The persona doctrine is unchanged.

## Relationship to the existing panels

The eight hand-built panels in `experiments/ai-simulations` (shell-scenario,
terrain, vc, root-cause, pmf, synthyra, filmmaking, expert-panel) are the
provenance and reference material for this skill. The skill canonizes their
shared methodology — a single kernel with composable overlays — and encodes the
persona template rules they were built with. It never modifies them.

## Discovery

The skill is registered by symlinking:

```bash
ln -s /Users/braydon/projects/experiments/agent-designer ~/.agents/skills/agent-designer
```

The symlink points at this repo root, so it always resolves to the current
tagged release of the skill bundle.

## License

MIT — see [LICENSE](LICENSE).
