# The Rule of 2 (Chromium Security)

Chromium's Rule of 2 is a triad constraint for handling untrustworthy input: pick at most two of untrustworthy inputs, unsafe (memory-unsafe) language, and high privilege. The page is a working engineering policy, not an essay — it names the approved safe languages, the blessed IPC types, the exceptions (RE2, Protobuf), and the honest list of standing violations — and it is the clearest articulation anywhere of "containment as a design constraint" rather than an afterthought.

---

## What it argues

The rule exists because bugs in high-privilege code that touches internet input are Critical-or-High severity by definition. Rather than demanding perfect code, the policy reduces the *blast radius* of inevitable bugs by forcing at least one leg of the triad to give way. The evidentiary base is blunt: ~70% of Microsoft's annual security fixes are memory safety issues, and in March 2019 only ~5 of 130 public Chromium Critical bugs were not obviously memory corruption.

Three definitions carry the weight:

- **Untrustworthy input** is nearly everything. An input is trustworthy only if its grammar is so trivial you'd parse it safely *regardless of source* — and such grammars are vanishingly rare. So provenance matters, and "any arbitrary peer on the Internet" is untrusted unless cryptographically proven otherwise.
- **Unsafe languages** are C, C++, assembly. Rust's `unsafe` is treated as an unsafe subset inside the safe language: tolerable only if every unsafe block is reviewable "with simple local reasoning."
- **High privilege** is relative — from firmware down to the Chrome browser process, anything that can act as the user counts.

## Key quotes

> "Pick no more than 2 of [untrustworthy inputs, unsafe implementation languages, high privilege]."

The whole policy in one line. Its power is that it's checkable at code-review time: a CL description can name which leg it sheds, and Chrome Security "will generally not approve landing a CL" that sheds none.

> "We'd love to reduce the severity of such bugs by reducing the amount of damage they can do (lowering their privilege)… Security engineers would like to advance the state of engineering to where memory safety issues are much more rare."

Severity reduction over bug prevention — an acceptance that bugs are inevitable, which is what makes the rule durable. Contrast with any policy premised on developers simply writing correct C++.

> "All unsafe usage must be able to be reviewed and verified by humans with simple local reasoning."

The Rust carve-out done right: unsafety isn't banned, it's *localized*. This is containment applied inside a language, mirroring the process-level sandboxing elsewhere in the document.

> "We know there is code in Chromium that violates the Rule of 2… Our top priority is avoiding any *new* violations."

The most credible sentence in the doc. A rule that admitted zero exceptions would be fiction; instead there's a ratchet — freeze new violations, retire old ones (the C++→Rust JSON parser swap as the worked example).

## Themes

#concept #pattern #tool

- **Design-time containment**: security as a constraint on architecture (which process, which language, which grammar), enforced at review, not a post-hoc audit.
- **Privilege as a resource**: the doc treats privilege like a budget to spend, and normalization (PNG → `SkBitmap` in a sandbox, then parse the simple form at high privilege) as a way to *launder* untrusted data into trusted structured objects.
- **Honest exceptions**: RE2, Protobuf-with-caveats, Mojo's C++ deserializer accepted for performance — each exception is named, scoped, and reasoned about rather than smuggled in.

## Opinionated take

This is one of the great security engineering documents because it is *boring*: no novelty, just a counting rule plus the discipline to keep it. The triad framing generalizes far beyond memory safety — swap "unsafe language" for "unreviewed agent-written code" and "high privilege" for "production credentials," and you have a workable policy for agentic systems: don't let untrusted input reach powerful tools through unverified code paths. The weak joint is the same one the doc admits: the existing-violations backlog only shrinks when someone prioritizes it, and "the network process on Windows is not sandboxed" stood for years. Rules like this need a ratchet mechanism with an owner, or they decay into tribal lore.

The grammar-simplification idea (parse complex hostile formats only in a low-privilege process, then pass simple structured objects upward) is the most transferable technique here — it's essentially the dataflow discipline that modern agent-sandbox designs reinvent.

## Related pages

- [[Security and Sandboxing]] — this doc is the ur-text of design-time containment; the topic page's agent-era sandboxing debates are direct descendants of the same privilege-reduction logic.
- [[A Deep Dive on Agent Sandboxes]] — the agent-sandbox taxonomy restates the Rule of 2's "sandbox the parsing" leg for LLM tool execution; Chromium shows the pattern at industrial maturity a decade earlier.
- [[Zero Trust for AI Agents]] — Chromium's cryptographic-provenance leg (pinned TLS, hash-verified content over untrusted channels) is zero-trust input handling avant la lettre; the AI-era version extends the same distrust to model outputs.
- [[Nango — Running Untrusted Customer Code at Scale]] — Nango applies the same triad to executing customer code: untrusted input is a given, so isolation and language/runtime choice carry the load.

---
*Sources: [[raw/rule-of-2-md]], [[summary/rule-of-2-md]]*
*Last updated: 2026-10-11*
