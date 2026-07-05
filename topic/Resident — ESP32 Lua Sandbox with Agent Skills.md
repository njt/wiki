# Resident — ESP32 Lua Sandbox with Agent Skills

Sandboxed Lua 5.4 runtime for ESP32 microcontrollers that lets AI coding agents write and deploy apps to physical devices over WebSocket. Hardware peripherals (displays, sensors, LEDs, buttons) are exposed to Lua through a ~100-line driver API; apps are compiled and hot-reloaded in-place without rebooting the device. Ships with a Claude Code plugin so agents can generate, validate, and push Lua apps from natural-language descriptions. Created by [[Stockyard|Jesse Vincent (obra)]].

## Architecture

~1,100 lines of C++ in a single `.cpp` file (`src/ResidentSandbox.cpp`), plus 7 small headers. The core class `Resident::Sandbox` composes three concerns: an optional Courier networking layer (WiFi + WebSocket), a Lua 5.4 VM, and an extension system for hardware drivers.

**Two-tier extension system** (`src/ResidentExtension.h`, `src/ResidentDriver.h`): `Extension` is the base — pure virtual `name()` returns the Lua global name, optional `begin()`/`update()`/`registerModule()`/`onAppReset()` hooks. `Driver` extends it with `sendEvent()` and `onAppRunning()`, queueing hardware events into the sandbox's ring buffer for delivery to Lua's `on_event(ctx, event)`.

**Lua binding via template trampolines** (`src/ResidentLuaModule.h`): The `LuaModule` builder lets drivers register C++ member functions as Lua-callable with zero per-method boilerplate. `Trampoline<C, &C::fn>::call` is a static `lua_CFunction` that pulls the `Extension*` from a Lua upvalue and `static_cast`s to the derived class — avoids `dynamic_cast` (which requires `-frtti`, disabled on Arduino/ESP32 builds).

**Opt-in networking via `std::optional<Courier::Config>`**: When `cfg.network` is set, the Sandbox constructs a `Courier::Client` internally, drives WiFi (WiFiManager captive portal on first boot) and WebSocket transport. When unset, the sandbox runs standalone — no WiFi code linked, `loop()` ticks Lua at 10 FPS unconditionally. This is a clean compile-time choice expressed as a runtime check.

**Event system**: Fixed 8-slot ring buffer. Driver events are serialized as compact JSON into 256-byte buffers; `processNextEvent()` does a hand-rolled char-by-char JSON parse to avoid allocations in the Lua call path.

**Agent plugin** (`tools/agent-plugin/`): Five Claude Code skills — `create-app` (natural language → Lua, reads DEVICE-SKILL.md), `validate-app` (local Lua interpreter under stub harness), `push-app` (sends to WebSocket relay), `write-device-skill` (interactively author device surface docs), `hello-resident` (liveness check). Distributed via Claude Code marketplace.

## Key techniques

- **Trampoline template without RTTI** — `LuaModule::method<C, &C::fn>()` stores `Extension*` as a Lua upvalue, casts back via `static_cast`. Type-safe via `static_assert<is_base_of<Extension, C>>`. Zero overhead at runtime, but requires Driver to be the leftmost base class in multi-inheritance.
- **Plain fn pointer instead of std::function for Driver events** — Keeps Driver allocation-free for embedded targets where static construction may happen before the heap.
- **PSRAM allocator for Lua** — On ESP32, all Lua allocations (`lua_newstate`) route through `heap_caps_malloc(MALLOC_CAP_SPIRAM)`, preserving internal SRAM for WiFi and driver buffers.
- **Runtime error rate limiting** — `on_tick` fires at 10 Hz. Errors are capped at 3 per burst, then require 5-second cooldown before reporting more via telemetry. `init` and `on_event` errors always report.
- **Shader template system** — `ShaderTemplateFn` converts key/value fields (e.g. `{"expr":"rgb(sin(t),0,0)"}`) into full Lua source. The `rgb()` function returns a negative packed integer to signal "this is a color" to template code.
- **Idempotent init via static helper** — `Extension::beginExtension(e)` checks a private `_begun` flag. User code can call it early (e.g. display init before sandbox), and Sandbox's own call is a safe no-op.

## Design decisions

**Optimized for embedded constraints**: Fixed-size buffers everywhere — 8 extensions max, 8 event slots, 256-byte event data, 32-char event names. No dynamic allocation in the event dispatch path. The 10 FPS Lua tick is chosen for display refresh cadence, not CPU limits.

**Single-slot callbacks (last registration wins)**: Simpler than multi-subscriber — one `std::function` per callback type, no vector allocation. The tradeoff is you can't have two independent subscribers for connection events.

**Single .cpp file**: All 1,125 lines of sandbox implementation in one compilation unit. Unusual by modern C++ standards, but pragmatic for ESP32 where each `.cpp` adds linker and flash overhead.

**Leftmost-base rule**: Because the trampoline template `static_cast`s from `Extension*` to the driver class, Driver must be the leftmost base in multi-inheritance. Documented as a hard rule with compile-time detection, but it's a footgun for new contributors.

**Device ID as authentication**: The 8-character hex chip MAC serves as the device's identity on the public relay. The README acknowledges this is insecure ("treat it like an API key for development") but sufficient for the prototyping use case.

## Comparison notes

Unlike [[MimiClaw]] (which puts an AI agent ON an ESP32 via Telegram), Resident puts a programmable Lua runtime on the ESP32 and lets an agent control it from the outside. MimiClaw is agent-on-device; Resident is agent-controls-device. The Resident model means the device doesn't need internet-facing connectivity beyond a WebSocket, and the agent can be any model running anywhere.

Unlike Tasmota/ESPHome (YAML/config-driven), Resident exposes hardware to a general-purpose language with hot-reload, making it suitable for creative/artistic applications — the example apps include a flying toasters screensaver, a wind particle simulation, and a musical daisy chime.

Unlike [[Claude Lamp]] (which lets Claude control a single LED via Bluetooth), Resident is a general platform — any hardware peripheral can be exposed to Lua with a ~50-line driver subclass. The agent plugin generalizes the "agent controls hardware" pattern.

The DEVICE-SKILL.md pattern — a single Markdown file documenting the Lua surface for both humans and agents — is the most transferable design idea. The agent reads it to know what Lua calls are available; the local validator reads it to build stub harnesses. It's a spec-as-interface between human hardware knowledge and agent code generation.

## Critical analysis

The C++ template machinery for Lua binding is the strongest technical element — cleaner than the typical Lua C API boilerplate, type-safe, zero runtime overhead. The opt-in networking via `std::optional` is also well done: most embedded frameworks force networking always-on, but Resident's standalone mode is genuinely useful.

The hand-rolled JSON parser in `processNextEvent()` is the weakest link — ~80 lines of char-by-char parsing that doesn't handle escaped quotes or nested objects. The justification (avoiding allocation in the hot path) is sound, but the implementation is fragile.

The 8-extension limit is a silent overflow (`if (count >= MAX) break;`) — no warning when you exceed it. The event ring buffer also drops events silently when full. Both are defensible for embedded targets but poorly signposted.

The agent plugin is the real innovation. Resident isn't just a Lua runtime — it's a platform for agent-driven device programming. The four-step bring-up guide (hardware → Resident → drivers → agent skills) is a complete workflow for adding new hardware to an agent's capabilities.

#agent-platform #embedded #esp32 #lua #sandbox #claude-code-plugin

*Source: https://github.com/inanimate-tech/resident, ingested 2026-05-22*
