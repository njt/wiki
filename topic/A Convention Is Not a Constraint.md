# A Convention Is Not a Constraint

Simple Thread's field report on the quiet trade multi-tenant systems make when they consolidate: an isolation guarantee that used to live in the deployment topology gets relocated into code, where it must be rebuilt as something the system enforces — not something the team merely agrees to remember. The distinction that carries the whole post: a convention holds only while everyone follows it; a constraint holds whether anyone remembers it or not.

---

## Key Quotes

> "Every multi-tenant application makes a promise: Tenant A will never see Tenant B's data. And in every multi-tenant application, *something* is responsible for keeping that promise. My question is one I think every team should be able to answer quickly and out loud: **what is that something?** Point at it."

The opening diagnostic, and the article's real practical contribution sits here rather than in the code: *before* changing the architecture, name the thing doing the enforcing. If you can't point at it, the promise is being kept by accident — and accidents are what future changes quietly remove.

> "The enforcer was the topology, and a topology leaves no trace inside the code: you will not find a line you can point to and say 'this is what keeps tenants apart,' because there wasn't one."

This is the sharpest framing in the post. Isolation enforced by the shape of the deployment is real but invisible — it asks nothing of the codebase, which means the codebase records nothing to lose. Exactly the trap [[Multi-Tenancy Isn't About Databases]] names with "you are providing deployment isolation, you are not necessarily providing schema isolation."

> "a convention is only as strong as everyone's discipline in following it; a constraint is enforced whether anyone remembers it or not. So be clear about which of your guarantees is which."

The thesis, stated once as a definition and again as the article's close. It is the database-world twin of the agent-governance insight in [[Guardrails and Feedback Loops]] and [[Structural Backpressure Beats Smarter Agents]]: "English is simply the wrong medium in which to enforce it."

> "'We always set the correct schema, and always tear it down' is a convention. 'This connection physically cannot touch more than one tenant, and touches none without explicit context' is a constraint. The convention decides *which* tenant; the constraint guarantees the answer is always exactly one."

The two-layer decomposition that makes the post worth reading. Layer one (`search_path`) is correctness — right only as long as the code is right. Layer two (the per-tenant PostgreSQL role) is confinement — it survives the day the code isn't.

> "Layer two does not check that we picked the *right* tenant."

The honest limitation, and the reason the article lands as engineering rather than marketing. If the wrong schema is stamped upstream, both layers faithfully point at the same wrong tenant. The role bounds *what a task can do given that decision*; it never second-guesses the decision itself. The trust boundary is the same django-tenants request resolution the entire web tier already relies on.

> "Make it so that the day you're wrong is not the day your customers find out."

The closing imperative, and the whole point of paying for a constraint instead of a convention.

## Key Themes

- **#concept** — **Convention vs. constraint.** The distinction that organizes the post: a convention is discipline-dependent; a constraint is enforced by the system. The convention decides which tenant; the constraint guarantees the answer is always exactly one.
- **#pattern** — **Two-layer defense, different failure modes.** `search_path` (correctness) guards the unqualified-query path; the per-tenant role (confinement) guards the schema-qualified and no-context paths. Each catches what the other cannot.
- **#pattern** — **Fail closed, twice.** A missing `tenant_schema` raises before the task runs; a missing decorator strands the task in `public` under a role that can reach nothing. Both turn "you forgot something" into a hard stop, not a silent default.
- **#tool** — **django-tenants + dramatiq + PostgreSQL roles.** The substrate: schema-per-tenant via `search_path`, a shared worker queue, and per-tenant roles with `GRANT USAGE` confined to their own schema plus `public`.

## Critical Analysis

**The contribution is the two-layer decomposition, not the decorator.** The decorator pattern (stamp the tenant on the message, set context, `finally` teardown) is a well-trodden shape. What makes the post valuable is its precision about *what each layer does and does not protect*. Layer one is correctness and carries all the weight of "if everything works." Layer two is confinement and is immune to the task's own code — but only because the author is willing to state its ceiling: it does not verify the tenant was chosen correctly. Most writeups oversell their defense-in-depth; this one draws the boundary between layers and refuses to let layer two claim credit it doesn't earn.

**"Name the enforcer out loud" is the transferable step.** The move from per-tenant deployments to a shared fleet is a standard consolidation most growing teams make. The part that's easy to skip — and that the article argues should precede the architecture change — is saying out loud what had been doing the enforcing. A guarantee with no line of code behind it is invisible exactly when you change the thing providing it. That framing connects directly to [[Multi-Tenancy Isn't About Databases]]'s "what are you trying to isolate?" and to the first-question-before-first-answer instinct in [[Software Engineering Craft]].

**The isolation ladder is useful but under-used.** The four-rung spectrum (database per tenant → deployment per tenant → schema per tenant with runtime context → shared schema with tenant column) is presented as an aside, yet it does the most general work: you pick where the isolation bill gets paid, then you must know exactly what on that rung is doing the enforcing. [[Nubase]] sits at the top rung and pays operational cost for a connection-string enforcer; this article sits one rung down and pays application discipline for a role-grant enforcer. Neither is wrong — the sin is not knowing which bill you've chosen.

**Where it connects to the agent story.** The failure modes here — stale context inherited across recycled work, teardown skipped by an early bail, a boundary that lives only in someone's memory — are the same shapes that [[Nango — Running Untrusted Customer Code at Scale]] found when a warm Lambda environment serves two customers in turn. The shared worker reusing one database connection across tenants is the database analog of the reused warm environment, and the medicine is identical: make the boundary structural, not remembered.

## Related Pages

- [[Multi-Tenancy Isn't About Databases]] — This post is the worked example of Comartin's "deployment isolation is not schema isolation" distinction, and it sharpens the spectrum by insisting you name *what* on your chosen rung is enforcing.
- [[Guardrails and Feedback Loops]] — The convention/constraint line is the database-flavored restatement of "linters beat prompts": encode the invariant in the substrate, not in the instruction.
- [[Nubase]] — Sits at the top of the same ladder (database-per-tenant); this post shows the middle rung and the constraint that makes it viable.
- [[Nango — Running Untrusted Customer Code at Scale]] — The identical failure shape — reused execution context leaking tenant state — solved the same way: a boundary enforced by the system rather than remembered by the code.

---
*Sources: [[raw/a-convention-is-not-a-constraint]], [[summary/a-convention-is-not-a-constraint]]*
*Last updated: 2026-08-26*
