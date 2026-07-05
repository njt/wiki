---
url: https://github.com/Aaronontheweb/dotnet-slopwatch
date_fetched: 2026-07-05
backfilled: true
---

**// LLM anti-cheat**

  A .NET tool that detects LLM "reward hacking" behaviors in code changes.

Runs as a Claude Code hook or in CI/CD pipelines to catch when AI coding assistants take shortcuts instead of properly fixing issues.

When LLMs generate code, they sometimes take shortcuts that make tests pass or builds succeed without actually solving the underlying problem. These patterns include:

- **Disabling tests**instead of fixing them (- `[Fact(Skip="flaky")]`)
- **Suppressing warnings**instead of addressing them (- `#pragma warning disable`)
- **Swallowing exceptions**with empty catch blocks
- **Adding arbitrary delays**to mask timing issues (- `Task.Delay(1000)`)
- **Project-level warning suppression**(- `<NoWarn>`,- `<TreatWarningsAsErrors>false</TreatWarningsAsErrors>`)
- **Bypassing Central Package Management**with- `VersionOverride`or inline- `Version`attributes
- And more...

Slopwatch catches these patterns before they make it into your codebase.

```
# Install as a global tool
dotnet tool install --global Slopwatch.Cmd
# Or install locally
dotnet new tool-manifest
dotnet tool install --local Slopwatch.Cmd
```
```
# 1. Install slopwatch
dotnet tool install --global Slopwatch.Cmd
# 2. Initialize in your project (creates baseline from existing code)
cd your-project
slopwatch init
# 3. Commit the baseline to your repository
git add .slopwatch/baseline.json
git commit -m "Add slopwatch baseline"
# 4. From now on, only NEW slop is detected
slopwatch analyze
```
The baseline approach ensures slopwatch catches **new** slop being introduced without flagging legacy code. Your CI/CD pipeline will fail if someone introduces new slop patterns.

```
# Create baseline from existing code
slopwatch init
# Force overwrite existing baseline
slopwatch init --force
```
This creates `.slopwatch/baseline.json` containing all existing detections. Commit this file to your repository.

```
# Analyze current directory (requires baseline by default)
slopwatch analyze
# Analyze specific directory
slopwatch analyze -d src/
# Skip baseline and report ALL issues (not recommended for CI)
slopwatch analyze --no-baseline
```
When you intentionally add code that triggers slopwatch (with proper justification):

```
# Add new detections to existing baseline
slopwatch analyze --update-baseline
```
```
# Human-readable console output (default)
slopwatch analyze
# JSON for programmatic use
slopwatch analyze --output json
```
```
# Fail if errors found (default)
slopwatch analyze --fail-on error
# Fail on warnings too
slopwatch analyze --fail-on warning
```
If you're concerned about performance on large projects, use `--stats` to see how many files are being analyzed and how long it takes:

```
slopwatch analyze --stats
# Output: Stats: 44 files analyzed in 1.64s
```
| Rule | Severity | Description | 
|---|---|---|
| SW001 | Error | Disabled tests via Skip, Ignore, or #if false | 
| SW002 | Warning | Warning suppression via pragma or SuppressMessage | 
| SW003 | Error | Empty catch blocks that swallow exceptions | 
| SW004 | Warning | Arbitrary delays in test code (Task.Delay, Thread.Sleep) | 
| SW005 | Warning | Project file slop (NoWarn, TreatWarningsAsErrors=false) | 
| SW006 | Error | CPM bypass via VersionOverride or inline Version attributes | 

Add slopwatch as a hook to catch slop patterns during AI-assisted coding. Add the following to your project's `.claude/settings.json`:

```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "slopwatch analyze -d . --hook",
            "timeout": 60000
          }
        ]
      }
    ]
  }
}
```
The `--hook` flag enables Claude Code integration mode which:

- **Uses**- makes hooks near-instant even on large repos- `git status`to only analyze dirty files
- Outputs errors to stderr in a readable format
- Suppresses all other output
- Blocks on warnings and errors
- Exits with code 2 on failure (blocking the edit)

Claude will see the formatted error message and can then fix the issue properly.


Note:Hook mode requires git. If git is unavailable, it falls back to full analysis.

```
- name: Install Slopwatch
  run: dotnet tool install --global Slopwatch.Cmd
- name: Run Slopwatch
  run: slopwatch analyze -d . --output json --fail-on error
```
**Note:** The baseline file (`.slopwatch/baseline.json`) should be committed to your repository. Run `slopwatch init` locally first.

```
- task: DotNetCoreCLI@2
  displayName: 'Install Slopwatch'
  inputs:
    command: 'custom'
    custom: 'tool'
    arguments: 'install --global Slopwatch.Cmd'
- script: slopwatch analyze -d . --fail-on error
  displayName: 'Run Slopwatch'
```
Create a `.slopwatch/config.json` file to define suppressions:

```
{
  "suppressions": [
    {
      "ruleId": "SW002",
      "pattern": "**/Generated/**",
      "justification": "Generated code from tooling cannot be manually changed"
    },
    {
      "ruleId": "SW006",
      "pattern": "src/Legacy/**",
      "justification": "Legacy CPM migration in progress; tracked in issue #123"
    }
  ],
  "globalSuppressions": []
}
```
Use the `-c` or `--config` option to specify a custom suppression config file location:

`slopwatch analyze -d . --config path/to/config.json`To exclude files or directories from analysis entirely, use `--exclude`:

`slopwatch analyze -d . --exclude "**/Generated/**,**/obj/**,**/bin/**"````
dotnet build
dotnet test
dotnet pack
```
- Fork and create a feature branch
- Make changes and add tests
- Submit a pull request

Note: This project uses slopwatch on itself - your PR will be analyzed for slop patterns!

Apache 2.0 - see LICENSE for details.

- Slopometry - Python equivalent for Claude Code
- Incrementalist - Git diff analysis patterns
