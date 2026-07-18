# Common Expression Language (CEL)

Google's embeddable expression language for policy evaluation, validation, and data transformation in performance-critical systems. Non-Turing complete by design — the constraint is the killer feature.

---

CEL is an expression language built for embedding. It evaluates predicates and simple data transformations at nanosecond-to-microsecond speeds, and it's designed to be safe: non-Turing complete, sandboxed to data the host provides, with predictable cost per evaluation. Google uses it inside Kubernetes (validation rules), Firebase (security rules), and IAM (policy conditions). The syntax is C-like and familiar — `account.balance >= transaction.withdrawal` — and it supports protocol buffers and JSON as first-class data shapes.

## Key Quotes

> "An expression language that's fast, portable, and safe to execute in performance-critical applications."

The landing page's three-word pitch. "Safe" is doing the most work here — safety through incompleteness, not sandboxing. CEL can't loop, can't recurse, can't allocate unbounded memory. It's a language that refuses to be dangerous.

> "CEL is especially useful for predicate logic and simple data transformations."

The humility is the point. CEL doesn't try to be a general-purpose language. It's a predicate evaluator that happens to look like code. This is the opposite of "configuration languages grow until they can read mail" — CEL stays in its lane by structural constraint, not developer discipline.

> "A one-time configuration cost for validating the expression" followed by frequent evaluation at negligible cost.

The compile-then-evaluate-repeatedly pattern. The insight is that in policy systems, expressions change rarely but execute constantly. Optimizing for eval speed over parse speed is the right call.

## Key Themes

- **#tool**: CEL is infrastructure, not a product. It's the expression evaluator you embed in your thing, not a thing you use directly.
- **#pattern**: Non-Turing-completeness as safety guarantee. Rather than sandboxing a dangerous language, design a language that can't express danger. [[Guardrails and Feedback Loops]] applies here — deterministic enforcement beats runtime pleading.
- **#concept**: The "evaluate often, modify rarely" optimization profile. Same shape as JIT compilation, but for policy: pay compile cost once, reap eval speed forever.

## Critical Analysis

**The constraint-is-feature design is CEL's real contribution.** Most embedded scripting languages (Lua, JS, Python) try to be general-purpose and then need sandboxing, resource limits, and timeout guards bolted on. CEL starts from "what can we express if we forbid unbounded computation?" and discovers the answer is "most of what policy and validation actually need." This is the same design philosophy as Starlark (Google's Python subset for Bazel) — limit expressiveness to gain predictability — and it's the right tradeoff for infrastructure.

**The competition is JSON Schema and friends, not Lua.** CEL's natural enemies aren't scripting languages but declarative validation formats. Against JSON Schema, CEL wins on expressiveness (actual predicates, not just structural constraints). Against a full policy language like Rego, CEL wins on simplicity and embeddability. The trade: CEL can't do graph traversals or recursive policy evaluation. If you need that, you're in Rego territory.

**The Google monoculture problem.** CEL is designed by Google, for Google-shaped problems (protobufs, gRPC services, IAM policies). The protobuf integration is deep and first-class. If your stack isn't on protobufs, CEL still works (it supports JSON natively), but you're not the design target. This is the same dynamic as gRPC vs REST — Google's internal architecture manifesting as open source, and the rest of the industry adapting around it.

**Non-Turing-completeness is underrated as a security primitive.** Turing-completeness is the root of all sandbox escapes. If your language can't express an infinite loop, you don't need a timeout. If it can't allocate, you don't need memory limits. CEL demonstrates that a surprisingly large class of real-world computation fits in this box. The question it raises: what else could we build by accepting this constraint up front? [[Constraint Decay]] explores the flip side — how constraints degrade agent performance — but CEL shows constraints can also be design tools.

## Sources

- [[raw/cel]] — cel.dev landing page

*Last updated: 2026-07-18*
