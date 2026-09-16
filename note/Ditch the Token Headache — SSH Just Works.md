# Ditch the Token Headache — SSH Just Works

A short, practical post by Shan Kuehn arguing that developers doing day-to-day Git work should abandon Personal Access Tokens in favour of SSH keys, with a five-minute setup guide. It matters because it takes a position in a quiet but ongoing argument about credential lifetime: Kuehn's claim is that for interactive human use, the best credential is one you configure once and never think about again — the exact opposite of the direction most of the security industry is pushing.

---

## The argument in one paragraph

Kuehn claims that fine-grained PATs impose ongoing bookkeeping — per-repo scope selection, expiration tracking, mid-project regeneration — that SSH keys eliminate, because SSH authentication is bound to the host (github.com) rather than to individual repositories. One registered key therefore authenticates every repo you own, contribute to, or clone, indefinitely, with no per-repo setup beyond ensuring the remote URL uses SSH rather than HTTPS. The falsifiable core: that configure-once, never-expires credentials are net better for interactive developers than short-lived, scoped tokens. If key theft and compromise turn out to be common enough that expiry is worth the friction, the claim fails — which is precisely the bet the broader industry is currently making against it.

## Key quotes

> "GitHub __officially retired password auth over HTTPS back in August 2021__ in favor of token- or SSH-based authentication."

The framing device for the whole post: the status quo ante is gone, so the only real question is which of the two replacements you pick. Kuehn uses it to absolve the reader — the pain is not their fault — before advocating.

> "I'm perpetually the lazy sysadmin."

The self-description that explains the entire argument. This is a post optimising for zero ongoing maintenance, not for blast-radius minimisation. Read it as an honest statement of priorities rather than a security analysis.

> "PATs are definitely great for CI pipelines or scripts where you need scoped, expiring access. But for day-to-day work, they mean picking permissions per token and re-authenticating whenever one expires."

The concession that keeps the post defensible: scoped, expiring credentials are right where the credential is non-interactive and the scope can be narrow. The argument is about *interactive* use, where the human is present to be annoyed by re-auth.

> "SSH auth is keyed to the *host* (github.com), not to individual repositories. Once your key is registered with your GitHub account, it authenticates you for every repo you own, contribute to, or clone, forever, with no per-repo setup."

The load-bearing technical insight, and the source of both the convenience and the risk. Host-level authentication is why setup is one-time — and also why a stolen key is a stolen identity across everything, not one repo.

> "Spend the five minutes once, and you're done thinking about git authentication for every repo you'll ever create."

The closing claim, stated as pure win. Note what it optimises away: "done thinking about" is the goal, but in security, not thinking about a credential is exactly what makes it dangerous when the machine it lives on is compromised.

## Critical analysis

The non-obvious thing here is the host-vs-repo distinction. Many developers hit the password-auth wall, generate a fine-grained PAT, and silently accept per-repo scope management without realising the alternative has fundamentally different granularity. Explaining that SSH keys attach to the account and host — not the repository — is genuinely useful, and the `git remote set-url` instructions for converting existing HTTPS clones close the last practical gap.

But the post is weakest exactly where it is most confident. "Forever, with no per-repo setup" is presented as an unalloyed good, yet it means an SSH key is a long-lived, unscoped, non-expiring credential with full account reach. The security industry's current direction — 30-day NuGet keys, OIDC-based Trusted Publishing, short-lived tokens everywhere — is a direct response to stolen-credential attacks, and Kuehn's post doesn't engage with that at all. The passphrase recommendation ("optional but recommended if this is a machine other people can touch") is the only nod to compromise, and it undersells the threat: laptops are stolen and malware exfiltrates `~/.ssh` routinely, and the threat model is not "other people can touch this machine" but "this machine will eventually be breached."

There's also an unstated assumption that deserves surfacing: this advice is written for a solo developer's interactive use. The moment an agent or automation needs GitHub access, the calculus flips — agents can't do an `ssh-agent` passphrase dance, and least-privilege scoping becomes essential rather than annoying. Kuehn's own PAT concession points at this but doesn't develop it.

What's left out: key rotation practice, hardware-backed keys (FIDO2 `ed25519-sk`), GitHub's key-expiry settings for SSH keys, and any mention of fine-grained SSH key policies. A post arguing for set-and-forget credentials in 2026 should at least acknowledge that "set" deserves more thought than five minutes.

## Related

- [[You Dont Want Long-Lived Keys]] — directly complicates this source: that page argues long-lived credentials are the root problem in agent-era auth, while Kuehn argues a never-expiring SSH key is the whole point; the two are a clean statement of the human-convenience vs blast-radius tradeoff.
- [[Agentcookie]] — strengthens the same underlying goal from the agent side: where Kuehn eliminates re-auth ceremony for a human's Git pushes, Agentcookie eliminates it for an agent's browser sessions, both betting that configure-once session inheritance beats repeated authentication.
- [[OneCLI]] — nuances this source's binary choice: OneCLI shows a third path where the agent never touches real credentials at all, which is the scoped-delegation answer to the problem Kuehn solves with a single all-powerful key.
- [[Authentication Is Largely Solved]] — bears on Kuehn's framing that auth pain is a solved-once problem; this post is a practitioner's data point that for plain Git hosting, the "solved" answer is indeed boring and stable.

---
*Sources: [[raw/ditch-the-token-headache-ssh-just-works]], [[summary/ditch-the-token-headache-ssh-just-works]]*
