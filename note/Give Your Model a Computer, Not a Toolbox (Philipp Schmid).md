# Give Your Model a Computer, Not a Toolbox (Philipp Schmid)

Philipp Schmid's AI Engineer "Code 2026" talk is an argument from experiment: build the same coding agent with a full 2025-era tool suite and with a single bash tool, run both against ~350 tasks, and compare. Scores come out roughly identical; tokens and dollars do not. The bash-only harness is ~2x cheaper per task, and with Gemini 4 Carbon up to 3x fewer tokens, because a capable model chains multiple commands per call and writes one inline Python script instead of ten read/edit turns. His thesis: models keep "eating the harness," accuracy can no longer distinguish harnesses, and the industry should start measuring token efficiency — and treat every new model release as a prompt to delete harness, not add to it.

---

## Key quotes

> "And the model tries to basically solve that task to use different tools. And then based on the score from the verifier, we update the model. But no one is telling the model which tools to use."

Post-training discovers tool strategy by reward alone. That's why dedicated tool taxonomies are a bet that loses value every model generation: whatever structure we impose is re-derived, or worked around, inside the model.

> "On the left side we roughly need around 10 turns to make the same changes to a coding repository compared to the two turns on the right side just using bash and it also reflects in the amount of tokens... around 10x more tokens."

The one-edit-per-turn pattern of dedicated file tools is an artifact of harness design, not model capability. Reading parallelises fine; editing serialises. Bash collapses both into one call.

> "If we just look at the score column, all of them look roughly identical... But if we look at the token column or even more at a dollar per task, there's a picture which is changing."

This is the talk's most damning empirical point: on modern models, accuracy-based evals are nearly blind to harness quality. Score-only benchmarking systematically hides a 2x cost difference.

> "If your harness is more complex to use the model will work around it, will find solutions around it, but needs more turns and more tokens to achieve the same results."

Harness complexity is a tax paid in tokens. The model doesn't fail against a bloated harness; it succeeds expensively.

> "Give your model a computer, not a toolbox."

The line that names the whole design philosophy. A computer (shell + filesystem) is a universal interface; a toolbox is a frozen guess about what the model will need.

## Key themes

#concept #pattern #tool #person

- **Harness shrinkage as a capability graph**: 2023 workflows → 2024 many-tools agents → 2025 10–20-tool coding agents → 2026 one-tool. Every category of dedicated tool (planning, memory, subagents) gets reabsorbed: the filesystem is memory, bash can re-invoke the agent CLI itself.
- **Token efficiency as the missing eval axis**: score is saturating; tokens/task and dollars/task are where harness differences now live. Cache hit rate improves too, since shorter, similar trajectories are cheaper to cache.
- **The capacity threshold**: bash-only only pays off above some model capability. Flash used bash as a one-command tool; Carbon chains 2.2–3.7 commands per call. A minimal harness is a bet on the model's ceiling.
- **A countercurrent, honestly flagged**: Schmid predicts code-mode (direct TypeScript/Python) and MCP-with-structured-outputs return — not for model capability but for auditability and security, since raw bash is hard to proxy and inspect.

## Analysis

This is one of the more falsifiable talks in the harness-design literature, and its discipline is what makes it land: same model, same tasks, four harnesses, and the refusal to call a winner on score alone. The honest caveats (single run per task, so score variance could be noise) are stated out loud rather than hidden — which makes the token result, which is far outside noise, more credible.

The provocative implication is that most harness engineering of 2025 was compensating for weak models, and those compensations are now liabilities. There's a satisfying convergence with the general "simplest harness wins" current in this wiki, but Schmid adds the economics: the case for minimal harnesses was usually made on grounds of elegance or trust; here it's made on dollars per task, which is harder to argue with. The filesystem-as-memory claim is also quietly radical — no memory subsystem, just `cat` and `tee`.

The weak point is generalisation. His eval is his own ~350-task set, run once per task, with one model family; "models will keep eating the harness" is a trend extrapolated from two model generations at one lab. And the security caveat he waves at — bash being unproxyable — is doing a lot of load-bearing work in his own predicted future: if code-mode and MCP return for auditability, then the pure bash endgame may never be shippable in enterprises, and the minimal-harness frontier may bifurcate into "cheapest" and "auditable."

## Related pages

This talk strengthens [[Honey I Shrunk the Coding Agent]] with harder numbers: where that page argues harnesses should shrink, Schmid quantifies the shrinkage at ~2x cost on identical scores. It nuances [[Harness Engineering]] by attacking its premise — the eleven-agent anatomy tradition assumes tool design is the craft, and Schmid argues the craft is now deciding what to delete. It complicates [[Components of a Coding Agent]] by showing entire components (planning, memory, subagent dispatch) can be replaced by one bash tool plus a filesystem, and it speaks directly to [[Reducing Token Spend with Deterministic Workflows]] from the model side: instead of making the workflow deterministic to save tokens, make the interface flexible enough that the model saves them itself.

---
*Sources: [[raw/philipp-schmid]], [[summary/philipp-schmid]]*
*Last updated: 2026-11-26*
