---
url: https://www.infoq.com/articles/rightsizing-platform-engineering/
title: "Rightsizing Platform Engineering"
author: InfoQ (Wehkamp platform engineering team)
date_fetched: 2026-09-20
topics:
  - software-engineering-craft
---

A field report from Wehkamp, a large Dutch e-commerce company, on a decade of building — and repeatedly shrinking — their internal developer platform, expanding on a KubeCon EU 2026 talk. The story begins with a leadership mandate to move from quarterly to weekly releases, which pushed ownership onto product teams and promptly generated a new kind of toil: every team repeating the same infrastructure R&D.

The article's central claim is that platform engineering succeeds by rightsizing, not by comprehensiveness. Wehkamp tried and abandoned the two fashionable extremes — a fully self-serve portal with complex RBAC, and multiple attempts to adopt Backstage — because both shifted maintenance burden without reducing cognitive load. What worked was narrower: opinionated golden paths over a small set of application-delivery primitives (repo, build, package, run, monitor, dependencies), a chatbot-to-Terraform self-service flow, and an "apply or explain" rule where divergence from the golden path requires escalating proof of need.

Two governance ideas give the piece its structure: splitting platform resources into consume-only versus multiparty, and mapping capabilities on a spectrum from platform-provided primitives to community-contributed building blocks. The conclusion is deliberately deflationary — the right platform is the simplest one that solves your organisation's actual bottlenecks, measured by delivery improvement and reduced cognitive load, not feature count.
