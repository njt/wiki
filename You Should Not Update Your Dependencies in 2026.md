# You Should Not Update Your Dependencies in 2026

Olivier Gambier (Mendral, ex-Docker distribution team lead) argues that every dependency update should be treated like a PR from an unknown external contributor. Dependabot auto-merge — once the right call — is now an attack vector: registries are compromised, maintainers are burned out, and AI coding agents are flooding the supply chain with unreviewed code. The fix isn't going back to manual review (impossible at scale) but programmatic AI-assisted review inside CI that mechanically examines every dep diff, checks for typo-squatting, evaluates CVE reachability, and flags behavioral drift. The sharpest reframe: *dependency updates are untrusted contributions.* Adopt that mental model and you don't need Gambier's product — just stop auto-merging Dependabot PRs and start treating every lockfile change as a code review event.

---

## Key Quotes

> "Open-source maintainers are not just free labor, they are also overworked, under-equipped, wildly understaffed"

The economics of open source have always been broken, but Gambier connects this directly to supply chain security: burned-out maintainers with no funding are the upstream attack surface. This isn't a labor-rights argument dressed as security advice — it's a genuine structural diagnosis. When the people who control your dependencies can't afford to secure them, you inherit that risk.

> "Blind installation and blind updating dependencies continuously (a-la dependabot)... became the number one vector of distribution for highly publicized supply chain compromises in the past 12 months."

This is the claim that makes the piece controversial. Dependabot was sold as the responsible thing — "always be up to date" — and Gambier is calling it the primary attack vector. The parallel to [[I Don't Want Your PRs Anymore]] is striking: tools designed to increase contribution velocity (Dependabot for deps, LLMs for PRs) have become threat surfaces because they outpace review capacity.

> "Dependency updates must be considered as untrusted code contributions."

The single best line in the piece. It's a mental-model shift, not a tool recommendation. If you internalize this, everything else follows: you review dep changes like code changes, you gate them on the same CI checks, and you never merge them on green build alone.

> "Going back is not a strategy."

Gambier dismisses the reactionary impulse — private forks, multi-month cooldowns, registry proxying — as the modern equivalent of "the sysadmin clutching Apache 1.3 config and going down with the righteous ship." This is where his argument converges with [[Probabilistic Engineering and the 24-7 Employee]]: the volume problem is real and won't be solved by slowing down. The only viable response is automation that matches the threat's speed.

> "Humans can no longer be in charge of the modern software supply chain security."

Provocative but defensible. The volume of dependency updates, the sophistication of supply chain attacks, and the asymmetry (attacker needs one success, defender needs zero failures) make human-only review structurally inadequate. The question isn't whether to automate — it's whether the automation is programmatic verification (Gambier's argument) or blind trust (the current default).

> "AI cannot outperform the best humans" — but it excels at "the mechanical, repetitive, volume-bound work" of reading every diff, checking changelogs against actual changes, and cross-referencing against known compromise patterns.

A useful honesty. Gambier isn't selling AI as magic. He's making the case that supply chain review is exactly the kind of work AI is good at: pattern matching at scale, across every update, on every repo, without fatigue or boredom. The [[Harness Engineering]] parallel: computational verification beats inferential review when the problem is volume, not judgment.

---

## Key Themes

#concept — **Dependency updates as untrusted contributions.** The central reframe. Every lockfile change is a PR from a stranger. Treat it accordingly.

#concept — **Supply chain compromise vs. supply chain vulnerability.** A distinction the industry has been sloppy about. Vulnerabilities are passive (bugs exist). Compromises are active (attackers ship weaponized code through the distribution channel). Different threat models, different defenses.

#pattern — **Programmatic CI review of dependency changes.** The mechanical-verification approach: typo-squatting checks, age gating (<7 days raises scrutiny, <72 hours maxes it out), behavioral sandbox inspection, CVE reachability analysis, posture auditing. Each check is automatable. Combined, they form a [[Supply Chain Security for Software Developers]]-style layered defense.

#tool — **Mendral** (Gambier's company): CI-native agent that treats dep updates as untrusted contributions, runs a suite of programmatic checks, and comments/approves/blocks on PRs.

#person — **Olivier Gambier**, ex-Docker distribution team lead. Deep credibility on supply chain from years inside the container/supply chain intersection. Now building Mendral.

---

## Critical Analysis

**The strongest insight isn't about AI — it's the reframe.** *Dependency updates are untrusted contributions.* Adopt that mental model and you don't need Gambier's product. Just stop auto-merging Dependabot PRs and start treating every lockfile change as a code review event. The AI reviewer is optimization of a process most teams haven't even started doing manually. This is the [[Feedback Loop is All You Need]] pattern applied to dependencies: the gate matters more than how you enforce it.

**The 7-day age gate is already canon.** The [[Supply Chain Security for Software Developers]] gist made this case in the aftermath of TeamPCP, and Gambier integrates it into a broader CI pipeline. What's new here is the scrutiny-maxing at <72 hours and the behavioral sandbox inspection — going beyond age-gating into active compromise detection. The complementary relationship between these two sources is strong: one gives you the config, the other gives you the pipeline.

**What the article glosses over matters.** Three gaps: (1) The patient adversary who waits out the 7-day cooldown — Gambier's behavioral sandbox partially addresses this but he doesn't model the attacker who ships clean at t=0 and activates at t=14 days via a time-bombed conditional. (2) CI latency cost — sandboxed behavioral analysis per dependency update isn't free, and teams with 50+ weekly Dependabot PRs will feel the queue. (3) Whether AI can really spot a one-line behavioral change buried in a 2,000-line diff — this is the same trust problem as AI code review generally, and Gambier's "it can't outperform the best humans" admission is doing a lot of work there.

**The "going back is not a strategy" argument is right but undersells the cooldown approach.** The piece lumps registry proxying and multi-month cooldowns in with "reactionary" responses, but the 7-day rule is itself a cooldown — just a programmatic one. The boundary between "smart automation" (Gambier's position) and "sensible friction" (the [[Supply Chain Security for Software Developers]] position) is blurrier than the piece admits. Both are saying: *don't consume dependencies the moment they're published.* One does it with an age gate, the other with an AI reviewer. They're complementary, not opposed.

**The AI skeptic's rejoinder: who reviews the reviewer?** If Mendral's agent says "LGTM, no compromise markers found," the overworked human reviewer will click merge with the same blind trust they currently give Dependabot's green checkmark. The failure mode shifts from "I didn't look at the dep update" to "the AI said it was fine." This is the [[Compound Engineering]] problem — you've added a system, but you haven't added trustworthiness unless the system's verdicts are themselves verifiable. Gambier doesn't address this, and it's the hardest problem in the space.

**The most honest line is buried: "AI cannot outperform the best humans."** This constrains the ambition appropriately. The goal isn't a superhuman supply chain auditor. It's replacing the *current* baseline — which for most teams is zero review — with something systematically better than nothing. That's a low bar, and hitting it reliably is genuinely valuable. The question is whether "better than nothing" creates a false sense of security that's worse than honest neglect.

---

*Sources: [[raw/you-should-not-update-dependencies]]*
*Last updated: 2026-05-31*
