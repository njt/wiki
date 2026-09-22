---
url: https://github.com/github/spec-kit
title: "Spec Kit"
author: GitHub (github/spec-kit maintainers and contributors)
date_fetched: 2026-09-22
date_published: 2025-08-22
topics:
  - specifications-as-the-product
  - coding-agents-and-frameworks
---

GitHub's open-source toolkit for Spec-Driven Development (SDD): a Python CLI (`specify-cli`) that scaffolds a `.specify/` directory of templates and installs slash commands into 40+ coding agents in each agent's native format (markdown slash commands, TOML for Gemini, YAML recipes, SKILL.md skills directories). The coding agent — not the CLI — is the runtime: each `/speckit.*` command is a markdown "program" the LLM executes, with deterministic shell/Python scripts handling only the brittle parts (branch creation, prerequisite checks).

The core process is constitution → specify → plan → tasks → implement → converge, run once per project and once per feature respectively. The `specify` command turns a natural-language description into a technology-agnostic spec with a self-generated quality checklist; `plan` produces research.md, data-model.md, contracts/ and quickstart.md; `tasks` derives a parallelizable task list; `implement` executes it; `converge` re-assesses the codebase against the artifacts and appends remaining work as new tasks until it reports Converged. Two opt-in extensions add independent processes: bug fixing (assess → fix → test, ending in a verified/partial/failed verdict) and idea assessment (intake → research → define → shape → decide, ending in go / needs-clarification / kill).

The interesting engineering is around the prompts, not in them. A `IntegrationManifest` records a SHA-256 hash for every installed file so uninstall touches only unmodified files; a YAML workflow engine with 12 step types (including human `gate` steps that pause and resume from persisted state) drives agent CLIs as subprocesses; extension and workflow catalogs are fetched with SSRF checks, byte limits, and zip-member sanitization in a dedicated `_download_security.py`; and ruff rules S602/S604/S605 are locked in so `shell=True` cannot sneak back into the subprocess layer.

At v1.0.9 (2026-09-21, first release 2025-08-22, 1.0.0 in August 2026), the project is a striking inversion of expected proportions: ~60k lines of Python orchestration and 207 test files wrapped around roughly 3k lines of templates — because the templates *are* the product, and everything else exists to deliver them safely to heterogeneous agent runtimes.
