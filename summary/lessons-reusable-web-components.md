---
url: https://dan-webnotes.com/posts/2026-06-07-lessons-reusable-web-components/
title: Lessons for Reusable Web Components
author: Daniel De Pietro
date_fetched: 2026-06-09
date_published: 2026-06-07
topics:
  - software-engineering-craft
---

# Lessons for Reusable Web Components

Many people commented that they expected the CSS-native parallax effect to have an interactive demo, so I re-opened and modernized a small code-sandbox web component I built last year.

Since then, I've dropped it into a handful of projects. Reusing the thing confirmed some of the choices I've made, and gave me some new useful ideas.

> ⚠️ Note: This post doesn't touch advanced web-components techniques, like the shadow DOM or using `<template>` and `<slot>` elements. These tips can apply broadly to UI components and CSS libraries.

## Own a short, distinctive namespace

Every class and custom property in the component now starts with `csb`. For example, I have `.csb-header`, `.csb-controls`, and so on.

- **Short,** because it repeats on every element and every variable, and being short might shave a few bytes and a few minutes of typing.
- **Distinctive,** because the point of a component is to land in a page you don't control, and a generic `.header` or `.button` might collide with the host's styles.

## Theme with namespaced CSS variables

This is the most important. Rather than ship override classes, every value worth changing is a **CSS custom property, with its default written in as the fallback:**

```css
.csb-header {
  border-color: var(--csb-border-color, light-dark(#ddd, #6b6b6b));
}
```

The component renders correctly with zero configuration, because the fallback is the default. Theming it means setting variables on the root element, or any parent element, and let the cascade do its thing:

```css
code-sandbox {
  --csb-border-color: var(--my-color);
  --csb-editor-bg: #282c34;
}
```

No `!important`, no descendant selectors reaching three levels deep, no cascade to fight. The list of variable names is the whole public surface, which also means that you should try to keep these ones stable between versions.

## Let modern CSS and JS do the magic

A reusable component lands in contexts you can't predict, so using the modern features that the platform offers is one of the best way to achieve adaptability and resilience:

- **Container queries, not media queries.** You don't know if a component will be added in a narrow sidebar or full width. Use `container-type: inline-size` on the parent element, so the child can responds to its own width.
- `light-dark()` with `color-scheme` to have dark mode by default.
- System fonts (`system-ui`, `ui-monospace`) and system colours (`Canvas`) so the default always works, and is consistent as possible with the underlying OS.
- Logical properties (`border-block-end`, `padding-inline`) so it can support right-to-left layout, if needed.
- `em` for sizing, so the whole component scales with whatever font size it inherits.

## Publish it instead of pasting it

While it can sound like a good idea copying the JS and CSS into each project by hand to achieve the final simplicity, copying and pasting gets tiring quite fast. It's now on NPM, and it can be imported in a line and updated with a version bump rather than a copy-paste tour of old projects.

```
npm i @danieledep/code-sandbox
```

Publishing it on NPM gives you for free command-line installing and **CDN hosting**, useful for the no-build case. So a static HTML page can use it by importing it in the `<head>` of the file.

```html
<script type="module" src="https://cdn.jsdelivr.net/npm/@danieledep/code-sandbox@1/src/code-sandbox.js"></script>
```

## Then document all of it

A module only counts as reusable once someone other than you can actually use it and customize it. The code-sandbox README carries the theming table, the layout and highlighter guides, and the install instructions.

## Repository

- [danieledep/code-sandbox](https://github.com/danieledep/code-sandbox)

## Resources

- [@danieledep/code-sandbox on NPM](https://www.npmjs.com/package/@danieledep/code-sandbox)
- [Fullscreen API](https://developer.mozilla.org/en-US/docs/Web/API/Fullscreen_API)
- [\<system-color\> CSS type](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/system-color)
- [light-dark() CSS function](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/color_value/light-dark)
- [Making Fullscreen Experiences](https://web.dev/articles/fullscreen)
- [Shiki Syntax highlighter](https://shiki.style/)
