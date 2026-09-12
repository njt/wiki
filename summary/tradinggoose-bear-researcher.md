---
url: https://github.com/TradingGoose/TradingGoose.github.io/blob/main/supabase/functions/agent-bear-researcher/index.ts
title: "TradingGoose: agent-bear-researcher Edge Function"
author: TradingGoose (crafted with Claude Code)
date_fetched: 2026-05-15
date_published: unknown
topics:
  - agent-architecture
---

# agent-bear-researcher — Supabase Edge Function

This is a Supabase Edge Function that implements the **Bear Researcher** agent in TradingGoose's multi-agent stock analysis system. It receives analysis requests from a coordinator, gathers insights from peer agents (market analyst, social media, news, fundamentals), constructs a position-aware prompt, calls an AI provider, and saves bearish analysis results back to the database. It participates in a multi-round debate with a Bull Researcher agent.

## Architecture

The agent runs as a Deno-based Supabase Edge Function (`serve()`). On invocation:
1. Validates the POST request (requires `analysisId`, `ticker`, `userId`, `apiSettings`)
2. Initializes Supabase client and sets up a self-retry timeout (3 retries, 180s timeout)
3. Checks for analysis cancellation
4. Fetches insights from peer agents (`agent_insights`: marketAnalyst, socialMediaAnalyst, newsAnalyst, fundamentalsAnalyst)
5. Builds position context tailored to portfolio state (profit-taking vs stop-loss vs no-position)
6. Constructs a debate-round-aware prompt incorporating previous bull/bear rounds
7. Calls AI provider with retry logic
8. Atomically saves results via `updateAgentInsights` and `updateDebateRounds`
9. Notifies coordinator to check if more debate rounds are needed

## Position-Aware Prompting

The agent's prompt construction is the most interesting part. It tailors its bearish stance based on portfolio context:

- **Profitable position (above profit target):** argues for taking profits before reversal, warns about greed, suggests reducing to target allocation
- **Losing position (below stop loss):** argues for cutting losses, warns about catching a falling knife, questions the investment thesis
- **Within-range position:** presents risk/reward analysis, suggests reduction or exit
- **No position:** argues why staying out is correct, suggests alternatives for available cash

Position sizing is evaluated against configurable min/max thresholds with near-threshold warnings.

## Debate Pattern

The Bear Researcher engages in a structured debate with a Bull Researcher across multiple rounds. Each round:
- The bear receives the bull's arguments from the current round
- Must directly counter specific bull claims
- Must add NEW risks and evidence (not repeat previous points)
- References specific overly optimistic assumptions in the bull case

## Error Handling

Categorizes AI provider errors into: `rate_limit`, `api_key`, `ai_error`, `data_fetch`, `database`, `timeout`, `other`. On error, calls `setAgentToError` which notifies the coordinator.

## Imports (shared modules)

- `atomicUpdate.ts` — `updateAgentInsights`, `updateWorkflowStepStatus`, `updateAnalysisPhase`, `updateDebateRounds`, `setAgentToError`
- `cancellationCheck.ts` — `checkAnalysisCancellation`
- `aiProviders.ts` — `callAIProviderWithRetry`, `SYSTEM_PROMPTS`
- `coordinatorNotification.ts` — `notifyCoordinatorAsync`
- `agentSelfInvoke.ts` — `setupAgentTimeout`, `clearAgentTimeout`, `getRetryStatus`
- `types.ts` — `AgentRequest`, `getHistoryDays`

## Technology

Deno, TypeScript, Supabase (PostgreSQL, Edge Functions, Auth), pluggable AI providers (OpenAI, Anthropic, Google, DeepSeek, etc.).
