# Removing the Noise for Better Prompts

Ken Muse argues that prompts should be audited phrase by phrase: every sentence must contribute a goal, relevant context, a constraint, an expected output, or a verification step — anything else is *noise* that dilutes attention and leaves the model guessing at the definition of success. The essay uses a realistic multi-goal prompt (release notes + dependency checks) to show both failure modes: unrelated goals bundled into one request, and under-specified requests that force the agent to invent policy.

---

The core move is a five-part test applied to every phrase: does it define the goal, provide relevant context, add a constraint, describe the expected output, or explain how to verify the result? If not, it is noise. Muse grounds this in how transformers work — attention doesn't weight every token equally, but the model must process all of them, so irrelevant wording adds ambiguity and makes important instructions less prominent.

The villain examples are the ones every developer has written:

> "Please update the release notes with the changes in this pull request. Please review the package.json to gather the dependencies. Please check dependencies for higher version numbers."

The diagnosis isn't length — "a longer prompt can be precise, and a short prompt can be unclear" — it's that three unrelated goals are bundled with no statement of how they relate. Is the dependency list part of the notes? Update or just report? The model must invent an answer while planning its response.

On politeness, the claim is deliberately modest: "please" usually adds no task-specific information, and in ambiguous prompts it can even be read as "optionally". He cites research finding roughly 5% accuracy loss from excessive politeness in some evaluations, but hedges properly: "removing 'please' does not guarantee a better answer. It simply removes noise."

The sharper half of the essay is the under-specification list. "Check dependencies for higher version numbers" hides at least six unstated decisions — patch/minor/major, prereleases or not, direct vs. transitive, report vs. modify manifests and lockfiles, other versioned assets, and whether to run tests, commit, or open a PR. His conclusion points somewhere the prompting literature often avoids: some of this work shouldn't go to an agent at all. Dependency maintenance is a policy problem with deterministic tools (Dependabot); "use deterministic tools whenever you can."

The rewrite shows the alternative:

> "Update the release notes with summaries of user-visible changes in this pull request. Follow the existing format and style."

Every sentence has a job: source of truth named, audience defined ("user-visible"), repository context implied by the format instruction.

---

**Take:** this is prompt-hygiene boilerplate elevated by two things most versions of the argument miss. First, the separation of *bundled goals* from *under-specified requests* as distinct failures — most "be specific" advice collapses them. Second, the escape hatch to determinism: the best prompt for dependency maintenance is no prompt at all. That said, the essay underplays its own strongest point — the six unstated decisions in the dependency example are exactly what [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] calls the moment an agent makes a decision nobody signed for. Muse treats them as prompt defects; they're really *decision ownership* defects that a better prompt can only paper over.

**Key themes:** #concept #pattern

Related pages:

- [[How I Prompt (Thorsten Ball)]] — strengthens this source's case from the other direction: Thorsten Ball's prompts are pointers plus constraints, which is the "every phrase has a job" discipline applied in practice; where Muse says delete filler, Ball says replace prose with artifacts the reader can fetch.
- [[Prompt Debt]] — nuances this: Muse audits single prompts, but the debt framing asks what happens when noise and unstated decisions accumulate across a project's prompt corpus.
- [[New Rules of Context Engineering]] — complicates the attention argument: Muse treats extra tokens as pure dilution, while the context-engineering literature treats some "irrelevant" context as insurance against retrieval failures — the line between noise and redundancy is less clean than the five-part test implies.
- [[Prompting Claude Fable 5.1]] — model-specific counterpoint: Anthropic's own guidance shows that what counts as noise is model-dependent, so a universal audit checklist can only be a starting point.

---
*Sources: [[raw/removing-the-noise-for-better-prompts]], [[summary/removing-the-noise-for-better-prompts]]*
*Last updated: 2026-09-23*
