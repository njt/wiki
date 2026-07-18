# Personal Agents

The "my own AI assistant" movement has produced a remarkable range of approaches -- from a markdown file and a shell alias ([[life-system]]) to a $5 ESP32 microcontroller ([[MimiClaw]]) to full PostgreSQL + pgvector stacks ([[mira-OSS]]). What unites them is the bet that general-purpose AI assistants (ChatGPT, Claude.ai) are insufficient for personal automation. You need something that knows your context, runs on your infrastructure, connects to your services, and persists across sessions. The community around this -- the Vibez WhatsApp group, Jesse Vincent's Superpowers methodology, Dan Shapiro's tools -- is as important as any individual framework.

---

## The Landscape

### The Infrastructure Spectrum

**Zero infrastructure.** [[life-system]] is just markdown files and Claude Code. 10-year vision, annual goals, daily journal, decision records. The AI reads all of it and holds context. No database, no server, no proprietary format. The lightest-weight entry in the personal agent space.

**Single container.** [[PiClaw]] packages a coding agent into a self-hosted Docker container with web UI, persistent sessions, scheduled tasks, and authentication. Thinking about hosting as a first-class concern. [[clawdBot]] (now OpenClaw) targets "normal people" with one-line install and multi-platform messaging -- WhatsApp, Telegram, Slack, Discord, iMessage.

**Full stack.** [[mira-OSS]] runs PostgreSQL + pgvector + Valkey + Vault in Docker. First-person narrative memory, text-based LoRA for personality evolution, dynamic tool management. The most technically ambitious personal agent framework. [[Hermes]] (149k stars) is the batteries-included option: self-improving skills, 40+ tools, cron scheduler, MCP integration, every messaging platform.

**Hardware.** [[MimiClaw]] runs on a $5 ESP32 microcontroller -- pure C, Telegram interface, ReAct loop, persistent memory in NVS flash. 5.4k stars. The appeal is "AI on a chip" -- the logical extreme of running agents on your own hardware.

### Design Philosophies

**Continuity.** [[mira-OSS]]'s core bet: one conversation thread, forever. Memory decay, first-person framing, segment collapse. The agent accumulates personality and knowledge over time. Heavy infrastructure for a personal assistant, but genuine continuity across sessions.

**Self-improvement.** Both [[Hermes]] and [[clawdBot]] feature self-improving skill loops where the agent creates and refines skills from experience. [[Hermes]]'s Skills Hub (agentskills.io) adds a community sharing dimension. The question is whether self-improvement actually works or just accumulates drift.

**Knowledge-first.** [[Rowboat]] builds a persistent knowledge graph from email, meetings, and docs, then acts on it. Not retrieval but modeling -- explicit graph of people, companies, decisions, and topics. Obsidian-compatible vault for transparency. Folded because the business model wasn't there, but the approach is sound.

**Tiered autonomy.** [[Chief of Staff]] is the most operationally sophisticated: rule-based scanning every 30 minutes (zero LLM cost), LLM classification once daily, graduated autonomy with measurable criteria and revocable trust. A 90-day rolling window, not a permanent achievement. Built on a repurposed MacBook Pro for $100/month in LLM costs.

**Task completion.** [[Serf]] and [[Ralph]] take the opposite approach from conversational assistants: give it a task, it works until done. No human in the loop during execution. [[Kata]] provides the issue tracker that makes this structured: agent-friendly CLI with JSON output, human-facing TUI for oversight, SQLite backend.

### The Community

[[Awesome Vibez]] documents the Vibez WhatsApp community -- about 8 active GitHub members building tools around agents. The roster matters:

- **Jesse Vincent** (@obra): Superpowers workflow system, container sandboxing, episodic memory, external subagents
- **Dan Shapiro** (@danshapiro): [[Trycycle]], Kilroy (attractor dark factory), [[Fresh Eyes]]
- **Wes McKinney** (@wesm): agentsview, msgvault, [[Kata]], [[Radical Accountability]]
- **Harper Reed** (@harperreed): [[AI Agents with Human-Like Collaborative Tools]], [[Summarize Meetings Skill]]

The pattern: nearly everyone builds tools *around* agents rather than agents themselves. The harness is solved; the workflow around it is still being figured out. The community dimension matters because personal agents are still early enough that practitioners learn primarily from each other's experiments.

### Mobile and Accessibility

[[happy]] (20.6k stars) wraps Claude Code with mobile access, push notifications, and end-to-end encryption. [[Claude Code on the Go]] documents the supervisor-from-phone pattern: agents on a cloud VM, monitored from an iPhone. Both recognize that personal agents need to be accessible from wherever you are, not just your development workstation.

## Key Tensions

**Infrastructure weight vs. capability.** [[life-system]] (files + shell alias) vs. [[mira-OSS]] (PostgreSQL + pgvector + Valkey + Vault). The heavier the infrastructure, the more capable the agent -- but also the more you have to maintain. [[Chief of Staff]]'s repurposed MacBook Pro is the pragmatic middle ground.

**Privacy vs. capability.** Running on your own hardware means your data stays private. Calling cloud APIs means your conversations leave your machine. [[Gemma Gem]] and [[maclocal-api]] push toward local inference, but capability trails cloud models significantly. Most personal agents resolve this by running locally but calling cloud APIs for inference -- privacy for data, capability from the cloud.

**Self-improvement vs. stability.** [[Hermes]] and [[clawdBot]] feature skill learning loops. But how do you audit what the agent has learned? How do you prevent skill drift? [[mira-OSS]]'s text-based LoRA evolves behavioral directives every seven days from accumulated feedback, which is the most principled approach but also the most opaque.

**Messaging platform vs. terminal.** [[clawdBot]] meets users on WhatsApp and Telegram. [[Hermes]] connects to every platform. [[life-system]] and [[Serf]] are terminal-only. The messaging approach reaches non-technical users; the terminal approach gives technical users more control. No single interface serves both well.

**Community vs. solo operation.** [[Hermes]]'s Skills Hub and [[clawdBot]]'s community skill repository add sharing. But sharing skills between agents requires trust -- a skill from a stranger could contain malicious instructions. Nobody has solved skill vetting for personal agent communities.

## The Theoretical Ceiling

[[Guardian Angels|Gwern's Guardian Angels]] represents the limit case of what the personal agent movement is building toward: an LLM finetuned so precisely on a single person's corpus that it can substitute for them in most contexts, handling execution while the human focuses exclusively on "what is worth doing." The gap between current personal agents (which *serve* you) and Guardian Angels (which *are* you, as nearly as possible) is both technical and philosophical — most of the personal agent community would not endorse full substitution as a goal, even if it were achievable. But Gwern's anti-principles (no engagement optimization, no brand-safety sanding, >$1,000/month as a feature not a bug) are a useful stress test for any personal agent design.

## What's Missing

**Evaluating personal agents.** How do you know your personal agent is performing well? There are no evals for "did my assistant handle my email correctly?" or "did the scheduling make my day better?" [[Demystifying Evals for AI Agents]] covers coding agents; personal agent evaluation is entirely uncharted.

**Migration between frameworks.** If you invest in [[mira-OSS]] and want to switch to [[Hermes]], your accumulated memory, skills, and personality are lost. No interchange format exists for personal agent state.

**Multi-user personal agents.** [[Chief of Staff]] manages four email accounts. But most personal agent frameworks assume a single user. Family agents, shared assistants, and small-team agents need multi-user coordination that none of these tools provide.

**Cost transparency.** [[Chief of Staff]] discloses $100/month in LLM costs. Nobody else does. Running a personal agent has ongoing costs that are rarely discussed.

## Key Themes

#personal-agents #self-hosted #community #messaging #autonomy #infrastructure-spectrum
