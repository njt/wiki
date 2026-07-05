# Agent Orchestration for the Timid

Mark Ferree's skeptical tour through five AI agent orchestration tools — a heavy Claude Code user's conclusion that custom skills and slash commands beat all of them. After a bad experience with Gas Town, Ferree tests Vibe Kanban, Conductor, Claude Squad, Claude-Flow, and Taskmaster, finding each either too janky, too commercial, too close to his own tmux workflow, or too much overhead. His final position: lean harder into Claude-native orchestration features rather than bolting on an external orchestrator. The piece is less a tool review than a practitioner's argument for simplicity — orchestration overhead should earn its keep, and for Ferree, none of these tools clear that bar.

---

## Key Quotes

> "I'm getting plenty of value out of ai-centric coding today without letting an orchestrator take over."

This is the thesis and it lands with the weight of someone who's actually tried. Ferree isn't an orchestration skeptic on principle — he used and liked Agor, and he's willing to test new tools. His conclusion isn't ideological; it's empirical. The orchestrators he tested didn't add enough value to justify the learning curve, the context-switching, or the loss of control.

> "Gas Town was incredibly complex and token wasteful."

Ferree's specific beef with Gas Town: it abused model context for conflict resolution when simpler planning would have sufficed. This converges with Hartcher's critique in [[Gas Town After 10,000 Hours of Claude Code]] that Gas Town's delegation model is worse than pair-programming agency, and with Appleton's observation in [[Gas Town's Agent Patterns]] that vibe design produces incoherent architecture at scale. Three independent practitioners, same conclusion: Gas Town over-engineered the wrong problem.

> "MCP should probably be table stakes for any ai-orchestration tool."

Ferree says this while reviewing Taskmaster, but the point generalizes. In a landscape where [[MCP Is Dead; Long Live MCP]] debates whether MCP is even necessary locally, Ferree takes the pragmatic position: if you're building an orchestration tool in 2026 and it doesn't speak MCP, you've already lost. This aligns with [[Building Agents for Production Systems with MCP]] and the broader ecosystem move toward MCP as the standard integration layer.

> "Claude Squad is a friendly Tmux wrapper around standard Claude sessions."

Ferree's dismissal of Claude Squad is telling: as a power tmux user, he already has this workflow. The tool doesn't add value for him because it just repackages what he already does manually. This is the orchestration paradox: the more sophisticated your existing setup, the harder it is for an orchestrator to justify itself. [[Claude Code is a Beast — Tips from 6 Months of Hardcore Use]] describes a similarly sophisticated custom setup that no off-the-shelf orchestrator could replace.

> "Claude-Flow felt like it was in the 'vibe coded fever dream' category alongside Gastown."

Brutal but specific. Ferree's criticism isn't snobbery — he describes concrete failures: first task had no ID, subsequent commands broken, multiple install methods that felt unprofessional. This is a recurring theme in agent tooling: the gap between "works on the author's machine" and "works for strangers" is where most tools die. See [[What Ralph Wiggum Loops Are Missing]] for the structural problems that cause this failure mode.

## Key Themes

**#concept Orchestration skepticism**: Ferree represents a practitioner archetype worth naming: the "orchestration skeptic" who's tried the tools and concluded that custom skills + slash commands beat all of them. This isn't Luddism — it's the position that orchestration overhead has to earn its keep, and for individual practitioners, it usually doesn't.

**#tool Vibe Kanban**: Described as "less buggy but less capable Agor" and "feels like Jira." Already covered in the wiki at [[vibe-kanban]], which notes it's sunsetting. Ferree's review adds texture: he'd consider it as an Agor replacement, suggesting Agor occupied a genuine niche that nothing has filled since.

**#pattern Claude-native orchestration**: Ferree's alternative to external orchestrators: custom skills, slash commands, and Claude's built-in sub-agent features. This converges with [[How Intercom Uses Claude Code]] (13 plugins, 100+ skills) and [[Agent Coding Workflow]]'s finding that practitioners evolve toward bespoke setups. The common thread: external orchestrators solve a coordination problem that, for solo or small-team practitioners, doesn't exist yet.

**#person Mark Ferree**: Heavy Claude user, prior Agor user, tmux power user. Represents the practitioner archetype that orchestration tools need to win over — and currently aren't.

## Critical Analysis

Ferree's review is useful precisely because he's not an orchestration evangelist. He's a practitioner asking "does this help me?" and answering honestly. The methodology isn't systematic — he's trying tools casually, bouncing off rough edges, making gut calls — but that's actually more informative than a structured evaluation would be. Most developers approach tools the same way: try, bounce, maybe revisit. Ferree's bounce rate (4 out of 5 dismissed, 1 kept on the maybe pile) is data.

The piece's limitation is that it's a single-practitioner snapshot. Ferree is a solo developer with a mature tmux workflow and deep Claude Code customizations. Tools that don't fit that profile might still be valuable for teams, for less experienced users, or for different AI coding platforms. His dismissal of Conductor.build on the basis of a waitlist form tells you nothing about Conductor and everything about Ferree's impatience with friction — which may be a feature, not a bug, for a review about reducing friction.

The most interesting thing Ferree doesn't say: he never defines what "orchestration" means for him. He's testing tools that range from kanban boards (Vibe Kanban) to terminal multiplexers (Claude Squad) to CI/CD-like pipelines (Claude-Flow). These solve fundamentally different problems. Conflating them under "orchestration" is like reviewing a hammer, a saw, and a drill under "woodworking tools" and concluding none of them help you hang a picture. The right question isn't "which orchestration tool?" but "what coordination problem do I actually have?" — and Ferree's answer seems to be "none that custom skills don't already solve."

Still, the convergence with other practitioners in this wiki is striking. Ferree, Hartcher ([[Gas Town After 10,000 Hours of Claude Code]]), and Willison ([[Designing Agentic Loops]]) all land in the same place: invest in your harness, not in someone else's orchestrator. Ferree adds the concrete counterpoint: he actually tried the orchestrators before rejecting them. That makes his conclusion harder to dismiss than pure skepticism would be.

The [[Agent Orchestration]] synthesis page frames the same tension — external orchestrators vs. built-in coordination primitives — and Ferree's review is a data point for the "built-in wins for individuals" column. For teams and enterprises, the calculus may differ, but for the solo practitioner that Ferree represents, the state of orchestration tools in early 2026 looks like a solution in search of a problem.

---
*Sources: [[summary/agent-orchestration-for-the-timid]]*
*Last updated: 2026-05-22*
