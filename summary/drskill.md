---
url: https://github.com/dbreunig/drskill
title: "drskill"
author: dbreunig (David Breunig)
date_fetched: 2026-07-25
date_published: 2026-07-25
---

`drskill` is a Python CLI that acts as `brew doctor` for AI coding agent skill
loadouts. It scans all coding agents detected on a machine or in a repo,
resolves each agent's effective skill set and MCP server configurations, and
checks for problems: shadowing, duplicates, description collisions, injection
surfaces, broken links, token budget violations, MCP misconfigurations, and
tool poisoning. Every finding ends in a fix command or an ack command. The tool
is strictly read-only — it never installs, edits, or deletes skills.

The scan runs in three tiers. Static (always runs) reads config files and skill
directories with no network or LLM calls. MCP connect (`--mcp-connect`, opt-in)
handshakes with configured MCP servers, enumerates tools, and writes snapshots.
Deep mode (`--deep`, opt-in) sends skill-pair descriptions flagged as
overlapping to an LLM (via dspy/LiteLLM) for judgment, budgeted to 25 calls by
default.

Key design points include: a fingerprint system (SHA-256 of check + content
hashes) so acks survive only while the exact situation persists; a committed
verdict cache so one person runs the LLM judgments and the whole team gets them;
a self-calibrating lockfile verifier that won't cry wolf when upstream hashing
differs; and scope-aware ack routing that separates machine-level from
project-level concerns. Seven static injection checks flag suspicious patterns
without storing secrets or making LLM calls — the tool quotes evidence and
defers intent judgments to the user.

The tool defines 70+ coding agents in `data/harnesses.toml`, each with search
paths, precedence rules, and verified/unverified facets. Uncertainty about
harness behaviour is tracked precisely: only findings that actually depend on an
unverified facet inherit a `?` marker.

Trade-offs include heuristic description checks tuned against public skill
corpora (will miss paraphrased conflicts), approximate token counting via
tiktoken's `o200k_base`, and a trust model where both the verdict cache and ack
ledger are unsigned — anyone with repo commit access can silence a warning. The
tool additionally supports trace auditing (`drskill audit`) across Claude Code,
Codex, Pi, and Copilot session traces to report which skills and MCP tools were
actually invoked.
