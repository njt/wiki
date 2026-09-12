# What Frontend Developers Still Hate — 2026 Survey

An informal whiteboard survey of 120+ frontend developers at JSNation and React Summit 2026 reveals what AI hasn't fixed: date pickers still top the annoyance list after a decade, AI tooling creates its own class of drudgery, and the hardest problems aren't technical — they're coordination problems that component libraries can paper over but never solve.

---

## Key Quotes

> "It sucked to build a date picker 10 years ago and it still sucks today."

The survey's thesis in one quote. Date pickers got 9 mentions — the most of any single category — and have been the canonical "looks easy, is actually a nightmare" component since jQuery UI. Timezones, accessibility, date ranges, keyboard navigation, mobile touch targets. Every framework ships one. Every team builds their own. Both decisions are wrong for different reasons. The date picker is a standing rebuke to the idea that tooling progress eliminates pain — it just shifts the pain around.

> "AI is clearly changing how developers build software, but it hasn't eliminated the need for thoughtful engineering."

The author's careful framing. AI has shifted developer effort toward reviewing, refining, and improving generated code rather than writing every line themselves. The question she leaves open — "whether that's an improvement or not is something only time will tell" — is the honest one. Review fatigue is real (see [[Human-in-the-Loop is Tired]]). The satisfying part shrank; the exhausting part grew.

> AI-generated code violates the rules of hooks, creates unnecessary `useState` calls, and places state outside components.

The specific AI failures developers reported. These aren't subtle architectural mistakes — they're mechanical violations of framework rules that a linter catches in milliseconds. The fact that AI routinely gets them wrong is a category error: the models treat React's rules as statistical patterns rather than hard constraints. This is the same phenomenon [[Constraint Decay]] documents at the backend level — give an agent constraints and it falls apart, not because it's dumb, but because constraint enforcement is a different capability from code generation.

> "The difficulty now (as it always has been) is in the integration of tools into cohesive systems."

The author's central argument, and the one that survives the vendor pitch. Better date pickers exist. Better data grids exist. Better accessibility tooling exists. The problem isn't a lack of solutions — it's that assembling them into a working product requires cross-team coordination, design alignment, and organizational decisions that no npm package can make for you.

---

## Key Themes

- **#concept** **The persistence of hard UI problems** — Date pickers, data grids, rich text editors, and comboboxes have been notoriously difficult for 10+ years. They're the "looks simple, contains a universe of edge cases" class of component. AI hasn't changed this because the difficulty isn't in generating code — it's in handling the combinatorial explosion of states, inputs, and accessibility requirements that make these components hard. A date picker that works for one locale and one timezone is easy. One that works for all of them is still hand-built.

- **#concept** **AI as drudgery displacement, not elimination** — The second-most-annoying category on Day 1 was "AI-related work": prompt engineering, integrating AI features, wrangling AI-generated designs. The tooling sold as drudgery-eliminator has created its own class of tedious work. This is the Jevons paradox applied to developer experience: making code generation cheaper increased the *amount* of code being generated, which increased the amount of reviewing, debugging, and wrangling required. See also [[Laura Tacho — Data vs Hype]] for the organizational version of this finding.

- **#concept** **Coordination problems masquerading as technical problems** — The recurring themes across both days (accessibility, design systems, Figma-to-code, localization, testing) all share a common structure: they're not about writing code, they're about aligning multiple people, teams, and tools. Accessibility requires design, engineering, and QA to share a standard. Design systems require component teams and feature teams to coordinate. Figma-to-code requires design and engineering to agree on a shared vocabulary. These are organizational problems that technical solutions can support but never replace. See [[Software Engineering at the Tipping Point]] for the same insight at the system level.

- **#pattern** **Rules-of-hooks violations as a canary** — AI's inability to consistently follow the rules of hooks is a perfect diagnostic: these are deterministic, well-documented, mechanically verifiable rules. If AI can't follow them, the problem isn't model capability — it's that transformer-based code generation has no mechanism for hard constraint enforcement. The implication is that [[Guardrails and Feedback Loops]] (deterministic enforcement, not prompt-level pleading) is the only architecture that works. You can't prompt your way to constraint compliance.

- **#concept** **The component library pitch as incomplete answer** — The author's recommendation is to use component libraries (specifically Progress Telerik/Kendo UI, her employer's products). This isn't wrong — a good component library does eliminate the date picker problem — but it's incomplete. It solves the component-level pain while leaving the coordination problems untouched. A design system is a component library plus organizational agreement. Buying components doesn't buy you the agreement.

- **#tool** **CSS as AI's blind spot** — CSS got 4 mentions on Day 2 as something AI still gets wrong, and appeared as a recurring theme across both days. This tracks with [[Constraint Decay]]'s finding that convention-heavy frameworks are traps for agents. CSS is the ultimate convention-heavy, implicit-behavior system: specificity wars, stacking contexts, layout modes that interact in non-obvious ways. AI generates CSS that looks right and behaves wrong — exactly the failure mode you'd predict for a system that learns statistical patterns rather than the cascade algorithm.

---

## Critical Analysis

**The survey is methodologically informal but empirically useful.** A whiteboard and sticky notes at two conferences isn't a randomized controlled trial, and the author doesn't pretend it is. But the consistency of responses across two different prompts at two different conferences is genuinely interesting. When 9 out of 120 people independently write "date picker" as their most annoying component, you're not looking at noise. The methodology limits what conclusions you can draw, but it doesn't invalidate the signal.

**The most interesting finding is buried: AI-related work is itself annoying.** Eight developers listed prompt engineering, AI-generated designs, and AI feature integration as their most annoying tasks. This is the finding the author's employer (Progress, which sells component libraries) has the least incentive to highlight, and it's the most important one. The tool that was supposed to make developers' lives better has become a source of frustration. Not because AI is bad — because integrating AI into real products is genuinely tedious work that nobody has figured out how to make elegant. The "last mile" of AI integration is all glue code and prompt tweaking.

**The rules-of-hooks finding is a stronger indictment than it looks.** If you asked 100 developers in 2019 what AI coding tools would be bad at, "following React's rules of hooks" would not have made the list. It's a mechanical rule. It's well-documented. There are linters for it. And yet AI violates it routinely. This means our intuitions about what AI "should" be good at are systematically wrong. We assume AI will fail at creative tasks and succeed at mechanical ones. The evidence says the opposite: AI is surprisingly good at generating plausible designs and surprisingly bad at following deterministic rules. This inverts the automation thesis that's been driving tooling investment.

**The vendor agenda is visible but doesn't invalidate the findings.** The author works for Progress (Telerik/Kendo UI). The article's conclusion — "buy a component library" — aligns with her employer's commercial interests. But this is disclosed (her bio is prominently featured), and the survey data was collected before the recommendation was written. The conflict of interest is a framing problem, not a data fabrication problem. The survey results stand on their own. The recommendation is a commercial pitch that happens to be directionally correct — component libraries do solve some of these problems — while conveniently omitting that open-source alternatives exist and that the coordination problems are untouched by any library, commercial or otherwise.

**The silence on frameworks is strategic.** React was the context for Day 2's prompt. But the survey doesn't ask whether React itself is part of the problem. The rules-of-hooks violations, the CSS difficulties, the state management complaints — all are at least partially React-specific. A survey at a Svelte or Solid conference would surface different pain points. This isn't a criticism of React; it's an observation that framework choice shapes which problems you have, and the survey's framing takes React as a given rather than a variable.

**The Figma-to-code complaint deserves more attention.** It appeared as a recurring theme but didn't get top billing. This is the problem that has consumed entire VC-backed startups (Framer, Builder.io, Locofy, Anima) and remains unsolved. The reason it's unsolved isn't technical — it's that Figma designs and production code serve different masters. Figma optimizes for visual fidelity and designer ergonomics. Production code optimizes for maintainability, performance, and accessibility. The translation between them requires judgment that neither design tools nor AI have demonstrated. This is the coordination problem in its purest form: two groups with different goals sharing an artifact that can't serve both.

**The survey inadvertently makes the case for spec-driven frontend development.** If AI can generate plausible components but can't integrate them into a coherent system, the valuable artifact isn't the generated code — it's the specification of what the system should do. [[Specifications as the Product]] applies to frontend as much as backend. The design system is the spec. The accessibility requirements are the spec. The component API contracts are the spec. AI can generate implementation against a spec; it can't generate the spec itself. The frontend teams that thrive will be the ones that invest in specifications, not the ones that generate the most code.

---

## Related

- [[Constraint Decay]] — AI's structural failure mode when constraints accumulate; CSS and framework rules are the frontend equivalent of database backends
- [[Laura Tacho — Data vs Hype]] — 121K-developer dataset confirming AI adoption doesn't equal transformation; the organizational version of this survey
- [[Human-in-the-Loop is Tired]] — Laura Summers on review fatigue; "the satisfying part shrank, the exhausting part grew" maps directly to AI shifting work from writing to reviewing
- [[Guardrails and Feedback Loops]] — Why deterministic enforcement beats prompt pleading; rules-of-hooks violations are the perfect case study
- [[AI Slop Starts with the Codebase Itself]] — AI output quality depends on the codebase it's generating into; CSS and React conventions are path-dependent
- [[Nicole Forsgren on AI and Developer Productivity]] — The inner-loop/outer-loop bottleneck shift; AI accelerates coding but coordination remains the constraint
- [[Software Engineering at the Tipping Point]] — AI as 10× amplifier; amplifies existing problems as much as existing capabilities
- [[Specifications as the Product]] — If AI can't handle integration, the spec is the durable artifact
- [[Lessons for Reusable Web Components]] — Practical rules for the component-level work that AI can assist but not own
- [[Performative UI]] — Satirical catalog of frontend tropes; the design-convergence risk when AI generates from statistical patterns
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines; the architecture that might keep AI in the decision role while deterministic tools handle constraint enforcement
- [[Agentic Testing]] — Slack's empirical study of agent-driven testing; same pattern of AI struggling with complex, constraint-heavy domains
- [[React useMemo and useCallback]] — Josh Comeau's definitive mental-model explainer for the hooks AI most often misuses; his "profile first, optimize in response to data" discipline is exactly what AI-generated React code lacks

---
*Sources: [[raw/ai-cant-solve-all-frontend-developers-survey]]*
*Last updated: 2026-07-18*
