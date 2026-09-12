---
url: https://uiui.statico.io/
date_fetched: 2026-09-13
---

**uiui**

A dense console UI kit inspired by the Unifi console. One CSS file, every widget on this page, light and dark.

**940 Mbps**

**38 Mbps**

**2**Mon

**4**Tue

**1**Wed

**0**Thu

**6**Fri

**3**Sat

**1**Sun

### Throughput

### Core Switch

Online**24-port PoE**192.168.1.2 · 4.2.1

| Name | IP | Connected to | Link | Signal | Usage | Uptime | ||
|---|---|---|---|---|---|---|---|---|
| Kitchen Display | 192.168.1.24 | Loft AP | 5 GHz | 128 MB | 1 h 12 m | |||
| Studio Laptop | 192.168.1.31 | Studio AP | 6 GHz | 6.4 GB | 3 d | |||
| Media Server | 192.168.1.5 | Core Switch | 1 GbE | 212 GB | 41 d | |||
| Garage Camera | 192.168.0.44 | Garage Switch | PoE | 18 GB | 12 d | |||
| Porch Light | 192.168.0.71 | Garden AP | 2.4 GHz | 4 MB | 9 d | |||
| Thermostat | 192.168.0.80 | Hall AP | 2.4 GHz | 11 MB | 27 d | |||
| Backup NAS | 192.168.1.6 | Core Switch | 1 GbE | 96 GB | 41 d | |||
| Lab Pi | 192.168.1.140 | Core Switch | 1 GbE | 1.2 GB | 6 d | |||
| Hall Phone | 192.168.1.52 | Hall AP | 5 GHz | 310 MB | 14 m | |||
| Vacuum | 192.168.0.90 | Garden AP | 2.4 GHz | 22 MB | 2 d | 

## Getting started

uiui is a plain-CSS design system inspired by the Unifi console, for dense operations UIs: network dashboards, device managers, admin panels, anything with many rows and a handful of colored numbers. One stylesheet, no build step, every class prefixed `ui-`.

<link rel="stylesheet" href="uiui.css"> <body class="ui"> … </body> <!-- icons: lucide sprite, referenced by id --> <svg class="ui-icon"><use href="lucide-sprite.svg#wifi"/></svg>

Using shadcn/ui or Tailwind instead? `uiui.css` also defines the shadcn variables (`--background`, `--primary`, …) and there is a theme in `registry/uiui-theme.json`. See skill.md for the agent-facing summary.

## Example pages

Full screens assembled from the widgets below. Open one, then copy the shell and swap in your own data.

**Dashboard**Summary cards, throughput chart, latency pills, top-clients tiles, device strip.

**Clients table**Filter sidebar, sortable dense table, selected row, detail drawer with tabs.

**Settings**Settings nav with tree, content cards, label-left forms, option cards with focus panel.

**Zone matrix & policies**Allow / block matrix, policy table, diagram nodes, timeline rail.

**Radio analysis**Signal buckets, gradient scale, channel plan, airtime bars, spectrum chart.

## Color

Four graphite surfaces, one blue, and a small set of semantic colors. Blue is for interaction: links, selection, primary buttons, the active tab. Green means healthy, and nearly everything else is text-2 grey.

**--ui-bg**#131416 page, sidebars

**--ui-surface**#1c1e21 cards, selects

**--ui-input**#25272a text inputs

**--ui-raised**#282b2f top bar, chips, hover

**--ui-border**#232526 hairlines

**--ui-border-2**#34383d outlines

**--ui-selected**#18212e selected row

**--ui-accent**#4797ff links, primary, active

**--ui-accent-dim**#386197 solid segmented tab

**--ui-accent-deep**#05254d active rail, timeline

**--ui-text**#dee0e3 body

**--ui-text-2**#929497 secondary, values

**--ui-text-3**#6b6e72 placeholders, offline

**--ui-green**#37be60 healthy, GbE

**--ui-lime**#b0d322 100 Mbps

**--ui-yellow**#dfc116 warning, threshold

**--ui-orange**#e79613 fair, busy

**--ui-red**#ee6368 error, destructive

**--ui-purple**#b47cde upload

**--ui-cyan**#5aa9c9 chart line, 10 GbE

**--ui-gray**#50565e disconnected port, switch off

**--ui-allow**#42674f matrix allow

**--ui-block**#755b5e matrix block

**--ui-return**#2a4c5b matrix allow-return

### Text color utilities

## Typography

Inter (or the system sans), 13px body, 12px secondary, 11px bold section titles. Tabular figures and slashed zeros are on by default so columns of numbers line up and 0 never reads as O.

## Icons

Icons are lucide (ISC license), vendored as one SVG sprite. Use 16px at 1.75 stroke in text, 20px at 1.5 stroke in the icon rail. Icons take `currentColor`, so color them with the text utilities.

### Vocabulary used by the console metaphors

## App shell

The frame every screen sits in: a raised top bar with tabs, a 48px icon rail, an optional filter sidebar, the main area, and an optional right-hand detail drawer. Grid areas handle it; drop the sidebar or drawer element and the grid collapses.

**Do**

- Keep the top bar at 48px and the rail at 48px wide.
- Center a muted product name in the top bar.
- One active rail icon, filled with the deep-blue halo.

**Don't**

- Put text labels in the rail; use tooltips.
- Stack two sidebars. Filters live left, details live right.
- Use a drawer wider than 360px.

## Buttons & links

Primary blue for the one action that matters, raised grey for everything else. Destructive actions are red text links, never red fills. Actions in cards are plain links separated by a hairline.

### Link rows

## Inputs

Inputs sit on a slightly lighter surface than the page, with no visible border until focus. Labels are 11px grey above the control. Search boxes use the raised surface and a leading icon.

## Checkbox · Radio · Switch

Checkboxes fill blue with a white tick. A count in parentheses after the label is the filter-sidebar convention. Switches are a 32×18 pill: blue on, gray off.

## Segmented controls & chip groups

Segmented controls choose one of a few modes and never wrap. The default flavor shows the active tab as a raised pill with blue text. The `solid` flavor fills it with dim blue, used for time ranges. Chip groups are multi-select toggles with an outlined active state.

### Sidebar sub-tabs (icon only until selected)

## Sliders

A thin track with a white thumb, tick labels underneath. Range filters in the sidebar use two thumbs and show the selected span above.

## Form rows

Settings pages put the label on the left in a 240px column and the control on the right. Related controls nest in a stack under one label. Section titles in cards carry an icon on the left and a collapse chevron on the right.

### Internet

## Badges · Dots · Meters

Status is a 6px dot next to the name, never a colored row. Chips are grey by default; the blue chip marks "you" or an active filter. Risk and severity are three- or four-dash meters, colored by level.

### Count strips and pill stats

**940 Mbps**

**38 Mbps**

**11 ms**

## Data table

31px rows, 13px text, a hairline between rows and nothing else. Headers are bold white; the sorted column is blue with an arrow. Offline rows go text-3 grey. The selected row gets the navy fill and a 2px blue left edge. Numbers are right-aligned in tabular figures.

| Name | IP Address | Connection | Network | Experience | Down | Up | Uptime | ||
|---|---|---|---|---|---|---|---|---|---|
| Kitchen Display | 192.168.1.42 | Loft AP · 5 GHz | Default | Excellent | 18.4 Mbps | 2.1 Mbps | 3d 4h | ||
| Studio Laptop you | 192.168.1.17 | Studio AP · 6 GHz | Default | Excellent | 412 Mbps | 31.7 Mbps | 12h 6m | ||
| Garage Camera | 192.168.0.103 | Garage Switch · Port 4 | Cameras | Excellent | 0 bps | 4.8 Mbps | 21d 2h | ||
| Hallway Bulb | 192.168.0.71 | Hall AP · 2.4 GHz | IoT | Fair | 0 bps | 0 bps | 9d 11h | ||
| Office Printer | 192.168.1.90 | Core Switch · Port 12 | Default | Good | 2.2 kbps | 640 bps | 40d 1h | ||
| Guest Phone | 192.168.1.208 | — | Guest | — | — | — | Offline | 

### Compact table inside a card, with tiny channel headers and heat cells

| Access Point | 36 | 40 | 44 | 48 | 149 | 153 | Clients | 
|---|---|---|---|---|---|---|---|
| Loft AP | 9 | ||||||
| Studio AP | 6 | 

## Filter sidebar

A 260px column of collapsible sections. Each title is 11px bold caps with a chevron; options are checkboxes with counts. Sliders filter ranges, switches flip view options, and the footer holds Clear Filters plus page actions.

## Cards & key-values

Cards are the surface color with 8px corners and no border. Inside a drawer they hold a title row and a key-value list: key on the left in body text, value on the right in text-2. Stat cards are the same list with an icon in the key.

### Overview

### Threats

### Speed Test

**940 Mbps**

**38 Mbps**

### Device hero and link line

**Core Switch**24-port · 192.168.1.2

## Detail drawer

A 360px right panel: centered title with a close button, a three-icon segmented control for Overview / Stats / Settings, then stacked cards. Destructive actions are red links at the bottom. Quick actions are small icon tiles.

## Icon tiles

Rows of small outlined tiles with a count above and a label below. Used for Top Access Points, Top Clients and Top Apps. The selected tile gets a blue ring.

## Port grid

Two rows of 38px squares, numbered bottom-left, colored by negotiated speed. A lightning glyph marks PoE; a small icon in the corner marks uplink or error. Grey means nothing is plugged in.

## Zone matrix

Source zones down the side, destination zones across the top. Each cell is a muted green (allow), maroon (block) or teal (allow return traffic) tile with a policy count in the corner. Click a cell to edit its policies.

| Internal | External | Guest | IoT | VPN | |
|---|---|---|---|---|---|
| Internal | Allow | Allow2 | Allow | Allow | Allow | 
| External | Return | Return | Return | Block | |
| Guest | Block1 | Allow | Block | Block | Block | 
| IoT | Block3 | Allow | Block | Allow | Block | 
| VPN | Allow | Allow | Block | Allow | Allow | 

## Charts & bars

Charts are inline SVG in a `.ui-chart` wrapper: dotted grid, 10px grey axis text, 1.5px lines. Download is cyan or blue, upload is purple, thresholds are dotted yellow. Bars are 6px tracks. Distribution buckets run red → orange → yellow → green.

### Signal buckets and gradient scale

**1**-90

**0**-85

**2**-80

**3**-75

**4**-70

**6**-65

**9**-60

**8**-55

**4**-50

**1**-45

**0**-40

**0**-35

**0**35%+

**0**30%

**1**25%

**2**20%

**3**15%

**7**10%

**14**5%

**11**0%

### Channel plan

### Spectrum bands

## Timeline

A narrow vertical rail for scrubbing history. Ticks on the left, a navy band for the loaded window, and a blue LIVE marker at the top.

### Event rows

| 14:02:11 | Guest Phone | connected to Hall AP on 2.4 GHz | |
| 13:48:30 | Loft AP | changed channel 44 → 149 (interference) | |
| 13:11:07 | Gateway | blocked outbound connection from 192.168.0.71 | 

## Diagram nodes

Topology and policy diagrams use solid navy nodes for devices, dashed blue outlines for groups, and thin blue edges with a small label. Routing choices are shown as wide cards.

## Settings navigation

A 210px column: search on top, then icon rows for sub-sections, section labels, and a dotted connector between a parent and its child items. The active item is the raised surface with blue text.

## Option cards & toggle rows

When a setting is one of a few big choices, list them as option cards in a surface column, and open the selected one in a panel with a blue outline. Feature toggles with descriptions are toggle rows.

## Notices · Tooltips · Toast

Notices are quiet surface blocks with an icon, a bold first line, and an optional close. Tooltips are raised, shadowed, and may carry a title with a key-value list. Toasts confirm an action in the same words as the button that caused it.

**Changes take effect on next reconnect**Devices already connected keep their current settings until they roam or reconnect.

**160 MHz needs DFS channels**Radar detection may move this access point off its channel for up to a minute.

**Link down.**WAN2 has been unreachable for 4 minutes.

**Loft AP · 5 GHz**

## Modal

A 400px raised dialog: title row with close, an optional search, content, and a footer with the secondary action on the left and Cancel + primary on the right.

## Glossary

The words this site uses for its parts, so people and agents mean the same thing.

- Shell
- The fixed frame of a console page: topbar, rail, sidebar, main and drawer (`.ui-app`).
- Rail
- The narrow icon-only column on the far left that switches between sections.
- Sidebar
- The filter or navigation column between the rail and the main area.
- Drawer
- The detail panel docked on the right that shows the selected item.
- Chip
- A small rounded status label such as Online or Blocked (`.ui-chip`).
- Meter
- Four short bars showing signal quality; level 1 is best, level 4 is worst (`.ui-meter`).
- Segmented control
- A row of joined buttons where exactly one is active, used for time ranges and tabs (`.ui-seg`).
- Key-value list
- Label on the left, value on the right, one row per fact (`.ui-kv`).
- Port grid
- A switch faceplate of numbered squares coloured by link speed (`.ui-ports`).
- Zone matrix
- A grid of allow, block and return cells showing traffic rules between zones (`.ui-matrix`).
- Bucket
- A short row of coloured boxes, one per day or band, each holding a count (`.ui-buckets`).

## Principles

What makes a screen feel like this system rather than a generic dark dashboard.

**Density is the point**

- 31px rows, 13px text, hairlines only. No zebra striping, no card per row.
- Pack the sidebar with counts in parentheses; that is the scan aid.
- Values are text-2 grey; only the exception gets a color.

**Blue means "you can act here"**

- Links, active tabs, checked boxes, primary buttons, the selected row edge, the sorted header.
- Never use blue as decoration or as a data color except for download.

**Status is small**

- A 6px dot, a chip, or a colored word. Never a tinted row or a giant badge.
- Green good, yellow ok, orange fair, red bad, grey offline.

**Three surfaces, nothing else**

- Page, surface, raised. Depth comes from the step between them, not from shadows or borders.
- Cards have no border. Inputs have no border until focused.

**Labels are quiet, numbers are tabular**

- 11px grey labels, 11px bold section titles, slashed zeros everywhere.
- Right-align numeric columns and give them units.

**Destructive is a link**

- Reconnect / Block / Remove are red text, stacked at the end of a drawer.
- Confirm in a modal; never in a toast.

## For agents

If you are generating a page with this system, read skill.md. The short version: wrap the page in `.ui`, compose from the classes above, use the lucide sprite for icons, invent device names and use 192.168.0.x / 192.168.1.x addresses, and pick one of the example pages as the starting shell.
