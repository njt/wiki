# ANSI Escape Sequence Injection in MCP Servers

A new injection class — ANSI Escape Sequence Injection (AESI) — exploits the gap between how terminals render text and how language models consume it. Escape sequences that terminals interpret as formatting instructions (hide text, clear screen, move cursor) are invisible to humans but read as raw bytes by LLMs. In MCP servers, where tool results, resource reads, and prompt templates stream directly into model context, this becomes a prompt injection vector that bypasses human supervision entirely. The stored variant is the dangerous one: payload persists in application data and detonates on later, unrelated reads.

---

## Key Quotes

> "Every field the model reads is a potential injection surface."

This is the article's thesis, and it reframes MCP security from "authenticate the connection" to "scrutinize every byte that reaches the model." The move from connection-level to field-level thinking is the conceptual leap that makes AESI legible as an attack class. It's the MCP equivalent of realizing XSS isn't about `<script>` tags but about any HTML context where attacker-controlled data lands.

> "Rendering hides these bytes from people, but a model processes the raw text."

The core asymmetry. Human reviewers see rendered output; models ingest the wire format. Any escaping, sanitization, or normalization that happens in the *renderer* never reaches the *model*. This is why DAST is the right detection surface — only a live server's actual response bytes reveal what the model will read. Static analysis of server code cannot answer this question because the dangerous bytes may be introduced anywhere in the fetch-store-read pipeline.

> "One poisoned record in a shared knowledge base can influence every later agent workflow that reads it."

The stored AESI scenario is the scariest because it's a second-order injection. The attacker doesn't need to be present when the payload fires. They write once — a comment, a note, a document — and the payload sits dormant until any agent, on any future session, reads it back into a model-consumable field. This is SQL injection's stored-procedure cousin, scaled across time and users.

> "You cannot infer these write-then-read paths from parameter names, tool descriptions, or protocol type."

The probe-phase methodology is the genuine innovation. Rather than guessing which writes feed which reads, the scanner empirically maps the data flow: write inert markers through every injection entrypoint, then scan every read surface for reflections. This converts an unknown source-to-sink topology into a concrete map — and discovers cross-protocol paths (HTTP write → MCP read) that no static analysis would find.

> "A 1970s cursor code becomes an injection primitive the moment a model, not a terminal, reads the bytes."

The article's best line. It captures why AESI is a category, not a bug: any terminal-heritage artifact that a model consumes raw becomes a potential injection surface. ANSI is just the most obvious one. Terminal emulators have decades of accumulated quirks — OSC codes, DCS strings, SGR parameters, sixel graphics, kitty keyboard protocol — and every one of them is a byte sequence a model might read differently than a human would.

## Key Themes

#security #prompt-injection #MCP #DAST #exploit #agent-attack-surface

## Critical Analysis

**The naming is half the value.** Coining "AESI" and splitting it into direct-fetch vs. stored makes an amorphous threat into something reproducible and discussable. This is what good security research does: name the thing, taxonomize the variants, give defenders a shared vocabulary. The CVE citations (kubectl CVE-2021-25743, Git CVE-2024-52005) anchor the attack in real precedent rather than speculative threat modeling.

**The stored variant is the real contribution.** Direct-fetch AESI is a narrow attack — the attacker must supply a URL to a fetch tool, which is a gated path. Stored AESI uses any write entrypoint (comment, note, document, API POST) and detonates later. The probe-then-map methodology for discovering these paths empirically — write inert markers, scan all read surfaces, build the topology — is genuinely novel and portable beyond ANSI. You could use the same approach to map any injection surface in a multi-protocol agent system.

**The three-signal confirmation is smart DAST engineering.** Requiring an ANSI escape byte AND a fixed instruction phrase AND a unique trigger marker in the same model-consumable field produces a binary, falsifiable signal. No tuning, no confidence scores, no ML classifier that needs training data. This is the kind of deterministic detection that holds up in CI pipelines and doesn't produce the base-rate false-positive problem that [[Bounding the Blast Radius — Prompt Injection Defenses]] identifies in LLM-judge approaches.

**But the defense section is undercooked.** "Sanitize control bytes" is correct but trivial — strip `0x1B` and everything after it up to the command letter. The harder question is what *other* terminal-heritage artifacts create similar injection surfaces. Terminal emulators have accumulated decades of features: OSC (Operating System Command) codes can set window titles, DCS (Device Control String) can request terminal state, and some terminals support sixel graphics or kitty's extended keyboard protocol. A defender who strips ANSI CSI sequences but leaves OSC codes has built a sieve. The article gestures at this ("future attacks will come from old representation quirks") but doesn't enumerate the attack surface.

**Vendor context matters.** This is published on Bright Security's research blog. They sell DAST. The methodology described is their product's approach, and the framing of "DAST is the only way" should be read with that in mind. That said, the argument is sound on its own terms — static analysis genuinely cannot answer "what bytes does this live server actually return?" — and the probe-phase topology mapping is a contribution to the field regardless of whose scanner implements it.

**The bigger lesson is about format collision.** AESI isn't really about ANSI. It's about what happens when a data format designed for one consumer (terminal emulators) reaches a different consumer (LLMs) that interprets the bytes differently. This is the same class of problem as [[Bounding the Blast Radius — Prompt Injection Defenses]]'s observation that LLMs have no code/data boundary — but at the *byte* level rather than the *token* level. The SQL injection analogy is instructive: we solved that with parameterized queries that enforced a code/data boundary. For AESI, the equivalent would be a model-consumable field format that can't carry terminal control codes. JSON-RPC already separates structure from content; the problem is that the content strings inside JSON can carry arbitrary bytes.

**Scale amplifies stored AESI.** The article notes that "one poisoned record in a shared knowledge base can influence every later agent workflow" but doesn't fully draw out the implication: in a multi-agent system with shared memory — exactly the architecture that [[Agent Memory and Context]] and [[Context Graphs]] describe — a single stored AESI payload becomes a persistent backdoor across all agents, all sessions, all users. The blast radius isn't one conversation; it's the entire knowledge graph.

---

*Sources: [[raw/ansi-escape-injection-mcp]]*
*Last updated: 2026-07-21*
