# DKIM2 and DMARCbis

A major overhaul of email's two core authentication protocols, shipped in mid-2026. DKIM2 (draft -04) replaces the 2008 DKIM standard with per-hop envelope signatures, replay-proof chains of custody, and reversible content "recipes" that let forwarding survive authentication. DMARCbis (RFCs 9989–9991) splits the monolithic DMARC spec into three Standards Track documents and replaces the brittle Public Suffix List with a DNS tree-walk algorithm. Stalwart Mail Server is first to implement both.

---

## Key Themes

### Replay-Proof Chains of Custody (#concept)

DKIM1's fundamental flaw was that a signature said "this domain vouches for this content" with no binding to recipient or hop. A signed message could be replayed to millions of addresses and every copy verified. DKIM2 fixes this by embedding the SMTP envelope (`MAIL FROM` and `RCPT TO`) inside each per-hop signature, creating a verifiable chain where each hop's sending domain must align with the prior hop's recipient domain.

> "A captured signed message can be replayed to millions of recipients; every copy still verifies on the original signer's reputation."

This is genuinely clever. Rather than bolt replay prevention onto DKIM (which would have required per-recipient signatures, a non-starter at scale), DKIM2 builds it into the protocol's spine. The `donotexplode` flag forbidding fan-out is the kind of sharp, single-purpose mechanism that good protocol design runs on.

### Recipes: The ARC Killer (#pattern)

The most ambitious part of DKIM2 is the **recipe** — a base64-encoded JSON instruction set inside the `Message-Instance` header that tells verifiers how to reverse forwarding modifications and reconstruct the original message. A mailing list adds a subject tag and footer? The recipe says "remove the tag, chop off the last N lines, here's the original hash." Apply the recipe, recheck the originator's signature, and it passes.

> "A footer tweak and a malicious body swap no longer look the same."

This absorbs ARC's entire reason for existing. ARC (RFC 8617) tried to solve the forwarding problem with a separate trust fabric — intermediate servers would sign their modifications, and receivers would evaluate a chain of trust. It never achieved internet-wide adoption because trust fabrics at scale are a coordination nightmare. DKIM2's recipe approach is strictly better: let intermediate hops describe what they changed, and let verifiers undo it. No trust in intermediaries required. The IETF DMARC working group is now moving to reclassify ARC as Historic.

### Bounces That Don't Backscatter (#concept)

DKIM2 makes bounces (DSNs) trustworthy for the first time by tying them to the signed envelope chain. A bounce must be addressed to the highest-numbered hop's `MAIL FROM`, must be a DKIM2-signed `multipart/report`, and must carry the original message for re-verification.

> "Pass all three and the bounce is provably about a message you really sent."

This is what enables **deferred bounces** — a receiving server can accept a message, later determine it's spam, and send a signed bounce back without risking backscatter to innocent third parties. It's a small change with outsized operational impact: it closes one of the oldest abuse vectors in email.

### DNS Tree Walk Replaces the PSL (#concept)

DMARC1's dependency on the Mozilla-maintained Public Suffix List was an embarrassing structural weakness. A volunteer-edited text file sat on the critical path of global email authentication, with no mandated refresh cadence and no way for domain owners to correct errors. DMARCbis replaces it with an 8-query-capped DNS tree walk that walks labels leftward from the author domain, using `psd=y`/`psd=n` tags to let domain owners declare their own organizational boundaries.

> "The Organizational Domain is now defined by DNS records that the domain owner controls."

This is the platonic ideal of a protocol upgrade: remove an external dependency, push control to domain owners, and do it with a deterministic algorithm that's simpler than what it replaces.

### p=reject Gets Teeth, Then Loses Them (#pattern)

DMARCbis adds real nuance to `p=reject`, which had become a blunt instrument. Domains using it MUST apply valid DKIM (not SPF alone), SHOULD NOT publish it if users post to mailing lists, and — critically — receivers MUST NOT reject solely because a policy says `reject`. The policy is downgraded from a trigger to a signal.

This is a useful corrective. DMARC1's `p=reject` was cargo-culted by domains that didn't understand the mailing list consequences, and receivers that treated it as gospel broke legitimate mail. The new language makes it clear: policy is a weight in a larger decision, not a command.

### Stalwart: The First Mover (#tool)

Stalwart Mail Server v0.16.12 is first to ship both protocols, backed by an open-source Rust library (`mail-auth`) that covers DKIM1, DKIM2, SPF, DMARC, DMARCbis, and ARC. The browser playground compiles the same code to WebAssembly, so anyone can test without standing up a mail server.

> "Being early to a standard is how the standard gets tested against real mail before it hardens."

This is the right kind of first-mover flex. Rather than shipping a proprietary implementation, Stalwart released the library as open source and built a zero-friction test environment. Standards die in the gap between specification and running code; Stalwart is closing that gap.

## Critical Analysis

**The recipe mechanism is elegant but under-specified on adversarial cases.** What happens when a malicious intermediary crafts a recipe that claims to reverse changes it never made, or omits changes it did make? The spec trusts the recipe's author — which means the security model assumes intermediaries are cooperative. That's fine for mailing lists run by well-intentioned operators, but it's an assumption worth naming. A recipe that lies is indistinguishable from a recipe that tells the truth, because the verifier has no independent view of the pre-modification message.

**The replay prevention chain has a cold-start problem.** The first hop in a DKIM2 chain has no prior hop to validate against. The `donotexplode`/`exploded` mechanism prevents fan-out within the chain but doesn't address the case where an attacker captures the *first* signed message and replays it. For that, you're still relying on the originator's signature freshness — which DKIM2 doesn't address.

**DMARCbis's tree walk is capped at eight queries, which is reasonable but arbitrary.** Domain names can have up to 127 labels. The spec says "strip labels until 7 remain, then walk one at a time to 8 max." For a 9-label domain like `a.b.c.d.e.f.g.h.i.example.com`, that's fine. For a deeply nested organizational structure that uses many subdomain layers, it could miss records. The cap is pragmatic — no one wants a 127-query DNS walk per incoming message — but it's worth documenting as a known limitation.

**The `t` (testing) flag is the sleeper feature of DMARCbis.** It lets domains get full reporting on enforcement without actually enforcing — reject becomes quarantine, quarantine becomes none. Combined with the nuanced `p=reject` guidance, this makes the DMARC deployment curve dramatically safer. Domains can now test their DMARC posture without gambling their deliverability. The old `pct` tag tried to do this with percentage sampling and failed because 1% of reject is still reject for the unlucky 1%. The `t` flag is a clean downgrade that keeps reports flowing.

**The post isn't neutral — it's a Stalwart launch announcement.** The technical detail is solid and the protocol analysis is accurate, but the framing ("Stalwart Speaks Them First") is marketing. The `mail-auth` library being open source is genuinely good for the ecosystem, but the protocol specs are still drafts (DKIM2) or freshly published (DMARCbis). First-mover advantage in protocol implementation is real, but it's also an invitation for others to find the bugs. That's the point Mauro is making — but it's worth reading the article with that lens.

## Connections

- [[Bounding the Blast Radius — Prompt Injection Defenses]] — Same structural insight applied to a different domain: defense in depth, not silver bullets. DMARCbis's layered approach (tree walk + `psd` tags + `t` flag + nuanced `p=reject`) mirrors Abdu's four-layer prompt injection taxonomy.
- [[AgentMail]] — AgentMail's documentation notably omits SPF/DKIM/DMARC handling. DKIM2's replay prevention and DMARCbis's subdomain policies are directly relevant to any platform giving agents email addresses. Every agent inbox needs to *not* become a spam vector.
- [[Memento]] — Email as the universal fallback protocol. DKIM2 and DMARCbis are infrastructure upgrades to that protocol — invisible to end users but critical to keeping the channel trustworthy as agents join it.
- [[Event-Driven vs Polling Architectures]] — DMARC aggregate reporting (RFC 9990) is an event-driven architecture: domains push reports to `rua` addresses, which are consumed asynchronously. The same delivery-contract considerations apply.

---
*Sources: [[raw/stalwart-dkim2-dmarcbis]]*
*Last updated: 2026-07-11*
