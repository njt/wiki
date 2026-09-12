---
title: "Claude Lamp"
url: https://github.com/bobek-balinek/claude-lamp
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# Claude Lamp: Physical Status Indicator for Claude Code

## Overview
Control your Moonside LED lamp via BLE based on Claude Code's state. The lamp becomes a physical status indicator -- animated themes while Claude works, green when idle, purple when it needs your input.

## State-Based Visual Feedback
- Working: BEAT2 theme (white/navy animation)
- Idle: Solid sunset mango (255, 180, 50)
- Needs Input: Solid purple (200, 0, 255)
- Off: LED disabled at session end

## Architecture
Uses a persistent BLE connection daemon that avoids reconnection delays. A shell hook writes state to a temporary file, and the background daemon reads it every 200ms. Hooks into eight Claude Code events: SessionStart, UserPromptSubmit, Stop, PreToolUse, PostToolUse, PermissionRequest, Notification, and SessionEnd.

## Requirements
- macOS (BLE via CoreBluetooth)
- Python 3.10+
- Moonside lamp (tested with Halo; compatible with One, Aurora, Lighthouse)
- bleak library for Python

The daemon auto-exits after 30 minutes of inactivity. Also includes standalone BLE controller for color settings, themes, brightness, and interactive REPL mode.
