---
url: https://idpro.org/authorization-terminology-is-a-mess-lets-fix-it/
title: Authorization Terminology Is a Mess — Let's Fix It
author: Andrea Chiarelli
date_fetched: 2026-09-04
---

Andrea Chiarelli argues that authorization's terminology problem isn't a lack of vocabulary — it's too much vocabulary, accumulated across research, vendors, and standards, all funneled into one flat question: "what model is this?" MAC, DAC, RBAC, ABAC, ReBAC, ACL, and PBAC get compared as if they were competing answers to the same question, when most of them answer different ones.

He generalizes his earlier PBAC argument (PBAC is architecture, not model) into a six-axis taxonomy. Each axis has its own independent answers, and every familiar term lands on exactly one of them: **administration** (centralized = MAC, decentralized = DAC, hybrid), **model** (identity-based ACL, role-based RBAC, attribute-based ABAC, relationship-based ReBAC), **policy** (hardcoded conditionals, structured JSON/YAML, declarative languages like XACML/Rego/Cedar, database rows), **information** (wired-in, token-based, looked up, environmental), **decision** (inline code, a library, or a centralized policy engine — which is where PBAC lives), and **enforcement** (inline, middleware, or distributed at a gateway/sidecar). A real system's full description is a tuple across all six axes, not a single word.

The practical payoff: choose the model based on what your access rules naturally depend on (roles, attributes, relationships), and separately choose the architecture (where decisions get made and enforced) based on operational constraints — how many services share policy, latency tolerance, who audits or changes rules. Bundling both into one choice ("we're doing RBAC" or "we're doing PBAC") hides half of what needs deciding.

A legal metaphor runs through the whole piece: a legislature drafts rules, the rules get written as statutes, courts weigh statutes against the facts, and police carry out the verdict. The metaphor is useful precisely because it breaks down in the same places software terminology does — nobody calls a court "administrative law," and nobody should call a system "PBAC" as if that exhausted it.
