---
url: https://ai.statico.io/2026/07/01/team-wide-agentic-harness/
title: "Team-Wide Agentic Harness"
author: Ian Langworth
date_fetched: 2026-07-18
date_published: 2026-07-01
---

# Team-Wide Agentic Harness

Ian Langworth
2026·07·01
3 MIN

I've been gradually becoming a Harness Guy. It's similar to being a Tools Person, but specifically around AI agents: optimizing the environment around coding agents so they produce better, more consistent, more correct results. You know the type — someone who knows an unusual amount about crafting CLAUDE.md files, mapping MCP endpoints, and sizing their context window.

## Brown bags and checked-in skills

Our team loves a brown bag. Over the last few months, I've been running AI brown bags over lunch. Each session, we gather in a room and trade tips on how we've been using AI agents during the week. I'll usually demonstrate a skill that I use.

Today, for example, I demonstrated a Linear-management skill that reviews our queue, checks progress against the roadmap, organizes releases, and generates release notes that are tailored to specific customers.

Since skills are files, they're inherently shareable. I can say, "Here's the one I use. If you want to run it on your machine, put it here. Let's check it in."

## The parts that don't check in

We can check skills in to a project repo, but the conventions that make those skills function well — like the conventions you use for sandboxing — are harder to share.

The most important sandbox rule: scope all work into a single directory, giving the sandbox access only to that directory. Code can't access your dotfiles. It can't read your email. It can't install packages. It produces temporary files in a `tmp` directory that doesn't get checked in. Worktrees all go into a `worktrees` directory, never committed. There's a `plans` directory or a `notes` directory that's a loosely organized open bucket of agent output artifacts — easy to view in an app like Obsidian.

This is the kind of convention that lives uneasily in a document. It's best shared through use and observation: "Here's where I put things," and it works because your harness has more or less the same layout as mine.

## The harness

All of this — the conventions, the skills, the knowledge — should be checked in. I want to check in an entire top-level directory called the harness. Call it whatever you want.

I read The AI-Native Startup Handbook, which argues that AI-native organizations should share "a common, version-controlled repository of context, plugins, and skills across the entire organization." That mostly codified what I've been doing already.

My setup has multiple repos, research, notes, plans, skills. Much of it is for my eyes only. But I realized that a whole lot of it is shareable and that making it visible gives other people a starting point from which to shape their own setups. I should also have an `evergreen` docs directory, a place with descriptions of the company, the product, and procedures we do often. These are the atomic blocks of context you feed into your context window before you dispatch an agent to do something.

## Why check it in at all?

Because skills are code. A skill is a set of instructions given to an agent. Any code change should be reviewed. The harness is a codebase of its own, and skills, docs, and context all go through a PR, a diff, and another set of eyes before they affect everyone's agent behavior.

Honestly, I've been flying solo on this for a while. I'd love to hand this to the team and see what happens. It's time to find out if conventions that live comfortably in one person's head survive contact with everyone else's.

*Sketched by [a human](/about)*

Tags: agents, ai, automation, book-notes, knowledge-base, obsidian, sandboxing, skills, tooling
