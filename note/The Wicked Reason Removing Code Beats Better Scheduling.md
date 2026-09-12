# The Wicked Reason Removing Code Beats Better Scheduling

Alex Russell's response to Marko Ilić's piece on scheduling critical-path work, arguing that when remediating poor web performance, teams should remove code before they reorder when it loads. The two approaches share the same deep cost — understanding page behaviour — but only removal is durable, defensible, and resilient to the changing conditions real products face.

---

## Key Quotes

> "The primary cost of both code removal and scheduling interventions is the investment to deeply understand page behaviours. This presents a narrative challenge to the proposition that scheduling is an easier fix, as both approaches share the largest cost. Ergo, changes to resource ordering cannot be assumed cheaper or easier."

The thesis in miniature. Scheduling is sold as the pragmatic shortcut — "we can't remove the code, so let's at least load it later" — but Russell's point is that *knowing what to reorder and why* costs the same deep investigation as *knowing what to delete*. The shortcut is a mirage.

> "The bytes that are only deferred still land on the main thread, potentially delaying above-the-fold resources in ways that only become visible in the tail of the connection quality curve. Worse, JS resources that are fetched late generate heavy 'thuds' when residual allocations from background compilation arrive on the main thread."

The technical mechanism that makes deferral less benign than it looks. Deferred code isn't free code — it's delayed code, and the delay hides the cost in the tail where the marginal user lives. The "thuds" from background compilation are why INP regressions from innocent `loading="lazy"` changes are so maddening to chase down.

> "Many groups I've worked with have confidently asserted that *'everyone has 5G'* or *'everyone's on fast devices,'* despite neither their own dashboards nor the global baseline supporting that presumption."

The misidentified marginal user. Teams build to an imagined baseline of fast networks and fast devices, then reorder endlessly without durable gains. The fix is a management act — actually *look* at the network, CPU, and memory limits of the users at the margins — not an engineering one.

> "Systems that rely on scheduling to assure reasonable performance are brittle. Reordering work to appear responsive for one user journey can help, but such systems have a comparatively harder time adapting to changing needs."

The durability argument. A removal win survives product growth, new features, and a shifting user base; a scheduling win is a carefully balanced card tower that any new feature or device class can topple.

> "In the tail of the distribution, SPA architectures assure that every choice to fill the channel with Team A's code harms Team B's experience."

The organizational failure mode. In large organizations, scheduling becomes competitive advocacy — every feature owner lobbies to load their code first, and a degraded commons raises the stakes on everyone. The result is the "preloading" arms race and a Gordian Knot of `-common.js` and `-vendor.js` files that nobody can untangle.

> "Going from decent to excellent performance is a technical challenge, but moving from poor to decent performance is fundamentally a management and culture problem."

The closing frame that splits the field. The technical community's tools — the profiling loops, the scheduling techniques — address the decent→excellent leg. Russell claims most organizations he consults with are stuck on the poor→decent leg, which no amount of reordering fixes because the blocker is a lack of size budgets, regression prevention, and launch gates.

## Key Themes

- **#concept** Removal over scheduling: the default answer to "how do we fix performance?" should be "send less code," not "load code in a better order." Scheduling only pays off after removal reaches diminishing returns, and only in teams with strict size budgeting and regression prevention (Level 4+ Performance Management Maturity).

- **#concept** The shared-cost argument: the reason scheduling isn't the easy option is that both approaches require the same deep understanding of page behaviour. Since that understanding is the expensive part, reordering can't be assumed cheaper or easier — it just moves where the difficulty lives.

- **#pattern** Performance Management Maturity: Russell repeatedly invokes a maturity ladder (Level 4 regression-prevention attributes, the "ladder" managers climb by confronting real user limits). Low-maturity orgs lack processes to push back on code growth, and scheduling fixes don't survive in them.

- **#concept** The brittle-vs-durable tradeoff: removal is durable because it reduces the system's surface area and its sensitivity to conditions; scheduling is brittle because it optimizes one journey at the expense of others and assumes a stable user base.

- **#tool** INP as the tell: deferred loading's damage shows up in Interaction to Next Paint data as stochastic "thuds" from background compilation, but is hard to attribute because of its unpredictable interplay with other resources and main-thread scheduling.

## Critical Analysis

**The strongest move is reframing the difficulty question.** The obvious objection to "remove code" is that it's *hard* — code is there for a reason, deleting it means negotiation and rework. Russell doesn't deny that; he denies the premise that scheduling is easier. By observing that both interventions cost the same deep understanding of page behaviour, he relocates the debate from "removal is hard" to "which hard thing is more durable once you've paid for it." That's a genuinely useful rhetorical and analytical shift.

**The "thuds" point is the most concretely technical argument and it's under-explained.** The claim that late-fetched JS generates main-thread stalls from residual background-compilation allocations is real and important — it's why naive code-splitting and `loading="lazy"` regress INP — but Russell gestures at it rather than explaining the mechanism. A reader who hasn't lived through a deferred-loading INP regression may not grasp *why* the bytes "thud," only that they do.

**The advice is calibrated to a specific, common-but-not-universal regime: large organizations running SPAs.** The bun-fight over the loading order, the "Team A's code harms Team B's experience" dynamics, the advocacy for preloading — these are artifacts of many-teams-one-SPA. A small team shipping a mostly-static site genuinely *can* get durable wins from lazy loading with near-zero coordination cost, and Russell mostly concedes this (scheduling is "special occasion food," consulting should be "trace-driven"). The essay's force is that its target regime is, in his experience, the modal one.

**The "evict all preloading" prescription is bracing but blunt.** He recommends evicting every preloading/deferred-loading system, sizing the product as it stands, then forcing trims — a shock-and-awe corrective for teams who've followed preloading down the primrose path. As he acknowledges, this isn't always right; a trace-driven team might find some preloading genuinely serving a latency-sensitive flow. The value is in the accountability-through-visibility reset, not the specific eviction.

**The connection to older org theory is there but thin.** The invocation of Fred Brooks ("communication effects dominate") and Adam Smith's pin factory is the right instinct — this is an essay about coordination costs wearing performance clothes — but Russell spends only a sentence on it. The deeper version: scheduling is a *global optimization problem* that needs product-owner priority, while removal is a *local* problem a team can own. That's the argument, and it's almost fully present in the text but never stated that crisply.

**What the essay shares with [[Queues Don't Fix Overload]] is the symptom-vs-cause structure.** A queue defers overload rather than eliminating it, and the slowness it hides is the canary. Resource scheduling defers bytes rather than eliminating them, and the reordering hides the true cost in the tail. Both essays argue the cheap, procedural fix is the one that makes the real problem invisible and the eventual failure more catastrophic. Hebert's "red arrow" — the hard limit no local optimization bypasses — has a frontend twin: bytes on the wire.

**The cleanest bridge to the agentic-performance literature is [[The Economic Benefit of Refactoring]].** Fowler measures the token cost of code an agent must *read*; Russell argues the durability of code a browser must *load*. Both share the reduction-over-rearrangement instinct — the payoff is in having less of the expensive thing in play, not in cleverly ordering the same amount of it. And Russell's decent→excellent / poor→decent split mirrors [[Performance Optimization Loop]]'s implicit scope: Gordon's loop is a decent→excellent tool, useless to a team whose real blocker is that nobody enforces a size budget.

---

*Sources: [[raw/notes-on-performance-remediation-strategies]], [[summary/notes-on-performance-remediation-strategies]]*
*Last updated: 2026-08-25*
