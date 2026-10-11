---
url: https://kenforthewin.github.io/blog/posts/llm-nethack-ascension/
title: "GPT 6 Astra beat NetHack"
author: kenforthewin
date_fetched: 2026-10-11
date_published: 2026-09-21
topics:
  - agent-architecture
  - agent-memory-and-context
---

On September 21, 2026, GPT-6 Astra — run as a coding agent — became the first recorded LLM agent to ascend NetHack, winning the 3.6.7 game on the Hardfought public server as a dwarven Valkyrie after 37,140 turns across a run that began September 9. The author had spent January building a handcrafted Python harness around the NetHack Learning Environment and reached dungeon level 10; this time he wrote no code at all.

The interesting result is not the win itself but who built the scaffolding. Given a loose brief (play NetHack through a terminal, stream it to Twitch, spoilers allowed), the agent constructed its own harness: a tmux-pane terminal observation layer with compact annotated text, a checked-batching movement system whose preflight and post-step guards are concrete predicates over terminal state rather than a second model call, graph-search route and Sokoban helpers, and plain Markdown/JSON run journals and emergency references that let it resume after pauses and context compaction. When those tools failed — e.g. the status parser rejecting elemental-plane level labels late in the winning run — the agent patched, tested, and checkpointed its own code.

The author is careful about caveats: open-book access to spoilers, wiki, and NetHack source (not closed-book knowledge); no fixed benchmark budget; tools and instructions changed during play, so no attribution is possible between model, harness, memory, and compute; BALROG's 13.2% average NetHack progress for the same model measures something different. The claim of first-ever LLM ascension is supported by an inspectable evidence chain — local hash-linked audit recorder plus independent Hardfought server recordings — with an explicit statement of what is and is not proven. Cost: a $200/month Codex subscription plus ~$300 in extra credits.
