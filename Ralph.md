# Ralph

Two distinct implementations of the same idea: run a coding agent in a loop until the work is done, resetting context each iteration. Both named after Ralph Wiggum from The Simpsons. One is an open-source tool (snarktank/ralph); the other is Geoffrey Huntley's bare-metal technique using a bash while loop.

---

## Ralph (snarktank/ralph) — The Tool

The "Wiggum loop" as a productized tool: an autonomous agent that iterates through a PRD (Product Requirements Document) until every story passes. Each iteration spawns a fresh AI instance with clean context, implements one story, runs quality checks (typecheck, tests), commits if they pass, and moves on. Continuity across iterations comes from git history, progress.txt (append-only learnings), and prd.json (completion tracking). 19k stars, supports both Amp and Claude Code.

### Architecture

Ralph's architecture is built around two constraints: context windows are finite, and agents make mistakes. The fresh-context-per-iteration design means each run starts clean, avoiding context pollution. The git-as-memory pattern means the agent can see what changed without carrying full conversation history. The mandatory quality checks (typecheck, tests) act as a ratchet -- progress only counts if it passes.

The "right-sized stories" requirement is the key: tasks must fit a single context window. This forces decomposition discipline that most human developers skip.

### Strengths and Weaknesses

Strong: the PRD-driven loop is simple and effective for well-specified features. Git-as-memory avoids context pollution. The quality-check ratchet prevents regression. The AGENTS.md auto-documentation (each iteration updates codebase docs) is a clever side effect.

Weak: works only for features that decompose into independent stories that each fit a context window. Complex features with cross-cutting concerns don't decompose cleanly. The PRD must be human-written, so the bottleneck shifts from "writing code" to "writing stories."

## Ralph (Huntley's Technique) — The Bash Loop #tool

Geoffrey Huntley's approach strips the concept to its minimum: `while :; do cat PROMPT.md | claude-code ; done`. No framework, no PRD tracking, no progress.txt. Just a prompt file, a plan file (`fix_plan.md`), spec documents, and Claude Code on repeat.

**Key claim:** "Ralph can replace the majority of outsourcing at most companies for greenfield projects."

### How It Works

Each loop iteration gets one task. The LLM picks what to work on from fix_plan.md, ranked by importance. Continuity comes from:

- **fix_plan.md** — prioritized TODO list, regenerated periodically
- **AGENT.md** — self-updating best-practices log (correct commands, build tricks, test workflows)
- **Spec documents** — stdlib and language specifications that steer generation
- **Git history** — same as snarktank/ralph

### Two Phases Per Iteration

**Generation** is cheap and controllable through specs. If Ralph produces wrong patterns, update the stdlib spec to redirect future loops.

**Backpressure (validation)** is the bottleneck. Huntley stacks multiple validators: type systems (he favours Rust for correctness), security scanners, static analyzers (Dialyzer, PyreFlag), and unit tests. "The speed of the wheel turning matters, balanced against correctness."

### Subagent Delegation

Huntley runs up to 500 parallel subagents for search, exploration, and planning -- but restricts build/test validation to a single subagent to avoid backpressure failures. This preserves the primary context window for orchestration.

### Prompt Engineering Details

The prompt for his CURSED compiler project mandates:
- Search the codebase before implementing (ripgrep false negatives cause duplicate code)
- No placeholder implementations ("DO IT OR I WILL YELL AT YOU")
- Tests must document *why* they exist, so future iterations can distinguish real bugs from obsolete tests
- Run tests after every change

### Real-World Results

- A $50k outsourcing contract replicated for $297 (~168:1 cost reduction)
- Building CURSED, a new programming language + compiler (lexer, parser, LLVM codegen, self-hosting stdlib) — a language that doesn't exist in Claude's training data

### Limitations

- **Greenfield only.** "There's no way in heck would I use Ralph in an existing code base."
- **Targets ~90% completion.** The last 10% still needs human work.
- **Requires senior guidance.** "Engineers are still needed. There is no way this is possible without senior expertise guiding Ralph."
- **Eventual consistency mindset.** You must accept chaos during construction and trust that more iterations resolve it.

### Broader Claims

Huntley argues we're in "post-AGI territory" — current models and tools are sufficient, you just need tokens. He frames LLMs as "mirrors of operator skill": the human's ability to craft prompts, read failures, and steer the system determines outcomes.

## Comparing the Two Ralphs

| | snarktank/ralph | Huntley's technique |
|---|---|---|
| **Structure** | Productized tool with PRD tracking | Bare bash loop + prompt file |
| **Task selection** | Human writes stories in PRD | LLM picks from fix_plan.md |
| **Memory** | progress.txt, prd.json, git | AGENT.md, fix_plan.md, specs, git |
| **Validation** | Typecheck + tests | Type system + scanners + analyzers + tests |
| **Subagents** | No (single agent per iteration) | Yes (up to 500 parallel) |
| **Best for** | Well-decomposed feature work | Greenfield projects from specs |

Both validate the same core insight: fresh context per iteration plus a ratchet mechanism (tests that must pass) is more reliable than a single long-running agent session. The disagreement is about how much structure to impose. snarktank/ralph gives you a framework; Huntley says the framework is a bash one-liner and the real work is in the specs.

## Cross-References

Connects to [[Cord]] (dynamic task decomposition at runtime vs. Ralph's static decomposition), [[Serf]] (non-interactive task completion), [[Planning With Files]] (fix_plan.md and progress.txt as filesystem-as-memory), [[Feedback Loop is All You Need]] (validation as the real work), [[Compound Engineering]] (backpressure phase as compound loop), [[Specifications as the Product]] (Huntley's specs-as-steering aligns with the synthesis), [[Building low-level software with only coding agents]] (Pixo's similar cost economics), [[Agent Coding Workflow]] (Ralph sits at the "dark factory" end of the maturity spectrum), [[Scaling Long-Running Agents]] (fresh context per iteration as an alternative to planner/worker/judge), [[napkin]] (AGENT.md is napkin by another name), [[ralph-ban]] (kanban board for agent task management).

---
*Sources: [[raw/ralph]], [[raw/ralph-ghuntley]]*
*Last updated: 2026-05-14*
