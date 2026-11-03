# Building Apps in Grist with the MCP Connector (Webinar)

A Grist webinar transcript in which a presenter builds two apps live with Claude through Grist's MCP connector — one greenfield ("from prompt") and one migration ("from spreadsheet") — and distils the prompting disciplines each mode demands, plus a five-point argument for why Grist's architecture is unusually well-suited to agent-built apps.

---

## The two modes demand two prompts

The central practical distinction: greenfield and migration are different jobs, so they get different prompting styles.

- **From prompt (no existing data):** a deliberately structured three-part prompt — (1) use case and data, (2) important views and workflows, (3) user permissions — written "robotically" on permissions to avoid confusion. The presenter names no fields at all, instead asking Claude to "perform desktop research to find typical workflow statuses and safety procedures," explicitly harvesting the model's training rather than micromanaging.
- **From spreadsheet (migration):** a two-phase "harness." Phase one is planning only: Claude must write a markdown plan file broken into three stages (data structure and formulas, views, permissions), with the instruction "Do not build it in grist yet" — because, as the presenter notes, it may build anyway ("you're right, you told me not to. Here's what happened. I built it anyway."). Phase two builds stage by stage with human feedback between stages.

Two heuristics decide whether staged building is needed: could you explain the spreadsheet to someone else without confusing yourself, and do you have more hidden sheets than unhidden sheets? Either yes means build in stages.

> "Claud's here building a castle every time. This is why I normally do the next step... What do I actually need? What don't I actually need?"

The brewery demonstration is the thesis made concrete: given a hobbyist spreadsheet, Claude invented purchasing workflows and pages that were never in the original Excel. The harness exists precisely to convert the agent's enthusiasm into a reviewable artifact — the plan file also serves as an audit trail: "what did we say we were going to build? What did we actually build?"

> "when you want to put a harness on it is what I call it. Don't let it build everything because it's going to build things that work for other companies but not yours."

This is the migration-specific version of a general truth: training-data assumptions are fine for the general case but wrong for *your* organisation's specifics, which only you know.

## Why Grist, specifically

The presenter's falsifiable claim is that vibe coding works better in Grist than other app builders because of the architecture, not the model:

1. **No deployment step** — the app is live immediately; invite people and go.
2. **Human and bot can both edit** — "anything that it can do, I can do," removing dependency on the agent.
3. **Everything built to fit** — linking tables is a reference column ("one line of code"), and basic UI like tables already exists; other builders make the agent cobble assets together, which is slower and costlier.
4. **The app is a self-contained file** — copy it, template it, download it, roll back, with the custom-widget HTML/JS inside it.
5. **Auditable permissions** — access rules are checked in the UI via "view as" by the business owner, and enforced server-side even inside custom widgets (viewing as "Marcus" filtered his data inside the HTML/JS widget Claude wrote).

Custom widget code now lives in the document's metadata (no more manual uploads), and the MCP connector injects instructions the agent wouldn't follow on its own — dummy data, Grist-native styling, light/dark mode. Two nice emergent behaviours: Claude applied database normalisation unprompted (collapsing fermentables/hops/flavorings sheets into one `ingredients` table with a category column), and in the greenfield app it added a safety-check gate before status could move to "in progress" — "Claude cares. I didn't ask it to do that."

## What's left unresolved

The transcript is honest about its gaps, which makes it more useful than most vendor material:

- **No cost accounting.** "It's taking forever" is the only token/time metric; the free plan's 3,000 MCP calls are mentioned but never measured against real builds.
- **Anecdotal error rates.** "It normally gets you 80% of the way there" — two successful builds, no data on corruption or repair frequency.
- **No staging-vs-production story.** Merge improvements for copies are acknowledged as future work.
- **Privacy is a workaround, not a design.** The structure-only download (Pierre's tip) is the only data-exposure answer given.
- **The brewery was never evaluated.** The overbuild was staged to make a point and then never judged for usability — the core tradeoff of the harness went undemonstrated.
- **A lock-in contradiction.** The presenter stopped hosting widget code externally ("I just put it all in Gris now"), quietly cutting against the portability argument.

## Take

This is vendor material, and it sells — but the prompting discipline is real and portable. The "plan file as audit trail" move is the same pattern as spec-driven development, applied at spreadsheet scale, and the permission-verbosity advice ("robotic language, who should have access, who should not") is a small but genuinely transferable habit. The five-reasons argument is the most interesting part because it's an architectural claim, not a model claim: the moat is that the agent is operating inside a runtime where deployment, UI primitives, access control, and file-format portability are already solved. Whether that survives contact with the unresolved staging story and the anecdotal error rates is exactly what the webinar doesn't test.

## Related pages

This source gives [[Grist]] its first agent-facing story — the existing page covers the engine and file format that the presenter's "self-contained file" and server-side access-rule claims rest on, and the two-pass access control described there is what makes "view as" auditing actually trustworthy.

It strengthens the plan-then-build discipline in [[SDD Case Study — 13 Apps in 70 Days]] — same "correct the plan, not the output" instinct, but here motivated by a single non-engineer user rather than a spec corpus across sessions, showing the pattern scales down as well as up.

It nuances [[MCPs Aren't APIs — Stop Treating Them Like One]] — the Grist connector is a case where tool-specific injected instructions (dummy data, styling) are the difference between an agent that builds and one that merely advises from scraped docs, exactly the semantic-gap argument that page makes.

And it complicates the greenfield optimism in [[Vibe Coding as a Team Sport]] — Udell's weak-human-plus-machine thesis holds here, but the brewery demo shows the machine's default contribution is overbuilding, making the harness (not the model) the real team player.

---
*Sources: [[raw/a3b0a59b252436f6e1cae33b13ef861e]], [[summary/a3b0a59b252436f6e1cae33b13ef861e]]*
*Last updated: 2026-11-03*
