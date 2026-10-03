# winapp CLI — Microsoft's Agent-First Windows App Development CLI

Microsoft's winapp CLI (public preview, ~v0.7–1.0) is a single command-line interface for the unglamorous half of Windows app development: SDK setup, package identity, MSIX packaging, signing, manifests, and certificates. What makes it wiki-worthy is that it is one of the clearest examples yet of a platform vendor building a *developer tool whose primary user is an AI coding agent* — with command ergonomics, JSON envelopes, and even a multi-agent desktop-arbitration protocol designed around agent loops rather than human terminals.

---

## Architecture

Three layers in the repo:

- **`src/winapp-CLI`** — the core, ~108k lines of C#. `WinApp.Cli` has 64 command classes under `Commands/` (init, run, package, sign, manifest, cert, find-api, find-ui, ui/*, target/*, guest/*), backed by ~90 interface/service pairs in `Services/` (BuildToolsService, CppWinrtService, MsixService, DevModeService…). Classic .NET DI + Spectre.Console + System.CommandLine. Separate `WinApp.UIAutomation` assemblies implement the UIA layer.
- **`src/winapp-npm`** — the `@microsoft/winappcli` npm package. `cli.ts` wraps the self-contained native binary (win-x64/win-arm64 shipped inside the package) and *intercepts* `init` and `restore` to add Node.js-native-addon generation (Electron/C++/C# addon scaffolding in `cpp-addon-utils.ts`, `jsbindings/`) before delegating to the .NET CLI.
- **`plugins/winapp`** — an agent plugin (agent-plugins.org schema + Claude plugin manifest + Copilot agent `.md`) shipping 11 skills: winapp-setup, find-api, find-ui, ui-automation, sandbox, package, signing, manifest, identity, frameworks, maui, troubleshoot. Each SKILL.md is a dense operational prompt, not documentation.

Distribution is tri-channel: winget, npm, NuGet (`Microsoft.Windows.SDK.BuildTools.WinApp`).

## Key techniques

- **API-surface grounding over recall.** `winapp find-api` builds a lexical index from the project's *restored* .winmd/.dll metadata and refreshes it when `project.assets.json` changes. Its skill file is remarkably prescriptive: "prefer it over recall and over web search… when they disagree, find-api wins," map compile errors (CS0246, CS0117, CS1061, XAML unknown member) to specific lookups, and **batch subjects into one call** because "the dominant cost of a lookup is the round trip." That is prompt engineering against LLM economics, baked into a CLI.
- **Cooperative desktop turn-taking.** Windows has one foreground window and one input stream, so concurrent agent workflows stomp on each other. `winapp ui` makes mutating commands take cooperative turns keyed by `WINAPP_UI_WORKFLOW_ID`, with a 4-second idle grace, owner-affinity-then-FIFO ordering, an explicit `yield` verb, and a documented rule that after a reasoning gap the agent must re-inspect because transient UI may have died. This is a coordination protocol for multi-agent UI driving, solved inside a CLI.
- **Stable JSON envelopes as contracts.** `ui inspect --json` emits nested `windows[]` with per-window DPI, coordinate space, and `selector` handles (`btn-save-c3d4`), with a changelog noting removed fields — the CLI treats agent consumers as an API surface with versioned schemas (`docs/cli-schema.json`).
- **Sandbox execution with honest boundaries.** `--on sandbox` runs the built app in Windows Sandbox with push/pull file transfer, native-guest screenshots, and video capture — while the docs insist builds still run on the host and one Sandbox is a *shared*, not isolated, environment. No silent host fallback: sandbox means sandbox or fail.
- **A hidden child-process verb** in `Program.cs` runs the XAML triage pass in isolation because the modern dbgeng.dll must not be "poisoned" by system32's dbghelp.dll already loaded in the parent — a real-world DLL-hell workaround inside an agent tool.

## Design decisions

The trade is human ergonomics for agent ergonomics: verbose `--json` shapes, exit codes as gates, batching-first design, and skills that read like operator manuals for a robot. Rather than abstract Windows packaging into a new model, winapp keeps `Package.appxmanifest` and `winapp.yaml` as the artifacts and wraps the existing toolchain (signtool, makeappx, CppWinrt) — optimizing for trust and CI/CD compatibility over novelty. Sacrifices: a 108k-line C# core for what is partly a prompt-delivery vehicle, and npm wrapper complexity (interception, pre/post hooks) that adds a second mental model.

## Comparison notes

- Unlike [[FlaUInspect]], which is a human-facing UIA tree inspector, `winapp ui` wraps the same Windows API but outputs versioned JSON envelopes and turn-taking semantics aimed at agent loops.
- It is a strong concrete instance of [[10 Principles for Agent-Native CLIs]]: batch calls, stable structured output, exit codes as gates, and commands whose help text addresses the agent directly ("Agent-first: …pair it with --json").
- Its sandbox story is the practical, conservative counterpart to [[The Agent Access Model]]: rather than new identity infrastructure, it leans on an OS primitive and says plainly where the isolation stops (builds on host, shared guest).
- The skills directory is a vendor-shipped example of the pattern described in [[Writing a Good CLAUDE.md]] and the [[10 Principles for Agent-Native CLIs]] material — judgment encoded as markdown, versioned with the tool.

---
*Sources: [[raw/winappcli]], [[summary/winappcli]]*
*Last updated: 2026-10-03*
