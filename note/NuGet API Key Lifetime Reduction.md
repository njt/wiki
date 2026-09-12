# NuGet API Key Lifetime Reduction

Microsoft is cutting NuGet.org API key lifetimes from 365 days to 30 days (August 2026), expiring all legacy keys by November 2026, and pushing package publishers toward OIDC-based Trusted Publishing. The change mirrors similar moves by other package registries and is motivated by the same stolen-credential attack pattern that compromised npm's `nx` package. The Trusted Publishing architecture is sound but gated on CI/CD support that doesn't yet cover Azure DevOps — the natural home for many .NET teams.

---

## Key Quotes

> "A NuGet.org API key is effectively a password for publishing packages. When a long-lived key is stored in a repository secret, copied between systems, or embedded in a build configuration, a disclosure can give an attacker an extended opportunity to publish unauthorized package versions."

This is the same argument made in [[You Dont Want Long-Lived Keys]]: long-lived credentials are convenient and catastrophic. The nuance here is that NuGet keys are *passwords*, not tokens — they can be reused indefinitely until they expire. There's no built-in revocation mechanism short of deleting the key.

> "The changes don't make API keys safe, but they reduce the duration by which a lost API key can be used."

An honest admission. This is harm reduction, not a fix. The real fix is Trusted Publishing. The 30-day window is still long enough for a patient attacker to do damage — the NX compromise played out in 36 minutes — but it eliminates the "key sitting in a leaked `.env` file for 11 months before anyone notices" scenario.

> "Trusted Publishing allows a CI/CD workflow to authenticate to NuGet.org through OpenID Connect (OIDC). An OIDC workflow presents a signed, short-lived identity token to the publishing pipeline. NuGet.org validates that token against a policy configured by the package owner and issues a temporary API key for that publishing operation."

This is the same architecture that PyPI and npm adopted for their trusted publishing. The OIDC token is signed by the CI/CD provider (GitHub, GitLab), not the user, so the publishing pipeline never sees a long-lived secret. It's the [[You Dont Want Long-Lived Keys]] principle operationalized at the package registry level.

> From the comments: "The lack of org-level support is why we haven't adopted Trusted Publishing yet."

This is the real adoption blocker. Individual repo-level policy configuration doesn't scale for organizations with dozens or hundreds of NuGet packages. Tracked at `NuGet/NuGetGallery#10581`.

> From the comments: "An individual who gets hit by a bus should not block the entire project/org."

The "bus factor" problem with per-user API keys. When publishing credentials are tied to an individual's account rather than an org-level identity, that person becomes a single point of failure — both operationally and, if their key is compromised, as an attack vector.

---

## Key Themes

#concept — **API key lifetime reduction as harm reduction.** Cutting from 365 to 30 days doesn't eliminate the threat of stolen credentials, but it shrinks the exposure window from "forever" to "this month." NuGet is explicit that further reductions may follow. The trajectory is toward hours, not days.

#concept — **OIDC Trusted Publishing as the real solution.** The architectural pattern — CI/CD provider signs an identity token, registry validates it against a policy, issues a single-use API key — eliminates the long-lived secret entirely. This is the same model adopted by PyPI (trusted publishers) and npm (provenance + trusted publishing). The [[Supply Chain Security for Software Developers]] discussion of axios's OIDC failure is the cautionary companion: trusted publishing only works if you *remove* the legacy token. Having both is worse than having neither — it gives the appearance of security without the reality.

#tool — **NuGet Trusted Publishing** (launched September 2025). Supports GitHub Actions and GitLab. Azure DevOps is the conspicuous absence. The gap isn't just a missing feature — it's a structural mismatch between Microsoft's own CI/CD product and its own package registry.

#pattern — **Credential rotation as a forcing function for modernization.** By expiring all legacy keys on November 1, 2026, Microsoft is making continued API key use *deliberately painful*. The 30-day rotation cycle is sustainable for small teams but breaks at scale — the intended effect is to make Trusted Publishing the path of least resistance.

#concept — **Org-level identity as the unsolved problem.** Individual API keys and per-repo Trusted Publishing policies both fail the "bus factor" test. The community is asking for org-level policy management — configure once, apply to all repos. This is the same tension between individual and organizational identity that runs through [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] and [[The Agent Access Model]].

---

## Critical Analysis

**The timing is late but defensible.** npm and PyPI made similar moves earlier. NuGet is a follower here, not a leader, but the .NET ecosystem's slower adoption of CI/CD automation (many teams still publish manually or from on-prem Azure DevOps) makes a more gradual transition politically necessary. The question is whether the 30-day window is *too* gradual — it doesn't stop an attacker who steals a key on day 2 and uses it for 28 days.

**The Azure DevOps gap is structural, not incidental.** NuGet Trusted Publishing supports GitHub Actions and GitLab — both competitors to Azure DevOps. The most natural publishing path for .NET enterprise teams (Azure DevOps Pipelines → NuGet.org) doesn't support the recommended security model. This isn't a missing checkbox; it's Microsoft's left hand not talking to its right hand. The commenter who pointed this out is doing the most important work in the thread.

**Org-level policy management is the adoption ceiling.** The current Trusted Publishing model requires configuring OIDC policies per-repo. For an org with 100 NuGet packages across 50 repos, that's an unreasonable operational burden — especially when the alternative is "just keep using API keys, they still work for 30 days." Until org-level policy exists, Trusted Publishing only scales down (to solo maintainers) and up (to GitHub Actions-native teams), leaving a large middle excluded.

**The bus-factor problem is real and underappreciated.** When publishing is tied to an individual's API key or GitHub account, that person is a single point of failure for the entire package — both for legitimate publishing (they leave the company) and for security (their credentials are the attack surface). The [[You Should Not Update Your Dependencies in 2026]] thesis applies here too: dependency infrastructure is only as secure as its weakest individual maintainer.

**The `nx` compromise citation is well-chosen but incomplete.** Mentioning the NX console npm attack establishes the threat model, but the post doesn't connect it to the axios OIDC lesson from [[Supply Chain Security for Software Developers]] — that trusted publishing and legacy tokens *must not coexist*. Teams migrating to Trusted Publishing need to be told explicitly: delete your old API keys after verifying the OIDC flow works. The post implies this but doesn't state it with the force it deserves.

**What's good: the operational checklist.** The ten-point action list for API key users is practical and honest. "Inventory every workflow," "use the narrowest scope," "never commit an API key" — these are table-stakes hygiene, but the fact that they need to be stated says something about the state of NuGet publishing practices. The inclusion of "watch for new CI/CD environments that support Trusted Publishing" is a nice touch — it signals that the current support matrix is a floor, not a ceiling.

**What's missing: a migration timeline for Azure DevOps.** The post says "we'll share updates with the community as we add support for more CI/CD environments" without naming Azure DevOps specifically. For the largest segment of .NET enterprise publishers, this is the only question that matters. A concrete ETA — even a rough one — would do more for adoption than the rest of the post combined.

---

*Sources: [[raw/strengthening-nuget-supply-chain-security-reducing-api-key-lifetime]], [[summary/strengthening-nuget-supply-chain-security-reducing-api-key-lifetime]]*
*Last updated: 2026-08-07*
