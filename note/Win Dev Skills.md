# Win Dev Skills

Microsoft's win-dev-skills repo is an Agent Plugins 1.0 package (v0.6.1, preview) that gives coding agents — GitHub Copilot CLI, Claude Code, OpenAI Codex, OpenClaw, OpenCode — eight skills and a `winui-dev` orchestrator agent for the full WinUI 3 / Windows App SDK inner loop: scaffold → design → build → run → test → package → signed MSIX. What makes it worth studying is not the Windows domain but the shape: an unusually disciplined statement of the idea that agent skills are *prompts plus playbooks* whose leverage comes from ground-truth tooling, backed by two compiled tools that replace hallucination with verification, and a packaging story that keeps one canonical skill set alive across five agent clients without duplicating a line.

---

## Architecture

The repo is small (~130 files) and strictly layered, and its own review tooling names the layers:

- **Tier 3 — skills (prose).** Eight `SKILL.md` files under `plugins/winui/agent-plugin/skills/`, each with YAML frontmatter (`name`, `description`) plus optional `references/` deep-dives. `winui-dev-workflow` (111 lines) and `winui-design` (175) load by default; `winui-setup`, `winui-ui-testing` (368 lines, the largest), `winui-code-review`, `winui-packaging`, `winui-wpf-migration`, and `winui-session-report` load on demand.
- **Tier 2 — glue scripts.** `BuildAndRun.ps1` wraps project-mode `winapp run` with analyzer injection and crash diagnostics; `Analyze-Session.ps1` parses Copilot CLI / Claude Code session JSONL into a `session-report.md`.
- **Tier 1 — tools (enforcement).** `src/tools/winui-analyzer/` — a Roslyn analyzer (~1,000 lines across ten rule classes, with its own test project) — and `src/tools/winmd-cli/` — a ~3,500-line Native-AOT CLI published as an ~8 MB single-file `winmd.exe`.

The **orchestrator** (`winui-dev.agent.md`) is short and opinionated: it orders the agent to load the workflow and design skills up front, forbids self-hopping (`don't call task with agent_type: "winui:winui-dev"`), and permits `task` delegation only for scoped helpers like codebase mapping.

**winmd-cli** is the most engineered piece. It is two-phase: `update` discovers metadata from `project.assets.json` (falling back to `packages.config`), direct project references, the Windows SDK's `UnionMetadata` tree, and `Get-AppxPackage Microsoft.WindowsAppRuntime.*`, then parses `.winmd` files and adjacent XML doc files (including `microsoft.windowsappsdk.winui` and `microsoft.windows.sdk.net.ref`, 48K+ API descriptions) into a structured JSON cache under `%LOCALAPPDATA%\winmd-cache`. Query subcommands — `search`, `members`, `types`, `enums`, `check-property`, `namespaces`, `packages`, `projects`, `stats` — read only the cache, auto-refreshing when `project.assets.json` is newer than the cached manifest, with refresh serialized through a `.lock` file.

**Packaging** is one canonical portable package (`plugins/winui/agent-plugin/plugin.json`, validated against the Agent Plugins 1.0 schema) surrounded by compatibility shells: `.claude-plugin/` for the Claude Code marketplace, `.agents/plugins/` for Codex, `openclaw.plugin.json` + `index.js` for OpenClaw, and — because the spec does not standardize custom agents — the agent lives under the reverse-domain `com.github.copilot/` namespace and is mirrored into the outer package's `agents/` for Claude Code, with CI requiring the two copies to stay content-identical.

## Key techniques

- **A ground-truth oracle before codegen.** `winmd check-property <Type> <Prop>` answers the question an LLM would otherwise guess: does this property exist, including inherited and attached properties? On a miss it suggests similar properties *and names which types actually have the property*. `members` prints the same XML doc text Visual Studio IntelliSense shows and flags UWP-era traps (`Window.Current` "always returns null; use DispatcherQueue"). The attack on hallucination is moved from the prompt layer to a tool with real metadata behind it.
- **Hand-rolled tiered scoring.** `Scoring.cs` ranks matches exact=100, prefix=80, contains=60, initialism (PascalCase initials)=50, all-words=40, fuzzy subsequence=20, and warns when a name exists in multiple namespaces (`FileAttributes` in both `System.IO` and `Windows.Storage`). No fuzzy library — a deliberate simplicity trade for an AOT binary.
- **Anti-clobber MSBuild injection.** `BuildAndRun.ps1` loads the bundled analyzer via `CustomAfterDirectoryBuildProps` in a temporary props file and *reserves* that property — if the user passes `-p CustomAfterDirectoryBuildProps=...`, the wrapper errors out rather than silently dropping the analyzer. It also version-gates `winapp >= 0.6.0` and defaults `--debug-output` on, which runs stowed-exception triage to surface the real XAML error behind an opaque `0x8000FFFF`.
- **The same mistake encoded twice.** `x:Bind` silently defaulting to `OneTime` and `Converter={x:Null}` crashing at runtime appear both as prose in `winui-design` ("XAML landmines") and as `WUI2xxx` analyzer rules. The prose teaches; the analyzer is the durable version that survives the next model.
- **Distilled platform rubrics as prompts.** `winui-design` carries an app-shape anchor table (six silhouettes mapped to reference apps like Windows Terminal and Photos), a control map that short-circuits cross-framework instincts ("tabular → ListView with a Grid-based ItemTemplate; WinUI has no DataGrid, and the CommunityToolkit DataGrid's columns can't use `x:Bind`"), and a window-sizing rubric (width = widest row + 48, rounded up to 20; `AppWindow.Resize` takes physical pixels, so P/Invoke `GetDpiForWindow` because `XamlRoot.RasterizationScale` is null in the constructor).
- **Grounded sample lookup.** `winapp find-ui` searches a cached corpus of WinUI Gallery, Community Toolkit, and curated core patterns — the agent consults real shipping samples instead of generating XAML from priors.
- **An error encyclopedia as prompt content.** The workflow skill tabulates a dozen opaque diagnostics with root causes: `0x80073CF9` "Failed to reach state Staged" (registered packages hold live references to their layout directory, so Debug and Release must not share one), `MSB3073` with no `.xaml` named (an old WindowsAppSDK XAML-compiler bug fixed in ≥ 2.1.3), `0x8007000B` (AnyCPU). This is tribal knowledge that generic web context reliably lacks.
- **Batch UI testing over the accessibility layer.** `winapp ui` drives Windows UI Automation, so one AutomationId-based `ui-tests.ps1` template works on Win32, WPF, WinForms, and WinUI 3 alike — and the skill encodes PowerShell traps (`$Pid` is read-only; use `throw`, not `exit 1`, inside test scriptblocks).
- **Consent boundaries in skill frontmatter.** `winui-setup`'s description forbids other skills from invoking it automatically — a missing prerequisite must be reported, not self-healed — and the README's install prompt tells the agent to ask before triggering UAC. `winui-session-report` obliges the agent to repeat a privacy warning because reports embed unredacted transcripts.
- **The repo reviews itself with agents.** `.github/skills/pr-review/` fans out parallel subagents over dimensions (skill content, skill↔tool boundary, tool correctness, payload provenance, docs/manifests, multi-model cross-check) with a shared finding contract, and CI byte-compares the committed analyzer DLL against a fresh build (`analyzer-provenance`) so shipped binaries can't drift from source.

## Design decisions

- **Tools over prose, explicitly.** The review skill's doctrine: skills are "Tier 3 instructions agents frequently ignore — adding prose is the last resort"; the analyzer is "Tier 1 enforcement" and the preferred home for behavior. The cost is honest: only what compiles into a Roslyn rule gets enforced; the rest stays fragile markdown.
- **Spec-first portability.** Rather than hand-maintaining per-client forks, the repo bets on Agent Plugins 1.0 as canonical and ships thin compatibility shells, referencing — never copying — `agent-plugin/skills/`. Marketplace artwork is deliberately *not* in the portable manifest because `logo` isn't in the 1.0 schema.
- **Unsigned binaries, stated plainly.** The DLL and `winmd.exe` ship unsigned from the repo; the README calls this "an explicit launch-preview trade-off, not a long-term distribution model" and CI provenance checks are the interim mitigation. When the analyzer becomes a NuGet package, `BuildAndRun.ps1` is slated for deletion.
- **Warning-only analyzer.** Every rule ships at `Warning` severity with a `helpLinkUri` — advisory, never build-breaking — with immutable `WUIxxxx` IDs (retired numbers never reused, tracked in `RULES.md` as the single source of truth).
- **Thin glue by design.** Restore, build, architecture selection, packaged-vs-unpackaged detection, runtime setup, registration, and launch all belong to WinApp CLI; this repo owns only knowledge and verification. Almost nothing here could not be deleted if `winapp` absorbed it — and the skills say so.
- **Preview velocity over stability.** No SemVer until 1.0; the marketplace install tracks `main` HEAD with pinning only via `git URL#tag`; a staging→main promotion-PR release model with CI-enforced version bumps and changelog entries.

## Comparison notes

- [[claude-plugins (Darling Data)]] ships one SKILL.md to three harnesses by hand-maintaining native adapters; win-dev-skills instead adopts a formal spec (Agent Plugins 1.0) with compatibility manifests and CI content-identity checks — and goes further by backing prose with compiled tools, where Darling's verification lives inside a stdlib script.
- [[Cerulean]] solves multi-agent portability by codegenning ~22 native adapter files for ~20 agents from a 102-line skill; win-dev-skills makes the opposite bet that the ecosystem converges on a shared format, accepting five shells now to avoid per-client codegen forever.
- [[Habit Hooks]] turns linter output into refactoring prompts the agent reads; win-dev-skills runs the same loop in reverse — distilling observed agent failures into analyzer rules so the failure never reaches the prompt at all. Together they bracket the prompt↔linter boundary from both sides.
- [[Architectural Guardrails for AI-Generated Code]] argues for structural enforcement over instructions; win-dev-skills is Microsoft's concrete instantiation of that stance in a domain stack, with the twist that the guardrails ship inside the skill package itself rather than in the consumer's repo.

---
*Sources: [[raw/win-dev-skills]], [[summary/win-dev-skills]]*
*Last updated: 2026-09-22*

#tool #project #agents #claude-code #guardrails #windows
