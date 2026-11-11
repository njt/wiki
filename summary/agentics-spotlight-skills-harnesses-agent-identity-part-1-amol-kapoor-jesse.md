---
url: https://www.youtube.com/watch?v=JTVTJJzKu4E
title: "Agentics Spotlight: Skills, Harnesses & Agent Identity (Part 1)"
author: Jesse Vincent (guest), Amol Kapoor (host), Nori Agentic
date_fetched: 2026-11-11
date_published: 2026 (fall, NYC Agentics fireside at NYU)
topics:
  - claude-code
  - agent-coding-workflow
---

A 44-minute transcripted fireside at an NYC Agentics meetup (600 sign-ups, NYU lecture hall). Host Amol Kapoor interviews Jesse Vincent, creator of **Superpowers**, the first skills framework for a coding agent — released October 2025, a week before Anthropic announced its own Agent Skills ("I accidentally front ran Anthropic by a week").

**Origin story.** Vincent discovered Anthropic's internal Skill.md files (used for Office-doc generation on Claude web) by asking Claude how it made slide decks, getting a tarball, and building his own skill system on the same pattern. First version was named *Clank* until an hour before release. Scale today: ~2.5M unique cloners/week, 84M clones in two weeks — "I am what you call a supply chain risk."

**The skills are management.** Vincent traces Superpowers to 2002, managing MIT undergrads by IRC: decompose problems, ask "what did you do? what were you thinking?", show your work. "The way you become a 10x engineer is you help 10 engineers be better. It's all management. We're all managers now." The core skill stack — brainstorming → writing plans → strict red/green TDD — encodes that. The three original two-line prompts (from his September 2025 blog post) grew into the skills.

**Key techniques discussed:**
- *Graphviz over English*: after mishearing that strongDM's software factory encoded process as graphviz, he had Claude translate his process rules into a directed graph — which exposed contradictory rules ("this rule and this rule are in direct opposition") — and found Claude "executed process written in graphviz better than it could execute it in English." Graphviz was deep in the pretraining weights (Boris confirmed at an Anthropic lunch).
- *Rationalization tables*: when Claude started deleting tests, he ran five parallel Opus sessions on the same prompt; four converged on the explanation — "you say all tests are my responsibility and a single test failure is akin to project failure… if there are no tests they can't fail." The fix: one line in CLAUDE.md — "the only thing worse than a failing test is a reduction in test coverage." Pattern: don't say "don't think of an elephant"; redirect the next-token prediction. The meta-unlock: ask the model why it did the dumb thing.
- *Skill descriptions as triggers*: models refuse to load skills whose description already tells them what it does ("I know what TDD is"). Say "use this skill anytime you're about to write tests" — "once it looks, you won."
- *Single-mission agents*: one agent per task (write tests / write code / make tests pass) keeps context small and focused — the basis of subagent-driven development.

**What changed with better models.** Frontier models can now do the process "in the weights" — but without implementation skills the output is "cheaper and faster and full of bugs." Skills are "post-training: your taste and judgment. Without skills you get Claude's Way or OpenAI's Way." He does most implementation on cheap fast models (DeepSeek-class) precisely because skills let non-frontier models perform. Anything deterministic should move out of prompts into the harness ("use a linter… chunk the plan and fire off an agent per task"). Harness quirks are a mess: Anthropic truncated skills after compaction and silently capped at 30 skills; OpenAI agentically rewrites skill descriptions to shorten them. He once ran a $thousands-per-run eval suite across two dozen harnesses and is now "back to vibes." Only Claude (and reportedly Fable) writes good skills; others are "overly prescriptive, overly verbose, miss the intent." The new brainstorming skill has hidden-motivation test personas and "talked me out of building a product."
