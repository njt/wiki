---
url: https://tangled.org/danabra.mov/woodshed
date_fetched: 2026-07-05
backfilled: true
---

# woodshed#

Create, run, rate, and iterate on your Claude Skills.

## ⚠️ WARNING ⚠️#

This is alpha software written for personal use.

- It runs Claude in **yolo mode**which can and will wipe your data.
- It can also **burn through a ton of tokens**if your Skills aren't lean.
- It is 100% vibecoded, and this time I **have not**read the code.

If you end up burning all your tokens only to brick your computer, I'm not responsible.

## Usage#

Create a new workspace:

```
npx woodshed create my-idea
```
That gives you a place work on your Skills:

```
cd my-idea
```
Time to shed:

```
npx woodshed
```
By default, the entire matrix runs 10 times.

Shed is idempotent so by default re-running it will instantly "skip over" past results as if they happened instantly.

Pass `npx woodshed --reset` to force re-runs. This will delete each re-run's data in `results/` before attempting it. You can also delete the `results/` yourself if you want.

Take a close look at the `results/` folder after your first successful run. It contains the log from the main agent with your fixture's `prompt.md`, the log from the evaluating agent with your fixture's `eval.md`, the workspace folder in which they both run, and probably some other junk.

## Folder Conventions#

```
my-idea/
  skills/
    # Skills you want to create or refine
    my-skill/
      # Each Skill can have one or more variants
      baseline/SKILL.md
      experiment/SKILL.md
      silly/SKILL.md
  fixtures/
    # Fixtures that test your Skill(s)
    my-fixture/
      prompt.md
      eval.md
  assets/
    # Data shared by fixtures
    words.txt
  results/
    # Outputs and past runs appear here
```
## Workflow#

I recommend running another Claude instance and to talk with it about the `/results`.

Then you can use insights from that convo to refine your `eval.md` and `SKILL.md`.

Tip: If you're iterating on a skill, ask Claude to write down each experiment in a doc so you can see what works and what doesn't.

## Options#

```
--runs <n>      Number of runs per variant (default: 10)
--reset         Delete old results and start fresh
--reeval        Re-run evaluation on existing workspaces
--cache-only    Show cached results only
```
## License#

MIT
