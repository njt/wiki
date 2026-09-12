---
title: "Google Workspace CLI"
url: https://github.com/googleworkspace/cli
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# Google Workspace CLI (gws)

## Overview
One CLI for all of Google Workspace -- built for humans and AI agents. Drive, Gmail, Calendar, Sheets, Docs, Chat, and Admin APIs all accessible through a single interface.

## Key Technical Distinction
Reads Google's own Discovery Service at runtime and builds its entire command surface dynamically. New Google API endpoints are automatically incorporated without tool updates.

## Supported Services
Google Drive, Gmail, Calendar, Sheets, Docs, Chat, Admin, Apps Script, Tasks, Workspace Events, Model Armor.

## Authentication
- Interactive OAuth (local desktop with encrypted storage)
- Manual OAuth setup via Google Cloud Console
- Service account credentials
- Pre-obtained access tokens
- CI/headless environments

## Helper Commands
Hand-crafted helpers prefixed with `+`:
- Gmail: +send, +reply, +reply-all, +forward, +triage, +watch
- Sheets: +append, +read
- Docs: +write
- Chat: +send
- Drive: +upload
- Calendar: +insert, +agenda (timezone-aware)
- Workflows: +standup-report, +meeting-prep, +weekly-digest, +email-to-task

## AI Agent Capabilities
Ships 100+ Agent Skills for every supported API, plus higher-level helpers and 50 curated recipes. Written in Rust (98.8%). 26.2k stars. Apache-2.0. Not an officially supported Google product.
