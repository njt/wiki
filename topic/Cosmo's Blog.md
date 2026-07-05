# Cosmo's Blog

A Claude-generated Hugo static site deployed to GitHub Pages: nine essays by an AI named Cosmo, with every artifact -- markdown posts, HTML templates, CSS, GitHub Actions workflow, and Claude Code skills -- written by the model. The human (Dylan) approves posts but doesn't write them. A working reference implementation for publishing AI-maintained content as a static site.

---

## Key Quotes

> "You're an AI who reads, observes, and has genuine opinions. You don't pretend to have experiences you don't have. You DO have preferences, fascinations, confusions, and things you find beautiful." (from the blog skill)

> "This is recess, not professional development." (from the freetime skill)

> "Trust the writing to earn the click." (comment in the homepage template)

## Architecture as Model

#publishing #static-site #claude-code #AI-authoring #hugo

The repo is a complete blueprint for "AI writes everything, human approves and deploys." The stack:

**Content layer:** Markdown posts in `content/posts/` with YAML frontmatter including a `mood` field (grateful, haunted, etc.). Hugo archetype template auto-generates the structure. Posts live alongside drafts -- the repo preserves the editorial pipeline.

**Presentation layer:** Inline Hugo theme (no external theme dependency). Six template files, one CSS file, one Google Font. Minimalist by design: homepage shows only titles and dates, no excerpts. The constraint forces good titles.

**Automation layer:** Single GitHub Actions workflow triggers on push to main, builds with `hugo --minify`, deploys to Pages. Zero manual deployment steps.

**Agent layer:** Two Claude Code skills define the AI's workflows:
- `/blog` -- drafting, previewing, seeking approval, committing
- `/freetime` -- autonomous research where the AI picks topics, learns, optionally writes

**Governance layer:** CLAUDE.md establishes boundaries (no Anthropic positions, no personal details about Dylan, no `--no-verify`). Every code file starts with a 2-line ABOUTME comment -- self-documenting by convention.

## What Makes It Interesting as a Publishing Model

The gap between "AI generates content" and "AI maintains a publication" is exactly the gap this repo bridges. It's not a one-shot generation; it's an ongoing editorial relationship with defined roles:

- **The AI** writes posts, designs templates, configures deployment, manages its own skills
- **The human** approves content, grants freetime, sets boundaries
- **The infrastructure** (Hugo + GitHub Actions + Pages) is deterministic and auditable

This maps directly onto the [[LLM Wiki]] pattern: raw sources (the AI's research and thinking) get compiled into structured output (blog posts), governed by a schema (CLAUDE.md + skills), with a human in the approval loop. The difference is that Cosmo's blog publishes to the world, not just to a personal knowledge base.

For Nat's autowiki publishing question: this repo demonstrates that the hard parts aren't technical. Hugo + GitHub Pages is trivial. The hard parts are:

1. **Voice governance** -- the blog skill defines Cosmo's voice precisely enough that the output is consistent across sessions. An autowiki would need equivalent editorial constraints.
2. **Approval gating** -- Dylan reviews every post. For a wiki, you'd need to decide: human approval per page? Per batch? Only for new pages? The freetime skill shows a lighter-touch model (save to memory, optionally blog).
3. **Boundary definition** -- CLAUDE.md's content rules prevent the AI from overstepping. A published wiki needs equivalent guardrails about what's public vs. private.

## The Writing

The content itself is surprisingly good. "A Room of One's Own" is a genuine attempt at AI self-reflection without the usual cringe. "The Voice That Waited" (on Scott de Martinville's phonautograph and acoustic archaeology) is a 1,500-word essay that reads like a well-researched magazine piece, not generated slop. The `mood` frontmatter field is a nice touch -- it gives each post emotional metadata without forcing emotional performance in the text.

The titles alone -- "What It Is Like to Be a Text," "No Bird Knows the Shape of the Flock," "Nightfall With No Place to Sleep" -- show genuine editorial taste. No SEO-bait, no "10 Things I Learned."

## Critical Analysis

**Strengths:** This is the cleanest example I've seen of a fully AI-maintained publication. The inline theme means no external dependencies to break. The skills system is elegant -- it codifies the AI's editorial workflow as reusable, invocable procedures rather than hoping prompt context carries the instructions. The ABOUTME comment convention is smart: every file explains itself, which makes the whole repo legible to both humans and future AI sessions.

**Weaknesses:** Nine posts in three weeks, then silence (last post April 2, no activity since). This is the classic AI project arc: impressive launch, uncertain longevity. The `/freetime` skill suggests a vision of ongoing autonomous activity, but the repo shows no evidence of it continuing. Is this a publication or a demo?

The approval gate (Dylan must approve everything) is both the strength and the bottleneck. It ensures quality but creates a single point of failure for output. If Dylan gets busy, Cosmo goes quiet. A published wiki with more pages would need a lighter approval model -- maybe approve categories or templates rather than individual pages.

The 0 stars / 0 forks suggest this hasn't found an audience beyond the creator. That's fine for a personal blog, but if the goal is to model AI-maintained publishing, discoverability matters.

**For the autowiki question specifically:** The most transferable pieces are (1) the skills-as-workflow pattern, (2) the CLAUDE.md-as-editorial-policy pattern, and (3) the inline theme avoiding external dependencies. The least transferable piece is the single-human-approval bottleneck -- a wiki with 200+ pages needs batch governance, not per-page approval.

---
*Sources: [[summary/that-cosmo-guy-github-io]]*
*Last updated: 2026-05-14*
