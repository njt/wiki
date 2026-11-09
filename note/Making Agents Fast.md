# Making Agents Fast — What I've Learnt Over the Last 12 Months

Martin Alderson applies a career of web/API performance optimisation to agentic workflows, and lands on a contrarian framing: inference speed is the obvious lever but usually not the biggest one. His rough model says halving inference time only removes ~20% of total task time once provisioning, tool calls, and suspend/resume are counted. The real wins come from reading tool-call traces, reading the model's thinking traces, building the tools the agent is visibly wishing for, and not forgetting that CPU cores, NVMe, and broken IPv6 still matter.

---

## Key quotes

> "Rough numbers again, but halving inference time only takes 20% off the task when provisioning, tools and suspend/resume stay the same."

The whole essay in one sentence. Performance engineering for agents is systems engineering, not model selection — the model is one term in a much larger equation.

> "A 'fast' tier model with thinking set to maximum or xhigh will be *vastly* slower than a 'slow' tier model with low or medium thinking."

Alderson calls thinking budgets "overwhelmingly the lowest hanging fruit" and admits he constantly overlooks them himself. This is a rare piece of advice that costs nothing to act on and can dwarf provider-tier differences.

> "With agentic workflows, you can often just *observe* what the agent does, and deduce what would work better."

This is where the essay breaks from classic optimisation. In web perf you profile a request; here you can watch the agent *wish* for a tool it doesn't have. Open-weight models make this literal — the thinking blocks before slow calls contain "hmm, could do x, but no y" moments you can simply fix by building tool y.

> "Treating `for` loops with `sleep` as a red flag. This is an indication that the agent wants to be *notified* of something but is instead having to poll for it."

A beautiful diagnostic: the agent's own code becomes a requirements document for your infrastructure. Polling loops in agent traces are a symptom of missing async completion hooks in external services — the same event-driven-versus-polling argument made for human-built systems, rediscovered from traces.

> "Before you know it you can be spending dramatically more time *starting* the shell than *running* the command."

Because many agents spawn a fresh shell per command, mundane `.bashrc` cruft (nvm, etc.) becomes an O(commands) tax. A detail only someone who has actually watched agent traces would know.

## Themes

- #concept — TTFT vs tok/s as TTFB vs bandwidth for agents; workload shape determines which matters
- #concept — thinking budgets as the overlooked dominant latency lever
- #pattern — reading CoT traces as a tooling requirements-discovery method (open weights advantage)
- #tool — the agent's environment as a performance surface: shell startup, NVMe vs networked disks, sandbox copy-on-write, IPv6 breakage, suspend/resume memory hygiene

## Analysis

The essay's strength is its provenance: someone who spent years making websites fast notices that agent slowness is mostly *familiar* slowness wearing new clothes. The TTFT/tok/s framing maps cleanly onto TTFB/bandwidth, and "noisy neighbour" transfers from hypervisors to inference providers — except with the crucial difference that provider load is invisible; there is no `vmstat` steal column for an LLM API. Alderson's footnote on quantisation (FP8 vs INT4 across providers means "same model" comparisons can be apples-to-oranges) is the kind of caveat most model-comparison posts omit.

The genuinely original contribution is the CoT-trace-as-product-research loop. Labs read thinking traces for safety; Alderson reads them for feature requests. That only works with open weights, which quietly becomes an argument for local/open models beyond cost and privacy — an argument rarely made explicitly. The companion diagnostic, treating `sleep`-polling loops as missing-notification symptoms, turns agent misbehaviour into an actionable infrastructure backlog.

The weakest spot is the tension between "give the agent the tools *it wants*" and his own admission that weak models botch parallelisation: the same agent that tells you what tool it wants will also misuse it. His answer — build the parallelism into the tool rather than the prompt — is right but underdeveloped. And the 20%-per-halving figure is presented as illustrative without a worked example; the shape of the claim is convincing, the number is vibes.

## Relations

This source strengthens [[Wall-Clock Time and the Qwen3.8-27B Daily Driver]]: that post argued wall-clock time, not tokens-per-second, is the metric that matters for a solo founder; Alderson generalises it — tok/s is only one term, and halving inference saves just ~20% of task time.

It nuances [[Controlling Reasoning Effort in LLMs]]: Raschka maps the design space of reasoning-effort *control*; Alderson supplies the field-data reason you'd turn that knob — thinking budgets dwarf provider-tier differences and are "overwhelmingly the lowest hanging fruit".

It complicates [[A Deep Dive on Agent Sandboxes]]: the sandbox that contains the agent is also a performance surface — copy-on-write file IO, page-streaming microVM resumes, and broken IPv6 are among Alderson's biggest real-world taxes.

---
*Sources: [[raw/making-agents-fast]], [[summary/making-agents-fast]]*
*Last updated: 2026-11-09*
