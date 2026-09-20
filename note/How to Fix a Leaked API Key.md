# How to Fix a Leaked API Key

A step-by-step incident-response tutorial from freeCodeCamp on what to do when an API key ends up in a Git repository — arguing that the order of operations matters more than the tooling, and that revocation must always come before history cleanup.

---

## The argument in one paragraph

The article claims that once an API key has been committed to Git, the only correct first move is to invalidate it — deleting the key from the latest commit, adding it to `.gitignore`, or even rewriting history are all downstream of that, because Git's history preserves every version of every file and automated scanners find exposed credentials faster than humans can clean them. It further claims that cleanup must extend beyond the repository itself (forks, CI logs, build artifacts, PR comments all retain secrets), and that prevention is structural rather than personal: least-privilege keys, per-environment credentials, and secret scanning in the commit path. This is falsifiable in a specific way: if scanners were slow, or if private repos were genuinely isolated, the invalidate-first rule would be overkill and a delete-and-push would suffice. The article stakes everything on both being false.

## Key quotes

> If an API key has been committed to Git, assume it has been copied and compromised, even if you delete it immediately.

This is the load-bearing assumption of the whole piece, and it is stated as an axiom rather than justified with evidence about scanner latency. It is almost certainly correct for public repos, but the article never quantifies "how fast is fast enough to delete safely" — which is the question a sceptical reader actually has.

> Deleting a picture of the key doesn't matter if someone already picked up the physical key.

The house-key analogy does real work here: it reframes Git cleanup as hygiene rather than security, which is exactly the mental shift practitioners fail to make under stress. The best sentences in the article are the ones that separate what feels like remediation from what actually is.

> A private repository is safer than a public repository, but it isn't a secret vault.

The private-repo section is the most valuable part for experienced developers, because the "it's a private repo" rationalisation is the most common excuse for committing credentials. The list of escape routes — compromised accounts, contractors, integrations, CI logs, forks, backups, screenshots — is a compact threat model in nine words of bullets.

> Rewriting your repository doesn't erase copies that already exist somewhere else.

This sentence quietly demolishes the fantasy that `git filter-repo` makes a leak "unhappen." It also explains why the article's ordering is not arbitrary: if history rewriting cannot undo exposure, then the only thing that changes the security posture is making the credential itself useless.

> Humans are excellent programmers and occasionally terrible search engines.

The best line in the piece, and a genuine insight about why secret scanning belongs in tooling rather than in vigilance. It frames automation not as a substitute for care but as compensation for a specific, predictable human failure mode — which is a healthier framing than most security writing manages.

## Critical analysis

The non-obvious contribution is the *ordering*, not the content. Every individual technique here — environment variables, `.gitignore`, `git filter-repo`, Gitleaks — is documented a thousand times elsewhere. What is rare is an explicit, defended sequence: invalidate before investigating, investigate before cleaning, and never treat history rewriting as the fix. The article is right that this ordering is where panicking developers go wrong; the instinct to scrub the evidence first is nearly universal and nearly always harmful, because it wastes the window in which revocation prevents actual damage.

The weaknesses are mostly of omission. First, the article never mentions secret managers' harder problems — it gestures at them in one paragraph ("centralized credential storage, access controls, auditing") without touching the operational reality that migrating to one is its own project. Second, there is nothing about *detecting* the leak in the first place beyond noticing a strange bill; GitHub's push-protection and provider-side leaked-key detection deserve a mention, since they collapse the response window from days to seconds. Third, the `git filter-repo` instructions are correct but thin on the failure modes — the article says "test on your backup clone first" but doesn't show what a botched rewrite looks like or how to recover. Finally, the piece is silent on the agent-era twist: coding agents now read `.env` files, echo environment variables into logs and transcripts, and paste secrets into prompts, which multiplies the leak surfaces well beyond the Git commit this article treats as the boundary of the problem.

What the article gets right that most similar guides don't: the insistence on checking billing and usage logs (most leaks are never abused, and knowing that matters for the incident report), the warning that `.gitignore` does not untrack already-committed files, and the per-environment credential separation, which converts any single future leak from catastrophic to annoying.

## Related

- [[NuGet API Key Lifetime Reduction]] — strengthens this article's revocation-first doctrine from the registry side: Microsoft's move to 30-day NuGet keys and OIDC Trusted Publishing is the same insight (credentials must die quickly) implemented structurally, so the leak window this article asks you to survive manually gets shortened by design.
- [[OneCLI]] — complicates this article's "environment variables are a common and practical solution" advice: OneCLI argues agents should never hold real credentials at all, using placeholder keys injected at an HTTP gateway — a stronger containment model than the `.env`-plus-restriction pattern recommended here.
- [[Ditch the Token Headache — SSH Just Works]] — takes the opposite side of the credential-lifetime argument this article implies: Kuehn wants long-lived, never-thought-about SSH keys for humans, while this piece pushes short-lived, restricted, per-environment credentials — the tension between interactive convenience and blast-radius control is unresolved between them.
- [[Supply Chain Security for Software Developers]] — complements this piece as a sibling checklist from the same threat family: where this article covers the leak-your-own-secret incident, that page covers the malicious-package campaign, and both converge on the same conclusion that prevention belongs in automated commit-path checks rather than developer vigilance.

---
*Sources: [[raw/how-to-fix-a-leaked-api-key]], [[summary/how-to-fix-a-leaked-api-key]]*
