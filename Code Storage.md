# Code Storage

API-first Git infrastructure designed for machines rather than humans. Pierre Computer Company ($23M from CRV + O1A) is building "Stripe for Git repos" — programmable repo creation, custom-domain Git endpoints, warm/cold storage tiering, and SDKs in TypeScript/Python/Go. The pitch: existing Git hosting was designed for humans clicking buttons; agents need an API that treats repos as disposable infrastructure.

---

## Key Quotes

> "Off the shelf Git infrastructure for your AI application."

The positioning is surgical. Not "Git hosting" — that's GitHub's territory. Not "GitOps" — that's a workflow. This is *infrastructure*, the plumbing layer beneath the platform. The "off the shelf" language borrows from AWS's playbook: stop building it yourself, buy the commodity version.

> "60x faster clones than all r2/s3-based storage solutions"

This is the technical moat. Object-storage-backed Git (what GitHub and GitLab use internally) was an optimization for human-scale operations. When agents are cloning repos at machine speed, the storage architecture matters. They're selling speed as a feature, but the real implication is *cost* — faster clones mean less compute time wasted waiting for I/O.

> "YOUR CODE IS A BLOB / WE HOLD YOUR BLOBS IN STORAGE / EACH STORED BLOB IS BACKED BY A GIT REPOSITORY"

This is refreshingly direct marketing from an infra company. No "revolutionizing collaborative development" — just blobs in storage. The honesty loops back around to being clever. The third line is the actual value: every blob gets a Git repo, meaning every piece of code is versioned by default.

---

## Key Themes

#tool #infrastructure #git #agents #platform

---

## Critical Analysis

**The bet that matters.** code.storage is betting that agent-created repos will outnumber human-created repos. If you buy that premise, the entire GitHub/GitLab model — rate-limited APIs, OAuth flows designed for human consent, repo counts priced per-seat — is a mismatch. The question isn't whether the API is clean (it is); it's whether the market of "applications that programmatically create repos" is big enough to support a standalone company, or whether this gets absorbed into a larger platform.

**The warm/cold tiering is the killer feature nobody's talking about.** Agent workflows produce enormous numbers of ephemeral repos — spin up, do work, push, abandon. Traditional Git hosting charges the same to store a repo touched yesterday and one untouched for three years. The 7-day warm/cold boundary ($1.00 → $0.15/GB) acknowledges that most agent-created repos are cold by default. This isn't a pricing gimmick; it's an architectural assumption about how machines use Git differently from humans.

**The GitHub sync engine is a hedge.** They're not trying to replace GitHub — they're offering a different entry point (API-first) with a bridge back to the platform where humans review code. This is smart positioning: don't compete on the social layer, compete on the infrastructure layer. The sync engine means an agent can create repos via code.storage's API and humans can interact with them on GitHub without knowing code.storage exists.

**But who is this actually for?** The obvious customers are AI coding platforms (Replit, CodeSandbox, Cursor) and agent orchestration frameworks that need Git as a substrate. These are real, but together they represent maybe a few dozen companies. The growth case requires either (a) every SaaS product becoming an AI coding platform, or (b) individual developers hitting GitHub's rate limits with their agent workflows. The $23M raise says VCs are betting on (a).

**The team matters more than usual here.** Git infrastructure is a distributed systems problem where subtle bugs become data loss. The team's pedigree (Cloudflare, GitHub, Stripe, Discord) isn't vanity — it's the minimum qualification for anyone trusting their code storage to an early-stage company. The question is whether the same team that can *build* it can also *sell* it to enterprises who've used GitHub for 15 years.

**Comparison to the wiki's Git landscape:**
- [[Radicle]] is the opposite bet: decentralized, P2P, no company, no API key. code.storage is centralized, API-first, pay-per-use. Both are "Git infrastructure for a new era" but they disagree on whether that era needs a company or a protocol.
- [[InsForge]] provides the full BaaS stack (database, auth, storage) for coding agents; code.storage provides *just* the Git layer. Complementary, not competitive — an agent platform might use both.
- [[Dolt]] treats databases like Git repos; code.storage treats Git repos like databases. Same idea, inverted.
- [[Graft]] replicates SQLite via object storage; code.storage replicates Git via sharded ref storage. Both are "storage tiering for developer infrastructure" but at different layers of the stack.

---

*Sources: [[raw/code-storage]]*
*Last updated: 2026-05-31*
