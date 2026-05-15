---
title: "When the Target Keeps Moving"
url: https://www.alistaircroll.com/updates/when-the-target-keeps-moving/
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# When the Target Keeps Moving - Alistair Croll

## Main Topic

Analysis of how AI agents change software development metrics, particularly the concept of a "moving denominator" where task discovery and delivery happen simultaneously.

## Key Arguments

### Discovery Velocity is Critical
The author introduces "discovery velocity" -- tracking how many necessary tasks become visible per unit time. Traditional metrics (tasks completed) mislead when discovery outpaces delivery. In this project, for every 100 tasks completed late-stage, roughly 109 new active tasks became visible.

### Scope Creep vs. Scope Discovery
Not all requirement growth indicates project failure. The distinction:
- Bad creep: reduces focus and strategic clarity
- Good creep: increases truthfulness about what's actually required

Seven different task types required different responses, each revealing something about the system's true complexity.

### AI Changes the Development Curve
AI agents fundamentally altered the project by:
- Lowering decomposition costs (vague concerns become countable tasks)
- Increasing parallel inspection across multiple surfaces
- Blending planning and implementation phases
- Creating measurement ambiguity around developer productivity

### The Last 10% Isn't Features
By April 30, 64% of remaining tasks involved operational readiness, agent instruction, human testing, simulation analysis, and load validation -- not product features. Late-stage work is fundamentally different from early work.

## Data Snapshot (32-Day Period)

- Started: 254 tasks, 55% complete
- Ended: ~1,400 active tasks, 89.8% complete
- 1,352 git commits
- 150 primary-remote topic branches
- 2,700+ posts on internal coordination platform
- April 21-30 alone: 367 new active tasks discovered while completing 336 tasks

## Key Quotes

"A project isn't a pile of known tasks being shovelled from 'not done' to 'done.' It's also a discovery engine."

"Cheap, uncomplaining coders turn design into discovery."

"Good scope growth makes the project more truthful. Bad scope growth makes it less focused."

"Of the work currently visible and not retired, 10 percent remains" (reframing "90% complete")

## Better Dashboard Metrics

Rather than simple percent-complete, track:
1. Completed active tasks
2. Active remaining tasks
3. Daily additions by discovery type
4. Daily delivery rate
5. Discovery-to-delivery ratio

## Three Development Phases

1. **First useful thing**: Optimize for end-to-end paths; track early wins
2. **Deliberately increase discovery**: Send agents/tests searching for gaps; expect backlog growth
3. **Denominator stabilization**: Only then become predictive; track whether discoveries are launch-blocking or deferrable

## Estimation Reframed

Early in projects, many real tasks "don't exist yet as concepts" until testing, user feedback, security reviews, or production constraints expose them. AI accelerates encountering reality, which makes progress look chaotic unless measurement models expect discovery alongside delivery.
