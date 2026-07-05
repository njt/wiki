# I Don't Want Your PRs Anymore

DPC's essay on how LLMs have inverted the economics of open-source contribution. When a maintainer can generate code faster and more safely than they can review a stranger's PR, the traditional contribution model breaks. The response isn't rejection — it's redirection toward higher-leverage help: feedback, design discussion, bug reports, prototype PRs with their prompts, code review, and independent forks.

---

## Key Quotes

> "I always have to assume that you might be trying to sneak in something malicious along with your changes"

This is the trust cost that LLMs eliminate. AI-generated code carries no adversarial intent. Every outside PR carries a non-zero probability of being a supply-chain attack. The maintainer bears that risk alone; the contributor doesn't share it.

> "the code in your PR doesn't help me much with any of these"

The three real bottlenecks — understanding existing code, designing architecture, reviewing for correctness — are all cognitive tasks that someone else's finished code doesn't address. A PR dumps the output of a design process without sharing the design process itself. This is why DPC later says prototype PRs *with prompts* are welcome: the prompt is the design, the code is just evidence.

> "A good bug report is 3/4 of the bug itself being fixed"

The most undervalued contribution in open source. Well-described reproduction steps are a form of investigation that the maintainer can't parallelize across users. Every good bug report saves the maintainer hours of narrowing — the work they can't automate.

> "Just fork. Add support for your own use case, do things your way, ask neither for permission nor forgiveness."

The most provocative line in the piece. Forks have always been the escape hatch of open source, but DPC reframes them as the *preferred* outcome, not the fallback. A fork saves the maintainer from consensus-building and multi-use-case design — work that has nothing to do with code and everything to do with politics.

---

## Key Themes

#open-source #llm-economics #maintainership #trust #forking #contribution-model

---

## Argument Structure

The essay runs on three premises:

1. **Trust asymmetry is real.** Outside PRs carry security risk that LLM-generated code doesn't. This is not paranoia — supply-chain attacks through open-source contributions are a documented threat vector. The maintainer is the one who burns if they miss something.

2. **Code generation is no longer the bottleneck.** LLMs have made producing code near-zero cost. The bottlenecks are understanding, designing, and reviewing. A stranger's PR helps with none of these.

3. **Coordination overhead now dominates.** Review cycles, CI, merge conflicts, timezone ping-pong — the friction of collaborating with an unknown contributor now exceeds the friction of just implementing the change yourself with an LLM.

The conclusion follows cleanly: don't send implementation PRs. Send the inputs to the design process instead — bug reports, feedback, ideas, prototype code *with* the prompts that generated it, and code review.

---

## Critical Analysis

This is one of the sharper pieces I've read on how LLMs reshape open-source economics, and it's sharper precisely because it's from a practitioner rather than a pundit. DPC runs actual projects and feels the pain directly.

**What it gets right.** The reframing of "help" is the essay's real contribution. Most maintainer burnout discourse frames the problem as "maintainers need more help" and the solution as "lower the barrier to contribution." DPC argues the opposite: the barrier *should* be higher for code, and lower for everything else. Bug reports, design discussions, and prototype PRs with prompts are all higher-leverage than finished code from a stranger. This is a genuinely useful taxonomy.

The fork-as-feature argument is underrated. Consensus-building is a tax on maintainers that serves the project's user base at the maintainer's expense. Encouraging forks offloads that tax. The open-source world has stigmatized forking for decades; DPC makes a case that the stigma is backwards.

**What it glosses over.** The essay assumes the maintainer has both the domain knowledge and the LLM skill to implement changes faster than contributors can propose them. This is true for DPC's projects. It is not true for the median open-source project, where the maintainer is barely keeping up and doesn't have deep LLM workflows. The advice scales down poorly.

The "share your prompts" request is directionally right but practically tricky. Prompts are often multi-turn, messy, full of dead ends. The polished PR is a cleaned-up artifact; the raw prompting session is closer to a browser history. Asking for the prompt is asking for the sausage factory — valuable, but a different kind of artifact than most contributors are prepared to share.

The security argument cuts both ways. Yes, LLM-generated code has no adversarial intent. But LLM-generated code also has no *accountability*. If a human contributor sneaks in a backdoor, there's at least a GitHub account to trace. If an LLM generates vulnerable code at the maintainer's direction, the maintainer owns it wholly. The trust model shifts, but it doesn't disappear.

**The bigger pattern.** This essay belongs in a cluster with [[Radical Accountability]], [[Coding Agents and Complexity Budgets]], and [[Cyborgs Will Kill the Corporation]] — all arguments that LLMs change the *economics* of software, not just the *mechanics*. Transaction costs are collapsing. The question is what reorganizes around the new cost structure. DPC's answer for open source: maintainers become editors, not authors. Contributors become sources and reviewers, not implementers. Code flows from the person who owns the vision.

---

*Sources: [[summary/i-dont-want-your-prs-anymore]]*
*Last updated: 2026-05-15*
