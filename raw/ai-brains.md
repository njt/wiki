---
url: https://consulting.secondcoffee.ai/ai-brains/
date_fetched: 2026-10-24
---

AI brains landscape · audited October 2026

# AI brain providers, compared.

A working map of AI brain tools sorted by what you need them to do: keep a personal brain, remember people, understand company knowledge, answer from documents, run workflows, or give employees an approved assistant where they already work.

Start with the job

## What do you need AI to do?

Use cases first, plumbing second. Pick a need state to understand the category, then use the full provider comparison below for the details.

| Provider | Use-case category | MCP | Integrations | Pricing | Details | 
|---|

Audit note — every entry re-verified Oct. 18, 2026 against vendor sites, docs, repos, and package registries, with five app-backend entries added Oct. 19: 251 source URLs. Of the links probed, all but three answered (one MCP endpoint correctly requires auth; one community site was transiently down; one cited licence path proved a licence file did not exist, which is now the recorded finding). Of 29 GitHub-linked projects, none is archived and the oldest push is 102 days (Kombo Company Brain). 34 of 53 entries ship a first-party MCP server — each server claim was independently re-checked against npm, PyPI, and the vendor's own docs, and where a package turned out to belong to an unrelated project it was rejected. Open **any row** to see that entry's sources. This is a public snapshot, not procurement advice: recheck pricing, security, and maturity before buying.

**Thesis:**the useful split is shared company memory (“what does the organization know?”) versus agent-specific adaptive memory (“what has this AI worker learned from doing the job?”). Serious systems eventually need both, plus governance.

How to read this landscape

## Use cases first, labels second.

This page is a public snapshot, not procurement advice. The most useful question is not “which category is canonical?” but “what job do you need the brain to do?” The table is grouped in four tiers — products you buy, memory infrastructure, build substrates, and personal or emerging tools — so you can tell a thing you deploy apart from a thing you assemble. Use the comparison as a starting map, then recheck pricing, security, and product maturity before making a buying decision.

**Start with the job**Personal memory, company knowledge, document research, agent workflow, or approved employee assistant.

**Separate product from plumbing**Some entries are buyer-facing apps; others are developer infrastructure that a brain may be built on. The tier grouping keeps those apart.

**Check MCP before you commit**A server means agents can connect to it. A client means it can reach other tools. Most vendors now claim MCP somewhere; the column distinguishes a shipped first-party server from a community wrapper, a roadmap promise, and nothing at all.

**Check the memory model**User preferences, agent experience, document context, temporal events, and company knowledge behave differently.

**Look for transactive memory**The strongest company brains help people and agents know who knows what, where the authoritative source lives, and which context matters for the task.

**Verify governance early**Permissions, provenance, audit logs, retention, deletion, and memory-poisoning defenses matter before rollout.

**Treat market signals as directional**GitHub stars, public pricing, funding, and customer claims change quickly and should be rechecked.

Current thesis

## The winner probably needs two kinds of memory.

**Shared company memory** answers: what does the organization know across docs, customers, workflows, decisions, tools, and people?

**Transactive memory** answers: who or what knows this, where is the authoritative version, and how should the right context be retrieved before work starts?

**Agent-specific adaptive memory** answers: what has this AI worker learned from doing its job for this customer, team, or process?

The serious product will need both, plus governance. Otherwise you either get a smart search box that never learns from work, or a learning agent with no trustworthy company context.
