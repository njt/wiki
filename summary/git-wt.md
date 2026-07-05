---
url: https://gabri.me/blog/git-wt
title: "Introducing git-wt: Worktrees Simplified"
author: Ahmed El Gabri
date_fetched: 2026-05-15
date_published: 2026-02-04
---

# Introducing git-wt: Worktrees Simplified

Ahmed El Gabri introduces `git-wt`, a Bash-based wrapper tool (built as a custom Git command) that simplifies Git's worktree functionality -- particularly the "bare repo pattern" he described in a previous post. The tool is open source and hosted on GitHub.

## The Problem with Native Worktrees

Four key friction points are identified:

1. **No upstream tracking** -- "Create a worktree from `origin/feature`, your first push fails with 'no upstream configured.'"
2. **Orphaned branches** -- Removing a worktree leaves behind the local branch.
3. **Manual fetching** -- Creating a worktree without a prior fetch means working with stale refs.
4. **Setup dance** -- The bare repo structure requires memorizing "10 commands or dig through notes."

## Commands and Features

| Command | Purpose |
|---|---|
| `clone` | Automates bare structure setup |
| `migrate` | Converts existing repos while preserving work |
| `add` | Creates worktrees with auto-fetch and upstream config |
| `remove` | Deletes worktree + local branch |
| `destroy` | Removes worktree + local branch + remote branch |
| `update` | Fetches all remotes, fast-forwards default branch |
| `switch` | Interactive fzf-based picker to navigate between worktrees |

Standard `git worktree` subcommands (`list`, `prune`, `lock`) pass through unchanged.

## Code Examples

**Cloning with structure:**
```
git wt clone https://github.com/user/repo.git
```
Resulting directory layout:
```
my-project/
├── .bare/            # all git data lives here
├── .git              # pointer file
└── main/             # worktree: main branch
```

**Adding a worktree (interactive):** The single command `git wt add` fetches first, then presents branches in fzf with a commit preview.

**Adding a worktree (direct):**
```
git wt add feature-auth origin/feature-auth   # existing branch
git wt add -b my-feature my-feature           # new branch
```

**Removing a worktree:**
```
git wt remove feature-payments
```

**Nuclear option:**
```
git wt destroy old-experiment
```
Requires typing the branch name as confirmation.

**Updating:**
```
git wt update
```

**Switching worktrees with a shell function** for `.bashrc`/`.zshrc`:
```sh
wt() {
    local dir
    dir=$(git wt switch)
    [[ -n "$dir" ]] && cd "$dir"
}
```

## Migration (Experimental)

Marked with a warning: "This command restructures your repository." Running `git wt migrate` inside an existing normal clone converts it to the bare structure while preserving branches, stashes, and history. The current working directory becomes a worktree.

## Key Quote

> The goal is making worktrees feel as natural as `git checkout` - but with full isolation.

## Installation

- **Homebrew:** `brew tap ahmedelgabri/git-wt && brew install git-wt`
- **Nix:** `nix run github:ahmedelgabri/git-wt` or as a flake input
- **Manual:** Copy the single Bash file into `$PATH` (dependencies: git, standard Unix tools, fzf for interactive mode)

## Shell Completion

Zsh, Bash, and Fish completions are included; installed automatically via Homebrew and Nix.

## Why a Wrapper?

The bare repo pattern is "powerful but has sharp edges." The tool smooths them by providing correct setup by default, automatic upstream tracking, clean deletion to prevent orphans, and commands that "match mental model" rather than requiring multi-step ritual.

## Closing Recommendation

Try the tool for a week with "The friction disappears" -- either via `clone` for new projects or `migrate` for existing ones. The post links to the GitHub repo and the earlier blog post "Git Worktrees Done Right."
