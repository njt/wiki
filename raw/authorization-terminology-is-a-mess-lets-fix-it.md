---
url: https://idpro.org/authorization-terminology-is-a-mess-lets-fix-it/
date_fetched: 2026-09-04
---

**by Andrea Chiarelli**

In Is Policy-Based Access Control (PBAC) an Authorization Model?, I argued that PBAC gets compared to Role-Based Access Control (RBAC) and Attribute-Based Access Control (ABAC) as if it belonged in the same category, when it actually answers a different question.

RBAC and ABAC describe *what data* a decision is based on: a role, an attribute. PBAC describes *how* that decision gets made: through a centralized policy engine instead of logic wired into the application. One is about the shape of the rule. The other is about where the rule lives and who evaluates it.

In my opinion, treating them as competing options is like comparing a recipe to a kitchen.

That distinction attempted to solve one specific confusion, but it left a bigger one untouched: PBAC isn’t the only term that gets misfiled this way.

Mandatory Access Control (MAC) and Discretionary Access Control (DAC) get compared to RBAC and ABAC too, even though they describe who administers the rules, not what the rules look like.

Access Control List (ACL) and Relationship-Based Access Control (ReBAC) show up in the same breath as “authorization models,” but one is arguably a special case of the other, depending on which paper you read.

The terminology around authorization has accumulated for decades across access control research, identity vendors, and standards bodies, and a lot of it answers different questions while using the language of a single one: “*what model is this?*“

This piece generalizes the model-versus-architecture distinction into a full taxonomy. Instead of asking “*which authorization model is this system using?*“, I want to ask five narrower questions about any authorization system, answer each one independently, and only then see where familiar labels like RBAC, ABAC, MAC, and PBAC actually land. If the terms stop competing once they’re placed on the right axis, that’s a sign the axes are doing real work.

## The Core Problem of Authorization

Authorization answers one question: **can this subject perform this action on this object?** A user reading a file, a service calling an API, a process writing to a database. Every authorization decision reduces to that same triple: subject, action, object.

In the simplest possible system, there’s no decision to make because there’s no separation between the party asking and the party deciding. Picture your own diary, kept in a drawer in your own room. You want to read it, so you open the drawer and read it. Nobody has to grant you permission, because nobody else is involved: wanting and getting are the same act.

Real systems rarely stay this simple, because the thing being accessed usually belongs to more than one interested party. Put that same diary in a house with housemates, or move it into a company’s filing cabinet, and the picture changes immediately. Not everyone who wants to open the drawer should be allowed to, and somebody has to decide, case by case, who does and who doesn’t.

A resource server does exactly that job for software. A multi-tenant SaaS application, an internal API, a cloud storage bucket accessed by dozens of services: all of them need something that looks at a request and decides whether to honor it. That “something” is what the rest of this piece is about, and it turns out to have more moving parts than a single word like “RBAC” or “PBAC” can capture.

## How an Authorization Request Gets Processed

Follow a single authorization request through a resource server and a consistent set of stages emerges, regardless of the specific technology involved: a request arrives, the server gathers context about the user and the resource, a policy evaluates that context against a rule. The evaluation produces an allow or deny decision, and the server enforces that decision by letting the request through or rejecting it.

The same five stages appear in essentially every authorization system:

- **Someone defines the rules.**A developer hardcodes a check; a security team writes a policy; a resource owner sets sharing permissions on their own file.
- **The rules take some concrete shape.**A conditional in application code; a JSON document; an XACML policy; a row in a spreadsheet someone exports into a database table.
- **Some data has to feed the decision.**The user’s role; the department listed in a database; the time of day; the relationship between the requesting user and the resource’s owner.
- **A decision gets computed.**The rule and the data get evaluated together, producing allow or deny.
- **The decision gets enforced.**Something has to actually act on the result: block the request or redirect it.

These five stages map onto a vocabulary that’s been around in access control architecture for a while:

- Policy Administration Point (who defines the rules)
- Policy Information Point (what data feeds the decision)
- Policy Decision Point (where the decision gets computed)
- Policy Enforcement Point (where it gets enforced)

Brossard’s overview of ABAC architecture lays out this PAP/PIP/PDP/PEP breakdown in more detail, and it’s worth reading if these terms are new.

There’s a related, more academic framing worth pulling in here, from a 2022 systematic literature review of authorization and access control research by Mohamed, Auer, Hofer, and Küng. The paper draws a ladder with four levels: an **authorization strategy** (discretionary, mandatory, or hybrid) sits above an **authorization model** (the subjects, objects, and other components a system reasons about), which sits above an **authorization policy** (the concrete rule instance). In parallel, an **access control model** (the logic for deciding allow/deny) is enforced by an **access control mechanism** (the actual running software). It’s a rigorous way of saying the same thing this piece is arguing: strategy, model, policy, and mechanism are separate layers, and conflating them is where the terminology breaks down.

None of these five stages, by themselves, tells you whether a system “is RBAC” or “is PBAC.” Each stage is actually a separate question, with its own independent set of answers. That’s the seed of the classification below.

## A Consistent Classification

Six axes cover the five stages above (Authorization Policy gets split from Authorization Model, since a model can be instantiated by more than one kind of concrete policy artifact). Each axis has a small number of common answers, and every familiar term (ACL, RBAC, ABAC, ReBAC, MAC, DAC, PBAC) is really an answer to one specific axis, not a label for the whole system.

A useful way to keep the six axes distinct is a metaphor borrowed from outside software entirely: the machinery of law. A legislature drafts rules, the rules get written down as statutes, courts weigh those statutes against the facts of a case, and police carry out the verdict. Swap “legal system” for “authorization system” and the same six jobs reappear below, one at a time.

### Authorization Administration: who sets the rules?

Before a rule can be evaluated, someone has to have the authority to write it in the first place. That authority can be concentrated in one place or spread across many, and a system’s answer here is independent of what the rules actually say. Authorization administration can be:

- **Centralized**: a security team or administrator defines rules for everyone. This is the pattern behind MAC (Mandatory Access Control): access is determined by a central authority, not by the resource owner.
- **Decentralized**: the owner of a resource decides who else can access it. This is DAC (Discretionary Access Control): the classic “- *share this file with these people*” pattern.
- **Hybrid**: some rules come from a central authority, others from individual resource owners, layered together.

Mohamed et al’s literature review treats strategy this way too, and their comparative analysis is instructive: when they classified access control models against this axis, RBAC, ABAC, and ReBAC families all landed in the hybrid category, not cleanly in DAC or MAC. That’s a useful data point on its own. It confirms administration strategy is genuinely independent of authorization model: a role-based system can be centrally administered, owner-administered, or both, and it’s still RBAC either way.

The legal system runs on exactly this split. A parliament, or a king under older systems, legislates for an entire population: centralized administration. Two neighbors agreeing on where a fence line runs, or a homeowner deciding who gets a key, is decentralized administration, closer to how contract law and property law actually work. Most real legal systems mix both, the same way most authorization systems do.

### Authorization Model: what data type drives the decision?

Once the authority to write a rule is settled, the next question is what the rule actually reasons about: the specific kind of information a decision checks each time it runs. This is the axis most people mean when they say “authorization model,” and it’s the one my earlier PBAC article focused on. Here are a few common authorization models:

- **Identity-based (ACL)**: the decision checks whether the specific subject appears on a list attached to the object.
- **Role-based (RBAC)**: the decision checks whether the subject holds a role that’s been granted the requested permission.
- **Attribute-based (ABAC)**: the decision checks attributes of the subject, object, action, or environment against a rule (department = “finance”, clearance ≥ “secret”, time between 9 AM and 5 PM).
- **Relationship-based (ReBAC)**: the decision checks the relationship between subject and object, often by traversing a graph (is this user a member of the team that owns this document?).

It’s worth being honest about the fuzziness here. Depending on which academic source you read, ACL and RBAC can be described as special cases of ABAC, where the attribute in question happens to be identity or role membership. These four categories are the most useful cut for a working taxonomy, not a claim that the categories are mutually exclusive at the mathematical level.

The law makes the same kind of distinction between what a rule is actually about. Constitutional and administrative law usually define who may act by office, “*only the president may sign this treaty*“, which is role-based. Regulatory law more often reaches for attributes, “*any business with revenue over ten million dollars must file this report*“. Family and contract law hinge on a relationship: spouse, parent, business partner. And a law or injunction naming one specific person or company is the legal equivalent of an ACL. Different areas of law reach for different criteria for the same reason different authorization systems do: each criterion fits a different kind of decision.

### Authorization Policy: what shape does the rule take?

Once a model is chosen, it still has to be written down somewhere as a concrete artifact:

- Hardcoded conditionals in application code.
- A structured document (JSON, YAML) loaded and interpreted at runtime.
- A declarative policy language purpose-built for authorization, like XACML, Rego (Open Policy Agent), or Cedar.
- A row in a database table.

This axis matters because two systems can share the same authorization model (both are RBAC) while differing completely in maintainability, auditability, and who’s allowed to change the rules, purely because of how the policy is expressed.

In the legal metaphor, this is simply the law itself: the actual text of the statute or code, independent of who passed it and independent of which area of law it belongs to.

### Authorization Information: where does the decision-relevant data come from?

A rule is only as good as the data it’s evaluated against, and that data doesn’t all arrive the same way. It’s worth separating what the rule depends on from how the system actually gets its hands on it:

- **Wired in**: the data is already available in the application’s normal request flow, no extra lookup required.
- **Token-based**: the data arrives as claims in a JWT or similar credential, populated by the identity provider at issuance time.
- **Looked up**: the application queries a database, directory, or external service at decision time.
- **Environmental**: the data describes the context of the request itself rather than the subject or object: time, location, device posture, network.

This is a place where even careful academic treatments blur a real distinction. Mohamed et al’s framework folds environmental attributes into the authorization model’s component list, alongside subjects and objects, rather than treating “*where does this data physically come from at runtime*” as its own architectural concern. I think that’s a small version of the same mistake the PBAC article calls out: knowing that a decision depends on “time of day” is a modeling question but knowing that the request has to make a network round-trip to a directory service to get it is an architecture question, with real consequences for latency and failure modes.

A courtroom depends on the same kind of input: facts and events. Testimony, documents, timestamps, prior records. The same law applied to a different set of facts produces a different verdict, exactly the way the same policy applied to different token claims or different database lookups produces a different authorization decision. And, just as in an authorization system, where those facts come from (a witness on the stand, a document requested via subpoena, or an expert report) is its own separate concern from what the law says.

### Authorization Decision: how and where is the decision computed?

Once a rule and its data are both available, something has to actually run the evaluation. That “something” can live in a few different places, and where it lives is a question about architecture, not about the rule itself:

- **Wired into application code**: an if statement or a framework-native permission check inline with the business logic.
- **A dedicated library or module**: decision logic factored out but still running inside the application process.
- **A centralized policy engine**: a separate service that receives context and returns a decision, often shared across many applications.

**PBAC lives here**, not on the “authorization model” axis. Centralizing the decision point into a policy engine is an architectural choice about *where* evaluation happens, and it’s compatible with any of the authorization models: a PBAC engine can evaluate role-based rules, attribute-based rules, or a mix. That’s exactly why PBAC isn’t a peer of RBAC and ABAC. It’s an answer to a different question than the one RBAC and ABAC answer.

This is where the courtroom metaphor lands squarely: a court, and specifically a judge, is the decision engine. The law and the facts go in, a verdict comes out. And a legal system’s habit of routing different kinds of cases to different, specialized courts (traffic court, family court, a supreme court) without changing what the underlying law says is a good picture of what PBAC actually is. It’s a choice about which court hears the case, not a change to the law itself.

### Authorization Enforcement: how and where is the decision enforced?

A decision only matters if something acts on it, and that action doesn’t have to happen in the same place the decision was made. Authorization enforcement can live in:

- **Wired into application code**: the same code path that made the decision also acts on it.
- **Dedicated middleware**: a framework-level component intercepts the request based on the decision.
- **Distributed enforcement**: a gateway, sidecar, or proxy enforces decisions at the network edge, independent of the application itself.

Like the authorization decision axis, this is architecture, not model. A system enforcing ABAC decisions through an API gateway and a system enforcing ABAC decisions through inline middleware are both, unambiguously, ABAC. They just made different choices about where enforcement lives, usually driven by concerns like operational scale or the number of services that need to share one policy.

This is the police, in the metaphor: the verdict gets carried out regardless of whether that happens inside the courtroom or on a street corner miles away. The court decides; the police act. It’s the same separation between authorization decision and enforcement in an authorization system.

### Putting it together

**A real authorization system’s full description is a tuple across all six axes, not a single word**. “RBAC” says something about the authorization model axis and nothing about the other five axes. An authorization system can have:

- Centralized administration (MAC)
- Role-based model (RBAC)
- JSON policy documents
- Token-based information
- A shared policy engine (PBAC architecture)
- Gateway-level enforcement.

That’s a coherent, common real-world setup, and every label in it is accurate simultaneously, because each one answers a different question. The confusion in most authorization discussions comes from picking one label per system and treating it as exhaustive, when it’s really just the answer to whichever axis happened to come up first in the conversation.

The legal metaphor running through this section is useful precisely because it breaks down in the same places the software terminology does. Nobody would call a law court an “administrative law” or claim the existence of a “witness-based law” because a judge renders a verdict considering the deposition of a witness. But the six jobs underneath it (who legislates, what kind of law applies, what the law actually says, what facts it’s applied to, who judges, who enforces) are as separable in a courtroom as they are in a policy engine, and confusing them causes the same kind of category error either way.

## Conclusion

The terminology problem in authorization isn’t that the field lacks vocabulary. If anything, it has too much: MAC, DAC, RBAC, ABAC, ReBAC, ACL, PBAC, and more, each coined by a different paper, standard, or vendor, at a different layer of the same request lifecycle. The problem is a habit of treating all of it as one flat list of “authorization models” to choose between, when most of these terms don’t compete with each other at all.

Separating the questions helps in a very practical way when you’re actually choosing an approach for a real system: pick the authorization model based on what your access rules naturally depend on (roles, attributes, relationships). Separately, pick the architecture (where decisions get made and enforced) based on your operational constraints (how many services need to share policy, how much latency you can tolerate, who needs to audit or change rules). Those are two separate decisions with two separate sets of tradeoffs and bundling them into a single choice (“*we’re doing PBAC*” or “*we’re doing RBAC*“) tends to hide half of what actually needs deciding.

None of this makes the individual labels wrong. RBAC is a real, useful description of a model. PBAC is a real, useful description of an architecture. They’re just not describing the same thing, and a consistent classification is what lets you say precisely which thing each one is.

## References

- Is Policy-Based Access Control (PBAC) an Authorization Model?
- A Taxonomy of Modern Authorization Models
- The State of the Union of Authorization
- A systematic literature review for authorization and access control: definitions, strategies and models. *International Journal of Web Information Systems*, 18(2/3), 156-180. https://doi.org/10.1108/IJWIS-04-2022-0077

Andrea Chiarelli is a Principal Developer Advocate at Auth0 (Okta). He has extensive experience in software development, holding various technical roles. In recent years, he has focused on Identity, developing a passion for the core concepts, which he aims to disseminate to developer communities.
