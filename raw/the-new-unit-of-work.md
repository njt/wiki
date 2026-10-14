---
url: https://lexifina.com/blog/the-new-unit-of-work
date_fetched: 2026-10-14
---

# The new unit of work for document collaboration

A prompt that writes, lints, compiles and tests is a new unit of work, which needs a corresponding checkpoint.

## Why coding never had multiplayer

A document can send every keystroke to everyone live and nothing fails. Code does not survive, a half-typed function simply does not compile. So collaboration moved from live editing to a discrete set of steps (commits). A commit keeps the state but loses the reasoning behind it.[1] You work in isolation, push, and collaborate around that (review, merge, resolve conflicts, etc). Software development follows from this system of checkpoints. Pull requests exist because the unit arrives too big to read at once. Some workarounds like stacked pull requests now cut that unit back into layers after the fact, but the premise remains the same.[2]

With an llm, a prompt writes the change, lints it, compiles it and runs the tests before it hands anything back: from outside, one atomic action, with working code on either side. That makes the prompt a checkpoint. It is much smaller than a commit, and much bigger than a keystroke. It is also the first checkpoint with its own explanation. A commit message is written afterwards, from memory; a prompt is written before the work, and the run is recorded, not reconstructed. That explanation is still the agent’s own account: it justifies the change but does not verify it.

A contract is a document that behaves partly like code. Its sentences survive being half-edited; its defined terms and cross-references do not. Change a defined term halfway, or renumber one clause, and the rest of the contract is wrong until the edit is finished. That is why lawyers work alone and send the draft round: an emailed set of redlines is their commit. The email is the push, the comparison is the diff and the reply is the review. Rounds are days apart, so both sides drift before either sees the other: code’s merge-conflict problem.

Agents can emulate this by proposing edits as tracked changes. One run is one set of proposals. Accepting or rejecting one is recorded as the decision of the person who did it, and the proposal still counts as whoever suggested it. Git separates author from committer, but the approver lives in the pull request, never in blame. Here the author, the approver and the reason sit on the same character. Most AI-generated pull requests get no recorded human review;[3] a decision recorded per proposal makes oversight measurable.

So, we now record this agent step. But we never lose the more granular measurement approach: we utilise the two version control systems in conjunction. Reconciling a documents drafting changes one character at a time, as well as via “jump” from semantic redlines.

| Collaborator | Unit of work | Amount | 
|---|---|---|
| A person | Keystrokes, grouped into slices of active time of up to 30 seconds. | Characters they added and removed. | 
| A person operated agent | One run per prompt, from when it started to when it finished. | Tokens and cost of every model call in the run. | 
| A subagent | The same, plus each instruction someone gave it. | The same, plus the tracked changes it proposed. | 

Writes made by the server carry who they were made for, so an agent’s proposal and a person accepting a batch of them do not collapse into one generic “AI”. Sub-runs are counted against the run that started them.

Select text and each part of the selection is listed with its author, the time and version of the edit, and the words it replaced. Git blame only names the last commit to touch a line. For text an agent wrote, it also shows who accepted it and the reason the agent gave. To get there, the indexer replays each paragraph’s records in order, keeping one author per character:

## Utility to agents

Before the agent edits a clause, it is told if a lawyer rewrote its earlier wording there or rejected its earlier suggestions, and it can read the clause’s history with a tool before proposing something similar. This allows it to avoid previous mistakes.

## Open questions

- A prompt is only atomic if its checks pass. A contract has no compiler, so the checks that make a run safe to accept are weaker than lint and tests. We look at downstream “knock-on” effects and dependency trees, though the utility of these depends on the nature of each document. Generally, these are “warn-level” logs.
- Paragraph history is not completely robust to cutting, changing and moving text; lineage reconstruction is a multi-layered best attempt.

## References

- Nathan Sobo, ‘Software Is Made Between Commits’ (*Zed Blog*, 11 June 2026) <zed.dev/blog/introducing-deltadb>
- ‘Stacked pull requests generally available’ (*GitHub Changelog*, 6 October 2026) <github.blog/changelog/2026-10-06-stacked-pull-requests-generally-available/>
- Kacper Duma and others, ‘These Aren’t the Reviews You’re Looking For: How Humans Review AI-Generated Pull Requests’ (EASE 2026, arXiv:2605.02273, 4 May 2026) <arxiv.org/abs/2605.02273>
