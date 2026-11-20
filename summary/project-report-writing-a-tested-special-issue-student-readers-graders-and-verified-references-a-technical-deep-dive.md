---
url: https://parc.yolo.scapegoat.dev/note/projects/2026/11/19/project-report-writing-a-tested-special-issue-student-readers-graders-and-verified-references-a-technical-deep-dive
title: "Writing a Tested Special Issue: Student Readers, Graders and Verified References"
author: Manuel (wesen), parc.yolo.scapegoat.dev
date_fetched: 2026-11-20
date_published: 2026-11-19
topics:
  - guardrails-and-feedback-loops
  - agent-orchestration
---

A ten-article teaching series, "Compiling on a Handheld", was researched, measured, written, tested on model readers and published in one working day. This report documents the process, not the compilers: the roles, the pipeline, the costs, the failures, and which checks did the work.

The central claim is that the document improved most at points where a claim was checked against something outside the author — a source text, a measurement, the repository, or a reader who had only the page. Five independent checks were arranged: three verifier subagents confirmed 107 references before any drafting (103 confirmed, 4 corrected); scripts recomputed every quantitative claim from the repository, each instrument validated against a known value first; three weak "student reader" sessions (deepseek-4.1-flash) read the 800-word expert brief and proposed a teaching order that replaced the author's topic-based plan; 60 reader sessions per round answered quiz questions before and after reading in fresh directories; and five stronger grader subagents per round scored answers and confirmed each reader complaint against the text (68 defects after round 1, 42 after round 2).

The most transferable findings concern measurement design. Post-reading quiz scores were useless (11.9–12.0 of 12 — a reader with the text open finds anything in it), while pre-reading scores exposed that 24 of 60 questions needed no article; round 2 rewrote questions so a correct answer must require something only in the document. Reader feedback was never read raw — it passed through graders instructed to check each complaint against the article, quote it, and flag ambiguous answer keys. Every stage wrote its full output to disk while returning only summaries, keeping the author's context small and leaving a complete audit record. The author is explicit about limits: model readers do not establish that people learn from the text; they locate defects, which a human editor judges.
