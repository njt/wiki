---
title: "A Non-Anthropomorphized View of LLMs"
author: "halvar.flake (Thomas Dullien)"
date: 2025-07-06
url: https://addxorrol.blogspot.com/2025/07/a-non-anthropomorphized-view-of-llms.html
fetched: 2026-05-14
topics:
  - ai-research-and-models
---

# A Non-Anthropomorphized View of LLMs

Author: halvar.flake (Thomas Dullien)
Date: Sunday, July 06, 2025

## The Space of Words

The author describes tokenization and embedding as mapping words to vectors in ℝⁿ space. Text becomes "a path through this space" with sequential words. Using a game-like analogy, he compares it to Snake played in high-dimensional space, where the tail truncates as new words appear within a context length constraint.

An LLM with fixed randomness functions as a mapping from (ℝⁿ)^c to (ℝⁿ)^c, calculating probabilities and randomly selecting the next point. The author suggests these generated paths resemble "strange attractors in dynamical systems."

## Learning the Mapping

LLMs are trained using human-written text, expert corpora, and automatically generated validated content to mimic human language patterns.

## Paths to Avoid

Certain language sequences warrant avoidance because they mirror undesirable aspects of empirical human writing. The challenge: designers cannot mathematically specify "undesirable" content precisely, so they use examples and counterexamples to nudge distributions away from problematic outputs.

## "Alignment" for LLMs

The author defines proper alignment as: "we should be able to quantify and bound the probability with which certain undesirable sequences are generated." For any fixed model and sequence, the generation probability is calculable, but integrating across all undesirable sequences remains computationally intractable. The core problem becomes mathematical and computational rather than philosophical.

## The Surprising Utility of LLMs

Modern LLMs solve previously intractable problems. Natural language processing has largely been solved. Users can request document summaries in JSON format, generate children's stories with illustrations -- tasks that seemed impossible years ago. The improvement curve remains steep, with more currently-intractable problems becoming solvable.

## Where Anthropomorphization Loses the Author

The author becomes "lost" when people attribute "consciousness," "ethics," "values," or "morals" to learned mappings. He emphasizes these are "big recurrence equations" requiring external input to generate words. Wondering if LLMs will "wake up" seems equally bizarre as asking if a meteorological simulation might gain consciousness.

The author expresses bewilderment at AI discourse treating "a function to generate sequences of words" as human-like. Statements about "AI agents becoming insider threats" simultaneously make sense (randomized generators can produce anything) and confound (why anthropomorphize dice as conspiratorial).

Using anthropocentric concepts like "behaviors," "ethical constraints," and "harmful actions in pursuit of goals" muddies thinking and obscures actual problems. Historical parallels: humanity blamed earthquakes and famines on divine wrath rather than natural causes.

The clearer framing: think of LLMs as functions generating sequences, where context prefixes steer probabilities through word-space. For any undesirable output sequence shorter than context length, one can identify inputs maximizing its probability.

## Why AI Luminaries Tend to Anthropomorphize

The author suggests self-selection bias: many prominent AI researchers entered the field believing they might create AGI -- "creating a god," something life-like and superior. They're unlikely to abandon the anthropomorphic worldview underlying their career choice.

## Why Human Consciousness Isn't Comparable to LLMs

Humans represent fundamentally different entities than (ℝⁿ)^c mappings. Hundreds of millions of years of evolution produced human thought through countless iterations where only variants survived. Human cognition involves "enormously many neurons, extremely high-bandwidth input, an extremely complicated cocktail of hormones, constant monitoring of energy levels, and millions of years of harsh selection pressure."

Humans remain poorly understood. Given a person and word sequence, one cannot assign meaningful probability to specific outputs. Applying human concepts like "ethics," "survival instinct," or "fear" to LLMs parallels absurdly asking about "feelings of a numerical meteorology simulation."

## The Real Issues

Despite never reaching AGI, current LLM deployment will dramatically reshape society -- comparable perhaps to electrification. The author's grandfather (1904-1981) witnessed transformations from gas lamps to electric power, horse carriages to automobiles, nuclear technology, transistors, and computers, alongside wars and ideological upheavals.

Managing upcoming decades' dramatic changes while avoiding world wars and murderous ideologies remains difficult without "muddying our thinking" through anthropomorphic framings.
