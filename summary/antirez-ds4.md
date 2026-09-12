---
url: https://antirez.com/news/165
title: "A few words on DS4"
author: antirez (Salvatore Sanfilippo)
date_fetched: 2026-05-16
date_published: 2026-05-15
publication: antirez.com
topics:
  - local-and-open-source-inference
---

antirez reflects on the rapid popularity of DwarfStar 4 (DS4), hosted on GitHub under antirez/ds4, a single-model integration focused local AI experience.

He suggests there was pent-up demand driven by several converging factors: a quasi-frontier model that is both large and fast enough to transform local inference; its excellent performance with an asymmetric 2/8 bit quantization recipe requiring only 96–128GB of RAM; and the accumulated experience of the local AI community, which could be leveraged more readily thanks to GPT 5.5. Without GPT 5.5, he notes, "you can't build DS4 in one week."

The past week was "funny and also tiring," averaging 14-hour workdays, compared to his normal 4–6 hours per day since the early Redis era — though he notes the first months of Redis were similarly intense.

On the project's future, antirez clarifies that DS4 is not tied exclusively to DeepSeek v4 Flash; the underlying model can change over time. The space will be occupied by "the best current open weights model" that runs practically fast on high-end Macs or GPU-in-a-box setups like the DGX Spark. He speculates the next contender will be DeepSeek v4 Flash itself via new checkpoints, possibly "a version specifically tuned for coding," along with other expert-variant models (ds4-coding, ds4-legal, ds4-medical), arguing that for local inference you simply "load what you need depending on the question."

This marks the first time since he began experimenting with local inference that he's relied on a local model for serious tasks he would normally ask Claude or GPT. He calls this "really a big thing." He also notes that with vector steering, the LLM can be used "with more freedom." He praises DeepSeek v4 Flash as impressive, describing the local model experience as A, the frontier online model as B, and DS4 as "a lot more B than A."

Looking ahead after the chaotic launch, he hopes the project will focus on quality benchmarks, possibly adding a coding agent as part of the project, a dedicated hardware setup at home for CI testing to ensure long-term quality, more ports, and notably "distributed inference (both serial and parallel)."

He closes by thanking the community for the support, stating: "AI is too critical to be just a provided service."
