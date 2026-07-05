---
url: https://github.com/Ataraxy-Labs/sem
date_fetched: 2026-07-05
backfilled: true
---

Part of the Ataraxy Labs stack— agent-native infrastructure for software development. See also: weave (entity-level git merge driver) · inspect (semantic code review) · opensessions (tmux sidebar for coding agents).Read the manifesto: https://ataraxy-labs.com/#thesis · Essays: https://ataraxy-labs.com/blogs · LLMs: https://ataraxy-labs.com/llms.txt


  **Semantic version control built on Git.**

  Instead of lines changed, sem tells you what entities changed: functions, methods, classes.

Why sem? · Install · Commands · Agents (MCP) · Releases

sem is a semantic version control tool that works on top of Git. It parses your code with tree-sitter, extracts every function, class, and method as an entity, and diffs at the entity level instead of lines. This means you see "function `blahh` was modified" instead of "lines x-y changed."

It works in any Git repo with no setup.

`curl -fsSL https://raw.githubusercontent.com/Ataraxy-Labs/sem/main/install.sh | sh`Or via Homebrew:

`brew install sem-cli`Or via winget on Windows:

`winget install AtaraxyLabs.sem`Or install the npm wrapper into `node_modules`:

`npm install --save-dev @ataraxy-labs/sem`With Bun, trust the package so its `postinstall` script can download the binary:

```
bun add -d @ataraxy-labs/sem
bun pm trust @ataraxy-labs/sem
```
Once installed, update to the latest release any time:

`sem update`Or build from source (requires Rust):

`cargo install --git https://github.com/Ataraxy-Labs/sem sem-cli`Or grab a binary from GitHub Releases.

Or run via Docker:

```
docker build -t sem .
docker run --rm -it -u "$(id -u):$(id -g)" -v "$(pwd):/repo" sem diff
```
GNU Parallel ships a `sem` binary (`/usr/bin/sem`) as a symlink to `parallel`. If you have both installed, they'll collide. Run `sem --version` to check which one you're using. (#77)

**Quick fixes:**

```
# Option 1: alias in your shell profile (~/.bashrc, ~/.zshrc)
alias sem="$HOME/.cargo/bin/sem"
# Option 2: make sure cargo bin comes first in PATH
export PATH="$HOME/.cargo/bin:$PATH"
# Option 3: if installed via Homebrew
export PATH="$(brew --prefix)/bin:$PATH"
```
If you installed via npm/bun, the binary lives in `node_modules/.bin/sem` and is invoked through `npx sem` or `bunx sem`, which avoids the conflict entirely.

Works in any Git repo. No setup required. Also works outside Git for arbitrary file comparison.

sem stores its SQLite entity cache outside the repository, under the OS cache directory by default. Set `SEM_CACHE_DIR=/path/to/cache` to override the cache root; repo-local overrides are ignored so cache files do not dirty the working tree.

Entity-level diff with rename detection, structural hashing, and word-level inline highlights.

```
# Semantic diff of working changes
sem diff
# Staged changes only
sem diff --staged
# Specific commit
sem diff --commit abc1234
# Commit range
sem diff --from HEAD~5 --to HEAD
# Verbose mode (word-level inline diffs for each entity)
sem diff -v
# Plain text output (git status style)
sem diff --format plain
# JSON output (for AI agents, CI pipelines)
sem diff --format json
# Markdown output (for PRs, reports)
sem diff --format markdown
# Compare any two files (no git repo needed)
sem diff file1.ts file2.ts
# Read file changes from stdin (no git repo needed)
echo '[{"filePath":"src/main.rs","status":"modified","beforeContent":"...","afterContent":"..."}]' \
  | sem diff --stdin --format json
# Only specific file types
sem diff --file-exts .py .rs
```
Cross-file dependency graph shows what breaks if an entity changes.

```
# Full impact analysis
sem impact authenticateUser
# Direct dependencies only
sem impact authenticateUser --deps
# Direct dependents only
sem impact authenticateUser --dependents
# Affected tests only
sem impact authenticateUser --tests
# JSON output
sem impact authenticateUser --json
# Disambiguate by file
sem impact authenticateUser --file src/auth.ts
# Include default-excluded paths such as generated, fixture, vendor, benchmark, and build trees
sem impact authenticateUser --no-default-excludes
```
Entity-level blame showing who last modified each function, class, or method.

```
sem blame src/auth.ts
# JSON output
sem blame src/auth.ts --json
```
Track how a single entity evolved through git history.

```
sem log authenticateUser
# Verbose mode (show content diff between versions)
sem log authenticateUser -v
# Limit commits scanned
sem log authenticateUser --limit 20
# JSON output
sem log authenticateUser --json
```
With no entity, `sem log` analyzes recent repo history at the entity level:
**hotspots** (most-changed functions/classes, with author counts) and
**co-change pairs** (entities that repeatedly change in the same commits —
"if you touch one, don't forget the other"):

```
sem log                 # repo hotspots + co-change pairs (last 50 commits)
sem log --limit 200     # deeper history
sem log --file src/auth.ts   # scoped to one file
sem log --json          # full data
```
List all entities under a file or directory path. No path is the same as `.`.

```
sem entities
sem entities .
sem entities src/auth.ts
# JSON output
sem entities --json
sem entities src/auth.ts --json
# Include default-excluded paths such as generated, fixture, vendor, benchmark, and build trees
sem entities --no-default-excludes
```
Token-budgeted context for LLMs: the entity, its dependencies, and its dependents, fitted to a strict content token budget.
When the target signature itself does not fit, JSON output reports `target_omitted: true`.

```
sem context authenticateUser
# Custom token budget
sem context authenticateUser --budget 4000
# JSON output
sem context authenticateUser --json
# Include default-excluded paths such as generated, fixture, vendor, benchmark, and build trees
sem context authenticateUser --no-default-excludes
```
Replace `git diff` output with entity-level diffs. Agents and humans get sem output automatically without changing any commands.

`sem setup`Now `git diff` shows entity-level changes instead of line-level. No prompts, no agent configuration needed. Everything that calls `git diff` gets sem output automatically. Also installs a pre-commit hook that shows entity-level blast radius of staged changes.

On macOS and Linux, `sem setup` also wires sem into your Claude Code sessions (free, local, no login): a **warm resident graph** so structural queries answer in single-digit milliseconds instead of rebuilding each time, and **prompt-time context** so the code an agent would otherwise forage for arrives at the start of the turn. It edits `~/.claude/settings.json` idempotently, backs it up first, and leaves any hooks you already have untouched.

To disable and go back to normal git diff (also removes the session hooks):

`sem unsetup`Add the GitHub Action and every PR gets one sticky comment showing which functions, classes, and methods changed — updated in place on each push, and calling out cosmetic-only PRs (formatting/comments) explicitly:

```
# .github/workflows/entity-diff.yml
name: Entity diff
on: pull_request
permissions:
  contents: read
  pull-requests: write
jobs:
  entity-diff:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Ataraxy-Labs/sem/action@v0.15.1
```
No config, no API keys, never fails your build. See action/ for details.

Local is always free and, after `sem setup`, always warm — the resident graph keeps your repo hot on your own machine, so day-to-day queries are instant with no login. You do not pay to make your laptop fast.

Cloud is for what a laptop can't do. On a very large monorepo the first local graph build can take a few seconds; a shared team graph shouldn't be rebuilt per developer; and CI wants the graph without checking anything out. `sem login` connects those cases to sem cloud, which keeps a warm, pre-built graph for your registered repos and serves the heavy queries from it (on a large repo like deno, an `impact` query is ~86ms from the cloud vs ~573ms rebuilt locally).

```
sem login                              # GitHub device flow, one time
sem impact myFunc --file src/foo.rs    # served from the cloud's warm graph
```
It is fully optional and transparent:

- Not logged in, or the cloud is unreachable? sem computes locally and prints the exact same output. No failures, no difference in results.
- `SEM_LOCAL=1`forces local computation even when logged in.
- Small repos see no change, local is already fast. The win is for large codebases where rebuilding the graph each time is the bottleneck.

32 programming languages with full entity extraction via tree-sitter:

| Language | Extensions | Entities | 
|---|---|---|
| TypeScript | `.ts``.tsx``.mts``.cts` | functions, classes, interfaces, types, enums, exports | 
| JavaScript | `.js``.jsx``.mjs``.cjs` | functions, classes, variables, exports | 
| Python | `.py` | functions, classes, decorated definitions | 
| Go | `.go` | functions, methods, types, vars, consts | 
| Rust | `.rs` | functions, structs, enums, impls, traits, mods, consts | 
| Java | `.java` | classes, methods, interfaces, enums, fields, constructors | 
| C | `.c``.h` | functions, structs, enums, unions, typedefs | 
| C++ | `.cpp``.cc``.hpp` | functions, classes, structs, enums, namespaces, templates | 
| C# | `.cs` | classes, methods, interfaces, enums, structs, properties | 
| Ruby | `.rb` | methods, classes, modules | 
| PHP | `.php` | functions, classes, methods, interfaces, traits, enums | 
| Swift | `.swift` | functions, classes, protocols, structs, enums, properties | 
| Elixir | `.ex``.exs` | modules, functions, macros, guards, protocols | 
| Bash | `.sh` | functions | 
| Fish | `.fish` | functions | 
| Lua | `.lua` | functions (global, local, table, and method forms) | 
| HCL/Terraform | `.hcl``.tf``.tfvars` | blocks, attributes (qualified names for nested blocks) | 
| Kotlin | `.kt``.kts` | classes, interfaces, objects, functions, properties, companion objects | 
| Fortran | `.f90``.f95``.f` | functions, subroutines, modules, programs | 
| Vue | `.vue` | template/script/style blocks + inner TS/JS entities | 
| XML | `.xml``.plist``.svg``.csproj` | elements (nested, tag-name identity) | 
| ERB | `.erb``.html.erb` | blocks, expressions, code tags | 
| Svelte | `.svelte``.svelte.js``.svelte.ts` | component blocks + rune JS/TS modules | 
| Perl | `.pl``.pm``.t` | subroutines, packages | 
| Dart | `.dart` | classes, mixins, extensions, enums, type aliases, functions | 
| OCaml | `.ml``.mli` | values, modules, types, classes, externals | 
| Scala | `.scala``.sc``.sbt` | classes, objects, traits, enums, functions, vals, extensions | 
| Nix | `.nix` | bindings, inherit declarations | 
| Haskell | `.hs` | functions, signatures, data types, newtypes, classes, instances, type synonyms | 
| Elm | `.elm` | value declarations, type aliases, type declarations, port annotations, infix declarations | 
| Clojure | `.clj``.cljs``.cljc` | vars, functions, macros, multimethods, protocols, records, types | 
| D | `.d``.di` | modules, functions, classes, structs, interfaces, unions, enums, templates, aliases, unittests | 
| Zig | `.zig` | functions, tests, variables | 
| SQL | `.sql``.psql``.pgsql``.ddl` | tables, views, functions, indexes, types, schemas, triggers, sequences | 

Plus structured data formats:

| Format | Extensions | Entities | 
|---|---|---|
| JSON | `.json` | properties, objects (RFC 6901 paths) | 
| YAML | `.yml``.yaml` | sections, properties (dot paths) | 
| TOML | `.toml` | sections, properties | 
| EDN | `.edn` | top-level map entries (keyword keys) | 
| CSV | `.csv``.tsv` | rows (first column as identity) | 
| Markdown | `.md``.mdx` | heading-based sections | 

Everything else falls back to chunk-based diffing.

For files with non-standard extensions, create a `.semrc` in your project root:

```
.xyz = cpp
.j = json
.mypy = python
```
sem also reads `.gitattributes` patterns (`diff=` and `linguist-language=`) if you already have those set up. `.semrc` takes priority when both define the same extension.

For files with no extension at all, sem detects the language automatically from content (imports, declarations, shebang lines, vim modelines). This covers 19 languages with no config needed.

Three-phase entity matching:

- **Exact ID match**— same entity in before/after = modified or unchanged
- **Structural hash match**— same AST structure, different name = renamed or moved (ignores whitespace/comments)
- **Fuzzy similarity**— >80% token overlap = probable rename

This means sem detects renames and moves, not just additions and deletions. Structural hashing also distinguishes cosmetic changes (whitespace, formatting) from real logic changes.

`sem mcp` starts a Model Context Protocol server over stdin/stdout. It's not a command you run and read yourself: it's a server your coding agent launches in the background so it can ask sem questions while it works. That's the reason `mcp` lives alongside the normal commands. The agent gets 6 tools, all entity-level: `sem_impact`, `sem_context`, `sem_diff`, `sem_entities`, `sem_blame`, `sem_log`.

Why an agent wants these: instead of reading whole files and burning tokens, it can ask "what breaks if I change `submitOrder`" (`sem_impact`) or "give me just the context to refactor this function" (`sem_context`) and get a precise answer from the dependency graph.

Add it once, then talk to your agent normally. It calls the tools on its own.

**Claude Code:**

`claude mcp add sem -- sem mcp`Or one command that also installs the skill, so the agent knows *when* to reach for sem:

`npx @ataraxy-labs/sem-skill`**Cursor, Claude Desktop, or any client with an  mcpServers config:**

```
{
  "mcpServers": {
    "sem": {
      "command": "sem",
      "args": ["mcp"]
    }
  }
}
```
If `sem` isn't on the agent's PATH, use the absolute path to the binary. No separate install is needed: `sem mcp` ships in the same binary as every other command.

`sem diff --format json````
{
  "summary": {
    "fileCount": 2,
    "added": 1,
    "modified": 1,
    "deleted": 1,
    "moved": 0,
    "renamed": 0,
    "reordered": 0,
    "binary": 0,
    "orphan": 0,
    "total": 3
  },
  "changes": [
    {
      "entityId": "src/auth.ts::function::validateToken",
      "changeType": "added",
      "entityType": "function",
      "entityName": "validateToken",
      "startLine": 12,
      "endLine": 18,
      "oldStartLine": null,
      "oldEndLine": null,
      "filePath": "src/auth.ts"
    }
  ],
  "binaryChanges": []
}
```
The named change-type buckets (`added`, `modified`, `deleted`, `moved`, `renamed`, `reordered`) always sum to `total`. `orphan` is a cross-cutting metadata count for module-level changes, and those changes are already included in the named change-type buckets.

sem-core can be used as a Rust library dependency:

```
[dependencies]
sem-core = { git = "https://github.com/Ataraxy-Labs/sem", version = "0.5" }
```
Used by weave (semantic merge driver) and inspect (entity-level code review).

- **tree-sitter**for code parsing (native Rust, not WASM)
- **git2**for Git operations
- **rayon**for parallel file processing
- **xxhash**for structural hashing
- Plugin system for adding new languages and formats

sem collects anonymous usage data: the command name (e.g. `diff`, `impact`), CLI version, and operating system. Nothing else — no code, file paths, repo names, or user identity. Events are batched locally and sent in the background, so commands never wait on the network.

Disable it any time:

`export SEM_NO_TELEMETRY=1   # or DO_NOT_TRACK=1`Want to add a new language? See CONTRIBUTING.md for a step-by-step guide.

MIT OR Apache-2.0
