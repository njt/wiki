---
url: https://ai.engineer/code/2026/talks/justin-mccarthy
title: "Stop Building Software Factories (Justin McCarthy, ai.engineer 2026)"
author: Justin McCarthy
date_fetched: 2026-11-26
date_published: 2026
topics:
  - agent-coding-workflow
  - ai-product-and-business
---

Justin McCarthy (founder/CEO of Diffusion) opens his ai.engineer 2026 talk by half-jokingly telling the audience to stop building software factories — then draws the line that makes the title coherent: factories are right for **brownfield** (existing products with proven demand) and wrong for **greenfield discovery**, where you don't yet know what you want and industrialising it just ships the wrong thing faster.

His brownfield recipe: identify your "promises" (features, price, security, reliability — anything you'd hate to regress), encode them as verification loops that feed signal back into the input like an amplifier's feedback leg (cut it and you amplify noise), and run the thing continuously — agent-maintained loops flatten the old daily/weekly commit seasonality into a 24/7 straight line. A key observed dynamic: factories **simplify over time** — skills get shorter, harnesses get more general — largely because model improvements do the Raptor-1→Raptor-2 simplification for you for free. Barely get it working and a smoother version arrives next model release.

The instruction to give a brownfield factory is "same but faster, cheaper, safer" — never "better," which requires human taste and belongs to discovery. Building it takes, he claims, about a week.

The second half escalates: the brownfield factory can't build a moat, and it's producing a **glut of working software customers didn't ask for** — they wanted margin, customers, differentiation. So he proposes the "**demand factory**": agents asking better market questions, synthesising and re-ranking customer signal overnight, scheduling price-elasticity experiments, and lowering the latency of saying yes. Fill your "brand envelope" (Spindrift can ship caffeine, maybe energy drinks — not motor oil), then more SKUs, more product lines, and eventually acquire your competitor.
