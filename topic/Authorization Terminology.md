# Authorization Terminology

Andrea Chiarelli's six-axis taxonomy for authorization, generalizing his earlier "PBAC isn't a model" argument into a full disambiguation: MAC, DAC, RBAC, ABAC, ReBAC, ACL, and PBAC don't compete with each other — each answers one of six independent questions about how a request gets authorized, and the field's chronic confusion comes from treating them as a flat list of "authorization models."

---

## Key Quotes

> "Treating them as competing options is like comparing a recipe to a kitchen."

The article's thesis in a line. RBAC and ABAC describe *what data* a decision is based on (a role, an attribute); PBAC describes *where and how* the decision gets made (a centralized policy engine). One is the shape of the rule, the other is the machinery that evaluates it. They were never rivals.

> "A real authorization system's full description is a tuple across all six axes, not a single word."

The core reframe. Saying a system "is RBAC" answers only the model axis and says nothing about administration, policy shape, information source, decision location, or enforcement. The author's worked example — centralized administration (MAC), role-based model (RBAC), JSON policies, token-based information, a shared policy engine (PBAC), gateway enforcement — shows every label true at once because each answers a different question.

> "RBAC is a real, useful description of a model. PBAC is a real, useful description of an architecture. They're just not describing the same thing."

The conclusion that makes the piece land as taxonomy rather than pedantry. None of the individual labels are wrong; the error is using any one of them as an exhaustive description.

## Key Themes

#concept **Six axes, one per question.** Administration (who sets rules: MAC centralized, DAC decentralized, hybrid), model (what data drives the decision: ACL, RBAC, ABAC, ReBAC), policy (what shape the rule takes: code, JSON, declarative language, DB row), information (where decision data comes from: wired-in, token, lookup, environmental), decision (where evaluation happens — PBAC lives here), and enforcement (where the decision is acted on). Each axis has its own answers and its own tradeoffs.

#pattern **Model vs. architecture is the load-bearing distinction.** The most important cut is between what a rule reasons about (model) and where/how the decision is computed and enforced (architecture). Chiarelli borrows a ladder from a 2022 literature review — strategy above model above policy, with access-control model enforced by mechanism — and folds it into the same point: conflating layers is where the terminology breaks down.

#concept **PBAC relocated to the decision axis.** The article's sharpest move is placing PBAC on the decision axis, not the model axis. A PBAC engine can evaluate role-based, attribute-based, or relationship-based rules; it's a choice about *which court hears the case*, not a change to the law. This quietly refutes a decade of vendor "PBAC vs. ABAC" marketing.

## Critical Analysis

**The axes earn their keep by ending arguments, not just filing terms.** The test Chiarelli proposes — "if the terms stop competing once placed on the right axis, the axes are doing real work" — is a good one, and it mostly passes. MAC/DAC stop competing with RBAC once you see they answer the administration question. PBAC stops competing with everything once it's on the decision axis. That's a genuinely useful diagnostic to bring to any "RBAC vs. ABAC vs. ReBAC" debate.

**But the taxonomy is heuristic, not mathematical, and the author says so.** The honest admission that ACL and RBAC "can be described as special cases of ABAC" quietly undercuts the clean "each label answers one axis" story. If RBAC is just ABAC with a role-shaped attribute, then the model axis isn't four disjoint buckets — it's a spectrum with a socially-useful cut. The six axes are a *working* taxonomy, not a claim about exclusive categories, and the piece is better for conceding it than it would be for insisting otherwise.

**The legal metaphor does real work, with one limit.** The legislature/statute/court/police mapping is genuinely clarifying — "nobody would call a law court 'administrative law'" is the exact category error the software field keeps making. But the metaphor has an unexamined seam: a single legislature and police force are far more *centralized* than real authorization systems, which is precisely the administration axis the metaphor is meant to illuminate. It clarifies more than it obscures, but it's an analogy, not a proof.

**What it doesn't touch — and where this wiki lives.** The piece is about classical, human-shape software authorization, and it stops before agents. It never asks what happens when the "subject" is an AI agent acting on behalf of multiple humans, or when the rules have to be evaluated per-action at machine speed. Those are exactly the open problems in [[The Agent Access Model]] (multiplayer access control is unsolved) and [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] (the identity-ambiguity diagnostic). Chiarelli's taxonomy would say those problems live on the administration and information axes, where agent identity is genuinely unsettled — so the taxonomy frames the frontier even where the article doesn't walk to it.

**A practical use the article implies but doesn't spell out.** On the policy axis, Chiarelli lists declarative languages (XACML, Rego, Cedar) alongside code and JSON as interchangeable shapes. But [[Celly — Native .NET CEL Implementation]] shows why the shape matters enormously in practice: CEL's termination guarantee and type checker are what make it safe to let an agent *generate* policy. Two systems can share a model and differ completely in maintainability and who's allowed to change rules — which is the axis's whole point, and the agent case makes the stakes concrete.

---

*Sources: [[raw/authorization-terminology-is-a-mess-lets-fix-it]], [[summary/authorization-terminology-is-a-mess-lets-fix-it]]*
*Last updated: 2026-09-04*
