# Introducing git-wt — Worktrees Simplified

Ahmed El Gabri's `git-wt` wraps Git's worktree feature into seven commands that feel native -- `clone`, `add`, `remove`, `destroy`, `update`, `switch`, and `migrate`. It's a Bash script masquerading as a Git subcommand that automates the bare-repo setup dance, guarantees upstream tracking, and never leaves orphaned branches behind. The interactive mode uses fzf to present branches with commit previews, and `destroy` requires typing the branch name as confirmation before it nukes the remote too.

---

## Key Quotes

> "Create a worktree from `origin/feature`, your first push fails with 'no upstream configured.'"

The opening diagnosis. Four words that anyone who's used worktrees has felt. Git's worktree plumbing is powerful but user-hostile -- every sharp edge El Gabri names is one the core tools have left unfiled for years.

> "The goal is making worktrees feel as natural as `git checkout` -- but with full isolation."

> "The bare repo pattern is powerful but has sharp edges."

The wrapper philosophy in one line. Don't replace the tool -- smooth it. Commands should "match mental model" rather than requiring a ritual of five git incantations.

---

## Key Themes

#tool #git #developer-tools #cli #shell

The four frictions El Gabri identifies are the same ones that bite every worktree user: no upstream tracking, orphaned branches on remove, stale refs from pre-fetch, and the memorization tax of the bare repo setup. `git-wt` solves all four with a single Bash file.

The `migrate` command is the boldest move -- restructure a live repo in place, converting it to the bare pattern while preserving all branches, stashes, and history. It's marked experimental, but its existence signals ambition beyond "neat wrapper."

The `switch` + shell function pattern (`wt() { cd "$(git wt switch)"; }`) is the moment the tool becomes muscle memory. Interactive fzf picker, single keypress, you're in another branch in another directory. That's the killer feature.

---

## Critical Analysis

This is a good tool that solves a real problem, and I'm annoyed it took until 2026 for someone to build it. Git worktrees have been around since 2015 and the UX has been bad the entire time. The fact that a single Bash file (plus fzf) fixes what core Git hasn't is both a testament to El Gabri's design sense and an indictment of `git worktree`'s stagnation.

The real insight isn't any single command -- it's that worktrees need a **lifecycle**, not just CRUD operations. `clone` → `add` → `switch` → `remove`/`destroy` → `update` maps the full arc of working with branches in isolation. Native Git gives you `add` and `remove` and calls it done.

That said, `migrate` makes me nervous. Restructuring a live repo is the kind of operation where edge cases hide -- submodules, git hooks, work-in-progress, reflog oddities. The experimental label is honest, but `migrate` is also the feature most likely to eat someone's morning. If you use it, push everything first.

For agent workflows, this tool is quietly important. [[Agent of Empires]] builds worktree isolation into its session manager; [[Teaching Claude to QA a Mobile App]] recounts the disaster when an agent escaped its worktree and contaminated the main repo. `git-wt` doesn't solve the escape problem (that's a [[claude-ctrl]] problem), but it makes the correct setup the path of least resistance -- and in agent safety, defaults are everything. See also [[Before Reading Code]] for the five git commands that diagnose codebase health before you read a line.

Compare `git-wt`'s philosophy to the `wt` function in [[MobileVibe]] -- same idea, different ergonomics. Both converge on the same insight: worktree isolation is the right primitive for parallel development, whether the "parallel" is you-on-three-branches or five agents across five features.

GitHub's [[gh-stack]] takes a different approach to the same problem: instead of isolating parallel branches into separate directories, it chains them into a linear stack where each branch's PR is based on the one below it. Where `git-wt` enables breadth (many independent branches), `gh stack` enables depth (one change decomposed into reviewable layers). The tools compose: worktree for parallel stacks, gh-stack within each one.

---

*Sources: [[summary/git-wt]]*
*Last updated: 2026-05-15*
