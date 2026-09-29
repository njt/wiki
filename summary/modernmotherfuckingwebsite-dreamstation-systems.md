---
url: https://modernmotherfuckingwebsite.dreamstation.systems/
title: "This is a modern motherfucking website"
author: Robin Reel
date_fetched: 2026-09-29
date_published: 2026-09-24
topics:
  - software-engineering-craft
  - ideas-and-culture
---

A profanity-laced sequel to the 2013 "motherfucking website" screed, updated for the meta-framework era. The author, Robin Reel, argues that everything developers reach for toolchains to accomplish — accessibility, SEO, structured data, social cards, performance, responsive images, dark mode — is natively a platform feature of the browser, and that the framework ecosystem has merely hidden those features behind plugins so people forget they were free.

The piece is structured as a tour of platform primitives: semantic landmarks (`<nav>`, `<main>`, `<dialog>`, `<details>`), JSON-LD structured data as a plain script tag, seven Open Graph meta tags, resource hints (`preconnect`, `preload`, speculation rules for prerendering), `<picture>` with AVIF/WebP, and modern CSS. The killer argument: client-rendered SPAs inject meta tags after crawlers leave, then bolt on server-side rendering to get back what writing HTML gave you for free.

The concession is honest and worth keeping: if you're building an actual application (spreadsheet, video editor), use a framework. But most of the web is documents, "the web was invented for documents," and the whole methodology is "Start with HTML. Add what you need. Stop when you're done." The page itself is a ~23 KB single HTML file with no dependencies, offered as proof of concept.
