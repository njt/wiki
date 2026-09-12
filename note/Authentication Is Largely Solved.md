# Authentication Is Largely Solved

Phil Windley's book-launch essay for *Authorization in Action* (Manning) argues the identity field has spent two decades solving the wrong half of its problem: authentication is now "about as good as we can reasonably ask," while authorization — deciding what someone may do, under what conditions, on whose behalf, in which context — is still improvised, buried in app code, and reinvented badly on every team. AI agents turn that gap from a chronic annoyance into the whole game.

---

## Key Quotes

> "Authentication, the act of proving who someone is, has become about as good as we can reasonably ask; passkeys and FIDO have quietly closed most of the gap that kept me up at night in 2005."

The claim the whole essay leans on. Windley isn't saying authentication is perfect — he's saying it crossed the line from unsolved to "good enough, shipping, done." The rhetorical move is worth naming: the essay needs authentication to be *closed* so the interesting work can be *reallocated* to authorization. Whether the long tail (account recovery, shared devices, legacy systems) actually cleared that bar is the part he compresses past.

> "Knowing who someone is tells you almost nothing about what they should be able to do; the interesting, unsolved work all lives on the far side of authentication."

The thesis distilled. A login gets you through the door; it says nothing about which rooms you can enter, what you can carry out, or whether the person who sent you had the standing to send you at all. This is the same seam [[Authorization Terminology]] picks at from the terminology side — "who" is the identity question, "what you may do" is a different question entirely, and conflating them is how teams end up with role sprawl and improvised checks.

> "What can this person do; under what conditions; on whose behalf; and in which context?"

The essay's sharpest decomposition. Authorization isn't one question but four, and missing any one makes the rest stop meaning much — the same request is right for a manager at noon from the office and wrong for a contractor at midnight from an unknown device. The "on whose behalf" clause is the one that matters most for agents, and it's the one classical authorization frameworks have the least language for.

> "An agent that can read your calendar, spend your money, or send mail as you is only as trustworthy as the boundaries around it, and those boundaries are authorization."

Where the technical question turns human. The essay's real claim is that delegation infrastructure *is* the agent problem: without the ability to say precisely what an agent may do on our behalf, we're left choosing between agents that can't do anything useful and agents we have to trust blindly. "Authorization is the infrastructure that lets us delegate real authority to software while keeping it bounded, accountable, and revocable" — language nearly identical to [[Golem Covenant]]'s "bounded, answerable, revocable agents," converging from opposite directions.

> "Fine-grained authorization doesn't only make a system safer, it makes it more usable."

The most interesting and least-defended claim, learned on the AWS Identity team building Amazon Verified Permissions and Cedar. When rules about who can do what are explicit and external to the code, you can hand people exactly the access they need without drowning them in prompts or handing over the keys. It inverts the conventional security-versus-usability tradeoff — and it's the thesis that only an internal build would produce, because the evidence is organizational intuition, not measurement.

## Key Themes

#person **Phil Windley.** Identity's long arc in one career: CIO of Utah (2001), co-founder of the Internet Identity Workshop with Doc Searls and Kaliya Young (now 42 meetings), author of *Digital Identity*, *The Live Web*, and *Learning Digital Identity* — each book answering the question the previous one raised, with "identity is about relationships, not identifiers" as the landing point. This fourth book is the next rung.

#concept **Authentication closed, authorization open.** Passkeys and FIDO closed the phishing-resistant-login gap; the decision about *what you may do* never got the same standards treatment. Windley's framing is that the field's energy was misallocated for twenty years — the hard problem was always one question over.

#concept **The four sub-questions of authorization.** What, under what conditions, on whose behalf, in which context. A decomposition that anticipates the identity-ambiguity diagnostic in [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] (is the agent acting as itself or as a delegate?) — "on whose behalf" is exactly the axis agents scramble.

#tool **Cedar and Amazon Verified Permissions as the concrete instantiation.** Windley's answer isn't abstract: externalize policy in a declarative language with a real engine, separating the *decision* from the *enforcement* — the same decide/enforce split [[Authorization Terminology]] places on its decision axis. The commenter's note that Cedar lead Sarah Cecchetti wrote the book's foreword, and demonstrated Cedar for AI-agent guardrails on a standalone "Clawdrey Hepburn" Mac mini (clawdery.com), closes the loop: the essay's abstract argument has a shipping reference implementation.

#pattern **Externalized policy as usability.** Fine-grained, code-external authorization stops the safety/pleasantness fight. The mechanism — explicit, auditable rules instead of prompts or god-mode — is the same argument [[The Agent Access Model]] makes for runtime enforcement: a boundary you can't talk your way past is the one that can also be generous without being dangerous.

## Critical Analysis

**It's a book announcement, and it should be read as one.** The essay's job is to make the thesis feel inevitable, not to defend it. The four sub-questions, the Cedar reference, the agent framing — all are compressed versions of arguments the book presumably spends chapters on. That's not a flaw, but it means this source carries *thesis*, not *mechanics*: it tells you the right question, not how to answer it. For mechanics, the field's actual answers live in [[The Agent Access Model]] (runtime, ratcheted, task-scoped) and [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]] (design-time, four-pillar).

**"Authentication is solved" is doing rhetorical work.** Passkeys and FIDO did close the phishing-resistant-login gap for mainstream web sign-in — that part is real and underrated. But authentication is a bundle, not a single problem: recovery, shared devices, legacy systems, and the long tail of non-browser contexts are nowhere near "about as good as we can reasonably ask." Windley needs the door to be closed so the essay can relocate the interesting work; the reality is the door is closed *for the case that mattered to him in 2005*. It's a fair provocation with a caveat he leaves implicit.

**The four sub-questions are the genuine contribution — and they expose why the field is stuck.** "On whose behalf" and "in which context" are precisely where RBAC/ABAC tooling is thinnest, and they're precisely where agents live. Windley's decomposition predicts the difficulty curve the security literature independently found: single-principal authorization is tractable, multiplayer (agent-for-multiple-humans) authorization is not — [[The Agent Access Model]] refuses to claim it, [[Beyond Zero — Enterprise Security for the AI Era]] defers it to a world model nobody has. Windley names the right sub-questions without pretending they're answered.

**The Cedar endorsement has a self-interest caveat, and survives it.** Windley worked on the AWS Identity team that shipped Cedar and Verified Permissions, so "the answer is the thing I helped build" is a fair suspicion. But Cedar is genuinely one of the few policy languages to ship with a formal semantic foundation rather than just a grammar — which matters more than most realize when the thing *writing* the policy is an agent, not a human. Cecchetti writing the foreword confirms the alignment; it doesn't settle the independence. The honest read is that Windley is arguing for a *category* (externalized, language-based policy) and using the best-built instance of it.

**The "bounded, accountable, revocable" triad is convergent, not original — and that's the point.** The essay lands on the same three properties as [[Golem Covenant]]'s spec for bounded, answerable, revocable agents, from a completely different direction (enterprise identity rather than agent-infrastructure safety). When a twenty-year identity veteran and a speculative v0.1 spec independently name the same three properties as the thing authorization must deliver, it's evidence the field is converging on what "delegation done right" means — even if nobody can yet build it end to end.

**The usability claim is the one worth taking seriously and the one with no evidence.** That externalized fine-grained authorization makes systems *more pleasant to use* inverts decades of security folklore (security is friction). If true, it's the most important claim in the essay — it's what would get enterprises to actually adopt the thing. But it's supported only by Windley's experience at AWS scale. The test will be whether *Authorization in Action* moves past assertion to a worked demonstration, because "safer *and* nicer to use" is either the field's future or its sales pitch.

---

*Sources: [[raw/authentication-is-largely-solved]], [[summary/authentication-is-largely-solved]]*
*Last updated: 2026-09-11*
