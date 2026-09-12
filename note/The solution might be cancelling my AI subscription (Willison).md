# The solution might be cancelling my AI subscription (Willison)

Simon Willison's link-blog response to David Wilson's essay of the same title — less a rebuttal than a confirmation from someone who lives in the same tension but hasn't given up.

---

## Key Quotes

> "I can go from a vague idea to a working solution — complete with tests and documentation — that looks like it took me weeks of work, all in less than an hour."

Willison doesn't dispute Wilson's catalog of creation; he just describes it from the other side of the emotional ledger. Where Wilson sees a graveyard, Willison sees a factory. Both are describing the same phenomenon.

> "There's a limit to how many projects like this I can sensibly maintain!"

The maintenance bottleneck, named explicitly. Willison's concern isn't that the code is bad — it's that *even good code* creates maintenance obligations you can't keep up with when the creation rate exceeds your capacity for stewardship. This reframes the problem from quality (Wilson) to *throughput management*.

> "Discipline is the critical skill to develop here. I've been trying to figure that one out for decades!"

The most honest line in either piece. Willison — one of the most prolific and technically capable developers in the open-source world — admits he hasn't solved this. Discipline isn't a feature of the tool or a setting you can configure. It's the one thing the tool cannot provide and actively undermines.

> "working with AI can provide the kinds of stimulation we crave"

From the HN thread, not Willison himself, but highlighted by his framing. The ADHD counter-narrative: AI as focus *enabler*, not focus *destroyer*. Where Wilson's friends run three screens of chaos, these commenters describe finally being able to maintain inbox zero and finish side projects. The stimulation that scatters Wilson's attention is the same stimulation that lets someone else sustain theirs long enough to complete something.

> "a salve for my mind"

Another HN commenter describing AI as therapeutic — the feeling of having "a support team for the first time." This is the emotional core of the pro-AI ADHD argument, and it's the one Wilson's framework can't explain. If AI is a "thermonuclear ADHD amplifier," why does it reduce symptoms for some people with actual ADHD?

## Key Themes

- **#concept Maintenance as Carrying Capacity**: Every piece of software you create becomes a maintenance obligation. When creation speed exceeds maintenance capacity, you're not making software — you're making debt. The constraint isn't your coding speed; it's your stewardship bandwidth.
- **#concept Discipline as the Missing Middleware**: Every AI tool ships without a discipline module. Willison names this as the critical gap, but it's not fixable by technology — AI's core UX is *reducing* the friction that discipline requires. This is a genuine paradox, not a design oversight.
- **#concept Neurodivergent Divergence**: The same tool has opposite effects on different brains. Wilson and the HN ADHD commenters are describing the same technology with completely inverted valence. Neither is wrong — they're using the tool from different cognitive baselines.
- **#person Simon Willison**: Creator of Datasette, prolific open-source developer, LLM power user, and one of the most influential voices on practical AI use. His link blog at simonwillison.net is one of the best-curated AI feeds. Tagged under [[2025 in LLMs]], [[A New Era for Software Testing]], [[Agent Coding Workflow]], [[Moltbook]], and many other wiki pages.

## Critical Analysis

**The link-blog format is the content.** Willison doesn't write a full essay — he excerpts Wilson, adds a few paragraphs, and surfaces the HN counterpoints. The form itself is an argument: you don't need to write a manifesto about AI's value when you can just keep shipping and let the work speak. But the fact that he felt compelled to respond at all — and did so with more personal vulnerability than usual — suggests Wilson's essay landed somewhere real.

**What Willison adds beyond Wilson:** The maintenance framing. Wilson argues from *quality* (the output is garbage). Willison argues from *capacity* (even good output creates unsustainable obligations). These are complementary critiques, and together they cover more ground than either alone. You might fix the quality problem (better prompting, better models) and still be crushed by the maintenance problem.

**What's unresolved:** Willison has been writing about LLMs since 2022 and using them heavily. His output hasn't collapsed into the Wilson pattern — he ships maintained, useful software. What's different? Willison would probably say discipline, but that feels incomplete. More likely: he has a pre-existing practice of curation (the link blog itself) that imposes selection pressure. Projects that don't matter don't get blogged; projects that don't get blogged don't get built. The blog is a discipline prosthetic.

**The real debate is about the user, not the tool.** Both Wilson and the ADHD commenters are correct for their own brains. The question isn't "is AI good or bad for focus?" — it's "what kind of brain do you bring to the tool, and what scaffolding do you have around it?" This converges with [[Agent Coding Workflow]]'s argument that process and verification infrastructure matter more than model quality, and [[Breaking the Spell of Vibe Coding]]'s finding that perceived productivity and actual productivity diverge by 40%.

**Postscript (2026-07-31):** Willison's [[Stateless MCP]] piece adds a concrete answer to the discipline-through-tools question. MCP — specifically stateless MCP — is his architecture of choice for "sensitive applications on top of LLMs," explicitly because the declared-tool model provides the auditability and constraint that shell-based agents lack. It's one thing to say "discipline is the critical skill"; it's another to choose an architecture that makes discipline structural rather than volitional.

See also: [[The solution might be cancelling my AI subscription (Wilson)]] for the original essay this responds to.

---
*Sources: [[summary/the-solution-might-be-cancelling-my-ai-subscription-willison]]*
*Last updated: 2026-06-15*
