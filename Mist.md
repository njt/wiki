# Mist

Google Docs for Markdown. Real-time collaborative editing in the browser, no accounts required. Drag and drop a file, or `curl https://mist.inanimate.tech/new -T file.md` from the terminal.

---

## Key Quotes

> "Share and edit Markdown together, quickly."

## Key Themes

#tool #markdown #collaboration #writing

The appeal is radical simplicity. No sign-up, no vaults, no configuration. Upload a Markdown file (or start fresh) and share the URL. Multiple people can edit simultaneously. It's the Google Docs model applied to the one format developers actually use.

The `curl` interface for creating new documents from the terminal is a nice touch -- it means agents and scripts can create collaborative documents programmatically.

## Critical Analysis

Work in progress, and it shows -- the feature set is minimal. The question is whether "Google Docs for Markdown" needs to be a separate product or whether it's a feature that belongs inside existing tools. HackMD, CodiMD, and Hedgedoc all offer collaborative Markdown editing with more features. Mist's bet is that simpler is better.

Compared to [[MarkText]] (a desktop Markdown editor), Mist trades features for instant collaboration. Compared to Obsidian (where this wiki lives), it trades the vault model for ephemeral shared documents. Different tools for different moments.

The MIT license and minimal design make it a good candidate for self-hosting if you want collaborative Markdown without sending content to a third party.

---
*Sources: [[raw/mist]]*
*Last updated: 2026-05-14*
