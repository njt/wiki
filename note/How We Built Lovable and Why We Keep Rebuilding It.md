# How We Built Lovable, and Why We Keep Rebuilding It

Anton Osika's retrospective on Lovable's path from GPT Engineer to 70 million user-built projects. The thesis: every major architectural win came from **constraining the problem, controlling execution, and learning from the results** — and the biggest wins were reversals, most notably deleting almost all of their autonomous-agent code in favor of a predictable tool chain, and replacing planned fine-tuning with retrieved "knowledge files."

---

## Key Quotes

> "Constrain the problem, control execution, and learn from the results."

The whole essay compressed to one line. Greenfield-only, opinionated stack, one-click hosting: narrowing to new apps spared them "the enormous variety of existing codebases, dependencies, and environments," so one improvement to the common setup helped every project at once. This is the same logic as [[Lessons from Building Cursor]]'s incentive-shaped context and the [[The GUS Stack — Go, Unix, SQLite]] prescription for training-data-dense boring stacks — control the environment the model works in, not just the model.

> "Giving them more freedom made the product less dependable."

The sharpest admission in the piece. Mårten Wiman built a sophisticated async design where the agent delegated to subagents; it produced rabbit holes and unpredictable finish times. The fix was deletion — "we deleted almost all agent code and went back to one user request resulting in a rather predictable chain of tools," with edits wrapped in `<lov-write>` tags and parsed afterward because native tool calling wasn't reliable enough in late 2024. Direct tension with the subagent enthusiasm in most of this wiki's orchestration sources: for a non-programmer user base, dependability beat capability.

> "No fine-tuning, just text. The pattern will be familiar to anyone using agent skills today."

The knowledge-files reversal. The 2023 plan assumed fine-tuning on product-generated data; instead engineers read broken-app chat logs and wrote short instruction notes (e.g. "Only use icons that exist in lucide-react. Never invent an icon name."), retrieved into context per task. Osika calls it "a scaling law" — a lesson from one broken app helps the next user without a training run. This is a foundational-data-point for [[Agent Memory]]'s procedural-memory category and the skills pattern generally, four years of lore pre-dating the current skills boom. Lovable Stack Overflow, retrieving problem/solution pairs, cut stuck-user rates by 5%.

> "A coding agent's speed depends on how quickly it can check and correct its work, too."

The feedback-loop infrastructure section: persistent sandboxes keeping each project's dev server alive, incremental TypeScript checks that reuse prior work, errors flowing back to the model while the app keeps running. The point wasn't capability but cycle time — echoing [[Agentic Verification]]'s thesis that loop speed *is* the incentive structure.

> "We needed to preserve the running processes, too. Our stateful sandboxes checkpoint the machine's memory, CPU state and disk."

Forking and merging whole computers, not just code: resume days later in seconds, fork a full application environment to try approaches from the same starting point. What began as latency optimization became the platform's foundational primitive — and, per the closing section, a candidate building block for "a better software stack" alongside durable execution and verifiable security.

## Key Themes

#pattern #tool #concept

- **Freedom must match model ability.** Autonomy is a dial calibrated to current model reliability, not an aspiration; over-constraining is reversible, unpredictability isn't.
- **Text over training.** Knowledge files beat the fine-tuning plan; the skills-adjacent pattern (write lessons down, retrieve per task) compounded faster than any model update.
- **The edit-check-correct cycle as the real product surface.** Sandboxes, incremental checks, and machine checkpointing all serve cycle time and resumability.
- **Trust as the open frontier.** The named hardest problem: "One misunderstood request from a user can delete data or break something that can't be undone."

## Critical Analysis

The most valuable thing here is the honesty about regression. Most agent-company retrospectives narrate toward autonomy; this one documents walking it back and winning because of it. That said, the essay is also a recruiting post, and the three "open problems" section reads partly as a roadmap pitch — "verifiable security and privacy guarantees" is named without any of the how. The knowledge-files story is the genuinely transferable contribution: it's the cleanest first-person account of the insight behind the skills ecosystem, and it cost almost nothing to implement. The checkpoint-the-whole-machine work is the least replicable but the most interesting long-term — if forking computer snapshots really does become a primitive, it changes what "agent session" even means, which is exactly the territory [[A Deep Dive on Agent Sandboxes]] approaches from the containment side rather than the persistence side.

## Related Pages

- [[A Deep Dive on Agent Sandboxes]] — complements this source: Freeman covers sandboxing as containment (deny by default at the OS level); Lovable's sandboxes are the same primitive aimed at persistence and speed instead of security.
- [[Agent Memory]] — strengthens Jones's procedural-memory category with a production origin story: knowledge files are retrieved how-to knowledge that beats fine-tuning, exactly the quadrant Jones flags as under-exploited.
- [[Lessons from Building Cursor]] — convergent findings from another top AI coding product: controlling what the model sees and in what environment matters more than raw model capability.
- [[Agentic Verification]] — Osika's edit-check-correct infrastructure is a production embodiment of the loop-speed argument: the agent that corrects fastest wins on feel, not just accuracy.

---
*Sources: [[raw/how-we-built-lovable-and-why-we-keep-rebuilding-it]], [[summary/how-we-built-lovable-and-why-we-keep-rebuilding-it]]*
*Last updated: 2026-11-09*
