---
url: https://shapeofthesystem.com/posts/2026/02/03/bounded-cognition
title: Engineering for Bounded Cognition
author: Matt Williams
date_fetched: 2026-07-03
date_published: 2026-02-03
topics:
  - software-engineering-craft
---

# Engineering for Bounded Cognition

Matt Williams, shapeofthesystem.com, 3 February 2026

## The Central Premise

The article opens with a challenge to the famous "seven plus or minus two" figure from George Miller's 1956 paper. Miller himself was skeptical of the number. Later research by Nelson Cowan (2001) revised the honest capacity of working memory—when you strip away all rehearsal and chunking tricks—down to roughly **four items**.

The author then contrasts this with the device you're reading on, which runs "tens of millions of lines of code." The gap between what we build and what we can hold in mind is the foundational problem of software.

## The Fragile Cognitive Instrument

The mind is described as having three limitations simultaneously:
1. **Small capacity**: about four chunks
2. **Narrow attention**: likened not to a floodlight but to a "torch beam in a dark warehouse"
3. **Rapid decay**: citing Peterson & Peterson (1959), unrehearsed information fades within about twenty seconds

The author cites the famous **gorilla experiment** (Simons & Chabris, 1999) where roughly half of observers failed to notice a person in a gorilla suit beating their chest for nine seconds while they focused on counting basketball passes. Even stranger is the **door study** (Simons & Levin, 1998): people giving directions didn't notice their conversation partner had been swapped for a completely different person behind a passing door.

The punchline: "That's the instrument we build software with."

## Rethinking "Human Error"

The article argues that when incident reports blame a person who typed the wrong command or missed a warning, the fault typically lies in the system, not the individual. A warning that roughly half of attentive people will look straight at and not see "isn't really a warning. It's a decoration." Systems designed for a mythical operator—one who never gets tired, never looks away, and holds the entire machine in their head—are "already broken" before they ever fail.

## AI Has the Same Problem

The author extends the argument to large language models, comparing their **context window** to human working memory. Citing Liu et al. (2023)'s "Lost in the Middle" paper, the article notes that LLMs answer well when needed facts are at the start or end of a long input but perform notably worse when facts are buried in the middle. Providing more retrieved documents can actually pull accuracy *below* the baseline of having no retrieved documents at all, because the model's attention is a fixed quantity. The model "loses the thread" in exactly the same way a tired person does.

## The Central Question of Engineering

Since the gap between mind size and system size is permanent and cannot be overcome by cleverness, hiring, or bigger models, the real question shifts:

> "how do we shape the thing so that a small mind can work on it without bringing it all down"

## Concrete Engineering Moves

The article lists several practices that move information from fragile memory into stable structure:

- Giving something a precise name means you no longer have to hold that fact in your head
- Drawing a boundary creates a promise you can stop re-checking
- Writing a test parks a decision somewhere where it cannot fade
- Anything that is undoable grants permission to be wrong

These all take something that would otherwise burden the four-slot mind and "moves it out into the structure, where it stays put while you blink."

## The OXO Good Grips Lesson

The author recounts how Sam Farber designed a better vegetable peeler in 1990 after watching his wife Betsey struggle due to arthritis. The resulting soft, yielding grip became one of the world's best-selling kitchen tools—bought by millions who didn't have arthritis. The principle: "Designing for the most constrained user isn't some charity that the rest of us put up with." It produces a better thing for everyone. The same applies to software: systems made safe for the tired, distracted, or novice engineer, and for the machine that loses context, end up being the system everyone reaches for when their attention narrows.

## Closing

The author positions this essay as the philosophical foundation beneath a larger **manifesto** ("The Shape of the System"). The piece is described as "the idea sitting underneath the whole thing," with the manifesto being the working-out of that idea into concrete moves.

## Sources Referenced

| Source | Key Finding |
|--------|-------------|
| **Cowan 2001** | Working memory limit is ~4 chunks, not 7 |
| **Liu et al. 2023** | LLMs lose information in the middle of long contexts |
| **Miller 1956** | Original "magical number seven" paper |
| **OXO Good Grips 1990** | Constrained-case design benefits everyone |
| **Peterson & Peterson 1959** | Unrehearsed items decay in seconds |
| **Simons & Chabris 1999** | Inattentional blindness (gorilla experiment) |
| **Simons & Levin 1998** | Change blindness (door study) |
