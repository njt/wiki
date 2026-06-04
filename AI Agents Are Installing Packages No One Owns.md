# AI Agents Are Installing Packages No One Owns

AI coding agents are autonomously pulling dependencies that no human has vetted, no team owns, and — in some cases — no one even published. The accountability gap is structural: agents install packages, but organizations have no answer to "who approved this?" Worse, LLM-hallucinated package names are spreading through agent skill files into hundreds of repos, creating a new class of supply chain vulnerability where the attack vector is literally a plausible-sounding name an LLM invented.

---

## The Accountability Gap

Willem Delbare, CEO of Aikido Security, frames the problem in organizational terms:

> "There is no accountability"

The gap isn't just technical — it's that AI coding agents are being used by marketing, sales, and product teams who have no security training and no mental model of what a dependency even is. The attack surface has expanded from developers (who at least know what `npm install` does) to everyone with a Claude Code or Copilot subscription.

This connects directly to [[You Should Not Update Your Dependencies in 2026]] — Gambier's argument that every dependency update is an untrusted code contribution. But Delbare adds a dimension Gambier doesn't: the user installing the dependency may not even understand they're doing it. The agent just does it.

## The Hallucinated Package Problem

Charlie Eriksen (Aikido researcher, Jan 2026) documented a case study that makes the threat concrete:

An LLM hallucinated a package called `react-codeshift` — a plausible mashup of real packages `jscodeshift` (Facebook's codemod runner) and `react-codemod` (React team's transforms). The fake package appeared with realistic arguments like `react-codeshift/transforms/rename-unsafe-lifecycles.js`.

The traceable origin: commit `65e5cb0` in the `wshobson/agents` repo (Oct 17, 2025) dumped 47 LLM-generated "Agent Skills" without human review. Two skill files contained the hallucinated command.

By the time Eriksen investigated, **237+ GitHub repositories** referenced the non-existent package. Most were forks, but one user copied it into 30+ of their own repos. There was a Japanese translation. Some variants used `bunx` instead of `npx`.

Eriksen claimed the package name on npm before any attacker could. After claiming it, download telemetry showed **1-4 downloads per day** — proving real AI agents were executing the hallucinated instructions in the wild.

> "The supply chain just got a new link, made of LLM dreams."

## The Attack Surface Chain

The vulnerability has five links, each independently plausible:

1. **LLM hallucinates a plausible package name** — mashup of real tools, realistic API surface
2. **Skill file embeds it as an executable command** — looks like documentation, functions as a script
3. **Agent follows instructions literally** — "that's their job"
4. **Agent hits `y` on the npx prompt** — "The agent hits y. So would most people"
5. **Unclaimed npm names are first-come, first-served** — no registry gate prevents squatting

This is a qualitatively new kind of supply chain attack. Traditional typo-squatting exploits human typos. This exploits LLM hallucination — and the hallucination spreads *faster* than human error because skills are copied, forked, and translated without anyone reading the commands.

## Aikido's Product Response

Aikido ships three products aimed at this problem:

- **Aikido Endpoint** — pre-install inspection of packages, plugins, IDE/browser extensions. Covers Gemini, OpenAI, GitHub Copilot, xAI, MCP Servers, Claude Code, and skills.sh
- **Aikido Infinite** — continuous AI penetration testing
- **Aikido Intel** — dual LLM models scanning changelogs for undisclosed vulnerabilities

## Competitive Landscape

The AI agent security space is fragmenting into specializations:

| Player | Angle |
|--------|-------|
| Aikido | Endpoint inspection + AI pentesting |
| Socket | Package security scanning |
| Endor Labs (AURI) | AI agent dependency risk |
| Chainguard | Container/supply chain integrity |
| Snyk | Developer-first security |
| Arcjet | API security for AI apps |
| Mobb | Auto-remediation |

The fragmentation itself is a signal: no one has the full answer yet, and the problem spans endpoint, package registry, CI/CD, and runtime.

---

## Key Themes

#concept — **Agent supply chain accountability** is a people problem, not a technology problem. The agent doesn't know it shouldn't install things; the human doesn't know the agent installed something. Neither is "responsible" in any organizational sense.

#concept — **Hallucinated dependencies as attack vector.** Typo-squatting requires the attacker to predict a typo. Hallucination-squatting only requires monitoring LLM output for unclaimed plausible names. The search space is smaller and the names are more realistic.

#pattern — **The skill-copy propagation chain.** Skills function as executable documentation. When they spread via fork/copy/translate without review, hallucinated commands spread with them. This is the same dynamic as [[I Don't Want Your PRs Anymore]] — velocity outpaces review, and the artifact spreads before anyone notices.

#tool — **Aikido Endpoint / Infinite / Intel** — three-product suite: pre-install blocking, continuous pentesting, changelog scanning for undisclosed vulns.

#person — **Willem Delbare** — Aikido CEO, framing the problem as accountability rather than technology.

---

## Critical Analysis

**The hallucinated package finding is more important than the Aikido product pitch.** The `react-codeshift` case study is a genuinely novel attack surface: LLM hallucination as supply chain vulnerability. The 237-repo spread with zero human review is the kind of empirical finding that changes threat models. The Aikido product response is fine but secondary — the real contribution is documenting that this is already happening, at scale, with live agents executing hallucinated install commands.

**The article undersells the human dimension.** The framing is security-vendor-appropriate ("buy our product"), but the deeper story is organizational: teams beyond engineering are using AI agents with no security training, no dependency awareness, and no accountability structure. This isn't solvable with a scanner. It requires changing who can use what tools and what they're allowed to install — the [[Zero Trust for AI Agents]] "Least Agency" principle applied to humans, not just agents.

**The 237 repos number needs scrutiny.** Eriksen's blog post is transparent that most were forks, and one user accounted for 30+. The "spread" is real but the *independent adoption* number might be much smaller. That said, the 1-4 daily downloads after claiming the package is the more damning metric — that's actual agent execution, not just code sitting in repos.

**The competitive landscape slide is noise.** Listing seven competitors doesn't clarify anything — it just signals "this is a market now." The meaningful distinction is between endpoint-level inspection (Aikido, Socket), CI/CD-level gating ([[Supply Chain Security for Software Developers]]' 7-day rule, Mendral's AI reviewer), and registry-level integrity (Chainguard, Sigstore). These are complementary layers, not competitors.

**What's missing: the "just don't let agents install packages" answer.** The article assumes agent autonomy is desirable and the problem is securing it. But the simplest defense — don't give agents package-install permissions — goes unmentioned. [[Security and Sandboxing]] covers this: deny by default, whitelist by necessity. An agent that can't run `npx` or `pip install` can't install hallucinated packages. The counterargument (agents need to install things to be useful) is weaker than it sounds — most agent workflows don't require *arbitrary* package installation, just a known, reviewed set.

**The meta-problem: skills as executable code.** The hallucinated `react-codeshift` spread because skill files look like documentation but function like shell scripts. The format invites casual copying. This is the same class of problem as GitHub Actions pinning to tags instead of SHAs — the convenient thing is the dangerous thing, and the safe thing requires knowing the threat model exists. [[Guardrails and Feedback Loops]] applies: deterministic validation (don't execute skills without scanning for unregistered package names) beats hoping humans read what they copy.

---

*Sources: [[raw/aikido-ai-agents-security]], [Aikido blog — Agent Skills Spreading Hallucinated npx Commands](https://www.aikido.dev/blog/agent-skills-spreading-hallucinated-npx-commands)*
*Last updated: 2026-06-04*
