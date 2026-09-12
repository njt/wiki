---
url: https://github.com/inanimate-tech/resident
title: Resident — Sandboxed Lua Runtime for ESP32 with Hot Reload
author: inanimate-tech (Jesse Vincent / obra)
date_fetched: 2026-05-22
date_published: 2025-01
topics:
  - misc
---

# Resident — Full Architectural Analysis

## What it is

Resident provides a sandboxed Lua 5.4 runtime for ESP32 devices with network hot-reload. Hardware peripherals are exposed to Lua through a driver interface, so apps can draw to displays, read sensors, and control outputs without touching C++. Apps are pushed over WebSocket as Lua source strings; the sandbox compiles and runs them in-place, replacing any previously running app. An optional Claude Code plugin lets AI coding agents author, validate, and push Lua apps to devices from a chat interface.

## Project stats

- Core library: ~1,700 lines of C++ across 8 source files (headers + single .cpp)
- Tests: PlatformIO native unit tests with hand-rolled Arduino/Courier/ezTime stubs
- Examples: 4 buildable PlatformIO projects (M5StickC Plus2, Adafruit ESP32-S2 TFT Feather, ESP-IDF basic, and a Claude Code permission-hook relay)
- Dependencies: Courier (networking), ArduinoJson, ezTime (NTP/timezone), Esp32Lua (Lua 5.4 on ESP32), WiFiManager

## Architecture

### Entry point: `Resident::Sandbox` (src/ResidentSandbox.h, src/ResidentSandbox.cpp)

The single public class. Combines an optional Courier networking layer with a Lua VM. Constructed with a `SandboxConfig`, then driven by Arduino's `setup()`/`loop()` pattern.

Construction is two-phase:
1. **Constructor** — applies config, constructs Courier::Client if network is configured, derives device ID from chip MAC
2. **`setup()`** — idempotent. Fires user's `onConfigureNetwork` callback, wires internal Courier hooks (status display/LED updates, reserved-type message routing), calls `initialize()` to create Lua state and register extensions, then kicks Courier's WiFi+transport setup
3. **`loop()`** — drives Courier loop, updates status display, ticks extensions at full rate, fires Lua `on_tick` at 10 FPS (gated on `isConnected()` when networked), dispatches one queued event per iteration

### Extension system (src/ResidentExtension.h, src/ResidentDriver.h)

Two-tier hierarchy:
- **`Extension`** — base class. Pure virtual `name()` returns the Lua global name. Optional `begin()`, `update()`, `registerModule()`, `onAppReset()`. Uses a static helper `beginExtension()` for idempotent init.
- **`Driver`** — extends Extension with `sendEvent()` (protected) and `onAppRunning()` hook. Events are queued into the sandbox's ring buffer and delivered to Lua's `on_event`. Implements RTTI-free downcast via `asDriver()` returning `this` — avoids `dynamic_cast` which requires `-frtti` (disabled by Arduino/ESP32 builds).

### Lua binding system (src/ResidentLuaModule.h)

A builder pattern. Extensions receive a `LuaModule&` in `registerModule()` and chain `.method<>()`, `.staticMethod()`, `.constant()` calls. The template machinery (`Trampoline<C, &C::fn>`) stores the `Extension*` as a Lua upvalue and casts it back to the derived class pointer on call. This works without RTTI because the cast is `static_cast<C*>(void*)`, which is correct only when Extension is the leftmost base class.

### Event system

Fixed-size ring buffer (8 slots) in `Sandbox::_events[]`. Three event types:
- **BUTTON** (legacy, not used in current code)
- **DRIVER** — from `Driver::sendEvent()`. Fields serialized as compact JSON into a 256-byte buffer, then parsed at delivery time by flattening onto the Lua event table.
- **APP_EVENT** — from WebSocket messages of type `"app_event"`. Data parsed into an `event.data` subtable.

### Message protocol

Three reserved JSON message types routed internally:
- `{"type":"app","code":"..."}` — compiles and runs Lua source
- `{"type":"shader","expr":"..."}` — converts via user-provided ShaderTemplateFn, then runs
- `{"type":"app_event","name":"...","data":{...}}` — queues an event for Lua's `on_event`

All other types are forwarded to the user's `onMessage` callback.

### Networking (opt-in via std::optional<Courier::Config>)

When `cfg.network` is set, the Sandbox constructs an internal `Courier::Client` (from the sibling Courier library) which handles WiFi (via WiFiManager captive portal on first boot), NTP time sync, and WebSocket transport. When unset, the sandbox runs standalone — no WiFi code is linked, `isConnected()` always returns false, and `loop()` ticks Lua unconditionally.

### PSRAM allocator

On ESP32 (`#ifdef ESP_PLATFORM`), a custom Lua allocator routes all Lua allocations to PSRAM via `heap_caps_malloc(MALLOC_CAP_SPIRAM)`. This preserves internal SRAM for the WiFi stack and driver buffers.

### Claude Code agent plugin (tools/agent-plugin/)

Five skills at paths like `tools/agent-plugin/skills/*/SKILL.md`:
- **`create-app`** — reads DEVICE-SKILL.md + sandbox docs, generates Lua from natural language, validates, retries up to 3 times
- **`validate-app`** — runs Lua through a local interpreter under a permissive stub harness derived from DEVICE-SKILL.md. Catches syntax errors, missing lifecycle functions, nil dereferences
- **`push-app`** — sends Lua to the WebSocket relay at `resident.inanimate.tech/devices/<deviceId>/send` or a self-hosted endpoint
- **`write-device-skill`** — interactively authors DEVICE-SKILL.md for new hardware
- **`hello-resident`** — liveness check

The plugin is distributed via a Claude Code marketplace (`inanimate-tech/agent-plugins`).

## Key techniques

1. **Template trampoline for Lua binding without RTTI**: `Trampoline<C, &C::fn>::call` is a static function compatible with `lua_CFunction`. It pulls the `Extension*` from an upvalue and casts to `C*`. The `static_assert` guarantees C derives from Extension. This avoids `dynamic_cast` (which requires `-frtti`, disabled on ESP32) and avoids per-method thunks in the derived class.

2. **void* + fn pointer instead of std::function for Driver event sink**: `Driver` stores `EventSinkFn _eventSinkFn` (plain function pointer) and `void* _eventSinkCtx` rather than `std::function`. This keeps Driver allocation-free for embedded targets — important when drivers may be constructed statically before heap is available.

3. **std::optional<Courier::Config> as networking opt-in**: The entire networking code path is gated on `_courier.has_value()`. This is a compile-time choice encoded as a runtime check — the linker can't strip the Courier code since it's always compiled, but the pattern means a single Sandbox class serves both networked and standalone use cases without `#ifdef` soup.

4. **Idempotent init via static helper**: `Extension::beginExtension(Extension& e)` checks a private `_begun` flag. Users can call it early (e.g., to initialize a display before the sandbox), and the Sandbox's own call is a safe no-op. This avoids the "who calls begin() first" ordering problem.

5. **Runtime error rate limiting**: `on_tick` fires at 10 Hz — a broken app could spam telemetry. The sandbox limits to 3 errors in a burst, then requires a 5-second cooldown before reporting more. `init` and `on_event` errors are always reported since they fire less frequently.

6. **Hand-rolled JSON parser for event delivery**: Rather than linking ArduinoJson into the hot path of `processNextEvent()`, the sandbox does a single-pass char-by-char parse of the compact-JSON event data. This avoids allocation in the event dispatch path and keeps the JSON library out of the Lua call stack (where heap pressure matters).

7. **`rgb()` returns negative packed values**: The shader convention is that `rgb(r,g,b)` returns `-(packed_rgb)` — a negative integer signals "this is a color, not a number" to shader template code. This is a clever overloading of Lua's number type (all Lua numbers are doubles, but integers up to 2^53 are exact).

## Design decisions

- **Single .cpp file for the entire sandbox (1,125 lines)**: Deliberate choice — keeps the linker happy on ESP32 where each .cpp adds overhead. The header files are clean interfaces; the implementation is one compilation unit. This is unusual by modern C++ standards but pragmatic for embedded.
- **10 FPS Lua tick**: Chosen for display refresh cadence, not CPU limits. The `TICK_INTERVAL` is 100ms. Extensions' `update()` runs at full main-loop rate, so drivers for buttons/sensors can poll faster than Lua ticks.
- **Fixed-size buffers everywhere**: 8 extensions max, 8 events, 256-byte event data, 32-char event names. No dynamic allocation in the event path. This is correct for embedded but means you can't have a device with 12 peripherals or deep event payloads.
- **Single-slot callbacks (last registration wins)**: Simpler than multi-slot. No need for a vector of `std::function` — just one `std::function` per callback type. The tradeoff is you can't have two independent subscribers.
- **Extension owns the Lua module name**: `Extension::name()` returns the string used as `lua_setglobal`. This means the Lua namespace IS the C++ class identity — you can't register the same driver under two names, and the name is fixed at compile time.
- **Leftmost-base inheritance rule**: Because `LuaModule::method<>` uses `static_cast` from the stored `Extension*`, Driver must be the leftmost base in multi-inheritance. Documented as a hard rule with compile-time detection via `static_assert` on `is_base_of`. A footgun for new contributors but zero-overhead at runtime.

## Comparison notes

- vs. **MicroPython**: MicroPython is a full Python interpreter on ESP32. Resident is a thin Lua sandbox that expects to be remote-controlled. MicroPython is for developers writing firmware; Resident is for agents writing apps. The design constraint is inverted.
- vs. **Tasmota/ESPHome**: Both expose hardware to configuration/scripting, but are YAML/rule-based. Resident exposes hardware directly to a general-purpose language (Lua) with hot-reload, making it suitable for creative/artistic applications (the example apps are visual demos: flying toasters, wind particle sims, daisy chime).
- vs. **Toit**: Toit is another ESP32 language runtime with hot-reload, but is a custom language. Resident uses standard Lua 5.4, which has decades of tooling, documentation, and developer familiarity.
- vs. **WLED**: WLED is purpose-built for LED control. Resident is general-purpose — the shader template system shows it can do LED patterns, but it can also drive displays, read sensors, and respond to network events.
- vs. other agent-to-device systems: Most "agent controls device" systems use HTTP APIs or MQTT. Resident inverts this — the agent pushes code that runs locally on the device. This means network latency doesn't affect per-frame behavior (the Lua runs at 10 FPS locally) and the device works offline once the app is loaded.

## Critical analysis

**Strengths:**
- The C++ template machinery for Lua binding is elegant — minimal boilerplate per driver method, type-safe, zero runtime overhead. Much cleaner than the typical Lua C API boilerplate.
- The opt-in networking via `std::optional` is a clean design. Most embedded frameworks force networking always-on. Resident's "standalone mode" is genuinely useful — a device that just runs a pre-loaded Lua app with no WiFi.
- The agent plugin is the real innovation. Resident isn't just a Lua runtime — it's a platform for agent-driven device programming. The four-step bring-up guide (hardware → Resident → drivers → agent skills) is a complete workflow for adding new hardware to an agent's capabilities.
- The DEVICE-SKILL.md pattern is smart: a single Markdown file that documents the Lua surface for both humans and agents. The agent reads it to know what Lua functions are available; the local validator reads it to build a stub harness.

**Weaknesses:**
- The 8-extension limit is a reasonable embedded constraint but poorly discoverable — it's a silent overflow in the `Extensions` initializer_list constructor (`if (count >= MAX) break;`).
- The event ring buffer drops oldest events silently. For button presses this is fine (you want the latest), but for ordered event sequences it could lose state.
- The hand-rolled JSON parser in `processNextEvent()` is ~80 lines of char-by-char parsing that duplicates ArduinoJson's functionality. The justification (avoiding allocation in the hot path) is valid but the code is fragile — it doesn't handle escaped quotes in JSON strings or nested objects.
- No authentication beyond device ID. The README acknowledges this ("treat it like an API key for development") but it means Resident devices on the public relay are effectively open to anyone who knows the 8-char hex ID.
- The leftmost-base rule is a real footgun. The `static_assert` catches it at compile time, but the error message won't explain the fix to someone who doesn't already know the rule.

**Surprising choices:**
- Using Lua 5.4 (not 5.1/5.2/LuaJIT which are more common in embedded). The `fischer-simon/Esp32Lua` port handles the ESP32-specific build flags.
- The telemetry format uses a nested JSON string for `generationId` — it's JSON-inside-JSON that requires double-escaping. This is odd but works with the relay's protocol.
- The `onTransportsWillConnect` callback sets a default WS path of `/agents/<type>-agent/<id>` but all examples override it to `/devices/<id>`. This suggests the `/agents/` path is the "real" protocol and `/devices/` is a backwards-compatible alias.

## For AI/agent contexts

- **Memory architecture**: Stateless between app loads. Each `loadApp()` fully resets: old function refs are unreffed, globals cleared, extensions get `onAppReset()`. The only persistent state is driver hardware state (display framebuffer, LED state) and the timezone cache in ezTime's EEPROM. No app-to-app memory.
- **Agent topology**: Single-agent model. The Claude Code plugin is a tool the user invokes — it doesn't run autonomously. The device is a passive receiver of code; it doesn't initiate agent interactions.
- **Context management**: Lua source is the context. No chunking, no summarization — the entire app must fit in the Lua state's memory (PSRAM on ESP32, typically 2-8MB). The sandbox imposes no artificial size limit beyond what fits in PSRAM.
- **Tool/capability model**: Fixed by the driver set registered at construction time. An app running on an M5Stick sees `screen.*`, `imu.*`, `buzzer.*`, `button.*`. An app on a Feather sees `screen.*`, `led.*`, `battery.*`. The capability surface is static per device.
- **Prompt engineering**: The DEVICE-SKILL.md format is effectively a structured prompt for the agent — it tells the agent exactly what Lua calls are available, with argument shapes and examples. The create-app skill reads this and generates appropriate code.
