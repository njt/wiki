---
url: https://pydantic.dev/articles/the-human-in-the-loop-is-tired
title: "The Human-in-the-Loop is Tired"
author: Laura Summers
date_fetched: 2026-07-18
date_published: 2026-02-18
site: Pydantic
---

# The Human-in-the-Loop is Tired

**By Laura Summers, published February 18, 2026 on pydantic.dev**

## Central Thesis

LLM-assisted programming is simultaneously useful and destabilizing. These two realities coexist, and ignoring the destabilizing effect leads to burnout.

## "Hands in the fabric" — Background & Context

Summers describes learning to code in her early twenties as a self-taught designer-turned-programmer, recalling the sensation of touching "some deep fundamental layer of abstraction." She contrasts this with the unmet promises of 2010s low-code/no-code tools like Dreamweaver, and argues that this time, the gap between promise and reality has finally narrowed meaningfully — which is what makes it unsettling.

## What "the code writes itself" actually feels like

A key anecdote involves colleague Douwe, who maintains Pydantic AI, waking to 30 PRs generated overnight by AI. The temptation to delegate review to AI raised the existential question: *"at that point, what am I still doing here?"*

Summers describes spending two full days writing a plan for an LLM to execute, only to have it produce "errors of coherence" — smart enough for plausible code, not smart enough to maintain coherent intent across complex changes. This creates a new kind of fatigue: the fatigue of *supervision*, holding intent in one's head while reviewing "mostly-correct output."

Douwe's observation about loss: "everything I write goes into some AI black hole. There's no person on the other side actually learning anything."

## The intensity trap

Summers cites a Berkeley Haas study (highlighted by Simon Willison) showing AI increases work *intensity* — the pull of "one more prompt." She describes staying up until 2am prompting. Colleague Marcelo joked about opening five Claude sessions simultaneously, which Summers says captures something true: you can start vastly more things, but finishing requires the one non-parallelizable resource — your brain.

**The core concept introduced:** "the human reward function problem." Hand-coding provided small, frequent dopamine hits (solving problems, understanding logic, watching code compile). LLM-assisted work replaces those with the cognitive load of review and supervision. The result: "The satisfying part shrank. The exhausting part grew."

She describes the work as "intensely solitary" — natural moments to turn to a colleague get replaced by another prompt. The addictive nature follows a "Skinner Box" pattern, and switching between LLM-assisted and manual work is "jarring and uncomfortable."

## Breakpoints — The Responsive Design Analogy

Summers draws a parallel to the 2009 transition from fixed-width to responsive web design. Designers experienced existential "loss of control" over pixel-perfect layouts. Those who thrived reframed their skills: "The craft didn't die, it evolved." She acknowledges the stakes and pace are different now — the current shift is "measured in months" — but holds that the pattern of *craft evolving rather than dying* still applies.

## What survives

The distinguishing markers in an era of AI-generated code become: "taste, nuance, mature architectural opinions, and the contrarian calls that come from genuine expertise."

She notes that teams succeed most when guiding LLMs in domains they deeply understand. In shallower areas, outputs become "impressionistic" — plausible-looking but less correct.

**Emerging practices:**
- Running "pre-mortems" where a fresh LLM session diagnoses why a complex plan might fail
- A colleague built a tool extracting rules from thousands of past code review comments to seed an `AGENTS.md` file — described as expertise being "distilled" rather than lost

Those finding their footing share: strong opinions earned through practice, ability to distinguish lasting principles from mere bandwidth constraints, and willingness to evolve workflow without abandoning standards.

## A view from inside the loop

Summers rejects the idea that this represents the end of software engineering, but calls it "a serious contraction and a fundamental reshaping of what the work *is*." She identifies three legitimate fears: obsolescence, skill rot, and being left behind.

The core reframe: "the bottleneck was never the code. It was always the human attention, the engineering judgment, the ability to hold a coherent vision for a system." Now that code-writing is automated, those human capacities are revealed as the scarce resource.

## Final thoughts

She closes by affirming that the Pydantic team building these tools is experiencing the same destabilization: "We're debugging our reward functions in real time, same as you." The humans are still in the loop — "We're just tired."
