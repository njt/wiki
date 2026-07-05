---
url: https://gist.github.com/db554463163daf8bfbb4cfafac89e1c7
title: "Building when it feels like there's nothing left to build"
author: Chip Huyen
date_fetched: 2026-07-04
date_published: 2026-05-18
---

Talk by Chip Huyen at The Pragmatic Summit (hosted by The Pragmatic Engineer), transcribed and summarized via ytx gist.

## Summary

The talk covers these major areas:

- **The Ghibli moment of software:** Just as AI can generate any describable visual style, it can replicate any describable software, removing the need for imagination.
- **Existing moats are evaporating:** Data isn't a moat — it's just expensive. DeepSeek and GPT replicability shows traditional defenses collapse.
- **Broken incentive structure:** When anyone can replicate what you build quickly, the question becomes: why build at all? The speaker received an email within a day of launching a side project from someone who'd recreated it with Claude Code.
- **Long-tail problem targeting:** AI excels at common problems but edge cases never disappear. The sweet spot is problems big enough to solve but too niche for big companies or frontier models.
- **Local human preference:** Cultural, geographical, and age-dependent nuances create problems requiring deep understanding. Example: Vietnam deploys voice bots before text chatbots because people are on motorbikes and dislike typing. Conversational latency norms differ radically (80ms US vs. 200-300ms in some Asian cultures).
- **Outdated human-to-human collaboration workflows:** A senior engineer reviews AI-generated code line-by-line for mentoring, but teammates don't read feedback because they didn't write the code. Feedback must shift from code to how one instructs AI.
- **World is not agent-ready:** Web search mimics human behavior (visiting pages, extracting snippets, re-querying), which is wildly inefficient for AI. Rate limits designed for human speed become bottlenecks. Infrastructure needs rebuilding for agentic interaction.
- **Reversibility as a guardrail:** Code and databases are reversible via commits/backups, but as agents submit forms on external sites or act in the physical world (cars), actions become irreversible — "where things get really, really scary."
- **Building for joy:** The speaker normalizes building for fun — creating apps as birthday gifts, like a tea-tracking app. The question shifts from "how to build" to "what to build" and "who imagines what doesn't exist yet."

## Key Quotes

1. "I just fluctuate between excitement and despair."
2. "Anyone can build anything I want. So what is the incentive structure for me to do anything?"
3. "Data is not a moat, it's just expensive."
4. "If you can describe a software, then AI can build it for you."
5. "Problems follow the long tail distributions" and "the edge cases will never go away."
6. He seeks a market "big enough to make some profit, but not too big that the sharks get in."
7. On the Postgres incident: "it was trying to create a new app and I already have another Postgres running."
8. "There are a lot of environments where the action cannot be reversible."
9. "The question is what you build. Who's going to build the things that don't exist yet?"
10. "I do enjoy building, and AI does make my life, for now, happier, except when I'm feeling very, very depressed."

Full transcript available via the ytx gist.
