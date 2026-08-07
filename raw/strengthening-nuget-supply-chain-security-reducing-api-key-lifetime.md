---
url: https://devblogs.microsoft.com/dotnet/strengthening-nuget-supply-chain-security-reducing-api-key-lifetime/
date_fetched: 2026-08-07
---

Many developers publish packages to NuGet.org via API keys. In recent years, API keys and Personal Access Tokens (PATs) have been frequently used as an attack vector to gain access to systems. There are multiple problems with API keys and PATs; the two biggest ones are that they are loose strings that are easy to lose, and they have traditionally had long durations. We have a two-pronged solution to this: cut the API key duration and encourage developers to use NuGet Trusted Publishing. This post will discuss both.

**API key lifetime changes**

These changes are necessary to secure the NuGet ecosystem. Other package managers have made similar changes.

If you publish packages to NuGet.org, review your publishing workflows now. We strongly recommend moving to NuGet Trusted Publishing.

## Why are we making this change?

A NuGet.org API key is effectively a password for publishing packages. When a long-lived key is stored in a repository secret, copied between systems, or embedded in a build configuration, a disclosure can give an attacker an extended opportunity to publish unauthorized package versions.

Long-lived credentials are convenient, but they also significantly increase the impact of accidental disclosure. The NX console NPM package was recently compromised using stolen credentials. The attacker published a malicious NX Console package using the stolen credentials. According to the NX Dev Team, the package was activated 6000 times in 36 minutes, before it was taken down. Other similar attacks have had much wider blast radius.

## API Key Reduction Plan

We have a two-part plan for reducing API key duration that will start on August 17, 2026.

On August 17th:

- New API keys will be limited to a 30-day maximum duration.
- A 365-day duration will no longer be available for new API keys.

On November 1st:

- All keys created before August 17th will expire.

The changes don’t make API keys safe, but they reduce the duration by which a lost API key can be used.

## Recommended: Move to Trusted Publishing

Trusted Publishing is a modern publishing workflow that was launched in September 2025. It allows a CI/CD workflow to authenticate to NuGet.org through OpenID Connect (OIDC). An OIDC workflow presents a signed, short-lived identity token to the publishing pipeline. NuGet.org validates that token against a policy configured by the package owner and issues a temporary API key for that publishing operation. This is in contrast to retrieving a long-lived API key from a secret store.

Trusted Publishing provides several advantages:

- **No long-lived publishing secret**stored in your repository or CI/CD system.
- **Short-lived credentials**are created and expire automatically.
- **Workload identity validation**against the repository, workflow, and optional environment configured in your policy.
- **No user-maintained secrets**or secret rotation.
- **Reduced secret disclosure impact**compared with a long-lived and reusable API key.

See Trusted Publishing for setup instructions.

## If you continue using API keys

We recognize that package publishers use a range of CI/CD systems and that not every workflow can move at the same pace.

If you continue using API keys:

- Inventory every workflow that publishes to NuGet.org.
- Identify keys that were created before August 17, 2026.
- Update automation so it can safely use and rotate keys with shorter expiration.
- Use the narrowest package scope and permissions required.
- Never commit an API key to source control or include it in logs.
- Delete API keys that get lost or exposed, immediately.
- Confirm that expiration notifications reach an actively monitored account.
- Plan a migration to Trusted Publishing.
- Watch for new CI/CD environments that support NuGet.org Trusted Publishing.
- Package publishing through the NuGet.org web interface remains available for manual scenarios.

## Take action now

If you maintain packages on NuGet.org, please review your publishing configuration before August 17, 2026. For GitHub Actions and GitLab users, begin migrating to Trusted Publishing now. For other publishing environments, make sure your workflow can operate with API keys that expire within 30 days and be prepared for the expiration of remaining long-lived keys on November 1, 2026. We’ll share updates with the community as we add support for more CI/CD environments.

We appreciate the work package maintainers do for the .NET community. These changes are intended to reduce credential risk while giving publishers a clear path toward safer, automated package publishing. API key duration may be reduced further in the future. We welcome feedback as we continue improving the Trusted Publishing experience.

Cool. Glad it works with Azure DevOps CI/CD. Oh wait…

We have some packages published on AzureDevOps Pipelines and some are published by GitHub Actions, I’d migrated one publish to trusted publish on GitHub, but seems the trusted policy could only managed for each the GitHub repo/workflow, that makes it different to migrate since we have a lot separated repos for nuget packages.

Could we improve the user experience for the trusted publish before making change to the api keys? A short-time expiry for API key is safer while it means for more effort to maintaining the api key.

The lack of org-level support is why we haven’t adopted Trusted Publishing yet. Tracked at https://github.com/NuGet/NuGetGallery/issues/10581.

An individual who gets hit by a bus should not block the entire project/org.

This is going to be incredibly painful – we raised https://github.com/NuGet/NuGetGallery/issues/10690 back in January 2026, but nothing seems to have improved since.
