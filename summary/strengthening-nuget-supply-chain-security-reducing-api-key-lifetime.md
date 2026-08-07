---
url: https://devblogs.microsoft.com/dotnet/strengthening-nuget-supply-chain-security-reducing-api-key-lifetime/
title: "Strengthening NuGet Supply Chain Security: Reducing API Key Lifetime"
author: NuGet Team (Microsoft)
site: Microsoft DevBlogs
date_published: 2026-08-06
date_fetched: 2026-08-07
---

# Summary: Strengthening NuGet Supply Chain Security

Microsoft announces a two-part plan to reduce NuGet.org API key lifetimes, coupled with a strong recommendation to adopt Trusted Publishing via OIDC.

## The Change

- **August 17, 2026**: New API keys limited to 30-day maximum. The existing 365-day option is removed.
- **November 1, 2026**: All keys created before August 17 expire.

The changes don't make API keys safe, but they dramatically reduce the window in which a lost or stolen key can be exploited. The post cites the NX console NPM package compromise — a stolen credential used to publish a malicious package activated 6,000 times in 36 minutes — as motivation.

## Trusted Publishing

Launched September 2025, Trusted Publishing replaces long-lived API keys with OIDC-based authentication. A CI/CD workflow presents a signed, short-lived identity token; NuGet.org validates it against a configured policy and issues a temporary API key scoped to that single publish operation. Benefits: no stored secrets, automatic expiry, workload identity validation, no rotation burden.

Trusted Publishing currently supports GitHub Actions and GitLab. Azure DevOps support is absent — a gap noted by multiple commenters, who also flagged the lack of org-level policy management as a blocker.

## For API Key Users

The post lists ten actions for teams that can't yet migrate: inventory publishing workflows, identify pre-August-17 keys, ensure automation handles rotation, use narrowest scope, never commit keys, delete exposed keys immediately, monitor expiration notifications, plan migration, and watch for new CI/CD support.

## Community Response

Commenters noted Trusted Publishing lacks Azure DevOps CI/CD support and org-level management (tracked at NuGet/NuGetGallery#10581). One raised #10690 from January 2026 about pain points that remain unaddressed. The "bus factor" concern — a single individual's account blocking an entire org — was also raised.
