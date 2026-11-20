# Octopus, Gecko and Tortoise — Scanners Hunting AI Keys

A nine-week passive observation of three secrets-scanning botnets that spoof AI crawler identities and specifically target the credential files that AI coding agents (Claude Code, Codex, Cursor, Gemini CLI) leave in home directories and project folders. The notable finding is not that scanners probe `.env` — that is old — but that the target lists now track the agent ecosystem file-by-file, and that the agent-config files themselves have become the new secret-exposure surface.

---

The author (Emre, tin.computer) watched three scanners on ten sites he controls from late September to 18 November 2026, and named them after their signatures:

- **Octopus** — spoofs 32 AI crawler names, one per request, from 97 ephemeral Google Cloud machines across 15 regions. Browses like Chrome first, fingerprints the framework (static / Next.js / Vite get different path lists), then fires ~300 secret-path requests in ten seconds. A canary `/z9x8c7v6b5-debug-trigger-<hostname>` opens every run.
- **Gecko** — a .NET program (matched byte-for-byte against .NET 9's HttpClient handshake) whose Chrome 128 user agent is cut at exactly 60 characters, deleting the literal string "like Gecko". Runs from Google Cloud Belgium plus €5–45/month offshore Windows VPS. ~200 fixed-path requests in under a second, and its AI-tool path count grew from 4 to 26 in five weeks.
- **Tortoise** — a small Go scanner, one request every few seconds, probing Spring/PHP/Rails debug pages. Seen twice.

---

## Key quotes

> If GPTBot or ClaudeBot asks your site for `/.env`, it is not OpenAI or Anthropic. It is a scanner we call Octopus.

The single most operationally useful line in the piece. Real crawlers read pages; secret-file requests with crawler names all came from rented machines. Identity-by-user-agent is dead as a security primitive.

> A name in a list does not show that a company runs a crawler with that name. Check the address.

The sharpest indictment here is of the community, not the attacker: 29 of Octopus's 32 fake names sit in the popular ai.robots.txt allowlist, and the pull request that added Hunyuan and MoonshotBot noted the traffic looked like vulnerability probes — and was merged anyway. Blocking by name punishes real crawlers while the scanners just rotate names.

> AI tool files were about 4% of Gecko's requests in October. In the week of 16 November they were 10%.

The targeting shift is the story. Gecko asks for `/.claude/.credentials.json`, `/.codex/auth.json`, `/.cursor/mcp.json`, `/.gemini/settings.json` — the exact files where coding agents store logins and MCP server keys. The scanners are adapting to agent adoption faster than most defenders have even considered these files sensitive.

> Those files reach a website when a build copies the whole project into its output, when a container serves the wrong folder, or when someone commits `.claude/settings.local.json`.

This is the novel exposure mechanism: agent configuration living in the project tree means the traditional deploy pipeline is now a credential-exfiltration channel. Your `.gitignore` and build output are part of your agent threat model.

> Octopus adds new exploit paths one to three weeks after a run of advisories. The time between a public bug and a request for it on your site is short, and it is getting shorter.

The scanners are maintained software, not abandoned scripts: Langflow/Flowise paths appeared two weeks after advisories and were removed a day later; LiteLLM key-exposure paths arrived 12 November; the home-folder paths were dropped 24 October as if pruned.

## Themes

#concept #security #tool #pattern

- **Identity spoofing at scale** — crawler names as camouflage, ephemeral infrastructure (a fresh GCE machine per target, most living under an hour) defeating IP-based blocking.
- **Agent credentials as the new attack surface** — the home-directory agent config files (`~/.claude/.credentials.json`, `mcp.json`) as loot, and the build pipeline as the leak path.
- **Behaviour over identity** — block dotfile paths (except `/.well-known/`), the truncated user agent, and canary paths; never block by crawler name.
- **Fingerprinting before exploitation** — both scanners read the home page, robots.txt and build manifests first, then tailor the request list to the framework.

## Analysis

This is one of the best pieces of empirical attacker research in the agent-security space: nine weeks of passive observation, TLS-fingerprint attribution (Go vs .NET identified from cipher lists, verified by replaying the handshake), infrastructure forensics down to SSH host keys used to prove two distinct operators, and an honest separation of "Found" from "Not proven". The authors even connected to live attacker machines and noted that two vanished within minutes — evidence someone was watching back.

The important twist for this wiki's readers: the agent ecosystem has created a new class of secrets that neither scanners nor defenders fully catalogued six months ago. Gecko's list is effectively a *census of where coding agents store credentials*, maintained as attentively as a product roadmap. That means every new agent tool that writes a config file with an API key into a project directory is, by default, publishing a target. The defence side is unglamorous but concrete: 404-check your own secret paths, `find public dist out -name '.*'` before deploy, gitignore the agent state, never expose a Vite dev server (22 dedicated Octopus paths), and rotate anything that was ever public — "scanners use keys within seconds".

The caveat worth holding: this is one observer, ten sites, self-selected traffic, and the operator attribution is inference (strong inference, but inference). The conclusion that blocking AI crawler names is useless is solid; the tighter claim that these particular operators are criminal rather than, say, aggressive security researchers is deliberately left unproven by the author — appropriately.

## Related pages

This source strengthens [[Defending Against an ARTEX AI-Driven Attack (Eye Security)]] with the reconnaissance side of the same story — Eye Security documented stolen secrets being used within minutes; this piece documents the industrial scanning layer that harvests them, and both converge on the collapsed find-to-use gap. It nuances [[OpenShell]]'s credential-injection-at-the-proxy design: NVIDIA's answer to "agents never hold secrets" is the platform fix for exactly the leak surface Gecko farms, where agent-held credential files end up in deployable artifacts. It complicates the [[Claude Code]] topic page: the product's own credential storage locations (`/.claude/.credentials.json`, `settings.local.json`) are now enumerated targets in attacker tooling, which is a operational-security dimension the configuration guides don't discuss. And it gives [[MCP-FSD — Filesystem Capability Laundering in MCP Servers]] a field companion: that paper measures filesystem-capability risks in MCP servers themselves, while Gecko shows attackers already probing `mcp.json` files for the keys those servers hold.

---
*Sources: [[raw/fake-ai-crawler-secrets-scanners]], [[summary/fake-ai-crawler-secrets-scanners]]*
*Last updated: 2026-11-20*
