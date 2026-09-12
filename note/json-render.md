# json-render

Vercel Labs' open-source framework for generating UI from AI prompts via a constrained JSON intermediate layer. Define a component catalog with Zod schemas, let an LLM generate JSON that conforms to it, and render progressively as the stream arrives. React and React Native. 15k GitHub stars.

---

## How It Works

Three-step loop:

1. **Define a catalog** — Zod schemas declare which components, props, actions, and data bindings the AI is allowed to use. This is the guardrail.
2. **AI generates JSON** — The model outputs structured JSON constrained to the catalog. No arbitrary code execution.
3. **Render progressively** — Components appear as JSON tokens stream in, not after generation completes.

The catalog is the key idea. It turns "generate me a UI" from an unbounded code-generation problem into a constrained structured-output problem. The AI can only assemble pre-approved building blocks.

## Component Surface

41 components across six categories: layout (Stack, Grid, Card, Tabs, Dialog, Drawer), input (text fields, selects, toggles, sliders, dropdowns), display (text, badges, avatars, images, metrics, progress bars), data visualization (BarGraph, LineGraph, Table), navigation (Pagination), and feedback (Alert). Six predefined actions handle interactions and data operations.

## Data Binding

Props connect to state via `$state`, `$item`, and `$index` references with two-way binding — the generated UI can both read from and write to application state.

## Code Export

Generated UIs can be exported as standalone React code with no runtime dependency on json-render. The export includes `package.json`, component files, styles, and all dependencies. This is the escape hatch: you are never locked into the framework.

## Key Quotes

> "Set guardrails by specifying which components, actions, and data bindings AI can utilize."

> "Components render progressively as JSON streams arrive."

## Key Themes

- **Structured output as guardrail** — Constraining AI to a schema is more reliable than asking it to write arbitrary code. Same thesis as "linters beat prompts" applied to UI generation.
- **Catalog-driven design** — The developer's job shifts from writing components to curating which components exist and what props they accept. The schema is the product.
- **Progressive rendering** — Streaming JSON tokens into live UI is the UX differentiator over batch code generation.
- **Escape velocity** — Code export means json-render is a generation tool, not a runtime dependency. This is the right call for adoption.

## Critical Analysis

json-render is the "guardrails, not prompts" philosophy applied to frontend generation, and that makes it more interesting than most generative UI projects. The catalog-as-schema pattern directly parallels [[Spec-First Development at Benchling]] — define the contract once, let consumers (here, an LLM) work within it. The Zod integration is smart: it gives you the same validation language your backend already uses.

The 41-component library is both the strength and the ceiling. It covers dashboards and admin panels well, but anything with custom interaction patterns (drag-and-drop, canvas, rich text editing) hits the wall fast. The real question is whether the catalog is extensible enough for teams to add their own design system components, or whether you are stuck with the built-in set.

The streaming progressive render is genuinely useful — it makes LLM latency feel like intentional loading rather than a wait state. This is the same trick that made ChatGPT's token-by-token display feel fast.

Code export is the feature that makes this viable for production. Without it, you are coupling your application to a generation framework, which is a hard sell. With it, json-render becomes a prototyping accelerator that gets out of the way.

At 15k stars, Vercel Labs clearly has distribution. The risk is the same as with any Vercel project: it exists to pull you toward the Vercel platform. But the escape hatch (code export) mitigates that.

Compared to [[vibes-cli]]'s approach (collapse everything into single-file HTML), json-render is the enterprise-grade answer to the same question: how do non-specialists create UIs with AI? vibes-cli says "make it simple enough that you don't need components." json-render says "make the components safe enough that the AI can assemble them."

---

*Sources: [[summary/json-render]]*
*Last updated: 2026-05-14*
