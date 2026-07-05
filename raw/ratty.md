---
url: https://github.com/orhun/ratty
date_fetched: 2026-07-05
backfilled: true
---

**Ratty: A GPU-rendered terminal emulator with inline 3D graphics** 🧀

Inspired by TempleOS | Built with Rust & Ratatui


## ratty-demo-with-audio.mp4

"Rodent-obsessed developer creates Ratty to bring 3D graphics to the command line" - The Register

"This New Terminal is Absurd (But Totally Fun)" - It's FOSS

"10 weird OSS projects you need right now... " - Fireship

"Your terminal can render in 3D now, and Rust made it surprisingly usable" - MakeUseOf

- Spinning rat cursor (customizable)
- Traditional 2D and new 3D mode!
- Inline 3D objects
- GPU-backed text rendering
- Image support (via Kitty Graphics Protocol >:()


📚 Read the behind the scenes blog post here!

Ever wondered what's *behind* the terminal? Press `Ctrl`+`Alt`+`Enter`!

## ratty-3d-with-audio.mp4

Requirements:

- A GPU / graphics stack supported by Bevy and wgpu
- Melted cheese (optional but recommended)

`cargo install ratty``sudo pacman -S ratty`See the Nix packaging docs for flake, NixOS, and Home Manager usage.

`nix run github:orhun/ratty`Prebuilt binaries are available on the GitHub releases page for direct download.

Requirements:

- Rust toolchain with Cargo
- on Bazzite / Bluefin: `sudo rpm-ostree install gcc fontconfig-devel wayland-devel`(then reboot)
- on Debian / Ubuntu: `sudo apt-get update ; sudo apt-get install gcc pkgconf libfontconfig-dev libwayland-dev`
- on Fedora: `sudo dnf install gcc fontconfig-devel wayland-devel`

`cargo install --git https://github.com/orhun/ratty`The default configuration file is available in `config/ratty.toml`.

You can copy this file to `$HOME/.config/ratty/ratty.toml` and customize it.

```
[cursor.model]
path = "CairoSpinyMouse.obj"
scale_factor = 6.0
brightness = 0.5
x_offset = 0.5
plane_offset = 18.0
visible = true
[cursor.animation]
spin_speed = 1.4
bob_speed = 2.2
bob_amplitude = 0.08
```
For `cursor.model.path`, Ratty supports both `.obj`, `.glb`, and `.stl` assets.

Other useful cursor fields are:

- `scale_factor`: scales the model relative to the terminal cell size
- `brightness`: adjusts the cursor material brightness
- `x_offset`: shifts the cursor model horizontally inside the cell
- `plane_offset`: pushes the cursor away from the warped terminal surface in 3D mode
- `visible`: show the custom 3D cursor model instead of only the terminal cursor

| Key | Action | 
|---|---|
| Ctrl+Alt+C | Copy selection | 
| Ctrl+Alt+V | Paste clipboard | 
| Ctrl+Alt+Enter | Toggle 2D / 3D mode | 
| Ctrl+Alt+M | Toggle Mobius mode | 
| Ctrl+Alt+Up | Increase warp | 
| Ctrl+Alt+Down | Decrease warp | 
| Alt+PageUp | Scroll one page up | 
| Alt+PageDown | Scroll one page down | 
| Alt+Up | Scroll one line up | 
| Alt+Down | Scroll one line down | 
| Ctrl+= | Increase font size | 
| Ctrl+- | Decrease font size | 
| Ctrl+Alt+0 | Reset font size | 

Ratty uses its own protocol, the Ratty Graphics Protocol, to place inline 3D objects in terminal space.

RGP supports:

- registering `.obj`,`.glb`, and`.stl`assets by path
- placing them at terminal cell anchors
- animation, scale, color, depth and other attributes

There is a Ratatui widget called `ratatui-rgp` available in
`widget/` if you want to build your own terminal applications that involve inline 3D objects.

Places a single oversized rat directly in your terminal:

## ratty-big-rat-with-audio.mp4

TempleOS-inspired document demo with editable text and embedded inline 3D objects:

## ratty-document-with-audio.mp4

Split-pane drawing demo with a 2D canvas on the left and a live 3D preview on the right:

## ratty-draw-with-audio.mp4

Interactive 3D Rubik's cube demo:

## ratty-rubiks-cube.mp4

Here are some applications explicitly built around Ratty's Graphics Protocol:

Terminal CAD:

## ratscad-demo.mp4

Endless runner built for Ratty:

## ratty-run.mp4

A blazingly fast serial monitor with plotter TUI and 3D telemetry

## 2026-05-23_18-01-42.mp4

The terminal surface currently uses `ratatui` for the UI buffer,
`parley_ratatui` for text shaping/rendering
and Bevy for scene presentation.

Current workflow:

- Ratatui buffer on CPU
- Parley/Vello renders on GPU
- Read back RGBA to CPU
- Copy into Bevy image
- Bevy presents that image in 2D and 3D

Terminal drawing is GPU-rendered through Parley/Vello, but the main terminal image still crosses back through CPU memory before Bevy presents it. This is a GPU-powered bridge, not a fully GPU-resident shared-texture path.

If the project later moves to a fully GPU-resident path, that will require a dedicated Bevy render integration that renders into a Bevy-owned texture on Bevy's render-world device instead of using the current readback bridge.

- *"This is like a legitimately cool project but also I just spent like 20 minutes adjusting the config for the rat spinning to see him spin faster and more erratically and it cracked me up"*- @vimlena.com

## bluesky-video-1777558230431.mp4

- 
*"These kinds of experiments are where creativity is born."*- @Coko7
- 
*"No comments. Just support."*- @Raphamorim (creator of Rio terminal)

## Screencast.from.2026-05-04.12-11-50.webm

All code is licensed under The MIT License.

🦀 ノ( º \_ º ノ) - respect crables!

Ratty logo designed by @Strophox & @Harunocaksiz

Copyright © 2026, Orhun Parmaksız

The author does not have a rat under the hat!
