---
title: "Building Production-Ready Voice Agents"
url: https://shekhargulati.com/2026/01/03/building-production-ready-voice-agents/
date_fetched: 2026-05-14
section: "Producing and Operating Software"
---

Shekhar Gulati documents building a production voice agent platform for IT support at higher education institutions. Three developers, handling password resets, FAQ responses, and call routing.

Stack: Python/FastAPI, Pipecat, Deepgram (STT), OpenAI GPT-4.1, Cartesia (TTS), Twilio, PostgreSQL.

Key lessons:

1. State Machines for Conversation Management -- model conversations as state machine graphs. Prevents "context pollution" where earlier instructions leak into later states.

2. Avoid Drag-and-Drop Flow Builders Initially -- code-based flows are testable. Start with code, consider visual editors only after understanding patterns.

3. Data Confirmation Through Repetition -- "Always repeat back what you captured and get explicit confirmation before proceeding." NATO phonetic alphabet for alphanumeric data.

4. Admin Portal Investment -- "roughly 50% of our development effort goes into the admin portal -- not the voice agent itself." Turn-level conversation analysis, replay, configuration management, RBAC.

5. Human Transfer as Mandatory Fallback -- every conversation stage should offer transfer options. Complex routing logic around business hours, holidays, departments.

6. Function Calling Reliability -- highest failure point. Validation in code handlers, not LLM. Function-calling failures appear as hallucinations.

7. API Failures and Timeouts -- mirrors distributed systems principles: timeouts everywhere, circuit breakers, graceful degradation, idempotent operations.

8. Latency -- users hang up 40% more with responses exceeding one second. P95 latencies 1.5-2.5 seconds depending on model.

9. Voice-Specific Prompt Engineering -- cap responses at 30-50 words. Users can't scroll back.

"The demo shows the happy path -- a cooperative user, clear audio, no edge cases. Production shows you everything else."

"Silence is death."