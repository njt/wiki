---
url: https://gist.github.com/lhl/f171eaea45df31a0b9287d7bf380657a
title: "Supply Chain Security for Software Developers"
author: lhl (GitHub)
date_fetched: 2026-05-15
date_published: 2026-04-02
---

Practical layered defenses against package supply chain attacks, written after the March-April 2026 wave of compromises targeting Trivy (TeamPCP), LiteLLM, and axios (Sapphire Sleet/UNC1069).

## Three Incidents

**Trivy (TeamPCP, March 19-22):** Misconfigured `pull_request_target` workflow stole a PAT. Incomplete credential rotation allowed TeamPCP to retain access. They force-pushed 75 of 76 `trivy-action` tags and all 7 `setup-trivy` tags to malicious commits, published an infected binary (v0.69.4), and harvested SSH keys, cloud credentials, Kubernetes tokens, Docker registry credentials, database passwords, and private keys from runner memory. A persistent systemd backdoor was installed on developer workstations.

**LiteLLM (TeamPCP, March 24):** Using PyPI credentials stolen from LiteLLM's CI pipeline (which ran Trivy without version pinning), TeamPCP published malicious versions 1.82.7 and 1.82.8. Version 1.82.8 used a `.pth` file executing "on _every_ Python process startup." The packages were live for roughly 3-5 hours. Docker image users were unaffected since the official image pins dependencies. LiteLLM has ~95 million monthly downloads.

**axios (Sapphire Sleet / UNC1069, March 31):** The npm account of the lead axios maintainer was compromised. Two malicious versions (1.14.1 and 0.30.4) were published directly via npm CLI using a stolen long-lived access token, bypassing the project's OIDC trusted publishing. The versions injected `plain-crypto-js` as a dependency whose `postinstall` script dropped a cross-platform RAT. Packages were live ~3 hours. axios has roughly 100 million weekly downloads.

## Minimum Individual Settings

### npm
Add to `.npmrc`:
```
min-release-age=7
ignore-scripts=true
save-exact=true
```
Always use `npm ci` (not `npm install`) in CI. Always commit `package-lock.json`. Don't run random `npx` commands.

### pnpm
In `pnpm-workspace.yaml`:
```
minimumReleaseAge: 10080
blockExoticSubdeps: true
trustPolicy: no-downgrade
```
In `.npmrc`: `save-exact=true`. Use `pnpm install --frozen-lockfile` in CI. Commit `pnpm-lock.yaml`. Run `pnpm approve-builds` to explicitly allow install scripts.

### Python (uv)
```
export UV_EXCLUDE_NEWER=$(date -d '7 days ago' -u +%Y-%m-%dT00:00:00Z)
uv sync --frozen
```
Commit `uv.lock`. Use `uv lock --check` to verify the lockfile is clean.

### Python (pip)
```
pip install \
  --require-hashes \
  --only-binary :all: \
  --uploaded-prior-to $(date -d '7 days ago' -u +%Y-%m-%dT00:00:00Z) \
  -r requirements.txt
```
`--require-hashes` enables all-or-nothing hash checking. `--only-binary :all:` prevents source distributions from executing arbitrary code during build. `--uploaded-prior-to` (pip 26.0+) implements the 7-day age gate. For generating pinned+hashed requirements: `uv pip compile --generate-hashes`.

### Bun
In `bunfig.toml`:
```
[install]
minimumReleaseAge = 604800
```
Bun refuses dependency lifecycle scripts by default -- only add packages to `trustedDependencies` after review.

### GitHub Actions
Pin all third-party actions to full commit SHAs, not mutable version tags. Bad: `actions/checkout@v4`. Good: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2`.
Additional hardening: `permissions:` to least privilege; avoid `pull_request_target` and `workflow_run` with untrusted code; use OIDC federation for cloud credentials; restrict allowed actions to verified creators; use ephemeral/JIT runners.

## The 7-Day Rule
"The single most effective defense against short-lived malicious publishes." The axios attack had a roughly 3-hour window before detection -- a 7-day policy would have avoided it. Emergency security patches with actual CVEs may bypass the age gate, but that should be a conscious decision.

## What Not To Do
- Don't run `npm install` in CI
- Don't use `npx` for random packages
- Don't allow install scripts by default
- Don't use version ranges (`^`, `~`)
- Don't pin GitHub Actions to tags
- Don't store secrets in CI environment variables unnecessarily
- Don't use `--extra-index-url` for private packages

## Defense Tiers

**Tier 1 -- Lockfiles and Frozen Installs:** Commit lockfiles, use frozen installs in CI, pin exact versions. "Your seatbelt."

**Tier 2 -- The 7-Day Cooldown:** Never install a package less than 7 days old. Compromised packages are typically detected and removed within hours to days.

**Tier 3 -- Disable Lifecycle Scripts:** `postinstall`/`preinstall` scripts are the primary execution vector. Set `ignore-scripts=true`. For Python, the `.pth` file technique is worse -- it runs on every Python process. Use `--only-binary :all:`, virtual environments, and containers.

**Tier 4 -- Hash Verification:** Lockfiles with integrity hashes ensure install fails if content differs. "Does not protect against publisher compromise -- the malicious LiteLLM 1.82.8 wheel passed all hash checks."

**Tier 5 -- Provenance and Attestation:** npm provenance can cryptographically prove a package came from a verified CI/CD pipeline. "Provenance only protects if you remove legacy publishing paths." Check with `npm audit signatures`. pnpm's `trustPolicy: no-downgrade` fails closed when trust evidence worsens.

**Tier 6 -- Runtime and Network Defense:** Egress filtering (the axios RAT called `sfrclak[.]com:8000`), secrets management via OIDC, sandboxed ephemeral builds, and push protection for secret scanning.

**Tier 7 -- Organizational:** SBOMs, private registry/proxy as a single chokepoint, dependency review on PRs, minimal dependencies, and rapid containment using `overrides` or constraints files.

## If You Think You Were Exposed
"Assume breach, not just 'bad dependency.'" Stop all installs/builds, rotate all secrets, check lockfiles for unexpected changes, check for IOCs (unexpected outbound connections, new system services, unfamiliar files), alert the team, audit CI/CD.

## Why Layered Controls
No single control is sufficient:
- Lockfile: protects against version drift but not first-time bad versions
- Hash verification: protects against registry tampering but not publisher compromise
- Age gate: protects against short-lived publishes but not slow-burn implants
- Script blocking: protects against install-time code execution but not runtime payloads or `.pth` injection
- Provenance: protects against unauthorized publishing but not compromised CI/CD or legacy token coexistence

"You want the layers together."

## Scope and Limitations
Covers npm, pnpm, Bun, pip, uv, and GitHub Actions -- "the ecosystems hit in the March-April 2026 incidents." Cargo, Go modules, Maven/Gradle acknowledged but not covered.

## References (19 total)
Blog posts from Aqua, Microsoft, Google Cloud, Snyk, Huntress, Wiz, Datadog, Palo Alto Networks, LiteLLM, and official documentation for npm, pnpm, pip, GitHub Actions/security features, SLSA, uv, and Dependabot.
