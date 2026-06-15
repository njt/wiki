---
url: https://github.com/trycua/cua
title: Cua — Computer Use Agent Platform
author: TryCua
date_fetched: 2026-06-15
date_published: 2025
---

# Full Analysis: trycua/cua

## Project Summary

Cua is a multi-language Computer Use Agent (CUA) platform. It provides SDKs (Python, TypeScript, Swift), a macOS desktop driver, a macOS VM orchestration tool (Lume), cloud sandbox infrastructure, benchmarking tools, and a web playground — all in a single monorepo. The core value proposition: one `ComputerAgent.run()` call works with Claude, GPT, Gemini, or open-source VLMs by auto-selecting the appropriate agent loop and computer backend.

## Scale

- ~170K lines Python
- ~50K lines Swift
- ~25K lines TypeScript
- ~245K total lines across ~15 packages
- 80+ CI/CD workflows
- Python 3.12+, Swift 6.0, TypeScript (Bun/Node)

## Architecture

### Layer Stack (top to bottom)

```
ComputerAgent (agent.py)
  ├── Agent Loop (loops/*.py)        ← per-model prompt/response adaptation
  ├── Computer Handler (computers/*.py)  ← abstract computer interface
  │     └── Computer (computer/computer.py)  ← platform orchestration
  │           ├── Interface (computer/interface/*.py)  ← per-OS drivers
  │           │     ├── macOS: ScreenCaptureKit + AX API (Swift via CuaDriver)
  │           │     ├── Linux: X11/Wayland
  │           │     ├── Windows: Win32 API
  │           │     └── Browser: Playwright
  │           └── VM Providers (computer/providers/*.py)
  │                 ├── Cloud: api.cua.ai → provision remote sandbox
  │                 └── Local: Lume (macOS VMs), Docker containers
  └── Callbacks (callbacks/*.py)     ← preprocessing/postprocessing pipeline
```

### Key Modules

**Agent Core** (`libs/python/agent/cua_agent/agent.py`, 1133 lines):
- `ComputerAgent` class: entry point. Takes model name, auto-selects loop, manages tool resolution, runs the main agent loop.
- Deferred initialization: tools are resolved lazily because `Computer.interface` may not exist until the computer is started.
- Retry logic: exponential backoff on transient API errors (rate limits, timeouts, 5xx).
- Ollama guard: explicitly rejects Ollama models for computer use (no vision support).

**Agent Loops** (`libs/python/agent/cua_agent/loops/`):
- `anthropic.py` (1959 lines) — Anthropic's computer-use tool. Maps between the internal "responses_items" format and Anthropic's hosted tools. Handles coordinate scaling (downscale screenshot to 1024x768 max, track scale factors, upscale coordinates in responses). Supports multiple tool versions (20241022, 20250124, 20251124) auto-selected by Claude model version.
- `openai.py` (426 lines) — OpenAI's computer-use-preview. Simpler than Anthropic since OpenAI's API is closer to the internal format.
- `gemini.py` (1028 lines) — Google's Gemini with computer use support.
- `fara/config.py` (661 lines) — FARA-7B, a browser-specific VLM. Uses `qwen_vl_utils.smart_resize` for image preprocessing with min/max pixel constraints. Parses model output from both tool_calls and raw text (`<tool_call>` tags, OpenRouter format). Has browser-native actions (visit_url, web_search, history_back) beyond generic click/type/scroll.
- Also: `qwen35.py`, `internvl.py`, `opencua.py`, `uitars.py`, `uitars2.py`, `moondream3.py`, `glm45v.py`, `holo.py`, `qwen3vl.py`, `gelato.py`, `gta1.py`, `omniparser.py` — each a dedicated loop for a specific VLM.

**Decorator-based Registration** (`libs/python/agent/cua_agent/decorators.py`):
- `@register_agent(models=r".*claude-.*")` — each loop registers with a regex pattern. `find_agent_config(model)` matches the model string against all registered patterns, selecting by priority. Supports `cua/<provider>/` routing prefix (e.g., `cua/anthropic/claude-sonnet-4-6` strips to `claude-sonnet-4-6`).
- `tool_type` parameter: models declare "browser" if they're browser-specific (FARA); the agent then auto-wraps Computer objects in BrowserTool.

**Computer Layer** (`libs/python/computer/computer/computer.py`, 1884 lines):
- `Computer` class: orchestrates the full lifecycle — VM creation/startup, interface connection, screen capture, input injection.
- Plugable VM providers: Cloud (api.cua.ai) or Local (Lume for macOS VMs, Docker containers, QEMU VMs).
- `DioramaComputer` — experimental app-use feature: creates virtual desktops from app names.
- Interface factory: auto-detects OS type and loads the right platform driver.
- `ComputerConfig` model (Pydantic): display, memory, CPU, OS type, image, shared directories, experiments, etc.

**Swift Driver** (`libs/cua-driver/swift/Sources/CuaDriverCore/`):
- **Capture** (`WindowCapture.swift`, 467 lines): uses ScreenCaptureKit (SCShareableContent) for screenshots. Tracks scale factors (pixels-per-point) for coordinate conversion. Supports window-level capture with per-window permissions. Handles `streamingFailed` error distinct from generic `captureFailed` — an actionable error when SCK can't stream a specific window.
- **Input** (`AXInput.swift`, `MouseInput.swift` 876 lines, `KeyboardInput.swift` 268 lines): uses Accessibility API (`AXUIElementCopyElementAtPosition`, `AXUIElementPerformAction`) plus `CGEventPost` for mouse/keyboard. Includes SkyLight private API for event posting.
- **Browser** (`CDPClient.swift`, `WebInspectorXPC.swift`, `AXPageReader.swift`, `BrowserJS.swift`): Chrome DevTools Protocol integration for browser control — page reading, element inspection, JS execution.
- **Cursor** (`AgentCursor.swift`, `AgentCursorOverlayWindow.swift`): renders a visual agent cursor overlay with motion paths (Bezier curves), distinct from the system cursor.
- **Recording** (`RecordingSession.swift`): records agent sessions — screenshots, cursor positions, click markers — for replay and trajectory analysis.
- **Daemon** (`DaemonServer.swift`): Unix domain socket daemon for persistent permission grants.
- **MCP Server** (`libs/cua-driver/swift/Sources/CuaDriverServer/`): 30+ MCP tools (screenshot, click, type, scroll, drag, launch_app, list_apps, list_windows, hotkey, etc.) plus Claude Code compatibility layer.

**Lume** (`libs/lume/`):
- macOS VM manager using Apple's Virtualization.framework (`VZVirtualMachine`).
- `LumeController.swift` (1694 lines): CRUD for VMs — create, run, stop, pause, resume, delete. Supports custom images, shared directories, recovery mode.
- `VMVirtualizationService.swift` (538 lines): wraps `VZVirtualMachine` lifecycle with async/await.
- `DarwinImageLoader.swift`: loads macOS restore images (.ipsw) for VM creation.
- `VNCService.swift`: VNC server integration for remote desktop access to VMs.
- `MCPServer.swift`: MCP server for VM management via LLMs.
- Container registry support (GCS, generic HTTP).

**TypeScript SDKs** (`libs/typescript/`):
- `core`: HTTP client and telemetry (PostHog).
- `computer`: mirrors Python Computer — `BaseComputerInterface` with WebSocket connection to VM, platform-specific interface implementations (macOS, Linux, Windows), cloud VM provider.
- `agent`: agent SDK client wrapping the computer layer.
- `cua-cli`: Bun-based CLI for auth, sandbox management, image management, MCP server, skill management.
- `playground`: React web UI with VNC viewer, chat interface, sandbox management, trajectory viewer.

**Other notable packages**:
- `libs/python/som`: Set-of-Mark visual grounding for UI elements.
- `libs/python/mcp-server`: Python MCP server wrapping Computer for Claude Desktop integration.
- `libs/python/computer-server`: HTTP server for remote computer control.
- `libs/cuabot`: Discord/Telegram bot for computer use.
- `libs/cua-bench`: benchmarking framework (cuabench).
- `libs/kasm`, `libs/xfce`, `libs/qemu-docker`: container sandbox implementations.

### Callback System

The agent has an extensible callback pipeline (`callbacks/`). Each lifecycle hook (`on_run_start`, `on_llm_start`, `on_llm_end`, `on_computer_call_start`, `on_api_start`, etc.) iterates all registered callbacks:

- `OperatorNormalizerCallback`: always first, normalizes tool call formats.
- `ImageRetentionCallback`: keeps only N most recent images.
- `BudgetManagerCallback`: tracks token costs, stops when budget exceeded.
- `TrajectorySaverCallback`: saves screenshots + responses for replay.
- `TelemetryCallback`: PostHog product analytics.
- `OtelCallback`: OpenTelemetry for operational metrics (four golden signals).
- `LoggingCallback`: configurable verbosity.
- `PromptInstructionsCallback`: injects system instructions.
- `PIIAnonymizationCallback`: optionally strips PII from messages.

## Key Techniques

### 1. Multi-Provider Agent Abstraction

The `AsyncAgentConfig` protocol is the spine of the system. Every model gets its own class implementing `predict_step()`, `predict_click()`, and `get_capabilities()`. The `@register_agent` decorator with regex model matching makes adding a new model a matter of writing one class file and decorating it. This is cleaner than the typical "one big adapter with if/else" pattern but costs code duplication — Anthropic's loop is 1959 lines largely because it has to convert between Anthropic's tool format and the internal format in both directions.

### 2. Coordinate Scaling Chain

For Anthropic (which limits screenshots to 1024x768 internally), coordinates go through a careful upscale/downscale pipeline:
1. Screenshot is downscaled before sending (PIL/LANCZOS resize)
2. Scale factors stored: `(original_w / new_w, original_h / new_h)`
3. Tool dimensions also capped to match
4. Response coordinates are upscaled: `x * scale_x`
5. Both the tool schema dimensions and the actual image dimensions stay in sync

This avoids the "coordinate drift" bug that many computer-use implementations hit.

### 3. FARA's smart_resize Integration

For Qwen-based VLMs (FARA), images go through `qwen_vl_utils.smart_resize` with `min_pixels=3136`, `max_pixels=12845056`, `factor=28`. Coordinates are then scaled from resized→original space. The "hints" (`min_pixels`, `max_pixels`) are attached to image blocks in the message so the model sees them.

### 4. Dual Tool Call Parsing

FARA's loop handles two output formats from the model:
- Structured: `message.tool_calls[]` array (Ollama Cloud format)
- Unstructured: raw text with `<tool_call>` XML tags (OpenRouter format)

This dual parsing is pragmatic — different inference providers return different formats, and trying to force one format across all providers is a losing game.

### 5. LiteLLM as Universal Translation Layer

The entire agent stack uses `litellm.acompletion()` as the single API surface. This is a significant architectural choice: instead of each loop calling provider SDKs directly, they all go through LiteLLM. Benefits: one retry/logging/cost-tracking layer. Cost: another abstraction to debug when providers diverge.

### 6. Browser-Specific Agent Mode

The FARA loop has actions that generic computer-use loops don't: `visit_url`, `web_search`, `history_back`. These are not mapped to click/type — they go directly to the browser's Playwright instance. The `tool_type="browser"` declaration on the `@register_agent` decorator triggers `ComputerAgent._resolve_tools()` to auto-wrap regular `Computer` objects in `BrowserTool`.

### 7. Native macOS Input Injection

The Swift driver uses `CGEventPost` via the private SkyLight framework (`SkyLightEventPost.swift`) for reliable input injection. macOS's accessibility API alone can be unreliable for certain apps — the private API bridge fills the gap. Combined with `AXUIElementCopyElementAtPosition` for element resolution and `AXUIElementPerformAction` for semantic actions (press, confirm, etc.), this gives the driver both precision and semantic understanding.

### 8. WebSocket-Based Remote Desktop

The TypeScript `BaseComputerInterface` connects to VMs via WebSocket (not HTTP polling). This provides real-time screen updates and input command delivery. The connection is authenticated via API key headers (`X-API-Key`, `X-VM-Name`).

## Design Decisions

**Optimized for: multi-model support, macOS sandbox fidelity, extensibility via callbacks.**

**Sacrificed: simplicity, cross-platform parity, streaming granularity.**

### Trade-offs

1. **Loop code duplication vs. unified adapter**: Each model gets its own loop file (some 400-2000 lines). The alternative — a single adapter with model-specific branches — would be fewer lines but harder to evolve independently. The project chose duplication. For a small team, this is the right call: adding a new model means writing one new class, not modifying a shared adapter.

2. **LiteLLM dependency**: Adds translation flexibility but means every API error goes through two abstraction layers (provider → LiteLLM → agent loop). Debugging requires understanding LiteLLM's internal routing. The `_strip_cua_prefix` function reveals that Cua adds its own routing layer on top (`cua/<provider>/<model>`), which LiteLLM must strip before matching.

3. **macOS-first, but not macOS-only**: Lume (macOS VM orchestration) and CuaDriver (native macOS input/capture) are macOS-only Swift code. The Python/TypeScript computer interfaces have cross-platform abstractions, but the highest-fidelity experience requires macOS. This is practical — Apple Silicon Macs are uniquely positioned for running macOS VMs — but limits deployment.

4. **Protocol-based abstraction with streaming trade-offs**: `AsyncAgentConfig.predict_step()` returns the full response, not a stream. The agent loop itself supports streaming internally (LiteLLM's `stream=True`), but the protocol doesn't expose it. This simplifies the callback system but means callers can't render partial responses.

5. **Callback pipeline as state mutation**: Callbacks modify the message list as a side effect during `on_llm_start`/`on_llm_end`. This is powerful (e.g., PII anonymization transparently modifies messages) but opaque — the ordering and interaction of callbacks matters, and there's no formal contract beyond "call in insertion order."

6. **VM lifecycle is external to the agent**: The `Computer` class handles VM creation/startup separately from the agent loop. The agent just gets a `computer_handler` and calls `screenshot()`/`click()`/etc. This separation is clean but means the agent can't reason about VM state (e.g., "the VM is still booting") — that's handled by timeouts and retries at the computer layer.

## Innovation Points

1. **Model auto-detection with priority ordering**: Not just matching — priority system means newer Claude models get the `computer_20251124` tool version while older ones fall back to `computer_20241022`. This is model-version-aware routing, not just model-name routing.

2. **Lume's macOS-on-Apple-Silicon virtualization**: Running macOS VMs on Apple Silicon for sandboxed computer use is genuinely rare. Most computer-use platforms use Linux containers or cloud Windows. Cua can spin up isolated macOS desktops locally.

3. **CuaDriver's multi-layer input strategy**: Combines Accessibility API (structured element interaction), CGEvent posting (reliable injection), and Chrome DevTools Protocol (browser-specific) — each for its strength. Not many tools blend all three.

4. **Trajectory recording and replay**: The agent records full sessions — screenshots, cursor positions (sampled), click markers, API responses — and can render them as video or replay them. This is infrastructure for training data collection and debugging.

5. **`cua/<provider>/<model>` routing prefix**: A custom model namespace that routes through Cua's own proxy/load-balancing layer before hitting LiteLLM. This lets Cua offer a unified model string format regardless of the actual provider.

## Comparison Notes

**vs. Anthropic's computer-use demo code**: Cua wraps Anthropic's computer-use API in a multi-provider framework. Anthropic's reference code is single-provider; Cua adds model auto-selection, callback extensibility, and cross-platform computer backends.

**vs. Browser Use**: Browser Use is browser-only; Cua covers desktop (macOS/Linux/Windows) plus browser. Cua has a broader scope but more complexity. Browser Use's anti-detection is more advanced.

**vs. Webwright**: Webwright is Microsoft Research's approach to making coding models into browser agents via terminal + Playwright (~1.5K LoC). Cua is a production SDK at ~245K lines with cloud infrastructure. Webwright's insight (code-as-action) is elegant; Cua's approach (comprehensive SDK) is practical.

**vs. xa11y**: xa11y uses accessibility tree queries (structured element selection) rather than vision-based models. Cua's CuaDriver uses BOTH: AX API for structured input plus ScreenCaptureKit for vision. The hybrid approach is more flexible but more complex.

**vs. OpenAI's Operator**: Cua provides the SDK layer that something like Operator would be built on. It's not a consumer product — it's infrastructure for building computer-use agents.

## Tags

#tool #project #agents #computer-use #sdk #macos #sandbox #vlm
