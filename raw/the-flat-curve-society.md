---
url: https://steve-yegge.medium.com/the-flat-curve-society-36c8b01eb33b
title: The Flat Curve Society
author: Steve Yegge
date_fetched: 2026-06-22
date_published: 2026-06-20
platform: Medium
reading_time: 18 min
---

# The Flat Curve Society

The piece opens with Yegge observing that model intelligence has become dangerous, pointing to Dario's earlier prediction and the US government briefly shutting down "Fable" as a visible sign of crossing into risky territory. He had hoped for a few more generations of upgrades before security became an issue, but the Mythos class — with Fable as a loosely guardrailed release — has spooked everyone.

Yegge argues the AI race won't slow and capability will keep growing exponentially, but "most of you aren't going to see it progress anymore." He now believes we are only two or three model generations away from AI being controlled like nuclear weapons. Only a select few will have access to superintelligence above this year's classes, and it will be supervised.

## Superintelligence Under Lock and Key

He predicts every government will restrict access independently, comparing the chokepoint to enriched uranium for nuclear weapons. The supply chain is what governments can clamp down on. China will lock superintelligence within its borders just as the US will. If China takes the frontier lead, it only shifts where power concentrates, not the overall shape of the world.

## A World of Mediocre Models

Open-source models trail the frontier by roughly seven months and stay on the curve by training on compute that increasingly requires international-relations-level dealmaking. Distillation or peer-to-peer training might keep them in the race, but pushing past Fable class would require doing so while the entire supply chain gets locked down like the nuclear chain was, and frontier labs won't help train the next dangerous open model. If OSS hits Fable class anyway, that's great, but "open models are not going to blow past Fable class."

Yegge concludes today's models are roughly as good as we're going to get. He finds this disappointing in some ways but still full of upside: Fable-class models are good enough to transform coding and knowledge work, though it will take a big, multi-year pivot effort. He assumes for the rest of the post that Fable will return and we may get one higher class before further advancements become inaccessible.

He notes that many people expected the hockey-stick AI curve to level out, predicting AI wouldn't replace human engineers. "In a way, you turned out to be right." Behind the scenes the exponential growth continues, visible in data center growth, but the curve will appear to flatten for two reasons: dangerous models will be kept out of most people's hands, and a second phenomenon he explores.

## A World of Mediocre Users

Some people report they can't tell the difference between Opus 4.8 and Fable 5. Yegge calls this the "discernment horizon": every human has a ceiling on model intelligence past which models feel similar. But there are two ceilings.

The **demand horizon** is set by the hardest problem you bring. If you only have easy problems, a smarter model has no room to pull ahead. He collects "back-pocket evals" — projects a model fails at — and tests each new model against them. For example, no Opus-class model could write the React client for his game; Fable handled it easily. The demand horizon simply means your work isn't hard enough yet.

The **discernment horizon proper** is set by the hardest answer you can judge. Past this line, "you can't tell whether the model is right, because checking the work is itself beyond you." Everyone has a discernment horizon — even Dario. Beyond some capability level, no human can verify model output.

This circles back to why models are being locked down: "You can't hand out an intelligence engine that nobody can supervise." Superhuman means unverifiable. Safety people see a weapon; the rest see a tool they can't effectively supervise. Companies face both horizons — for many, Fable already exceeds their demand horizon, while for harder shops, the binding limit is discernment (ungradeable AI output).

As a result, "the curve is flattening for most of us." Commodity intelligence will soon stop growing exponentially, or at least appear to.

## SaaS is Back, Baby

Yegge argues it's too expensive to rebuild all SaaS at the top of the pyramid. Models that could do it exist, but access and cost are prohibitive. SaaS has already come "rocketing back" after being on the ropes, as companies learned about token efficiency the hard way, with large firms blowing yearly budgets in months. The buy-vs-build decision now tilts heavily toward buy. Vibe-coding replacements could be an expensive gamble.

With a plateau in accessible model capabilities, other AI-in-SaaS dreams fade too — replacing SaaS or transforming it with agentic behavior. Today's models aren't good enough to replace a person (jailbreakable, confusable), and models that could reliably replace humans may be too dangerous to give to most people.

SaaS still has problems — unused features subsidized by users, dollars extracted from local economies, enshittification — but it remains about crystallization of knowledge. "The AI models powerful enough to replace most of that 'easily' will either be unavailable or prohibitively expensive." Yegge feels SaaS is here to stay.

## AI Literacy 101

Today's models are capable but difficult to work with. Even Fable likely struggles with large monoliths and complex legacy code. Efficiency is a monstrous issue. Yegge had hoped for models smart enough that little training is needed, but today's models require help to use effectively.

Why does AI literacy matter? Two factors: companies must pivot to using AI, and employees feel anxious about it, creating a tension loop. Pivoting changes everyone's job, feeding anxiety. Pushing change without addressing literacy — quantitatively and empathetically — fuels resistance.

He calls AI adoption "the key culture challenge of 2026–2027." Once employees get past the hurdle and become excited about using AI, magic happens — they begin reshaping business processes toward supervised agentic flows. He cites Arkana Labs and their VP Eng Owen Parker as an example where employees themselves got excited about opportunities.

## Beginner Cohorts (Netflix Study)

Yegge describes a presentation from Ezra Savard at Netflix, who ran a training study from December through March, presented at Gene Kim's AI Summit in San Jose. The study trained Netflix engineers on agentic coding and measured impact.

Three beginner cohorts emerged, defined by token spend on a "qualified" heavy-use day:

- **0M tokens/day**: developers not using coding agents for regular work
- **4M tokens/day**: using a single agent synchronously throughout the workday
- **12M–15M tokens/day**: letting 2 to 4 agents work without watching

His working definition: if your org isn't at least at single-agent literacy, people will resist bringing in more AI.

Some power users spent over 50M/day. Beyond 15M/day, token spend is no longer a useful measure, since people invent reasons to burn tokens. Up to that point, measuring token spend provides powerful insight into where the organization stands.

People can "jump cohorts in 5 hours" — graduating from AI illiteracy to savviness in that timeframe, and they stay there. 96% of trainees remained in the second cohort for six weeks after the course.

The training formula: teams of 5–10 people with their manager, during regular work hours as blessed company time, bringing actual work, with instructors helping them learn to use agents. Cutting corners — shorter classes, larger audiences, individual opt-in — didn't produce the same results. The multi-agent course is another 5 hours teaching skills to wrangle multiple asynchronous agents.

Impact findings included a large difference in code produced by agentic coders, entirely attributable to additional test code. The course had a large positive impact on productivity.

Yegge recommends starting with an AI literacy audit, then training everyone up to at least the single-agent cohort.

## Advanced Cohorts

After solving the first culture problem (getting people over the FUD hump), the second emerges: "teaching people how NOT to spend tokens." Token efficiency is advanced. Many ways exist for models to steer users wrong; the most efficient coders maximize outcomes for a given token budget.

He shares a joke from Pierre Racz, CEO of Genetec, who prefers to write code by hand and observed he's just "extremely token-efficient." The lesson: if you can trivially do a task by hand, do it. Typing `git push` instead of asking the agent saves roughly 100k tokens each time.

He describes a bell-curve meme where beginner and master do the same thing — here the mastered thing is low token spend. Token spend signals literacy on the way up, then flips to measuring token waste. Beginner cohorts are "absolute token pigs" — that's fine. Encourage exploration. People need to master spending before they can focus on saving.

At the top of the literacy curve, thinking becomes strategic: buy vs. build decisions, routing tasks to the dumbest model that can handle it, building an intelligence-tier router — "the discernment horizon encoded as infrastructure."

## A Craft Needs a Plateau

Yegge summarizes: "We are seeing a plateau in intelligence." It's artificial — the exponential increase continues behind the scenes, gated away. The Mythos graduating class becomes the accepted trade-off between capability and risk. Incremental updates will patch edge-case behavior but nothing like the jumps of recent years.

The plateau is not bad: "A plateau lets us set up a camp and start building." It gives firm footing after unstable ground where everything was obsoleted with each model release. He describes an engineering problem ahead — learning task decomposition and breaking up software monoliths to stay within model limits. Engineers are still needed.

He likes the coming plateau. Stability feels like a precondition for the new craft of building software with smart helpers — a craft that "only gets harder, and more valuable, the weaker your models are." Sonnet-class and Opus-class will remain relevant for years. The models that would obsolete today's hard-won techniques are "evidently too dangerous to give to us anyway."

There is a large engineering effort underway to build the control plane that lets today's models run today's large businesses.

## Train Your Flat-Curvers

The key takeaway: there is a massive AI training and literacy problem ahead, but it's solvable with time and effort. Today's models won't one-shot an entire Fortune 100 code base. They require grown-up human supervision.

Engineers will still be needed. The trends of impromptu 2-pizza teams forming, 2-to-3-person teams being a sweet spot, and roles blurring together will likely continue. "But everyone will need training and time and patience and careful budget management."

He closes: "AI Literacy does not come for free. The only thing you get for free is AI Anxiety." Teaching people to spend tokens is fairly easy. Teaching them to save tokens — that's the new meta.

He signs off saying he'll see readers at the AI Engineer Conference in San Francisco at the end of the month.
