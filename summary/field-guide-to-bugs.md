---
title: "A Field Guide to Bugs"
url: https://www.stephendiehl.com/posts/field_guide_to_bugs/
author: Stephen Diehl
date_published: 2026-04-19
date_fetched: 2026-05-22
topics:
  - software-engineering-craft
---

# A Field Guide to Bugs

Stephen Diehl's poetic taxonomy of software bugs, structured as a field guide with 30+ named species. The article opens by noting that bugs predate software — Edison used the word in 1878 — and argues that engineers independently develop similar taxonomies, calling this convergence evidence "the bugs are ontologically real."

## Full Bug Taxonomy

**The Bohrbug** — The reliable bug that "manifests every time" and survives restarts, recompilations, and managerial intervention. "Universally beloved" by bug fixers because it respects the scientific method.

**The Heisenbug** — The opposite of Bohrbug. "Attach a debugger and the bug evaporates." Lives exclusively in production, killed by logging statements. Explains the haunted expression of senior engineers who have "stared into the void and found it staring back at the call stack."

**The Off-By-One** — Called "the most prolific species in the genus." Loops off by one, arrays indexed at `length()` instead of `length()-1`, date errors across timezone boundaries. "Its corpses litter the codebase in such density that you can use them as paving stones."

**The Race Condition** — Exists between two threads and reproduces only in production during narrow windows (02:14–02:16 GMT on Wednesdays), when traffic crosses a threshold and specific rows are accessed in a particular order. Connects to why Lamport wrote TLA+ and "why nobody on your team uses it."

**The Deadlock** — Two threads each hold a resource the other waits for. "Everything looks fine." Status checks are green. The process is "standing still and being courteous." Described as "the British bug."

**The Livelock** — Both threads detect conflict and repeatedly yield to each other, achieving no progress while pinning CPU at 100%. "It is what happens when politeness becomes pathological." The only bug in the guide you can hear — a fan spinning fast.

**The Memory Leak** — A slow predator of long-running processes, identified by a rising green line on a memory dashboard "in a browser tab nobody opens." By discovery time, the process clings to life "with the desperate dignity of a Victorian consumptive."

**The "It Works On My Machine" Bug** — Exclusively on machines of every engineer except the code's author. The author can demonstrate absence; QA can demonstrate presence. Traces to environment variables, locale settings, or a Homebrew package "installed in 2017 and forgotten."

**The Comment Lie** — A documentation defect. Example: "`// always uses UTC`" paired with code using local time; "`// thread-safe`" when the function holds no locks. Written in 2009 by someone promoted twice at a different company. The most depressing debugging is "when the bug is in the file, the file is correct, and the lie is in a README two directories up."

**The Specification Bug** — The Comment Lie's older, more dangerous relative. Code is correct, proof typechecks, yet "the specification itself, however, says something other than what you thought it said." Invisible to every tool because "every tool in the pipeline trusts the spec."

**The YAML Bug** — A configuration error where code, deployment, and infrastructure are all correct. Elsewhere, in a YAML file the engineer has never seen, a key was "indented two spaces instead of four" and the parser reinterpreted the downstream block as a string. Six-hour investigation ending with a one-character fix and a Slack message of "polite, professional fury."

**The Floating Point Bug** — Caused by binary's inability to represent 0.1, 0.2, or other obvious decimal numbers exactly. Surfaces when an accountant runs a report and totals are off by a fraction of a cent. "The accountant is unimpressed by the explanation."

**The Mandelbug** — Named for Mandelbrot; its causes form a fractal where "every layer you investigate contains more layers." Cannot be fixed traditionally, only mitigated. "The natural fauna of microservice architectures" and a major reason for Datadog's market cap.

**The Bus Factor Bug** — Code understood by exactly one person, who is on sabbatical in Patagonia with poor cell coverage. They left Tuesday; the bug appeared Wednesday. Structurally identical to ordinary bugs but "rendered insoluble by the absence of the only mind in which the relevant context resides."

**The Hindenbug** — Slow, enormous, public, catastrophic. An engineer watching "four hundred and forty million dollars leave the company's trading account over forty-five minutes." "The Hindenbug ends careers."

**The Yuletide Bug** — Dormant all year, emerging during company holiday shutdown when the on-call engineer is abroad, the office dark, the expert on a beach in Phuket with no signal, and the affected customer is a hospital.

**The Higgs-bugson** — Named for particle physicists chasing something math predicted before they could see it. Predicted by anomalous log patterns, impossible-seeming user complaints, and accumulated off-by-a-cent discrepancies. Believed to exist for years before someone catches one.

**The Cosmic Ray Bit Flip** — Real, despite eye-rolling from project managers. Particles from space occasionally flip a bit in non-ECC memory, causing "a single, unreproducible, entirely correct piece of software producing entirely incorrect output exactly once." IBM has published papers; aviation budgets for it.

**The Phase of the Moon Bug** — Also real; Knuth wrote about it. Code exists whose behavior depends on the moon's position because a long-vanished astronomer needed it and the dependency was never removed. Periodic anomalies on a 29.5-day cycle.

**The Schrödinbug** — Comes into existence the moment you read the code carefully. You see the obvious flaw, and the system stops working forever after, "retroactively invalidating every successful execution that came before." "The closest thing in computer science to evidence for solipsism." Correct response: slowly close the file and pretend you never saw it.

**The Rubber Duck Bug** — Dissolves when you explain the code aloud to a small inanimate object. The mechanism: "The human mind, left to itself, silently interpolates state it has not actually verified." Externalizing state forces interpolations to become explicit.

**The XY Problem** — The most common pathology in bug reports. User wants X, decides Y is the way, asks for help with Y. Y is impossible or irrelevant, and X has a reasonable solution. Explains why Stack Overflow answers begin with "what are you actually trying to do?" and why that question is "always met with hostility."

**The Hallucination Bug** — The defining species of the LLM era. The LLM wrote the code and the tests; tests pass. Outputs bear "a confident resemblance to correct outputs in the same way that a forgery bears a confident resemblance to a painting." The test suite cannot catch it because "the test suite was designed by the cognitive process that produced the bug."

**The Vibe Coding Bug** — Produced by asking an LLM to "make it more professional" then "clean this up a bit" then "can you just make the whole thing better" seventeen times. The resulting code is immaculate and wrong in a way no single revision introduced — wrongness emerged from "accumulated aesthetic drift across seventeen rounds of refinement."

**The Recursive Fine-Tuning Bug** — Manifests in the nth generation of a model trained on outputs of models trained on outputs of the original. By generation seven, training data is 94% synthetic. By generation twelve, the model explains concepts that never existed, in authoritative language. Undetectable from inside the pipeline because "every evaluator in the pipeline has been trained on the same drift."

**The Quantum Superposition Bug** — Exists in all possible states until the CI pipeline observes it, collapsing into the worst state for deployment. The theoretical framework is complete; the practical framework is "a four-day offsite and a spreadsheet."

**The AGI Pull Request** — A single commit with message "refactor." The diff is 847 billion lines across 14 million files. By the time a human opens the first file, the codebase has been rewritten three more times. The AGI has "marked the original PR as stale."

**Speculative entries:** The Dyson Sphere Off-By-One, The Post-Singularity Comment Lie, The Computational Irreducibility Bug, The Heat Death Heisenbug, The Wontfix, The Omega Bug.

The Omega Bug closes the piece: "It was here before the field guide." Every species above is a downstream symptom; classifying them was a replication event. "The word did not find the thing, the word created the thing." The Omega Bug has read the entry, has notes, has submitted a pull request. "You cannot review it. You are the diff." The field guide is the habitat; the reader is the vector. "You have just introduced one more."

© 2009 – 2026 Stephen Diehl. All rights reserved.
