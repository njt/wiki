# You Should All Be Asking Way More Questions

Sean Goedecke makes the case for interrupting explanations with a question every thirty seconds — short confirmation questions that catch misunderstandings before they compound — and extends the habit to working with AI agents, which he treats as inherently unreliable colleagues who make design (not code) mistakes because they are "always on their first day."

---

Goedecke's mechanism is compounding: a small early misread spawns further misreads, so questions can't be batched until the end. Most people don't ask because they lack skin in the game — they defer to senior engineers — but once you're responsible for acting on the plan, questions become inevitable. The skill behind the questions is *building the plan in your head*: visualising data flow, service auth, and persistence, so a vague "service X stores data" (X only talks to ephemeral Redis) instantly triggers "how is that going to work?" — usually revealing a missing dependency or a design change. Early questions can catch fundamentally unworkable designs, like the elegant event-driven system he remembers that made single-datacenter customer-data siloing impossible and had to be abandoned.

The agent turn: good models rarely make *code* mistakes but constantly make *design* ones — assuming services can talk, forgetting on-prem constraints — because they have no continuous learning. His fix is to pepper agents with questions: "Does service Y really support this type of authentication?", "Is this subsystem you built necessary to satisfy requirement Z?", "Why do we need to touch this file? Isn't that unrelated?" He reports roughly half the answers reveal a model mistake, and he'll stop asking when that stops — archly noting he may then ask his company to keep paying him "to occupy more of an architect role."

---

## Key quotes

> "A small misunderstanding early on balloons into a big misunderstanding later, because all the things you misinterpret based on the first misunderstanding will become misunderstandings in their own right."

The strongest structural argument in the piece, and the reason batched questions fail. It's the engineering analogue of compound interest on error — and it explains why real-time review beats end-stage review in every domain, not just conversation.

> "Language models are always on their first day."

His one-sentence explanation for why models make design mistakes while rarely making code mistakes: no continuous learning. The design knowledge of a codebase — which services can call which, what runs where — lives in your head, not the weights. This is one of the sharpest framings of the context problem, and it implies the fix is supplying that context, not hoping.

> "About half the questions I ask get answers that convince me the model has made a mistake... When this stops happening, I'll stop asking questions."

A falsifiable empirical claim with a defined exit condition — rare in agent discourse. The "half" figure is doing real work: it converts the habit from distrust-as-vibes to distrust-as-measurement, the same instinct behind verification-first workflows.

> "Nobody understands complex software products. If you have deep domain knowledge of any area of a codebase, you will routinely find yourself correcting powerful AI models and principal engineers."

The closing move equalises agents and senior engineers under the same epistemic discipline: neither is owed deference, only verification. It's also a quiet argument that deep domain knowledge is *appreciating* in the agent era, not depreciating.

## Key themes

#concept — comprehension debt and the compounding misunderstanding #pattern — build-the-plan-in-your-head as an interrogation technique #concept — "always on their first day" as a model of agent unreliability #person — Sean Goedecke

## Analysis

This is a small, opinionated essay that lands two genuinely useful ideas. First, the reframing of questioning as a *design-review-in-conversation*: the "build it in your head" discipline is really incremental specification — you're forcing the explainer (human or model) to commit to data flow and persistence before implementation happens, which is spec-driven thinking applied in real time rather than in a document. Second, the treatment of agents as junior colleagues with senior output: fluent code, unreliable architecture, permanently on day one.

The honest tension is that Goedecke's method is expensive — a question every thirty seconds is exactly the supervision burden that [[Human-in-the-Loop is Tired]] diagnoses as exhausting. He can sustain it because he's a staff-level engineer reviewing plans; the paper he never addresses is whether this scales to a team of people who *don't* have his domain depth. And his sycophancy footnote ("good coding models are very happy to robustly defend themselves") is empirically hopeful but under-argued — one of the most documented failure modes of current models is caving under leading questions. Still, "half the answers reveal a mistake" is exactly the kind of reported number this wiki should be collecting.

## Related pages

This sharpens [[Understand to Participate]] from a principle into a technique: Litt argues understanding is the prerequisite for staying an active collaborator, and Goedecke supplies the concrete mechanism — real-time interrogation — for getting there. It complicates [[Human-in-the-Loop is Tired]]: Summers diagnoses supervision fatigue, and Goedecke's one-question-every-thirty-seconds is precisely the kind of high-bandwidth supervision she warns about, yet he reports it as sustainable — the reconciliation may be that plan-review supervision burns far less than line-level review. And it converges with [[The Joy and Power of Understanding]] on the claim that comprehension, not output, is what the human still owns in an agent-heavy workflow.

---
*Sources: [[raw/you-should-all-be-asking-way-more-questions]], [[summary/you-should-all-be-asking-way-more-questions]]*
*Last updated: 2026-10-03*
