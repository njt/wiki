---
title: "life-system"
url: https://github.com/davidhariri/life-system
date_fetched: 2026-05-14
section: "Personal Agents"
topics:
  - agent-coding-workflow
  - personal-agents
---

# Life System Starter Kit

## Main Purpose
A personal life operating system built on plain-text markdown files, powered by Claude Code as an AI thinking partner. Combines inspiration from John Carmack's .plan files and Benjamin Franklin's systematic self-improvement approach.

## Core Philosophy
The system works by having Claude read your life plan, annual goals, and values before each session, then holding you accountable when daily actions drift from stated priorities. "The files are the source of truth. Claude is the accountability partner who never forgets what you wrote."

## Key Components

**Planning & Reflection:**
- 10-year vision and life chapters
- Annual goals with yearly "bets" and anti-goals
- Daily journaling using Franklin's morning/evening questions
- Decision records for important choices

**Operational Files:**
- Inbox for task/idea capture
- Values and habits documentation
- People notes for contacts and research subjects
- Research directories for deep dives on topics

## Daily Workflow

**Morning:** Review yesterday's journal, create today's entry, answer "What good shall I do this day?", verify priorities align with annual goals.

**During day:** Auto-logging with timestamps, inbox capture, decision documentation, people research, topic deep-dives.

**Evening:** Reflect on accomplishments against plans using Franklin's closing question.

## Technical Features

- Wiki-link convention (`[[filename]]`) for cross-referencing between files
- Optional QMD integration for semantic search across accumulated notes
- Git version control for tracking plan evolution
- Simple shell alias (`jrn`) for journal creation and template management

## Setup Requirements

Installation involves copying configuration files to `~/.claude/`, setting up a morning skill, installing a journal script, organizing life directory structure, and optionally initializing git for history tracking.
