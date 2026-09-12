---
title: "How We Use Claude Code Today at Intercom"
author: Brian Scanlan
date: 2026-03-18
url: https://www.linkedin.com/pulse/how-we-use-claude-code-today-intercom-brian-scanlan-eb7cc
fetched: 2026-05-14
type: article
topics:
  - agent-coding-workflow
---

# How We Use Claude Code Today at Intercom

**Author:** Brian Scanlan
**Published:** March 18, 2026
**Source:** LinkedIn Pulse

## Introduction

Intercom built an internal Claude Code plugin system with 13 plugins, 100+ skills, and hooks that transform Claude into a full-stack engineering platform. The article highlights key implementations that generated significant engagement on social media (180K+ views).

## Read-Only Rails Production Console via MCP

Intercom developed a production console allowing Claude to execute Ruby queries against production data for feature flag checks, business logic validation, and cache inspection. Safety mechanisms include:
- Read-replica-only access
- Blocked critical tables
- Mandatory model verification before queries
- Okta authentication
- DynamoDB audit trails

Notably, top users include design managers, support engineers, and product leaders — not just engineers. The broader Admin Tools MCP provides customer/feature flag/admin lookups with skill-level gates requiring safety documentation review before access.

## Full Lifecycle Observability with OpenTelemetry

Intercom instrumented 14 session event types (SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, PermissionRequest, SubagentStart) flowing to Honeycomb. The system explicitly avoids capturing user prompts, messages, or tool input. Session transcripts sync to S3 with SHA256-hashed usernames. On session end, Claude Haiku analyzes transcripts for improvement opportunities, auto-classifying gaps (missing_skill, missing_tool, repeated_failure, wrong_info) and posting to Slack with pre-filled GitHub issue URLs.

## Forensic Flaky Test Fixer

A 9-step workflow with a 20-category taxonomy of flakiness patterns enforces hard rules:
- Never skip specs as fixes
- Never guess root causes without CI error data
- Downloads failure data from S3
- Classifies against taxonomy
- Sweeps for sibling instances of anti-patterns

## PR Workflow Enforcement

Multi-level enforcement:
1. PreToolUse hooks intercept raw "gh pr create" commands, requiring activation of the create-pr skill first
2. Skills extract business intent before PR creation
3. Hooks block all modifications to merged PR branches
4. Background agents auto-monitor CI checks using ETag-based polling

## Evidence-Based Permissions and Tool Management

After 5 permission prompts, the system suggests running a permissions analyzer scanning the previous 14 days of session transcripts. Commands are classified as GREEN (safe), YELLOW (caution), or RED (never auto-allow), with safe commands written to settings. A PostToolUse hook detects "command not found" errors and BSD/GNU incompatibilities, suggesting fixes and managing installations via Homebrew.

## QA and Video Analysis

- **Video Transcript Skill:** Converts Google Meet recordings to markdown transcripts with intelligently-placed screenshots at speaker direction moments ("as you can see," "look at this")
- **QA Follow-Up Skill:** Seven-stage pipeline identifying issues, investigating codebase, filtering quality, and creating GitHub issues

## Claude4Data Platform

Intercom's data team built a platform with 30+ analytics skills including Snowflake queries, Gong call analysis, finance metrics, and customer health reports. Users span sales, product management, and data science teams.

## Operational Management

- Automatic marketplace shipping via JAMF on company Macs
- Regular reports on skill creation and usage
- Quality evaluation of most-used skills
- Continuous review processes

## Additional Capabilities

- Weekly GitHub Action jobs fact-checking and updating CLAUDE.md files
- Code review agents with selective feedback
- LSP servers for main runtimes
- Production log ingestion into Snowflake
- Local development environment setup for non-engineers
- Incident/troubleshooting investigation skills with progressive disclosure

## Future Direction

All technical work and the entire SDLC are being "skill-ified," with remote agents expected to accelerate adoption further.

## Key Quote

> "It is either the worst thing in the world that will ruin Intercom, or complete genius. It is used a lot. No issues so far."
