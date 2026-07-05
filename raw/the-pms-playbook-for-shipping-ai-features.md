---
url: https://www.oreilly.com/radar/the-pms-playbook-for-shipping-ai-features-that-actually-work-in-production/
date_fetched: 2026-07-05
backfilled: true
---

**The demo to production Death Valley**

If you’ve worked on an AI feature, you know the feeling. You start building something that you are excited about, set launch timelines. The model spits out a perfect response, the prototype works magically, and everybody in the room is mentally calculating how big this product will be when we launch. I’ve been in that room a lot many times and it’s fun.

Then you try to test before you ship.

Latency spikes to 10 seconds on mobile. The model starts hallucinating on edge cases that happen to represent 15% of actual user queries. Your A/B test shows no statistically significant engagement lift because the variance in AI outputs makes traditional hypothesis testing basically meaningless. The safety team flags 340 failure cases in the first week, and you’re now debugging nondeterministic cases that fail in creative, novel ways every single day.

Most often than not, it’s not a model problem but an engineering discipline problem. Shipping an AI product is very different from traditional software. I’ve figured this out the hard way. This playbook shares my learnings.

**Latency budgets**

Every AI feature comes with a latency tax. Large language model inference takes time. We’re talking 500 milliseconds to 5 or even 50 seconds depending on model size, input length, and infrastructure setup. For consumer products where people expect sub-200-millisecond interactions, this is a hard constraint you have to design around.

The mistake I see most often is teams measuring only p50 latency. A feature with 800 milliseconds p50 sounds fine until you discover the p90 is 15 seconds. That means 10 in every 100 users sit there waiting for 15+ seconds. At scale, that’s thousands of terrible experiences per day.

The way I think about it is you define your latency budget by interaction type, not globally: **Synchronous interactions**, where the user is staring at a spinner, need to resolve under 1 second.** Progressive interactions**,** **where output streams token by token, need first token in under 500 milliseconds and full response under 5 seconds. **Asynchronous interactions**, where the user keeps doing other stuff, can take up to 20 seconds with a progress indicator.

You also need to measure cold starts separately. The first request after a model loads into memory can be 10 times slower than subsequent requests, and if your traffic is bursty, cold starts will disproportionately punish your most engaged users arriving during peak hours.

Besides, you also need to budget for the full pipeline, not just inference. A typical AI feature pipeline including input preprocessing (tokenization, context assembly, and prompt construction), model inference, output postprocessing (parsing, formatting, safety filtering, etc.), and a full response delivery adds up. Optimizing inference while ignoring the rest is like tuning your engine while driving on flat tires.

Lastly, use streaming aggressively for generative features. Pushing tokens to the user as they’re generated instead of waiting for the full response changes how users perceive latency. A four-second response that starts appearing at 300 milliseconds feels dramatically faster than one that pops in all at once. Perception is reality when it comes to user experience.

**Designing fallbacks**

Traditional software fails in boring, predictable ways. AI features fail in novel, unpredictable, and occasionally creative ways. I once saw a model respond to a product recommendation query with a poem about loneliness. Your fallback strategy needs to be considerably more sophisticated than a try/catch block.

I think about fallbacks as a hierarchy. First, model fallback: When your primary model fails, drop to a simpler, faster, and more reliable model. Most failure cases get handled without the user ever knowing. Second, cache fallback: For queries similar to stuff you’ve seen before, serve a cached response. Third, template fallback: When generation fails completely, fall back to prewritten templates. Degraded beats dead every time. Fourth, graceful omission: Sometimes the best fallback is to simply not show the AI feature at all rather than showing a broken version.

The design principle underneath all of this is that users should never encounter an unhandled AI failure. Every failure mode maps to a specific level, and transitions between levels should be invisible whenever you can manage it.

**Quality measurement**

Quality in traditional software is binary. The button works or it doesn’t. AI feature quality is continuous and subjective, and it changes depending on context. I’ve landed on a four-layer quality pyramid.

The foundation is safety, and it’s nonnegotiable. Does the output contain harmful content, PII, or made-up facts? This layer is binary, and you measure it with automated classifiers running against 100% of outputs.

The second layer is factual correctness, which is domain specific. Is the output actually right? For a coding assistant that means generated code compiles and passes tests. For a writing tool it means grammatical, stylistically appropriate output. You measure this with domain specific evaluation suites.

The third layer is usefulness, and it’s user centered. Did the person actually benefit? Track acceptance rate, edit distance, time to task completion, and repeat usage. This is where traditional product metrics meet AI specific ones.

The fourth layer is delight, which is experimental. Does the output feel good? Hardest to measure but often most important for adoption. Sometimes the numbers say the feature works but users’ guts say it doesn’t. This layer catches that gap.

**A/B testing AI features**

A/B testing AI features is fundamentally harder than traditional features because AI outputs are nondeterministic. The same user doing the same thing twice might get different outputs, introducing variance that traditional frameworks weren’t built to handle.

The core challenge is that intratreatment variance inflates the sample size you need for statistical significance, often by three to five times. If you’re running your AI experiment with normal sample size assumptions, you’re probably looking at noise and calling it signal.

Then there’s the metric selection problem. A chatbot generating entertaining but factually wrong responses might show amazing engagement numbers while actively misleading users. You have to measure engagement and quality together. “Engaged interactions where quality score exceeds threshold” is more meaningful than raw engagement alone.

The temporal problem matters too. AI feature value changes over time as users learn how to work with it. Short experiments will underestimate long-term value if there’s a learning curve, or overestimate it if there’s a novelty bump.

My practical guidance: budget two to three times more time and traffic for AI experiments than traditional ones. Lean on Bayesian methods as they handle high variance better. And always pair quantitative tests with qualitative research. Ten user interviews will surface failure modes that no amount of statistical analysis will catch.

**Model drift monitoring**

Model drift is the slow, invisible rot of AI output quality over time, and there are multiple culprits.

Data drift happens because the world changes and user behavior evolves. A model trained on 2024 data performs worse on 2026 queries referencing new concepts, slang, and cultural moments.

Provider drift happens because third-party APIs change without your consent. OpenAI acknowledged that GPT-4’s behavior shifted measurably between March and June 2023, and Stanford researchers documented significant performance swings. The fix: Pin your model versions so updates happen on your schedule, after your testing.

Evaluation drift is the subtlest form. Even your quality metrics can become inadequate and the evaluation criteria that made sense at launch might become inadequate as usage patterns shift and user expectations change. Quarterly reviews of your evaluation suites are essential.

At minimum you need daily automated quality evaluations on 1% to 5% of production traffic, weekly analysis of input distribution characteristics, and monthly human evaluation of 100 to 500 examples. Shipping an AI feature without drift monitoring is like deploying a service without alerting. You won’t know it’s broken until your users tell you, and by then they’re angry.

**Evaluation frameworks**

How do you know if your AI feature is good enough? You need two fundamentally different approaches, and you genuinely need both.

Automated evaluation gives you speed. Build a golden dataset of 500 to 2,000 labeled examples, train a classifier or use a capable model as judge, and validate against human judgment quarterly targeting 85% agreement. Automated evals chew through thousands of examples per hour, making them essential for velocity. The pitfall: They miss novel failure modes not in the training data.

Human evaluation catches what automation misses. Structure it with five to seven evaluators mixing domain experts and representative users. Use a consistent rubric covering accuracy, helpfulness, tone, completeness, and safety. Run weekly during development, monthly in production. The trade-offs: expensive at $15 to $30 per example, slow with 24 to 72 hour turnaround, and subject to human biases. Manage by rotating evaluators and capping sessions at two hours.

The model as judge approach is an increasingly viable middle ground. Judging quality is often easier than generating it, which means a model can reliably evaluate outputs even for tasks where it couldn’t produce them itself. Use it for high-volume evaluation but always validate against human judgment.

**Graceful degradation and prompt engineering**

Graceful degradation means when capabilities decrease, the experience gets worse smoothly instead of falling off a cliff. Design for capability levels, not binary states. Define four to five levels with specific behaviors at each. For example, for an AI writing assistant: Level 5 is full capability with real-time suggestions, tone adjustment, and structure recommendations. Level 4 is delayed suggestions appearing after a two- to three-second pause because latency is up. Level 3 is basic suggestions only like grammar and spelling with no style feedback. Each level is a deliberate design decision, not an accident.

Make degradation invisible when possible. Users shouldn’t see a “broken” experience. They see a less detailed one. That’s a huge difference psychologically. However, when the degradation is significant enough that users will notice, proactive communication like “AI suggestions are temporarily limited” builds trust infinitely more than silently pushing poor-quality outputs.

Prompt engineering in production is software engineering. In production, prompts are code, and they need version control, testing, monitoring, and maintenance. Version controls every prompt. Parameterize prompts, don’t hardcode context. Production prompts should be templates with clearly defined injection points for user context, system state, and dynamic instructions. This makes them testable because you can inject known inputs and verify outputs, and it makes them maintainable because changing how you handle context shouldn’t require rewriting the entire prompt from scratch.

Test prompts against regression suites. Maintain 200 to 500 test cases covering the full distribution of expected inputs, including edge cases and adversarial inputs. Run the suite against every prompt change before deployment.

Monitor prompt performance in production. Track output quality metrics like acceptance rate, user edits, and regeneration requests, segmented by prompt version. When you deploy a new version, compare its production metrics against the previous one for at least 72 hours before calling it stable. This is basically canary deployment for prompts.

**Ship it right**

These systems aren’t optional add ons you can bolt on after launch. Every feature I’ve seen fail was built first with plans to “add production hardening later.” Later never comes.

AI features are probabilistic and nondeterministic, and they change over time without anyone touching them. Build these systems, staff them properly, and treat them with the same seriousness you’d give your core infrastructure. The gap between demo and production is wide, but it’s absolutely crossable if you build the right bridge.

*Note: The research work pertaining to this article was done in a personal capacity. Views are of my own and do not reflect my employer’s views in any way.*
