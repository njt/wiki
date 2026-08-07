# Supply Chain Security for Software Developers

Practical, layered defenses against package supply chain attacks -- written in the aftermath of the March-April 2026 TeamPCP campaign that compromised Trivy, LiteLLM, and axios. The gist is a wake-up call dressed as a configuration checklist. If you adopt nothing else: pin exact versions, use a 7-day cooldown on new packages, and disable install scripts.

---

## The Incidents

The gist opens with three case studies that establish the threat model: each attack was short-lived (3-5 hours), each exploited a different vector, and each would have been stopped by at least one of the recommended controls.

> On the Trivy attack: "An earlier breach via a misconfigured `pull_request_target` workflow stole a PAT. Incomplete credential rotation allowed TeamPCP to retain access."

The Trivy compromise was the most severe because it cascaded -- Trivy's CI ran with unpinned actions, which let TeamPCP pivot to LiteLLM's PyPI credentials. The attack harvested SSH keys, cloud credentials, Kubernetes tokens, and Docker registry passwords from runner memory, then installed a persistent systemd backdoor on developer workstations. This is the nightmare scenario that motivates the rest of the guide.

> On LiteLLM: "Version 1.82.8 used a `.pth` file executing on _every_ Python process startup."

The `.pth` mechanism is worse than npm's `postinstall` scripts because it fires on every Python interpreter start, not just at install time. Virtual environments and containers don't prevent it -- they just limit the blast radius. Docker image users were protected only because the official image pinned dependencies.

> On axios: "OIDC only works if the legacy token is removed -- the workflow also passed `NPM_TOKEN`, and npm uses the token when both are present."

This is the sharpest technical insight in the gist. axios had set up OIDC trusted publishing (the new, secure path) but left a long-lived access token in their CI variables. npm's resolver uses the token when both are present, so the attacker's stolen token won. Provenance is a wraparound porch with a hole in the wall.

---

## The 7-Day Rule

The central thesis: "the single most effective defense against short-lived malicious publishes." All three attacks were detected and removed within hours. A 7-day age gate would have blocked all three. The gist is careful to note this isn't a substitute for other controls -- slow-burn implants or long-compromised maintainer accounts can still slip through -- but as a low-cost first line of defense, it's uncommonly effective.

The rule has a built-in exception for emergency CVEs, but the gist insists that should be a conscious, reviewed decision, not the default.

---

## Configuration Recipes

The gist's most practical value is its copy-paste-ready configuration snippets for npm, pnpm, pip, uv, Bun, and GitHub Actions. Every recipe combines four controls: exact version pinning, lockfile enforcement, age gating, and script blocking.

For npm:

> "`save-exact=true` ... Always use `npm ci` (not `npm install`) in CI. Don't run random `npx` commands."

For pip, the recipe is more involved because Python's tooling hasn't converged on a single package manager:

> "`--require-hashes` enables all-or-nothing hash checking. `--only-binary :all:` prevents source distributions from executing arbitrary code during build. `--uploaded-prior-to` (pip 26.0+) implements the 7-day age gate."

For GitHub Actions, the rule is simple and often violated:

> "Pin all third-party actions to full commit SHAs, not mutable version tags. Bad: `actions/checkout@v4`. Good: `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2`."

---

## Defense in Depth

The gist's closing argument is a table showing that no single control is sufficient:

- **Lockfile**: protects against version drift but not first-time bad versions
- **Hash verification**: protects against registry tampering but not publisher compromise (the LiteLLM 1.82.8 wheel passed all hash checks)
- **Age gate**: protects against short-lived publishes but not slow-burn implants
- **Script blocking**: protects against install-time execution but not runtime payloads or `.pth` injection in Python
- **Provenance**: protects against unauthorized publishing but not compromised CI/CD or -- critically -- the legacy token coexistence that sank axios

> "You want the layers together."

This is the same argument made in [[Security and Sandboxing]]: no single constraint works. You need defense-in-depth that assumes every individual layer will be bypassed.

---

## Key Themes

#pattern -- Layered defense against supply chain attacks. The 7-day rule, pinning, and script blocking form a stack where each layer covers the others' blind spots.

#concept -- **The 7-day cooldown** is the novel contribution: apply a minimum release age to every dependency. It's simple, automatable, and would have stopped all three March-April 2026 incidents.

#tool -- pip 26.0's `--uploaded-prior-to` flag, npm's `min-release-age`, pnpm's `minimumReleaseAge`, and Bun's `minimumReleaseAge` are the ecosystem-specific implementations of the 7-day rule.

#concept -- **OIDC alone is not provenance**: the axios attack proves that provenance only works if you remove the legacy publishing path. Trusted publishing and long-lived tokens can't coexist.

---

## Critical Analysis

**What makes this gist valuable is its specificity.** Most supply chain security advice is vague ("be careful with dependencies"). This gist gives you exact `.npmrc` lines, exact pip flags, exact commit SHA patterns for GitHub Actions. It's actionable enough to implement in an afternoon and systematic enough to serve as a checklist for CI/CD hardening.

**The 7-day rule is brilliant in its simplicity but will be politically difficult in practice.** Teams will want fresh releases, especially of their own internal packages. The gist acknowledges this tension but undersells it -- the real challenge isn't technical, it's organizational. "We need this new feature" will always feel more urgent than "we might be compromised."

**The gist's ecosystem coverage reveals the real problem: fragmentation.** Every package manager has a different flag for the same concept (age gating), and some (pip) need three flags to achieve what npm does with one. This isn't a criticism of the gist -- it documents the mess accurately -- but it's a reminder that supply chain security is harder than it should be because the tooling is inconsistent.

**The OIDC + legacy token coexistence bug should scare you.** If you set up trusted publishing but forgot to delete your old `NPM_TOKEN`, you have the appearance of security without the reality. This pattern -- adding a new control without removing the old bypass -- is probably widespread.

**The registry-level response is arriving.** In August 2026, Microsoft announced NuGet.org will cut API key lifetimes from 365 days to 30 days and expire all legacy keys by November 2026, while pushing publishers toward OIDC-based Trusted Publishing. This is the same architectural pattern that failed axios — and the same coexistence danger applies: teams must delete old API keys after migrating, not leave them sitting in CI variables alongside the new OIDC flow. See [[NuGet API Key Lifetime Reduction]].

**The missing piece: developer experience.** The gist is configuration-focused, but nobody talks about what happens when a package is blocked by the age gate and a developer can't get their work done. The workflow for "consciously bypass the gate for this specific CVE fix" needs to be just as smooth as the gate itself, or the gate will be disabled wholesale.

---

*Sources: [[summary/supply-chain-security-software-developers]]*
*Last updated: 2026-05-15*
