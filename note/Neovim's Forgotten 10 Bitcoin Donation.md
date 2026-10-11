# Neovim's Forgotten 10 Bitcoin Donation

A Hacker News post observes that Neovim's bitcoin donation address received a 10 BTC donation in 2023 (worth ~$800,000 at October 2026 prices) which the project has apparently never touched — the last outbound transaction was 2019. It is a small, verifiable, and quietly damning snapshot of how open source projects handle funding they never planned for.

---

## What it says

The poster did what anyone can do with a public blockchain: looked up the donation address printed in Neovim's site footer. What they found was a single large donation sitting motionless for years.

> "I saw this massive 10 Bitcoin donation from back in 2023 (worth $800,000 now...)"

The "now..." does a lot of work — the poster is pointing out that the value has appreciated far past whatever the donor intended as a rounding-error gesture.

> "From the activity history, neovim last sent bitcoin out of the address in 2019 so it's been max 7 years since they've definitely had access."

The blockchain proves access, not awareness. The keys demonstrably work; whether anyone on the team has looked at the balance recently is the actual open question, and the post ends as a public shout into the void: "Does anyone from the neovim project know about this?"

## Key themes

- #tool — Neovim as the subject: a volunteer-maintained editor with a crypto tip jar nobody audits
- #concept — Open source funding inertia: money can arrive faster than the governance to spend it
- #pattern — Blockchain as a public audit log anyone can read, even for a project's finances

## Analysis

The interesting thing here is not the money but the governance gap it exposes. A 10 BTC gift in 2023 was maybe $25,000–$30,000; at $80,000/BTC it is roughly $800,000 — likely more than Neovim's entire visible sponsorship apparatus moves in a year. Yet the project's operational posture (footer donation address, no visible treasury process) hasn't changed since 2019. Crypto donations are uniquely suited to this failure: they arrive pseudonymously, can't be reverse-charged, and require a volunteer to both notice and be comfortable converting them. Compare [[The Zero-Cost Fallacy of Open Source]], which argues maintainers systematically undervalue their own costs — here the value is literally sitting in a wallet, unconverted, because the human side of the loop is missing.

It's also a mild comedy of transparency: the ledger is fully public, so the donation is neither hidden nor claimed — just orphaned. The poster's framing is generous (a daily user hoping the money helps) rather than accusatory, which is the right register; the fail mode here is neglect, not graft.

## Related pages

- [[The Zero-Cost Fallacy of Open Source]] — this is the fallacy made concrete: a maintainer team sitting on seven figures of unclaimed support while funding remains haphazard
- [[Zed]] — the editor that grew out of the Neovim-adjacent Tree-sitter ecosystem; contrast a venture-funded editor with a volunteer project that can't spend its donations
- [[The Golden Age of Open Source Applications]] — the claim that open source applications are flourishing; Neovim's funding picture nuances it, with abundance of users and scarcity of treasury

---
*Sources: [[raw/item-news-ycombinator]], [[summary/item-news-ycombinator]]*
*Last updated: 2026-10-11*
