# What You Don't Build

Liam Nugent's essay on the two features he refuses to build in consumer-facing fintech apps — the document hub and the notifications centre — and what that refusal teaches about running costs versus build costs, a culture that rewards production over removal, and the agent-era inversion that agents are better spent pruning than generating.

---

## The argument

Nugent's two hard-pass features look like reasonable answers to real problems. Legacy systems produce PDFs because they used to post them; compliance wants "persistent communications channels"; a stakeholder asks for a bell icon with a red dot. But every stakeholder "wants to do a Columbo and add just one more thing," and the simple list-of-files becomes a Google Drive rebuild with tagging, archiving, sharing and per-channel quirks — and that is just the build. The maintenance bill arrives every iOS release cycle, when some foible the feature depended on gets removed unilaterally by Apple, or Google, or Microsoft, or Amazon.

His working answer: show the running costs, not the build costs, then take each use case on its own merits and ask whether it "makes the boat go faster." Most of the time the answer is something simple or something that already exists. Advice in two parts: "Be extremely choosy about what you let be added to your system. And be militant about taking things out."

## Key quotes

> "The lazy solution is a document hub. A list of files you can select and open."

The understatement is the point. Nobody budgets for the Columbo effect — each addition is individually defensible, the aggregate is a platform nobody chose.

> "What I've found actually works is illustrating the running costs. Not the build cost. The running costs, laid out over years. That tends to bring people back from the brink."

The load-bearing move of the essay. Engineers already feel the running cost; stakeholders don't, because it's distributed across future release cycles they'll have forgotten they caused. The argument that works is an accounting argument, not a design argument.

> "Those who create and launch are the people who are rewarded and looked up to because we still have a culture that rewards the production of things over everything else. To review, to maintain, to remove—this is all seen as lesser work." — Gerry McGovern

Nugent is careful to say the second half of his advice is hard "and it isn't simply cowardice." This is why: removal is structurally unrewarded, so "be militant about taking things out" is an org-design problem wearing an engineering hat.

> "Using agents to do the pruning seems like the smarter move to me. To do the maintenance. To remove things judiciously, and more carefully than anyone is going to manage by hand."

The essay's real news, and the inversion of the usual agent-as-builder framing: DHH's endless execution is easy to misread as "make more stuff all the time," when the scarce skill in a world of cheap building is deciding what deserves to exist.

> "(I'd have written a shorter post, but I couldn't work out what to take out.)"

The confession that proves the thesis on the author himself. Subtractive change costs real cognitive effort even for the person advocating it.

## Key themes

- #concept **Running costs over build costs** — the framing that actually changes stakeholder behaviour, and the one that gets *more* important as build cost collapses.
- #concept **Subtractive neglect** — McGovern's Top Tasks data (stripping 80–90% of site content improved sales and support) and Klotz et al. in Nature (people systematically overlook subtractive changes even when removal is obviously better) give the argument an empirical spine.
- #concept **Success is in the customer's world** — "success is usually something out there in the world of the customer, rather than something in here, in the world of the organisation."

## Analysis

The fintech setting is almost incidental; the mechanism is universal. The document hub and notification centre are org-shaped artifacts — they exist so the organisation has somewhere to put its outputs, not because a user needs to accomplish anything. "Letting the organisation off the hook at the expense of the user" is as clean a definition of enterprise UX debt as you'll find, and the Columbo effect explains why no individual decision is ever the culprit.

The running-costs reframe gets stronger, not weaker, in the AI era. If build cost collapses, running cost becomes a larger share of the total — which means the pre-AI discipline Nugent describes is the discipline that matters *more* when agents make building nearly free. The essay's weakest joint is its one concession: "most of the time — not always, to be fair" is doing enormous work. Platforms are sometimes the right answer (payments, auth, audit), and the boat-go-faster heuristic offers no test for those cases. It's an essay about a bias, not a decision procedure, and it's honest about that without dwelling on it.

The agent claim is the boldest and least evidenced: "agents for the pruning" is asserted, not demonstrated — there's no field report here of an agent successfully retiring a feature. But it's directionally backed by measurement elsewhere, and it sharpens a genuinely important point: the binding constraint in the agent era is judgment about what should exist, not capacity to make things. Nugent's two-part advice — choosy about what enters, militant about what leaves — is a statement of that constraint in organisational terms.

## Related pages

- [[Simplicity in the Age of AI-Assisted]] strengthens this source's central inversion: that note argues LLMs make acting on complexity judgment nearly free, and Nugent extends the same logic from rebuilding to *removal* as a first-class use of agents.
- [[The Minimum Viable Unit of Saleable Software]] nuances Brandur's "cheap but not zero" economics: Nugent relocates the real cost entirely to the running line — the part LLMs don't discount at all.
- [[The Economic Benefit of Refactoring]] complements the pruning thesis with the measurement Nugent's essay lacks: Fowler's controlled experiment puts a token-level ROI on spending agent effort on maintenance rather than generation.
- [[The AI Productivity Paradox]] complicates the same target from the product side: Cagan's output-versus-outcomes gap is Nugent's "success is out there in the world of the customer" argued at org scale.

---
*Sources: [[raw/what-you-dont-build]], [[summary/what-you-dont-build]]*
*Last updated: 2026-09-19*
