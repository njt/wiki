---
url: https://blog.sqlauthority.com/2026/09/25/ai-generated-etl/
title: "AI Generated ETL Was the Thing I Was Most Sure Would Go Badly"
author: Pinal Dave
date_fetched: 2026-10-03
date_published: 2026-09-25
topics:
  - databases-and-data
  - specifications-as-the-product
---

Pinal Dave asked AI to rewrite a routine daily CSV-into-warehouse ETL job (staging table, merge, logging, error handling) that had been written by a departed employee in 2016. The generated pipeline ran clean on the first try — and that turned out to be the problem. Over six weeks, four failure modes surfaced that the old "ugly" job silently handled: the vendor's Monday file runs late (the mystery wait loop), the vendor resends corrected files (the hash lookup), a file can be structurally valid but contain all-zero amounts (a rule to page someone when the daily total drops below half the seven-day average), and an encoding change mangled accented names.

Dave's verdict is not that the AI failed but that his specification failed: none of those four problems were in what he asked for. The old job was "a pipeline plus nine years of bad mornings" — every odd check was added the day after something broke, and the reasoning lived only in code where it looked like mess. A good contractor would have asked what happens when the file doesn't show up; AI doesn't ask, so nobody remembered 2019.

His new workflow: ask the AI first to enumerate every way the pipeline could fail (including quiet failures where nothing errors but the data is wrong), take that list to the people who ran the old job to learn which ones have actually happened, and only then ask for the pipeline with each failure handled on purpose. His closing line is the thesis: "A brand new pipeline is not missing code, it is missing every bad morning the old one survived."
