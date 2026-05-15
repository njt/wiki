# When the Target Keeps Moving

Alistair Croll's framework for tracking software projects where AI accelerates both delivery and discovery. The key insight: LLMs get you to the hard part sooner, and the hard part is learning what you don't know yet. In his project, a 32-day sprint started with 254 tasks at 55% complete and ended with ~1,400 active tasks at 89.8% complete -- for every 100 tasks completed late-stage, roughly 109 new ones appeared. Track the ratio of discovery to delivery to know whether you're converging or diverging.

---

## Key Quotes

> "A project isn't a pile of known tasks being shovelled from 'not done' to 'done.' It's also a discovery engine."

> "Cheap, uncomplaining coders turn design into discovery."

> "Good scope growth makes the project more truthful. Bad scope growth makes it less focused."

## Key Themes

#discovery-vs-delivery #task-tracking #management

### The Discovery/Delivery Distinction

Croll introduces "discovery velocity" -- how many necessary tasks become visible per unit time. Traditional metrics (tasks completed, percent done) mislead when discovery outpaces delivery. The critical distinction is between scope creep (bad -- reduces focus) and scope discovery (good -- increases truthfulness about what's actually required). Seven different task types required different responses, each revealing something about the system's true complexity.

### How AI Changes the Curve

AI agents fundamentally alter the development curve by:
- Lowering decomposition costs (vague concerns become countable tasks)
- Increasing parallel inspection across multiple surfaces
- Blending planning and implementation phases
- Creating measurement ambiguity around developer productivity

### The Last 10% Isn't Features

By April 30, 64% of remaining tasks involved operational readiness, agent instruction, human testing, simulation analysis, and load validation -- not product features. Late-stage work is fundamentally different from early work.

### Five Dashboard Metrics

Rather than simple percent-complete, track:
1. **Completed active tasks** -- how much have we done so far?
2. **Active remaining tasks** -- how much is left?
3. **Daily task additions by discovery type** -- how much are we adding, and what kind?
4. **Daily task delivery** -- what did we get done today?
5. **The discovery/delivery ratio** -- is the ratio improving?

### Three Development Phases

1. **First useful thing**: Optimize for end-to-end paths; track early wins
2. **Deliberately increase discovery**: Send agents/tests searching for gaps; expect backlog growth
3. **Denominator stabilization**: Only then become predictive; track whether discoveries are launch-blocking or deferrable

This pairs naturally with [[kata]] for task tracking (each kata issue could be tagged as discovery or delivery) and with [[Spec-Driven Development]] (specs are the artifacts of discovery, code is delivery). [[Zero Alignment]] makes the complementary argument: when delivery is cheap, the hard question is "should we build it?" -- which is a discovery question.

The framework also connects to [[Trycycle]]'s plan-strengthen-review loop: each review round is discovery, and the ratio of review rounds to code changes tells you whether the plan is converging.

## Critical Analysis

The data from Croll's own project makes this more than a framework -- it's an empirical observation. 254 tasks becoming 1,400 over 32 days, with 1,352 commits and 150 topic branches, shows exactly how AI agents surface hidden work. The "of the work currently visible and not retired, 10 percent remains" reframing of "90% complete" is sharp and immediately useful for anyone reporting to stakeholders.

The five metrics are simple, measurable, and any team using a task tracker could start computing the discovery/delivery ratio today. The framework correctly identifies that AI doesn't eliminate unknowns -- it just surfaces them faster. The three-phase model (first useful thing, deliberate discovery, denominator stabilization) gives teams a practical playbook for when to expect chaos and when to start trusting estimates.

The limitation: "discovery type" could use more concrete taxonomy. What categories matter? Requirement changes vs. technical unknowns vs. integration surprises? The article mentions seven types but doesn't fully specify them. But as a conceptual frame backed by real project data, this is one of the sharpest takes on AI-era project management.

---
*Sources: [[raw/when-the-target-keeps-moving]]*
*Last updated: 2026-05-14*
