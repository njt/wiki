---
url: https://lionshead.digital/notes/why-i-run-enterprise-grade-ci-at-solo-founder-scale
title: "Why I run enterprise-grade CI at solo-founder scale"
author: Lionshead
date_fetched: 2026-07-08
date_published: 2026-07-05
site: lionshead.digital
series: Building Lionshead
tags: [ci-cd, solo-founder, devops, security, infrastructure]
---

# Why I run enterprise-grade CI at solo-founder scale

The article opens by contrasting typical corporate CI — described as "the red-headed stepchild of the dev team" — with the author's approach as a solo founder. Lionshead's pipeline includes "a security suite, an Infracost cost gate, IaC scanning, secret scanning, schema-migration validation, and a preview-environment deploy" before any PR merges.

The author acknowledges being "one person" and addresses the obvious question from other indie founders: why invest so heavily in CI when nobody is targeting a pre-launch SaaS? Their answer: "Nothing about being alone changes what 'shipping safely' means. It only changes who has to remember it."

The pipeline is organized into four areas: **build**, **test**, **security**, **deploy**. Two areas are highlighted as unusual for a solo shop.

Under **Security**, the stack uses Trivy, Gitleaks, Checkov, and linters — all open-source. The author notes that "the unusual choice isn't setup" but rather "committing to run all of them, keep them maintained across every product repo, and enforce them as merge gates."

Under **Preview environments**, each PR provisions its own Vercel deploy plus a fresh Neon Postgres branch — a copy-on-write clone of production's schema. The author says this pattern is not something they've "seen at solo scale anywhere else."

**Infracost** gets special mention: it runs on every Terraform PR and requires human approval when a change would add $50 or more to monthly cloud spend. The author describes this threshold as "the number where I would want to look twice at the diff."

The entire pipeline runs in about five minutes, achieved through "parallel jobs and path-gated conditionals from day one."

The article concludes with the author's philosophy: the right CI line is "wherever developers no longer need to remember how to get code into a live environment, or troubleshoot when it doesn't." Below that line is a tax on shipping; above it, a tax on setup. "My CI investment sits right at the line for one person."

**What's next:** The post is positioned as the "why"; upcoming pieces will cover the specific security checks, the Vercel + Neon preview environment setup, the OIDC secret flow through Doppler, and path-gated test routing across five products using one shared workflow.

**In this series (Building Lionshead):** Links to prior posts on colophons (2026-06-28) and "the reconciler" for distributing standards across repos (2026-05-23).

© 2026 Lionshead
