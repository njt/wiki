# QR Generator (delphi.tools)

A single-purpose web tool for generating styled QR codes with live preview, batch mode, and extensive visual customization — part of delphi.tools, an indie web tools collection built on "No logins. No tracking. Long live the handmade web."

---

## Key Quotes

> "No logins. No tracking. Long live the handmade web."

The site's tagline and the whole thesis in eight words. Every feature — no auth, no analytics, flat pages — follows from this commitment. It's a reminder that the web was once a collection of tools you could just *use*, before we convinced ourselves every utility needed a user table, a pricing page, and an onboarding flow.

> Quick Styles: Classic, Rounded, Dots, Classy, Indigo, Rose, Teal

The style names reward a close reading. "Bouba" and "Kiki" for bit shapes are a direct reference to the psychological effect where rounded shapes map to "bouba" and angular shapes to "kiki" across cultures and languages. Someone on this team is having fun, and that playfulness is the signature of good indie tool design — it signals that the maker built this because they *wanted* to, not because a product manager prioritized it.

> Error Correction: L, M, Q, H

The tool exposes QR code error correction levels directly to the user. This is the right call — higher correction lets you embed logos at the cost of density, and the user needs to make that tradeoff based on their use case. Most QR generators hide this behind a "quality" slider or omit it entirely. Exposing the mechanism is honest tool design. Andrew T's [[Dithered QR Codes]] spends that same correction budget in the opposite direction — shrinking the data modules and dithering a photo into the freed space, trading scannability for aesthetics rather than preserving it.

## Key Themes

#tool #web #design #qr-code #indie-web

### The Handmade Web as Counter-Current

delphi.tools embodies a philosophy that's become rare: single-purpose utilities you can visit, use, and leave. No sign-up. No state. No analytics script weighing down the page load. The site is a collection of dozens of tools organized by category, each one a self-contained page with a clear job.

This is the web that [[Simplicity in the Age of AI-Assisted]] argues for — stripping away inherited complexity until only the essential remains. The difference: delphi.tools never accumulated the complexity in the first place. It's simplicity by design, not simplicity by demolition.

### One Tool, One Job

The QR Generator does exactly what it says and nothing else. It doesn't try to be a design platform, a marketing tool, or a QR analytics dashboard. [[Handy]] takes the same approach: "one tool, one job." The interface reflects the constraint: enter content, customize appearance, export. The live preview keeps the feedback loop tight — you see what you're getting before you commit to an export.

### Exposing the Mechanism

Where most QR tools abstract away the technical details, this one surfaces them: error correction levels by name (not a vague "quality" slider), bit shapes with precise terminology, separate controls for eyes and pupils. This is a power-user pattern — giving direct access to the mechanism rather than hiding it behind a simplified interface — and it connects to the argument in [[Two Kinds of User Are Emerging]] that power users want direct capability, not wrappers.

### Tool Design as Hospitality

The batch mode is the quiet killer feature. Anyone who's ever needed to generate QR codes for 50 conference badges knows the pain of single-mode-only tools. Including batch mode signals that the maker thought about actual use cases, not just the feature checklist. This is what [[Intent Is the Interface]] gets right: when you design from capabilities rather than screens, batch generation is obviously the same capability with a different cardinality.

## Critical Analysis

The tool is good. The philosophy behind it is better.

**What works:** The customization range is genuinely impressive for a free web tool. Six bit styles, three eye shapes, two pupil shapes, custom colors, and logo embedding — this covers the design space that most people actually need. The live preview eliminates the generate-inspect-regenerate loop that plagues form-based QR tools. The export options (PNG, SVG, copy) cover real workflows: PNG for documents, SVG for further editing, copy for quick pasting.

**What's missing:** No bulk import for batch mode. If I have a CSV of 200 URLs, I'm pasting them one at a time or reaching for a CLI tool. No URL validation before encoding — you get a QR code that scans to a broken or mistyped URL. No estimated capacity indicator — at higher error correction and with logos, QR codes run out of data capacity, and the tool doesn't warn you before you've committed to an unencodeable content string.

**The bigger picture:** delphi.tools matters beyond this one tool. It's a living counterexample to the default assumption that web tools need accounts, tracking, and monetization. In a world where [[AI Killing B2B SaaS]] debates whether AI will eat the SaaS industry, these indie tools have already opted out of that game entirely. They don't need to be disrupted because they never joined the disruption economy. The site runs on the oldest business model on the web: someone wanted it to exist, so they made it.

The tension: can the handmade web survive in a world where AI generates polished alternatives at near-zero cost? The answer is probably yes, for the same reason handmade furniture survives IKEA. The craft is the point. "No logins. No tracking" is a trust claim that an AI-generated clone can't make without being auditable — and auditing requires access to the source, which is exactly what the handmade web provides.

## Cross-Links

- [[Simplicity in the Age of AI-Assisted]] — the philosophical companion; these tools never accumulated the complexity LLMs let you demolish
- [[Handy]] — same "one tool, one job" ethos applied to speech-to-text
- [[AI Killing B2B SaaS]] — indie tools as the pre-existing alternative to bloated SaaS platforms
- [[Two Kinds of User Are Emerging]] — exposing mechanisms (error correction levels, bit shapes) is a power-user pattern
- [[Intent Is the Interface]] — single-purpose tools nail the interface because intent is unambiguous
- [[Software Engineering Craft]] — fundamentals of tool design that don't change regardless of technology
- [[Doing]] — another local-first, no-cloud tool built on "just work" philosophy

---
*Sources: [[summary/qr-genny-delphi-tools]]*
*Last updated: 2026-05-18*
