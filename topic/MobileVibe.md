# MobileVibe

MobileVibe is a mobile app that turns your phone into the remote control for AI coding agents running on your own desktop. Rather than running agents in the cloud or on a VM (the [[Claude Code on the Go]] approach), it runs everything on your real machine — your CPU, GPU, filesystem, VS Code, extensions, and config — with an end-to-end encrypted phone-to-machine link. Think of it as "your desktop is the server, your phone is the terminal."

---

## Key Quotes

> "Your machine does the work. You stay in control from anywhere."

The inversion of cloud-everything: computation stays local, control goes mobile.

> "No sync. No fake environment. No limitations."

Positioned directly against cloud IDEs (GitHub Codespaces, Gitpod) and remote desktop. The argument is that your real environment is irreplaceable — extensions, config, local services, GPU access, all the accumulated context that makes a dev environment *yours*.

> "The MobileVibe iOS app was built this way."

Dogfooding claim: they shipped their own iOS app using MobileVibe to control agents, which in turn used GitHub Actions + TestFlight. This is also one of their use cases — "Ship iOS apps without a Mac."

> "Ship from bed."

The bluntest expression of the mobile-first development fantasy. Not just monitoring — actually shipping. Whether this is aspirational or real depends on how much the "smart auto-approve" rules can handle without human intervention.

---

## Key Themes

#tool #mobile-development #coding-agents #claude-code #local-first #encryption

MobileVibe sits at the intersection of several threads in this wiki:

- **Local execution, mobile control** — the opposite architecture from [[Claude Code on the Go]] (cloud VM) and a different tradeoff from [[happy]] (which syncs through a backend). Your machine must be on and connected; the upside is zero cloud cost and full access to local hardware.

- **Agent supervision, not coding** — Like [[Claude Code on the Go]], the real use case isn't writing code on a phone keyboard. It's monitoring long-running agent tasks, approving decisions, and getting push notifications when the agent needs input. This maps to the "supervisory engineering" role identified in [[ThoughtWorks Future of Software Engineering Retreat]].

- **Push notifications as infrastructure** — The notification loop (kick off task, pocket phone, get pinged when agent needs you) is the same insight from [[Claude Code on the Go]] and [[Agent of Empires]]. Without it, mobile agent control means constant polling. With it, agent runs become background jobs.

- **Smart auto-approve** — Configurable rules that let the agent proceed without human approval for certain actions. This is a [[Compound Engineering]] pattern: rather than manually approving every tool call, you build a system (rules) that handles the routine cases and escalates only the exceptions.

- **MCP server integration** — Claims support for Playwright, Unity, Blender, and Firecrock via MCP. This connects to [[Building Agents for Production Systems with MCP]] and suggests MobileVibe sees itself as more than a Claude Code wrapper — it's angling to be a general agent control plane.

---

## Critical Analysis

MobileVibe is making a bet that "your real machine" matters more than "available anywhere." [[Claude Code on the Go]] runs on a cloud VM you can spin up and tear down; MobileVibe requires your desktop to be on and connected. The cloud VM approach wins on availability (your desktop isn't always on) and isolation (worktrees, disposable environments). MobileVibe wins on fidelity (your actual setup, your GPU, your local services) and cost (no VM billing).

The question is whether fidelity matters enough. If your agent is editing code and running tests, does it need your *actual* GPU, or just *a* GPU? For most development tasks, probably not. For the use cases MobileVibe highlights — video editing, system tuning, GPU-heavy workloads — the answer is yes. But those are niche compared to "fix this bug while I'm at a coffee shop."

The "Ship from bed" rhetoric is great marketing but oversells the product. You can't *really* ship from bed unless your auto-approve rules are so comprehensive the agent never needs you. And if that's true, you're in [[Ralph]] territory — fully autonomous agents — at which point why do you need the phone app at all?

The security model is the strongest differentiator. "Runs on your machine. Nothing in the cloud." is the right answer for anyone who can't or won't stream their codebase through a third-party service. [[happy]] requires trusting their backend's encryption; MobileVibe doesn't have a backend to trust. For corporate environments, this is a meaningful advantage.

The eat-your-own-dogfood story (built their iOS app with their own tool) is either impressive or circular, depending on how much human intervention was involved. If the agents really did the bulk of the work, it's a strong proof point. If humans wrote most of the code and agents handled minor tasks, it's marketing.

The use cases are thoughtfully chosen — they're not the generic "build a todo app" but specific workflows where mobile supervision makes genuine sense: incident response at 2am, project triage during meetings, system diagnostics when you're away from your desk. These feel real rather than invented.

Missing from the page: any mention of what happens when the connection drops mid-task, how conflicts between phone and desktop state are resolved, whether multiple phones can connect to one machine, and what the "free forever tier" actually limits.

---

*Sources: [[summary/mobilevibe]]*
*Last updated: 2026-05-15*
