---
url: https://html.moldandyeast.com/?open=shop
date_fetched: 2026-10-24
---

# Shop — html.moldandyeast.com (how it's built)

Source: https://html.moldandyeast.com/?open=shop — the 'Shop' artifact (https://html.moldandyeast.com/a/shop), a single dependency-free HTML file of 696,293 bytes: two inline module scripts (145 KB and 385 KB), 71 KB of CSS, and 35 text/plain preset programs. This archive is assembled from the site's llms.txt, the Shop index entry, the comments in the app's CSS and scripts in file order, and two of the preset programs.

## The site (from llms.txt)

# html.moldandyeast.com

> A library of single-file, dependency-free HTML artifacts. There are no prompts here: download a file and hand it to your agent as the reference ("like this, but…").

MCP server (read-only, no auth): https://html.moldandyeast.com/mcp — tools: search_artifacts, get_artifact, list_tags
Full index (JSON): https://html.moldandyeast.com/search-index.json

Shop's index entry: "Shop is a design canvas built on code, where you and AI agents design side by side and every edit stays real, reviewable code." Added 2026-10-21.

## Source comments, in file order

Layout: two panes, IDE left (code · agent · grammar), canvas right with floating Layers and Inspector.
Look: black and white, Arial, #1a1a1a ink, 1px light-grey hairlines, flat surfaces.

IR + graph + export views

black & white, Arial, hairlines

v2 workspace: docked sidebars like Paper / Figma

each part keeps its own column: a closed (display:none) panel must not let the canvas slide into its 0-wide column

layer rows: quieter selection, lock, drag & drop

menus: context menu, zoom menu, export

smart guides, measurements, marquee

one selection colour in both themes, readable on black type and white boards

Lab: semantic UI + learned taste

co-design: agents on the canvas

toasts sit under the toolbars, clear of the agent bar at the bottom of the canvas

a checked switch keeps a visible knob in both themes

co-design: review panel never taller than a third of the view; narrow docks drop the meta line into the panel

Trellis core — grammar → IR → DAG → runtime

small utils

Token namespaces follow Tailwind v4 / Paper: --color-*, --spacing-*, --radius-* …

icons (24×24, stroked)

placeholder imagery that never leaves the page

source locations for canvas ↔ code linking

Grammar: N is the one builder node. Everything is a fluent chain.
Any value may be a lambda (c => …). Lambdas that read c.s (state)
stay live at runtime; the rest are folded at build time.

`.size(n)` on a Text node means font size; on frames it means width/height.

primitives

combinators: they are just structure lambdas with a label

components: named factories; instances keep their name and args

design document

Compile: Design + param values → IR (flat node table) + DAG

evaluation context: dependency tracking through proxies

style value normalisation (lambda results land here too)

build: N → IR nodes

Design-tool sizing semantics (Paper/Figma): a fixed size on the main axis is fixed, it never flex-shrinks.

Preformatted text is auto-width in Paper: it hugs its content instead of stretching.

paths: stable, human-readable addresses used by canvas edits

a "/" inside a layer name would split its path, so it reads as "∕" (division slash) there

edits made on the canvas, written back into code as edits({...})

Edits apply in rounds: a layer added or moved in one round can be the target of edits in the next.

$parent names the target by its edit key; fall back to its path now, then its path in code

a moved layer keeps answering to its old path for the rest of this compile

Paper rounds text line height up to whole pixels; do the same so boxes match.

runtime: the same lambdas, re-run when state changes

* keep runtime state across a recompile (e.g. a slider moved during Play)

* run an event or tick handler with a writable state, then propagate

Renderer: IR → live DOM (the canvas is real HTML/CSS, like Paper)

Paper gives single-line text layers whole-pixel widths; hugging parents grow with them.

only hugging text snaps; stretched or filling text takes its width from the parent

Exporters: Paper HTML, standalone HTML, JSX, IR JSON, builder code

* HTML that Paper's write_html understands: inline styles, flex, layer-name, no margins.

Paper collapses whitespace, so leading spaces and runs of spaces (code, aligned figures) travel as no-break spaces.

Web fonts: Paper serves Google Fonts, Inter with its optical-size axis (opsz 14–32). Match that exactly.

One stylesheet per family, so a family Google does not serve (or a weight it lacks) never blocks the others.

Paper text layers do not inherit type from their parents, so carry it down explicitly.

Paper always rotates about the top-left corner; CSS rotates about the centre. Move the box so it lands in the same place.

builder code from IR — used for Paper import, duplicate and "eject"

An imported <svg> drawn from the icon set comes back as Icon(name, size, { color }).

builder N from IR (for duplicate / paste on canvas)

serialise builder N → code (for edits $add)

Paper JSX / HTML → builder code

style={{ ... }} → style="..."

Paper writes type onto every text layer. On the way back, a font family shared by every text moves up to the root.

Running user code

semantic UI: ui(`screen ... `, theta) → builder nodes styled by a learned style vector (see the Lab)

Running code: new Function where the page allows eval; otherwise an injected
<script> (with the page's nonce) so strict CSPs still work.

run a snippet with a given scope (agents' exec code); same eval / nonce-script fallback as programs

SyntaxErrors from new Function carry no line; find it with a parse-only <script>.

calibrate stack offset once (engines differ in Function wrapper lines)

Trellis Learn — a semantic UI language that streams, a style policy,
and a hand-wired preference model that learns from your picks.

SUL (semantic UI language): one line per element, indentation nests.
Every prefix of a program is a valid program, so it renders while it streams.
screen "Sign in" device=mobile
header "Welcome back" sub="Sign in to continue"
field email label="Email" value="mara@northwind.studio" icon=mail
action "Continue" emphasis=primary
The program says WHAT is on screen. A style vector θ (16 numbers in 0..1)
says HOW it looks. The model learns θ from what you prefer.

1. schema: the typed vocabulary

2. tokenizer, incremental parser, repair, grammar mask

parse(text) → { root, repairs, tokens, lines }. Incomplete trailing lines are skipped (so streams render cleanly).

children only where the schema allows them; otherwise attach to the nearest container

a program without a screen gets one

what may come next at the end of a partial program: typed (schema) and ranked (learned table)

find the parent at this indent

3. a learned table over the grammar: P(kind | parent, previous sibling)

4. style policy θ: 16 numbers in 0..1

5. SUL → Trellis builder nodes, styled by θ

a Trellis Design for a SUL program and θ (used by the Lab to render and measure)

builder code: keeps the semantic program as the source of truth

6. features of a rendered screen (exact geometry, read from the DOM)

7. the preference model: a hand-wired ReGLU net, written out by hand (forward + backward)

score(x) = lin·x + quad·x² + w3·ReGLU(ReGLU(x)) — inputs centred at 0.5.
The linear and quadratic terms learn "more of this, up to a point" from a handful of picks;
the ReGLU layers learn the interactions once there is enough data. Forward and backward are written out by hand.

pairs: { a: x, b: x, y: 1 (a better) | 0 (b better) | 0.5 (tie) }.
Five scorers trained on bootstrap resamples: their mean is the prediction, their disagreement is honest doubt.

add a judged pair, train each member on its own bootstrap, undo if the replay got worse (handwired's trainOrUndo)

8. candidates: explore around the best style, never far from taste

9. corpus: screens written in SUL (also the local generator's memory)

local stand-in for a generator: pick the closest corpus screen to a brief (no model, said plainly in the UI)

Shop UI — IDE (left) · canvas, IR, graph, export (right) · agent

the builder factories, for nodes the canvas creates

presets

app state

DOM refs

Clearing a panel blurs (and Chrome "changes") the field being typed in while it is still attached; commits check this flag

a "/" inside a layer name reads as "∕" in paths (paths are names joined by "/")

Editor

Compile pipeline

Parameters

Canvas

Load the design's web fonts (Inter with optical sizing, like Paper); re-measure once they arrive.

a stage that is not laid out yet (a frame still opening) would fit the design to a sliver: fit once it has a size

overlay: hover, selection, handles, drop line

picking

pointer interaction

Canvas edits → code

A layer moved on the canvas keeps its edits under the path it had in code, so the edits block stays one entry per layer.

Edits stay keyed by the address a layer had before this compile renamed or moved anything, so renaming one

"Row" never makes its sibling "Row#2" answer to the first one's edits.

Inspector edits go to every selected layer (artboard-only keys stay on artboards).

its own edits and its descendants' go with it, so nothing is left pointing at a missing layer

Layers

a board drawn on the canvas: its "+" entry and its inspector edits both move to the new name

keep the renamed layer selected (its path changed; a same-named sibling may now own the old one)

Inspector

Fields accept arithmetic ("300*2", "48+8") without eval, and keep focus across the re-render a commit causes.

commit on blur (Enter blurs): Chrome fires "change" when a re-render removes the field, which is not a commit

Play mode

IR view

Graph view (the DAG)

Export view + Paper bridge

Grammar reference (Docs tab + agent prompt share it)

Agent

Shell: tabs, views, tools, presets, splitter, keys

opens a project: a template's own copy (id = the template's id) or a project made in Shop (see projects.js)

Workspace v2 — the interactions people expect from Paper or Figma.
Every action still lands in code as an edits({...}) entry.

panels

menus

clipboard

structure: wrap, unwrap, order, lock, hide

align & distribute (absolute layers and artboards)

context menu

layers: drag & drop, find, lock

block the click that ends a drag from changing the selection

layer rows get lock buttons, search matches and child-selection tint; patch renderLayers' output after it runs

smart guides, snapping, measurements, marquee

moving with snapping: replace startMove for absolute layers and artboards

marquee on empty canvas (space, the hand tool or the middle button still pan)

inside a single artboard → its top-level layers, else whole artboards

marquee instead of pan on empty canvas; capture phase runs before the original handler

on an artboard's own background (not on a layer) a drag selects layers inside it

command palette

zoom menu

export: PNG, SVG, HTML for the selection

carry inherited type from ancestors

inspector polish: scrubbable labels, sizing, padding, radius, stroke, clip, align

align / distribute bar at the top

sizing modes inline: W [Fixed ▾]

padding: horizontal / vertical, or four sides

fill & border → fill, stroke, radius per corner

keyboard

pick: locked layers are skipped on the canvas

review fixes: rename by double-click, range select, Tab, duplicate in place, refit

live inspector numbers while dragging or resizing

platform-correct shortcut labels in static chrome

unnamed frames get a quiet hint from their first text in the layers panel

colour picker (HSV square, hue, alpha, hex, document swatches)

swap the native colour input in colour fields for the picker

values that differ across the selection read "Mixed"; typing applies to all of them

init

a page that opens narrow (a panel animating open) and then widens gets the desktop layout, refitted

Lab — semantic UI that streams, a style policy, and a model that
learns your taste from A/B picks. Typed, calibrated decisions:
it acts alone when it is sure and asks when it is not.

render a SUL program under θ: measured offscreen, or shown scaled

one round: propose styles, score them, decide or ask

challengers: the options the five scorers disagree about most, each visibly different from the rest

order is random so position never leaks into taste

epistemic doubt = how much the five scorers disagree; low doubt + low confidence means "these are equally good"

a choice among the options: i = index of the winner, 'tie' = all fine, null = skip

honest calibration data: the predictions made before learning from this choice

streaming generation: Claude when connected, a local example otherwise

SUL changed (typed or streamed): parse, show repairs and what may come next, re-render the pair

views

learned style: sliders are also a way to teach it

the Lab page

Co-design — agents work on the canvas beside the person.
One surface for every agent (window.shop, the page's own Claude,
WebMCP): tiered reads, code-mode writes that apply all-or-nothing,
turns the person reviews and can reject without losing their own
edits, pins addressed to the agent, protection and notes that live
in the program, and a taste model that learns from what is kept.

values, diffs and ops on the edits block

persistence: builder nodes are stored as their code and rebuilt on load

paths

steps, pins and blame hold edit keys (stable addresses); these resolve them to the layer's path now

measuring at 100% so numbers do not depend on zoom

checks: what a careful designer would flag after a change

reading: outline, one layer, context

writing: code mode, all-or-nothing

the keys the inspector writes; writing one clears its aliases so two spellings never fight in one entry

removing or hiding a layer takes everything inside it along, protected layers included

not a sandbox, but agent code does not get the page's obvious doors (network, storage, capabilities, this API)

one turn at a time: another agent's call waits while Claude in the page is mid-turn, otherwise it closes the old turn

an agent commit must not throw the person out of the field they are typing in

commit: one undo step per turn

added = layers that did not exist before (a renamed or moved layer keeps its identity)

source-level steps: for components, tokens and folding canvas edits into the code

what the person protected, as builder code, plus their $keep/$note entries: a patch may not change either

review: accept, reject (keeps the person's own edits), learn

the value to restore: what was there before the agent, skipping values that came from steps already rejected

edits the reject left pointing at layers that no longer exist (a removed layer's later tweaks) go too

⌘Z and redo keep turns honest: an undone turn reads as undone, an undone reject puts the turn back up for review

authorship: who changed what

pins: requests anchored to a layer

events, so an agent can wait for the person

persistence per preset

presence: cursor, working boards, flashes, pending outlines

where a pin sits on screen: on its layer, or (layer gone) at its artboard's corner, marked detached

removed layers show as ghosts while reviewing (they would otherwise sit over whatever moved into their space)

pin popover

the dock on the canvas: review bar + composer

what the person is typing in the bar survives the re-render

inspector: protection, notes, history

canvas: comment tool, menus, keys, palette

the guide every agent reads

window.shop: the API every agent uses

WebMCP: browsers that expose navigator.modelContext get the same tools

Claude in the page: same tools, one reviewable turn per request

Claude sessions co-design through the artifact's database ----------
A published Shop declares the db capability (owner and editors only). Any Claude session
with database access writes requests to `inbox/<id>`; the page, open in the person's
browser, runs each one as an agent step they review like any other, writes the result
to `outbox/<id>`, and keeps `state/now` (the context) and `state/guide` current.

another tab may have run it already, or the session withdrew it

reads failed: run it anyway

the page's own state changes feed the bridge

init

Projects — every design is a project; templates (the presets) are where
projects start. Local first in this browser, mirrored to the artifact's
database for everyone who can edit Shop. Versions are restore points:
taken automatically (before agent turns, while you work) and by hand (⌘S).
A project carries a brief that every agent reads, and leaves as one file.

the index

first run: every template you have opened or edited becomes a project of yours, edits included

opening, creating, changing

every compile saves the code; a real change bumps the project, mirrors it, and may take an automatic version

versions

this browser keeps every named version (up to 24) and the 8 newest automatic ones; the database keeps all

the database mirror (owner and editors)

the snapshot fallback refresh keeps going

one project, one file

top bar

the Projects dialog

inspector: the project when nothing is selected

commands and keys

agents: projects through the same API

every agent turn starts with a restore point, so a whole turn can always be taken back

the context and the guide carry the project

the database bridge: projects in the state, and requests that list, open or create them

init

Capabilities: Claude (agent), Paper (host MCP), downloads

boot

## Example preset programs

Preset attributes: data-group="Examples" data-id="01-hello" data-title="1 · Hello, frame"
```js
// 1 · Hello, frame
// A design is a chain. Artboards hold nodes, and nodes are chains too.
// Click anything on the canvas: the editor jumps to the line that made it.

design("Hello")
  .artboard("Card", { w: 440, h: "fit" }, ab => ab
    .bg("#F3F5FA").pad(40).gap(12)
    .add(
      Text("Hello, Shop").size(40).weight(800).tracking(-0.035),
      Text("Every node is a fluent chain. Every value can be a lambda.")
        .size(17).leading(1.5).color("#596072"),
      Row(
        Button("Get started", { icon: "arrow-right" }),
        Button("Read the grammar", { variant: "ghost" }),
      ).gap(8).pt(16),
    ))
```

Preset attributes: data-group="Examples" data-id="02-tokens" data-title="2 · Tokens and mixins"
```js
// 2 · Tokens and mixins
// Tokens become CSS variables (--color-*, --spacing-*, --radius-*), the same
// names Paper uses. "$name" picks the namespace from the property it sits in.
// A mixin is any function n => n…; .with() applies it.

const surface = n => n.bg("$surface").radius("$lg").pad("$lg").border(1, "$line")
const label = n => n.size("$xs").weight(600).upper().tracking(0.08).color("$muted")

design("Tokens")
  .tokens({
    color:   { canvas: "#EDF0F4", surface: "#FFFFFF", ink: "#14161B", muted: "#697083", line: "#DDE2EA", accent: "#3355FF" },
    spacing: { sm: 8, md: 16, lg: 24 },
    radius:  { md: 10, lg: 18 },
    text:    { xs: 11, body: 15, title: 26 },
  })
  .artboard("Profile", { w: 460, h: "fit" }, ab => ab
    .bg("$canvas").pad("$lg").gap("$sm")
    .add(
      Stack(
        Row(
          Avatar("Mara Lindqvist", 56),
          Stack(
            Text("Mara Lindqvist").size("$title").weight(750).tracking(-0.02),
            Text("Product designer · Stockholm").color("$muted"),
          ).gap(2),
        ).gap("$md").align("center"),
        Row(Button("Follow"), Button("Message", { variant: "outline" })).gap("$sm"),
      ).gap("$lg").with(surface),

      Row(
        [["Projects", "48"], ["Followers", "12.4k"], ["Following", "312"]].map(([k, v]) =>
          Stack(Text(k).with(label), Text(v).size(24).weight(700).tabular()).gap(6).fill().with(surface)),
      ).gap("$sm"),
    ))
```
