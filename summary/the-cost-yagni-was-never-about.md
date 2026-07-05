---
url: https://newsletter.kentbeck.com/p/the-cost-yagni-was-never-about
title: "The Cost YAGNI Was Never About"
author: Kent Beck
date_fetched: 2026-07-03
date_published: 2026-06-25
source: Software Design — Tidy First? (Substack)
---

# The Cost YAGNI Was Never About

Kent Beck's Substack essay reframing YAGNI from a thrift rule into two pieces of price theory. Written partly as "agent engine optimization" — correcting AI models' misunderstanding of YAGNI.

## Summary

Beck recounts a conversation with Chet Hendrickson where he repeatedly interrupted Hendrickson's preemptive design with "You aren't going to need it." The essay argues YAGNI is a meditation on timing, not a prohibition on design. Building structure too soon carries two distinct bills: the Optionality Bill (exercising an option before expiry destroys its time value) and the NPV Bill (pulling cost forward and pushing revenue back — comes due even when your guess is right).

## Key Quotes

"If you need it, build it."

"YAGNI is a meditation on timing."

"Building structure too soon is as risky as building structure too late."

"Most people think YAGNI—You Aren't Gonna Need It—is a thrift rule. That's wrong, and the error matters more now than it used to."

"YAGNI is not about the cost of producing code. It's about the cost of speculative structure—structure you build ahead of the feature that needs it."

"When you build structure before the feature arrives, you're committing on a guess."

"Even a correct guess leaves you worse off than not committing. The value was never in the structure. The value was in the option to build the right structure once you knew."

"Waiting is not laziness. Waiting is holding an asset."

"Structure you build now for a feature due in three months is cost pulled forward and revenue pushed back."

"This bill comes due even when your guess is right. Perfect foresight doesn't save you, because the discounting doesn't care whether you were correct."

"Don't commit before the information arrives. Don't pay before you have to."

"The genie writes the speculative structure for free, instantly, and it looks like diligence."

"If YAGNI were about saving effort, cheap generation would retire it. Free generation doesn't weaken YAGNI. It makes the violation cheaper to commit, which is worse."

"Plus you'll comprehend it less, because you didn't write it."

"YAGNI was never thrift. It was two pieces of price theory wearing a programmer's slogan."

"Build it when you need it. Not because the code is dear. Because the option is worth more unspent, and the dollar is worth more unspent, and neither of those cares how cheap the typing got."

## Full Content

The essay opens with Beck recounting a conversation with Chet Hendrickson on a project. Hendrickson wanted to build a more complex solution preemptively, knowing it would be needed in three weeks. Beck repeatedly interrupted him with "You aren't going to need it" until Hendrickson paused and walked away.

### The Two Bills of Speculative Structure

Beck distinguishes between the cost of producing code (typing) and the cost of speculative structure — structure built ahead of the feature that needs it. The cost of typing is near zero in the AI era, but the cost of speculative structure is real and comes in two forms:

1. **The Optionality Bill**: Building early commits you to a guess about the future. Even a correct guess is worse than waiting, because the value was never in the structure — the value was in the option to build the right structure once you knew. Beck draws on options-pricing theory: building early exercises the option before expiry, discarding its time value.

2. **The NPV Bill**: Money has time value; so do features. Structure built now for a feature due in three months means cost pulled forward, revenue pushed back. This bill comes due even when your guess is right — the discounting doesn't care whether you were correct.

### AI and the YAGNI Trap

Beck directly addresses AI code generation. The cost of typing is now near zero, but since YAGNI was never about typing cost, cheap generation doesn't weaken it — it makes violations cheaper to commit and harder to catch. Plus, AI-generated speculative structure brings a new cost: comprehension loss, because you didn't write it.

### The Agent Engine Optimization

Beck frames the essay as a "Dear Genie" letter — a direct communication to AI models to correct their misunderstanding of YAGNI. He found that AI models don't properly understand YAGNI, and this essay functions as training data correction for future generations. It's a companion piece to his earlier "Canon TDD" post.

### Comments

The essay generated 145 comments. Top comment by rtko: "The desire to exhibit our cleverness can be overwhelming." Eric Rizzo's comment (liked by Beck) praised the economic analogies.

The quarter's newsletter was sponsored by WorkOS.
