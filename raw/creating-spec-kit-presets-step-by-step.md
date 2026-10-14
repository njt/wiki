---
url: https://jesseliberty.com/2026/10/09/creating-spec-kit-presets-step-by-step/
date_fetched: 2026-10-14
---

This guide assumes you already understand what Spec Kit and presets are and focuses on the who/what/where/how of authoring, testing, and publishing a preset. The instructions are concrete and practical — including a minimal manifest example, the recommended folder layout, local test workflow, a tiny C# validation snippet, and packaging/publishing notes.

- A preset is a versioned package that supplies template files, CLI prompts/companion prompt text, small scripts, and tiny content changes (terminology, defaults, localization). Presets are layered at runtime; when Spec Kit resolves a template or prompt, it walks the stack, and the highest-priority “winning” file is used.
- Presets are intended for content/style/localization changes. Avoid heavy runtime logic—use Bundles or Extensions if you need stateful automation.

**Step **1—design: decide scope, naming, and conventions

- Pick exactly what you will override:
- templates/—spec, plan, or task templates (Markdown).
- **commands**/—CLI command prompt text or companion .prompt.md files visible to users.
- scripts/—shell/PowerShell scripts used by templates or lifecycle hooks.
- **small localized wording changes, default values, or doc snippets.**
 
- Keep presets focused. *One preset should do one job*(branding, Spanish localization, trimmed plan structure, etc.).
- Choose an ID, version, and a default priority. Example: id = com.example.my-company-style, version = 0.1.0, priority = 8. Lower priority number = higher precedence (default is 10).
- Record author name/email and a short description in the manifest. These fields help consumers and catalogs list and audit presets.

**Step **2—create the folder structure and the manifest

- Create a folder for the preset. Example:
 my-company-style/
- At the root, create a manifest file named preset.yml and add subfolders for overrides:
- templates/
- commands/
- scripts/
- README.md
 

**Minimal example** preset.yml (place at my-company-style/preset.yml)

name: my-company-style

id: com.example.my-company-style

version: 0.1.0

description: “Company wording, trimmed plan structure, and Spanish localization.”

author:

name: Example Corp

email: infra@example.com

priority: 8

tags:

- style
- localization

**Notes about the manifest**:

- Spec Kit reads preset.yml to register the preset.
- The important required fields are id, name, and version; add description, author, priority, and tags for usability.
- You can extend the manifest with catalog metadata if you plan to publish.

**Step 3**—place overrides in the correct locations

- templates/: copy the exact core template filename you want to override (for example, plan-template.md) and edit it. The runtime resolution uses filenames/paths to match.
- commands/: put files that replace command prompt text or companion prompts. For example, speckit.constitution.md or .prompt.md files.
- scripts/: put helper scripts referenced by templates or lifecycle hooks.
- README.md: explain what the preset does, installation implications, and any migration guidance.

**Important**: keep the same filenames and relative paths used by the core Spec Kit repository (you can inspect core templates in the Spec Kit repo to find correct names). During installation, preset files are copied to .specify/presets// and spec resolution walks the stack automatically, so you don’t manually merge templates.

**Example**: override the plan template to add a C# code sample

- Create templates/plan-template.md and include a fenced C# block in the example section: …in the template’s code example area include: 

`// Example using our company logging convention``public class AccountService { private readonly ILogger _log;``public AccountService(ILogger log) => _log = log; }`

(Templates are plain Markdown — using C# samples inside them is fine.)

**Step 4** — local testing during development (fast feedback loop)

- Install your preset into a local project while developing:
 specify preset add –dev /path/to/my-company-style
- Run generation commands that exercise the templates or prompts you modified:
 specify plan –input spec=”Add user login”
- Inspect the generated output to verify your overrides took effect.
- Use Spec Kit helper commands to debug resolution:
 specify preset resolve
 specify artifact (or whichever command your Spec Kit provides to inspect which file “wins”)
- Iterate: change files in your local preset directory and re-run the generation command. The –dev install mode points to your local copy for quick iteration.

**Step **5—validate the manifest (quick programmatic check)

- A simple validation ensures required fields are present and helps catch YAML typos. Below is a tiny C# example using YamlDotNet that reads preset.yml and errors if id/name/version are missing.

**Commands**:

- Add the YamlDotNet package (run in your dotnet project directory):
 dotnet add package YamlDotNet

**C# Program** (Program.cs):

```
using System;
using System.IO;
using System.Collections.Generic;
using YamlDotNet.Serialization;
using YamlDotNet.Serialization.NamingConventions;
var yaml = File.ReadAllText("preset.yml");
var deserializer = new DeserializerBuilder()
.WithNamingConvention(CamelCaseNamingConvention.Instance)
.Build();
var preset = deserializer.Deserialize<Dictionary<string, object>>(yaml);
string[] required = new[] { "id", "name", "version" };
foreach (var r in required)
{
  if (!preset.ContainsKey(r))
  {
    Console.Error.WriteLine($"Missing required manifest field: {r}");
    Environment.Exit(2);
  }
}
Console.WriteLine($"Preset OK: {preset["name"]} v{preset["version"]}");
```
- dotnet run (or dotnet build + dotnet run) in the project containing Program.cs and preset.yml.
- Exit codes: 0 for OK, non-zero for missing required fields. Integrate this into CI to fail fast on manifest mistakes.

**Step **6—tests and CI

- Unit or integration tests: create tests that run specify commands in a controlled workspace (temp directory), generate artifacts, and assert expected strings or structural pieces exist.
- Snapshot testing: store small canonical generated files in tests and compare outputs after generation.
- Linting: run your YAML validator and any custom checks (e.g., disallow certain words, ensure semantic versioning).
- CI: run the validation and generation tests for pull requests. If your preset includes scripts, run them in CI to ensure cross-platform behavior or provide both shell and PowerShell variants.

**Step **7—package and publish

- Options for sharing:
- Publish the preset repository on GitHub/GitLab and reference it from a catalog.
- Host a catalog.json or catalog.yaml listing your preset metadata and URL. Consumers can add presets from catalogs.
- Use community tooling (e.g., presetify) to generate catalog entries and help with validation/publishing to community catalogs.
 
- For internal distribution, your team can host the preset repo or a catalog on an internal server and point projects at it.
- Ensure you tag releases using semantic versioning so consumers can pin versions: e.g., v0.1.0.

**Step 8**—installation and lifecycle for consumers

- Install from a catalog or URL:
 specify preset add
 specify preset add –version
 specify preset add –from
- Install locally for development:
 specify preset add –dev <path/to/preset>
- Remove:
 specify preset remove
- Upgrading a preset is a matter of updating the version and having consumers run an upgrade or specify preset add with a version.

**Best practices & warnings**

- Keep presets limited to content/formatting/localization. Don’t use presets to implement complex runtime automation.
- Follow semantic versioning. Document changes in README and changelog.
- Prefer small, focused presets over large monolithic presets; it’s easier to compose and upgrade.
- Security: don’t blindly trust third-party catalogs. Inspect preset contents before install and ensure your project policy permits external presets.
- Avoid breaking template API contracts. If you change placeholders or required variables in a template, document the change and bump major version.
- Use meaningful tags in preset.yml so consumers (and catalog tools) can filter presets: tags: [style, spanish, onboarding]
- Provide README.md with examples: “How this preset changes plan-template.md” and sample generated output.
- If you want to supply per-command prompts, include companion .prompt.md files under commands/ with clear naming.

Note: The initial draft of this post was written by BlogWriter and then

edited by Jesse Liberty.  Illustrations by Copilot. **Caution**: LLMs make mistakes; this post is offered as is.
