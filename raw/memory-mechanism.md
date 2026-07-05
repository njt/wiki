---
url: https://docs.z.ai/devpack/resources/memory-mechanism
date_fetched: 2026-07-05
backfilled: true
---

**automatic memory, project memory**, and

**session memory**.

**Why do coding agents need memory?**

Traditional large language models do not preserve state between calls. As a result, they cannot remember project context across sessions, accumulate problem-solving experience over time, or consistently adapt to user preferences.
Agent systems address this limitation through **external memory**. A typical architecture looks like this:

**A complete memory architecture for modern coding agents**

At a high level, a complete agent memory architecture typically looks like this:
**Core memory types in coding agents**

- Session Memory
- Project Memory
- Semantic Memory
- Episodic Memory
- Procedural Memory

Session memory is the contextual information associated with the current task. It includes the current conversation history, recent tool outputs, the current execution plan, and the contents of the files currently in scope. This information typically lives in the model’s context window.For example:These execution steps all fall under session memory.

**The standard memory pattern used by coding agents**

In real-world systems, agents typically follow a consistent memory workflow.
Memory retrieval

Before starting a task, the agent retrieves relevant project memory, knowledge base entries, and prior experience, then injects them into the working context.

Context construction

The retrieved memories are assembled into a complete context and passed to the model.

**How to use memory correctly in coding agents**

In mainstream agent systems, memory is generally designed to be **layered, controllable, retrievable, and updatable**.

In most cases, memory is divided into 

**short-term memory**and**long-term memory**. Short-term memory is mainly used to preserve state within the current thread or session, while long-term memory is maintained through explicit files, rule configurations, vector retrieval, or other persistent storage mechanisms.Take 

**Claude Code**as an example. Its official documentation explicitly states that each session begins with a fresh context window. Knowledge is carried across sessions primarily through persistent instruction files such as CLAUDE.md and through**auto memory**. Similarly, in**LangChain / LangGraph**, memory is also divided into**thread-scoped short-term memory**and**long-term memory**that persists across sessions.### * Separate instruction memory from learning memory

One of the most practical principles for general-purpose coding agents is to distinguish between two fundamentally different kinds of memory:- **Instruction memory**: written by humans to tell the agent how it should work. This usually includes coding standards, directory conventions, build commands, test procedures, naming conventions, commit requirements, and team-level safety rules. In Claude Code, this maps to persistent instruction files such as- `CLAUDE.md`.
- **Learning memory**: not predefined in advance, but accumulated by the agent over time from your corrections, preferences, failed attempts, common commands, and project habits. Claude Code refers to this capability as auto memory, and its documentation states that it is loaded at the start of every conversation together with instruction files. For subagents, Claude Code can also maintain a separate persistent memory directory, and the first 200 lines of- `MEMORY.md`are included automatically.

### * Layered memory management

Organization-level memory


Organization-level memory

This layer contains rules defined and distributed at the team or company level, and applies across all developers and all relevant projects. Typical examples include:

- security and compliance requirements
- baseline code review standards
- restricted directories that must not be read from or written to
- dependency and license constraints
- organization-wide engineering standards

`sysytem.md` can be deployed to a system-level path and should not be easily excluded by individual users. In practice, this can also be distributed through centralized management tools such as hosted configuration, MDM, Group Policy, or Ansible. In a more general agent architecture, this means **organization-level memory should be treated as the highest-priority governance layer and should not be casually bypassed**.Project-level memory


Project-level memory

This is the team-shared project context, and it is the most important memory layer for a coding agent. It should be version-controlled and shared across all collaborators. Typical examples include:

- project architecture documentation
- directory structure conventions
- build and test commands
- where APIs should live
- naming conventions
- common development workflows

`project.md`, and its `/init ` command can generate an initial draft automatically. That draft can then be refined with rules the model is unlikely to infer on its own. The key property of this layer is that it is **shared across the project, tracked in version control, and stable over time**.User-level memory


User-level memory

This layer captures a developer’s personal preferences that apply across projects. It is best stored under the user’s home directory and treated as reusable personal context for all workspaces. In Claude Code, user instructions are stored separately from project instructions, and both are loaded at session start. This layer is a good place for:

- your preferred coding style
- your usual debugging sequence
- your preferred output format
- your personal workflow shortcuts

Local memory


Local memory

This layer is specific to your local copy of a project, but should not be committed to Git. A file such as local.md is a good place to store project-specific preferences that should remain private or machine-specific, such as:

- personal test accounts
- local development ports
- temporary mock service endpoints
- machine-specific runtime notes
- experimental workflows that are not ready to share

**allows individuals to work efficiently without polluting team-shared memory**.Subagent / role-specific memory


Subagent / role-specific memory

Another pattern worth generalizing is role-specific memory for subagents. Different subagents can maintain their own memory scopes rather than sharing a single global memory. This is especially important in multi-agent systems, where one of the most common failure modes is context pollution across roles.
A better pattern is to let each subagent retain only the memory relevant to its role:

- let the **testing agent**remember test commands, CI behavior, and assertion style
- let the **refactoring agent**remember module boundaries, restricted dependencies, and migration strategies
- let the **documentation agent**remember glossary terms, documentation templates, and audience-specific style

### * Loading `.md` files by path

Claude Code’s official documentation offers a very useful pattern for organizing memory in large codebases. For larger repositories, it recommends splitting instructions into multiple Markdown files under `.claude/rules/`, with each file focused on a single topic such as `testing.md`, `api-design.md`, or `security.md`.
Claude Code also supports **scoping rules to specific subdirectories or file types**, and these rules are loaded only when Claude is working with matching files. That reduces irrelevant noise and helps conserve context window space. As a general design pattern for coding agents, this can be summarized in three principles:

- **Keep the main memory file limited to global shared context**, such as project background, high-level architecture, and cross-project conventions.
- **Keep specialized rules modular**, with one rule file per topic.
- If a rule can be loaded by path, **do not load it globally; bring it into context only when needed**.

This structure offers three advantages:

- Easier to maintain. Each rule file focuses on a single topic, so the rule set is less likely to become bloated or disorganized. Claude Code explicitly recommends topic-specific files with descriptive names.
- Easier to load on demand. When the agent is working on tests, it does not need to load frontend conventions or database-specific rules into the context window.
- Better for team collaboration. Different teams or subteams can maintain their own rule directories instead of competing to edit a single monolithic instruction file.

### * Write memory rules as concrete instructions

When writing agent memory, use**specific, verifiable rules**whenever possible rather than abstract principles. The clearer the instructions are, the more stable the agent’s behavior will be. In general, it is recommended to:

- keep instructions **concise and explicit**
- keep rules **consistent**with one another
- keep the main memory file **under 200 lines**where possible
- use **Markdown headings and lists**to improve readability
- phrase requirements as rules that can be **checked and executed**

Instead, prefer rules like:

- Use **2-space indentation**in all new TypeScript files
- **Run**after modifying business logic- `pnpm test`
- Place **all API handlers under**`src/api/handlers/`
- Keep React page components **under 300 lines**; split larger ones into hooks or child components

### * Separate shared rules from personal preferences

When designing an agent memory structure, it is important to clearly define**the scope of each rule and who is responsible for it**. A common approach is to organize memory by scope:

- **Project**: shared by all team members and maintained through version control
- **Organization**: defined centrally by IT or DevOps, such as security standards or development processes
- **User**: applies only to an individual, such as personal coding habits
- **Local**: applies only to the current machine or working environment and should not be committed to Git
- **Role / Agent-specific**: used only by a specific specialized agent

who owns it, who shares it, and who it applies to.For example:

- team-wide conventions → project level
- company security policies → organization level
- personal coding habits → user level
- machine-specific configuration → local level
- rules for a specialized agent → role level

### * Reuse memory through imports and rule packages

In real projects, many rules are**shared engineering conventions across repositories**. Rewriting them in every repo increases maintenance overhead and makes inconsistency more likely. Using

**Claude Code**as an example, its documentation explains that:

- `CLAUDE.md`can import other rule files using- `@path/to/import`
- `.claude/rules/`can share rules through- **symbolic links**(symlinks)
- imported content can be **expanded recursively**, and symlinks are resolved normally

**reusable rule packages**, such as:

- `company-security-rules`
- `frontend-react-rules`
- `backend-api-rules`
- `python-testing-rules`

- **rules can be maintained centrally and updated consistently**
- **different projects can share the same engineering language**, making agent behavior more consistent across repositories

**Memory troubleshooting**

The coding agent is not following my `.md` memory files


The coding agent is not following my `.md` memory files

`.md` memory files are typically provided to the agent as contextual instructions, not as enforced configuration.The agent will read them and try to follow them, but it cannot guarantee strict compliance when the rules are vague, unclear, or conflicting.If the agent is not following the rules, you can check the following:- Run `/memory`(or the equivalent command) to confirm that the .md memory files have been loaded.
- Check whether the `.md`files are located in a path or scope that is allowed to load in the current session.
- Check whether there are conflicting rules across multiple `.md`files. If different files give different instructions for the same behavior, the agent may choose one arbitrarily.

I don’t know what auto memory has saved


I don’t know what auto memory has saved

Most coding agents maintain auto memory in the background to capture project context, user preferences, or common actions.You can inspect it in the following ways:

- Run `/memory`(or a similar command) to view the current auto memory directory.
- Auto memory is typically stored as Markdown files that you can read, edit, or delete directly.

My memory files are too large


My memory files are too large

Oversized memory files consume more of the context window, reduce the agent’s adherence to instructions, and increase the likelihood of conflicts.It is recommended to split detailed content into multiple Markdown files and use file references or imports (such as 

`@path/to/file`), or move rules into a dedicated rules directory such as `rules/`.Instructions disappear after context compression


Instructions disappear after context compression

Many coding agents 

**compress or summarize context**during long conversations in order to reduce context length.In most cases, memory files are**reloaded from disk**after compression, so only content that has been written into memory files will persist. If certain rules disappear after compression, that means those rules**existed only in the conversation**and were never written into a memory file.To fix this:- write long-term instructions into `.md`memory files
- do not rely on the conversation alone to preserve rules
