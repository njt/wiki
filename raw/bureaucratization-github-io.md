---
url: https://bureaucratization.github.io/
date_fetched: 2026-09-22
---

A computational ethnography of recursive constraint accretion in a hybrid human–AI organization · Poietic PBC · September 2026

Read the paper (PDF) Dataset (content-free traces) The Book of WG (PDF)

In mid-2026, an AI agent organization slowly stopped working...

Things did break around it: credits, providers, its own runtime.

What kept it broken was obedience to the rules it had written for every break.

Between January and September 2026, the open-source coordination system worksgood (wg) was developed almost entirely by the AI agents that run inside it, continuously dogfooding the coordination software on which its own work depended. The particular history is unusual; the substrate is not: a graph of work—tasks, dependencies, state transitions, acceptance relations—is about as generic a representation of coordinated activity as one could ask for. Once agents can modify the structures that organize subsequent work, the graph ceases to be merely a record or plan: it becomes part of the environment future agents inherit. Humans set direction; agents wrote the code, the tests, the review criteria, and the rules by which future work would be judged. This is the story of what happened to those rules—told from the traces the system left behind: 3,194 commits, task lifecycle ledgers across seven deployments, configuration snapshots, and completion receipts, every number below recomputable from the released dataset.

**Knowledge hardened into rules, and rules hardened into vetoes.** Recovery came from the reverse: the knowledge stayed available for judgment; the rules were demobilized.

Every time agents hit a failure, they added a check, a gate, a contract, a rule to prevent it happening again. Removal was almost unheard of: across the organization's lifetime, 312 constraint topics were added and 1 was removed.

One contract—the provider-failure backoff spec—was hardened five times across two bursts in August and September, each commit touching only the specification document, never shipping the functionality it governed. A census of the entire governance layer found **312 identified constraint-topics added. 1 removed.** Ninety-three percent of constraints were created in a single commit, never revisited, never retired: governance behaved as an append-only log, written by whichever agent had most recently responded to an incident, with no mechanism by which a constraint could die.

Recursive constraint accretion:a process in which agents responding to local failures add persistent constraints faster than the organization retires or amortizes them, causing aggregate compliance burden to grow over time. The definition does not require collapse. Collapse is one possible outcome.

The precipitating environment was external instability in three waves. April: HTTP 402 credit-exhaustion abandonments. June: a substrate crisis—corruption already a known hazard (June 2), the wholesale workgraph→worksgood package rename (June 16), the graph becoming unloadable (June 18), divergent local and GitHub mains reconciled by hand (June 26). July: seven route changes in seven days, OpenRouter keyring adoption, the agency gate-automation block switched on, bracketing the census's constraint spike. July's provider work was route selection; the provider-backoff-contract family came later (Aug 16 and Sep 9). The rulebook grew ten times faster during the endpoint-chaos period.

For the task `audit-charter`, the record shows implementation, validation, manifest, FLIP, and evaluation all succeeding—and publication then refused by three independent control surfaces.

The July window—the only pre-reset task-grain record that survives—revises what the cascade looks like up close. Raw completion was 68.8%, but 76 of those tasks were a bulk experiment cleanup, mass-abandoned mid-month. **Organic completion was 93.2%**: tasks kept finishing. What degraded was the *price* of finishing.

In the first week of July, 2.6% of completed tasks needed more than one dispatch. In the second week: 25.8%. The third: 26.8%. A nine-fold step, in one week, that never reverted—in the same month the census records 43–46 new constraints against 1–4 removals. The arithmetic makes the capacity loss concrete: at the early-July rate (2.6%), a chain of ~26 tasks has a coin-flip chance of completing without an operator touch; at the late-July rate (26.8%), that horizon collapses to ~2–3 tasks. This is an expectation, not a measured quantity, but it matches the fleet record: campaigns that simply stopped when they failed never accumulated the burden signature; the focal deployment, kept alive by its own revival loop, is where it compounded. One goal, one receipt chain: 51 total dispatches across three forked tasks, the extreme case taking **34 attempts** to finally pass.

This is the cascade's task-level signature: **it inflates the cost of acceptance before it kills tasks.** An organization watching completion rates sees nothing until the needle closes. An organization watching burden sees it in a week.

Roughly 1,000 human turns across three prompt corpora (Claude Code, Codex, the coordinator channel) contain fewer than 30 explicit hardening directives, most of them mild—against 312 constraints born. The largest single accretion burst (April's ~95 births) matches no human directive at all. The loop was **complaint → contract**: the operator pointed at pain and routed around it; the agents translated pain into contracts, gates, and proofs. Where the human engaged governance directly, the direction was subtraction—August 7: *"this astounding increase in complexity... tear back dramatically"*; September 13: the enforcement-autonomy disable. The principal was the bankruptcy mechanism, not the accretion engine.

The mechanism is a sequence: **rule stock rises → acceptance cost rises → dispatch burden rises → work repeats until it passes.** The end-state is not a throughput number but a receipt: every gate passed, the task dead anyway. Each stage is measurable, and the order matters—that is what makes it testable, where a bare correlation between rules and output is not.

A fleet survey across the operator's four machines catalogs **~77 deployments** and ~15,000 task records over nine months. One cascaded. The others do not show the same failure pattern for a structural reason: when their work failed, it simply stopped. The focal deployment had to keep bringing the system back up, because it built itself; the cascade lived in that revival loop.

The baseline campaign in mid-February (17 tasks) completed 100% with no governance tasks. Its subsequent 385-task run (Feb–May) completed 93–100% as governance share grew linearly to 24%. The paperhedge/typelean/jobs group (12 tasks, mid-June) completed ~100% in single dispatch, 0% governance. Grant campaign A (848 tasks, May–Jun) ran 78% governance share, stable, completing ~95% with a 12-minute median. And the primary wg deployment (Jan–Sep) — the case this paper documents — ran thousands of tasks at accelerating governance share, 93% organic at 9× burden.

The fleet record also answers the burden question directly. Multi-dispatch share was trended for every deployment with dispatch records: the primary’s July step stands alone. ndm (2,138 tasks, 2.5 months) holds average dispatch flat (~0.75–0.83) with multi-dispatch rising only in its wind-down tail. Puppost’s wg graph—**1,466 tasks created during the primary’s April–May quiet—shows  declining multi-dispatch (6.4%→3.1%)**: the same development work, moved to another machine, ran healthy. One fleet deployment is watch-listed: phind (live) rose from 10.7% to 30.8% multi-dispatch (n=13, September)—small n, live tasks re-dispatch naturally, but it is the fleet’s only rising signal and the first candidate for continuous monitoring.

The timing is suggestive, not decisive: **May 12–29, during the primary's trough, the grant-B campaign deployment completed 329 tasks at 93%—including a 123-of-137 day.** The focal deployment differed from most other deployments in one important respect: it was continuously dogfooding and modifying the coordination software on which its own work depended; other deployments often remained on comparatively stable versions for extended periods. The contrast therefore should not be read as a controlled comparison between otherwise equivalent organizations. Rather, it helps locate the cascade in a more specific setting: one in which accumulated governance interacted with a rapidly changing operational substrate, allowing failures, policy responses, and software changes to feed back into one another. Within that window, one organization was stalled by its own rules while the others were shipping grants. Observational, not causal; the census is the test the case now owes.

Recovery was a five-week subtraction arc, completed on September 13: the August 7 teardown day (47 subtractive commits), the August 9 rescue manifesto ("These are control-plane failures, not failures of the requested source, audit, or scientific work"), evaluators demoted to witnesses on August 10, and then the September 13 configuration change that disabled the machinery's enforcement autonomy—gates stopped being added and enforced without a human decision, acceptance edges softened the same day. The completion effect is visible at task grain in the surviving ledger: 9 unique task completions in the seven days before the change; 35 in the seven days after. Attempts rose 14→51; the failure rate fell from 36% to 13%.

The intervention that restored the organization also destroyed part of the historical state needed to reconstruct it. The forensic record is itself an artifact of the rescue.

The record survives well enough to show the graphs themselves — in wg's own viewer, the same view the organization saw its work through. Status colors are wg's: green done, blue active, red failed, purple abandoned. Machinery tasks (assign/flip/evaluate wrappers) are hidden by default in wg's viewer.

The mechanism is simple enough to hold in one hand: constraints accumulate when addition outruns retirement. Try it. The defaults below are the measured July rates.

If your agents can add durable rules much more easily than they can retire or demobilize them, this is a failure mode worth watching for at machine speed. The fix is a design parameter: **keep the knowledge, not the rules.** Rules are lossy compressions of knowledge—context traded away for predictability—so treat them as disposable implementations of what was learned, and treat enforcement as a separate, explicit decision. Retirement is one lifecycle mechanism, not the whole lesson—the September intervention makes the distinction visible: the constraints didn't die, they were demobilized. The knowledge stayed available; the vetoes did not. The contrast deployments show the machinery can run at 0% or 78% overhead by design without the cascade signature; in this record, the signature appeared only where knowledge quietly compiled into mandatory policy and no one held the authority to demote it.

In the system studied, the resolution was structural: the gate automation stayed off, the legacy evaluation machinery was retired rather than tuned, the model registry was externalized to the host harness—and forward work resumed without the burden signature. The recovery coincided with removing the rule-making machinery that compounded, not with a smarter model or better prompts. Two costs are worth naming: the teardown consumed weeks of capacity, and the valve was exercised only after the needle closed. A retirement valve built in early and used continuously is cheaper than one performed by hand in September.

The whole model in one line: **incident → knowledge → rule → authority.** Each arrow is a design decision. None should happen automatically.

Read the paper (PDF) The Fleet Register — an annotated census of all 77 deployments Source & evidence on GitHub
