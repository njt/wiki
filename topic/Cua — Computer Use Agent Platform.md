# Cua — Computer Use Agent Platform

Cua is a ~245K-line multi-language monorepo providing the full stack for building computer-use AI agents: Python/TypeScript/Swift SDKs, a native macOS desktop driver (screen capture + input injection via Accessibility API and ScreenCaptureKit), a macOS VM orchestrator (Lume, using Virtualization.framework on Apple Silicon), cloud sandbox infrastructure, benchmarking tools, and a web playground. Its core insight: one `ComputerAgent.run()` call works with Claude, GPT, Gemini, or open-source VLMs by auto-detecting the model and selecting the appropriate agent loop, tool schema, and coordinate scaling strategy.

## Architecture

The system is a **layered adapter stack** with a protocol-based spine:

```
ComputerAgent (cua_agent/agent.py, 1133 lines)
  ├── Agent Loop (loops/*.py) — per-model prompt/response adaptation
  │     ├── @register_agent decorator maps regex → loop class
  │     └── Each loop converts between the internal "responses_items"
  │         format and the model's native tool/response format
  ├── Computer Handler (computers/cua.py, 184 lines)
  │     └── AsyncComputerHandler protocol: screenshot/click/type/etc.
  ├── Computer (computer/computer.py, 1884 lines)
  │     └── Orchestrates VM lifecycle + platform interface + providers
  ├── Platform Drivers
  │     ├── macOS: CuaDriver (Swift, ScreenCaptureKit + AX API)
  │     ├── Linux: X11/Wayland (Python)
  │     ├── Windows: Win32 API (Python)
  │     └── Browser: Playwright (Python, via BrowserTool)
  └── Callbacks (callbacks/*.py) — preprocessing/postprocessing pipeline
        ├── ImageRetentionCallback: keep N most recent screenshots
        ├── BudgetManagerCallback: stop when token budget exceeded
        ├── TrajectorySaverCallback: record sessions for replay
        ├── TelemetryCallback: PostHog + OpenTelemetry
        └── OperatorNormalizerCallback: normalize tool call formats
```

**Key files**: `libs/python/agent/cua_agent/agent.py` (main loop), `libs/python/agent/cua_agent/loops/anthropic.py` (1959 lines, Anthropic adapter), `libs/python/agent/cua_agent/loops/fara/config.py` (661 lines, browser VLM adapter), `libs/python/computer/computer/computer.py` (1884 lines, computer orchestration), `libs/cua-driver/swift/Sources/CuaDriverCore/` (macOS native driver), `libs/lume/src/LumeController.swift` (1694 lines, macOS VM management), `libs/typescript/computer/src/computer/providers/cloud.ts` (cloud VM provider).

## Key Techniques

**Model auto-detection with version-aware routing.** Each agent loop registers with `@register_agent(models=r".*claude-.*", priority=0)`. `find_agent_config(model)` matches the model string against all patterns. Crucially, Anthropic's loop supports three tool versions (`computer_20241022`, `computer_20250124`, `computer_20251124`) auto-selected by Claude model version — newer Opus 4.6 gets the 2025-11-24 beta, Claude 3.5 falls back to 2024-10-22. See `_get_tool_config_for_model()` in `loops/anthropic.py:82`.

**Coordinate scaling chain.** Anthropic's API limits screenshots to 1024×768 internally. Cua proactively downscales (PIL/LANCZOS), tracks `(scale_x, scale_y)`, caps tool dimensions to match, then upscales all response coordinates. The scaling is exact — not fuzzy — because the same scale factors apply to both image and tool dimensions. See `_convert_completion_to_responses_items()` in `loops/anthropic.py:746`.

**FARA's smart_resize + dual parsing.** For Qwen-based browser VLMs, images pass through `qwen_vl_utils.smart_resize(factor=28, min_pixels=3136, max_pixels=12845056)`. Model output is parsed from two formats: structured `tool_calls[]` (Ollama Cloud) and raw `<tool_call>` XML tags (OpenRouter). Browser-native actions (`visit_url`, `web_search`, `history_back`) bypass click/type and go directly to Playwright. See `loops/fara/config.py:303-344` for image preprocessing, `:372-457` for dual parsing.

**LiteLLM as universal API surface.** Every loop calls `litellm.acompletion()` rather than provider SDKs directly. This gives one retry/logging/cost-tracking layer but adds an abstraction to debug. The custom `cua/<provider>/<model>` routing prefix lets Cua proxy through its own load balancer.

**Native macOS hybrid input.** CuaDriver blends Accessibility API (`AXUIElementCopyElementAtPosition`, structured element interaction), `CGEventPost` via private SkyLight framework (reliable injection), and Chrome DevTools Protocol (browser-specific). No other desktop automation tool combines all three at this level.

**Tool type negotiation.** Models declare `tool_type="browser"` on their `@register_agent` decorator. The agent's `_resolve_tools()` automatically wraps generic `Computer` objects in `BrowserTool` when the model requires it. This means the same computer object works with both Claude (flexible) and FARA (browser-only) without the caller knowing the difference.

## Design Decisions

**Optimized for multi-model support at the cost of loop code duplication.** Each model gets its own 400-2000 line loop file rather than branches in a shared adapter. Right call for a small team — adding a model means writing one new class, not modifying a shared file with regression risk.

**macOS-first, not macOS-only.** Lume (macOS VMs on Apple Silicon) and CuaDriver (native macOS input) are Swift-only. Cross-platform abstraction exists (Linux/Windows interfaces in Python), but the highest-fidelity experience requires macOS — pragmatic since Apple Silicon Macs uniquely run macOS VMs.

**Callback pipeline as state mutation.** Callbacks modify the message list during `on_llm_start`/`on_llm_end`. Powerful (PII stripping is transparent) but opaque — ordering matters and there's no formal contract beyond "insertion order."

**Protocol sacrifices streaming.** `AsyncAgentConfig.predict_step()` returns the full response, not a stream. Simplifies callbacks but means callers can't render partial output — a real limitation for chat-like UX.

## Comparison Notes

**vs. [[Browser Use]]**: Browser-only vs. Cua's desktop + browser scope. Browser Use has better anti-detection; Cua has broader platform coverage and cloud infrastructure.

**vs. [[Webwright]]**: Webwright (~1.5K LoC) is a research prototype proving code-as-action for browser agents. Cua (~245K lines) is a production SDK with cloud, replay, benchmarking, and multi-provider support. Webwright's insight is elegant; Cua's implementation is comprehensive.

**vs. [[xa11y — Desktop Automation via Accessibility APIs]]**: xa11y is structured AX-tree queries only. Cua uses AX + vision (screenshots) — the hybrid is more flexible but adds the coordinate scaling complexity that vision brings.

**vs. Anthropic's reference computer-use code**: Cua wraps Anthropic's API in a multi-provider framework with model auto-selection, callbacks, and cross-platform backends. It's what you'd build if you needed to support Claude, GPT, Gemini, and FARA from one codebase.

**vs. [[Computer Use is 45x More Expensive Than Structured APIs]]**: Cua's entire existence validates this observation — computer use IS expensive (vision tokens), which is why the platform invests so heavily in image retention callbacks, budget management, and model selection that can route to cheaper VLMs for simpler tasks.

**vs. [[LongHorizon-Harness]]**: Cua builds computer-use *agents* (drivers, VMs, per-model loops); LongHorizon-Harness builds the *loop around existing agents* (Claude Code/Codex/OpenCode) with a Manager/Executor/Auditor verify-and-checkpoint cycle, getting screen control secondhand via MCP plugins rather than native drivers.

## Tags

#tool #project #agents #computer-use #sdk #macos #sandbox #vlm #browser-automation

*Source: [github.com/trycua/cua](https://github.com/trycua/cua) — ingested 2026-06-15*
