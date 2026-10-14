---
url: https://jesseliberty.com/2026/10/09/creating-spec-kit-presets-step-by-step/
title: "Creating Spec Kit Presets — Step by Step"
author: Jesse Liberty (initial draft by BlogWriter, illustrations by Copilot)
date_fetched: 2026-10-14
date_published: 2026-10-09
topics:
  - coding-agents-and-frameworks
  - specifications-as-the-product
---

Jesse Liberty's practical tutorial on authoring, testing, and publishing presets for Spec Kit, GitHub's spec-driven development toolkit. A preset is a versioned package that supplies template files, CLI command prompt text, companion `.prompt.md` files, helper scripts, and small content changes (terminology, defaults, localization). Presets are layered at runtime: when Spec Kit resolves a template or prompt, it walks the stack and the highest-priority file "wins" (lower priority number = higher precedence; default is 10).

The guide walks eight steps: design the scope (templates, commands, scripts, small wording changes — one preset, one job); create the folder structure with a `preset.yml` manifest (required fields: id, name, version; plus description, author, priority, tags); place overrides in the correct paths keeping exact core filenames, since runtime resolution matches by path and installs into `.specify/presets/`; test locally with `specify preset add --dev` for a fast iteration loop, using `specify preset resolve` to debug which file wins; validate the manifest programmatically (a small C# YamlDotNet snippet checking required fields, suitable for CI); add tests and CI (snapshot tests of generated artifacts, YAML linting, cross-platform script checks); package and publish (GitHub/GitLab repos, catalog.json/yaml listings, community tooling like presetify, semantic-version tags); and consumer lifecycle (`specify preset add/remove`, version pinning, upgrades).

Best practices: keep presets to content/formatting/localization only — use Bundles or Extensions for stateful automation; prefer small focused presets over monolithic ones; document template contract changes and bump major versions when placeholders change; and a security caution not to blindly trust third-party catalogs — inspect preset contents before installing. The post is explicitly AI-assisted: "the initial draft of this post was written by BlogWriter and then edited by Jesse Liberty," offered "as is."
