---
url: https://argemma.com/blog/long-lived-keys/
date_fetched: 2026-07-05
backfilled: true
---

Long-lived keys are liabilities that, broadly, compound over time:

You can manage this risk in two ways. The first is to reduce the scope of what a given key can do. This is ideal but not always possible: a key may just need to be inherently powerful in order to do its job. The more general risk reduction is rotating keys. Key rotation is, also, an incredible pain. I suspect many readers have had an experience like:

Systems built around ephemeral keys (i.e., keys that are valid for roughly 1 day or less) sidestep a lot of this pain as "rotation" is a built-in feature. Replacing long-lived keys with ephemeral keys is, for my money, one of the best uses of security engineering effort. Some specific examples:

Long-lived production SSH keys may be copied around, hardcoded into configuration files, and potentially forgotten about until there is an incident. If you replace long-lived SSH keys with a pattern like EC2 instance connect, SSH keys become temporary credentials that require a recent authentication and authorization check.

Instead of relying on a static PyPI token that somehow ended up in the CTO's *personal* 1Password and, less accidentally, re-used across several release pipelines of varying quality, you can use trusted publishers. This allows you to use specific GitHub Actions workflows (which you might already be using for releases) to generate temporary credentials for package releases. When you go to revoke that static PyPI token, you may even find that it made its way into some one-off release pipeline that you forgot about until you broke it just now.

A big reason why SSO is useful is that you can replace a user-selected long-lived password at each application with a short-lived authentication assertion from a trusted identity provider (IdP). An attacker can guess a poorly selected password. They can't really guess a signed XML document in the same way.2

Your inner security nerd might be annoyed. Surely it's not plausible to get rid of all long-lived keys? That SSO example above is glossing over the fact that while each authentication assertion is ephemeral, the key that signed the assertion is not.

This is true. You may not be able to eliminate every single long-lived key. However, there is still significant upside in reducing long-lived keys.

First, reducing the number of long-lived keys means you can concentrate your security efforts. It is much harder to reason about, say, the security of an arbitrary Engineer's laptop than it is an EC2 instance that exists exclusively to tell KMS to sign something. By reducing the number of long-lived keys, you also tend to create smaller and more focused pieces of infrastructure that are easier to harden and reason about.

Another upside is that security-sensitive infrastructure tends to require more rigor than most tools. Being rigorous often means taking things a bit slow. You don't always need rigor. You also want to be deliberate about where you sacrifice speed. The cryptographic key limits I mentioned before? You don't want that to be a tertiary concern for an arbitrary engineering team, eating up time spent on feature work. It is more likely to be forgotten, de-prioritized, or done incorrectly. You can, instead, make it part of someone's (or some group's) core focus and solve this once for everyone else.

Proper maintenance of long-term keys takes work. My general advice for long-lived keys is something like:

Limit the scope of what a given key or credential can do. You might, for example, want to scope a data encryption key to a specific shard of customer data.

Reason through the limits of the maximum lifetime of a given key. The typical security model for cryptographic keys is something like "It would take a supercomputer billions of years to guess the key." You want your limits to have similarly strong and confident assertions.

Aim to rotate long-lived keys at least quarterly. From an operational perspective, rotating a key is like exercising a muscle. If you do it regularly you'll likely have more accurate and useful tooling or documentation. Good tooling and docs means you are less likely to ~~pull a muscle~~ miss an SLA.

This setup is, admittedly, toilsome. Don't distribute that toil to everyone. You can concentrate that effort into a group that is incentivized to be rigorous *and* solve it once, for everyone. Reducing toil and consolidating rigor is a major advantage of robust security infrastructure.

For example, if you encrypt more than `2^32` messages with the same AES-GCM key you begin to risk message forgery attacks. ↩︎

Just put aside that SAML is a nightmare protocol for now! ↩︎
