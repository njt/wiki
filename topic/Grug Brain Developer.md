# Grug Brain Developer

A cult-classic essay on software development written in a deliberately simple
"caveman" voice, distilling decades of hard-won engineering lessons into ~40
short, funny, and piercingly accurate aphorisms. The grug persona — a
self-deprecating developer who "not so smart" but has "program many long year"
— is a vehicle for observations so sharp they'd feel preachy in standard prose.
Published anonymously around 2022–2023, it has become one of the most beloved
and frequently cited pieces of software engineering writing on the internet.

---

## Key Quotes

> "apex predator of grug is complexity. complexity bad. say again: complexity *very* bad. *you* say now: complexity *very*, *very* bad."

The essay's refrain, repeated as a litany. The deliberate simplicity of the
phrasing is the point: complexity is so obviously the enemy that it doesn't
need a sophisticated argument. It needs a chant you can't forget. The
repetition has become a meme, but the meme works because every experienced
developer has felt the complexity demon enter a codebase between one day and
the next.

> "best weapon against complexity spirit demon is magic word: 'no'"

This is the operational advice behind the diagnosis. If complexity is the apex
predator, the primary defense is refusing to let it in. "No, grug not build
that feature. No, grug not build that abstraction." Grug notes this is "good
engineering advice but bad career advice" — "yes" earns more shiney rocks and
promotions — but grug must "to grug be true." This is the same tradeoff Kent
Beck frames in [[The Cost YAGNI Was Never About]]: saying no isn't thrift, it's
preserving optionality. Grug articulates it as moral clarity; Beck articulates
it as price theory. Same conclusion, different vocabulary.

> "grug try not to factor in early part of project and then, at some point, good cut-points emerge from code base. good cut point has narrow interface with rest of system: small number of functions or abstractions that hide complexity demon internally, like trapped in crystal."

The essay's advice on when to abstract — not never, but not early. Wait for the
"shape" of the system to emerge, then trap complexity behind narrow interfaces
when the cut points become visible. Grug admits "sometimes grug go too early
and get abstractions wrong, so grug bias towards waiting." This is the
practitioner's version of the "premature abstraction is the root of much evil"
principle. [[The Economic Benefit of Refactoring]] provides the modern,
quantified version: semantic decomposition after the fact reduces token costs
by 83%.

> "unit tests fine, ok, but break as implementation change (much compared api!) and make refactor hard and, frankly, many bugs anyway often due interactions other code. often throw away when code change."

Grug's testing philosophy in one sentence. Unit tests are fragile; end-to-end
tests are opaque; integration tests at API boundaries are the sweet spot. Write
tests *after* prototyping, when the code has firmed up — not before you
understand the domain. The one exception: when a bug is found, write a
regression test first, *then* fix. This is a pragmatic middle ground between
"test-first always" dogma and "test-never" chaos, and it rhymes with the
[[Guardrails and Feedback Loops]] principle that deterministic verification at
stable interfaces beats brittle end-to-end coverage.

> "grug very like type systems make programming easier. for grug, type systems most value when grug hit dot on keyboard and list of things grug can do pop up magic. this 90% of value of type system or more to grug."

A heresy to type-system purists, and almost certainly correct. Grug values type
systems primarily as an IDE tool (autocompletion, navigation), not as a
correctness proof. "Big brain type system shaman often say type correctness
main point type system, but grug note some big brain type system shaman not
often ship code." The observation that the people most vocal about type-system
correctness are often the least productive is unfair but not wrong. See also
[[The GUS Stack — Go, Unix, SQLite]] on picking languages agents already know —
Go's type system is deliberately simple, and that simplicity is a feature.

> "grug much prefer put code on the thing that do the thing. now when grug look at the thing grug know the thing what the thing do, alwasy good relief!"

The essay's case for Locality of Behavior (LoB) over Separation of Concerns
(SoC). Grug is "much more sour faced" about SoC than about DRY, and argues for
keeping related code together rather than splitting it across files by
architectural role. This is the same instinct behind HTMX ([[The GUS Stack]])
and the broader reaction against front-end framework over-engineering. When you
have to grep five files to understand what one button does, the separation has
become a tax rather than a benefit.

> "grug not like big complex front end libraries everyone use. grug make htmx and hyperscript to avoid."

Grug reveals himself as the creator of HTMX (Carson Gross). This explains the
essay's particular hostility toward front-end complexity: it's not just a rant,
it's the philosophical foundation for an alternative tooling ecosystem. The
essay's argument that "now you have two complexity demon spirit lairs" when you
split front-end and back-end with an SPA + GraphQL stack is both funny and
substantively the case for server-rendered HTML with minimal JS.

> "always grug one of two states: grug is ruler of all survey, wield code club like thor OR grug have no idea what doing. grug is mostly latter state most times, hide it pretty well though."

The essay's closing observation on impostor syndrome. Every developer oscillates
between god-mode and cluelessness, and the second state is far more common. The
advice: accept it, hide it reasonably well, and recognize that "nobody imposter
if everybody imposter."

## Key Themes

#concept #software-craft #simplicity #abstraction #testing #type-systems #api-design #frontend #impostor-syndrome #humor #cult-classic

### Complexity Is the Apex Predator

The essay's central thesis, repeated as a mantra. Complexity enters codebases
through well-meaning developers and project managers who don't fear it enough.
Once inside, it makes changes in one place break unrelated things elsewhere. The
complexity demon "mock mock mock" — it can't be seen, only sensed through its
effects. This is the same insight as [[Simplicity in the Age of AI-Assisted]],
which argues that LLMs' real value is as demolition tools for inherited
complexity. Grug got there first, with better jokes.

### Say No, Then Find the 80/20

The practical weapons against complexity: (1) say no to features and
abstractions, (2) when you must say yes, build the 80/20 solution — "80 want
with 20 code" — rather than the full spec. Grug notes you can sometimes just
build the 80/20 version without telling the project manager, because "easier
forgive than permission." This is the folk-wisdom version of [[The Cost YAGNI
Was Never About]]: both argue that building less is the highest-leverage
engineering decision, and both note that the organizational incentives push the
other way.

### Wait for Cut Points

Don't factor early. Wait for the system's shape to emerge, then refactor at the
natural seams — the "cut points" where a narrow interface can trap complexity.
Big-brained developers who invent abstractions at project start get it wrong,
and grug has to maintain the result. The advice to "give them a UML diagram or
demand a working demo tomorrow" is a management technique disguised as a joke.
[[The Economic Benefit of Refactoring]] provides the modern empirical version:
semantic decomposition after the fact (not before) yields 83% token savings.

### Integration Tests Are the Sweet Spot

Unit tests are fragile (break on refactor), end-to-end tests are opaque (hard to
debug), integration tests at API boundaries are "peak grug testing." Write
tests after prototyping, focus ferocious integration test effort as cut points
stabilize, maintain a small curated E2E suite for the most important paths.
Dislike mocking — use only when absolutely necessary, and only coarse-grained.
This testing philosophy is more thoughtful than most "TDD vs not-TDD" debates,
and it's held up well in practice.

### Tools Over Process

Grug is agnostic about methodology ("agile not terrible, not good") but
passionate about tools: a good debugger is "worth weight in shiney rocks," IDE
code completion makes Java "nearly impossible without it," and logging is so
important that Google put Rob Pike on it. The essay's advice to learn your
debugger deeply — conditional breakpoints, expression evaluation, stack
navigation — teaches "more about computer than university class often." This
aligns with [[Software Engineering Craft]]'s emphasis on operational craft over
methodology.

### Type Systems Are for Autocompletion

The essay's most contrarian technical claim: 90% of a type system's value is
hitting "." and seeing what methods are available. Correctness is secondary.
Grug warns that big-brained type-system enthusiasts "think in type systems and
talk in lemmas" and produce code that's "astral projection of platonic generic
turing model" — elegant but useless for counting club inventory. Generics are
"especially dangerous" and should be limited to container classes. This is a
deliberately provocative take, but it captures something real about how most
working developers actually use types.

### Locality of Behavior

The essay's alternative to Separation of Concerns: put the code on the thing
that does the thing. When you look at a button, you should see what the button
does. LoB is the design principle behind HTMX and the broader "HTML-first"
movement. It's a reaction against the front-end pattern where understanding one
UI element requires reading files scattered across component, style, test,
story, and state management directories. Grug's phrasing is funny, but the
underlying critique of SoC-overreach has aged well.

### Front-End Complexity Is a Disaster

"Splitting front-end and back-end codebases with a hot new SPA library talking
to a GraphQL JSON API" creates "two complexity demon spirit lairs." The front-end
one is worse — "even more powerful and have deep spiritual hold on entire front
end industry." Grug's alternative (HTMX + hypermedia) isn't for every project,
but the diagnosis that the front-end ecosystem has normalized pathological
complexity for simple use cases is widely shared. [[What Frontend Developers
Still Hate — 2026 Survey]] confirms that the hardest front-end problems remain
coordination problems masquerading as technical ones.

### Impostor Syndrome Is Universal

The essay closes by normalizing impostor syndrome: every developer oscillates
between "ruler of all survey" and "no idea what doing," with the latter being
far more common. The advice to senior developers to say "this is too complex
for me" out loud — to defeat FOLD (Fear Of Looking Dumb) — is genuinely good
management advice. Making it safe for juniors to admit confusion removes "major
source of complexity demon power over developer."

## Critical Analysis

**Why this essay works.** Most software engineering wisdom is delivered in
earnest blog posts that make reasonable points nobody remembers. Grugbrain.dev
delivers the same points in a voice so distinctive that the takeaways stick.
The grug persona isn't a gimmick — it's a rhetorical device that lets the
author make sweeping claims without pomposity, admit mistakes without
defensiveness, and deliver genuinely sharp observations without sounding like a
consultant. The essay is funny, but it's not *just* funny. The humor is
load-bearing: it makes the medicine go down.

**What's substantively right.** The complexity-as-apex-predator thesis is
correct and has become more correct over time. The advice to wait for cut
points before abstracting is the right default for most projects. The
integration-testing sweet spot is real and underappreciated. The observation
that type systems' primary practical value is tooling, not correctness, is
heretical in some circles but describes how most developers actually work. The
critique of front-end over-engineering has been vindicated by the HTMX
movement's success.

**What's oversimplified.** The essay treats all complexity as bad, but some
complexity is load-bearing — the Kubernetes cluster exists because you actually
need multi-region deployment. As [[Simplicity in the Age of AI-Assisted]] notes,
the hard judgment is distinguishing essential from inherited complexity. Grug's
framework doesn't help with that distinction. The essay also underplays the
cases where saying "yes" is the right call: platform investments that unlock
multiple features, shared infrastructure with network effects, or situations
where the cost of deferring is higher than the cost of building now.

**The HTMX connection.** Knowing that grug is Carson Gross changes how you read
the essay. The front-end complexity critique stops being a general rant and
becomes the philosophical foundation for a specific technical alternative. This
doesn't invalidate the critique — HTMX's real-world adoption suggests it solves
real problems — but it means the essay is also a manifesto for a particular
approach to web development. Readers who don't share Gross's architectural
preferences may find the front-end sections less persuasive than the
language-agnostic wisdom about complexity, testing, and tools.

**What's missing.** The essay doesn't engage with distributed systems,
security, observability, or operations — the domains where complexity is often
most load-bearing and least optional. Grug's world is implicitly the world of
monolithic applications built by small teams, and the advice is strongest in
that context. It also doesn't address the organizational dynamics that make
complexity inevitable: Conway's Law, the tendency of successful products to
accumulate features, the career incentives that reward building over removing.
Grug names the problem but doesn't offer a structural solution beyond
individual discipline.

**The essay's relationship to AI-era development.** Written before coding
agents went mainstream, the essay's advice has become *more* relevant, not
less. When agents can generate code 10x faster, they generate bad abstractions
10x faster too. Grug's "wait for cut points" and "say no to abstractions"
become even more important when the cost of creating complexity has dropped to
near-zero. The essay's emphasis on tooling (debugger, IDE, logging) also
applies to agent-assisted development: the harness is the tool, and investing
in harness quality pays compounding returns. See [[Loop Engineering]] and
[[Coding Agents and Complexity Budgets]].

---

*Sources: [[raw/grugbrain-dev]], [[summary/grugbrain-dev]]*
*Last updated: 2026-08-08*
