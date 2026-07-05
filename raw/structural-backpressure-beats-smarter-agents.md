---
url: https://reubenbrooks.dev/blog/structural-backpressure-beats-smarter-agents/
date_fetched: 2026-07-05
backfilled: true
---

# Structural Backpressure Beats Smarter Agents

Formal verification gates for AI coding loops, in the language you're already shipping.

Some of the most serious software bugs are also the most boring. A user should not be able to read another tenant’s data. Nobody disagrees with this, nobody stands up in a design review to defend Alice reading Bob’s records, and yet broken access control remains the #1 category on the OWASP Top 10.

These bugs ship because the rule has been placed in the wrong part of the system. It lives in a prompt, in a review checklist, in the shared expectation that every future engineer, and now every future model invocation, will remember the invariant and reapply it correctly.

That assumption was already weak, and with AI generating most of the code, it fails outright. You can do all the obvious things: put rules in `CLAUDE.md`, write a careful system prompt, add “authorization IS VERY IMPORTANT” to the agent instructions, and you should do all of that. But after the model has written sixteen thousand lines, the real question remains: how do you *know* the code does what you wanted? Tests help, but tests are empirical. They check the cases you and the model remembered to write, and they cannot speak for the handler someone adds next week.

I want to pull a different lever. My bet, stated plainly, is this: **for a wide class of production software, structural backpressure beats incremental improvements in agent intelligence.** Existing models can already write almost all of your code. The limiting factor is whether you can *know* that they did what you wanted, and that knowledge comes from the substrate they write against, not from waiting for a smarter model.

Shen-Backpressure is the tool and methodology I built to explore that bet. I will show what it does through a running demo, and then show how to wire the same loop into your own project.

## Behavioral Gates And Structural Gates

Most prompt-level constraints are *behavioral gates*. We tell the model “do not skip authorization,” “validate inputs,” “use the shared helper.” Models follow these instructions often enough to be useful and fail often enough to make the whole arrangement unstable. A behavioral gate depends on the model remembering the rule, recognizing where it applies, resisting the gravitational pull of local context, and then on a human reviewer maintaining the same invariant across the whole codebase.

*Structural gates* are different. A compiler, a type checker, a test runner, a linter, a proof checker. Each produces a concrete answer about the artifact in front of it. The answer is not perfect, but it is real, and inside its scope it refuses when the code is wrong.

That refusal is the point. It lets us move work out of the model’s instruction space and into the substrate the model is building on. Instead of spending tokens begging the model to remember an invariant, we arrange the code so the invariant is hard to violate by accident: take the property you care about most, express it in a form a machine can check, project it into the implementation, and let the loop bounce off that check until the emergent artifact satisfies it.

This is what makes backpressure, in the sense Geoff Huntley’s Ralph and the essay Don’t Waste Your Backpressure use the term, powerful. When previous errors are piped into the next iteration, a deterministic gate gives the loop something firmer than vibes to push against. That loop is no longer a niche idea: Codex CLI now ships `/goal`, OpenAI’s own take on the Ralph loop, keeping a goal alive across turns and refusing to stop until it is met.

## The Substrate Move

The invariants worth enforcing are usually easy to state precisely. *A user may access a resource only if authenticated, a member of the tenant, and the resource belongs to that tenant.* That is a complete, bounded rule. English is simply the wrong medium in which to *enforce* it.

Shen-Backpressure uses Shen, a small, statically-typed Lisp with a sequent-calculus type system, to write that kind of rule in a form a machine can project into the substrate: the target-language types, constructors, and gate commands the model has to write against. You write the spec once; a code generator (`shengen`) lowers it into guard types in your target language. The model writing Go or TypeScript never needs to know Shen exists. It needs the code to compile and the gates to pass.

## A Proof Chain For Multi-Tenant Auth

Here is the heart of the multi-tenant API demo, an excerpt from `specs/core.shen`:

The horizontal line does the work. Premises above the line must be satisfied before the conclusion below it can be constructed. To get a `resource-access` you need a `tenant-access` and proof the resource is owned, and to get a `tenant-access` you need an authenticated principal and proof of membership. The full chain runs `jwt-token → authenticated-user → tenant-access → resource-access`. See the full spec for the intermediate rules.

These types are witnesses. Constructing a value of one of them requires discharging the premises declared in its rule.

## From Spec To Guard Types

`shengen` lowers each rule into a guard type in the target language. In Go, the fields are unexported, and the generated constructor is the only way to populate one:

There is no exotic trick here, only ordinary Go visibility. Code outside the package cannot write `TenantAccess{isMember: true}` because the fields are lowercase. The constructor is the only path to a populated value, and it refuses `isMember == false`. Sum types like `authenticated-principal` get a sealed interface the same way. See the generated guards.

Go is the example throughout this post, but the concepts and the tool are not Go-specific. Go and TypeScript are the production targets today, with reference emitters for Python and Rust. The target language is a real choice with real tradeoffs: the seal is only as strong as the encapsulation the language gives you, and agents are not language-neutral either.

Smart constructors are old. Type wrappers are old. Codegen is old. The useful move is putting them in the loop as a single refusal surface, sourced from a spec shorter and more reviewable than the enforcement code it generates.

## Authorization Without The Hand-Written Check

The normal way to write a multi-tenant handler is to put an `if` in every endpoint:

That pattern is reasonable, and it is exactly the sort of reasonable thing that gets forgotten on the seventh handler or the third refactor. In the Shen-Backpressure version the membership check still exists. There is still a database query, but it is concentrated at the construction boundary for `TenantAccess` instead of scattered across handlers as a convention:

The handler then operates on a value that *represents the already-traversed chain*. The proof travels with the value. In the running demo, Alice, a member of Acme, can list Acme’s resources and is refused Globex’s:

If the agent tries to skip the chain and pass a raw value, the build fails before a binary exists:

That short, mechanical “no” is the backpressure. I want more of those, and fewer paragraphs in the prompt.

## Try It

The demo is built to be read end to end. Clone the repo and open `examples/multi-tenant-api/`: it ships the spec, the generated guards, the Ralph loop that built it (`cmd/ralph/`), and a curl transcript in `demo.md`.

To wire it into your own project, install the `sb` CLI and run:

`sb init` scaffolds a starter spec and the gate scripts, and with `-config` it also scaffolds the `sb.toml` manifest. Every iteration of the loop runs a fixed set of gates, declared in `sb.toml`:

| Gate | Command | Catches | 
|---|---|---|
| shengen | `sb gen` | Drift between the Shen spec and the generated guards | 
| test | `go test ./...` | Runtime invariant failures and ordinary regressions | 
| build | `go build ./...` | Type-signature mismatches, invalid proof-chain usage | 
| shen tc+ | `bin/shen-check.sh` | Internal inconsistency in the Shen spec | 
| tcb audit | `bin/shenguard-audit.sh` | Hand edits to generated guard code | 

Those five are the default set. `sb` also has an optional, still-experimental sixth gate (`shen-derive`, for spec-equivalence testing) that runs only when explicitly configured. When a gate fails, the failure feeds into the next prompt as concrete context. That is the backpressure. You can run `sb gates` yourself between iterations if you want to drive the loop by hand. The harness is pluggable: Claude Code (`claude -p`) is the default; Cursor, Codex, and others work by setting `RALPH_HARNESS`. Gate 4 needs a Shen runtime: `brew tap Shen-Language/homebrew-shen && brew install shen-sbcl`. `sb init -lang ts` wires the whole gate loop for TypeScript, not just codegen. It is a full target alongside Go. And the `/sb:*` commands `sb init` installs into `.claude/` include `/sb:create-shengen`, a complete prompt for generating a `shengen` emitter for a new target language.

## Costs And Limits

Writing a spec is not free. You decide which invariants are worth encoding, express them in a notation that is both readable and projectable, and maintain the generator and audit scripts. The generated guard code is sacred; edit it by hand and the audit gate rejects it. Your trusted computing base now includes the Shen type checker, the generator, and the target compiler.

And it does not make bypasses *impossible*. This is where the target language tradeoffs really become apparent. In Go, code inside the guard package could forge values; reflection and zero values are theoretically available as escape hatches. A careless SQL query can hand the constructor a `true` it should not. My claim is deliberately narrow: using shengen to lower spec proofs into the target language makes the specified invariants *practically impossible* to bypass *by accident*, not categorically impossible to bypass at all. For a formal-methods audience that is unremarkable, and a fairly weak claim. But for a practitioner shipping LLM-generated code this is an extremely high-leverage tool. Now, the forgotten check, the leaked tenant ID, and the copied-but-incomplete handler all become *structurally difficult and expensive* to introduce by accident.

The spec itself can be wrong, the generator can drift, and tests still miss cases. Naming those limits is important to understand the tool, to leverage what it offers in a disciplined way.

The cost of installing these gates is itself falling: writing the spec, the emitter, and the audit script is exactly the work models keep getting better at. Better models do not make the substrate unnecessary; they make skipping it harder to justify.

## The Thesis

For production AI coding loops, you need *better backpressure* more than you need a better model. You need deterministic signals that tell you whether the artifact has the shape you intend. Tests give you one such signal, compilers give you another, and a Shen spec lowered into guard types extends the compiler’s refusal surface further still — proof-shaped constraints that travel from design intent into the code itself.

None of this is a bet against better models. But capability and certainty are different things. “The model is reliable” is a claim about the writer; “this artifact upholds the invariant” is a claim about the one object in front of you. Someone could vibe-code an implementation that passes every test you thought to write, and the reliability of whoever wrote it, model or human, still would not tell you what a structural gate tells you about the artifact itself. That is why the wrong paths being structurally hard to take by accident matters at every level of model capability.

And the same gate that gives you that certainty gives you the artifact to demonstrate it: “we used a capable model” is not something you can hand a regulator or an auditor, but a spec, a passing gate, and a green CI run are.
