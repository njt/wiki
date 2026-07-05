---
url: https://github.com/sleuth-io/sx
date_fetched: 2026-07-05
backfilled: true
---

⭐ Star this repo · 🌐 Website · 📋 Changelog · 📄 License

AI assets — skills, MCPs, agents, rules, commands, hooks — usually live inside a single Git repo. The moment you want them in another repo you copy-paste, and they drift out of sync with no source of truth. `sx` manages complex sharing and distribution of AI assets for real-world teams:

- **Share across projects and teams**— manage an asset once, install it into any repos or teams
- **Share across clients, including the web ones**— one asset installs into every major AI assistant
- **No Git knowledge required**— gives non-technical users easy access; the plumbing stays hidden
- **Install the right assets, not all of them**— scope to an org, repo, path, team, bot, or user, no context bloat
- **Version, observe, and govern**— update once; track adoption with- `sx stats`; audit access with- `sx audit`

**Install via Homebrew (macOS/Linux):**

```
brew tap sleuth-io/tap
brew install sx
```
**Or via shell script:**

`curl -fsSL https://raw.githubusercontent.com/sleuth-io/sx/main/install.sh | bash`Then initialize in your vault or project:

`sx init`From here the workflow is three steps — **manage** your assets, **distribute** them, and **govern** what ships.

Add assets to your vault (`sx` auto-detects the type):

```
sx add /path/to/my-skill
sx add ~/.claude/skills/my-skill            # an existing Claude Code skill
sx add code-review@claude-plugins-official  # a plugin from a registry
```
Your AI assets stay exactly as they are — `sx` just wraps them with metadata for versioning and stores them in its vault format.

**Multiple vaults?** Use profiles to switch between them:

```
sx profile add work        # Add a new profile
sx profile use work        # Switch to it
sx profile list            # See all profiles
```
**See what's actually used** — track adoption and token usage across your team:

```
sx stats                   # adoption dashboard
sx stats --since 7d --json # machine-readable
```
```
# Install everything scoped to you, into the current project
sx install
```
**Install targets** — pick who sees which asset:

```
sx install my-skill --org                               # everyone in the vault
sx install my-skill --repo github.com/acme/infra        # only inside that repo
sx install my-skill --path github.com/acme/infra#docs/  # one path in a repo
sx install my-skill --team platform                     # every member of a team
sx install my-skill --user alice@acme.com               # a single user
sx install my-skill --bot python-backend                # a bot identity (CI runner, agent)
```
See docs/scoping.md for the full overview and links to a per-scope doc for each install target.

**Use your vault from claude.ai or chatgpt.com** — expose it as an MCP
endpoint via the skills.new relay:

```
sx cloud connect       # opens skills.new, paste back the attach line
sx cloud serve         # keep this running — Ctrl+C exits
sx cloud status        # prints the MCP URL to paste into claude.ai / chatgpt.com
```
The relay forwards requests over a WebSocket — vault content stays local. See docs/cloud-relay.md.

**Audit** — every team and install mutation is recorded:

```
sx audit                                   # recent team/install mutations
sx audit --actor alice@acme.com --since 30d --event install.set
```
**Migrate a whole vault** (assets + versions, teams, bots, scopes, audit, usage):

```
sx vault copy --from skills-new --to git-vault   # preview (read-only)
sx vault copy --from skills-new --to git-vault --yes
```
See docs/copy.md for directionality and what's lossy. A gated change-request flow (RBAC) is on the roadmap.

- **Skills**- Custom prompts and behaviors for specific tasks
- **Rules**- Coding standards and guidelines that apply to specific file types or paths
- **Agents**- Autonomous AI agents with specific goals
- **Commands**- Slash commands for quick actions
- **Hooks**- Automation triggers for lifecycle events
- **MCP Servers**(experimental) - Model Context Protocol (MCP) servers for external integrations
- **Plugins**- Claude Code plugin bundles with commands, skills, and more

An AI agent is only as good as what it knows and what it's allowed to do. `sx` lets you describe that **once** — define a Bot and attach the agent's prompt plus the skills, rules, commands, hooks, and MCP servers it depends on — and install it unchanged across any client, coding or not.

- **One definition, every tool**— the same agent runs unchanged on every supported client, coding or not;- `sx`writes each one's native format on install
- **Bundle the whole capability**— skills, rules, commands, hooks, and MCP config travel together as versioned assets, not loose files scattered across repos and machines
- **Decoupled from any one vendor**— AI tools are commoditizing; the agents running on them shouldn't be locked in. Describe them in a portable format you own and carry them between tools as the landscape shifts

Choose the right distribution model for your team:

Perfect for easily sharing personal tools across multiple personal projects

`sx init --type path --path my/vault/path`Share assets through a shared git vault

`sx init --type git --repo git@github.com:yourteam/skills.git`Centralized management with a UI for discovery, creation, sharing, and usage analytics

`sx init --type sleuth`sx follows the manifest-and-lock pattern used by npm, cargo, and uv:

- **Manifest (**— the vault's source of truth. Lists every managed asset, its install scopes (- `sx.toml`)- `org`,- `repo`,- `path`,- `team`,- `bot`,- `user`), and team definitions (members, admins, repositories). Committed to git / path vaults. See docs/manifest-spec.md.
- **Lock file**— a per-user resolved artifact.- `sx install`reads the manifest, resolves team and user scopes against the caller's git identity, and writes the result to the user's cache directory (- `~/<cache>/sx/lockfiles/`). When the resolved lock changes, the previous file is rotated with a timestamp so old installs stay reproducible.
- **Audit + usage streams**— every team/install mutation appends an audit entry to- `.sx/audit/YYYY-MM.jsonl`; usage events append to- `.sx/usage/YYYY-MM.jsonl`. Query them with- `sx audit`/- `sx stats`.

High level: **manage** assets in one vault, **distribute** them
globally, per repo, per path, per team, per bot, or per user —
auto-installing on new Claude Code sessions so everyone stays in sync — and
**govern** every change through the audit and usage streams.

Everything the CLI does to a vault is also a package. Publish skills and
agents, manage bots and teams, and browse or download assets from your own
program against Skills.new, Git, or local Path vaults through one `Client`:

```
import "github.com/sleuth-io/sx/pkg/sxvault"
ctx := context.Background()
client, err := sxvault.OpenSkillsNew("https://app.skills.new", token)
if err != nil {
    log.Fatal(err)
}
if err := client.PutSkillZip(ctx, sxvault.SkillZipSpec{
    Name: "lint-helper", Version: "1.0.0", ZipData: zip,
}); err != nil {
    log.Fatal(err)
}
```
See docs/library.md for the full API guide.

| Client | Status | Notes | 
|---|---|---|
| Claude Code | ✅ Supported | Full support for all asset types | 
| Cline | ✅ Supported | Skills, rules, workflows as commands, MCP servers, hooks | 
| Codex | ✅ Supported | Skills, commands, agents, MCP servers | 
| Cursor | ✅ Supported | Skills, rules, commands, MCP servers, hooks | 
| GitHub Copilot | ✅ Supported | Skills, rules, commands, agents, MCP servers, local hooks | 
| Gemini (CLI/VS Code) | ✅ Supported | Skills, rules, commands, MCP servers, hooks | 
| Gemini (JetBrains) | ✅ Supported | Rules, MCP servers only (no commands/hooks) | 
| Gemini (Android Studio) | ✅ Supported | Rules, MCP-remote only (HTTP, no stdio) | 
| Kiro | ✅ Supported | Skills, rules, commands, MCP servers | 
| Openclaw | ✅ Supported | Skills, rules, commands | 
| OpenCode | ✅ Supported | Skills, commands, agents, rules, MCP servers | 
| claude.ai (web) | ✅ Supported | Via the skills.new cloud relay | 
| chatgpt.com (web) | ✅ Supported | Via the skills.new cloud relay | 

- ✅ Local, Git, and Skills.new vaults
- ✅ Claude Code support
- ✅ Cline support
- ✅ Cursor support
- ✅ GitHub Copilot support
- ✅ Gemini support
- ✅ Codex support
- ✅ Kiro support
- ✅ Openclaw support
- ✅ OpenCode support
- ✅ claude.ai and chatgpt.com support via the skills.new cloud relay
- ✅ Org, Team, Bot, Repository & Personal installation targets for all vault types
- ✅ Skill discovery - Use Skills.new to discover relevant skills from your code and architecture
- ✅ Analytics - Track skill usage and impact
- **RBAC and change request flow**- Support a gated skill update flow

See LICENSE file for details.

## Click to expand development instructions

- Vault Spec - Vault directory structure
- Manifest Spec - sx.toml source-of-truth format (assets, scopes, teams)
- Lock Spec - Per-user resolved lock file
- Scoping - Install targets and links to per-scope docs (orgs, repos, teams, users, bots)
- Vault copy - `sx vault copy`cross-vault migration (assets, teams, bots, scopes, audit, usage)
- Audit log - Event catalog, `sx audit`filters, storage format
- Usage analytics - `sx stats`dashboard, JSON output, event format
- Metadata Spec - Asset metadata format
- MCP Spec - MCP server and query tool
- Profiles - Multiple configuration profiles
- Clients - Client support model and IDE vs CLI limitations
- Cloud relay - Expose your vault to claude.ai and chatgpt.com via skills.new
- Library - Use sx as a Go library via the `pkg/sxvault`public API

Go 1.25 or later is required. Install using gvm:

```
# Install gvm
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
# Install Go (use go1.4 as bootstrap if needed)
gvm install go1.4 -B
gvm use go1.4
export GOROOT_BOOTSTRAP=$GOROOT
gvm install go1.25
gvm use go1.25 --default
```
```
make init           # First time setup (install tools, download deps)
make build          # Build binary
make install        # Install to GOPATH/bin
```
```
make test           # Run tests with race detection
make format         # Format code with gofmt
make lint           # Run golangci-lint
make prepush        # Run before pushing (format, lint, test, build)
```
Tag and push to trigger automated release via GoReleaser:

```
git tag v0.1.0
git push origin v0.1.0
```
