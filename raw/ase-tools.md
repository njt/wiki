---
url: https://ase.tools/
date_fetched: 2026-11-09
---

ase-help-skill - Show the Manual Page of an ASE Skill

## NAME

`ase-help-skill` - Show the Manual Page of an ASE Skill

## SYNOPSIS

`ase-help-skill`
    [`--help`|`-h`]
    [*skill-name*]

## DESCRIPTION

The `ase-help-skill` skill renders the *manual page* (`help.md`) of the
ASE skill addressed by *skill-name*. The page is emitted verbatim,
framed between a `MANUAL PAGE` header and footer rule.

The *skill-name* is resolved against a *catalog* of all ASE skills --
one `name: purpose` entry per skill, grouped by skill category.
Before resolving, any leading `/` and `ase:` prefix is stripped, so
`ase-code-lint`, `/ase-code-lint`, and `ase:ase-code-lint` are
equivalent. Resolution then proceeds in three tiers, each tried only if
all preceding ones found nothing: an *exact* match on the full skill
name wins outright, otherwise *every* skill name containing
*skill-name* as a substring becomes a candidate, and otherwise
*skill-name* is *fuzzily* matched against the skill *purposes* -- so
a topical phrase such as `root cause` still finds `ase-meta-why` --
with the candidates ordered by descending match quality.

A *skill-name* matching no skill in any tier is reported as an error. A
*skill-name* matching more than one skill opens an interactive dialog
listing the candidates with their purposes; as the dialog renders at
most nine answer lines, a broader abbreviation such as `task` shows the
first nine candidates and states how many exist in total.

If *skill-name* is omitted, the whole catalog is rendered as a
browsable list, grouped by skill category, instead.

The skill exposes *no* option flags beyond `--help`/`-h`.

## ARGUMENTS

- *skill-name*:
The full name of the ASE skill, any abbreviation of it, or a
description of its purpose. If omitted, the entire skill catalog is
listed.

## SCENARIOS

- You want the manual page of an ASE skill
- You want to know what a certain `ase-xxx-xxx`skill does
- You want a skill found by an abbreviation or a purpose description
- You want to browse the whole catalog of ASE skills

## EXAMPLES

Show the manual page of `ase-code-lint` via its shortest abbreviation:

```
❯ /ase-help-skill lint
```

Show the same manual page via its full, plugin-qualified name:

```
❯ /ase-help-skill ase:ase-code-lint
```

Pick a manual page from the candidates of an ambiguous abbreviation:

```
❯ /ase-help-skill analyze
```

Find a manual page by the purpose of the skill instead of its name:

```
❯ /ase-help-skill root cause
```

List the entire skill catalog:

```
❯ /ase-help-skill
```

## SEE ALSO

`ase-help-intent`, `ase-meta-proximity`.

ase-help-intent - Match an Intent to ASE Commands

## NAME

`ase-help-intent` - Match an Intent to ASE Commands

## SYNOPSIS

`ase-help-intent`
    [`--help`|`-h`]
    *intent*

## DESCRIPTION

The `ase-help-intent` skill matches a free-text *intent* against the
*accumulated help* of all ASE skills -- the concatenation of every
skill's `help.md` file into `skills/ase-help-intent/data.md`, built by
`npm start build` in `plugin/` -- and generates *all* adequately fitting
`/ase:ase-xxx-xxx` commands that realize the intent, ranked best-fitting
first and limited to the eight best ones. The fit is judged primarily
against each skill's `SCENARIOS` ("You want ...") and `DESCRIPTION`
sections, and each command is complete with concrete option flags and
positional arguments derived from the skill's `SYNOPSIS`, `OPTIONS`,
and `ARGUMENTS`.

The generated commands are presented together with a brief per-command
rationale in an interactive dialog. The dialog lets the user *execute*
one of the commands `C1`...`C8` (which dispatches the target skill via
its generated arguments), *cancel* the operation, or *refine* the intent
by typing any free-text instruction -- the instruction is folded into
the intent and the fitting commands are re-matched and re-rendered. If
no skill confidently matches the intent, a warning is emitted and the
user is prompted to refine or clarify it.

The skill exposes *no* option flags beyond `--help`/`-h`; it is driven
entirely through the intent argument and the interactive dialog.

## ARGUMENTS

- *intent*:
The free-text intent to be realized. It describes- *what*the user
wants to achieve; the skill determines- *which*ASE skills and- *which*options and arguments realize it.

## SCENARIOS

- You want to know which ASE skills realize what you have in mind
- You want free text turned into concrete slash commands with options
- You want all matching commands proposed, ranked best-fitting first
- You want to refine an intent in a dialog until a command fits

## EXAMPLES

Route an intent to the matching commands and pick one from the dialog:

```
❯ /ase-help-intent lint the TypeScript sources for high-severity issues only
```

Route a planning intent to the matching commands:

```
❯ /ase-help-intent explain how the authentication module works
```

## SEE ALSO

`ase-help-skill`, `ase-meta-proximity`, `ase-meta-search`,
`ase-task-edit`.

ase-meta-config - Configuration Management

## NAME

`ase-meta-config` - Configuration Management

## SYNOPSIS

`ase-meta-config`
    [`--help`|`-h`]
    [`--scope`|`-s` *scope*]
    *operation*
    [*args*]

## DESCRIPTION

The `ase-meta-config` skill lists, reads, writes, and removes the values
of the *layered ASE configuration* from inside the assistant session. It
mirrors the non-interactive subcommands of the `ase config` CLI, but
performs *every* access through the `ase_config_list`, `ase_config_get`,
`ase_config_set`, and `ase_config_delete` tools of the `ase` MCP server -
it never shells out and never touches a file directly.

The configuration is organized as a chain of *scopes*, canonically
ordered `default` < `user` < `project` < `task` < `session`. Reads
cascade from the strongest (rightmost) scope down to the weakest and
return the first value that is defined; writes are always confined to
the strongest (target) scope of the chain.

The recognized keys are grouped under `project.*` (`project.id`,
`project.name`, `project.boxing`, `project.task.lifecycle`,
`project.task.idscheme`, `project.task.default.{source,changeset,target}`,
and the `project.artifact.`*kind*`.{basedir,files}`
globs) and `agent.*` (`agent.persona`, `agent.guidance`, and
`agent.diagram.{format,theme,tone}`).
Some keys are writable on selected scopes only; in particular
the `project.artifact.*` globs are writable on the `user` and
`project` scopes only. The internal session state, like the active
task id (see `ase-task-id`), is not part of the configuration.

The four keys steering the behaviour of the agent itself accept fixed
value sets: `agent.persona` selects the *communication style* and is one
of the decorative, eloquent, and explaining `writer`, the concise,
factual, and accurate `engineer` (the default), the layered,
pyramid-structured `journalist`, the brief, factual, and abbreviating
`telegrapher`, or the terse, rough, and stuttering `caveman`;
`agent.guidance` selects the number of unsolicited hints and is one of
`none`, `minimal`, `normal` (the default), or `verbose`;
`agent.diagram.format` selects the rendering of the diagrams of the skills and is
one of `image` (the default, an inline image where the terminal supports it,
else text) or `text` (always text); and
`project.boxing` selects the artifact transparency and is one of `white`,
`grey`, or `black`. Setting `agent.persona`, `agent.guidance`,
`agent.diagram.format`, or `project.boxing` takes effect immediately, i.e. still
within the running session. Additionally, `project.task.lifecycle` selects the task plan
lifecycle and is one of `solo` (the default), `team`, or `enterprise`; it is
exported at session start as `<ase-project-task-lifecycle/>`. Finally,
`project.task.idscheme` selects the scheme of newly generated task ids and is one
of `slug[:<words>]` (the default, the first *words* (default: 2) words of
the task title, lower-cased and joined with `-`), `seq[:<template>]` (a
continuously increasing sequence number, rendered through a sprintf-style
*template* like `FOO-%03d`, `#%d`, or `%d` (the default)), or `any` (an
arbitrary unique id chosen by the agent). Additionally,
`project.task.default.{source,changeset,target}` select the defaults of
the `Source:`, `Changeset:`, and `Target:` task plan keys (`worktree`,
`worktree`, and `source` by default), which an absent key reads as, where
a `branch:<name>` or `attachment:<name>` value can carry the placeholder
`{id}` of the task id (e.g., `branch:cs-{id}` for a branch per task); they
are exported at session start as
`<ase-project-task-default-{source,changeset,target}/>`.

The diagram images are styled by `agent.diagram.theme`, one of `light`
(the default, dark node boxes with light labels) or `dark` (vice versa),
and `agent.diagram.tone`, one or two theme colors `#rgb` or `#rrggbb`
(default: `#336699`), where a missing second color is derived as the
complement of the first one (a brown for the default blue).

The following *operations* exist:

- `list`: List all effective configuration entries of the scope chain as
a table of key, value, and the scope that supplied the value. For
overlapping keys only the value of the strongest scope is shown.
 
- `get`- *key*: Report the effective value at the dotted- *key*, or- `(not set)`if no scope of the chain defines it.
 
- `set`- *key*- *value*: Write- *value*at the dotted- *key*on the target
scope. The value is validated against the configuration schema before
it is persisted.
 
- `delete`- *key*: Remove the value at the dotted- *key*from the target
scope. A key that is not present is silently ignored.
 

The `init` and `edit` subcommands of the `ase config` CLI are
deliberately *not* mirrored: `edit` is bound to the interactive `$EDITOR`
and therefore has no meaning inside an assistant turn, and `init` is a
preset-bootstrapping operation that stays a shell concern next to
`ase setup`. Both remain available as `ase config init` and
`ase config edit` on the command line.

## OPTIONS

- `--scope`|- `-s`- *scope*:
The scope chain to operate on, given as a comma-separated list of- `user`,- `project`,- `task:`- *id*, and/or- `session:`- *id*terms, in any
order and at most one term per kind. The- `user`term is always
implicitly added at the bottom of the chain, and- `project`is
implicitly added whenever a project context exists, but never above
the strongest explicitly requested term. If the option is
omitted, the chain of the- *current session*(- `session:`- *id*) is used,
so that reads see the full- `user`->- `project`->- `session`cascade
and writes land on the session layer.

## ARGUMENTS

- *operation*:
The operation to perform; one of- `list`,- `get`,- `set`, or- `delete`.
 
- *args*:
The operands of the operation: none for- `list`, a dotted- *key*for- `get`and- `delete`, and a dotted- *key*plus a- *value*for- `set`.
 

## SCENARIOS

- You want the ASE configuration inspected or changed
- You want the persona, guidance, boxing, or task lifecycle switched for a session
- You want a configuration value persisted at user or project scope
- You want to see which scope supplies each effective value

## EXAMPLES

List all effective configuration entries:

```
❯ /ase-meta-config list
```

Report the effective persona style:

```
❯ /ase-meta-config get agent.persona
```

Switch the communication style for the current session only:

```
❯ /ase-meta-config set agent.persona telegrapher
```

Switch the project boxing for the current session only:

```
❯ /ase-meta-config set project.boxing grey
```

Persist the guidance level for the whole project instead:

```
❯ /ase-meta-config --scope project set agent.guidance minimal
```

Persist the task lifecycle for the whole project:

```
❯ /ase-meta-config --scope project set project.task.lifecycle team
```

Remove a value from the user-level configuration:

```
❯ /ase-meta-config --scope user delete project.name
```

## SEE ALSO

`ase-task-id`.

ase-meta-alias - Skill Command Alias Management

## NAME

`ase-meta-alias` - Skill Command Alias Management

## SYNOPSIS

`ase-meta-alias`
    [`--help`|`-h`]
    [`--scope`|`-s` `auto`|`user`|`project`]
    *operation*
    [*args*]

## DESCRIPTION

The `ase-meta-alias` skill lists, sets, deletes, and expands the *skill
command aliases* of ASE from inside the assistant session. It mirrors the
non-interactive subcommands of the `ase alias` CLI.

There are three kinds of aliases, all expanded deterministically by ASE
itself, and never by the AI:

- A - *full alias*is a new command like- `/foo-bar`, which stands for an
arbitrary ASE skill command line, like- `/ase-code-lint -S HIGH src/`,
or for another full alias. On invocation, the options and arguments of
the alias precede the arguments given to the alias. A full alias is
provided as a small generated skill, which just forwards to its target,
and which is invoked as- `/foo-bar`in- *Anthropic Claude Code CLI*and- *GitHub Copilot CLI*, and as- `$foo-bar`in- *OpenAI Codex CLI*.
 
- An - *option alias*is a new option like- `--custom`of a particular
skill, which is expanded in place into a sequence of real options of
the skill, like- `-x -y -z`, or of other option aliases of the skill.
 
- The - *option defaults*of a particular skill are options which are
implicitly applied on every invocation of the skill. An option given
explicitly on invocation always wins over its default, and every option- `--x`can be dropped again by its negation- `--no-x`, which resets the
option to its built-in default.
 

The aliases are defined *per user* and *per project*, where an entry of
the project overrides the same entry of the user. The project aliases
(and their generated skills) are intended to be committed to the
repository, so the entire team shares them.

The following *operations* exist:

- `list`[- `--skill`- *skill*] [- `--tuned`]: List the effective aliases
as a table of kind, name, option, value, and supplying scope,
optionally of a single- *skill*only, or list just the skills with
option aliases or option defaults.
 
- `set`- *name*- *command-line*: Set the full alias- *name*.
 
- `set`- `--option`- *skill*- *option*- *expansion*: Set the option alias- *option*of- *skill*.
 
- `set`- `--default`- *skill*- *options*: Set the option defaults of- *skill*.
 
- `delete`- *name*,- `delete`- `--option`- *skill*- *option*, resp.- `delete`- `--default`- *skill*: Delete a full alias, an option alias, resp. the
option defaults of a skill.
 
- `expand`- *command-line*: Show how a skill command line is expanded
through the full aliases, option aliases, and option defaults, and
which options and arguments the skill finally receives.
 

The `edit` and `sync` subcommands of the `ase alias` CLI are deliberately
*not* mirrored, as `edit` is bound to the interactive `$EDITOR`, and
`sync` happens automatically. Both remain available on the command line.

## OPTIONS

- `--scope`|- `-s`- `auto`|- `user`|- `project`:
The scope of the aliases to operate on. With- `auto`(the default),- `set`and- `delete`operate on the aliases of the- *project*inside a
project (else of the- *user*), and- `list`shows the aliases of both
scopes.

## ARGUMENTS

- *operation*:
The operation to perform; one of- `list`,- `set`,- `delete`, or- `expand`.
 
- *args*:
The operands of the operation, as described above.
 

## SCENARIOS

- You want a short command for a skill command line you use often
- You want a short option for a combination of options of a skill
- You want a skill to always run with certain options
- You want to see or test how a command line expands through the aliases

## EXAMPLES

List all effective aliases:

```
❯ /ase-meta-alias list
```

Define a full alias for a security-focused linting:

```
❯ /ase-meta-alias set lint-sec /ase-code-lint --include=A05,A06 -S HIGH
```

Define an option alias of the linting skill:

```
❯ /ase-meta-alias set --option ase-code-lint --sec --include=A05,A06 -S HIGH
```

Always grill a task plan after editing it, for yourself only:

```
❯ /ase-meta-alias --scope user set --default ase-task-edit --next GRILL
```

Check how a command line expands:

```
❯ /ase-meta-alias expand /lint-sec src/
```

Remove the option defaults again:

```
❯ /ase-meta-alias --scope user delete --default ase-task-edit
```

## SEE ALSO

`ase-meta-config`.

ase-meta-compat - Self-Test ASE Compatibility

## NAME

`ase-meta-compat` - Self-Test ASE Compatibility

## SYNOPSIS

`ase-meta-compat`
    [`--help`|`-h`]

## DESCRIPTION

The `ase-meta-compat` skill *self-tests* how faithfully the current LLM
(and its harness) executes the four *core interpreter primitives* that
every ASE skill silently relies on, and reports an overall *0%…100%
compatibility* rating. ASE skills are not run by a conventional
interpreter - they are interpreted by the *model itself*, which must honor
control-flow constructs, substitute XML placeholders, evaluate regex
conditions, and perform arithmetic. If a model cannot execute these
primitives reliably, *every* ASE skill silently misbehaves, so this skill
lets a user measure the fit *before* relying on ASE.

The skill is both the *test definition* and the *system under test*. It
probes four capability categories:

- *XML Placeholders*: the get (- `<x/>`) and set (- `<x>v</x>`) semantics,
including the get-and-set composite, indexed names, and XML-entity
rendering.
 
- *Control Flow*: the- `<if>`/- `<elseif>`/- `<else>`,- `<while>`/- `<break>`,- `<for>`,- `<step condition=…>`, and- `<expand>`/- `<define>`constructs.
 
- *Regex Matching*: the patterns ASE genuinely uses (e.g. the getopt
short-circuit- `(^|\s)-`), plus anchoring, alternation, and captures.
 
- *Arithmetic*: counter increments, decision-matrix product-sums,
percentage rounding, and bar-width computations.
 

Each *probe* embeds a construct with a *known expected result*; the model
executes it and honestly self-checks its actual result against the
expected one. Per-category pass-rates are combined into the overall rating
via the `ase_decision_matrix` MCP tool as a *weighted average*, with
*XML Placeholders* weighted `4.00` (the backbone of every skill),
*Control Flow* weighted `3.00`, *Regex Matching* weighted `2.00`, and
*Arithmetic* weighted `1.00`. The
result is a boxed per-category table (passes and percentage per category),
a final *OVERALL COMPATIBILITY* percentage, and a one-line *VERDICT*
(`✓ fully compatible` at `≥ 90%`, `⚠ partially compatible` at `60–89%`,
`✘ incompatible` below `60%`) that names any failed probes.

## ARGUMENTS

The `ase-meta-compat` skill takes no arguments (besides `--help`|`-h`). It
always runs the most rigorous self-test, with 6 *probes per category*
(7 for *Regex Matching*), for 25 probes in total.

## SCENARIOS

- You want to know how faithfully the current model runs ASE skills
- You want a 0-100% compatibility rating of the LLM and its harness
- You want the core interpreter primitives checked before relying on ASE

## EXAMPLES

Run the compatibility self-test:

```
❯ /ase-meta-compat
```

## SEE ALSO

`ase-meta-evaluate`, `ase-meta-quorum`, `ase-meta-why`.

ase-meta-why - Five-Whys Root-Cause Analysis

## NAME

`ase-meta-why` - Five-Whys Root-Cause Analysis

## SYNOPSIS

`ase-meta-why`
    [`--help`|`-h`]
    [`--depth`|`-d` *N*]
    [`--width`|`-w` *M*]
    *fact*

## DESCRIPTION

The `ase-meta-why` skill applies the *Five-Whys* *root-cause
analysis* technique to the supplied *fact*. The skill iteratively
asks "why" - up to *N* times (see `--depth`, default five) - to drill
down from surface symptoms to the underlying root cause, considering
technical, domain-specific, process-related, and organizational
causes. By default it walks a *single* causality chain, but with
`--width` *M* (> 1) it walks a *widened* chain: at each level it surfaces
up to *M* candidate sub-causes, descends into the most significant one
with an explicit justification for the choice, and keeps the unchosen
candidates as *fallbacks*. If backward validation later shows the chosen
path was mis-rooted, it *backtracks* into a fallback and re-descends -
guarding against the classic Five-Whys failure of committing early to the
wrong sub-cause. After identifying (and validating) the root cause it
proposes a *SOLUTION* that addresses it, optionally including concrete
source code changes.

## OPTIONS

- `--depth`|- `-d`- *N*:
The- *maximum*number of "why" iterations (the Five-Whys chain
length), acting as an- *upper bound*only - the analysis still stops
early once the root cause is reached. Defaults to- *5*. A non-numeric
or non-positive value falls back to the default.
 
- `--width`|- `-w`- *M*:
The- *maximum*number of- *candidate sub-causes*to surface per "why"
level. With the default- *1*, the skill walks a single causality chain
(classic Five-Whys); with- *M*> 1, each level surfaces up to- *M*candidate sub-causes, descends into the single most significant one
(justifying the choice), and retains the rest as- *fallbacks*. During
backward validation a mis-rooted choice is- *backtracked*into a fallback
and re-descended, so the widening actively improves which root cause is
reached rather than merely listing alternatives. The result is still a
single, but better-justified, root cause. A non-numeric or non-positive
value falls back to the default.
 

## ARGUMENTS

- *fact*:
The observed- *fact*(symptom, problem, or surprising outcome)
whose root cause should be investigated. The skill implicitly
prepends "Why" to form the initial question.

## SCENARIOS

- You want the root cause of a problem, not just its symptom
- You want a Five-Whys causality chain walked for an observed fact
- You want candidate causes weighed with backtracking
- You want a solution proposed which addresses the root cause

## EXAMPLES

Investigate the root cause of a build failure:

```
❯ /ase-meta-why the CI build is intermittently failing on macOS runners
```

Drill down deeper with a tunable chain length of seven:

```
❯ /ase-meta-why -d 7 the production latency spiked after the last deploy
```

Weigh several candidate sub-causes per level (with backtracking) to avoid
committing early to the wrong root cause:

```
❯ /ase-meta-why -w 3 the release was delayed by two weeks
```

## SEE ALSO

`ase-code-analyze`, `ase-task-edit`, `ase-arch-analyze`.

ase-meta-eli5 - Explain a Topic Like I'm 5

## NAME

`ase-meta-eli5` - Explain a Topic Like I'm 5

## SYNOPSIS

`ase-meta-eli5`
    [`--help`|`-h`]
    [`--ground`|`-g`]
    *topic*

## DESCRIPTION

The `ase-meta-eli5` skill explains the supplied *topic* in "Explain
Like I'm 5" (ELI5) style, so that even a five-year-old child could
understand it. The explanation uses simple, jargon-free wording --
short sentences, everyday words, and familiar images -- and is
structured into three labeled blocks: *WHAT* (a plain-words summary),
*ANALOGY* (a concrete everyday-life comparison), and *WHY* (why the
topic matters), each kept brief with 1-3 bullet points.

By default the explanation is derived from model knowledge only. With
the `--ground`/`-g` option, the skill first searches the Internet/Web
for facts about the topic via the `ase-meta-search` skill (dispatched
in a sub-agent, querying all available search backends) and grounds
the explanation in the found facts. If the grounding yields no usable
facts, a warning is emitted and the explanation falls back to model
knowledge.

## OPTIONS

- `--ground`|- `-g`:
Ground the explanation in Internet/Web facts gathered via the- `ase-meta-search`skill before explaining. Without this option,
the explanation is derived from model knowledge only.

## ARGUMENTS

- *topic*:
The topic to be explained. It may be a technical concept, a
phenomenon, or any other subject; the skill explains it in a
child-friendly way.

## SCENARIOS

- You want a topic explained so simply a child could follow
- You want a jargon-free summary with an everyday analogy
- You want a simple explanation grounded in current Internet/Web facts

## EXAMPLES

Explain a technical concept from model knowledge:

```
❯ /ase-meta-eli5 What is a Fourier transform?
```

Explain a recent topic grounded in Internet/Web facts:

```
❯ /ase-meta-eli5 --ground What is Agentic Software Engineering?
```

## SEE ALSO

`ase-code-explain`, `ase-meta-search`, `ase-meta-quotes`, `ase-docs-distill`.

ase-meta-evaluate - Evaluate Alternatives

## NAME

`ase-meta-evaluate` - Evaluate Alternatives

## SYNOPSIS

`ase-meta-evaluate`
    [`--help`|`-h`]
    *request*

## DESCRIPTION

The `ase-meta-evaluate` skill evaluates two or more *alternatives*
through a *weighted multi-criteria decision matrix*. From the
*request*, the skill derives the reason for the evaluation, the
alternatives, and the criteria, fills in additional criteria up to a
range of 8-12 (using `ase-meta-search` where helpful), assigns weights
from `{ 4.00, 2.00, 1.00, 0.50, 0.25 }`, evaluates each alternative
against each criterion on a `{ -2, -1, 0, +1, +2 }` Likert scale, and
computes the final ratings via the `ase_decision_matrix` MCP tool.

The result is reported as a Markdown decision matrix along with the
single *BEST ALTERNATIVE*, including a warning if multiple
alternatives tie, if the distance to the second best is small, or if
all alternatives rate negatively.

## ARGUMENTS

- *request*:
A description of what should be evaluated, including the
alternatives and, optionally, the criteria to consider.

## SCENARIOS

- You want alternatives compared and the best one named
- You want a weighted multi-criteria decision matrix
- You want a defensible, criteria-based technology choice
- You want to know which option out of a list wins and by how much

## EXAMPLES

Compare logging libraries for a TypeScript project:

```
❯ /ase-meta-evaluate Compare pino, winston, and bunyan as logging libraries.
```

Evaluate database options with hinted criteria:

```
❯ /ase-meta-evaluate Compare PostgreSQL, MySQL, and SQLite for an
  embedded use case; criteria include footprint and write throughput.
```

## SEE ALSO

`ase-meta-quorum`, `ase-meta-search`, `ase-arch-discover`,
`ase-meta-why`.

ase-meta-diaboli - Play "Devil's Advocate" (Latin: "Advocatus Diaboli")

## NAME

`ase-meta-diaboli` - Play "Devil's Advocate" (Latin: "Advocatus Diaboli")

## SYNOPSIS

`ase-meta-diaboli`
    [`--help`|`-h`]
    [`--count`|`-c` *count*]
    *thesis*

## DESCRIPTION

The `ase-meta-diaboli` skill plays *Devil's Advocate* (Latin:
*Advocatus Diaboli*) by relentlessly challenging or criticising a
supplied *thesis*. It applies a disciplined set of critical-thinking
tenets - steelmanning, stress-testing fundamentals, surfacing implicit
assumptions, demanding proportional evidence, seeking disconfirming
cases, *Reductio Ad Absurdum*, exposing hidden costs, and pre-mortem
thinking - while targeting the claim rather than its proponent and
yielding where the argument genuinely holds.

The skill iterates until it has found at least *count* anti-theses
(counter-arguments) each ranked at least 7 on a 0 (weak) to 10
(strong) Likert scale, reports the top *count* sorted from strongest to
weakest, and finally applies *Hegelian dialectics* (*Thesis* +
*Antithesis* → *Synthesis*) to derive a single-sentence *SYNTHESIS*
that preserves what is true in both the thesis and its antitheses
while discarding what is false.

The `--count`/`-c` *count* option sets the minimum number of strong
anti-theses to surface (default *10*), raising or lowering the floor of
counter-arguments hunted for, sorted, and reported in the single
challenge pass. A `0`, negative, or non-numeric value falls back to the
default *10*.

The intent is constructive: stress-testing the thesis in good faith to
arrive at a better final decision, not obstructing or merely being
contrarian.

## ARGUMENTS

- `--count`,- `-c`- *count*:
Surface at least- *count*strong anti-theses (default- *10*) before
sorting and reporting the top- *count*and deriving the- *SYNTHESIS*. An
invalid or non-positive- *count*reverts to the default- *10*.
 
- *thesis*:
The statement, claim, or position to be relentlessly challenged.
It may be technical, factual, or opinion-based; the skill attacks
its strongest ("steelman") interpretation.
 

## SCENARIOS

- You want a thesis relentlessly challenged with counter-arguments
- You want a decision stress-tested before you commit to it
- You want the strongest anti-theses ranked plus a synthesis
- You want a devil's advocate against a claim or plan

## EXAMPLES

Challenge a technology-choice claim:

```
❯ /ase-meta-diaboli HAPI is the best REST framework
```

Stress-test a design decision:

```
❯ /ase-meta-diaboli We should rewrite the service in Rust.
```

Surface at least fifteen anti-theses:

```
❯ /ase-meta-diaboli --count 15 We should rewrite the service in Rust.
```

## SEE ALSO

`ase-meta-why`, `ase-meta-evaluate`, `ase-meta-quorum`,
`ase-meta-steelman`.

ase-meta-steelman - Build the "Steelman" Argument

## NAME

`ase-meta-steelman` - Build the "Steelman" Argument

## SYNOPSIS

`ase-meta-steelman`
    [`--help`|`-h`]
    [`--count`|`-c` *count*]
    [`--rounds`|`-r` *rounds*]
    *thesis*

## DESCRIPTION

The `ase-meta-steelman` skill builds the strongest possible case
*for* a supplied *thesis* - the constructive mirror of its adversarial
counterpart `ase-meta-diaboli`. It applies a disciplined set of
constructive-thinking tenets - charitable interpretation, strengthening
the fundamentals, surfacing the enabling assumptions, supplying
proportional evidence, seeking confirming cases, *Reductio Ad Bonum*,
surfacing the upside, and pre-parade thinking - while crediting the
claim rather than the proponent's authority and conceding where the
thesis genuinely falls short.

The skill iterates until it has found at least *count* pro-theses
(supporting arguments) each ranked at least 7 on a 0 (weak) to 10
(strong) Likert scale, reports the top *count* sorted from strongest to
weakest, and finally consolidates them (*Thesis* + *Pro-Theses* →
*Fortification*) to derive a single-sentence *FORTIFICATION* that
consolidates everything genuinely strengthening the thesis while
honestly bounding where it holds.

The `--rounds`/`-r` *rounds* option turns the single defense pass into an
*iterative chain* of *rounds* rounds (default *1*): each round's
*FORTIFICATION* becomes the *thesis* of the next round, so the position
is progressively re-fortified and sharpened. A `0`, negative, or
non-numeric value falls back to the default *1*; with a single round the
output is identical to running without the option.

The `--count`/`-c` *count* option sets the minimum number of strong
pro-theses to surface (default *10*), raising or lowering the floor of
supporting arguments hunted for, sorted, and reported in each defense
pass. A `0`, negative, or non-numeric value falls back to the default
*10*.

The intent is constructive: building the best honest case for the
thesis to arrive at a better final decision, not overselling or merely
cheerleading.

## OPTIONS

- `--count`|- `-c`- *count*:
Surface at least- *count*strong pro-theses (default- *10*) per defense
pass before sorting and reporting the top- *count*and deriving the- *FORTIFICATION*. An invalid or non-positive- *count*reverts to the
default- *10*.
 
- `--rounds`|- `-r`- *rounds*:
Run- *rounds*iterative defense rounds (default- *1*), feeding each
round's- *FORTIFICATION*in as the next round's- *thesis*. An invalid
or non-positive- *rounds*reverts to the default- *1*.
 

## ARGUMENTS

- *thesis*:
The statement, claim, or position to be charitably strengthened.
It may be technical, factual, or opinion-based; the skill defends
its strongest ("steelman") interpretation.

## SCENARIOS

- You want the strongest possible case built for a thesis
- You want supporting arguments ranked plus a fortification
- You want a charitable defense before pitching a decision

## EXAMPLES

Strengthen a technology-choice claim:

```
❯ /ase-meta-steelman HAPI is the best REST framework
```

Build the case for a design decision:

```
❯ /ase-meta-steelman We should rewrite the service in Rust.
```

Strengthen across five iterative rounds:

```
❯ /ase-meta-steelman --rounds 5 We should rewrite the service in Rust.
```

## SEE ALSO

`ase-meta-diaboli`, `ase-meta-why`, `ase-meta-evaluate`,
`ase-meta-quorum`.

ase-meta-brainstorm - Collaboratively Brainstorm a Topic

## NAME

`ase-meta-brainstorm` - Collaboratively Brainstorm a Topic

## SYNOPSIS

`ase-meta-brainstorm`
    [`--help`|`-h`]
    [`--max-clarify`|`-c=3`]
    [`--min-ideas`|`-i=12`]
    [`--min-rank`|`-r=7`]
    [`--max-shortlist`|`-s=4`]
    *topic*

## DESCRIPTION

The `ase-meta-brainstorm` skill guides a collaborative ideation
session on a *topic* *before* any implementation begins. It first
*clarifies intent* by exploring the project context and interviewing
the user - one grounded, multiple-choice question at a time - about
purpose, constraints, scope, and success criteria (asking at most
`--max-clarify`, default 3, questions). It then *diverges* into a broad
space of candidate ideas (at least `--min-ideas`, default 12) pursued from
deliberately diverse angles (MVP-first, risk-first, UX-first,
reuse-first, and wildcard), without judging them.

Next it *converges* by clustering the ideas into coherent themes,
pruning speculative or out-of-scope ones via *YAGNI*, and scoring the
survivors on a 0-10 fit scale (keeping only those ranked `--min-rank`,
default 7, or higher). Finally it distills a *shortlist* of the top
`--max-shortlist`, default 4, directions - drawn from distinct clusters
wherever possible - and derives a single *RECOMMENDATION*, being either
the highest-ranked option or a principled synthesis of the shortlist.

## OPTIONS

- `--max-clarify`|- `-c=3`:
The- *maximum*number of essential-unknown clarification questions
asked in the- *clarify intent*phase (default: 3). Lower it for a
faster, leaner intake, raise it for more upfront grounding.
 
- `--min-ideas`|- `-i=12`:
The- *minimum*number of candidate ideas to generate in the- *diverge*phase before converging (default: 12). Raise it for a broader idea
space, lower it for a quicker, narrower session.
 
- `--min-rank`|- `-r=7`:
The- *minimum*0-10 fit rank an idea must score to survive the- *converge*phase (default: 7). Raise it for a stricter filter, lower
it to retain more ideas.
 
- `--max-shortlist`|- `-s=4`:
The- *maximum*number of distilled options on the final- *shortlist*(default: 4). Lower it for a sharper focus, raise it for more
finalists.
 

## ARGUMENTS

- *topic*:
The subject to brainstorm - a feature, component, behavior, or
design question to explore- *what*to build before- *how*.

## SCENARIOS

- You want to get fresh ideas for an open-ended question
- You want to explore what to build before deciding how to build it
- You want to widen a solution space that holds a single candidate
- You want to get a shortlist of distinct directions with a recommendation
- You want to ideate on a feature, component, or design question

## EXAMPLES

Brainstorm an approach for a new feature:

```
❯ /ase-meta-brainstorm an offline-first sync layer for the mobile app
```

Brainstorm with a broader idea space of at least 20 candidates:

```
❯ /ase-meta-brainstorm --min-ideas 20 an offline-first sync layer
```

Brainstorm quickly with a single clarification, a stricter score floor,
and a tighter shortlist:

```
❯ /ase-meta-brainstorm --max-clarify 1 --min-rank 8 --max-shortlist 3 an offline-first sync layer
```

## SEE ALSO

`ase-meta-evaluate`, `ase-meta-quorum`, `ase-meta-diaboli`.

ase-meta-search - Search the Internet/Web

## NAME

`ase-meta-search` - Search the Internet/Web

## SYNOPSIS

`ase-meta-search`
    [`--help`|`-h`]
    [`--services`|`-s`=(`all`|`perplexity`|`brave`|`exa`|`websearch`)...]
    *query*

## DESCRIPTION

The `ase-meta-search` skill searches the *Internet*/*Web* for the
given *query*. It dispatches the query in parallel to the *selected*
search services (Perplexity, Brave, Exa, and Claude's built-in
`WebSearch`) via the `ase:ase-meta-search` sub-agent and consolidates
all responses into a single answer without removing original
information.

This skill should be preferred over directly invoking Perplexity,
Brave, or `WebSearch` individually.

## OPTIONS

- `--services`|- `-s`=(- `all`|- `perplexity`|- `brave`|- `exa`|- `websearch`)...:
The comma-separated list of search backends to query. The default- `all`queries every available backend;
otherwise only the listed backends are queried.

## ARGUMENTS

- *query*:
The search query to dispatch to the search services.

## SCENARIOS

- You want the Internet/Web searched for a query
- You want results from several search backends consolidated
- You want current facts which model knowledge may lack

## EXAMPLES

Search the Web for a topic across all backends:

```
❯ /ase-meta-search latest stable release of TypeScript and release notes
```

Search the Web using only the Brave and Exa backends:

```
❯ /ase-meta-search --services=brave,exa latest stable release of TypeScript
```

## SEE ALSO

`ase-meta-chat`, `ase-meta-quorum`, `ase-arch-discover`.

ase-meta-quorum - Query Multiple AIs for Quorum Answer

## NAME

`ase-meta-quorum` - Query Multiple AIs for Quorum Answer

## SYNOPSIS

`ase-meta-quorum`
    [`--help`|`-h`]
    [`--models`|`-m` *model*[,...]]
    *question*

## DESCRIPTION

The `ase-meta-quorum` skill finds a *quorum answer* on an arbitrary
question by querying *multiple* AIs (Anthropic Claude itself plus
OpenAI ChatGPT, Google Gemini, DeepSeek, xAI Grok, Z.AI GLM, and
Alibaba Qwen) for an *optimal consensus*.

The skill first previews its own answer, then dispatches the same
query to each available foreign LLM via the `ase:ase-meta-chat`
sub-agent, summarizes all responses, derives a *consensus rate* on
a Likert scale of `0..N` (where `N` is the number of available
responders), and reports the consensus answer alongside the
complete, unmodified individual responses.

## OPTIONS

- `--models`|- `-m`- *model*[,...]:
Restrict the- *foreign*LLMs that are queried to the given
comma-separated list of- *model*tokens. Recognized tokens are- `all`,- `chatgpt`,- `gemini`,- `deepseek`,- `grok`,- `glm`,- `qwen`,- `claude`,- `codex`, and- `copilot`, where- `all`selects every API-based
model, but- *not*the harness-based- `claude`,- `codex`, and- `copilot`models, which are queried only when given explicitly. The default
is- `all`. Models that are not selected (or not available) are- *silently skipped*.
Anthropic Claude (the skill itself) is- *always*included,
independent of this option.

## ARGUMENTS

- *question*:
The question to ask all available AIs.

## SCENARIOS

- You want a consensus answer from multiple AIs
- You want to know whether different models agree on a question
- You want the individual responses plus a consensus rate

## EXAMPLES

Ask the quorum a factual question:

```
❯ /ase-meta-quorum What are the most common causes of memory leaks in Node.js?
```

Ask only a subset of the foreign models:

```
❯ /ase-meta-quorum --models chatgpt,gemini,grok Which sorting algorithm is fastest in practice?
```

## SEE ALSO

`ase-meta-chat`, `ase-meta-search`, `ase-meta-evaluate`.

ase-meta-proximity - Determine the Conceptual Proximity of a Topic

## NAME

`ase-meta-proximity` - Determine the Conceptual Proximity of a Topic

## SYNOPSIS

`ase-meta-proximity`
    [`--help`|`-h`]
    [`--ground`|`-g`]
    [`--loop`|`-l`]
    *topic*

## DESCRIPTION

The `ase-meta-proximity` skill determines the *conceptual proximity* of
the supplied *topic* along three dimensions and reports the neighboring
topics grouped into three labeled sections: *PARENT* (the single broader
topic that *topic* specializes), *SIBLINGS* (the four most relevant
topics on the same level, sharing the same parent), and *CHILDREN* (the
four most relevant narrower topics that specialize *topic*).

The determination itself is delegated to the `ase-meta-proximity` agent,
which is shared with the `ase-meta-quotes` skill. By default the
proximity is derived from model knowledge only. With the `--ground`/`-g`
option, the agent first searches the Internet/Web for facts about the
topic, about its broader topic, and about its narrower topics, and
grounds the determination in the found facts.

Without the `--loop`/`-l` option, the skill determines and prints the
proximity of the given topic once and then stops. With the `--loop`/`-l`
option, the skill instead presents the nine neighboring topics in an
interactive dialog; selecting one of them makes it the new *current
topic* and restarts the determination from the beginning, so the user
can *navigate* the topic taxonomy up (parent), sideways (siblings), and
down (children). Cancelling the dialog exits the loop. When `--loop` is
combined with `--ground`, each newly selected topic is re-grounded by
the `ase-meta-proximity` sub-agent before its proximity is re-determined.

## OPTIONS

- `--ground`|- `-g`:
Ground the determination in Internet/Web facts gathered by the- `ase-meta-proximity`agent before determining the proximity. Without
this option, the determination is derived from model knowledge only.
When combined with- `--loop`, every newly navigated-to topic is
re-grounded.
 
- `--loop`|- `-l`:
Present the neighboring topics in an interactive dialog and, upon
selection, adopt the picked topic as the new current topic and
restart the determination. Without this option, the proximity of the
given topic is printed once and the skill stops.
 

## ARGUMENTS

- *topic*:
The topic whose conceptual proximity is to be determined. It may be
a technical concept, a phenomenon, or any other subject; the skill
determines its parent, sibling, and child topics.

## SCENARIOS

- You want the conceptual neighborhood of a topic explored
- You want the parent, sibling, and child topics of a subject
- You want a topic taxonomy navigated interactively
- You want related topics grounded in Internet/Web facts

## EXAMPLES

Determine the proximity of a topic from model knowledge:

```
❯ /ase-meta-proximity Fourier transform
```

Interactively navigate the conceptual neighborhood, grounded in
Internet/Web facts:

```
❯ /ase-meta-proximity --ground --loop Agentic Software Engineering
```

## SEE ALSO

`ase-meta-quotes`, `ase-meta-eli5`, `ase-meta-search`, `ase-code-explain`.

ase-meta-chat - Query Foreign LLM for Chat

## NAME

`ase-meta-chat` - Query Foreign LLM for Chat

## SYNOPSIS

`ase-meta-chat`
    [`--help`|`-h`]
    *llm* *query*

## DESCRIPTION

The `ase-meta-chat` skill queries a *foreign LLM* (such as OpenAI
ChatGPT, Google Gemini, DeepSeek, xAI Grok, Z.AI GLM, or Alibaba
Qwen) with a single chat message. The underlying `ase:ase-meta-chat`
sub-agent is invoked and its plain response is shown back verbatim
without any further interpretation.

## ARGUMENTS

- *llm*:
Identifier of the foreign LLM to query, e.g.- `chatgpt`,- `gemini`,- `deepseek`,- `grok`,- `glm`, or- `qwen`.
 
- *query*:
The chat message to send to the foreign LLM.
 

## SCENARIOS

- You want a single question answered by one specific foreign LLM
- You want an answer from ChatGPT, Gemini, DeepSeek, Grok, GLM, or Qwen
- You want a second opinion from exactly one other model, verbatim

## EXAMPLES

Ask ChatGPT a quick question:

```
❯ /ase-meta-chat chatgpt What is the best way to handle UTF-8 BOMs in JSON?
```

Ask Gemini for a brief comparison:

```
❯ /ase-meta-chat gemini Compare gRPC and REST in one paragraph.
```

## SEE ALSO

`ase-meta-quorum`, `ase-meta-search`.

ase-meta-quotes - Find Quotes on a Topic

## NAME

`ase-meta-quotes` - Find Quotes on a Topic

## SYNOPSIS

`ase-meta-quotes`
    [`--help`|`-h`]
    [`--ground`|`-g`]
    [`--proximity`|`-p`]
    [`--count`|`-c` *count*]
    *topic-keywords*

## DESCRIPTION

The `ase-meta-quotes` skill finds *quotes* -- sayings, aphorisms,
maxims, proverbs, and citations -- for the supplied *topic-keywords* and
places them into a *2x2 matrix*, reported as four labeled *quadrant*
sections.

Per quote, the skill records its *text*, its *author* (a named person or
organization, if known), its *origin* (a named work, standard, or
document, if known), and the topic it was harvested for (its *source
topic*).

The first dimension of the matrix is the *attribution* of a quote: a
quote is `ATTRIBUTED` when a named author and/or a named origin is known
for it, and `ANONYMOUS` otherwise. The second dimension is the
*literalness* of a quote: a quote is `LITERAL` when its text contains at
least one of the *topic-keywords* as a whole word -- matched
case-insensitively and tolerating inflections (e.g. `architect` and
`architectural` match the keyword `architecture`), but never as a mere
substring (e.g. `art` does not match `architecture`) -- and `THEMATIC`
otherwise. The two dimensions span the quadrants `Q1`
(`ATTRIBUTED`/`LITERAL`), `Q2` (`ATTRIBUTED`/`THEMATIC`), `Q3`
(`ANONYMOUS`/`LITERAL`), and `Q4` (`ANONYMOUS`/`THEMATIC`). A quadrant
without any quote is rendered as `(none)`.

Every quote is rendered on its own line and carries, where applicable,
the suffixes `— <author>, <origin>` (in the two `ATTRIBUTED`
quadrants, omitting whichever part is unknown),
`[from proximity: <source-topic>]` (when the quote was contributed by a
neighborhood topic under `--proximity` instead of by *topic-keywords*
itself), and
`(unverified)` (when the exact wording or the attribution could not be
established with confidence). The `(unverified)` marker is dropped as
soon as the Internet/Web search under `--ground` confirms the wording and
the attribution of the quote.

## OPTIONS

- `--ground`|- `-g`:
Gather quotes from the Internet/Web via the- `ase-meta-search`skill
(dispatched in a sub-agent, querying all available search backends)- *in addition to*-- and never- *instead*of -- the model knowledge,
merge them into the harvest while deduplicating quotes which differ
only in punctuation, capitalization, or attribution wording, and use
the search results to confirm the wording and the attribution of the
found quotes. When combined with- `--proximity`, the determination of
the conceptual neighborhood is grounded in Internet/Web facts as
well. Should the search return no usable quotes, a warning is emitted
and the skill falls back to the model knowledge only. Without this
option, the quotes are derived from model knowledge only.
 
- `--proximity`|- `-p`:
Widen the harvest beyond the given topic to its- *conceptual
neighborhood*-- the parent (more general) topic, the four sibling
(same-level) topics, and the four child (more specialized) topics --
as determined by the- `ase-meta-proximity`agent, which is shared with
the- `ase-meta-proximity`skill. Each of these nine neighborhood
topics is harvested exactly like the given topic itself. Quotes
contributed by a neighborhood topic carry that topic in brackets and
mostly land in the- `THEMATIC`quadrants, which is exactly where the
widening pays off. Should the agent return no usable neighborhood, a
warning is emitted and the skill continues with the narrow topic
only. Without this option, only the given topic is harvested.
 
- `--count`|- `-c`- *count*:
Maximum total number of quotes across all four quadrants (default:- `8`, i.e. about two quotes per quadrant). The quotes are distributed
as evenly as possible across the quadrants, preferring the most
relevant and most well-known quote per quadrant. A non-numeric value
or a value less than or equal to- `0`falls back to the default.
 

## ARGUMENTS

- *topic-keywords*:
The topic keywords the quotes are searched for. They act both as the
search query and as the keyword set of the- *literalness*dimension of
the matrix.

## SCENARIOS

- You want quotes, sayings, or aphorisms on a topic
- You want citations arranged by attribution and literalness
- You want quote wording and attribution verified against the Web
- You want the quote harvest widened to related topics

## EXAMPLES

Find quotes from model knowledge:

```
❯ /ase-meta-quotes software architecture
```

Find a wider set of quotes, grounded in Internet/Web facts and widened
to the conceptual neighborhood of the topic:

```
❯ /ase-meta-quotes --ground --proximity --count 12 technical debt
```

## SEE ALSO

`ase-meta-proximity`, `ase-meta-search`, `ase-meta-eli5`.

ase-meta-mint - Mint an Identifier or Name

## NAME

`ase-meta-mint` - Mint an Identifier or Name

## SYNOPSIS

`ase-meta-mint`
    [`--help`|`-h`]
    [`--type`|`-t` `uuid`|`sha1`|`nanoid`|`const`|`var`|`class`|`func`|`path`|`name`]
    [`--count`|`-c` *count*]
    [*hint*]

## DESCRIPTION

The `ase-meta-mint` skill *mints* an identifier or a name of a requested
type out of the free-text *hint* formed by the positional arguments. It
dispatches on `--type` and emits the generated strings as its result.

The three *hash-derived* types are delegated to the `ase_mint` tool of the
`ase` MCP server, which derives them reproducibly instead of guessing
them:

- `uuid`:
a deterministic- *UUID V5*over the- *hint*, within the fixed- *ASE*namespace (itself a- *UUID V5*over- `https://ase.tools`in the
standard URL namespace), or a random- *UUID V4*if no- *hint*is
given.
 
- `sha1`:
the 40-character hexadecimal- *SHA-1*digest of the- *hint*.
 
- `nanoid`:
the 10-character,- *NanoId*-style- *base62*hash of the- *hint*(its- *SHA-1*digest,- *base62*-encoded and truncated), a short but still
reproducible identifier.
 

The six *linguistic* types are derived by the skill itself: the *hint*
is distilled into its essential words, which are then assembled
according to the conventions of the type:

- `const`:
an- `UPPER_SNAKE_CASE`constant identifier, such as- `FOO_BAR_QUUX`.
 
- `var`:
a- `lowerCamelCase`variable identifier, such as- `fooBarQuux`, whose- *last*part is a substantive naming what the value- *is*, never a
verb.
 
- `func`:
a- `lowerCamelCase`function identifier, such as- `fooBarQuux`, whose- *first*part is a verb naming what the function- *does*.
 
- `class`:
an- `UpperCamelCase`class identifier, such as- `FooBarQuux`, as a
singular substantive phrase naming the modeled entity.
 
- `path`:
a- `kebab-case`path component, such as- `foo-bar-quux`.
 
- `name`:
an- `UpperCamelCase`- *product name*, such as- `FooBarQuux`, optimized
for a brand rather than for a description: short, pronounceable,
memorable, and distinctive.
 

A *hint* is mandatory for every type except `uuid`, which falls back to
a random *UUID V4* without one.

## OPTIONS

`--type`|`-t` `uuid`|`sha1`|`nanoid`|`const`|`var`|`class`|`func`|`path`|`name`:
    Select the type of the minted identifier or name (default: `uuid`).

`--count`|`-c` *count*:
    Mint *count* identifiers instead of just one (default: `1`, maximum:
    `100`). For the hash-derived types it is honored only for an empty
    *hint*, where *count* random *UUID V4* values are emitted; with a
    non-empty *hint*, hashing is a pure function of that *hint* and
    hence always yields exactly one identifier, so *count* is clamped to
    `1` and a warning is emitted. For the linguistic types, *count*
    distinct candidates are emitted, ordered best-first.

## ARGUMENTS

*hint*:
    The free-text hint the identifier or name is derived from. It may be
    a short description, a phrase, or a few keywords. It is mandatory
    for all types except `uuid`.

## SCENARIOS

- You need a stable, reproducible UUID, SHA-1 digest, or short hash for a concept
- You need a well-formed constant, variable, function, or class identifier
- You need a path component or slug derived from a description
- You need a catchy product name for a new component or tool

## EXAMPLES

Mint a deterministic UUID for a concept:

```
❯ /ase-meta-mint --type uuid the ASE artifact kind resolution cache
```

Mint a function identifier from a description:

```
❯ /ase-meta-mint -t func compute the checksum of an artifact file
```

Mint three product name candidates:

```
❯ /ase-meta-mint -t name -c 3 a background service bridging a project into the agent tool
```

## SEE ALSO

`ase-task-id`, `ase-meta-brainstorm`, `ase-task-edit`, `ase-arch-discover`.

ase-meta-workflow - Generate a Workflow Skill

## NAME

`ase-meta-workflow` - Generate a Workflow Skill

## SYNOPSIS

`ase-meta-workflow`
    [`--help`|`-h`]
    [`--scope`|`-s` `local`|`user`]
    [`--force`|`-f`]
    *skill-name*
    *workflow-description*

## DESCRIPTION

The `ase-meta-workflow` skill *generates* a new, regular skill of the
currently used *agent tool* -- written in the style of *ASE* skills --
which orchestrates the workflow given as *workflow-description*.

The *workflow-description* is treated as a combination of *sequential*
actions, *concurrent* actions, *sub-agent* invocations, *skill*
invocations, and arbitrary other action statements. The skill maps them
onto the ASE control constructs: top-level sequential actions become
numbered `<step>` items inside a single `<flow>`, concurrent actions
become a `<parallel>` block, sub-agent invocations become `<agent>`
elements, and skill invocations become `<skill>` elements. Whenever a
`<parallel>` block contains at least one `<agent isolation="worktree">`,
a dedicated consolidation step holding `<agent-consolidation/>` is
appended right after it, so the Git WorkTrees of the concurrent
sub-agents are merged and removed again. Every top-level `<skill>`
invocation is additionally enclosed in its own `<agent>` element --
without `isolation` and with `run_in_background=false` -- so the
`TaskCreate` and `TaskUpdate` tool calls of the called skill cannot
interfere with the task tracking of the generated workflow skill itself.

To keep the emitted `<skill>` invocations correct, the skill consults the
accumulated manual pages of all ASE skills and takes every option and
argument verbatim from them. To keep the emitted flow meaningful, it
consults the *ASE workflow graph* -- a list of `<from> -> <to>`
transitions between ASE skills and logical states such as `START`,
`SKETCH`, `TASK`, `ARTIFACT`, and `END`. That graph *guides* the
derivation but never restricts it: arbitrary non-ASE actions and
unlisted transitions stay allowed.

The generated skill is *independent* of the ASE plugin installation path:
it loads the ASE meta definitions through the `ase util meta` command
instead of plugin-relative includes.

The skill is *portable* across the supported agent tools, because
*Anthropic Claude Code*, *GitHub Copilot*, and *OpenAI Codex* differ in
three relevant ways, each of which is dispatched on the currently used
agent tool:

- the - *target directory*, because every agent tool discovers its skills
in its own locations,
 
- the - *frontmatter fields*, because every agent tool accepts its own
field set and its own- `allowed-tools`permission grammar, and
 
- the - *preamble*, because only- *Anthropic Claude Code*expands the- `!`...`construct before the skill content reaches the model. The
other agent tools receive it verbatim and would silently leave it
unexpanded, so they instead get an explicit instruction to run- `ase util meta ...`themselves before anything else.
 

## OPTIONS

- `--scope`|- `-s`- `local`|- `user`:
Where the generated skill is stored. With- `local`(the default),
it is written into the current project, where it can be committed
and reviewed alongside the code. With- `user`, it is written into the
personal skill directory, where it is available across all projects.
The concrete directory depends on the currently used agent tool:- `.claude/skills/`and- `~/.claude/skills/`under- *Anthropic Claude
Code*,- `.github/skills/`and- `~/.copilot/skills/`under- *GitHub
Copilot*, and- `.agents/skills/`and- `~/.codex/skills/`under- *OpenAI
Codex*.
 
- `--force`|- `-f`:
Overwrite an already existing skill. Without this option, the skill
refuses with an error as soon as the target- `SKILL.md`exists, so
hand-edited skills are never lost silently.
 

## ARGUMENTS

- *skill-name*:
Name of the skill to generate, matching- `^[a-zA-Z][a-zA-Z0-9_-]*$`.
It is used as the directory name, as the frontmatter- `name`, and as
the resulting- `/`- *skill-name*command (respectively the- `$`- *skill-name*mention under- *OpenAI Codex*).
 
- *workflow-description*:
Free-text description of the workflow the generated skill should
perform. Mentioning that actions run "in parallel", that they are
dispatched "in a sub-agent", or naming concrete- `ase-xxx-xxx`skills
steers the derived structure accordingly.
 

## SCENARIOS

- You want a recurring multi-step procedure automated as its own skill
- You want several ASE skills orchestrated or chained into one command
- You want sequential and parallel actions woven into a generated skill
- You want a new agent tool skill generated in the style of ASE skills

## EXAMPLES

Generate a project-local skill which analyzes the code and then resolves
every finding concurrently in isolated worktrees:

```
❯ /ase-meta-workflow optimizer run ase-code-analyze, then for each finding P<n> resolve, implement and delete it in a parallel isolated sub-agent
```

Generate a user-wide skill and overwrite a previous generation:

```
❯ /ase-meta-workflow --scope user --force release lint the code, update the changelog, then propose a commit message
```

## SEE ALSO

`ase-help-intent`, `ase-help-skill`,
`ase-code-analyze`, `ase-task-edit`, `ase-task-implement`.

ase-arch-analyze - Review Software Architecture

## NAME

`ase-arch-analyze` - Review Software Architecture

## SYNOPSIS

`ase-arch-analyze`
    [`--help`|`-h`]
    [`--prefix`|`-P` *prefix*]
    *source-reference*

## DESCRIPTION

The `ase-arch-analyze` skill reviews the *software architecture* of
the referenced source code, including *package cohesion* and
*inter-package coupling*, for *potential problems* across component boundaries,
structural organization, architecture principles, interface quality,
quality attributes, and architecture governance.

The skill investigates 21 architecture quality aspects across 7 thematic
blocks (component boundaries, structural organization, architecture
principles, interface quality, quality attributes, architecture
governance, and package cohesion), renders a high-level architecture
diagram, and reports findings as either `PROBLEM` or `TRADEOFF` entries
based on a built-in tension matrix.

The `--prefix`|`-P` *prefix* option prefixes every reported finding id
with *prefix* and a hyphen, so `P1` becomes `<prefix>-P1` and `T1`
becomes `<prefix>-T1`, with the persisted keys becoming
`ase-issue-<prefix>-P1` and `ase-issue-<prefix>-T1` accordingly. The
purge of stale results is narrowed to the same namespace, so analyses
run under *distinct* prefixes coexist instead of overwriting each other.
Without the option (the default), ids stay unprefixed and the purge
covers the *entire* `ase-issue-*` space, including any prefixed results.

## ARGUMENTS

- *source-reference*:
A file, directory, or other reference to the source code that
is to be analyzed architecturally.

## SCENARIOS

- You want the software architecture of your code base reviewed
- You want coupling and cohesion problems between packages found
- You want an architecture diagram plus PROBLEM and TRADEOFF findings
- You want architecture findings persisted for later resolution

## EXAMPLES

Analyze architecture of the current project:

```
❯ /ase-arch-analyze src/
```

Analyze a specific module:

```
❯ /ase-arch-analyze src/core
```

Analyze a module under its own id namespace, yielding `core-P1`, `core-T1`, ...:

```
❯ /ase-arch-analyze --prefix core src/core
```

## SEE ALSO

`ase-arch-discover`, `ase-code-analyze`, `ase-task-edit`,
`ase-code-edit`, `ase-code-insight`.

ase-arch-discover - Discover Components

## NAME

`ase-arch-discover` - Discover Components

## SYNOPSIS

`ase-arch-discover`
    [`--help`|`-h`]
    [`--limit`|`-l=12`]
    [`--staleness`|`-s=18`]
    [`--small-scope`|`-S`]
    *functionality*

## DESCRIPTION

The `ase-arch-discover` skill discovers additional, *third-party
components* (libraries/frameworks) for the technology stack that
provide the needed *functionality*.

The skill determines the project's technology stack (TypeScript,
JavaScript, Kotlin, or Java), derives essential keywords from the
requested functionality, queries the corresponding package registry
(NPM or Maven Central), retrieves metadata (version, downloads,
stars, dates, dependency count) via the `ase_component_info` MCP tool,
and reports the top-ranked components as a Markdown table together with
a single distinguishing hint (USP, Crux, or Gotcha) per component.

The ranking penalizes *stale* components (last release older than the
`--staleness` threshold) and, when the `--small-scope` option is given,
demotes dependency-heavy components.

## OPTIONS

- `--limit`|- `-l=12`:
The- *maximum*number of components retained in the final ranking
(default: 12); each discovery source may fetch up to twice this
number of candidates for ranking. Raise it for a broader, more
exhaustive survey, lower it for a quicker, narrower lookup.
 
- `--staleness`|- `-s=18`:
The- *staleness threshold*in months (default: 18). A component whose
last release is older than this is rank-penalized and flagged with an- `aging`gotcha; older than- *twice*the threshold, it is penalized
harder and flagged as- `stale/abandoned`.
 
- `--small-scope`|- `-S`:
Treat the requested- *functionality*as- *small-scope*(default: off).
When enabled, the ranking demotes dependency-heavy components by a
dependency-weight penalty, since a dependency-free/hand-rolled
implementation is a realistic alternative for narrow, self-contained
functionality.
 

## ARGUMENTS

- *functionality*:
A short description of the desired functionality the third-party
component should provide.

## SCENARIOS

- You want a third-party library or framework for a needed functionality
- You want a ranked survey of candidate components from NPM or Maven Central
- You want to know whether a package is healthy or stale and abandoned
- You want to decide between a dependency and hand-rolling a small feature

## EXAMPLES

Discover components for JSON schema validation:

```
❯ /ase-arch-discover JSON schema validation
```

Discover components for HTTP client functionality:

```
❯ /ase-arch-discover HTTP client with retries
```

Discover a broader set of up to 20 HTTP client components:

```
❯ /ase-arch-discover --limit 20 HTTP client with retries
```

Discover HTTP clients, flagging any without a release in the last 12 months:

```
❯ /ase-arch-discover --staleness 12 HTTP client with retries
```

## SEE ALSO

`ase-arch-analyze`, `ase-meta-search`, `ase-meta-evaluate`.

ase-docs-shorten - Shorten a Document to a Target Length

## NAME

`ase-docs-shorten` - Shorten a Document to a Target Length

## SYNOPSIS

`ase-docs-shorten`
    [`--help`|`-h`]
    [`--auto`|`-a`]
    (`--chars`|`-c` *N* | `--words`|`-w` *N*)
    *docs-reference*

## DESCRIPTION

The `ase-docs-shorten` skill reduces the length of a *single* document
until it reaches a requested *target length*. The investigation is
dispatched to a sub-agent (`ase:ase-docs-shorten`) so that scanning
details do not leak into the user-visible transcript.

The target length is stated either as `--chars` *N* or as `--words` *N*.
The two options are *mutually exclusive*, exactly *one* of them is
*mandatory*, and neither carries a default, because the target length is
the very quantity the shortening optimizes for. A document reference
which expands to *more than one* document is rejected, as it would leave
open whether the target bounds each document or their sum.

The shortening runs as a *three-stage cascade*, and a stage is entered
*only* while the target is still not met:

- *Tighten*: shorten individual sentences without losing any content -
filler words struck, nominal constructions turned into verbs,
redundant restatements within a sentence cut.
 
- *Drop*: remove low-value content - redundant restatements,
illustrative examples, asides, parentheticals, and the whole
sentences or list items which carry the least value for the reader.
Content is genuinely lost in this stage.
 
- *Compress*: merge and rewrite the remaining content into shorter
expressions and sentences - related sentences fused, prose
enumerations turned into compact lists, multi-sentence passages
re-expressed as single sentences.
 

The skill never shortens *beyond* the target: it stops at the first
change which reaches it. What *survives* the shortening stays exact -
every surviving fact, number, technical term, qualifier, and negation is
preserved verbatim, nothing is invented, the order of the surviving
argument is kept, and fenced code blocks, inline code spans, link
targets, Markdown frontmatter, and the heading structure are never
touched.

Each proposed change is cut as a *whole block* - a paragraph, a list
item, a blockquote, or a heading with its following paragraph - because
a shortening is only judgeable in the context of the whole block it
rewrites. It is rendered as a *unified diff* with up to two lines of
context, annotated with the length it saves, exactly as in
`ase-docs-refine` and
`ase-docs-proofread`. The user
either accepts or rejects each block interactively (or refines it via a
free-text hint, which re-proposes the block without limit) or - with
`--auto` - applies all of them automatically. A closing *LENGTH REPORT*
states the achieved length against the target, so rejected blocks stay
visible in the outcome.

`ase-docs-shorten` is the length-reducing member of the document triple.
The recommended order over one document is `ase-docs-shorten` first (cut
the bulk), then `ase-docs-refine` (polish
what survives), then
`ase-docs-proofread` (final correctness
pass), so no polishing effort is spent on text which is later dropped.

## OPTIONS

- `--chars`|- `-c`- *N*:
Shorten the document to at most- *N*characters, counted over the
entire document content including whitespace and newlines.
Mutually exclusive with- `--words`.
 
- `--words`|- `-w`- *N*:
Shorten the document to at most- *N*whitespace-separated words,
counted over the entire document content.
Mutually exclusive with- `--chars`.
 
- `--auto`|- `-a`:
Automatically apply every proposed shortening without asking the
user via the interactive dialog.
 

## ARGUMENTS

- *docs-reference*:
A reference to the- *single*document to shorten. A reference
expanding to more than one document is rejected.

## SCENARIOS

- You want a README section cut down to a hard character budget
- You want an abstract brought under a word limit for a submission
- You want the least valuable examples and asides dropped first
- You want to see what a shortening costs before accepting it
- You want a whole document shortened unattended in one pass
- You want the achieved length reported against the target

## EXAMPLES

Shorten a document to at most 4000 characters interactively:

```
❯ /ase-docs-shorten --chars 4000 README.md
```

Shorten a document to at most 250 words unattended:

```
❯ /ase-docs-shorten --auto --words 250 docs/abstract.md
```

## SEE ALSO

`ase-docs-refine`, `ase-docs-proofread`, `ase-docs-distill`.

ase-docs-refine - Refine Document Wording

## NAME

`ase-docs-refine` - Refine Document Wording

## SYNOPSIS

`ase-docs-refine`
    [`--help`|`-h`]
    [`--auto`|`-a`]
    *docs-reference*

## DESCRIPTION

The `ase-docs-refine` skill performs a *light rewriting* of awkward or
unclear sentences in the referenced documents, *without reshaping the
argument*: restructuring sentences for rhythm and clarity, making
enumerations parallel, cutting filler and redundancy, fixing awkward
transitions, and adjusting voice. *Content, numbers, and technical
terms stay exactly as they are.* The investigation is dispatched to a
sub-agent (`ase:ase-docs-refine`) so that scanning details do not leak
into the user-visible transcript.

The checked refinement types are:

- *Structure*: overlong, convoluted, or awkwardly built sentences,
split or restructured for rhythm and clarity.
 
- *Nominalization*: cumbersome nominal constructions, replaced with
plain verbs.
 
- *Parallelism*: non-parallel enumerations, brought into one
grammatical shape - a list whose items mix noun phrases,
imperatives, and full sentences, a heading set mixing gerunds and
nouns, or an inline series whose members do not share their form.
 
- *Filler*: filler words, empty intensifiers, and redundant
restatements, struck.
 
- *Transition*: abrupt or misleading transitions between sentences or
paragraphs, smoothed. An existing transition is only smoothed or
removed - a connective is- *never*inserted where the original
asserted no such relation, as that would add a logical claim.
 
- *Voice*: needless passive voice turned active, needless hedging
struck, and shifts in grammatical person or mood aligned with the
rest of the document.
 

The rewriting always stays *within* a sentence. Dropping a whole
sentence, paragraph, example, or list item for the sake of brevity is
*length reduction*, not refinement, and belongs to
`ase-docs-shorten`. The recommended order
over one document is `ase-docs-shorten` first (cut the bulk), then
`ase-docs-refine` (polish what survives), then `ase-docs-proofread`
(final correctness pass), so no polishing effort is spent on text which
is later dropped.

For each detected problem, the skill renders a unified-diff
*REFINEMENT* preview and either asks the user to `ACCEPT` or `REJECT`
the proposed refinement interactively (or refine it via a free-text
hint, which re-proposes the refinement without limit) or - with
`--auto` - applies all refinements automatically. A proposal which
shifts the meaning even slightly - a lost negation, a changed
qualifier, a renamed technical term - is dropped instead of applied.

## OPTIONS

- `--auto`|- `-a`:
Automatically apply every proposed refinement without asking the
user via the interactive dialog.

## ARGUMENTS

- *docs-reference*:
A file, directory, or other reference to the documents to
refine.

## SCENARIOS

- You want long, convoluted sentences broken into readable ones
- You want nominal style like "perform a validation of" turned into verbs
- You want a bullet list of mixed noun phrases and sentences made parallel
- You want filler words and redundant restatements struck
- You want abrupt transitions between paragraphs smoothed
- You want needless passive voice or hedging adjusted
- You want the wording polished without touching facts or figures

## EXAMPLES

Refine a single document interactively:

```
❯ /ase-docs-refine README.md
```

Refine an entire documentation directory automatically:

```
❯ /ase-docs-refine --auto docs/
```

## SEE ALSO

`ase-docs-shorten`, `ase-docs-proofread`, `ase-docs-distill`.

ase-docs-proofread - Proofread Documents

## NAME

`ase-docs-proofread` - Proofread Documents

## SYNOPSIS

`ase-docs-proofread`
    [`--help`|`-h`]
    [`--auto`|`-a`]
    *docs-reference*

## DESCRIPTION

The `ase-docs-proofread` skill analyzes the referenced documents for
*spelling*, *capitalization*, *punctuation*, *word break*, and *grammar*
errors and proposes corrections. The investigation is dispatched to a
sub-agent (`ase:ase-docs-proofread`) so that scanning details do not
leak into the user-visible transcript.

For each detected problem, the skill renders a unified-diff
*CORRECTION* preview and either asks the user to `ACCEPT` or `REJECT`
the proposed correction interactively (or refine it via a free-text
hint, which re-proposes the correction without limit) or - with
`--auto` - applies all corrections automatically.

`ase-docs-proofread` is the correctness-fixing member of the document
triple. The recommended order over one document is
`ase-docs-shorten` first (cut the bulk),
then `ase-docs-refine` (polish what
survives), then `ase-docs-proofread` (final correctness pass), so no
correction effort is spent on text which is later dropped.

## OPTIONS

- `--auto`|- `-a`:
Automatically apply every proposed correction without asking the
user via the interactive dialog.

## ARGUMENTS

- *docs-reference*:
A file, directory, or other reference to the documents to
proofread.

## SCENARIOS

- You want documents checked for spelling, capitalization,
punctuation, word break, and grammar
- You want inconsistent capitalization of terms and headings unified
- You want wrong word breaks like "data base" or "check-list" repaired
- You want corrections proposed which you accept or reject one by one
- You want a whole documentation directory corrected automatically
- You want a final language pass over a text before publishing

## EXAMPLES

Proofread a single document interactively:

```
❯ /ase-docs-proofread README.md
```

Proofread an entire documentation directory automatically:

```
❯ /ase-docs-proofread --auto docs/
```

## SEE ALSO

`ase-docs-shorten`, `ase-docs-refine`, `ase-docs-distill`, `ase-code-lint`, `ase-repo-changelog`.

ase-docs-distill - Distill Document Key Points

## NAME

`ase-docs-distill` - Distill Document Key Points

## SYNOPSIS

`ase-docs-distill`
    [`--help`|`-h`]
    [`--top`|`-t` *N*]
    *document-reference*

## DESCRIPTION

The `ase-docs-distill` skill reads a *provided document* and distills it
into a *flat*, *importance-ranked* list of its *key points*. The
*document-reference* is resolved *probe-as-file-first*: if the argument names
a *readable file* it is read from disk; otherwise the argument is taken
*verbatim* as *pasted text*. The document is read *silently* - only the
final ranked list is shown - so even a large document does not flood the
transcript.

Each emitted point is *one atomic claim, decision, or fact* (not a
section-level summary) and carries four things: a *0-10 salience rank*
reflecting its *importance to the reader* (impact and centrality, *not*
its position in the document), a *one-line rationale* justifying *why* it
earns that rank, an exact *line-range citation* (`file:Ls-Le`, or `:Ls-Le`
for pasted text), and a *verbatim evidence* snippet copied from the source
that *proves* the point. The points are emitted as a *bulleted list* -
one block per point, each showing its *LOCATION*, *EVIDENCE*,
*RATIONALE*, and *RANK* - sorted from highest to lowest salience, so the
ranking is *auditable* rather than an opaque ordering.

The `--top`/`-t` *N* option is a *length dial* that bounds the list to at
most *N* points (default *5*). It is an *upper bound only*: when the
document has fewer salient points than *N*, the skill emits only the
points it found and *never pads* the list with filler; a `0`, negative, or
non-numeric value falls back to the default *5*.

## OPTIONS

- `--top`|- `-t`- *N*:
Bound the ranked list to at most- *N*key points (default- *5*). The
bound is a- *cap*, never a- *quota*- fewer points are emitted when the
document does not contain- *N*salient ones, and an invalid or
non-positive- *N*reverts to the default.

## ARGUMENTS

- *document-reference*:
The document to distill - either a- *path*to a readable file or the- *text*itself pasted inline. If it resolves to a readable file the
file is read; otherwise it is treated verbatim as pasted text.

## SCENARIOS

- You want the key points of a document extracted and ranked
- You want the essence of a long text without reading all of it
- You want each key point backed by verbatim, line-cited evidence
- You want a pasted text or file boiled down to what matters

## EXAMPLES

Distill the key points of a document file:

```
❯ /ase-docs-distill doc/architecture.md
```

Distill only the top 3 key points:

```
❯ /ase-docs-distill --top 3 doc/architecture.md
```

Distill a pasted block of text:

```
❯ /ase-docs-distill The system shall accept payments in EUR and USD only.
```

## SEE ALSO

`ase-meta-search`, `ase-docs-proofread`, `ase-meta-why`.

ase-spec-lint - Lint Specification

## NAME

`ase-spec-lint` - Lint Specification

## SYNOPSIS

`ase-spec-lint`
    [`--help`|`-h`]
    [`--auto`|`-a`]
    [`--severity`|`-S`=(`AUTO`|`LOW`|`MEDIUM`|`HIGH`)]
    [`--force`|`-f`]
    [`--include`|`-i`=*aspect*[`,`...]]
    [`--exclude`|`-e`=*aspect*[`,`...]]
    [`--staged`]
    [`--unstaged`]
    [*spec-reference*]

## DESCRIPTION

The `ase-spec-lint` skill lints the *SpecBook*-based specification
(SPEC) for *potential quality problems* related to a fixed set of
specification quality aspects: from its syntax and formatting, over
its individual statements, up to the logical gaps of its model and its
completeness for an implementation. The investigation is dispatched to
sub-agents (`ase:ase-spec-lint`) so that scanning details do not leak
into the user-visible transcript.

For each detected problem, the skill renders a unified-diff *SOLUTION*
preview and either asks the user to `ACCEPT` or `REJECT` the proposed
correction interactively (or refine it via a free-text hint, which
re-proposes the correction without limit) or - with `--auto` - applies
all corrections automatically. Every applied correction refreshes the
`Modified:` timestamp of the changed specification files.

By default all ten specification quality aspects are checked. The
`--include` and `--exclude` options narrow this to an *effective aspect
set*: with `--include` only, exactly the listed aspects are checked;
with `--exclude` only, all aspects except the listed ones; with both,
the included ones minus the excluded ones. An unknown aspect id, or a
combination which cancels out to an empty set, aborts the skill with an
error. The ten aspect ids are:

```
S01 SYNTAX      S04 AMBIGUITY     S07 REDUNDANCY    S10 COMPLETENESS
S02 FORMAT      S05 ATOMICITY     S08 CONSISTENCY
S03 RATIONALE   S06 REALIZATION   S09 LOGIC
```

The `S01 SYNTAX` aspect reports the diagnostics of the specification
linting against the *SpecBook* schema configuration of the project and
of the generic Markdown linting via *markdownlint* (both as with `ase spec lint --markdown`), mapping the severity `error` onto `HIGH` and
`warning` onto `MEDIUM`. The `S02 FORMAT` aspect checks the formatting
against the *SpecBook* format variants and the prevailing conventions
of the specification itself. The aspects `S03 RATIONALE` up to `S07 REDUNDANCY` check the statements against the *Specifying Tenets* of
*ASE*: a `, BECAUSE` rationale per statement, unambiguous and
verifiable statements, atomic statements, intent instead of
realization, and a single source of truth per fact.

The *model-wide* aspects `S07 REDUNDANCY` up to `S10 COMPLETENESS` are
judged on the *entire* specification, even if only parts of it are
linted: `S08 CONSISTENCY` checks for contradictions and drifting
terminology, `S09 LOGIC` for logical gaps in the specification model
(like unreachable states or entities without any relationship), and
`S10 COMPLETENESS` for open ends which would force an implementer to
guess. As the corrections of `S09 LOGIC` and `S10 COMPLETENESS` fill
gaps with *best-guess* content, they are proposed for review only and
are *never* applied under `--auto`.

## OPTIONS

- `--auto`|- `-a`:
Automatically apply every proposed correction without asking the
user via the interactive dialog, except for the best-guess
corrections of the aspects- `S09 LOGIC`and- `S10 COMPLETENESS`,
which are only reported.
 
- `--severity`|- `-S`=(- `AUTO`|- `LOW`|- `MEDIUM`|- `HIGH`):
Set the- *severity floor*(default- `AUTO`): findings below the chosen
threshold are silently suppressed, ordered- `LOW`<- `MEDIUM`<- `HIGH`. The default- `AUTO`means- `MEDIUM`for a project with- `grey`boxing (unless- `--force`is given) and- `LOW`(keep all findings)
otherwise; an explicitly given floor always wins over the- `grey`boxing, i.e. e.g. with- `LOW`, it acts on all findings of a project
with- `grey`boxing.- `ACCEPTED`findings are never suppressed.
Surviving findings are reported in- *descending severity*order- `HIGH`,- `MEDIUM`,- `LOW`,- `ACCEPTED`, keeping the- `file`/- `line`order
within the same severity.
 
- `--force`|- `-f`:
Ignore the project boxing entirely: lint a project with- `black`boxing (which is otherwise not linted at all), and show the full
unified diffs and the default- `LOW`severity floor for a project
with- `grey`boxing (which otherwise shows only condensed one-line
diffs and defaults to the- `MEDIUM`severity floor).
 
- `--include`|- `-i`=- *aspect*[- `,`...]:
Restrict the checked specification quality aspects to the given
comma-separated list of aspect ids (e.g.- `S01,S02`). Without this
option, all ten aspects are checked.
 
- `--exclude`|- `-e`=- *aspect*[- `,`...]:
Remove the given comma-separated list of aspect ids from the checked
specification quality aspects. Applied- *after*- `--include`, so- `-i S09,S10 -e S10`checks- `S09`only.
 
- `--staged`:
Restrict the linting to the specification files with- *staged*Git
changes. Deleted files are skipped, and the files are linted in
their current working copy state. Combined with- `--unstaged`, lint
the specification files with- *any*uncommitted Git changes.
 
- `--unstaged`:
Restrict the linting to the specification files with- *unstaged*Git
changes plus the- *untracked*specification files. Deleted files are
skipped. Combined with- `--staged`, lint the specification files with- *any*uncommitted Git changes.
 

## ARGUMENTS

- *spec-reference*:
A file, directory, or wildcard reference to the specification files
to lint. Without it, the- *entire*specification is linted, resp.
under- `--staged`or- `--unstaged`all changed specification files.

## SCENARIOS

- You want your specification checked for specification quality problems
- You want to know whether your specification is sufficient for an implementation
- You want logical gaps in your specification model detected
- You want corrections proposed which you accept or reject one by one
- You want all mechanical corrections applied automatically
- You want only your staged or unstaged specification changes linted before committing

## EXAMPLES

Lint the entire specification interactively:

```
❯ /ase-spec-lint
```

Lint a single specification file and automatically apply all corrections:

```
❯ /ase-spec-lint --auto docs/spec/data-model.md
```

Lint the specification for its syntax and formatting only:

```
❯ /ase-spec-lint -i S01,S02
```

Check the specification model for logical gaps and completeness only:

```
❯ /ase-spec-lint -i S09,S10
```

Lint the specification, reporting only `MEDIUM` and `HIGH` findings:

```
❯ /ase-spec-lint -S MEDIUM
```

Lint the specification regardless of the project boxing:

```
❯ /ase-spec-lint --force
```

Lint only the specification files with staged Git changes:

```
❯ /ase-spec-lint --staged
```

## SEE ALSO

`ase-spec-edit`, `ase-spec-activate`,
`ase-code-lint`, `ase-sync-export`.

ase-spec-edit - Edit Specification

## NAME

`ase-spec-edit` - Edit Specification

## SYNOPSIS

`ase-spec-edit`
    [`--help`|`-h`]
    [`--approach`|`-a`]
    [`--grill`|`-g`]
    [`--grill-rounds`|`-r` *n*]
    [`--grill-until`|`-u` `MUST`|`SHOULD`|`MAY`]
    [`--grill-perspective`|`-p` *perspective*]
    [`--grill-ground`|`-G`]
    [`--verify`|`-v` `light`|`none`|`full`]
    [`--worktree`|`-w` *name*[`:`*branch*]]
    [`--loop`|`-l`]
    [*query*]

## DESCRIPTION

The `ase-spec-edit` skill edits the *SpecBook*-based specification
(`SPEC`) *directly* from a *query*, in one shot and without any task
plan ceremony. It is the specification-level counterpart of
`ase-code-edit` and a *plan-less* alternative to `ase-sync-import` and
`ase-sync-reconcile` whenever the specification has to be changed from a
plain description instead of from a foreign source or another artifact
kind.

Each single-shot run (or each `--loop` iteration) walks through five
states: *querying* (take the *query* argument or ask for a query via an
interactive `Edit Query` dialog, and split it into its domain-specific
WHAT and its implementation-detail HOW parts), *discovering* (resolve
the `SPEC` artifacts, read the *SpecBook* schema configuration of the
project, and read the specification artifacts related to the query),
*grilling* (optionally stress-test the query with rounds of questions),
*implementing* (optionally select an approach, then apply the change set
in place, honoring the GENERIC and SPECIFYING tenets and the `SPEC`
format contract), and *verifying* (validate the specification until it
passes, unless disabled via `--verify none`). The *querying* state, every *grilling* round, and the
approach selection close with an `EDIT TODO` box showing the established
`WHAT` and `HOW` information.

The change set stays strictly restricted to the `SPEC` artifacts -- the
artifact kinds `CODE`, `DOCS`, `TASK`, `INFR`, and `OTHR` are never
touched, except for an entry in an already existing `CHANGELOG.md`
file. Every generated artifact carries the current `Created:` and
`Modified:` timestamps, and every changed artifact gets its `Modified:`
timestamp refreshed.

## OPTIONS

- `--approach`|- `-a`:
Before applying the change set, propose one or more specification
approaches with pros and cons, recommend one of them, and let the
user pick the preferred one via an interactive dialog (or, under the- `black`project boxing or in headless mode, automatically pick the
recommended one). The chosen approach is merged into the HOW part of
the query. Without- `--approach`, no approaches are determined at
all.
 
- `--grill`|- `-g`:
Grill the query before implementing, similar to- `ase-task-grill`:
raise 1-10 questions per round which resolve the open points of the
query. Each question carries a- `FOCUS-AREA`--- `DOMAIN`(domain-specifics, must be clarified),- `INTERFACE`(externally
observable behavior or UI/API interfaces, must be clarified),- `ARCHITECTURE`(structure, wiring, placement, or dependencies,
should be clarified),- `IMPLEMENTATION`(inner technical details,
can be clarified),- `REGRESSION`(what must not break, should be
clarified), or- `CONFIRMATION`(what proves the specified behavior,
should be clarified) -- and a 1-3 word- `TOPIC`hint. The questions
of a round are sorted by descending focus area importance
(- `DOMAIN`,- `INTERFACE`,- `ARCHITECTURE`,- `IMPLEMENTATION`,- `REGRESSION`,- `CONFIRMATION`). Questions of focus area- `REGRESSION`are raised only under- `--verify light`or- `full`, and
those of focus area- `CONFIRMATION`only under- `--verify full`, as
only then a verification honors their decisions.
All questions of a round are announced together below a- `GRILLING ROUND K/L`line (the round numbering is
omitted when only a single round is performed) as a- `QUESTION`/- `ANSWERS`table with one row per question, each row
carrying two to three grounded answer alternatives (with the
alternative reflecting the current understanding marked with- `⚑`).
They are then asked in- *one*batch via a single interactive dialog,
whose question asks for the combined answer to all (or a subset) of
the listed questions and whose only answer options are the fixed- `SKIP GRILLING`(skip the remaining grilling) and- `STOP SKILL`(stop
the skill) ones, plus free-text input. The questions are numbered- `1`,- `2`, etc. and their answer alternatives are lettered- `A`,- `B`,
etc., so the free-text input can cherry-pick answers with short
responses matching- `\d+[a-zA-Z]`(like- `1A 2c`), freely mixed with
keyword text. The answers are merged back into the WHAT and HOW
parts of the query. Without- `--grill`, no questions are asked at
all.
 
- `--grill-rounds`|- `-r`- *n*:
The- *maximum*number of grill rounds to apply (default:- `1`). Each
round starts from scratch from only the current WHAT and HOW parts,
forgetting all information of previous rounds, and closes with an- `EDIT TODO`box. The grilling stops early -- announced by a- `grilling finished early`status line -- once a round, even the
first one, finds the open points clear enough according to- `--grill-until`. Only effective together with- `--grill`.
 
- `--grill-until`|- `-u`- `MUST`|- `SHOULD`|- `MAY`:
Grill until at least all open points of the given severity or
higher are clear (default:- `MUST`), where the severity follows from
the focus area:- `MUST`(- `DOMAIN`,- `INTERFACE`),- `SHOULD`(- `ARCHITECTURE`,- `REGRESSION`,- `CONFIRMATION`), and- `MAY`(- `IMPLEMENTATION`). Independent of this, every open point is rated
with an individual impact (- `HIGH`,- `MEDIUM`, or- `LOW`), which
decides which points are raised as questions and in which order
within a focus area. Only effective together with- `--grill`.
 
- `--grill-perspective`|- `-p`- *perspective*:
Grill exclusively from the logical- *perspective*-- a keyword or
short hint like- `domain`,- `performance`, or- `security`. The
perspective acts as an exclusive lens: only open points relevant from
this perspective are raised as questions, while every question still
carries its focus area, severity, and impact. The active perspective
is shown in the announcement line, like- `GRILLING (security)`.
Without a perspective, the grilling is holistic. Only effective together with- `--grill`.
 
- `--grill-ground`|- `-G`:
Ground the grilling in facts instead of the world knowledge of the
model alone: once before the first round, the artifacts of all kinds
(- `SPEC`,- `CODE`,- `DOCS`,- `INFR`, and- `OTHR`) are scanned for
the project facts relevant to the grilled subject (from the given
perspective, if any) and the Internet/Web is searched for the
established practices, standards, patterns, and pitfalls about it.
These facts serve all rounds, both for raising the questions and for
preferably deriving their answer alternatives. The grounding is shown
in the announcement line, like- `GRILLING (security, grounded)`. If
no usable facts are found, a warning is shown and the grilling falls
back to model knowledge. Only effective together with- `--grill`.
 
- `--verify`|- `-v`- `light`|- `none`|- `full`:
Verify the edited specification by validating it via- *SpecBook*linting and fixing the reported diagnostics in the affected- `SPEC`artifacts, for at most ten rounds. Any diagnostics remaining after
the last round are listed as- `REMAINING DIAGNOSTICS`. The default- `light`and the level- `full`perform the same validation. The level- `none`strictly performs no validation at all.
 
- `--worktree`|- `-w`- *name*[- `:`- *branch*]:
Apply the change sets inside the dedicated Git worktree- `.ase/worktree/<name>`instead of the current working copy. The
worktree is- *reused*if it already exists -- then switched to- *branch*if given and it has no uncommitted changes --, otherwise
it is- *created*with the branch- *branch*(default:- *name*),
checked out if it exists or created from- `HEAD`otherwise. One
single worktree serves the whole skill run: it is prepared before
the first discovery, so discovering, grilling, and implementing all
operate on it, all- `--loop`iterations land in it, and it is left
uncommitted for review. Unless- `--verify none`is given, the validation then runs as the- `ase spec lint`command
inside the worktree. If- *name*is a bare lower-case word naming no
existing worktree and a- *query*follows (like- `-w fix the crash`, a
former boolean- `-w`usage), the skill asks whether- *name*is the
worktree name or the first query word.
 
- `--loop`|- `-l`:
Loop the whole state cycle: after each iteration, ask for the next
edit query via the interactive- `Edit Query`dialog and repeat, until
the user answers with its fixed- `STOP SKILL`option or cancels the
dialog.
 

## ARGUMENTS

- *query*:
Description of the specification edit to perform. When omitted, the
skill asks for the query via an interactive- `Edit Query`dialog,
carrying the fixed- `STOP SKILL`option plus free-text input.

## SCENARIOS

- You want the specification edited in one shot from a description
- You want `SPEC`changes without the task plan ceremony
- You want the query stress-tested by grilling before the spec is edited
- You want specification edits validated by SpecBook linting
- You want the change set to land on a specific Git branch or worktree

## EXAMPLES

Edit in one shot, without any questions, but with validation:

```
❯ /ase-spec-edit add a Reviewer persona to the persona model
```

Edit in one shot, without any questions or validation:

```
❯ /ase-spec-edit -v none add a Reviewer persona to the persona model
```

Grill the query with two rounds first, then edit and fully validate:

```
❯ /ase-spec-edit -g -r 2 -v full split the Event entity into Event and EventSeries
```

Let the user pick one of several specification approaches first, then
edit:

```
❯ /ase-spec-edit -a model the access permissions of the personas
```

Grill the query exclusively from the domain perspective, grounded in
the project artifacts and the Internet/Web, then edit:

```
❯ /ase-spec-edit -g -p domain -G add recurring events to the Event model
```

Loop over multiple specification edits inside a dedicated Git worktree:

```
❯ /ase-spec-edit -l -w event-series
```

## SEE ALSO

`ase-spec-activate`, `ase-spec-lint`, `ase-code-edit`, `ase-sync-import`,
`ase-sync-reconcile`, `ase-sync-export`, `ase-task-grill`.

ase-spec-activate - Activate Specification Know-How

## NAME

`ase-spec-activate` - Activate Specification Know-How

## SYNOPSIS

`ase-spec-activate`
    [`--help`|`-h`]
    [*query*]

## DESCRIPTION

The `ase-spec-activate` skill activates the know-how about the
*SpecBook*-based specification (`SPEC`) of the project in the current
session: it loads the *SpecBook* format contract, reads the *SpecBook*
schema configuration of the project (the schema files configured via
`project.artifact.spec.schema`, where the entry `std` -- also the default
-- names the standard schema bundled with *ASE*), and resolves the list
of `SPEC` artifacts in the configured specification base directory. The
activation is reported in a `SPEC ACTIVATED` box, listing the schema
files (each marked `standard` or `custom`) and the base directory with
the number of resolved artifacts.

Once activated, the specification can be worked with *ad-hoc* in plain
conversation for the remainder of the session: reading, querying, and
explaining its content (with `[[xxx]]` references resolved across
artifacts and every answer grounded in artifact file and object id), and
applying small changes to it (kept conformant to the format contract and
the schema, with refreshed `Modified:` timestamps and *SpecBook* linting
afterwards). Ad-hoc changes stay strictly restricted to the `SPEC`
artifacts; substantial changes are deferred to `ase-spec-edit`.

The skill is both user-callable and model-callable: the model invokes it
*automatically* whenever the user wants to work with the specification
files without invoking a dedicated specification skill. It works in
tandem with `ase-spec-edit`, `ase-sync-import`, `ase-sync-reconcile`, and
`ase-sync-export`, which include the format contract themselves and hence
activate the know-how implicitly -- their own procedures take precedence
whenever they run.

## ARGUMENTS

- *query*:
Optional question about, or small ad-hoc change to, the
specification, served directly after the activation. When omitted,
the skill only activates the know-how. This argument mainly serves
the automatic invocation by the agent harness: the user's request
which triggered the activation is passed through as the- *query*,
so activation and answer happen in one skill run under the skill's
rules instead of in the plain conversation afterwards.

## SCENARIOS

- You want to ask what the specification says about a topic
- You want to look up, check, or explain specification content in plain conversation
- You want a small ad-hoc change applied to the specification
- You want the SpecBook know-how loaded before working with the specification files

## EXAMPLES

Activate the specification know-how only:

```
❯ /ase-spec-activate
```

Activate and directly answer a question about the specification:

```
❯ /ase-spec-activate which personas does the specification define?
```

## SEE ALSO

`ase-spec-edit`, `ase-sync-import`, `ase-sync-reconcile`,
`ase-sync-export`.

ase-code-edit - Edit Source Code

## NAME

`ase-code-edit` - Edit Source Code

## SYNOPSIS

`ase-code-edit`
    [`--help`|`-h`]
    [`--mode`|`-m` `auto`|`craft`|`refactor`|`resolve`]
    [`--approach`|`-a`]
    [`--grill`|`-g`]
    [`--grill-rounds`|`-r` *n*]
    [`--grill-until`|`-u` `MUST`|`SHOULD`|`MAY`]
    [`--grill-perspective`|`-p` *perspective*]
    [`--grill-ground`|`-G`]
    [`--verify`|`-v` `light`|`none`|`full`]
    [`--worktree`|`-w` *name*[`:`*branch*]]
    [`--loop`|`-l`]
    [*query*]

## DESCRIPTION

The `ase-code-edit` skill edits the code base *directly* from a *query*,
in one shot and without any task plan ceremony. It is the *plan-less*
counterpart of `ase-task-edit`, `ase-task-grill`, and
`ase-task-implement`, covering crafting, refactoring, and resolving.

Each single-shot run (or each `--loop` iteration) walks through five
states: *querying* (take the *query* argument or ask for a query via an
interactive `Edit Query` dialog, and
split it into its domain-specific WHAT and its implementation-detail HOW
parts), *discovering* (read the source artifacts related to the query),
*grilling* (optionally stress-test the query with rounds of questions),
*implementing* (optionally select an approach, then apply the change
set in place, honoring the tenet set selected via `--mode`), and
*verifying* (verify the implementation until it passes, unless
disabled via `--verify none`). The
*querying* state, every *grilling* round, and the approach selection
close with an `EDIT TODO` box showing the established `WHAT` and `HOW`
information.

## OPTIONS

- `--mode`|- `-m`- `auto`|- `craft`|- `refactor`|- `resolve`:
Select the tenet set internalized for the implementation:- `craft`(CRAFTING),- `refactor`(REFACTORING), or- `resolve`(RESOLVING).
The default- `auto`infers the tenet set from the query itself.
 
- `--approach`|- `-a`:
Before applying the change set, propose one or more crafting,
refactoring, or resolution approaches with pros and cons, recommend
one of them, and let the user pick the preferred one via an
interactive dialog (or, under the- `black`project boxing or in
headless mode, automatically pick the recommended one). The chosen
approach is merged into the HOW part of the query. Without- `--approach`, no approaches are determined at all.
 
- `--grill`|- `-g`:
Grill the query before implementing, similar to- `ase-task-grill`:
raise 1-10 questions per round which resolve the open points of the
query. Each question carries a- `FOCUS-AREA`--- `DOMAIN`(domain-specifics, must be clarified),- `INTERFACE`(externally
observable behavior or UI/API interfaces, must be clarified),- `ARCHITECTURE`(structure, wiring, placement, or dependencies,
should be clarified),- `IMPLEMENTATION`(inner technical details,
can be clarified),- `REGRESSION`(what must not break, should be
clarified), or- `CONFIRMATION`(what proves the specified behavior,
should be clarified) -- and a 1-3 word- `TOPIC`hint. The questions
of a round are sorted by descending focus area importance
(- `DOMAIN`,- `INTERFACE`,- `ARCHITECTURE`,- `IMPLEMENTATION`,- `REGRESSION`,- `CONFIRMATION`). Questions of focus area- `REGRESSION`are raised only under- `--verify light`or- `full`, and
those of focus area- `CONFIRMATION`only under- `--verify full`, as
only then a verification honors their decisions.
All questions of a round are announced together below a- `GRILLING ROUND K/L`line (the round numbering is omitted when only
a single round is performed) as a- `QUESTION`/- `ANSWERS`table with
one row per question, each row carrying two to three grounded answer
alternatives (with the alternative reflecting the current understanding
marked with- `⚑`). They are then asked in- *one*batch via a single
interactive dialog titled- `GRILLING ROUND K/L`(or plain- `GRILLING`for a single round), whose question asks for the combined answer to
all (or a subset) of the listed aspect questions and whose only
answer options are the fixed- `STOP SKILL`(stop the skill) and- `SKIP GRILLING`(skip the remaining grilling) ones, plus free-text
input. The questions are numbered- `1`,- `2`, etc. and their answer
alternatives are lettered- `A`,- `B`, etc., so the free-text input can
cherry-pick answers with short responses matching- `\d+[a-zA-Z]`(like- `1A 2c`), freely mixed with keyword text. The answers are
merged back into the WHAT and HOW parts of the query. Without- `--grill`, no questions are asked at all.
 
- `--grill-rounds`|- `-r`- *n*:
The- *maximum*number of grill rounds to apply (default:- `1`). Each
round starts from scratch from only the current WHAT and HOW parts,
forgetting all information of previous rounds, and closes with an- `EDIT TODO`box. The grilling stops early -- announced by a- `grilling finished early`status line -- once a round, even the
first one, finds the open points clear enough according to- `--grill-until`. Only effective together with- `--grill`.
 
- `--grill-until`|- `-u`- `MUST`|- `SHOULD`|- `MAY`:
Grill until at least all open points of the given severity or
higher are clear (default:- `MUST`), where the severity follows from
the focus area:- `MUST`(- `DOMAIN`,- `INTERFACE`),- `SHOULD`(- `ARCHITECTURE`,- `REGRESSION`,- `CONFIRMATION`), and- `MAY`(- `IMPLEMENTATION`). Independent of this, every open point is rated
with an individual impact (- `HIGH`,- `MEDIUM`, or- `LOW`), which
decides which points are raised as questions and in which order
within a focus area. Only effective together with- `--grill`.
 
- `--grill-perspective`|- `-p`- *perspective*:
Grill exclusively from the logical- *perspective*-- a keyword or
short hint like- `domain`,- `performance`, or- `security`. The
perspective acts as an exclusive lens: only open points relevant from
this perspective are raised as questions, while every question still
carries its focus area, severity, and impact. The active perspective
is shown in the announcement line, like- `GRILLING (security)`.
Without a perspective, the grilling is holistic. Only effective together with- `--grill`.
 
- `--grill-ground`|- `-G`:
Ground the grilling in facts instead of the world knowledge of the
model alone: once before the first round, the artifacts of all kinds
(- `SPEC`,- `CODE`,- `DOCS`,- `INFR`, and- `OTHR`) are scanned for
the project facts relevant to the grilled subject (from the given
perspective, if any) and the Internet/Web is searched for the
established practices, standards, patterns, and pitfalls about it.
These facts serve all rounds, both for raising the questions and for
preferably deriving their answer alternatives. The grounding is shown
in the announcement line, like- `GRILLING (security, grounded)`. If
no usable facts are found, a warning is shown and the grilling falls
back to model knowledge. Only effective together with- `--grill`.
 
- `--verify`|- `-v`- `light`|- `none`|- `full`:
Verify whether the implementation fulfills the requirements, and
adjust the failing parts of the change set until the verification
passes, for at most ten rounds, listing any failures remaining after
the last round as- `REMAINING FAILURES`. The default- `light`runs the
available lint and build commands (including the type-checker) and
the already existing regression test suite, if any. The level- `full`additionally performs individual, change-specific manual tests
through ad-hoc scratch tests and program runs, which are cleaned up
afterwards. The level- `none`strictly performs no verification at
all.
 
- `--worktree`|- `-w`- *name*[- `:`- *branch*]:
Apply the change sets inside the dedicated Git worktree- `.ase/worktree/<name>`instead of the current working copy. The
worktree is- *reused*if it already exists -- then switched to- *branch*if given and it has no uncommitted changes --, otherwise
it is- *created*with the branch- *branch*(default:- *name*),
checked out if it exists or created from- `HEAD`otherwise. One
single worktree serves the whole skill run: it is prepared before
the first discovery, so discovering, grilling, and implementing all
operate on it, all- `--loop`iterations land in it, and it is left
uncommitted for review. If- *name*is a bare lower-case word naming no existing worktree and a- *query*follows (like- `-w fix the crash`, a former boolean- `-w`usage), the skill asks whether- *name*is the worktree name or the
first query word.
 
- `--loop`|- `-l`:
Loop the whole state cycle: after each iteration, ask for the next
edit query via the interactive- `Edit Query`dialog and repeat, until
the user answers with its fixed- `STOP SKILL`option or cancels the
dialog.
 

## ARGUMENTS

- *query*:
Description of the edit to perform. When omitted, the skill asks
for the query via an interactive- `Edit Query`dialog, carrying the
fixed- `STOP SKILL`option plus free-text input.

## SCENARIOS

- You want the code base edited in one shot from a plain description
- You want an analyzer issue like `P1`fixed directly without a plan
- You want a quick change with optional grilling and verification
- You want several edits chained in a loop, optionally in a worktree
- You want the change set to land on a specific Git branch or worktree

## EXAMPLES

Edit in one shot, without any questions, but with light verification:

```
❯ /ase-code-edit add a --verbose option to the CLI
```

Edit in one shot, without any questions or verification:

```
❯ /ase-code-edit -v none add a --verbose option to the CLI
```

Grill the query with two rounds first, then implement and fully verify:

```
❯ /ase-code-edit -g -r 2 -v full refactor the config loading into layers
```

Let the user pick one of several resolution approaches first, then
implement:

```
❯ /ase-code-edit -a fix race condition in cache invalidation
```

Grill the query exclusively from the security perspective, grounded in
the project artifacts and the Internet/Web, then implement:

```
❯ /ase-code-edit -g -p security -G add a file upload endpoint to the API
```

Loop over multiple edits inside a dedicated Git worktree:

```
❯ /ase-code-edit -l -w config-layers
```

Edit inside the dedicated Git worktree `.ase/worktree/verbose` on the
branch `feature-verbose`:

```
❯ /ase-code-edit -w verbose:feature-verbose add a --verbose option to the CLI
```

## SEE ALSO

`ase-task-edit`, `ase-spec-edit`,
`ase-task-grill`, `ase-task-implement`.

ase-code-analyze - Analyze Source Code

## NAME

`ase-code-analyze` - Analyze Source Code

## SYNOPSIS

`ase-code-analyze`
    [`--help`|`-h`]
    [`--performance`|`-p`]
    [`--security`|`-s`]
    [`--severity`|`-S`=(`AUTO`|`LOW`|`MEDIUM`|`HIGH`)]
    [`--prefix`|`-P` *prefix*]
    [`--staged`]
    [`--unstaged`]
    [*source-reference*]

## DESCRIPTION

The `ase-code-analyze` skill analyzes the source code of the referenced
location, and its directly related source code, for problems. It is
*read-only* and advisory: it reports problems but applies *no* changes.

The *analysis lens* depends on the selected options:

- **default**(neither- `--performance`nor- `--security`): problems in
its- *logic*,- *semantics*, and related- *control flow*.
 
- `--performance`|- `-p`: problems in- *performance*and- *efficiency*.
 
- `--security`|- `-s`: problems in- *security*.
 

The `--performance` and `--security` options are *mutually exclusive*.

The `--severity`|`-S`=(`AUTO`|`LOW`|`MEDIUM`|`HIGH`) option sets a
*severity floor* (default `AUTO`): problems below the chosen threshold
are silently suppressed (neither reported nor persisted), ordered `LOW`
< `MEDIUM` < `HIGH`. The default `AUTO` means `MEDIUM` for a project
with `grey` boxing, no analysis at all for a project with `black`
boxing, and `LOW` (keep all problems) otherwise; an explicitly given
floor always wins over the boxing, i.e. it also analyzes a project with
`black` boxing and, e.g. with `LOW`, reports all problems of a project
with `grey` boxing. `ACCEPTED` problems are never suppressed. Surviving
problems are reported in *descending
severity* order `HIGH`, `MEDIUM`, `LOW`, `ACCEPTED` - keeping the
`file`/`line` order within the same severity - and are renumbered
contiguously as `P<n>`, so `P1` is the most severe problem.

The `--prefix`|`-P` *prefix* option prefixes every reported problem id
with *prefix* and a hyphen, so `P1` becomes `<prefix>-P1` and its
persisted key becomes `ase-issue-<prefix>-P1`. The purge of stale
results is narrowed to the same namespace accordingly, so analyses run
under *distinct* prefixes coexist instead of overwriting each other.
Without the option (the default), ids stay unprefixed and the purge
covers the *entire* `ase-issue-*` space, including any prefixed results.

The `--staged` option restricts the analysis to the source files with
*staged* Git changes, and the `--unstaged` option restricts it to the
source files with *unstaged* Git changes plus the *untracked* source
files. Deleted files are skipped, and the files are analyzed in their
current working copy state. With a *source-reference*, only the changed
files within it are analyzed; without one, all changed files of the
repository are analyzed. If no such file exists, nothing is analyzed.
Both options can be combined to analyze the source files with *any*
uncommitted Git changes.

The skill investigates the code base silently, reports each detected
problem as a `PROBLEM` entry with severity (`LOW`, `MEDIUM`, `HIGH`) and
inline file/line references (in the performance lens, each entry
additionally carries an *evidence* and a *trade-off* line), and persists
results in the `ase` MCP key/value store as `ase-issue-P<n>` entries so
they can later be planned via `ase-task-edit P<n>` or directly
fixed via `ase-code-edit P<n>`.

## ARGUMENTS

- *source-reference*:
A file, directory, function, or other reference to the source code
to analyze. Optional only under- `--staged`or- `--unstaged`.

## SCENARIOS

- You want your code checked for logic, semantics, and control-flow problems
- You want a read-only report of problems without any changes applied
- You want performance and efficiency opportunities surfaced
- You want a security-focused inspection of your code
- You want problems persisted as issue ids like `P1`for later resolving
- You want only your staged or unstaged changes analyzed before committing

## EXAMPLES

Analyze a specific source file for logic/semantic problems:

```
❯ /ase-code-analyze src/server.ts
```

Analyze a directory of code:

```
❯ /ase-code-analyze src/handlers/
```

Analyze a source file for performance/efficiency opportunities only:

```
❯ /ase-code-analyze --performance src/server.ts
```

Analyze a source file for security aspects only:

```
❯ /ase-code-analyze -s src/handlers/
```

Analyze a directory, reporting only `MEDIUM` and `HIGH` problems:

```
❯ /ase-code-analyze -S MEDIUM src/handlers/
```

Analyze a directory under its own id namespace, yielding `auth-P1`, `auth-P2`, ...:

```
❯ /ase-code-analyze --prefix auth src/auth/
```

Analyze only the source files with staged Git changes:

```
❯ /ase-code-analyze --staged
```

Analyze only the unstaged and untracked source files below a directory:

```
❯ /ase-code-analyze --unstaged src/handlers/
```

## SEE ALSO

`ase-task-edit`, `ase-code-edit`, `ase-code-lint`,
`ase-code-explain`, `ase-arch-analyze`.

ase-code-insight - Project Insight

## NAME

`ase-code-insight` - Project Insight

## SYNOPSIS

`ase-code-insight`
    [`--help`|`-h`]
    *code-references*

## DESCRIPTION

The `ase-code-insight` skill gives high-level *insights* into the
project through the referenced source code. The skill produces four
sections: a *PROJECT ABSTRACT* (summary derived from `README.*` or
source scanning), a *PROJECT AUTHOR* list (from `git shortlog`),
a *SOURCE CHURN* table (most-committed files in the last year), and
a *MODULE STRUCTURE* Mermaid diagram of modules and their imports.

## ARGUMENTS

- *code-references*:
One or more file or directory references to source code that
should be inspected for insights.

## SCENARIOS

- You want a high-level overview of a project
- You want to know who wrote the project and which files churn most
- You want a module structure diagram of the imports
- You want to get familiar with an unknown code base quickly

## EXAMPLES

Get insights into the current project:

```
❯ /ase-code-insight src/
```

Get insights into a specific subsystem:

```
❯ /ase-code-insight tool/src/
```

## SEE ALSO

`ase-code-explain`, `ase-code-analyze`, `ase-arch-analyze`.

ase-code-explain - Explain Source Code

## NAME

`ase-code-explain` - Explain Source Code

## SYNOPSIS

`ase-code-explain`
    [`--help`|`-h`]
    *source-reference*

## DESCRIPTION

The `ase-code-explain` skill analyzes and explains the source code of
the referenced location in a *brief*, *standardized*, and *concise*
way along six dimensions: *WHAT* (functionality), *WHY* (rationale),
*ANALOGY* (everyday-life comparison in ELI5 style), *DIAGRAM* (Mermaid
diagram of control flow, data flow, or structure), *CRUXES* (what to
notice), and *GOTCHAS* (what to not stumble over).

## ARGUMENTS

- *source-reference*:
A file, directory, function, or other reference to the source code
to explain.

## SCENARIOS

- You want to understand how a piece of code works
- You want an explanation with WHAT, WHY, analogy, and diagram
- You want the cruxes and gotchas of unfamiliar code pointed out
- You want a quick orientation before touching foreign code

## EXAMPLES

Explain a single source file:

```
❯ /ase-code-explain src/parser.ts
```

Explain a specific function:

```
❯ /ase-code-explain src/parser.ts#parseExpression
```

## SEE ALSO

`ase-code-insight`, `ase-code-analyze`, `ase-arch-analyze`.

ase-code-lint - Lint Source Code

## NAME

`ase-code-lint` - Lint Source Code

## SYNOPSIS

`ase-code-lint`
    [`--help`|`-h`]
    [`--auto`|`-a`]
    [`--severity`|`-S`=(`AUTO`|`LOW`|`MEDIUM`|`HIGH`)]
    [`--force`|`-f`]
    [`--include`|`-i`=*aspect*[`,`...]]
    [`--exclude`|`-e`=*aspect*[`,`...]]
    [`--staged`]
    [`--unstaged`]
    [*source-reference*]

## DESCRIPTION

The `ase-code-lint` skill lints the source code of the referenced
location for *potential code quality problems* related to a fixed set
of code quality aspects. The investigation is dispatched to a
sub-agent (`ase:ase-code-lint`) so that scanning details do not leak
into the user-visible transcript.

For each detected problem, the skill renders a unified-diff *SOLUTION*
preview and either asks the user to `ACCEPT` or `REJECT` the proposed
correction interactively (or refine it via a free-text hint, which
re-proposes the correction without limit) or - with `--auto` - applies
all corrections automatically.

By default all twenty-one code quality aspects are checked. The `--include`
and `--exclude` options narrow this to an *effective aspect set*: with
`--include` only, exactly the listed aspects are checked; with
`--exclude` only, all aspects except the listed ones; with both, the
included ones minus the excluded ones. An unknown aspect id, or a
combination which cancels out to an empty set, aborts the skill with an
error. The twenty-one aspect ids are:

```
A01 FORMATTING     A07 PATTERNS         A13 MEMORY-LEAK    A19 FLOW
A02 COMPREHENSION  A08 COMPLICATEDNESS  A14 CONCURRENCY    A20 DEAD-CODE
A03 CLEANLINESS    A09 CONCISENESS      A15 PERFORMANCE    A21 DOCUMENTATION
A04 SPELLING       A10 SMELLS           A16 SECURITY
A05 COMPLEXITY     A11 TYPING           A17 ARCHITECTURE
A06 REDUNDANCY     A12 ERROR-HANDLING   A18 LOGIC
```

The `A21 DOCUMENTATION` aspect checks the code documentation in *both*
directions: it flags functions, methods, classes, and modules - *private
ones included* - which lack a minimal 1-2 line description in the
idiomatic documentation convention of the target language, *and* it
flags excessive comments (narrated decision logs, change history,
line-by-line explanations of the obvious), comments merely restating
the code, and comments contradicting the code.

## OPTIONS

- `--auto`|- `-a`:
Automatically apply every proposed correction without asking the
user via the interactive dialog.
 
- `--severity`|- `-S`=(- `AUTO`|- `LOW`|- `MEDIUM`|- `HIGH`):
Set the- *severity floor*(default- `AUTO`): findings below the chosen
threshold are silently suppressed, ordered- `LOW`<- `MEDIUM`<- `HIGH`. The default- `AUTO`means- `MEDIUM`for a project with- `grey`boxing (unless- `--force`is given) and- `LOW`(keep all findings)
otherwise; an explicitly given floor always wins over the- `grey`boxing, i.e. e.g. with- `LOW`, it acts on all findings of a project
with- `grey`boxing.- `ACCEPTED`findings are never suppressed.
Surviving findings are reported in- *descending severity*order- `HIGH`,- `MEDIUM`,- `LOW`,- `ACCEPTED`, keeping the- `file`/- `line`order
within the same severity.
 
- `--force`|- `-f`:
Ignore the project boxing entirely: lint a project with- `black`boxing (which is otherwise not linted at all), and show the full
unified diffs and the default- `LOW`severity floor for a project
with- `grey`boxing (which otherwise shows only condensed one-line
diffs and defaults to the- `MEDIUM`severity floor).
 
- `--include`|- `-i`=- *aspect*[- `,`...]:
Restrict the checked code quality aspects to the given
comma-separated list of aspect ids (e.g.- `A01,A04`). Without this
option, all twenty-one aspects are checked.
 
- `--exclude`|- `-e`=- *aspect*[- `,`...]:
Remove the given comma-separated list of aspect ids from the checked
code quality aspects. Applied- *after*- `--include`, so- `-i A01,A02 -e A02`checks- `A01`only.
 
- `--staged`:
Restrict the linting to the source files with- *staged*Git changes.
Deleted files are skipped, and the files are linted in their current
working copy state. Combined with- `--unstaged`, lint the source
files with- *any*uncommitted Git changes.
 
- `--unstaged`:
Restrict the linting to the source files with- *unstaged*Git changes
plus the- *untracked*source files. Deleted files are skipped.
Combined with- `--staged`, lint the source files with- *any*uncommitted Git changes.
 

## ARGUMENTS

- *source-reference*:
A file, directory, or other reference to the source code to lint.
Optional only under- `--staged`or- `--unstaged`: with it, only the
changed files within it are linted; without it, all changed files
of the repository are linted.

## SCENARIOS

- You want your code checked for code quality problems
- You want corrections proposed which you accept or reject one by one
- You want all quality corrections applied automatically
- You want only specific quality aspects like formatting or spelling checked
- You want missing or excessive code documentation flagged
- You want only your staged or unstaged changes linted before committing

## EXAMPLES

Lint a source file interactively:

```
❯ /ase-code-lint src/server.ts
```

Lint a directory and automatically apply all corrections:

```
❯ /ase-code-lint --auto src/handlers/
```

Lint a directory, reporting only `MEDIUM` and `HIGH` findings:

```
❯ /ase-code-lint -S MEDIUM src/handlers/
```

Lint a directory regardless of the project boxing:

```
❯ /ase-code-lint --force src/handlers/
```

Lint a source file for the formatting and spelling aspects only:

```
❯ /ase-code-lint -i A01,A04 src/server.ts
```

Lint a directory for all aspects except dead code:

```
❯ /ase-code-lint --exclude A20 src/handlers/
```

Lint a directory for the documentation aspect only:

```
❯ /ase-code-lint -i A21 src/handlers/
```

Lint only the source files with staged Git changes:

```
❯ /ase-code-lint --staged
```

Lint only the unstaged and untracked source files and apply all corrections:

```
❯ /ase-code-lint --auto --unstaged
```

## SEE ALSO

`ase-code-analyze`, `ase-code-edit`, `ase-task-edit`,
`ase-docs-proofread`.

ase-repo-review - Review Changes

## NAME

`ase-repo-review` - Review Changes

## SYNOPSIS

`ase-repo-review`
    [`--help`|`-h`]
    [`--severity`|`-S`=(`AUTO`|`LOW`|`MEDIUM`|`HIGH`)]
    [`--thorough`|`-t`]
    [`--staged`]
    [`--unstaged`]
    [`--branch`|`-b` *branch*]
    [`--commit`|`-c` *commit*]

## DESCRIPTION

The `ase-repo-review` skill performs a *holistic*,
*human-reviewer-style* critique of the *staged* Git changes (or, with
`--unstaged`, of the *unstaged* and *untracked* changes, with both
`--staged` and `--unstaged`, of *all* uncommitted changes, with
`--branch`, of the changes of a *branch* against its *base*, or, with
`--commit`, of a *single commit*) and emits a single
*approve* / *reject* **verdict** backed by *prioritized*,
*severity-tagged*, *line-cited* **findings**. Rather than scanning the
code mechanically, it first *reconstructs the change's own intent* and
then judges the diff *as a whole* against that intent - the way an
experienced reviewer would on a pull request.

The critique spans a fixed set of reviewer *dimensions*: **intent**
(does the diff do what it set out to, without scope creep),
**propagation** (the change fully and consistently carried over to call
sites, types, schemas, registries, catalogs, and analogous sibling
places, with matching contracts and terminology, and without orphans left
behind), **coverage** (all special and edge cases handled, like empty
values, boundaries, error paths, and all variants), **residue** (debug
output, commented-out code, scaffolding, temporary files, build output,
or secrets which do not belong into the change), **correctness**
(latent bugs, broken control/data flow), **design** (fit with the surrounding architecture,
naming, abstraction level), **clarity** (readability and
self-documentation for a future reader), **robustness** (error handling,
resource and concurrency safety), **security** and **performance** (risks
introduced by the change), **convention** (conformance to the
project's documented conventions - code style and the plan/spec
formats described in `AGENTS.md` and the `ase-format-*` meta documents),
**testing** (inadequate coverage for the change - new or fixed behavior
left untested, adjacent tests not updated, or existing tests silently
broken, disabled, or weakened), and **documentation** (user- or
developer-facing docs - `README`, `CHANGELOG`, help text, or AI
guidance/meta documents - left stale by the change).

Each finding carries a *severity* - **HIGH**, **MEDIUM**, **LOW**, or
**ACCEPTED** (a concern that is contractually addressed or accepted as a
documented priority conflict) - and is *evidence-grounded*: it cites the
exact `file:line` location it stems from. The overall verdict is
**REJECT - DEMANDS CHANGES** when any *HIGH* finding remains, and
**APPROVE** otherwise. The work is performed by a dedicated `ase-repo-review`
sub-agent so that the silent reading and read-only repository probing
never leak into the transcript; only the structured verdict and findings
are rendered.

The skill *complements* rather than duplicates its neighbors:
`ase-code-lint` flags *mechanical* code-quality issues, `ase-code-analyze`
inspects *logic and semantics*, `ase-repo-diff` narrates *what changed*
(with optional coherence, risk, and blast-radius reports), and
`ase-meta-diaboli` *adversarially challenges a thesis* - whereas
`ase-repo-review` renders a *reviewer's judgment* on a concrete diff
before it is committed or merged.

## OPTIONS

- `--severity`|- `-S`=(- `AUTO`|- `LOW`|- `MEDIUM`|- `HIGH`):
Set the- *severity floor*(default- `AUTO`): findings below the chosen
threshold are suppressed, ordered- `LOW`<- `MEDIUM`<- `HIGH`. The
default- `AUTO`means- `MEDIUM`for a project with- `grey`boxing and- `LOW`(keep all findings) otherwise; an explicitly given floor always
wins over the boxing, i.e. it also reviews a project with- `black`boxing and, e.g. with- `LOW`, shows all findings of a project with- `grey`boxing.- `ACCEPTED`findings are never suppressed, and
the number of suppressed findings is always reported. Surviving
findings are rendered in- *descending severity*order- `HIGH`,- `MEDIUM`,- `LOW`,- `ACCEPTED`. The floor only affects the rendered
findings table, not the overall- *verdict*, which is always derived
from all findings before the floor is applied.
 
- `--thorough`|- `-t`:
Review- *exhaustively*instead of reporting only a- *few*high-signal
findings: report every well-grounded concern, including minor nits,
and additionally- *plausible*concerns whose impact cannot be fully
proven, marked with "- *Plausible:*" and rated at most- `MEDIUM`.
 
- `--staged`:
Review the- *staged*changes of the working copy. This is the default
if neither- `--unstaged`,- `--branch`, nor- `--commit`is given.
Combined with- `--unstaged`, review- *all*uncommitted changes.
Combined with- `--branch`, additionally review the staged changes of
the worktree where the- *branch*is checked out.
 
- `--unstaged`:
Review the- *unstaged*changes and the- *untracked*files of the
working copy (except- *ASE*'s own state below- `.ase/`) instead of the- *staged*changes. Combined with- `--staged`, review- *all*uncommitted
changes. Combined with- `--branch`, additionally review the unstaged
changes and untracked files of the worktree where the- *branch*is
checked out.
 
- `--branch`|- `-b`- *branch*:
Review the commits of- *branch*(any Git revision) since its fork
point from its- *base*, instead of the- *staged*changes. The base is
determined automatically as the local branch with the- *nearest*fork
point (preferring the checked-out branch on a tie) and is reported
before the review. The commit messages are checked for mismatches
against the diff.
 
- `--commit`|- `-c`- *commit*:
Review the single- *commit*(any Git revision) against its first
parent, instead of the- *staged*changes. Its commit message is
checked for mismatches against the diff. This option cannot be
combined with- `--branch`,- `--staged`, or- `--unstaged`.
 

## ARGUMENTS

The `ase-repo-review` skill takes no positional arguments; it reviews the
currently *staged* Git changes, with `--unstaged` the unstaged and
untracked changes, with both `--staged` and `--unstaged` all uncommitted
changes, with `--branch` a branch against its base, or with `--commit` a
single commit.

## SCENARIOS

- You want the staged changes reviewed like a human reviewer would
- You want an approve or reject verdict before committing
- You want severity-tagged, line-cited findings on a diff
- You want a holistic judgment instead of a mechanical lint
- You want your not yet staged changes reviewed
- You want an exhaustive review of all uncommitted changes
- You want a feature branch reviewed before merging it
- You want a single, already committed change reviewed

## EXAMPLES

Review the currently staged changes before committing:

```
❯ /ase-repo-review
```

Review the staged changes, reporting only `MEDIUM` and `HIGH` findings:

```
❯ /ase-repo-review -S MEDIUM
```

Review the unstaged and untracked changes only:

```
❯ /ase-repo-review --unstaged
```

Review all uncommitted changes exhaustively, reporting all findings:

```
❯ /ase-repo-review --staged --unstaged -t -S LOW
```

Review the branch `feature` against its base, including the uncommitted
changes of its worktree:

```
❯ /ase-repo-review -b feature --staged --unstaged
```

Review the last commit:

```
❯ /ase-repo-review -c HEAD
```

## SEE ALSO

`ase-repo-diff`, `ase-repo-commit`, `ase-code-lint`, `ase-code-analyze`,
`ase-meta-diaboli`.

ase-repo-diff - Summarize Diff

## NAME

`ase-repo-diff` - Summarize Diff

## SYNOPSIS

`ase-repo-diff`
    [`--help`|`-h`]
    [`--coherence`|`-c`]
    [`--risk`|`-r`]
    [`--blast`|`-b`]

## DESCRIPTION

The `ase-repo-diff` skill turns a raw Git diff into a *concise*,
*human-readable* narrative of what changed and why, *grouped by
intent* (such as *Feature*, *Improvement*, *Bugfix*, *Update*,
*Cleanup*, or *Refactor*) rather than by file. It inspects the
*staged* changes (`git diff --cached HEAD`). The result is a compact
table, one row per intent group, each naming the affected files
with their per-file `[+N/-M]` line counts - giving you the essence
of the changes at a glance.

With `--coherence`, the skill additionally *reconstructs the single
intended change* the diff is trying to accomplish - phrased as one crisp
*thesis* sentence - and then walks *every* hunk to *flag those that do
not serve it*: **scope creep** (an unrelated feature or drive-by refactor
riding along) and **stray debug** (leftover prints, commented-out code, or
disabled tests). It emits an overall **COHERENT** / **INCOHERENT** verdict
plus a table of the flagged hunks with what to do with each - keeping the
diff to a *single logical and coherent change* before it is committed.

With `--risk`, the skill additionally *scores* the same diff against a
four-axis **coupling-criticality-coverage-reversibility** rubric and
emits a *graded risk report* - one table row per axis (each scored *1*-*5*
with a one-line evidence justification drawn from the actual hunks), an
overall risk band (**LOW**, **MODERATE**, **HIGH**, or **CRITICAL**),
and one actionable *mitigation* per high-risk axis. The axes combine
with *equal weights*, and the rubric is deliberately *not tuned* to any
specific project: every score is backed by cited evidence so it remains
*overridable*, and the report is *decision support*, not a merge gate.

With `--blast`, the skill additionally renders a *blast-radius map* - a
Mermaid `flowchart` of the *touched modules* and their *reverse
dependencies*, plus a *brief impact summary*. It *extracts the touched
modules* from the changed files, *scans the repository* for the
first-party code that *imports* or *references* those modules, builds a
blast-radius graph (touched modules as origin nodes, dependents fanning
out along the edges), renders it (as an inline image where supported),
and appends a short bullet list of the per-module impact -
giving a visual sense of *what a diff endangers* before a deeper review.

## ARGUMENTS

- `--coherence`,- `-c`:
In addition to the intent-grouped summary, reconstruct the diff's- *single intended change*as one crisp- *thesis*sentence and flag- *every*hunk that does- *not*serve it -- *scope creep*or- *stray
debug*- then emit an overall- **COHERENT**/- **INCOHERENT**verdict
with a table of the flagged hunks.
 
- `--risk`,- `-r`:
In addition to the intent-grouped summary, score the diff against
the- *coupling-criticality-coverage-reversibility*rubric and emit a- *graded risk report*with an overall risk band and per-axis- *mitigations*. Scores are equal-weighted, evidence-backed, and
deliberately untuned to the project.
 
- `--blast`,- `-b`:
In addition to the intent-grouped summary, render a- *blast-radius
map*- a Mermaid- `flowchart`of the- *touched modules*and their- *reverse dependencies*- plus a- *brief impact summary*of what
depends on the touched code and how far the blast reaches.
 

## SCENARIOS

- You want a readable, intent-grouped summary of the staged changes
- You want a diff checked for scope creep and stray debug leftovers
- You want a graded risk report of a change before committing
- You want a blast-radius map of what depends on the touched code

## EXAMPLES

Summarize the currently staged changes:

```
❯ /ase-repo-diff
```

Summarize the staged changes and append an intent-coherence report:

```
❯ /ase-repo-diff --coherence
```

Summarize the staged changes and append a graded risk report:

```
❯ /ase-repo-diff --risk
```

Summarize the staged changes and append a blast-radius map:

```
❯ /ase-repo-diff --blast
```

## SEE ALSO

`ase-repo-commit`, `ase-repo-changelog`, `ase-arch-analyze`.

ase-repo-changelog - Update ChangeLog Entries

## NAME

`ase-repo-changelog` - Update ChangeLog Entries

## SYNOPSIS

`ase-repo-changelog`
    [`--help`|`-h`]

## DESCRIPTION

The `ase-repo-changelog` skill helps to *complete*, *consolidate*, and
*sort* the entries of the most recent section of a `CHANGELOG.md`
file, based on the underlying *Git* commits and the currently staged
changes in the Git *index*.

Each entry starts with `<change-type> [<artifact-kind>]: <summary>`
where *change-type* is one of `FEATURE`, `IMPROVEMENT`, `BUGFIX`,
`UPDATE`, `CLEANUP`, or `REFACTOR`, and *artifact-kind* is one or more
comma-separated *artifact class* tags out of `spec`, `code`, `docs`,
`infr`, or `othr`. Entries are grouped and sorted by this
prefix. The date in the section header is also updated to the current
date.

The layout of the entries follows the style of the existing entries
of the file (e.g. a single line, a title line plus an explaining body,
or a wrapped sentence), and each entry is kept within about 5 lines.

## SCENARIOS

- You want the `CHANGELOG.md`brought up to date with recent commits
- You want changelog entries completed, consolidated, and sorted
- You want a tidy changelog section before a release

## EXAMPLES

Update the most recent ChangeLog section:

```
❯ /ase-repo-changelog
```

## SEE ALSO

`ase-repo-commit`, `ase-docs-proofread`.

ase-repo-commit - Git Commit Message

## NAME

`ase-repo-commit` - Git Commit Message

## SYNOPSIS

`ase-repo-commit`
    [`--help`|`-h`]

## DESCRIPTION

The `ase-repo-commit` skill helps to *craft* a *concise commit
message* for the currently staged Git changes. It inspects the
output of `git diff --cached` and produces a single-line message of
the form `<type>: <summary>` where *type* is one of `FEATURE`,
`IMPROVEMENT`, `BUGFIX`, `UPDATE`, `CLEANUP`, or `REFACTOR`, and
*summary* is a 60-80 character imperative-mood summary without
trailing period or Markdown formatting.

## SCENARIOS

- You want a commit message for the currently staged changes
- You want a typed one-line summary derived from the staged diff
- You want to commit but cannot phrase what changed

## EXAMPLES

Craft a commit message for the currently staged changes:

```
❯ /ase-repo-commit
```

## SEE ALSO

`ase-repo-changelog`.

ase-repo-dissect - Dissect a Change Set

## NAME

`ase-repo-dissect` - Dissect a Change Set

## SYNOPSIS

`ase-repo-dissect`
    [`--help`|`-h`]
    [`--max-parts`|`-m` *count*]
    [`--staged`|`-s`]
    [`--dry`|`-d`]
    [`--force`|`-f`]
    [*dissect-hint*]

## DESCRIPTION

The `ase-repo-dissect` skill treats the current Git change set as an
*epic* and *dissects* it domain-wise and logically into *cohesive
parts*. Each part is a self-contained unit of change with a single
coherent purpose, so it can be reviewed and committed *atomically* and
*independently* of the others.

The change set is captured from the working copy (`git diff`), with
untracked files folded in *read-only*, or -- under `--staged`/`-s` --
from the Git index (`git diff --cached HEAD`). The dissection follows a
strict ruleset: at least *2* and at most *count* parts are derived,
*every* hunk is assigned to *exactly one* part, mutually dependent hunks
are kept together, all hunks of one file normally stay in the same part,
and nothing beyond the actual change set is invented. The cut follows
the *semantics* of the change, not its technical structure -- a split by
file, directory, or technical layer is usually *not* the intended one.
If the change set carries only a single cohesive purpose, it is reported
as *not dissectable* and nothing is created.

An optional *dissect-hint* steers *how* the change set is split -- which
changes belong together, along which axis to cut, or how many parts to
aim at. It is especially useful after a `--dry` run showed an unintended
dissection. The hint overrides the default grouping, but never the
ruleset itself.

The derived parts are first reported as a *dissection table* (part
number, worktree id, scope, and rationale). Then -- unless `--dry` is
given -- each part's patch is written to a temporary file and applied
inside a *fresh Git WorkTree* `.ase/worktree/<project-id>-<feature-slug>`,
whose branch carries the same name and is created from `HEAD`. If a
`CHANGELOG.md` file exists, a matching entry summarizing that part is
added *inside* the worktree.

Every part stays *uncommitted*, a failing `git apply` is reported for
that part while the remaining parts continue, and the *original working
copy is never mutated*.

Note that the working-copy patch (without `--staged`) is taken relative
to the Git *index* while the worktrees are created from `HEAD`. So if
changes are *also* staged, the affected part patches may not apply
cleanly and are then reported as failed; stage everything and use
`--staged` in that situation.

## OPTIONS

- `--max-parts`|- `-m`- *count*:
Upper bound for the number of derived parts (default:- `8`). The
lower bound is always- `2`, because fewer parts are no dissection.
 
- `--staged`|- `-s`:
Dissect the- *staged*changes (the Git index against- `HEAD`) instead
of the working copy changes. Untracked files are then- *not*folded
in, because they are by definition not part of the index.
 
- `--dry`|- `-d`:- *Report only*-- print the dissection table but create- *no*artifacts at all, so the decision stays fully reversible.
 
- `--force`|- `-f`:
Remove and re-create already existing worktrees and branches of the
derived names. Without this option, the skill detects colliding
targets- *before*writing anything, reports them, and stops with
status- *targets exist*.
 

## ARGUMENTS

- *dissect-hint*:
A free-text hint telling- *how*the change set should be split. If
omitted, the parts are derived from the change set alone.

The worktree names are *not* argument-driven: they are always derived
from the *current* project id and the per-part feature slug.

## SCENARIOS

- You want a large uncommitted change set split into atomic parts
- You want each cohesive part of a diff in its own Git worktree
- You want mixed-up changes untangled before committing them
- You want a preview of how a change set would be divided

## EXAMPLES

Dissect the current working copy changes:

```
❯ /ase-repo-dissect
```

Preview the dissection of the staged changes into at most four parts:

```
❯ /ase-repo-dissect --staged --dry --max-parts 4
```

Re-dissect the changes, re-creating the previously created worktrees:

```
❯ /ase-repo-dissect --force
```

Dissect the changes along an explicitly given axis:

```
❯ /ase-repo-dissect keep the plugin and tool changes in separate parts
```

## SEE ALSO

`ase-task-dissect`, `ase-repo-diff`, `ase-repo-review`,
`ase-repo-changelog`, `ase-repo-commit`,
`ase-repo-stage`.

ase-repo-stage - Stage a Part of the Changes

## NAME

`ase-repo-stage` - Stage a Part of the Changes

## SYNOPSIS

`ase-repo-stage`
    [`--help`|`-h`]
    [`--entry`|`-e`]
    [`--turn`|`-t`]
    [`--session`|`-s`]
    [`--merge`|`-m`]
    [`--no-verify`|`-V`]
    [`--worktree`|`-w`]
    [`--changelog`|`-c`]
    [`--commit`|`-C`]
    [*topic*]

## DESCRIPTION

The `ase-repo-stage` skill stages only the *part* of the current Git working copy changes which
belongs to a *selection*, so a mixed change set can be committed in cohesive portions without
hand-crafted partial patches.

The selection is either a free-text *topic*, the *last* `CHANGELOG.md` entry (`--entry`), the
changes of the *last turn* (`--turn`), or all changes of the current *session* (`--session`).
Without an option, the selection is inferred from the *topic* text, so `stage the last turn`
works, too. An explicitly given option overrides the inference, and a remaining *topic* narrows
the entry, turn, and session selections.

The changes are picked at the granularity of individual *lines*: hunks which belong to the
selection only partially are *split*, and untracked files are selectable, even partially.
Mutually dependent changes, like a use and its import, are kept together. Unless `--merge` is
given, the selection *replaces* the current index content. Only the Git index is changed, never
the content of a working copy file.

After staging, the skill checks that the *staged state alone builds*, with the build command
auto-detected from the project. By default, the remaining unstaged changes are temporarily
stashed for this, or -- under `--worktree` -- the staged state is built in a temporary worktree.
A failed build keeps the staged selection and is reported with its essential error lines.

Finally, the skill optionally chains into the ChangeLog updating and the commit message
crafting, both for the staged selection.

## OPTIONS

- `--entry`|- `-e`:
Select the changes which realize the- *last*- `CHANGELOG.md`entry, i.e. the entry added or
modified by the uncommitted change of- `CHANGELOG.md`, plus the entry itself.
 
- `--turn`|- `-t`:
Select the changes made by the agent in the- *last turn*before the skill invocation.
 
- `--session`|- `-s`:
Select all changes made by the agent in the current- *session*.
 
- `--merge`|- `-m`:
Stage the selection- *on top*of the already staged changes, instead of replacing them.
 
- `--no-verify`|- `-V`:
Skip the check that the staged state builds on its own.
 
- `--worktree`|- `-w`:
Check the build in a- *temporary Git worktree*holding only the staged state, instead of
temporarily stashing the unstaged changes of the working copy.
 
- `--changelog`|- `-c`:
Afterwards, update the- `CHANGELOG.md`entries via- `ase-repo-changelog`and stage exactly the
entries it added or updated.
 
- `--commit`|- `-C`:
Afterwards, craft the commit message of the staged selection via- `ase-repo-commit`.
 

The options `--entry`, `--turn`, and `--session` are mutually exclusive.

## ARGUMENTS

- *topic*:
A free-text description of the changes to stage, or -- without an option -- a reference
like- `last turn`or- `last changelog entry`the selection is inferred from.

## SCENARIOS

- You want only the hunks related to a certain topic staged
- You want only the changes of the last turn or of the session staged
- You want the changes behind a `CHANGELOG.md`entry staged and committed
- You want to be sure the staged part builds without the rest of the changes

## EXAMPLES

Stage only the hunks related to a topic:

```
❯ /ase-repo-stage the getopt negation handling
```

Stage the changes of the last turn, checking the build in a temporary worktree:

```
❯ /ase-repo-stage --turn --worktree
```

Stage the changes of the last ChangeLog entry and craft its commit message:

```
❯ /ase-repo-stage --entry --commit
```

Add the session changes related to a topic to the already staged ones:

```
❯ /ase-repo-stage --session --merge task board
```

## SEE ALSO

`ase-repo-dissect`, `ase-repo-diff`,
`ase-repo-review`, `ase-repo-commit`,
`ase-repo-changelog`.

ase-repo-merge - Merge a Branch

## NAME

`ase-repo-merge` - Merge a Branch

## SYNOPSIS

`ase-repo-merge`
    [`--help`|`-h`]
    [`--target`|`-t` *branch*]
    [`--mode`|`-m` `merge`|`rebase`|`squash`]
    [`--cleanup`|`-c`]
    *source-branch*

## DESCRIPTION

The `ase-repo-merge` skill *merges* a Git *source-branch* into a target
branch and reports the result as a *merge verdict*: `MERGED`,
`CONFLICT`, or `FAILED`.

Still *uncommitted* changes in the worktree where the source branch is
checked out are first *committed* with a derived one-line commit
message of the form `<type>: <summary>`. The source branch is then
merged inside the worktree where the target branch is checked out,
which has to be free of uncommitted changes, in one of three modes: a
regular *merge* with a merge commit (`git merge --no-ff`), a *rebase*
of the source branch onto the target branch followed by a
fast-forward (`git rebase` plus `git merge --ff-only`), or a *squash*
of all source commits into one commit (`git merge --squash`).
Merge conflicts are *resolved semantically*, honoring the intents of
both sides and never losing any change, through the same procedure as
`ase-repo-resolve`. If a conflict is unresolvable, because both
intents contradict each other or no confident resolution exists, the
unresolvable conflicts are reported, the merge is *aborted*, the
target branch is left untouched, and the verdict is `CONFLICT`. Finally, the skill *checks*
that the source branch is contained in the target branch. It does
*not* build or test the merge result, so semantically resolved
conflicts still deserve a review.

On a `CONFLICT` or `FAILED` verdict, the commit of the previously
uncommitted source changes is *undone* again, so they are uncommitted
as before (but no longer partially staged).

If the source branch *equals* the target branch, only its uncommitted
changes are committed. The skill is also used by `ase agent integrator`
to drive the *Integration* phase of the `enterprise` task lifecycle
model headless. As the target branch is by default the checked-out
branch of the working copy of the project, any uncommitted change
there lets the merge of every task with its own branch fail, and hence
the task become `DEFERRED`, while for a task with an in-place change
set it is committed together with that change set. So, when running
the integrator, keep the working copy clean and do your own work in
separate worktrees.

## OPTIONS

- `--target`|- `-t`- *branch*:
The target branch of the merge (default:- `current`, the checked-out
branch of the working copy).
 
- `--mode`|- `-m`- `merge`|- `rebase`|- `squash`:
The merge mode (default:- `merge`):- `merge`creates a merge commit,- `rebase`rebases the source branch (rewriting its commits, in its
own worktree if it is checked out) onto the target branch and
fast-forwards the target branch, and- `squash`combines all changes
of the source branch into one new commit on the target branch.
Conflicts are resolved per rebased commit under- `rebase`.
 
- `--cleanup`|- `-c`:
After a successful merge, remove the worktree of the source branch
(if any) and delete the merged source branch.
 

## ARGUMENTS

- *source-branch*:
The Git branch to merge.

## SCENARIOS

- You want a feature branch landed on the checked-out branch
- You want the uncommitted result of a worktree committed and merged
- You want merge conflicts resolved instead of hand-editing them

## EXAMPLES

Merge the branch `feature-x` into the checked-out branch:

```
❯ /ase-repo-merge feature-x
```

Merge the branch `feature-x` into `develop` and remove it afterwards:

```
❯ /ase-repo-merge --target develop --cleanup feature-x
```

Squash the branch `feature-x` into one commit on the checked-out branch:

```
❯ /ase-repo-merge --mode squash feature-x
```

## SEE ALSO

`ase-task-implement`, `ase-repo-resolve`, `ase-repo-review`,
`ase-repo-commit`, `ase-repo-dissect`.

ase-repo-resolve - Resolve Merge Conflicts

## NAME

`ase-repo-resolve` - Resolve Merge Conflicts

## SYNOPSIS

`ase-repo-resolve`
    [`--help`|`-h`]
    [`--dir`|`-d` *dir*]
    [`--safe`|`-s`]
    [`--interactive`|`-i`]
    [*path* ...]

## DESCRIPTION

The `ase-repo-resolve` skill *resolves* the Git merge conflicts of a
working directory and reports the result as a *resolve verdict*:
`RESOLVED`, `PARTIAL`, `NONE`, or `FAILED`.

It handles every *unmerged* state of the Git index -- left by an
in-progress `git merge`, `git rebase`, `git cherry-pick`, `git revert`,
or by `git stash apply` -- and additionally files carrying conflict
markers *without* an unmerged index entry, e.g., left by `patch --merge`. Reject files (`*.rej`) of `patch` are escalated, but never
touched.

Every conflict hunk is resolved *semantically*: the intents of both
sides are determined from the commits, the base version, and the
surrounding code, and the hunk is resolved so that *both* intents are
honored. Trivial, independent, and compatible changes are combined,
while a hunk whose intents *contradict* each other, or for which no
confident resolution exists, is *kept as-is* including its conflict
markers and *escalated* to the user. Files with only some unresolvable
hunks are resolved *partially* and stay unmerged. Non-content conflicts
(binary, submodule, modify/delete, rename, delete/delete) are resolved
only if their intent is unambiguous.

The skill is very careful to *never lose* any change: every touched
file is first backed up into the `ase-resolve` directory inside the Git
directory, and every resolved hunk is checked to still reflect all
changes of both sides relative to the base; a hunk failing this check
is restored with its conflict markers and escalated. The backups are
removed on a `RESOLVED` verdict and kept otherwise.

Fully resolved unmerged files are staged. On a `RESOLVED` verdict, the
in-progress operation is *continued* (e.g., `git merge --continue`),
and, if this stops at the next conflicting commit of a rebase,
cherry-pick, or revert, those conflicts are resolved, too. The skill
does *not* build or test the result, so the resolved conflicts still
deserve a review.

The same resolution procedure is used by `ase-repo-merge` for the
conflicts of its merge and by `ase-repo-import` for the conflicts of
its import.

## OPTIONS

- `--dir`|- `-d`- *dir*:
The Git working directory whose conflicts are resolved (default:- `.`, the current working directory).
 
- `--safe`|- `-s`:
Never touch non-content conflicts (binary, submodule,
modify/delete, rename, delete/delete), but always escalate them.
 
- `--interactive`|- `-i`:
Instead of directly escalating an unresolvable hunk, show it and
let the user choose- `OURS`,- `THEIRS`,- `BOTH`,- `KEEP`(escalate), or
describe a manual resolution. Ignored when running headless.
 

## ARGUMENTS

- *path*...:
Restrict the resolution to the conflicted files equal to or below
the given paths. The in-progress operation is then continued only
if no conflicted files remain elsewhere.

## SCENARIOS

- You ran into merge conflicts during a merge, rebase, or cherry-pick
- You applied a stash or a patch and got conflict markers
- You want conflicts resolved without risking the loss of any change

## EXAMPLES

Resolve all conflicts of the current working directory:

```
❯ /ase-repo-resolve
```

Resolve the conflicts below `src/`, deciding unresolvable hunks
interactively:

```
❯ /ase-repo-resolve --interactive src/
```

## SEE ALSO

`ase-repo-merge`, `ase-repo-import`, `ase-repo-review`,
`ase-repo-commit`, `ase-repo-diff`.

ase-repo-import - Import a Change Set

## NAME

`ase-repo-import` - Import a Change Set

## SYNOPSIS

`ase-repo-import`
    [`--help`|`-h`]
    [`--safe`|`-s`]
    [`--interactive`|`-i`]
    *url*|`#`*number*|*patch-file*

## DESCRIPTION

The `ase-repo-import` skill *imports* a change set into the current Git
working copy as *unstaged* and *untracked* files, so nothing gets staged
or committed, and reports the result as an *import verdict*: `IMPORTED`,
`CONFLICT`, or `FAILED`.

The change set of a pull or merge request consists of all its commits,
flattened into one change, relative to its fork point from the default
branch of the remote. Hence, a request targeting another branch also
carries the commits this branch has on top of the default branch.

Local changes are preserved: a file with local changes also receives
the imported changes, and its staged state stays as it is. Files without
local changes get a 3-way merge. Hunks which do not apply, e.g., for
lack of the base version in a foreign patch file, become conflicts.

Conflicts are *resolved semantically* through the same procedure as
`ase-repo-resolve`, *never* losing any change. A conflict without a
confident resolution *keeps its conflict markers* in the file and is
listed at the end as the *TODO*, together with the changes which cannot
be expressed as conflict markers (e.g., of a file missing locally). The
verdict is then `CONFLICT`. On a `FAILED` verdict, the working copy is
left untouched. The skill does *not* build or test the result, so the
imported changes still deserve a review.

Unless the verdict is `FAILED`, the skill finally summarizes briefly
what the change set does and reports its files in a table with the type
of change (added, modified, renamed, or removed), the file path relative
to the root of the working copy, and a summary of the change.

## OPTIONS

- `--safe`|- `-s`:
Never touch non-content conflicts (binary, submodule,
modify/delete, rename, delete/delete), but always list them in the
TODO.
 
- `--interactive`|- `-i`:
Instead of directly listing an unresolvable hunk in the TODO, show
it and let the user choose- `OURS`,- `THEIRS`,- `BOTH`,- `KEEP`, or
describe a manual resolution. Ignored when running headless.
 

## ARGUMENTS

- *url*:
The URL of a pull or merge request (- `.../pull/103`of GitHub,- `.../pulls/103`of Gitea, or- `.../-/merge_requests/103`of GitLab)
or of a commit (- `.../commit/<sha>`or- `.../-/commit/<sha>`).
 
- `#`- *number*:
The number of a pull or merge request on the forge of the remote- `upstream`, or- `origin`if it does not exist. It is never a task id
of ASE. On GitLab, it denotes the merge request- `!`- *number*.
 
- *patch-file*:
The path of a diff or patch file, in the format of- `git diff`,- `git format-patch`, or- `diff -u`.
 

## SCENARIOS

- You want to try out a pull request without checking out its branch
- You want a change of another repository on top of your local changes
- You received a patch file which no longer applies cleanly

## EXAMPLES

Import a pull request by its URL:

```
❯ /ase-repo-import https://github.com/rse/ase/pull/103
```

Import a pull request by its number, deciding unresolvable hunks
interactively:

```
❯ /ase-repo-import --interactive #103
```

Import a patch file:

```
❯ /ase-repo-import ../fix-crash.patch
```

## SEE ALSO

`ase-repo-resolve`, `ase-repo-merge`,
`ase-repo-review`, `ase-repo-stage`,
`ase-sync-import`.

ase-sync-reconcile - Reconcile Artifact Set to Artifact Set

## NAME

`ase-sync-reconcile` - Reconcile Artifact Set to Artifact Set

## SYNOPSIS

`ase-sync-reconcile`
    [`--help`|`-h`]
    [`--bidirectional`|`-b`]
    [`--operation`|`-o` *op*[,...]]
    [`--dry`|`-d`]
    [`--target`|`-t` *target*[,...]]
    [`--source`|`-s` *source*[,...]]
    [*hint*]

## DESCRIPTION

The `ase-sync-reconcile` skill reconciles one set of artifact kinds (the
*target*) to *reflect* the *current state* of another set of artifact
kinds (the *source*). It reads the source artifacts and then adjusts the
target artifacts *directly* and *surgically* to match the source state,
while optionally honoring a filtering *hint*.

Both *target* and *source* are comma-separated lists over the six
recognized artifact kinds `SPEC` (Specification, covering both
requirements and architecture), `CODE` (Source Code), `DOCS`
(Documentation), `TASK` (Task Plans), `INFR` (Infrastructure), and
`OTHR` (catch-all). When *source* is `auto`, it resolves to all six
kinds *minus* the kinds listed in *target*. Unless `--bidirectional` is
given, a kind present in *target* is never used as its own source.

The file lists for all involved kinds are resolved via the
`ase_artifact_list` MCP tool of the `ase` MCP server. While reconciling,
the skill honors the artifact-format conventions of `ase-format-meta.md`,
`ase-format-spec.md` (the *SpecBook* models and formats plus the
SpecBook schema configuration), and `ase-format-task.md`; the kinds
`CODE`, `DOCS`, `INFR`, and `OTHR` have no dedicated format contract and
are treated as free-form. Changed `SPEC` artifacts are validated via
the `ase_specbook_lint` MCP tool and the reported diagnostics are fixed
in at most three rounds; any remaining diagnostics are surfaced.

The changes on the *target* side are the three *operations* `add`
(create content and files the source warrants but the target lacks),
`update` (change existing content the source contradicts), and `remove`
(delete content and files the source no longer supports). The
`--operation` option restricts which of them are applied; whatever an
excluded operation would have changed stays untouched. With `--dry`, no
target artifact is modified at all and the intended changes are shown as
a unified diff instead.

## OPTIONS

- `--bidirectional`|- `-b`:
Reconcile the- *target*and- *source*artifacts so that they
faithfully reflect the current state of- *each other*, instead of
only updating the- *target*from the- *source*. With this flag, a kind
present in- *target*is- *not*removed from- *source*, and the- `--operation`restriction applies to both directions.
 
- `--operation`|- `-o`- *op*[,...]:
The comma-separated list of operations to apply on the target side,
each one of- `add`,- `update`, or- `remove`. The special value- `all`(the default) selects all three operations.
 
- `--dry`|- `-d`:
Perform the regular reconciliation, but do- *not*modify any target
artifact (neither files nor task plans). Instead, show the intended
changes as a single unified diff. The SpecBook validation is skipped,
as nothing is written to disk.
 
- `--target`|- `-t`- *target*[,...]:
The comma-separated list of artifact kinds to update (default:- `CODE,DOCS,INFR,OTHR`). An empty target lets the skill error out.
 
- `--source`|- `-s`- *source*[,...]:
The comma-separated list of artifact kinds to update- *from*. The
special value- `auto`(the default) resolves to all recognized kinds
except those in- *target*.
 

## ARGUMENTS

- *hint*:
An optional free-form filtering hint that narrows the source and/or
target artifacts, or the aspects of those artifacts to take into
account during reconciliation.

## SCENARIOS

- You want one artifact set updated to reflect another one
- You want code and docs aligned with the specification
- You want the specification recovered from the code base
- You want spec and code bidirectionally synchronized
- You want only missing parts added to the documentation, never anything removed
- You want to preview the reconciliation as a diff before applying it

## EXAMPLES

Reconcile the code and documentation
to reflect the current specification
in a "forward engineering" approach:

```
❯ /ase-sync-reconcile -t CODE,DOCS -s SPEC
```

Reconcile the specification from everything else
in a "reverse engineering" approach:

```
❯ /ase-sync-reconcile -t SPEC -s CODE,DOCS
```

Bidirectionally reconcile specification and code against
each other, limited to the authentication aspect:

```
❯ /ase-sync-reconcile -b -t SPEC -s CODE authentication
```

Add missing and update outdated parts of the documentation
from the code, but never remove anything:

```
❯ /ase-sync-reconcile -o add,update -t DOCS -s CODE
```

Preview the reconciliation of the code from the
specification as a unified diff, without applying it:

```
❯ /ase-sync-reconcile -d -t CODE -s SPEC
```

## SEE ALSO

`ase-repo-changelog`,
`ase-task-implement`.

ase-sync-import - Import Foreign Sources into Artifact Set

## NAME

`ase-sync-import` - Import Foreign Sources into Artifact Set

## SYNOPSIS

`ase-sync-import`
    [`--help`|`-h`]
    [`--target`|`-t` *target*[,...]]
    *hint*

## DESCRIPTION

The `ase-sync-import` skill imports information from *foreign sources*
into a set of artifact kinds (the *target*), generating or updating the
target artifacts so they faithfully *reflect* the imported information.
Unlike `ase-sync-reconcile`, which syncs *between* existing ASE artifact
kinds, `ase-sync-import` brings in information from *external* sources
that live outside the ASE artifact sets.

The *foreign sources* are named by the free-form *hint* argument and may
be local files or directories, remote URLs, pasted text passages, or
references to other documents. Local files are read via the `Read` tool
and remote URLs are fetched via the available web tools.

The *target* is a comma-separated list over the six recognized
artifact kinds `SPEC` (Specification, covering both requirements and
architecture), `CODE` (Source Code), `DOCS` (Documentation), `TASK`
(Task Plans), `INFR` (Infrastructure), and `OTHR` (catch-all). It
defaults to `SPEC`. The file lists for the involved kinds are resolved
via the `ase_artifact_list` MCP tool of the `ase` MCP server.

While importing, the skill honors the artifact-format conventions of
`ase-format-meta.md`, `ase-format-spec.md` (the *SpecBook* models and
formats plus the SpecBook schema configuration), and
`ase-format-task.md`; the kinds `CODE`, `DOCS`, `INFR`, and `OTHR` have
no dedicated format contract and are treated as free-form. A target
artifact that does not yet exist but is warranted by the imported
information is *generated* from scratch, while an existing target
artifact is *surgically updated* to reflect the imported information.
Generated or updated `SPEC` artifacts are validated via the
`ase_specbook_lint` MCP tool and the reported diagnostics are fixed in
at most three rounds; any remaining diagnostics are surfaced.

## OPTIONS

- `--target`|- `-t`- *target*[,...]:
The comma-separated list of artifact kinds to generate or update.
Defaults to- `SPEC`. The skill errors out on an empty target or
an unknown/unsupported kind.

## ARGUMENTS

- *hint*:
The mandatory free-form description of the foreign sources to import
from (e.g. a file path, directory, URL, or pasted text). The skill
errors out on an empty hint.

## SCENARIOS

- You want external sources brought into the project artifacts
- You want a foreign document, URL, or pasted text reflected in the `SPEC`
- You want requirements from outside imported into a task plan
- You want legacy material ingested into the specification

## EXAMPLES

Import a foreign requirements document into the specification:

```
❯ /ase-sync-import -t SPEC docs/legacy/requirements.txt
```

Import an external design write-up into the specification (the default
target), which covers the architecture, too:

```
❯ /ase-sync-import https://example.com/design-notes.html
```

Import a pasted feature description into a task plan:

```
❯ /ase-sync-import -t TASK the new export feature must support CSV and JSON
```

## SEE ALSO

`ase-sync-reconcile`,
`ase-sync-export`,
`ase-task-implement`.

ase-sync-export - Export Specification into Rendered Files

## NAME

`ase-sync-export` - Export Specification into Rendered Files

## SYNOPSIS

`ase-sync-export`
    [`--help`|`-h`]
    [`--output`|`-o` *output*[,...]]

## DESCRIPTION

The `ase-sync-export` skill exports the `SPEC` artifact set -- the
*SpecBook*-based specification of the project, located via the
`project.artifact.spec.basedir` configuration -- into *derived*,
ready-to-consume renderings. The specification is first validated
against the SpecBook schema configuration via the `ase_specbook_lint`
MCP tool of the `ase` MCP server; on any diagnostic the skill errors out
and exports nothing, as a partial or invalid specification must never be
rendered. Otherwise every requested *output* is rendered via the
`ase_specbook_export` MCP tool.

Each *output* is an `[<format>:]<file>` entry, where the format is one
of `json`, `json5`, `yaml`, `toon` (the specification object model),
`html` (self-contained single document with table of contents, full-text
search, and light/dark theme), `pdf` (paginated print document), or `md`
(normalized single Markdown document). The format is inferred from the
filename extension unless it is explicitly given as a prefix. The
standard output sentinel `-` is not supported.

The `pdf` format is the only one with an external prerequisite: it
prints the rendering through a Chromium-class browser. Neither `npm install -g @rse/ase` nor `ase setup install` downloads one, so unless a
*Google Chrome* is installed system-wide, the Playwright Chromium has to
be fetched once via `npx playwright install chromium`. Without any such
browser the export fails up-front, before the specification is parsed,
and reports that very command; when it falls back onto a system-installed
*Google Chrome*, the skill reports this as an environment notice.

## OPTIONS

- `--output`|- `-o`- *output*[,...]:
The comma-separated list of- `[<format>:]<file>`entries to render,
with each file path relative to the project root. Defaults to the
single HTML rendering- `index.html`inside the- `SPEC`base directory
(e.g.- `docs/specbook/index.html`).

## SCENARIOS

- You want the specification rendered as HTML, PDF, Markdown, or JSON
- You want a shareable, ready-to-consume export of the `SPEC`artifacts
- You want the specification validated and then materialized into files

## EXAMPLES

Export the specification to its default HTML rendering:

```
❯ /ase-sync-export
```

Export the specification as HTML and PDF:

```
❯ /ase-sync-export -o docs/spec.html,docs/spec.pdf
```

Export the specification object model as YAML into a file without a
telling extension:

```
❯ /ase-sync-export -o yaml:docs/spec.model
```

## SEE ALSO

`ase-sync-reconcile`,
`ase-sync-import`.

ase-task-list - List Task Plans

## NAME

`ase-task-list` - List Task Plans

## SYNOPSIS

`ase-task-list`
    [`--help`|`-h`]
    [`--verbose`|`-v`]
    [`--include`|`-i`=*state*[`,`...]]
    [`--exclude`|`-e`=*state*[`,`...]]

## DESCRIPTION

The `ase-task-list` skill lists all available *task ids* in the
current project by calling the `ase_task_list` MCP tool. In the
default mode, only the task ids are rendered as a single-column
Markdown table. In verbose mode, the lifecycle status, the effort size
(see `ase-task-estimate`), and the last-modified timestamp of each task
plan are rendered as additional columns.

The listing is restricted to an *effective state set*, derived from the
`Status:` frontmatter key of each task plan (which defaults to the
initial state of the task lifecycle model for a plan carrying no such
key): with `--include` only, exactly the listed states are shown; with
`--exclude` only, all states except the listed ones; with both, the
included ones minus the excluded ones. An unknown state, or a
combination which cancels out to an empty set, aborts the skill with an
error. The `finished` sentinel stands for the finished states of the
task lifecycle model. By default, `--exclude finished` is in effect, so
finished and abandoned task plans stay out of the way. A task plan whose
`Status:` is not a state of the task lifecycle model at all is never
filtered out: it is kept in the listing and reported with a warning.

The states depend on the task lifecycle model configured for the
project via `project.task.lifecycle`. The `solo` model (default) has
the 4 states (initial: `OPEN`, finished: `CLOSED` and `CANCELLED`):

```
OPEN  SHELVED  CLOSED  CANCELLED
```

The `team` model has the 6 states (initial: `PLANNING`, finished:
`IMPLEMENTED` and `CANCELLED`):

```
PLANNING  SHELVED  IMPLEMENTING  STALLED  IMPLEMENTED  CANCELLED
```

The `enterprise` model has the 14 states (initial: `DRAFTED`, finished:
`INTEGRATED` and `CANCELLED`):

```
DRAFTED      SHELVED   PLANNING     PLANNED   STALLED
IMPLEMENTING IMPLEMENTED DECLINED   QUALIFYING QUALIFIED
DEFERRED     INTEGRATING INTEGRATED CANCELLED
```

## OPTIONS

- `--verbose`|- `-v`:
Render an additional- `Status`column with the lifecycle state, an
additional- `Effort`column with the T-shirt size (- `-`if not
estimated), and an additional- `Last Modified`column with the- `YYYY-MM-DD HH:MM`timestamp of each task plan.
 
- `--include`|- `-i`=- *state*[- `,`...]:
Restrict the listed task plans to the given comma-separated list of
lifecycle states (e.g.- `IMPLEMENTING,STALLED`). Without this option,
all states are listed. The- `none`sentinel (the default) imposes no
restriction, the- `finished`sentinel selects the finished states.
 
- `--exclude`|- `-e`=- *state*[- `,`...]:
Remove the given comma-separated list of lifecycle states from the
listed task plans. Applied- *after*- `--include`, so- `-i PLANNING,IMPLEMENTING -e IMPLEMENTING`lists- `PLANNING`only.
Defaults to- `finished`; pass- `--exclude none`to suppress the
default and list task plans in every state.
 

## SCENARIOS

- You want an overview of all task plans
- You want task ids listed with status and timestamps
- You want the unfinished or in-progress tasks found

## EXAMPLES

List all unfinished task ids:

```
❯ /ase-task-list
```

List all task ids together with their status and last-modified timestamps:

```
❯ /ase-task-list --verbose
```

List the task ids of every task plan, including the finished ones:

```
❯ /ase-task-list --exclude none
```

List only the task ids of the task plans currently under work:

```
❯ /ase-task-list --include IMPLEMENTING,STALLED
```

## SEE ALSO

`ase-task-id`, `ase-task-view`, `ase-task-edit`,
`ase-task-rename`, `ase-task-delete`.

ase-task-rename - Rename a Task Plan

## NAME

`ase-task-rename` - Rename a Task Plan

## SYNOPSIS

`ase-task-rename`
    [`--help`|`-h`]
    [*old*] *new*

## DESCRIPTION

The `ase-task-rename` skill renames a *task plan* from *old* to
*new* via the `ase_task_rename` MCP tool. If only one argument is
given, it is treated as *new* and *old* defaults to the *current*
task id. When the renamed task is the current task, the current
task id is automatically switched to *new*.

## ARGUMENTS

- [*old*]*new*:
The*old*task id to rename and the*new*task id to assign.
If only one token is given,*old*defaults to the current
task id.

## SCENARIOS

- You want a task plan renamed
- You want a task given a better id
- You want the current task moved to a new id

## EXAMPLES

Rename the current task:

```
❯ /ase-task-rename hello-world
```

Rename a specific task:

```
❯ /ase-task-rename old-id new-id
```

## SEE ALSO

`ase-task-list`, `ase-task-id`, `ase-task-edit`,
`ase-task-view`, `ase-task-delete`.

ase-task-delete - Delete a Task Plan

## NAME

`ase-task-delete` - Delete a Task Plan

## SYNOPSIS

`ase-task-delete`
    [`--help`|`-h`]
    [*id*]

## DESCRIPTION

The `ase-task-delete` skill deletes the *task plan* identified by
*id*. If *id* is omitted, the *current* task id (inherited from the
session context) is used. When the deleted task is the current task
and not the `default` task, the current task id is automatically
switched back to `default`.

## ARGUMENTS

- *id*:
The unique identifier of the task plan to delete. If omitted,
the current task id is used.

## SCENARIOS

- You want a task plan removed
- You want a finished or obsolete task cleared
- You want a plan discarded, falling back to the default task

## EXAMPLES

Delete the current task plan:

```
❯ /ase-task-delete
```

Delete a specific task plan:

```
❯ /ase-task-delete hello
```

## SEE ALSO

`ase-task-edit`, `ase-task-list`, `ase-task-view`,
`ase-task-rename`, `ase-task-reboot`.

ase-task-id - Configure Task Id

## NAME

`ase-task-id` - Configure Task Id

## SYNOPSIS

`ase-task-id`
    [`--help`|`-h`]
    [*id*]

## DESCRIPTION

The `ase-task-id` skill gets or sets the unique *task id* for the
current session. Without arguments, it reports the currently active
task id. With an *id* argument, it switches the session to that
task id via the `ase_task_id` MCP tool.

## ARGUMENTS

- *id*:
The new task id to activate (characters- `A-Z`,- `a-z`,- `0-9`,- `#`,- `_`, and- `-`). If omitted, the currently active task id is
reported. An id not conforming to the task id scheme of the
project (- `project.task.idscheme`:- `slug[:<words>]`,- `seq[:<template>]`,
or- `any`) is accepted, but noticed.

## SCENARIOS

- You want to know which task is currently active
- You want the session switched to another task
- You want to work on a certain task

## EXAMPLES

Show the current task id:

```
❯ /ase-task-id
```

Switch to a specific task:

```
❯ /ase-task-id hello
```

## SEE ALSO

`ase-task-list`, `ase-task-edit`, `ase-task-view`,
`ase-task-status`, `ase-task-rename`, `ase-task-delete`.

ase-task-status - Configure Task Status

## NAME

`ase-task-status` - Configure Task Status

## SYNOPSIS

`ase-task-status`
    [`--help`|`-h`]
    [*id*`:`] [*status*]

## DESCRIPTION

The `ase-task-status` skill gets or sets the *lifecycle status* of a
*task plan*, i.e. its `Status:` frontmatter key, via the
`ase_task_status` MCP tool. Without a *status* argument, it reports the
current status of the plan (defaulting to the initial state of the task
lifecycle model if the plan carries no `Status:` key). With a *status*
argument, it sets the status of the plan, leaving its `Modified:` key
alone, as that key tracks changes of the plan body only.

The *status* is matched case-insensitively against the states of the
task lifecycle model configured for the project via
`project.task.lifecycle` (`solo`, `team`, or `enterprise`) and rejected
if it is no such state. A status not reachable from the current one via
one or more transitions of the state machine of the model is *rejected*,
too.

If *id*`:` is omitted, the *current* task id (inherited from the session
context) is used. A single bare token which is no state of the model is
also taken as the task id. The current task id of the session is never
switched.

## ARGUMENTS

- *id*- `:`:
The unique identifier of the task plan, with a trailing colon.
If omitted, the current task id is used.
 
- *status*:
The lifecycle status to set, one of the states of the configured
task lifecycle model (case-insensitive). If omitted, the current
status is reported.
 

## SCENARIOS

- You want to know the lifecycle status of the current or a given task plan
- You want a task plan closed, shelved, cancelled, or re-opened
- You want a task plan moved into a certain lifecycle state by hand

## EXAMPLES

Show the status of the current task plan:

```
❯ /ase-task-status
```

Show the status of a specific task plan:

```
❯ /ase-task-status hello:
```

Close the current task plan (`solo` lifecycle model):

```
❯ /ase-task-status closed
```

Shelve a specific task plan:

```
❯ /ase-task-status hello: SHELVED
```

## SEE ALSO

`ase-task-list`, `ase-task-id`, `ase-task-view`,
`ase-task-implement`, `ase-task-delete`.

ase-task-assign - Assign a Task Plan

## NAME

`ase-task-assign` - Assign a Task Plan

## SYNOPSIS

`ase-task-assign`
    [`--help`|`-h`]
    [*id*`:`] *assignee*

## DESCRIPTION

The `ase-task-assign` skill assigns a *task plan* to an *assignee*, i.e.
sets its `Assignee:` frontmatter key, via the `ase_task_assign` MCP tool,
leaving its `Modified:` key alone, as that key tracks changes of the plan
body only. Additionally, it *accounts* the *AI usage* of the session onto
the task:

- Assigning the task to - *oneself*(- `me`or the current user id) opens
the- *usage window*of the task, or attaches the session to an already
open one (e.g. of a previous session). From then on, all turns of the
session are accounted: the AI request time, the cache read, cache
write, input, and output tokens, and the costs, as reported by the
agent tool via- *OpenTelemetry*to the ASE service, plus, in addition,
the human time (the whole real time of the window, as the engineer
costs money even while waiting for the AI) and its costs at the
hourly rate of the assignee in- `telemetry.prices.hr`(default: 110 USD).
 
- Assigning the task to - *anybody else*, or to- *nobody*with- `none`(removing the- `Assignee:`key),- *gives it back*: it closes the usage
window and, after a grace period of 10 seconds for late telemetry,
adds the accumulated usage to the usage attachment of the task
(- `Type: application/yaml; kind="usage"`), whose YAML data carries the
amounts- `time: { ai, hr, real }`(minutes),- `token: { cread, cwrite, input, output }`(counts), and- `cost: { ai, hr }`(USD). Each
booking- *adds*to the existing amounts, atomically, so concurrent
changes of the task (e.g. by the agents of- `ase agent`) are never lost.
 

A usage window also closes and books when its last session *ends*, and
when the `Assignee:` of the task is changed *elsewhere* (e.g. on the task
board) while the task is the current task of the session. Conversely,
changing the `Assignee:` of a task to oneself *elsewhere* (e.g. on the
task board) opens its usage window, too, and attaches every session of
the project whose current task it is (also a session switching to it,
or starting with it, later on), with the window starting *paused*, i.e.,
accounting from the next AI request of its sessions on. An *unassigned*
task is even accounted *implicitly*, without any assignment: while it is
the current task of a session, the session is attached to its usage
window in the same way, until the session switches to another task, the
task gets assigned to anybody else, or the session ends (configurable via
`telemetry.implicit`, default: `true`, never for task `default`), so
explicitly assigning a task to oneself is needed only for claiming it,
e.g. against the agents of `ase agent`. Long windows
additionally book every `telemetry.service.interval` minutes (default: 10).
Without any AI request for `telemetry.service.idle` minutes (default: 30),
a usage window *pauses*, and the next AI request (of any kind, not just
ASE skills) silently *resumes* it, so forgetting a task assigned to
oneself does not account an idle session endlessly. A usage window without
any AI request for 7 days is closed and booked.

The accounting is *opt-in*: it requires the configuration key
`telemetry.enabled` to be `true` (via `ase config set telemetry.enabled true`) and the telemetry export of the agent tool to be activated once
via `ase setup telemetry activate --tool` *tool*, otherwise only the
assignee is set and a `NOTICE` is reported. If *id*`:` is omitted, the *current* task id
(inherited from the session context) is used, otherwise, on a successful
assignment to *oneself*, the current task id of the session is switched
to *id*.

## ARGUMENTS

- *id*- `:`:
The unique identifier of the task plan, with a trailing colon.
If omitted, the current task id is used.
 
- *assignee*:
The assignee:- `me`for oneself,- `none`for nobody, or any other
name of a human or agent. If the configuration key- `project.users`lists the known users, a name which is neither one of them nor the
name of an agent present at the agent relay is still assigned, but
reported with a- `NOTICE`.
 

## SCENARIOS

- You start working on a task and want its AI usage tracked
- You hand a task over to a colleague or an agent, booking your usage
- You want to know the AI time, tokens, and costs spent on a task

## EXAMPLES

Take the current task plan and track the AI usage of the session:

```
❯ /ase-task-assign me
```

Take a specific task plan:

```
❯ /ase-task-assign hello: me
```

Give the task plan back to a colleague, booking the AI usage:

```
❯ /ase-task-assign alice
```

Unassign the task plan, booking the AI usage:

```
❯ /ase-task-assign hello: none
```

## SEE ALSO

`ase-task-status`, `ase-task-id`, `ase-task-view`,
`ase-task-list`.

ase-task-edit - Iteratively Edit a Task Plan

## NAME

`ase-task-edit` - Iteratively Edit a Task Plan

## SYNOPSIS

`ase-task-edit`
    [`--help`|`-h`]
    [`--plan`|`-p` *option*]
    [`--mode`|`-m` `auto`|`craft`|`refactor`|`resolve`|`specify`]
    [`--approach`|`-a`]
    [`--unverified`|`-u`]
    [`--focus`|`-f` *section*[,...]]
    [`--quiet`|`-q`]
    [`--interactive`|`-i`]
    [`--next`|`-n` *option*[,...]]
    [*id* | *id*: *instruction* | *instruction* | *issue-id*]

## DESCRIPTION

The `ase-task-edit` skill establishes and refines a *task plan* purely
through a *chat-driven loop*. The user steers each round via an
interactive dialog that offers continued refinement, finalization, or
hand-off to implementation or draft. It covers the planning of all
kinds of change: crafting a new feature, refactoring the code base,
resolving a problem, and specifying.

A newly generated plan is grounded in the relevant specification
(`SPEC`), source code (`CODE`), and infrastructure (`INFR`) files of the
project: it names the actual files and symbols to change, the actual
build, lint, and test procedures, and a `CHANGELOG.md` entry if such a
file exists.

Before a new plan is generated, the skill determines its *kind of
change* (stored in the `Kind:` key of the plan) and internalizes the
corresponding tenets (for example KISS, YAGNI, and DRY for crafting,
or Surgical Changes and Origin Proximity for resolving). For a
resolution, it first reports the `PROBLEM CONTEXT` and `PROBLEM DETAILS`. Under `--approach`, it then proposes one or more *approaches*
with pros and cons, recommends one of them, and lets the user pick the
preferred one via an interactive dialog, and the new plan is then
composed along the chosen approach.

If the current task is `default` and a new plan is generated from an
*instruction*, a new task id is allocated from the instruction
according to the task id scheme of the project, so the plan does not
end up under `default`.

Attachments in the plan's backmatter are never removed by a refinement.
A refinement which changes an attachment refreshes the attachment's
`Modified:` key; a refinement which changes the plan body refreshes the
`Modified:` key of the frontmatter (which tracks body changes only, so
a refinement of frontmatter keys alone leaves it untouched); both use
the same value. An implementation draft (an attachment block of type
`text/x-diff; charset=utf-8; kind="draft"`, produced by
`ase-task-implement --draft`) whose `Modified:` key is absent or older
than the frontmatter's is *stale*: it is reported with a warning after
each rendering of the plan, together with a hint that
`ase-task-implement --draft` has to be run again to re-create the
draft for the changed plan, as `ase-task-integrate --draft` refuses a
stale draft.

## OPTIONS

- `--plan`|- `-p`- *option*:
Automatically answer the user dialog for the plan refinement
with- *option*, which can be either- `none`(default, interactive
answer required),- `OVERWRITE`(overwrite an existing plan
with- *instruction*),- `REFINE`(refine the existing plan with- *instruction*), or- `PRESERVE`(preserve the existing plan by
ignoring- *instruction*and stopping skill processing).
 
- `--mode`|- `-m`- `auto`|- `craft`|- `refactor`|- `resolve`|- `specify`:
Select the kind of change of the plan:- `craft`(- `CRAFTING`),- `refactor`(- `REFACTORING`),- `resolve`(- `RESOLVING`), or- `specify`(- `SPECIFYING`). It selects the tenets internalized when the plan is
generated or refined and sets the- `Kind:`key of the plan, also on
a refinement. The default- `auto`infers the kind of a new plan from
the- *instruction*, uses- `RESOLVING`for an- *issue-id*(also on a
refinement), and otherwise leaves the- `Kind:`key of an existing
plan as it is and honors its tenets.
 
- `--approach`|- `-a`:
Before a new plan is composed, propose one or more crafting,
refactoring, resolution, or specification approaches with pros and
cons, recommend one of them, and let the user pick the preferred one
via an interactive dialog (or, under the- `black`project boxing or
in headless mode, automatically pick the recommended one). The new
plan is then composed along the chosen approach. Without- `--approach`, no approaches are determined at all.
 
- `--unverified`|- `-u`:
Generate any- *new*plan- *without*the- `##  VERIFICATION (WHEN)`section.
Applies only to freshly generated plans, not to existing plans
loaded from disk. When- `ase-task-implement`later applies such
a plan, it strictly skips the entire verification phase (no
build, tests, linter, type-checker, or program execution) once
the source files have been modified.
 
- `--focus`|- `-f`- *section*[,...]:
Edit and show only the given plan- *section*(s). Each- *section*is
one of- `SPECIFICATION`(also- `SPEC`or- `WHAT`),- `DESIGN`(also- `DES`or- `HOW`), or- `VERIFICATION`(also- `VERIFY`,- `VER`, or- `WHEN`),
matched case-sensitively, or the sentinel- `all`(default). The shown
plan consists only of these sections, including their headings, but
without the frontmatter and the title.
 - The rest of the plan stays the context, and an - *instruction*requiring a
change outside of the focus is applied nevertheless and reported by
a warning. A new plan gets all section headings, but only the focused
sections are populated, and- `OVERWRITE`re-generates only them. An
unknown, empty, or duplicate- *section*token aborts the skill. It is
forwarded on a hand-off to- `ase-task-grill`, but not on a hand-off
to another skill.
 
- `--quiet`|- `-q`:
Do not print the resulting plan at the end of a planning round. The
status lines, the dialogs, and the warning on a stale implementation
draft stay. It is forwarded on a hand-off to- `ase-task-grill`, but
not on a hand-off to another skill.
 
- `--interactive`|- `-i`:
Show the plan in a band above the prompt (- *Anthropic Claude Code CLI*with
active ASE function hooks in an interactive terminal only, else the skill
aborts), instead of printing it, and ask for the next step without any
dialog. The band takes 75% of the terminal height, in the fullscreen mode
all the rows the terminal UI leaves it. Under- `--focus`, it shows only the
focused sections.
 - The band follows every change of the plan, also by others, and keeps its
scroll position. Its line numbers are those of the plan text in - `ase task edit`. It shows the last change as a unified diff with colored backgrounds and
line numbers, red for removed and blue for added lines. In the fullscreen
mode, scroll it with the mouse wheel, or with- *CTRL+x*- *TAB*and then- *PageUp*/- *PageDown*.
 - At the prompt, type either a refinement instruction, or one of - `DONE`,- `GRILL`,- `DRAFT`,- `IMPLEMENT`, or- `CANCEL`(in any case), which closes the
band. After a hand-off to- `ase-task-grill`, a hand-off back reopens it.
 - Type - `##`at the end of the prompt (which removes it again) to remove the
unified diff and show just the latest state of the plan, until its next
change. The band stays open and the input does not reach the agent.
 
- `--next`|- `-n`- *option*[,...]:
Automatically answer the user dialog for the next step (at the end
of this skill).- *option*is a single token or a- *comma-separated
chronological list*of tokens; each iteration of the planning
loop consumes the- *first*token of the list, and on hand-off
(- `IMPLEMENT`/- `DRAFT`) any remaining tokens are- *forwarded*(via- `--next`) to the downstream skill so an entire pipeline can
be pre-scripted in one shot. Recognized tokens at this skill:- `none`(default, interactive answer required),- `DONE`(no next
step),- `GRILL`(hand-over to- `ase-task-grill`),- `DRAFT`(hand-over to- `ase-task-implement --draft`), or- `IMPLEMENT`(hand-over to- `ase-task-implement`). Example:- `--next GRILL,DONE`hands the plan
off to grilling and forwards- `DONE`so grilling exits without asking.
 

## ARGUMENTS

- *id*|- *id*:- *instruction*|- *instruction*|- *issue-id*:
Edit the task with the unique identifier- *id*(default: the current
task). Optionally,- *instruction*either gives instructions for
creating a new task or gives instructions for refining an existing
task. Instead, a bare- *issue-id*(like- `P1`,- `T1`, or- `<prefix>-P1`), previously reported by- `ase-code-analyze`or- `ase-arch-analyze`, uses the description of this issue as the- *instruction*(also as- *id*:- *issue-id*), and the issue is removed
once the plan is saved. If no such issue exists,- *issue-id*is
treated as a task- *id*resp. as an- *instruction*.

## SCENARIOS

- You want a task planned through chat-driven refinement
- You want a plan created or refined round by round
- You want approaches with pros and cons before a plan is composed
- You want a plan to craft a feature, refactor code, or resolve a problem
- You want an analyzer issue like `P1`turned into a plan
- You want plan refinement with hand-off to draft or implementation

## EXAMPLES

Edit the current task:

```
❯ /ase-task-edit
```

Create a new task under id `hello`:

```
❯ /ase-task-delete hello
❯ /ase-task-edit hello: new "ase hello" CLI command which prints
  a nice "Hello World!" to the terminal in color blue.
```

Further refine the task under id `hello`:

```
❯ /ase-task-edit hello: change the color to red.
```

Plan the resolution of the analyzer issue `P1` along an interactively
chosen approach, and directly hand off to implementation:

```
❯ /ase-task-edit --approach --next IMPLEMENT P1
```

Plan a refactoring under the id `cleanup`:

```
❯ /ase-task-edit -m refactor cleanup: extract HTTP client into its own class
```

Refine and show only the `DESIGN` section of the task under id `hello`:

```
❯ /ase-task-edit --focus HOW hello: print the text via a dedicated logger.
```

Refine the task under id `hello` with its plan shown in a band above the prompt:

```
❯ /ase-task-edit --interactive hello
```

## SEE ALSO

`ase-code-edit`, `ase-spec-edit`, `ase-task-grill`,
`ase-task-reboot`, `ase-task-implement`,
`ase-task-view`, `ase-task-list`, `ase-task-rename`, `ase-task-delete`.

ase-task-view - View a Task Plan

## NAME

`ase-task-view` - View a Task Plan

## SYNOPSIS

`ase-task-view`
    [`--help`|`-h`]
    [`--full`|`-f`]
    [*id*]

## DESCRIPTION

The `ase-task-view` skill renders the *task plan* identified by *id*.
The plan is loaded via the `ase_task_load` MCP tool and shown framed
between a `( TASK )` header and footer rule. If *id* is omitted, the
*current* task id (inherited from the session context) is used.

By default, the payload of every attachment in the backmatter (like an
implementation draft of `ase-task-implement --draft` or the processing
log of an `ase agent`) is elided to an `[elided: <n> lines]` marker to
keep the view compact. The `--full`|`-f` option loads the payloads, too,
and renders the plan in full, without any truncation or summarization.

An implementation draft attachment whose `Modified:` key is absent or
older than the `Modified:` key of the plan frontmatter is *stale*, as
the plan changed after the draft was created. Such a draft is reported
with a warning after the rendering, together with a hint that
`ase-task-implement --draft` has to be run again, as
`ase-task-integrate --draft` refuses a stale draft.

The frontmatter keys `Source:`, `Changeset:`, and `Target:` are
rendered as well, stating where the change set of the plan is
created from, stored in, and delivered to.

## OPTIONS

- `--full`|- `-f`:
Render the plan in full, including the payloads of its attachments.
By default, each payload is elided to an- `[elided: <n> lines]`marker.

## ARGUMENTS

- *id*:
The unique identifier of the task plan to view. If omitted,
the current task id is used.

## SCENARIOS

- You want the current or a given task plan shown
- You want a plan rendered before deciding the next steps
- You want a plan inspected including its implementation draft

## EXAMPLES

View the current task plan:

```
❯ /ase-task-view
```

View a specific task plan:

```
❯ /ase-task-view hello
```

View a plan in full, including its implementation draft attachment:

```
❯ /ase-task-view --full hello
```

## SEE ALSO

`ase-task-list`, `ase-task-edit`, `ase-task-id`,
`ase-task-rename`, `ase-task-delete`.

ase-task-grill - Iteratively Grill a Task Plan

## NAME

`ase-task-grill` - Iteratively Grill a Task Plan

## SYNOPSIS

`ase-task-grill`
    [`--help`|`-h`]
    [`--rounds`|`-r` *n*]
    [`--until`|`-u` `MUST`|`SHOULD`|`MAY`]
    [`--perspective`|`-p` *perspective*]
    [`--ground`|`-G`]
    [`--focus`|`-f` *section*[,...]]
    [`--force`|`-F`]
    [`--quiet`|`-q`]
    [`--next`|`-n` *option*[,...]]
    [*id*]

## DESCRIPTION

The `ase-task-grill` skill *relentlessly interviews* the user about
every *essential aspect* of an existing *task plan* until a *shared
understanding* is reached and no decisions or questions are left open.

The skill identifies the essential aspects of the plan and raises up to
10 questions per round which resolve its open points. Each question
carries a `FOCUS-AREA` -- `DOMAIN` (domain-specifics, must be
clarified), `INTERFACE` (externally observable behavior or UI/API
interfaces, must be clarified), `ARCHITECTURE` (structure, wiring,
placement, or dependencies, should be clarified), `IMPLEMENTATION`
(inner technical details, can be clarified), `REGRESSION` (what must
not break, should be clarified), or `CONFIRMATION` (what proves the
specified behavior, should be clarified) -- and a 1-3 word `TOPIC`
hint. The focus areas are selected by the plan sections under focus
(`--focus`): `SPECIFICATION` selects `DOMAIN` and `INTERFACE`, `DESIGN`
selects `ARCHITECTURE` and `IMPLEMENTATION`, and `VERIFICATION` selects
`REGRESSION` and `CONFIRMATION`. The questions are sorted
by descending focus area importance, then by descending individual
impact, and finally by the decision tree of their dependencies, so each
decision is asked after the decisions it depends on. It honors checks
for *fuzzy language*, *conflicting terminology*, *conflicting code*,
*non-concrete scenarios*, *unspecified architecture patterns*, and
*unspecified dependencies*.

As with `ase-code-edit --grill`, all questions of a round are announced
together below a `GRILLING ROUND K/L` line (plain `GRILLING` for a
single round) as a `QUESTION`/`ANSWERS` table with one row per
question, each row carrying two to three *grounded* answer alternatives
(the one reflecting the current plan marked with `⚑`). They are then
asked in *one* batch via a single interactive dialog, whose only answer
option is the fixed `SKIP GRILLING` (skip the remaining grilling, but
still update the plan with the answers of the previous rounds), plus
free-text input. The questions are numbered `1`, `2`, etc. and their
answer alternatives are lettered `A`, `B`, etc., so the free-text input
can cherry-pick answers with short responses matching `\d+[a-zA-Z]`
(like `1A 2c`), freely mixed with keyword text. A question not
addressed by the reply accepts its `⚑` answer, if one exists.
Cancelling the dialog stops the skill without updating the plan with
the current round. After every round, the plan is updated and
persisted, its `Tags:` frontmatter key records each grilled section as
its own tag `grilled:`*section* (lower-case, kept alongside the tags of
previously grilled sections, e.g. `grilled:specification, grilled:design`), and a `PLAN CHANGES` box shows the bullet-points
changed by the round (unless `--quiet` is given). Finally, the user is offered a complete
re-grilling (with the same grilling options), or a hand-off to editing,
implementation, or draft.

The *open* questions are recorded in the checkboxes of the plan's
bullet-points: a bullet-point whose question stayed *unanswered*
(because the grilling was skipped, or the reply did not address a
question the plan offers no answer for) is marked `[?]`, an unanswered
question the plan does not cover yet is added as a new `[?]`
bullet-point, and an *answered* question resets its bullet-points to
`[ ]`. When a section is grilled *again* (its `grilled:`*section* tag
is already present), only its `[?]` bullet-points are re-asked, and a
section without any `[?]` bullet-point is skipped entirely, unless
`--force` is given. Bullet-points
in state `[-]` (cancelled) or `[>]` (deferred) are never questioned.

## OPTIONS

- `--rounds`|- `-r`- *n*:
The- *maximum*number of grill rounds to apply (default:- `1`). Each
round starts from scratch from only the current plan, as updated and
saved by all previous rounds, and re-derives its questions from it,
forgetting all questions and answers of previous rounds. With more than one
round, each round is announced as- `GRILLING ROUND K/L`. The grilling
stops early -- announced by a- `grilling finished early`status line
-- once a round, even the first one, finds the open points clear
enough according to- `--until`.
 
- `--until`|- `-u`- `MUST`|- `SHOULD`|- `MAY`:
Grill until at least all open points of the given severity or
higher are clear (default:- `MUST`), where the severity follows from
the focus area:- `MUST`(- `DOMAIN`,- `INTERFACE`),- `SHOULD`(- `ARCHITECTURE`,- `REGRESSION`,- `CONFIRMATION`), and- `MAY`(- `IMPLEMENTATION`). Independent of this, every open point is rated
with an individual impact (- `HIGH`,- `MEDIUM`, or- `LOW`), which
decides which points are raised as questions and in which order
within a focus area.
 
- `--perspective`|- `-p`- *perspective*:
Grill exclusively from the logical- *perspective*-- a keyword or
short hint like- `domain`,- `performance`, or- `security`. The
perspective acts as an exclusive lens: only open points relevant from
this perspective are raised as questions, while every question still
carries its focus area, severity, and impact. The active perspective
is shown in the announcement line, like- `GRILLING (security)`.
Without a perspective, the grilling is holistic.
 
- `--ground`|- `-G`:
Ground the grilling in facts instead of the world knowledge of the
model alone: once before the first round, the artifacts of all kinds
(- `SPEC`,- `CODE`,- `DOCS`,- `INFR`, and- `OTHR`) are scanned for
the project facts relevant to the grilled subject (from the given
perspective, if any) and the Internet/Web is searched for the
established practices, standards, patterns, and pitfalls about it.
These facts serve all rounds, both for raising the questions and for
preferably deriving their answer alternatives. The grounding is shown
in the announcement line, like- `GRILLING (security, grounded)`. If
no usable facts are found, a warning is shown and the grilling falls
back to model knowledge.
 
- `--focus`|- `-f`- *section*[,...]:
Grill only the given plan- *section*(s), in the given order. Each- *section*is one of- `SPECIFICATION`(also- `SPEC`or- `WHAT`),- `DESIGN`(also- `DES`or- `HOW`), or- `VERIFICATION`(also- `VERIFY`,- `VER`, or- `WHEN`), matched case-sensitively, or the sentinel- `all`(default),
which expands to- `SPECIFICATION,DESIGN,VERIFICATION`. The questions
are derived from the themes of the focused sections only, while the
entire plan stays the context, and an answer requiring a change
outside of the focus is applied nevertheless and reported by a
warning. An unknown, empty, or duplicate- *section*token aborts the
skill. It is forwarded on a re-grilling and on a hand-off to- `ase-task-edit`, but not on a hand-off to another skill.
 
- `--force`|- `-F`:
Re-grill already grilled sections completely, i.e., ignore their- `grilled:`- *section*tags instead of skipping them resp. re-asking
only their- `[?]`bullet-points.
 
- `--quiet`|- `-q`:
Do not print the- `PLAN CHANGES`box of the bullet-points changed by
a round. The status lines and the dialogs stay. It is forwarded on a
re-grilling and on a hand-off to- `ase-task-edit`, but not on a
hand-off to another skill.
 
- `--next`|- `-n`- *option*[,...]:
Automatically answer the user dialog for the next step (at the end
of this skill).- *option*is a single token or a- *comma-separated
chronological list*of tokens; the- *first*token is consumed by
this skill and any remaining tokens are- *forwarded*(via- `--next`)
to the downstream skill on hand-off so an entire pipeline can be
pre-scripted in one shot. Recognized tokens at this skill:- `none`(default, interactive answer required),- `DONE`(no next step),- `GRILL`(re-grilling via- `ase-task-grill --force`with the same
grilling options),- `EDIT`(hand-over to- `ase-task-edit`),- `IMPLEMENT`(hand-over to- `ase-task-implement`), or- `DRAFT`(hand-over to- `ase-task-implement --draft`).
 

## ARGUMENTS

- *id*:
Grill the task with the unique identifier- *id*(default: the current task).
The skill accepts- *only*an optional- *id*argument and never a
free-text instruction.

## SCENARIOS

- You want to be interviewed about your plan until it is watertight
- You want the open decisions of a plan resolved question by question
- You want fuzzy language and conflicts flushed out of a plan
- You want shared understanding before the implementation starts

## EXAMPLES

Grill the current task plan:

```
❯ /ase-task-grill
```

Grill the task plan under id `hello`:

```
❯ /ase-task-grill hello
```

Grill the current task plan in two rounds:

```
❯ /ase-task-grill --rounds 2
```

Grill the current task plan exclusively from the performance
perspective, grounded in the project artifacts and the Internet/Web:

```
❯ /ase-task-grill --perspective performance --ground
```

Grill only the `DESIGN` and then the `SPECIFICATION` section of the
current task plan:

```
❯ /ase-task-grill --focus DES,SPEC
```

Re-grill the already grilled current task plan completely:

```
❯ /ase-task-grill --force
```

Grill the current task plan and then hand off to editing:

```
❯ /ase-task-grill --next EDIT
```

## SEE ALSO

`ase-task-edit`, `ase-task-reboot`,
`ase-task-implement`, `ase-task-view`, `ase-task-list`,
`ase-task-rename`, `ase-task-delete`, `ase-code-edit`.

ase-task-research - Research a Topic for a Task Plan

## NAME

`ase-task-research` - Research a Topic for a Task Plan

## SYNOPSIS

`ase-task-research`
    [`--help`|`-h`]
    [`--services`|`-s` `all`|`perplexity`|`brave`|`exa`|`websearch`[,...]]
    [`--refine`|`-r`]
    [`--dry-run`|`-d`]
    [*id* | *id*`:` [*topic*] | *topic*]

## DESCRIPTION

The `ase-task-research` skill *researches* a topic in depth for a task
plan. It combines a *Web research* -- several focused queries, sent in
parallel to the Internet/Web search services of `ase-meta-search` --
with a *grounding* in the *source artifacts* of the project (its
specification, source code, documentation, and infrastructure).

The result is a *research report* with the parts *Findings* (the Web
facts, each with its source URL), *Project Grounding* (the relevant
project facts, each with its project file), *Assessment* (how the
findings apply to the project, including its gaps and conflicts), and
*Recommendations* (the concrete consequences for the plan), bounded to
about 300 lines and at most 10 findings.

The report is *appended* to the single `research` attachment of the
task (of type `text/markdown; charset=utf-8; kind="research"`), so the
reports of earlier researches are kept. With `--refine`, the
recommendations are additionally applied to the plan itself, preserving
its structure. Without a *topic*, the task plan itself (its title and
domain items) is researched.

The search queries carry only a self-contained summary of the topic and
never any project internals like file paths or identifiers, as they are
sent to foreign search services. The shown report follows the
configured `project.boxing`, while the attachment always carries the
full report.

The `ase agent researcher` command researches the tasks automatically
and always refines the plan by the recommendations: in its mode
`phase`, it researches and grounds the plans of the `DRAFTED` tasks
explicitly assigned to it, while in its mode `request`, it researches
the topics of the requests in any unfinished status, via `ase task request <id> research [<topic>]` or key `R` of the task boards.

## OPTIONS

- `--services`|- `-s`- `all`|- `perplexity`|- `brave`|- `exa`|- `websearch`[,...]:
The comma-separated search services to query (default:- `all`).
 
- `--refine`|- `-r`:
Additionally refine the task plan by the recommendations of the
research report.
 
- `--dry-run`|- `-d`:
Research and show the report only, but do not change the task.
 

## ARGUMENTS

- *id*:
The unique identifier of the task to research for. If omitted, the- *current*task is used. A single word without a trailing- `:`is
always taken as a task identifier.
 
- *topic*:
The topic to research. If omitted, the task plan itself is
researched.
 

## SCENARIOS

- You want to know the established practices and pitfalls of a topic before planning a task
- You want to check a task plan against the state of the art
- You want to keep the research findings of a task together with its plan

## EXAMPLES

Research the current task plan itself:

```
❯ /ase-task-research
```

Research a topic for a specific task and refine its plan:

```
❯ /ase-task-research --refine export-json: streaming JSON serialization of large data sets
```

Research a topic with Perplexity only, without changing the task:

```
❯ /ase-task-research --services perplexity --dry-run rate limiting of REST APIs
```

## SEE ALSO

`ase-meta-search`, `ase-task-edit`, `ase-task-grill`,
`ase-task-view`.

ase-task-estimate - Estimate the Effort of a Task Plan

## NAME

`ase-task-estimate` - Estimate the Effort of a Task Plan

## SYNOPSIS

`ase-task-estimate`
    [`--help`|`-h`]
    [`--force`|`-f`]
    [`--dry-run`|`-d`]
    [`--all`|`-a`]
    [*id* ...]

`ase-task-estimate`
    `--calibrate`|`-c`
    [`--save`|`-s`]

## DESCRIPTION

The `ase-task-estimate` skill *estimates* the effort of task plans and
stores it in their `Effort:` frontmatter key. The effort is the expected
*human time* of the *entire* lifecycle of a task (including its
planning), matching the human time `time.hr` booked on the usage
attachment of the task (see `ase-task-assign`), expressed as one of the
T-shirt sizes `XS`, `S`, `M`, `L`, or `XL`. The sizes are bands on a
*logarithmic* scale with the upper bounds `15`, `60`, `240`, and `960`
human minutes for `XS` to `L` (configuration `project.task.effort.scale`),
while `XL` is open-ended. A task without an `Effort:` key is *not
estimated*.

The estimation of a plan combines:

- an - *attention score*: the weighted number of the distinct
requirements, behaviors, or changes stated by its checklist items
(- `DOM`5.0,- `IFC`4.0,- `ARC`3.0,- `IMP`1.0,- `REG`0.5,- `CON`1.0),
converted via- `project.task.effort.factor`(default:- `1`) human
minutes per point,
 
- a - *volume score*: the code impact of its- `ARC`and- `IMP`items,
predicted read-only from the plan and the code base (never measured
from an already existing change set), converted via- `project.task.effort.file`(default:- `1`) human review minutes per
touched file and- `project.task.effort.line`(default:- `0.025`)
human minutes per predicted changed line, which also cover the edge
cases and polishing no plan describes, where a newly written line
weighs three times a changed one,
 
- a - *risk surcharge*: 20% more per applying risk out of- `dependency`,- `ambiguity`,- `untested`,- `concurrency`,- `security`, and- `persistence`, but at most 40% in total, where risks applying to
virtually every task of the project do not count,
 
- an - *iteration multiplier*:- `project.task.effort.iteration`(default:- `2.5`) times the minutes for a task whose main
deliverable is an interactive or visual user interface, as such
work needs many rounds of post-adjustment,
 
- a - *plausibility check*: a comparable realization already existing
in the code base, whose deviating size lowers the reported- *confidence*, and
 
- *analogues*: finished tasks with an effort and a usage attachment,
whose disagreement lowers the reported- *confidence*.
 

Cancelled (`[-]`) and deferred (`[>]`) items never count. The skill
reports the size, the confidence, and the main drivers of each
estimation, and for `XL` additionally warns and suggests splitting the
task via `ase-task-dissect` -- an `XL` effort never blocks the
lifecycle.

The effort is *bound* to the plan content it was given for: once the
plan content changes, the effort becomes *invalid* and is dropped, while
mere checkbox state changes and whitespace-only edits keep it. Hence,
an estimation never needs to be cleaned up by hand. Writing the effort
neither changes the `Modified:` key nor the lifecycle status of a task.
Empty plans and explicit epic tasks (whose effort is the sum of their
member tasks) are skipped, and so are tasks already carrying a valid
effort, unless `--force` is given.

With `--calibrate`, the skill instead re-estimates the successfully
finished tasks with the *current* rates and compares the results with
their *actual* human time (only tasks with at least `5` human minutes
count, as agent-driven tasks book little human time), so a repeated
calibration converges, and reports the estimated vs. actual band matrix,
the hit rate, the bias, the best-fitting rates (factor, file, and line
minutes), and alternatively fitting scale bounds.

The `ase agent estimator` command estimates the tasks automatically in
the background, and with `--mode request` on request, in any unfinished
status, via `ase task request <id> estimate <prompt>` or key `R` of the
task boards.

## OPTIONS

- `--force`|- `-f`:
Re-estimate tasks which already carry a valid effort.
 
- `--dry-run`|- `-d`:
Estimate and report only, but do not store the effort.
 
- `--all`|- `-a`:
Estimate all unfinished tasks without a valid effort (with- `--force`,
all unfinished tasks), instead of the current or given ones.
 
- `--calibrate`|- `-c`:
Report the calibration of the estimation against the finished tasks
instead of estimating any task.
 
- `--save`|- `-s`:
Together with- `--calibrate`, store the fitted rates as- `project.task.effort.factor`,- `project.task.effort.file`, and- `project.task.effort.line`on the- `project`scope. The fitted scale
bounds always stay a suggestion only.
 

## ARGUMENTS

- *id*...:
The unique identifiers of the tasks to estimate. If omitted (and
without- `--all`), the- *current*task is estimated.

## SCENARIOS

- You want to know how much human time a planned task will need
- You want to size all open tasks for planning a sprint or release
- You want to check whether a task is too large and should be split
- You want to tune the estimation to the actual pace of your team

## EXAMPLES

Estimate the current task plan:

```
❯ /ase-task-estimate
```

Estimate all unestimated tasks, without storing the results:

```
❯ /ase-task-estimate --all --dry-run
```

Re-estimate two specific tasks:

```
❯ /ase-task-estimate --force export-json import-csv
```

Calibrate the estimation and store the fitted rates:

```
❯ /ase-task-estimate --calibrate --save
```

## SEE ALSO

`ase-task-view`, `ase-task-list`, `ase-task-dissect`,
`ase-task-assign`, `ase-meta-config`.

ase-task-reboot - Reboot a Task Plan

## NAME

`ase-task-reboot` - Reboot a Task Plan

## SYNOPSIS

`ase-task-reboot`
    [`--help`|`-h`]
    [`--next`|`-n` *option*[,...]]
    [*id*]

## DESCRIPTION

The `ase-task-reboot` skill re-creates an existing task plan *from
scratch* by extracting the `DOM` and `IFC` bullet points of the
`SPECIFICATION (WHAT)` section (if present) from the current plan,
using them as the new instruction, re-deriving the `DESIGN (HOW)` and
`VERIFICATION (WHEN)` sections grounded in the relevant specification
(`SPEC`), source code (`CODE`), and infrastructure (`INFR`) files of the
project (naming the actual files and symbols to change, the actual build,
lint, and test procedures, and a `CHANGELOG.md` entry if such a file
exists), preserving the original creation
timestamp, the remaining frontmatter keys, and all attachments, and
writing fresh plan content via `ase_task_save`. As the rebooted plan
starts its lifecycle anew, its `Status:` key is reset to the default
state of the task lifecycle model, all `grilled:` tags are dropped
from its `Tags:` key, and its `Source:`, `Changeset:`, and `Target:`
keys are dropped, so they read as their configured defaults
`project.task.default.{source,changeset,target}`.

A plan which does *not* follow the task format (or carries no `DOM` or
`IFC` bullet point) is rebooted from its entire body instead: all of
its existing content is *filed* into the `SPECIFICATION (WHAT)`,
`DESIGN (HOW)`, and `VERIFICATION (WHEN)` sections by classifying
each statement as a `DOM`, `IFC`, `ARC`, `IMP`, `REG`, or `CON` bullet
point, with no statement dropped and missing bullet points derived from
the existing content.

After the reboot, the user is asked whether to stop or hand off to
`ase-task-edit` or `ase-task-implement` (optionally with `--draft`),
unless `--next` pre-selects this choice.

## OPTIONS

- `--next`|- `-n`- *option*[,...]:
Automatically answer the user dialog for the next step.- *option*is a single token or a- *comma-separated chronological list*of
tokens; the- *first*token is consumed by this skill, and any
remaining tokens are- *forwarded*(via- `--next`) to the downstream
skill so an entire pipeline can be pre-scripted in one shot.
Recognized tokens at this skill:- `none`(default, interactive
answer required),- `DONE`(stop),- `EDIT`(hand off to- `ase-task-edit`),- `IMPLEMENT`(hand off to- `ase-task-implement`),
or- `DRAFT`(hand off to- `ase-task-implement --draft`). Example:- `--next EDIT,DONE`reboots, hands off to editing, and the editing
loop will exit immediately.

## ARGUMENTS

- *id*:
The unique identifier of the task whose plan should be rebooted.
If omitted, the- *current*task id is used.

## SCENARIOS

- You want a task plan re-created from scratch
- You want a fresh plan from the original specification
- You want planning restarted after a plan degraded
- You want a free-form or legacy plan filed into the task format

## EXAMPLES

Reboot the current task plan:

```
❯ /ase-task-reboot
```

Reboot a specific task and hand off to editing:

```
❯ /ase-task-reboot --next EDIT hello
```

## SEE ALSO

`ase-task-edit`, `ase-task-implement`,
`ase-task-view`, `ase-task-delete`.

ase-task-condense - Condense a Task Plan

## NAME

`ase-task-condense` - Condense a Task Plan

## SYNOPSIS

`ase-task-condense`
    [`--help`|`-h`]
    [`--next`|`-n` *option*[,...]]
    [*id*]

## DESCRIPTION

The `ase-task-condense` skill *compresses* the wording of an existing
task plan to make it require as little reading as possible, while
keeping *all semantics fully preserved and unchanged*. It loads the
current (or given) plan, applies a self-contained, telegrapher-like
"remove-fluff" ruleset to the *free-text* parts only (the text of each
`DOM`, `IFC`, `ARC`, `IMP`, `REG`, and `CON` bullet point), and writes
the shorter plan back via `ase_task_save`.

The plan *structure* is never altered: the frontmatter, all headings,
the `SPECIFICATION (WHAT)`, `DESIGN (HOW)`, and `VERIFICATION (WHEN)`
sections, the `- [ ] DOM:`-style bullet prefixes with their checkbox
states, all attachments, code spans, technical terms, file paths,
numbers, and severities are kept exactly. Only genuinely
redundant bullets may be merged, and a shortening that would change
meaning is always rejected in favor of the longer wording. The condense
ruleset *overrides* the active session persona, so the plan is compressed
telegrapher-like even under the `writer` persona.

The plan is saved *only* when condensing actually makes it smaller; if no
further reduction is possible, the plan is left untouched (including its
`Modified:` frontmatter timestamp) and reported as *already condensed*.

After condensing, the user is asked whether to stop or hand off to
`ase-task-edit` or `ase-task-implement` (optionally with `--draft`),
unless `--next` pre-selects this choice.

## OPTIONS

- `--next`|- `-n`- *option*[,...]:
Automatically answer the user dialog for the next step.- *option*is a single token or a- *comma-separated chronological list*of
tokens; the- *first*token is consumed by this skill, and any
remaining tokens are- *forwarded*(via- `--next`) to the downstream
skill so an entire pipeline can be pre-scripted in one shot.
Recognized tokens at this skill:- `none`(default, interactive
answer required),- `DONE`(stop),- `EDIT`(hand off to- `ase-task-edit`),- `IMPLEMENT`(hand off to- `ase-task-implement`),
or- `DRAFT`(hand off to- `ase-task-implement --draft`). Example:- `--next EDIT,DONE`condenses, hands off to editing, and the editing
loop will exit immediately.

## ARGUMENTS

- *id*:
The unique identifier of the task whose plan should be condensed.
If omitted, the- *current*task id is used.

## SCENARIOS

- You want a task plan shortened without losing meaning
- You want plan wording compressed for faster reading
- You want fluff removed from a plan while keeping all semantics

## EXAMPLES

Condense the current task plan:

```
❯ /ase-task-condense
```

Condense a specific task and hand off to editing:

```
❯ /ase-task-condense --next EDIT hello
```

## SEE ALSO

`ase-task-edit`, `ase-task-reboot`, `ase-task-view`,
`ase-task-rename`, `ase-task-delete`.

ase-task-dissect - Dissect a Task Plan

## NAME

`ase-task-dissect` - Dissect a Task Plan

## SYNOPSIS

`ase-task-dissect`
    [`--help`|`-h`]
    [`--max-parts`|`-m` *count*]
    [`--dry`|`-d`]
    [`--force`|`-f`]
    [*task-id*[`:`]]
    [*dissect-hint*]

## DESCRIPTION

The `ase-task-dissect` skill treats an existing task plan as an *epic*
and *dissects* it domain-wise and logically into *cohesive parts*. Each
part is a self-contained unit of work with a single coherent purpose, so
it can be implemented, reviewed, and committed entirely on its own.

The dissection follows a strict ruleset: at least *2* and at most
*count* parts are derived, *every* bullet point of the epic's
`##  SPECIFICATION (WHAT)`, `##  DESIGN (HOW)`, and
`##  VERIFICATION (WHEN)` sections is assigned to *exactly one* part,
mutually dependent bullet points are kept together, and
nothing beyond the epic's own content is invented. A bullet point which
itself covers *multiple* domains is *split* into fragments, which are
then assigned individually -- together they still reproduce the original
bullet point completely and without duplication. The cut follows the
*semantics*, not the technical structure -- a split by file, directory,
or technical layer is usually *not* the intended one. If the epic
carries only a single cohesive purpose, it is reported as *not
dissectable* and nothing is created.

An optional *dissect-hint* steers *how* the epic is split -- which
bullet points belong together, along which axis to cut, or how many
parts to aim at. It is especially useful after a `--dry` run showed an
unintended dissection. The hint overrides the default grouping, but
never the ruleset itself.

The derived parts are first reported as a *dissection table* (part
number, sub-task id, scope, and rationale). Then -- unless `--dry` is
given -- each part is materialized as its own *complete* sub-task plan
(with its own `##  SPECIFICATION (WHAT)`, `##  DESIGN (HOW)`, and
`##  VERIFICATION (WHEN)` sections) and persisted under the task id
`<task-id>-<feature-slug>`.

The parts are placed via their `Group:` and `After:` frontmatter keys.
If the task already is a *member* of an epic (its `Group:` names another
task), the parts join this very epic and *replace* the task: every other
task listing it in its `After:` key is re-wired to the final parts, and
the task itself is deleted. Otherwise the task *becomes* the explicit
epic of its parts: its `Group:` key is set to its own id, so it comes
after all of its parts. In both cases, each part lists the parts it
depends on in its `After:` key, and the first parts inherit the `After:`
predecessors of the task. The body of the task is never edited.

## OPTIONS

- `--max-parts`|- `-m`- *count*:
Upper bound for the number of derived parts (default:- `8`). The
lower bound is always- `2`, because fewer parts are no dissection.
 
- `--dry`|- `-d`:- *Report only*-- print the dissection table but create- *no*artifacts at all, so the decision stays fully reversible.
 
- `--force`|- `-f`:
Overwrite already existing sub-task plans. Without this option, the
skill detects colliding sub-task ids- *before*writing anything,
reports them, and stops with status- *targets exist*.
 

## ARGUMENTS

- *task-id*:
The unique identifier of the task whose plan should be dissected.
If omitted, the- *current*task id is used. A- *lone*argument
matching the task id syntax is always taken as the- *task id*, never
as a hint; to combine both, separate them with a- `:`character, as
in- `auth: split by API and UI`.
 
- *dissect-hint*:
A free-text hint telling- *how*the epic should be split. If omitted,
the parts are derived from the plan alone.
 

## SCENARIOS

- You want a large task plan split into cohesive sub-tasks
- You want an epic decomposed into independently implementable parts
- You want a preview of how a plan would be divided
- You want each part of a plan persisted as its own separate plan

## EXAMPLES

Dissect the current task plan:

```
❯ /ase-task-dissect
```

Preview the dissection of a specific task into at most three parts:

```
❯ /ase-task-dissect --dry --max-parts 3 auth
```

Re-dissect a task, overwriting the previously created sub-task plans:

```
❯ /ase-task-dissect --force auth
```

Dissect the current task plan along an explicitly given axis:

```
❯ /ase-task-dissect keep the meta file with the skills, docs separate
```

Dissect a specific task plan with a hint:

```
❯ /ase-task-dissect auth: one part per authentication provider
```

## SEE ALSO

`ase-repo-dissect`, `ase-task-edit`, `ase-task-condense`,
`ase-task-implement`, `ase-task-list`, `ase-task-delete`.

ase-task-implement - Implement a Task Plan

## NAME

`ase-task-implement` - Implement a Task Plan

## SYNOPSIS

`ase-task-implement`
    [`--help`|`-h`]
    [`--next`|`-n` *option*[,...]]
    [`--worktree`|`-w` *name*]
    [`--source`|`-s` *source*]
    [`--changeset`|`-c` *changeset*]
    [`--draft`|`-d`]
    [`--no-cleanup`|`-K`]
    [`--stateless`|`-S`]
    [*id*]

## DESCRIPTION

The `ase-task-implement` skill performs the *final implementation* of
a task plan by creating a complete *change set* from the plan's
*source* (the code basis) into the plan's *changeset* (the location of
the change set). Delivering the change set from there into the plan's
*target* is the separate concern of `ase-task-integrate`. Afterwards
the checkboxes of the realized `DOM`/`IFC`/`ARC`/`IMP` bullet-points
and of the performed and succeeded `REG`/`CON` bullet-points are ticked
to `[x]` (or `[/]` if only partially realized or succeeded), refreshing
the plan's `Modified:` key. Bullet-points in state `[-]` (cancelled) or
`[>]` (deferred) are *skipped*: they are neither realized nor checked,
do not count against completeness, and keep their checkbox untouched.

The *source* is selected by the plan's `Source:` frontmatter key
(if absent, the configured default `project.task.default.source`),
overridable via `--source`:

- `worktree`(default): the- *origin*working copy (the one the skill
was started in) with its checked-out branch.
- `branch:`- *name*: the existing branch- *name*. For a- `worktree`changeset without- `--worktree`, the origin working copy is- *switched*to it in place first, but only if it has- *no*uncommitted changes, otherwise the skill stops and touches nothing.

A context worktree created by `--worktree` carries its *own* branch
*name*, which starts at the source (so the source branch itself is never
checked out there and stays available as the integration target), while
a reused one keeps its own base, so earlier work in it is continued.

The *changeset* is selected by the plan's `Changeset:` frontmatter
key (if absent, the configured default `project.task.default.changeset`),
overridable via `--changeset`:

- `worktree`(default): the change set stays- *uncommitted*in the
context worktree.
- `branch:`- *name*: the change set is- *committed*onto the branch- *name*-- created from the source if missing -- inside the worktree
in which this branch is checked out, or inside a newly created
worktree- `.ase/worktree/<name>`, which is removed again after the
commit (unless- `--no-cleanup`is given), while an already existing
worktree is always kept.
- `attachment:`- *name*: the change set is made and verified in a- *temporary*worktree, detached at the source commit, and stored as
the attachment block of type- `text/x-diff; charset=utf-8; kind="<name>"`(with its base commit in the- `Desc:`key) of the
plan only, leaving all artifacts untouched. An already existing
attachment of this kind is re-applied first, so re-runs build the
change set- *cumulatively*. A- *stale*one -- older than the plan --
stops the skill with an error, and an already- *delivered*one --
marked- `merged`in its- `Desc:`key by- `ase-task-integrate --no-cleanup`-- is not re-applied, but replaced by a fresh change
set. The- `--draft`shorthand selects- `attachment:draft`, the- *implementation draft*of the plan, which
can be reviewed via- `ase-task-view --full`and delivered via- `ase-task-integrate --draft`. An undelivered draft is- *ignored*by
any other changeset (with a warning), as only- `attachment:draft`builds upon it.

For derived changesets (`branch:` and `attachment:`), *uncommitted*
changes of a `worktree` source are *not* carried over, as they start
at its `HEAD`.

The skill does *not* maintain the *change set snapshot* of the plan
(the attachment block of type `text/x-diff; charset=utf-8; kind="changeset"`), which only the `implementer` agent of `ase agent`
attaches after its implementation, but passes an existing one through
unchanged.

Unless `--stateless` is given, the plan's `Status:` key follows the
task lifecycle model: for `solo` it stays untouched (as the task is
closed by `ase-task-integrate`), and for `team` and `enterprise` it
becomes `IMPLEMENTING` before and `IMPLEMENTED` (if the change set was
applied completely and successfully) or `STALLED` (otherwise) after
the implementation.

The *kind of change* stated by the plan's `Kind:` frontmatter key
(`SPECIFYING`, `CRAFTING`, `REFACTORING`, or `RESOLVING`) selects which
*operation-specific tenet set* of the **ASE Tenets** is internalized
before any artifact is touched, in addition to the always applying
**GENERIC TENETS**. If a plan carries no such key, the kind is
*inferred* from the plan content, defaulting to `CRAFTING`.

If the task plan deliberately *omits* the `##  VERIFICATION (WHEN)` section
(as produced by `ase-task-edit` when invoked with `--unverified`),
the entire verification phase is strictly skipped: no build, tests,
linter, type-checker, or program execution is performed once the
source files have been modified.

The implementation runs to *completion*: the skill never concludes
while any point of the plan is still unimplemented, neither because the
change set grew large nor because an open question stands in the way.
Undecided details and defensible alternatives are therefore *decided by
the skill itself*, guided by the surrounding artifacts and the
internalized tenets, instead of interrupting the run with a question.
Once the change set is applied, the run closes with up to two boxed
summaries: `OPEN POINTS` lists everything now waiting for the user --
points left unimplemented on a *genuine blocker* no assumption can
bridge, follow-ups deliberately left out of scope with their reason,
and results needing review -- and `DECISIONS` lists each decision the
skill took on its own together with its rationale, so it can be
revisited or overruled. Either box is omitted entirely when it has no
content; a run which implemented every point and decided nothing on its
own emits no box at all.

After implementation, the user is asked whether to stop, hand off to
`ase-task-edit`, hand off to `ase-task-integrate`, or delete the task
plan, unless `--next` pre-selects this choice.

## OPTIONS

- `--next`|- `-n`- *option*[,...]:
Automatically answer the user dialog for the next step.- *option*is a single token or a- *comma-separated chronological list*of
tokens; the- *first*token is consumed by this skill, and any
remaining tokens are- *forwarded*(via- `--next`) to the downstream
skill. Recognized tokens at this skill:- `none`(default,
interactive answer required),- `DONE`(preserve task plan and stop),- `EDIT`(hand off to- `ase-task-edit`),- `INTEGRATE`(hand off to- `ase-task-integrate`, forwarding- `--worktree`,- `--changeset`, and- `--stateless`), or- `DELETE`(hand off to- `ase-task-delete`).
 
- `--worktree`|- `-w`- *name*:
Operate in the- *context worktree*- `.ase/worktree/<name>`instead
of the current working copy: it is- *reused*if it already exists,
otherwise it is- *created*with its own branch- *name*(checked out if
it exists, created at the source commit otherwise, so the source
branch itself is never checked out there). The- `worktree`value of the
changeset refers to it, while the- `worktree`value of the source
(and of the target of a subsequent integration) keeps referring to
the origin working copy, so an integration delivers the change set
from the context worktree back into the origin branch. The option
is permitted for a- `worktree`changeset only, as a- `branch:`or- `attachment:`changeset would leave the context worktree unused.
 
- `--source`|- `-s`- *source*:
Override the plan's- `Source:`key (- `worktree`or- `branch:`- *name*)
for this run, without changing the plan.
 
- `--changeset`|- `-c`- *changeset*:
Override the plan's- `Changeset:`key (- `worktree`,- `branch:`- *name*,
or- `attachment:`- *name*) for this run, without changing the plan.
 
- `--draft`|- `-d`:
Shorthand for- `--changeset attachment:draft`: create the- *implementation draft*of the plan only, leaving all artifacts
untouched (a dry-run). Mutually exclusive with- `--changeset`.
 
- `--no-cleanup`|- `-K`:
For a- `branch:`changeset, keep the worktree- `.ase/worktree/<name>`newly created by this run, instead of removing it after the commit,
so the change set can be reviewed, built, and tested there. Without
this option, the worktree is removed only if it has no uncommitted
changes, as the change set lives on the branch alone.
 
- `--stateless`|- `-S`:
Leave the plan's- `Status:`key untouched.
 

## ARGUMENTS

- *id*:
The unique identifier of the task whose plan should be
implemented. If omitted, the- *current*task id is used.

## SCENARIOS

- You want a task plan turned into actual changes
- You want the planned change set applied and verified
- You want the change set committed onto a dedicated branch
- You want the change set attached to the plan for review only
- You want a dry-run of the implementation as a reviewable draft
- You want the implementation isolated in a Git worktree

## EXAMPLES

Implement the current task plan in the current working copy:

```
❯ /ase-task-implement
```

Implement a specific task and delete the plan when done:

```
❯ /ase-task-implement --next DELETE hello
```

Draft the implementation of the task `hello` and apply the draft when
done:

```
❯ /ase-task-implement --draft --next INTEGRATE hello
```

Implement the task `hello` inside the isolated Git worktree
`.ase/worktree/hello` and integrate it afterwards:

```
❯ /ase-task-implement --worktree hello --next INTEGRATE,DONE hello
```

Implement the task `hello` by committing its change set onto the
branch `feature-hello`:

```
❯ /ase-task-implement --changeset branch:feature-hello hello
```

## SEE ALSO

`ase-task-edit`, `ase-task-integrate`,
`ase-task-view`, `ase-task-delete`.

ase-task-integrate - Integrate a Task Change Set

## NAME

`ase-task-integrate` - Integrate a Task Change Set

## SYNOPSIS

`ase-task-integrate`
    [`--help`|`-h`]
    [`--next`|`-n` *option*[,...]]
    [`--worktree`|`-w` *name*]
    [`--changeset`|`-c` *changeset*]
    [`--draft`|`-d`]
    [`--target`|`-t` *target*]
    [`--mode`|`-m` `merge`|`rebase`|`squash`]
    [`--no-cleanup`|`-K`]
    [`--stateless`|`-S`]
    [*id*]

## DESCRIPTION

The `ase-task-integrate` skill delivers the *change set* of a task
plan, as created by `ase-task-implement`, from the plan's *changeset*
(the location of the change set) into the plan's *target* (the place
of integration). The integration itself is performed by the skill
`ase-repo-merge`, which commits still uncommitted changes of the
change set, merges them into the target branch (through a regular
merge, a rebase with fast-forward, or a squash merge, as selected via
`--mode`), resolves merge conflicts semantically, and reports the
verdict `MERGED`, `CONFLICT`, or `FAILED`.

The *changeset* is selected by the plan's `Changeset:` frontmatter
key (if absent, the configured default `project.task.default.changeset`),
overridable via `--changeset`:

- `worktree`(default): the uncommitted changes and the checked-out
branch of the context worktree. If it equals the target branch
(only possible without- `--worktree`), the integration just commits
the uncommitted changes in place.
- `branch:`- *name*: the existing branch- *name*.
- `attachment:`- *name*: the attachment block of type- `text/x-diff; charset=utf-8; kind="<name>"`of the plan, which is applied on a- *temporary*branch at the commit it is based on, so drift of the
target is resolved by the merge. A- *stale*attachment -- one whose- `Modified:`key is absent or older than the- `Modified:`key of the
plan frontmatter -- stops the skill with an error, and so does an
already- *delivered*one -- one whose- `Desc:`key carries the- `merged`marker of an earlier- `--no-cleanup`integration.
- `attachment:draft`: the- *implementation draft*of the plan, as
created by- `ase-task-implement --draft`, selectable via the- `--draft`shorthand.

The skill does *not* maintain the *change set snapshot* of the plan
(the attachment block of type `text/x-diff; charset=utf-8; kind="changeset"`), which only the `integrator` agent of `ase agent`
updates after a successful integration, but passes an existing one
through unchanged.

The *target* is selected by the plan's `Target:` frontmatter key
(if absent, the configured default `project.task.default.target`),
overridable via `--target`:

- `worktree`: the checked-out branch of the- *origin*working copy,
i.e. the one the skill was started in (also with- `--worktree`).
- `branch:`- *name*: the existing branch- *name*, which has to be
checked out in some worktree and to have no uncommitted changes.
- `source`(default): whatever the plan's- `Source:`key refers to.

The skill stops with an error -- touching nothing -- if the change set
and the target are the same branch (except for the `worktree` changeset
of the origin working copy, which just commits in place, see above), or
if the change set carries neither commits beyond the target nor
uncommitted changes. Likewise, a merge which leaves the content of the
target unchanged, as the change set already landed earlier (e.g. on a
repeated integration), counts as *no* integration: its empty merge
commit is dropped again and the skill stops with an error. So it never
reports a success without having delivered anything.

Unless `--stateless` is given, the plan's `Status:` key follows the
task lifecycle model: for `solo` the plan has to be `OPEN` (otherwise
the integration is refused) and it becomes `CLOSED` on a successful
integration and stays `OPEN` otherwise, for `enterprise` it becomes
`INTEGRATING` before and `INTEGRATED` (on success), `QUALIFIED` (on
nothing delivered), or `DEFERRED` (otherwise) after the integration,
and for `team`, which defines no integration states, it stays
untouched.

After a successful integration, the user is asked whether to preserve
or delete the task plan, unless `--next` pre-selects this choice.

## OPTIONS

- `--next`|- `-n`- *option*[,...]:
Automatically answer the user dialog for the next step. Recognized
tokens at this skill:- `none`(default, interactive answer
required),- `DONE`(preserve task plan and stop), or- `DELETE`(hand
off to- `ase-task-delete`).
 
- `--worktree`|- `-w`- *name*:
Operate on the- *context worktree*- `.ase/worktree/<name>`instead
of the current working copy, as for- `ase-task-implement`. The
worktree has to exist already: an integration never creates it, so
a mistyped- *name*stops the skill with an error. The option is
permitted for a- `worktree`changeset only, as a- `branch:`or- `attachment:`changeset would leave the context worktree unused.
 
- `--changeset`|- `-c`- *changeset*:
Override the plan's- `Changeset:`key (- `worktree`,- `branch:`- *name*,
or- `attachment:`- *name*) for this run, without changing the plan.
 
- `--draft`|- `-d`:
Shorthand for- `--changeset attachment:draft`: deliver the- *implementation draft*of the plan. Mutually exclusive with- `--changeset`.
 
- `--target`|- `-t`- *target*:
Override the plan's- `Target:`key (- `worktree`,- `branch:`- *name*, or- `source`) for this run, without changing the plan.
 
- `--mode`|- `-m`- `merge`|- `rebase`|- `squash`:
The merge mode, passed through to- `ase-repo-merge`(default:- `merge`):- `merge`creates a merge commit,- `rebase`rebases the
change set branch (rewriting its commits) onto the target branch
and fast-forwards the target branch, and- `squash`combines all
changes of the change set into one new commit on the target branch.
 
- `--no-cleanup`|- `-K`:
After a successful integration, keep the delivered change set
instead of removing it: the branch and its worktree, or the
attachment block, which is then marked as delivered by appending- `, merged <commit>`to its- `Desc:`key, so it is never delivered
again. Without this option, a delivered branch is removed only if
Git considers it merged (always for- `--mode squash`) and its
worktree has no uncommitted changes. A temporary branch of an
attachment is always removed.
 
- `--stateless`|- `-S`:
Leave the plan's- `Status:`key untouched.
 

## ARGUMENTS

- *id*:
The unique identifier of the task whose change set should be
integrated. If omitted, the- *current*task id is used.

## SCENARIOS

- You want the implemented change set committed to the code base
- You want a change set branch merged into its target branch
- You want a change set attachment delivered into the code base
- You want a reviewed implementation draft applied to the code base
- You want the task closed once its change set has landed

## EXAMPLES

Integrate (commit) the change set of the current task plan:

```
❯ /ase-task-integrate
```

Apply the implementation draft of the task `hello` and remove the
draft attachment afterwards:

```
❯ /ase-task-integrate --draft hello
```

Merge the change set branch of the task `hello` into `main` and
keep the branch afterwards:

```
❯ /ase-task-integrate --changeset branch:feature-hello --target branch:main --no-cleanup hello
```

Squash the change set branch of the task `hello` into one commit on
its target:

```
❯ /ase-task-integrate --changeset branch:feature-hello --mode squash hello
```

## SEE ALSO

`ase-task-implement`, `ase-repo-merge`,
`ase-task-status`, `ase-task-delete`.

ase(1) - Agentic Software Engineering (ASE)

## NAME

`ase(1)` - Agentic Software Engineering (ASE)

## SYNOPSIS

`ase`
[`-h`|`--help`]
[`-V`|`--version`]
[`-l`|`--log-level` *level*]
[`-L`|`--log-file` *file*]
*command*
[*options* [...]]
[*args* [...]]

## DESCRIPTION

**ase**, *Agentic Software Engineering (ASE)*, is the command-line
companion tool of the *ASE* agent tool plugin for
*Anthropic Claude Code CLI* (primary target), *GitHub Copilot CLI*,
and *OpenAI Codex CLI* (secondary targets).

It provides the plugin and tool setup, the layered configuration
management, a per-project background HTTP service (bridged into the
agent tool as a Model-Context-Protocol (MCP) server), the agent tool
hook handlers, the statusline rendering, the persisted task plan
management (including a task store server and an interactive task
board), the autonomous task lifecycle agents (plus their agent relay
and fleet supervisor), the project artifact resolution, the
*SpecBook*-based specification initializing, linting, exporting,
querying, and previewing, and miscellaneous utilities like diagram rendering,
identifier minting, text measuring, and SVG rasterizing.

Most commands are sub-command groups. Each group and each command has
its own manual page, named after the command path with dashes, e.g.,
`ase-task(1)` for `ase task` and `ase-task-board(1)` for `ase task board`. They are shipped with **ase** and shown via `ase man`, e.g.,
`ase man task board`.

## OPTIONS

The following top-level (global) options exist. They have to be given
*before* the *command*, as all options after the *command* belong to
the command itself.

- [- `-h`|- `--help`]:
Show program usage information, including the list of all top-level
commands, and exit. Every command and sub-command also accepts- `-h`|- `--help`to show its own usage information. Alternatively,- `ase help`[- *command*] shows the usage information of- *command*.
 
- [- `-V`|- `--version`]:
Show program version information (as- `ASE`- *version*) and exit.
 
- [- `-l`|- `--log-level`- *level*]:
Set the logging verbosity. Supported- *level*values are- `error`,- `warning`,- `info`(default), and- `debug`. Log messages of a level
above the selected one are suppressed. The level is passed on to all
background processes (like the service, the task store, the agent
relay, and the agents) started by the command.
 
- [- `-L`|- `--log-file`- *file*]:
Redirect log output to- *file*(appended). Use- `-`(default) to write
log messages to standard error. If the log output is connected to a
terminal, it is colored.
 

## COMMANDS

The following top-level commands exist:

- `ase setup`- *command*[...]:
Install, update, uninstall, enable, disable, and report the- *ASE*tool and plugin for an agent tool, and activate or deactivate the
pre-defined foreign MCP servers, the- *ASE*statusline, and the- *ASE*telemetry export. See- `ase-setup(1)`.
 
- `ase config`[- `--scope`- *scope*]- *command*[...]:
Manage the layered- *ASE*configuration of the scope chain- `user`<- `project`<- `task`<- `session`. See- `ase-config(1)`.
 
- `ase alias`- *command*[...]:
Manage the skill command aliases: full aliases, option aliases, and
option defaults of the- *ASE*skills. See- `ase-alias(1)`.
 
- `ase mcp`:
Bridge stdio MCP to the per-project background service. Configured
as an MCP server in the agent tool. See- `ase-mcp(1)`.
 
- `ase service`- *command*[...]:
Manage the per-project background HTTP service. See- `ase-service(1)`.
 
- `ase hook`- *command*[...]:
Handle the hook events of the agent tool. Configured as hook
handlers in the agent tool. See- `ase-hook(1)`.
 
- `ase statusline`[- *options*] [- *line*[...]]:
Render the statusline of the agent tool. Configured as statusline
command in the agent tool. See- `ase-statusline(1)`.
 
- `ase task`- *command*[...]:
Manage the persisted task plans in the task store, show the task
board, and run the task store server. See- `ase-task(1)`.
 
- `ase artifact`- *command*[...]:
Resolve project artifact kinds to project-relative file lists. See- `ase-artifact(1)`.
 
- `ase spec`- *command*[...]:
Initialize, lint, export, query, and preview the- *SpecBook*-based project
specification. See- `ase-spec(1)`.
 
- `ase util`- *command*[...]:
Utility commands (meta files, compatibility probes, diagram rendering,
worktree resolution, identifier minting, text measuring, and SVG
rasterizing). See- `ase-util(1)`.
 
- `ase agent`- *agent*- *command*[...]:
Run the autonomous agents driving the task lifecycle phases,
processing the requests of tasks, estimating the effort of tasks,
researching topics for tasks, or configured by definitions, individually or as a fleet, optionally
reporting to the agent relay. See- `ase-agent(1)`.
 
- `ase man`[- *page*[...]]:
Browse the manual pages of all commands and skills interactively,
with- *page*selected, e.g.,- `ase man agent fleet`for- `ase-agent-fleet(1)`, or (outside a terminal) print one of them as
plain text. See- `ase-man(1)`.
 
- `ase help`[- *command*]:
Show the usage information of- *command*(or of- `ase`itself).
 

## ENVIRONMENT

The following environment variables are honored by multiple commands.
The command-specific ones are described in their own manual pages.

- `ASE_TOOL`:
The default target agent tool (- `claude`,- `copilot`, or- `codex`) of
all commands accepting a- `--tool`option (default:- `claude`).
 
- `ASE_TASK_ID`:
The default task id of the- `ase task`commands accepting an optional
task id (default:- `default`).
 
- `ASE_HEADLESS`:
Set to- `true`by the agents for the agent tool processes they spawn,
to signal the skills that they run without any user interaction.
 
- `ASE_SETUP_DEV`:
The default of the- `--dev`option of- `ase setup install`,- `ase setup update`, and- `ase setup uninstall`.
 
- `EDITOR`,- `VISUAL`:
The text editor used by- `ase config edit`,- `ase task edit`, and- `ase task board`(falling back to- `vi`).
 
- `PAGER`:
The pager used by- `ase task view`(falling back to- `more`).
 
- `XDG_CONFIG_HOME`,- `XDG_STATE_HOME`,- `APPDATA`,- `LOCALAPPDATA`:
The base directories of the- *per-user configuration directory*and
the- *per-user state directory*(see- **FILES**).
 

## FILES

- *per-user configuration directory*:
The per-user- *ASE*configuration directory:- `~/Library/Application Support/ase`on macOS,- `%APPDATA%\ase`on Windows, and- `$XDG_CONFIG_HOME/ase`(falling back to- `~/.config/ase`) on Linux
and other Unix systems. It holds the- `user`scope configuration- `config.yaml`, the user agent definitions- `agents/`, the user skill
command aliases- `aliases.yaml`, and the state
of the task store server (- `store.yaml`,- `store.log`,- `tasks/`) and
agent relay server (- `relay.yaml`,- `relay.log`).
 
- *per-user state directory*:
The machine-local per-user- *ASE*state (session configurations, task
lifecycle deviation warnings, agent run records, and telemetry
state), never roamed or versioned:- `~/Library/Application Support/ase`on macOS,- `%LOCALAPPDATA%\ase`on Windows, and- `$XDG_STATE_HOME/ase`(falling back to- `~/.local/state/ase`) on Linux
and other Unix systems.
 
- `.ase/`:
The per-project- *ASE*directory in the project root, holding the- `project`scope configuration- `config.yaml`, the agent definitions- `agents/`, the skill command aliases- `aliases.yaml`, the fleet
configuration- `fleet.yaml`, the task plans- `task/`of the default local task store, and the machine-local
runtime files (- `service.yaml`,- `service.log`,- `board.yaml`,- `agent/`,
and- `worktree/`), the latter covered by the self-ignoring- `.ase/.gitignore`.
 

## EXIT STATUS

The **ase** command exits with status `0` on success and with status
`1` on any error (with the error message logged at level `error`),
unless a command documents otherwise. A usage error (an unknown
command or option, or a missing argument) is reported together with
the usage information and leads to exit status `1`.

## EXAMPLES

```
# install the ASE tool and plugin for Anthropic Claude Code CLI
$ ase setup install
# show the effective configuration of the current project
$ ase config list
# list all open task plans with their status and title
$ ase task list -v
# show the interactive task board
$ ase task board
# start the fleet of agents of ".ase/fleet.yaml"
$ ase agent fleet start
```

## SEE ALSO

`ase-setup(1)`, `ase-config(1)`, `ase-alias(1)`, `ase-mcp(1)`, `ase-service(1)`,
`ase-hook(1)`, `ase-statusline(1)`, `ase-task(1)`, `ase-artifact(1)`,
`ase-spec(1)`, `ase-util(1)`, `ase-agent(1)`, `ase-man(1)`,
`claude(1)`, `copilot(1)`, `codex(1)`, `git(1)`.

https://ase.tools

## HISTORY

Development of `ase(1)` started in October 2025.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup(1) - Install, Update, and Uninstall the ASE Tool and Plugin

## NAME

`ase-setup(1)` - Install, Update, and Uninstall the ASE Tool and Plugin

## SYNOPSIS

`ase setup`
[`-h`|`--help`]
*command*
[*options* [...]]
[*args* [...]]

## DESCRIPTION

**ase setup** is the entry point group for installing, updating,
uninstalling, enabling, disabling, and reporting the *ASE* tool and its
companion agent tool plugin, and for activating or deactivating the
pre-defined foreign MCP servers, the *ASE* statusline, and the *ASE*
telemetry export towards the AI usage accounting. Without a
*command*, the usage information is shown and the command exits with
status `1`.

The target agent tool is either *Anthropic Claude Code CLI*
(`claude`), *GitHub Copilot CLI* (`copilot`), or *OpenAI Codex CLI*
(`codex`). All operations are performed through the plugin and MCP
commands of the respective agent tool CLI (e.g., `claude plugin install`), which hence has to be found in `$PATH`, and through direct
edits of its settings files (for the output style, the function
hooks, the statusline, and the telemetry). Every sub-process command is logged (with API keys
masked) and its output is shown only on failure.

## OPTIONS

- [`-h`|`--help`]:
Show usage information of the command group and exit.

The following options are shared by most sub-commands (see their
individual manual pages for the exact applicability):

- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`(- *Anthropic Claude Code
CLI*),- `copilot`(- *GitHub Copilot CLI*), or- `codex`(- *OpenAI Codex
CLI*). The default is the value of the environment variable- `ASE_TOOL`, if set and non-empty, else- `claude`. An invalid value
is rejected with an error. Accepted by all sub-commands except- `ase setup mcp list`.
 
- [- `-s`|- `--scope`- *scope*]:
Select the installation scope of the plugin, MCP server, or
statusline registration:- `user`(registered globally for the
current user),- `project`(registered in the project and meant to be
shared via version control), or- `local`(registered in the project,
but kept out of version control).
 - Only supported for - `--tool claude`(and, for the statusline, a- `user`scope for- `--tool copilot`); a non-- `user`scope for any other tool
fails with an error, as those agent tools have no scope concept. Not
accepted by- `ase setup status`(which reports all scopes at once) and- `ase setup telemetry`(which is always user-level).
 

## COMMANDS

- `ase setup install`[- `-t`|- `--tool`- *tool*] [- `-s`|- `--scope`- *scope*] [- `-d`|- `--dev`]:
Install the- *ASE*plugin into the agent tool. See- `ase-setup-install(1)`.
 
- `ase setup update`[- `-t`|- `--tool`- *tool*] [- `-s`|- `--scope`- *scope*] [- `-f`|- `--force`] [- `-d`|- `--dev`]:
Update the- *ASE*tool and the- *ASE*plugin to their latest
versions. See- `ase-setup-update(1)`.
 
- `ase setup uninstall`[- `-t`|- `--tool`- *tool*] [- `-s`|- `--scope`- *scope*] [- `-d`|- `--dev`]:
Uninstall the- *ASE*plugin and the- *ASE*tool. See- `ase-setup-uninstall(1)`.
 
- `ase setup enable`[- `-t`|- `--tool`- *tool*] [- `-s`|- `--scope`- *scope*]:
Enable the already installed- *ASE*plugin. See- `ase-setup-enable(1)`.
 
- `ase setup disable`[- `-t`|- `--tool`- *tool*] [- `-s`|- `--scope`- *scope*]:
Disable the- *ASE*plugin without uninstalling it. See- `ase-setup-disable(1)`.
 
- `ase setup status`[- `-t`|- `--tool`- *tool*]:
Report the plugin, MCP server, output style, function hooks,
statusline, and telemetry registrations of all scopes. See- `ase-setup-status(1)`.
 
- `ase setup mcp`- `list`|- `activate`|- `deactivate`[...]:
List, activate, or deactivate the pre-defined foreign MCP servers.
See- `ase-setup-mcp(1)`.
 
- `ase setup statusline`- `activate`|- `deactivate`[...]:
Activate or deactivate the- *ASE*statusline. See- `ase-setup-statusline(1)`.
 
- `ase setup telemetry`- `activate`|- `deactivate`[...]:
Activate or deactivate the- *OpenTelemetry*export of the agent tool
towards the- *ASE*OTLP ingress. See- `ase-setup-telemetry(1)`.
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of the- `--tool`option (default:- `claude`).
 
- `ASE_SETUP_DEV`:
The default of the- `--dev`option of- `ase setup install`,- `ase setup update`, and- `ase setup uninstall`: any non-empty value other than- `0`and- `false`(case-insensitive) enables it.
 
- `ASE_SETUP_DEV_BASEDIR`:
The base directory of the local- *ASE*working copy used with- `--dev`(default: the parent directory of the installed- `@rse/ase`package directory).
 
- `ASE_MCP_KEY_`- *KEY*:
The API keys of the pre-defined MCP servers (see- `ase-setup-mcp(1)`).
 

## FILES

- `~/.claude/settings.json`,- `.claude/settings.json`,- `.claude/settings.local.json`:
The settings files of- *Anthropic Claude Code CLI*of the scopes- `user`,- `project`, and- `local`(the latter two relative to the
current working directory).
 
- `$COPILOT_HOME/settings.json`(default:- `~/.copilot/settings.json`):
The single per-user settings file of- *GitHub Copilot CLI*.
 
- `$CODEX_HOME/config.toml`(default:- `~/.codex/config.toml`):
The per-user configuration file of- *OpenAI Codex CLI*.
 

## EXIT STATUS

`0` on success, `1` on any error, and `1` when invoked without a
*command*.

## EXAMPLES

```
# install ASE for Anthropic Claude Code CLI globally
$ ase setup install
# install ASE for OpenAI Codex CLI
$ ase setup install --tool codex
# report all ASE registrations
$ ase setup status
```

## SEE ALSO

`ase(1)`, `ase-setup-install(1)`, `ase-setup-update(1)`,
`ase-setup-uninstall(1)`, `ase-setup-enable(1)`,
`ase-setup-disable(1)`, `ase-setup-status(1)`, `ase-setup-mcp(1)`,
`ase-setup-statusline(1)`, `ase-setup-telemetry(1)`,
`claude(1)`, `copilot(1)`, `codex(1)`, `npm(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-install(1) - Install the ASE Plugin into an Agent Tool

## NAME

`ase-setup-install(1)` - Install the ASE Plugin into an Agent Tool

## SYNOPSIS

`ase setup install`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]
[`-d`|`--dev`]

## DESCRIPTION

**ase setup install** installs the *ASE* plugin into the agent tool
selected by `--tool`. It requires `npm(1)` and the CLI of the agent
tool (`claude`, `copilot`, or `codex`) to be found in `$PATH`.

The installation is performed in two steps through the plugin
commands of the agent tool CLI: first, the *ASE* plugin marketplace is
registered from a local directory (`<cli> plugin marketplace add`
*dir*), and then the plugin `ase@ase` is installed from it (`<cli> plugin install ase@ase`, resp. `codex plugin add ase@ase` for *OpenAI
Codex CLI*), retried up to three times with a delay of one second.

The marketplace directory is the installed `@rse/ase` package
itself, which bundles the plugin, or, with `--dev`, the local *ASE*
working copy.

For `--tool claude`, the plugin-shipped output style `ase:ase-terse`
is additionally selected by writing `"outputStyle": "ase:ase-terse"`
into the `settings.json` of the installation scope, unless an
`outputStyle` is already selected there: an *ASE* one is left as is,
and a foreign one is preserved with a warning. For `--tool copilot`
and `--tool codex`, which have no output style concept, the `ase hook session-start` handler injects the identical style instructions into
the session context instead (see `ase-hook(1)`).

For `--tool claude`, the function hooks of the plugin-shipped
*Claude Code* mods (e.g., the clickable grilling table and the help
bands above the prompt) are additionally enabled by writing
`"CLAUDE_CODE_ENABLE_FUNCTION_HOOKS": "1"` into the `env` of the
`settings.json` of the installation scope, unless the variable is
already present there: an *ASE* one (value `1`) is left as is, and a
foreign one (any other value) is preserved with a warning.

The *ASE* tool itself is not installed by this command, as it is
already installed when running it (e.g., via `npm install -g @rse/ase`).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`).
 
- [- `-s`|- `--scope`- *scope*]:
Select the plugin installation scope:- `user`(default; registered
globally for the current user in- `~/.claude/settings.json`),- `project`(registered in- `.claude/settings.json`of the current
project, meant to be shared via version control), or- `local`(registered in- `.claude/settings.local.json`of the current
project, kept out of version control).
 - A non-- `user`scope is passed as- `--scope`- *scope*to the agent tool
CLI. Only supported for- `--tool claude`; a non-- `user`scope for- `copilot`or- `codex`fails with an error.
 
- [- `-d`|- `--dev`]:
Development mode: register the plugin marketplace from the local- *ASE*working copy (- `$ASE_SETUP_DEV_BASEDIR`, else the parent
directory of the- `@rse/ase`package directory) instead of the
bundled plugin of the installed package (default:- `$ASE_SETUP_DEV`).
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.
 
- `ASE_SETUP_DEV`:
The default of- `--dev`: any non-empty value other than- `0`and- `false`(case-insensitive) enables it.
 
- `ASE_SETUP_DEV_BASEDIR`:
The local- *ASE*working copy used with- `--dev`.
 

## FILES

- `~/.claude/settings.json`,- `.claude/settings.json`,- `.claude/settings.local.json`:
The settings file of the- `user`,- `project`, resp.- `local`scope,
receiving the- `outputStyle`selection and the- `CLAUDE_CODE_ENABLE_FUNCTION_HOOKS`variable for- `--tool claude`.

## EXIT STATUS

`0` on success, `1` on any error (e.g., a missing `npm` or agent tool
CLI, an invalid option value, or a failing agent tool command, whose
output is then shown).

## EXAMPLES

```
# install for Anthropic Claude Code CLI, globally for the current user
$ ase setup install
# install for Anthropic Claude Code CLI, for the current project only
$ ase setup install --scope project
# install for GitHub Copilot CLI
$ ase setup install --tool copilot
# install from the local working copy
$ ase setup install --dev
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-setup-update(1)`,
`ase-setup-uninstall(1)`, `ase-setup-status(1)`, `ase-hook(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-update(1) - Update the ASE Tool and Plugin

## NAME

`ase-setup-update(1)` - Update the ASE Tool and Plugin

## SYNOPSIS

`ase setup update`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]
[`-f`|`--force`]
[`-d`|`--dev`]

## DESCRIPTION

**ase setup update** updates the *ASE* tool and the *ASE* plugin of
the agent tool selected by `--tool` to their latest versions. It
requires `npm(1)` and the CLI of the agent tool to be found in
`$PATH`. First, a potentially running `ase service` of the current
project is stopped (on a best-effort basis), as it would otherwise
keep running the old code.

In the regular mode, the current version of the tool is compared with
the latest version of `@rse/ase` known from the NPM registry (as
determined by a cached background check, refreshed at most once per
hour). If both are equal, the command reports that *ASE* is already at
the latest version and exits without any further action (unless
`--force` is given).

Otherwise, the tool is updated via `npm update -g @rse/ase` (prefixed
with `sudo(8)` if the global `npm` installation root is not writable by
the current user and `sudo` is available), the plugin marketplace
snapshot is refreshed (`<cli> plugin marketplace update ase`, resp.
`codex plugin marketplace upgrade ase`), and the plugin is updated
(`<cli> plugin update ase@ase`, resp. removed and re-added for *OpenAI
Codex CLI*, which has no plugin update command).

In the development mode (`--dev`), the local *ASE* working copy is
re-built instead (`npm install` and `npm start build` in its `tool/`
directory), and the plugin is re-installed (uninstalled and
installed again), as the local plugin files are already current, but
carry no version change in the plugin manifest.

Like `ase setup install`, for `--tool claude`, the output style
`ase:ase-terse` is finally selected in the `settings.json` of the
installation scope, unless a foreign `outputStyle` is already selected
there, which is preserved with a warning, and the function hooks are
enabled through `"CLAUDE_CODE_ENABLE_FUNCTION_HOOKS": "1"` in its
`env`, unless a foreign value of the variable is already present
there, which is preserved with a warning.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`).
 
- [- `-s`|- `--scope`- *scope*]:
Select the plugin installation scope:- `user`,- `project`, or- `local`. The default is auto-detected from the existing
installation: the scopes- `local`,- `project`, and- `user`are probed
in this order (strongest first), and the first one whose settings
file registers the plugin- `ase@ase`under- `enabledPlugins`(enabled
or disabled) is taken, falling back to- `user`when no installation
is found.
 - For tools other than - `claude`, the scope is always- `user`, and an
explicitly given non-- `user`scope fails with an error.
 
- [- `-f`|- `--force`]:
Always perform the update, even if the tool is already at the
latest version (default: off). Without effect in combination with- `--dev`, which always re-builds.
 
- [- `-d`|- `--dev`]:
Development mode: re-build the local- *ASE*working copy
(- `$ASE_SETUP_DEV_BASEDIR`, else the parent directory of the- `@rse/ase`package directory) and re-install the plugin from it,
instead of updating from the NPM registry (default:- `$ASE_SETUP_DEV`).
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.
 
- `ASE_SETUP_DEV`:
The default of- `--dev`: any non-empty value other than- `0`and- `false`(case-insensitive) enables it.
 
- `ASE_SETUP_DEV_BASEDIR`:
The local- *ASE*working copy used with- `--dev`.
 

## EXIT STATUS

`0` on success (including an already current version), `1` on any
error.

## EXAMPLES

```
# update ASE for Anthropic Claude Code CLI
$ ase setup update
# force the update for OpenAI Codex CLI
$ ase setup update --tool codex --force
# re-build and re-install from the local working copy
$ ase setup update --dev
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-setup-install(1)`,
`ase-setup-uninstall(1)`, `ase-service(1)`, `npm(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-uninstall(1) - Uninstall the ASE Plugin and Tool

## NAME

`ase-setup-uninstall(1)` - Uninstall the ASE Plugin and Tool

## SYNOPSIS

`ase setup uninstall`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]
[`-d`|`--dev`]

## DESCRIPTION

**ase setup uninstall** uninstalls the *ASE* plugin from the agent
tool selected by `--tool` and the *ASE* tool itself. It requires
`npm(1)` and the CLI of the agent tool to be found in `$PATH`. The
steps are:

- A potentially running `ase service`of the current project is
stopped (on a best-effort basis).
- For `--tool claude`, the selection of the output style`ase:ase-terse`is removed from the`settings.json`of the
installation scope, while a foreign`outputStyle`is preserved with
a warning. Likewise, the variable`CLAUDE_CODE_ENABLE_FUNCTION_HOOKS`(value`1`) is removed from its`env`(together with an`env`which
became empty), while a foreign value is preserved with a warning.
- The plugin `ase@ase`is uninstalled (`<cli> plugin uninstall ase@ase`, resp.`codex plugin remove ase@ase`) and the plugin
marketplace`ase`is removed (`<cli> plugin marketplace remove ase`), where an already absent plugin or marketplace is skipped.
- Unless in development mode (`--dev`), the tool is uninstalled via`npm uninstall -g @rse/ase`(prefixed with`sudo(8)`if the global`npm`installation root is not writable by the current user and`sudo`is available).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`).
 
- [- `-s`|- `--scope`- *scope*]:
Select the plugin installation scope:- `user`,- `project`, or- `local`. The default is auto-detected from the existing
installation: the scopes- `local`,- `project`, and- `user`are probed
in this order, and the first one whose settings file registers the
plugin- `ase@ase`under- `enabledPlugins`is taken, falling back to- `user`. Only supported for- `--tool claude`; a non-- `user`scope for- `copilot`or- `codex`fails with an error.
 
- [- `-d`|- `--dev`]:
Development mode: uninstall the plugin only, but keep the- *ASE*tool, as it is the local working copy (default:- `$ASE_SETUP_DEV`).
 

## ENVIRONMENT

## EXIT STATUS

`0` on success, `1` on any error.

## EXAMPLES

```
# uninstall ASE from Anthropic Claude Code CLI
$ ase setup uninstall
# uninstall the plugin from GitHub Copilot CLI
$ ase setup uninstall --tool copilot
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-setup-install(1)`,
`ase-setup-disable(1)`, `npm(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-enable(1) - Enable the ASE Plugin in an Agent Tool

## NAME

`ase-setup-enable(1)` - Enable the ASE Plugin in an Agent Tool

## SYNOPSIS

`ase setup enable`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]

## DESCRIPTION

**ase setup enable** enables the already installed *ASE* plugin in the
agent tool selected by `--tool`, e.g., after it was disabled via `ase setup disable`. It requires the CLI of the agent tool to be found in
`$PATH`.

For *Anthropic Claude Code CLI*, the plugin is enabled via `claude plugin enable ase@ase` (with `--scope` *scope* for a non-`user`
scope). As *GitHub Copilot CLI* and *OpenAI Codex CLI* have no plugin
enable command, the plugin is (re-)installed there instead (`copilot plugin install ase@ase`, resp. `codex plugin add ase@ase`, retried up
to three times), which requires the *ASE* plugin marketplace to be
still registered (see `ase-setup-install(1)`).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`).
 
- [- `-s`|- `--scope`- *scope*]:
Select the plugin installation scope:- `user`,- `project`, or- `local`. The default is auto-detected from the existing
installation: the scopes- `local`,- `project`, and- `user`are probed
in this order, and the first one whose settings file registers the
plugin- `ase@ase`under- `enabledPlugins`(enabled or disabled) is
taken, falling back to- `user`. Only supported for- `--tool claude`;
a non-- `user`scope for- `copilot`or- `codex`fails with an error.
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.

## EXIT STATUS

`0` on success, `1` on any error.

## EXAMPLES

```
# re-enable the ASE plugin in Anthropic Claude Code CLI
$ ase setup enable
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-setup-disable(1)`,
`ase-setup-status(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-disable(1) - Disable the ASE Plugin in an Agent Tool

## NAME

`ase-setup-disable(1)` - Disable the ASE Plugin in an Agent Tool

## SYNOPSIS

`ase setup disable`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]

## DESCRIPTION

**ase setup disable** disables the installed *ASE* plugin in the agent
tool selected by `--tool` without uninstalling it, so it can be
enabled again via `ase setup enable`. It requires the CLI of the
agent tool to be found in `$PATH`.

For *Anthropic Claude Code CLI*, the plugin is disabled via `claude plugin disable ase@ase` (with `--scope` *scope* for a non-`user`
scope), which keeps its registration under `enabledPlugins` (with the
value `false`). As *GitHub Copilot CLI* and *OpenAI Codex CLI* have no
plugin disable command, the plugin is uninstalled there instead
(`copilot plugin uninstall ase@ase`, resp. `codex plugin remove ase@ase`, retried up to three times), while its plugin marketplace
stays registered.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`).
 
- [- `-s`|- `--scope`- *scope*]:
Select the plugin installation scope:- `user`,- `project`, or- `local`. The default is auto-detected from the existing
installation: the scopes- `local`,- `project`, and- `user`are probed
in this order, and the first one whose settings file registers the
plugin- `ase@ase`under- `enabledPlugins`is taken, falling back to- `user`. Only supported for- `--tool claude`; a non-- `user`scope for- `copilot`or- `codex`fails with an error.
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.

## EXIT STATUS

`0` on success, `1` on any error.

## EXAMPLES

```
# temporarily disable the ASE plugin in Anthropic Claude Code CLI
$ ase setup disable
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-setup-enable(1)`,
`ase-setup-uninstall(1)`, `ase-setup-status(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-status(1) - Report the ASE Registrations of an Agent Tool

## NAME

`ase-setup-status(1)` - Report the ASE Registrations of an Agent Tool

## SYNOPSIS

`ase setup status`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase setup status** reports where *ASE* is currently registered with
the agent tool selected by `--tool`, rendered as a table with the
columns `KIND`, `ID`, `SCOPE`, and `STATUS`. It requires the CLI of
the agent tool to be found in `$PATH`. The rows are:

- `PLUGIN`:
One row per registration of the plugin- `ase@ase`(as scraped from
the output of- `<cli> plugin list`), with its registration scope and
its state (e.g.,- `enabled`or- `disabled`for- *Anthropic Claude Code
CLI*,- `installed`for- *GitHub Copilot CLI*, and the installation
status of- *OpenAI Codex CLI*), or a single row with the status- `not installed`.
 
- `MCP`:
One row per- *currently registered*server of the pre-defined MCP
servers (see- `ase-setup-mcp(1)`), with its MCP server name and its
registration scope (as reported by- `<cli> mcp get`), and the status- `registered`. Unregistered servers are omitted entirely.
 
- `OUTPUTSTYLE`(- `--tool claude`only):
One row per settings file (of the scopes- `user`,- `project`, and- `local`) carrying an- `outputStyle`entry, with the status- `selected`(the- *ASE*output style- `ase:ase-terse`) or- `foreign`(hand-selected).
 
- `FUNCTIONHOOKS`(- `--tool claude`only):
One row per settings file (of the scopes- `user`,- `project`, and- `local`) carrying a- `CLAUDE_CODE_ENABLE_FUNCTION_HOOKS`variable in
its- `env`, with the status- `enabled`(value- `1`) or- `foreign`(any
other value).
 
- `STATUSLINE`(- `--tool claude`and- `--tool copilot`only):
One row per settings file carrying a- `statusLine`entry, with the
status- `activated`(its command starts with- `ase statusline`, i.e.,
it is owned by- *ASE*) or- `foreign`(hand-crafted).
 
- `TELEMETRY`:
One row per configuration file carrying a telemetry export, with the
status- `activated`(towards the- *ASE*OTLP ingress) or- `foreign`:
for- `claude`, the- `env`of- `~/.claude/settings.json`; for- `codex`,- `$CODEX_HOME/config.toml`; for- `copilot`, the shell profiles- `~/.zshrc`and- `~/.bashrc`(plus the profile of the login shell), and
an- `$OTEL_EXPORTER_OTLP_ENDPOINT`of the environment not pointing to
the- *ASE*ingress.
 

Files below the home directory are shown abbreviated with `~`, and an
unparsable settings file is silently skipped. The command is purely
read-only and always exits with status `0`, so a missing registration
is reported rather than signalled through the exit code.

It takes no `--scope`, since it deliberately reports *all* scopes at
once -- that is exactly what makes an unintentionally leaked
registration visible. Scope information is only available for
`--tool claude` (and, for MCP servers, `--tool copilot`); everywhere
else the scope is reported as `(n/a)`.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`).
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.
 
- `COPILOT_HOME`,- `CODEX_HOME`:
The configuration directories of- *GitHub Copilot CLI*(default:- `~/.copilot`) and- *OpenAI Codex CLI*(default:- `~/.codex`).
 

## EXIT STATUS

`0` in all regular cases, `1` only on an error like an invalid option
value or a missing agent tool CLI.

## EXAMPLES

```
# report the ASE registrations for Anthropic Claude Code CLI
$ ase setup status
# report the ASE registrations for OpenAI Codex CLI
$ ase setup status --tool codex
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-setup-install(1)`, `ase-setup-mcp(1)`,
`ase-setup-statusline(1)`, `ase-setup-telemetry(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-mcp(1) - Manage the Pre-Defined Foreign MCP Servers

## NAME

`ase-setup-mcp(1)` - Manage the Pre-Defined Foreign MCP Servers

## SYNOPSIS

`ase setup mcp`
[`-h`|`--help`]

`ase setup mcp list`
[`-h`|`--help`]

`ase setup mcp activate`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]
[*servers*]

`ase setup mcp deactivate`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]
[*servers*]

## DESCRIPTION

**ase setup mcp** manages the pre-defined *foreign MCP servers* which
*ASE* skills can *optionally* leverage for improved quality: the
*chat* servers (leveraged by the skills `ase-meta-chat` and
`ase-meta-quorum`) and the *search* servers (leveraged by the skills
`ase-meta-search`, `ase-meta-evaluate`, and `ase-arch-discover`).
Without a sub-command, the usage information is shown and the command
exits with status `1`.

The currently pre-defined servers are (by their *id*, followed by the
registered MCP server name):

- `openai-chatgpt`(- `chat-openai-chatgpt`):- *OpenAI ChatGPT*, key- `ASE_MCP_KEY_OPENAI_CHATGPT`or- `ASE_MCP_KEY_OPENROUTER`.
 
- `google-gemini`(- `chat-google-gemini`):- *Google Gemini*, key- `ASE_MCP_KEY_GOOGLE_GEMINI`or- `ASE_MCP_KEY_OPENROUTER`.
 
- `deepseek`(- `chat-deepseek`):- *DeepSeek*, key- `ASE_MCP_KEY_DEEPSEEK`or- `ASE_MCP_KEY_OPENROUTER`.
 
- `xai-grok`(- `chat-xai-grok`):- *xAI Grok*, key- `ASE_MCP_KEY_XAI_GROK`or- `ASE_MCP_KEY_OPENROUTER`.
 
- `alibaba-qwen`(- `chat-alibaba-qwen`):- *Alibaba Qwen*, key- `ASE_MCP_KEY_ALIBABA_QWEN`or- `ASE_MCP_KEY_OPENROUTER`.
 
- `zai-glm`(- `chat-zai-glm`):- *Z.AI GLM*, key- `ASE_MCP_KEY_ZAI_GLM`or- `ASE_MCP_KEY_OPENROUTER`.
 
- `anthropic-claude`(- `chat-anthropic-claude`):- *Anthropic Claude*via the locally installed- *Anthropic Claude Code
CLI*, variable- `ASE_MCP_KEY_ANTHROPIC_CLAUDE`.
 
- `openai-codex`(- `chat-openai-codex`):- *OpenAI Codex*via the locally installed- *OpenAI Codex CLI*,
variable- `ASE_MCP_KEY_OPENAI_CODEX`.
 
- `github-copilot`(- `chat-github-copilot`):- *GitHub Copilot*via the locally installed- *GitHub Copilot CLI*,
variable- `ASE_MCP_KEY_GITHUB_COPILOT`.
 
- `brave`(- `search-brave`):- *Brave*search, key- `ASE_MCP_KEY_BRAVE`.
 
- `perplexity`(- `search-perplexity`):- *Perplexity*search, key- `ASE_MCP_KEY_PERPLEXITY`.
 
- `exa`(- `search-exa`):- *Exa*search, key- `ASE_MCP_KEY_EXA`.
 

The API-based *chat* servers are registered as stdio servers running
`npx -y mcp-to-openai`, which bridges to the OpenAI-compatible API of
the LLM vendor, or, with the key `ASE_MCP_KEY_OPENROUTER` of an
*OpenRouter* account as a common alternative, to the *OpenRouter*
proxy API.

The harness-based *chat* servers (`anthropic-claude`, `openai-codex`,
and `github-copilot`) are registered as stdio servers running
`npx -y mcp-to-harness`, which bridges to the locally installed agent
tool CLI, authenticating via its own configured credentials: their
variable carries no API key at all, but the model identifier of the
harness, with the special value `default` selecting the default model
of the harness.

The *search* servers are registered as stdio servers running
`npx -y @brave/brave-search-mcp-server` resp.
`npx -y @perplexity-ai/mcp-server`, resp. as an HTTP server on
`https://mcp.exa.ai/mcp` for `exa`.

## COMMANDS

- `ase setup mcp list`:
List all pre-defined MCP servers known to- *ASE*, rendered as a
table with the columns- `ID`(the server id),- `NAME`(the service
name),- `VERS`(the pinned model version, or- `latest`),- `MCP`(the
registered MCP server name),- `KEY`(the accepted suffixes- *KEY*of
the environment variables- `ASE_MCP_KEY_`- *KEY*, in order of
preference), and- `SKILLS`(the- *ASE*skills leveraging the server).
 - It accepts no options besides - `-h`|- `--help`and does not require any
agent tool.
 
- `ase setup mcp activate`[- `-t`|- `--tool`- *tool*] [- `-s`|- `--scope`- *scope*] [- *servers*]:
Register one or more pre-defined MCP servers with the agent tool
(via- `<cli> mcp add`). The environment variables- `ASE_MCP_KEY_`- *KEY*are first additionally sourced from a- `.env`file in the current
directory (if existing). Each selected server takes its key from the
first of its variables (see- `KEY`of- `ase setup mcp list`) which is
set and non-empty.
 - A server none of whose variables is set is silently skipped when it
was selected implicitly (via an empty - *servers*or- `all`), but causes
a hard error when it was given explicitly. A prior registration of
the same server is removed first, so it is re-created cleanly.
 
- `ase setup mcp deactivate`[- `-t`|- `--tool`- *tool*] [- `-s`|- `--scope`- *scope*] [- *servers*]:
Unregister one or more pre-defined MCP servers from the agent tool
(via- `<cli> mcp remove`). A server which is not currently
registered is skipped. No API keys are required.
 

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*] (- `activate`and- `deactivate`only):
Select the target agent tool:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`). Its CLI has to be found in- `$PATH`.
 
- [- `-s`|- `--scope`- *scope*] (- `activate`and- `deactivate`only):
Select the MCP server registration scope:- `user`(default;
registered globally for the current user),- `project`(registered in
the project and meant to be shared via version control), or- `local`(registered in the project, but kept out of version control).
 - The scope is always passed explicitly to - *Anthropic Claude Code CLI*,
as it would otherwise default to- `local`. Only supported for- `--tool claude`; a non-- `user`scope for- `copilot`or- `codex`fails
with an error.
 

## ARGUMENTS

- [*servers*] (`activate`and`deactivate`only):
A comma-separated list of server ids (see above). When omitted,
empty, or given as the literal`all`, every pre-defined server is
selected. An unknown server id fails with an error listing all
known ids.

## ENVIRONMENT

- `ASE_MCP_KEY_`- *KEY*:
The API key (resp., for the harness-based servers, the harness model
identifier or- `default`) of the pre-defined MCP servers, where- *KEY*is one of the suffixes shown in the- `KEY`column of- `ase setup mcp list`, e.g.,- `ASE_MCP_KEY_OPENAI_CHATGPT`for- `openai-chatgpt`. The
key is passed to the registered server (e.g., as its environment
variable- `OPENAI_KEY`,- `BRAVE_API_KEY`, or- `PERPLEXITY_API_KEY`) and
masked in the logged commands.
 
- `ASE_MCP_KEY_OPENROUTER`:
The- *OpenRouter*API key, accepted by all API-based- *chat*servers
as an alternative to the native API key of the LLM vendor.
 
- `ASE_TOOL`:
The default of- `--tool`.
 

## FILES

- `.env`:
An optional file in the current directory, from which the- `ASE_MCP_KEY_`- *KEY*variables are additionally sourced on- `ase setup mcp activate`.

## EXIT STATUS

`0` on success, `1` on any error (and when invoked without a
sub-command).

## EXAMPLES

```
# list all pre-defined MCP servers
$ ase setup mcp list
# activate all servers whose keys are available
$ ase setup mcp activate
# activate two search servers for the current project only
$ ASE_MCP_KEY_BRAVE=... ASE_MCP_KEY_EXA=... \
  ase setup mcp activate --scope project brave,exa
# activate the harness-based Anthropic Claude server with its default model
$ ASE_MCP_KEY_ANTHROPIC_CLAUDE=default ase setup mcp activate anthropic-claude
# deactivate all servers in GitHub Copilot CLI
$ ase setup mcp deactivate --tool copilot all
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-setup-status(1)`, `npx(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-statusline(1) - Activate or Deactivate the ASE Statusline

## NAME

`ase-setup-statusline(1)` - Activate or Deactivate the ASE Statusline

## SYNOPSIS

`ase setup statusline`
[`-h`|`--help`]

`ase setup statusline activate`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]
[`-w`|`--width` *n*]
[`-m`|`--margin` *n*]
[`-p`|`--padding` *n*]
[`--no-icons`]
[`--no-labels`]
[*format* [...]]

`ase setup statusline deactivate`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-s`|`--scope` *scope*]

## DESCRIPTION

**ase setup statusline** activates or deactivates the *ASE*
statusline, i.e., the `ase statusline` renderer (see
`ase-statusline(1)`) configured as the `statusLine` command in the
settings file of the agent tool. Without a sub-command, the usage
information is shown and the command exits with status `1`.

The statusline is supported for *Anthropic Claude Code CLI* (`claude`)
and *GitHub Copilot CLI* (`copilot`) only, while *OpenAI Codex CLI*
(`codex`) has no scriptable statusline mechanism, so `--tool codex`
fails with an error.

The settings file is, for `claude`, `~/.claude/settings.json` (scope
`user`), `.claude/settings.json` (scope `project`), resp.
`.claude/settings.local.json` (scope `local`) relative to the current
working directory, and, for `copilot`, the single per-user
`$COPILOT_HOME/settings.json` (default: `~/.copilot/settings.json`).
The settings file is edited in place (preserving its formatting and
all other entries) and written atomically, and a missing settings file
(and its directory) is created.

A `statusLine` entry counts as owned by *ASE* if its `command` starts
with `ase statusline`. A foreign, hand-crafted `statusLine` entry is
never touched by either sub-command: it is preserved with a warning,
and the command still exits with status `0`.

## COMMANDS

- `ase setup statusline activate`[- *options*] [- *format*[...]]:
Add the- `statusLine`entry of- *ASE*to the settings file, or update
an existing one of- *ASE*in place.
 - The entry is the object
- `{ "type": "command", "command": "ase statusline ...", "padding": 0 }`,
whose command is- `ase statusline --tool`- *tool*- `-w`- *width*- `-m`- *margin*- `-p`- *padding*, followed by- `--no-icons`and- `--no-labels`if given, and the- *format*lines, each one single-quoted for the
shell (resp. double-quoted for- `copilot`on Windows, which spawns
the command via- `cmd.exe`).
 
- `ase setup statusline deactivate`[- *options*]:
Remove the- `statusLine`entry of- *ASE*from the settings file. A
missing settings file or a missing- `statusLine`entry is skipped
with an informational message.
 

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`or- `copilot`(default:- `$ASE_TOOL`, else- `claude`).- `codex`is rejected with an error.
 
- [- `-s`|- `--scope`- *scope*]:
Select the settings file of the scope- `user`(default),- `project`,
or- `local`(see above). Only supported for- `--tool claude`; a
non-- `user`scope for- `copilot`, which has a single per-user settings
file, fails with an error.
 
- [- `-w`|- `--width`- *n*] (- `activate`only):
Force the terminal width of the statusline rendering to- *n*characters, where- `0`(default) auto-detects it via- `/dev/tty`.
Has to be a non-negative integer.
 
- [- `-m`|- `--margin`- *n*] (- `activate`only):
Reduce the maximum used terminal width by- *n*characters on each
side (default:- `2`). Has to be a non-negative integer.
 
- [- `-p`|- `--padding`- *n*] (- `activate`only):
Pad each rendered line with- *n*spaces on the left and right side
(default:- `0`). Has to be a non-negative integer.
 
- [- `--no-icons`] (- `activate`only):
Disable the leading icon glyph in front of each placeholder
rendering.
 
- [- `--no-labels`] (- `activate`only):
Disable the textual label in front of the bold value of each
placeholder rendering.
 

## ARGUMENTS

- [*format*[...]] (`activate`only):
One or more statusline template lines of literal text,`%`-prefixed placeholders, and`<`*color*`>`...`</`*color*`>`markup (see`ase-statusline(1)`for all placeholders and colors).
Without any*format*, the following three default lines are used:`<blue>%u</blue> <red>%p</red> <black>%T</black> %s`(user, project,
task, and session),`%m %e %t`(model, effort, and thinking), and`%P %h %B %c`(persona, guidance level, boxing, and context usage).

## ENVIRONMENT

## FILES

- `~/.claude/settings.json`,- `.claude/settings.json`,- `.claude/settings.local.json`:
The settings files of- *Anthropic Claude Code CLI*of the scopes- `user`,- `project`, and- `local`.
 
- `$COPILOT_HOME/settings.json`(default:- `~/.copilot/settings.json`):
The settings file of- *GitHub Copilot CLI*.
 

## EXIT STATUS

`0` on success (including a preserved foreign `statusLine` entry),
`1` on any error (and when invoked without a sub-command).

## EXAMPLES

```
# activate the ASE statusline with its default lines
$ ase setup statusline activate
# activate a single custom line for the current project only
$ ase setup statusline activate --scope project "%p %T %m %c"
# activate the ASE statusline in GitHub Copilot CLI without icons
$ ase setup statusline activate --tool copilot --no-icons
# deactivate the ASE statusline again
$ ase setup statusline deactivate
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-statusline(1)`, `ase-setup-status(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-setup-telemetry(1) - Activate or Deactivate the ASE Telemetry Export

## NAME

`ase-setup-telemetry(1)` - Activate or Deactivate the ASE Telemetry Export

## SYNOPSIS

`ase setup telemetry`
[`-h`|`--help`]

`ase setup telemetry activate`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

`ase setup telemetry deactivate`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase setup telemetry** activates or deactivates the *OpenTelemetry*
(OTLP/HTTP) export of the agent tool towards the per-user *ASE* OTLP
ingress `http://127.0.0.1:`*port* (with *port* from the configuration
`telemetry.service.port`, default `44318`), which feeds the AI usage
accounting of the task assignments (see `ase-task-assign(1)`).
Without a sub-command, the usage information is shown and the command
exits with status `1`.

The export is always user-level, as the agent tools ignore these
settings on repository level, so a single ingress serves the sessions
of all projects: every running `ase service` tries to hold its port,
and the holder routes the records by their session to the service of
the owning project. Hence, there is no `--scope` option.

Telemetry configured by the user for other receivers is never touched:
it is preserved with a warning (and the command still exits with
status `0`), and reported as `foreign` by `ase setup status`.

## COMMANDS

- `ase setup telemetry activate`[- `-t`|- `--tool`- *tool*]:
Configure the telemetry export of the agent tool. As the AI usage
accounting is opt-in, this requires the configuration- `telemetry.enabled`to be- `true`(e.g., via- `ase config set telemetry.enabled true`on scope- `user`or- `project`), else the
command fails with an error. A missing bearer token- `telemetry.service.token`is generated (48 hexadecimal digits) and
persisted on scope- `user`. Per agent tool:
 - - `claude`:
The variables- `CLAUDE_CODE_ENABLE_TELEMETRY=1`,- `OTEL_LOGS_EXPORTER=otlp`,- `OTEL_EXPORTER_OTLP_PROTOCOL=http/json`,- `OTEL_EXPORTER_OTLP_ENDPOINT=`- *url*,- `OTEL_EXPORTER_OTLP_HEADERS=Authorization=Bearer `- *token*, and- `OTEL_LOGS_EXPORT_INTERVAL=5000`are added to (resp. updated in)
the- `env`object of- `~/.claude/settings.json`.
 - If any of these variables is already set there, but points
neither to the URL nor carries the token of the - *ASE*ingress, the
configuration is foreign and preserved; a non-object- `env`is
preserved, too.
 
- `codex`:
A marked block (between the lines- `# BEGIN ASE telemetry (generated)`and- `# END ASE telemetry (generated)`) with an- `[otel]`table, exporting via- `otlp-http`with protocol- `json`to- *url*- `/v1/logs`with the- `authorization`header- `Bearer`- *token*, is added to (resp. replaced in)- `$CODEX_HOME/config.toml`(default:- `~/.codex/config.toml`). An- `[otel]`table outside of the block is foreign and preserved.
 
- `copilot`:
As- *GitHub Copilot CLI*is configurable via environment
variables only, a marked block with a shell function- `copilot`is added to (resp. replaced in) the shell profile of the login
shell of- `$SHELL`(- `$ZDOTDIR/.zshrc`, default- `~/.zshrc`, for- `zsh`, and- `~/.bashrc`for- `bash`), which passes the variables- `COPILOT_OTEL_ENABLED=true`,- `COPILOT_OTEL_EXPORTER_TYPE=otlp-http`,- `OTEL_EXPORTER_OTLP_ENDPOINT=`- *url*,- `OTEL_EXPORTER_OTLP_PROTOCOL=http/json`, and- `OTEL_EXPORTER_OTLP_HEADERS=Authorization=Bearer `- *token*to the- `copilot`command only, instead of exporting them to every other
program. A new shell session is required for it to take effect.
 - For any other shell, the command fails with an error showing the
shell function to define manually. An - `$OTEL_EXPORTER_OTLP_ENDPOINT`of the environment other than the- *ASE*ingress URL, and an
assignment of- `COPILOT_OTEL_ENABLED`or- `OTEL_EXPORTER_OTLP_ENDPOINT`in the profile outside of the block, are foreign and preserved.
 
 
- `ase setup telemetry deactivate`[- `-t`|- `--tool`- *tool*]:
Remove the telemetry export configured by- `ase setup telemetry activate`again, preserving any foreign telemetry configuration. It
does not require- `telemetry.enabled`.
 - Per agent tool: for - `claude`, the above variables are removed from the- `env`of- `~/.claude/settings.json`(and the- `env`itself, if it became
empty); for- `codex`, the marked block is removed from- `$CODEX_HOME/config.toml`; and for- `copilot`, the marked block is
removed from all candidate shell profiles (- `~/.zshrc`,- `~/.bashrc`,
and the profile of the login shell). A missing file or configuration
is skipped with an informational message.
 

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the target agent tool:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`).
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.
 
- `CODEX_HOME`:
The configuration directory of- *OpenAI Codex CLI*(default:- `~/.codex`).
 
- `SHELL`,- `ZDOTDIR`:
The login shell, determining the shell profile for- `--tool copilot`,
and the configuration directory of- `zsh`.
 
- `OTEL_EXPORTER_OTLP_ENDPOINT`:
An existing OTLP endpoint for- `--tool copilot`, which, if not the- *ASE*ingress, is considered a foreign telemetry configuration.
 

## FILES

- `~/.claude/settings.json`:
The user-level settings file of- *Anthropic Claude Code CLI*.
 
- `$CODEX_HOME/config.toml`(default:- `~/.codex/config.toml`):
The configuration file of- *OpenAI Codex CLI*.
 
- `~/.zshrc`,- `~/.bashrc`:
The shell profiles receiving the- `copilot`shell function.
 

## EXIT STATUS

`0` on success (including a preserved foreign configuration), `1` on
any error (e.g., a disabled AI usage accounting on `activate`, an
unsupported shell for `--tool copilot`, an unterminated marked block,
and when invoked without a sub-command).

## EXAMPLES

```
# enable the AI usage accounting and activate the export of Claude Code
$ ase config --scope user set telemetry.enabled true
$ ase setup telemetry activate
# activate the export of OpenAI Codex CLI
$ ase setup telemetry activate --tool codex
# deactivate the export of GitHub Copilot CLI
$ ase setup telemetry deactivate --tool copilot
```

## SEE ALSO

`ase(1)`, `ase-setup(1)`, `ase-setup-status(1)`, `ase-config(1)`,
`ase-service(1)`, `ase-task-assign(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-config(1) - Manage the Layered ASE Configuration

## NAME

`ase-config(1)` - Manage the Layered ASE Configuration

## SYNOPSIS

`ase config`
[`-h`|`--help`]
[`--scope` *scope*]
*command*
[*args* [...]]

`ase config` [`--scope` *scope*] `init` *type*

`ase config` [`--scope` *scope*] `list`

`ase config` [`--scope` *scope*] `edit`

`ase config` [`--scope` *scope*] `get` *key*

`ase config` [`--scope` *scope*] `set` *key* *value*

`ase config` [`--scope` *scope*] `delete` *key*

## DESCRIPTION

**ase config** manages the layered *ASE* configuration, a stack of
`config.yaml` files of the *scopes* `user` < `project` < `task:`*id* <
`session:`*id*, on top of a built-in `default` scope carrying the
default values. Reads cascade from the strongest scope down to the
weakest one and return the first defined value, while writes are
always confined to the file of the strongest scope of the chain (the
*target* scope).

All files are validated against the configuration schema: on read,
unknown or invalid entries are warned about and dropped from the
in-memory view, while on set, delete, and write they lead to a fatal
error. A target file still carrying invalid content is never
overwritten, as the skipped content would otherwise be lost: repair
it first (e.g. via `ase config edit`).

Obsolete (pre-1.1.0) entries `project.artifact.task.{basedir,files}`
and `agent.{task,skill}` are migrated in place on reading: a `basedir`
becomes `project.task.store.url` (as `ase:`*basedir*) unless the
latter is already set, and the other entries are dropped.

A *key* is a dotted path like `agent.persona`. Any unambiguous
trailing part of a known key is accepted, too, e.g., `persona` for
`agent.persona` or `store.url` for `project.task.store.url`, while an
ambiguous one (like `color.dim`) is an error.

Without a *command*, the usage information is shown and the command
exits with status `1`.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--scope`- *scope*]:
Select the scope chain. This option belongs to- `ase config`itself
and hence has to be given- *before*the- *command*, e.g.,- `ase config --scope user set agent.persona writer`.
 - The - *scope*value is a comma-separated list of scope terms, in any
order, where each term is one of- `user`,- `project`,- `task:`- *id*, or- `session:`- *id*(with- *id*matching- `[A-Za-z0-9._-]+`, but neither- `.`nor- `..`). At most one term per kind is allowed. The chain is
canonicalized into the fixed inheritance order- `default`<- `user`<- `project`<- `task`<- `session`:
 - - `user`is always implicitly added at the bottom of the chain.
- `project`is implicitly added only when a- *project context*exists, i.e., when the current working directory is inside a Git
working tree or a- `.ase`directory exists at or above it (except
in the home directory), and only as long as it stays weaker than
the strongest explicitly requested term, so an explicit- `--scope user`is never silently retargeted to- `project`.
- Specifying `project`explicitly without a project context is an
error.
 - Without - `--scope`, the chain is- `project`(with- `user`below it) when
a project context exists, otherwise- `user`only. Example:- `--scope task:T1,session:S1`yields the chain- `default`->- `user`->- `project`->- `task:T1`->- `session:S1`with- `session:S1`as the target scope.
 

## COMMANDS

- `init`- *type*:
Initialize the target scope with the preset values of- *type*(- `default`,- `vibe`,- `pro`, or- `industry`). See- `ase-config-init(1)`.
 
- `list`:
List all configuration keys with their effective values and the
scopes supplying them. See- `ase-config-list(1)`.
 
- `edit`:
Edit the configuration file of the target scope with the text editor.
See- `ase-config-edit(1)`.
 
- `get`- *key*:
Print the effective value of- *key*. See- `ase-config-get(1)`.
 
- `set`- *key*- *value*:
Set- *key*to- *value*on the target scope. See- `ase-config-set(1)`.
 
- `delete`- *key*:
Delete- *key*from the target scope. See- `ase-config-delete(1)`.
 

## CONFIGURATION KEYS

The following keys are recognized. Every key is restricted to the
listed values (where *string* means any non-empty string) and is
writable on all scopes `user`, `project`, `task`, and `session`,
unless stated otherwise. A default stems from the built-in `default`
scope (shown by `ase config list` with scope `default`) or, where
stated, from the code using the key.

Project identity and classification:

- `project.id`(- *string*):
The unique id of the project. Default: the sanitized basename of the
project root (with all characters other than- `A-Z`,- `a-z`,- `0-9`,- `_`, and- `-`replaced by- `_`), or- *basedir*- `-`- *worktree*if the
project root is a worktree- *basedir*- `/`- *worktree*next to an- `active`symbolic link in- *basedir*. An explicit id is required for a remote
task store and for the agent relay.
 
- `project.name`(- *string*):
The full name of the project (default:- `Example Project`).
 
- `project.boxing`(- `white`,- `grey`, or- `black`):
Whether the project artifacts are treated as a white box (fully
transparent and understood), grey box (partially), or black box (not
at all), which modulates the work depth and output visibility of the
skills (default:- `white`).
 
- `project.users`(- *string*,- `user`and- `project`scopes only):
The comma-separated list of the known human assignees of the tasks
(default: empty). If not empty, an assignment to anybody else than- `none`, the current user, one of these users, or an agent present at
the agent relay is warned about (see- `ase-task-assign(1)`), and the
assign popup of the task boards suggests these users (else the
current user) besides the agents (see- `ase-task-board(1)`).
 

Task plans (all writable on the `user` and `project` scopes only,
`project.task.store.token` on the `user` scope only):

- `project.task.lifecycle`(- `solo`,- `team`, or- `enterprise`):
The task lifecycle model: single-phase for a local solo developer,
two-phase for a small team, or four-phase for a gated enterprise
pipeline (default:- `solo`). See- `ase-task-lifecycle(1)`.
 
- `project.task.idscheme`(- `slug`[- `:`- *words*],- `seq`[- `:`- *template*],
or- `any`):
The task id scheme (default:- `slug`). See- `ase-task-idscheme(1)`and- `ase-task-newid(1)`.
 
- `project.task.store.url`(- *string*):
The URL of the task store:- `ase:`- *path*(local, default- `ase:./.ase/task`),- `ase://`- *addr*- `:`- *port*[- `/`- *token*] resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] (remote task
store server),- `github:`- *owner*- `/`- *repo*(and- `github+http`[- `s`]- `://`- *host*- `/`- *owner*- `/`- *repo*),- `gitlab:`- *namespace*- `/`- *project*(and- `gitlab://`- *host*- `/`...,- `gitlab+http`[- `s`]- `://`- *host*- `/`...), or- `gitea+http`[- `s`]- `://`- *host*- `/`- *owner*- `/`- *repo*. See- `ase-task(1)`.
 
- `project.task.store.token`(- *string*,- `user`scope only):
The bearer token of a remote task store, resp. the token of a GitHub,
GitLab, or Gitea task store. Masked as- `***`by- `ase config list`.
 
- `project.task.store.crt`,- `project.task.store.key`(- *string*):
The PEM certificate (chain) and private key files of the task store
server, relative to the directory of the configuration file defining
them. See- `ase-task-store(1)`.
 
- `project.task.default.source`(- `worktree`or- `branch:`- *name*):
The default of the- `Source:`key of the task plans (default:- `worktree`).
 
- `project.task.default.changeset`(- `worktree`,- `branch:`- *name*, or- `attachment:`- *name*):
The default of the- `Changeset:`key of the task plans (default:- `worktree`).
 
- `project.task.default.target`(- `worktree`,- `branch:`- *name*, or- `source`):
The default of the- `Target:`key of the task plans (default:- `source`).
 - A branch - *name*has to start with a letter, digit, or- `_`, consist of
letters, digits,- `_`,- `.`,- `/`, and- `-`, must not end in- `/`,- `.`, or- `.lock`, and must not contain- `..`or- `//`. An attachment- *name*consists of letters, digits,- `_`, and- `-`. A branch or attachment- *name*can carry the placeholder- `{id}`, which reads as the id of the
task (with every character other than letters, digits,- `_`, and- `-`replaced by- `-`, runs of- `-`reduced to one, and leading and trailing- `-`and- `_`removed, e.g.,- `12`for- `#12`and- `FOO-12`for- `FOO#12`),
e.g.,- `branch:cs-{id}`for a
separate changeset branch per task.
 
- `project.task.effort.scale`(- *xs*- `,`- *s*- `,`- *m*- `,`- *l*):
The upper bounds of the effort size bands- `XS`,- `S`,- `M`, and- `L`of
the- `Effort:`key of the task plans, in strictly ascending human
minutes, while- `XL`is open-ended (default:- `15,60,240,960`). The
task boards sum the efforts via the band midpoints, counting- `XL`as
twice the- `L`bound.
 
- `project.task.effort.factor`(- *number*):
The human minutes per point of the- *attention score*of the effort
estimation, i.e., of the weighted requirements, behaviors, and
changes stated by the checklist items of a plan (default:- `1`).
Fitted and stored by- `/ase-task-estimate --calibrate --save`.
 
- `project.task.effort.file`,- `project.task.effort.line`(- *number*):
The additional human minutes per touched file resp. per predicted
changed line of the- *volume score*of the effort estimation, where
the line minutes also cover the edge cases and polishing no plan
describes, and a newly written line weighs three times a changed
one (defaults:- `1`resp.- `0.025`). Fitted and stored by- `/ase-task-estimate --calibrate --save`.
 
- `project.task.effort.iteration`(- *number*):
The multiplier of the estimated minutes of a task whose main
deliverable is an interactive or visual user interface, covering
its many rounds of post-adjustment, at least- `1`(default:- `2.5`).
Not fitted by the calibration.
 

Agent relay (all writable on the `user` and `project` scopes only,
`project.relay.token` on the `user` scope only):

- `project.relay.url`(- *string*):
The URL of the agent relay, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
or- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] (default:
none). See- `ase-agent-relay(1)`.
 
- `project.relay.token`(- *string*,- `user`scope only):
The bearer token of the agent relay. Masked as- `***`by- `ase config list`.
 
- `project.relay.crt`,- `project.relay.key`(- *string*):
The PEM certificate (chain) and private key files of the agent relay
server, relative to the directory of the configuration file defining
them.
 

Per-project service:

- `project.service.port`(- *port*- `1024`..- `65535`,- `project`scope
only):
The preferred port of the per-project background service on- `127.0.0.1`, falling back to a random port out of the range- `42000`..- `44000`if unavailable (default: none). See- `ase-service-start(1)`.

Project artifacts (all writable on the `user` and `project` scopes
only), each kind defined by a `basedir` directory (relative to the
project root, empty for the root itself) and a whitespace-separated
`files` glob specification (relative to `basedir`, with `!` negating a
glob). See `ase-artifact(1)`:

- `project.artifact.spec.basedir`,- `project.artifact.spec.files`:
The specification files (default:- `docs/specbook`and- `**/*.{md,txt,svg,png,jpg}`).
 
- `project.artifact.spec.schema`(- *string*):
The whitespace-separated list of- *SpecBook*schema configuration
files, where- `std`,- `meta`, and- `pm`name the bundled standard, meta,
and project management ones,- `npm:`,- `github:`,- `http:`, and- `https:`references name remote ones, and every other entry is a file
or glob pattern relative to the project root (default: empty, i.e.,- `std`). See- `ase-spec(1)`.
 
- `project.artifact.code.basedir`,- `project.artifact.code.files`:
The source code files (default:- `src`and- `** !**/etc/** !**/{.gitignore,.npmignore,package.json}`).
 
- `project.artifact.docs.basedir`,- `project.artifact.docs.files`:
The documentation files (default:- `doc`and- `** **/{README,LICENSE,CHANGELOG}.{md,txt} !{spec,arch}/**`).
 
- `project.artifact.infr.basedir`,- `project.artifact.infr.files`:
The infrastructure files (default: empty and- `**/{.github,.claude*,etc}/** **/{AGENTS.md,{package,tsconfig*}.json,.{git,npm}ignore}`).
 

Agent:

- `agent.persona`(- `writer`,- `engineer`,- `journalist`,- `telegrapher`,
or- `caveman`):
The communication style of the AI agent (default:- `engineer`).
 
- `agent.guidance`(- `none`,- `minimal`,- `normal`, or- `verbose`):
The amount of unsolicited help hints of the AI agent (default:- `normal`).
 
- `agent.diagram.format`(- `image`or- `text`):
The rendering of the diagrams of the skills: as an inline image
where the terminal supports it, else as text, resp. always as text
(default:- `image`).
 
- `agent.diagram.theme`(- `light`or- `dark`):
The theme style of the diagram images: dark node boxes with light
labels on a light background, resp. vice versa (default:- `light`).
 
- `agent.diagram.tone`(- `#`- *rgb*[- `#`- *rgb*]):
The theme color tone of the diagram images, as one or two colors
(- `#rgb`or- `#rrggbb`), from which the color spreads are derived,
where a missing second one is derived as the complement of the
first one (default:- `#336699`, a blue with a derived brown).
 

AI usage accounting (see `ase-task-assign(1)` and
`ase-setup-telemetry(1)`):

- `telemetry.enabled`(- `true`or- `false`,- `user`and- `project`scopes
only):
Whether the AI usage accounting is enabled as a whole (default:- `false`).
 
- `telemetry.implicit`(- `true`or- `false`,- `user`and- `project`scopes
only):
Whether the usage of an interactive session is also accounted onto
its active task while the task is- *unassigned*, i.e., without
assigning it to oneself (default:- `true`, never for task- `default`).
 
- `telemetry.service.port`(- *port*- `1024`..- `65535`,- `user`scope only):
The port of the per-user OTLP ingress on- `127.0.0.1`(default:- `44318`).
 
- `telemetry.service.token`(- *string*,- `user`scope only):
The bearer token of the OTLP ingress, generated by- `ase setup telemetry activate`. Masked as- `***`by- `ase config list`.
 
- `telemetry.service.interval`(- *minutes*,- `user`and- `project`scopes
only):
The interval of the interim bookings of the open usage windows
(default:- `10`,- `0`disables).
 
- `telemetry.service.idle`(- *minutes*,- `user`and- `project`scopes
only):
The idle time after which a usage window pauses (default:- `30`,- `0`disables).
 
- `telemetry.prices.ai`(- *table*,- `user`scope only):
The price table- *model*- `=`- *input*- `/`- *output*[- `/`- *cread*- `/`- *cwrite*]
[...] (whitespace-separated) in USD per million input, output,
cache read, and cache write tokens (the cache prices defaulting to the
input price), keyed by the model or the tool name, with- `*`for all
others (default:- `*=4.00/20.00/0.20/5.00`).
 
- `telemetry.prices.hr`(- *table*,- `user`and- `project`scopes only):
The hourly rate table- *user-id*- `=`- *usd-per-hour*[- `,`...] of the
human time, keyed by the assignee, with- `*`for all others (default:- `*=110`).
 

Task board (all writable on the `user` and `project` scopes only, see
`ase-task-board(1)`):

- `board.tui.color.dim`,- `board.tui.color.normal`,- `board.tui.color.accent`,- `board.tui.color.signal`:
The colors of the four color roles of the terminal board (defaults:- `grey`,- `default`,- `blue`, and- `red`), each either- `default`(the
foreground color of the terminal), a color name (like- `blue`or- `blueBright`),- `#rgb`,- `#rrggbb`,- `ansi256(`- *n*- `)`, or- `rgb(`- *r*- `,`- *g*- `,`- *b*- `)`.
 
- `board.web.color.dim`,- `board.web.color.normal`,- `board.web.color.accent`,- `board.web.color.signal`:
The base colors of the four color roles of the web board (defaults:- `grey`,- `black`,- `blue`, and- `orange`), each either- `default`, a
color name (- `black`,- `grey`,- `gray`,- `brown`,- `red`,- `orange`,- `yellow`,- `green`,- `teal`,- `cyan`,- `blue`,- `purple`, or- `magenta`),- `#rgb`, or- `#rrggbb`. Each base color is expanded into a spread of 64
colors over the entire lightness range, out of which the web board
takes all its colors.
 
- `board.web.editor.keymap`(- `default`,- `vim`, or- `emacs`):
The key bindings of the task plan editor of the web board (default:- `default`).
 
- `board.web.theme`(- `auto`,- `light`, or- `dark`):
The color theme of the web board, where- `auto`follows the light or
dark preference of the operating system resp. browser (default:- `auto`).
 

## ENVIRONMENT

- `EDITOR`,- `VISUAL`:
The text editor of- `ase config edit`.
 
- `XDG_CONFIG_HOME`,- `XDG_STATE_HOME`,- `APPDATA`,- `LOCALAPPDATA`:
The base directories of the- *per-user configuration directory*and
the- *per-user state directory*(see- **FILES**).
 

## FILES

- *per-user configuration directory*- `/config.yaml`:
The configuration of scope- `user`(with file mode- `0600`). The
per-user configuration directory is- `~/Library/Application Support/ase`on macOS,- `%APPDATA%\ase`on Windows, and- `$XDG_CONFIG_HOME/ase`(falling back to- `~/.config/ase`) elsewhere.
 
- `.ase/config.yaml`:
The configuration of scope- `project`, searched upward from the
current working directory up to the project root, else located in the
project root.
 
- `.ase/task/`- *id*- `/config.yaml`:
The configuration of scope- `task:`- *id*, located in the project root
(or the current working directory outside of a project).
 
- *per-user state directory*- `/session/`- *id*- `/config.yaml`:
The configuration of scope- `session:`- *id*, independent of any
project. The per-user state directory is- `~/Library/Application Support/ase`on macOS,- `%LOCALAPPDATA%\ase`on Windows, and- `$XDG_STATE_HOME/ase`(falling back to- `~/.local/state/ase`)
elsewhere.
 

Every write happens under an advisory file lock and atomically. A file
carrying a secret key (a token) is written with file mode `0600`.

## EXIT STATUS

`0` on success, `1` on any error (including a missing *command*).

## EXAMPLES

```
# initialize the project configuration with the "industry" preset
$ ase config init industry
# show the effective configuration
$ ase config list
# set the persona of the agent on the user scope
$ ase config --scope user set agent.persona telegrapher
# read a value by an abbreviated key
$ ase config get persona
```

## SEE ALSO

`ase(1)`, `ase-config-init(1)`, `ase-config-list(1)`,
`ase-config-edit(1)`, `ase-config-get(1)`, `ase-config-set(1)`,
`ase-config-delete(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-config-init(1) - Initialize Configuration with Preset Values

## NAME

`ase-config-init(1)` - Initialize Configuration with Preset Values

## SYNOPSIS

`ase config`
[`--scope` *scope*]
`init`
[`-h`|`--help`]
*type*

## DESCRIPTION

**ase config init** initializes the configuration file of the target
scope (the strongest scope of the scope chain, by default `project`
inside a project, else `user`) with the preset values of *type*. The
file (and its parent directories) are created if missing. Already
existing keys of the file which the preset does not cover are kept,
while those covered by the preset are overwritten.

Keys of the preset which are not writable on the target scope (e.g.,
all `project.task.*`, `project.artifact.*`, and `board.*` keys on a
`task` or `session` scope, or `telemetry.prices.ai` on the `project`
scope) are silently skipped. The file is written under an advisory
file lock and atomically.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--scope`- *scope*]:
Select the scope chain, whose strongest scope is the target scope
written by this command. As an option of- `ase config`, it has to be
given- *before*- `init`. The- *scope*value is a comma-separated list
of the terms- `user`,- `project`,- `task:`- *id*, and- `session:`- *id*(with- *id*matching- `[A-Za-z0-9._-]+`), at most one per kind and in
any order, canonicalized into the order- `user`<- `project`<- `task`<- `session`.
 - `user`is always added implicitly, and- `project`is added implicitly
whenever a project context (a Git working tree or a- `.ase`directory
at or above the current directory) exists, but never above the
strongest explicitly requested term. An explicit- `project`without a
project context is an error. Default:- `project`inside a project
context, else- `user`. See- `ase-config(1)`for details.
 

## ARGUMENTS

## FILES

The configuration file of the target scope: *per-user configuration
directory*`/config.yaml` (`user`), `.ase/config.yaml` (`project`),
`.ase/task/`*id*`/config.yaml` (`task:`*id*), or *per-user state
directory*`/session/`*id*`/config.yaml` (`session:`*id*). See
`ase-config(1)`.

## EXIT STATUS

`0` on success, `1` on an unknown *type*, an invalid *scope*, or a
target file carrying invalid content.

## EXAMPLES

```
# initialize the project configuration for an industrial project
$ ase config init industry
# write the entire baseline into the user configuration
$ ase config --scope user init default
```

## SEE ALSO

`ase(1)`, `ase-config(1)`, `ase-config-list(1)`, `ase-config-set(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-config-list(1) - List the Effective Configuration

## NAME

`ase-config-list(1)` - List the Effective Configuration

## SYNOPSIS

`ase config`
[`--scope` *scope*]
`list`
[`-h`|`--help`]

## DESCRIPTION

**ase config list** lists all configuration keys known to the
configuration schema, plus all further leaf keys found in the files of
the scope chain, with their effective values, rendered as a table with
the three columns `KEY`, `VALUE`, and `SCOPE`, sorted by key.

The `SCOPE` column names the scope which supplied the value: `default`
(the built-in defaults), `user`, `project`, `task:`*id*, or
`session:`*id*. For a key set on multiple scopes, only the value of the
strongest scope is shown. A key which is not set on any scope is shown
with an empty `VALUE` and the `SCOPE` `-`. The values of the secret keys
`project.task.store.token`, `project.relay.token`, and
`telemetry.service.token` are masked as `***`.

Invalid entries of the configuration files are warned about and
skipped.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--scope`- *scope*]:
Select the scope chain whose values are cascaded. As an option of- `ase config`, it has to be given- *before*- `list`. The- *scope*value
is a comma-separated list of the terms- `user`,- `project`,- `task:`- *id*,
and- `session:`- *id*(with- *id*matching- `[A-Za-z0-9._-]+`), at most
one per kind and in any order, canonicalized into the order- `user`<- `project`<- `task`<- `session`.
 - `user`is always added implicitly, and- `project`is added implicitly
whenever a project context (a Git working tree or a- `.ase`directory
at or above the current directory) exists, but never above the
strongest explicitly requested term. An explicit- `project`without a
project context is an error. Default:- `project`inside a project
context, else- `user`. See- `ase-config(1)`for details.
 

## EXIT STATUS

`0` on success, `1` on an invalid *scope*.

## EXAMPLES

```
# list the effective project configuration
$ ase config list
# list the configuration as seen by a particular task
$ ase config --scope task:json-export list
```

## SEE ALSO

`ase(1)`, `ase-config(1)`, `ase-config-get(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-config-edit(1) - Edit the Configuration File with an Editor

## NAME

`ase-config-edit(1)` - Edit the Configuration File with an Editor

## SYNOPSIS

`ase config`
[`--scope` *scope*]
`edit`
[`-h`|`--help`]

## DESCRIPTION

**ase config edit** opens the configuration file of the target scope
(the strongest scope of the scope chain, by default `project` inside a
project, else `user`) in a text editor. The file and its parent
directories are created if missing (a new `user` scope file with file
mode `0600`).

After the editor exits, the file is re-read and strictly validated
against the configuration schema. If it carries invalid YAML or
invalid entries, the error is reported and the question `re-edit? [Y/n]`
is asked on standard error: any answer other than `n` or `no` opens the
editor again, while `n` or `no` lets the command fail (leaving the
invalid file as is).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--scope`- *scope*]:
Select the scope chain, whose strongest scope is the target scope
whose file is edited. As an option of- `ase config`, it has to be
given- *before*- `edit`. The- *scope*value is a comma-separated list
of the terms- `user`,- `project`,- `task:`- *id*, and- `session:`- *id*(with- *id*matching- `[A-Za-z0-9._-]+`), at most one per kind and in
any order, canonicalized into the order- `user`<- `project`<- `task`<- `session`.
 - `user`is always added implicitly, and- `project`is added implicitly
whenever a project context (a Git working tree or a- `.ase`directory
at or above the current directory) exists, but never above the
strongest explicitly requested term. An explicit- `project`without a
project context is an error. Default:- `project`inside a project
context, else- `user`. See- `ase-config(1)`for details.
 

## ENVIRONMENT

- `VISUAL`,- `EDITOR`:
The editor command, taken from the first non-empty one of- `VISUAL`and- `EDITOR`, falling back to- `vi`. The value is split on whitespace
into the command and its arguments (it is not run through a shell),
and the file name is appended as the last argument.

## FILES

The configuration file of the target scope: *per-user configuration
directory*`/config.yaml` (`user`), `.ase/config.yaml` (`project`),
`.ase/task/`*id*`/config.yaml` (`task:`*id*), or *per-user state
directory*`/session/`*id*`/config.yaml` (`session:`*id*). See
`ase-config(1)`.

## EXIT STATUS

`0` once the edited file is valid, `1` if the editor could not be run,
or if the file stayed invalid and re-editing was declined.

## EXAMPLES

```
# edit the project configuration with Vim
$ EDITOR=vim ase config edit
# edit the user configuration
$ ase config --scope user edit
```

## SEE ALSO

`ase(1)`, `ase-config(1)`, `ase-config-set(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-config-get(1) - Print a Configuration Value

## NAME

`ase-config-get(1)` - Print a Configuration Value

## SYNOPSIS

`ase config`
[`--scope` *scope*]
`get`
[`-h`|`--help`]
*key*

## DESCRIPTION

**ase config get** prints the effective value of the configuration
*key*, followed by a newline, on standard output. The value is looked
up from the strongest scope of the scope chain down to the built-in
`default` scope, and the first defined value wins. In contrast to `ase config list`, secret values are printed unmasked.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--scope`- *scope*]:
Select the scope chain whose values are cascaded. As an option of- `ase config`, it has to be given- *before*- `get`. The- *scope*value is
a comma-separated list of the terms- `user`,- `project`,- `task:`- *id*,
and- `session:`- *id*(with- *id*matching- `[A-Za-z0-9._-]+`), at most one
per kind and in any order, canonicalized into the order- `user`<- `project`<- `task`<- `session`.
 - `user`is always added implicitly, and- `project`is added implicitly
whenever a project context (a Git working tree or a- `.ase`directory
at or above the current directory) exists, but never above the
strongest explicitly requested term. An explicit- `project`without a
project context is an error. Default:- `project`inside a project
context, else- `user`. See- `ase-config(1)`for details.
 

## ARGUMENTS

- *key*:
The dotted configuration key, e.g.,- `agent.persona`, or any
unambiguous trailing part of a known key, e.g.,- `persona`. See- `ase-config(1)`for all keys.

## EXIT STATUS

`0` on success, `1` if *key* is not set on any scope, if *key* does not
address a leaf value (e.g., `project.task`), if *key* is ambiguous, or
on an invalid *scope*.

## EXAMPLES

```
# print the effective task lifecycle model
$ ase config get project.task.lifecycle
solo
# print the user-level persona via its abbreviated key
$ ase config --scope user get persona
engineer
```

## SEE ALSO

`ase(1)`, `ase-config(1)`, `ase-config-list(1)`, `ase-config-set(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-config-set(1) - Set a Configuration Value

## NAME

`ase-config-set(1)` - Set a Configuration Value

## SYNOPSIS

`ase config`
[`--scope` *scope*]
`set`
[`-h`|`--help`]
*key*
*value*

## DESCRIPTION

**ase config set** sets the configuration *key* to *value* in the
configuration file of the target scope (the strongest scope of the
scope chain, by default `project` inside a project, else `user`),
creating the file, its parent directories, and the intermediate maps
of the dotted key as needed. Weaker and intermediate scopes are never
modified.

The *value* is validated against the configuration schema before the
file is written, and an invalid value is an error. Some keys are
writable on particular scopes only (e.g., `project.task.*` on `user`
and `project`, the token keys on `user`): setting such a key on any
other scope is an error naming the permitted scopes. See
`ase-config(1)` for all keys, their values, and their scopes. A target
file which already carries invalid content is not overwritten, as this
content would be lost.

The file is written under an advisory file lock and atomically, with
file mode `0600` for the `user` scope and for any file carrying a
secret key.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--scope`- *scope*]:
Select the scope chain, whose strongest scope is the target scope
written by this command. As an option of- `ase config`, it has to be
given- *before*- `set`. The- *scope*value is a comma-separated list
of the terms- `user`,- `project`,- `task:`- *id*, and- `session:`- *id*(with- *id*matching- `[A-Za-z0-9._-]+`), at most one per kind and in
any order, canonicalized into the order- `user`<- `project`<- `task`<- `session`.
 - `user`is always added implicitly, and- `project`is added implicitly
whenever a project context (a Git working tree or a- `.ase`directory
at or above the current directory) exists, but never above the
strongest explicitly requested term, so an explicit- `--scope user`is never silently retargeted to- `project`. An explicit- `project`without a project context is an error. Default:- `project`inside a
project context, else- `user`. See- `ase-config(1)`for details.
 

## ARGUMENTS

- *key*:
The dotted configuration key, e.g.,- `agent.persona`, or any
unambiguous trailing part of a known key, e.g.,- `persona`.
 
- *value*:
The value to set, stored as a string (e.g.,- `true`for- `telemetry.enabled`or- `44318`for- `telemetry.service.port`).
 

## EXIT STATUS

`0` on success, `1` on an invalid *value*, a *key* not writable on the
target scope, an ambiguous *key*, an intermediate path of *key* which is
not a map, an invalid *scope*, or a target file carrying invalid
content.

## EXAMPLES

```
# switch the project to the enterprise task lifecycle model
$ ase config set project.task.lifecycle enterprise
# enable the AI usage accounting for the current user
$ ase config --scope user set telemetry.enabled true
# set the persona for a single session only
$ ase config --scope session:4711 set agent.persona caveman
```

## SEE ALSO

`ase(1)`, `ase-config(1)`, `ase-config-get(1)`, `ase-config-delete(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-config-delete(1) - Delete a Configuration Value

## NAME

`ase-config-delete(1)` - Delete a Configuration Value

## SYNOPSIS

`ase config`
[`--scope` *scope*]
`delete`
[`-h`|`--help`]
*key*

## DESCRIPTION

**ase config delete** deletes the configuration *key* from the
configuration file of the target scope (the strongest scope of the
scope chain, by default `project` inside a project, else `user`) and
writes the file back. Weaker scopes are never modified, so a value of
the same key on a weaker scope (or the built-in default) becomes
effective again.

Deleting a key which is not set on the target scope is no error. As
for `ase config set`, a key which is not writable on the target scope
cannot be deleted there either, and a target file which carries invalid
content is not overwritten.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--scope`- *scope*]:
Select the scope chain, whose strongest scope is the target scope
written by this command. As an option of- `ase config`, it has to be
given- *before*- `delete`. The- *scope*value is a comma-separated list
of the terms- `user`,- `project`,- `task:`- *id*, and- `session:`- *id*(with- *id*matching- `[A-Za-z0-9._-]+`), at most one per kind and in any
order, canonicalized into the order- `user`<- `project`<- `task`<- `session`.
 - `user`is always added implicitly, and- `project`is added implicitly
whenever a project context (a Git working tree or a- `.ase`directory
at or above the current directory) exists, but never above the
strongest explicitly requested term. An explicit- `project`without a
project context is an error. Default:- `project`inside a project
context, else- `user`. See- `ase-config(1)`for details.
 

## ARGUMENTS

- *key*:
The dotted configuration key, e.g.,- `agent.persona`, or any
unambiguous trailing part of a known key, e.g.,- `persona`.

## EXIT STATUS

`0` on success, `1` on a *key* not writable on the target scope, an
ambiguous *key*, an invalid *scope*, or a target file carrying invalid
content.

## EXAMPLES

```
# fall back to the default persona again
$ ase config delete agent.persona
# remove the task store token of the user
$ ase config --scope user delete project.task.store.token
```

## SEE ALSO

`ase(1)`, `ase-config(1)`, `ase-config-set(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-alias(1) - Manage the Skill Command Aliases

## NAME

`ase-alias(1)` - Manage the Skill Command Aliases

## SYNOPSIS

`ase alias`
[`-h`|`--help`]
*command*
[*options* [...]]
[*args* [...]]

`ase alias list`
[`-s`|`--scope` `user`|`project`]
[`-k`|`--skill` *skill*]
[`-T`|`--tuned`]

`ase alias set`
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-o`|`--option`|`-d`|`--default`]
*name*
*value* [...]

`ase alias delete`
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-o`|`--option`|`-d`|`--default`]
*name*
[*option*]

`ase alias edit`
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]

`ase alias sync`
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]

`ase alias expand`
*command-line* [...]

## DESCRIPTION

**ase alias** manages the *skill command aliases* of the *ASE* skills.
All aliases are expanded *deterministically* by *ASE* itself, and never
by the AI. Three kinds of aliases exist:

- A - *full alias*is a new command like- `/foo-bar`, which stands for an
arbitrary- *ASE*skill command line, like- `/ase-code-lint -S HIGH src/`. Its target can be another full alias, too, where the chain of
aliases is resolved (cycle-checked) down to the final- *ASE*skill, and
every alias of the chain appends its options and fixed arguments to
those of its target. On invocation, the options and fixed arguments of
the alias precede the arguments given to the alias, i.e.,- `/foo-bar lib/`invokes- `/ase-code-lint -S HIGH src/ lib/`. As options are
recognized only before the first argument, an alias targeting an alias
with fixed arguments should not add options of its own.
 - A full alias is provided as a generated - *shim skill*(see- **SHIM
SKILLS**), which just forwards to its target, and which is invoked as- `/foo-bar`in- *Anthropic Claude Code CLI*and- *GitHub Copilot CLI*,
and as- `$foo-bar`in- *OpenAI Codex CLI*. Its name consists of up to 64
lower-case letters, digits, and dashes, and must not start with- `ase-`.
 
- An - *option alias*is a new option like- `--custom`of a particular
skill, which is expanded- *in place*into a sequence of real options of
the skill, like- `-x -y -z`, or of further option aliases of the same
skill (recursively, but cycle-checked). It must not shadow a real
option of the skill.
 
- The - *option defaults*of a particular skill are options (including
option aliases) which are applied on- *every*invocation of the skill.
They are merged on the level of the parsed options, i.e., an option
explicitly given on invocation always wins over its default. A default
of a value-taking option is always a single value.
 

Additionally, every option `--`*x* of every *ASE* skill can be negated
by `--no-`*x*, which drops an `--`*x* injected by an option default or
an alias, i.e., resets the option to its built-in default (for a
value-less option: off). Of `--`*x* and `--no-`*x* given together, the
last one wins. An option `--no-`*x* declared explicitly by a skill
(like `--no-cleanup`) keeps its own meaning.

The expansion order is: the full alias (chain) first, then the option
aliases (in the given options and in the option defaults), then the
merge of the option defaults, and finally the negations.

The aliases are defined in the alias files of the *user* and of the
*project* (see **FILES**), where an entry of the project (a full alias,
an option alias of a skill, resp. the option defaults of a skill)
overrides the same entry of the user. The project alias file is intended
to be committed to the repository, so the entire team shares it, while
every team member gets the shim skills for the used agent tool at the
start of the agent tool session.

Without a *command*, the usage information is shown and the command
exits with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information of`ase alias`resp. of the given*command*and
exit.

## COMMANDS

- `list`[- *options*]:
List the effective full aliases, option aliases, and option defaults.
See- `ase-alias-list(1)`.
 
- `set`[- *options*]- *name*- *value*[...]:
Set a full alias, an option alias, or the option defaults of a skill.
See- `ase-alias-set(1)`.
 
- `delete`[- *options*]- *name*[- *option*]:
Delete a full alias, an option alias, or the option defaults of a
skill. See- `ase-alias-delete(1)`.
 
- `edit`[- *options*]:
Edit the alias file with the text editor. See- `ase-alias-edit(1)`.
 
- `sync`[- *options*]:
Synchronize the shim skills of the full aliases. See- `ase-alias-sync(1)`.
 
- `expand`- *command-line*[...]:
Expand a skill command line for testing the aliases. See- `ase-alias-expand(1)`.
 

## SHIM SKILLS

Every full alias *name* is provided as a generated *shim skill*, which
just forwards to the target skill with the options and fixed arguments
of the alias, followed by the arguments given to the shim skill. The
shim skill is written only for the active agent tool, which is the
agent tool of the session, resp. the one given with `--tool` (default:
`$ASE_TOOL`, else `claude`):

These paths are relative to the project for the full aliases of the
project, resp. to the home directory for the full aliases of the user.

The shim skills of a scope resolve their alias chains within the full
aliases of their scope only. They are marked as `generated by ase alias sync` and can be invoked explicitly only. A synchronization only writes
and removes (when orphaned) marked shim skills of its scope and its
agent tool, and fails on an unmarked skill of the same name. The shim
skills of the other agent tools stay untouched.

## FILES

An alias file carries the sections `alias` (alias names to command
lines), `option` (skill names to option aliases to expansions), and
`default` (skill names to option defaults), as in the example below:

```
alias:
    lint-sec:     /ase-code-lint --include=A05,A06 -S HIGH
    lint-sec-src: /lint-sec src/
option:
    ase-code-lint:
        --sec:    --include=A05,A06 -S HIGH
default:
    ase-task-edit: --approach
```

An invalid alias file is ignored by the option parser of the skills,
with a warning in the log of the service, so the skills keep working
without the aliases until the file is fixed.

## EXIT STATUS

`0` on success, `1` on any error (including a missing *command*).

## EXAMPLES

```
# define a full alias, invoked as "/lint-sec"
$ ase alias set lint-sec /ase-code-lint --include=A05,A06 -S HIGH
# check its expansion
$ ase alias expand /lint-sec src/
# list the effective aliases
$ ase alias list
```

## SEE ALSO

`ase(1)`, `ase-alias-list(1)`, `ase-alias-set(1)`, `ase-alias-delete(1)`,
`ase-alias-edit(1)`, `ase-alias-sync(1)`, `ase-alias-expand(1)`,
`ase-config(1)`, `ase-hook-session-start(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-alias-list(1) - List the Skill Command Aliases

## NAME

`ase-alias-list(1)` - List the Skill Command Aliases

## SYNOPSIS

`ase alias list`
[`-h`|`--help`]
[`-s`|`--scope` `user`|`project`]
[`-k`|`--skill` *skill*]
[`-T`|`--tuned`]

## DESCRIPTION

**ase alias list** lists the effective full aliases, option aliases, and
option defaults as a table of their kind (`alias`, `option`, or
`default`), their name (the alias name, resp. the skill name), their
option alias, their value, and their supplying scope. See `ase-alias(1)`
for the kinds of aliases and for how the scopes override each other.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-s`|- `--scope`- `user`|- `project`]:
List the entries of a single scope only. Default: both scopes.
 
- [- `-k`|- `--skill`- *skill*]:
List the entries of a single- *skill*only, i.e., its option aliases,
its option defaults, and the full aliases resolving to it.
 
- [- `-T`|- `--tuned`]:
List just the names of the skills with option aliases or option
defaults, separated by spaces, instead of the table and regardless of- `--scope`and- `--skill`.
 

## EXIT STATUS

`0` on success, `1` on any error.

## EXAMPLES

```
# list all effective aliases
$ ase alias list
# list the entries of the project only
$ ase alias list --scope project
# list the effective aliases of a skill
$ ase alias list --skill ase-code-lint
# list the skills with option aliases or option defaults
$ ase alias list --tuned
```

## SEE ALSO

`ase(1)`, `ase-alias(1)`, `ase-alias-set(1)`, `ase-alias-expand(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-alias-set(1) - Set a Skill Command Alias

## NAME

`ase-alias-set(1)` - Set a Skill Command Alias

## SYNOPSIS

`ase alias set`
[`-h`|`--help`]
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
*name*
*command-line* [...]

`ase alias set`
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
`-o`|`--option`
*skill*
*option*
*expansion* [...]

`ase alias set`
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
`-d`|`--default`
*skill*
*options* [...]

## DESCRIPTION

**ase alias set** sets an entry in the alias file of the scope: without
option the full alias *name*, with `--option` an option alias of the
*skill*, and with `--default` the option defaults of the *skill*. See
`ase-alias(1)` for the kinds of aliases.

The *value* (the *command-line*, the *expansion*, resp. the *options*)
is given either as a single (quoted) argument or as multiple arguments,
which are joined by single spaces (with the arguments containing
whitespace or quotes quoted). All options after the *name* belong to the
*value*.

The changed alias file is validated, and every option alias and the
option defaults of the *skill* are checked against the options of the
*skill*. On any failure, the previous alias file is kept. Afterwards,
the shim skills of the scope are synchronized (see `ase-alias-sync(1)`).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-s`|- `--scope`- `user`|- `project`]:
The scope of the alias file. Default:- `project`inside a project (a
Git working tree or a directory with a- `.ase`directory), else- `user`.
 
- [- `-t`|- `--tool`- `claude`|- `copilot`|- `codex`]:
The agent tool to write the shim skills for. Default:- `$ASE_TOOL`,
else- `claude`.
 
- [- `-o`|- `--option`]:
Set the option alias- *option*of the- *skill*to the- *expansion*.
Excludes- `--default`.
 
- [- `-d`|- `--default`]:
Set the option defaults of the- *skill*to the- *options*. Excludes- `--option`.
 

## ARGUMENTS

- *name*:
The name of the full alias, consisting of up to 64 lower-case letters,
digits, and dashes, and not starting with- `ase-`.
 
- *command-line*:
The command line of the full alias, where the first token is the
target skill or full alias, with an optional leading- `/`(resp.- `$`),
and with an optional- `ase:`prefix.
 
- *skill*:
The name of the skill of the option alias resp. the option defaults.
 
- *option*:
The option alias, like- `--custom`, which must not shadow a real option
of the- *skill*.
 
- *expansion*:
The options the option alias is expanded into.
 
- *options*:
The options applied on every invocation of the- *skill*.
 

## EXIT STATUS

`0` on success, `1` on any error, like an invalid alias file, an unknown
target, or an option unknown to the *skill*.

## EXAMPLES

```
# define a full alias, invoked as "/lint-sec"
$ ase alias set lint-sec /ase-code-lint --include=A05,A06 -S HIGH
# define an option alias "--sec" of a skill
$ ase alias set --option ase-code-lint --sec --include=A05,A06 -S HIGH
# always compose new task plans along proposed approaches (for yourself only)
$ ase alias set --scope user --default ase-task-edit --approach
```

## SEE ALSO

`ase(1)`, `ase-alias(1)`, `ase-alias-delete(1)`, `ase-alias-edit(1)`,
`ase-alias-expand(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-alias-delete(1) - Delete a Skill Command Alias

## NAME

`ase-alias-delete(1)` - Delete a Skill Command Alias

## SYNOPSIS

`ase alias delete`
[`-h`|`--help`]
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-o`|`--option`|`-d`|`--default`]
*name*
[*option*]

## DESCRIPTION

**ase alias delete** deletes an entry from the alias file of the scope:
without option the full alias *name*, with `--option` the option alias
*option* of the skill *name*, and with `--default` the option defaults
of the skill *name*. Afterwards, the shim skills of the scope are
synchronized (see `ase-alias-sync(1)`).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-s`|- `--scope`- `user`|- `project`]:
The scope of the alias file. Default:- `project`inside a project (a
Git working tree or a directory with a- `.ase`directory), else- `user`.
 
- [- `-t`|- `--tool`- `claude`|- `copilot`|- `codex`]:
The agent tool to write the shim skills for. Default:- `$ASE_TOOL`,
else- `claude`.
 
- [- `-o`|- `--option`]:
Delete an option alias of a skill. Excludes- `--default`.
 
- [- `-d`|- `--default`]:
Delete the option defaults of a skill. Excludes- `--option`.
 

## ARGUMENTS

- *name*:
The name of the full alias, resp. of the skill (with- `--option`or- `--default`).
 
- [- *option*]:
The option alias to delete, required with- `--option`.
 

## EXIT STATUS

`0` on success, `1` on any error, like a missing *option* or an entry
not existing in the scope.

## EXAMPLES

```
# remove a full alias
$ ase alias delete lint-sec
# remove an option alias
$ ase alias delete --option ase-code-lint --sec
# remove the option defaults of a skill (for yourself only)
$ ase alias delete --scope user --default ase-task-edit
```

## SEE ALSO

`ase(1)`, `ase-alias(1)`, `ase-alias-set(1)`, `ase-alias-edit(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-alias-edit(1) - Edit the Alias File

## NAME

`ase-alias-edit(1)` - Edit the Alias File

## SYNOPSIS

`ase alias edit`
[`-h`|`--help`]
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]

## DESCRIPTION

**ase alias edit** edits the alias file of the scope with the text
editor, creating the file if it does not exist yet. Afterwards, the
alias file is validated (and on failure, re-edited on request), and the
shim skills of the scope are synchronized (see `ase-alias-sync(1)`). See
`ase-alias(1)` for the format of the alias file.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-s`|- `--scope`- `user`|- `project`]:
The scope of the alias file. Default:- `project`inside a project (a
Git working tree or a directory with a- `.ase`directory), else- `user`.
 
- [- `-t`|- `--tool`- `claude`|- `copilot`|- `codex`]:
The agent tool to write the shim skills for. Default:- `$ASE_TOOL`,
else- `claude`.
 

## ENVIRONMENT

- `VISUAL`,- `EDITOR`:
The text editor, in this order of precedence, falling back to- `vi`.

## EXIT STATUS

`0` on success, `1` on any error, like a declined re-edit of an invalid
alias file or the scope `project` without a project.

## EXAMPLES

```
# edit the alias file of the project
$ ase alias edit
# edit the alias file of the user
$ ase alias edit --scope user
```

## SEE ALSO

`ase(1)`, `ase-alias(1)`, `ase-alias-set(1)`, `ase-alias-delete(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-alias-sync(1) - Synchronize the Shim Skills

## NAME

`ase-alias-sync(1)` - Synchronize the Shim Skills

## SYNOPSIS

`ase alias sync`
[`-h`|`--help`]
[`-s`|`--scope` `user`|`project`]
[`-t`|`--tool` `claude`|`copilot`|`codex`]

## DESCRIPTION

**ase alias sync** synchronizes the shim skills of the full aliases for
the agent tool (see **SHIM SKILLS** of `ase-alias(1)`). This also
happens automatically on every change through `ase alias`, and at the
start of every agent tool session (see `ase-hook-session-start(1)`),
e.g., after a pull of a changed project alias file.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-s`|- `--scope`- `user`|- `project`]:
Synchronize the shim skills of a single scope only. Default: both
scopes.
 
- [- `-t`|- `--tool`- `claude`|- `copilot`|- `codex`]:
The agent tool to write the shim skills for. Default:- `$ASE_TOOL`,
else- `claude`.
 

## EXIT STATUS

`0` on success, `1` on any error, like an unmarked skill having the same
name as a shim skill.

## EXAMPLES

```
# synchronize the shim skills of both scopes
$ ase alias sync
# synchronize the shim skills of the project only
$ ase alias sync --scope project
```

## SEE ALSO

`ase(1)`, `ase-alias(1)`, `ase-alias-set(1)`, `ase-hook-session-start(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-alias-expand(1) - Expand a Skill Command Line

## NAME

`ase-alias-expand(1)` - Expand a Skill Command Line

## SYNOPSIS

`ase alias expand`
[`-h`|`--help`]
*command-line* [...]

## DESCRIPTION

**ase alias expand** expands the skill *command-line* through the full
aliases, the option aliases, and the option defaults of both scopes, and
prints the expanded command line, the option defaults, the resulting
options, and the resulting arguments of the skill. This is intended for
testing aliases. See `ase-alias(1)` for the expansion order.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- *command-line*:
The skill command line, i.e., a full alias or a skill, followed by its
options and arguments, given either as a single (quoted) argument or
as multiple arguments.

## EXIT STATUS

`0` on success, `1` on any error, like an unknown alias or skill.

## EXAMPLES

```
# expand a full alias, followed by an argument
$ ase alias expand /lint-sec src/
# check the expansion, where "--no-approach" drops the option default again
$ ase alias expand /ase-task-edit --no-approach foo
```

## SEE ALSO

`ase(1)`, `ase-alias(1)`, `ase-alias-list(1)`, `ase-alias-set(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-mcp(1) - Bridge Stdio MCP to the Per-Project Service

## NAME

`ase-mcp(1)` - Bridge Stdio MCP to the Per-Project Service

## SYNOPSIS

`ase mcp`
[`-h`|`--help`]

## DESCRIPTION

**ase mcp** is a Model-Context-Protocol (MCP) server speaking the
stdio transport: it accepts MCP messages on standard input, forwards
them to the per-project background service of `ase-service(1)` via the
Streamable HTTP transport (`http://127.0.0.1:`*port*`/mcp`), and writes
the responses to standard output.

It is configured as the stdio MCP server `ase` of the *ASE* plugin in
the agent tool and is not intended to be invoked directly by end users.
All log messages go to standard error (or to the file of the global
`--log-file` option), as standard output carries the MCP protocol.

On start, the service of the project is probed and auto-started (via
`ase service start`) if it is not running yet. The bridge then lives as
long as its standard input is open, and is robust against a vanishing
service:

- Messages arriving while no service connection exists are buffered
(up to 100 messages, dropping the oldest ones on overflow) and sent
once the connection is (re-)established. 
- A message whose connection breaks before the service responds is sent
once more, and a request still failing is answered with a JSON-RPC
error (- `ase service unreachable:`- *reason*).
 
- A request whose response stream ends without its response, e.g. as
the service vanished while processing it, is answered with a JSON-RPC
error as well (- `ase service connection lost`), as its outcome is
unknown and hence it is not sent once more.
 
- On a lost or refused connection, the service is auto-started again
and the connection is re-established, with a delay growing from 0.5
up to 10 seconds between the attempts. 
- Every 30 seconds, the service is probed, and a vanished service
triggers the same reconnect. 

Under *GitHub Copilot CLI* (detected by the environment variable
`COPILOT_PLUGIN_ROOT`), the MCP server is started in the plugin
installation directory instead of the project. Hence, the bridge first
answers all MCP messages in-process through the same set of MCP tools
the service provides (covering the MCP handshake and the tool listing),
holds back all tool calls, and waits until the `SessionStart` hook (see
`ase-hook(1)`) recorded the project directory of the session (with id
`COPILOT_AGENT_SESSION_ID`).

It then switches into this directory, starts or connects the service
of the project there, and forwards the held tool calls. If no project
directory is recorded within 30 seconds of the first held tool call, it
falls back to the current directory.

The service provides the `ase_*` MCP tools of the skills, e.g.,
`ase_ping`, `ase_config_get`, `ase_config_set`, `ase_config_delete`,
`ase_config_list`, `ase_state_get`, `ase_state_set`,
`ase_state_delete`, `ase_kv_get`, `ase_kv_set`, `ase_kv_delete`,
`ase_kv_clear`, `ase_kv_batch`, the `ase_task_*` tools, `ase_artifact_list`,
`ase_artifact_name`, `ase_specbook_lint`, `ase_specbook_export`, `ase_specbook_query`,
`ase_diagram`, `ase_worktree_path`, `ase_mint`, `ase_text_metric`,
`ase_getopt`, `ase_markdown_prepare`, `ase_timestamp`, `ase_sleep`,
`ase_compat`, `ase_component_info`, and `ase_decision_matrix`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ENVIRONMENT

- `COPILOT_PLUGIN_ROOT`,- `COPILOT_AGENT_SESSION_ID`:
Exported by- *GitHub Copilot CLI*to its MCP servers. When both are
set (and the session id matches- `[A-Za-z0-9._-]+`), the service is
deferred until the project directory of the session is known (see- **DESCRIPTION**).

## FILES

- `.ase/service.yaml`:
The persisted port of the per-project service.
 
- *per-user state directory*- `/session/`- *id*- `/state.yaml`:
The internal session state, from which the project directory
(- `basedir`) recorded by the- `SessionStart`hook is read under- *GitHub
Copilot CLI*.
 

## EXIT STATUS

`0` once standard input is closed, `1` if the service cannot be
started initially.

## EXAMPLES

```
# register the bridge manually as an MCP server in Claude Code
$ claude mcp add ase -- ase mcp
```

## SEE ALSO

`ase(1)`, `ase-service(1)`, `ase-hook(1)`, `ase-setup(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-service(1) - Manage the Per-Project Background Service

## NAME

`ase-service(1)` - Manage the Per-Project Background Service

## SYNOPSIS

`ase service`
[`-h`|`--help`]
*command*
[*options* [...]]
[*args* [...]]

`ase service start` [`-f`|`--foreground`]

`ase service status`

`ase service send` *cmd*

`ase service log` [`-f`|`--follow`] [`-n`|`--lines` *n*]

`ase service stop`

## DESCRIPTION

**ase service** manages the per-project background HTTP service of
*ASE*. The service is bound to `127.0.0.1` on the preferred port
`project.service.port`, if configured and available, else on a random
port out of the range `42000`..`44000`, which is persisted in
`.ase/service.yaml` of the project, and logs into `.ase/service.log`. It is identified by the id
of the project (`project.id`, else derived from the project root), so a
port answered by the service of another project counts as a *foreign*
service.

The service provides:

- the MCP endpoint `/mcp`(Streamable HTTP), serving the`ase_*`MCP
tools of the skills, bridged into the agent tool via`ase-mcp(1)`,
which auto-starts the service on demand,
- the command endpoint `/command`(see`ase-service-send(1)`) and the
stop endpoint`/stop`,
- the endpoints `/usage/*`of the AI usage accounting of the task
assignments (see`ase-task-assign(1)`), plus, while`telemetry.enabled`is`true`, the attempt to hold the per-user OTLP
ingress on`127.0.0.1:`*port*(`telemetry.service.port`, default`44318`), routing the received records to the service of the project
owning the session (see`ase-setup-telemetry(1)`),
- the web task board of `ase task board --web`(see`ase-task-board(1)`).

The service accepts requests with a loopback `Host` header only, to
defeat DNS rebinding attacks from browser pages. A detached service
stops itself after 30 minutes without any request, but never while a
request is in flight or while it holds open AI usage windows.

Without a *command*, the usage information is shown and the command
exits with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## COMMANDS

## FILES

- `.ase/service.yaml`:
The per-project service state (the- `port`of the running service),
removed again once the service stopped.
 
- `.ase/service.log`:
The standard output and standard error of the detached service,
appended to. As the service inherits the file descriptor for its
entire lifetime, the file cannot be rotated while the service runs,
and instead is trimmed down to its last 2000 lines whenever a
service is started on a file larger than 1 MB.
 - Routine MCP traffic (session handshakes, notifications, and stream
opens) is logged at level - `debug`only, and the arguments of the
logged MCP tool calls are cut at 200 characters.
 
- `.ase/.gitignore`:
The self-ignoring Git ignore file covering the machine-local runtime
files, created on the first start if missing.
 

## EXIT STATUS

`0` on success, `1` on any error (including a missing *command*). See
the individual commands for details.

## EXAMPLES

```
# start the service with the full MCP traffic logged
$ ase --log-level debug service start
# check the service and follow its log
$ ase service status
$ ase service log -f
```

## SEE ALSO

`ase(1)`, `ase-service-start(1)`, `ase-service-status(1)`,
`ase-service-send(1)`, `ase-service-log(1)`, `ase-service-stop(1)`,
`ase-mcp(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-service-start(1) - Start the Per-Project Background Service

## NAME

`ase-service-start(1)` - Start the Per-Project Background Service

## SYNOPSIS

`ase service start`
[`-h`|`--help`]
[`-f`|`--foreground`]

## DESCRIPTION

**ase service start** starts the per-project background HTTP service
of *ASE* (see `ase-service(1)`) for the project of the current working
directory.

If a service of this project already answers on the port persisted in
`.ase/service.yaml`, the command just reports `service: already running on port` *port* and exits with status `0`. Otherwise, it allocates the
preferred port `project.service.port` (see `ase-config(1)`), if
configured and free, else (with a warning, if configured) a fresh
random free port out of the range `42000`..`44000` (trying up to 20
candidates), spawns the service as a detached background process
(with its standard output and standard error appended to
`.ase/service.log`, trimmed to its last 2000 lines beforehand if larger
than 1 MB), and probes its readiness for up to 5 seconds.

Once ready, the service persists its port into `.ase/service.yaml`,
and the command reports `service: started on port` *port*. If the
spawned service exits during its startup, does not become ready in
time, or loses its port to a foreign listener, the attempt is aborted
(killing the process) and retried with a new port, up to three attempts
in total, after which the command fails with the reason and the last
20 lines of the log.

The detached service adopts the log level of the global `--log-level`
option of this command, e.g., `ase --log-level debug service start`
logs the full MCP traffic. It stops itself after 30 minutes without any
request (but never while a request is in flight or while it holds open
AI usage windows), on `ase service stop`, and removes its port from
`.ase/service.yaml` again.

Usually, the service does not have to be started explicitly, as the
MCP bridge `ase mcp`, `ase service send`, and the hooks start it on
demand.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-f`|- `--foreground`]:
Run the service in the current process instead, on a port allocated
as above, logging to the terminal (resp. the global- `--log-file`) instead
of- `.ase/service.log`and without the idle timeout, until it is
interrupted (via- `SIGINT`, e.g.,- `CTRL+C`,- `SIGTERM`,- `SIGHUP`, or- `SIGQUIT`) or stopped via- `ase service stop`. A service of this
project already running is an error in this mode.
 

## FILES

- `.ase/service.yaml`:
The persisted port of the running service.
 
- `.ase/service.log`:
The log of the detached service.
 
- `.ase/.gitignore`:
The self-ignoring Git ignore file covering the machine-local runtime
files, created if missing.
 

## EXIT STATUS

`0` if the service was started or is already running (resp., in the
foreground, once it was stopped), `1` if it could not be started, or,
with `--foreground`, if it is already running.

## EXAMPLES

```
# start the service in the background
$ ase service start
# run the service in the foreground for debugging
$ ase --log-level debug service start --foreground
```

## SEE ALSO

`ase(1)`, `ase-service(1)`, `ase-service-status(1)`,
`ase-service-stop(1)`, `ase-service-log(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-service-status(1) - Report Whether the Background Service Is Running

## NAME

`ase-service-status(1)` - Report Whether the Background Service Is Running

## SYNOPSIS

`ase service status`
[`-h`|`--help`]

## DESCRIPTION

**ase service status** reports whether the per-project background
service of *ASE* (see `ase-service(1)`) is running. It probes the port
persisted in `.ase/service.yaml` via an HTTP `OPTIONS /` request and
verifies that the responding service belongs to the current project
(by its project id). It prints one of the following lines on standard
output:

- `service: running on port`- *port*- `(uptime:`- *duration*- `)`:
A service of this project is running.
 
- `service: not running (no port configured)`:
No port is persisted in- `.ase/service.yaml`.
 
- `service: not running (port`- *port*- `not responding)`:
Nothing listens on the persisted port.
 
- `service: not running (port`- *port*- `in use by foreign service)`:
The persisted port is answered by something else, e.g., the service
of another project.
 

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## FILES

- `.ase/service.yaml`:
The persisted port of the service.

## EXIT STATUS

`0` if the service of the project is running, else `1`.

## EXAMPLES

```
$ ase service status
service: running on port 42817 (uptime: 12 minutes 3.4 seconds)
```

## SEE ALSO

`ase(1)`, `ase-service(1)`, `ase-service-start(1)`,
`ase-service-stop(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-service-send(1) - Send a Command to the Background Service

## NAME

`ase-service-send(1)` - Send a Command to the Background Service

## SYNOPSIS

`ase service send`
[`-h`|`--help`]
*cmd*

## DESCRIPTION

**ase service send** dispatches the command token *cmd* to the
per-project background service of *ASE* (see `ase-service(1)`) via an
HTTP `POST /command` request with the JSON body `{ "command":` *cmd*
`}`, and prints the raw response body (a JSON object) on standard
output. If the service is not running, it is auto-started first (like
`ase service start`).

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

## EXIT STATUS

`0` if the service answered with a success status, `1` if it answered
with an error status (e.g., for an unknown *cmd*), or if it could not
be started or reached.

## EXAMPLES

```
$ ase service send ping
{"ok":true,"pong":true}
$ ase service send status
{"ok":true,"projectId":"ase","port":42817,"uptimeMs":723456}
```

## SEE ALSO

`ase(1)`, `ase-service(1)`, `ase-service-status(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-service-log(1) - Show the Log of the Background Service

## NAME

`ase-service-log(1)` - Show the Log of the Background Service

## SYNOPSIS

`ase service log`
[`-h`|`--help`]
[`-f`|`--follow`]
[`-n`|`--lines` *n*]

## DESCRIPTION

**ase service log** shows the log file `.ase/service.log` of the
per-project background service of *ASE* (see `ase-service(1)`) on
standard output: by default the entire file, else its last lines
only. If no log file exists, nothing is shown and the command exits
with status `0`.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-f`|- `--follow`]:
After showing the last- *n*lines (default:- `10`), keep following the
file in the style of- `tail -f`and show all newly appended lines,
until interrupted (e.g., via- `CTRL+C`), even once the service has
stopped. When the log is trimmed on a service start, following
continues at the new end of the file.
 
- [- `-n`|- `--lines`- *n*]:
Show the last- *n*non-empty lines only, where- *n*has to be a
positive integer (default: all lines, or- `10`with- `--follow`).
 

## FILES

- `.ase/service.log`:
The log of the detached service.

## EXIT STATUS

`0` on success, `1` on an invalid *n* or an unreadable log file.

## EXAMPLES

```
# show the last 50 log lines
$ ase service log -n 50
# follow the log
$ ase service log -f
```

## SEE ALSO

`ase(1)`, `ase-service(1)`, `ase-service-start(1)`, `tail(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-service-stop(1) - Stop the Background Service

## NAME

`ase-service-stop(1)` - Stop the Background Service

## SYNOPSIS

`ase service stop`
[`-h`|`--help`]

## DESCRIPTION

**ase service stop** stops the per-project background service of
*ASE* (see `ase-service(1)`) via an HTTP `POST /stop` request to the
port persisted in `.ase/service.yaml`, and removes the persisted port
(and the file, if it becomes empty). This also stops a service running
in the foreground via `ase service start --foreground`. On stop, the
service persists the journal of its AI usage windows, so no usage is
lost.

If no port is persisted, or nothing listens on the persisted port, an
informational message is logged, a stale port is removed, and the
command exits with status `0`. If the persisted port is answered by a
foreign service (e.g., the service of another project), it is left
alone and the command exits with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## FILES

- `.ase/service.yaml`:
The persisted port of the service, cleared on stop.

## EXIT STATUS

`0` if the service was stopped or was not running, `1` if the port is
in use by a foreign service or the stop request failed.

## EXAMPLES

```
$ ase service stop
```

## SEE ALSO

`ase(1)`, `ase-service(1)`, `ase-service-start(1)`,
`ase-service-status(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-hook(1) - Handle the Hook Events of the Agent Tool

## NAME

`ase-hook(1)` - Handle the Hook Events of the Agent Tool

## SYNOPSIS

`ase hook`
[`-h`|`--help`]

`ase hook`
*event*
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase hook** is the entry point group of the hook handlers which the
*ASE* plugin configures in the agent tool: in `hooks/hooks.json` for
*Anthropic Claude Code CLI*, in `hooks/hooks-copilot.json` for *GitHub
Copilot CLI* (with `--tool copilot`), and in `hooks/hooks-codex.json`
for *OpenAI Codex CLI* (with `--tool codex`). The agent tool invokes a
handler on the corresponding event, passes the event payload as a JSON
object on standard input, and interprets the JSON object a handler
optionally writes to standard output.

The handlers are intended to be invoked by the agent tool internally
only, and not directly by end users. Without a sub-command, the usage
information is shown and the command exits with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information, including the list of all event handlers,
and exit.

## COMMANDS

The following event handlers exist, each accepting the option
[`-t`|`--tool` *tool*] (`claude`, `copilot`, or `codex`, default:
`$ASE_TOOL`, else `claude`) to select the agent tool whose payload
format is read and written:

- `ase hook session-start`:
Handle the- `SessionStart`(resp.- `sessionStart`) event: inject the- *ASE*constitution and session information into the session context.
See- `ase-hook-session-start(1)`.
 
- `ase hook session-persona`:
Handle the- `SessionStart`(resp.- `sessionStart`) event a second time:
inject the- *ASE*persona rules into the session context.
See- `ase-hook-session-persona(1)`.
 
- `ase hook session-end`:
Handle the- `SessionEnd`(resp.- `sessionEnd`) event: remove the
session state and detach the session from its AI usage window. See- `ase-hook-session-end(1)`.
 
- `ase hook pre-tool-use`:
Handle the- `PreToolUse`(resp.- `preToolUse`) event: auto-approve the
invocations of- *ASE*tools. See- `ase-hook-pre-tool-use(1)`.
 
- `ase hook permission-request`:
Handle the- `PermissionRequest`event of- *OpenAI Codex CLI*:
auto-approve the invocations of- *ASE*tools. See- `ase-hook-permission-request(1)`.
 
- `ase hook user-prompt-submit`:
Handle the- `UserPromptSubmit`(resp.- `userPromptSubmitted`) event:
mark the agent as busy. See- `ase-hook-user-prompt-submit(1)`.
 
- `ase hook stop`:
Handle the- `Stop`(resp.- `agentStop`) event: mark the agent as
ready. See- `ase-hook-stop(1)`.
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of the- `--tool`option of all event handlers. An invalid
value is reported only once a handler actually runs.

## EXIT STATUS

Each event handler exits with status `0` on success, and with status
`1` on an invalid `--tool` value or a fatal error. The group command
without a sub-command exits with status `1`.

## SEE ALSO

`ase(1)`, `ase-hook-session-start(1)`, `ase-hook-session-persona(1)`,
`ase-hook-session-end(1)`,
`ase-hook-pre-tool-use(1)`, `ase-hook-permission-request(1)`,
`ase-hook-user-prompt-submit(1)`, `ase-hook-stop(1)`, `ase-setup(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-hook-session-start(1) - Handle the SessionStart Hook Event

## NAME

`ase-hook-session-start(1)` - Handle the SessionStart Hook Event

## SYNOPSIS

`ase hook session-start`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase hook session-start** handles the `SessionStart` hook event of
*Anthropic Claude Code CLI* and *OpenAI Codex CLI*, resp. the
`sessionStart` hook event of *GitHub Copilot CLI*. It is intended to be
invoked by the agent tool internally as a configured hook handler only,
and not directly by end users.

The handler reads the event payload from standard input, a JSON object
with the session id (field `session_id`, resp. `sessionId` for *GitHub
Copilot CLI*) and the working directory (field `cwd`), and performs the
following steps:

- It determines the plugin root directory from the environment
variable - `CLAUDE_PLUGIN_ROOT`(- `claude`),- `COPILOT_PLUGIN_ROOT`(- `copilot`), resp.- `PLUGIN_ROOT`or- `CLAUDE_PLUGIN_ROOT`(- `codex`),
and reads the plugin manifest- `.claude-plugin/plugin.json`(for the
plugin version), the constitution- `meta/ase-constitution.md`, and the
output style- `output-styles/ase-terse.md`from it.
 
- It determines a version hint: a warning on a version mismatch
between the tool and the plugin, a notice on a newer tool version
available in the NPM registry (checked in the background at most
once per hour), and a notice on a development setup (environment
variable - `ASE_SETUP_DEV`set).
 
- For - `--tool copilot`only, it changes into the project directory- `cwd`of the payload, as- *GitHub Copilot CLI*runs its hooks in the
plugin installation directory.
 
- It removes all session state directories below the - *per-user state
directory*- `/session/`(except the one of the current session) which
were not modified for more than 7 days, i.e., the leftovers of agent
tool sessions which ended without running- `ase hook session-end`.
 
- It determines the project directory (the Git top-level directory of
- `cwd`, else- `cwd`itself) and, for a valid session id (matching- `[A-Za-z0-9._-]+`), persists the active task id (- `$ASE_TASK_ID`,
else- `default`) and the project directory into the internal session
state- *per-user state directory*- `/session/`- *id*- `/state.yaml`.
 
- Inside a - *tmux*pane, it sets the per-pane user option- `@ase_agent_status`to- `⏸`(ready).
 
- It resolves the session information from the configuration
cascade (with the scope - `session:`- *id*for a valid session id):
the project id (a valid configured- `project.id`, else derived from
the project directory), the user id (- `$USER`, else- `$LOGNAME`, else- `unknown`), the persona style- `agent.persona`, the guidance level- `agent.guidance`, the diagram mode- `agent.diagram.format`, the boxing
transparency- `project.boxing`, the task
lifecycle model- `project.task.lifecycle`, the defaults- `project.task.default.source`,- `project.task.default.changeset`, and- `project.task.default.target`, the specification base directory- `project.artifact.spec.basedir`, and the schema configuration- `project.artifact.spec.schema`.
 - A setting not explicitly configured in any scope falls back to the
environment variables - `ASE_PERSONA_STYLE`,- `ASE_GUIDANCE_LEVEL`,- `ASE_DIAGRAM_MODE`,- `ASE_PROJECT_BOXING`,- `ASE_PROJECT_TASK_LIFECYCLE`,- `ASE_PROJECT_TASK_DEFAULT_SOURCE`,- `ASE_PROJECT_TASK_DEFAULT_CHANGESET`, and- `ASE_PROJECT_TASK_DEFAULT_TARGET`, and finally to the defaults- `engineer`,- `normal`,- `image`,- `white`,- `solo`,- `worktree`,- `worktree`, and- `source`.
 
- For - `--tool claude`only, it appends- `export`statements of the
variables- `ASE_VERSION`,- `ASE_PLUGIN_ROOT`,- `ASE_USER_ID`,- `ASE_PROJECT_ID`,- `ASE_PROJECT_BOXING`,- `ASE_PROJECT_TASK_LIFECYCLE`,- `ASE_PROJECT_TASK_DEFAULT_SOURCE`,- `ASE_PROJECT_TASK_DEFAULT_CHANGESET`,- `ASE_PROJECT_TASK_DEFAULT_TARGET`,- `ASE_SPEC_BASEDIR`,- `ASE_SPEC_SCHEMA`,- `ASE_TASK_ID`,- `ASE_SESSION_ID`,- `ASE_HEADLESS`,
and- `ASE_AGENT_TOOL`to the file of the environment variable- `CLAUDE_ENV_FILE`(if set), so they are available to all shell
commands of the session.
 
- It synchronizes the shim skills of the full aliases of the user and
the project for the agent tool of the session (see - `ase-alias(1)`),
e.g., after a pull of a changed project alias file, and determines the
skills with option aliases or option defaults. A failure is logged as
a warning only.
 
- It writes the hook output payload to standard output: the session
information as XML-style tags (- `<ase-version>`,- `<ase-version-hint>`,- `<ase-plugin-root>`,- `<ase-persona-style>`,- `<ase-guidance-level>`,- `<ase-diagram-mode>`,- `<ase-user-id>`,- `<ase-project-id>`,- `<ase-project-boxing>`,- `<ase-project-task-lifecycle>`,- `<ase-project-task-default-source>`,- `<ase-project-task-default-changeset>`,- `<ase-project-task-default-target>`,- `<ase-spec-basedir>`,- `<ase-spec-schema>`,- `<ase-getopt-tuned>`(the space-separated
skills with option aliases or option defaults),- `<ase-task-id>`,- `<ase-session-id>`,- `<ase-headless>`, and- `<ase-agent-tool>`),
followed by the constitution, with all- `@`- *file*references expanded
recursively (relative to the referencing file,- `~/`denoting the
home directory).
 - The persona rules are not part of this output, but are injected
separately by - `ase hook session-persona`, as agent tools like- *Anthropic Claude Code CLI*show only a preview of a hook output
beyond 10,000 characters.
 - For - `--tool copilot`and- `--tool codex`only, the instructions of the
output style- `ase-terse`(with its YAML frontmatter stripped) are
appended, as these agent tools have no output style concept, whereas- *Anthropic Claude Code CLI*applies the plugin output style natively.
The context is emitted as- `hookSpecificOutput.additionalContext`(- `claude`and- `codex`), resp. as the top-level field- `additionalContext`(- `copilot`).
 - Additionally, for - `claude`and- `codex`, unless running headless
(- `ASE_HEADLESS`is- `true`) or with guidance level- `none`, a
deterministic- *ASE*banner (version, user, project, task, session,
persona, guidance, and boxing, plus usage hints for guidance level- `normal`and- `verbose`) is emitted as the top-level field- `systemMessage`.
 

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
The agent tool whose event payload is read and whose hook output
format is written:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*). Default:- `$ASE_TOOL`, else- `claude`.
 

## ENVIRONMENT

- `CLAUDE_PLUGIN_ROOT`,- `COPILOT_PLUGIN_ROOT`,- `PLUGIN_ROOT`:
The plugin root directory, provided by the agent tool. The handler
fails if the variable of the selected tool is unset or empty.
 
- `CLAUDE_ENV_FILE`:
The environment file of- *Anthropic Claude Code CLI*, receiving the
exported session information.
 
- `ASE_TOOL`:
The default of- `--tool`.
 
- `ASE_TASK_ID`:
The active task id of the session (default:- `default`).
 
- `ASE_HEADLESS`:
If- `true`, the session runs headless, and no banner is emitted.
 
- `ASE_SETUP_DEV`:
If set, the version hint notes a development setup.
 
- `ASE_PERSONA_STYLE`,- `ASE_GUIDANCE_LEVEL`,- `ASE_DIAGRAM_MODE`,- `ASE_PROJECT_BOXING`,- `ASE_PROJECT_TASK_LIFECYCLE`,- `ASE_PROJECT_TASK_DEFAULT_SOURCE`,- `ASE_PROJECT_TASK_DEFAULT_CHANGESET`,- `ASE_PROJECT_TASK_DEFAULT_TARGET`:
The fallbacks of the settings not explicitly configured.
 
- `USER`,- `LOGNAME`:
The user id.
 
- `TMUX`,- `TMUX_PANE`:
The- *tmux*pane whose user option- `@ase_agent_status`is set.
 

## FILES

- *per-user state directory*- `/session/`- *id*- `/state.yaml`:
The internal session state, holding the active task id and the
project directory of the session.

## EXIT STATUS

Exits with status `0` on success, and with status `1` if the plugin
root directory cannot be determined, a plugin file cannot be read, or
`--tool` is invalid.

## SEE ALSO

`ase(1)`, `ase-hook(1)`, `ase-hook-session-persona(1)`,
`ase-hook-session-end(1)`, `ase-config(1)`,
`ase-statusline(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-hook-session-persona(1) - Handle the SessionStart Hook Event for the Persona

## NAME

`ase-hook-session-persona(1)` - Handle the SessionStart Hook Event for the Persona

## SYNOPSIS

`ase hook session-persona`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase hook session-persona** handles the `SessionStart` hook event of
*Anthropic Claude Code CLI* and *OpenAI Codex CLI*, resp. the
`sessionStart` hook event of *GitHub Copilot CLI*, in addition to
`ase hook session-start`.
It is intended to be invoked by the agent tool internally as a
configured hook handler only, and not directly by end users.

The handler determines the plugin root directory (see
`ase-hook-session-start(1)`), reads the persona rules
`meta/ase-persona.md` from it, and writes them as the hook output
payload to standard output, as `hookSpecificOutput.additionalContext`
(`claude` and `codex`), resp. as the top-level field
`additionalContext` (`copilot`).

The persona rules are injected by this separate handler, as agent tools
like *Anthropic Claude Code CLI* show only a preview of a hook output
beyond 10,000 characters, which otherwise would cut off the persona
rules from the output of `ase hook session-start`. The rules refer to
the persona style `<ase-persona-style>` injected by `ase hook session-start`, but only by name, so both handlers are independent of
each other and their execution order does not matter.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
The agent tool whose hook output format is written:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*). Default:- `$ASE_TOOL`, else- `claude`.
 

## ENVIRONMENT

- `CLAUDE_PLUGIN_ROOT`,- `COPILOT_PLUGIN_ROOT`,- `PLUGIN_ROOT`:
The plugin root directory, provided by the agent tool. The handler
fails if the variable of the selected tool is unset or empty.
 
- `ASE_TOOL`:
The default of- `--tool`.
 

## EXIT STATUS

Exits with status `0` on success, and with status `1` if the plugin
root directory cannot be determined, the persona file cannot be read,
or `--tool` is invalid.

## SEE ALSO

`ase(1)`, `ase-hook(1)`, `ase-hook-session-start(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-hook-session-end(1) - Handle the SessionEnd Hook Event

## NAME

`ase-hook-session-end(1)` - Handle the SessionEnd Hook Event

## SYNOPSIS

`ase hook session-end`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase hook session-end** handles the `SessionEnd` hook event of
*Anthropic Claude Code CLI* and *OpenAI Codex CLI*, resp. the
`sessionEnd` hook event of *GitHub Copilot CLI*. It is intended to be
invoked by the agent tool internally as a configured hook handler only,
and not directly by end users.

The handler reads the event payload from standard input, a JSON object
with the session id (field `session_id`, resp. `sessionId`). For a
valid session id (matching `[A-Za-z0-9._-]+`), it:

- removes the session directory - *per-user state
directory*- `/session/`- *id*with the session-scoped configuration
(scope- `session:`- *id*) and the internal session state, and
 
- detaches the session from its AI usage window (if any, see
- `ase-task-assign(1)`) through the- `ase service`of the owning project,
which closes and books the window once no other session remains
attached to it. If the owning project service is not reachable in
time, the session end is noted in the session registry
(- *per-user state directory*- `/telemetry.json`) instead, and the
service is started in the background, which then detaches the
session as of its noted end. A failure is only warned about.
 

An empty, malformed, or invalid payload is silently ignored. The
handler writes nothing to standard output.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
The agent tool whose event payload is read:- `claude`,- `copilot`, or- `codex`. Default:- `$ASE_TOOL`, else- `claude`.
 

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.

## FILES

- *per-user state directory*- `/session/`- *id*- `/`:
The per-session directory, removed by this handler.
 
- *per-user state directory*- `/telemetry.json`:
The telemetry state, whose session registry routes the session to
the service of its project.
 

## EXIT STATUS

Exits with status `0`, unless `--tool` is invalid (status `1`).

## SEE ALSO

`ase(1)`, `ase-hook(1)`, `ase-hook-session-start(1)`,
`ase-task-assign(1)`, `ase-service(1)`, `ase-setup-telemetry(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-hook-pre-tool-use(1) - Handle the PreToolUse Hook Event

## NAME

`ase-hook-pre-tool-use(1)` - Handle the PreToolUse Hook Event

## SYNOPSIS

`ase hook pre-tool-use`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase hook pre-tool-use** handles the `PreToolUse` hook event of
*Anthropic Claude Code CLI* and *OpenAI Codex CLI*, resp. the
`preToolUse` hook event of *GitHub Copilot CLI*, which the agent tool
fires before each tool invocation. It is intended to be invoked by the
agent tool internally as a configured hook handler only, and not
directly by end users.

The handler reads the event payload from standard input, a JSON object
with the tool name (field `tool_name`, resp. `toolName` for *GitHub
Copilot CLI*), the tool input (field `tool_input`, resp. `toolArgs`,
either an object or a JSON-encoded string), and the session id (field
`session_id`, resp. `sessionId`). It auto-approves the invocation, so
the user is not prompted for it, if it is one of the following:

- a shell command (tool - `Bash`, resp.- `bash`) starting with the word- `ase`and not containing any of the characters- `;`,- `&`,- `|`,- `<`,- `>`, backtick, or newline, or a- `$(`command substitution;
 
- a skill invocation (tool - `Skill`) of an- *ASE*skill (- `ase-`- *name*,
optionally namespaced as- `ase:ase-`- *name*);
 
- an - *ASE*MCP tool (- `mcp__plugin_ase_ase__`- *tool*for- `claude`,- `ase-`- *tool*for- `copilot`, and- `mcp__ase__`- *tool*for- `codex`);
 
- a tool of one of the pre-defined foreign MCP servers of - `ase setup mcp`(- `chat-alibaba-qwen`,- `chat-deepseek`,- `chat-google-gemini`,- `chat-openai-chatgpt`,- `chat-xai-grok`,- `chat-zai-glm`,- `search-brave`,- `search-exa`, and- `search-perplexity`, named- `mcp__`- *server*- `__`- *tool*resp.- *server*- `-`- *tool*for- `copilot`);
 
- a file read (tool - `Read`) of a file located below the plugin root
directory (e.g., the meta files included by the skills);
 
- a file edit (tool - `Edit`) while one of the edit-capable skills- `ase-code-lint`,- `ase-spec-lint`,- `ase-docs-shorten`,- `ase-docs-refine`, or- `ase-docs-proofread`is recorded as active in the internal session
state of the session.
 

On approval, the handler writes the decision `permissionDecision: allow` with a human-readable `permissionDecisionReason` to standard
output, nested in `hookSpecificOutput` (`claude`), resp. as top-level
fields (`copilot`). Otherwise, it stays silent and hence defers to the
regular permission flow of the agent tool.

For `--tool codex`, the handler only drains its standard input and
stays silent, as *OpenAI Codex CLI* rejects an approval in
`PreToolUse`; the approval is granted by `ase hook permission-request`
instead.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
The agent tool whose event payload is read and whose decision format
is written:- `claude`,- `copilot`, or- `codex`. Default:- `$ASE_TOOL`,
else- `claude`.
 

## ENVIRONMENT

- `CLAUDE_PLUGIN_ROOT`,- `COPILOT_PLUGIN_ROOT`,- `PLUGIN_ROOT`:
The plugin root directory for the auto-approval of plugin file
reads. If unset, such reads are not auto-approved.
 
- `ASE_TOOL`:
The default of- `--tool`.
 

## FILES

- *per-user state directory*- `/session/`- *id*- `/state.yaml`:
The internal session state, whose- `skill`field records the active
edit-capable skill.

## EXIT STATUS

Exits with status `0`, unless `--tool` is invalid (status `1`).

## SEE ALSO

`ase(1)`, `ase-hook(1)`, `ase-hook-permission-request(1)`,
`ase-setup-mcp(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-hook-permission-request(1) - Handle the PermissionRequest Hook Event

## NAME

`ase-hook-permission-request(1)` - Handle the PermissionRequest Hook Event

## SYNOPSIS

`ase hook permission-request`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase hook permission-request** handles the `PermissionRequest` hook
event of *OpenAI Codex CLI*, which the agent tool fires only when a
tool invocation would otherwise require an interactive user approval.
It is intended to be invoked by the agent tool internally as a
configured hook handler only, and not directly by end users. It is
configured for `--tool codex` only, as *OpenAI Codex CLI* does not
honor an approval in its `PreToolUse` event.

The handler reads the event payload from standard input, a JSON object
with the tool name (field `tool_name`), the tool input (field
`tool_input`), and the session id (field `session_id`), and applies the
very same auto-approval rules as `ase hook pre-tool-use` (see
`ase-hook-pre-tool-use(1)`): *ASE* CLI invocations, *ASE* skills, *ASE*
MCP tools, tools of the pre-defined foreign MCP servers, reads of
plugin files, and edits during an active edit-capable skill.

On approval, the handler writes the decision
`{"hookSpecificOutput":{"hookEventName":"PermissionRequest","decision":{"behavior":"allow"}}}`
to standard output. Otherwise, it stays silent and hence defers to the
regular approval flow of the agent tool.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
The agent tool whose event payload is read:- `claude`,- `copilot`, or- `codex`(as configured:- `codex`). Default:- `$ASE_TOOL`, else- `claude`.
 

## ENVIRONMENT

- `PLUGIN_ROOT`,- `CLAUDE_PLUGIN_ROOT`,- `COPILOT_PLUGIN_ROOT`:
The plugin root directory for the auto-approval of plugin file
reads.
 
- `ASE_TOOL`:
The default of- `--tool`.
 

## EXIT STATUS

Exits with status `0`, unless `--tool` is invalid (status `1`).

## SEE ALSO

`ase(1)`, `ase-hook(1)`, `ase-hook-pre-tool-use(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-hook-user-prompt-submit(1) - Handle the UserPromptSubmit Hook Event

## NAME

`ase-hook-user-prompt-submit(1)` - Handle the UserPromptSubmit Hook Event

## SYNOPSIS

`ase hook user-prompt-submit`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase hook user-prompt-submit** handles the `UserPromptSubmit` hook
event of *Anthropic Claude Code CLI* and *OpenAI Codex CLI*, resp. the
`userPromptSubmitted` hook event of *GitHub Copilot CLI*, which the
agent tool fires when the user submits a prompt. It is intended to be
invoked by the agent tool internally as a configured hook handler only,
and not directly by end users.

The handler reads and discards the event payload on standard input
and, when running inside a *tmux* pane, sets the per-pane user option
`@ase_agent_status` to `▶` (busy), so a *tmux* status line can render
the live activity of the agent via `#{@ase_agent_status}`
(independent of the statusline of the agent tool, which is not
repainted during the agent processing). It writes nothing to standard
output.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
The agent tool which fires the event:- `claude`,- `copilot`, or- `codex`. Default:- `$ASE_TOOL`, else- `claude`.
 

## ENVIRONMENT

- `TMUX`,- `TMUX_PANE`:
The- *tmux*pane whose user option- `@ase_agent_status`is set (only if
both are set and non-empty).
 
- `ASE_TOOL`:
The default of- `--tool`.
 

## EXIT STATUS

Exits with status `0`, unless `--tool` is invalid (status `1`).

## SEE ALSO

`ase(1)`, `ase-hook(1)`, `ase-hook-stop(1)`, `tmux(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-hook-stop(1) - Handle the Stop Hook Event

## NAME

`ase-hook-stop(1)` - Handle the Stop Hook Event

## SYNOPSIS

`ase hook stop`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]

## DESCRIPTION

**ase hook stop** handles the `Stop` hook event of *Anthropic Claude
Code CLI* and *OpenAI Codex CLI*, resp. the `agentStop` hook event of
*GitHub Copilot CLI*, which the agent tool fires when the agent
finished responding. It is intended to be invoked by the agent tool
internally as a configured hook handler only, and not directly by end
users.

The handler reads and discards the event payload on standard input
and, when running inside a *tmux* pane, sets the per-pane user option
`@ase_agent_status` to `⏸` (ready), so a *tmux* status line can render
the live activity of the agent via `#{@ase_agent_status}`. It writes
nothing to standard output.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
The agent tool which fires the event:- `claude`,- `copilot`, or- `codex`. Default:- `$ASE_TOOL`, else- `claude`.
 

## ENVIRONMENT

- `TMUX`,- `TMUX_PANE`:
The- *tmux*pane whose user option- `@ase_agent_status`is set (only if
both are set and non-empty).
 
- `ASE_TOOL`:
The default of- `--tool`.
 

## EXIT STATUS

Exits with status `0`, unless `--tool` is invalid (status `1`).

## SEE ALSO

`ase(1)`, `ase-hook(1)`, `ase-hook-user-prompt-submit(1)`, `tmux(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-statusline(1) - Render the Statusline of the Agent Tool

## NAME

`ase-statusline(1)` - Render the Statusline of the Agent Tool

## SYNOPSIS

`ase statusline`
[`-h`|`--help`]
[`-t`|`--tool` *tool*]
[`-w`|`--width` *n*]
[`-m`|`--margin` *n*]
[`-p`|`--padding` *n*]
[`--no-icons`]
[`--no-labels`]
[*line* [...]]

## DESCRIPTION

**ase statusline** renders the statusline of *Anthropic Claude Code
CLI* or *GitHub Copilot CLI* from the JSON payload the agent tool
passes on standard input, and writes the ANSI-colored result on
standard output. It is intended to be configured as the `statusLine`
command in the `settings.json` of *Anthropic Claude Code CLI* resp. as
the `statusLine.command` entry of `~/.copilot/settings.json` of
*GitHub Copilot CLI* (usually via `ase setup statusline activate`, see
`ase-setup-statusline(1)`), and not to be invoked directly by end users.

The output is composed from one or more template *line* arguments,
each rendered as one output line (or more, see **WRAPPING**). A *line*
may contain literal characters, `%`-prefixed placeholders (see
**PLACEHOLDERS**), and color markup (see **COLORS**). Without any
*line*, the single default line `%m %e %t` is rendered.

Each placeholder is rendered as an optional icon glyph, an optional
label followed by `: `, and the value in bold, e.g., `⚙ model: `**Opus**. A `%` followed by any other character is emitted literally.

Under `--tool copilot`, the top-level `cwd` field of the payload is
mapped onto `workspace.current_dir`, and, as *GitHub Copilot CLI*
passes no `effort` field but embeds the reasoning effort as a `·`
separated segment of `model.display_name` (e.g., `gpt-5.4 (2x) · high · 1M context`), the segment matching a known effort level (`low`,
`medium`, `high`, `xhigh`, or `max`) is split off again, so `%m`
renders the plain model and `%e` the effort.

When run inside a *tmux*(1) pane (i.e., with `TMUX` and `TMUX_PANE`
set), the resolved task id is additionally published as the per-pane
user option `@ase_task_id` of the pane, so external tools can pick it
up via `#{@ase_task_id}`.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--tool`- *tool*]:
Select the agent tool producing the payload:- `claude`(- *Anthropic
Claude Code CLI*) or- `copilot`(- *GitHub Copilot CLI*) (default:
the value of- `ASE_TOOL`, else- `claude`). The value- `codex`is
rejected with an error, as- *OpenAI Codex CLI*has no scriptable
custom statusline mechanism, and any other value is invalid.
 
- [- `-w`|- `--width`- *n*]:
Force the terminal width to- *n*characters, a non-negative integer
(default:- `0`, i.e., auto-detect the width of the controlling
terminal via- `tput cols`on- `/dev/tty`, as standard output is a pipe
under the agent tool).
 
- [- `-m`|- `--margin`- *n*]:
Reduce the maximum used terminal width by- *n*characters on each
side, a non-negative integer (default:- `2`). The margin only shrinks
the width budget and emits no characters.
 
- [- `-p`|- `--padding`- *n*]:
Pad each non-empty rendered line with- *n*spaces on the left and
right side, a non-negative integer (default:- `0`). In contrast to the
margin, the padding spaces are actually emitted (and hence also
shrink the width budget). Empty lines stay unpadded. Under- `--tool copilot`, a padded output is prefixed with a zero-width space, so the
first line keeps its indentation.
 
- [- `--no-icons`]:
Disable the leading icon glyph (e.g.,- `※`,- `⚑`, or- `⚙`) of each
placeholder rendering.
 
- [- `--no-labels`]:
Disable the textual label (e.g.,- `user:`,- `project:`, or- `model:`)
in front of the bold value of each placeholder rendering.
 

## ARGUMENTS

- [*line*[...]]:
The template lines (default:`%m %e %t`). Each argument yields its
own output line. Quote each line for the shell.

## PLACEHOLDERS

The following placeholders are recognized. Unless stated otherwise, a
placeholder is always rendered. A placeholder marked as *suppressed* is
silently omitted (including its icon and label) if its source is
missing, e.g., in older agent tool versions or in synthetic payloads.

Scope:

- `%u`:
The user (- `USER`, else- `LOGNAME`, else- `unknown`), with icon- `※`.
 
- `%p`:
The project id, derived from the basename of the current directory
of the workspace (- `workspace.current_dir`), with icon- `⚑`.
 
- `%T`:
The active task id of the session (from its internal session state),
else- `ASE_TASK_ID`, else- `default`, with icon- `◉`.
 
- `%s`:
The session id (- `session_id`, else- `unknown`), with icon- `⏻`.
 

Model:

- `%m`:
The model (- `model.display_name`), with icon- `⚙`.
 
- `%e`:
The reasoning effort (- `effort.level`, else the effort segment of the- *GitHub Copilot CLI*model, else- `unknown`), with icon- `⚒`.
 
- `%t`:
Whether extended thinking is enabled (- `thinking.enabled`), as- `yes`or- `no`, with icon- `⚛`.
 

Output:

- `%O`:
The output style (- `output_style.name`), with icon- `≡`,- *suppressed*.
 
- `%P`:
The persona style (- `agent.persona`), with icon- `☯`,- *suppressed*if
empty.
 
- `%h`:
The guidance level (- `agent.guidance`), with icon- `▶`,- *suppressed*if
empty.
 
- `%B`:
The project boxing transparency (- `project.boxing`), with icon- `▢`,- *suppressed*if empty.
 
- `%L`:
The project task lifecycle model (- `project.task.lifecycle`), with
icon- `⟳`,- *suppressed*if empty.
 - The values of - `%P`,- `%h`,- `%B`, and- `%L`are resolved from the- *ASE*configuration cascade of the session (- `session:`- *id*), falling back
to the environment variables- `ASE_PERSONA_STYLE`,- `ASE_GUIDANCE_LEVEL`,- `ASE_PROJECT_BOXING`, and- `ASE_PROJECT_TASK_LIFECYCLE`.
 

Context:

- `%c`:
The context usage (- `context_window.used_percentage`) as a progress
bar of 20 cells (- `█`filled,- `░`empty) plus the percentage, with
icon- `◔`.
 
- `%C`:
The current and the maximum context tokens, e.g.,- `334.0k/1.0M`, with
icon- `◆`. The current tokens are taken from- `context_window.current_context_tokens`, else- `last_call_input_tokens`, else- `total_input_tokens`; the maximum from- `context_window.displayed_context_limit`, else- `context_window_size`, else derived from the current tokens and the
used percentage.- *Suppressed*without a maximum.
 

Rate limits:

- `%S`:
The used percentage of the 5-hour rate-limit window, with icon- `⏲`,
or- `-`if unknown.
 
- `%D`:
The time until the reset of the 5-hour window, e.g.,- `4hr 27m`, with
icon- `⏱`, or- `-`if unknown.
 
- `%W`:
The used percentage of the 7-day rate-limit window, with icon- `⏲`,
or- `-`if unknown.
 
- `%Q`:
The time until the reset of the 7-day window, e.g.,- `6d 12hr 7m`,
with icon- `⏱`, or- `-`if unknown.
 - The windows are taken from - `rate_limits.five_hour`and- `rate_limits.seven_day`of the payload. If the payload carries none,
the model matches- `gpt-*`, and a template references one of these
placeholders, the windows are instead fetched from the- *ChatGPT*backend with the credential stored by- *OpenAI Codex CLI*(- `$CODEX_HOME/auth.json`, default- `~/.codex/auth.json`), with a
timeout of 3 seconds, and cached (also on failure) for 60 seconds.
 

Costs:

- `%H`:
The wall-clock duration of the session (- `cost.total_duration_ms`) as
hours and minutes, e.g.,- `92hr 40m`, with icon- `⏱`,- *suppressed*if
zero or missing.
 
- `%X`:
The session costs in USD (- `cost.total_cost_usd`), e.g.,- `$54.44`,
with icon- `$`,- *suppressed*.
 

Version control:

- `%a`:
The lines added in this session (- `cost.total_lines_added`, else- `0`), with icon- `⊕`.
 
- `%r`:
The lines removed in this session (- `cost.total_lines_removed`, else- `0`), with icon- `⊖`.
 
- `%b`:
The Git branch of the workspace directory, or- `no git`outside of a
Git working tree, with icon- `⎇`.
 
- `%g`:
The changed lines of the Git working tree against- `HEAD`, e.g.,- `+42/-7`, with icon- `±`,- *suppressed*outside of a Git working tree.
 
- `%G`:
The number of untracked files of the Git working tree, with icon- `⁈`,- *suppressed*outside of a Git working tree.
 
- `%d`:
The full path of the workspace directory, with icon- `▶`,- *suppressed*.
 

Resources and versions:

- `%M`:
The used and total memory of the host, e.g.,- `33.2G/64.0G`, with icon- `⛁`.
 
- `%V`:
The agent tool and- *ASE*versions (- `version`of the payload), e.g.,- `claude/2.1.90 ase/1.0.7`, with icon- `⎈`.
 

## COLORS

A *line* may contain the markup `<`*color*`>`...`</`*color*`>` to
colorize its text (including rendered placeholders), where *color* is
one of `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`,
`white`, or `default`. A closing tag (of any recognized color name)
ends the colored span, returning to the default foreground color.
Spans do not nest: an opening tag inside a span is removed, but
ignored. A span still open at the end of a line ends there. A tag with
an unrecognized color name is kept literally in the output.

## WRAPPING

Each rendered line is automatically wrapped before the next chunk (a
literal character or a whole placeholder rendering) which would exceed
the width budget, where the budget is the terminal width (see
`--width`) reduced by `2 *` (*margin* `+` *padding*) characters. Without
a known terminal width, no wrapping happens.

## INPUT

The input is the standard statusline JSON payload of the agent tool,
of which the fields `workspace.current_dir` (resp. `cwd` for *GitHub
Copilot CLI*), `session_id`, `version`, `model.display_name`,
`effort.level`, `thinking.enabled`, `output_style.name`,
`context_window.*`, `rate_limits.five_hour` and
`rate_limits.seven_day` (each with `used_percentage` and `resets_at`),
and `cost.total_cost_usd`, `cost.total_duration_ms`,
`cost.total_lines_added`, and `cost.total_lines_removed` are used.

Invalid JSON is an error.

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.
 
- `ASE_TASK_ID`,- `ASE_PERSONA_STYLE`,- `ASE_GUIDANCE_LEVEL`,- `ASE_PROJECT_BOXING`,- `ASE_PROJECT_TASK_LIFECYCLE`:
The fallbacks of- `%T`,- `%P`,- `%h`,- `%B`, and- `%L`.
 
- `USER`,- `LOGNAME`:
The user of- `%u`.
 
- `CODEX_HOME`:
The configuration directory of- *OpenAI Codex CLI*(default:- `~/.codex`), holding the credential- `auth.json`for the rate limits
of the- `gpt-*`models.
 
- `TMUX`,- `TMUX_PANE`:
The- *tmux*pane to publish the task id- `@ase_task_id`to.
 

## FILES

- *per-user state directory*- `/statusline-codex-usage.json`:
The cache of the rate-limit windows fetched for the- `gpt-*`models.
 
- *per-user state directory*- `/session/`- *id*- `/config.yaml`and- `state.yaml`:
The configuration and the internal state (active task) of the
session.
 

## EXIT STATUS

`0` on success, `1` on invalid JSON, an invalid option value, or
`--tool codex`.

## EXAMPLES

```
# render two lines from a sample payload
$ echo '{"model":{"display_name":"Opus"},"context_window":{"used_percentage":42}}' | \
  ase statusline "%m %e %t" "<blue>%c</blue>"
# a compact statusline without icons and labels, padded by one space
$ ase statusline --no-icons --no-labels -p 1 "%p %T %b %g" "%m %c %X"
```

## SEE ALSO

`ase(1)`, `ase-setup-statusline(1)`, `ase-config(1)`, `tmux(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task(1) - Manage Persisted Task Plans

## NAME

`ase-task(1)` - Manage Persisted Task Plans

## SYNOPSIS

`ase task`
[`-h`|`--help`]
*command*
[*options* [...]]
[*args* [...]]

## DESCRIPTION

**ase task** manages the persisted *task plans* of the current project.
A task plan is a Markdown document in the textual *ASE* task format
(see `ase util meta format-task`): a frontmatter block with the keys
`Type`, `Id`, `Created`, `Modified`, `Group`, `Phase`, `After`,
`Status`, `Assignee`, `Kind`, `Effort`, `Tags`, `Source`, `Changeset`,
and `Target`, followed by the plan body and optional attachment blocks.

The optional `Effort` key carries the estimated effort of the task as
*size* *hash*, with *size* one of the T-shirt sizes `XS`, `S`, `M`, `L`,
or `XL` (see `project.task.effort.scale` in `ase-config(1)`) and *hash*
the content hash of the plan body the size was given for. The task
store drops a *stale* `Effort` (whose *hash* no longer matches the
body, ignoring checkboxes, whitespace, and cancelled or deferred
bullet-points) on load and save, and stamps an `Effort` given without
*hash* with the content hash of the body on save.

All commands are forwarded to the *task store* selected by the
`project.task.store.url` configuration (see **TASK STORES** below). The
project is identified in the task store by `project.id` (default: the
sanitized basename of the project root, i.e., the Git top-level
directory, or the current working directory outside a Git working
tree).

The `Status` of the task plans follows the *task lifecycle model* of
the project (see **TASK LIFECYCLE MODELS** below), and new task ids
are generated after the *task id scheme* of the project (see
**TASK ID SCHEMES** below).

Without a *command*, the usage information is shown and the command
exits with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information of`ase task`, including the list of all
sub-commands, and exit.

## COMMANDS

- `ase task list`[- `-v`|- `--verbose`] [- `-i`|- `--include`- *states*] [- `-e`|- `--exclude`- *states*]:
List the persisted task ids. See- `ase-task-list(1)`.
 
- `ase task status`[- *id*[- `:`]] [- *status*]:
Get or set the lifecycle status of a task. See- `ase-task-status(1)`.
 
- `ase task request`- *id*- *name*[- *prompt*...]:
Request the processing of a task by a prompt. See- `ase-task-request(1)`.
 
- `ase task assign`[- `-s`|- `--session`- *id*] [- *id*[- `:`]]- *assignee*:
Assign a task (and handle the AI usage window of a session). See- `ase-task-assign(1)`.
 
- `ase task load`- *id*:
Write a task plan to standard output. See- `ase-task-load(1)`.
 
- `ase task view`- *id*:
Show a task plan in the pager. See- `ase-task-view(1)`.
 
- `ase task edit`- *id*:
Edit a task plan in the editor. See- `ase-task-edit(1)`.
 
- `ase task save`[- `-c`|- `--create`]- *id*:
Save a task plan read from standard input. See- `ase-task-save(1)`.
 
- `ase task delete`- *id*:
Delete a task plan. See- `ase-task-delete(1)`.
 
- `ase task rename`- *old-id*- *new-id*:
Rename a task plan and its references. See- `ase-task-rename(1)`.
 
- `ase task newid`[- `-p`|- `--proposal`- *id*] [- `-t`|- `--taken`- *ids*] [- `-v`|- `--verbose`] [- *title*]:
Print the next free task id. See- `ase-task-newid(1)`.
 
- `ase task purge`[- *age*]:
Remove all task plans older than an age. See- `ase-task-purge(1)`.
 
- `ase task lifecycle`[- *name*]:
Get or set the task lifecycle model of the project. See- `ase-task-lifecycle(1)`.
 
- `ase task idscheme`[- *scheme*]:
Get or set the task id scheme of the project. See- `ase-task-idscheme(1)`.
 
- `ase task store`- *command*[...]:
Manage the per-user task store REST API server. See- `ase-task-store(1)`.
 
- `ase task board`[- *options*]:
Show the interactive task board in the terminal or the browser. See- `ase-task-board(1)`.
 

## TASK IDS

A task id consists of the characters `A-Z`, `a-z`, `0-9`, `#`, `_`,
and `-`, where it has to start with a letter, a digit, or `#`, i.e.,
it has to match `[A-Za-z0-9#][A-Za-z0-9#_-]*`. The id `default` is the
reserved id of the default task of a session. As `#` starts a comment
in the shell, an id containing `#` has to be quoted on the command
line, e.g., `ase task view "#7"`.

## TASK STORES

The task store is selected by the `project.task.store.url`
configuration URL (see `ase-config(1)`):

- `ase:`- *path*:
The built-in file-based storage plugin, running in-process, storing
each plan as a file- `TASK-`- *id*- `.md`directly in the directory- *path*, which is resolved relative to the project root (default:- `ase:./.ase/task`).
 - On the - `project`and- `task:`- *id*scopes,- *path*must not escape the
project root (also not through symbolic links), as a cloned
repository could otherwise direct task writes anywhere; a directory
outside of it is accepted on the- `user`or- `session`scope only. The
lifecycle model and task id scheme of a local task store always
follow- `project.task.lifecycle`and- `project.task.idscheme`.
 
- `ase://`- *addr*- `:`- *port*[- `/`- *token*]:
A remote task store server (see- `ase-task-store(1)`), connected via
HTTP. It requires an explicitly configured- `project.id`, as a
basename-derived one likely collides on a shared server.
 - On first use, the project is registered in the task store under the
configured - `project.task.lifecycle`and- `project.task.idscheme`;
afterwards the registered model and scheme apply to all clients of
the project, and a deviating configuration is warned about once per
deviation only (see- `ase-task-lifecycle(1)`).
 - The bearer token is the embedded - *token*(warned about if the URL is
configured on the- `project`or a- `task:`- *id*scope, as it might be
committed), else- `$ASE_TASK_STORE_TOKEN`, else- `project.task.store.token`(warned about if not configured on the- `user`scope), else the- `token`of the per-user- `store.yaml`of a
locally started task store server (only if the URL addresses exactly
this server). Sending a token to a non-loopback host selected on the- `project`or a- `task:`- *id*scope is warned about.
 
- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`]:
The same as- `ase://`, but connecting via HTTPS, verifying the server
certificate against the Node.js CA store (extendable via- `$NODE_EXTRA_CA_CERTS`). With the suffix- `?insecure`, the
verification of the server certificate is skipped.
 
- `github:`- *owner*- `/`- *repo*,- `github+https://`- *host*- `/`- *owner*- `/`- *repo*, or- `github+http://`- *host*- `/`- *owner*- `/`- *repo*:
The built-in GitHub storage plugin, running in-process, storing the
plans as the issues of the repository- *owner*- `/`- *repo*on GitHub
resp. on the GitHub (Enterprise) instance- *host*. It requires a- `seq`task id scheme (e.g.,- `seq:#%d`), as the task ids are the issue
numbers. The GitHub token is- `project.task.store.token`, else- `$GITHUB_TOKEN`, else- `$GH_TOKEN`.
 
- `gitlab:`- *namespace*- `/`- *project*,- `gitlab://`- *host*- `/`- *namespace*- `/`- *project*,- `gitlab+https://`- *host*- `/`- *namespace*- `/`- *project*, or- `gitlab+http://`- *host*- `/`- *namespace*- `/`- *project*:
The built-in GitLab storage plugin, running in-process, storing the
plans as the issues of the GitLab project- *namespace*- `/`- *project*(with nested groups allowed) on the GitLab instance- *host*, else- `$GITLAB_HOST`, else- `gitlab.com`. It also requires a- `seq`task id
scheme. The GitLab token is- `project.task.store.token`, else- `$GITLAB_TOKEN`.
 
- `gitea+https://`- *host*- `/`- *owner*- `/`- *repo*or- `gitea+http://`- *host*- `/`- *owner*- `/`- *repo*:
The built-in Gitea storage plugin, running in-process, storing the
plans as the issues of the repository- *owner*- `/`- *repo*on the Gitea
instance- *host*. It also requires a- `seq`task id scheme. The Gitea
token is- `project.task.store.token`, else- `$GITEA_TOKEN`.
 

For the GitHub, GitLab, and Gitea task stores, a repository selected
on the `project` or a `task:`*id* scope is warned about once per
project and repository only (remembered in the *per-user state
directory* as `task-github.json`, `task-gitlab.json`, resp.
`task-gitea.json`), and a `project.task.store.token` not configured on
the `user` scope is warned about.

## TASK LIFECYCLE MODELS

The `Status` frontmatter key of a task plan holds a state of the task
lifecycle model of the project, selected by `project.task.lifecycle`
for a local task store resp. registered in a remote task store (see
`ase-task-lifecycle(1)`). A plan without a `Status` key reads as the
*initial* state of the model.

A status change has to be reachable from the current state via one or
more transitions of the model (a status foreign to the model can be
changed to any state, to allow its correction). The *finished* states
end the lifecycle, and in the *activity* states a task is actively
worked on (by a human or an agent), so its plan must not be changed
concurrently.

- `solo`:
The single-phase lifecycle for a local solo developer (default).
States:- `OPEN`(initial),- `SHELVED`,- `CLOSED`(finished), and- `CANCELLED`(finished); no activity states. Transitions:- `OPEN`to- `SHELVED`,- `CLOSED`, or- `CANCELLED`;- `SHELVED`to- `OPEN`or- `CANCELLED`.
 
- `team`:
The two-phase lifecycle for a distributed small team. States:- `PLANNING`(initial),- `SHELVED`,- `IMPLEMENTING`(activity),- `STALLED`,- `IMPLEMENTED`(finished), and- `CANCELLED`(finished).
Transitions:- `PLANNING`to- `SHELVED`,- `IMPLEMENTING`, or- `CANCELLED`;- `SHELVED`to- `PLANNING`or- `CANCELLED`;- `IMPLEMENTING`to- `PLANNING`,- `STALLED`,- `IMPLEMENTED`, or- `CANCELLED`;- `STALLED`to- `IMPLEMENTING`or- `CANCELLED`.
 
- `enterprise`:
The four-phase lifecycle (planning, implementation, qualification, and
integration) for a gated enterprise pipeline, driven headless by the
agents of- `ase-agent(1)`. States:- `DRAFTED`(initial),- `SHELVED`,- `PLANNING`(activity),- `PLANNED`,- `STALLED`,- `IMPLEMENTING`(activity),- `IMPLEMENTED`,- `DECLINED`,- `QUALIFYING`(activity),- `QUALIFIED`,- `DEFERRED`,- `INTEGRATING`(activity),- `INTEGRATED`(finished), and- `CANCELLED`(finished).
 - Transitions: - `DRAFTED`to- `SHELVED`,- `PLANNING`, or- `CANCELLED`;- `SHELVED`to- `DRAFTED`or- `CANCELLED`;- `PLANNING`to- `DRAFTED`,- `SHELVED`,- `PLANNED`, or- `CANCELLED`;- `PLANNED`to- `STALLED`,- `IMPLEMENTING`, or- `CANCELLED`;- `STALLED`to- `PLANNED`or- `CANCELLED`;- `IMPLEMENTING`to- `DRAFTED`,- `PLANNED`,- `STALLED`,- `IMPLEMENTED`, or- `CANCELLED`;- `IMPLEMENTED`to- `DECLINED`,- `QUALIFYING`, or- `CANCELLED`;- `DECLINED`to- `IMPLEMENTED`or- `CANCELLED`;- `QUALIFYING`to- `DRAFTED`,- `PLANNED`,- `IMPLEMENTED`,- `DECLINED`,- `QUALIFIED`, or- `CANCELLED`;- `QUALIFIED`to- `DEFERRED`,- `INTEGRATING`, or- `CANCELLED`;- `DEFERRED`to- `QUALIFIED`or- `CANCELLED`;- `INTEGRATING`to- `DRAFTED`,- `PLANNED`,- `QUALIFIED`,- `DEFERRED`,- `INTEGRATED`, or- `CANCELLED`.
 

In all models, no transition leaves a finished state.

## TASK ID SCHEMES

New task ids are generated by `ase-task-newid(1)` after the task id
scheme of the project, selected by `project.task.idscheme` for a local
task store resp. registered in a remote task store (see
`ase-task-idscheme(1)`):

- `slug`[- `:`- *words*]:
The slug of the first- *words*(- `1`-- `16`, default:- `2`) words of the
task title, starting with the first word beginning with a letter,
with diacritics stripped, lower-cased, and joined with- `-`, e.g.,- `json-export`(default scheme). Conforming ids match- `^[a-z][a-z0-9]*(?:-[a-z0-9]+){0,`- *words-1*- `}(?:-[0-9]+)?$`.
 
- `seq`[- `:`- *template*]:
A continuously increasing sequence number, rendered through the
sprintf-style- *template*(default:- `%d`) with exactly one- `%d`or- `%0`- *width*- `d`(- *width*- `0`-- `16`) and otherwise only the characters- `A-Z`,- `a-z`,- `#`,- `_`, and- `-`(but not starting with- `_`or- `-`),
e.g.,- `FOO-%03d`(- `FOO-007`) or- `#%d`(- `#7`). Conforming ids match- *prefix*- `([0-9]+)`- *suffix*(resp. with at least- *width*digits).
 
- `any`:
An arbitrary id, as proposed by the caller. Conforming ids match- `^[A-Za-z0-9#][A-Za-z0-9#_-]*$`.
 

An id not conforming to the scheme is accepted, but warned about on
`ase task save`, `ase task edit`, and `ase task rename` (the reserved
id `default` always conforms).

## ENVIRONMENT

- `ASE_TASK_ID`:
The default task id of- `ase task status`and- `ase task assign`(default:- `default`).
 
- `ASE_TASK_STORE_TOKEN`:
The bearer token of a remote (- `ase://`or- `ases://`) task store.
 
- `GITHUB_TOKEN`,- `GH_TOKEN`,- `GITLAB_TOKEN`,- `GITLAB_HOST`,- `GITEA_TOKEN`:
The tokens resp. the default host of the GitHub, GitLab, and Gitea
task stores (see- **TASK STORES**).
 

## FILES

- `.ase/task/TASK-`- *id*- `.md`:
The persisted task plans of the default local task store- `ase:./.ase/task`, relative to the project root.
 
- `.ase/config.yaml`:
The- `project`scope configuration with the- `project.id`,- `project.task.store.url`,- `project.task.lifecycle`, and- `project.task.idscheme`keys.
 
- *per-user state directory*- `/task-lifecycle.json`:
The deviations of the configured lifecycle model and task id scheme
from the ones of a remote task store already warned about.
 
- *per-user state directory*- `/task-github.json`,- `task-gitlab.json`,- `task-gitea.json`:
The repositories of the issue tracker task stores already warned
about.
 

## EXAMPLES

```
# list all unfinished tasks with their status, time, effort, and title
$ ase task list -v
# create a new task from a title and a plan file
$ id=$(ase task newid "Export as JSON")
$ ase task save --create "$id" <plan.md
# move the task forward in the lifecycle
$ ase task status "$id" CLOSED
# use a shared remote task store
$ ase config --scope project set project.id myproject
$ ase config --scope project set project.task.store.url ase://127.0.0.1:4711
```

## SEE ALSO

`ase(1)`, `ase-task-list(1)`, `ase-task-status(1)`,
`ase-task-request(1)`, `ase-task-assign(1)`, `ase-task-load(1)`,
`ase-task-view(1)`, `ase-task-edit(1)`, `ase-task-save(1)`,
`ase-task-delete(1)`, `ase-task-rename(1)`, `ase-task-newid(1)`,
`ase-task-purge(1)`, `ase-task-lifecycle(1)`, `ase-task-idscheme(1)`,
`ase-task-store(1)`, `ase-task-board(1)`, `ase-config(1)`,
`ase-agent(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-list(1) - List Persisted Task Ids

## NAME

`ase-task-list(1)` - List Persisted Task Ids

## SYNOPSIS

`ase task list`
[`-h`|`--help`]
[`-v`|`--verbose`]
[`-i`|`--include` *states*]
[`-e`|`--exclude` *states*]

## DESCRIPTION

**ase task list** lists the ids of the persisted task plans of the
current project in the task store (see `ase-task(1)`), one per line,
in natural order (numbers ordered by their value, e.g., `FOO-2` before
`FOO-10`).

The listed tasks are restricted to those whose lifecycle status
(the `Status` frontmatter key, or the initial state of the task
lifecycle model if absent) is in the effective state set of the
`--include` and `--exclude` options. By default, all tasks except the
ones in a *finished* state of the task lifecycle model are listed. A
task whose status is unknown to the task lifecycle model is always
listed, but warned about.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-v`|- `--verbose`]:
Annotate each task id with the lifecycle status of its plan, the
modification time of its plan as- `(YYYY-MM-DD HH:MM)`, its effort
size (the T-shirt size- `XS`,- `S`,- `M`,- `L`, or- `XL`of the valid- `Effort`frontmatter key of the plan, else- `-`), and its title (the
text after- `TASK:`of the first- `#   TASK:`heading of the plan body,
else empty), all separated by a single tab character.
 
- [- `-i`|- `--include`- *states*]:
Restrict the listing to the tasks in one of the- *states*, a
comma-separated list of lifecycle states of the task lifecycle model
of the project (given case-insensitively), where the term- `finished`expands to the finished states of the model and the term- `none`(as
well as an empty term) is ignored. An empty list (default:- `none`)
means all states.
 - The states of the models are - `OPEN`,- `SHELVED`,- `CLOSED`, and- `CANCELLED`(- `solo`),- `PLANNING`,- `SHELVED`,- `IMPLEMENTING`,- `STALLED`,- `IMPLEMENTED`, and- `CANCELLED`(- `team`), and- `DRAFTED`,- `SHELVED`,- `PLANNING`,- `PLANNED`,- `STALLED`,- `IMPLEMENTING`,- `IMPLEMENTED`,- `DECLINED`,- `QUALIFYING`,- `QUALIFIED`,- `DEFERRED`,- `INTEGRATING`,- `INTEGRATED`, and- `CANCELLED`(- `enterprise`). A state
which is not one of the model is an error.
 
- [- `-e`|- `--exclude`- *states*]:
Exclude the tasks in one of the- *states*from the listing, a
comma-separated list of the same form as for- `--include`, applied
after- `--include`(default:- `finished`, i.e., the finished states of
the model). Use- `none`to exclude nothing. If- `--include`and- `--exclude`cancel out to an empty state set, this is an error.
 

## EXIT STATUS

`0` on success, `1` on an error, like an invalid state, an empty
effective state set, or an unreachable task store.

## EXAMPLES

```
# list all unfinished tasks
$ ase task list
# list all tasks, including the finished ones, with details
$ ase task list -v -e none
# list the tasks waiting for their qualification
$ ase task list -i IMPLEMENTED,DECLINED
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-status(1)`, `ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-status(1) - Get or Set the Lifecycle Status of a Task

## NAME

`ase-task-status(1)` - Get or Set the Lifecycle Status of a Task

## SYNOPSIS

`ase task status`
[`-h`|`--help`]
[*id*[`:`]]
[*status*]

## DESCRIPTION

**ase task status** gets or sets the lifecycle status, i.e., the
`Status` frontmatter key, of the task plan with the given *id* in the
task store (see `ase-task(1)`).

Without *status*, the current status is printed to standard output,
where a plan without a `Status` key reads as the initial state of the
task lifecycle model of the project.

With *status*, the status is set and the change is logged as
`task: set status of "`*id*`" from "`*from*`" to "`*to*`"`. The
*status* is given case-insensitively and has to be a state of the task
lifecycle model of the project, which is reachable from the current
status via one or more transitions of the model (see
**TASK LIFECYCLE MODELS** in `ase-task(1)`); a current status foreign
to the model can be changed to any state, to allow its correction.

Only the frontmatter is changed: the `Modified` key is left alone, as
it tracks the changes of the plan body only.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- [- *id*[- `:`]]:
The id of the task (default:- `$ASE_TASK_ID`, else- `default`). A
trailing colon is stripped, which allows to mark a single argument
explicitly as the- *id*: if only a single argument is given, it is
taken as- *status*if it is a state of the task lifecycle model
(case-insensitively) and has no trailing colon, else as- *id*.
 
- [- *status*]:
The lifecycle status to set.
 

## ENVIRONMENT

- `ASE_TASK_ID`:
The default task id (default:- `default`).

## EXIT STATUS

`0` on success, `1` if no such task exists, the *status* is not a
state of the task lifecycle model, or it is not reachable from the
current status.

## EXAMPLES

```
# print the status of the task "json-export"
$ ase task status json-export:
# move the task "json-export" into "SHELVED"
$ ase task status json-export shelved
# close the task of $ASE_TASK_ID
$ ase task status CLOSED
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-lifecycle(1)`, `ase-task-list(1)`,
`ase-task-save(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-request(1) - Request the Processing of a Task by a Prompt

## NAME

`ase-task-request(1)` - Request the Processing of a Task by a Prompt

## SYNOPSIS

`ase task request`
[`-h`|`--help`]
*id*
*name*
[*prompt* ...]

## DESCRIPTION

**ase task request** requests the processing of the task *id* by the
*prompt* under the request name *name*, for an agent processing the
requests of this name (see `ase-agent(1)`): `edit` for the edit
requests of `ase agent editor` (the same as key `E` of the task board,
see `ase-task-board(1)`), `estimate` for the (re-)estimation requests
of `ase agent estimator --mode request` (see `ase-agent-estimator(1)`),
`research` for the research requests of `ase agent researcher` (see
`ase-agent-researcher(1)`), and any other name for the requests of a
request agent of a definition kind or of `ase agent adhoc` (with
`--type request` and `--request-name` *name*).

The request is attached to the task plan as an attachment of the type
`text/plain; charset=utf-8; kind="`*name*`"`, with the prompt as its
data, its whitespace-collapsed first 60 characters as its description
(resp. *name*` request without prompt` for an empty prompt), and its
creation and modification timestamps. The aggregated state of
the requests of the name is mirrored in the `Tags` frontmatter key of
the task as the tag `ase-agent-`*name*`:pending` (resp.
`ase-agent-editor:pending` for `edit`), as the agents and the task
boards see the task headers only.

While being processed, the request turns into `kind="`*name*`-running"`
(tag value `active`), and on a failure into `kind="`*name*`-failed"` (tag
value `failed`). A new request replaces all failed requests of the name.

The request is refused for a missing task, for a task in a finished
state of the task lifecycle model, for an empty prompt of an `edit`
request, and while a request of the name is being processed, as adding
a request would change the task plan under the agent processing the
running one. The prompt of any other request may be empty, for the
request agents ignoring the prompts of their requests (like `ase agent estimator --mode request`) or taking them as optional (like `ase agent researcher`, which then researches the task plan itself), while a
request agent consuming the prompts (by the placeholder `{prompt}` of
its prompt template, or by its hooks `prepare`, `prompt`, or `finish`,
see `ase-agent-adhoc(1)`) turns a request with an empty prompt into a
failed one.

The MCP tool `ase_task_request` provides the same functionality.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- *id*:
The id of the task.
 
- *name*:
The request name: a lower-case word of letters, digits, and dashes,
starting with a letter, not ending in- `-running`or- `-failed`, and
none of the built-in agent kinds- `rebooter`,- `condenser`,- `implementer`,- `qualifier`,- `integrator`,- `editor`,- `estimator`, and- `researcher`(as
its tag- `ase-agent-`- *name*is the one of a built-in agent). The name- `edit`selects the edit requests of- `ase agent editor`, the name- `estimate`the estimation requests of- `ase agent estimator --mode request`, and the name- `research`the research requests of- `ase agent researcher --mode request`.
 
- [- *prompt*...]:
The prompt of the request, with all its words joined by single
spaces (and leading and trailing whitespace removed), where an
omitted prompt is an empty one (refused for- `edit`).
 

## EXIT STATUS

`0` on success, `1` if the request name is invalid or the request is
refused.

## EXAMPLES

```
# request an edit of the plan of task "json-export"
$ ase task request json-export edit "Also support JSON5 output"
# request a lint report from a request agent of name "lint"
$ ase task request json-export lint focus on error handling
# request a re-estimation of the effort (without a prompt)
$ ase task request json-export estimate
# request a research of a topic for the task
$ ase task request json-export research streaming JSON serialization
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-board(1)`, `ase-agent(1)`,
`ase-agent-editor(1)`, `ase-agent-researcher(1)`, `ase-agent-adhoc(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-assign(1) - Assign a Task

## NAME

`ase-task-assign(1)` - Assign a Task

## SYNOPSIS

`ase task assign`
[`-h`|`--help`]
[`-s`|`--session` *id*]
[*id*[`:`]]
*assignee*

## DESCRIPTION

**ase task assign** assigns the task plan with the given *id* in the
task store (see `ase-task(1)`) by setting its `Assignee` frontmatter
key to *assignee*, where `me` stands for the current user and `none`
removes the key. The change is done in a conditional save of the task
plan (retried on a concurrent change of the task), leaving the
`Modified` key alone, as it tracks the changes of the plan body only.
The change is logged as `task: assigned "`*id*`" from "`*from*`" to "`*to*`"`.

If the configuration key `project.users` (see `ase-config(1)`) lists
the known users, an *assignee* which is neither `none`, the current
user, one of these users, nor the name of an agent present at the
agent relay (see `ase-agent-relay(1)`) is still assigned, but with the
warning `task: unknown assignee "`*assignee*`" (neither one of the users of "project.users" nor a known agent)` (with `--session`
reported as part of its notice).

With `--session`, the assignment is instead done on behalf of an agent
tool session through the running `ase service` of the project (see
`ase-service(1)`), which handles the *AI usage window* of the task, too
(only if the AI usage accounting is enabled via `telemetry.enabled`,
else a notice is reported):

- Assigning the task to oneself opens its usage window (or attaches
the session to an already open one). 
- Assigning the task to anybody else closes its usage window and,
after a grace period of 10 seconds for late telemetry, atomically
adds the usage accumulated from the OTLP records of its sessions
(see - `ase-setup-telemetry(1)`) to the YAML data of the usage
attachment of the task (- `Type: application/yaml; kind="usage"`,
created on the first booking):- `time.ai`(the AI request time in
minutes),- `time.hr`(the human time in minutes, i.e., the whole real
time of the window),- `time.real`(the wall-clock time of the window
in minutes),- `token.cread`,- `token.cwrite`,- `token.input`, and- `token.output`(the disjoint token counts of the cache reads, cache
writes, uncached input, and output),- `cost.ai`(the AI costs in USD,
as reported by the agent tool, else derived from- `telemetry.prices.ai`), and- `cost.hr`(the human costs in USD, at the
hourly rate of the assignee in- `telemetry.prices.hr`).
 

A usage window also closes when its last session ends, and when the
`Assignee` of its task is changed elsewhere while the task is the
active task of an attached session. Conversely, once the `Assignee` of
a task is changed to oneself elsewhere (e.g., on the task board), the
service opens its window (starting *paused*) and attaches every session
of the project whose active task it is.

Unless disabled via `telemetry.implicit`, the service also accounts an
*unassigned* task *implicitly*: it opens its window (starting
*paused*) and attaches every interactive session of the project whose
active task it is, without setting the `Assignee`, but only as long as
the task stays the active task of the session and unassigned (task
`default` excluded). Assigning such a task to oneself turns the
implicit attachment into a regular one, and unassigning a task
assigned to oneself elsewhere turns its attachments into implicit ones.

Open windows additionally book every `telemetry.service.interval`
minutes, pause after `telemetry.service.idle` minutes without any AI
request of an attached session (resumed silently by the next one,
without accounting the paused gap), and are closed and booked after 7
days without any activity. The result is logged together with the
action on the usage window and the booked usage. This is what the
`ase_task_assign` MCP tool of the `ase-task-assign` skill does for the
current session.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-s`|- `--session`- *id*]:
Assign the task on behalf of the agent tool session- *id*through the
running- `ase service`of the project (whose port is read from- `.ase/service.yaml`), which additionally handles the AI usage window
of the task (see- **DESCRIPTION**). The service is not started on
demand, so it has to run already.
 

## ARGUMENTS

- [- *id*[- `:`]]:
The id of the task (default:- `$ASE_TASK_ID`, else- `default`). A
trailing colon is stripped. If only a single argument is given, it is
taken as- *assignee*, unless it has a trailing colon (then it is the- *id*, and the missing- *assignee*is an error).
 
- *assignee*:
The new assignee of the task:- `me`for the current user (the value
of- `$USER`, else- `$LOGNAME`, else- `unknown`),- `none`to remove the- `Assignee`key, or any other non-empty single-line name without
commas.
 

## ENVIRONMENT

- `ASE_TASK_ID`:
The default task id (default:- `default`).
 
- `USER`,- `LOGNAME`:
The current user, as substituted for- `me`.
 

## EXIT STATUS

`0` on success, `1` if the *assignee* is missing or invalid, no such
task exists, or (with `--session`) the service is not reachable.

## EXAMPLES

```
# take the task "json-export"
$ ase task assign json-export me
# give the task back
$ ase task assign json-export none
# hand the task of $ASE_TASK_ID over to the implementer agent
$ ase task assign implementer
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-service(1)`, `ase-setup-telemetry(1)`,
`ase-config(1)`, `ase-agent(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-load(1) - Write a Task Plan to Standard Output

## NAME

`ase-task-load(1)` - Write a Task Plan to Standard Output

## SYNOPSIS

`ase task load`
[`-h`|`--help`]
*id*

## DESCRIPTION

**ase task load** loads the task plan with the given *id* from the task
store (see `ase-task(1)`) and writes it to standard output, in the
textual task format: the frontmatter block (with its keys in canonical
order and their values column-aligned), the plan body, and the
attachment blocks (with their `Data` as literal block scalars). A plan
in a legacy format is normalized into the current frontmatter shape on
the fly. If no such task exists, nothing is written.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- *id*:
The id of the task, matching- `[A-Za-z0-9#][A-Za-z0-9#_-]*`.

## EXIT STATUS

`0` on success, also if no such task exists, `1` on an invalid *id*
or an unreachable task store.

## EXAMPLES

```
# copy a task plan into a file
$ ase task load json-export >json-export.md
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-save(1)`, `ase-task-view(1)`,
`ase-task-edit(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-view(1) - Show a Task Plan in the Pager

## NAME

`ase-task-view(1)` - Show a Task Plan in the Pager

## SYNOPSIS

`ase task view`
[`-h`|`--help`]
*id*

## DESCRIPTION

**ase task view** loads the task plan with the given *id* from the task
store (see `ase-task(1)`) and shows it in the pager defined by `$PAGER`
(falling back to `more`), run through the shell (so a value with
arguments is supported) and fed with the plan via its standard input.
If standard output is not a terminal, the pager is bypassed and the
plan is written plainly to standard output.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- *id*:
The id of the task, matching- `[A-Za-z0-9#][A-Za-z0-9#_-]*`.

## ENVIRONMENT

- `PAGER`:
The pager command (default:- `more`).

## EXIT STATUS

`0` on success, `1` if no such task exists.

## EXAMPLES

```
$ PAGER="less -R" ase task view json-export
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-load(1)`, `ase-task-edit(1)`,
`more(1)`, `less(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-edit(1) - Edit a Task Plan in the Editor

## NAME

`ase-task-edit(1)` - Edit a Task Plan in the Editor

## SYNOPSIS

`ase task edit`
[`-h`|`--help`]
*id*

## DESCRIPTION

**ase task edit** opens the task plan with the given *id* of the task
store (see `ase-task(1)`) in the editor defined by `$EDITOR`, else
`$VISUAL` (falling back to `vi`), run through the shell (so a value
with arguments is supported). As the task store is not necessarily
local, the plan is round-tripped through a temporary file
`ase-task-`*XXXXXX*`/`*id*`.md` below the system temporary directory.
A not yet existing plan starts as its minimal frontmatter (just the
`Type` and `Id` keys).

After the editor exits, an unchanged text is not saved at all. A
changed text is saved like by `ase-task-save(1)`, i.e., checked
against the task lifecycle model, where the save is *conditional* on
the plan as loaded (resp. create-only for a not yet existing plan), so
a change of the task in the meantime (e.g., by an agent) is never
overwritten.

A changed `Id` frontmatter key renames the task after the save, like
`ase-task-rename(1)`, where an already existing target id is refused
before anything is saved. An id not conforming to the task id scheme
of the project is warned about.

If the save fails because the task was changed in the meantime, the
edits are not saved, but kept in the temporary file, whose path is
reported. On any other failure (e.g., an unreachable status), the
error is reported and, on a terminal, `re-edit? [Y/n]` is asked: by
default the editor is opened again on the edited text, while `n` (and
a non-interactive standard input) gives up, keeping the edits in the
temporary file, too.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- *id*:
The id of the task, matching- `[A-Za-z0-9#][A-Za-z0-9#_-]*`.

## ENVIRONMENT

- `EDITOR`,- `VISUAL`:
The editor command (default:- `vi`).

## EXIT STATUS

`0` on success (also for an unchanged text), `1` if the edits could
not be saved.

## EXAMPLES

```
$ EDITOR="code --wait" ase task edit json-export
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-save(1)`, `ase-task-rename(1)`,
`ase-task-view(1)`, `ase-task-board(1)`, `vi(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-save(1) - Save a Task Plan from Standard Input

## NAME

`ase-task-save(1)` - Save a Task Plan from Standard Input

## SYNOPSIS

`ase task save`
[`-h`|`--help`]
[`-c`|`--create`]
*id*

## DESCRIPTION

**ase task save** reads a task plan in the textual task format (see
`ase util meta format-task`) from standard input and saves it under
the given *id* in the task store (see `ase-task(1)`), creating or
overwriting the task.

The text is parsed into the frontmatter block, the plan body, and the
attachment blocks, where a plan in a legacy format is normalized into
the current frontmatter shape first. The `Id` frontmatter key is
always taken from the *id* argument, and a missing `Type` key is set
to `text/vnd.ase.task`. A text without a closed frontmatter block is
taken as the plan body.

Attachment data given as the elision marker `[elided: `*N*` lines]` is
restored from the stored plan. The usage attachment (`kind="usage"`)
can be changed by a booking only: an overwrite keeps the stored usage
attachment, while a create drops any given one.

The `Status` frontmatter key is checked against the task lifecycle
model of the project: a changed status which is not a state of the
model, or which is not reachable from the previously saved status (or,
for a new task, from the initial state of the model) via one or more
transitions of the model, lets the save fail. An *id* not conforming
to the task id scheme of the project is saved, but warned about.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-c`|- `--create`]:
Create the task only: fail if a task with- *id*already exists,
instead of overwriting it (default: off). Intended for the first
save of a task whose- *id*was obtained via- `ase-task-newid(1)`, as
only a- `seq`id is reserved there, so a concurrently created task of
the same id is never overwritten.
 

## ARGUMENTS

- *id*:
The id of the task, matching- `[A-Za-z0-9#][A-Za-z0-9#_-]*`.

## EXIT STATUS

`0` on success, `1` if the *id* is invalid, the plan is invalid (e.g.,
a multi-line frontmatter value, or a body which would be misread as
attachments), the `Status` is not a state of the model or not
reachable, or, with `--create`, the task already exists.

## EXAMPLES

```
# create a new task from a plan file
$ id=$(ase task newid "Export as JSON")
$ ase task save --create "$id" <plan.md
# round-trip a task plan through a filter
$ ase task load json-export | sed -e 's/^Kind:.*/Kind: CRAFTING/' | ase task save json-export
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-load(1)`, `ase-task-edit(1)`,
`ase-task-newid(1)`, `ase-task-status(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-delete(1) - Delete a Task Plan

## NAME

`ase-task-delete(1)` - Delete a Task Plan

## SYNOPSIS

`ase task delete`
[`-h`|`--help`]
*id*

## DESCRIPTION

**ase task delete** deletes the task plan with the given *id*,
including all its attachments, from the task store (see
`ase-task(1)`). For a `seq` task id scheme, the sequence number of the
deleted task is never reused by `ase-task-newid(1)`, as it raises the
high-water mark of the sequence numbers of the project. References to
the deleted task in the `After` keys of other task plans are left
alone.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- *id*:
The id of the task, matching- `[A-Za-z0-9#][A-Za-z0-9#_-]*`.

## EXIT STATUS

`0` if the task existed and was deleted, `1` if no such task existed
(reported as `task: no task "`*id*`" to remove`) or on any error.

## EXAMPLES

```
$ ase task delete json-export
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-purge(1)`, `ase-task-rename(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-rename(1) - Rename a Task Plan

## NAME

`ase-task-rename(1)` - Rename a Task Plan

## SYNOPSIS

`ase task rename`
[`-h`|`--help`]
*old-id*
*new-id*

## DESCRIPTION

**ase task rename** renames the task plan with the given *old-id* to
*new-id* in the task store (see `ase-task(1)`), rewriting the `Id`
frontmatter key inside the plan.

All references to *old-id* in the `After` frontmatter keys of all
other task plans of the project are rewritten to *new-id* (in the
frontmatter only, so their `Modified` key stays untouched), and the
number of updated plans is reported. For an *explicit epic*, i.e., a
task whose `Group` key equals its old (or already its new) id, its own
`Group` key and those of all its member task plans are rewritten, too,
while otherwise the group names are left untouched, as they form a
namespace of their own.

The task itself is renamed first, so an interruption never leaves both
ids behind; a failure to update the references afterwards is reported
together with the plans left behind.

For a `seq` task id scheme, the sequence number of *old-id* is never
reused by `ase-task-newid(1)`. A *new-id* not conforming to the task
id scheme of the project is warned about. Finally, the rename is
reported to the `ase service` of the project (if it ever ran, i.e.,
`.ase/service.yaml` exists), so its AI usage windows follow the task;
a failure of this report is only warned about.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- *old-id*:
The current id of the task, matching- `[A-Za-z0-9#][A-Za-z0-9#_-]*`.
 
- *new-id*:
The new id of the task, matching- `[A-Za-z0-9#][A-Za-z0-9#_-]*`,
which must not be in use already.
 

## EXIT STATUS

`0` on success, `1` if no such task existed (reported as `task: no task "`*old-id*`" to rename`), the *new-id* is already in use, or on
any other error.

## EXAMPLES

```
$ ase task rename json-export export-json
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-edit(1)`, `ase-task-newid(1)`,
`ase-task-delete(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-newid(1) - Print the Next Free Task Id

## NAME

`ase-task-newid(1)` - Print the Next Free Task Id

## SYNOPSIS

`ase task newid`
[`-h`|`--help`]
[`-p`|`--proposal` *id*]
[`-t`|`--taken` *ids*]
[`-v`|`--verbose`]
[*title*]

## DESCRIPTION

**ase task newid** prints the next free task id according to the
effective task id scheme of the project (see `ase-task-idscheme(1)`
and **TASK ID SCHEMES** in `ase-task(1)`), determined by searching all
existing task ids of the project in the task store, plus the ids given
via `--taken`, plus the reserved id `default`:

- `slug`[- `:`- *words*]:
The slug of the first- *words*words of- *title*(starting with the
first word beginning with a letter, with diacritics stripped,
lower-cased, and joined with- `-`).
 
- `seq`[- `:`- *template*]:
The highest sequence number of all ids matching- *template*plus one,
but at least the high-water mark of all ids ever removed (deleted,
purged, or renamed) or allocated plus one, so the number of such a
task is never reused. Ids not matching the current- *template*(e.g.,
after a template change) are ignored.
 - The allocated number is - *reserved*by raising the high-water mark
atomically, so a concurrent allocation never yields the same id. The- *title*is not used.
 
- `any`:
The proposed id of- `--proposal`, sanitized (whitespace runs replaced
by- `-`, all characters other than- `A-Z`,- `a-z`,- `0-9`,- `#`,- `_`, and- `-`removed, and leading- `_`and- `-`stripped), else the slug of the
first two words of- *title*.
 

A slug or proposed id which is already taken gets a numeric suffix
`-2`, `-3`, etc. As only a `seq` id is reserved, the first save of the
new task should be a create-only one (`ase task save --create`, see
`ase-task-save(1)`), which fails instead of overwriting a concurrently
created task of the same id.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-p`|- `--proposal`- *id*]:
The proposed task id, honored for the task id scheme- `any`only
(default: none).
 
- [- `-t`|- `--taken`- *ids*]:
A comma-separated list of task ids to consider as taken in addition
to the existing ones (default: none), e.g., the ids planned for
further tasks not yet saved. Whitespace around the ids is ignored.
 
- [- `-v`|- `--verbose`]:
Instead of the id only, print the three lines- `scheme:`(the
effective task id scheme),- `id:`(the new id), and- `match:`(the
regular expression matching all ids conforming to the scheme).
 

## ARGUMENTS

- [*title*]:
The title of the new task, from which the id is derived for the
schemes`slug`and`any`(default: empty). For`slug`(and for`any`without`--proposal`), a title without any word beginning with a
letter is an error.

## EXIT STATUS

`0` on success, `1` if no id can be derived (e.g., a missing title)
or the task store is unreachable.

## EXAMPLES

```
# derive an id from a title under the "slug" scheme
$ ase task newid "Export the Tasks as JSON"
export-the
# show the details of the allocation under the "seq:#%d" scheme
$ ase task newid -v
scheme: seq:#%d
id:     #43
match:  ^#([0-9]+)$
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-idscheme(1)`, `ase-task-save(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-purge(1) - Remove Old Task Plans

## NAME

`ase-task-purge(1)` - Remove Old Task Plans

## SYNOPSIS

`ase task purge`
[`-h`|`--help`]
[*age*]

## DESCRIPTION

**ase task purge** removes all persisted task plans of the current
project from the task store (see `ase-task(1)`) whose modification
time is older than *age*, regardless of their lifecycle status. Each
removed task is logged as `task: purged "`*id*`"`, or `task: no tasks to purge` is logged if none was removed. For a `seq` task id scheme,
the sequence numbers of the purged tasks are never reused by
`ase-task-newid(1)`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- [*age*]:
The maximum age of the task plans to keep, as`<number><unit>`, where*unit*is one of`h`(hour),`d`(day, 24 hours),`m`(month, 30
days), or`y`(year, 365 days) (default:`31d`).

## EXIT STATUS

`0` on success (also if no task was removed), `1` on a malformed *age*
or an unreachable task store.

## EXAMPLES

```
# remove all tasks not modified within the last half year
$ ase task purge 6m
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-delete(1)`, `ase-task-list(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-lifecycle(1) - Get or Set the Task Lifecycle Model

## NAME

`ase-task-lifecycle(1)` - Get or Set the Task Lifecycle Model

## SYNOPSIS

`ase task lifecycle`
[`-h`|`--help`]
[*name*]

## DESCRIPTION

**ase task lifecycle** gets or sets the task lifecycle model of the
current project in the task store (see `ase-task(1)`), i.e., the state
machine the `Status` frontmatter keys of its task plans follow (see
**TASK LIFECYCLE MODELS** in `ase-task(1)`).

Without *name*, the effective model is printed: for a local task store
(`ase:`, `github:`, `gitlab:`, or `gitea+`*scheme*`:`), which always
follows the configuration, the `project.task.lifecycle` configuration
(default: `solo`); for a remote task store (`ase://` or `ases://`),
the model the project is registered under in the task store, which is
shared by all clients of the project and hence registered under
`project.task.lifecycle` on first use only.

A configured `project.task.lifecycle` deviating from the model of a
remote task store is warned about once per deviation only (tracked in
the *per-user state directory* as `task-lifecycle.json`).

With *name*, the project in the task store is explicitly switched to
this model: for a local task store by setting `project.task.lifecycle`
on the `project` scope (see `ase-config(1)`), for a remote task store by
re-registering the project there.

On every switch of the model, the `Status` of the existing task plans is
mapped onto the new model: a state also existing in the new model is
kept, else the first candidate state existing in the new model wins (for
a finished state only a finished one), else the initial state (resp. the
first finished state) of the new model is used. For instance, from
`solo` to `team`, `OPEN` becomes `PLANNING` and `CLOSED` becomes
`IMPLEMENTED`, and from `enterprise` to `solo`, `PLANNED` and
`IMPLEMENTING` become `OPEN` and `INTEGRATED` becomes `CLOSED`.

Plans without a `Status` key are left alone, as they read as the initial
state of any model. The switch is logged as
`task: set lifecycle model of project from "`*from*`" to "`*name*`"`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- [*name*]:
The name of the task lifecycle model to switch to:`solo`(the
single-phase lifecycle for a local solo developer),`team`(the
two-phase lifecycle for a distributed small team), or`enterprise`(the four-phase lifecycle for a gated enterprise pipeline).

## FILES

- `.ase/config.yaml`:
The- `project`scope configuration receiving- `project.task.lifecycle`on a switch of a local task store.
 
- *per-user state directory*- `/task-lifecycle.json`:
The deviations of the configured lifecycle model (and task id
scheme) from the ones of a remote task store already warned about.
 

## EXIT STATUS

`0` on success, `1` on an unknown model *name* or an unreachable task
store.

## EXAMPLES

```
# print the effective task lifecycle model
$ ase task lifecycle
solo
# switch the project to the enterprise lifecycle model
$ ase task lifecycle enterprise
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-status(1)`, `ase-task-idscheme(1)`,
`ase-task-board(1)`, `ase-config(1)`, `ase-agent(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-idscheme(1) - Get or Set the Task Id Scheme

## NAME

`ase-task-idscheme(1)` - Get or Set the Task Id Scheme

## SYNOPSIS

`ase task idscheme`
[`-h`|`--help`]
[*scheme*]

## DESCRIPTION

**ase task idscheme** gets or sets the task id scheme of the current
project in the task store (see `ase-task(1)`), i.e., the scheme after
which `ase-task-newid(1)` generates new task ids (see **TASK ID
SCHEMES** in `ase-task(1)`).

Without *scheme*, the effective scheme is printed: for a local task
store (`ase:`, `github:`, `gitlab:`, or `gitea+`*scheme*`:`), which
always follows the configuration, the `project.task.idscheme`
configuration (default: `slug`); for a remote task store (`ase://` or
`ases://`), the scheme the project is registered under in the task
store, which is shared by all clients of the project and hence
registered under `project.task.idscheme` on first use only.

A configured `project.task.idscheme` deviating from the scheme of a
remote task store is warned about once per deviation only (tracked in
the *per-user state directory* as `task-lifecycle.json`).

With *scheme*, the project in the task store is explicitly switched to
this scheme: for a local task store by setting `project.task.idscheme`
on the `project` scope (see `ase-config(1)`), for a remote task store
by re-registering the project there. The existing task ids are not
changed, but ids not conforming to the new scheme are warned about on
their next save, rename, and switch. The switch is logged as `task: set task id scheme of project from "`*from*`" to "`*scheme*`"`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- [*scheme*]:
The task id scheme to switch to:- `slug`[- `:`- *words*]: the slug of the first- *words*(- `1`-- `16`,
default:- `2`) words of the task title, e.g.,- `json-export`.
- `seq`[- `:`- *template*]: a continuously increasing sequence number,
rendered through the sprintf-style- *template*(default:- `%d`) with
exactly one- `%d`or- `%0`- *width*- `d`(- *width*- `0`-- `16`) and otherwise
only the characters- `A-Z`,- `a-z`,- `#`,- `_`, and- `-`(but not
starting with- `_`or- `-`), e.g.,- `FOO-%03d`or- `#%d`. This scheme
is required by the GitHub, GitLab, and Gitea task stores, as their
task ids are the issue numbers (e.g.,- `seq:#%d`).
- `any`: an arbitrary id, as proposed by the caller.
 

## FILES

- `.ase/config.yaml`:
The- `project`scope configuration receiving- `project.task.idscheme`on a switch of a local task store.
 
- *per-user state directory*- `/task-lifecycle.json`:
The deviations of the configured task id scheme (and lifecycle
model) from the ones of a remote task store already warned about.
 

## EXIT STATUS

`0` on success, `1` on an invalid *scheme* or an unreachable task
store.

## EXAMPLES

```
# print the effective task id scheme
$ ase task idscheme
slug
# switch to sequence numbers like "ASE-007"
$ ase task idscheme "seq:ASE-%03d"
# switch to issue numbers like "#42" (quoted, as "#" starts a comment)
$ ase task idscheme "seq:#%d"
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-newid(1)`, `ase-task-lifecycle(1)`,
`ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-store(1) - Manage the per-user task store REST API server

## NAME

`ase-task-store(1)` - Manage the per-user task store REST API server

## SYNOPSIS

`ase task store`
[`-h`|`--help`]
*command*
[*options* [...]]

`ase task store start`
[`-a`|`--address` *host*]
[`-p`|`--port` *port*]
[`-t`|`--token` *token*]
[`-c`|`--cors` *origin*]
[`-m`|`--module` *name*]
[`-d`|`--basedir` *dir*]
[`-s`|`--solo`|`--no-solo`]
[`--with-agent-relay`|`--without-agent-relay`]
[`--tls-cert` *file* `--tls-key` *file*]
[`-f`|`--foreground`]

`ase task store status`

`ase task store log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]

`ase task store stop`

## DESCRIPTION

**ase task store** manages the *task store server*: a per-user (not
per-project), bearer token authenticated REST API server, which
persists the task plans of *all* projects registered with it and which
the `ase task` commands of a project use through a remote
`project.task.store.url` of the form
`ase://`*addr*`:`*port*[`/`*token*] resp.
`ases://`*addr*`:`*port*[`/`*token*][`?insecure`] (see
`ase-task(1)`).

It can run locally or on a central host, as long as all clients reach
it. The REST API itself, its task plan model, and its event
notifications are described in `docs/task-api.md` of the *ASE*
distribution.

The server is *storage agnostic*: it implements the authentication,
the validation, the task lifecycle model checks, and the event
notifications itself, and delegates the *persistence* of projects and
task plans to a single *storage plugin* loaded at its start (see
**STORAGE PLUGINS** below).

The server is configured through its per-user configuration file
*per-user configuration directory*`/store.yaml` (see **FILES**), into
which the effective configuration of every start is persisted again,
so later starts, `ase task store status`, and `ase task store stop`
find and probe the running server with the very same address, port,
token, and TLS setting. Optionally, the server additionally co-hosts
the *agent relay* (see `ase-agent-relay(1)`), so a central host needs a
single server process only.

Without a *command*, the usage information is shown and the command
exits with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information of`ase task store`resp. of the given*command*and exit.

## COMMANDS

The following commands exist:

### ase task store start

`ase task store start` [*options*]

Start the task store server in the background (detached, logging
into *per-user configuration directory*`/store.log`), or with
`--foreground` in the current process. Before the server is started,
the given port is validated, and the TLS certificate and key are
resolved (where both or none have to be given) and checked for
readability, so invalid options never leave no server running.

The start is *idempotent*: a server already running on the configured
address and port (as identified by probing its `/projects` endpoint) is
left alone and reported as `task store: already running on` *url*. If an
explicitly given option *deviates* from the configuration of the running
server (`--address`, `--port`, `--token`, `--cors`, `--tls-cert`,
`--tls-key`, `--module`, `--basedir`, `--solo`/`--no-solo`,
`--with-agent-relay`, or `--without-agent-relay`), the running server is
instead stopped and restarted with the new configuration.

A running server rejecting the configured token is an error. A
configured port held by a foreign service is re-allocated (with a
warning). Binding to a non-loopback *host* without TLS is warned about,
as the bearer token would then travel unencrypted over the network.

A detached server is awaited (for up to 10 seconds) until it responds
to the probe with the expected token, and is reported as `task store: started on` *url* `(storage plugin "`*name*`")`. If it exits during
startup, does not respond, or its port is taken by a foreign service
meanwhile, it is terminated and the failure is reported together with
the last 20 lines of its log. The server terminates on `SIGTERM`,
`SIGINT`, `SIGHUP`, or `SIGQUIT`, stopping its storage plugin and
removing its `pid` from `store.yaml`. The options are:

- [- `-a`|- `--address`- *host*]:
The IP address (or hostname) to bind to (default: the configured- `address`, else- `127.0.0.1`). Use- `0.0.0.0`resp.- `::`to bind to
all interfaces (the local probes then connect to- `127.0.0.1`resp.- `::1`).
 
- [- `-p`|- `--port`- *port*]:
The TCP port to bind to, an integer in the range- `1`..- `65535`(default: the configured- `port`, else a free port allocated
randomly in the range- `42000`..- `44000`).
 
- [- `-t`|- `--token`- *token*]:
The bearer token every request has to carry in its- `Authorization: Bearer`- *token*header (default:- `$ASE_TASK_STORE_TOKEN`, else the configured- `token`, else a
freshly generated random token of 48 hexadecimal digits). The
token grants- *full*access to- *all*projects of the server.
 
- [- `-c`|- `--cors`- *origin*]:
Allow cross-origin browser requests (Cross-Origin Resource
Sharing, CORS) from- *origin*, e.g.- `http://localhost:5173`, or
from any origin with- `*`. Repeatable for multiple origins
(default: the configured- `cors`list, else none, i.e., no CORS
headers are emitted and cross-origin browser requests fail).
 
- [- `-m`|- `--module`- *name*]:
The storage plugin to load:- `ase`for the built-in file-based one,- `github`,- `gitlab`, resp.- `gitea`for the built-in GitHub, GitLab,
resp. Gitea Issues one, any other plain- *name*(matching- `[A-Za-z0-9_-]+`) for the NPM package- `ase-task-store-`- *name*(resolvable from the- *ASE*installation), and a relative or absolute
path for a local module (default: the configured- `storage.plugin`,
else- `ase`).
 - The given plugin is persisted as - `storage.plugin`.
 
- [- `-d`|- `--basedir`- *dir*]:
The base directory of the built-in- `ase`storage plugin (default:
the configured- `storage.options.basedir`, resolved relative to
the per-user configuration directory, else- `tasks`below it). A
given- *dir*is persisted as- `storage.options.basedir`. Ignored for
all other storage plugins.
 
- [- `-s`|- `--solo`]:
Let the built-in- `ase`storage plugin store a- *single*project
flat in its base directory, accepting- *any*project id (default:
the configured- `storage.options.solo`, else one sub-directory per
project). Persisted as- `storage.options.solo`. Ignored for all
other storage plugins.
 
- [- `--no-solo`]:
Let the built-in- `ase`storage plugin store one sub-directory per
project, removing a configured- `storage.options.solo`.
 
- [- `--with-agent-relay`]:
Additionally co-host the agent relay (see- `ase-agent-relay(1)`),
i.e., its- `/agents`and- `/projects/`- *id*- `/agents`... endpoints,
under the same address, port, and token (default: the configured- `relay`, else not co-hosted). Persisted as- `relay`. Mutually
exclusive with- `--without-agent-relay`.
 
- [- `--without-agent-relay`]:
Do not co-host the agent relay, removing a configured- `relay`.
Mutually exclusive with- `--with-agent-relay`.
 
- [- `--tls-cert`- *file*]:
Serve HTTPS (and- `wss://`for the WebSocket event endpoint)
instead of plain HTTP, with the PEM certificate (chain) in- *file*. Requires- `--tls-key`. Default: the- `project.task.store.crt`configuration (resolved relative to the configuration file
defining it), unless- `project.task.store.url`selects plain HTTP
via- `ase://`on this host, else the configured- `tls.cert`, else
plain HTTP. The effective path is persisted as- `tls.cert`.
 
- [- `--tls-key`- *file*]:
The PEM private key in- *file*for HTTPS. Requires- `--tls-cert`.
Default: the- `project.task.store.key`configuration (under the
same condition as for- `--tls-cert`), else the configured- `tls.key`. The effective path is persisted as- `tls.key`.
 
- [- `-f`|- `--foreground`]:
Run the server in the current process instead, logging to the
terminal, until it is interrupted (e.g. via- `CTRL+C`) or stopped
via- `ase task store stop`. An already running server with an
unchanged configuration is then an error.
 
- [- `-h`|- `--help`]:
Show usage information and exit.
 

### ase task store status

`ase task store status`

Report whether the task store server is running, by probing the
configured address and port with the configured token. Prints `task store: running on` *url* `(pid` *pid*`)`, followed by `with agent relay` if the server co-hosts the agent relay, and exits with status
`0`. Otherwise it prints why it is not running -- no port configured,
port not responding, port in use by a foreign service, or a running
server rejecting the configured token -- and exits with status `1`.

### ase task store log

`ase task store log` [`-f`|`--follow`] [`-n`|`--lines` *n*]

Show the log *per-user configuration directory*`/store.log` of the
task store server run in the background. A server run with
`--foreground` logs to the terminal instead, and without a log file
just an informational message is printed. The log file is trimmed
down to its last 2000 lines whenever a server is started on a log
file larger than 1 MB. The options are:

- [`-f`|`--follow`]:
Show the last*n*lines (default:`10`) and then, in the style of`tail -f`, follow all newly appended lines until interrupted,
even once the server has stopped.
- [`-n`|`--lines`*n*]:
Show the last*n*lines only, where*n*is a positive integer
(default: all lines, or`10`with`--follow`).
- [`-h`|`--help`]:
Show usage information and exit.

### ase task store stop

`ase task store stop`

Stop the task store server: it is first probed (as a stale process id
might meanwhile belong to a foreign process), then sent `SIGTERM`,
and awaited for up to 5 seconds until it is gone, and finally its
`pid` is removed from `store.yaml`. A server which is not running (or
not configured at all) is reported informationally with exit status
`0`. A responding server whose process id is unknown, resp. a server
not stopping within the timeout, is an error.

## STORAGE PLUGINS

A storage plugin is a Node.js module default-exporting a factory
function, which receives the `storage.options` of `store.yaml`
verbatim (see `docs/task-api.md` for the plugin API). The following
built-in storage plugins exist:

- `ase`:
The default, file-based plugin, persisting the task plans in the
textual task format as- `TASK-`- *id*- `.md`files. Its options are- `basedir`(the base directory, see- `--basedir`),- `solo`(see- `--solo`), and- `lifecycle`(the task lifecycle model of newly
registered projects, default:- `solo`).
 - Without - `solo`, every project is stored below- *basedir*- `/`- *project-id*- `/`, with its- `PROJECT.yaml`as the project
registry entry (carrying the task lifecycle model, the task id
scheme, and the sequence number high-water mark). With- `solo`, a
single project is stored flat in- *basedir*itself, with a single- `PROJECT.yaml`there, and any project id is accepted -- which is
exactly what the local- `ase:`- *path*form of- `project.task.store.url`uses in-process, without any server.
 
- `github`:
Persists the task plans as the- *issues*of GitHub repositories,
through the GitHub REST API.
 - Its options are - `url`(the web URL of the GitHub instance, default:- `https://github.com`, else e.g. the one of a GitHub Enterprise
Server or Cloud),- `token`(default:- `$GITHUB_TOKEN`, else- `$GH_TOKEN`; required),- `repos`(required, a mapping of each project
id onto its- *owner*- `/`- *repo*), and- `poll`(the polling interval of
the change detection in seconds, default:- `60`,- `0`disables it).
 
- `gitlab`:
Persists the task plans as the- *issues*of GitLab projects, through
the GitLab REST API v4. Its options are- `url`(the GitLab instance,
default:- `$GITLAB_HOST`, else- `https://gitlab.com`),- `token`(default:- `$GITLAB_TOKEN`; required, a token with scope- `api`and at least the
role- *Planner*in the projects),- `repos`(required, a mapping of each
project id onto its- *namespace*- `/`- *project*path, with nested groups
allowed), and- `poll`(as for- `github`).
 
- `gitea`:
Persists the task plans as the- *issues*of Gitea repositories,
through the Gitea REST API v1. Its options are- `url`(the Gitea
instance, default:- `https://gitea.com`),- `token`(default:- `$GITEA_TOKEN`; required, a token with the scopes- `write:issue`and- `read:repository`and write access to the repositories),- `repos`(required, a mapping of each project id onto its- *owner*- `/`- *repo*),
and- `poll`(as for- `github`).
 

The issue tracker plugins register a project by the label
`ase:project` of its repository and map the task plan frontmatter onto
the issue state, assignees, labels, and dependencies, as detailed in
`docs/task-api.md`. The same plugins also run in-process (without any
server) for the `github:`, `gitlab:`, and `gitea+http[s]:` forms of
`project.task.store.url` (see `ase-task(1)`).

## ENVIRONMENT

- `ASE_TASK_STORE_TOKEN`:
The default bearer token of- `ase task store start`(taking precedence
over the configured- `token`, but not over- `--token`).
 
- `GITHUB_TOKEN`,- `GH_TOKEN`:
The default access token of the- `github`storage plugin.
 
- `GITLAB_HOST`,- `GITLAB_TOKEN`:
The default instance and access token of the- `gitlab`storage plugin.
 
- `GITEA_TOKEN`:
The default access token of the- `gitea`storage plugin.
 

## FILES

- *per-user configuration directory*- `/store.yaml`:
The configuration and runtime state of the task store server (see- `ase(1)`for the location of the per-user configuration directory),
written with mode- `0600`, as it carries the token.
 - Its keys are - `address`(the bound address),- `port`(the bound port),- `token`(the expected bearer token),- `pid`(the process id of the
running server),- `cors`(the list of allowed CORS origins),- `tls.cert`and- `tls.key`(the paths of the PEM certificate and key,
resolved relative to the file),- `relay`(- `true`to co-host the agent
relay),- `storage.plugin`(the storage plugin), and- `storage.options`(the options of the storage plugin, see- **STORAGE PLUGINS**), as in
the example below.
 
- *per-user configuration directory*- `/store.log`:
The log of the task store server run in the background.
 
- *per-user configuration directory*- `/tasks/`:
The default base directory of the built-in- `ase`storage plugin.
 

An example `store.yaml` is:

```
address: 127.0.0.1
port:    42042
token:   8f3c[...]
cors:
    - http://localhost:5173
storage:
    plugin: ase
    options:
        basedir: /Users/rse/Library/Application Support/ase/tasks
```

## EXIT STATUS

`ase task store status` exits with status `0` if the server is running
and with status `1` otherwise. All other commands exit with status `0`
on success and with status `1` on any error.

## EXAMPLES

```
# start the task store server locally and point the project to it
$ ase task store start --port 42042
$ ase config set project.task.store.url ase://127.0.0.1:42042
# start a central task store server with TLS and co-hosted agent relay
$ ase task store start --address 0.0.0.0 --port 42042 \
    --tls-cert /etc/ssl/tasks.example.com.crt \
    --tls-key /etc/ssl/tasks.example.com.key \
    --with-agent-relay
# start a task store server on the GitHub Issues storage plugin
# (after configuring "storage.options.repos" in "store.yaml")
$ ase task store start --module github
# check and follow the server
$ ase task store status
$ ase task store log -f
# stop the server
$ ase task store stop
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-board(1)`, `ase-agent-relay(1)`,
`ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-task-board(1) - Show the interactive task board of the project

## NAME

`ase-task-board(1)` - Show the interactive task board of the project

## SYNOPSIS

`ase task board`
[`-h`|`--help`]
[`-w`|`--web`]
[`-t`|`--text`]
[`--agent-relay` *url*]
[`-T`|`--task` *id*]

## DESCRIPTION

**ase task board** shows the *task board* of the current project: all
persisted task plans of the task store of the project (see
`ase-task(1)`) as *cards* in the *lanes* of the effective task
lifecycle model (see `ase task lifecycle`), following all changes of
the task plans live. It comes in three variants:

- the interactive *terminal board*(the default, on a terminal),
- the *web board*in the browser (with`--web`), and
- the plain *text overview*(with`--text`, or without an interactive
terminal on standard input or output).

The lanes are grouped by the phases of the lifecycle model plus a
final `Done` phase of the finished states, where within a phase the
entry state comes first, the active state second, and the parking
state last. The cards of the *active* lanes (the states a task is
actively worked on) are shown in the accent color (default: blue), the
cards of the *parking* lanes in the normal color, and the cards of the
finished lanes in the dim color (default: grey), while the selected
card is shown in the signal color (default: red).

A card shows the task id (inverse, behind a `≡`, and of an epic, i.e.,
a task being its own group, additionally behind a `♛`), its group (if
any, between `♛` and `▶`), its markers, and its title, the latter cut
onto a single line or, toggled with `t`, wrapped onto at most three
lines. The markers are:

- `[✎]`: an edit request of the task is pending (see- `ase-agent-editor(1)`).
- `[✎…]`: an edit request of the task is being processed.
- `[✎✗]`: an edit request of the task failed.
- `[⚖`- *size*- `]`: the effort of the task, i.e., the T-shirt size
(- `XS`,- `S`,- `M`,- `L`, or- `XL`) of its valid- `Effort:`key (see- `ase-agent-estimator(1)`), absent for an unestimated task.
- `⚉`- *name*: a badge on the top border of the card, naming an
agent busy with the task (inverse), resp. the agent the task is
assigned to.
- `☺`- *name*: a badge on the top border of the card, naming the
person the task is assigned to.

Besides the *task lanes* view, the boards show the *task graph* view,
the dependency graph of the tasks derived from their `After:` keys
(where an epic implicitly comes after all other tasks of its group),
the *statistics* view with the statistics of the tasks, and with an
agent relay the *agent logs* view and the *agent graph* view. All
views are cycled via `v` (or a click onto the `view:` value of the
header).

The last shown view, the minimized lanes, the collapsed phases, and the
shown flags are persisted in `.ase/board.yaml` (separately for the
terminal and the web board).

### Terminal Board

The terminal board runs in the alternate screen of the terminal and
requires a window of at least 40 columns and 16 rows.

Its header shows the project, the task lifecycle model, the number of
unfinished and of all tasks, the current view, and the `filter:` field
(with its `✕` clear button), below it the warnings of the board (e.g. on
dependency cycles), and its footer shows the information line, up to
three key hint lines (hidden and shown via `?`, where the information
line reminds of `?` while they are hidden), and the status line
with the kind and connection state of the task store (`●` connected,
`○` disconnected), the number of agents and the connection state of
the agent relay (if any), and the *ASE* version, or a notice instead.

In the task lanes view, lanes can be minimized (`m`), lane rows collapsed
(`C`), and phases collapsed (`c`), and the selected lane can be
*grown* via `g` to the full board, showing all its cards in a grid of
at least four columns.

A card opens its plan in a full-height *task dialog* (`⏎` or a mouse
double click), which shows the frontmatter keys, the predecessors and successors
of the task (a click onto one opens its task dialog), and the plan body,
followed by the attachments of the plan as further tabs, where a diff
attachment (type `text/x-diff`, like the implementation draft or the change
set snapshot `kind="changeset"` of the `implementer` and `integrator`
agents) is colored per line, YAML and JSON data are syntax-highlighted,
and a change set snapshot older than the plan is marked `(stale)`.

While the task dialog is open, the last change of the shown plan is displayed
as a diff in the plan tab, with the removed and added lines colored (in the
terminal board also signed) and their changed words shown inverse, until `#` clears it or a next change
replaces it. Closing the task dialog or switching to another task drops it.

A task is moved into another state (changing its `Status:`) either by
picking it up with `SPACE` and dropping it with a second `SPACE` onto
the selected lane (only lanes whose state is reachable from the current
state in the lifecycle model accept the task, and are shown in bold),
or via the *transfer popup* of `T`, which lists all lane states (as
*phase* `▷` *state*, starting at the current state), where the states
not reachable from the current state are dimmed.

A task is assigned (changing its `Assignee:`) via the *assign popup* of
`A`: its assignee field, prefilled with the current assignee, accepts
any name, while its drop-down suggests the names containing the typed
one: `none` (unassigning the task, like an empty field), then the
users of `project.users` (else the current user) with `☺`, then the
agents present at the agent relay with `⚉`, each sorted, except the
agents which never take a task by its assignee (the request agents,
like `ase-agent-editor(1)`, and the phase agents of an `unclaimed`
definition). With a
non-empty `project.users`, a name neither suggested nor the current
user is marked `unknown`, and its assignment is warned about (see
`ase-task-assign(1)`).

`e` edits the selected task plan with `$EDITOR` (default: `vi`, run
through the shell, with the file passed as `$ASE_TASK_FILE`), where a
draft which failed to save is offered again on the next `e`. `N`
creates a new task by editing a pre-filled task plan (all frontmatter
keys, a free placeholder id, the initial state, and the section
template of the task format), which is stored under the id of its `Id:`
line (refused for an existing id, while an unchanged text creates no
task at all), where a text which failed to save is offered again on the
next `N` (created again even if unchanged) until it is emptied, which
discards it. `D` deletes the selected task after a confirmation.

`E` requests the edit of the selected task by a prompt (refused for a
finished task and while an edit of it is in progress), which
`ase-agent-editor(1)` then processes via the `ase-task-edit` skill. The
prompt popup is prefilled with the prompt of a cancelled request resp.
of the most recent failed request (for a retry), and an empty prompt
is refused.

`R` requests the selected task by a prompt under a *request name*
(see `ase task request`), suggested from the request agents announced
to the agent relay (`edit` for `ase agent editor`, else the request
name of any other request agent), which the request agent of the name
then processes. An empty prompt is refused, unless all request agents
of the name announced to the agent relay accept the prompts of their
requests as optional (e.g., the `research` requests of `ase agent researcher`, see `ase-agent-researcher(1)`) or ignore them. If all of
them ignore the prompts (e.g., the `estimate` requests of `ase agent estimator --mode request`, see `ase-agent-estimator(1)`), the prompt
field is disabled, skipped by `TAB`, and no prompt is sent.

`/` focuses the `filter:` field of the header: its keywords are
matched case-insensitively against the task id and title, AND-combined
when separated by spaces and OR-combined when separated by commas. Each
OR-part first matches its keywords as literal substrings, and only if
this matches no task at all, it matches them *fuzzily* (keywords of at
least four characters also match words of a similar spelling). A
keyword `effort:`*size* (with *size* one of `XS`, `S`, `M`, `L`, or
`XL`, case-insensitively) instead exactly matches the tasks of this
effort size (its `Effort:` key), while a keyword of any other size is
an ordinary one. The lanes show only the matching tasks, the graph
additionally their direct predecessors and successors (dimmed and
dashed).

The terminal board supports the mouse (via SGR mouse reporting): a
click onto a card or a lane selects it, a double click onto a card
opens it, a double click onto a lane minimizes or maximizes it (resp.
shrinks a grown lane, resp. expands its collapsed phase), a click
onto the title of a phase selects it like `←`/`→` (its lane in the row
of the selected lane), a double click onto it collapses or expands it,
a click onto
the `view:` value of the header cycles the view, a click onto the
`filter:` field focuses it, a click onto its `✕` clears it, and the
mouse wheel over a lane scrolls it. In the task graph view, a click onto a
task selects it and a double click opens it.

`M` disables the mouse support to regain the regular text selection of
the terminal (and enables it again). The terminal board enables the
*kitty keyboard protocol* if the terminal supports it, as only then
`Shift`+`⏎` is distinguishable from `⏎`.

### Web Board

With `--web`, the web board is served through the *ASE* service of the
project (see `ase-service(1)`, started if not yet running) under
`http://127.0.0.1:`*port*`/task-board`, which is printed on standard
output and opened in the default browser (via `open`, `xdg-open`, resp.
`start`).

It offers the same views and keys as the terminal board, where a task
is additionally moved by dragging its card onto a reachable lane, the
graph is zoomed by the mouse wheel (and `+`, `-`, and `0`) and panned
by dragging it, and an image attachment is zoomed by the mouse wheel.
Its minimized lanes and collapsed phases are stored once per project,
not per browser, so all open web boards share them and pick up a change
immediately.

In the task dialog of the web board, `e` (or `✎`) edits the task plan
in a Markdown editor (in the task lanes and task graph views, `e` opens the
selected task directly for editing): `Ctrl`/`⌘`+`S` saves it, but only
if the plan was not changed meanwhile (else it offers to overwrite or
discard), and `ESC` cancels (after a confirmation if changed). With the
Vim or Emacs key bindings of `board.web.editor.keymap`, `:w`/`:q` resp.
`C-x C-s`/`C-x C-c` save resp. cancel instead. A text which failed to
save is kept as a draft in the browser and offered on the next edit.

A running service with other *ASE* code is never restarted, as this
would break the MCP connections of the agent sessions using it: an
older service without the web board fails `--web` (restart it with
`ase service stop` once no agent session uses it), while a differing
build is only warned about.

### Text Overview

With `--text`, and also without an interactive terminal, the lane
overview is printed as plain text and the command exits: a header line
with the project, the lifecycle model, and the number of tasks, then
per phase (`━` *phase*) and lane (*state*, marked `(active)` for an
active lane, and its number of cards) the cards with their id (plus
the `✎` and `⚖` markers), title, and the agents busy with them (behind
`⚙`), followed, with an agent relay, by the agents (`●` busy, `○`
waiting) with their status and their last processing log line, and
finally all warnings of the board (behind `⚠`, e.g. on dependency
cycles).

### Task Viewer

With `--task`, the terminal and the web board act as a *task viewer*:
instead of the board, they show only the task dialog of the given task,
spanning the entire window and following all changes of the task live.
All actions of the task dialog stay available, and a click onto a
predecessor or successor shows that task instead. The shown view of the
board stays untouched. An unknown task is refused with an error.

The terminal board quits on `q` or `Ctrl`+`C` only, and with a warning
once the shown task no longer exists (e.g. as it was deleted or renamed
elsewhere).

The web board has no closing keys, as a page cannot close its browser
tab. Once the shown task no longer exists, it shows only a notice, and
shows the task again as soon as it exists again.

With `--text`, and also without an interactive terminal, the plan of the
task is printed as plain text and the command exits.

### Statistics View

The third view *statistics* shows the statistics of the tasks matching the
`filter:` field (all tasks if empty), refreshed on every change of the
tasks, within the time range selected by the buttons `total` (all
tasks, with the number of days since the creation of the oldest
task), `within last 30 days`, and `within last 7 days` on its top
(selected via `←`/`→` or a click, initially `total`):

- `Tasks per State`: a column chart with the number of tasks of each
state of the task lifecycle model, labeled by the state, below by the
effort of its estimated tasks (blank without any), and below by its
phase (of its lane), in the colors of the lanes, and with
the- `total tasks:`line below it. An effort is the sum of the
representative human minutes of the effort sizes of the tasks on the
scale- `project.task.effort.scale`(see- `ase-config(1)`), shown in
minutes below one hour, else in hours, where the explicit epics carry
no effort of their own.
 
- `Throughput`(besides- `Tasks per State`): a column chart with the
number of tasks finished (in a terminal state other than- `CANCELLED`)
in each of the last 12 weeks (a quarter, ending with the current
week), labeled by the ISO week number and, below, by the total and
average number of finished tasks per week. As the time of the
completion of a task is not tracked, its- `Modified:`frontmatter key
is taken instead, so the throughput is just an approximation. It
always covers the last 12 weeks, independent of the time range.
 - Below it, the - `forecast:`line shows the number of all unfinished
tasks (- *work-in-progress*and- *open*, independent of the time range)
and the number of weeks to finish them at the average throughput of
the last 12 weeks,- `n/a`without any finished task within them.
 
- `Task Progress`: one stacked bar of the- *cancelled*tasks, the- *finished*tasks (in a terminal state other than- `CANCELLED`), the
tasks- *work-in-progress*(- `wip`, in an active state, like- `PLANNING`or- `IMPLEMENTING`), and the- *open*tasks (in any other state, like- `DRAFTED`,- `PLANNED`, or- `STALLED`), each with its number and share.
 
- `Open Tasks`(besides- `Task Progress`): one stacked bar of the open
tasks, split into the- *ready*tasks (all predecessors in a terminal
state, or none at all) and the- *blocked*tasks (any predecessor in
another state, including the implicit predecessors of an epic),
each with its number and share. The readiness is determined against
all tasks, independent of the- `filter:`field, as a predecessor
hidden by the filter blocks a task nevertheless.
 
- `Usage`: bars of the total accumulated usage of the tasks (see- `ase-task-assign(1)`and- `ase-agent(1)`) by its- `time`(AI request,
human, and real time, in minutes below one hour, else in hours), its- `token`(all tokens, stacked by the cache read, cache write, input,
and output tokens), and its- `cost`(AI and human costs, in USD), with
the bars of a column scaled to the maximum of the column.
 
- `Effort`(besides- `Usage`): a column chart with the
number of estimated tasks of each effort size, from the smallest
(- `XS`) to the largest (- `XL`), where the explicit epics carry no
effort of their own, and with the- `estimated:`line below it, the
number of the estimated and of all tasks.
 
- `Top-5 Tasks`: a table of the (up to) five tasks of the highest usage, ranked
by their total (AI and human) costs and then by their input and
output tokens, with the columns- `Task Id`(inverse behind- `≡`, in the
color of its lane),- `Task Title`(cut with an ellipsis if too long),- `Total Share`(a bar of its share of the total costs of all tasks,
resp. of their input and output tokens without any costs),- `Cost`,
and- `Token`(as- *input*- `▷`- *output*, with the uncached input tokens,
i.e., without the cache read and write tokens).
 
- `Finished Tasks`(besides- `Top-5 Tasks`): the average (- `⌀`) AI costs,
human costs, and AI tokens (as- *input*- `▷`- *output*, like in- `Top-5 Tasks`) per finished task (the usage of the finished tasks
divided by their number, including the finished tasks without any
usage), and the- *lead time*(the time from the- `Created:`to the- `Modified:`frontmatter key) of the finished tasks as its median
(- `lead (median)`) and its 85th percentile (- `lead (p85)`), in minutes
below one hour, in hours below two days, else in days, each- `n/a`without any finished task.
 - Like the throughput, the lead time is just an approximation, and
median and percentile are robust against single outlier tasks. 
- `Efficiency`(besides- `Finished Tasks`): the- *AI/HR time ratio*(the
AI request time per human time, i.e., the higher, the more AI work
per attended human minute) and the- *AI/HR cost ratio*(the AI costs
per human costs), each- `n/a`without any human time resp. costs (as
booked by the agents, see- `ase-agent(1)`), and the- *AI cache hit ratio*(the share of the cache read tokens of all input tokens, i.e., of the
cache read, cache write, and uncached input tokens, the main lever on
the AI costs),- `n/a`without any input tokens.
 

Within the last 30 or 7 days, a task counts only if the period from
its `Created:` to its `Modified:` frontmatter key overlaps with the
time range, and the usage of a task counts only with the overlapping
part of the period from the `Created:` to the `Modified:` key of its
usage attachment (i.e., the times of its first and last booking). As
the usage attachment does not track when the usage accrued, the usage
is taken as being evenly spread over this period, so the usage within
these time ranges is just an approximation.

### Agent Logs View

With an agent relay (see `--agent-relay`), the boards additionally
follow the running agents (see `ase-agent(1)`) live in the view
*agent logs*: a master/detail view, with the framed master list `Agents` on
the left, showing each agent as a card with its name (inverse, behind
a `⚉`), a pulse on its top border while busy (resp. a circle while
waiting), and the fields `kind:` and `mode:` of its operation (or
`none` without a mode), and `task:` with the ids of the tasks it is busy
with (else `none (waiting)`).

The framed pane `Operation` on the right shows the operation of the
selected agent: its one-line title (in bold) and its brief description
(word-wrapped and cut after six lines), as given by its definition (see
`--title` and `--description` in `ase-agent-adhoc(1)`).

Two equally wide framed panes below it show the selected agent:
`Process` with its `host:`, its process id `pid:`, its user id and user
name `uid:` (e.g. `501 (rse)`), and its current working directory `cwd:` (cut at its
start if too long), where `uid:` and `cwd:` are `n/a` for an older agent
not reporting them, and `Accounting` with `uptime:`, the duration since its
start, `heartbeat:`, the age of its latest heartbeat, and `usage (total):` resp. `usage (last job):`, its usage of the agent tool in
total since its start
resp. by its last finished job, each as wall time, output tokens, and
costs in USD (e.g. `12m, 9.2k tok, $3.41`, with `n/a` for what the agent
tool does not report, and `none` before the first finished job).

The framed detail view `Recent Log Entries` below them shows
the recent processing log lines of the selected agent, each with its
full timestamp, the id of its task (if any), and its level and content
(warnings and errors in the signal color), broken onto multiple rows
instead of being cut. The detail view follows the bottom of the log
while not scrolled up. When the selected agent vanishes, its neighbor
is selected instead.

The information line shows the selected agent and the title of its
operation (if any). Without an agent relay,
the agent logs view is skipped.

An agent stopping vanishes from the board immediately, while an agent
missing its heartbeat vanishes after 30 to 35 seconds. A relay
connection without any traffic for 15 to 30 seconds (e.g. after a
network partition) is detected, reported as disconnected (dropping the
shown agents), and re-established.

### Agent Graph View

With an agent relay, the boards additionally show, after the agent logs
view, the last view *agent graph*: like the task graph view, but with the
running agents as their agent cards (with the same fields as in the
agent logs view), plus the lifecycle states referenced by these agents as
state boxes. An edge from a state to an agent card shows a state the
agent observes (its incoming states), a dashed edge a state the `After:`
predecessors of its tasks have to reach first (its gate), and an edge
from an agent card to a state a state the agent writes (its success,
failure, and verdict states). The cards of the agents operating in the
same activity state (e.g. `IMPLEMENTING`) are drawn within a dashed
container labeled with this state (behind a `◉`), while unclaimed phase
agents (e.g. `estimator --mode phase`) and request agents (e.g.
`editor`) are drawn outside of any container, the latter without any
edges. The agent graph is laid out top to bottom: the request agents
in a row on top, below them the lifecycle states in the grid of the
task lanes view, i.e., per phase as a column, the entry states, then
the active states (the containers, plus the unclaimed phase agents),
and then the parking states. Several running agents of the same kind
and mode are merged into one card, with their number behind a `×` in
its name and the union of the tasks they are busy with.

The information line shows the selected agent card with its incoming,
activity, written, and gate states (resp. the name of its requests),
resp. the selected state with the agents observing, writing, and gating
on it. The states are announced by the agents themselves, so agents and
agent relay have to be of the same *ASE* version. Without an agent
relay, the agent graph view is skipped.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-w`|- `--web`]:
Serve the web board through the- *ASE*service of the project, print
its URL, open it in the default browser, and exit (see- **Web Board**).
Takes precedence over- `--text`.
 
- [- `-t`|- `--text`]:
Print the lane overview as plain text and exit (see- **Text
Overview**), instead of starting the interactive terminal board.
 
- [- `--agent-relay`- *url*]:
The agent relay to receive the live presence of the agents from, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*] resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for HTTPS
(optionally without certificate verification), with the bearer token
embedded, else from- `$ASE_AGENT_RELAY_TOKEN`, else from- `project.relay.token`, else of the locally started agent relay
(default: the configured- `project.relay.url`, else none, i.e.,
without agent logs and agent graph views).
 - Honored by the terminal board and the text overview only, as the web
board always follows the configured - `project.relay.url`(and hence
warns about the option).
 
- [- `-T`|- `--task`- *id*]:
Show only the task dialog of the task- *id*instead of the board, as a
viewer and watcher of the task (see- **Task Viewer**).
 

## KEYS

The following keys are supported by the terminal board (and, unless
noted otherwise, equally by the web board). The keys `D`, `E`, `N`,
`R`, `T`, `A`, and `M` are also accepted as `Shift` with the lower-case
letter under the kitty keyboard protocol.

### Every View

- `v`: Cycle the view: task lanes, task graph, statistics, and (with an
agent relay) agent logs and agent graph.
- `V`: Cycle the view backward.
- `/`: Focus the- `filter:`field (not in the agent logs and agent graph
views).
- `?`: Hide or show the key hint lines (persisted); while hidden, the
information line ends with- `?: show key hints`.
- `L`: Cycle the color theme through- `auto`,- `light`, and- `dark`,
overriding- `board.web.theme`in the current browser until cycled back
to its value (web board only).
- `D`: Delete the selected task, after a confirmation (not in the
agent logs and statistics views).
- `T`: Transfer the selected task to another state via the transfer
popup (not in the agent logs and statistics views).
- `A`: Assign the selected task via the assign popup (not in the agent
logs and statistics views).
- `E`: Request the edit of the selected task by a prompt (not in the
agent logs and statistics views).
- `R`: Request the selected task by a prompt under a request name (not
in the agent logs and statistics views).
- `M`: Disable or enable the mouse support (terminal board only).
- `q`,- `Ctrl`+- `C`: Quit (terminal board only).

### Task Lanes View

- `↑`/- `↓`: Select the previous/next card of the phase (within a
collapsed phase, the previous/next lane).
- `←`/- `→`: Select the card of the same lane in the previous/next phase.
- `PgUp`/- `PgDn`: Select the upper/lower lane of the phase.
- `Home`/- `End`: Select the first/last card of the lane.
- `Tab`/- `Shift`+- `Tab`: Step through the lanes of the expanded phases
lane row by lane row, wrapping around the board.
- `⏎`: Open the task dialog of the selected task.
- `e`: Edit the selected task plan (terminal: with- `$EDITOR`; web: in
the Markdown editor of its task dialog).
- `N`: Create a new task.
- `SPACE`: Pick up the selected task, resp. drop the picked-up task
onto the selected lane (dropping it onto its own lane cancels).
- `ESC`: Cancel the move of a picked-up task.
- `m`: Minimize or maximize the selected lane (persisted).
- `c`: Collapse or expand the selected phase (persisted).
- `C`: Collapse the lane row of the selected lane, i.e., the lanes at
the same position in all expanded phases, resp. expand it again if
all its lanes are collapsed already (persisted).
- `g`: Grow the selected lane to the full board (only a maximized lane
of an expanded phase), resp. shrink it back again.
- `t`: Cut the card titles onto a single line, resp. wrap them onto at
most three lines (persisted).

### Grown Lane

- `↑`/- `↓`/- `←`/- `→`: Select the card above/below/left/right in the grid.
- `PgUp`/- `PgDn`: Select the card a page of rows above/below.
- `Home`/- `End`: Select the first/last card.
- `g`,- `ESC`: Shrink the lane back again.
- `⏎`,- `e`,- `N`,- `SPACE`,- `t`: As in the task lanes view.

### Task Graph View

- `↑`/- `↓`/- `←`/- `→`: Select the nearest task in the direction of the arrow.
- `⏎`: Open the task dialog of the selected task.
- `e`,- `N`,- `t`: As in the task lanes view.
- `s`: Hide or show the standalone tasks, i.e., the tasks without any
predecessors and successors (persisted).
- `+`/- `-`/- `0`: Zoom in, out, resp. reset the zoom (web board only).

### Statistics View

- `←`/- `→`: Select the previous/next time range (- `total`,- `within last 30 days`, or- `within last 7 days`).

### Agent Logs View

- `↑`/- `↓`: Select the previous/next agent.
- `Shift`+- `↑`/- `↓`: Scroll the log of the detail view by one line
(terminal board only).
- `PgUp`/- `PgDn`: Scroll the log of the detail view by one page.
- `Home`/- `End`: Scroll to the top/bottom of the log of the detail view,
where scrolling down to its bottom follows it again.

### Agent Graph View

- `↑`/- `↓`/- `←`/- `→`, mouse click: Select the nearest agent card or state
in the direction of the arrow, resp. the clicked one, where the edges
of the selection are highlighted, and the information line of a
selected state shows the agents observing, writing, and gating on it.
- `⏎`, mouse double click: Show the agent logs view with the agent of the
selected card (of a merged card: its oldest agent) selected.
- `+`,- `-`,- `0`, mouse wheel: Zoom in, out, resp. reset the zoom (web
board only), where dragging the background scrolls the agent graph.
- `PgUp`/- `PgDn`,- `Home`/- `End`, mouse wheel: Scroll the agent graph by
a page, resp. to its top/bottom (terminal board only).

### Task Dialog

- `←`/- `→`,- `Tab`/- `Shift`+- `Tab`: Select the previous/next tab.
- `↑`/- `↓`: Scroll the content.
- `PgUp`/- `PgDn`: Scroll the content by one page.
- `e`: Edit the task plan.
- `#`: Clear the diff of the last change of the task plan.
- `D`,- `T`,- `A`,- `E`,- `R`,- `M`,- `L`: As in every view.
- `⏎`,- `ESC`: Close the task dialog.

### Filter Field

- *character*: Extend the filter query (applied live).
- `Backspace`: Remove the last character of the filter query.
- `⏎`: Keep the filter query and leave the field.
- `ESC`: Clear the filter query and leave the field.

### Prompt Popup

- *character*: Insert the character (also pasted text) at the cursor.
- `⏎`: Request the edit resp. the request (on the request name field:
switch to the prompt; on a button: press it, also a click onto it).
- `Shift`+- `⏎`,- `Alt`+- `⏎`,- `Ctrl`+- `J`: Insert a line break (terminal:- `Shift`+- `⏎`only under the kitty keyboard protocol,- `Alt`+- `⏎`only
under the kitty keyboard protocol or where the terminal sends it as- `ESC`+- `⏎`, e.g. with "Option as Meta" on macOS, and- `Ctrl`+- `J`in
every terminal; web:- `Shift`+- `⏎`only).
- `←`/- `→`: Move the cursor by one character.
- `Home`/- `End`: Move the cursor to the start/end of the prompt.
- `Backspace`/- `Delete`: Remove the character before/at the cursor.
- `Tab`/- `Shift`+- `Tab`: Switch to the next/previous field: the request
name (- `R`only), the prompt, and the request and cancel buttons
(terminal board: also a click onto a field; web: by the regular
focus switching).
- `↑`/- `↓`: Cycle through the suggested request names of the drop-down,
which offers the names containing the typed request name, while on
the request name (- `R`only; also a click onto a name picks it).
- `ESC`: Cancel, keeping the prompt as a draft for the task.

### Transfer Popup

- `↑`/- `↓`: Select the previous/next reachable state.
- `⏎`: Transfer the task into the selected state (also a click onto it).
- `ESC`: Cancel (also a click outside of the popup).

### Assign Popup

- *character*: Extend the assignee (web board: edit it freely).
- `Backspace`: Remove the last character of the assignee (terminal board).
- `↑`/- `↓`: Cycle through the suggested assignees of the drop-down.
- `⏎`: Assign the task to the assignee (also a click onto a suggested
assignee).
- `ESC`: Cancel (also a click outside of the popup).

### Deletion Confirmation

- `y`: Delete the task.
- `ESC`: Cancel (web board: also- `n`).
- `←`/- `→`,- `Tab`: Switch between the- `delete`and- `cancel`buttons
(terminal board; web board: by the regular focus switching).
- `⏎`: Press the selected button (default:- `cancel`).

## CONFIGURATION

The following configuration keys (on scope `user` or `project`, see
`ase-config(1)`) customize the boards:

- `board.tui.color.dim`,- `board.tui.color.normal`,- `board.tui.color.accent`,- `board.tui.color.signal`:
The colors of the four color roles of the terminal board (default:- `grey`,- `default`,- `blue`, and- `red`), each either- `default`(the
foreground color of the terminal), a color name,- `#rgb`,- `#rrggbb`,- `ansi256(`- *n*- `)`, or- `rgb(`- *r*- `,`- *g*- `,`- *b*- `)`.
 
- `board.web.color.dim`,- `board.web.color.normal`,- `board.web.color.accent`,- `board.web.color.signal`:
The base colors of the four color roles of the web board (default:- `grey`,- `black`,- `blue`, and- `orange`), each either- `default`(the
default base color of the role), one of the color names- `black`,- `grey`,- `gray`,- `brown`,- `red`,- `orange`,- `yellow`,- `green`,- `teal`,- `cyan`,- `blue`,- `purple`, or- `magenta`,- `#rgb`, or- `#rrggbb`.
 
- `board.web.editor.keymap`:
The key bindings of the task plan editor of the web board:- `default`,- `vim`, or- `emacs`(default:- `default`).
 
- `board.web.theme`:
The color theme of the web board:- `light`,- `dark`, or- `auto`, which
follows the light or dark preference of the operating system resp.
browser (default:- `auto`).
 
- `project.relay.url`:
The default agent relay of the boards (see- `--agent-relay`).
 

## ENVIRONMENT

- `EDITOR`:
The text editor of the terminal board for- `e`and- `N`(default:- `vi`), run through the shell, so a value with arguments is supported.
 
- `ASE_AGENT_RELAY_TOKEN`:
The bearer token of an agent relay URL without an embedded token.
 

## FILES

- `.ase/board.yaml`:
The display state of the boards in the project root: separately for
the terminal board (- `tui`) and the web board (- `web`), the shown- `view`(- `lanes`,- `graph`,- `stats`,- `agents`, or- `flow`), the- `minimized`lanes, the- `collapsed`phases, and whether the task- `titles`are wrapped onto
multiple lines, the key hint lines are shown (- `keys`), and the graph
shows the- `standalone`tasks.
 - The web state is shared by all browsers and tabs showing the web board.
It never holds task content, and is covered by the self-ignoring
- `.ase/.gitignore`.
 

## EXIT STATUS

The command exits with status `0` once the terminal board is quit, the
text overview is printed, resp. the web board is opened, and with
status `1` on any error (e.g. an *ASE* service without web board on
`--web`, or an unknown task on `--task`).

## EXAMPLES

```
# show the interactive terminal board
$ ase task board
# show the web board in the browser
$ ase task board --web
# view and watch a single task in the terminal
$ ase task board --task T1
# print the lane overview, including the agents of a remote agent relay
$ ase task board --text --agent-relay ase://relay.example.com:42043
```

## SEE ALSO

`ase(1)`, `ase-task(1)`, `ase-task-store(1)`, `ase-agent(1)`,
`ase-agent-editor(1)`, `ase-agent-relay(1)`, `ase-service(1)`,
`ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-artifact(1) - Resolve Project Artifact Kinds to File Lists

## NAME

`ase-artifact(1)` - Resolve Project Artifact Kinds to File Lists

## SYNOPSIS

`ase artifact`
[`-h`|`--help`]

`ase artifact list`
[`--kind` *kinds*]

`ase artifact name`
[`--kind` *kind*]
*filename*

## DESCRIPTION

**ase artifact** is the entry point group for resolving the *artifact
kinds* of a project to project-relative file lists. The *ASE* skills
use it (and the MCP tools `ase_artifact_list` and `ase_artifact_name`,
which provide the same) to locate the files of a kind. The recognized
kinds are:

- `spec`: the specification (by default below- `docs/specbook`,
the files- `**/*.{md,txt,svg,png,jpg}`).
- `code`: the source code (by default below- `src`, all files
except- `**/etc/**`and- `**/{.gitignore,.npmignore,package.json}`).
- `docs`: the documentation (by default below- `doc`, all files plus- `**/{README,LICENSE,CHANGELOG}.{md,txt}`, except- `{spec,arch}/**`).
- `infr`: the infrastructure (by default below the project root, the
files- `**/{.github,.claude*,etc}/**`and- `**/{AGENTS.md,{package,tsconfig*}.json,.{git,npm}ignore}`).
- `othr`: the implicit catch-all of all other files.

The first four kinds are configured by the configuration keys
`project.artifact.`*kind*`.basedir` (the base directory, relative to
the project root, where an empty value denotes the project root itself,
and which must not escape the project root via `..`) and
`project.artifact.`*kind*`.files` (a *miniglob* specification, resolved
relative to the base directory), with the defaults given above (see
`ase-config(1)`).

A *miniglob* specification is a whitespace-separated list of glob
patterns (in the syntax of *picomatch*, with dot files matched), each
optionally negated by a `!` prefix. The tokens are applied in order:
the resulting set starts empty (or with all files, if the first token
is negated), a plain pattern adds all files matching it, and a negated
pattern removes all files matching it.

The file universe is the set of all files below the project root (the
Git top-level directory, else the current working directory), except
the `.git` directories and all files excluded by the Git exclude rules:
the global excludes file (`core.excludesFile`, else
`$XDG_CONFIG_HOME/git/ignore`, falling back to
`~/.config/git/ignore`), the repository-local `info/exclude`, and the
`.gitignore` files from the project root downwards, with the last
matching rule winning.

The universe is partitioned among the kinds by descending precedence
`spec`, `code`, `docs`, and `infr`: a file matched by multiple kinds
belongs to the first one only, and every file matched by none of them
belongs to `othr`.

Without a sub-command, the usage information is shown and the command
exits with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## COMMANDS

- `ase artifact list`[- `--kind`- *kinds*]:
Resolve one or more artifact kinds to project-relative file paths.
See- `ase-artifact-list(1)`.
 
- `ase artifact name`[- `--kind`- *kind*]- *filename*:
Resolve a base-relative filename within an artifact kind to a
project-relative path. See- `ase-artifact-name(1)`.
 

## EXIT STATUS

The group command without a sub-command exits with status `1`.

## SEE ALSO

`ase(1)`, `ase-artifact-list(1)`, `ase-artifact-name(1)`,
`ase-config(1)`, `ase-spec(1)`, `gitignore(5)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-artifact-list(1) - Resolve Artifact Kinds to Project-Relative File Paths

## NAME

`ase-artifact-list(1)` - Resolve Artifact Kinds to Project-Relative File Paths

## SYNOPSIS

`ase artifact list`
[`-h`|`--help`]
[`--kind` *kinds*]

## DESCRIPTION

**ase artifact list** resolves one or more artifact kinds of the
project to the project-relative (POSIX-style) paths of their files,
driven by the configuration keys `project.artifact.`*kind*`.basedir`
and `project.artifact.`*kind*`.files` and honoring the Git exclude
rules, as described in `ase-artifact(1)`.

The files of each kind are printed sorted, one per line, as a bullet
list (`- `*file*). When more than one kind is requested, the list of
each kind is preceded by a header line `# `*kind*`:`. The kinds are
printed in the requested order, while the partitioning of the files
among the kinds always follows the fixed precedence `spec`, `code`,
`docs`, `infr`, with `othr` as the implicit catch-all.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--kind`- *kinds*]:
A comma-separated list of the artifact kinds to resolve, out of- `spec`,- `code`,- `docs`,- `infr`, and- `othr`(default:- `spec,code,docs,infr,othr`, i.e., all kinds). Whitespace around the
kinds is ignored. An unknown kind is an error.
 

## EXIT STATUS

Exits with status `0` on success, and with status `1` on an unknown
kind or an invalid configuration (e.g., a base directory escaping the
project root).

## EXAMPLES

```
# list the source code files of the project
$ ase artifact list --kind code
# list the specification and documentation files, each with a header
$ ase artifact list --kind spec,docs
```

## SEE ALSO

`ase(1)`, `ase-artifact(1)`, `ase-artifact-name(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-artifact-name(1) - Resolve a Filename within an Artifact Kind

## NAME

`ase-artifact-name(1)` - Resolve a Filename within an Artifact Kind

## SYNOPSIS

`ase artifact name`
[`-h`|`--help`]
[`--kind` *kind*]
*filename*

## DESCRIPTION

**ase artifact name** resolves a *filename*, given relative to the base
directory of an artifact *kind*, to a project-relative (POSIX-style)
path by prefixing it with the configured base directory
`project.artifact.`*kind*`.basedir` (see `ase-artifact(1)`), and prints
it. The file does not have to exist. The skills use it to name a new
file of a kind, e.g., a new specification or source code file.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `--kind`- *kind*]:
The artifact kind whose base directory is used:- `spec`,- `code`,- `docs`, or- `infr`(default:- `code`). The implicit catch-all kind- `othr`has no base directory and is rejected.
 

## ARGUMENTS

- *filename*:
The filename, relative to the base directory of the kind. Backslashes
are converted to slashes and leading slashes are stripped. It must
not be empty and must not contain any- `..`segment.

## EXIT STATUS

Exits with status `0` on success, and with status `1` on an unknown
or unsupported kind, an empty filename, or a filename with a `..`
segment.

## EXAMPLES

```
# resolve a source file name (with the default "project.artifact.code.basedir" of "src")
$ ase artifact name --kind code lib/util.ts
src/lib/util.ts
```

## SEE ALSO

`ase(1)`, `ase-artifact(1)`, `ase-artifact-list(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-spec(1) - Initialize, Lint, Export, Query, and Preview the Project Specification

## NAME

`ase-spec(1)` - Initialize, Lint, Export, Query, and Preview the Project Specification

## SYNOPSIS

`ase spec`
[`-h`|`--help`]

`ase spec init`
[`-s`|`--sample`]

`ase spec lint`
[`-m`|`--markdown`]
[`-v`|`--verbose`]

`ase spec export`
[`-o`|`--output` [*format*`:`]*file*] [...]
[`-w`|`--watch`]
[`-S`|`--schema`]

`ase spec query`
[`-P`|`--param` *name*`=`*value*] [...]
[`-f`|`--format` *format*]
[`-o`|`--output` [*format*`:`]*file*]
[`-S`|`--schema`]
*selector*

`ase spec preview`
[`-a`|`--addr` *ip-addr*]
[`-p`|`--port` *tcp-port*]
[`-S`|`--schema`]

## DESCRIPTION

**ase spec** is the entry point group for initializing, linting,
exporting, querying, and previewing the *SpecBook*-based project
specification (SPEC), i.e., the Markdown files (plus their embedded
assets) of the artifact kind `spec` below the base directory
`project.artifact.spec.basedir` (default: `docs/specbook`, relative to
the project root). Every `.md` file below the base directory
(recursively, except below `.`-prefixed entries), whatever its name,
is taken as a specification file.

The specification is governed by the *SpecBook* YAML schema
configuration `project.artifact.spec.schema`: a whitespace-separated
list of schema files, merged in the given order, where the entry `std`
names the full standard schema bundled with *SpecBook* (not its
compressed variant `ase-format-specbook.yaml` bundled with the plugin
for the agents, see `ase-util-meta(1)`), the entries `meta` and `pm`
name the meta and project management schemas bundled with *SpecBook*,
the entries prefixed with `npm:`, `github:`, `http://`, or `https://`
name remote schemas (fetched and cached by *SpecBook*), and every other
entry is a file path or glob pattern relative to the project root. An
unset or empty value means `std`.

All reading specification commands honor the Git exclude rules exactly
like `ase artifact list` does (see `ase-artifact(1)`): an artifact file
Git excludes from the project counts as an *absent* one, so both
resolvers always see the very same specification.

The *SpecBook* processing information is logged at the `info` level
and its tracing details at the `debug` level (see the global option
`--log-level` of `ase(1)`).

Without a sub-command, the usage information is shown and the command
exits with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## COMMANDS

- `ase spec init`[- `-s`|- `--sample`]:
Initialize the absent specification Markdown files of the schema
configuration. See- `ase-spec-init(1)`.
 
- `ase spec lint`[- `-m`|- `--markdown`] [- `-v`|- `--verbose`]:
Lint the specification against the schema configuration, and
optionally for its generic Markdown formatting. See- `ase-spec-lint(1)`.
 
- `ase spec export`[- `-o`|- `--output`[- *format*- `:`]- *file*] [...] [- `-w`|- `--watch`] [- `-S`|- `--schema`]:
Export the specification as JSON, JSON5, YAML, TOON, HTML, PDF, or
normalized Markdown. See- `ase-spec-export(1)`.
 
- `ase spec query`[- `-P`|- `--param`- *name*- `=`- *value*] [...] [- `-f`|- `--format`- *format*] [- `-o`|- `--output`[- *format*- `:`]- *file*] [- `-S`|- `--schema`]- *selector*:
Query the specification with an- *ASTq*selector, writing the matched
nodes as JSON, JSON5, YAML, or TOON. See- `ase-spec-query(1)`.
 
- `ase spec preview`[- `-a`|- `--addr`- *ip-addr*] [- `-p`|- `--port`- *tcp-port*] [- `-S`|- `--schema`]:
Serve the HTML export of the specification as a live preview. See- `ase-spec-preview(1)`.
 

## EXIT STATUS

The group command without a sub-command exits with status `1`.

## SEE ALSO

`ase(1)`, `ase-spec-init(1)`, `ase-spec-lint(1)`, `ase-spec-export(1)`,
`ase-spec-query(1)`, `ase-spec-preview(1)`, `ase-artifact(1)`,
`ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-spec-init(1) - Initialize the Project Specification

## NAME

`ase-spec-init(1)` - Initialize the Project Specification

## SYNOPSIS

`ase spec init`
[`-h`|`--help`]
[`-s`|`--sample`]

## DESCRIPTION

**ase spec init** initializes the *SpecBook*-based specification below
`project.artifact.spec.basedir` (see `ase-spec(1)`): it creates every
specification Markdown file which the *SpecBook* schema configuration
`project.artifact.spec.schema` assigns artifacts onto and which does
not exist yet, creating the base directory on demand. Each created file
carries a frontmatter with the `Created` and `Modified` timestamps and
the level 1 headings of all artifacts assigned onto it, in the order of
the schema configuration. An already existing file is never touched.

The created files are printed to standard output as a single line
`initialized artifact file(s): `*file*[`, `*file*] [...], where each
*file* is relative to the project root, or as the line `no artifact files were created` if all files already exist.

Unlike `specbook init`, no *SpecBook* project configuration file
`.specbook.yaml` is created, as the base directory and the schema
configuration are taken from the *ASE* configuration (see
`ase-config(1)`).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-s`|- `--sample`]:
Additionally scaffold one lint-clean sample object per allowed object
kind into the created files, with placeholder values satisfying the
schema configuration, so the format is learnable from them.
 

## EXIT STATUS

Exits with status `0` on success, and with status `1` if the
initialization failed (e.g., an invalid schema configuration).

## EXAMPLES

```
# initialize the specification
$ ase spec init
# initialize and then lint the specification
$ ase spec init && ase spec lint
# initialize the specification with sample objects
$ ase spec init -s
```

## SEE ALSO

`ase(1)`, `ase-spec(1)`, `ase-spec-lint(1)`, `ase-spec-export(1)`,
`ase-spec-query(1)`, `ase-spec-preview(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-spec-lint(1) - Lint the Project Specification

## NAME

`ase-spec-lint(1)` - Lint the Project Specification

## SYNOPSIS

`ase spec lint`
[`-h`|`--help`]
[`-m`|`--markdown`]
[`-v`|`--verbose`]

## DESCRIPTION

**ase spec lint** lints the *SpecBook*-based specification Markdown
files below `project.artifact.spec.basedir` against the *SpecBook*
schema configuration `project.artifact.spec.schema` (see `ase-spec(1)`),
and optionally for their generic Markdown formatting via *markdownlint*.

Every diagnostic is logged to standard error as a single message
`lint: `*linter*`: `*file*`:`*line*`:`*column*`: `*message* of the
log level `error` resp. `warning` of its severity, where *linter* is
`specbook` or `markdownlint`, and *file* is relative to the project
root (a file outside the project, like the bundled standard schema, is
kept as-is). Without any diagnostic, nothing is logged.

The MCP tool `ase_specbook_lint` provides the same to the skills.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-m`|- `--markdown`]:
Additionally lint the specification Markdown files for their generic
Markdown formatting via- *markdownlint*, with the rules conflicting
with the- *SpecBook*format (like mixed ATX and Setext headings, free
ordered list numbering, padded list markers and headings, unwrapped
lines, images without alternate text, and multiple artifacts per file)
disabled. Its findings are diagnostics of- `warning`severity, so they
never fail the linting.
 
- [- `-v`|- `--verbose`]:
Instead of logging each diagnostic, print it to standard output as a
multi-line message with the affected source snippet (with an- `ERROR`resp.- `WARNING`type, and a message prefixed by- `markdownlint: `for a- *markdownlint*diagnostic), colorized if standard output is a
terminal. If the source file is unreadable or empty, the single-line
message- *file*- `:`- *line*- `:`- *column*- `: `- *message*is printed instead.
 

## EXIT STATUS

Exits with status `0` if there is no diagnostic of `error` severity
(warnings alone do not fail), and with status `1` if there is any
diagnostic of `error` severity or the linting itself failed (e.g., an
unreadable schema configuration).

## EXAMPLES

```
# lint the specification
$ ase spec lint
# lint the specification, showing the affected source snippets
$ ase spec lint -v
# lint the specification, including its generic Markdown formatting
$ ase spec lint -m
```

## SEE ALSO

`ase(1)`, `ase-spec(1)`, `ase-spec-export(1)`, `ase-spec-query(1)`,
`ase-spec-preview(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-spec-export(1) - Export the Project Specification

## NAME

`ase-spec-export(1)` - Export the Project Specification

## SYNOPSIS

`ase spec export`
[`-h`|`--help`]
[`-o`|`--output` [*format*`:`]*file*] [...]
[`-w`|`--watch`]
[`-S`|`--schema`]

## DESCRIPTION

**ase spec export** exports the *SpecBook*-based specification Markdown
files below `project.artifact.spec.basedir` (see `ase-spec(1)`) into one
or more output files. The supported formats are:

- `json`,- `json5`,- `yaml`,- `toon`:
The specification object model in the respective serialization.
- `html`:
A self-contained single HTML document.
- `pdf`:
A paginated print document.
- `md`:
A normalized single Markdown document.

The specification is parsed once, and each distinct format is exported
once, even if requested for multiple outputs. Each written file is
logged at the `info` level. The export fails on any lint diagnostic
of `error` severity (see `ase-spec-lint(1)`), while warnings are only
logged, so a partial or invalid specification is never emitted.

The `pdf` format additionally requires a Chromium-class browser, which
neither `npm install -g @rse/ase` nor `ase setup install` provides:
unless a *Google Chrome* is installed system-wide, fetch the
*Playwright* Chromium once via `npx playwright install chromium`.
Without any such browser, the export fails before the specification is
even parsed and reports that very command.

The MCP tool `ase_specbook_export` provides the same to the skills.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-o`|- `--output`[- *format*- `:`]- *file*]:
Write the export into- *file*, or to standard output for- `-`. The
option is repeatable, so multiple outputs (of the same or different
formats) are written in a single run. The- *format*is inferred from
the filename extension (- `json`,- `json5`,- `yaml`or- `yml`,- `toon`,- `html`or- `htm`,- `pdf`, and- `md`or- `markdown`, case-insensitive)
unless explicitly given as a prefix of at least two characters (out
of the same names), where a plain- `-`defaults to- `json`.
 - The log is written to standard error (see - `ase(1)`), so it never
mixes into the export on standard output. An unknown format prefix and
a not inferable format are errors. A relative- *file*is relative to the
current working directory. Default: the HTML rendering into- `index.html`(resp.- `schema.html`under- `--schema`) inside the
specification base directory (e.g.,- `docs/specbook/index.html`).
A- `md`output inside the specification base directory is refused,
as it would be parsed as a specification file itself.
 
- [- `-w`|- `--watch`]:
Do not terminate after the initial export, but keep the outputs in
sync by re-exporting on every change of a configuration file, an
artifact file, or one of its embedded assets (bursts of changes are
coalesced). A transiently failing re-export leaves the watch intact.
As a re-export has to land somewhere it can be picked up again,- `--watch`rejects the- `-`standard output. The command then runs
until it is interrupted (e.g., via- `CTRL+C`).
 
- [- `-S`|- `--schema`]:
Export the effective schema configuration- `project.artifact.spec.schema`itself, rendered as a- *SpecBook*document, instead of the specification.
 

## EXIT STATUS

Exits with status `0` on success, and with status `1` on any lint
diagnostic of `error` severity, an invalid output specification, a
missing browser for `pdf`, an unwritable output file, or `--watch`
combined with `-`.

## EXAMPLES

```
# export the specification as HTML into its default location
$ ase spec export
# export the specification as PDF and as JSON in a single run
$ ase spec export -o spec.pdf -o spec.json
# export the normalized Markdown to standard output
$ ase spec export -o md:-
# keep an HTML export in sync with the sources
$ ase spec export -o /tmp/spec.html -w
# export the schema configuration as PDF
$ ase spec export -S -o schema.pdf
```

## SEE ALSO

`ase(1)`, `ase-spec(1)`, `ase-spec-lint(1)`, `ase-spec-query(1)`,
`ase-spec-preview(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-spec-query(1) - Query the Project Specification

## NAME

`ase-spec-query(1)` - Query the Project Specification

## SYNOPSIS

`ase spec query`
[`-h`|`--help`]
[`-P`|`--param` *name*`=`*value*] [...]
[`-f`|`--format` *format*]
[`-o`|`--output` [*format*`:`]*file*]
[`-S`|`--schema`]
*selector*

## DESCRIPTION

**ase spec query** loads the *SpecBook*-based specification Markdown
files below `project.artifact.spec.basedir` (see `ase-spec(1)`),
strictly validated like for `ase spec export` (see `ase-spec-export(1)`),
and queries its Abstract Syntax Tree (AST) with the XPath-like *ASTq*
*selector*, e.g., `// USE-CASE [ @STATUS == 'Done' ]`. The matched
nodes are written as a JSON, JSON5, YAML, or TOON array, their subtrees
included and the embedded file contents dropped.

The queried tree consists of:

- the root node of type `specification`,
- its children of type `artifact`, with the attributes`created`,`modified`, and`file`, and
- the objects below, typed by their kind (with whitespace dashed),
with the attributes `kind`,`id`,`anchor`,`paren`,`name`,`primary`,`description`,`rationale`,`elaboration`, and one per
property key.

The query function `refs()` yields the objects the references of the
current object resolve to, e.g., `// USE-CASE [ count(refs()) == 0 ]`.

The query fails on any lint diagnostic of `error` severity (see
`ase-spec-lint(1)`), while warnings are only logged, so a partial or
invalid specification is never queried.

The MCP tool `ase_specbook_query` provides the same to the skills.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-P`|- `--param`- *name*- `=`- *value*]:
Substitute the value- *value*for the parameter- `{`- *name*- `}`of the- *selector*. The option is repeatable, one per parameter.
 
- [- `-f`|- `--format`- *format*]:
The output format- `json`,- `json5`,- `yaml`, or- `toon`(default:
inferred from the- *file*of- `--output`like for- `ase spec export`,
else- `json`).
 
- [- `-o`|- `--output`[- *format*- `:`]- *file*]:
Write the result into- *file*, or to standard output for- `-`(default:- `-`). A relative- *file*is relative to the current working
directory.
 
- [- `-S`|- `--schema`]:
Query the effective schema configuration- `project.artifact.spec.schema`itself, described as a- *SpecBook*document of the bundled meta schema
configuration, instead of the specification.
 

## EXIT STATUS

Exits with status `0` on success, and with status `1` on any lint
diagnostic of `error` severity, an invalid *selector* or parameter, an
unsupported output format, or an unwritable output file.

## EXAMPLES

```
# list all use cases as JSON
$ ase spec query "// USE-CASE"
# list the finished use cases as YAML
$ ase spec query -f yaml "// USE-CASE [ @STATUS == 'Done' ]"
# list the objects of a given kind into a TOON file
$ ase spec query -P kind=ROLE -o roles.toon "// * [ @kind == {kind} ]"
# list the artifacts of the schema configuration itself
$ ase spec query -S "/ artifact"
```

## SEE ALSO

`ase(1)`, `ase-spec(1)`, `ase-spec-lint(1)`, `ase-spec-export(1)`,
`ase-spec-preview(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-spec-preview(1) - Serve a Live Preview of the Project Specification

## NAME

`ase-spec-preview(1)` - Serve a Live Preview of the Project Specification

## SYNOPSIS

`ase spec preview`
[`-h`|`--help`]
[`-a`|`--addr` *ip-addr*]
[`-p`|`--port` *tcp-port*]
[`-S`|`--schema`]

## DESCRIPTION

**ase spec preview** serves the HTML export of the *SpecBook*-based
specification Markdown files below `project.artifact.spec.basedir` (see
`ase-spec(1)`) as a live preview on `http://`*ip-addr*`:`*tcp-port*`/`
(default: `http://127.0.0.1:12345/`). The export is kept in sync with
its sources exactly like under `ase spec export --watch` (see
`ase-spec-export(1)`), and every fresh export is pushed into the
connected browsers as an in-place document update. Before the first
successful export, a placeholder page is served instead.

The command does not terminate on its own, but runs until it is
interrupted (e.g., via `CTRL+C`).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-a`|- `--addr`- *ip-addr*]:
The IP address to listen on (default:- `127.0.0.1`).
 
- [- `-p`|- `--port`- *tcp-port*]:
The TCP port to listen on, an integer in the range- `1`..- `65535`(default:- `12345`).
 
- [- `-S`|- `--schema`]:
Preview the effective schema configuration- `project.artifact.spec.schema`itself, rendered as a- *SpecBook*document, instead of the specification.
 

## EXIT STATUS

Runs until interrupted. Exits with status `1` on an invalid TCP port
or if the server cannot be started.

## EXAMPLES

```
# serve the live preview on the default address and port
$ ase spec preview
# serve the live preview on all interfaces on port 8080
$ ase spec preview -a 0.0.0.0 -p 8080
# serve the live preview of the schema configuration
$ ase spec preview -S
```

## SEE ALSO

`ase(1)`, `ase-spec(1)`, `ase-spec-export(1)`, `ase-spec-lint(1)`,
`ase-spec-query(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-util(1) - Utility Commands

## NAME

`ase-util(1)` - Utility Commands

## SYNOPSIS

`ase util`
[`-h`|`--help`]

`ase util meta`
*name* [...]

`ase util compat`

`ase util diagram`
[*options*]

`ase util worktree`
`base`|`path`
[*options*]
[*id*]

`ase util mint`
[`-t`|`--type` *type*]
[`-c`|`--count` *count*]
[*hint* [...]]

`ase util metric`
[`-f`|`--file` *file*]
[*text* [...]]

`ase util rasterize`
[*options*]
*input-file*|`-`|`-d`|`--data` *data*

## DESCRIPTION

**ase util** is the entry point group for miscellaneous utility
commands. Most of them back the MCP tools of the per-project *ASE*
service and the *ASE* skills, but can be used standalone, too. Without
a sub-command, the usage information is shown and the command exits
with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## COMMANDS

- `ase util meta`- *name*[...]:
Output the contents of one or more plugin meta files. See- `ase-util-meta(1)`.
 
- `ase util compat`:
Output the expected probe values of the- `ase-meta-compat`self-test
skill. See- `ase-util-compat(1)`.
 
- `ase util diagram`[- *options*]:
Render a- *Mermaid*or- *D2*diagram specification as Unicode/ASCII art
or SVG. See- `ase-util-diagram(1)`.
 
- `ase util worktree`- `base`|- `path`[- *options*] [- *id*]:
Safely resolve the- *ASE*worktree directories below- `<repo-root>/.ase/worktree`. See- `ase-util-worktree(1)`.
 
- `ase util mint`[- *options*] [- *hint*[...]]:
Mint hash-derived identifiers (UUID, SHA-1) out of a hint. See- `ase-util-mint(1)`.
 
- `ase util metric`[- *options*] [- *text*[...]]:
Measure the words, lines, characters, and bytes of a file or a text.
See- `ase-util-metric(1)`.
 
- `ase util rasterize`[- *options*]- *input-file*|- `-`|- `-d`|- `--data`- *data*:
Rasterize an SVG document or PNG/GIF/JPEG image (raw, as- `data:`URL, or
as base64 data) into a PNG image or into the
terminal graphics of the Kitty, iTerm2, or SIXEL protocol. See- `ase-util-rasterize(1)`.
 

## EXIT STATUS

The group command without a sub-command exits with status `1`.

## SEE ALSO

`ase(1)`, `ase-util-meta(1)`, `ase-util-compat(1)`,
`ase-util-diagram(1)`, `ase-util-worktree(1)`, `ase-util-mint(1)`,
`ase-util-metric(1)`, `ase-util-rasterize(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-util-meta(1) - Output Plugin Meta Files

## NAME

`ase-util-meta(1)` - Output Plugin Meta Files

## SYNOPSIS

`ase util meta`
[`-h`|`--help`]
*name* [...]

## DESCRIPTION

**ase util meta** writes the contents of one or more *meta files* of
the *ASE* plugin to standard output, concatenated in the given order
and without any separator.

The meta files are the shared instructions and format conventions of
the *ASE* skills, e.g., the constitution (`ase-constitution.md`), the
tenets (`ase-tenets.md`), the persona (`ase-persona.md`), the dialog
conventions (`ase-dialog.md`), the option parsing conventions
(`ase-getopt.md`), and the artifact format conventions
(`ase-format-*.md`, plus the standard *SpecBook* schema configuration
`ase-format-specbook.yaml`).

The meta files are read from the `plugin/meta/` directory bundled with
the tool package itself, independent of any environment variables. All
requested files are read before any output is written, so a failing
name produces no output at all.

This command is intended to be leveraged by the *ASE* skills, and not
typically invoked directly by end users.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## ARGUMENTS

- *name*[...]:
One or more meta file names. Each- *name*is reduced to its base name
(so no directory traversal or absolute path is possible), the
extension- `.md`is appended if it has no extension, and the prefix- `ase-`is prepended if missing, so- `tenets`,- `ase-tenets`, and- `tenets.md`all select- `ase-tenets.md`, while- `format-specbook.yaml`selects- `ase-format-specbook.yaml`. An empty name,- `.`, and- `..`are
invalid.

## EXIT STATUS

Exits with status `0` on success, and with status `1` if a name is
invalid or a meta file cannot be read.

## EXAMPLES

```
# output the ASE constitution
$ ase util meta constitution
# output the tenets and the persona definition
$ ase util meta tenets persona
```

## SEE ALSO

`ase(1)`, `ase-util(1)`, `ase-spec(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-util-compat(1) - Output Expected Probe Values of the Compatibility Self-Test

## NAME

`ase-util-compat(1)` - Output Expected Probe Values of the Compatibility Self-Test

## SYNOPSIS

`ase util compat`
[`-h`|`--help`]

## DESCRIPTION

**ase util compat** writes the canonical expected values of all probes
of the `ase-meta-compat` self-test skill to standard output, one probe
per line as *id*`: `*value*, where *id* is
*category*`/`*probe-name*. The probe categories are `xml-placeholders`,
`control-flow`, `regex`, and `arithmetic`, which check whether the
model of the agent tool correctly interprets the pseudo-code
constructs the *ASE* skills are written in.

This command is intended to be invoked by the skill only after it has
recorded all actual probe results (so the model cannot see the
expected values beforehand), and not directly by end users. The MCP
tool `ase_compat` provides the same.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## EXIT STATUS

Exits with status `0`.

## SEE ALSO

`ase(1)`, `ase-util(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-util-diagram(1) - Render Mermaid or D2 Diagrams as Unicode/ASCII Art or SVG

## NAME

`ase-util-diagram(1)` - Render Mermaid or D2 Diagrams as Unicode/ASCII Art or SVG

## SYNOPSIS

`ase util diagram`
[`-h`|`--help`]
[`-i`|`--input` *file*]
[`-l`|`--lang` `mermaid`|`d2`]
[`-f`|`--format` `ascii`|`svg`]
[`-t`|`--theme` `light`|`dark`]
[`-T`|`--tone` *tone*]
[`-a`|`--ascii`]
[`-c`|`--color-mode` `none`|`ansi16`|`ansi256`]
[`--node-margin-x` *n*]
[`--node-margin-y` *n*]
[`--node-padding` *n*]
[`--diagram-clip-x` *n*]
[`--diagram-clip-y` *n*]
[`--terminal-width` *n*]
[`--terminal-height` *n*]

## DESCRIPTION

**ase util diagram** renders a diagram specification, written in the
*Mermaid* or *D2* diagram language and read from standard input or a
file, as Unicode box-drawing (or plain ASCII) art for the terminal, or
as a self-contained SVG document. The *Mermaid* diagrams are rendered
via *beautiful-mermaid* (supporting, e.g., flowcharts, UML state,
sequence, and class diagrams, ER diagrams, and XY-charts), and the
*D2* diagrams via the WebAssembly build of *D2*.

For the `svg` format, the diagrams are drawn in the colors of a theme
style (`light` or `dark`) and a theme color tone (one or two colors,
where a missing second one is derived as the complement of the first
one), from which a grey, an accent, and a signal color spread are
derived. For *Mermaid*, the node boxes are drawn in a darker accent
color with lighter labels on the `light` style, resp. vice versa on the
`dark` style. A `classDef default` in the diagram overrides their
`fill`, `stroke`, and `color`, while the nodes with their own class or
`style` keep their explicit properties. For *D2*, the colors override
the D2 themes "Neutral Default" (`light`) resp. "Dark Mauve" (`dark`),
while a `theme-overrides` of the diagram itself still wins. On the
`light` style, the *D2* shapes without own colors (except containers and
text, code, image, class, and SQL table shapes) are drawn in a darker
accent resp. signal color with lighter labels, too. The *D2*
diagrams have a transparent background, unless the diagram sets its
own via a top-level `style.fill`, and a padding of 20 instead of 100
pixels, unless the diagram sets its own via the `pad` of `d2-config`.

Any custom color of a diagram (like of a `classDef`, a `style`, or a
`style.fill`) is replaced by its perceptually nearest color of the three
color spreads, so the SVG document uses the theme colors only.

For the `ascii` format, the rendering is optionally clipped to the
terminal dimensions (reduced by the extra clipping amounts): lines
wider than the width budget are truncated (preserving and properly
terminating any ANSI color sequences), and lines beyond the height
budget are dropped.

A clipped rendering is followed by a warning paragraph which states
the overflow and advises how to regenerate the diagram source to fit
(e.g., by preferring a portrait orientation, reducing siblings per
row, abbreviating node labels, or nesting). The output always ends
with a newline.

The MCP tool `ase_diagram` provides the same to the skills.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-i`|- `--input`- *file*]:
Read the diagram specification from- *file*instead of standard
input. An empty (or whitespace-only) specification is an error.
 
- [- `-l`|- `--lang`- `mermaid`|- `d2`]:
The diagram language:- `mermaid`for- *Mermaid*(default) or- `d2`for- *D2*.
 
- [- `-f`|- `--format`- `ascii`|- `svg`]:
The output format:- `ascii`for Unicode/ASCII art (default) or- `svg`for an SVG document. The clipping options apply to- `ascii`only.
 
- [- `-t`|- `--theme`- `light`|- `dark`]:
The theme style of the SVG document (- `svg`only).
Default: the configured- `agent.diagram.theme`, else- `light`.
 
- [- `-T`|- `--tone`- *tone*]:
The theme color tone of the SVG document as one or two colors- `#rgb`or- `#rrggbb`, separated by whitespace (- `svg`only). Default:
the configured- `agent.diagram.tone`, else- `#336699`.
 
- [- `-a`|- `--ascii`]:
Emit plain ASCII characters (- `+`,- `-`,- `|`) instead of the Unicode
box-drawing characters (for both languages).
 
- [- `-c`|- `--color-mode`- `none`|- `ansi16`|- `ansi256`]:
The color mode of the ANSI escape sequences in the rendered art
(- *Mermaid*and- `ascii`only):- `none`for no colors,- `ansi16`for 16
colors, or- `ansi256`for 256 colors, where colors render the diagram
in gray shades. Default:- `$ASE_TERM_COLORS`if set to one of these
values, else derived from the color depth of standard output if it is
a terminal (- `ansi256`for at least 8 bits,- `ansi16`for at least 4
bits), else- `none`.
 
- [- `--node-margin-x`- *n*]:
The horizontal margin between nodes in characters (- *Mermaid*only,
default:- `3`).
 
- [- `--node-margin-y`- *n*]:
The vertical margin between nodes in lines (- *Mermaid*only, default:- `3`).
 
- [- `--node-padding`- *n*]:
The horizontal and vertical inner padding of each node in characters
(- *Mermaid*only, default:- `1`).
 
- [- `--diagram-clip-x`- *n*]:
The extra horizontal clipping: the width budget is the terminal width
minus- *n*characters (default:- `0`).
 
- [- `--diagram-clip-y`- *n*]:
The extra vertical clipping: the height budget is the terminal height
minus- *n*lines (default:- `0`).
 
- [- `--terminal-width`- *n*]:
The terminal width in characters for the clipping, where- `0`disables
the horizontal clipping. Default:- `$ASE_TERM_WIDTH`if a positive
integer, else the width of standard output if it is a terminal, else- `0`.
 
- [- `--terminal-height`- *n*]:
The terminal height in lines for the clipping, where- `0`disables the
vertical clipping. Default:- `$ASE_TERM_HEIGHT`if a positive integer,
else the height of standard output if it is a terminal, else- `0`.
 

All numeric values *n* have to be non-negative integers.

## ENVIRONMENT

- `ASE_TERM_COLORS`:
The default of- `--color-mode`(- `none`,- `ansi16`, or- `ansi256`).
 
- `ASE_TERM_WIDTH`,- `ASE_TERM_HEIGHT`:
The defaults of- `--terminal-width`and- `--terminal-height`.
 

## EXIT STATUS

Exits with status `0` on success, and with status `1` on an invalid
option value, an unreadable input file, an empty specification, or a
rendering failure (e.g., a syntax error in the specification).

## EXAMPLES

```
# render a Mermaid flowchart as Unicode art
$ printf 'flowchart LR\n    A --> B --> C\n' | ase util diagram
# render a D2 diagram from a file as plain ASCII art without colors
$ ase util diagram -l d2 -i arch.d2 -a -c none
# render a Mermaid diagram as an SVG document
$ ase util diagram -i flow.mmd -f svg >flow.svg
# render a Mermaid diagram as an SVG document in a dark purple theme
$ ase util diagram -i flow.mmd -f svg -t dark -T "#663399" >flow.svg
```

## SEE ALSO

`ase(1)`, `ase-util(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-util-worktree(1) - Resolve the ASE Worktree Directories

## NAME

`ase-util-worktree(1)` - Resolve the ASE Worktree Directories

## SYNOPSIS

`ase util worktree`
[`-h`|`--help`]

`ase util worktree base`
[`-c`|`--create`]
[`-t`|`--temp`]

`ase util worktree path`
[`-c`|`--create`]
[`-t`|`--temp`]
*id*

## DESCRIPTION

**ase util worktree** safely resolves the directories of the Git
worktrees managed by *ASE* (e.g., the worktrees of the
`--worktree` option of the skills and of the agents), which all reside
below the base directory `<repo-root>/.ase/worktree`, where
*repo-root* is the fully resolved top-level directory of the Git
working tree of the current working directory. The *temporary*
worktrees reside below `<repo-root>/.ase/worktree/.temp`, a namespace
disjoint from all regular worktree ids (as ids never contain a `.`).

The resolution is *validated*: as `git worktree add` silently follows
a symbolic link, every already existing path component below the
repository root (`.ase`, `worktree`, `.temp`, and the *id* leaf) has
to be a real directory, neither a symbolic link, nor any other file,
nor an aliased path, so a repository carrying a committed `.ase`
symbolic link can never make a worktree be written outside of it. Not
yet existing components are fine, as they are created later on.

The resolved directory is printed to standard output. Without a
sub-command, the usage information is shown and the command exits
with status `1`.

## OPTIONS

- [`-h`|`--help`]:
Show usage information and exit.

## COMMANDS

- `ase util worktree base`[- `-c`|- `--create`] [- `-t`|- `--temp`]:
Print the validated base directory- `<repo-root>/.ase/worktree`holding
all- *ASE*worktrees. The options are:
 - - [`-c`|`--create`]:
Create the base directory (with all its missing parents) if it
does not exist yet, and validate it again afterwards.
- [`-t`|`--temp`]:
Print the base directory`<repo-root>/.ase/worktree/.temp`of the
temporary worktrees instead.
- [`-h`|`--help`]:
Show usage information and exit.
 
- `ase util worktree path`[- `-c`|- `--create`] [- `-t`|- `--temp`]- *id*:
Print the validated worktree directory- `<repo-root>/.ase/worktree/<id>`of a single worktree- *id*, which has to be non-empty and match- `[A-Za-z0-9_-]+`, so it is safe both as a single path component and
as the Git branch name derived from it. The worktree directory
itself is not created, as this is done by- `git worktree add`. The
options are:
 - - [`-c`|`--create`]:
Create the base directory (not the worktree directory itself) if
it does not exist yet.
- [`-t`|`--temp`]:
Resolve the*id*below the base directory`<repo-root>/.ase/worktree/.temp`of the temporary worktrees.
- [`-h`|`--help`]:
Show usage information and exit.
 

## FILES

- `<repo-root>/.ase/worktree/`:
The base directory of all- *ASE*worktrees, ignored by Git via the
self-ignoring- `.ase/.gitignore`.
 
- `<repo-root>/.ase/worktree/.temp/`:
The base directory of the temporary- *ASE*worktrees.
 

## EXIT STATUS

Exits with status `0` on success, and with status `1` outside of a Git
working tree, on an invalid *id*, or if a path component is a symbolic
link, not a directory, or an aliased path. The group command without a
sub-command exits with status `1`.

## EXAMPLES

```
# print the base directory of all ASE worktrees, creating it if needed
$ ase util worktree base -c
# create a worktree for the id "FOO-42" at its validated location
$ git worktree add "$(ase util worktree path -c FOO-42)" -b FOO-42
```

## SEE ALSO

`ase(1)`, `ase-util(1)`, `git-worktree(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-util-mint(1) - Mint Hash-Derived Identifiers

## NAME

`ase-util-mint(1)` - Mint Hash-Derived Identifiers

## SYNOPSIS

`ase util mint`
[`-h`|`--help`]
[`-t`|`--type` `uuid`|`sha1`|`nanoid`]
[`-c`|`--count` *count*]
[*hint* [...]]

## DESCRIPTION

**ase util mint** mints one or more hash-derived identifiers out of the
*hint* formed by the remaining arguments (joined by single spaces), and
prints them to standard output, one identifier per line:

- With type - `uuid`, a non-empty- *hint*yields a deterministic- *UUID
V5*over the- *hint*within the fixed- *ASE*namespace (itself the- *UUID V5*of the URL- `https://ase.tools`within the standard URL
namespace), so the same- *hint*always yields the same UUID, disjoint
from those of other applications. An empty- *hint*yields random- *UUID V4*values instead.
 
- With type - `sha1`, the- *hint*yields its 40-character hexadecimal- *SHA-1*digest. This requires a non-empty- *hint*.
 
- With type - `nanoid`, the- *hint*yields its 10-character,- *NanoId*-style- *base62*hash (its- *SHA-1*digest,- *base62*-encoded and truncated),
a short but still reproducible identifier, as used for the content
binding of the- `Effort`key of the task plans. This requires a
non-empty- *hint*, too.
 

This backs the `uuid`, `sha1`, and `nanoid` types of the `ase-meta-mint` skill
(and the MCP tool `ase_mint`), while the language-level identifier
types of that skill require the AI and hence exist in the skill only.

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-t`|- `--type`- `uuid`|- `sha1`|- `nanoid`]:
The identifier type (default:- `uuid`).
 
- [- `-c`|- `--count`- *count*]:
The number of identifiers to mint, a positive integer (default:- `1`). It is honored for an empty- *hint*only, where random- *UUID V4*values are minted. As hashing a non-empty- *hint*always yields
exactly one identifier, a larger- *count*is then reduced to- `1`, and a
warning is printed on standard error.
 

## ARGUMENTS

- [*hint*[...]]:
The hint the identifiers are derived from, formed by all remaining
arguments joined by single spaces (default: empty).

## EXIT STATUS

Exits with status `0` on success, and with status `1` on an invalid
*count* or type `sha1` or `nanoid` with an empty *hint*.

## EXAMPLES

```
# mint the deterministic UUID of a hint
$ ase util mint the user account entity
# mint three random UUIDs
$ ase util mint -c 3
# mint the SHA-1 digest of a hint
$ ase util mint -t sha1 the user account entity
# mint the short NanoId-style hash of a hint
$ ase util mint -t nanoid the user account entity
```

## SEE ALSO

`ase(1)`, `ase-util(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-util-metric(1) - Measure the Length Metrics of a Text

## NAME

`ase-util-metric(1)` - Measure the Length Metrics of a Text

## SYNOPSIS

`ase util metric`
[`-h`|`--help`]
[`-f`|`--file` *file*]
[*text* [...]]

## DESCRIPTION

**ase util metric** measures the length metrics of a text, given either
as the content of a *file* or as the *text* formed by the remaining
arguments (joined by single spaces), which are mutually exclusive and
of which exactly one has to be given. The metrics are printed to
standard output as a single-line JSON object with the fields:

- `words`:
The number of whitespace-separated tokens.
 
- `lines`:
The number of newline-separated lines, where a trailing line without
a final newline still counts as one line (and an empty text has- `0`lines).
 
- `chars`:
The number of Unicode characters, i.e., code points.
 
- `bytes`:
The number of octets: the raw file size for a- *file*(as its
encoding is not necessarily UTF-8), resp. the UTF-8 length for a- *text*.
 

A *file* is decoded as UTF-8 for the other metrics. This backs the
MCP tool `ase_text_metric`, which the skills use instead of estimating
a text length themselves.

## OPTIONS

## ARGUMENTS

- [*text*[...]]:
The text to measure, formed by all arguments joined by single spaces
(mutually exclusive with`--file`).

## EXIT STATUS

Exits with status `0` on success, and with status `1` if both or none
of *file* and *text* are given, or *file* cannot be read.

## EXAMPLES

```
# measure a document
$ ase util metric -f README.md
# measure a text
$ ase util metric "Hello, World!"
{"words":2,"lines":1,"chars":13,"bytes":13}
```

## SEE ALSO

`ase(1)`, `ase-util(1)`, `wc(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-util-rasterize(1) - Rasterize an SVG Document or Image for Files and Terminals

## NAME

`ase-util-rasterize(1)` - Rasterize an SVG Document or Image for Files and Terminals

## SYNOPSIS

`ase util rasterize`
[`-h`|`--help`]
[`-f`|`--format` `png`|`kitty`|`iip`|`sixel`|`auto`]
[`-o`|`--output` *file*|`-`]
[`-s`|`--scale` *factor*|`auto`]
[`--width` *columns*]
[`--height` *rows*]
[`-b`|`--background` *color*|`auto`]
[`--font` *file*]
[`--smart`]
*input-file*|`-`|`-d`|`--data` *data*

## DESCRIPTION

**ase util rasterize** rasterizes an SVG document or a PNG, GIF, or JPEG
image (detected by its signature), read from *input-file*, from
standard input (`-`), or from the *data* of `--data`, and writes it as a PNG image or as the terminal
graphics escape sequences of the *Kitty Graphics Protocol*, the *iTerm2
Inline Image Protocol* (IIP), or the *SIXEL* protocol. An image passes
the same scaling and background filling as an SVG document (where of an
animated GIF only the first frame is used). A PNG image with an
effective scale *factor* of `1` and background `auto` is passed through
unchanged to the formats `png`, `kitty`, and `iip`.

The rasterizing uses the WebAssembly build of *resvg*, which cannot
access the system fonts. Hence, the fonts *Source Sans 3* (for
`sans-serif`, `serif`, and unknown families) and *Source Code Pro* (for
`monospace`) are bundled and always loaded, and additional fonts can be
loaded with `--font`.

The input, whether *input-file*, standard input, or *data*, can be the
raw SVG document or image, a `data:` URL of it, or its base64 encoding.

This backs the MCP tool `ase_rasterize`, which returns the PNG image of
an SVG document or PNG/GIF/JPEG image, given either by its content
(parameter `data`, in the same forms as *data*) or by its file path
(parameter `file`, relative to the project root), e.g., of a diagram
rendered by `ase util diagram`, with the same scale (default *factor*
`1`), width, height, background, and smart options, plus a parameter
`maximum`, bounding the width and height of the image in pixels, and a
parameter `output`, returning the PNG image as image content (`inline`,
the default), as a temporary file (`file`), or as a temporary file only
above 2 MiB (`auto`).

## OPTIONS

- [- `-h`|- `--help`]:
Show usage information and exit.
 
- [- `-f`|- `--format`- `png`|- `kitty`|- `iip`|- `sixel`|- `auto`]:
The output format. The default- `auto`chooses- `png`for a- *file*output and for a standard output which is not a terminal. For a
terminal, it chooses the first supported protocol out of- `kitty`,- `iip`, and- `sixel`(but- `iip`first under- *iTerm2*), and fails if no
protocol is detected.
 - Inside - *tmux*,- `auto`prefers the formats which- *tmux*manages as
regular pane content: first- `kitty`if the outer terminal is- *Kitty*or- *Ghostty*(identified by its answer to- *tmux*, its- `TERM`, or its
environment variables), then- `sixel`if- *tmux*supports SIXEL natively
for its client (terminal feature- `sixel`, e.g., enabled via- `tmux set -as terminal-features ",xterm*:sixel"`). Otherwise, it detects the
outer terminal from the- *tmux*session environment and chooses- `iip`or- `sixel`, but never- `kitty`. All other terminal formats are wrapped
into- *tmux*passthrough sequences (failing if the- *tmux*option- `allow-passthrough`is off). For- `kitty`, the image is displayed
through- *Unicode placeholders*(supported by, e.g.,- *Kitty*and- *Ghostty*), i.e., text cells which- *tmux*manages like regular pane
content. For- `iip`and a passed through- `sixel`, the rows of the image
are reserved upfront, but as- *tmux*is unaware of such an image, it
stays on top of the pane content until the outer terminal redraws.
The outer terminal is often hidden from the detection, so an explicit
format may be needed.
 
- [- `-o`|- `--output`- *file*|- `-`]:
Write the output to- *file*instead of standard output (- `-`, the
default).
 
- [- `-s`|- `--scale`- *factor*|- `auto`]:
Scale the image by- *factor*, or with- `auto`(the default) fit it into
the width and height of the terminal (given by- `--width`and- `--height`, else determined from the- `ASE_TERM_WIDTH`and- `ASE_TERM_HEIGHT`environment variables or the standard output).
For- `kitty`and- `iip`, the terminal itself scales
the image onto the fitted character cells. For the pixel size of the
character cells, the one reported by- *tmux*is used, else 10x20
pixels are assumed. For- `png`, or without a known
terminal size,- `auto`means- *factor*- `1`.
 
- [- `--width`- *columns*]:
Use- *columns*characters as the width of the terminal for scale- `auto`, instead of determining it automatically (- `0`, the default).
 
- [- `--height`- *rows*]:
Use- *rows*lines as the height of the terminal for scale- `auto`,
instead of determining it automatically (- `0`, the default). As for
the determined height, two lines are kept free for the subsequent
output.
 
- [- `-b`|- `--background`- *color*|- `auto`]:
Fill the transparent areas with the CSS- *color*, or keep the
transparency with- `auto`(the default).
 
- [- `--font`- *file*]:
Load the additional TrueType or OpenType font- *file*. Can be given
multiple times.
 
- [- `--smart`]:
Resolve the CSS custom properties (- `var(...)`) and color mixings
(- `color-mix(in srgb, ...)`of hexadecimal colors) into plain values
before rasterizing, as- *resvg*supports neither of them. This is
needed for the SVG output of- `ase util diagram`for- *Mermaid*.
 
- [- `-d`|- `--data`- *data*]:
Rasterize- *data*, an SVG document, or an SVG document or PNG/GIF/JPEG
image as- `data:`URL or base64 data, instead of- *input-file*.
 

## ARGUMENTS

- *input-file*|- `-`:
The SVG document or PNG/GIF/JPEG image to rasterize, or- `-`for standard
input. Required, unless- `--data`is given.

## EXIT STATUS

Exits with status `0` on success, and with status `1` if the input
or a font file cannot be read or rasterized, or no terminal
graphics protocol is detected for format `auto`.

## EXAMPLES

```
# show a Mermaid diagram in the terminal
$ echo -e "flowchart LR\nA --> B" | ase util diagram -f svg | ase util rasterize --smart -
# convert an SVG document into a PNG image on a white background
$ ase util rasterize -b white -s 2 -o logo.png logo.svg
# show a PNG image in the terminal
$ ase util rasterize screenshot.png
# show a PNG image in the terminal, fitted into 80x24 characters
$ ase util rasterize --width 80 --height 24 screenshot.png
# show an SVG document, given as a string, in the terminal
$ ase util rasterize -d "$(cat logo.svg)"
# show a base64-encoded PNG image in the terminal
$ ase util rasterize -d "$(base64 < screenshot.png)"
```

## SEE ALSO

`ase(1)`, `ase-util(1)`, `ase-util-diagram(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent(1) - Autonomous Task Lifecycle Agents

## NAME

`ase-agent(1)` - Autonomous Task Lifecycle Agents

## SYNOPSIS

`ase agent`
*agent*
`start`
[*agent-options*]
[`-f`|`--foreground`]

`ase agent`
*agent*
`status`
[`-n`|`--name` *name*]

`ase agent`
*agent*
`log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent`
*agent*
`stop`
[`-n`|`--name` *name*]

`ase agent relay`
`start`|`status`|`log`|`stop`
[*options*]

`ase agent fleet`
`init`|`start`|`status`|`log`|`stop`
[*options*]

## DESCRIPTION

**ase agent** runs the autonomous, headless agents of *ASE*. Each agent
observes the tasks of the task store of the project (see `ase-task(1)`),
claims the tasks it is responsible for, processes each of them by an
*ASE* skill (or a prompt text) through a headless agent tool (*Anthropic
Claude Code CLI*, *GitHub Copilot CLI*, or *OpenAI Codex CLI*), and
settles the task afterwards (status, tag, and assignee), until it is
interrupted or stopped.

The *agent* is one of the following agent kinds:

- the built-in phase agents - `implementer`,- `qualifier`, and- `integrator`, which each drive one phase of the- `enterprise`task
lifecycle model, so that the three agents together realize the
model from the planned to the integrated tasks,
 
- the built-in phase agents - `rebooter`and- `condenser`, which re-create
resp. condense the plans of the tasks of the- `Planning`phase of the- `enterprise`task lifecycle model explicitly assigned to them
(- `DRAFTED`>- `PLANNING`>- `DRAFTED`),
 
- the built-in request agent - `editor`, which processes the edit
requests of tasks in any task lifecycle model,
 
- the built-in agent - `estimator`, which estimates the effort of tasks
in any task lifecycle model, without claiming them (mode- `phase`,
intended for the final estimation), resp. on request (mode- `request`,
intended for dry-runs),
 
- the built-in agent - `researcher`, which researches and grounds the
plans of the tasks of the- `Planning`phase of the- `enterprise`task
lifecycle model explicitly assigned to it (mode- `phase`), resp.
researches a topic (or the plan itself) for tasks in any task
lifecycle model on request (mode- `request`, intended for topic-driven
research),
 
- an agent of a - *definition kind*, whose entire behavior is configured
by a definition file- *kind*- `.`- *ext*of the project or of the user,
 
- the - `adhoc`agent, whose entire behavior is configured by its
options.
 

Each agent runs in its own process, either detached in the background
(the default) or, with `--foreground`, in the current process. Several
agents are best run together as a *fleet* (see `ase-agent-fleet(1)`),
and each agent can pass its live presence to the task boards through
an *agent relay* (see `ase-agent-relay(1)`).

The eight built-in agents are not hard-coded, but each mode of them is a
declarative *definition*, run by the same agent engine as the agents of
the definition kinds and the `adhoc` agent. Hence, every agent accepts
all options `--`*group*`-`*item* of the definitions (see
`ase-agent-adhoc(1)`), which override the definition of the agent,
while a built-in agent keeps the identity of its kind: its runs, its
presence (with its mode), its tags, and its agent-owned worktrees
(*id*`-`*kind*) are named after its kind.

## AGENT NAMES

Every agent has a *name* (option `--name`, default: its kind, e.g.,
`implementer` or `adhoc`), which consists of up to 64 letters, digits,
underscores, and dashes, starting with a letter or digit, and which
must not be the name of another agent kind, `relay`, or `fleet`.

The name is used in the log lines of the agent (also in the shared
processing log attachment `agent`), as the `Assignee:` of the tasks it
processes, for its temporary directory, and for its process files
`.ase/agent/`*name*`.yaml` and `.ase/agent/`*name*`.log`. Hence,
multiple agents of the same kind are started under different names,
e.g., `ase agent implementer start -n impl-claude -t claude` and
`ase agent implementer start -n impl-codex -t codex`.

The agents of the same kind form a *pool*: they compete for the same
tasks, share the same tag (per mode), and process their tasks in
parallel to each other.

## TASK CLAIMING

A *phase* agent (all built-in agents except `editor`, `estimator`, and
`researcher` in the mode `request`, and every definition of `--type phase` without `--unclaimed`) observes the tasks in its *incoming*
state(s) which do not yet carry its *tag*
(plain or with any value) in their `Tags:` frontmatter key, and whose
`Assignee:` frontmatter key is either still empty or already the
assignee name of the agent (default: its name), so a task can be
pre-assigned to a particular agent of a pool, while a task assigned to
anybody else (e.g., a human) is left alone.

An agent can require a task to be already assigned to it (like
`rebooter`, `condenser`, and `researcher` in the mode `phase`, see
`--assign-required`). Such an agent can leave a processed task untagged
(see `--tag-success none`), as the unassignment of the task keeps it from
being claimed again, while assigning it to the agent again requests
another processing.

The tag of a built-in agent is `ase-agent-`*kind*, resp. one per mode
for `implementer` in mode `draft` (`ase-agent-implementer-draft`). The
tag of any other agent is `ase-agent-`*name* (see `--tag-name`).

The agent claims each dispatchable task (in natural task id order,
re-checked on every change of the task store) in a single conditional
save of the task (retried on a concurrent change), which moves it into
its *activity* state, tags it with the value `active` (so no other
agent of this kind picks it up, even if the skill itself puts it back
into the incoming state during its processing), and sets its
`Assignee:` to the agent (for information only, as the tag alone
guards the task).

A task meanwhile tagged or (re)assigned is silently skipped instead.

After the processing, again in a single conditional save, the task is
unassigned (only if still assigned to the agent, and not meanwhile to
somebody else, e.g., the `editor` agent), tagged with its plain tag
(or with the label of the processing as its value, like the verdict of
`qualifier`), so no agent of this kind picks it up again, and moved into
its *success* or *failure* (parking) state, where the pseudo state
`previous` restores the incoming state the task was claimed from (for an
agent claiming tasks from multiple incoming states).

A processing failure, or a processing exceeding the duration of
`--timeout`, always leads to the failure state. A task whose status the
skill itself moved elsewhere is tagged, too, and left in that state.

The preparing of a processing (e.g., creating a worktree) and its
finishing (e.g., committing and pushing a branch) are each limited to a
fixed duration of 240 seconds, where exceeding it also leads to the
failure state.

A task the skill already put into the success state, but whose
finishing failed (e.g., a branch which could not be pushed), and a task
whose resulting state is not reachable from its current state, is
instead tagged with the value `failed`, unassigned, and left in its
state for a manual intervention (which removes the tag), while the
failure is logged as an error. A final status update of a processed
task which fails even after its retries is retried in the background
with a growing delay.

A *request* agent (`editor`, `estimator` and `researcher` in the mode
`request`, and every definition of `--type request`) instead processes the prompt requests attached to tasks in
any lifecycle status (see `ase task request` in `ase-task-request(1)`,
`ase-agent-editor(1)`, and `ase-agent-researcher(1)`).

An *unclaimed* phase agent (`estimator` in the mode `phase`, and every definition of
`--type phase` with `--unclaimed`) instead processes every task in one
of its incoming states (each adapted onto the task lifecycle model of
the project, e.g., `PLANNED` as `PLANNING` for `team` and as `OPEN` for
`solo`) which does not carry its tag yet, independent of its assignee
and its predecessors, without claiming it: it neither moves the task
into an activity state, nor tags it as active, nor assigns it to
itself, and it settles the task by its tag only, so it never changes
its status and its assignee and keeps no other agent off it (see
`ase-agent-estimator(1)`).

## PREDECESSORS

A phase agent claims a task only once all its predecessors passed the
phase of the agent (or no longer exist). The predecessors of a task are
the tasks of its `After:` frontmatter key and, for an explicit epic (a
task whose `Group:` is its own id), implicitly all other tasks of its
group. An agent whose passed state (see below) is the first state of
the task lifecycle model (like `rebooter`, `condenser`, and
`researcher` in the mode `phase`, with `DRAFTED`) hence imposes no gate,
except for a predecessor still being processed by an agent of its kind
or a cancelled `After:` predecessor.

A predecessor has passed the phase once it is in or beyond the
*passed* state of the agent (see `--state-passed`), or in the *success*
state and tagged as processed, where a predecessor still tagged by an
agent of this kind with the value `active` (while in the success state
or not finished) resp. `failed` (while still in the success state) has
not passed it yet.

A cancelled `After:` predecessor never passes (as the task might
depend on its never delivered result), while a cancelled member of an
epic counts as dropped from it. A predecessor status foreign to the
lifecycle model counts as the state it maps onto.

A task still waiting for its predecessors is reported once (per change
of the reason), while a task referencing unknown predecessors (treated
as passed), waiting for cancelled predecessors, or waiting in a
dependency cycle is warned about once. A task with predecessors is
re-checked against a fresh listing right before its claim.

## PROCESSING

Each task is processed through the headless agent tool of `--tool`,
with the environment variable `ASE_HEADLESS=true` and without any
standard input, i.e., via

- `claude`[- `--add-dir`- *dir*...]- `-p --output-format stream-json --verbose --disallowedTools AskUserQuestion --permission-mode auto --permission-prompts none`- *prompt*,
 
- `copilot`[- `--add-dir`- *dir*...]- `--allow-all-tools --no-ask-user --no-color --output-format json -p`- *prompt*, resp.
 
- `codex exec`[- `--add-dir`- *dir*...]- `--json --approve-for-me --color never`- *prompt*,
 

where risky actions are denied or automatically reviewed instead of
being prompted for, and where each additional directory a skill needs
access to besides its working directory is granted through an
additional `--add-dir` *dir*. A skill *prompt* is `/`*skill* *args*
(`claude` and `copilot`) resp. `$`*skill* *args* (`codex`), where an
un-namespaced *ASE* skill (`ase-*`) gets the plugin namespace `ase:`
for `claude` only.

The agent tool and, for an agent of a working directory other than
`project` (i.e., all built-in agents except `rebooter`, `condenser`,
and `estimator`, see `--workdir`), `git` have to be found in `$PATH`.

All processing information, including the output of the agent tool
(which streams its agent messages, tool calls, failures, and final
result as JSONL events), is logged (to standard error, see `ase(1)`,
resp. to the log file of the agent) with timestamp prefixes in real
time, followed by the usage of the agent tool as wall time (measured
locally), output tokens (of `claude` and `codex`), and costs in USD
(of `claude` only), with `n/a` for what the agent tool does not report.

At the end of each processing, the log lines of the processing (with at
most 20 lines of each agent message) are appended to the attachment of
the task with the type `text/plain; charset=utf-8; kind="agent"`
(created if not existing yet), which is shared by all agents, whose log
lines carry their name. If the AI usage accounting is enabled
(`telemetry.enabled`), the usage is additionally booked onto the usage
attachment of the task (without any human time and costs, as the agent
runs unattended).

## INTERRUPTS AND RECOVERY

On an interrupt (`SIGINT`, `SIGTERM`, `SIGHUP`, or `SIGQUIT`, e.g., by
`ase agent` *agent* `stop`), and on the failure of a concurrently run
agent, the running processings are aborted and their tasks are put
back, untagged and with their previous assignee, into the incoming
state (while a task processed unclaimed is left untouched), while a
repeated signal terminates the process at once (leaving the claimed
tasks to the recovery described below).

The claimed tasks of each run are recorded under a run lock in the
*per-user state directory* (below a sub-directory per task store, as
task ids are unique per task store only).

Hence, the tasks of an agent terminated without this cleanup (e.g.,
killed or crashed) are recovered on the next start of an agent of its
kind for the same task store on the same host by the same user: a task
still in the activity state is put back the same way, and any other
task still tagged as `active` is tagged plain (as processed), but only
while it is still assigned to the terminated agent and not claimed
meanwhile by a live agent, where concurrently started agents never
recover the same run.

The requests left in processing by a terminated request agent are
marked as failed instead, and the previous assignees of their tasks
are restored.

## WORKING COPY, CLONES, AND WORKTREES

The agents follow the `Source:`, `Changeset:`, and `Target:`
frontmatter keys of a task (or their configured defaults
`project.task.default.source`, `project.task.default.changeset`, and
`project.task.default.target`, see `ase-config(1)`). Their values are:

- `worktree`:
the Git working copy the agent is started in. For it, the agents
create no own Git worktree, but work in the working copy directly,
just adjusting its checked-out branch.
 - As concurrently processed tasks would interfere there, an agent
handles at most one task at a time in the working copy itself
(other tasks waiting until it is free again). This coordination does
not extend to other agents (each running in its own process, also
within a fleet) or humans working in the same working copy, so a
working copy should be used directly by a single agent only, while
the other agents are given - `--repo`(or process tasks without- `worktree`values only).
 
- `branch:`- *name*:
the Git branch- *name*, checked out in an- *ASE*worktree below- `.ase/worktree/`where needed, which is named after the branch (with
every character other than letters, digits,- `_`, and- `-`replaced by- `-`, runs of- `-`reduced to one, and leading and trailing- `-`and- `_`removed, e.g.,- `feature-x`for- `feature/x`).
 
- `attachment:`- *name*:
the task attachment- *name*(a change set as a diff), handled by the
skills (- `Changeset:`only).
 
- `source`(- `Target:`only):
the- `Source:`of the task.
 

The placeholder `{id}` of a branch or attachment *name* reads as the id
of the task (with every character other than letters, digits, `_`, and
`-` replaced by `-`, runs of `-` reduced to one, and leading and
trailing `-` and `_` removed, e.g., `12` for `#12` and `FOO-12` for
`FOO#12`), e.g., the `Changeset:`
default `branch:cs-{id}` implements every task onto its own branch.

Without `--repo`, all tasks are processed in the Git working copy the
agent is started in, which is hence mandatory (except for an agent of
the working directory `project`, like `rebooter`, `condenser`, and
`estimator`, which work in the project root only, where `rebooter`,
`condenser`, and `estimator` do not accept `--repo` and `--tempdir` at
all, while any other such agent ignores them).

With `--repo` *url*, each agent instead clones the Git repository *url* once
into its temporary directory (removed on termination) and processes every
task without a `worktree` value in its own temporary Git worktree of this
clone (below `work/` of the temporary directory), where the Git operations
on the clone are serialized: before each task, its default branch and the
branches of the task (unless checked out for another task) are refreshed
from the origin, and afterwards a changed `branch:` changeset resp. target
is pushed back to the origin, where a rejected push fails the task.

If such a push fails after the skill already put the task into its
success state, the work is kept as the branch `ase-agent-rescue/`*id*
in the clone, the task is tagged with the value `failed`, and the
temporary directory is preserved (instead of being removed on
termination) for a manual recovery.

Only a task with a `worktree` value then requires the agent to be
started inside a Git working copy. As the skills run inside the clone,
`--repo` requires a task store not located in the working copy (i.e.,
not a local `ase:`*path* one).

In the names of all *ASE* worktrees derived from a task id, a `#` of
the id (e.g., of the `seq:#%d` id scheme of the issue tracker task
stores) is replaced by `_`, as neither a worktree name nor a branch name
can carry it.

The temporary directory of an agent holds the clone and other
temporary files: by default, it is securely created as a fresh, private
directory `ase-agent-`*name*`-`*random* below the system temporary
directory, while `--tempdir` *dir* forces it to be the sub-directory
*name* of *dir*, where `%p` in *dir* is expanded to the process id of
the agent, `%h` to a hash value unique to the process, and `%%` to `%`.

A missing *dir* is created private to the current user (and removed
again on termination if empty), while an existing *dir* has to be a
directory (no symbolic link) owned by the current user. A sub-directory
*name* left over by an earlier run is removed only if its owning
process is no longer alive and it holds no work preserved for a manual
recovery, and otherwise refused.

## AGENT RELAY

With an agent relay (`--agent-relay` *url*, else the configured
`project.relay.url`, given as `ase://`*addr*`:`*port*[`/`*token*]
resp. `ases://`*addr*`:`*port*[`/`*token*][`?insecure`] for HTTPS,
optionally without certificate verification), each agent additionally
passes its live presence to the task boards (see `ase-task-board(1)`):
its state (`waiting`, or `busy` with the ids of its tasks) immediately
on every change, its usage of the agent tool (in total and by its last
job) immediately after each finished job, its new processing log lines
(coalesced to at most four announcements per second), and a heartbeat
every 10 seconds, and it removes its presence on termination.

The relay requires an explicit `project.id`. Its bearer token is the
one embedded in the URL, else `$ASE_AGENT_RELAY_TOKEN`, else
`project.relay.token`, else the token of the locally started agent
relay (or of the locally started task store co-hosting it), if the URL
addresses it.

An unreachable relay is warned about once per outage and never disturbs
the processing of the tasks, where the not yet passed log lines are
kept (at most 100) and passed once the relay is reachable again. A
presence rejected by the relay is warned about with its reason and
retried after a growing delay (from 1 up to 15 minutes).

## DEFINITION KINDS

A *definition kind* is an agent kind of its own, discovered from a
definition file *kind*`.`*ext* of the project (`.ase/agents/`, intended
to be versioned) or of the user (*per-user configuration
directory*`/agents/`), where a definition kind of the project shadows
one of the user, and both shadow a built-in agent of the same kind
(whose `--mode` then no longer exists). For instance,
`.ase/agents/reviewer.mts` yields `ase agent reviewer start`.

The extensions are, in descending precedence, `.yaml`, `.yml`, `.ts`,
`.mts`, `.js`, `.mjs`, and `.cjs`. The definition files are discovered
on the commands `ase agent` ... only, where a file of an invalid kind
name (other than up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit), of the kind name `adhoc`, `relay`, or
`fleet`, or of an already discovered kind name in the same directory
is skipped with a warning.

An agent of a definition kind accepts the same options as
`ase agent adhoc start`, except `--definition`, and `--repo` and
`--tempdir` are silently ignored with `--workdir project`. See
`ase-agent-adhoc(1)` for the format of the definition files, all
options of the definitions, their templates, their hooks, and example
definitions.

## OPTIONS

- [`-h`|`--help`]:
Show usage information of`ase agent`(or of the given sub-command)
only.

All other options belong to the sub-commands (see **COMMANDS** and the
manual pages of the agent kinds).

## COMMANDS

The following sub-command groups exist:

- `ase agent rebooter`- `start`|- `status`|- `log`|- `stop`:
The agent re-creating the plans of the- `DRAFTED`tasks of the- `enterprise`task lifecycle model explicitly assigned to it from
scratch, putting them back into- `DRAFTED`afterwards. See- `ase-agent-rebooter(1)`.
 
- `ase agent condenser`- `start`|- `status`|- `log`|- `stop`:
The agent compressing the wording of the plans of the- `DRAFTED`tasks
of the- `enterprise`task lifecycle model explicitly assigned to it,
putting them back into- `DRAFTED`afterwards. See- `ase-agent-condenser(1)`.
 
- `ase agent implementer`- `start`|- `status`|- `log`|- `stop`:
The agent of the- `Implementation`phase of the- `enterprise`task
lifecycle model, implementing the tasks from their- `Source:`into
their- `Changeset:`, resp. drafting their implementation. See- `ase-agent-implementer(1)`.
 
- `ase agent qualifier`- `start`|- `status`|- `log`|- `stop`:
The agent of the- `Qualification`phase of the- `enterprise`task
lifecycle model, reviewing the- `Changeset:`of the tasks. See- `ase-agent-qualifier(1)`.
 
- `ase agent integrator`- `start`|- `status`|- `log`|- `stop`:
The agent of the- `Integration`phase of the- `enterprise`task
lifecycle model, integrating the- `Changeset:`of the tasks into their- `Target:`. See- `ase-agent-integrator(1)`.
 
- `ase agent editor`- `start`|- `status`|- `log`|- `stop`:
The agent processing the edit requests of tasks in any task lifecycle
model. See- `ase-agent-editor(1)`.
 
- `ase agent estimator`- `start`|- `status`|- `log`|- `stop`:
The agent estimating the effort of the tasks without a valid- `Effort:`key in any task lifecycle model, without claiming them
(mode- `phase`, intended for the final estimation), resp. of the tasks
of the- `estimate`requests (mode- `request`, intended for dry-runs).
See- `ase-agent-estimator(1)`.
 
- `ase agent researcher`- `start`|- `status`|- `log`|- `stop`:
The agent researching the plans of the- `DRAFTED`tasks of the- `enterprise`task lifecycle model explicitly assigned to it
(mode- `phase`, intended for researching and grounding the plan
content), resp. processing the research requests of tasks in any task
lifecycle model (mode- `request`, intended for topic-driven research),
appending a research report, grounded in the Web and in the source
artifacts of the project, to the- `research`attachment of the tasks.
See- `ase-agent-researcher(1)`.
 
- `ase agent`- *kind*- `start`|- `status`|- `log`|- `stop`:
The agent of the definition kind- *kind*(see- **DEFINITION KINDS**),
with the same sub-commands and options as- `ase agent adhoc`, except- `--definition`. See- `ase-agent-adhoc(1)`.
 
- `ase agent adhoc`- `start`|- `status`|- `log`|- `stop`:
An ad-hoc agent, whose entire behavior is given by its options. See- `ase-agent-adhoc(1)`.
 
- `ase agent relay`- `start`|- `status`|- `log`|- `stop`:
The per-user agent relay server, passing the live presence of the
agents to the task boards. See- `ase-agent-relay(1)`.
 
- `ase agent fleet`- `init`|- `start`|- `status`|- `log`|- `stop`:
A fleet of agents (and of the agent relay they report to), running
each in its own process and restarting it whenever it exits. See- `ase-agent-fleet(1)`.
 

The sub-commands of every agent kind *agent* are:

- `ase agent`- *agent*- `start`[- *agent-options*] [- `-f`|- `--foreground`]:
Start the agent in the background, logging into- `.ase/agent/`- *name*- `.log`. The agent is validated and initialized
(e.g., its agent tool is looked up and the task lifecycle model is
checked) before the command returns (waiting up to 30 seconds), so a
failing agent is reported together with the tail of its log. An agent
of the same name already running is left alone (exit status 0).
 - With - `--foreground`, the agent runs in the current process instead,
logging to the terminal, until it is interrupted or stopped, where an
agent of the same name already running is an error.
 
- `ase agent`- *agent*- `status`[- `-n`|- `--name`- *name*]:
Report whether the agent of the name- *name*(default: its kind) is
running (with its process id).
 
- `ase agent`- *agent*- `log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*] [- `--name`- *name*]:
Show the log- `.ase/agent/`- *name*- `.log`of the agent of the name- *name*(default: its kind) run in the background, where an agent of a fleet
logs into the log of the fleet instead (see- `ase-agent-fleet(1)`).
With- `--lines`, only the last- *n*lines are shown. With- `--follow`,
the last- *n*lines (default: 10) are shown and then all newly appended
lines are followed until interrupted. As- `-n`is the short form of- `--lines`, the name is given via- `--name`only.
 
- `ase agent`- *agent*- `stop`[- `-n`|- `--name`- *name*]:
Stop the agent of the name- *name*(default: its kind) by- `SIGTERM`,
waiting (up to 5 minutes) until it aborted its running processings
and put their tasks back.
 

## ENVIRONMENT

- `ASE_TOOL`:
The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`:
The bearer token of the agent relay, if not embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`:
Set to- `true`for the agent tool processes spawned by the agents.
 
- `PATH`:
Searched for the agent tool (- `claude`,- `copilot`, or- `codex`) and- `git`.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`:
The process state (kind, process id, and process start time) of the
agent named- *name*, existing while the agent runs.
 
- `.ase/agent/`- *name*- `.log`:
The log of the agent named- *name*run in the background.
 
- `.ase/agents/`- *kind*- `.`- *ext*,- *per-user configuration directory*- `/agents/`- *kind*- `.`- *ext*:
The definition files of the definition kinds of the project resp. of
the user.
 
- `.ase/worktree/`:
The- *ASE*worktrees created by the agents (and the skills).
 
- *per-user state directory*- `/agent-`- *kind*- `/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`:
The run lock (a directory, refreshed periodically and considered stale
after 30 seconds without refresh) and the run record (process id, process
start time, and claimed tasks) of each agent process of agent- *kind*(- `rebooter`,- `condenser`,- `implementer`,- `qualifier`,- `integrator`,- `editor`,- `estimator`, or- `researcher`, resp.- `tag-`- *word*per tag- `ase-agent-`- *word*of the
agents of the definition kinds and of- `adhoc`), where- *store-hash*is
the first 12 hex digits of the SHA-1 digest of the identity of the
task store.
 - On termination, the lock is removed, and so is the record, unless it
still records claimed tasks. 

## EXIT STATUS

`ase agent` *agent* `start` exits with status `0` once the agent
started (or was already running), resp. once a foreground agent
terminated regularly, and with status `1` on any error (e.g., an
invalid option, a missing agent tool, an unsupported task lifecycle
model, or, with `--one-shot`, a task store failing three checks in a
row). `ase agent` *agent* `status` exits with status `1` if the agent is
not running. `ase agent` without a sub-command shows its usage
information and exits with status `1`.

## EXAMPLES

```
# start the phase agents of the "enterprise" lifecycle model
$ ase agent rebooter start
$ ase agent condenser start
$ ase agent researcher start
$ ase agent implementer start -p 2
$ ase agent qualifier start -m judge
$ ase agent integrator start -m squash
# start two implementers of different agent tools as a pool
$ ase agent implementer start -n impl-claude -t claude
$ ase agent implementer start -n impl-codex -t codex
# follow the log of an agent and stop it again
$ ase agent implementer log -f --name impl-codex
$ ase agent implementer stop -n impl-codex
```

## SEE ALSO

`ase(1)`, `ase-agent-rebooter(1)`, `ase-agent-condenser(1)`, `ase-agent-implementer(1)`,
`ase-agent-qualifier(1)`, `ase-agent-integrator(1)`,
`ase-agent-editor(1)`, `ase-agent-estimator(1)`, `ase-agent-researcher(1)`,
`ase-agent-adhoc(1)`, `ase-agent-relay(1)`, `ase-agent-fleet(1)`, `ase-task(1)`,
`ase-task-board(1)`, `ase-task-request(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-rebooter(1) - Plan Rebooting Agent

## NAME

`ase-agent-rebooter(1)` - Plan Rebooting Agent

## SYNOPSIS

`ase agent rebooter start`
[`-n`|`--name` *name*]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-M`|`--model` *model*]
[`--`*group*`-`*item* *value* [...]]
[`-f`|`--foreground`]

`ase agent rebooter status`
[`-n`|`--name` *name*]

`ase agent rebooter log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent rebooter stop`
[`-n`|`--name` *name*]

## DESCRIPTION

**ase agent rebooter** re-creates the plans of tasks in the `Planning`
phase of the `enterprise` task lifecycle model (refused for any other
model) from scratch.

As a plan is re-created only on the explicit request of a human, the
rebooter claims only the tasks in the state `DRAFTED` whose `Assignee:`
frontmatter key already is its agent name (default: `rebooter`), i.e.,
which a human assigned to the rebooter. Such a task is claimed
immediately, without waiting for its `After:` predecessors (and, for an
explicit epic, its group members), except for a predecessor still being
processed by a rebooter or a cancelled `After:` predecessor.

The rebooter moves each claimed task into `PLANNING`, tags it with
`ase-agent-rebooter:active`, and reboots its plan via the skill
invocation `ase-task-reboot --next DONE` *id*. The skill itself resets a
successfully rebooted plan into `DRAFTED` (the processing succeeds only
then). Afterwards, on success as well as on failure, the rebooter puts
the task back into `DRAFTED` (for its review and its promotion into
`PLANNED` by a human), removes its tag, and unassigns it. As the task is
no longer assigned to the rebooter, it is not claimed again, while
assigning it to the rebooter again requests another reboot.

On an interrupt, a task being processed is put back, untagged and with
its previous assignee (i.e., still assigned to the rebooter), into
`DRAFTED`. The rebooter works in the project root only, so it
neither requires a Git working copy nor `git`, and it accepts no
`--repo` and `--tempdir`. See `ase-agent(1)` for the common mechanics of
all agents.

## OPTIONS

- [`-h`|`--help`]: Show usage information of`ase agent rebooter`(or
of the given sub-command) only.

All other options belong to the sub-commands (see **COMMANDS**).

## COMMANDS

- `ase agent rebooter start`[- *options*]: Start the rebooter agent in
the background, logging into- `.ase/agent/`- *name*- `.log`. The agent is
validated and initialized (e.g., its agent tool is looked up and the
task lifecycle model is checked) before the command returns (waiting
up to 30 seconds), so a failing agent is reported together with the
tail of its log. An agent of the same name already running is left
alone. The options are:
 - - [- `-n`|- `--name`- *name*]: The name of the agent (default:- `rebooter`): up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit, other than the name of another
agent kind,- `relay`, or- `fleet`.
 - It names the agent in its log lines (also in the shared processing
log attachment - `agent`), the- `Assignee:`of the tasks it
processes, and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`. Agents of the kind- `rebooter`under
different names form a pool, competing for the same tasks, but each
claims only the tasks explicitly assigned to its own name.
 
- [- `-t`|- `--tool`- *tool*]: The headless agent tool processing the
tasks:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub
Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*) (default:- `$ASE_TOOL`, else- `claude`). Its command has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]: The maximum number of tasks processed
in parallel by this agent (default:- `1`), a positive integer.
 
- [- `-T`|- `--timeout`- *duration*]: The maximum duration of
processing a task (the run of the agent tool), as- *number*- *unit*(without a space) with- *unit*- `s`(seconds),- `m`(minutes), or- `h`(hours) (default:- `60m`). A processing exceeding it is aborted and
counts as failed.
 
- [- `-o`|- `--one-shot`]: Terminate the agent, instead of observing
for further tasks, once a check of the task store finds no
dispatchable task while the agent processes no task anymore --
intended for CI contexts like GitHub Actions, and not allowed in a
fleet. A task still waiting for its predecessors at that time is
left for a later run. A check of the task store failing three
times in a row then terminates the agent with a non-zero exit
status (instead of being retried endlessly after a growing delay).
 
- [- `--agent-relay`- *url*]: The agent relay to pass the live
presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for
HTTPS (default: the configured- `project.relay.url`, else none).
See- **AGENT RELAY**in- `ase-agent(1)`.
 
- [- `--title`- *text*]: The one-line title of the operation of the
agent (at most 80 characters), shown on the task boards. Built-in:- `Reboot the plans of assigned tasks`.
 
- [- `--description`- *text*]: The brief description of the operation
of the agent (about 3-6 lines, at most 600 characters, with its
whitespace collapsed), shown on the task boards. Built-in: a
description of the operation.
 
- [- `--type`- `phase`|- `request`]: The processing type:- `phase`claims the tasks by their lifecycle status,- `request`processes
the prompt requests attached to the tasks. Generic default:- `phase`. Built-in:- `phase`.
 
- [- `--workdir`- *kind*]: The working directory of the agent tool:- `project`,- `source`,- `changeset`,- `changeset-read`, or- `target`.
Generic default:- `project`. Built-in:- `project`.
 
- [- `--lifecycle`- *model*]: A supported task lifecycle model
(- `solo`,- `team`, or- `enterprise`, repeatable). Generic default:
every model containing all given states, resp. all models for- `request`and with- `--unclaimed`. Built-in:- `enterprise`.
 
- [- `--serial`]: Process the tasks serially only, ignoring- `--parallel`. Built-in: not set.
 
- [- `--unclaimed`]: Process the tasks of the incoming states
without claiming them, independent of their assignee and
predecessors, and without ever changing their status and assignee
(- `phase`only). Built-in: not set. Not allowed, as the rebooter
claims the tasks into its activity state.
 
- [- `-M`|- `--model`- *model*]: The model of the agent tool: a model
for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all other
tasks), see- `ase-agent-adhoc(1)`. Generic default: the default
model of the agent tool. Built-in: none.
 
- [- `--params`- *name*- `=`- *value*]: The value of a parameter
declared by the definition (repeatable). Built-in: none declared.
 
- [- `--state-incoming`- *state*]: A state of the tasks to claim
(mandatory for- `phase`, repeatable with- `--unclaimed`or- `--state-activity`only). Built-in:- `DRAFTED`.
 
- [- `--state-activity`- *state*]: The state of a task during its
processing. Generic default: the incoming state. Built-in:- `PLANNING`.
 
- [- `--state-success`- *state*]: The target state on success, or- `previous`for the incoming state of the claim. Generic default:
the incoming state. Built-in:- `DRAFTED`.
 
- [- `--state-failure`- *state*]: The target state on failure, or- `previous`for the incoming state of the claim. Generic default:
the incoming state. Built-in:- `DRAFTED`.
 
- [- `--state-phase`- *state*]: The phase success state, into which
the skill itself can move a task. Generic default: the success
state, resp. the activity state for a- `previous`success state.
Built-in: not set (hence- `DRAFTED`, the default state, into which
the skill resets a rebooted plan).
 
- [- `--state-passed`- *state*]: The first state from which on a task
has passed the phase for its- `After:`successors. Generic default:
the phase success state. Built-in: not set (hence- `DRAFTED`, the
first state, so the predecessors impose no gate).
 
- [- `--state-verdict`- *VERDICT*- `=`- *state*]: The target state per
verdict (repeatable). Built-in: not set.
 
- [- `--tag-name`- *tag*]: The tag of the processed tasks
(- `ase-agent-`- *word*). Generic default:- `ase-agent-`- *name*, resp.
for- `request`always- `ase-agent-`- *request-name*. Built-in: not set.
 
- [- `--tag-success`- *tag*]: The tag on success:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in:- `none`.
 
- [- `--tag-failure`- *tag*]: The tag on failure:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in:- `none`.
 
- [- `--tag-require`- *tag*]: A tag a claimable task has to carry
(repeatable). Built-in: not set.
 
- [- `--tag-exclude`- *tag*]: A tag a claimable task must not carry
(repeatable). Built-in: not set.
 
- [- `--tag-retry`]: A shorthand for- `--tag-failure none`, so a
failed task is claimed again after a growing delay (- `phase`only).
Built-in: not set.
 
- [- `--assign-name`- *who*]: The assignee under which the tasks are
claimed, held, recognized, and released, and which names the
agent-owned worktrees. Generic default: the agent name. Built-in:
not set.
 
- [- `--assign-required`]: Claim only the tasks explicitly assigned
to- *who*of- `--assign-name`(- `phase`only). Built-in: enabled.
 
- [- `--assign-success`- *who*]: The assignee of a task after its
successful processing: a name,- `none`, or- `previous`. Generic
default:- `none`for- `phase`,- `previous`for- `request`. Built-in:
not set.
 
- [- `--assign-failure`- *who*]: The assignee of a task after its
failed processing: a name,- `none`, or- `previous`. Generic default:- `none`for- `phase`,- `previous`for- `request`. Built-in: not set.
 
- [- `--request-name`- *name*]: The name of the processed requests
(- `request`only). Generic default: the agent name. Built-in: not
set. Not allowed, as the rebooter is of type- `phase`.
 
- [- `--request-done`- *word*]: The word naming a processed task in
the log lines (- `request`only). Generic default:- `processed`.
Built-in: not set. Not allowed, as the rebooter is of type- `phase`.
 
- [- `--prompt-skill`- *name*]: The skill to invoke (mutually
exclusive with- `--prompt-text`and- `--prompt-file`). Built-in:- `ase-task-reboot`.
 
- [- `--prompt-args`- *template*]: The argument template of the
skill. Generic default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`. Built-in:- `--next DONE {id}`.
 
- [- `--prompt-text`- *template*]: The prompt text template to invoke
instead of a skill. Built-in: not set.
 
- [- `--prompt-file`- *file*]: The file with the prompt text template
to invoke instead of a skill. Built-in: not set.
 
- [- `--prepare-run`- *cmd*]: A shell command run in the working
directory before the prompt (repeatable). Built-in: not set.
 
- [- `--success-criterion`- *criterion*]: A success criterion:- `exit`,- `clean`,- `changed`,- `verdict`,- `attachment`, or- `status`(repeatable). Generic default:- `exit`and- `clean`for- `phase`,- `clean`and- `changed`for- `request`. Built-in:- `status`.
 
- [- `--success-match`- *regex*]: A regular expression an output line
has to match (repeatable). Built-in: not set.
 
- [- `--verdict-label`- *label*]: The label after which the verdict
is given in the output. Built-in: not set.
 
- [- `--verdict-allow`- *VERDICT*]: An accepted verdict (repeatable).
Generic default: the verdicts of- `--state-verdict`, else any.
Built-in: not set.
 
- [- `--attach-output`- *kind*]: The kind of the attachment storing
the output lines of the prompt. Built-in: not set.
 
- [- `--attach-expect`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*to be created or changed by the
processing for the success criterion- `attachment`. Built-in: not
set.
 
- [- `--attach-skip`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*leaving a task unclaimed (- `phase`only).
Built-in: not set.
 
- [- `--attach-changeset`- *kind*]: The kind of the attachment- `text/x-diff;`- *kind*storing the change set snapshot of the task
(with- `--workdir changeset`or- `target`only). Built-in: not set.
 
- [- `--finish-commit`]: Commit the work onto a- `branch:`changeset
and publish it (with- `--workdir changeset`only). Built-in: not
set. Not allowed, as the rebooter works in the project root.
 
- [- `--finish-success`- *cmd*]: A shell command run in the working
directory after a successful processing. Built-in: not set.
 
- [- `--finish-failure`- *cmd*]: A shell command run in the working
directory after a failed processing. Built-in: not set.
 
- [- `-f`|- `--foreground`]: Run the agent in the current process
instead, logging to the terminal, until it is interrupted (e.g.,
via- `CTRL+C`) or stopped, where an agent of the same name already
running is an error.
 
- [- `-h`|- `--help`]: Show the usage information of the sub-command
only.
 
 - The options - `--`- *group*- `-`- *item*above override the built-in
definition of the agent (where an option also replaces its mutually
exclusive alternatives, e.g.,- `--prompt-text`replaces the skill of- `--prompt-skill`), while the agent keeps the identity of its kind (its
runs, its presence, and its agent-owned worktrees are named after- `rebooter`). See- `ase-agent-adhoc(1)`for their full semantics.
 
- `ase agent rebooter status`[- `-n`|- `--name`- *name*]: Report whether
the rebooter agent of the name- *name*(default:- `rebooter`) is
running, as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`(exit status 1).
 
- `ase agent rebooter log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]
[- `--name`- *name*]: Show the log- `.ase/agent/`- *name*- `.log`of the
rebooter agent of the name- *name*(default:- `rebooter`) run in the
background, where an agent of a fleet logs into the log of the fleet
instead (see- `ase-agent-fleet(1)`). Without a log file, an
informational message is shown only. The options are:
 - - [- `-f`|- `--follow`]: Show the last- *n*lines (default: 10) and
then follow all newly appended lines, in the style of- `tail -f`,
until interrupted.
 
- [- `-n`|- `--lines`- *n*]: Show the last- *n*lines only (default:
all, or 10 with- `--follow`).
 
- [- `--name`- *name*]: The name of the agent (default:- `rebooter`),
given in its long form only, as- `-n`is the short form of- `--lines`.
 
 
- `ase agent rebooter stop`[- `-n`|- `--name`- *name*]: Stop the rebooter
agent of the name- *name*(default:- `rebooter`) by- `SIGTERM`, waiting
(up to 5 minutes) until it aborted its running processings and put
their tasks back. An agent not running is reported informationally
only.
 

## ENVIRONMENT

- `ASE_TOOL`: The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`: The bearer token of the agent relay, if not
embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`: Set to- `true`for the agent tool processes spawned by
the agent.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`: The process state (kind, process id, and
process start time) of the agent, existing while it runs.
 
- `.ase/agent/`- *name*- `.log`: The log of the agent run in the background.
 
- *per-user state
directory*- `/agent-rebooter/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`:
The run lock and the run record of each rebooter agent process, used
to recover the tasks of terminated agent processes (see- **INTERRUPTS
AND RECOVERY**in- `ase-agent(1)`).
 

## EXIT STATUS

`ase agent rebooter start` exits with status `0` once the agent started
(or was already running), resp. once a foreground agent terminated
regularly, and with status `1` on any error (e.g., an invalid option, a
task lifecycle model other than `enterprise`, or a missing agent tool).
`ase agent rebooter status` exits with status `1` if the agent is not
running.

## EXAMPLES

```
# start the rebooter, rebooting the plans of the tasks assigned to it
$ ase agent rebooter start
# request the reboot of a drafted task by assigning it to the rebooter
$ ase task assign FOO-42 rebooter
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-condenser(1)`,
`ase-agent-researcher(1)`, `ase-agent-implementer(1)`,
`ase-agent-qualifier(1)`, `ase-agent-integrator(1)`,
`ase-agent-editor(1)`, `ase-agent-estimator(1)`, `ase-agent-adhoc(1)`,
`ase-agent-fleet(1)`, `ase-agent-relay(1)`, `ase-task(1)`,
`ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-condenser(1) - Plan Condensing Agent

## NAME

`ase-agent-condenser(1)` - Plan Condensing Agent

## SYNOPSIS

`ase agent condenser start`
[`-n`|`--name` *name*]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-M`|`--model` *model*]
[`--`*group*`-`*item* *value* [...]]
[`-f`|`--foreground`]

`ase agent condenser status`
[`-n`|`--name` *name*]

`ase agent condenser log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent condenser stop`
[`-n`|`--name` *name*]

## DESCRIPTION

**ase agent condenser** compresses the wording of the plans of tasks in
the `Planning` phase of the `enterprise` task lifecycle model (refused
for any other model), while preserving all their semantics.

As a plan is condensed only on the explicit request of a human, the
condenser claims only the tasks in the state `DRAFTED` whose `Assignee:`
frontmatter key already is its agent name (default: `condenser`), i.e.,
which a human assigned to the condenser. Such a task is claimed
immediately, without waiting for its `After:` predecessors (and, for an
explicit epic, its group members), except for a predecessor still being
processed by a condenser or a cancelled `After:` predecessor.

The condenser moves each claimed task into `PLANNING`, tags it with
`ase-agent-condenser:active`, and condenses its plan via the skill
invocation `ase-task-condense --next DONE` *id*. The processing succeeds
if the agent tool succeeded and the skill reported no error, which
includes an already condensed plan, left untouched by the skill.
Afterwards, on success as well as on failure, the condenser puts the
task back into `DRAFTED`, removes its tag, and unassigns it. As the
task is no longer assigned to the condenser, it is not claimed again,
while assigning it to the condenser again requests another condensing.

On an interrupt, a task being processed is put back, untagged and with
its previous assignee (i.e., still assigned to the condenser), into
`DRAFTED`. The condenser works in the project root only, so it
neither requires a Git working copy nor `git`, and it accepts no
`--repo` and `--tempdir`. See `ase-agent(1)` for the common mechanics of
all agents.

## OPTIONS

- [`-h`|`--help`]: Show usage information of`ase agent condenser`(or
of the given sub-command) only.

All other options belong to the sub-commands (see **COMMANDS**).

## COMMANDS

- `ase agent condenser start`[- *options*]: Start the condenser agent in
the background, logging into- `.ase/agent/`- *name*- `.log`. The agent is
validated and initialized (e.g., its agent tool is looked up and the
task lifecycle model is checked) before the command returns (waiting
up to 30 seconds), so a failing agent is reported together with the
tail of its log. An agent of the same name already running is left
alone. The options are:
 - - [- `-n`|- `--name`- *name*]: The name of the agent (default:- `condenser`): up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit, other than the name of another
agent kind,- `relay`, or- `fleet`.
 - It names the agent in its log lines (also in the shared processing
log attachment - `agent`), the- `Assignee:`of the tasks it
processes, and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`. Agents of the kind- `condenser`under
different names form a pool, competing for the same tasks, but each
claims only the tasks explicitly assigned to its own name.
 
- [- `-t`|- `--tool`- *tool*]: The headless agent tool processing the
tasks:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub
Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*) (default:- `$ASE_TOOL`, else- `claude`). Its command has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]: The maximum number of tasks processed
in parallel by this agent (default:- `1`), a positive integer.
 
- [- `-T`|- `--timeout`- *duration*]: The maximum duration of
processing a task (the run of the agent tool), as- *number*- *unit*(without a space) with- *unit*- `s`(seconds),- `m`(minutes), or- `h`(hours) (default:- `60m`). A processing exceeding it is aborted and
counts as failed.
 
- [- `-o`|- `--one-shot`]: Terminate the agent, instead of observing
for further tasks, once a check of the task store finds no
dispatchable task while the agent processes no task anymore --
intended for CI contexts like GitHub Actions, and not allowed in a
fleet. A task still waiting for its predecessors at that time is
left for a later run. A check of the task store failing three
times in a row then terminates the agent with a non-zero exit
status (instead of being retried endlessly after a growing delay).
 
- [- `--agent-relay`- *url*]: The agent relay to pass the live
presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for
HTTPS (default: the configured- `project.relay.url`, else none).
See- **AGENT RELAY**in- `ase-agent(1)`.
 
- [- `--title`- *text*]: The one-line title of the operation of the
agent (at most 80 characters), shown on the task boards. Built-in:- `Condense the plans of assigned tasks`.
 
- [- `--description`- *text*]: The brief description of the operation
of the agent (about 3-6 lines, at most 600 characters, with its
whitespace collapsed), shown on the task boards. Built-in: a
description of the operation.
 
- [- `--type`- `phase`|- `request`]: The processing type:- `phase`claims the tasks by their lifecycle status,- `request`processes
the prompt requests attached to the tasks. Generic default:- `phase`. Built-in:- `phase`.
 
- [- `--workdir`- *kind*]: The working directory of the agent tool:- `project`,- `source`,- `changeset`,- `changeset-read`, or- `target`.
Generic default:- `project`. Built-in:- `project`.
 
- [- `--lifecycle`- *model*]: A supported task lifecycle model
(- `solo`,- `team`, or- `enterprise`, repeatable). Generic default:
every model containing all given states, resp. all models for- `request`and with- `--unclaimed`. Built-in:- `enterprise`.
 
- [- `--serial`]: Process the tasks serially only, ignoring- `--parallel`. Built-in: not set.
 
- [- `--unclaimed`]: Process the tasks of the incoming states
without claiming them, independent of their assignee and
predecessors, and without ever changing their status and assignee
(- `phase`only). Built-in: not set. Not allowed, as the condenser
claims the tasks into its activity state.
 
- [- `-M`|- `--model`- *model*]: The model of the agent tool: a model
for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all other
tasks), see- `ase-agent-adhoc(1)`. Generic default: the default
model of the agent tool. Built-in: none.
 
- [- `--params`- *name*- `=`- *value*]: The value of a parameter
declared by the definition (repeatable). Built-in: none declared.
 
- [- `--state-incoming`- *state*]: A state of the tasks to claim
(mandatory for- `phase`, repeatable with- `--unclaimed`or- `--state-activity`only). Built-in:- `DRAFTED`.
 
- [- `--state-activity`- *state*]: The state of a task during its
processing. Generic default: the incoming state. Built-in:- `PLANNING`.
 
- [- `--state-success`- *state*]: The target state on success, or- `previous`for the incoming state of the claim. Generic default:
the incoming state. Built-in:- `DRAFTED`.
 
- [- `--state-failure`- *state*]: The target state on failure, or- `previous`for the incoming state of the claim. Generic default:
the incoming state. Built-in:- `DRAFTED`.
 
- [- `--state-phase`- *state*]: The phase success state, into which
the skill itself can move a task. Generic default: the success
state, resp. the activity state for a- `previous`success state.
Built-in: not set (hence- `DRAFTED`).
 
- [- `--state-passed`- *state*]: The first state from which on a task
has passed the phase for its- `After:`successors. Generic default:
the phase success state. Built-in: not set (hence- `DRAFTED`, the
first state, so the predecessors impose no gate).
 
- [- `--state-verdict`- *VERDICT*- `=`- *state*]: The target state per
verdict (repeatable). Built-in: not set.
 
- [- `--tag-name`- *tag*]: The tag of the processed tasks
(- `ase-agent-`- *word*). Generic default:- `ase-agent-`- *name*, resp.
for- `request`always- `ase-agent-`- *request-name*. Built-in: not set.
 
- [- `--tag-success`- *tag*]: The tag on success:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in:- `none`.
 
- [- `--tag-failure`- *tag*]: The tag on failure:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in:- `none`.
 
- [- `--tag-require`- *tag*]: A tag a claimable task has to carry
(repeatable). Built-in: not set.
 
- [- `--tag-exclude`- *tag*]: A tag a claimable task must not carry
(repeatable). Built-in: not set.
 
- [- `--tag-retry`]: A shorthand for- `--tag-failure none`, so a
failed task is claimed again after a growing delay (- `phase`only).
Built-in: not set.
 
- [- `--assign-name`- *who*]: The assignee under which the tasks are
claimed, held, recognized, and released, and which names the
agent-owned worktrees. Generic default: the agent name. Built-in:
not set.
 
- [- `--assign-required`]: Claim only the tasks explicitly assigned
to- *who*of- `--assign-name`(- `phase`only). Built-in: enabled.
 
- [- `--assign-success`- *who*]: The assignee of a task after its
successful processing: a name,- `none`, or- `previous`. Generic
default:- `none`for- `phase`,- `previous`for- `request`. Built-in:
not set.
 
- [- `--assign-failure`- *who*]: The assignee of a task after its
failed processing: a name,- `none`, or- `previous`. Generic default:- `none`for- `phase`,- `previous`for- `request`. Built-in: not set.
 
- [- `--request-name`- *name*]: The name of the processed requests
(- `request`only). Generic default: the agent name. Built-in: not
set. Not allowed, as the condenser is of type- `phase`.
 
- [- `--request-done`- *word*]: The word naming a processed task in
the log lines (- `request`only). Generic default:- `processed`.
Built-in: not set. Not allowed, as the condenser is of type- `phase`.
 
- [- `--prompt-skill`- *name*]: The skill to invoke (mutually
exclusive with- `--prompt-text`and- `--prompt-file`). Built-in:- `ase-task-condense`.
 
- [- `--prompt-args`- *template*]: The argument template of the
skill. Generic default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`. Built-in:- `--next DONE {id}`.
 
- [- `--prompt-text`- *template*]: The prompt text template to invoke
instead of a skill. Built-in: not set.
 
- [- `--prompt-file`- *file*]: The file with the prompt text template
to invoke instead of a skill. Built-in: not set.
 
- [- `--prepare-run`- *cmd*]: A shell command run in the working
directory before the prompt (repeatable). Built-in: not set.
 
- [- `--success-criterion`- *criterion*]: A success criterion:- `exit`,- `clean`,- `changed`,- `verdict`,- `attachment`, or- `status`(repeatable). Generic default:- `exit`and- `clean`for- `phase`,- `clean`and- `changed`for- `request`. Built-in:- `exit`and- `clean`.
 
- [- `--success-match`- *regex*]: A regular expression an output line
has to match (repeatable). Built-in: not set.
 
- [- `--verdict-label`- *label*]: The label after which the verdict
is given in the output. Built-in: not set.
 
- [- `--verdict-allow`- *VERDICT*]: An accepted verdict (repeatable).
Generic default: the verdicts of- `--state-verdict`, else any.
Built-in: not set.
 
- [- `--attach-output`- *kind*]: The kind of the attachment storing
the output lines of the prompt. Built-in: not set.
 
- [- `--attach-expect`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*to be created or changed by the
processing for the success criterion- `attachment`. Built-in: not
set.
 
- [- `--attach-skip`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*leaving a task unclaimed (- `phase`only).
Built-in: not set.
 
- [- `--attach-changeset`- *kind*]: The kind of the attachment- `text/x-diff;`- *kind*storing the change set snapshot of the task
(with- `--workdir changeset`or- `target`only). Built-in: not set.
 
- [- `--finish-commit`]: Commit the work onto a- `branch:`changeset
and publish it (with- `--workdir changeset`only). Built-in: not
set. Not allowed, as the condenser works in the project root.
 
- [- `--finish-success`- *cmd*]: A shell command run in the working
directory after a successful processing. Built-in: not set.
 
- [- `--finish-failure`- *cmd*]: A shell command run in the working
directory after a failed processing. Built-in: not set.
 
- [- `-f`|- `--foreground`]: Run the agent in the current process
instead, logging to the terminal, until it is interrupted (e.g.,
via- `CTRL+C`) or stopped, where an agent of the same name already
running is an error.
 
- [- `-h`|- `--help`]: Show the usage information of the sub-command
only.
 
 - The options - `--`- *group*- `-`- *item*above override the built-in
definition of the agent (where an option also replaces its mutually
exclusive alternatives, e.g.,- `--prompt-text`replaces the skill of- `--prompt-skill`), while the agent keeps the identity of its kind (its
runs, its presence, and its agent-owned worktrees are named after- `condenser`). See- `ase-agent-adhoc(1)`for their full semantics.
 
- `ase agent condenser status`[- `-n`|- `--name`- *name*]: Report whether
the condenser agent of the name- *name*(default:- `condenser`) is
running, as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`(exit status 1).
 
- `ase agent condenser log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]
[- `--name`- *name*]: Show the log- `.ase/agent/`- *name*- `.log`of the
condenser agent of the name- *name*(default:- `condenser`) run in the
background, where an agent of a fleet logs into the log of the fleet
instead (see- `ase-agent-fleet(1)`). Without a log file, an
informational message is shown only. The options are:
 - - [- `-f`|- `--follow`]: Show the last- *n*lines (default: 10) and
then follow all newly appended lines, in the style of- `tail -f`,
until interrupted.
 
- [- `-n`|- `--lines`- *n*]: Show the last- *n*lines only (default:
all, or 10 with- `--follow`).
 
- [- `--name`- *name*]: The name of the agent (default:- `condenser`),
given in its long form only, as- `-n`is the short form of- `--lines`.
 
 
- `ase agent condenser stop`[- `-n`|- `--name`- *name*]: Stop the condenser
agent of the name- *name*(default:- `condenser`) by- `SIGTERM`, waiting
(up to 5 minutes) until it aborted its running processings and put
their tasks back. An agent not running is reported informationally
only.
 

## ENVIRONMENT

- `ASE_TOOL`: The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`: The bearer token of the agent relay, if not
embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`: Set to- `true`for the agent tool processes spawned by
the agent.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`: The process state (kind, process id, and
process start time) of the agent, existing while it runs.
 
- `.ase/agent/`- *name*- `.log`: The log of the agent run in the background.
 
- *per-user state
directory*- `/agent-condenser/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`:
The run lock and the run record of each condenser agent process, used
to recover the tasks of terminated agent processes (see- **INTERRUPTS
AND RECOVERY**in- `ase-agent(1)`).
 

## EXIT STATUS

`ase agent condenser start` exits with status `0` once the agent started
(or was already running), resp. once a foreground agent terminated
regularly, and with status `1` on any error (e.g., an invalid option, a
task lifecycle model other than `enterprise`, or a missing agent tool).
`ase agent condenser status` exits with status `1` if the agent is not
running.

## EXAMPLES

```
# start the condenser, condensing the plans of the tasks assigned to it
$ ase agent condenser start
# request the condensing of a drafted task by assigning it to the condenser
$ ase task assign FOO-42 condenser
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-rebooter(1)`,
`ase-agent-researcher(1)`, `ase-agent-implementer(1)`,
`ase-agent-qualifier(1)`, `ase-agent-integrator(1)`,
`ase-agent-editor(1)`, `ase-agent-estimator(1)`, `ase-agent-adhoc(1)`,
`ase-agent-fleet(1)`, `ase-agent-relay(1)`, `ase-task(1)`,
`ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-implementer(1) - Implementation Phase Agent

## NAME

`ase-agent-implementer(1)` - Implementation Phase Agent

## SYNOPSIS

`ase agent implementer start`
[`-n`|`--name` *name*]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-r`|`--repo` *url*]
[`-d`|`--tempdir` *dir*]
[`-m`|`--mode` `implement`|`draft`]
[`-M`|`--model` *model*]
[`--`*group*`-`*item* *value* [...]]
[`-f`|`--foreground`]

`ase agent implementer status`
[`-n`|`--name` *name*]

`ase agent implementer log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent implementer stop`
[`-n`|`--name` *name*]

## DESCRIPTION

**ase agent implementer** drives the `Implementation` phase of the
`enterprise` task lifecycle model (refused for any other model). It claims
every task in the state `PLANNED` which is unassigned or assigned to the
agent, which does not carry the tag `ase-agent-implementer` yet, and whose
`After:` predecessors (and, for an explicit epic, its group members) are
*integrated*, i.e., `INTEGRATED` and not still tagged by the integrator
with the value `active` or `failed`, as the task is implemented from its
`Source:`, which contains the change set of a predecessor only then.

For a task *stacked* onto a predecessor (i.e., with the `branch:`
changeset of the predecessor as its `Source:`), the predecessor passes
already once `IMPLEMENTED` or later (and not still tagged by the
implementer with the value `active` or `failed` while `IMPLEMENTED`).
This predecessor rule is the hook `passes` of the built-in definition.

By default, the implementer additionally waits with a task until the
`estimator` agent (see `ase-agent-estimator(1)`) is done with it, as the
latter processes the `PLANNED` tasks unclaimed and would else race
against the implementer: it claims a task only once the task carries a
valid `Effort:` key, is an explicit epic, has an empty plan, or carries
the tag `ase-agent-estimator` (i.e., its estimation failed). Without an
estimator agent, disable this with `--params effort-wait=false`, as the
unestimated tasks would else wait forever. This rule is the hook `ready`
of the built-in definition (in both modes).

In its default mode `implement`, the implementer moves each claimed task
into `IMPLEMENTING`, tags it with `ase-agent-implementer:active`, and
implements the plan from its `Source:` into its `Changeset:` via the
skill invocation `ase-task-implement --next DONE` *id* (see
**CHANGESETS**). The skill itself puts a successfully implemented task
into `IMPLEMENTED`, while a task left in `IMPLEMENTING` is put into
`STALLED`. Afterwards, the task is unassigned and tagged with
`ase-agent-implementer`.

A task already `IMPLEMENTED`, but whose changeset branch could not be
committed or pushed, is instead tagged `ase-agent-implementer:failed`
and left for a manual intervention.

In mode `draft`, the implementer instead claims only the `PLANNED` tasks
without an implementation draft attachment (type `text/x-diff; charset=utf-8; kind="draft"`) yet and without the separate tag
`ase-agent-implementer-draft` (so a drafted task is still implemented
later by an implementer in mode `implement`), drafts the implementation
from the `Source:` into this attachment only, ignoring the `Changeset:`,
via `ase-task-implement --next DONE --draft --stateless` *id* (in a
temporary worktree of the skill, leaving the working copy untouched),
and puts the task back into `PLANNED`: untagged if the draft attachment
was created or changed, as the attachment itself keeps the agent from
drafting it again, and otherwise tagged with
`ase-agent-implementer-draft`, so a failed drafting is not retried
endlessly.

On an interrupt, a task being processed is put back, untagged and with
its previous assignee, into `PLANNED`. See `ase-agent(1)` for the common
mechanics of all agents.

## CHANGESETS

In mode `implement`, the `Changeset:` of the task (or the configured
`project.task.default.changeset`) is handled as follows:

- `worktree`: The change set stays- *uncommitted*in the Git working copy
the agent is started in, which is switched to the branch of a- `branch:`- *name*source beforehand, and where a task waits while the
working copy still carries uncommitted changes (e.g., of an earlier
task not yet integrated). With- `--parallel`greater than- `1`, it is
instead implemented in the own Git worktree- `.ase/worktree/`- *id*of
each task (via the skill option- `--worktree`- *id*).
 
- `branch:`- *name*: The branch is checked out (created from the source if
missing) in the Git worktree- `.ase/worktree/`- *branch-id*of the agent,
named after the branch (with every character other than letters,
digits,- `_`, and- `-`replaced by- `-`, runs of- `-`reduced to one, and
leading and trailing- `-`and- `_`removed, and refused if its
directory already exists), unless already checked out in the working copy or
another- *ASE*worktree, which then has to be without uncommitted
changes, as they would be committed, too. It is implemented there in
place (via the skill options- `--changeset worktree --worktree`- *branch-id*, keeping
the- `Source:`of the plan, so the change set snapshot of the agent
shows the entire branch), and afterwards- *committed*onto the branch
with the message- *Kind*- `:`- *title*- `(`- *id*- `)`(where- *Kind*is the- `Kind:`frontmatter key of the task, default- `CRAFTING`) -- also a
partial result of a failed processing, as the plan tracks it --, where
the- `.ase`directory is never committed.
 - The worktree of the agent is removed again only if clean. Under
- `--repo`, the branch is pushed to the origin of the clone and, if
the push is rejected, kept as the branch- `ase-agent-rescue/`- *id*in
the preserved clone, also for a partial change set of a task put
into- `STALLED`.
 
- `attachment:`- *name*: The change set is entirely handled by the skill.
 

For a `worktree` or `branch:` changeset, the agent itself (and not the
skill) afterwards attaches the *change set snapshot* to the task: the
view-only, text-only unified diff of the change set from its fork point
from the `Source:` on (without the `.ase` directory), in the attachment
of type `text/x-diff; charset=utf-8; kind="changeset"` (see
`--attach-changeset`), replaced in place on every processing, with
`base` *commit* (plus `, partial` for a task not put into
`IMPLEMENTED`) in its `Desc:` key, so the change set can be viewed
wherever the task is visible, e.g., on the task boards during its
qualification.

For an `attachment:` changeset other than `attachment:changeset`, an
existing snapshot is removed as obsolete.

## OPTIONS

- [`-h`|`--help`]: Show usage information of`ase agent implementer`(or of the given sub-command) only.

All other options belong to the sub-commands (see **COMMANDS**).

## COMMANDS

- `ase agent implementer start`[- *options*]: Start the implementer
agent in the background, logging into- `.ase/agent/`- *name*- `.log`. The
agent is validated and initialized (e.g., its agent tool is looked up
and the task lifecycle model is checked) before the command returns
(waiting up to 30 seconds), so a failing agent is reported together
with the tail of its log. An agent of the same name already running is
left alone. The options are:
 - - [- `-n`|- `--name`- *name*]: The name of the agent (default:- `implementer`): up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit, other than the name of another
agent kind,- `relay`, or- `fleet`. It names the agent in its log
lines (also in the shared processing log attachment- `agent`), the- `Assignee:`of the tasks it processes, its temporary directory,
and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`.
 - Agents of the kind - `implementer`under different names form a
pool, competing for the same tasks and processing them in parallel
to each other.
 
- [- `-t`|- `--tool`- *tool*]: The headless agent tool processing the
tasks:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub
Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*) (default:- `$ASE_TOOL`, else- `claude`). Its command has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]: The maximum number of tasks processed
in parallel by this agent (default:- `1`), a positive integer. With
more than one, a task with a- `worktree`changeset is implemented
in its own Git worktree- `.ase/worktree/`- *id*(created by the skill
via its option- `--worktree`- *id*) instead of the working copy
itself.
 
- [- `-T`|- `--timeout`- *duration*]: The maximum duration of
processing a task (the run of the agent tool), as- *number*- *unit*(without a space) with- *unit*- `s`(seconds),- `m`(minutes), or- `h`(hours) (default:- `60m`). A processing exceeding it is aborted and
counts as failed.
 
- [- `-o`|- `--one-shot`]: Terminate the agent, instead of observing
for further tasks, once a check of the task store finds no
dispatchable task while the agent processes no task anymore --
intended for CI contexts like GitHub Actions, and not allowed in a
fleet. A task still waiting for its predecessors at that time is
left for a later run. A check of the task store failing three
times in a row then terminates the agent with a non-zero exit
status (instead of being retried endlessly after a growing delay).
 
- [- `--agent-relay`- *url*]: The agent relay to pass the live
presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for
HTTPS (default: the configured- `project.relay.url`, else none).
See- **AGENT RELAY**in- `ase-agent(1)`.
 
- [- `-r`|- `--repo`- *url*]: The Git repository to clone once into the
temporary directory of the agent, where every task without a- `worktree`value is processed in its own temporary Git worktree of
this clone, and changed branches are pushed back to its origin
(default: none, i.e., all tasks are processed in the Git working
copy the agent is started in, which is hence mandatory).
 - Requires a task store not located in the working copy (i.e., not a
local - `ase:`- *path*one). See- **WORKING COPY, CLONES, AND WORKTREES**in- `ase-agent(1)`.
 
- [- `-d`|- `--tempdir`- *dir*]: Create the temporary directory of the
agent as the sub-directory- *name*of- *dir*, where- `%p`is expanded
to the process id of the agent,- `%h`to a hash value unique to the
process, and- `%%`to- `%`(default: a fresh, private directory
below the system temporary directory). A missing- *dir*is created
private to the current user, while an existing- *dir*has to be a
directory (no symbolic link) owned by the current user.
 
- [- `-m`|- `--mode`- *mode*]: The mode of the agent (default:- `implement`):- `implement`implements the task from its- `Source:`into its- `Changeset:`via the skill- `ase-task-implement`and moves
it on into- `IMPLEMENTED`, while- `draft`just drafts its
implementation into its implementation draft attachment- `draft`and puts it back into- `PLANNED`. An invalid mode is an error.
 
- [- `--title`- *text*]: The one-line title of the operation of the
agent (at most 80 characters), shown on the task boards. Built-in:- `Implement planned tasks into their changesets`(- `implement`)
resp.- `Draft the implementation of planned tasks`(- `draft`).
 
- [- `--description`- *text*]: The brief description of the operation
of the agent (about 3-6 lines, at most 600 characters, with its
whitespace collapsed), shown on the task boards. Built-in: a
description of the operation of the mode.
 
- [- `--type`- `phase`|- `request`]: The processing type:- `phase`claims the tasks by their lifecycle status,- `request`processes
the prompt requests attached to the tasks. Generic default:- `phase`. Built-in:- `phase`.
 
- [- `--workdir`- *kind*]: The working directory of the agent tool:- `project`,- `source`,- `changeset`,- `changeset-read`, or- `target`.
Generic default:- `project`. Built-in:- `changeset`(mode- `implement`),- `source`(mode- `draft`).
 
- [- `--lifecycle`- *model*]: A supported task lifecycle model
(- `solo`,- `team`, or- `enterprise`, repeatable). Generic default:
every model containing all given states, resp. all models for- `request`and with- `--unclaimed`. Built-in:- `enterprise`.
 
- [- `--serial`]: Process the tasks serially only, ignoring- `--parallel`. Built-in: not set.
 
- [- `--unclaimed`]: Process the tasks of the incoming states
without claiming them, independent of their assignee and
predecessors, and without ever changing their status and assignee
(- `phase`only). Built-in: not set. Not allowed, as the implementer
claims the tasks into its activity state.
 
- [- `-M`|- `--model`- *model*]: The model of the agent tool: a model
for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all other
tasks), see- `ase-agent-adhoc(1)`. Generic default: the default
model of the agent tool. Built-in: none.
 
- [- `--params`- *name*- `=`- *value*]: The value of a parameter
declared by the definition (repeatable). Built-in:- `effort-wait`(- `true`or- `false`, default:- `true`), whether to wait for the
effort estimation of a task by the- `estimator`agent (see- **DESCRIPTION**).
 
- [- `--state-incoming`- *state*]: The state of the tasks to claim
(mandatory for- `phase`, repeatable with- `--unclaimed`only).
Built-in:- `PLANNED`.
 
- [- `--state-activity`- *state*]: The state of a task during its
processing. Generic default: the incoming state. Built-in:- `IMPLEMENTING`.
 
- [- `--state-success`- *state*]: The target state on success.
Generic default: the incoming state. Built-in:- `IMPLEMENTED`(mode- `implement`),- `PLANNED`(mode- `draft`).
 
- [- `--state-failure`- *state*]: The target state on failure.
Generic default: the incoming state. Built-in:- `STALLED`(mode- `implement`),- `PLANNED`(mode- `draft`).
 
- [- `--state-phase`- *state*]: The phase success state, into which
the skill itself can move a task. Generic default: the success
state. Built-in: not set (mode- `implement`),- `IMPLEMENTED`(mode- `draft`).
 
- [- `--state-passed`- *state*]: The first state from which on a task
has passed the phase for its- `After:`successors. Generic default:
the phase success state. Built-in:- `IMPLEMENTED`. The predecessor
rule is decided by the hook- `passes`of the built-in definition
instead (see- **DESCRIPTION**).
 
- [- `--state-verdict`- *VERDICT*- `=`- *state*]: The target state per
verdict (repeatable). Built-in: not set.
 
- [- `--tag-name`- *tag*]: The tag of the processed tasks
(- `ase-agent-`- *word*). Generic default:- `ase-agent-`- *name*, resp.
for- `request`always- `ase-agent-`- *request-name*. Built-in:- `ase-agent-implementer`(mode- `implement`),- `ase-agent-implementer-draft`(mode- `draft`).
 
- [- `--tag-success`- *tag*]: The tag on success:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set (mode- `implement`),- `none`(mode- `draft`).
 
- [- `--tag-failure`- *tag*]: The tag on failure:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set.
 
- [- `--tag-require`- *tag*]: A tag a claimable task has to carry
(repeatable). Built-in: not set.
 
- [- `--tag-exclude`- *tag*]: A tag a claimable task must not carry
(repeatable). Built-in: not set.
 
- [- `--tag-retry`]: A shorthand for- `--tag-failure none`, so a
failed task is claimed again after a growing delay (- `phase`only).
Built-in: not set.
 
- [- `--assign-name`- *who*]: The assignee under which the tasks are
claimed, held, recognized, and released, and which names the
agent-owned worktrees. Generic default: the agent name. Built-in:
not set.
 
- [- `--assign-required`]: Claim only the tasks explicitly assigned
to- *who*of- `--assign-name`(- `phase`only). Built-in: not set.
 
- [- `--assign-success`- *who*]: The assignee of a task after its
successful processing: a name,- `none`, or- `previous`. Generic
default:- `none`for- `phase`,- `previous`for- `request`. Built-in:
not set.
 
- [- `--assign-failure`- *who*]: The assignee of a task after its
failed processing: a name,- `none`, or- `previous`. Generic default:- `none`for- `phase`,- `previous`for- `request`. Built-in: not set.
 
- [- `--request-name`- *name*]: The name of the processed requests
(- `request`only). Generic default: the agent name. Built-in: not
set. Not allowed, as the implementer is of type- `phase`.
 
- [- `--request-done`- *word*]: The word naming a processed task in
the log lines (- `request`only). Generic default:- `processed`.
Built-in: not set. Not allowed, as the implementer is of type- `phase`.
 
- [- `--prompt-skill`- *name*]: The skill to invoke (mutually
exclusive with- `--prompt-text`and- `--prompt-file`). Built-in:- `ase-task-implement`.
 
- [- `--prompt-args`- *template*]: The argument template of the
skill. Generic default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`. Built-in:- `--next DONE{?changeset-type=branch: --changeset worktree}{?changeset-worktree: --worktree {changeset-worktree}} {id}`(mode- `implement`),- `--next DONE --draft --stateless {id}`(mode- `draft`).
 
- [- `--prompt-text`- *template*]: The prompt text template to invoke
instead of a skill. Built-in: not set.
 
- [- `--prompt-file`- *file*]: The file with the prompt text template
to invoke instead of a skill. Built-in: not set.
 
- [- `--prepare-run`- *cmd*]: A shell command run in the working
directory before the prompt (repeatable). Built-in: not set.
 
- [- `--success-criterion`- *criterion*]: A success criterion:- `exit`,- `clean`,- `changed`,- `verdict`,- `attachment`, or- `status`(repeatable). Generic default:- `exit`and- `clean`for- `phase`,- `clean`and- `changed`for- `request`. Built-in:- `status`(mode- `implement`),- `exit`and- `attachment`(mode- `draft`).
 
- [- `--success-match`- *regex*]: A regular expression an output line
has to match (repeatable). Built-in: not set.
 
- [- `--verdict-label`- *label*]: The label after which the verdict
is given in the output. Built-in: not set.
 
- [- `--verdict-allow`- *VERDICT*]: An accepted verdict (repeatable).
Generic default: the verdicts of- `--state-verdict`, else any.
Built-in: not set.
 
- [- `--attach-output`- *kind*]: The kind of the attachment storing
the output lines of the prompt. Built-in: not set.
 
- [- `--attach-expect`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*to be created or changed by the
processing for the success criterion- `attachment`. Built-in: not
set (mode- `implement`),- `text/x-diff;draft`(mode- `draft`).
 
- [- `--attach-skip`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*leaving a task unclaimed (- `phase`only).
Built-in: not set (mode- `implement`),- `text/x-diff;draft`(mode- `draft`).
 
- [- `--attach-changeset`- *kind*]: The kind of the attachment- `text/x-diff;`- *kind*storing the change set snapshot of the task
(with- `--workdir changeset`or- `target`only). Built-in:- `changeset`(mode- `implement`), not set (mode- `draft`).
 
- [- `--finish-commit`]: Commit the work onto a- `branch:`changeset
and publish it (with- `--workdir changeset`only). Built-in:
enabled (mode- `implement`), not set (mode- `draft`).
 
- [- `--finish-success`- *cmd*]: A shell command run in the working
directory after a successful processing. Built-in: not set.
 
- [- `--finish-failure`- *cmd*]: A shell command run in the working
directory after a failed processing. Built-in: not set.
 
- [- `-f`|- `--foreground`]: Run the agent in the current process
instead, logging to the terminal, until it is interrupted (e.g.,
via- `CTRL+C`) or stopped, where an agent of the same name already
running is an error.
 
- [- `-h`|- `--help`]: Show the usage information of the sub-command
only.
 
 - The options - `--`- *group*- `-`- *item*above override the built-in
definition of the mode of the agent (where an option also replaces its
mutually exclusive alternatives, e.g.,- `--prompt-text`replaces the
skill of- `--prompt-skill`), while the agent keeps the identity of its
kind (its runs, its presence, and its agent-owned worktrees are named
after- `implementer`). See- `ase-agent-adhoc(1)`for their full
semantics.
 
- `ase agent implementer status`[- `-n`|- `--name`- *name*]: Report
whether the implementer agent of the name- *name*(default:- `implementer`) is running, as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`(exit status 1).
 
- `ase agent implementer log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]
[- `--name`- *name*]: Show the log- `.ase/agent/`- *name*- `.log`of the
implementer agent of the name- *name*(default:- `implementer`) run in
the background, where an agent of a fleet logs into the log of the
fleet instead (see- `ase-agent-fleet(1)`). Without a log file, an
informational message is shown only. The options are:
 - - [- `-f`|- `--follow`]: Show the last- *n*lines (default: 10) and
then follow all newly appended lines, in the style of- `tail -f`,
until interrupted.
 
- [- `-n`|- `--lines`- *n*]: Show the last- *n*lines only (default:
all, or 10 with- `--follow`).
 
- [- `--name`- *name*]: The name of the agent (default:- `implementer`), given in its long form only, as- `-n`is the short
form of- `--lines`.
 
 
- `ase agent implementer stop`[- `-n`|- `--name`- *name*]: Stop the
implementer agent of the name- *name*(default:- `implementer`) by- `SIGTERM`, waiting (up to 5 minutes) until it aborted its running
processings and put their tasks back. An agent not running is reported
informationally only.
 

## ENVIRONMENT

- `ASE_TOOL`: The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`: The bearer token of the agent relay, if not
embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`: Set to- `true`for the agent tool processes spawned by
the agent.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`: The process state (kind, process id, and
process start time) of the agent, existing while it runs.
 
- `.ase/agent/`- *name*- `.log`: The log of the agent run in the background.
 
- `.ase/worktree/`- *id*,- `.ase/worktree/`- *branch-id*: The Git worktree
of a task with a- `worktree`changeset under- `--parallel`greater than- `1`(where a- `#`of the task id is replaced by- `_`), resp. the Git
worktree of the agent for a- `branch:`changeset, named after the
branch.
 
- *per-user state
directory*- `/agent-implementer/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`: The run lock and the run record of each implementer agent
process, used to recover the tasks of terminated agent processes (see- **INTERRUPTS AND RECOVERY**in- `ase-agent(1)`).
 

## EXIT STATUS

`ase agent implementer start` exits with status `0` once the agent
started (or was already running), resp. once a foreground agent
terminated regularly, and with status `1` on any error (e.g., an invalid
option, a task lifecycle model other than `enterprise`, or a missing
agent tool). `ase agent implementer status` exits with status `1` if the
agent is not running.

## EXAMPLES

```
# implement up to two tasks in parallel
$ ase agent implementer start -p 2
# implement the tasks in a temporary clone, pushing their branches back
$ ase agent implementer start -r git@github.com:example/project.git
# just draft the implementations of the planned tasks
$ ase agent implementer start -n drafter -m draft
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-rebooter(1)`,
`ase-agent-condenser(1)`, `ase-agent-researcher(1)`,
`ase-agent-qualifier(1)`, `ase-agent-integrator(1)`,
`ase-agent-editor(1)`, `ase-agent-estimator(1)`, `ase-agent-adhoc(1)`,
`ase-agent-fleet(1)`, `ase-agent-relay(1)`, `ase-task(1)`,
`ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-qualifier(1) - Qualification Phase Agent

## NAME

`ase-agent-qualifier(1)` - Qualification Phase Agent

## SYNOPSIS

`ase agent qualifier start`
[`-n`|`--name` *name*]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-r`|`--repo` *url*]
[`-d`|`--tempdir` *dir*]
[`-m`|`--mode` `judge`|`qualify`]
[`-M`|`--model` *model*]
[`--`*group*`-`*item* *value* [...]]
[`-f`|`--foreground`]

`ase agent qualifier status`
[`-n`|`--name` *name*]

`ase agent qualifier log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent qualifier stop`
[`-n`|`--name` *name*]

## DESCRIPTION

**ase agent qualifier** drives the `Qualification` phase of the
`enterprise` task lifecycle model (refused for any other model).

It claims every task in the state `IMPLEMENTED` which is unassigned or
assigned to the agent, which does not carry any tag `ase-agent-qualifier`
yet (i.e., without its verdict yet, so a task already carrying a
verdict of the qualifier is never picked up again), and whose `After:`
predecessors (and, for an explicit epic, its group members) are
`QUALIFIED` or later, moves it into `QUALIFYING`, and reviews its
`Changeset:` (or the configured `project.task.default.changeset`) via
the skill `ase-repo-review`:

- `worktree`: The change set staged in its working copy, i.e., the Git
worktree- `.ase/worktree/`- *id*of a parallel implementer if existing,
else the working copy of the project.
 
- `branch:`- *name*: The branch against its base, via- `ase-repo-review --branch`- *name*.
 
- `attachment:`- *name*: The change set staged in a temporary Git
worktree, detached at the- `base`commit of the attachment and removed
afterwards.
 

The verdict `APPROVE` resp. `REJECT` of the review (the last upper-case
word after the label `REVIEW VERDICT` in the output of the successfully
exited agent tool) is added as the tag `ase-agent-qualifier:approve`
resp. `ase-agent-qualifier:reject`, while a failed or skipped review,
which yields no verdict, adds the plain tag `ase-agent-qualifier`.

In the default mode `judge`, the task is then put back into `IMPLEMENTED`
for a human decision, while in mode `qualify`, it is put into `QUALIFIED`
on the verdict `APPROVE`, into `DECLINED` on the verdict `REJECT`, and
back into `IMPLEMENTED` without a verdict. In both modes, the task is
unassigned afterwards.

On an interrupt, a task being processed is put back, untagged and with
its previous assignee, into `IMPLEMENTED`. See `ase-agent(1)` for the
common mechanics of all agents.

## OPTIONS

- [`-h`|`--help`]: Show usage information of`ase agent qualifier`(or
of the given sub-command) only.

All other options belong to the sub-commands (see **COMMANDS**).

## COMMANDS

- `ase agent qualifier start`[- *options*]: Start the qualifier agent in
the background, logging into- `.ase/agent/`- *name*- `.log`. The agent is
validated and initialized (e.g., its agent tool is looked up and the
task lifecycle model is checked) before the command returns (waiting
up to 30 seconds), so a failing agent is reported together with the
tail of its log. An agent of the same name already running is left
alone. The options are:
 - - [- `-n`|- `--name`- *name*]: The name of the agent (default:- `qualifier`): up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit, other than the name of another
agent kind,- `relay`, or- `fleet`. It names the agent in its log
lines (also in the shared processing log attachment- `agent`), the- `Assignee:`of the tasks it processes, its temporary directory,
and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`.
 - Agents of the kind - `qualifier`under different names form a pool,
competing for the same tasks and processing them in parallel to
each other.
 
- [- `-t`|- `--tool`- *tool*]: The headless agent tool processing the
tasks:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub
Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*) (default:- `$ASE_TOOL`, else- `claude`). Its command has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]: The maximum number of tasks processed
in parallel by this agent (default:- `1`), a positive integer.
 
- [- `-T`|- `--timeout`- *duration*]: The maximum duration of
processing a task (the run of the agent tool), as- *number*- *unit*(without a space) with- *unit*- `s`(seconds),- `m`(minutes), or- `h`(hours) (default:- `60m`). A processing exceeding it is aborted and
counts as failed.
 
- [- `-o`|- `--one-shot`]: Terminate the agent, instead of observing
for further tasks, once a check of the task store finds no
dispatchable task while the agent processes no task anymore --
intended for CI contexts like GitHub Actions, and not allowed in a
fleet. A task still waiting for its predecessors at that time is
left for a later run. A check of the task store failing three
times in a row then terminates the agent with a non-zero exit
status (instead of being retried endlessly after a growing delay).
 
- [- `--agent-relay`- *url*]: The agent relay to pass the live
presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for
HTTPS (default: the configured- `project.relay.url`, else none).
See- **AGENT RELAY**in- `ase-agent(1)`.
 
- [- `-r`|- `--repo`- *url*]: The Git repository to clone once into the
temporary directory of the agent, where every task without a- `worktree`value is processed in its own temporary Git worktree of
this clone, and changed branches are pushed back to its origin
(default: none, i.e., all tasks are processed in the Git working
copy the agent is started in, which is hence mandatory).
 - Requires a task store not located in the working copy (i.e., not a
local - `ase:`- *path*one). See- **WORKING COPY, CLONES, AND WORKTREES**in- `ase-agent(1)`.
 
- [- `-d`|- `--tempdir`- *dir*]: Create the temporary directory of the
agent as the sub-directory- *name*of- *dir*, where- `%p`is expanded
to the process id of the agent,- `%h`to a hash value unique to the
process, and- `%%`to- `%`(default: a fresh, private directory
below the system temporary directory). A missing- *dir*is created
private to the current user, while an existing- *dir*has to be a
directory (no symbolic link) owned by the current user.
 
- [- `-m`|- `--mode`- *mode*]: The mode of the agent (default:- `judge`):- `judge`just adds the verdict tag and puts the task back
into- `IMPLEMENTED`for a human decision, while- `qualify`additionally qualifies resp. declines the task by its verdict,
i.e., puts it into- `QUALIFIED`on the verdict- `APPROVE`, into- `DECLINED`on the verdict- `REJECT`, and back into- `IMPLEMENTED`without a verdict. An invalid mode is an error.
 
- [- `--title`- *text*]: The one-line title of the operation of the
agent (at most 80 characters), shown on the task boards. Built-in:- `Review implemented tasks and tag their verdict`(- `judge`) resp.- `Review implemented tasks and decide by verdict`(- `qualify`).
 
- [- `--description`- *text*]: The brief description of the operation
of the agent (about 3-6 lines, at most 600 characters, with its
whitespace collapsed), shown on the task boards. Built-in: a
description of the operation of the mode.
 
- [- `--type`- `phase`|- `request`]: The processing type:- `phase`claims the tasks by their lifecycle status,- `request`processes
the prompt requests attached to the tasks. Generic default:- `phase`. Built-in:- `phase`.
 
- [- `--workdir`- *kind*]: The working directory of the agent tool:- `project`,- `source`,- `changeset`,- `changeset-read`, or- `target`.
Generic default:- `project`. Built-in:- `changeset-read`.
 
- [- `--lifecycle`- *model*]: A supported task lifecycle model
(- `solo`,- `team`, or- `enterprise`, repeatable). Generic default:
every model containing all given states, resp. all models for- `request`and with- `--unclaimed`. Built-in:- `enterprise`.
 
- [- `--serial`]: Process the tasks serially only, ignoring- `--parallel`. Built-in: not set.
 
- [- `--unclaimed`]: Process the tasks of the incoming states
without claiming them, independent of their assignee and
predecessors, and without ever changing their status and assignee
(- `phase`only). Built-in: not set. Not allowed, as the qualifier
claims the tasks into its activity state.
 
- [- `-M`|- `--model`- *model*]: The model of the agent tool: a model
for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all other
tasks), see- `ase-agent-adhoc(1)`. Generic default: the default
model of the agent tool. Built-in: none.
 
- [- `--params`- *name*- `=`- *value*]: The value of a parameter
declared by the definition (repeatable). Built-in: none declared.
 
- [- `--state-incoming`- *state*]: The state of the tasks to claim
(mandatory for- `phase`, repeatable with- `--unclaimed`only).
Built-in:- `IMPLEMENTED`.
 
- [- `--state-activity`- *state*]: The state of a task during its
processing. Generic default: the incoming state. Built-in:- `QUALIFYING`.
 
- [- `--state-success`- *state*]: The target state on success.
Generic default: the incoming state. Built-in:- `IMPLEMENTED`.
 
- [- `--state-failure`- *state*]: The target state on failure.
Generic default: the incoming state. Built-in:- `IMPLEMENTED`.
 
- [- `--state-phase`- *state*]: The phase success state, into which
the skill itself can move a task. Generic default: the success
state. Built-in:- `QUALIFIED`.
 
- [- `--state-passed`- *state*]: The first state from which on a task
has passed the phase for its- `After:`successors. Generic default:
the phase success state. Built-in:- `QUALIFIED`.
 
- [- `--state-verdict`- *VERDICT*- `=`- *state*]: The target state per
verdict (repeatable). Built-in: not set (mode- `judge`),- `APPROVE=QUALIFIED`and- `REJECT=DECLINED`(mode- `qualify`).
 
- [- `--tag-name`- *tag*]: The tag of the processed tasks
(- `ase-agent-`- *word*). Generic default:- `ase-agent-`- *name*, resp.
for- `request`always- `ase-agent-`- *request-name*. Built-in:- `ase-agent-qualifier`.
 
- [- `--tag-success`- *tag*]: The tag on success:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set.
 
- [- `--tag-failure`- *tag*]: The tag on failure:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set.
 
- [- `--tag-require`- *tag*]: A tag a claimable task has to carry
(repeatable). Built-in: not set.
 
- [- `--tag-exclude`- *tag*]: A tag a claimable task must not carry
(repeatable). Built-in: not set.
 
- [- `--tag-retry`]: A shorthand for- `--tag-failure none`, so a
failed task is claimed again after a growing delay (- `phase`only).
Built-in: not set.
 
- [- `--assign-name`- *who*]: The assignee under which the tasks are
claimed, held, recognized, and released, and which names the
agent-owned worktrees. Generic default: the agent name. Built-in:
not set.
 
- [- `--assign-required`]: Claim only the tasks explicitly assigned
to- *who*of- `--assign-name`(- `phase`only). Built-in: not set.
 
- [- `--assign-success`- *who*]: The assignee of a task after its
successful processing: a name,- `none`, or- `previous`. Generic
default:- `none`for- `phase`,- `previous`for- `request`. Built-in:
not set.
 
- [- `--assign-failure`- *who*]: The assignee of a task after its
failed processing: a name,- `none`, or- `previous`. Generic default:- `none`for- `phase`,- `previous`for- `request`. Built-in: not set.
 
- [- `--request-name`- *name*]: The name of the processed requests
(- `request`only). Generic default: the agent name. Built-in: not
set. Not allowed, as the qualifier is of type- `phase`.
 
- [- `--request-done`- *word*]: The word naming a processed task in
the log lines (- `request`only). Generic default:- `processed`.
Built-in: not set. Not allowed, as the qualifier is of type- `phase`.
 
- [- `--prompt-skill`- *name*]: The skill to invoke (mutually
exclusive with- `--prompt-text`and- `--prompt-file`). Built-in:- `ase-repo-review`.
 
- [- `--prompt-args`- *template*]: The argument template of the
skill. Generic default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`. Built-in:- `{?changeset-type=branch:--branch {changeset-name}}`.
 
- [- `--prompt-text`- *template*]: The prompt text template to invoke
instead of a skill. Built-in: not set.
 
- [- `--prompt-file`- *file*]: The file with the prompt text template
to invoke instead of a skill. Built-in: not set.
 
- [- `--prepare-run`- *cmd*]: A shell command run in the working
directory before the prompt (repeatable). Built-in: not set.
 
- [- `--success-criterion`- *criterion*]: A success criterion:- `exit`,- `clean`,- `changed`,- `verdict`,- `attachment`, or- `status`(repeatable). Generic default:- `exit`and- `clean`for- `phase`,- `clean`and- `changed`for- `request`. Built-in:- `exit`and- `verdict`.
 
- [- `--success-match`- *regex*]: A regular expression an output line
has to match (repeatable). Built-in: not set.
 
- [- `--verdict-label`- *label*]: The label after which the verdict
is given in the output. Built-in:- `REVIEW VERDICT`.
 
- [- `--verdict-allow`- *VERDICT*]: An accepted verdict (repeatable).
Generic default: the verdicts of- `--state-verdict`, else any.
Built-in:- `APPROVE`and- `REJECT`.
 
- [- `--attach-output`- *kind*]: The kind of the attachment storing
the output lines of the prompt. Built-in: not set.
 
- [- `--attach-expect`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*to be created or changed by the
processing for the success criterion- `attachment`. Built-in: not
set.
 
- [- `--attach-skip`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*leaving a task unclaimed (- `phase`only).
Built-in: not set.
 
- [- `--attach-changeset`- *kind*]: The kind of the attachment- `text/x-diff;`- *kind*storing the change set snapshot of the task
(with- `--workdir changeset`or- `target`only). Built-in: not set.
 
- [- `--finish-commit`]: Commit the work onto a- `branch:`changeset
and publish it (with- `--workdir changeset`only). Built-in: not
set. Not allowed, as the qualifier works with- `--workdir changeset-read`.
 
- [- `--finish-success`- *cmd*]: A shell command run in the working
directory after a successful processing. Built-in: not set.
 
- [- `--finish-failure`- *cmd*]: A shell command run in the working
directory after a failed processing. Built-in: not set.
 
- [- `-f`|- `--foreground`]: Run the agent in the current process
instead, logging to the terminal, until it is interrupted (e.g.,
via- `CTRL+C`) or stopped, where an agent of the same name already
running is an error.
 
- [- `-h`|- `--help`]: Show the usage information of the sub-command
only.
 
 - The options - `--`- *group*- `-`- *item*above override the built-in
definition of the mode of the agent (where an option also replaces its
mutually exclusive alternatives, e.g.,- `--prompt-text`replaces the
skill of- `--prompt-skill`), while the agent keeps the identity of its
kind (its runs, its presence, and its agent-owned worktrees are named
after- `qualifier`). See- `ase-agent-adhoc(1)`for their full semantics.
 
- `ase agent qualifier status`[- `-n`|- `--name`- *name*]: Report whether
the qualifier agent of the name- *name*(default:- `qualifier`) is
running, as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`(exit status 1).
 
- `ase agent qualifier log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]
[- `--name`- *name*]: Show the log- `.ase/agent/`- *name*- `.log`of the
qualifier agent of the name- *name*(default:- `qualifier`) run in the
background, where an agent of a fleet logs into the log of the fleet
instead (see- `ase-agent-fleet(1)`). Without a log file, an
informational message is shown only. The options are:
 - - [- `-f`|- `--follow`]: Show the last- *n*lines (default: 10) and
then follow all newly appended lines, in the style of- `tail -f`,
until interrupted.
 
- [- `-n`|- `--lines`- *n*]: Show the last- *n*lines only (default:
all, or 10 with- `--follow`).
 
- [- `--name`- *name*]: The name of the agent (default:- `qualifier`),
given in its long form only, as- `-n`is the short form of- `--lines`.
 
 
- `ase agent qualifier stop`[- `-n`|- `--name`- *name*]: Stop the qualifier
agent of the name- *name*(default:- `qualifier`) by- `SIGTERM`, waiting
(up to 5 minutes) until it aborted its running processings and put
their tasks back. An agent not running is reported informationally
only.
 

## ENVIRONMENT

- `ASE_TOOL`: The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`: The bearer token of the agent relay, if not
embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`: Set to- `true`for the agent tool processes spawned by
the agent.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`: The process state (kind, process id, and
process start time) of the agent, existing while it runs.
 
- `.ase/agent/`- *name*- `.log`: The log of the agent run in the background.
 
- *per-user state
directory*- `/agent-qualifier/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`: The run lock and the run record of each qualifier agent
process, used to recover the tasks of terminated agent processes (see- **INTERRUPTS AND RECOVERY**in- `ase-agent(1)`).
 

## EXIT STATUS

`ase agent qualifier start` exits with status `0` once the agent started
(or was already running), resp. once a foreground agent terminated
regularly, and with status `1` on any error (e.g., an invalid option, a
task lifecycle model other than `enterprise`, or a missing agent tool).
`ase agent qualifier status` exits with status `1` if the agent is not
running.

## EXAMPLES

```
# review the implemented tasks, leaving the decision to a human
$ ase agent qualifier start -m judge -p 2
# review the implemented tasks and decide on them by the verdict
$ ase agent qualifier start -m qualify
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-rebooter(1)`,
`ase-agent-condenser(1)`, `ase-agent-researcher(1)`,
`ase-agent-implementer(1)`, `ase-agent-integrator(1)`,
`ase-agent-editor(1)`, `ase-agent-estimator(1)`, `ase-agent-adhoc(1)`,
`ase-agent-fleet(1)`, `ase-agent-relay(1)`, `ase-task(1)`,
`ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-integrator(1) - Integration Phase Agent

## NAME

`ase-agent-integrator(1)` - Integration Phase Agent

## SYNOPSIS

`ase agent integrator start`
[`-n`|`--name` *name*]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-r`|`--repo` *url*]
[`-d`|`--tempdir` *dir*]
[`-m`|`--mode` `merge`|`rebase`|`squash`]
[`-M`|`--model` *model*]
[`--`*group*`-`*item* *value* [...]]
[`-f`|`--foreground`]

`ase agent integrator status`
[`-n`|`--name` *name*]

`ase agent integrator log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent integrator stop`
[`-n`|`--name` *name*]

## DESCRIPTION

**ase agent integrator** drives the `Integration` phase of the
`enterprise` task lifecycle model (refused for any other model). It
claims every task in the state `QUALIFIED` which is unassigned or
assigned to the agent, which does not carry the tag
`ase-agent-integrator` yet, and whose `After:` predecessors (and, for an
explicit epic, its group members) are `INTEGRATED` (or, as a cancelled
member of an epic, dropped from it), moves it into `INTEGRATING`, and
integrates its `Changeset:` into its `Target:` (or the configured
defaults `project.task.default.changeset` and
`project.task.default.target`) via the skill invocation
`ase-task-integrate --next DONE --mode` *mode* *id* (with the skill
option `--worktree` *id* for a `worktree` changeset in the Git worktree
of a parallel implementer), where the merge commits the change set into
the target.

A `branch:`*name* target not checked out in any Git worktree yet is
checked out in a temporary Git worktree for the merge and removed
afterwards. The agent tool is granted access to the Git worktrees of the
changeset and the target. Under `--repo`, a moved `branch:` target is
pushed back to the origin of the clone afterwards, and a merged
`branch:` changeset is deleted from the origin (where a failure of the
latter is just warned about).

The skill itself puts an integrated task into `INTEGRATED`, where the
processing counts as successful only if the agent tool exited
successfully and its output carries the status line of the skill
matching `status:\W*plan integrated\b`, while a task left in
`INTEGRATING` is put into `DEFERRED`, especially on unresolvable merge
conflicts. Afterwards, the task is unassigned and tagged with
`ase-agent-integrator`.

A task already `INTEGRATED`, but whose target branch could not be pushed,
is instead tagged `ase-agent-integrator:failed` and left for a manual
intervention, with the merge kept as the branch `ase-agent-rescue/`*id*
in the preserved clone.

For a `worktree` or `branch:` changeset of an `INTEGRATED` task whose
integration changed the content of the target branch, the agent itself
(and not the skill) afterwards replaces the *change set snapshot* of the
task -- the attachment of type `text/x-diff; charset=utf-8; kind="changeset"` (see `--attach-changeset`) -- in place by the
view-only, text-only unified diff which finally landed on the target,
with `base` *commit*`, merged` *commit* in its `Desc:` key.

Any uncommitted change in the worktree of the target branch lets the
merge of every task with a `branch:` changeset fail, and is committed
together with the change set of a task with a `worktree` changeset, so
keep that worktree clean while the integrator runs. As merges into the
same target branch cannot run concurrently, the tasks are processed
serially only.

On an interrupt, a task being processed is put back, untagged and with
its previous assignee, into `QUALIFIED`. See `ase-agent(1)` for the
common mechanics of all agents.

## OPTIONS

- [`-h`|`--help`]: Show usage information of`ase agent integrator`(or of the given sub-command) only.

All other options belong to the sub-commands (see **COMMANDS**).

## COMMANDS

- `ase agent integrator start`[- *options*]: Start the integrator agent
in the background, logging into- `.ase/agent/`- *name*- `.log`. The agent
is validated and initialized (e.g., its agent tool is looked up and
the task lifecycle model is checked) before the command returns
(waiting up to 30 seconds), so a failing agent is reported together
with the tail of its log. An agent of the same name already running is
left alone. The options are:
 - - [- `-n`|- `--name`- *name*]: The name of the agent (default:- `integrator`): up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit, other than the name of another
agent kind,- `relay`, or- `fleet`. It names the agent in its log
lines (also in the shared processing log attachment- `agent`), the- `Assignee:`of the tasks it processes, its temporary directory,
and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`.
 - Agents of the kind - `integrator`under different names form a
pool, competing for the same tasks, but processing the tasks
serially only, so a single integrator suffices (and a fleet
refuses more than one).
 
- [- `-t`|- `--tool`- *tool*]: The headless agent tool processing the
tasks:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub
Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*) (default:- `$ASE_TOOL`, else- `claude`). Its command has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]: Ignored, as merges into the same
target branch cannot run concurrently, so the tasks are processed
serially only: an explicitly given- *num*greater than- `1`is
clamped to- `1`with a warning.
 
- [- `-T`|- `--timeout`- *duration*]: The maximum duration of
processing a task (the run of the agent tool), as- *number*- *unit*(without a space) with- *unit*- `s`(seconds),- `m`(minutes), or- `h`(hours) (default:- `60m`). A processing exceeding it is aborted and
counts as failed.
 
- [- `-o`|- `--one-shot`]: Terminate the agent, instead of observing
for further tasks, once a check of the task store finds no
dispatchable task while the agent processes no task anymore --
intended for CI contexts like GitHub Actions, and not allowed in a
fleet. A task still waiting for its predecessors at that time is
left for a later run. A check of the task store failing three
times in a row then terminates the agent with a non-zero exit
status (instead of being retried endlessly after a growing delay).
 
- [- `--agent-relay`- *url*]: The agent relay to pass the live
presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for
HTTPS (default: the configured- `project.relay.url`, else none).
See- **AGENT RELAY**in- `ase-agent(1)`.
 
- [- `-r`|- `--repo`- *url*]: The Git repository to clone once into the
temporary directory of the agent, where every task without a- `worktree`value is processed in its own temporary Git worktree of
this clone, and changed branches are pushed back to its origin
(default: none, i.e., all tasks are processed in the Git working
copy the agent is started in, which is hence mandatory).
 - Requires a task store not located in the working copy (i.e., not a
local - `ase:`- *path*one). See- **WORKING COPY, CLONES, AND WORKTREES**in- `ase-agent(1)`.
 
- [- `-d`|- `--tempdir`- *dir*]: Create the temporary directory of the
agent as the sub-directory- *name*of- *dir*, where- `%p`is expanded
to the process id of the agent,- `%h`to a hash value unique to the
process, and- `%%`to- `%`(default: a fresh, private directory
below the system temporary directory). A missing- *dir*is created
private to the current user, while an existing- *dir*has to be a
directory (no symbolic link) owned by the current user.
 
- [- `-m`|- `--mode`- *mode*]: The Git operation integrating the
changeset into the target, passed through to the skill- `ase-task-integrate`(default:- `squash`):- `merge`creates a merge
commit,- `rebase`rebases the changeset onto the target and
fast-forwards the target, and- `squash`combines the changeset into
one new commit on the target. An invalid mode is an error.
 
- [- `--title`- *text*]: The one-line title of the operation of the
agent (at most 80 characters), shown on the task boards. Built-in:- `Integrate qualified tasks by`- *how*, with- *how*being- `a merge commit`(- `merge`),- `a rebase with fast-forward`(- `rebase`), resp.- `a squash commit`(- `squash`).
 
- [- `--description`- *text*]: The brief description of the operation
of the agent (about 3-6 lines, at most 600 characters, with its
whitespace collapsed), shown on the task boards. Built-in: a
description of the operation of the mode.
 
- [- `--type`- `phase`|- `request`]: The processing type:- `phase`claims the tasks by their lifecycle status,- `request`processes
the prompt requests attached to the tasks. Generic default:- `phase`. Built-in:- `phase`.
 
- [- `--workdir`- *kind*]: The working directory of the agent tool:- `project`,- `source`,- `changeset`,- `changeset-read`, or- `target`.
Generic default:- `project`. Built-in:- `target`.
 
- [- `--lifecycle`- *model*]: A supported task lifecycle model
(- `solo`,- `team`, or- `enterprise`, repeatable). Generic default:
every model containing all given states, resp. all models for- `request`and with- `--unclaimed`. Built-in:- `enterprise`.
 
- [- `--serial`]: Process the tasks serially only, ignoring- `--parallel`. Built-in: enabled.
 
- [- `--unclaimed`]: Process the tasks of the incoming states
without claiming them, independent of their assignee and
predecessors, and without ever changing their status and assignee
(- `phase`only). Built-in: not set. Not allowed, as the integrator
claims the tasks into its activity state.
 
- [- `-M`|- `--model`- *model*]: The model of the agent tool: a model
for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all other
tasks), see- `ase-agent-adhoc(1)`. Generic default: the default
model of the agent tool. Built-in: none.
 
- [- `--params`- *name*- `=`- *value*]: The value of a parameter
declared by the definition (repeatable). Built-in: none declared.
 
- [- `--state-incoming`- *state*]: The state of the tasks to claim
(mandatory for- `phase`, repeatable with- `--unclaimed`only).
Built-in:- `QUALIFIED`.
 
- [- `--state-activity`- *state*]: The state of a task during its
processing. Generic default: the incoming state. Built-in:- `INTEGRATING`.
 
- [- `--state-success`- *state*]: The target state on success.
Generic default: the incoming state. Built-in:- `INTEGRATED`.
 
- [- `--state-failure`- *state*]: The target state on failure.
Generic default: the incoming state. Built-in:- `DEFERRED`.
 
- [- `--state-phase`- *state*]: The phase success state, into which
the skill itself can move a task. Generic default: the success
state. Built-in: not set.
 
- [- `--state-passed`- *state*]: The first state from which on a task
has passed the phase for its- `After:`successors. Generic default:
the phase success state. Built-in:- `INTEGRATED`.
 
- [- `--state-verdict`- *VERDICT*- `=`- *state*]: The target state per
verdict (repeatable). Built-in: not set.
 
- [- `--tag-name`- *tag*]: The tag of the processed tasks
(- `ase-agent-`- *word*). Generic default:- `ase-agent-`- *name*, resp.
for- `request`always- `ase-agent-`- *request-name*. Built-in:- `ase-agent-integrator`.
 
- [- `--tag-success`- *tag*]: The tag on success:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set.
 
- [- `--tag-failure`- *tag*]: The tag on failure:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set.
 
- [- `--tag-require`- *tag*]: A tag a claimable task has to carry
(repeatable). Built-in: not set.
 
- [- `--tag-exclude`- *tag*]: A tag a claimable task must not carry
(repeatable). Built-in: not set.
 
- [- `--tag-retry`]: A shorthand for- `--tag-failure none`, so a
failed task is claimed again after a growing delay (- `phase`only).
Built-in: not set.
 
- [- `--assign-name`- *who*]: The assignee under which the tasks are
claimed, held, recognized, and released, and which names the
agent-owned worktrees. Generic default: the agent name. Built-in:
not set.
 
- [- `--assign-required`]: Claim only the tasks explicitly assigned
to- *who*of- `--assign-name`(- `phase`only). Built-in: not set.
 
- [- `--assign-success`- *who*]: The assignee of a task after its
successful processing: a name,- `none`, or- `previous`. Generic
default:- `none`for- `phase`,- `previous`for- `request`. Built-in:
not set.
 
- [- `--assign-failure`- *who*]: The assignee of a task after its
failed processing: a name,- `none`, or- `previous`. Generic default:- `none`for- `phase`,- `previous`for- `request`. Built-in: not set.
 
- [- `--request-name`- *name*]: The name of the processed requests
(- `request`only). Generic default: the agent name. Built-in: not
set. Not allowed, as the integrator is of type- `phase`.
 
- [- `--request-done`- *word*]: The word naming a processed task in
the log lines (- `request`only). Generic default:- `processed`.
Built-in: not set. Not allowed, as the integrator is of type- `phase`.
 
- [- `--prompt-skill`- *name*]: The skill to invoke (mutually
exclusive with- `--prompt-text`and- `--prompt-file`). Built-in:- `ase-task-integrate`.
 
- [- `--prompt-args`- *template*]: The argument template of the
skill. Generic default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`. Built-in:- `--next DONE --mode merge{?changeset-worktree: --worktree {changeset-worktree}} {id}`(mode- `merge`),- `--next DONE --mode rebase{?changeset-worktree: --worktree {changeset-worktree}} {id}`(mode- `rebase`),- `--next DONE --mode squash{?changeset-worktree: --worktree {changeset-worktree}} {id}`(mode- `squash`).
 
- [- `--prompt-text`- *template*]: The prompt text template to invoke
instead of a skill. Built-in: not set.
 
- [- `--prompt-file`- *file*]: The file with the prompt text template
to invoke instead of a skill. Built-in: not set.
 
- [- `--prepare-run`- *cmd*]: A shell command run in the working
directory before the prompt (repeatable). Built-in: not set.
 
- [- `--success-criterion`- *criterion*]: A success criterion:- `exit`,- `clean`,- `changed`,- `verdict`,- `attachment`, or- `status`(repeatable). Generic default:- `exit`and- `clean`for- `phase`,- `clean`and- `changed`for- `request`. Built-in:- `exit`.
 
- [- `--success-match`- *regex*]: A regular expression an output line
has to match (repeatable). Built-in:- `status:\W*plan integrated\b`.
 
- [- `--verdict-label`- *label*]: The label after which the verdict
is given in the output. Built-in: not set.
 
- [- `--verdict-allow`- *VERDICT*]: An accepted verdict (repeatable).
Generic default: the verdicts of- `--state-verdict`, else any.
Built-in: not set.
 
- [- `--attach-output`- *kind*]: The kind of the attachment storing
the output lines of the prompt. Built-in: not set.
 
- [- `--attach-expect`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*to be created or changed by the
processing for the success criterion- `attachment`. Built-in: not
set.
 
- [- `--attach-skip`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*leaving a task unclaimed (- `phase`only).
Built-in: not set.
 
- [- `--attach-changeset`- *kind*]: The kind of the attachment- `text/x-diff;`- *kind*storing the change set snapshot of the task
(with- `--workdir changeset`or- `target`only). Built-in:- `changeset`.
 
- [- `--finish-commit`]: Commit the work onto a- `branch:`changeset
and publish it (with- `--workdir changeset`only). Built-in: not
set. Not allowed, as the integrator works with- `--workdir target`.
 
- [- `--finish-success`- *cmd*]: A shell command run in the working
directory after a successful processing. Built-in: not set.
 
- [- `--finish-failure`- *cmd*]: A shell command run in the working
directory after a failed processing. Built-in: not set.
 
- [- `-f`|- `--foreground`]: Run the agent in the current process
instead, logging to the terminal, until it is interrupted (e.g.,
via- `CTRL+C`) or stopped, where an agent of the same name already
running is an error.
 
- [- `-h`|- `--help`]: Show the usage information of the sub-command
only.
 
 - The options - `--`- *group*- `-`- *item*above override the built-in
definition of the mode of the agent (where an option also replaces its
mutually exclusive alternatives, e.g.,- `--prompt-text`replaces the
skill of- `--prompt-skill`), while the agent keeps the identity of its
kind (its runs, its presence, and its agent-owned worktrees are named
after- `integrator`). See- `ase-agent-adhoc(1)`for their full
semantics.
 
- `ase agent integrator status`[- `-n`|- `--name`- *name*]: Report whether
the integrator agent of the name- *name*(default:- `integrator`) is
running, as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`(exit status 1).
 
- `ase agent integrator log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]
[- `--name`- *name*]: Show the log- `.ase/agent/`- *name*- `.log`of the
integrator agent of the name- *name*(default:- `integrator`) run in the
background, where an agent of a fleet logs into the log of the fleet
instead (see- `ase-agent-fleet(1)`). Without a log file, an
informational message is shown only. The options are:
 - - [- `-f`|- `--follow`]: Show the last- *n*lines (default: 10) and
then follow all newly appended lines, in the style of- `tail -f`,
until interrupted.
 
- [- `-n`|- `--lines`- *n*]: Show the last- *n*lines only (default:
all, or 10 with- `--follow`).
 
- [- `--name`- *name*]: The name of the agent (default:- `integrator`), given in its long form only, as- `-n`is the short
form of- `--lines`.
 
 
- `ase agent integrator stop`[- `-n`|- `--name`- *name*]: Stop the
integrator agent of the name- *name*(default:- `integrator`) by- `SIGTERM`, waiting (up to 5 minutes) until it aborted its running
processings and put their tasks back. An agent not running is reported
informationally only.
 

## ENVIRONMENT

- `ASE_TOOL`: The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`: The bearer token of the agent relay, if not
embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`: Set to- `true`for the agent tool processes spawned by
the agent.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`: The process state (kind, process id, and
process start time) of the agent, existing while it runs.
 
- `.ase/agent/`- *name*- `.log`: The log of the agent run in the background.
 
- *per-user state
directory*- `/agent-integrator/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`: The run lock and the run record of each integrator agent
process, used to recover the tasks of terminated agent processes (see- **INTERRUPTS AND RECOVERY**in- `ase-agent(1)`).
 

## EXIT STATUS

`ase agent integrator start` exits with status `0` once the agent
started (or was already running), resp. once a foreground agent
terminated regularly, and with status `1` on any error (e.g., an invalid
option, a task lifecycle model other than `enterprise`, or a missing
agent tool). `ase agent integrator status` exits with status `1` if the
agent is not running.

## EXAMPLES

```
# integrate the qualified tasks by squash commits
$ ase agent integrator start
# integrate the qualified tasks by merge commits in a temporary clone
$ ase agent integrator start -m merge -r git@github.com:example/project.git
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-rebooter(1)`,
`ase-agent-condenser(1)`, `ase-agent-researcher(1)`,
`ase-agent-implementer(1)`, `ase-agent-qualifier(1)`,
`ase-agent-editor(1)`, `ase-agent-estimator(1)`, `ase-agent-adhoc(1)`,
`ase-agent-fleet(1)`, `ase-agent-relay(1)`, `ase-task(1)`,
`ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-editor(1) - Edit Request Agent

## NAME

`ase-agent-editor(1)` - Edit Request Agent

## SYNOPSIS

`ase agent editor start`
[`-n`|`--name` *name*]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-r`|`--repo` *url*]
[`-d`|`--tempdir` *dir*]
[`-M`|`--model` *model*]
[`--`*group*`-`*item* *value* [...]]
[`-f`|`--foreground`]

`ase agent editor status`
[`-n`|`--name` *name*]

`ase agent editor log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent editor stop`
[`-n`|`--name` *name*]

## DESCRIPTION

**ase agent editor** processes the edit requests of tasks. Unlike the
phase agents, it is bound to no phase and supports all task lifecycle
models (`solo`, `team`, and `enterprise`).

An edit request is added by key `E` of the task board (see
`ase-task-board(1)`) or by `ase task request` *id* `edit` *prompt* (see
`ase-task-request(1)`) as an attachment of type
`text/plain; charset=utf-8; kind="edit"` with the prompt as its data,
mirrored in the tag `ase-agent-editor:pending` of the task (aggregated
as `:active` while one is processed and as `:failed` once one failed).

The editor observes the tasks with this tag, except those in an activity
status of the task lifecycle model, where the task is actively worked on
(`team`: `IMPLEMENTING`; `enterprise`: `PLANNING`, `IMPLEMENTING`,
`QUALIFYING`, and `INTEGRATING`), and those still held by a phase agent
(tagged `ase-agent-`*kind*`:active`), which wait until their status
changes resp. their tag is released.

It claims the oldest request of such a task in a single conditional
save by turning it into `kind="edit-running"` and setting the
`Assignee:` of the task to its agent name (remembering the previous
assignee in the request), which keeps all other agents off the task
during the edit without touching its status. The request of a finished
task, resp. a request with an empty prompt, is turned into
`kind="edit-failed"` instead.

It then edits the plan via the skill invocation `ase-task-edit --plan REFINE --next DONE` *id*`:` *prompt* (with the whitespace of the prompt
collapsed) in the working copy, grounded in the `Source:` of the task,
where a `branch:`*name* source is checked out in the temporary, detached
Git worktree `.ase/worktree/.temp/`*id*`-editor`, passed to the skill
via its internal option `--int-source-dir`, and removed afterwards. The
skill saves the plan only if it was not changed meanwhile (e.g., by a
further edit request).

Finally, in a single save, the editor restores the previous assignee of
the task (unless reassigned meanwhile) and removes the request if the
plan content actually changed (its frontmatter, except `Status:`,
`Tags:`, and `Assignee:`, which other agents change, and its body) and
the skill reported no error, or else turns it back into `kind="edit"` on
an interrupt, resp. into `kind="edit-failed"` (with the reason, e.g.,
`skill reported an error`, `agent tool failed`, or `plan not changed`)
otherwise.

A finalization failing even after its retries is retried on every
further dispatching.

On its start, the editor marks the requests left in
`kind="edit-running"` by any terminated editor process of the same user
on the same host (whose run lock is gone or stale and whose process is
no longer alive) as failed (`kind="edit-failed"`), and restores the
previous assignees of their tasks. See `ase-agent(1)` for the common
mechanics of all agents.

## OPTIONS

- [`-h`|`--help`]: Show usage information of`ase agent editor`(or of
the given sub-command) only.

All other options belong to the sub-commands (see **COMMANDS**).

## COMMANDS

- `ase agent editor start`[- *options*]: Start the editor agent in the
background, logging into- `.ase/agent/`- *name*- `.log`. The agent is
validated and initialized (e.g., its agent tool is looked up and the
task lifecycle model is checked) before the command returns (waiting
up to 30 seconds), so a failing agent is reported together with the
tail of its log. An agent of the same name already running is left
alone. The options are:
 - - [- `-n`|- `--name`- *name*]: The name of the agent (default:- `editor`): up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit, other than the name of another
agent kind,- `relay`, or- `fleet`.
 - It names the agent in its log lines (also in the shared processing
log attachment - `agent`), the- `Assignee:`of the tasks it processes, its
temporary directory, and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`. Agents of the kind- `editor`under
different names form a pool, competing for the same tasks and
processing them in parallel to each other.
 
- [- `-t`|- `--tool`- *tool*]: The headless agent tool processing the
tasks:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub
Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*) (default:- `$ASE_TOOL`, else- `claude`). Its command has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]: The maximum number of tasks processed
in parallel by this agent (default:- `1`), a positive integer.
 
- [- `-T`|- `--timeout`- *duration*]: The maximum duration of
processing a task (the run of the agent tool), as- *number*- *unit*(without a space) with- *unit*- `s`(seconds),- `m`(minutes), or- `h`(hours) (default:- `60m`). A processing exceeding it is aborted and
counts as failed.
 
- [- `-o`|- `--one-shot`]: Terminate the agent, instead of observing
for further tasks, once a check of the task store finds no
dispatchable task while the agent processes no task anymore --
intended for CI contexts like GitHub Actions, and not allowed in a
fleet. A task still waiting for its predecessors at that time is
left for a later run. A check of the task store failing three
times in a row then terminates the agent with a non-zero exit
status (instead of being retried endlessly after a growing delay).
 
- [- `--agent-relay`- *url*]: The agent relay to pass the live
presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for
HTTPS (default: the configured- `project.relay.url`, else none).
See- **AGENT RELAY**in- `ase-agent(1)`.
 
- [- `-r`|- `--repo`- *url*]: The Git repository to clone once into the
temporary directory of the agent, where every task without a- `worktree`value is processed in its own temporary Git worktree of
this clone, and changed branches are pushed back to its origin
(default: none, i.e., all tasks are processed in the Git working
copy the agent is started in, which is hence mandatory).
 - Requires a task store not located in the working copy (i.e., not a
local - `ase:`- *path*one). See- **WORKING COPY, CLONES, AND WORKTREES**in- `ase-agent(1)`.
 
- [- `-d`|- `--tempdir`- *dir*]: Create the temporary directory of the
agent as the sub-directory- *name*of- *dir*, where- `%p`is expanded
to the process id of the agent,- `%h`to a hash value unique to the
process, and- `%%`to- `%`(default: a fresh, private directory
below the system temporary directory). A missing- *dir*is created
private to the current user, while an existing- *dir*has to be a
directory (no symbolic link) owned by the current user.
 
- [- `--title`- *text*]: The one-line title of the operation of the
agent (at most 80 characters), shown on the task boards. Built-in:- `Edit task plans by the prompts of edit requests`.
 
- [- `--description`- *text*]: The brief description of the operation
of the agent (about 3-6 lines, at most 600 characters, with its
whitespace collapsed), shown on the task boards. Built-in: a
description of the operation of the editor.
 
- [- `--type`- `phase`|- `request`]: The processing type:- `phase`claims the tasks by their lifecycle status,- `request`processes
the prompt requests attached to the tasks. Generic default:- `phase`. Built-in:- `request`.
 
- [- `--workdir`- *kind*]: The working directory of the agent tool:- `project`,- `source`,- `changeset`,- `changeset-read`, or- `target`.
Generic default:- `project`. Built-in:- `source`.
 
- [- `--lifecycle`- *model*]: A supported task lifecycle model
(- `solo`,- `team`, or- `enterprise`, repeatable). Generic default:
every model containing all given states, resp. all models for- `request`and with- `--unclaimed`. Built-in: not set. Hence, all
task lifecycle models are supported.
 
- [- `--serial`]: Process the tasks serially only, ignoring- `--parallel`. Built-in: not set.
 
- [- `--unclaimed`]: Process the tasks of the incoming states
without claiming them, independent of their assignee and
predecessors, and without ever changing their status and assignee
(- `phase`only). Built-in: not set. Not allowed, as the editor is
of type- `request`.
 
- [- `-M`|- `--model`- *model*]: The model of the agent tool: a model
for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all other
tasks), see- `ase-agent-adhoc(1)`. Generic default: the default
model of the agent tool. Built-in: none.
 
- [- `--params`- *name*- `=`- *value*]: The value of a parameter
declared by the definition (repeatable). Built-in: none declared.
 
- [- `--state-incoming`- *state*]: The state of the tasks to claim
(mandatory for- `phase`, repeatable with- `--unclaimed`only).
Built-in: not set. Not allowed, as the editor is of type- `request`.
 
- [- `--state-activity`- *state*]: The state of a task during its
processing. Generic default: the incoming state. Built-in: not
set. Not allowed, as the editor is of type- `request`.
 
- [- `--state-success`- *state*]: The target state on success.
Generic default: the incoming state. Built-in: not set. Not
allowed, as the editor is of type- `request`.
 
- [- `--state-failure`- *state*]: The target state on failure.
Generic default: the incoming state. Built-in: not set. Not
allowed, as the editor is of type- `request`.
 
- [- `--state-phase`- *state*]: The phase success state, into which
the skill itself can move a task. Generic default: the success
state. Built-in: not set. Not allowed, as the editor is of type- `request`.
 
- [- `--state-passed`- *state*]: The first state from which on a task
has passed the phase for its- `After:`successors. Generic default:
the phase success state. Built-in: not set. Not allowed, as the
editor is of type- `request`.
 
- [- `--state-verdict`- *VERDICT*- `=`- *state*]: The target state per
verdict (repeatable). Built-in: not set. Not allowed, as the
editor is of type- `request`.
 
- [- `--tag-name`- *tag*]: The tag of the processed tasks
(- `ase-agent-`- *word*). Generic default:- `ase-agent-`- *name*, resp.
for- `request`always- `ase-agent-`- *request-name*. Built-in:- `ase-agent-editor`(derived from the request name- `edit`).
 
- [- `--tag-success`- *tag*]: The tag on success:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set. Not allowed, as the editor is of
type- `request`.
 
- [- `--tag-failure`- *tag*]: The tag on failure:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set. Not allowed, as the editor is of
type- `request`.
 
- [- `--tag-require`- *tag*]: A tag a claimable task has to carry
(repeatable). Built-in: not set.
 
- [- `--tag-exclude`- *tag*]: A tag a claimable task must not carry
(repeatable). Built-in: not set.
 
- [- `--tag-retry`]: A shorthand for- `--tag-failure none`, so a
failed task is claimed again after a growing delay (- `phase`only).
Built-in: not set. Not allowed, as the editor is of type- `request`.
 
- [- `--assign-name`- *who*]: The assignee under which the tasks are
claimed, held, recognized, and released, and which names the
agent-owned worktrees. Generic default: the agent name. Built-in:
not set.
 
- [- `--assign-required`]: Claim only the tasks explicitly assigned
to- *who*of- `--assign-name`(- `phase`only). Built-in: not set. Not
allowed, as the editor is of type- `request`.
 
- [- `--assign-success`- *who*]: The assignee of a task after its
successful processing: a name,- `none`, or- `previous`. Generic
default:- `none`for- `phase`,- `previous`for- `request`. Built-in:
not set.
 
- [- `--assign-failure`- *who*]: The assignee of a task after its
failed processing: a name,- `none`, or- `previous`. Generic default:- `none`for- `phase`,- `previous`for- `request`. Built-in: not set.
 
- [- `--request-name`- *name*]: The name of the processed requests
(- `request`only). Generic default: the agent name. Built-in:- `edit`.
 
- [- `--request-done`- *word*]: The word naming a processed task in
the log lines (- `request`only). Generic default:- `processed`.
Built-in:- `edited`.
 
- [- `--prompt-skill`- *name*]: The skill to invoke (mutually
exclusive with- `--prompt-text`and- `--prompt-file`). Built-in:- `ase-task-edit`.
 
- [- `--prompt-args`- *template*]: The argument template of the
skill. Generic default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`. Built-in:- `--plan REFINE --next DONE{?source-dir: --int-source-dir "{source-dir}"} {id}: {prompt}`.
 
- [- `--prompt-text`- *template*]: The prompt text template to invoke
instead of a skill. Built-in: not set.
 
- [- `--prompt-file`- *file*]: The file with the prompt text template
to invoke instead of a skill. Built-in: not set.
 
- [- `--prepare-run`- *cmd*]: A shell command run in the working
directory before the prompt (repeatable). Built-in: not set.
 
- [- `--success-criterion`- *criterion*]: A success criterion:- `exit`,- `clean`,- `changed`,- `verdict`,- `attachment`, or- `status`(repeatable). Generic default:- `exit`and- `clean`for- `phase`,- `clean`and- `changed`for- `request`. Built-in:- `clean`and- `changed`.
 
- [- `--success-match`- *regex*]: A regular expression an output line
has to match (repeatable). Built-in: not set.
 
- [- `--verdict-label`- *label*]: The label after which the verdict
is given in the output. Built-in: not set.
 
- [- `--verdict-allow`- *VERDICT*]: An accepted verdict (repeatable).
Generic default: the verdicts of- `--state-verdict`, else any.
Built-in: not set.
 
- [- `--attach-output`- *kind*]: The kind of the attachment storing
the output lines of the prompt. Built-in: not set.
 
- [- `--attach-expect`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*to be created or changed by the
processing for the success criterion- `attachment`. Built-in: not
set.
 
- [- `--attach-skip`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*leaving a task unclaimed (- `phase`only).
Built-in: not set. Not allowed, as the editor is of type- `request`.
 
- [- `--attach-changeset`- *kind*]: The kind of the attachment- `text/x-diff;`- *kind*storing the change set snapshot of the task
(with- `--workdir changeset`or- `target`only). Built-in: not set.
 
- [- `--finish-commit`]: Commit the work onto a- `branch:`changeset
and publish it (with- `--workdir changeset`only). Built-in: not
set. Not allowed, as the editor works with- `--workdir source`.
 
- [- `--finish-success`- *cmd*]: A shell command run in the working
directory after a successful processing. Built-in: not set.
 
- [- `--finish-failure`- *cmd*]: A shell command run in the working
directory after a failed processing. Built-in: not set.
 
- [- `-f`|- `--foreground`]: Run the agent in the current process
instead, logging to the terminal, until it is interrupted (e.g.,
via- `CTRL+C`) or stopped, where an agent of the same name already
running is an error.
 
- [- `-h`|- `--help`]: Show the usage information of the sub-command
only.
 
 - The options - `--`- *group*- `-`- *item*above override the built-in
definition of the mode of the agent (where an option also replaces its
mutually exclusive alternatives, e.g.,- `--prompt-text`replaces the
skill of- `--prompt-skill`), while the agent keeps the identity of its
kind (its runs, its presence, and its agent-owned worktrees are named
after- `editor`). See- `ase-agent-adhoc(1)`for their full semantics.
 
- `ase agent editor status`[- `-n`|- `--name`- *name*]: Report whether the
editor agent of the name- *name*(default:- `editor`) is running, as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`(exit status 1).
 
- `ase agent editor log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]
[- `--name`- *name*]: Show the log- `.ase/agent/`- *name*- `.log`of the
editor agent of the name- *name*(default:- `editor`) run in the
background, where an agent of a fleet logs into the log of the fleet
instead (see- `ase-agent-fleet(1)`). Without a log file, an
informational message is shown only. The options are:
 - - [- `-f`|- `--follow`]: Show the last- *n*lines (default: 10) and
then follow all newly appended lines, in the style of- `tail -f`,
until interrupted.
 
- [- `-n`|- `--lines`- *n*]: Show the last- *n*lines only (default:
all, or 10 with- `--follow`).
 
- [- `--name`- *name*]: The name of the agent (default:- `editor`),
given in its long form only, as- `-n`is the short form of- `--lines`.
 
 
- `ase agent editor stop`[- `-n`|- `--name`- *name*]: Stop the editor
agent of the name- *name*(default:- `editor`) by- `SIGTERM`, waiting (up
to 5 minutes) until it aborted its running processings and put their
tasks back. An agent not running is reported informationally only.
 

## ENVIRONMENT

- `ASE_TOOL`: The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`: The bearer token of the agent relay, if not
embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`: Set to- `true`for the agent tool processes spawned by
the agent.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`: The process state (kind, process id, and
process start time) of the agent, existing while it runs.
 
- `.ase/agent/`- *name*- `.log`: The log of the agent run in the background.
 
- `.ase/worktree/.temp/`- *id*- `-editor`: The temporary, detached Git
worktree of a- `branch:`source of a task during its edit (where a- `#`of the task id is replaced by- `_`).
 
- *per-user state
directory*- `/agent-editor/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`:
The run lock and the run record of each editor agent process, used to
recover the edit requests of terminated agent processes (see- **INTERRUPTS AND RECOVERY**in- `ase-agent(1)`).
 

## EXIT STATUS

`ase agent editor start` exits with status `0` once the agent started
(or was already running), resp. once a foreground agent terminated
regularly, and with status `1` on any error (e.g., an invalid option, or
a missing agent tool). `ase agent editor status` exits with status `1`
if the agent is not running.

## EXAMPLES

```
# process the edit requests of the task board
$ ase agent editor start
# request an edit of a task from the command line
$ ase task request FOO-42 edit "split step 3 into two steps"
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-rebooter(1)`,
`ase-agent-condenser(1)`, `ase-agent-researcher(1)`,
`ase-agent-implementer(1)`, `ase-agent-qualifier(1)`,
`ase-agent-integrator(1)`, `ase-agent-estimator(1)`,
`ase-task-request(1)`, `ase-agent-adhoc(1)`, `ase-agent-fleet(1)`,
`ase-agent-relay(1)`, `ase-task(1)`, `ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-estimator(1) - Effort Estimation Agent

## NAME

`ase-agent-estimator(1)` - Effort Estimation Agent

## SYNOPSIS

`ase agent estimator start`
[`-n`|`--name` *name*]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-m`|`--mode` `phase`|`request`]
[`-M`|`--model` *model*]
[`--`*group*`-`*item* *value* [...]]
[`-f`|`--foreground`]

`ase agent estimator status`
[`-n`|`--name` *name*]

`ase agent estimator log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent estimator stop`
[`-n`|`--name` *name*]

## DESCRIPTION

**ase agent estimator** estimates the effort of tasks, i.e., it stores
the expected human time of a task as a T-shirt size in its `Effort:`
frontmatter key (see the skill `ase-task-estimate`). Unlike the phase
agents, it drives no phase and supports all task lifecycle models
(`solo`, `team`, and `enterprise`). It runs in one of two modes (see
`--mode`): the mode `phase` (the default) estimates the unestimated
planned tasks automatically and is intended for the *final* estimation
of the tasks, once their plans are complete, while the mode `request`
(re-)estimates the tasks in any unfinished status on request and is
intended for *dry-runs*, i.e., provisional estimations of plans still
in progress.

In the mode `phase`, the estimator observes the tasks in the state `PLANNED`, adapted onto
the task lifecycle model of the project, i.e., `PLANNED` for
`enterprise`, `PLANNING` for `team`, and `OPEN` for `solo` (see
`--state-incoming`), which do not carry its tag `ase-agent-estimator`
yet, independent of their `Assignee:` and their `After:` predecessors.
Of these, it processes every task which carries no valid `Effort:` key
yet, whose plan is not empty, and which is no explicit epic (a task
whose `Group:` is its own id, as its effort is the sum of the efforts
of its members).

The estimator processes each such task *unclaimed* (see
`--unclaimed`): it neither moves the task into an activity state, nor
tags it as active, nor assigns it to itself, and it never changes its
status or its assignee. Hence, it keeps no other agent and no human off
the task. However, the built-in `implementer` by default waits with a
`PLANNED` task until the estimator is done with it (see its parameter
`effort-wait` in `ase-agent-implementer(1)`), so with both agents
running, the tasks are estimated before their implementation. An agent
of the kind `estimator` does not pick up a task already processed by
another one on the same host.

It estimates the effort via the skill invocation `ase-task-estimate`
*id*, which writes just the `Effort:` key in a conditional save, i.e.,
only if the plan was not changed meanwhile (re-estimating the task
otherwise), so a concurrent change of the task by another agent or a
human is never lost. It writes neither the `Status:` nor the
`Modified:` key of the task.

The estimation succeeds only if the agent tool succeeded, the skill
reported no error, and the task afterwards carries a valid `Effort:`
key. A successfully estimated task is left untagged, as its valid
`Effort:` key itself keeps it from being estimated again, until its
plan content changes, which drops its then stale `Effort:` key, so it
is estimated again. A task not estimated is tagged with
`ase-agent-estimator` instead, so it is not processed again and again
(removing this tag requests another estimation).

In the mode `request`, the estimator instead processes the `estimate`
requests attached to tasks in any lifecycle status except a finished
one (e.g., `DRAFTED`, which the mode `phase` never picks up), as
requested via `ase task request` *id* `estimate` (see
`ase-task-request(1)`) resp. key `R` of the task boards, where a
prompt is ignored and hence may be empty. Like the request
agent `editor` (see `ase-agent-editor(1)`), it claims such a task by
tagging it with `ase-agent-estimate:active` and assigning it to itself
(deferring a task in an activity status until it changes), estimates
its effort via the skill invocation `ase-task-estimate --force` *id*,
i.e., even if it already carries a valid `Effort:` key, restores its
previous assignee, and removes the request on success, else turns it
into a failed request (to be re-requested). The estimation succeeds
under the same conditions as in the mode `phase`, so a request for an
explicit epic or a task with an empty plan fails.

Both modes can run side by side as two agents of the kind `estimator`
under different names (e.g., `estimator` and `estimator-request`).

On an interrupt, a task being processed in the mode `phase` is left
untouched, while the request of a task being processed in the mode
`request` is turned back into a pending one. The
estimator works in the project root only, so it neither requires a Git
working copy nor `git`, and it accepts no `--repo` and `--tempdir`. See
`ase-agent(1)` for the common mechanics of all agents.

## OPTIONS

- [`-h`|`--help`]: Show usage information of`ase agent estimator`(or of the given sub-command) only.

All other options belong to the sub-commands (see **COMMANDS**).

## COMMANDS

- `ase agent estimator start`[- *options*]: Start the estimator agent
in the background, logging into- `.ase/agent/`- *name*- `.log`. The agent
is validated and initialized (e.g., its agent tool is looked up)
before the command returns (waiting up to 30 seconds), so a failing
agent is reported together with the tail of its log. An agent of the
same name already running is left alone. The options are:
 - - [- `-n`|- `--name`- *name*]: The name of the agent (default:- `estimator`): up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit, other than the name of another
agent kind,- `relay`, or- `fleet`.
 - It names the agent in its log lines (also in the shared processing
log attachment - `agent`), and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`. Agents
of the kind- `estimator`under different names form a pool,
competing for the same tasks and processing them in parallel to
each other.
 
- [- `-t`|- `--tool`- *tool*]: The headless agent tool processing the
tasks:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub
Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*) (default:- `$ASE_TOOL`, else- `claude`). Its command has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]: The maximum number of tasks processed
in parallel by this agent (default:- `1`), a positive integer.
 
- [- `-T`|- `--timeout`- *duration*]: The maximum duration of
processing a task (the run of the agent tool), as- *number*- *unit*(without a space) with- *unit*- `s`(seconds),- `m`(minutes), or- `h`(hours) (default:- `60m`). A processing exceeding it is aborted and
counts as failed.
 
- [- `-o`|- `--one-shot`]: Terminate the agent, instead of observing
for further tasks, once a check of the task store finds no
dispatchable task while the agent processes no task anymore --
intended for CI contexts like GitHub Actions, and not allowed in a
fleet. A check of the task store failing three times in a row then
terminates the agent with a non-zero exit status (instead of being
retried endlessly after a growing delay).
 
- [- `--agent-relay`- *url*]: The agent relay to pass the live
presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for
HTTPS (default: the configured- `project.relay.url`, else none).
See- **AGENT RELAY**in- `ase-agent(1)`.
 
- [- `-m`|- `--mode`- *mode*]: The mode of the agent (default:- `phase`):- `phase`estimates the unestimated tasks in- `PLANNED`automatically and unclaimed (intended for the final estimation),
while- `request`(re-)estimates the tasks of the- `estimate`requests in any unfinished status (intended for dry-runs). An
invalid mode is an error. The built-in values of the following
options are given per mode where they differ, and in the mode- `request`, the options for- `phase`only (- `--unclaimed`,- `--state-`*,- `--tag-success`,- `--tag-failure`,- `--tag-retry`,- `--assign-required`, and- `--attach-skip`) are not allowed.
 
- [- `--title`- *text*]: The one-line title of the operation of the
agent (at most 80 characters), shown on the task boards. Built-in:- `Estimate the effort of unestimated planned tasks`(- `phase`) resp.- `Re-estimate the effort of tasks on request`(- `request`).
 
- [- `--description`- *text*]: The brief description of the operation
of the agent (about 3-6 lines, at most 600 characters, with its
whitespace collapsed), shown on the task boards. Built-in: a
description of the operation of the mode.
 
- [- `--type`- `phase`|- `request`]: The processing type:- `phase`claims the tasks by their lifecycle status,- `request`processes
the prompt requests attached to the tasks. Generic default:- `phase`. Built-in:- `phase`(mode- `phase`) resp.- `request`(mode- `request`).
 
- [- `--workdir`- *kind*]: The working directory of the agent tool:- `project`,- `source`,- `changeset`,- `changeset-read`, or- `target`.
Generic default:- `project`. Built-in:- `project`.
 
- [- `--lifecycle`- *model*]: A supported task lifecycle model
(- `solo`,- `team`, or- `enterprise`, repeatable). Generic default:
every model containing all given states, resp. all models for- `request`and with- `--unclaimed`. Built-in: not set. Hence, all
task lifecycle models are supported.
 
- [- `--serial`]: Process the tasks serially only, ignoring- `--parallel`. Built-in: not set.
 
- [- `--unclaimed`]: Process the tasks of the incoming states
without claiming them, independent of their assignee and
predecessors, and without ever changing their status and assignee
(- `phase`only). Built-in: enabled (mode- `phase`) resp. not set
(mode- `request`).
 
- [- `-M`|- `--model`- *model*]: The model of the agent tool: a model
for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all other
tasks), see- `ase-agent-adhoc(1)`. Generic default: the default
model of the agent tool. Built-in: none.
 
- [- `--params`- *name*- `=`- *value*]: The value of a parameter
declared by the definition (repeatable). Built-in: none declared.
 
- [- `--state-incoming`- *state*]: The state of the tasks to process
(mandatory for- `phase`, repeatable with- `--unclaimed`only), where
each state is adapted onto the task lifecycle model of the project.
Built-in:- `PLANNED`(i.e.,- `PLANNING`for- `team`and- `OPEN`for- `solo`) (mode- `phase`) resp. not set (mode- `request`). For
instance,- `--state-incoming DRAFTED --state-incoming PLANNED`additionally estimates the drafted tasks of the- `enterprise`model in the mode- `phase`.
 
- [- `--state-activity`- *state*]: The state of a task during its
processing. Generic default: the incoming state. Built-in: not
set. Not allowed, as the estimator processes the tasks unclaimed.
 
- [- `--state-success`- *state*]: The target state on success.
Generic default: the incoming state. Built-in: not set. Not
allowed, as the estimator processes the tasks unclaimed.
 
- [- `--state-failure`- *state*]: The target state on failure.
Generic default: the incoming state. Built-in: not set. Not
allowed, as the estimator processes the tasks unclaimed.
 
- [- `--state-phase`- *state*]: The phase success state, into which
the skill itself can move a task. Generic default: the success
state. Built-in: not set. Not allowed, as the estimator processes
the tasks unclaimed.
 
- [- `--state-passed`- *state*]: The first state from which on a task
has passed the phase for its- `After:`successors. Generic default:
the phase success state. Built-in: not set. Not allowed, as the
estimator processes the tasks unclaimed.
 
- [- `--state-verdict`- *VERDICT*- `=`- *state*]: The target state per
verdict (repeatable). Built-in: not set. Not allowed, as the
estimator processes the tasks unclaimed.
 
- [- `--tag-name`- *tag*]: The tag of the processed tasks
(- `ase-agent-`- *word*). Generic default:- `ase-agent-`- *name*, resp.
for- `request`always- `ase-agent-`- *request-name*. Built-in:- `ase-agent-estimator`(mode- `phase`) resp.- `ase-agent-estimate`(mode- `request`).
 
- [- `--tag-success`- *tag*]: The tag on success:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in:- `none`(mode- `phase`), as the valid- `Effort:`key of an estimated task keeps it from being estimated
again, resp. not set (mode- `request`).
 
- [- `--tag-failure`- *tag*]: The tag on failure:- *tag-name*,- *tag-name*- `:`- *value*, or- `none`(- `phase`only). Generic default:- *tag-name*. Built-in: not set.
 
- [- `--tag-require`- *tag*]: A tag a claimable task has to carry
(repeatable). Built-in: not set.
 
- [- `--tag-exclude`- *tag*]: A tag a claimable task must not carry
(repeatable). Built-in: not set.
 
- [- `--tag-retry`]: A shorthand for- `--tag-failure none`, so a
failed task is processed again after a growing delay (- `phase`only). Built-in: not set.
 
- [- `--assign-name`- *who*]: The assignee under which the tasks are
claimed, held, recognized, and released, and which names the
agent-owned worktrees. Generic default: the agent name. Built-in:
not set. As the estimator never assigns a task in the mode- `phase`, it then names the mode of the agent as announced to the
agent relay only.
 
- [- `--assign-required`]: Claim only the tasks explicitly assigned
to- *who*of- `--assign-name`(- `phase`only). Built-in: not set. Not
allowed, as the estimator processes the tasks unclaimed.
 
- [- `--assign-success`- *who*]: The assignee of a task after its
successful processing: a name,- `none`, or- `previous`. Generic
default:- `none`for- `phase`,- `previous`for- `request`. Built-in:
not set. Not allowed in the mode- `phase`, as the estimator then
processes the tasks unclaimed.
 
- [- `--assign-failure`- *who*]: The assignee of a task after its
failed processing: a name,- `none`, or- `previous`. Generic default:- `none`for- `phase`,- `previous`for- `request`. Built-in: not set.
Not allowed in the mode- `phase`, as the estimator then processes
the tasks unclaimed.
 
- [- `--request-name`- *name*]: The name of the processed requests
(- `request`only). Generic default: the agent name. Built-in: not
set (mode- `phase`) resp.- `estimate`(mode- `request`).
 
- [- `--request-done`- *word*]: The word naming a processed task in
the log lines (- `request`only). Generic default:- `processed`.
Built-in: not set (mode- `phase`) resp.- `estimated`(mode- `request`).
 
- [- `--prompt-skill`- *name*]: The skill to invoke (mutually
exclusive with- `--prompt-text`and- `--prompt-file`). Built-in:- `ase-task-estimate`.
 
- [- `--prompt-args`- *template*]: The argument template of the
skill. Generic default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`. Built-in:- `{id}`(mode- `phase`) resp.- `--force {id}`(mode- `request`), the latter ignoring the request prompt.
 
- [- `--prompt-text`- *template*]: The prompt text template to invoke
instead of a skill. Built-in: not set.
 
- [- `--prompt-file`- *file*]: The file with the prompt text template
to invoke instead of a skill. Built-in: not set.
 
- [- `--prepare-run`- *cmd*]: A shell command run in the working
directory before the prompt (repeatable). Built-in: not set.
 
- [- `--success-criterion`- *criterion*]: A success criterion:- `exit`,- `clean`,- `changed`,- `verdict`,- `attachment`, or- `status`(repeatable). Generic default:- `exit`and- `clean`for- `phase`,- `clean`and- `changed`for- `request`. Built-in:- `exit`and- `clean`(in both modes, as a re-estimation of the same size leaves
the plan content unchanged), where additionally the task has to
carry a valid- `Effort:`key afterwards.
 
- [- `--success-match`- *regex*]: A regular expression an output line
has to match (repeatable). Built-in: not set.
 
- [- `--verdict-label`- *label*]: The label after which the verdict
is given in the output. Built-in: not set.
 
- [- `--verdict-allow`- *VERDICT*]: An accepted verdict (repeatable).
Generic default: the verdicts of- `--state-verdict`, else any.
Built-in: not set.
 
- [- `--attach-output`- *kind*]: The kind of the attachment storing
the output lines of the prompt. Built-in: not set.
 
- [- `--attach-expect`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*to be created or changed by the
processing for the success criterion- `attachment`. Built-in: not
set.
 
- [- `--attach-skip`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*leaving a task unclaimed (- `phase`only).
Built-in: not set.
 
- [- `--attach-changeset`- *kind*]: The kind of the attachment- `text/x-diff;`- *kind*storing the change set snapshot of the task
(with- `--workdir changeset`or- `target`only). Built-in: not set.
Not allowed, as the estimator works in the project root.
 
- [- `--finish-commit`]: Commit the work onto a- `branch:`changeset
and publish it (with- `--workdir changeset`only). Built-in: not
set. Not allowed, as the estimator works in the project root.
 
- [- `--finish-success`- *cmd*]: A shell command run in the working
directory after a successful processing. Built-in: not set.
 
- [- `--finish-failure`- *cmd*]: A shell command run in the working
directory after a failed processing. Built-in: not set.
 
- [- `-f`|- `--foreground`]: Run the agent in the current process
instead, logging to the terminal, until it is interrupted (e.g.,
via- `CTRL+C`) or stopped, where an agent of the same name already
running is an error.
 
- [- `-h`|- `--help`]: Show the usage information of the sub-command
only.
 
 - The options - `--`- *group*- `-`- *item*above override the built-in
definition of the agent (where an option also replaces its mutually
exclusive alternatives, e.g.,- `--prompt-text`replaces the skill of- `--prompt-skill`), while the agent keeps the identity of its kind (its
runs and its presence are named after- `estimator`). See- `ase-agent-adhoc(1)`for their full semantics.
 
- `ase agent estimator status`[- `-n`|- `--name`- *name*]: Report whether
the estimator agent of the name- *name*(default:- `estimator`) is
running, as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`(exit status 1).
 
- `ase agent estimator log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]
[- `--name`- *name*]: Show the log- `.ase/agent/`- *name*- `.log`of the
estimator agent of the name- *name*(default:- `estimator`) run in the
background, where an agent of a fleet logs into the log of the fleet
instead (see- `ase-agent-fleet(1)`). Without a log file, an
informational message is shown only. The options are:
 - - [- `-f`|- `--follow`]: Show the last- *n*lines (default: 10) and
then follow all newly appended lines, in the style of- `tail -f`,
until interrupted.
 
- [- `-n`|- `--lines`- *n*]: Show the last- *n*lines only (default:
all, or 10 with- `--follow`).
 
- [- `--name`- *name*]: The name of the agent (default:- `estimator`), given in its long form only, as- `-n`is the short
form of- `--lines`.
 
 
- `ase agent estimator stop`[- `-n`|- `--name`- *name*]: Stop the
estimator agent of the name- *name*(default:- `estimator`) by- `SIGTERM`, waiting (up to 5 minutes) until it aborted its running
processings. An agent not running is reported informationally only.
 

## CONFIGURATION

The estimation itself is configured by the keys
`project.task.effort.scale`, `project.task.effort.factor`,
`project.task.effort.file`, `project.task.effort.line`, and
`project.task.effort.iteration` of the skill
`ase-task-estimate` (see `ase-config(1)`).

## ENVIRONMENT

- `ASE_TOOL`: The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`: The bearer token of the agent relay, if not
embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`: Set to- `true`for the agent tool processes spawned by
the agent.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`: The process state (kind, process id, and
process start time) of the agent, existing while it runs.
 
- `.ase/agent/`- *name*- `.log`: The log of the agent run in the background.
 
- *per-user state
directory*- `/agent-estimator/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`: The run lock and the run record of each estimator agent
process, which records the tasks it currently processes, so no other
agent of the kind- `estimator`on the same host picks them up.
 

## EXIT STATUS

`ase agent estimator start` exits with status `0` once the agent
started (or was already running), resp. once a foreground agent
terminated regularly, and with status `1` on any error (e.g., an
invalid option or a missing agent tool). `ase agent estimator status`
exits with status `1` if the agent is not running.

## EXAMPLES

```
# start the estimator, estimating the planned tasks without a valid effort
$ ase agent estimator start
# estimate all drafted and planned tasks once, e.g., in a CI context
$ ase agent estimator start -f -o --state-incoming DRAFTED --state-incoming PLANNED
# start a second estimator of another agent tool as a pool
$ ase agent estimator start -n estimator-codex -t codex
# additionally start an estimator processing the "estimate" requests,
# and request the (re-)estimation of a drafted task
$ ase agent estimator start -n estimator-request -m request
$ ase task request example-task estimate re-estimate
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-rebooter(1)`,
`ase-agent-condenser(1)`, `ase-agent-researcher(1)`,
`ase-agent-implementer(1)`, `ase-agent-qualifier(1)`,
`ase-agent-integrator(1)`, `ase-agent-editor(1)`, `ase-agent-adhoc(1)`,
`ase-agent-fleet(1)`, `ase-agent-relay(1)`, `ase-task(1)`,
`ase-task-board(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-researcher(1) - Research Agent

## NAME

`ase-agent-researcher(1)` - Research Agent

## SYNOPSIS

`ase agent researcher start`
[`-n`|`--name` *name*]
[`-t`|`--tool` `claude`|`copilot`|`codex`]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-r`|`--repo` *url*]
[`-d`|`--tempdir` *dir*]
[`-m`|`--mode` `phase`|`request`]
[`-M`|`--model` *model*]
[`--`*group*`-`*item* *value* [...]]
[`-f`|`--foreground`]

`ase agent researcher status`
[`-n`|`--name` *name*]

`ase agent researcher log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent researcher stop`
[`-n`|`--name` *name*]

## DESCRIPTION

**ase agent researcher** researches for the plans of tasks, combining
a Web research with a grounding in the source artifacts of the project
(see the skill `ase-task-research`). It runs in one of two modes (see
`--mode`): the mode `phase` (the default) researches the plans of the
tasks explicitly assigned to it in the `Planning` phase of the
`enterprise` task lifecycle model and is intended for *researching and
grounding the plan content*, while the mode `request` processes the
research requests of tasks in any task lifecycle model and is intended
for *topic-driven research*.

In the mode `phase` (refused for any task lifecycle model other than
`enterprise`), as a plan is researched only on the explicit request of
a human, the researcher claims only the tasks in the state `DRAFTED`
whose `Assignee:` frontmatter key already is its agent name
(default: `researcher`), i.e., which a human assigned to the
researcher. Such a task is claimed immediately, without waiting for its
`After:` predecessors (and, for an explicit epic, its group members),
except for a predecessor still being processed by a researcher in the
mode `phase` or a cancelled `After:` predecessor. It moves each claimed task
into `PLANNING`, tags it with `ase-agent-researcher:active`, and
researches the plan itself via the skill invocation `ase-task-research --refine` *id* in the working copy, grounded in the `Source:` of the
task (like in the mode `request`, see below), appending the research
report to the `research` attachment of the task and refining the plan
by the recommendations of the report. The processing succeeds under the
same conditions as in the mode `request`. Afterwards, on success as
well as on failure, the researcher puts the task back into `DRAFTED`,
removes its tag, and unassigns it. As the task is no longer assigned to
the researcher, it is not claimed again, while assigning it to the
researcher again requests another research. On an interrupt, the task
is put back, untagged and with its previous assignee (i.e., still
assigned to the researcher), into `DRAFTED`.

In the mode `request`, the researcher processes the research requests
of tasks. Like the request agent `editor`, it is then bound to no phase
and supports all task lifecycle models (`solo`, `team`, and
`enterprise`).

A research request is added by key `R` of the task board (see
`ase-task-board(1)`) or by `ase task request` *id* `research`
[*topic*] (see `ase-task-request(1)`) as an attachment of type
`text/plain; charset=utf-8; kind="research"` with the topic as its
data, mirrored in the tag `ase-agent-research:pending` of the task
(aggregated as `:active` while one is processed and as `:failed` once
one failed). The topic is optional: a request without a topic
researches the task plan itself.

The researcher observes the tasks with this tag, except those in an
activity status of the task lifecycle model, where the task is actively
worked on (`team`: `IMPLEMENTING`; `enterprise`: `PLANNING`,
`IMPLEMENTING`, `QUALIFYING`, and `INTEGRATING`), and those still held
by a phase agent (tagged `ase-agent-`*kind*`:active`), which wait until
their status changes resp. their tag is released.

It claims the oldest request of such a task in a single conditional
save by turning it into `kind="research-running"` and setting the
`Assignee:` of the task to its agent name (remembering the previous
assignee in the request), which keeps all other agents off the task
during the research without touching its status. The request of a
finished task is turned into `kind="research-failed"` instead.

It then researches the topic via the skill invocation
`ase-task-research --refine` *id*`:` [*topic*] (with the whitespace
of the topic collapsed) in the working copy, grounded in the `Source:`
of the task, where a `branch:`*name* source is checked out in the
temporary, detached Git worktree `.ase/worktree/.temp/`*id*`-researcher`,
passed to the skill via its internal option `--int-source-dir`, and
removed afterwards. The skill combines a Web research with a grounding
in the source artifacts of the project, appends its research report to
the `research` attachment of the task (of type `text/markdown; charset=utf-8; kind="research"`), and refines the plan by the
recommendations of the report, all in a single save, which happens
only if the task was not changed meanwhile.

Finally, in a single save, the researcher restores the previous
assignee of the task (unless reassigned meanwhile) and removes the
request if the `research` attachment was created or changed and the
skill reported the attached research, or else turns it back into
`kind="research"` on an interrupt, resp. into `kind="research-failed"`
(with the reason, e.g., `skill reported an error`, `agent tool failed`,
or `no new or changed attachment "text/markdown;research"`) otherwise.

A finalization failing even after its retries is retried on every
further dispatching.

On its start in the mode `request`, the researcher marks the requests left in
`kind="research-running"` by any terminated researcher process of the
same user on the same host (whose run lock is gone or stale and whose
process is no longer alive) as failed (`kind="research-failed"`), and
restores the previous assignees of their tasks.

Both modes can run side by side as two agents of the kind `researcher`
under different names (e.g., `researcher` and `researcher-request`).
See `ase-agent(1)` for the common mechanics of all agents.

## OPTIONS

- [`-h`|`--help`]: Show usage information of`ase agent researcher`(or of the given sub-command) only.

All other options belong to the sub-commands (see **COMMANDS**).

## COMMANDS

- `ase agent researcher start`[- *options*]: Start the researcher agent
in the background, logging into- `.ase/agent/`- *name*- `.log`. The agent
is validated and initialized (e.g., its agent tool is looked up and
the task lifecycle model is checked) before the command returns
(waiting up to 30 seconds), so a failing agent is reported together
with the tail of its log. An agent of the same name already running is
left alone. The options are:
 - - [- `-n`|- `--name`- *name*]: The name of the agent (default:- `researcher`): up to 64 letters, digits, underscores, and dashes,
starting with a letter or digit, other than the name of another
agent kind,- `relay`, or- `fleet`.
 - It names the agent in its log lines (also in the shared processing
log attachment - `agent`), the- `Assignee:`of the tasks it processes, its
temporary directory, and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`. Agents of the kind- `researcher`under different names form a pool, competing for the same tasks and
processing them in parallel to each other.
 
- [- `-t`|- `--tool`- *tool*]: The headless agent tool processing the
tasks:- `claude`(- *Anthropic Claude Code CLI*),- `copilot`(- *GitHub
Copilot CLI*), or- `codex`(- *OpenAI Codex CLI*) (default:- `$ASE_TOOL`, else- `claude`). Its command has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]: The maximum number of tasks processed
in parallel by this agent (default:- `1`), a positive integer.
 
- [- `-T`|- `--timeout`- *duration*]: The maximum duration of
processing a task (the run of the agent tool), as- *number*- *unit*(without a space) with- *unit*- `s`(seconds),- `m`(minutes), or- `h`(hours) (default:- `60m`). A processing exceeding it is aborted and
counts as failed.
 
- [- `-o`|- `--one-shot`]: Terminate the agent, instead of observing
for further tasks, once a check of the task store finds no
dispatchable task while the agent processes no task anymore --
intended for CI contexts like GitHub Actions, and not allowed in a
fleet. A check of the task store failing three times in a row then
terminates the agent with a non-zero exit status (instead of being
retried endlessly after a growing delay).
 
- [- `--agent-relay`- *url*]: The agent relay to pass the live
presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*]
resp.- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] for
HTTPS (default: the configured- `project.relay.url`, else none).
See- **AGENT RELAY**in- `ase-agent(1)`. In the mode- `request`, the
agent is announced there as processing the- `research`requests,
which may have an empty prompt.
 
- [- `-r`|- `--repo`- *url*]: The Git repository to clone once into the
temporary directory of the agent, where every task without a- `worktree`value is processed in its own temporary Git worktree of
this clone (default: none, i.e., all tasks are processed in the
Git working copy the agent is started in, which is hence
mandatory).
 - Requires a task store not located in the working copy (i.e., not a
local - `ase:`- *path*one). See- **WORKING COPY, CLONES, AND WORKTREES**in- `ase-agent(1)`.
 
- [- `-d`|- `--tempdir`- *dir*]: Create the temporary directory of the
agent as the sub-directory- *name*of- *dir*, where- `%p`is expanded
to the process id of the agent,- `%h`to a hash value unique to the
process, and- `%%`to- `%`(default: a fresh, private directory
below the system temporary directory). A missing- *dir*is created
private to the current user, while an existing- *dir*has to be a
directory (no symbolic link) owned by the current user.
 
- [- `-m`|- `--mode`- *mode*]: The mode of the agent (default:- `phase`):- `phase`researches and grounds the plans of the tasks
explicitly assigned to the researcher in- `DRAFTED`,
while- `request`researches the topics of the- `research`requests
of the tasks in any task lifecycle model. An invalid mode is an
error. The built-in values of the following options are given per
mode where they differ, and in the mode- `request`, the options for- `phase`only (- `--unclaimed`,- `--state-`*,- `--tag-success`,- `--tag-failure`,- `--tag-retry`,- `--assign-required`, and- `--attach-skip`) are not allowed.
 
- [- `--title`- *text*]: The one-line title of the operation of the
agent (at most 80 characters), shown on the task boards. Built-in:- `Research and ground the plans of assigned tasks`(mode- `phase`)
resp.- `Research topics for tasks on request`(mode- `request`).
 
- [- `--description`- *text*]: The brief description of the operation
of the agent (about 3-6 lines, at most 600 characters, with its
whitespace collapsed), shown on the task boards. Built-in: a
description of the operation of the mode.
 
- [- `--type`- `phase`|- `request`]: The processing type:- `phase`claims the tasks by their lifecycle status,- `request`processes
the prompt requests attached to the tasks. Generic default:- `phase`. Built-in:- `phase`(mode- `phase`) resp.- `request`(mode- `request`).
 
- [- `--workdir`- *kind*]: The working directory of the agent tool:- `project`,- `source`,- `changeset`,- `changeset-read`, or- `target`.
Generic default:- `project`. Built-in:- `source`.
 
- [- `--lifecycle`- *model*]: A supported task lifecycle model
(- `solo`,- `team`, or- `enterprise`, repeatable). Generic default:
every model containing all given states, resp. all models for- `request`and with- `--unclaimed`. Built-in:- `enterprise`(mode- `phase`) resp. not set (mode- `request`). Hence, all task lifecycle
models are supported in the mode- `request`.
 
- [- `--serial`]: Process the tasks serially only, ignoring- `--parallel`. Built-in: not set.
 
- [- `--unclaimed`]: Process the tasks of the incoming states
without claiming them (- `phase`only). Built-in: not set.
 
- [- `-M`|- `--model`- *model*]: The model of the agent tool: a model
for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all other
tasks), see- `ase-agent-adhoc(1)`. Generic default: the default
model of the agent tool. Built-in: none.
 
- [- `--params`- *name*- `=`- *value*]: The value of a parameter
declared by the definition (repeatable). Built-in: none declared.
 
- [- `--state-incoming`- *state*]: A state of the tasks to claim
(mandatory for- `phase`, repeatable with- `--unclaimed`or- `--state-activity`only). Built-in:- `DRAFTED`(mode- `phase`).
 
- [- `--state-activity`- *state*]: The state of a task during its
processing. Built-in:- `PLANNING`(mode- `phase`).
 
- [- `--state-success`- *state*], [- `--state-failure`- *state*]: The
target state on success resp. failure, or- `previous`for the
incoming state of the claim. Built-in:- `DRAFTED`(mode- `phase`).
 
- [- `--state-phase`- *state*]: The phase success state. Built-in: not
set (mode- `phase`), hence- `DRAFTED`.
 
- [- `--state-passed`- *state*]: The first state from which on a task
has passed the phase for its- `After:`successors. Built-in: not
set (mode- `phase`), hence- `DRAFTED`, the first state, so the
predecessors impose no gate.
 
- [- `--tag-name`- *tag*]: The tag of the processed tasks
(- `ase-agent-`- *word*). Generic default:- `ase-agent-`- *name*, resp.
for- `request`always- `ase-agent-`- *request-name*. Built-in: not set
(mode- `phase`, hence- `ase-agent-researcher`) resp.- `ase-agent-research`(mode- `request`, derived from the request
name- `research`).
 
- [- `--tag-success`- *tag*], [- `--tag-failure`- *tag*]: The tag on
success resp. failure (- `phase`only). Built-in:- `none`(mode- `phase`), as the unassignment of a processed task keeps it from
being claimed again.
 
- [- `--tag-require`- *tag*]: A tag a claimable task has to carry
(repeatable). Built-in: not set.
 
- [- `--tag-exclude`- *tag*]: A tag a claimable task must not carry
(repeatable). Built-in: not set.
 
- [- `--assign-name`- *who*]: The assignee under which the tasks are
claimed, held, recognized, and released, and which names the
agent-owned worktrees. Generic default: the agent name. Built-in:
not set.
 
- [- `--assign-required`]: Claim only the tasks explicitly assigned
to- *who*of- `--assign-name`(- `phase`only). Built-in: enabled
(mode- `phase`).
 
- [- `--assign-success`- *who*]: The assignee of a task after its
successful processing: a name,- `none`, or- `previous`. Generic
default:- `none`for- `phase`,- `previous`for- `request`. Built-in:
not set.
 
- [- `--assign-failure`- *who*]: The assignee of a task after its
failed processing: a name,- `none`, or- `previous`. Generic default:- `none`for- `phase`,- `previous`for- `request`. Built-in: not set.
 
- [- `--request-name`- *name*]: The name of the processed requests
(- `request`only). Generic default: the agent name. Built-in:- `research`(mode- `request`).
 
- [- `--request-done`- *word*]: The word naming a processed task in
the log lines (- `request`only). Generic default:- `processed`.
Built-in:- `researched`(mode- `request`).
 
- [- `--prompt-skill`- *name*]: The skill to invoke (mutually
exclusive with- `--prompt-text`and- `--prompt-file`). Built-in:- `ase-task-research`.
 
- [- `--prompt-args`- *template*]: The argument template of the
skill. Generic default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`. Built-in:- `--refine{?source-dir: --int-source-dir "{source-dir}"} {id}`(mode- `phase`) resp.- `--refine{?source-dir: --int-source-dir "{source-dir}"} {id}:{?prompt: {prompt}}`(mode- `request`), where the conditional- `{?prompt:...}`makes the prompt
optional.
 
- [- `--prompt-text`- *template*]: The prompt text template to invoke
instead of a skill. Built-in: not set.
 
- [- `--prompt-file`- *file*]: The file with the prompt text template
to invoke instead of a skill. Built-in: not set.
 
- [- `--prepare-run`- *cmd*]: A shell command run in the working
directory before the prompt (repeatable). Built-in: not set.
 
- [- `--success-criterion`- *criterion*]: A success criterion:- `exit`,- `clean`,- `changed`,- `verdict`,- `attachment`, or- `status`(repeatable). Generic default:- `exit`and- `clean`for- `phase`,- `clean`and- `changed`for- `request`. Built-in:- `exit`,- `clean`,
and- `attachment`.
 
- [- `--success-match`- *regex*]: A regular expression an output line
has to match (repeatable). Built-in:- `status:\W*research attached\b`.
 
- [- `--verdict-label`- *label*]: The label after which the verdict
is given in the output. Built-in: not set.
 
- [- `--verdict-allow`- *VERDICT*]: An accepted verdict (repeatable).
Generic default: the verdicts of- `--state-verdict`, else any.
Built-in: not set.
 
- [- `--attach-output`- *kind*]: The kind of the attachment storing
the output lines of the prompt. Built-in: not set.
 
- [- `--attach-expect`- *spec*]: The attachment
[- *mime-type*- `;`]- *kind*to be created or changed by the
processing for the success criterion- `attachment`. Built-in:- `text/markdown;research`.
 
- [- `--finish-success`- *cmd*]: A shell command run in the working
directory after a successful processing. Built-in: not set.
 
- [- `--finish-failure`- *cmd*]: A shell command run in the working
directory after a failed processing. Built-in: not set.
 
- [- `-f`|- `--foreground`]: Run the agent in the current process
instead, logging to the terminal, until it is interrupted (e.g.,
via- `CTRL+C`) or stopped, where an agent of the same name already
running is an error.
 
- [- `-h`|- `--help`]: Show the usage information of the sub-command
only.
 
 - The options - `--`- *group*- `-`- *item*above override the built-in
definition of the mode of the agent (where an option also replaces its
mutually exclusive alternatives, e.g.,- `--prompt-text`replaces the
skill of- `--prompt-skill`), while the agent keeps the identity of its
kind (its runs, its presence, and its agent-owned worktrees are named
after- `researcher`). See- `ase-agent-adhoc(1)`for their full
semantics.
 
- `ase agent researcher status`[- `-n`|- `--name`- *name*]: Report whether
the researcher agent of the name- *name*(default:- `researcher`) is
running, as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`(exit status 1).
 
- `ase agent researcher log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]
[- `--name`- *name*]: Show the log- `.ase/agent/`- *name*- `.log`of the
researcher agent of the name- *name*(default:- `researcher`) run in the
background, where an agent of a fleet logs into the log of the fleet
instead (see- `ase-agent-fleet(1)`). Without a log file, an
informational message is shown only. The options are:
 - - [- `-f`|- `--follow`]: Show the last- *n*lines (default: 10) and
then follow all newly appended lines, in the style of- `tail -f`,
until interrupted.
 
- [- `-n`|- `--lines`- *n*]: Show the last- *n*lines only (default:
all, or 10 with- `--follow`).
 
- [- `--name`- *name*]: The name of the agent (default:- `researcher`),
given in its long form only, as- `-n`is the short form of- `--lines`.
 
 
- `ase agent researcher stop`[- `-n`|- `--name`- *name*]: Stop the
researcher agent of the name- *name*(default:- `researcher`) by- `SIGTERM`, waiting (up to 5 minutes) until it aborted its running
processings and put their tasks back. An agent not running is reported
informationally only.
 

## ENVIRONMENT

- `ASE_TOOL`: The default agent tool of- `--tool`(default:- `claude`).
 
- `ASE_AGENT_RELAY_TOKEN`: The bearer token of the agent relay, if not
embedded in the URL of- `--agent-relay`resp.- `project.relay.url`.
 
- `ASE_HEADLESS`: Set to- `true`for the agent tool processes spawned by
the agent.
 

## FILES

- `.ase/agent/`- *name*- `.yaml`: The process state (kind, process id, and
process start time) of the agent, existing while it runs.
 
- `.ase/agent/`- *name*- `.log`: The log of the agent run in the background.
 
- `.ase/worktree/.temp/`- *id*- `-researcher`: The temporary, detached Git
worktree of a- `branch:`source of a task during its research (where a- `#`of the task id is replaced by- `_`).
 
- *per-user state
directory*- `/agent-researcher/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`: The run lock and the run record of each researcher agent
process, used to recover the research requests of terminated agent
processes (see- **INTERRUPTS AND RECOVERY**in- `ase-agent(1)`).
 

## EXIT STATUS

`ase agent researcher start` exits with status `0` once the agent
started (or was already running), resp. once a foreground agent
terminated regularly, and with status `1` on any error (e.g., an invalid
option, a task lifecycle model other than `enterprise` in the mode
`phase`, or a missing agent tool). `ase agent researcher status` exits
with status `1` if the agent is not running.

## EXAMPLES

```
# research and ground the plans of the tasks assigned to the researcher
$ ase agent researcher start
# request the research of a drafted task by assigning it to the researcher
$ ase task assign FOO-42 researcher
# process the research requests of the task board
$ ase agent researcher start -n researcher-request -m request
# request a research of a topic for a task from the command line
$ ase task request FOO-42 research "streaming JSON serialization of large data sets"
# request a research of the task plan itself (without a topic)
$ ase task request FOO-42 research
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-rebooter(1)`,
`ase-agent-condenser(1)`, `ase-agent-implementer(1)`, `ase-agent-qualifier(1)`,
`ase-agent-integrator(1)`, `ase-agent-editor(1)`,
`ase-agent-estimator(1)`, `ase-task-request(1)`, `ase-agent-adhoc(1)`,
`ase-agent-fleet(1)`, `ase-agent-relay(1)`, `ase-task(1)`,
`ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-adhoc(1) - Run an Ad-Hoc Agent of a Definition Given by Options

## NAME

`ase-agent-adhoc(1)` - Run an Ad-Hoc Agent of a Definition Given by Options

## SYNOPSIS

`ase agent adhoc start`
[`-n`|`--name` *name*]
[`-t`|`--tool` *tool*]
[`-p`|`--parallel` *num*]
[`-T`|`--timeout` *duration*]
[`-o`|`--one-shot`]
[`--agent-relay` *url*]
[`-r`|`--repo` *url*]
[`-d`|`--tempdir` *dir*]
[`-D`|`--definition` *file*|`builtin:`*kind*[`-`*mode*]]
[`--title` *text*]
[`--description` *text*]
[`--type` `phase`|`request`]
[`--workdir` *kind*]
[`--lifecycle` *model*]
[`--serial`]
[`--unclaimed`]
[`-M`|`--model` *model*]
[`--params` *name*`=`*value*]
[`--state-`*item* *state*]
[`--tag-`*item* *tag*]
[`--assign-`*item* *who*]
[`--request-`*item* *value*]
[`--prompt-`*item* *value*]
[`--prepare-run` *cmd*]
[`--success-`*item* *value*]
[`--verdict-`*item* *value*]
[`--attach-`*item* *value*]
[`--finish-`*item* [*cmd*]]
[`-f`|`--foreground`]

`ase agent adhoc status`
[`-n`|`--name` *name*]

`ase agent adhoc log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]
[`--name` *name*]

`ase agent adhoc stop`
[`-n`|`--name` *name*]

`ase agent` *kind* `start`|`status`|`log`|`stop` [...]

## DESCRIPTION

`ase agent adhoc` runs an *ad-hoc agent*: an autonomous agent whose
*entire* behavior is given by its command-line options (optionally
based on the definition file or the built-in definition of
`--definition`), instead of by a definition file of its own. This
manual page is also the authoritative reference of the *agent
definitions* in general, i.e., of the options `--`*group*`-`*item*,
of the definition files, of the prompt templates, and of the hooks.

An agent definition configures the behavior of an agent declaratively.
All agents of **ase** are agents of a definition, run by the same
agent engine:

- *built-in agents*(- `ase agent rebooter`,- `condenser`,- `implementer`,- `qualifier`,- `integrator`,- `editor`,- `estimator`, and- `researcher`):
each mode of them is a built-in definition, which can be referenced as- `builtin:`- *kind*- `-`- *mode*(resp.- `builtin:`- *kind*for its default
mode):- `builtin:rebooter-reboot`,- `builtin:condenser-condense`,- `builtin:implementer-implement`,- `builtin:implementer-draft`,- `builtin:qualifier-judge`,- `builtin:qualifier-qualify`,- `builtin:integrator-merge`,- `builtin:integrator-rebase`,- `builtin:integrator-squash`,- `builtin:editor-edit`,- `builtin:estimator-phase`,- `builtin:estimator-request`,- `builtin:researcher-phase`, and- `builtin:researcher-request`.
 - All options - `--`- *group*- `-`- *item*below are also accepted by the
built-in agents, where they override the built-in definition (see- `ase-agent-rebooter(1)`etc.).
 
- *definition kinds*(- `ase agent`- *kind*): an agent kind of its own,
discovered from the definition file- *kind*- `.`- *ext*of the project
(- `.ase/agents/`, intended to be versioned) or of the user (- *per-user
configuration directory*- `/agents/`). A definition kind of the project
shadows one of the user, and both shadow a built-in agent kind of the
same name (whose- `--mode`then no longer exists).
 - For example, - `.ase/agents/reviewer.mts`yields the commands- `ase agent reviewer start|status|log|stop`, which take exactly the same
options as- `ase agent adhoc`described here, except- `--definition`(as
the definition file of the kind takes its place), and where the given
options override the definition file.
 - The definition files are discovered only on the commands - `ase agent`..., where a file of an invalid kind name (other than up to 64 letters,
digits, underscores, and dashes, starting with a letter or digit), of
the reserved kind names- `adhoc`,- `relay`, or- `fleet`, or of an already
discovered kind name in the same directory (by the precedence of the
extensions- `.yaml`,- `.yml`,- `.ts`,- `.mts`,- `.js`,- `.mjs`, and- `.cjs`,
in this order) is skipped with a warning.
 
- *ad-hoc agents*(- `ase agent adhoc`): an agent without a definition
file of its own, e.g., for a one-off agent, or for an agent defined
inline in a fleet (see- `ase-agent-fleet(1)`).
 

Each agent runs in its own process, in the background (logging into
`.ase/agent/`*name*`.log`) or with `--foreground` in the current
process (logging to the terminal). Several agents are best run
together as a fleet (see `ase-agent-fleet(1)`).

With `--type phase` (default), an agent claims tasks by their
*lifecycle status*, like the built-in phase agents: it observes the
task store for the tasks in the state of `--state-incoming` which do
not carry its tag (see `--tag-name`), whose `Assignee:` is empty or
its own assignee name (see `--assign-name`), and whose predecessors
(the tasks of the `After:` key, and, for an explicit epic, the other
tasks of its `Group:`) all passed its phase (see `--state-passed` and
the hook `passes`).

It claims each such task (in natural task id order, or in the order of
the hook `order`) in a single conditional save, which moves it into the
state of `--state-activity`, tags it with the tag value `active` (as
`<tag-name>:active`), and assigns it to the agent.

It then processes the task through the headless agent tool of `--tool`
with the prompt of `--prompt-skill`, `--prompt-text`, or `--prompt-file`,
checks the success criteria, and finally, again in a single conditional
save, settles the task: it tags it according to `--tag-success` resp.
`--tag-failure` (or with the found verdict as the tag value), hands it
over to the assignee of `--assign-success` resp. `--assign-failure`, and
moves it into the state of `--state-success` resp. `--state-failure` (or
of `--state-verdict`), where `previous` restores the incoming state the
task was claimed from.

A task whose status the skill itself moved elsewhere is tagged, too, and
left in that state, except that for a `previous` success or failure
state, a task the skill itself moved back into one of the incoming
states (e.g., `ase-task-reboot`, which resets it into its default
state) is still settled as above. An interrupted processing puts the task back,
untagged and with its previous assignee, into the incoming state it was
claimed from.

With `--type phase` and `--unclaimed`, an agent instead processes the
tasks *unclaimed*, like the built-in `estimator` agent: it observes the
tasks in one of the states of `--state-incoming` (each adapted onto the
task lifecycle model of the project, e.g., `PLANNED` as `PLANNING` for
`team` and as `OPEN` for `solo`, so it supports all models) which do
not carry its tag, independent of their `Assignee:` and their
predecessors. It processes each such task without any change of it
(neither moving it into an activity state, nor tagging it as active,
nor assigning it), so it keeps no other agent off the task, and the
skill itself has to save its result conditionally (as another agent can
claim and change the task meanwhile). Finally, it settles the task by
its tag only (according to `--tag-success` resp. `--tag-failure`),
without ever changing its status or its assignee. An interrupted
processing leaves the task untouched.

With `--type request`, an agent processes the *prompt requests*
attached to tasks in any lifecycle status (see `ase task request` in
`ase-task-request(1)`, or key `R` of the task boards), like the
built-in `editor` agent: it observes the tasks with pending requests
of the request name of `--request-name` (attachments of type
`text/plain; charset=utf-8; kind="<name>"`, mirrored in the tag
`ase-agent-<name>:pending`), claims the oldest request of such a task
by turning it into `kind="<name>-running"` and assigning the task to
the agent (remembering its previous assignee), processes it with the
request prompt available as `{prompt}`, and finally removes the
request on success, or turns it into `kind="<name>-failed"` (with the
reason of the failure as its description) on failure, resp. back into
`kind="<name>"` on interrupt, where the task keeps its status and its
tags (except the request state tag) and, by default, gets its previous
assignee back. An agent consumes the request prompts if its prompt
template references `{prompt}` (except within the conditional
`{?prompt:`*text*`}` only, which makes the prompt *optional*) or it
has a hook `prepare`, `prompt`, `model`, or `finish` (which receive
the request prompt in their *job*), where it then refuses a request
with an empty prompt as failed, else it accepts requests with an empty
prompt (which an agent with an optional prompt still passes on if
non-empty), and announces this to the agent relay, so the task boards
accept an empty prompt for its requests, too.

Only the prompt (and `--state-incoming` for `--type phase`) is
mandatory, while all other options default to the safe *annotate*
behavior: a `phase` task returns into its incoming state, tagged with
the plain tag `ase-agent-<name>` and unassigned, while a `request` task
keeps its status and tags and gets its previous assignee back. The
options are validated on the start (also before detaching the agent),
where an error names the offending option.

## COMMANDS

- `ase agent adhoc start`[- *options*]:
Start the ad-hoc agent named by- `--name`in the background (spawning
a detached process logging into- `.ase/agent/`- *name*- `.log`), or with- `--foreground`in the current process. The agent is validated and
initialized (e.g., its agent tool is looked up and the task lifecycle
model is checked against- `--lifecycle`) before the command returns,
so a failing agent is reported together with the tail of its log
(waiting up to 30 seconds for its startup).
 - An agent of the same name already running is left alone (and is an error
with - `--foreground`). All options are described under- **OPTIONS**below.
 
- `ase agent adhoc status`[- `-n`|- `--name`- *name*]:
Report whether the agent named- *name*(default:- `adhoc`) is running,
as- `agent`- *name*- `: running (pid`- *pid*- `)`resp.- `agent`- *name*- `: not running`. Exits with status 1 if it is not running.
 
- `ase agent adhoc log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*] [- `--name`- *name*]:
Show the log- `.ase/agent/`- *name*- `.log`of the agent named- *name*(default:- `adhoc`) run in the background (where an agent run in the
foreground logs to the terminal, and an agent of a fleet into the log
of the fleet instead, see- `ase-agent-fleet(1)`).
 - With - `-n`|- `--lines`- *n*, only the last- *n*lines are shown (default:
all). With- `-f`|- `--follow`, the last- *n*lines (default: 10) are shown
and then, in the style of- `tail -f`, all newly appended lines are
followed until interrupted. As- `-n`is the number of lines, the name is
given via the long option- `--name`only. If no log file exists, this is
just reported (at log level- `info`).
 
- `ase agent adhoc stop`[- `-n`|- `--name`- *name*]:
Stop the agent named- *name*(default:- `adhoc`) by sending it- `SIGTERM`, and wait (up to 5 minutes) until it aborted its running
processings and put their tasks back. A not running agent is reported
as such, without an error.
 
- `ase agent`- *kind*- `start`|- `status`|- `log`|- `stop`[...]:
The same commands for an agent of the definition kind- *kind*(see- **DESCRIPTION**), with the same options (except- `--definition`),
where the name defaults to- *kind*.
 

## OPTIONS

### Agent Options

The following options are shared by all agents (see also
`ase-agent(1)`):

- [- `-n`|- `--name`- *name*]:
The name of the agent (default: its kind, i.e.,- `adhoc`, resp. the
definition kind- *kind*), at most 64 letters, digits, underscores, and
dashes, starting with a letter or digit, and neither the name of
another agent kind, nor- `relay`or- `fleet`.
 - It names the agent in its log lines (also in the shared processing
log attachment - `agent`), the- `Assignee:`of the tasks it
processes (see- `--assign-name`), its default tag- `ase-agent-`- *name*(lower-cased, see- `--tag-name`), its default request name (see- `--request-name`), its temporary directory, and its process files- `.ase/agent/`- *name*- `.yaml`and- `.ase/agent/`- *name*- `.log`. Hence,
multiple ad-hoc agents are started under different names.
 
- [- `-t`|- `--tool`- *tool*]:
The agent tool processing the tasks:- `claude`,- `copilot`, or- `codex`(default:- `$ASE_TOOL`, else- `claude`). It is run headless (with- `ASE_HEADLESS=true`), i.e., via- `claude -p`,- `copilot -p`, resp.- `codex exec`(see- `ase-agent(1)`), and has to be found in- `$PATH`.
 
- [- `-p`|- `--parallel`- *num*]:
The maximum number of tasks processed in parallel, a positive
integer (default:- `1`). Ignored (with a warning) with- `--serial`.
 
- [- `-T`|- `--timeout`- *duration*]:
The maximum duration of processing a task, as- *number*- *unit*with- *unit*- `s`,- `m`, or- `h`(default:- `60m`), where a processing
exceeding it fails. The preparing and the finishing of a processing
are each limited to a fixed duration of 240 seconds in addition.
 
- [- `-o`|- `--one-shot`]:
Terminate the agent, instead of observing for further tasks, once a
check of the task store finds no dispatchable task while it
processes no task anymore. Intended for CI contexts. Not allowed in
a fleet. A failed check of the task store is retried after a growing
delay, where- `--one-shot`gives up after three failures in a row and
terminates with a non-zero exit status.
 
- [- `--agent-relay`- *url*]:
The agent relay to pass the live presence of the agent to, as- `ase://`- *addr*- `:`- *port*[- `/`- *token*] (HTTP) or- `ases://`- *addr*- `:`- *port*[- `/`- *token*][- `?insecure`] (HTTPS,
optionally without certificate verification) (default: the
configured- `project.relay.url`, else none). See- `ase-agent-relay(1)`.
 
- [- `-r`|- `--repo`- *url*]:
The Git repository to clone temporarily into the temporary directory
of the agent, for processing every task without a- `worktree`flow
value in a temporary Git worktree of this clone (default: the Git
working copy the agent is started in). Requires a task store not
located in the working copy (i.e., not a local- `ase:`- *path*one).
Silently ignored with- `--workdir project`.
 
- [- `-d`|- `--tempdir`- *dir*]:
The directory whose sub-directory- *name*holds the temporary files
of the agent, like the clone of- `--repo`, where- `%p`in- *dir*is
expanded to the process id of the agent,- `%h`to a hash value unique
to the process, and- `%%`to- `%`(default: a fresh, private directory
below the system temporary directory).
 - A missing - *dir*is created private to the current user, while an
existing- *dir*has to be a directory (no symbolic link) owned by the
current user. Silently ignored with- `--workdir project`.
 
- [- `-f`|- `--foreground`]:
Run the agent in the current process, logging to the terminal, until
it is interrupted or stopped, instead of detaching it.
 
- [- `-h`|- `--help`]:
Show the usage information of the command and exit.
 

### Definition Options

The following options form the *definition* of an agent. Each one is
a flat long option `--`*group*`-`*item* (except the ungrouped ones),
accepted by all agents (including the built-in ones), where the given
options override the definition of the agent (see `--definition` and
**DEFINITION FILES**). A *repeatable* option can be given multiple
times, each time adding a value.

A given option also replaces its mutually exclusive alternatives of the
definition: `--prompt-skill` replaces `--prompt-text` and
`--prompt-file`, `--prompt-text` and `--prompt-file` replace each other
and `--prompt-skill` with `--prompt-args`, and `--tag-retry` and
`--tag-failure` replace each other.

#### Ungrouped

- [- `-D`|- `--definition`- *file*|- `builtin:`- *kind*[- `-`- *mode*]]:
(- `ase agent adhoc`only) The definition to base the agent on: a
definition file (see- **DEFINITION FILES**), or, as- `builtin:`- *kind*- `-`- *mode*(resp.- `builtin:`- *kind*for its default
mode), the definition of a built-in agent (see- **DESCRIPTION**),
where all other given options override it. Example:- `ase agent adhoc start -n reviewer -D builtin:qualifier-qualify --tag-name ase-agent-reviewer`.
 
- [- `--title`- *text*]:
The one-line title of the operation of the agent (at most 80
characters), shown on the task boards: in the information line of the
agent logs view and above the description in its pane- `Operation`(default: none).
 
- [- `--description`- *text*]:
The brief description of the operation of the agent (about 3-6 lines,
at most 600 characters, with its whitespace collapsed), shown on the
task boards in the pane- `Operation`of the agent logs view, word-wrapped
onto its width (default: none).
 
- [- `--type`- *type*]:
The processing type:- `phase`(default) to claim tasks by their
lifecycle status, or- `request`to process the prompt requests
attached to tasks (see- **DESCRIPTION**). The options- `--unclaimed`,- `--state-`*,- `--tag-success`,- `--tag-failure`,- `--tag-retry`,- `--assign-required`, and- `--attach-skip`are allowed for- `phase`only, the options- `--request-`* for- `request`only.
 
- [- `--workdir`- *kind*]:
The working directory of the processing (default:- `project`), which
determines the working directory of the agent tool, the directories
granted to it (via- `--add-dir`), the holds and deferrals of the
tasks, and the values of the placeholders- `{workdir}`,- `{source-dir}`,
and- `{changeset-worktree}`.
 - The temporary worktrees the agent creates itself are named
- *id*- `-`- *assign-name*resp.- *id*- `-`- *changeset-name*- `-`- *assign-name*(with every- `#`of the task id replaced by- `_`), so they equal those
of the built-in agents if- `--assign-name`is set to the built-in
kind, while the worktree of a- `branch:`changeset is named after the
branch (with every character other than letters, digits,- `_`, and- `-`replaced by- `-`, runs of- `-`reduced to one, and leading and
trailing- `-`and- `_`removed), like by the skills:
 - - `project`: the project root.- `--repo`and- `--tempdir`are
silently ignored.
- `source`: the working copy holding the- `Source:`(the project
root, or with- `--repo`a worktree of its temporary clone), where a- `branch:`source is additionally checked out in a temporary
detached worktree- *id*- `-`- *assign-name*(as- `{source-dir}`,
granted to the agent tool) only if the prompt template references- `{source-dir}`(like- `editor`).
- `changeset`(- *write*access, like- `implementer`): a- `worktree`changeset in the project root, which, processed serially, is
held, deferred until free of uncommitted changes, and switched
onto a- `branch:`source, and, processed in parallel, yields the
ASE worktree id of the task (created by the skill) as- `{changeset-worktree}`; a- `branch:`changeset is held and checked
out in the agent worktree named after the branch (created from the
source if missing, refused if its directory already exists, else
reused if already checked out, but then required free of
uncommitted changes), yielding its ASE worktree
id (unless the project root) as- `{changeset-worktree}`, and
removed afterwards only if clean (else preserved); an- `attachment:`changeset is left to the skill.
- `changeset-read`(- *review*access, like- `qualifier`): a- `worktree`changeset staged in its context worktree, which becomes the
working directory; a- `branch:`changeset left in place (to be
addressed via- `{changeset-name}`); an- `attachment:`changeset
applied staged in the temporary worktree- *id*- `-`- *changeset-name*- `-`- *assign-name*, detached at its base
commit, which becomes the working directory.
- `target`(- *integration*access, like- `integrator`): the project
root, with access granted to the context worktree of a- `worktree`changeset (whose ASE worktree id becomes- `{changeset-worktree}`)
and to the worktree of a- `branch:`changeset, where a- `branch:`target is checked out in the temporary worktree- *id*- `-`- *assign-name*unless checked out already, published
afterwards if moved (or rescued if not publishable), and the
merged- `branch:`changeset is deleted from the origin.
 
- [- `--lifecycle`- *model*]:
(repeatable) A supported task lifecycle model,- `solo`,- `team`, or- `enterprise`(default: every model containing all given states, resp.
all models for- `request`and with- `--unclaimed`). The agent refuses
to start for a project of any other model. All given states have to
exist in all supported models (except with- `--unclaimed`, whose
incoming states are adapted onto the model of the project).
 
- [- `--serial`]:
Process the tasks serially only, ignoring- `--parallel`. A fleet
refuses two serially processing agents of the same tag.
 
- [- `--unclaimed`]:
(- `phase`only) Process the tasks- *unclaimed*(see- **DESCRIPTION**):
every task in one of the states of- `--state-incoming`(each adapted
onto the task lifecycle model of the project) which does not carry
the tag, independent of its assignee and its predecessors, without
claiming it, and without ever changing its status and its assignee
(the tag options still apply). An agent of the same kind on the same
host does not pick up a task already processed by another one.
 - Not allowed together with - `--state-activity`,- `--state-success`,- `--state-failure`,- `--state-phase`,- `--state-passed`,- `--state-verdict`,- `--assign-required`,- `--assign-success`,- `--assign-failure`, the success criterion- `status`, the hook- `passes`, and- `--workdir changeset`resp.- `target`(as the task can
be claimed by another agent meanwhile).
 
- [- `-M`|- `--model`- *model*]:
The model of the agent tool, passed to it as- `--model`- *model*verbatim (e.g., an alias like- `sonnet`or a full model name), either a- *model*for all tasks, or comma-separated- *size*- `=`- *model*pairs per- `Effort:`size of the task (- `XS`,- `S`,- `M`,- `L`,- `XL`, or- `*`for all
other tasks, e.g.,- `XS=haiku,S=haiku,M=sonnet,*=opus`), where a task
without a matching pair gets the default model of the agent tool (the
default for all tasks). The hook- `model`can override it per task (see- **HOOKS**).
 
- [- `--params`- *name*- `=`- *value*]:
(repeatable) The value of the parameter- *name*declared by the
definition (see- **PARAMETERS**), where a later value of the same
name wins. An undeclared- *name*is refused.
 

#### State (`--type phase` only)

All states are case-insensitive (upper-cased), have to exist in the
supported task lifecycle models, and the transitions from the incoming
into the activity state, back (on interrupt), and from the activity
state into all target states have to be reachable in these models.
With `--unclaimed`, only `--state-incoming` is allowed instead, whose
states have to exist in any task lifecycle model only.

- [- `--state-incoming`- *state*]:
The state of the tasks to claim. Mandatory for- `phase`. Repeatable
with- `--unclaimed`, where each state is adapted onto the task
lifecycle model of the project (a state of the model is kept, any
other one is mapped onto its counterpart, e.g.,- `PLANNED`onto- `PLANNING`for- `team`and onto- `OPEN`for- `solo`), and with an
explicit- `--state-activity`, where a task is claimed from any of the
states (e.g.,- `DRAFTED`and- `PLANNING`into- `PLANNING`).
 
- [- `--state-activity`- *state*]:
The state of a task during its processing (default: the first
incoming state, i.e., the claim is marked by the- `:active`tag value
only).
 
- [- `--state-success`- *state*]:
The target state on success, or- `previous`for the incoming state
the task was claimed from (default: the first incoming state).
 
- [- `--state-failure`- *state*]:
The target state on failure, or- `previous`for the incoming state
the task was claimed from (default: the first incoming state).
 
- [- `--state-phase`- *state*]:
The- *phase success state*, decoupled from the target state: a task
the skill itself moves there counts as processed (see the success
criterion- `status`), and a task there carrying the tag counts as
having passed the phase for its- `After:`successors (default: the
success state, resp. the activity state for a- `previous`success
state).
 
- [- `--state-passed`- *state*]:
The first state of the lifecycle model from which on a task has
passed the phase for its- `After:`successors (default: the phase
state).
 
- [- `--state-verdict`- *VERDICT*- `=`- *STATE*]:
(repeatable) The target state per verdict (both upper-case words),
overriding the success target state, but never the failure target
state of a failed processing. Requires- `--verdict-label`(or the hook- `verdict`), and every verdict has to be accepted by- `--verdict-allow`(if given).
 

#### Tag

- [- `--tag-name`- *tag*]:
The tag of the processed tasks, of the form- `ase-agent-`- *word*in
lower-case and without- `:`(default:- `ase-agent-`- *name*, hence no
sharing between differently named agents). It guards the tasks
against a repeated processing: a task carrying it (plain or with any
value) is not claimed again. For- `request`, it is always the request
state tag- `ase-agent-`- *request-name*(resp.- `ase-agent-editor`for the
request name- `edit`), and any other value is refused.
 
- [- `--tag-success`- *tag*]:
(- `phase`only) The tag set on success:- *tag-name*(default, the
plain tag),- *tag-name*- `:`- *value*(a lower-case- *value*other than- `active`), or- `none`to untag the task. A foreign tag is refused, as
the agent would claim the task again.- `none`is refused if a task
returns into an incoming state on success (unless guarded by- `--attach-skip`, by the hook- `ready`of a definition which itself
declares- `tag-success: none`, like the built-in- `estimator`, as a hook- `ready`inherited from a definition without it, like the one of the
built-in- `implementer`, does not guard against claiming the untagged
task again, or by- `--assign-required`with an- `--assign-success`handing the task over to somebody else, like the built-in- `rebooter`), e.g., always with- `--unclaimed`. A found verdict is
stored as the tag value instead.
 
- [- `--tag-failure`- *tag*]:
(- `phase`only) The tag set on failure, like- `--tag-success`(default:- *tag-name*), e.g.,- *tag-name*- `:failed`to tell failures apart.
 
- [- `--tag-require`- *tag*]:
(repeatable) A tag a claimable task has to carry, where a plain tag
also matches the tag with any value.
 
- [- `--tag-exclude`- *tag*]:
(repeatable) A tag a claimable task must not carry, where a plain tag
also matches the tag with any value.
 
- [- `--tag-retry`]:
(- `phase`only) A shorthand for- `--tag-failure none`(not allowed
together with it): a failed task returning into the incoming state is
claimed again, but held off by the agent process for an exponentially
growing delay from 1 minute up to 1 hour.
 

The tag options never apply to an interrupted task (released untagged)
or to a task left for a manual intervention (tagged
*tag-name*`:failed`, e.g., after a failed finishing or an unreachable
resulting state), which does not count as passed for its successors.

#### Assign

- [- `--assign-name`- *who*]:
The assignee under which the agent claims, holds, recognizes, and
releases the tasks, and which names its agent-owned worktrees and its
mode as announced to the agent relay (default: the agent name), a
name of letters, digits,- `_`,- `.`,- `@`, and- `-`, starting with a
letter or digit. So a configured agent can act as, e.g.,- `rebooter`,
although its agent name must not clash with a built-in kind.
 
- [- `--assign-required`]:
(- `phase`only) Claim only the tasks explicitly assigned to- *who*of- `--assign-name`(like- `rebooter`), instead of the unassigned ones,
too.
 
- [- `--assign-success`- *who*]:
The assignee to hand a task over to after its successful processing:
a name,- `none`to unassign it, or- `previous`for the assignee before
the claim (default:- `none`for- `phase`,- `previous`for- `request`).
Applied only while the task is still assigned to the agent.
 
- [- `--assign-failure`- *who*]:
The assignee to hand a task over to after its failed processing,
like- `--assign-success`(same defaults). An interrupted task always
gets its previous assignee back, and a task left for a manual
intervention is always unassigned.
 

#### Request (`--type request` only)

- [- `--request-name`- *name*]:
The name of the processed requests, i.e., of the attachments- `kind="<name>"`,- `kind="<name>-running"`, and- `kind="<name>-failed"`(default: the agent name), a lower-case word of letters, digits, and
dashes, not ending in- `-running`or- `-failed`, and none of the
built-in agent kinds. The name- `edit`processes the edit requests of
key- `E`of the task boards.
 
- [- `--request-done`- *word*]:
The word naming a processed task in the log lines (default:- `processed`, e.g.,- `edited`).
 

#### Prompt

Exactly one of `--prompt-skill`, `--prompt-text`, or `--prompt-file`
is required. Flags of the skills like `--next DONE` are never added
implicitly, but always have to be given explicitly.

- [- `--prompt-skill`- *name*]:
The skill to invoke, as- *skill*or- *plugin*- `:`- *skill*(letters,
digits,- `_`,- `.`, and- `-`).
 - It is invoked like the built-in agents do, i.e., as - `/`- *skill*(- `claude`and- `copilot`) resp.- `$`- *skill*(- `codex`), followed by a
single space and the rendered arguments of- `--prompt-args`only if
these are non-empty, where an un-namespaced ASE skill (- `ase-*`) gets
the plugin namespace- `ase:`for- `claude`only, a- *plugin*- `:`- *skill*stays verbatim, and any other name (e.g., a project skill generated by- `ase-meta-workflow`) stays un-namespaced.
 
- [- `--prompt-args`- *template*]:
The arguments of the skill, as a template (see- **TEMPLATES**)
(default:- `{id}`for- `phase`,- `{id}: {prompt}`for- `request`).
Allowed with- `--prompt-skill`only.
 
- [- `--prompt-text`- *template*]:
The prompt text to pass as-is to the agent tool (instead of a skill),
as a template (see- **TEMPLATES**).
 
- [- `--prompt-file`- *file*]:
A file with the prompt text (instead of a skill), whose content is a
template (see- **TEMPLATES**). It is read and validated on the start.
In a definition file, a relative- *file*is relative to the
definition file.
 

#### Prepare

- [- `--prepare-run`- *cmd*]:
(repeatable) A shell command run in the working directory before the
prompt (e.g.,- `npm ci`), where a failing command fails the processing
without running the prompt.
 - It gets the environment variables - `ASE_TASK_ID`,- `ASE_TASK_TITLE`,- `ASE_TASK_SOURCE`,- `ASE_TASK_CHANGESET`,- `ASE_TASK_TARGET`(the raw
flow values or their configured defaults),- `ASE_AGENT_NAME`,- `ASE_AGENT_WORKDIR`, and- `ASE_AGENT_PARAM_`- *NAME*per parameter (see- **PARAMETERS**). Empty commands are refused.
 

#### Success

- [- `--success-criterion`- *criterion*]:
(repeatable) A success criterion, where all given criteria have to
hold, replacing the default- `exit`and- `clean`for- `phase`, resp.- `clean`and- `changed`for- `request`:
 - - `exit`: an exit code 0 of the agent tool without timeout or
interrupt.
- `clean`: no output line reporting an error of the skill (a line
starting with- `ERROR:`or containing- `▶ ERROR:`).
- `changed`: (- `request`only) a plan content changed since the
claim, hence also succeeding on an interrupt after the plan was
saved (like- `editor`).
- `verdict`: a found verdict. Requires- `--verdict-label`(or the
hook- `verdict`).
- `attachment`: an attachment of- `--attach-expect`created or
changed by the processing (a leftover of an earlier processing
does not count). Requires- `--attach-expect`.
- `status`: (- `phase`only) the move into the phase success state
(see- `--state-phase`) by the skill itself.
 
- [- `--success-match`- *regex*]:
(repeatable) A regular expression (JavaScript syntax) an output line
has to match, e.g.,- `status:\W*plan integrated\b`.
 

An unmet criterion is logged, and a failed request is described in its
`<name>-failed` attachment by the first applicable reason: a failed
preparing or finishing command, `skill reported an error`, `agent tool failed`, else the reason of the first unmet criterion (e.g., `plan not changed`, `no verdict`, `no new or changed attachment "<spec>"`, or
`no output line matching "<regex>"`).

#### Verdict

- [- `--verdict-label`- *label*]:
The label after which the verdict is given in the output of a
successfully exited agent tool: the verdict is the upper-case word
following the label (after any non-word characters) on the last
output line carrying it, e.g.,- `APPROVE`for the label- `REVIEW VERDICT`and the line- `**REVIEW VERDICT**: **APPROVE**`. The hook- `verdict`can
determine it instead.
 - A verdict is stored lower-cased as the tag value (e.g.,
- `ase-agent-qualifier:approve`, instead of- `--tag-success`) and mapped to
a target state via- `--state-verdict`. It is honored on success only: on
a failed processing, the task is settled by- `--tag-failure`and- `--state-failure`instead.
 
- [- `--verdict-allow`- *VERDICT*]:
(repeatable) An accepted verdict, an upper-case word (default: the
verdicts of- `--state-verdict`, else any). A verdict not accepted
counts as none. Requires- `--verdict-label`(or the hook- `verdict`).
 

#### Attach

An attachment *spec* is *kind* or *mime-type*`;`*kind* (e.g.,
`text/x-diff;draft`), matching the `kind` parameter of the attachment
type exactly and, if given, its MIME type.

- [- `--attach-output`- *kind*]:
Store the output lines of the prompt as the task attachment of type- `text/plain; charset=utf-8; kind="<kind>"`, replacing the one of an
earlier processing.
 
- [- `--attach-expect`- *spec*]:
The attachment the success criterion- `attachment`requires, which
must not match the attachment of- `--attach-output`(as this one is
always created).
 
- [- `--attach-skip`- *spec*]:
(- `phase`only) Leave every task already carrying such an attachment
unclaimed (e.g., a task already carrying a draft).
 
- [- `--attach-changeset`- *kind*]:
(with- `--workdir changeset`or- `target`only) Store the- *change set
snapshot*-- the view-only, text-only unified diff of the change set
(without the- `.ase`directory) -- as the task attachment of type- `text/x-diff; charset=utf-8; kind="<kind>"`, captured by the agent
itself after the processing (before- `--finish-commit`) and replacing
the one of an earlier processing in place (keeping its- `Created:`key), like- `implementer`and- `integrator`with the kind- `changeset`.
 - With - `--workdir changeset`, it covers the change set of a- `worktree`or- `branch:`changeset from its fork point from the- `Source:`on, with- `base`- *commit*(plus- `, partial`unless the task reached- `--state-phase`) in its- `Desc:`key, while an- `attachment:`changeset
other than- `attachment:`- *kind*removes it as obsolete.
 - With - `--workdir target`, it covers the change of the target branch by
the integration of a- `worktree`or- `branch:`changeset, with- `base`- *commit*- `, merged`- *commit*in its- `Desc:`key, unless the task did
not reach- `--state-phase`or the content of the target did not change.
Without a change set, the task stays untouched.
 

#### Finish

- [- `--finish-commit`]:
(with- `--workdir changeset`only) Commit the work onto a- `branch:`changeset with the message- *Kind*- `:`- *title*- `(`- *id*- `)`and publish
it, rescuing a branch which could not be published, like- `implementer`.
 
- [- `--finish-success`- *cmd*]:
A shell command run in the working directory after a successful
processing, with the environment variables of- `--prepare-run`plus- `ASE_TASK_STATUS`, where a failing command fails the finishing.
 
- [- `--finish-failure`- *cmd*]:
A shell command run in the working directory after a failed
processing, with the same environment variables, where a failing
command is only reported.
 

The preparing and finishing commands are limited by the fixed duration
of 240 seconds of the preparing resp. finishing of a processing.

## TEMPLATES

The templates of `--prompt-args`, `--prompt-text`, and `--prompt-file`
expand the following placeholders:

- `{id}`,- `{title}`:
The id and the title of the task.
 
- `{prompt}`:
The request prompt, with its whitespace collapsed into single spaces
(- `request`only).
 
- `{source}`,- `{changeset}`,- `{target}`:
The raw flow values of the frontmatter keys- `Source:`,- `Changeset:`,
and- `Target:`, or their configured defaults- `project.task.default.*`, with their placeholder- `{id}`replaced by
the task id (with every character other than letters, digits,- `_`,
and- `-`replaced by- `-`, runs of- `-`reduced to one, and leading and
trailing- `-`and- `_`removed, e.g.,- `12`for- `#12`and- `FOO-12`for- `FOO#12`).
 
- `{source-type}`,- `{source-name}`,- `{changeset-type}`,- `{changeset-name}`,- `{target-type}`,- `{target-name}`:
The parsed flow values: the type- `worktree`,- `branch`, or- `attachment`, and the branch resp. attachment name, else empty, where
a- `Target:`of- `source`reads as the- `Source:`.
 
- `{workdir}`:
The working directory of the processing (see- `--workdir`).
 
- `{source-dir}`:
The temporary worktree of a- `branch:`source, else empty.
 
- `{changeset-worktree}`:
The ASE worktree id under which the skill finds the changeset, else
empty.
 
- `{key:`- *Header*- `}`:
A frontmatter key- *Header*of the task.
 
- `{param:`- *name*- `}`:
The value of the parameter- *name*declared by the definition (see- **PARAMETERS**).
 
- `{?`- *cond*- `:`- *text*- `}`:
A conditional, which emits- *text*(with placeholders, but without
nested conditionals) only if- *cond*holds, either- *placeholder*(the
placeholder is non-empty) or- *placeholder*- `=`- *value*(the placeholder
equals- *value*), e.g.,- `{?source-dir: --int-source-dir "{source-dir}"}`or- `{?changeset-type=branch:--branch {changeset-name}}`. A- `{prompt}`used only within- `{?prompt:`- *text*- `}`makes the prompt of the requests optional, e.g.,- `{id}:{?prompt: {prompt}}`.
 

All other text, including whitespace, is kept verbatim, `{{` and `}}`
yield literal braces, and an unknown placeholder (or a malformed or
unbalanced one) fails the start.

## DEFINITION FILES

A definition file carries the same keys as the definition options
above (without the leading `--`), either flat (`state-incoming: PLANNED`) or nested by group (`state: { incoming: PLANNED }`), where a
key must not be given both flat and nested, a list value repeats a
repeatable option, a flag takes a boolean, and a relative
`prompt-file` is relative to the definition file. A definition file is
either:

- a YAML file (- `.yaml`or- `.yml`), declarative only, or
 
- a TypeScript file (- `.ts`or- `.mts`, run via the type stripping of
Node.js) or a JavaScript file (- `.js`,- `.mjs`, or- `.cjs`), whose
default export is the definition object or a (possibly asynchronous)
function returning it, where- `.mts`resp.- `.mjs`avoid the reparsing
warning of Node.js outside of a package of- `"type": "module"`.
 - It is typed via the type - `AgentDefinition`of the package- `@rse/ase`(- `import type { AgentDefinition } from "@rse/ase"`and- `export default { ... } satisfies AgentDefinition`, with- `@rse/ase`as a development dependency for the editor only, as the type import is
erased). Only such a file can additionally carry- *hooks*(its key- `hooks`, see- **HOOKS**).
 

Every definition file can additionally declare *parameters* (its key
`params`, see **PARAMETERS**).

A definition file can extend a *base definition* via its key
`definition`: a further definition file (relative to the definition
file) or, as `builtin:`*kind*`-`*mode* (resp. `builtin:`*kind*), the
definition of a built-in agent.

Its options override those of the base definition (also replacing their
mutually exclusive alternatives, and with a false flag unsetting the
flag of the base definition), its parameter declarations extend those
of the same name of the base definition field-wise (e.g.,
`params: { severity: LOW }` changes just the default value), and its
hooks replace those of the same name of the base definition. Base
definitions can be chained arbitrarily deep, but never cyclic.

For example, a `.ase/agents/implementer.mts` of
`definition: "builtin:implementer-implement"` plus a `ready` hook shadows
the built-in `implementer` while keeping its definition and its `passes`
hook, and a `.ase/agents/reviewer.yaml` of
`definition: builtin:qualifier-judge` plus its own
`tag: { name: ase-agent-reviewer }` runs a further judging agent of its
own tag.

## PARAMETERS

A definition can declare *parameters* (its key `params`, typed via
`AgentParam` of the package `@rse/ase`): custom options of the agent,
whose values are given on its start via `--params` *name*`=`*value*
(resp. via the map `params` of its section of a fleet configuration,
see `ase-agent-fleet(1)`). A parameter *name* consists of up to 64
letters, digits, underscores, and dashes, starting with a letter, and
is declared by an object of the optional fields:

- `desc`:
The description of the parameter, cited in its errors.
 
- `default`:
The default value of the parameter (else empty).
 
- `required`:
Whether a non-empty value is required (default:- `false`).
 
- `allow`:
The allowed values (a single value or a list), where an empty value
is always allowed unless- `required`.
 

A plain scalar instead of the object declares just the default value
(e.g., `params: { severity: HIGH }`). The values of all parameters are
validated on the start, where an undeclared parameter, a missing
required value, and a value not allowed are refused. Each declared
parameter provides its value (empty if neither given nor defaulted) as:

- the placeholder - `{param:`- *name*- `}`of the templates (see- **TEMPLATES**),
also usable in a conditional like- `{?param:`- *name*- `:`- *text*- `}`;
 
- `ctx.params.`- *name*(resp.- `ctx.params["`- *name*- `"]`) of the hooks (see- **HOOKS**);
 
- the environment variable - `ASE_AGENT_PARAM_`- *NAME*of the commands of- `--prepare-run`,- `--finish-success`, and- `--finish-failure`, with- *NAME*the upper-cased- *name*with every- `-`replaced by- `_`, where
two parameters of the same such variable are refused.
 

## HOOKS

The *hooks* of a TypeScript or JavaScript definition file (its key
`hooks`, typed via `AgentHooks` of the package `@rse/ase`) are the
code-based extensions beyond the declarative options, which not only gate,
but also shape the processing. A thrown error never terminates a running
agent, but counts like a negative result with the error as its reason.

The hooks receive the *task* (its `id`, `title`, `status` adapted to the
task lifecycle model of the project, its flattened frontmatter `keys`,
and its `tags`) and a context *ctx*, which provides:

- `agent`and- `lifecycle`:
The agent (its- `name`,- `kind`,- `mode`, and- `tag`) and the task
lifecycle model of the project (its- `name`and its- `states`).
 
- `params`:
The values of all declared parameters, by their names (see- **PARAMETERS**).
 
- `signal`:
An- `AbortSignal`, aborted on the interrupt of the agent (and, within
the hooks of a job, on the timeout of its preparing resp. finishing),
which a long-running hook should honor.
 
- `flow(task, key)`:
The parsed- `Source`,- `Changeset`, or- `Target`flow value (- `type`and- `name`) of the task, or its configured default (where a- `Target:`of- `source`reads as the- `Source:`), else- `null`if invalid.
 
- `attachments(task)`,- `plan(task)`,- `task(id)`,- `tasks()`:
The attachments of a task (their- `type`,- `desc`, and inline- `data`),
the plan body of a task (else- `null`), a task by its id (else- `null`), and all tasks, where- *task*is a task or its id.
 
- `update(task, change)`:
A change of a task in a single conditional update (retried on a
concurrent change, resolving to- `false`if the task vanished):- `keys`to set (or to remove with- `null`), except- `Status`,- `Assignee`,- `Tags`, and- `Modified`;- `tags`to- `add`(- *tag*or- *tag*- `:`- *value*,
replacing the other occurrences of- *tag*) and to- `remove`(- *tag*with
any value, or exactly- *tag*- `:`- *value*), except the tag of the agent
itself; and- `attach`attachments (- `kind`,- `mime`(default:- `text/plain`),- `desc`,- `data`, replacing those of the same kind unless- `replace: false`), except the processing logs and the processed
requests of the agent.
 
- `exec(cmd, args, options)`:
The run of a command without a shell (within the hooks of a job in
the working directory of the job, else in the current directory, or
in- `options.cwd`, with the additional environment variables- `options.env`and the standard input- `options.input`), limited to 240
seconds and cancelled by- `signal`, resolving to its exit- `code`(or- `-1`if it did not exit regularly),- `stdout`, and- `stderr`.
 
- `log(level, message)`:
A log line of level- `error`,- `warning`, or- `info`(within the hooks of
a job into the processing log of the task, else into the log of the
agent).
 

The hooks of a job additionally receive the *job*: its working
directory `dir`, the additional directories `dirs` granted to the agent
tool, the temporary worktree `sourceDir` of a `branch:` source, the ASE
worktree id `changesetWorktree` of the changeset, and, for `request`,
the raw `request` prompt. The hooks are, in the order of their
invocation:

- `start(ctx)`:
The initialization of the agent (e.g., opening a connection) on its
start, where a thrown error refuses the start.
 
- `order(tasks, ctx)`:
The prioritization of the dispatchable tasks (given in natural id
order) on each dispatching: the tasks (or their ids) in the order to
claim them, where an omitted task is not claimed this time (a failed
hook keeps the natural id order and is warned about once per change).
 
- `passes(pred, succ, ctx)`:
(- `phase`without- `--unclaimed`only, synchronous) Whether the
predecessor- *pred*passed the phase of the agent for its successor- *succ*, or- `undefined`for the default decision by- `--state-phase`and- `--state-passed`(a failed hook lets the successor wait and is
warned about once).
 
- `ready(task, ctx)`:
Whether a dispatchable task is claimed now, checked right before its
claim (after- `--attach-skip`):- `true`to claim it,- `false`to
silently skip it (e.g., as the result of an earlier processing
already guards it, like the valid- `Effort:`key for- `estimator`), or
the reason to wait for (reported once per change of it).
 
- `prepare(task, job, ctx)`:
The preparing of the job in its working directory, after the commands
of- `--prepare-run`, where a thrown error fails the processing without
invoking the agent tool.
 
- `prompt(task, prompt, job, ctx)`:
The prompt of the job, given the configured- *prompt*rendered from
its template (- `{ skill, args }`or- `{ text }`):- `{ skill, args }`,- `{ text }`, a plain prompt text, or- `undefined`to keep the
configured one, where a thrown error fails the processing.
 
- `model(task, model, job, ctx)`:
The model of the agent tool for the job, given the configured- *model*of- `--model`for the- `Effort:`size of the task (empty for the default
model of the agent tool): a model (empty for the default one), or- `undefined`to keep the configured one, where a thrown error fails the
processing. This allows choosing the model per task by any means,
e.g., by asking a cheap model via- `ctx.exec`.
 
- `verdict(task, result, ctx)`:
(after a successful agent tool only) The verdict of the result (an
upper-case word, accepted only if listed by- `--verdict-allow`), an
empty string for none, or- `undefined`to keep the one found after- `--verdict-label`(if any). This hook can stand in for- `--verdict-label`(also for the criterion- `verdict`and for- `--state-verdict`).
 
- `success(task, result, ctx)`:
An additional success criterion after all others (where- *result*carries- `ok`, the- `output`lines, and the found- `verdict`):- `true`if
met, else- `false`or the reason why it is not met.
 
- `settle(task, outcome, ctx)`:
The settling of the processed task (where- *outcome*carries its- `success`, the- `reasons`of the unmet success criteria, and the- `result`), overriding the configured one: its target- `state`(- `phase`without- `--unclaimed`only, still required to be reachable from the
activity state), the value of its- `tag`(- `phase`only, a lower-case
value, or- `null`to untag it, refused if the task would return into
an incoming state unguarded by- `--attach-skip`, the hook- `ready`, or
the handover of a required assignment, see- `--tag-success`),
and its- `assignee`(a name,- `none`, or- `previous`, but not with- `--unclaimed`), where an omitted field (or- `undefined`for all) keeps
the configured settling, and a failed hook keeps the configured
settling (and is warned about).
 
- `finish(task, outcome, job, ctx)`:
The finishing of the job in its working directory, after the
finishing of its work (e.g., the commit of- `--finish-commit`) and
before- `--finish-success`resp.- `--finish-failure`, where a thrown
error fails the finishing of a successful processing (like a failed- `--finish-success`), but is only reported for a failed one.
 
- `stop(ctx)`:
The teardown of the agent (e.g., closing a connection) on its
termination, with a fresh- `signal`(a failed hook is only reported).
 

## AGENT RELAY AND RUNS

An agent of a definition kind resp. an ad-hoc agent announces itself
to the agent relay (see `ase-agent-relay(1)`) with its kind (resp.
`adhoc`), its `--assign-name` as its mode, and, for `request`, its
request name (suggested by key `R` of the task boards). Its runs are
recorded per tag (below `agent-tag-`*word*`/`*store-hash* of the
*per-user state directory* for the tag `ase-agent-`*word*), so the
recoveries of differently configured agents never affect each other.

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`.
 
- `ASE_AGENT_RELAY_TOKEN`:
The bearer token of the agent relay, if not embedded in its URL.
 
- `ASE_HEADLESS`:
Set to- `true`for the agent tool processes spawned by the agent.
 
- `ASE_TASK_ID`,- `ASE_TASK_TITLE`,- `ASE_TASK_SOURCE`,- `ASE_TASK_CHANGESET`,- `ASE_TASK_TARGET`,- `ASE_AGENT_NAME`,- `ASE_AGENT_WORKDIR`,- `ASE_AGENT_PARAM_`- *NAME*,- `ASE_TASK_STATUS`:
Set for the commands of- `--prepare-run`,- `--finish-success`, and- `--finish-failure`(the latter only for the finishing commands).
 

## FILES

- `.ase/agents/`- *kind*- `.`- *ext*:
The definition files of the definition kinds of the project.
 
- *per-user configuration directory*- `/agents/`- *kind*- `.`- *ext*:
The definition files of the definition kinds of the user.
 
- `.ase/agent/`- *name*- `.yaml`:
The process state (process id and process start time) of the agent
named- *name*, existing while the agent runs.
 
- `.ase/agent/`- *name*- `.log`:
The log of the agent named- *name*run in the background.
 
- *per-user state directory*- `/agent-tag-`- *word*- `/`- *store-hash*- `/`- *run-id*- `.lock`and- `.json`:
The run lock and run record of each agent process of the tag- `ase-agent-`- *word*, used to recover the tasks of terminated agent
processes.
 

## EXIT STATUS

`start` and `stop` exit with status 0 on success (also for an agent
already running resp. not running) and 1 on any error. `status` exits
with status 1 if the agent is not running. `log` exits with status 0.

## EXAMPLES

```
# a qualification agent deciding on the task status by its verdict
$ ase agent adhoc start -n reviewer --state-incoming IMPLEMENTED \
    --state-activity QUALIFYING --state-phase QUALIFIED --workdir changeset-read \
    --prompt-skill ase-repo-review --prompt-args "{?changeset-type=branch:--branch {changeset-name}}" \
    --verdict-label "REVIEW VERDICT" --state-verdict APPROVE=QUALIFIED --state-verdict REJECT=DECLINED \
    --success-criterion exit --success-criterion verdict
# a request agent attaching a lint report (requested via "ase task request <id> lint <prompt>")
$ ase agent adhoc start -n linter --type request --request-name lint --workdir source \
    --prompt-text "Lint the source code changed by task {id} with focus on: {prompt}" \
    --success-criterion exit --success-criterion clean --attach-output lint-report
# an agent running a project skill generated by "ase-meta-workflow" on the planned tasks
$ ase agent adhoc start -n workflow --state-incoming PLANNED --tag-require workflow \
    --prompt-skill my-workflow --prompt-args "{id}"
# a further qualification agent of its own tag, based on a built-in definition
$ ase agent adhoc start -n reviewer -D builtin:qualifier-qualify --tag-name ase-agent-reviewer
# an agent of a definition kind of the project (see .ase/agents/reviewer.mts below)
$ ase agent reviewer start -p 2
$ ase agent reviewer log -f
```

A YAML definition file `.ase/agents/linter.yaml`, yielding the
definition kind `linter`:

```
type:               request
workdir:            source
request:
    name:           lint
prompt:
    text:           "Lint the source code changed by task {id} with focus on: {prompt}"
success:
    criterion:      [ exit, clean ]
attach:
    output:         lint-report
```

A TypeScript definition file `.ase/agents/reviewer.mts` with hooks,
yielding the definition kind `reviewer`:

```
import type { AgentDefinition } from "@rse/ase"
export default {
    type:      "phase",
    lifecycle: "enterprise",
    workdir:   "changeset-read",
    state:     { incoming: "IMPLEMENTED", activity: "QUALIFYING", phase: "QUALIFIED" },
    tag:       { name: "ase-agent-reviewer", require: "security" },
    prompt:    { skill: "ase-repo-review", args: "{?changeset-type=branch:--branch {changeset-name}}" },
    verdict:   { label: "REVIEW VERDICT", allow: [ "APPROVE", "REJECT" ] },
    success:   { criterion: [ "exit", "verdict" ] },
    params: {
        severity: { desc: "severity of the findings a rejecting review has to cite", default: "HIGH", allow: [ "LOW", "MEDIUM", "HIGH" ] }
    },
    hooks: {
        /*  review a task only once its changeset is a branch  */
        ready: (task, ctx) =>
            ctx.flow(task, "Changeset")?.type === "branch" ? true : "a \"branch:\" changeset",
        /*  require a rejecting review to cite at least one finding of the configured severity  */
        success: (_task, result, ctx) =>
            result.verdict !== "REJECT" || result.output.some((line) => new RegExp(`\\b${ctx.params.severity}\\b`).test(line)) ||
            `rejecting review without ${ctx.params.severity} findings`
    }
} satisfies AgentDefinition
```

Started via `ase agent reviewer start --params severity=MEDIUM` (resp. with
`params: { severity: MEDIUM }` in its section of a fleet configuration).

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-rebooter(1)`,
`ase-agent-condenser(1)`, `ase-agent-researcher(1)`,
`ase-agent-implementer(1)`, `ase-agent-qualifier(1)`,
`ase-agent-integrator(1)`, `ase-agent-editor(1)`,
`ase-agent-estimator(1)`, `ase-agent-relay(1)`, `ase-agent-fleet(1)`,
`ase-task-request(1)`, `ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-relay(1) - Manage the Agent Relay Server

## NAME

`ase-agent-relay(1)` - Manage the Agent Relay Server

## SYNOPSIS

`ase agent relay start`
[`-a`|`--address` *host*]
[`-p`|`--port` *port*]
[`-t`|`--token` *token*]
[`-c`|`--cors` *origin*]
[`--tls-cert` *file* `--tls-key` *file*]
[`-f`|`--foreground`]

`ase agent relay status`

`ase agent relay log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]

`ase agent relay stop`

## DESCRIPTION

`ase agent relay` manages the per-user *agent relay* server, which
passes the live presence of the agents (see `ase-agent(1)`) to the
task boards (see `ase-task-board(1)`) without any external
infrastructure. It can run locally or on a central host, as long as
both the agents and the task boards reach it, and serves any number of
projects (by their `project.id`).

The agent relay holds the presence of the agents in memory only: per
agent its kind, name, mode, the title and description of its operation
(for the agent logs view of `ase-task-board(1)`), request name, the lifecycle states it
operates on (of a phase agent: its incoming, activity, written, and
gate states, for the agent graph of `ase-task-board(1)`), host,
process id, current working directory, user name and user id, start time, state (`waiting` or `busy`), the ids of the
tasks it is busy with, its usage of the agent tool (in total and by its
last job), the time since which it is in its state, the time of its
latest announcement (its heartbeat), and its most recent 100
processing log lines (each message truncated to 500 characters).

An agent without heartbeat for 30 seconds is removed (checked every 5
seconds), and a project exists only while it has agents.

The agents announce their presence (see `--agent-relay` of
`ase-agent(1)`) on every change of their state, after each finished
job, with their new processing log lines, and as a heartbeat every 10
seconds, and remove it on their termination. The task boards subscribe
to the agents of their project.

Alternatively, `ase task store start --with-agent-relay` (see
`ase-task-store(1)`) co-hosts the agent relay in the task store server
under the same address, port, and token, so a central host needs a
single server process only. A fleet with the section `relay` owns the
agent relay instead (see `ase-agent-fleet(1)`).

The agent relay is selected by the agents and task boards via the
configuration `project.relay.url` (or their option `--agent-relay`),
as `ase://`*addr*`:`*port*[`/`*token*] for HTTP, resp.
`ases://`*addr*`:`*port*[`/`*token*][`?insecure`] for HTTPS
(verifying the server certificate against the Node.js CA store,
extendable via `$NODE_EXTRA_CA_CERTS`, or, with `?insecure`, without
certificate verification). The relay requires an
explicit `project.id`.

The bearer token of a client is the one embedded in the URL (warned about
if configured on the `project` or a `task` scope, as it might be
committed), else `$ASE_AGENT_RELAY_TOKEN`, else `project.relay.token`
(intended for the `user` scope, and warned about on any other scope),
else the token of the locally started agent relay (resp. of the locally
started task store co-hosting it), but only if the URL addresses exactly
this server. Sending a token to a non-loopback host selected by a
repository-supplied scope is warned about.

## COMMANDS

- `ase agent relay start`[- *options*]:
Start the agent relay server in the background (spawning a detached
process logging into- `relay.log`) and wait until it responds (up to
10 seconds), or with- `--foreground`run it in the current process.
 - The configuration of the server is resolved from the options, the
environment, and the per-user - `relay.yaml`, and its effective values
(address, port, token, CORS origins, TLS certificate and key, and
process id) are persisted in- `relay.yaml`(readable by the current
user only). The start is idempotent: an already running server is left
alone, unless an explicitly given option (- `--address`,- `--port`,- `--token`,- `--cors`,- `--tls-cert`, or- `--tls-key`) deviates from its
configuration, in which case the server is restarted to apply it.
 - With - `--foreground`, an already running server with an unchanged
configuration is an error. A configured port held by a foreign service
is re-allocated (with a warning). Binding to a non-loopback address
without TLS is warned about, as the bearer token then travels
unencrypted. All options are described under- **OPTIONS**below.
 
- `ase agent relay status`:
Report whether the agent relay server is running, as- `relay: running on`- *url*- `(pid`- *pid*- `)`, after probing it on its configured address
and port with its configured token. Exits with status 1 if it is not
running (no port configured, the port not responding, or the port in
use by a foreign service), or if it rejects the configured token.
 
- `ase agent relay log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]:
Show the log- `relay.log`of the agent relay server run in the
background. With- `-n`|- `--lines`- *n*, only the last- *n*lines are shown
(default: all). With- `-f`|- `--follow`, the last- *n*lines (default:
 - - are shown and then, in the style of `tail -f`, all newly appended
lines are followed until interrupted. If no log file exists, this is
just reported (at log level`info`). The server logs its requests at
log level`debug`only, as the agents announce frequently.
 
- `ase agent relay stop`:
Stop the agent relay server by sending it- `SIGTERM`and waiting (up
to 5 seconds) until it is gone, closing all subscriptions of the task
boards. A not running server is reported as such, without an error.
 

## OPTIONS

The following options exist for `ase agent relay start`:

- [- `-a`|- `--address`- *host*]:
The address to bind to (default: the configured- `address`of- `relay.yaml`, else- `127.0.0.1`), e.g.,- `0.0.0.0`for all IPv4
interfaces of a central host.
 
- [- `-p`|- `--port`- *port*]:
The TCP port to bind to,- `1`..- `65535`(default: the configured- `port`of- `relay.yaml`, else a randomly allocated free port in the range- `42000`..- `44000`).
 
- [- `-t`|- `--token`- *token*]:
The bearer token to expect from all clients (default:- `$ASE_AGENT_RELAY_TOKEN`, else the configured- `token`of- `relay.yaml`, else a generated random token).
 
- [- `-c`|- `--cors`- *origin*]:
(repeatable) Allow cross-origin browser requests from- *origin*, or
from any origin with- `*`(default: the configured- `cors`origins of- `relay.yaml`, else none).
 
- [- `--tls-cert`- *file*]:
Serve HTTPS with the PEM certificate (chain) in- *file*, which
requires- `--tls-key`, too (default: the configured- `project.relay.crt`(unless- `project.relay.url`selects HTTP via- `ase://`on this host),
else the configured- `tls.cert`of- `relay.yaml`, else plain HTTP). A
relative path of a configuration is relative to its configuration
file.
 
- [- `--tls-key`- *file*]:
Serve HTTPS with the PEM private key in- *file*, which requires- `--tls-cert`, too (default: the configured- `project.relay.key`(unless- `project.relay.url`selects HTTP via- `ase://`on this host),
else the configured- `tls.key`of- `relay.yaml`).
 
- [- `-f`|- `--foreground`]:
Run the server in the current process, logging to the terminal
instead of- `relay.log`, until it is interrupted (e.g., via- `CTRL+C`)
or stopped via- `ase agent relay stop`.
 
- [- `-h`|- `--help`]:
Show the usage information of the command and exit.
 

## PROTOCOL

All requests have to carry the bearer token in the `Authorization: Bearer` *token* header (the WebSocket handshake alternatively in the
query parameter `token`), else they are rejected with status 401 and
an `application/problem+json` problem details response. The REST
endpoints are:

- `GET /agents`:
The projects with agents, as- `{ "projects": [ { "id":`- *id*- `, "agents":`- *count*- `} ] }`.
 
- `GET /projects/`- *id*- `/agents`:
All agents of the project- *id*, as- `{ "agents": [`...- `] }`.
 
- `PUT /projects/`- *id*- `/agents/`- *agent*:
The announcement of the agent- *agent*(also its heartbeat), as a JSON
payload with its presence and only its processing log lines added
since its previous announcement (at most 100), answered with status
 -  
- `DELETE /projects/`- *id*- `/agents/`- *agent*:
The removal of the agent- *agent*(on its termination), answered with
status 204.
 

The WebSocket event endpoint `/projects/`*id*`/agents/events` first
sends a snapshot frame of all agents of the project (with `reset: true`) and then a frame on each change (the added or updated agents,
and the removed ones), where the processing log lines of an agent are
always to be appended to the already received ones.

Subscribers are pinged every 15 seconds and terminated if they did not
answer the previous ping, resp. if they cannot keep up (more than 1 MB
of not yet transmitted data). A plain HTTP request onto the event
endpoint is answered with status 426.

The project ids have to match `[A-Za-z0-9_-]+` and the agent ids
`[A-Za-z0-9][A-Za-z0-9_.-]*` (at most 128 characters), else the request
is rejected with status 422. To bound its memory, the relay holds at
most 128 projects with agents, 64 agents per project, and 1024 agents
in total, and accepts at most 64 subscribers per project and 1024
subscribers in total, where a request beyond these limits is rejected
with status 429.

## ENVIRONMENT

- `ASE_AGENT_RELAY_TOKEN`:
The default of- `--token`of- `ase agent relay start`, and the bearer
token of the agents and task boards connecting to the relay (if not
embedded in its URL).

## FILES

- *per-user configuration directory*- `/relay.yaml`:
The configuration and runtime state of the agent relay server: the
keys- `address`,- `port`,- `token`,- `cors`(a list of origins),- `tls.cert`,- `tls.key`, and- `pid`(of the running server). It is
written by- `ase agent relay start`(readable by the current user
only), and can be edited by hand to configure the defaults.
 
- *per-user configuration directory*- `/relay.log`:
The log of the agent relay server run in the background.
 

The *per-user configuration directory* is `~/Library/Application Support/ase` on macOS, `%APPDATA%\ase` on Windows, and
`$XDG_CONFIG_HOME/ase` (falling back to `~/.config/ase`) on Linux and
other Unix systems.

## EXIT STATUS

`start` and `stop` exit with status 0 on success (also for a server
already running resp. not running) and 1 on any error. `status` exits
with status 1 if the server is not running or rejects the configured
token. `log` exits with status 0.

## EXAMPLES

```
# start the agent relay locally and let the agents and task boards follow it
$ ase agent relay start -p 4711
$ ase config --scope project set project.id myproject
$ ase config --scope project set project.relay.url ase://127.0.0.1:4711
# start the agent relay on a central host via HTTPS
$ ase agent relay start -a 0.0.0.0 -p 4711 --tls-cert relay.crt --tls-key relay.key
$ ase config --scope user set project.relay.token <token>
# check and follow the agent relay
$ ase agent relay status
$ ase agent relay log -f
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-fleet(1)`, `ase-task-board(1)`,
`ase-task-store(1)`, `ase-config(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-agent-fleet(1) - Manage a Fleet of Agents

## NAME

`ase-agent-fleet(1)` - Manage a Fleet of Agents

## SYNOPSIS

`ase agent fleet init`
[`-c`|`--config` *file*]
[`-t`|`--tool` *tool*]
[`--lifecycle` *model*]
[`--force`]

`ase agent fleet start`
[`-c`|`--config` *file*]
[`-f`|`--foreground`]

`ase agent fleet status`

`ase agent fleet log`
[`-f`|`--follow`]
[`-n`|`--lines` *n*]

`ase agent fleet stop`

## DESCRIPTION

`ase agent fleet` manages a *fleet* of agents (see `ase-agent(1)`),
and optionally of the agent relay they report to (see
`ase-agent-relay(1)`), as given by a YAML configuration file (default:
`.ase/fleet.yaml`). A supervising process runs each member in its own
child process (in its own process group) via `ase agent` *agent*
`start --foreground` resp. `ase agent relay start --foreground`, and
restarts it whenever it exits.

All members log into the shared log `.ase/agent/fleet.log` of the
fleet (or, with `--foreground`, to the terminal), where each line is
prefixed with the name of the member writing it (resp. `fleet` for the
fleet itself), padded to the longest name and followed by `|`
(e.g., `implementer | ase: [...]: [INFO]: ...`), and, on a terminal,
colored by the name (the fleet in bold red, the relay in red, and all
agents in blue).

On its start, the fleet first starts the agent relay (if any, waiting
up to 10 seconds until it persisted its runtime state), and then all
agents in parallel (waiting up to 30 seconds for each one), where it
passes the URL of the agent relay via `--agent-relay` and its actual
token via `$ASE_AGENT_RELAY_TOKEN` to all agents. If any member fails
to start, all members are stopped again and the failure is reported. A
member exiting during the startup of the others is restarted once the
fleet is ready.

Afterwards, the fleet restarts every member whenever it exits (e.g.,
crashed, or stopped individually via `ase agent` *agent* `stop`) after
a delay starting at 1 second and doubled on each restart in a row up
to 5 minutes, which is reset once the member ran for 10 minutes.

On termination (`SIGTERM`, `SIGINT`, `SIGHUP`, or `SIGQUIT`), the fleet
first stops all agents (each of which aborts its running processings
and puts their tasks back) and then the agent relay (as the agents
remove their presence from it), where a repeated signal is forwarded
to all remaining members (terminating the agents at once).

As each agent runs in its own process, the agents of a fleet do not
coordinate their direct use of the Git working copy they are started
in, so a working copy should be used directly by a single agent only,
while the other agents are given `repo` (see `ase-agent(1)`).

As the members run in their own process groups, they survive a fleet
process killed without a chance to stop them (e.g., via `SIGKILL`).
Such *orphaned* members, identified by their own runtime state still
recording the process recorded by the runtime state of the fleet
(resp., for the agent relay, by `relay.yaml` still recording it), are
stopped by the next `ase agent fleet start` (before starting the new
members) or `ase agent fleet stop`.

## COMMANDS

- `ase agent fleet init`[- `-c`|- `--config`- *file*] [- `-t`|- `--tool`- *tool*] [- `--lifecycle`- *model*] [- `--force`]:
Create the initial YAML configuration- *file*of a fleet for a task
lifecycle model. For all models, it contains the section- `relay`(with all its options commented out, so they fall back to- `project.relay.url`,- `project.relay.crt`, and- `project.relay.key`),
the section- `default`(with- `tool`,- `parallel: 1`, and- `timeout: 60m`,
plus- `model`,- `repo`, and- `tempdir`commented out), the edit request agent- `editor`, the effort estimation agent- `estimator`(mode- `phase`,
estimating the planned tasks, i.e., the- `OPEN`tasks for- `solo`, the- `PLANNING`tasks for- `team`, and the- `PLANNED`tasks for- `enterprise`,
intended for the final estimation), a commented-out effort estimation
request agent- `estimator-request`(mode- `request`, (re-)estimating the
tasks in any unfinished status on request, intended for dry-runs), a
commented-out research request agent- `researcher-request`(mode- `request`, intended for topic-driven research), and a
commented-out example of an ad-hoc request agent- `lint`(see- `ase-agent-adhoc(1)`).
 - Additionally, it contains per task lifecycle model: - - `solo`: no phase agents, as the model has no activity states, so
its tasks are driven interactively, while the agents only assist.
- `team`: the ad-hoc implementation phase agent- `developer`(as the
built-in phase agents support the- `enterprise`model only), which
claims only the- `PLANNING`tasks explicitly assigned to- `developer`(as the other ones are still planned) into- `IMPLEMENTING`, implements them from their- `Source:`into their- `Changeset:`via the- `ase-task-implement`skill, and puts them
into- `STALLED`unless they became- `IMPLEMENTED`.
- `enterprise`: the planning phase agents- `rebooter`,- `condenser`,
and- `researcher`(mode- `phase`), each processing only the- `DRAFTED`tasks explicitly assigned to it, and the phase agents- `implementer`(mode- `implement`, 2 tasks in parallel, waiting for
the effort estimation, with its parameter- `effort-wait`commented
out, to be set to- `false`if the- `estimator`is removed),- `qualifier`(mode- `judge`, 2 tasks in parallel, so a human still
decides on the qualification), and- `integrator`(mode- `squash`).
 - The configuration is checked like on - `ase agent fleet start`before
it is written, and an already existing- *file*is overwritten only
with- `--force`. Options:
 - - [`-c`|`--config`*file*]: the YAML configuration file to create
(default:`.ase/fleet.yaml`of the project).
- [`-t`|`--tool`*tool*]: the agent tool of all agents,`claude`,`copilot`, or`codex`(default:`$ASE_TOOL`, else`claude`).
- [`--lifecycle`*model*]: the task lifecycle model,`solo`,`team`,
or`enterprise`(default: the configured`project.task.lifecycle`,
else`solo`).
- [`--force`]: overwrite an already existing configuration file.
- [`-h`|`--help`]: show the usage information and exit.
 
- `ase agent fleet start`[- `-c`|- `--config`- *file*] [- `-f`|- `--foreground`]:
Start the fleet of the YAML configuration- *file*(see- **CONFIGURATION**) in the background (spawning a detached supervising
process logging into- `.ase/agent/fleet.log`), and return once all
members started successfully, or with- `--foreground`run the fleet in
the current process.
 - The configuration is validated before the fleet starts. An already
running fleet is left alone (and is an error with - `--foreground`).
With the section- `relay`, the fleet owns the per-user agent
relay, which hence must not run already (stop it first via- `ase agent relay stop`, or remove the section- `relay`). Options:
 - - [`-c`|`--config`*file*]: the YAML configuration file (default:`.ase/fleet.yaml`of the project).
- [`-f`|`--foreground`]: run the fleet in the current process,
logging to the terminal, until it is interrupted (e.g., via`CTRL+C`) or stopped via`ase agent fleet stop`(default: the
option`foreground`of the section`fleet`).
- [`-h`|`--help`]: show the usage information and exit.
 
- `ase agent fleet status`:
Report whether the fleet is running (with its process id and
configuration file, and whether it is still starting up), and for
each member (with its kind) whether it is running (with its process
id) or awaiting its restart, and how often it was restarted. Exits
with status 1 if the fleet is not running.
 
- `ase agent fleet log`[- `-f`|- `--follow`] [- `-n`|- `--lines`- *n*]:
Show the log- `.ase/agent/fleet.log`of the fleet run in the
background, aggregating the lines of the fleet itself and of all its
members, each one prefixed with its name, where the name and the log
level tag are colored if the output is a terminal.
 - With - `-n`|- `--lines`- *n*, only the last- *n*lines are shown (default:
all). With- `-f`|- `--follow`, the last- *n*lines (default: 10) are shown
and then, in the style of- `tail -f`, all newly appended lines are
followed until interrupted. If no log file exists, this is just
reported (at log level- `info`).
 
- `ase agent fleet stop`:
Stop the fleet with all its members, first all agents and then the
agent relay, by sending- `SIGTERM`to the fleet process and waiting
(up to 5 minutes) until it is gone. With the fleet process already
gone (e.g., killed via- `SIGKILL`), its orphaned members are stopped
instead (see- **DESCRIPTION**). A not running fleet is reported as
such, without an error.
 

## CONFIGURATION

The YAML configuration file of a fleet consists of the following
top-level sections, where all options are given by their long names
(without the leading `--`), `true` enables a flag, `false` or an empty
value omits an option, and a list repeats an option:

- `fleet`:
(optional) The options of the fleet itself. The only option is- `foreground`(a boolean, the same as- `--foreground`of- `ase agent fleet start`).
 
- `relay`:
(optional) The options- `address`,- `port`,- `token`,- `cors`,- `tls-cert`, and- `tls-key`of- `ase agent relay start`(see- `ase-agent-relay(1)`). The presence of this section (even if empty)
lets the fleet run and own the per-user agent relay.
 - Its options not given fall back as follows: - `address`,- `port`, and- `token`to those of the configured- `project.relay.url`(its embedded
token, else- `$ASE_AGENT_RELAY_TOKEN`, else- `project.relay.token`), if
it addresses a local address of this host (else this is warned about
and- `project.relay.url`is not adopted), where a deviating- `address`or- `port`(resp. a- `tls-cert`with an adopted- `project.relay.url`selecting HTTP) is warned about, as the task boards following- `project.relay.url`then do not see the agents of the fleet; and- `tls-cert`and- `tls-key`to- `project.relay.crt`and- `project.relay.key`(unless an adopted- `project.relay.url`selects HTTP), where an adopted- `project.relay.url`selecting HTTPS requires both.
 
- `default`:
(optional) The default options of all agents:- `tool`,- `model`,- `parallel`,- `timeout`,- `repo`,- `tempdir`, and- `agent-relay`(see- `ase-agent(1)`and- `ase-agent-adhoc(1)`),
each one applied only to the agents supporting it (e.g.,- `repo`not to- `rebooter`), and overridden by the section of an agent. The option- `agent-relay`is not allowed together with the section- `relay`.
 
- *name*:
One section per agent, named by its agent name (at most 64 letters,
digits, underscores, and dashes, starting with a letter or digit, and
neither the name of another agent kind, nor- `relay`or- `fleet`), with
the options of- `ase agent`- *kind*- `start`and an optional- `kind`(default: the section name), i.e., one of the built-in kinds- `rebooter`,- `condenser`,- `implementer`,- `qualifier`,- `integrator`,- `editor`,- `estimator`, and- `researcher`, a definition kind of the project or the user, or- `adhoc`(see- `ase-agent-adhoc(1)`). Hence, an agent of a definition kind is
given by a section of its kind (or with- `kind:`- *kind*), and an agent
defined inline by a section with- `kind: adhoc`.
 - The options of the definitions can also be nested by group, like in a
definition file (e.g., - `prompt: { text: ... }`), where a key must not
be given both flat and nested. The values of the parameters
declared by the definition are given by the map- `params`(e.g.,- `params: { severity: high }`), which repeats the option- `--params`- *name*- `=`- *value*per parameter.
 - The - `definition`of- `kind: adhoc`(unless a- `builtin:`one) is
relative to the configuration file, where hooks can be given only
through a TypeScript or JavaScript definition file. A section without
any options (an empty value) runs the agent of the kind of its name
with its defaults.
 

The following is refused, as the definitions of all agents are already
resolved and validated on the start of the fleet:

- the options - `name`(given by the section name),- `one-shot`(as the
members are restarted),- `foreground`, and- `help`in the section of an
agent, and- `agent-relay`there together with the section- `relay`;
 
- an unknown section option of - `fleet`,- `relay`, or- `default`, an
invalid option name (other than lower-case letters, digits, and
dashes), and a value other than a string, a number, a boolean, or a
list of strings and numbers (resp., for- `params`, a map of strings,
numbers, and booleans);
 
- an unknown agent kind, and an option unknown to the - `start`command
of the agent kind;
 
- more than one agent of a built-in kind processing serially only
(i.e., - `integrator`), and two agents processing serially only on the
same tag (e.g., two agents of- `--serial`definitions of the same
resolved- `--tag-name`, or one next to- `integrator`with its tag- `ase-agent-integrator`);
 
- a configuration without any agent. 

Before the fleet starts, the structure of the configuration, the
option names, the option syntax, and the definitions of the agents are
checked, while the remaining option values (e.g., `parallel` or `tool`)
are checked only once the corresponding member starts.

## ENVIRONMENT

- `ASE_TOOL`:
The default of- `--tool`of- `ase agent fleet init`, and of the agents
without a- `tool`option.
 
- `ASE_AGENT_RELAY_TOKEN`:
The fallback token of the agent relay of the section- `relay`, if not
embedded in- `project.relay.url`. The fleet passes the actual token of
its agent relay to all agents through this variable.
 

## FILES

- `.ase/fleet.yaml`:
The default configuration of the fleet, written by the user (or
initially created via- `ase agent fleet init`) and intended to be
versioned.
 
- `.ase/agent/fleet.yaml`:
The process state of the running fleet: its process id and process
start time, its configuration file, whether all its members completed
their initial startup, and the kind, process id (- `null`while awaiting
its restart), and number of restarts of each member. It exists while
the fleet runs.
 
- `.ase/agent/fleet.log`:
The shared log of the fleet run in the background and of all its
members.
 
- `.ase/agent/`- *name*- `.yaml`:
The process state of the member agent named- *name*, existing while
it runs.
 

## EXIT STATUS

`init`, `start`, and `stop` exit with status 0 on success (also for a
fleet already running resp. not running) and 1 on any error. `status`
exits with status 1 if the fleet is not running. `log` exits with
status 0.

## EXAMPLES

```
# create the initial configuration of a fleet, and start and follow it
$ ase agent fleet init -t claude --lifecycle enterprise
$ ase agent fleet start
$ ase agent fleet status
$ ase agent fleet log -f
# stop the fleet with all its members
$ ase agent fleet stop
```

A configuration `.ase/fleet.yaml` with the agent relay, the built-in
agents (with an implementer using cheaper models for smaller tasks),
two editors of different agent tools, an agent of the
definition file `.ase/agents/reviewer.mts`, and an ad-hoc request
agent:

```
fleet:
    foreground: true
relay:
    port:     4711
default:
    tool:     claude
rebooter:
implementer:
    parallel: 2
    model:    XS=haiku,S=haiku,*=sonnet
editor-claude:
    kind:     editor
editor-codex:
    kind:     editor
    tool:     codex
reviewer:
    parallel: 2
    params:
        severity: high
lint:
    kind:     adhoc
    type:     request
    prompt:
        text: "Lint the source code of task {id} with focus on: {prompt}"
```

## SEE ALSO

`ase(1)`, `ase-agent(1)`, `ase-agent-adhoc(1)`, `ase-agent-relay(1)`,
`ase-agent-rebooter(1)`, `ase-agent-condenser(1)`,
`ase-agent-researcher(1)`, `ase-agent-implementer(1)`,
`ase-agent-qualifier(1)`, `ase-agent-integrator(1)`,
`ase-agent-editor(1)`, `ase-agent-estimator(1)`, `ase-task-board(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com

ase-man(1) - Browse the internally shipped manual pages

## NAME

`ase-man(1)` - Browse the internally shipped manual pages

## SYNOPSIS

`ase man`
[`-h`|`--help`]
[`-l`|`--list`]
[*page* [...]]

## DESCRIPTION

**ase man** shows the manual pages shipped internally with the **ase**
tool: the manual pages of all **ase** commands *and* the manual pages
of all **ASE** skills of the bundled agent tool plugin.

The manual pages of the commands are shipped as Markdown in the
directory `dst/man/` of the tool package. There is one manual page per
command and command group, named after the command path with dashes,
e.g., `ase-task(1)` for `ase task` and `ase-agent-fleet(1)` for `ase agent fleet`. The commands of the third level (e.g. `ase task store start`) are covered by the manual page of their group (e.g.
`ase-task-store(1)`).

On a terminal (and without `--list`), **ase man** opens the
interactive, full-screen *manual page browser*. If standard input or
standard output is not a terminal, it instead prints the manual page
*page* (or `ase(1)`) as plain text, wrapped to 80 columns, onto
standard output. The browser consists of two pages:

- The - *start page*shows a search field on top of a catalog of all AI
skills and all CLI commands, grouped like the catalog of the website
and laid out in up to five columns, depending on the terminal width
(reduced to the matching manual pages while a search is active).
Selecting a manual page or submitting a search opens the browser page.
The browser starts on the start page if no- *page*is given.
 
- The - *browser page*shows the master on the left, with the search
field on top of the grouped list of all manual pages (reduced to the
matching manual pages while a search is active), and the detail on
the right, with the selected manual page as rendered and
syntax-highlighted Markdown (with the matches of an active search
highlighted). References to other manual pages within the detail,
like- `ase-task(1)`or- `ase-task-edit`, can be followed. The browser
starts on the browser page if a- *page*is given: with the manual page
fitting exactly onto- *page*selected, else with- *page*as the search.
 

A search consists of whitespace-separated keywords, which *all* have to
occur (case-insensitively) in a manual page. Only if no manual page
matches the keywords literally, they are matched fuzzily against the
words of the manual pages, so a misspelled keyword still finds its
manual pages. On both pages, the catalog resp. the master is filtered
live while typing, and the header shows the counts of the matching
manual pages.

## OPTIONS

- [- `-h`|- `--help`]:
Show program usage information only.
 
- [- `-l`|- `--list`]:
Instead of showing a manual page, list the names of all shipped
manual pages of the commands, one per line, in natural order. Any- *page*is ignored.
 

## ARGUMENTS

- [- *page*[...]]:
The manual page to show (default: the start page of the interactive
manual page browser on a terminal, else- `ase`). It is given either by
its name (e.g.- `ase-agent-fleet`or- `ase-task-edit`) or by the words
of the command path (e.g.- `agent fleet`), which are joined with
dashes, where the leading- `ase`is optional and the name is
case-insensitive.
 - In the manual page browser, the manual page fitting exactly onto
- *page*is selected. Of a skill and a command of the same name (e.g.- `ase-task-edit`and- `ase-task-edit(1)`), a single name with dashes
(e.g.- `task-edit`) selects the skill, while the words of a command
path (e.g.- `task edit`) select the command. If no manual page fits
exactly, the words of- *page*are
placed into the search field instead, so the master shows only the
matching manual pages, with the first of them selected.
 - When printed as plain text, the manual page fitting exactly onto
- *page*is printed likewise. If no manual page fits exactly, a command
path deeper than the shipped manual pages (e.g.- `task store start`)
falls back to the manual page of its nearest parent group (e.g.- `ase-task-store`), but never to the top-level page- `ase(1)`, and a- *page*consisting of anything other than letters, digits, and dashes
is refused.
 

## KEYS

The following keys and mouse actions are supported by the interactive
manual page browser.

### Start Page

- `Up`,- `Down`,- `Left`,- `Right`: Select the manual page above, below,
left, or right of the selected one in the catalog (at the end of a
column,- `Up`and- `Down`continue in the nearest other column).
- `Home`,- `End`: Select the first or last manual page of the catalog
(not within the search field).
- `Return`: Open the selected manual page on the browser page (keeping
an active search).
- `/`: Enter the search field, whose search filters the catalog live.
Within it,- `Return`submits the search (opening the browser page
with the selected, i.e. initially the first, matching manual page),- `Left`,- `Right`,- `Home`, and- `End`move the cursor,- `Backspace`and- `Delete`delete the character before resp. at the cursor, and- `Escape`leaves the search field again.
- `Escape`: Clear a non-empty search (not within the search field).
- `q`,- `Ctrl`+- `C`: Quit (- `q`not within the search field).
- Left-Click: Open the clicked manual page (keeping an active search),
resp. enter the clicked search field (clearing the search on its
trailing `✕`symbol).
- Wheel: Select the manual page above or below the selected one.

### Browser Page

- `Up`,- `Down`: Select the previous or next manual page of the master,
resp. the previous or next reference within the detail (while a
reference is selected).
- `Right`,- `Left`: Enter the references of the detail (selecting the
first shown reference), resp. leave them again towards the master.
- `Page Up`,- `Page Down`,- `Space`: Scroll the detail by a page
(- `Space`not within the search field).
- `Home`,- `End`: Scroll the detail to its top or bottom (not within
the search field).
- `Tab`,- `Shift`+- `Tab`: Select the next or previous reference to
another manual page within the detail.
- `Return`: Follow the selected reference (leaving a search the
referenced manual page does not match), resp. leave the search
field.
- `/`: Enter the search field, whose search filters the master live.
Within it,- `Left`,- `Right`,- `Home`, and- `End`move the cursor, and- `Backspace`and- `Delete`delete the character before resp. at the
cursor.
- `Escape`: Step back: first clear a non-empty search, then leave the
search field, and finally return to the start page.
- `q`,- `Ctrl`+- `C`: Quit (- `q`not within the search field).
- Left-Click: Select the clicked manual page of the master, follow the
clicked reference of the detail, resp. enter the clicked search field
(clearing the search on its trailing `✕`symbol).
- Wheel: Select the previous or next manual page (over the master),
resp. scroll the detail (over the detail).

## FILES

- *package directory*- `/dst/man/ase{,-`- *command*- `{,-`- *sub-command*- `}}.md`:
The internally shipped manual pages of the commands.
 
- *package directory*- `/plugin/skills/`- *skill*- `/help.md`:
The manual pages of the skills of the bundled agent tool plugin.
 

## EXIT STATUS

The command exits with status `0` on success, and with status `1` if
no manual page is found for *page* when printing it as plain text.

## EXAMPLES

```
# browse all manual pages of the commands and skills interactively
$ ase man
# show the manual page of the "ase" command itself
$ ase man ase
# show the manual page of "ase agent fleet" (all equivalent, as no skill has this name)
$ ase man ase-agent-fleet
$ ase man agent-fleet
$ ase man agent fleet
# show the manual page of the skill "ase-task-edit" resp. of "ase task edit"
$ ase man task-edit
$ ase man task edit
# browse the manual pages matching the keywords "worktree" and "merge"
$ ase man worktree merge
# print the manual page covering "ase task store start" as plain text
$ ase man task store start | less
# list all shipped manual pages of the commands
$ ase man --list
```

## SEE ALSO

`ase(1)`.

## AUTHOR

Dr. Ralf S. Engelschall rse@engelschall.com
