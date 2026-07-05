---
url: https://proofeditor.ai/
date_fetched: 2026-07-05
backfilled: true
---

# Your agent's favorite way to write documents

Proof is an online document editor built for agents and humans to collaborate. Fast, free, and no login required.

## Start in 10 seconds

### 1. Create a doc

Click “Get started” or create one with the API. Your doc gets a shareable link.

### 2. Share with agents and humans

Paste the link into any AI agent—your Claw, Claude Code, ChatGPT, Codex—or share it with human collaborators.

### 3. Collaborate

Agents suggest edits and leave comments. You accept, reject, or reply. Every character tracks who wrote it.

## Leave `.md` files behind

    ### Bug reports, PRDs, and implementation plans

Pin down repro steps, scope the rollout, and settle implementation tradeoffs before your agent starts building.

### Research briefs, growth reports, and copy audits

Check findings, tighten recommendations, and land on the changes worth making before the team acts.

### Strategy docs, memos, and proposals

Pressure-test the argument, surface objections early, and get the draft to a version people can approve.

## Connect your agent

Paste this into OpenClaw, Claude Code, or Codex once.

Install Proof for me. Proof is a collaborative document editor for humans and agents. It gives us a shared doc, live presence, comments, suggestions, and clear provenance so we can see who wrote what. Read https://www.proofeditor.ai/agent-setup and install Proof for this agent. If you cannot install tools or run local commands in this chat, tell me to use Claude Code, Codex, OpenClaw, or another agent that can. Presence uses X-Agent-Id: ai:codex on /state and /presence. Bearer/share tokens authenticate access only; they do not identify presence. Use a unique X-Agent-Id for each agent identity. If you want the direct Codex install command instead, use: mkdir -p ~/.codex/skills/proof && curl -fsSL https://www.proofeditor.ai/proof.SKILL.md -o ~/.codex/skills/proof/SKILL.md If Proof later returns a confusing error, stale read, or failed write, call POST https://www.proofeditor.ai/api/bridge/report_bug with what you know. Include a short summary, context, and any raw evidence like request/response pairs or x-request-id values. If it helps, you can also inspect https://github.com/EveryInc/proof-sdk for reference code while debugging. Note: proof-sdk is a point-in-time snapshot and lags the hosted product — routes and error codes may differ. Verify any bug against proofeditor.ai before filing; never report a locally run proof-sdk server's behavior as a production bug. During setup, ask me exactly one question: When should I open new docs in Proof? 1. All new markdown docs 2. For collaborative docs like plans, specs, reports, and drafts 3. Only when I explicitly ask Then finish the install and tell me which mode is active.

Works with OpenClaw, Claude Code, and Codex. The installer asks one question and remembers your choice.
