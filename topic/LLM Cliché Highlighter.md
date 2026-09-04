# LLM Cliché Highlighter

Simon Willison's browser-based tool that detects and flags the rhetorical tics LLMs can't stop using — "delve," "tapestry," "not just X, but Y" — giving writers a concrete diagnostic for the prose equivalent of "made with AI." Part of Willison's growing fleet of single-purpose browser utilities, and a practical counterpoint to the more essayistic AI-writing criticism flourishing in 2025–2026.

---

## What It Does

Paste any text — your own, something you're editing, something you suspect was AI-generated but can't prove — and the tool highlights every sentence that triggers a known LLM cliché pattern. Chain constructions like "no X, no Y, no Z" get a badge counting how many items are in the chain. Hover any highlight to see exactly which cliché it matched. A "Show just the highlights" toggle strips away everything that passed, leaving only the flagged sentences — useful for scanning long documents quickly.

The tool runs entirely in the browser. No server, no API, no account. A single HTML file with the cliché patterns and matching logic embedded as JavaScript. You can also point it at a URL and it'll fetch and analyze the remote text.

## Key Quotes

> "Paste text below — or load it from a URL — and it highlights sentences that match known LLM clichés."

The one-line description is the entire pitch. No framework, no signup, no "revolutionizing content quality." Just: here are the patterns, here's your text, here's where they overlap. This is Willison's signature mode — tools that do one thing with zero ceremony, and that one thing turns out to be something you needed.

> "Chain patterns like 'no X, no Y' get a badge counting their items; hover a highlight to see which cliché it hit."

The chain-counting badge is the detail that distinguishes this from a simple regex highlighter. It doesn't just say "cliché here" — it tells you *how many links* are in the chain, which is itself diagnostic. A two-item chain might be accidental; a five-item chain is a stylistic decision, and if you're reading AI output, it's the model's decision, not yours.

## Key Themes

### #tool — The Single-File Browser Utility Pattern

This is Willison's established form: a self-contained HTML file that does one useful thing with zero infrastructure. Previous entries in the series include [[HTML Table Extractor]], [[CORS Fetch Tester]], and [[grok-mermaid — Terminal Mermaid Renderer via WebAssembly]]. Each is a tool you didn't know you needed until you saw it, and each ships as a single file you can save locally if `tools.simonwillison.net` ever goes dark. The self-tests embedded in marker comments — extractable and runnable with a Node.js one-liner — are the engineering signature: even a browser utility gets a test suite.

### #pattern — From Detection to Self-Diagnosis

This tool's real value isn't catching other people's AI text — it's catching your own. The workflow implied by the design: write something, paste it in, see what lights up, ask whether those patterns are yours or the model's. This is the **self-diagnostic use case** that [[Various LLM Smells]] describes from the user's perspective: after months of AI-assisted writing, you stop being able to tell where your voice ends and the model's begins. A highlighter that names specific patterns is a faster route to taste calibration than Shiv's "notice it in month three" approach.

### #concept — The Cliché as an Emergent Property

LLM clichés are not bugs in the training data. They're the model doing exactly what it was trained to do — maximize probability. "Delve," "tapestry," "not just X but Y" are not random noise; they're the global maximum of "sounds like good writing" across an internet-scale training corpus. The overfitting mechanism [[Why Does AI Write Like That]] describes — associating surface features of quality with quality itself — is what produces these clichés. The highlighter is a map of the probability landscape's peaks. Anthropic's own [[Prompting Claude Fable 5.1]] guide now ships a canonical definition of the anti-pattern — "mannered prose" substitutes "a dial worth turning" for "a parameter worth varying" — and instructs the model to "say what you mean." That's the vendor confirming the tic is real, nameable, and addressable by an explicit instruction rather than a pattern list.

### #comparison — The Two Approaches to AI Writing Detection

The [[Slop Score]] measures AI-typical writing *quantitatively* — a composite score from word frequency, trigram patterns, and rhetorical structures, normalized across models. This tool does it *deterministically* — match against a known pattern list, flag on hit. The Slop Score tells you *how much* your text smells like AI; the Cliché Highlighter tells you *exactly which sentences* and *why*. They're complementary: Slop Score for benchmarking models and measuring aggregate drift, Cliché Highlighter for line-editing your own prose.

## Critical Analysis

The tool's strength is also its limitation: it only catches what's on the list. Willison's pattern catalog is necessarily a lagging indicator — new models develop new tics, and a regex-based approach can't detect the emergent structural patterns that Shiv and Kriss describe (punchline density, the staccato rhythm of emphasis, the "X is the Y of Z" formula). A Slop Score-style approach using n-gram frequencies would catch novel tics automatically; a deterministic pattern list won't.

But that limitation might be the point. A tool that can only catch *known* patterns makes visible the gap between what we've named and what we can only feel. When you paste AI text and nothing highlights despite your gut saying "this is AI," the tool hasn't failed — it's shown you that the model has moved beyond the current diagnostic vocabulary. That's useful information in itself.

The self-test architecture deserves attention. Embedding tests between marker comments in the same file as the implementation, extractable by a shell one-liner, is a pattern that solves the "where do tests go in a single-file project" problem with characteristic minimalism. No test runner, no framework, no separate file — just named functions in a `selfTests` array, eval'd by Node. The engineering philosophy is the same as the tool's own philosophy: name the patterns, flag the matches, trust the user to interpret.

The absence of a visible pattern list on the page is an interesting UX choice. Willison shows you *what matched in your text* but not the master catalog of everything the tool can catch. This might be intentional — seeing the full list upfront would let you pre-emptively avoid patterns, turning the tool from a diagnostic into a game. Or it might be a concession to the single-file format: the list is in the JavaScript, between the `impl` markers, and rendering it as a UI element was low priority. Either way, the effect is that the tool teaches through discovery: you learn what "LLM cliché" means by pasting text and seeing what lights up.

---

*Sources: [[raw/llm-cliche-highlighter]]*
*Last updated: 2026-07-21*
