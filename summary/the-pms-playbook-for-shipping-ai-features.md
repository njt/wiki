---
url: https://www.oreilly.com/radar/the-pms-playbook-for-shipping-ai-features-that-actually-work-in-production/
title: "The PM's Playbook for Shipping AI Features That Actually Work in Production"
author: Gaurav Savla
date_fetched: 2026-06-15
date_published: 2026-06-10
source: O'Reilly Radar
tags: AI, product management, production engineering, evaluation, A/B testing, latency, model drift, prompt engineering
topics:
  - agent-architecture
---

# The PM's Playbook for Shipping AI Features That Actually Work in Production

Gaurav Savla, O'Reilly Radar, June 10, 2026

## The Demo to Production Death Valley

Savla opens by describing the familiar pattern: an exciting prototype that works magically in demo, followed by a brutal reality check when testing before ship. Problems include latency spikes to 10 seconds on mobile, hallucinations on edge cases affecting 15% of real queries, A/B tests showing no significant lift due to variance, and safety teams flagging hundreds of failure cases. He asserts that "most often than not, it's not a model problem but an engineering discipline problem."

## Latency Budgets

Savla explains that LLM inference takes 500ms to 50 seconds depending on model size, input length, and infrastructure. Consumer products demand sub-200ms, creating a "hard constraint you have to design around."

He warns against only measuring p50 latency, giving an example where 800ms p50 masks a p90 of 15 seconds — meaning 10% of users wait 15+ seconds.

**Three interaction types with budgets:**

1. **Synchronous** (user staring at spinner): resolve under 1 second
2. **Progressive** (streaming output): first token under 500ms, full response under 5 seconds
3. **Asynchronous** (user doing other things): up to 20 seconds with progress indicator

He stresses measuring cold starts separately — the first request after model loading can be 10x slower — and budgeting for the full pipeline (preprocessing, inference, postprocessing, delivery), not just inference. He advocates aggressive use of streaming: a four-second response that starts appearing at 300ms "feels dramatically faster" than one that arrives all at once.

## Designing Fallbacks

Savla contrasts traditional software (failing in "boring, predictable ways") with AI (failing in "novel, unpredictable, and occasionally creative ways"). He cites an example where a model responded to a product recommendation query "with a poem about loneliness."

**Four-level fallback hierarchy:**

1. **Model fallback** — drop to a simpler, faster, more reliable model
2. **Cache fallback** — serve cached responses for similar queries
3. **Template fallback** — fall back to prewritten templates when generation fails entirely ("Degraded beats dead every time")
4. **Graceful omission** — don't show the AI feature at all rather than showing a broken version

The core design principle: users should never encounter an unhandled AI failure, and transitions between fallback levels should be invisible where possible.

## Quality Measurement

Savla presents a four-layer quality pyramid:

1. **Safety** (nonnegotiable, binary) — harmful content, PII, made-up facts. Measure with automated classifiers on 100% of outputs.
2. **Factual correctness** (domain-specific) — e.g., code compiles and passes tests for a coding assistant. Measure with domain-specific evaluation suites.
3. **Usefulness** (user-centered) — acceptance rate, edit distance, time to task completion, repeat usage.
4. **Delight** (experimental, hardest to measure) — "Sometimes the numbers say the feature works but users' guts say it doesn't."

## A/B Testing AI Features

Savla notes that nondeterministic AI outputs create an "intratreatment variance" that inflates needed sample sizes by 3–5x. Running AI experiments with normal sample size assumptions means "you're probably looking at noise and calling it signal."

The metric selection problem: a chatbot generating entertaining but wrong responses might show great engagement while misleading users. He recommends measuring "engaged interactions where quality score exceeds threshold" over raw engagement.

The temporal problem: AI feature value changes as users learn to work with it. Short experiments may misestimate long-term value due to learning curves or novelty bumps.

**Practical guidance:** budget 2–3x more time and traffic for AI experiments, use Bayesian methods (better with high variance), and pair quantitative tests with qualitative research — "Ten user interviews will surface failure modes that no amount of statistical analysis will catch."

## Model Drift Monitoring

Savla calls model drift "the slow, invisible rot of AI output quality over time."

**Three drift types:**

- **Data drift** — world changes, user behavior evolves. A 2024-trained model performs worse on 2026 queries with new concepts and slang.
- **Provider drift** — third-party APIs change without consent. He notes OpenAI acknowledged GPT-4's behavior shifted measurably between March and June 2023, with Stanford researchers documenting significant performance swings. Fix: pin model versions.
- **Evaluation drift** — quality metrics themselves become inadequate as usage patterns shift.

**Recommended monitoring cadence:** daily automated evaluations on 1–5% of production traffic, weekly input distribution analysis, and monthly human evaluation of 100–500 examples. He warns that shipping without drift monitoring means "you won't know it's broken until your users tell you, and by then they're angry."

## Evaluation Frameworks

Savla says you need two fundamentally different approaches:

**Automated evaluation** (speed): Build a golden dataset of 500–2,000 labeled examples, train a classifier or use a model-as-judge, validate against human judgment quarterly targeting 85% agreement. The pitfall: they miss novel failure modes not in training data.

**Human evaluation** (catches what automation misses): 5–7 evaluators mixing domain experts and representative users, consistent rubric covering accuracy, helpfulness, tone, completeness, safety. Weekly during development, monthly in production. Trade-offs: $15–$30 per example, 24–72 hour turnaround, subject to human biases. Mitigate by rotating evaluators and capping sessions at two hours.

**Model as judge** — a viable middle ground where a model evaluates outputs even for tasks it couldn't produce itself. Use for high-volume work but always validate against human judgment.

## Graceful Degradation and Prompt Engineering

**Degradation design:** Define 4–5 capability levels with specific behaviors at each. Example for an AI writing assistant — Level 5: full real-time suggestions, tone adjustment, structure recommendations. Level 4: delayed suggestions after 2–3 seconds. Level 3: basic grammar/spelling only, no style feedback.

Make degradation invisible where possible — users see a "less detailed" experience, not a broken one. When degradation is significant enough to notice, proactive communication like "'AI suggestions are temporarily limited'" builds trust more than silently pushing poor outputs.

**Prompt engineering is software engineering:** In production, prompts need version control, testing, monitoring, and maintenance. Parameterize prompts — don't hardcode context. Production prompts should be templates with defined injection points for user context, system state, and dynamic instructions.

Test prompts against regression suites of 200–500 test cases covering expected inputs, edge cases, and adversarial inputs. Run the suite against every prompt change before deployment.

Monitor prompt performance in production by tracking output quality metrics segmented by prompt version. Deploy new versions using canary-style rollouts — compare against the previous version for at least 72 hours before declaring it stable.

## Ship It Right

Savla concludes that these systems "aren't optional add ons you can bolt on after launch." Features built first with plans to "add production hardening later" fail; later never comes. He emphasizes that AI features are probabilistic, nondeterministic, and change over time without anyone touching them. Building these systems requires treating them with the same seriousness as core infrastructure. "The gap between demo and production is wide, but it's absolutely crossable if you build the right bridge."

---

Disclaimer: Research was done in a personal capacity and views are his own, not his employer's.

Post topics tagged: AI & ML
