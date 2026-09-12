---
url: https://github.com/quant-sentiment-ai/claude-equity-research/blob/main/commands/trading-ideas.md
title: Claude Equity Research Plugin — Trading Ideas Command
author: quant-sentiment-ai
date_fetched: 2026-05-15
date_published: 2025-09-09
topics:
  - claude-code
---

# Claude Equity Research Plugin

A Claude Code plugin for institutional-grade equity research. Install with `/plugin marketplace add`. Generates professional buy/sell recommendations with comprehensive fundamental analysis, technical indicators, and risk assessment. Educational use only — not financial advice.

Built entirely with Claude Code. 511 stars, 59 forks, MIT licensed.

## Research Framework

The `/trading-ideas:research TICKER` command follows an 8-section institutional framework:

1. **Executive Summary** — Investment thesis with price target and timeframe
2. **Fundamental Analysis** — Revenue growth, margins, peer comparisons, forward estimates
3. **Catalyst Analysis** — Near-term (0-6mo) and medium-term (6-24mo) market drivers
4. **Valuation & Price Targets** — Bull/base/bear scenarios with probability weighting
5. **Risk Assessment** — Company-specific and macro risks with position sizing (1-5%)
6. **Technical Context** — Support/resistance, momentum, options flow, implied volatility
7. **Market Positioning** — Sector performance, rotation trends, relative strength
8. **Insider Signals** — Executive buying/selling, buybacks, institutional ownership

## Output Format

```markdown
# $TICKER - ENHANCED EQUITY RESEARCH

## EXECUTIVE SUMMARY
[BUY/SELL/HOLD] with $X price target (X% upside/downside) over [timeframe].

## RECOMMENDATION SUMMARY
| Metric | Value |
|--------|-------|
| Rating | BUY/SELL/HOLD |
| Conviction | High/Medium/Low |
| Price Target | $X |
| Timeframe | X months |
```

All sections require specific numbers, percentages, analyst citations, and probability weightings.

## Installation

**Plugin (recommended):**
```bash
/plugin marketplace add quant-sentiment-ai/claude-equity-research
/plugin install trading-ideas@claude-equity-research-marketplace
```

**Manual:**
```bash
curl -o ~/.claude/commands/trading-ideas.md \
  https://raw.githubusercontent.com/quant-sentiment-ai/claude-equity-research/main/commands/trading-ideas/commands/research.md
```

## Repository Structure

```
commands/trading-ideas/
  .claude-plugin/plugin.json    # Plugin manifest
  commands/research.md           # Slash command implementation
config/config.example.json
docs/{methodology,installation,customization}.md
examples/sample_reports/{AAPL,HOOD}_analysis.md
```
