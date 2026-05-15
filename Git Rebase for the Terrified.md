# Git Rebase for the Terrified

Aaron Brethorst's empathetic field guide to git rebase, written from the perspective of an open-source maintainer who's tired of merge commits and wants contributors to stop being scared. The core insight isn't technical — it's psychological: rebase fear is irrational because the nuclear option (delete your clone, re-clone) costs nothing. Once you internalize that, the rest is just muscle memory.

---

## Key Quotes

> "the worst case scenario for a rebase gone wrong is that you delete your local clone and start over"

This is the emotional center of the piece. Most git anxiety comes from treating your local clone as precious state. It isn't. Your remote is the source of truth; your local clone is a scratchpad. This reframe — "you can always nuke it" — is worth more than any specific git command.

> "A merge commit can combine them, but it creates a messy history with interleaved commits"

Brethorst is on the side of linear history as a maintainability practice, not an aesthetic preference. Clean history makes `git bisect` work, makes reverts simpler, and makes code review easier. Merge commits aren't wrong, but they're a tax on everyone who reads the history later.

> "Your job is to decide what the final code should look like"

The clearest explanation of conflict resolution I've seen. Not "your changes vs. their changes" but "you are the editor, the final text is your decision." This reframes conflict resolution from compromise to authorship.

> "Rebasing rewrites commit history. This is fine for feature branches that only you are working on"

The ethical boundary stated plainly. Rewriting shared history is violence against your teammates' work. Rewriting your own feature branch before merge is hygiene.

---

## Key Themes

- **#concept** The safety-net reframe: git anxiety is mostly a misunderstanding of what's recoverable. Your remote is the durable artifact; your local clone is disposable.
- **#pattern** `--force-with-lease` as the only acceptable force-push. It's a safety check, not a preference — it refuses to overwrite work you haven't seen.
- **#tool** `git rerere` (reuse recorded resolution) as compound engineering for conflict resolution: let Git learn from your past decisions rather than re-litigating the same conflict across commits.
- **#pattern** Squash-then-rebase: reduce multi-commit conflict pain by consolidating first, then replaying one commit instead of N.
- **#concept** The rebase-vs-merge divide is really about who bears the cleanup cost. Maintainers prefer rebase because it shifts the burden to the contributor (who has context) rather than the reviewer (who doesn't).

---

## Critical Analysis

Brethorst's piece succeeds precisely where most git tutorials fail: it addresses emotion first, mechanics second. The "for the Terrified" framing isn't clickbait — it's the correct diagnosis. Most people who avoid rebase aren't confused about the commands; they're afraid of losing work. The insight that your local clone is disposable solves that.

Where it falls short: it's a tutorial for the happy path plus one recovery scenario. It doesn't cover interactive rebasing (squashing, reordering, dropping commits), `--autosquash` with fixup commits, or what to do when someone force-pushes to the upstream you're rebasing against. These are real scenarios that terrify people too, and the article's brevity leaves them unaddressed.

The `rerere` recommendation is the most underrated tip in the piece. It's rarely covered in git tutorials despite being trivially simple to enable and solving a genuinely painful experience (re-resolving the same conflict across multiple rebase steps). This is [[Compound Engineering]] applied to version control: add a system rather than endure repeated manual toil.

The article pairs well with [[Before Reading Code]], which covers git commands for diagnostic work. Where Brethorst focuses on the contribution side (clean up before you submit), that page covers the review side (understand before you read). Together they form a practical git hygiene workflow.

---

*Sources: [[raw/git-rebase-for-the-terrified]]*
*Last updated: 2026-05-15*
