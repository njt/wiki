# Creating Spec Kit Presets — Step by Step

Jesse Liberty's eight-step tutorial on authoring, testing, validating, and publishing presets for [[Spec Kit]], GitHub's spec-driven development toolkit: a versioned package of template overrides, command prompt text, and small content changes that layer over the core templates by priority at runtime.

---

A **preset** is a versioned package supplying template files, CLI command prompt text, companion `.prompt.md` files, small scripts, and tiny content changes (terminology, defaults, localization). Resolution is a layered stack: when Spec Kit resolves a template or prompt it walks the stack and the highest-priority file wins. Presets are explicitly scoped to *content and style* — anything stateful belongs in Bundles or Extensions.

## The eight steps

1. **Design** — pick exactly what you override (`templates/`, `commands/`, `scripts/`, small wording/defaults), keep one preset to one job, choose an id (`com.example.my-company-style`), version, and priority (lower number = higher precedence; default 10).
2. **Structure & manifest** — a folder with `preset.yml` at the root plus override subfolders; required fields are `id`, `name`, `version`, with `description`, `author`, `priority`, `tags` for usability and catalog listing.
3. **Place overrides** — copy the exact core filenames and paths (resolution matches by path; files install into `.specify/presets/` and merge automatically — no manual merging). Example given: a `plan-template.md` override adding a company C# logging convention.
4. **Local test loop** — `specify preset add --dev /path` points at your working copy; run generation commands like `specify plan`, inspect output, and use `specify preset resolve` to see which file wins.
5. **Validate the manifest** — a tiny C# YamlDotNet program that fails (exit 2) if `id`/`name`/`version` are missing; intended for CI fail-fast.
6. **Tests & CI** — run `specify` commands in a temp workspace, snapshot-test generated artifacts, lint the YAML, and exercise any scripts cross-platform (shell + PowerShell).
7. **Package & publish** — GitHub/GitLab repo plus a `catalog.json`/`catalog.yaml` entry, community tooling like `presetify`, semantic-version tags so consumers can pin.
8. **Consumer lifecycle** — `specify preset add [--version|--from]`, `--dev` for local development, `specify preset remove`, and version-bump upgrades.

## Warnings

The best-practices list is the most opinionated part: keep presets to content/localization, never runtime logic; small focused presets over monoliths; document and major-bump any change to template placeholders (the "template API contract"); use meaningful tags; and — unusually for a tutorial — a **security caution**: don't blindly trust third-party catalogs; inspect preset contents before installing.

## Take

This is a how-to, not an essay, but two things make it more than documentation. First, it confirms Spec Kit's extension model is deliberately conservative: the preset layer can bend wording, templates, and localization but not behavior — a design that keeps the spec corpus portable across teams. Second, the security note is a rare admission in the preset/plugin ecosystem that "install a template package" is a supply-chain decision; a preset carries prompt text that agents will execute faithfully, so inspecting it matters as much as inspecting a dependency. The AI-assisted provenance (drafted by "BlogWriter," edited by Liberty, with an explicit caution that "LLMs make mistakes") makes it a fitting artifact: a tutorial about templating agent instructions, itself templated.

#tool #pattern #concept

## Relations

This how-to [[Spec Kit|documents the extension surface]] of the toolkit: it strengthens [[Spec Kit]]'s picture of a hash-tracked, layered-template system by showing the preset mechanism and its resolution rules in operational detail. It nuances [[SDDW (Spec-Driven Development Workflow)]] — sermakarevich's bespoke spec pipeline is exactly the kind of process customization Spec Kit now ships as a shareable preset rather than a hand-rolled plugin. It complicates [[SDD Case Study — 13 Apps in 70 Days]]: Fontoura's "correct the spec, not the code" discipline depends on spec templates, and presets make the template itself a versioned, upgradable artifact. And it adds a concrete packaging-and-inspection workflow to the trust concerns raised in [[Code Standards That Survive the Next Agent]] — the standard now travels as an installable package whose contents deserve review before the agent ever reads them.

---
*Sources: [[raw/creating-spec-kit-presets-step-by-step]], [[summary/creating-spec-kit-presets-step-by-step]]*
*Last updated: 2026-10-14*
