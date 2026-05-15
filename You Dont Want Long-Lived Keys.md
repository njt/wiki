# You Don't Want Long-Lived Keys

The case for ephemeral credentials: systems built around keys valid for roughly one day or less sidestep the rotation problem entirely, because "rotation" is a built-in feature. Replacing long-lived keys with ephemeral keys is one of the best uses of security engineering effort.

---

## Key Quotes

> "Replacing long-lived keys with ephemeral keys is, for my money, one of the best uses of security engineering effort."

## Key Themes

#security #agent-architecture

Three compounding risks make long-lived keys dangerous: departing employees retain knowledge of active credentials, attackers have more time to guess or steal them, and cryptographic keys degrade in security effectiveness after extended use.

Ephemeral keys eliminate all three by design. Practical examples:

- **SSH**: EC2 Instance Connect replaces static SSH keys with temporary credentials that require fresh authentication
- **Package publishing**: GitHub Actions "trusted publishers" generate temporary PyPI tokens instead of static secrets
- **Authentication**: SSO replaces passwords with ephemeral signed assertions

The article acknowledges that some long-lived keys are unavoidable (IdP signing keys, root certificates) but argues for minimizing their count so security rigor can be concentrated.

This principle applies doubly to AI agents. Agents running in [[Navaris]] or [[OpenSandbox]] sandboxes need API credentials, and those credentials should be as short-lived as possible. [[onecli]] addresses this from the other direction -- injecting credentials transparently so agents never hold them directly.

## Critical Analysis

This is a clear, well-argued piece that states a principle most security practitioners agree with but few organizations implement. The practical examples (EC2 Instance Connect, trusted publishers, SSO) are concrete and actionable. What's missing is a discussion of the failure modes of ephemeral key systems -- the "what happens when the token issuer goes down?" question that makes teams reluctant to abandon static credentials. The recommendation to rotate remaining long-lived keys quarterly minimum is reasonable but glosses over the operational difficulty of rotation in legacy systems. The strongest argument is the unstated one: if you're building a new system, there's no excuse for long-lived credentials when ephemeral alternatives exist for every common use case.

---
*Sources: [[raw/you-dont-want-long-lived-keys]]*
*Last updated: 2026-05-14*
