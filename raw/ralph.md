---
url: https://github.com/snarktank/ralph
date_fetched: 2026-07-05
backfilled: true
---

Ralph is an autonomous AI agent loop that runs AI coding tools (Amp or Claude Code) repeatedly until all PRD items are complete. Each iteration is a fresh instance with clean context. Memory persists via git history, `progress.txt`, and `prd.json`.

Based on Geoffrey Huntley's Ralph pattern.

Read my in-depth article on how I use Ralph

- One of the following AI coding tools installed and authenticated:
- Amp CLI (default)
- Claude Code (`npm install -g @anthropic-ai/claude-code`)
 
- `jq`installed (- `brew install jq`on macOS)
- A git repository for your project

Copy the ralph files into your project:

```
# From your project root
mkdir -p scripts/ralph
cp /path/to/ralph/ralph.sh scripts/ralph/
# Copy the prompt template for your AI tool of choice:
cp /path/to/ralph/prompt.md scripts/ralph/prompt.md    # For Amp
# OR
cp /path/to/ralph/CLAUDE.md scripts/ralph/CLAUDE.md    # For Claude Code
chmod +x scripts/ralph/ralph.sh
```
Copy the skills to your Amp or Claude config for use across all projects:

For AMP

```
cp -r skills/prd ~/.config/amp/skills/
cp -r skills/ralph ~/.config/amp/skills/
```
For Claude Code (manual)

```
cp -r skills/prd ~/.claude/skills/
cp -r skills/ralph ~/.claude/skills/
```
Add the Ralph marketplace to Claude Code:

`/plugin marketplace add snarktank/ralph`Then install the skills:

`/plugin install ralph-skills@ralph-marketplace`Available skills after installation:

- `/prd`- Generate Product Requirements Documents
- `/ralph`- Convert PRDs to prd.json format

Skills are automatically invoked when you ask Claude to:

- "create a prd", "write prd for", "plan this feature"
- "convert this prd", "turn into ralph format", "create prd.json"

Add to `~/.config/amp/settings.json`:

```
{
  "amp.experimental.autoHandoff": { "context": 90 }
}
```
This enables automatic handoff when context fills up, allowing Ralph to handle large stories that exceed a single context window.

Use the PRD skill to generate a detailed requirements document:

```
Load the prd skill and create a PRD for [your feature description]
```
Answer the clarifying questions. The skill saves output to `tasks/prd-[feature-name].md`.

Use the Ralph skill to convert the markdown PRD to JSON:

```
Load the ralph skill and convert tasks/prd-[feature-name].md to prd.json
```
This creates `prd.json` with user stories structured for autonomous execution.

```
# Using Amp (default)
./scripts/ralph/ralph.sh [max_iterations]
# Using Claude Code
./scripts/ralph/ralph.sh --tool claude [max_iterations]
```
Default is 10 iterations. Use `--tool amp` or `--tool claude` to select your AI coding tool.

Ralph will:

- Create a feature branch (from PRD `branchName`)
- Pick the highest priority story where `passes: false`
- Implement that single story
- Run quality checks (typecheck, tests)
- Commit if checks pass
- Update `prd.json`to mark story as`passes: true`
- Append learnings to `progress.txt`
- Repeat until all stories pass or max iterations reached

| File | Purpose | 
|---|---|
| `ralph.sh` | The bash loop that spawns fresh AI instances (supports `--tool amp`or`--tool claude`) | 
| `prompt.md` | Prompt template for Amp | 
| `CLAUDE.md` | Prompt template for Claude Code | 
| `prd.json` | User stories with `passes`status (the task list) | 
| `prd.json.example` | Example PRD format for reference | 
| `progress.txt` | Append-only learnings for future iterations | 
| `skills/prd/` | Skill for generating PRDs (works with Amp and Claude Code) | 
| `skills/ralph/` | Skill for converting PRDs to JSON (works with Amp and Claude Code) | 
| `.claude-plugin/` | Plugin manifest for Claude Code marketplace discovery | 
| `flowchart/` | Interactive visualization of how Ralph works | 

**View Interactive Flowchart** - Click through to see each step with animations.

The `flowchart/` directory contains the source code. To run locally:

```
cd flowchart
npm install
npm run dev
```
Each iteration spawns a **new AI instance** (Amp or Claude Code) with clean context. The only memory between iterations is:

- Git history (commits from previous iterations)
- `progress.txt`(learnings and context)
- `prd.json`(which stories are done)

Each PRD item should be small enough to complete in one context window. If a task is too big, the LLM runs out of context before finishing and produces poor code.

Right-sized stories:

- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

Too big (split these):

- "Build the entire dashboard"
- "Add authentication"
- "Refactor the API"

After each iteration, Ralph updates the relevant `AGENTS.md` files with learnings. This is key because AI coding tools automatically read these files, so future iterations (and future human developers) benefit from discovered patterns, gotchas, and conventions.

Examples of what to add to AGENTS.md:

- Patterns discovered ("this codebase uses X for Y")
- Gotchas ("do not forget to update Z when changing W")
- Useful context ("the settings panel is in component X")

Ralph only works if there are feedback loops:

- Typecheck catches type errors
- Tests verify behavior
- CI must stay green (broken code compounds across iterations)

Frontend stories must include "Verify in browser using dev-browser skill" in acceptance criteria. Ralph will use the dev-browser skill to navigate to the page, interact with the UI, and confirm changes work.

When all stories have `passes: true`, Ralph outputs `<promise>COMPLETE</promise>` and the loop exits.

Check current state:

```
# See which stories are done
cat prd.json | jq '.userStories[] | {id, title, passes}'
# See learnings from previous iterations
cat progress.txt
# Check git history
git log --oneline -10
```
After copying `prompt.md` (for Amp) or `CLAUDE.md` (for Claude Code) to your project, customize it for your project:

- Add project-specific quality check commands
- Include codebase conventions
- Add common gotchas for your stack

Ralph automatically archives previous runs when you start a new feature (different `branchName`). Archives are saved to `archive/YYYY-MM-DD-feature-name/`.
