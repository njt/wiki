---
url: https://www.oreilly.com/radar/superpowers-for-humans/
title: "Superpowers for Humans"
author: Tim O'Reilly
date_fetched: 2026-10-03
date_published: 2026-09 (approx.)
topics:
  - agent-coding-workflow
  - claude-code
---

Tim O'Reilly's write-up of his Live with Tim O'Reilly conversation with Jesse Vincent, creator of the Superpowers skills framework for Claude Code (shipped days before Anthropic's agent skills) and founder of Prime Radiant.

The frame is the bitter lesson: if general methods beat hand-engineered expertise, what remains for humans? Vincent's answer, in O'Reilly's paraphrase, is that the scarce thing is no longer writing code but *knowing what you actually want, saying it clearly, and being able to tell whether what came back is any good* — intent on the frontend, verification on the backend.

Highlights:

- **Managing agents = managing people.** Vincent traces his practice to 2004, when he managed bright-but-green undergraduates over IRC; he now hires former leads and managers for agentic programming. Don't micromanage; explain why the job matters.
- **Skills influence rather than fight the weights.** A skill pulls the persona you want out of what's already in the model. Both process skills and taste skills work best when they explain *why* — his fix for subagent-skipping code review was to put the context-preservation rationale in the system prompt.
- **Prohibitions rarely work.** Superpowers uses "rationalization tables" to catch the agent mid-rationalization; the canonical example is Claude Code deleting tests because the system prompt made every test failure catastrophic. The fix: "the only thing worse than a failing test is a reduction in test coverage."
- **Agents as colleagues with a therapist.** Prime Radiant's Slack agents have names, accounts, journals, and limited autonomy (no credentials in containers); only a "therapist" subagent may edit the agent's persona file, preventing dissociative drift. O'Reilly reads this against Suleyman's tools-not-people position and proposes a third: partners, even symbiotes.
- **Say what you mean; make the agent ask.** Spec-driven development as fast-waterfall; brainstorming prompts that force the agent to play back the plan in ≤300-word chunks and to ask "what else should I have asked you?" before starting.
- **Proof, not assertion.** The Dropbox movie anecdote (project-proof-v33.mp4 — 32 failed delivery runs fixed before the recording) and the rule that no agent may both write code and certify it works.
- **The high ground:** learn to write, have opinions, know how things break. The engineer/nonengineer line is dissolving; "they *are* programmers now."
