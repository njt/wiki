---
url: https://arxiv.org/html/2607.20300v1
title: "Don't Trust the Label: License Laundering in AI Supply Chains"
author: James Jewitt, Hao Li, Gopi Krishnan Rajbahadur, Bram Adams, Ahmed E. Hassan
date_fetched: 2026-07-25
date_published: 2026-07-22
---

An empirical study tracing 232,270 dataset→model→application chains across Hugging Face and GitHub to measure "license laundering" — the systematic stripping or replacement of legal rights as artifacts move through AI supply chains.

The authors define two forms: **Unknown laundering** (unlicensed artifacts acquiring license labels downstream) and **Category laundering** (obligation-bearing licenses like Copyleft or Sharealike being replaced, usually by Permissive ones).

Key finding: 62.3% of all supply chains pass through at least one artifact with no declared license. Among chains where all artifacts carry known licenses, 37.5% drop at least one license category — and obligation-bearing categories survive end-to-end at rates below 7%, while Permissive reaches 95.1%.

The problem concentrates in a small number of high-fan-out datasets: the top 10% of Unknown-licensed datasets account for nearly 90% of laundering transitions. When categories shift, the destination is almost always a Permissive label.

The paper includes the Books3 case study (pirated books packaged under an MIT label, later withdrawn after a Danish Rights Authority action, with the Bartz v. Anthropic settlement reaching $1.5B in 2026) as a concrete example of the legal risk.

Recommendations: practitioners should verify upstream licenses rather than trusting model labels; model publishers should declare training dataset licenses; rights holders should pair licenses with enforcement mechanisms; and platform builders should create cross-platform license tracing tools — the paper argues that manual upstream verification remains the only safeguard until such infrastructure exists.
