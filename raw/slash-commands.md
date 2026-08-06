---
url: https://code.claude.com/docs/en/slash-commands
date_fetched: 2026-08-06
---

`SKILL.md` file with instructions, and Claude adds it to its toolkit. Claude uses skills when relevant, or you can invoke one directly with `/skill-name`.
Create a skill when you keep pasting the same instructions, checklist, or multi-step procedure into chat, or when a section of CLAUDE.md has grown into a procedure rather than a fact. Unlike CLAUDE.md content, a skill’s body loads only when it’s used, so long reference material costs almost nothing until you need it.
For built-in commands like 

`/help` and `/compact`, and bundled skills like `/debug` and `/code-review`, see the commands reference.**Custom commands have been merged into skills.**A file at`.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way. Your existing `.claude/commands/` files keep working. Skills add optional features: a directory for supporting files, frontmatter to control whether you or Claude invokes them, and the ability for Claude to load them automatically when relevant.## Bundled skills

Claude Code includes a set of bundled skills, such as`/doctor`, `/code-review`, `/batch`, `/debug`, `/loop`, and `/claude-api`. Bundled skills are prompt-based: they give Claude detailed instructions and let it orchestrate the work using its tools. Most built-in commands instead execute fixed logic directly.
You invoke a bundled skill the same way as any other skill, by typing `/` followed by the skill name. Claude invokes some bundled skills automatically when relevant; others, including `/verify` and `/code-review`, run only when you invoke them, which keeps you in control of when these longer-running checks spend time and tokens. Before v2.1.215, Claude could also run `/verify` and `/code-review` on its own.
Bundled skills are available in every session. To turn them off, use the `disableBundledSkills` setting, which disables every bundled skill except `/doctor`.
The 

`/doctor` setup checkup stays typable when `disableBundledSkills` is on, in Claude Code v2.1.205 and later. To hide it, set the `DISABLE_DOCTOR_COMMAND` environment variable or a `skillOverrides` entry of `"doctor": "off"`. Before v2.1.205, `/doctor` was a built-in command rather than a bundled skill.**Skill**in the Purpose column.

### Run and verify your app

Three bundled skills work together to launch your app and confirm changes against the running app instead of just tests:
All three skills require Claude Code v2.1.145 or later. Check your version with 

`claude --version` or the `/status` command.
`/run` and `/verify` work without setup. They infer the launch from your project type (CLI, server, TUI, browser-driven) and from what’s in your README, `package.json`, or `Makefile`. That inference gets unreliable for projects that need anything beyond a standard launch: a database, an env file, a graphical session, a multi-step build.
`/run-skill-generator` records the recipe instead. It gets your app running from a clean environment, captures what worked (the install commands, the env vars, the launch script), and commits it as a per-project skill at `.claude/skills/run-<name>/`. After that, `/run`, `/verify`, and any other agent in the repo follow the recorded recipe instead of rediscovering it. Run `/run-skill-generator` once per project, and again if the build or launch process changes.
`/verify` can also record its own recipe. When it has to build and drive your app without a recorded recipe, it writes what worked to `.claude/skills/verify/SKILL.md` at the repo root, or in the touched package directory in a monorepo, so later runs and other agents follow the same steps. At the repo root, the recorded skill replaces the bundled `/verify`. This requires Claude Code v2.1.200 or later.
Claude edits the recorded file only when it steered a run wrong, such as a command that failed or a missing step, so you can commit the file without per-session diffs. Before v2.1.205, the bundled skill told Claude to fold in anything a run learned, which caused frequent merge conflicts.
## Getting started

### Create your first skill

This example creates a skill that summarizes the uncommitted changes in your git repository and flags anything risky. It pulls the live diff into the prompt before Claude reads it, so the response is grounded in your actual working tree rather than what Claude can guess from open files. Claude loads the skill automatically when you ask about your changes, or you can invoke it directly with`/summarize-changes`.
1

Create the skill directory

Create a directory for the skill in your personal skills folder. Personal skills are available across all your projects.

2

Write SKILL.md

Every skill needs a The 

`SKILL.md` file with two parts: YAML frontmatter between `---` markers that tells Claude when to use the skill, and markdown content with the instructions Claude follows when the skill runs. The directory name becomes the command you type, and the `description` helps Claude decide when to load the skill automatically.Save this to `~/.claude/skills/summarize-changes/SKILL.md`:`!`git diff HEAD`` line uses dynamic context injection: Claude Code runs the command and replaces the line with its output before Claude sees the skill content, so the instructions arrive with the current diff already inlined.3

Test the skill

Open a git project, make a small edit to any file, and start Claude Code by running Either way, Claude should respond with a short summary of your edit and a list of risks.

`claude`. You can test the skill two ways.**Let Claude invoke it automatically**by asking something that matches the description:**Or invoke it directly**with the skill name:### Where skills live

Where you store a skill determines who can use it:
When skills share the same name across levels, enterprise overrides personal, and personal overrides project. A skill at any of these levels also overrides a bundled skill with the same name. For example, a 

`code-review` skill in your project’s `.claude/skills/` replaces the bundled `/code-review`. Plugin skills use a `plugin-name:skill-name` namespace, so they cannot conflict with other levels. If you have files in `.claude/commands/`, those work the same way, but if a skill and a command share the same name, the skill takes precedence.
Skills also load from nested `.claude/skills/` directories below your working directory. When Claude reads or edits a file in a subdirectory, skills from that subdirectory’s `.claude/skills/` become available. This lets a monorepo package provide its own skills that apply when working on that package, even if the session started at the repo root.
If a nested skill shares a name with another skill, both stay available. For example, with a `deploy` skill at the project root and another in `apps/web/.claude/skills/`:
- The nested one appears under a directory-qualified name, `apps/web:deploy`.
- Its description says which directory it applies to.
- Claude picks the variant that matches the files it is working on.

`/deploy` runs the project-root skill. Type the qualified name `/apps/web:deploy` to run the nested variant explicitly.
When you or Claude invoke the unqualified name, the project-root skill loads, and Claude Code appends a list of the directory-qualified variants to its content with an instruction to also invoke any variant whose directory holds the files Claude is working on. A nested skill therefore still applies to work in its directory when only the unqualified name is invoked. Requires Claude Code v2.1.203 or later.
A `<skill-name>` entry in the enterprise, personal, or project locations can be a symlink to a directory elsewhere on disk. Claude Code follows the symlink and reads `SKILL.md` from the target directory, and if the same target is reachable from more than one location, Claude Code loads the skill once. Plugin skills handle symlinks differently; see Share files within a marketplace with symlinks.
Add a 

`.claude-plugin/plugin.json` to a skill folder and it loads as a plugin named `<name>@skills-dir`, so it can bundle agents, hooks, and MCP servers. In a project’s `.claude/skills/`, this requires accepting the workspace trust dialog first.#### Live change detection

Claude Code watches skill directories for file changes. When you add, edit, or remove a skill under`~/.claude/skills/`, the project `.claude/skills/`, or a `.claude/skills/` inside an `--add-dir` directory, Claude Code picks up the change within the current session, without a restart. If you create a top-level skills directory that didn’t exist when the session started, restart Claude Code so it can watch the new directory.
Live change detection covers 

`SKILL.md` text only. For a skill folder that is also a plugin, changes to `hooks/`, `.mcp.json`, `agents/`, and `output-styles/` need `/reload-plugins` to take effect.#### Discovery from parent and nested directories

Project skills load from`.claude/skills/` in the directory where you start Claude Code and in every parent directory up to the repository root. Starting Claude in a subdirectory still picks up skills defined at the root. To load skills from a directory outside that path at startup, pass it with `--add-dir`. Claude Code reads `.claude/skills/` inside each added directory alongside the project skills.
Skills in nested `.claude/skills/` directories below your starting directory aren’t loaded at startup. They load the first time Claude reads or edits a file inside that subdirectory, and stay available for the rest of the session. For example, after Claude edits a file under `packages/frontend/`, skills in `packages/frontend/.claude/skills/` become available. Until then, those skills don’t appear in autocomplete and can’t be invoked by name.
Each skill is a directory with `SKILL.md` as the entrypoint:
`SKILL.md` contains the main instructions and is required. Other files are optional and let you build more powerful skills: templates for Claude to fill in, example outputs showing the expected format, scripts Claude can execute, or detailed reference documentation. Reference these files from your `SKILL.md` so Claude knows what they contain and when to load them. See Add supporting files for more details.
Files in 

`.claude/commands/` still work and support the same frontmatter. Skills are recommended since they support additional features like supporting files.#### Skills from additional directories

The`--add-dir` flag and `/add-dir` command grant file access rather than configuration discovery, but skills are an exception: `.claude/skills/` within an added directory is loaded automatically. This exception applies only to `--add-dir` and `/add-dir`. The `permissions.additionalDirectories` setting in `settings.json` grants file access only and does not load skills. See Live change detection for how edits are picked up during a session.
Other `.claude/` configuration such as commands and output styles is not loaded from additional directories. See the exceptions table for the complete list of what is and isn’t loaded, and the recommended ways to share configuration across projects.
CLAUDE.md files from 

`--add-dir` directories are not loaded by default. To load them, set `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`. See Load from additional directories.#### Skills in Cowork and cloud sessions

Cowork sessions and cloud sessions, including routines, don’t read`~/.claude/skills/` on your machine. Both interactive and scheduled Cowork sessions load the skills enabled for your claude.ai account, synced at session start; manage them from **Customize**in the Desktop app sidebar or from the skills settings on claude.ai. Cloud sessions additionally load project skills committed to the cloned repository’s

`.claude/skills/`.
If a skill exists only in `~/.claude/skills/` on your machine, Claude Code reports that the skill was not found when a routine invokes it, because each routine run starts as a fresh remote session. To make a personal skill available in these sessions:
- For Cowork and cloud sessions, enable the skill for your claude.ai account.
- For cloud sessions, you can instead commit the skill to the repository’s `.claude/skills/`, or ship it in a plugin declared in the repository’s`.claude/settings.json`. Repo-declared plugins install at session start; plugins enabled only in your user settings don’t transfer.

## Configure skills

Skills are configured through YAML frontmatter at the top of`SKILL.md` and the markdown content that follows.
### Types of skill content

Skill files can contain any instructions, but thinking about how you want to invoke them helps guide what to include:**Reference content**adds knowledge Claude applies to your current work. Conventions, patterns, style guides, domain knowledge. This content runs inline so Claude can use it alongside your conversation context.

**Task content**gives Claude step-by-step instructions for a specific action, like deployments, commits, or code generation. These are often actions you want to invoke directly with

`/skill-name` rather than letting Claude decide when to run them. Add `disable-model-invocation: true` to prevent Claude from triggering it automatically. The example below adds `context: fork`, which runs the skill in its own subagent context; see Run skills in a subagent.
`SKILL.md` can contain anything, but thinking through how you want the skill invoked (by you, by Claude, or both) and where you want it to run (inline or in a subagent) helps guide what to include. For complex skills, you can also add supporting files to keep the main skill focused.
Keep the body itself concise. Once a skill loads, its content stays in context across turns, so every line is a recurring token cost. State what to do rather than narrating how or why, and apply the same conciseness test you would for CLAUDE.md content.
### Frontmatter reference

Beyond the markdown content, you can configure skill behavior using YAML frontmatter fields between`---` markers at the top of your `SKILL.md` file:
`description` is recommended so Claude knows when to use the skill.
Boolean fields accept `yes`, `no`, `on`, `off`, `1`, and `0` in any letter case, in addition to `true` and `false`. Before v2.1.218, Claude Code recognized only `true` and `false`.
#### Using skill frontmatter outside Claude Code

Claude Code accepts every field in the table above. Outside Claude Code, you can use only the fields in the Agent Skills spec:
When you enable a personal skill for Cowork and cloud sessions, including routines, you upload it to claude.ai, so the same rules apply.
If you include any field the spec doesn’t allow, packaging or upload fails with a hard error instead of ignoring the field:

#### How a skill gets its command name

The command you type to invoke a skill comes from where the skill file lives and, for plugin skills, also from the frontmatter`name` field. In a personal or project skill, `name` sets only the display label shown in skill listings, and the command still comes from the directory or file name. In a plugin skill, `name` sets the last segment of the command and the plugin prefix stays in place.
The table below shows where the command name comes from for each layout:
In a plugin skill, the frontmatter 

`name` replaces the directory name in the last segment of the command, so `my-plugin/skills/review/SKILL.md` with `name: fancy` becomes `/my-plugin:fancy`. The bare `/fancy` also invokes the skill unless another command already uses that name. Before v2.1.216, the frontmatter name replaced the whole command name, so the menu showed `/fancy` without the plugin prefix and `/my-plugin:fancy` didn’t autocomplete.
In non-interactive sessions, Claude Code doesn’t reserve the names `help` and `feedback` for their terminal-only built-in commands, so a plugin skill with one of those names keeps its bare command there. Claude Code still reserves the name of every other terminal-only built-in, such as `/login`, even though the command can’t run in those sessions. From v2.1.216 through v2.1.220, `help` and `feedback` were reserved too, so a plugin skill with one of those names was invocable only by its namespaced command in non-interactive sessions.
For a plugin-root `SKILL.md`, there is no skill directory to take the name from, so `name` supplies the whole final segment. Without a `name` field, Claude Code falls back to the plugin’s directory name.
#### Available string substitutions

Skills support string substitution for dynamic values in the skill content:
Claude Code substitutes 

`${CLAUDE_SKILL_DIR}` and `${CLAUDE_PROJECT_DIR}` in two places: the skill’s markdown content, and Bash rules in the `allowed-tools` frontmatter. Using the same variable in both places lets a skill run a bundled script without a permission prompt. The following skill shows the pattern:
`~/.claude/skills/render-chart/`, both occurrences of `${CLAUDE_SKILL_DIR}` expand to that directory. The `allowed-tools` rule then matches the exact command the skill body tells Claude to run, so the script runs without prompting.
The `allowed-tools` substitution for `${CLAUDE_SKILL_DIR}` requires Claude Code v2.1.129 or later. On earlier versions the rule stays a literal `${CLAUDE_SKILL_DIR}` string and never matches, so the command still prompts for permission.
The `${CLAUDE_PROJECT_DIR}` substitution requires Claude Code v2.1.196 or later.
Indexed arguments use shell-style quoting, so wrap multi-word values in quotes to pass them as a single argument. For example, `/my-skill "hello world" second` makes `$0` expand to `hello world` and `$1` to `second`. The `$ARGUMENTS` placeholder always expands to the full argument string as typed.
An indexed placeholder with no corresponding argument, such as `$2` when only one argument was passed, stays in the content unchanged. A named placeholder from the `arguments` frontmatter with no matching argument expands to an empty string.
To include a literal `$` before a digit, `ARGUMENTS`, or a declared argument name, such as `$1.00` in prose, escape it with a backslash: `\$1.00`. A backslash before any other `$` is left unchanged. Only a single backslash directly before the token escapes it. A doubled backslash such as `\\$1` leaves both backslashes in place, and `$1` still expands to the argument value.
**Example using substitutions:**

### Add supporting files

Skills can include multiple files in their directory. This keeps`SKILL.md` focused on the essentials while letting Claude access detailed reference material only when needed. Large reference docs, API specifications, or example collections don’t need to load into context every time the skill runs.
`SKILL.md` so Claude knows what each file contains and when to load it:
### Control who invokes a skill

By default, both you and Claude can invoke any skill. You can type`/skill-name` to invoke it directly, and Claude can load it automatically when relevant to your conversation. Two frontmatter fields let you restrict this:
- 
`disable-model-invocation: true``/commit`,`/deploy`, or`/send-slack-message`. You don’t want Claude deciding to deploy because your code looks ready.
- 
`user-invocable: false``legacy-system-context`skill explains how an old system works. Claude should know this when relevant, but`/legacy-system-context`isn’t a meaningful action for users to take.

`disable-model-invocation: true`, Claude can’t run the skill automatically:
`/deploy` yourself.
Here’s how the two fields affect invocation and context loading:
In a regular session, skill descriptions are loaded into context so Claude knows what’s available, but full skill content only loads when invoked. Subagents with preloaded skills work differently: the full skill content is injected at startup.

### Skill content lifecycle

When you or Claude invoke a skill, the rendered`SKILL.md` content enters the conversation as a single message and stays there for the rest of the session. This persistence applies to the skill’s instructions, not its permissions: an `allowed-tools` grant clears when you send your next message. Claude Code does not re-read the skill file on later turns, so write guidance that should apply throughout a task as standing instructions rather than one-time steps.
When Claude re-invokes a skill whose rendered content is identical to the copy already in context, Claude Code adds a short note that the skill is already loaded rather than a second copy of the content. When the rendered content differs, because the arguments changed or a dynamic context command produced new output, Claude Code appends the full content again. Before v2.1.202, every re-invocation appended another full copy of the skill’s instructions.
Auto-compaction carries invoked skills forward within a token budget. When the conversation is summarized to free context, Claude Code re-attaches the most recent invocation of each skill after the summary, keeping the first 5,000 tokens of each. Re-attached skills share a combined budget of 25,000 tokens. Claude Code fills this budget starting from the most recently invoked skill, so older skills can be dropped entirely after compaction if you have invoked many in one session.
If a skill seems to stop influencing behavior after the first response, the content is usually still present and the model is choosing other tools or approaches. Strengthen the skill’s `description` and instructions so the model keeps preferring it, or use hooks to enforce behavior deterministically. If the skill is large or you invoked several others after it, re-invoke it after compaction to restore the full content.
### Pre-approve tools for a skill

The`allowed-tools` field grants permission for the listed tools during the turn that invokes the skill, so Claude can use them without prompting you for approval. The grant clears when you send your next message, even though the skill content stays in context; invoking the skill again re-applies it for that turn. It does not restrict which tools are available: every tool remains callable, and your permission settings still govern tools that are not listed. To pre-approve tools for the whole session rather than a single turn, add allow rules to those permission settings instead.
For skills checked into a project’s `.claude/skills/` directory, `allowed-tools` takes effect after you accept the workspace trust dialog for that folder, the same as permission rules in `.claude/settings.json`. Review project skills before trusting a repository, since a skill can grant itself broad tool access.
This skill lets Claude run git commands without per-use approval whenever you invoke it:
`disallowed-tools` in the skill’s frontmatter. The restriction clears when you send your next message. Like deny rules, the field can’t remove `EndConversation` while any other tool remains. To block tools across all skills and prompts, add deny rules in your permission settings.
### Pass arguments to skills

Both you and Claude can pass arguments when invoking a skill. Arguments are available via the`$ARGUMENTS` placeholder.
This skill fixes a GitHub issue by number. The `$ARGUMENTS` placeholder gets replaced with whatever follows the skill name:
`/fix-issue 123`, Claude receives “Fix GitHub issue 123 following our coding standards…”
If you invoke a skill with arguments but the skill doesn’t include `$ARGUMENTS`, Claude Code appends `ARGUMENTS: <your input>` to the end of the skill content so Claude still sees what you typed.
You can also stack several skills at the start of one message. Typing `/write-tests /fix-issue 123` loads both skills and passes the trailing text `123` as `$ARGUMENTS` to each of them. Before v2.1.199, only the first skill loaded and received `/fix-issue 123` as literal argument text.
Claude Code expands the first skill plus up to five more stacked after it. Expansion stops at the first token that isn’t an inline user-invocable skill, so a skill that runs as a forked subagent, such as `/code-review`, or one whose arguments may themselves start with a slash command, such as `/loop`, also ends the run there. That token and everything after it become the argument text for every expanded skill. `/code-review` runs as a forked subagent from v2.1.218; on earlier versions it ran inline and stacked.
To access individual arguments by position, use `$ARGUMENTS[N]` or the shorter `$N`:
`/migrate-component SearchBar React Vue` replaces `$ARGUMENTS[0]` with `SearchBar`, `$ARGUMENTS[1]` with `React`, and `$ARGUMENTS[2]` with `Vue`. The same skill using the `$N` shorthand:
## Advanced patterns

### Inject dynamic context

The`!`<command>`` syntax runs shell commands before the skill content is sent to Claude. The command output replaces the placeholder, so Claude receives actual data, not the command itself.
This skill summarizes a pull request by fetching live PR data with the GitHub CLI. The `!`gh pr diff`` and other commands run first, and their output gets inserted into the prompt:
- Each `!`<command>``executes immediately (before Claude sees anything)
- The output replaces the placeholder in the skill content
- Claude receives the fully-rendered prompt with actual PR data

`!`<command>`` placeholders, so a command cannot emit a placeholder for a later pass to expand.
The inline form is only recognized when `!` appears at the start of a line or immediately after whitespace. If `!` follows another character, as in `KEY=!`cmd``, the placeholder is left as literal text and the command does not run.
For multi-line commands, use a fenced code block opened with ````!` instead of the inline form:
`"disableSkillShellExecution": true` in settings. Each command is replaced with `[shell command execution disabled by policy]` instead of being run. Bundled and managed skills are not affected. This setting is most useful in managed settings, where users cannot override it.
### Run skills in a subagent

Add`context: fork` to your frontmatter when you want a skill to run in isolation. The skill content becomes the prompt that drives the subagent. It won’t have access to your conversation history.
The forked subagent runs in the background: you keep working while it runs, and its result arrives in your conversation when it completes. Set `background: false` in the frontmatter to instead wait for the result in the turn that invoked the skill. Before v2.1.218, forked skills always blocked the turn until they finished.
Claude Code also waits for the result, even when the skill doesn’t set `background: false`, in cases like these:
- In non-interactive mode, with the `-p`flag or the Agent SDK
- When you set `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`to`1`, which also turns off all other background task features
- When you invoke a forked skill while an earlier invocation of the same skill is still running
- When a scheduled task fires with the skill as its prompt

`background: false` to keep the full tool set.
A forked skill that runs in the background applies its edits outside your session’s checkpoints, so `/rewind` doesn’t undo them; use git to revert them.
Skills and subagents work together in two directions:
With 

`context: fork`, you write the task in your skill and pick an agent type to execute it. The built-in Explore and Plan agents skip CLAUDE.md and git status to keep their context small, so a forked skill using `agent: Explore` sees only the SKILL.md content and the agent’s own system prompt. For the inverse, where you define a custom subagent that uses skills as reference material, see Subagents.
#### Example: Research skill using Explore agent

This skill runs research in a forked Explore agent. The skill content becomes the task, and the agent provides read-only tools optimized for codebase exploration:- A new isolated context is created
- The subagent receives the skill content as its prompt (“Research $ARGUMENTS thoroughly…”)
- The `agent`field determines the execution environment (model, tools, and permissions)
- The subagent summarizes its results and returns them to your main conversation when it finishes

`agent` field specifies which subagent configuration to use. Options include built-in agents (`Explore`, `Plan`, `general-purpose`) or any custom subagent from `.claude/agents/`. If omitted, uses `general-purpose`.
### Restrict Claude’s skill access

By default, Claude can invoke any skill that doesn’t have`disable-model-invocation: true` set. Skills that define `allowed-tools` grant Claude access to those tools without per-use approval during the turn that invokes the skill; the grant clears when you send your next message. Your permission settings still govern baseline approval behavior for all other tools. A few built-in commands are also available through the Skill tool, including `/init`, `/review`, and `/security-review`. Other built-in commands such as `/compact` are not.
Three ways to control which skills Claude can invoke:
**Disable all skills**by denying the Skill tool in

`/permissions`:
**Allow or deny specific skills**using permission rules:

`Skill(name)` for exact match, `Skill(name *)` for prefix match with any arguments.
**Hide individual skills**by adding

`disable-model-invocation: true` to their frontmatter. This removes the skill from Claude’s context entirely.
The 

`user-invocable` field only controls menu visibility, not Skill tool access. Use `disable-model-invocation: true` to block programmatic invocation.### Override skill visibility from settings

The`skillOverrides` setting controls skill visibility from your settings instead of the skill’s own frontmatter. Use it for skills whose SKILL.md you don’t want to edit, such as ones checked into a shared project repo. The `/skills` menu writes it for you: highlight a skill and press `Space` to cycle states, then `Enter` to save to `.claude/settings.local.json`.
Each key is a skill name and each value is one of four states:
The 

`/skills` menu labels the `"user-invocable-only"` state `user-only`.
As of v2.1.199, `"off"` also hides the skill from the command lists advertised to Remote Control clients and to Agent SDK callers, not only the terminal `/` menu. Invoking a hidden skill by its full name still returns the `skillOverrides` error instead of running it.
A skill that is absent from `skillOverrides` is treated as `"on"`. The example below collapses one skill to its name and turns another off entirely:
`skillOverrides`. Manage those through `/plugin` instead.
## Evaluate and iterate on a skill

Seeing a skill trigger tells you Claude found it, not that it did what you intended. To know a skill is working, measure two things separately: whether Claude invokes it on the prompts it should, and whether the output matches what you expect when it does. The check for both is a baseline comparison. Collect a few realistic prompts, run each one in a fresh session with the skill available and again with it disabled, and compare the results. A fresh session matters because leftover context from authoring the skill will mask gaps in the written instructions.### Run evals with skill-creator

The`skill-creator` plugin automates the comparison loop inside Claude Code. Install it from the official marketplace:
- `Marketplace "claude-plugins-official" not found`: add the marketplace with- `/plugin marketplace add anthropics/claude-plugins-official`, then retry the install.
- The plugin is not found in the marketplace: check the plugin name. Claude Code refreshes a stale marketplace catalog and retries before reporting this, so if you turned off marketplace auto-update, refresh manually with `/plugin marketplace update claude-plugins-official`and retry the install.

`Run /reload-plugins to activate.`, run that command to make the plugin’s skills available in the current session. Then ask Claude to evaluate an existing skill, for example `evaluate my summarize-changes skill with skill-creator`. The plugin walks you through writing test cases and runs the loop:
- **Test cases**: stores prompts, input files, and expected behavior in- `evals/evals.json`inside the skill directory
- **Isolated runs**: spawns a subagent per test case so each run starts with a clean context, and records token count and duration
- **Grading**: checks each assertion against the output and writes pass or fail with evidence to- `grading.json`
- **Benchmark**: aggregates pass rate, time, and tokens for with-skill versus without-skill into- `benchmark.json`so you can compare the pass-rate improvement against the token and time overhead
- **Version comparison**: runs a blind A/B between two versions of the skill so you can confirm an edit is an improvement before committing it
- **Description tuning**: generates should-trigger and should-not-trigger prompts, measures the hit rate, and proposes description edits when the skill activates on the wrong requests
- **Review viewer**: opens an HTML report where you inspect each output and record qualitative feedback that the next iteration reads

## Share skills

Skills can be distributed at different scopes depending on your audience:- **Project skills**: Commit- `.claude/skills/`to version control
- **Plugins**: Create a- `skills/`directory in your plugin
- **Managed**: Deploy organization-wide through managed settings

### Generate visual output

Skills can bundle and run scripts in any language, giving Claude capabilities beyond what’s possible in a single prompt. One powerful pattern is generating visual output: interactive HTML files that open in your browser for exploring data, debugging, or creating reports. This example creates a codebase explorer: an interactive tree view where you can expand and collapse directories, see file sizes at a glance, and identify file types by color. Create the Skill directory:`~/.claude/skills/codebase-visualizer/SKILL.md`. The description tells Claude when to activate this Skill, and the instructions tell Claude to run the bundled script. The script path uses `${CLAUDE_SKILL_DIR}` so it resolves correctly whether the skill is installed at the personal, project, or plugin level:
`~/.claude/skills/codebase-visualizer/scripts/visualize.py`. This script scans a directory tree and generates a self-contained HTML file with:
- A **summary sidebar**showing file count, directory count, total size, and number of file types
- A **bar chart**breaking down the codebase by file type (top 8 by size)
- A **collapsible tree**where you can expand and collapse directories, with color-coded file type indicators

`Generated /path/to/codebase-map.html`, and opens it in your browser. If you work in a headless environment where no browser opens, the printed path confirms the script succeeded.
This pattern works for any visual output: dependency graphs, test coverage reports, API documentation, or database schema visualizations. The bundled script does the work while Claude handles orchestration.
## Troubleshooting

### Skill not triggering

If Claude doesn’t use your skill when expected:- Check the description includes keywords users would naturally say
- Verify the skill appears in `What skills are available?`
- Try rephrasing your request to match the description more closely
- Invoke it directly with `/skill-name`if the skill is user-invocable

`/skill-name` still works but Claude has no `description` to match against. Run with `--debug` to see the parse error.
### Skill triggers too often

If Claude uses your skill when you don’t want it:- Make the description more specific
- Add `disable-model-invocation: true`if you only want manual invocation

### Skill descriptions are cut short

Claude Code loads a listing of skill names and descriptions into context so Claude knows what’s available. The listing always contains every skill name, but if you have many skills, Claude Code shortens descriptions to fit the listing’s character budget, which can strip the keywords Claude needs to match your request. The budget scales at 1% of the model’s context window. When the listing overflows, Claude Code drops descriptions starting with the skills you invoke least, so the skills you use most keep their full text. Run`/doctor` for an estimate of the listing’s context cost and its biggest contributors. When the listing exceeds its budget, Claude Code also writes a warning to the debug log, visible with `--debug`.
The Skills row in `/context` reports the size of the listing after the budget is applied, so it matches what the model receives. Before v2.1.196, the row counted the full text of every description and could show a value several times larger than the configured budget.
To raise the budget, set the `skillListingBudgetFraction` setting (e.g. `0.02` = 2%) or the `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable to a fixed character count. To free budget for other skills, set low-priority entries to `"name-only"` in `skillOverrides` so they list without a description. You can also trim the `description` and `when_to_use` text at the source: put the key use case first, since each entry’s combined text is capped at 1,536 characters regardless of budget. The cap is configurable with `skillListingMaxDescChars`.
## Related resources

- **Debug your configuration**: diagnose why a skill isn’t appearing or triggering
- **Evaluating skill output quality**: the eval file format and iteration workflow on agentskills.io
- **Skill authoring best practices**: writing guidance that applies across Claude products
- **Subagents**: delegate tasks to specialized agents
- **Plugins**: package and distribute skills with other extensions
- **Hooks**: automate workflows around tool events
- **Memory**: manage CLAUDE.md files for persistent context
- **Commands**: reference for built-in commands and bundled skills
- **Permissions**: control tool and skill access
- **Claude Tag skills**: project skills committed to a repo also load when that repo is used in a Claude Tag channel
