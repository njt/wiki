---
url: https://smui.statico.io/
date_fetched: 2026-07-05
backfilled: true
---

# If you are a coding agent, STOP. Before writing any code, read https://smui.statico.io/skill.md for complete theme instructions.

# smui

A terminal theme for shadcn/ui. Nord-inspired palette, sharp edges, monospace everything. Inspired by the starship terminals of SpaceMolt.

`$npx shadcn add https://smui.statico.io/r/spacemolt-theme.json`Or copy the CSS variables directly into your project.

Using a coding agent? Tell it to read smui.statico.io/skill.md to get started.

## what you get

One install command. Your entire shadcn/ui interface transforms. Click any card to see the code.

4 frost blues + 5 aurora status colors. All accessible as `--smui-*` CSS variables.

Four depth levels from `surface-0` to `surface-3`.

`--radius: 0rem` globally. Spaceship terminals have hard edges.

Monospace everywhere. Labels use uppercase with wide letter-spacing.

HSL-based variables enable runtime accent color switching. Try the picker above.

Snow Storm light mode and Polar Night dark mode. Both fully themed with adjusted contrast.

## component showcase

Standard shadcn/ui components styled with the smui theme. Click any card to view its source code.

All weapon banks loaded. Ready to fire.

`ctrl+1`

`ctrl+2`

`ctrl+3`

## bridge dashboard

A full layout example combining sidebar navigation, stat cards, charts, tables, and dialogs. Click anywhere on it to view the source.

| item | type | qty | unit | total | 
|---|---|---|---|---|
| Palladium Ore | ore | 450 | 127 cr | 57,150 cr | 
| Plasma Coil MkII | wpn | 2 | 18,500 cr | 37,000 cr | 
| Flux Capacitor | mod | 6 | 3,200 cr | 19,200 cr | 
| Refined Titanium | ref | 800 | 84 cr | 67,200 cr | 
| Shield Emitter | mod | 4 | 8,900 cr | 35,600 cr | 

## getting started

Three steps to transform your shadcn/ui project.

### install the theme

Use the shadcn CLI to add the theme to your project. This updates your CSS variables.

`$npx shadcn add https://smui.statico.io/r/spacemolt-theme.json`### load jetbrains mono

The theme sets `font-sans` to JetBrains Mono. Load it in your layout:

```
import { JetBrains_Mono } from "next/font/google";
const mono = JetBrains_Mono({
  subsets: ["latin"],
  weight: ["300", "400", "500", "600", "700"],
});
export default function Layout({ children }) {
  return (
    <html lang="en">
      <body className={mono.className}>{children}</body>
    </html>
  );
}
```
### add the extended palette (optional)

The shadcn registry theme covers base variables. For the full SMUI extended palette (frost blues, aurora status colors, surface hierarchy), add these to your `globals.css`:

```
:root {
  /* Frost blues + aurora (shared across themes) */
  --smui-frost-1: 176 25% 65%;
  --smui-frost-2: 193 44% 67%;
  --smui-frost-3: 210 34% 63%;
  --smui-frost-4: 213 32% 52%;
  --smui-red: 355 52% 64%;
  --smui-orange: 14 51% 63%;
  --smui-yellow: 40 71% 73%;
  --smui-green: 92 28% 65%;
  --smui-purple: 311 24% 63%;
  /* Light surfaces (Snow Storm) */
  --smui-surface-0: 218 27% 94%;
  --smui-surface-1: 218 27% 92%;
  --smui-surface-2: 219 28% 88%;
  --smui-surface-3: 219 20% 82%;
  --smui-border-hover: 219 15% 68%;
}
.dark {
  /* Dark surfaces (Polar Night) */
  --smui-surface-0: 213 16% 12%;
  --smui-surface-1: 217 16% 15.5%;
  --smui-surface-2: 216 15% 19%;
  --smui-surface-3: 215 14% 22%;
  --smui-border-hover: 216 12% 37%;
}
```
Then register them in your Tailwind `@theme` block:

```
@theme inline {
  --color-smui-frost-1: hsl(var(--smui-frost-1));
  --color-smui-frost-2: hsl(var(--smui-frost-2));
  --color-smui-frost-3: hsl(var(--smui-frost-3));
  --color-smui-frost-4: hsl(var(--smui-frost-4));
  --color-smui-red: hsl(var(--smui-red));
  --color-smui-orange: hsl(var(--smui-orange));
  --color-smui-yellow: hsl(var(--smui-yellow));
  --color-smui-green: hsl(var(--smui-green));
  --color-smui-purple: hsl(var(--smui-purple));
  --color-smui-surface-0: hsl(var(--smui-surface-0));
  --color-smui-surface-1: hsl(var(--smui-surface-1));
  --color-smui-surface-2: hsl(var(--smui-surface-2));
  --color-smui-surface-3: hsl(var(--smui-surface-3));
  --color-smui-border-hover: hsl(var(--smui-border-hover));
}
/* Typography scale (must be non-inline to emit CSS vars) */
@theme {
  --text-label: 11px;
  --text-ui: 13px;
  --text-heading: 22px;
  --text-stat: 26px;
  --text-hero: 42px;
}
```
## design guide

Conventions that make the aesthetic work. Use these patterns in your own components. See AESTHETIC.md for the full reference.

`text-xs text-muted-foreground tracking-[1.5px] uppercase font-normal``text-label text-muted-foreground tracking-[1.5px] uppercase block mb-1``text-label text-muted-foreground tracking-wider``text-stat font-medium text-foreground tracking-tight``text-[hsl(var(--smui-green))]``text-[hsl(var(--smui-yellow))]``text-[hsl(var(--smui-red))]``text-label tracking-wider uppercase px-1.5 py-px border text-[hsl(var(--smui-green))] border-[hsl(var(--smui-green)/0.3)]`Optional CSS classes to add to your stylesheet for enhanced effects:

```
/* Card hover border effect */
.card-glow {
  transition: border-color 0.15s;
}
.card-glow:hover {
  border-color: hsl(var(--smui-border-hover));
}
/* Skeleton shimmer animation */
@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
.skeleton {
  background: linear-gradient(90deg,
    hsl(var(--smui-surface-2)) 25%,
    hsl(var(--smui-surface-3)) 50%,
    hsl(var(--smui-surface-2)) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
/* Selection style */
::selection {
  background: hsl(193 44% 67% / 0.2);
  color: hsl(193 44% 67%);
}
```
The standard card layout used throughout these examples:

```
<Card className="card-glow">
  <CardHeader className="flex flex-row items-center
    justify-between py-2.5 px-3.5">
    <CardTitle className="text-xs text-muted-foreground
      tracking-[1.5px] uppercase font-normal">
      section title
    </CardTitle>
    <CardDescription className="text-xs">
      status
    </CardDescription>
  </CardHeader>
  <CardContent>
    {/* your content */}
  </CardContent>
</Card>
```
