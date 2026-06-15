---
url: https://phoenix-live-view.hexdocs.pm/welcome.html
title: "Phoenix LiveView — Welcome"
author: Phoenix Core Team / Chris McCord
date_fetched: 2026-06-15
date_published: 2019 (launch), continuously updated
---

# Phoenix LiveView — Welcome

Official HexDocs welcome page for Phoenix LiveView, the server-centric library for building real-time, rich user experiences using server-rendered HTML. Ships by default with Phoenix applications.

## What It Is

LiveViews are processes that receive events, update their state, and render updates to a page as diffs. The programming model is **declarative** — rather than imperatively instructing the DOM what to change on each event, you describe the state and let the framework handle diffs.

> "Events in LiveView are regular messages which may cause changes to the state. Once the state changes, the LiveView will re-render the relevant parts of its HTML template."

## Lifecycle: mount → render → handle_event

### mount/3
Called when the LiveView starts. Receives request parameters, session (typically signed/encrypted cookie data), and the socket. Initializes state by assigning to the socket; returns `{:ok, socket}`. Invoked inside a spawned LiveView process after the WebSocket connects (following initial static render).

### render/1
Receives the socket's `assigns` and returns HTML content using the `~H` sigil (HEEx templates — HTML+EEx). HEEx is "an extension of Elixir's builtin EEx templates, with support for HTML validation, syntax-based components, smart change tracking."

### handle_event/3
Matches DOM events sent from the client (triggered by bindings like `phx-click`). Receives event name, params, and socket. Returns `{:noreply, socket}` with updated state — no explicit reply beyond the re-rendered diff.

## Key Concepts

### Assigns and HEEx
The socket holds **assigns** — immutable Elixir data structures. Templates interpolate with `{@temperature}` syntax. HEEx components (`<.live_component>`, `<>`) enable function-based and stateful component composition.

### Bindings
DOM attributes like `phx-click="inc_temperature"` trigger server-side events. LiveView also supports **JS commands** via `Phoenix.LiveView.JS` that execute directly on the client without reaching the server — useful for UI toggles (dropdowns, tabs).

### Navigation
Browser pushState API-based navigation without full page reloads. **Patch** updates URL within the same LiveView; **navigate** moves to a new LiveView.

### First Render Is Static
"Every LiveView is first rendered statically as part of a regular HTTP request" — fast First Meaningful Paint, SEO-friendly. After that initial HTTP response, JS client establishes a persistent WebSocket and `mount/3` runs again inside a spawned Elixir process.

## Three Extension Mechanisms

1. **Function Components** (`Phoenix.Component`) — plain functions receiving assigns and returning `~H` templates. Can be paired with `attach_hook/4` for shared event handling.

2. **LiveComponents** (`Phoenix.LiveComponent`) — own `mount/1` and `handle_event/3` callbacks plus separate state with change tracking, but run in the **same process** as the parent LiveView. More complex; errors affect the whole view.

3. **Nested LiveViews** (`live_render/3`) — run in a **separate process** with error isolation. Child crash doesn't affect parent. Requires a unique `:id`; re-mounting requires a new ID. "A slightly expensive abstraction if all you want is to compartmentalize markup or events."

## Architecture

- **Process model**: Each LiveView is an Elixir process. State is "nothing more than functional and immutable Elixir data structures."
- **Event sources**: Client/browser via bindings, or internal application messages via `Phoenix.PubSub`.
- **Diffing**: After state changes, "LiveView will re-render the relevant parts of its HTML template and push it to the browser, which updates the page in the most efficient manner."
- **Efficiency**: Persistent connections reduce overhead vs. "stateless requests that have to authenticate, decode, load, and encode data on every request."

## Generators
- `mix phx.gen.live` — full CRUD LiveViews from database up
- `mix phx.gen.auth` — authentication with LiveView support

## Code Example

A thermostat LiveView demonstrates the full cycle: module uses `MyAppWeb, :live_view`, defines mount/render/handle_event, wired into router with `live "/thermostat", ThermostatLive`. Client-side imports `LiveSocket` from `phoenix_live_view`, initializes with CSRF token, calls `liveSocket.connect()` to establish WebSocket.
