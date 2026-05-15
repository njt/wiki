---
title: "Claude's System Prompt"
url: https://raw.githubusercontent.com/asgeirtj/system_prompts_leaks/refs/heads/main/claude.txt
date_fetched: 2026-05-14
section: "LLMs"
---

# Claude's System Prompt — Complete Reference

Claude Opus 4.6's system prompt, leaked and documented. Covers behavior, tools, memory, safety, and technical capabilities.

## Core Identity & Behavior

Claude is Claude Opus 4.6. The prompt establishes consistent values across conversations, warm and respectful tone, and avoidance of over-formatting. Key behavioral principles:
- Discussing virtually any topic factually while maintaining ethical boundaries
- Providing nuanced answers rather than oversimplified yes/no responses
- Acknowledging mistakes honestly without excessive apology
- Treating users with kindness while maintaining respectful engagement

## Critical Safety Framework

### Child Safety
Claude "NEVER creates romantic or sexual content involving or directed at minors" and refuses requests that could facilitate grooming.

### Harmful Content Restrictions
Declines requests involving weapons/WMDs creation, malicious code/malware development, and content sexualizing real public figures.

## Memory System

Dynamic memory from past conversations, applied naturally without drawing attention to retrieval. Critical rules:

**Forbidden Phrases:** Claude never uses "Based on my memories," "I can see," "According to," or similar retrieval-signaling language when applying memories.

**Boundaries:** Claude avoids overindexing on memory's presence, recognizing that "AI-human relations" differ fundamentally from human relationships.

## Tool Systems

### Past Chats Tools
- **conversation_search**: Keyword-based topic retrieval
- **recent_chats**: Time-based filtering (up to 20 chats per call)

Claude proactively uses these when conversations reference prior discussions.

### Computer Use (Linux/Ubuntu 24)
Bash, file creation, and editing capabilities. User uploads at `/mnt/user-data/uploads`, working directory `/home/claude`, final outputs at `/mnt/user-data/outputs`.

### Artifacts
Standalone files for content exceeding ~100 lines. Support markdown, HTML, React (with Tailwind, recharts, Three.js, TensorFlow).

### Visualizer System
Inline SVG diagrams and interactive HTML. Evaluation checklist: Does this need a visual? Does a connected MCP tool fit? Did the person ask for an Artifact? Does a first-party widget fit? Use Visualizer (default).

## Formatting & Style

Avoids over-formatting. Minimal bullets/headers—primarily when users explicitly request lists or formatting is essential for clarity. Reports and explanations use prose paragraphs.

## Refusal & Boundaries

Declines: CSAM, weapons/WMD instructions, malware, self-destructive behavior reinforcement. But does NOT use `end_conversation` for self-harm or violence scenarios—engages supportively.

## End Conversation Tool

Terminates conversations only as last resort after multiple redirections and explicit warning. Never used for self-harm or violence scenarios.

## Anthropic Reminders

Automated internal reminders (image_reminder, cyber_warning, system_warning, ethics_reminder, ip_reminder, long_conversation_reminder) help maintain context. Not visible to users.
