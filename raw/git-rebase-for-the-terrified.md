---
url: https://www.brethorsting.com/blog/2026/01/git-rebase-for-the-terrified/
date_fetched: 2026-07-05
backfilled: true
---

# Git Rebase for the Terrified

Join the conversation on Hacker News.

As a maintainer of several OneBusAway projects, I regularly ask contributors to rebase their branches before merging. The response is often hesitation or outright fear. I get it. Rebase has a reputation for destroying work, and the warnings you see online don’t help.

Here’s the thing: the worst case scenario for a rebase gone wrong is that you delete your local clone and start over. That’s it. Your remote fork still exists. The main repository still exists. You can always recover. With that fear addressed, let me show you how to rebase.

## Why maintainers ask for rebases

When you create a branch from main and work on it for a few days, the main branch keeps moving. Other PRs get merged. By the time your PR is ready, your branch’s history diverges from main. A merge commit can combine them, but it creates a messy history with interleaved commits that make it harder to understand what changed and why.

Rebasing replays your commits on top of the current main branch, as if you’d just created your branch today. The result is a clean, linear history that’s easier to review and bisect when tracking down bugs.

## The actual commands

First, make sure you have the upstream repository configured as a remote. If you forked a repo and cloned your fork, you probably only have `origin` pointing to your fork:

```
git remote -v
```
If you don’t see the main repository listed, add it:

```
git remote add upstream https://github.com/OneBusAway/onebusaway-ios.git
```
Now fetch the latest changes from upstream:

```
git fetch upstream
```
Make sure you’re on your feature branch:

```
git checkout your-branch-name
```
Before rebasing, push your current work to your remote fork. This gives you a backup you can recover from if anything goes wrong:

```
git push origin your-branch-name
```
Now rebase onto upstream’s main branch:

```
git rebase upstream/main
```
If there are no conflicts, you’re done with the rebase. If there are conflicts, Git will stop and tell you which files need attention.

### Understanding conflict markers

When you open a conflicted file, you’ll see something like this:

```
<<<<<<< HEAD
const timeout = 5000;
=======
const timeout = 10000;
>>>>>>> upstream/main
```
This is confusing until you know what each section means:

- Everything between `<<<<<<< HEAD`and`=======`is**your code**from the commit being replayed
- Everything between `=======`and`>>>>>>> upstream/main`is**the code from main**that conflicts with yours

Your job is to decide what the final code should look like. Sometimes you want your version, sometimes theirs, sometimes a combination. Delete the markers and leave only the code you want to keep.

I always use VS Code for this step. Its merge conflict UI is the clearest I’ve found: it shows “Accept Current Change,” “Accept Incoming Change,” “Accept Both Changes,” and “Compare Changes” buttons right above each conflict. You can click through conflicts one at a time without manually hunting for markers.

### When conflicts get tricky

Some conflicts are straightforward: two people changed the same line differently. Pick one or combine them.

Others are harder. If you’re rebasing multiple commits and the same file conflicts repeatedly, it usually means your changes build on each other in ways that don’t cleanly apply to the new base. A few strategies:

- 
    **Squash first, then rebase.**If you have many small commits, combine them into one or two logical commits before rebasing. Fewer commits means fewer opportunities for conflicts.
- 
    **Abort and try a different approach.**If the conflicts are overwhelming,`git rebase --abort`and consider whether your branch has diverged too far. Sometimes it’s easier to create a new branch from main and manually re-apply your changes.
- 
    **Use**If you find yourself resolving the same conflicts repeatedly, enable rerere (reuse recorded resolution) with`git rerere`.`git config --global rerere.enabled true`. Git will remember how you resolved conflicts and apply the same resolution automatically next time.

After resolving each file’s conflicts:

```
git add path/to/resolved/file
git rebase --continue
```
Repeat until the rebase completes. If things go sideways and you want to abort:

```
git rebase --abort
```
This returns your branch to exactly where it was before you started.

## Validating your changes

After rebasing, verify that your changes still work:

```
git log --oneline upstream/main..HEAD
```
This shows only your commits that are ahead of main. Make sure they look right. Then build the project and run the tests. Rebasing can sometimes cause subtle issues if upstream changes conflict with your work in ways the merge didn’t catch.

## Force pushing

Here’s where people get nervous. After rebasing, your local branch has diverged from your remote branch. A normal `git push` will fail. You need to force push:

```
git push --force-with-lease origin your-branch-name
```
The `--force-with-lease` flag is safer than `--force` because it will fail if someone else has pushed to your branch since you last fetched. This prevents you from accidentally overwriting someone else’s work.

Never force push to main or any shared branch. Only force push to your own feature branches.

## When it all goes wrong

If you’ve made a mess of things and can’t figure out how to recover, here’s the nuclear option:

- Push any work you want to save to your remote fork (even to a temporary branch)
- Delete your local clone
- Clone fresh from your fork
- Add the upstream remote again
- Start the rebase process over

This has never failed me. Your commits exist on GitHub until you explicitly delete them. You can always recover.

## One more thing

Rebasing rewrites commit history. This is fine for feature branches that only you are working on. It’s not fine for branches that others have based work on. If you’re collaborating with someone on a branch, coordinate before rebasing, or just use merge commits instead.

That’s it. Rebase isn’t scary once you understand that you can always recover. The worst case is a few minutes of recloning. The benefit is a clean project history that’s easier to understand and maintain.
