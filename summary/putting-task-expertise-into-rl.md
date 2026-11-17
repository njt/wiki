---
url: https://thinkingmachines.ai/news/putting-task-expertise-into-rl/
title: "Putting Task Expertise Into RL"
author: Yuxuan Zhu, Tengjun Jin, Yoojin Choi, Daniel Kang (Thinking Machines)
date_fetched: 2026-11-17
date_published: 2026-11
topics:
  - ai-research-and-models
  - guardrails-and-feedback-loops
---

Thinking Machines describes ReViSQL-K2.6, a Kimi-K2.6 fine-tune that reaches human-level accuracy on text-to-SQL (92.97% with self-consistency-16 on Arcwise-Plat-SQL) *without* scaffolding, at $0.56/task — more accurate than frontier models GPT-5.6 Sol Ultra and Claude Fable 5 at 12–15% of their cost. The two ingredients are both about quality of signal, not model capability.

First, data: the authors audited BIRD Train and found 61.1% of instances had at least one error — 52.1% of "golden SQL queries" were simply wrong. Since RLVR's scalar reward is the entire learning signal, mislabeled examples reverse the gradient, and they found no algorithmic tweak compensates. An LLM auditor (o3) was precise (90.6%) but caught only 24.5% of errors humans flagged, so cleaning required multi-stage expert review with conflict-resolution loops. The cleaned sets were released as BIRD-Platinum and Arcwise-Plat-SQL; training on verified data alone lifted the model to 88.55%, above frontier generalists.

Second, reward shaping: execution matching (same result set) is not semantic equivalence — 32.8% of positive rewards in a pilot went to non-equivalent queries that a wrong join key happened to pass on that database instance. They downweight such rewards using VeriEQL, a bounded SQL-equivalence verifier costing <0.1% of training compute. And outcome rewards can't see whether the model actually used the external knowledge provided with each question (24.2% of failures traced to ignoring it), so a rule-based process reward requires the model to emit a requirement block translating each knowledge entry into an explicit constraint, plus a verification block auditing the query against them.

The conclusion generalises their earlier financial-judgment work: when a domain task underperforms, the scaffold's embedded task knowledge belongs in the *training signal*, not in prompts around the model. Custom models trained on expert-curated data can beat frontier models at a fraction of the cost. Code, data and recipes are public (github.com/uiuc-kang-lab/ReViSQL; arXiv:2603.20004).
