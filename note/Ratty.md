# Ratty

A GPU-rendered terminal emulator that supports inline 3D graphics as first-class terminal content. Built on the Bevy game engine, it defines the Ratty Graphics Protocol (RGP) — a custom APC escape-sequence protocol for registering, placing, animating, and deleting 3D objects anchored to terminal cells. The terminal surface itself can warp, rotate, and transform into a Mobius strip via real-time mesh deformation.

---

## Architecture

Ratty embeds a terminal emulator inside a Bevy ECS application (~8,500 lines Rust). The system schedule in [[/tmp/repo-orhun--ratty/src/plugin.rs]] defines the entire execution pipeline:

```
pump_pty_output → handle_keyboard_input → handle_mouse_input
→ handle_window_resize → apply_terminal_presentation
→ apply_inline_objects → redraw_soft_terminal
→ sync_inline_objects → sync_rgp_objects
→ apply_instance_brightness → animate_mobius_transition
→ animate_terminal_plane_warp → sync_asset_to_terminal_cursor
```

**Three key resource types** drive the system:

- **`TerminalRuntime`** (`src/runtime.rs`) — Owns the PTY (via `portable-pty`), the vt100 parser with custom callbacks, and the stdin writer. PTY output streams through an `mpsc::sync_channel(16)` to the ECS world.
- **`TerminalSurface`** (`src/terminal.rs`) — Wraps Ratatui's `Terminal<ParleyBackend>` for text buffer management plus an offscreen wgpu renderer that produces the terminal texture. Every frame renders text to RGBA via GPU, reads back to CPU, then uploads as a Bevy `Image`.
- **`TerminalInlineObjects`** (`src/inline.rs`) — Central registry mapping `object_id -> InlineObject` (Kitty raster images or RGP 3D objects) and `object_id -> InlineAnchor` (cell position, span, style). Intercepts APC sequences from the PTY byte stream before they reach the vt100 parser.

**Two cameras, three modes** — `src/scene/mod.rs` sets up a Camera2d (order 0) for flat mode and an orthographic Camera3d (order 1) for 3D modes. The terminal exists as both a 2D sprite and a textured 3D plane mesh. `TerminalPresentationMode` toggles between `Flat2d`, `Plane3d`, and `Mobius3d`.

**Two protocols, one dispatch** — `consume_pty_output()` in `src/inline.rs` strips APC sequences from the byte stream and routes them to either the Kitty graphics protocol parser (`src/kitty.rs`) or the RGP parser (`src/rgp.rs`).

## Key techniques

### Ratty Graphics Protocol (RGP)
A custom terminal protocol for inline 3D objects via APC escape sequences:
```
ESC _ ratty;g;<verb>;<key=value>... ESC \
```
Five verbs: `s` (support query), `r` (register asset), `p` (place at cell), `u` (update transform/style), `d` (delete). Supports path-based and chunked-payload-based registration, full 6-DOF transforms, animation toggles, color tinting, and brightness. The support query response advertises capabilities so applications can feature-detect.

Unlike the Kitty graphics protocol — which only handles raster images — RGP extends the concept to 3D geometry. The design is explicitly inspired by TempleOS's DolDoc inline document graphics.

### Terminal as deformable geometry
The terminal plane isn't flat. `plane_surface_z()` (`src/systems.rs`) applies an exponential radial warp:
```
z = -(exp(-radius * 9.0) * 360.0 + exp(-(radius - 0.22)^2 * 18.0) * 72.0) * warp_amount
```
The warp creates a central depression with a raised ring — like pressing your thumb into a rubber sheet. `mobius_surface_point()` implements a full Mobius strip parameterization with configurable twist. Both deformations are applied by mutating mesh vertex positions directly in `animate_terminal_plane_warp()`.

### Scroll-aware anchor tracking
Inline objects are anchored to cell positions, but cells move when the terminal scrolls. `infer_upward_scroll()` compares pre-PTY and post-PTY screen rows to find the shift delta, then `apply_scroll()` adjusts anchor positions. Objects scrolled off-screen are automatically cleaned up.

### Mesh extrusion for flat OBJ models
`extrude_mesh()` (`src/systems.rs:985`) takes a flat OBJ mesh and gives it thickness: duplicates vertices front/back at ±half depth, reverses windings for back faces, identifies boundary edges (referenced exactly once in the index buffer), and generates side quads. Skips meshes that already have Z-extent (won't extrude volumetric models).

### Per-instance brightness via material cloning
Rather than passing brightness to shaders, `apply_instance_brightness()` walks the ECS parent chain from each material-bearing entity up to an RGP root or Cursor root, extracts the brightness value, clones the material, adjusts `base_color` and `emissive` in linear space, and tags the entity with `BrightnessAdjusted` so it won't be processed again.

### Embedded asset model loading
3D models ship inside the binary via `rust_embed` (`CairoSpinyMouse.obj`, `Ferris.glb`, `SpinyMouse.glb`). The asset loader has three resolution tiers: explicit path → embedded fallback → runtime asset root. OBJ meshes are centered and normalized to a unit bounding box on load.

## Design decisions

**Optimized for visual playfulness over terminal performance.** The architecture (Bevy game engine, offscreen GPU render with CPU readback, deformable mesh surfaces) is overengineered for a terminal — but exactly right for a terminal-as-canvas. The 3D cursor model (spinning, bobbing, configurable OBJ/GLB), plane warping, and Mobius mode are features no production terminal has.

**Sacrificed raw throughput.** Every frame round-trips terminal content through the CPU: GPU render → readback → texture upload. This is fundamentally slower than native terminal renderers that write directly to the framebuffer. The 16ms redraw throttle and 33ms low-power update interval keep it manageable.

**Dual protocol support as future-proofing.** Supporting both Kitty graphics (for raster images) and RGP (for 3D objects) means Ratty works with existing Kitty-compatible apps while enabling novel features. The APC dispatch handles both transparently.

**Heavy dependency tree.** Bevy + wgpu + portable-pty + vt100 + ratatui + parley_ratatui + tobj + arboard + rust_embed is a lot of crates for a terminal emulator. Release builds use `lto = "fat"` and `codegen-units = 1`, trading build time for binary size and runtime performance.

**Config-driven cursor as a first-class feature.** The cursor model (OBJ/GLB with scale, offsets, spin/bob animation, brightness) is deeply configurable and rendered as a proper 3D entity in the Bevy scene, not just a terminal escape-code cursor.

## Comparison notes

**vs. Kitty**: Kitty popularized the inline image protocol and GPU rendering in terminals, but it treats the terminal as a 2D surface. Ratty treats the terminal surface itself as 3D geometry and adds a protocol for 3D objects. Kitty does GPU rendering for performance; Ratty does it for expressiveness.

**vs. Alacritty/WezTerm**: These are GPU-accelerated terminals optimized for throughput. They render the terminal grid to the screen as efficiently as possible. Ratty renders the grid to a texture and then applies it to a 3D plane — a fundamentally different rendering model that's slower but more flexible.

**vs. TempleOS DolDoc**: Ratty explicitly cites DolDoc as inspiration for inline document graphics. DolDoc embedded sprites and forms in a text document; RGP embeds 3D objects in a terminal. The lineage is direct but the implementation is modern (wgpu, glTF, OBJ, PBR materials).

**vs. Glyph Protocol**: Like Glyph, RGP is a custom terminal protocol — but Glyph focuses on semantics and structured data, while RGP focuses on 3D graphics placement.

## Tags

#tool #terminal #3d #graphics #rust #protocol #bevy

## Related pages

- [[10 Principles for Agent-Native CLIs]] — Ratty's RGP protocol design has parallels: terminal objects as first-class capability, not overlay hack
- [[Surf CLI]] — Another unconventional CLI tool; both push the boundaries of what a terminal can be
- No direct terminal emulator pages exist in this wiki yet. This is the first.

---
*Source: [[summary/ratty]]*
*Last updated: 2026-05-31*
