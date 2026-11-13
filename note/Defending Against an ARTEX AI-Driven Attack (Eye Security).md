# Defending Against an ARTEX AI-Driven Attack (Eye Security)

Eye Security Labs' incident-response account of repelling an attack run by ARTEX, an autonomous LLM-driven pentesting framework: a swarm of cooperating agents that found secrets and used them within minutes, adapted to rate limiters and WAF rules in about three, and kept coming back through the victim's own deployment automation. The defenders won only by running their own agents — read-only, behind hard guardrails — against the logs.

---

## What happened

Initial access was a vulnerable custom PHP upload function — not an unpatched CVE, but custom code, exploited the same night as intake. From there the agents moved at a pace the authors repeatedly contrast with human attackers: environment variables exposed secrets, stolen API keys were in use within minutes, the cloud secret store fell, Neo-reGeorg was downloaded and modified within minutes to keep tunnel state on an internal cache server, an engineer's Git identity was abused to commit a malicious deployment, and self-hosted CI runners were re-registered every eight minutes all night.

The most quoted sequence: agents found API keys, scripted bulk scraping within fifteen minutes, defeated the rate limiter in about three minutes with a spoofed client-IP header, peaked at over 9,000 requests/minute, and when blocked went direct through a pool of 17,000+ residential proxies.

> "No human team can work through a rate limit, a block and a change of infrastructure in that amount of time, let alone in the middle of the night."

Their four-part post-mortem is worth taking whole:

- **The gap between finding and using has collapsed.** Stolen credential to first use: minutes. Every WAF rule buys about three minutes of adaptation time.
- **The agents are loud.** They name their own artifacts (`artx-pod`, `artx-probe-qa-1`), hammer endpoints no human would touch, and trip EDR detections. Where you have visibility, you see them immediately.
- **Breadth, not depth.** They try *every* path in parallel and keep *every* foothold. "Containment is no longer about finding *the* way in: it is now about finding *all* ways in."
- **Equality of arms.** "Defending at machine speed means defending with machines, and keeping people in charge of them."

The kicker: the attackers used the victim's own Git-backed deployment automation against them. "When a responder deleted the attacker's workloads, the deployment automation dutifully re-created them within a second; the automation works just as hard for the attacker." And the lull that tempted the team to go live was, they suspect, the attacker running out of tokens.

## The defenders' answer: agents behind hard guardrails

The IR team gave their own agents full investigative autonomy with read-only access, coordinated through a shared private Git repo — findings synced via pull requests, situation reports as commits — with a hook that blocked any command that could mutate state, read a secret, or print a token. Where access couldn't be scoped read-only, they had an agent build a wrapper enforcing it. Every EDR hunt got its own folder with hashes, so afterwards they could show exactly what was queried, when, and by whom.

The payoff was triage: agents walking process trees every 30 minutes and dismissing false "ransomware" and "sensor tampering" alerts within minutes — "exactly the kind of triage that eats a human analyst's night." The costs were coordination drift (agents reading stale documentation produced by others) and propagated guesses: one agent labelled a network address "probably guest Wi-Fi," the guess spread into five documents and a data request within a day, and only an agent working from live data corrected it. Their guardrail hook also once blocked an agent from editing a plain-text note merely because the text mentioned a cloud CLI's name — annoying, but "we'd rather err on the side of blocking."

> "An autonomous attacker is fast, but it turns out it is also predictable: it goes where the value is. That gives defenders a rare chance to get there first, but only if you make the hard decision of switching everything off."

## Key themes

#concept #pattern #tool

- **Machine speed as an adversarial property** — the "gap between finding something and using it" is the real threat metric, not novel exploits. ARTEX is powerful because of scale, parallelism, and shared findings, not because agents invent new attacks: "they reuse well-known tools and techniques, so the power lies in combining them faster and more broadly than a human team can."
- **Symmetric agentic defence with asymmetric guardrails** — the attackers' agents get full autonomy; the defenders' get read-only plus a blocking hook. Trust is enforced by the harness, not the prompt.
- **Git as the coordination substrate** — the same repo mechanism serves both sides: the attackers poison deployment automation through Git; the defenders sync agent findings through PRs. Coordination through artifacts beats coordination through chat.
- **AI-driven attacks are predictable in direction** — value-seeking behaviour makes the next target forecastable, which is a genuinely new defensive lever.

## Critical take

This is the most concrete public evidence yet for a claim the field has argued about abstractly: agentic offensive tooling operating against production infrastructure at a pace humans cannot match. Three things make it credible. First, the authors are IR practitioners who were professionally sceptical ("Less than a day later, battling the second wave of the attack, we had become true believers") — the conversion narrative is itself evidence. Second, the details are mundane rather than cinematic: header-spoofed rate-limit bypasses, reGeorg patches, user-agent reuse of a payment provider's exempted string. No zero-days; the power is orchestration over known techniques. Third, they publish IOCs, which is what real responders do.

Caveats worth holding. The vendor sells security services and has every incentive to dramatise — "we are bracing ourselves for the incoming stampede of incidents (sadly, we're not exaggerating)" is marketing cadence. The attackers' token-budget lull suggests the offensive operation was small and cost-limited, not a state-scale campaign; a human-run adversary with a bigger budget might have been quieter and more patient. And the "defenders' agents" story is told at a success arc — the notes-drift problem and the guardrail false-positive are admitted, but we never learn how much agent triage time was actually wasted on noise. Still, the structural lessons survive the spin: secrets sprawl, Git-connected deployment automation, and CI runner trust are the attack surface now, and containment must assume the attacker iterates at machine speed.

## Relation to the wiki

- [[AI Cybersecurity After Mythos — The Jagged Frontier]] — Fort argued the moat in AI cybersecurity is the scaffold and triage layer, not the model. This incident is the field confirmation from the other side of the line: ARTEX's power is orchestration over commodity techniques, and the defenders' edge came from their own harness, not a smarter model.
- [[Cybersecurity Is Proof of Work Now]] — Fort's "thousand adequate detectives" search-problem framing maps directly onto the attack: breadth over brilliance, every path tried in parallel, coverage beats cleverness — now demonstrated offensively rather than argued.
- [[The New Set of Problems for Engineers to Solve (Katelyn Lesse)]] — Lesse's warning that injection is lateral movement with real blast radius, and that agent diligence looks like malice, is exactly what this incident shows in production: agent-generated Git commits deploying attacker pods, and defenders unable to distinguish the two by inspection.
- [[Agent Swarm Model Economics]] — Cursor's report treats swarms as a generation-side architecture; ARTEX shows the same planner-worker economics weaponised, and the defenders' token-budget-lull observation is an adversary's cost curve made visible in an incident log.

---
*Sources: [[raw/how-we-defended-against-an-ai-driven-attack]], [[summary/how-we-defended-against-an-ai-driven-attack]]*
*Last updated: 2026-11-13*
