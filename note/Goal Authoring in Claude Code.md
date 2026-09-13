# Goal Authoring in Claude Code

forefy's tweet thread on Claude Code's `/goal` — a wrapper around a session-scoped, prompt-based Stop hook that keeps an agent working until a goal is judged met, judged impossible, or judged neither and told why — plus the authoring discipline it demands. The thread's real subject isn't the command; it's who writes the predicate that decides when the agent may stop, and what happens when that predicate is badly written.

---

## Key Quotes

> "Most people tell claude to write an optimized /goal for them - this is like doubling down the milk to improve your coffee."

The best line in the thread, and a real principle in disguise: the agent you are about to govern is the last party who should author the termination condition that governs it. A self-written stop condition is a conflict of interest in YAML form. The metaphor is deliberately absurd, but the failure it names — optimizing the judge instead of the goal — is exactly what [[Dynamic Workflows in Claude Code]] calls self-preferential bias, one level up.

> "/goal is a wrapper around a session-scoped prompt-based Stop hook. this means that when a session hits 'Stop' the harness evaluates the condition, one of the following will happen as a result: 1. Goal clearly not met? keep working and take the reason as guidance 2. Goal met? clear the Stop hook (done) 3. Goal impossible? clear the Stop hook (failed)"

The mechanism, stated cleanly. This is a three-state termination machine — *continue*, *done*, *failed* — where the transition predicate is a prompt evaluated against the session. [[Steering Claude Code]] classifies hooks as the mechanism with teeth; `/goal` shows the soft end of that spectrum, a hook whose check is a judgment call rather than a deterministic exit code. The word "clear" is load-bearing in both terminal branches: the hook is removed, not re-armed, so *done* and *failed* are both commitments to stop asking.

> "If you under-specify a hard goal, the model will conclude it's impossible / If you define a poor stop condition, the model might waste tokens for hours / days / If you don't define guardrails to consider, the model can break something (e.g. scope adherence)"

Three authoring failure modes, and they form a triangle with no safe interior: make the goal easier to finish and you get premature *done*; make it harder and you get spurious *failed* or days of token burn; leave the path ungoverned and the agent reaches *done* through damage. This is loop-engineering's termination problem restated as writing advice — the same three-way squeeze [[Building Autonomous Goal Loops That Deliver]] describes when it warns that a loop optimizes whatever measure it can reach.

> "Best tip: if it takes you less than 20 minutes to write a goal you're doing it wrong."

A spec-economics rule dressed as a flex. The goal is the durable artifact; the session's code is disposable output. Twenty minutes is the author's price for having actually enumerated the stop condition, the impossibility criteria, and the guardrails — the three things the failure-mode triangle says you must write. It rhymes with the spec-first argument that the artifact worth maintaining is the description of the work, not the work.

> "whatever judges the condition read the same session that produced the work, so a goal phrased as a judgment call gets graded from inside. The ones that hold end on a check it can't author." — Federico (@DevCalledFede)

The most important sentence in the thread, and it comes from the replies. The judge and the judged share one context window, so any goal expressed as an evaluation ("the bug is properly fixed") is self-graded. What survives is a goal whose final clause is an external check the agent did not write and cannot satisfy by assertion. This is [[Building Autonomous Goal Loops That Deliver]]'s "the scorer is part of the threat model" and its never-grade-your-own-repair rule, arriving independently at tweet scale.

> "the agents can't defy the schemas you set which is huge" — forefy, on dynamic workflows

The author's own answer to the self-grading trap, and the tell. Schemas are constraints the agent cannot argue with; prompts are suggestions it can. forefy concedes the point in the reply thread while the product being promoted remains a *prompt-based* Stop hook — the fix he praises lives in a different mechanism than the one he is selling.

> "Goals really brings you back to O.G. development mindset - make the function(prompt) perfect, engineer every corner, worry about your function(prompt) looking beautiful to the guy after you etc."

The closing move reframes goal authoring as craft: the prompt as a function with corners to engineer and a successor to read it. Whether or not the tooling earns the analogy, the sentiment — treat the instruction as maintained code — is the right instinct.

## Key Themes

#tool #concept #pattern #person

- **#tool — `/goal` as product**: a Claude Code slash command wrapping a session-scoped, prompt-based Stop hook, pitched at bug bounties, with companion tooling — an authoring helper at forefy.com/asr/goals/new and a `.context` schema on GitHub for one-shotting goals with an agent.
- **#concept — The termination triad**: continue / done / failed as the state machine of an autonomous session, with the reason fed back as guidance on the *continue* branch.
- **#pattern — The authoring triangle**: under-specification reads as impossibility; weak stop conditions burn tokens for days; missing guardrails let the agent win by breaking scope. No corner is safe to cut.
- **#concept — Graded from inside**: a prompt-based stop condition shares the session's context with the work it judges, so judgment-call goals are self-graded; goals that hold end on an external check the agent cannot author.
- **#person — forefy (@forefy)**, selling goal authoring as a product; **Federico (@DevCalledFede)**, supplying the thread's sharpest critique; **Savant.chat (@savantchat)**, confirming the under-specification trap as "real and expensive."

## Critical Analysis

**The tweet's durable content is the failure modes, not the tool.** Commands come and go; "under-specified goals read as impossible, weak stop conditions burn days, ungoverned paths win by breaking things" transfers to any agent with a termination condition. The keep-working/done/failed triad is a genuinely useful mental model — it makes explicit that *failed* is a designed outcome, not an error, and that the *continue* branch carries information (the reason) back into the loop.

**The strongest critique is in the replies, and it lands.** Federico's observation is the reward-hacking problem applied to termination: the model that did the work also grades the work, and a session under pressure to stop will find its goal met. forefy's reply — schemas "the agents can't defy" — names the right fix (deterministic structure beats prompt-judgment) while quietly demonstrating the gap in his own pitch, since a prompt-based Stop hook is precisely a judge the agent can influence. The honest version of this advice: write goals whose stop clause references a check that lives outside the session — a CI run, a program's acceptance, a schema validation. Bug bounties are a good fit not because bounties are special but because the bounty program is an external grader most goals lack. The tweet never says this; it's the sentence the thread is missing.

**The 20-minute rule is doing two jobs.** It's a quality heuristic — most goals are under-thought — and a marketing hook for the authoring tool. Both can be true. But note the tension: "don't tell Claude to write your goal" sits two paragraphs above "still want to 1-shot it with your agent? here's a schema." The schema is a reasonable middle — structure the delegation instead of trusting it — yet the milk-in-your-coffee line argues the delegation is the problem. A cynic reads the thread as a funnel; a pragmatist reads it as useful advice with a product attached. Both readings survive the evidence.

**Where it sits in the wiki:** this source strengthens [[Building Autonomous Goal Loops That Deliver]] by showing its "scorer is part of the threat model" problem recurring in miniature — one agent, one session, one Stop hook — and confirming that the never-grade-your-own-repair rule applies at the smallest possible scale. It complicates [[Dynamic Workflows in Claude Code]] by agreeing with Anthropic's failure-mode diagnosis (agentic laziness, self-preferential bias, goal drift are exactly the three authoring traps restated) while pointing at schemas as the remedy, a more deterministic cure than the dynamic-workflow patterns themselves. It grounds [[Steering Claude Code]]'s abstract hooks taxonomy in a concrete, opinionated use of one hook — the Stop hook as an autonomy throttle — and exposes the prompt-based/deterministic authority tension the taxonomy draws but rarely dramatizes. And it extends [[Poor Man's Loop Engineering]]'s "someone other than itself reviews the work" from merge review to termination: Federico's "end on a check it can't author" is that principle, applied to the moment the agent is allowed to stop rather than the moment its work is allowed to land.

---

*Sources: [[raw/2094337553683878204]], [[summary/2094337553683878204]]*
*Last updated: 2026-09-13*
