---
url: https://uiui.statico.io/
title: "uiui"
author: unknown
date_fetched: 2026-09-13
topics:
  - developer-tools
---

uiui is a plain-CSS design system for dense operations UIs — network dashboards, device managers, admin panels — styled after Ubiquiti's Unifi console. One stylesheet, no build step, every class prefixed `ui-`, with light and dark modes. The landing page doubles as a living component catalog: the app shell (48px top bar, icon rail, filter sidebar, detail drawer), 31px-row data tables, port grids, zone matrices, charts, sliders, segmented controls, badges/dots/meters, and a glossary of console metaphors.

The design language is austere by design. Four graphite surfaces plus one interaction blue and a small set of semantic colors (green = healthy, grey = everything else). Typography is Inter at 13px body / 12px secondary / 11px bold titles, with tabular figures and slashed zeros on by default so columns of numbers line up and 0 never reads as O.

The distinctive parts are the six named principles ("Density is the point", "Blue means you can act here", "Status is small", "Three surfaces, nothing else", "Labels are quiet, numbers are tabular", "Destructive is a link") that distinguish the system from a generic dark dashboard — and the agent-facing layer: the stylesheet also defines the shadcn CSS variables with a `registry/uiui-theme.json` theme, and it ships a `skill.md` for coding agents, alongside a glossary written explicitly "so people and agents mean the same thing."
