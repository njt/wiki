# Crew Resource Management — Joseph Pelrine (Craft 2025)

Joseph Pelrine — psychologist, agile pioneer, one-time flatmate of Kent Beck during Europe's first XP project — argues that high performance is an environment problem, not a talent problem. Borrowing Crew Resource Management (CRM) from aviation and medicine, he prescribes non-technical interaction ground rules as the thing that lets technical skill survive stress, and inverts the psychological-safety dogma: safety is not the prerequisite for high-performing teams but the emergent *result* of ground rules that build trust.

---

## What it argues

- **Talent ≠ team.** "You get a group of really talented individuals together and you think, wow, this is going to be a great team — and then they act like a bunch of idiots" (his example: the Swiss national football team, "every year"). Like functional vs non-functional requirements, teams need technical and non-technical skills — and the non-technical ones are what make the team perform. HR keeps hiring for technical skill because it can't test for team fit, so Pelrine's move is to stop optimizing hiring and change the environment instead.
- **Three failure modes**, from psychological research: human-factor errors (cognitive overload), fixation errors, and communication errors. Error rates: about one mistake every 30 minutes on routine unstressed tasks, every 5 minutes on complex unstressed ones, every 30 seconds under stress.
- **Overload → tunneling → fixation.** Under cognitive load you narrow focus and "keep doing more of the same." Google research: developers debug the wrong thing 50% of the time. Fixation is why police fixate on one suspect and why we all re-run the same failing test.
- **CRM is the fix.** NASA-originated ground rules for action and interaction, proven in aviation and medicine, applied by Pelrine in helicopter rescue, Lufthansa, Formula 1 (Ferrari), and Swiss champion football. The stakes he claims: "70% of production problems come from human factors, not tech errors."
- **The inversion.** Psychological safety is recast from prerequisite to result: ground rules → trust (an interpersonal construct) → psychological safety emerges in the culture. Grounded in Lewin's B = f(P,E) — change the environment, not the person. Culture is "a derivative property": you can't change it directly.
- **XP is already CRM.** Extreme Programming is a set of interaction ground rules, not a technical-skills spec. His best XP team, under stress, turned the practices "up to 11 or 12" instead of letting them slip — because trust meant nobody had to check anybody.
- **Systems beat blame.** The person-centered view of error ("bad people") yields fear and punishment; the systems approach treats human error as a consequence, not a cause. Via James Reason's Swiss cheese model, high-performing teams ask not "how do we get better" but "how can we make it systematically more difficult for these errors to become an incident?"

## Key quotes

> "You get a group of really talented individuals together and you think, wow, this is going to be a great team — and then they act like a bunch of idiots."

The opening problem statement, delivered with the Swiss national football team as his annual proof. It frames the whole talk: the deficit is never skill, it's interaction.

> "...a set of ground rules for action and interaction among the people in a team that give the team the ability to do their technical skills as well as possible — even when the shit hits the fan." (his definition of CRM)

Note what CRM is not: not a personality fix, not a hiring filter, not a values poster. A protocol layer under the technical work.

> "I have 15 minutes in a pre-flight briefing to take these people who have never met each other and make them into a high-performing team who I can risk with my life." — Sandra, Lufthansa purser and CRM instructor

Lufthansa never staffs the same crew twice — deliberately destroying team continuity to level the playing field. Her verdict: "Without CRM, it would be impossible. With CRM — guaranteed, every time." This is the strongest evidence in the talk, because it shows the ground rules working *without* familiarity, history, or friendship.

> "Every second counts is bullshit. If you think that way, people die." (introducing the 10-for-10 principle)

Hurry pushes you into Kahneman's System 1 — "quick, intuitive, and often wrong." The 10-for-10 countermeasure: stop 10 seconds, breathe, discuss what you'll do for the next 10 minutes or until something changes.

> "You're a crew — you're not one of these pseudo-agile self-organizing teams where everybody has equal power and you discuss things so long that nothing ever gets done."

A direct hit on the flattest reading of agility: CRM requires accepted roles and a leader, official or not. His Netherlands anecdote — decision in 15 minutes, then three hours re-discussing it — is the reductio.

> "It's just a lot easier to blame the people than to say our organization is screwed up." (paraphrasing James Reason)

The political economy of error handling in one line. Blaming is cheaper, which is exactly why the systems approach loses in most organizations.

> "CRM is not going to help your product manager from changing requirements — and it'll help you survive them doing so."

The most honest scoping claim in the talk: CRM is resilience engineering for the team, not a fix for the organization around it.

> "The only reason for HR's existence is to stop management from being sued from lawsuits. Sorry." (Q&A)

Delivered deadpan; it doubles as his explanation for why HR can't hire for team fit — the function was never designed for it.

## Practices worth stealing

- **10-for-10** — anti-fixation: stop 10 seconds, breathe, agree the next 10 minutes as a team. FOR-DEC (Facts, Options, Risks/benefits, Decide, Execute, Check) is the same loop formalized — "essentially doing scrum on medicine."
- **Closed-loop communication** — SBAR (Situation, Background, Assessment, Recommendation) for status handoffs; challenge-response from aviation; order-repeat from kitchens. Installable in standups, handoffs, and deploys tomorrow.
- **Pair-switching after 30 minutes of debugging** — his software-specific anti-tunneling rule, aimed straight at the "debugging the wrong thing 50% of the time" statistic.
- **Sunset-date experiments** — his Kent Beck pattern: "let's try this for a sprint," explicit end date, then review and let the team decide. "You can't force people to do things. But you can hold them to their commitment."
- **Systemic barriers over exhortation** — the Swiss pharmacy's QR prescription, non-confusable shelving, barcode check, second-person sign-off: about one minute added, error path closed. The Finnish reindeer antlers example shows the creative form *and* its failure mode (drivers mistook reflective antlers for pedestrians) — a lesson in unintended consequences the talk never extracts.
- **Reframe the retrospective** — not "who's to blame" but "how did our system defenses mess up?" Adoption advice is deliberately modest: pick one habit, don't memorize all fifteen.

## Themes

#person Joseph Pelrine — psychologist and agile veteran; #concept psychological safety as emergent result, systems vs person-centered error; #pattern closed-loop communication, systemic barriers, sunset-date experiments; #tool Stroop test, SBAR, FOR-DEC, Swiss cheese model.

## Critical take

This is the rare agile talk whose core content predates agile. CRM has fifty years of evidence in domains where error means corpses, and Pelrine has actually installed it in cockpits, operating theaters, and football squads — which gives him an authority no framework vendor can fake. The psychological-safety inversion is genuinely sharp: if safety is an emergent property of enforced interaction ground rules, then the industry's habit of exhorting safety directly ("be vulnerable!") is treating a derivative as an independent variable — exactly the confusion he names with "culture is a derivative property."

But the talk leans on borrowed credibility and knows it. The headline numbers (70% of production problems from human factors, Google's 50% wrong-debugging) are asserted without sources, and every success story is from a life-critical domain — nothing shows CRM moving the needle on a software team whose worst-case outcome is a late sprint. The transfer problem is real and he concedes it in one sentence: the 15 rules are "set up for medicine," aviation's differ, and software gets neither set. What does "trust each other with your life" even mean when the stakes are a deployment? Aviation also hands CRM a structural gift software lacks — a captain whose own life is on the line, which is why his answer to contested leadership legitimacy ("issues that will have to be dealt with outside of that") and his dodge of the SAFe question matter: the model assumes authority that most software teams don't have and can't confer. And the silence on remote teams is close to disqualifying in 2025: 15-minute pre-flight briefings and spoken challenge-response protocols assume co-location. Meanwhile the industry work that already occupies this ground — Google's Project Aristotle on team effectiveness, SRE blameless postmortems — goes unmentioned, which makes a genuinely new-sounding idea look less new than it is. The retro reframe and one-habit adoption advice are the honest, usable core; the rest is a powerful analogy still waiting for its software-specific translation.

## Relations

- [[Engineering for Bounded Cognition]] — strengthens it with the team-level layer: that page treats working memory and attention as individual constraints to design around; Pelrine supplies the failure cascade (overload → tunneling → fixation) and the interaction protocols that keep a *group* thinking clearly inside those same limits.
- [[The Forest and the Desert Are Parallel Universes]] — Kent Beck is a character in this talk (Pelrine shared a Munich flat with him doing Europe's first XP project, and uses Beck's sunset-date pattern verbatim), and both talks locate performance in the environment rather than the person; Pelrine's Lewinian B = f(P,E) is the psychological formalization of Beck's parallel universes.
- [[Building Resilience — Tricia Broderick (Craft 2025)]] — nuances it directly: Broderick treats trust, courage, and psychological safety as capacities leaders must cultivate; Pelrine argues safety can't be cultivated directly at all, only emerged from ground rules — a mechanism-first correction to her capacity-first framing.
- [[Shaped by Demand — The Power of Fluid Teams]] — complicates it from the other side: Dan North demolishes Tuckman's stable-team orthodoxy, and Pelrine's Lufthansa evidence (never fly with the same crew twice, high performance *with strangers*) shows stable-team formation isn't just unnecessary, it's deliberately engineered away in the highest-stakes teams that exist.

---
*Sources: [[raw/crew-resource-management-the-real-secret-to-high-performing-teams-joseph]], [[summary/crew-resource-management-the-real-secret-to-high-performing-teams-joseph]]*
*Last updated: 2026-09-13*
