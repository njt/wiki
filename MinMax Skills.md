# MinMax Skills

A collection of development skills for AI coding agents from MiniMax, covering frontend, mobile (including Flutter), creative media, and AI-powered features. Notable for the Flutter skill specifically, and for the breadth of the offering -- this is one of the larger curated skills libraries at 11.8k stars.

---

## Key Themes

#skills #flutter #mobile #agent-tooling

The skills span frontend (React/Next.js, Tailwind, animation libraries), mobile (Android/Kotlin, iOS/Swift, Flutter, React Native), creative (GLSL shaders, PDF/PPTX/XLSX/DOCX generation), and AI media (vision, voice, music, video, image generation via MiniMax's own API).

The Flutter skill is the standout for cross-platform mobile development. Flutter's widget tree architecture maps well to agent-generated code because it's declarative and compositional -- an agent can reason about the UI hierarchy without managing imperative state.

The C# dominance (68.5% of the codebase) is unexpected for a skills library that targets JavaScript-heavy frameworks. This suggests the document generation skills (Office formats via OpenXML SDK) are a significant portion of the codebase.

## Critical Analysis

The MiniMax API integration for media generation is both a feature and a vendor lock-in. Skills that depend on a specific company's API are less portable than pure-code skills. But for teams already using MiniMax's models, having ready-made skills is genuinely useful.

Available for Claude Code, Cursor, Codex, and OpenCode -- the multi-platform support matters because it means these skills aren't tied to a single agent harness. Compare with [[Awesome Vibez]], which catalogs projects from a community that's heavily Claude Code-centric.

---
*Sources: [[raw/minimax-skills]]*
*Last updated: 2026-05-14*
