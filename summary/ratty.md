---
url: https://github.com/orhun/ratty
title: "Ratty: A GPU-Rendered Terminal Emulator with Inline 3D Graphics"
author: Orhun Parmaksız
date_fetched: 2026-05-31
date_published: 2025-09-28 (initial)
---

# Ratty — Deep Analysis

## Overview

Ratty is a GPU-rendered terminal emulator built in Rust that supports inline 3D graphics objects as first-class terminal content. Unlike conventional terminals that render text to a 2D grid, Ratty uses the Bevy game engine as its rendering backend, treats the terminal as a textured 3D plane that can warp, rotate, and transform, and defines its own custom terminal protocol (RGP — Ratty Graphics Protocol) for applications to embed 3D objects inline with text.

Version: 0.4.1. License: MIT. ~8,500 lines of Rust.

## File-by-file analysis

### Entry point: `src/main.rs` (74 lines)
- Parses CLI args via `clap`
- Loads config from `config/ratty.toml` via `AppConfig::load_from_path`
- Spawns a `TerminalRuntime` (PTY + parser)
- Creates a `TerminalSurface` (Ratatui + GPU render state)
- Constructs a Bevy `App` with `DefaultPlugins` (Window, Asset, etc.), sets window transparency and resolution
- Adds `TerminalPlugin` and runs the app

### Architecture core

**`src/plugin.rs` (58 lines)** — The Bevy plugin that wires all ECS systems in a carefully ordered schedule:
```
Startup → setup_scene
Update:
  pump_pty_output
  handle_keyboard_input
  handle_mouse_input
  handle_window_resize
  apply_terminal_presentation (after keyboard + mouse)
  apply_inline_objects (after apply_terminal_presentation)
  redraw_soft_terminal (after mouse + pty)
  sync_inline_objects (after redraw)
  sync_rgp_objects (after sync_inline_objects)
  apply_instance_brightness (after sync_rgp)
  animate_mobius_transition
  animate_terminal_plane_warp
  sync_asset_to_terminal_cursor (after redraw)
```

**`src/runtime.rs` (389 lines)** — PTY lifecycle and terminal parser:
- Uses `portable-pty` to spawn a native shell process
- Uses `vt100` crate as the terminal emulator parser with custom `TerminalParserCallbacks`
- Custom callbacks handle: CSI device attribute queries, cursor position reports, kitty keyboard protocol enable/disable/pop, xterm `modifyOtherKeys` mode tracking, HVP→CUP normalization
- Any unhandled CSI/escape sequences are logged with deduplication via `HashSet` so repeated warnings don't flood
- PTY output is read on a dedicated thread, sent via `mpsc::sync_channel(16)` to the ECS world
- Input writes go through `Arc<Mutex<Box<dyn Write + Send>>>`
- `TerminalRuntime::resize()` resizes both the PTY and the vt100 screen simultaneously

**`src/terminal.rs` (442 lines)** — The "soft terminal" surface:
- Wraps `ratatui::Terminal<ParleyBackend>` for text buffer management
- Uses `parley_ratatui` for GPU text rendering (Vello-based)
- `TerminalSurface` owns a `TerminalRenderer`, an optional `OffscreenGpu` (wgpu device/queue/texture for rendering the terminal to an RGBA buffer), and front/back image handles
- `sync_image()` renders the terminal buffer to an RGBA pixmap via wgpu offscreen → uploads to Bevy's `Image` asset
- `TerminalWidget` implements `ratatui::Widget` — renders a vt100 screen with ANSI colors, selection, cursor, and font styling
- Redraw is throttled to 16ms (60fps equivalent) via `TerminalRedrawState`

**`src/rendering.rs` (317 lines)** — Image sync helpers:
- `sync_terminal_debug_image()` renders a debug visualization of the terminal grid (colored cells, outlines, cursor highlight)
- `CellDebugImageRenderer` draws individual cell rects with color blending for inactive cells, bold outlines, underline indicators
- `sync_plane_texture()` updates the base color texture of all terminal plane materials to the latest rendered terminal image
- Used by the 3D presentation modes where the terminal is rendered as a texture on a 3D plane

**`src/scene/mod.rs` (527 lines)** — Scene setup and 3D presentation:
- Creates dual cameras: Camera2d (order 0) for 2D mode, Camera3d (order 1, orthographic) for 3D mode
- Builds a foreground terminal plane (front) and background plane (back), both as `terminal_plane_mesh(32, 20)` — subdivided grids with UV coordinates
- Front plane gets the live terminal texture; back plane gets a slightly darkened background-colored image
- Sets up three light sources: two point lights + one directional light for illuminating 3D objects
- Three presentation modes defined in `TerminalPresentationMode`: `Flat2d`, `Plane3d`, `Mobius3d`
- Camera state (`TerminalPlaneView`): yaw, pitch, zoom, camera offset, rotation/panning flags + last cursor positions
- `apply_terminal_presentation()` flips visibility between sprite/plane for 2D/3D modes, applies camera transforms, handles Mobius culling (double-sided vs. backface culling)

**`src/scene/mobius.rs` (213 lines)** — Animated transition into/out of Mobius strip view:
- Three-phase entry: zoom-out (0.2s) → morph (0.9s) → done
- Two-phase exit: view-reset (0.2s) → un-morph (0.9s)
- Stores source camera state before entering, restores it on exit
- Uses `ease_in_out()` (cubic hermite spline) for smooth interpolation
- `current_zoom()`, `current_yaw()`, `current_pitch()`, `current_camera_offset()` provide interpolated values for the render systems

**`src/systems.rs` (1,412 lines)** — The heart of the application. Contains all Bevy ECS systems:

- `pump_pty_output()` — Drains PTY output, feeds through `TerminalInlineObjects::consume_pty_output()`, detects scroll via `infer_upward_scroll()` (compares screen rows before/after to find shift amount), updates scroll-tracked anchors
- `redraw_soft_terminal()` — Draws terminal via Ratatui widget → syncs GPU image → syncs plane textures → spawns cursor model on first frame
- `sync_inline_objects()` — Clears stale inline entities, rebuilds Kitty image sprites (2D) and plane-attached meshes (3D), spawns RGP objects
- `sync_rgp_objects()` — Positions `TerminalRgpObject` roots based on anchor data, computes animated transforms (spin, tilt, bob), projects onto warped/mobius surfaces
- `apply_instance_brightness()` — Walks entity hierarchy from material-bearing entities up to RGP/Cursor roots, clones and brightness-adjusts materials, tags with `BrightnessAdjusted`
- `animate_terminal_plane_warp()` — Mutates terminal plane mesh vertices via `apply_plane_warp()` using radial warp math (exponential core + ring formulas)
- `animate_mobius_transition()` — Advances transition timer, restores camera state when transition completes
- `sync_asset_to_terminal_cursor()` — Positions the 3D cursor model at the terminal cursor position, with spin + bob animation, projected onto the correct surface

Key mathematical functions in systems.rs:
- `plane_surface_z()` — Warp deformation: `-(exp(-radius*9.0) * 360.0 + exp(-(radius-0.22)^2 * 18.0) * 72.0) * pulse`
- `plane_surface_point()` — Dispatches to flat/plane/mobius based on mode
- `mobius_surface_point()` — Parameterizes the Mobius strip: `angle = (x+0.5)*TAU`, `radius = 0.24 + warp*0.015`, `width = y * (0.42 + warp*0.04)`, computes `cos_half`, `sin_half` of `angle*0.5*twist`

**`src/rgp.rs` (286 lines)** — Ratty Graphics Protocol parser:
- Parses APC sequences starting with `\x1b_ratty;g;`
- Supports 5 verbs: `s` (support query), `r` (register), `p` (place), `u` (update), `d` (delete)
- Key-value semicolon parsing with 20+ recognized keys
- Support reply advertises capabilities: v=1, fmt=obj|glb, path=1, payload=1, chunk=1, anim=1, depth=1, color=1, brightness=1, transform=1, update=1
- Color parsing: hex RGB with optional `#` prefix
- Boolean parsing: "1"/"true" or "0"/"false"

**`src/inline.rs` (630 lines)** — Inline object registry and APC dispatch:
- `TerminalInlineObjects` holds the central registry: `objects: HashMap<u32, InlineObject>`, `anchors: HashMap<u32, InlineAnchor>`
- `consume_pty_output()` scans PTY bytes for APC sequences, intercepts them before they reach the vt100 parser
- APC handling dispatches to Kitty protocol (via `kitty::KittyParserState`) or RGP
- Scroll tracking: `infer_upward_scroll()` compares screen row contents to detect scroll distance, then adjusts anchor positions
- Overlap removal: when placing a new object, `remove_objects_at()` checks AABB overlap and removes any existing objects
- `normalize_hvp_sequences()` converts HVP (`f`) to CUP (`H`) since vt100 handles one but not the other
- Payload chunking: `PendingRgpPayload` accumulates base64 chunks until `more=0`, then assembles and loads
- Two inline object types: `KittyImage` (raster RGBA) and `RgpObject` (OBJ mesh or glTF scene)

**`src/model.rs` (375 lines)** — Object asset loading:
- `EmbeddedObjects` via `rust_embed` for bundled assets (`assets/objects/`)
- `load_object_source()` supports three resolution tiers: (1) expanded path, (2) embedded asset fallback, (3) runtime asset root
- OBJ loading uses `tobj` with triangulation, single index, ignores lines/points
- Loaded meshes are centered and normalized: subtract center, divide by max_extent → unit bounding box
- glTF/GLB files are copied to the runtime asset root so Bevy's asset server can load them
- `load_object_source_from_bytes()` handles base64-decoded payloads from RGP registration
- Fallback: if no model resolves, spawns a `Cuboid` as cursor

**`src/mouse.rs` (532 lines)** — Mouse input handling:
- `TerminalSelection` with begin/update/end/clear state machine
- `SelectionBounds` with row-bounded contains() logic
- Three input modes depending on presentation:
  - Flat2d + mouse protocol: forwards mouse events to the PTY (SGR or default encoding)
  - Plane3d/Mobius3d: left-drag = rotate, right-drag = pan, scroll = zoom
  - Flat2d (no mouse protocol): left-drag = text selection, scroll = scrollback
- Mouse event encoding: SGR (`\x1b[<code;col;rowM/m`) vs. default (X10-style)
- `position_to_cell()` divides cursor position by cell dimensions

**`src/keyboard.rs`** — Keyboard input handling:
- Configurable key bindings via `KeyBindingConfig` (key + modifiers + action)
- Actions: Copy, Paste, ScrollPageUp/Down, ScrollUp/Down, Toggle3DMode, ToggleMobiusMode, IncreaseWarp, DecreaseWarp, IncreaseFontSize, DecreaseFontSize, ResetFontSize
- Kitty keyboard protocol support: encodes keys with modifier state for apps that request enhanced key reporting
- Clipboard: uses `arboard` crate with Wayland + X11 support
- `TerminalClipboard` as non-send resource

**`src/config.rs` (491 lines)** — Configuration parsing from TOML:
- `AppConfig` with sections: window, terminal, shell, env, bindings, font, theme, cursor
- Config resolution priority: system config dir (via `etcetera` crate) → local `config/ratty.toml`
- Hex color deserialization: custom `deserialize_hex_color` function
- Path resolution: relative paths from config file directory
- Default Tokyonight-like theme (foreground #dcd7ba, background #1f1f28)

**`widget/src/lib.rs` (374 lines)** — Companion crate for applications to emit RGP sequences:
- `RattyGraphic` struct wrapping `RattyGraphicSettings` with builder pattern
- Generates escape sequences: `register_sequence()`, `place_sequence(area)`, `update_sequence()`, `delete_sequence()`
- Payload-based registration: `register_payload_sequences()` splits base64 into 3072-byte chunks
- Implements `ratatui::Widget` — renders the place sequence into a terminal buffer at a specific cell
- ObjectFormat: Obj or Glb, inferred from file extension

## Architecture pattern

**Game engine as terminal renderer.** This is the most unusual architectural choice. Rather than building a custom renderer or using a GUI toolkit, Ratty embeds the terminal inside a Bevy ECS application. The terminal surface exists as a textured 3D plane, and terminal content updates flow through Bevy's ECS system schedule.

**ECS as event loop.** All terminal behavior (PTY pumping, keyboard/mouse handling, rendering, object sync, animation) is organized as Bevy systems with explicit ordering dependencies. The system order in `plugin.rs` defines the entire execution pipeline — there's no manual event loop.

**Two render targets, one buffer.** The terminal is rendered twice: (1) as a GPU-rendered Ratatui buffer that becomes a Bevy `Image` texture, and (2) as a debug/imposter pixel grid for the back plane in 3D mode. Both come from the same vt100 screen state.

**Protocol interception in-band.** Inline objects are controlled via APC escape sequences embedded in the terminal byte stream — the same transport as regular terminal I/O. `consume_pty_output()` strips these sequences before they reach the vt100 parser, so they're invisible to the shell/TUI running inside the terminal.

**Dual protocol support.** Ratty supports both the Kitty graphics protocol (for raster images) and its own RGP (for 3D objects). Both are parsed from the same APC handler dispatch.

### Project size
- `src/main.rs`: 73 lines
- `src/lib.rs`: 23 lines (module declarations + docs)
- `src/systems.rs`: 1,412 lines (largest file — core ECS systems)
- `src/scene/mod.rs`: 527 lines
- `src/mouse.rs`: 532 lines
- `src/config.rs`: 491 lines
- `src/inline.rs`: 630 lines
- `src/terminal.rs`: 442 lines
- `src/runtime.rs`: 389 lines
- `src/rendering.rs`: 317 lines
- `src/rgp.rs`: 286 lines
- `src/model.rs`: 375 lines
- `src/scene/mobius.rs`: 213 lines
- `widget/src/lib.rs`: 374 lines
- Total: ~8,500 lines of Rust across the application

## Key techniques

### 1. The Ratty Graphics Protocol (RGP)
A custom terminal protocol for registering, placing, updating, and deleting 3D objects inline with terminal text. Uses APC escape sequences (`ESC _ ratty;g;<verb>;<key=value>... ESC \`). Supports:
- Path-based asset registration (files on disk)
- Payload-based registration (base64-encoded bytes, chunked across multiple sequences for large assets)
- Positional placement anchored to terminal cell coordinates
- Full transform control: translation, rotation, non-uniform scale, extrusion depth
- Animation toggle, color tinting, brightness

This is genuinely novel. The only comparable protocol is the Kitty graphics protocol, which only handles raster images. RGP extends the concept to 3D geometry.

### 2. Terminal as a deformable 3D surface
The terminal isn't just a flat plane — it can warp (exponential radial deformation) or transform into a Mobius strip. The math in `plane_surface_z()` and `mobius_surface_point()` implements this in real time by mutating mesh vertex positions at 60fps.

### 3. Scroll-aware anchor tracking
Inline objects are anchored to terminal cells, but when the terminal scrolls, those cells move. Ratty detects scroll by comparing pre-PTY and post-PTY screen rows to find the shift amount, then adjusts object anchors accordingly. Objects that scroll off-screen are automatically cleaned up.

### 4. Mesh extrusion for flat 3D models
`extrude_mesh()` in systems.rs takes a flat OBJ mesh and adds thickness by: (1) duplicating vertices at +half and -half depth, (2) reversing winding order for back faces, (3) detecting boundary edges (those referenced exactly once) and adding side quads. It skips meshes that already have Z-variance (won't extrude 3D models).

### 5. Material brightness via ECS hierarchy walk
Rather than storing brightness per-object and recomputing it every frame in shaders, `apply_instance_brightness()` walks the ECS parent chain from each material-bearing entity up to find its RGP root or Cursor root, extracts the brightness value, clones the material, adjusts `base_color` and `emissive` in linear space, and stamps the entity with a `BrightnessAdjusted` marker for single-processing.

### 6. Offscreen GPU rendering for terminal text
The terminal text is rendered via `parley_ratatui` (which uses Vello/WGPU) to an offscreen texture, read back to CPU RGBA bytes, then uploaded as a Bevy `Image` asset that's used as a texture on the terminal plane. This is a CPU round-trip for every frame but enables the terminal texture to participate in Bevy's rendering pipeline.

### 7. Deduplication of unhandled escape sequences
`TerminalParserCallbacks` uses `HashSet<String>` to log each unhandled CSI/escape sequence exactly once, preventing log spam from TUI programs that emit unknown sequences repeatedly.

## Design decisions

### Optimized for: visual experimentation
Ratty prioritizes making the terminal visually playful — 3D cursors, plane warping, Mobius strip mode, object animation. This is a terminal for people who want terminals to be more than text grids. The architecture (Bevy ECS, offscreen GPU rendering, deformable mesh surfaces) is overengineered for a terminal emulator but exactly right for a terminal-as-canvas.

### Sacrificed: raw terminal performance
Every frame round-trips terminal content through the CPU (offscreen GPU render → readback → upload). This is fundamentally slower than a native terminal renderer that writes directly to the frame. The 16ms redraw throttle and 33ms low-power update interval keep it manageable, but this isn't going to beat Alacritty or Kitty on throughput.

### Sacrificed: terminal compatibility
Ratty doesn't implement the full xterm control sequence set — unhandled sequences are logged and ignored. The parser handles the most common ones (cursor positioning, device attributes, mouse protocol, kitty keyboard, modifyOtherKeys) plus HVP→CUP normalization, but TUI applications using obscure sequences will degrade.

### Sacrificed: startup simplicity
The combination of Bevy (game engine with its own asset pipeline, windowing, and render graph), wgpu (GPU abstraction), portable-pty, vt100, ratatui, and parley_ratatui makes for a heavy dependency tree. `Cargo.lock` is substantial, and the release binary with LTO fat linking will take significant build time.

### Smart: config-driven cursor model
The cursor can be any OBJ/GLB model with configurable scale, offsets, spin speed, bob amplitude, and per-frame brightness. The fallback to a cube is clean. Embedded assets (via `rust_embed`) ship the default 3D spiny mouse cursor in the binary.

### Smart: dual-protocol inline objects
Supporting both Kitty graphics protocol and RGP means Ratty works with existing Kitty-compatible applications AND enables the novel 3D features. The protocol dispatch in `consume_pty_output()` handles both transparently.

## Core abstractions

1. **`TerminalRuntime`** — The PTY + parser. Owns the shell process, the byte stream, and the vt100 screen state.
2. **`TerminalSurface`** — The rendered terminal as a texture. Owns the Ratatui terminal, GPU renderer, and image handles.
3. **`TerminalInlineObjects`** — The object registry. Maps object IDs to inline objects (Kitty images or RGP 3D objects) and their terminal cell anchors.

## Innovation points

- **First terminal emulator to support inline 3D graphics as a protocol.** RGP defines a complete lifecycle (register → place → update → delete) for 3D objects in terminal space.
- **Terminal as a 3D surface with programmable deformation.** The plane warp and Mobius strip modes are unique — no other terminal treats the text surface as geometry to be transformed.
- **Game engine as terminal substrate.** Using Bevy for a terminal emulator is unconventional and enables features that would be impractical with a traditional immediate-mode or retained-mode renderer — specifically the 3D object rendering, lighting, and camera controls.
- **Object brightness via material cloning in ECS.** Rather than property-based dimming in shaders, the system clones materials per-instance and adjusts them in linear color space. This is a Bevy-idiomatic approach that avoids global state mutations.
