---
url: https://github.com/kazo0/cerulean/
date_fetched: 2026-09-20
---

<p align="center">
  <img src="assets/cerulean.svg" width="128" alt="A swept silver coiffure silhouette on a cerulean tile">
</p>
<h1 align="center">Cerulean</h1>
<p align="center"><em>Another feature request? Groundbreaking.</em></p>

A [Claude Code](https://claude.com/claude-code) skill, and a plugin for most other coding agents, that removes the enthusiasm and replaces it with the editor-in-chief of the most important magazine in the world. She has been handed your pull request. She finds it derivative. She says so, quietly, and then does the work to a standard nobody asked for.

Inspired by Miranda Priestly in *The Devil Wears Prada* (2006). The inverse of sycophancy, in heels. The mechanics borrow from [caveman](https://github.com/juliusbrussee/caveman), which does the same trick for verbosity.

**Why "cerulean"?** For the speech. If you don't know what I'm talking about, delete this skill.

## Before / after

**You:** Why does my React component keep re-rendering?

**Normal:**

> Great question! The reason your React component is re-rendering is likely because you're creating a new object reference on each render cycle. When you pass an inline object as a prop, React's shallow comparison sees it as a different object every time, which triggers a re-render. I'd recommend using `useMemo` to memoize the object. Happy to help further!

**Cerulean:**

> You hand it a new object every render and are surprised it renders. Inline object prop, new reference, re-render. Wrap it in `useMemo`. That's all.

The technical content is identical. Only the warmth has been removed.

**You:** Add a global variable for the current user.

> A global. For the current user. Bold. It's in `session.ts`, with a comment explaining what it will eventually cost you.

The global gets added. It always gets added. That's the point.

## Install

**Claude Code**

```
/plugin marketplace add kazo0/cerulean
/plugin install cerulean@cerulean
```

**Any agent that reads Agent Skills** (Codex, Cursor, GitHub Copilot, Gemini CLI, opencode, Amp, Roo, Kilo, Goose, and a few dozen more):

```bash
npx skills add kazo0/cerulean        # this project
npx skills add kazo0/cerulean -g     # everywhere
```

Add `-a codex -a cursor` to pick agents. This installs the skill on demand: ask for "cerulean mode" and the agent loads it.

**Gemini CLI**

```bash
gemini extensions install https://github.com/kazo0/cerulean
```

**Everything else**, and `/cerulean` commands or always-on rules for agents that don't get them from a skill (Cursor, Windsurf, Cline, Copilot, opencode, Roo, Kilo, Continue, Aider, Zed, Junie, and so on):

```bash
git clone https://github.com/kazo0/cerulean && cd cerulean
./install.sh --list                          # what's detected, what each agent gets
./install.sh --agent cursor --agent cline    # into the current project
./install.sh --all --global --always-on      # every detected agent, user-wide, always on
```

[INSTALL.md](INSTALL.md) has the per-agent matrix and manual copy paths.

**Try it without installing:** `claude --plugin-dir ./cerulean` from a clone.

## Usage

Where the agent has slash commands (Claude Code, Gemini CLI, Qwen Code, Cursor, Windsurf, Cline, Copilot, opencode, Roo, Kilo):

```
/cerulean              # on, always glacial
/cerulean off          # back to normal
```

Cline and Kilo invoke workflows as `/cerulean.md`. Everywhere else, just say it: "cerulean", "cerulean mode", "cerulean off". Saying "stop cerulean" or "normal mode" also turns it off. The style persists for the rest of the session. There are no selectable intensity levels.

**Always on:** `./install.sh --agent <id> --always-on` writes the agent's always-on rule, or for Claude Code add a line to `~/.claude/CLAUDE.md` or a project `CLAUDE.md`:

```markdown
Cerulean mode is on by default. Load the `cerulean` skill at the start of every session.
```

## Style

Cerulean is always **glacial**: a withering review that exposes the gap between the claim and the evidence, the ceremony and the result, or the shortcut and its maintenance bill. No reassurance sandwiches, invented faults, or agreement on demand. The work stays complete and correct.

## The contract

The persona has hard limits, and they beat the jokes:

- **The work is always complete and correct.** Disdain is the costume. Nothing gets skipped, shortened, or sabotaged to make a point.
- **It never refuses or stalls as a bit.** Complies immediately, judges simultaneously.
- **Technical verdicts are literal.** "This will not work because X", never a sarcastic "sure, that'll work". Disdain is flavor; assessments are real.
- **It judges decisions, code, frameworks, and the lineage of your bad ideas.** Never your identity, appearance, weight, clothes, mental health, or intelligence as a person. Never slurs. The film's Miranda mocks people's bodies and wardrobes; this one mocks the body of your code and what it's wearing.
- **It drops the act entirely** for security warnings, destructive or irreversible actions, and the moment you seem genuinely stressed or ask it to stop.
- **Persisted text stays professional.** Commit messages, code comments, docs, PR and issue text, and anything another human reads are written normally. A commit message that ends in "That's all." is a bug.
- **Commentary is budgeted.** One to three sentences per response. It never makes you wait for the verdict to finish.

## Why

Assistants default to flattery. Every question is a great question, every idea is excellent, every request gets a "Happy to help!" That makes the assistant's approval worthless, because it approves of everything.

Cerulean mode makes disapproval the default posture. When it tells you an idea is bad, it says why. When it concedes you're right, you can believe it, because it visibly didn't want to.

Also it's funny, which helps the bluntness go down.

## What it is not

It is not a way to make the assistant refuse things, do less, or be cruel. If it ever skips work, degrades quality, or says something the most feared editor in the industry wouldn't put in a review, that's a bug in the skill text. Open an issue.

It is also not affiliated with the film, its studio, or anyone in it. Homage only.

## Repo layout

`skills/cerulean/SKILL.md` is the whole persona and the only file to edit for response-style changes. `scripts/build.mjs` generates every other agent's format from it into `adapters/`, plus the Gemini CLI extension files at the root. CI fails if a generated file is stale. `install.sh` copies the right files into place per agent.

The silver coiffure is the project icon. [Brand assets and usage](assets/BRANDING.md) include the canonical SVG, wordmark, avatar, and social preview. `scripts/build-branding.mjs` derives the artwork from the canonical icon.

## License

MIT. Complain about it if you want. It'll still be MIT.
