---
url: https://aibuddy.software/what-matters-after-the-software-factory-works/
date_fetched: 2026-09-15
---

# What Matters After the Software Factory Works

Six months and 1,700 pull requests into building an autonomous software factory, throughput turned out to be the least interesting part. What mattered was trust, calibrated actors, institutional memory, and intent. Those lessons eventually formed a governance model: PAAA.

In February I stopped talking to my coding agents in chat and built a loop instead. Issues in, code out, validation in the middle, reports at the end. If you follow this space you have seen a wave of posts lately describing the same move. The pattern even has a name now. People are calling them software factories.

Mine has been running for six months. Which is about as long as Dex Horthy was running his before he went back to reading all the code in *Harness Engineering is not Enough: Why Software Factories Fail*, so maybe consider this a retirement memoir.

**This is not a post about the architecture.** Plenty of people are writing those, and the architectures are converging anyway: a harness loop, a validation layer, an issue tracker doing double duty as memory.

This is a post about what turned out to matter *after the loop worked*. Five lessons, and then the shape I eventually noticed they made.

## Lesson 1: Throughput is the honeymoon

The loop worked almost immediately, and the numbers got silly fast. By June, four months in, the factory had merged around seventeen hundred pull requests. That figure impressed everyone I showed it to, including me.

It is also the least interesting thing about the whole experiment.

Throughput arrives fast because nothing is yet measuring whether it should. **A plausible change nobody has verified is not velocity. It is inventory.**

The honeymoon phase feels like magic precisely because the instruments that would tell you otherwise have not been built yet. Every factory post you read that leads with a big merge number, mine included, is describing this phase.

## Lesson 2: Trust is structural, not conversational

I spent a while trying to prompt my way to trustworthy output. Better instructions, better review prompts, sterner language. It did not move the needle, and in hindsight it could not have.

Trust came from structure.

Three structures, specifically.

Gates that fail closed: a change that cannot produce its required evidence does not merge, and when the gate machinery itself breaks, the system stops rather than shrugs.

Review made adversarial instead of confirmatory: reviewers charged to attack a change from a named angle, not to nod at it.

And decisions captured as architecture decision records the system has to obey, with conformance checked mechanically.

That last clause is the load-bearing one. **A rule without an enforcing check is an opinion, and a factory will politely route around opinions.**

The moment this lesson stuck: the factory once hit a resource limit and stopped completely, and nothing told me for two days. Silence and health look identical unless something is charged with telling the difference.

That became a rule with a check the same week.

## Lesson 3: Seats beat scripts

Somewhere in the middle months, the agents stopped being scripts and started being seats.

A product seat that grooms raw intent into well-formed work. Review seats that judge with standing charges, one attacking feasibility, one attacking security. Dissent filed by name. Decision records nobody, human or machine, can skip.

There was a rule underneath which seats changed hands.

Work converts to agents where its output can be verified mechanically; people survive where their judgment is the verification.

Then came the part I did not expect: *the judges themselves need judging.*

I started keeping ledgers of whether each seat's findings survived scrutiny, and re-checking old verdicts against what actually happened. Some findings turned out to be theater. Some reviewers agreed with everything, which is a defect, not a virtue.

**Named seats you can calibrate beat anonymous runs you can only rerun.**

That single sentence took me about four months to earn.

## Lesson 4: Memory is the unsolved half

Ask anyone running one of these systems what the hardest open problem is and you will hear some version of the same answer: memory.

Not context windows. *Institutional memory.*

What the system knows, why past decisions were made, which lessons have already been paid for.

What has worked for me, imperfectly: a wiki the factory maintains about its own repositories, decision records created as a side effect of work rather than as homework, and retrospectives run over the transcripts so the factory learns from its own history instead of re-living it.

An artifact is evidence on the day it is filed and memory ever after. The same records that prove the work teach the next run.

And memory does not have to mean infrastructure. **The smallest version is nothing more than decision records, filed where the next run will look.** The wiki and the retrospectives of build transcripts compound it; they are not the price of entry.

The test I use is simple: month six should be cheaper than month one, and the difference is memory.

When every run has to re-learn the codebase's quirks, you do not have a factory. You have an expensive amnesiac.

## Lesson 5: Intent is the scarce input

The strangest thing I observed in six months: I went quiet for two weeks, and the factory did not rest.

It hunted.

With no fresh direction coming in, its self-generated work became the dominant intake, and it busily optimized noise. Nothing was wrong, exactly. Everything was running.

It just was not running toward anything.

That taught me where the human seat actually is in one of these systems. It is not reviewing code, and increasingly it is not even reviewing evidence. It is upstream of everything: knowing what the system should want next.

The machinery can execute intent, refine it, and generate endless candidate intent. What it cannot reliably do is decide *what is worth wanting.*

Direction turned out to be the one input I could not automate, and the factory's behavior degraded fastest when it ran short.

## The shape the lessons made

Somewhere around month five I noticed these lessons were not a list. 

They were also not about the machinery. The machinery is becoming a product. The architectures converged first, and now the platforms are absorbing them; the loop I hand-built in February is on its way to being something you configure rather than something you write. Every lesson above lives in the layer no platform ships.

A governed development organization needs four things: Purpose, Articles, Actors, and Artifacts. Together, they form PAAA.

**Purpose** gives the work its direction.**Articles** define what must remain true: the rules, constraints, gates, and obligations the organization operates under.**Actors** do the work under delegated authority.**Artifacts** preserve what the organization knows and produces: its decisions, its evidence, its memory.

Throughput without trust is Actors without Articles. Calibrated seats are Actors done properly. The memory problem is Artifacts. The quiet-fortnight problem is Purpose.

PAAA is closed under its own governance: Articles govern Actors operating on Artifacts, and those same Actors can use Artifacts to propose, challenge, and ratify changes to the Articles themselves.

Six months of factory-building kept succeeding exactly where one of the four was strong and wobbling exactly where one was weak.

Here is what four letters look like on an ordinary Tuesday. I ask the factory to extend data import, and say why. That isPurpose. An old decision record says imported records are append-only, and a gate checks conformance mechanically.Articles. The product seat splits the work, a builder proposes a change that quietly mutates records in place, and the security seat attacks it by charge, not by mood.Actors. The rejected pull request, the dissent, and a fresh decision record all get filed.Artifacts. The next run reads that record and ships the append-only version without being told. No heroics, no prompting. The organization just remembered.

What I built, it turns out, was not a pipeline. It was a very small organization. A development organization in a box, with founding documents I wrote one incident at a time. Staffed by both humans and agents.

## The takeaway you can use

If you are running a factory experiment of your own, and the wave of posts suggests many of you are, you do not need my architecture.

Audit what you have with four questions instead.

Where does its direction come from, and what happens when that goes quiet?

What must remain true in your repo, and what enforces it when nobody is looking?

Who holds which seat, and how would you know if a seat went bad?

What does the system remember, and where does that memory live?

*The gaps you find are your next six months.*

They were mine.
