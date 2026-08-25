---
url: https://github.com/grimmory-tools/grimmory
title: Grimmory
author: grimmory-tools (community fork of Booklore)
date_fetched: 2026-08-25
---

Grimmory is a self-hosted digital library server for "people who take their reading seriously" — an independent community fork of Booklore. It indexes and serves eBooks (EPUB, MOBI, AZW3, FB2), PDFs, comics (CBZ/CBR/CB7), and audiobooks (M4B, MP3, etc.) from a browser, with rule-based "smart shelves," metadata lookup from Google Books / Open Library / Amazon (plus Goodreads, Audible, ComicVine, Hardcover, Douban, and others), and per-user progress, annotations, and highlights.

It is a Java monolith — Spring Boot 4.1 on Java 25, MariaDB with 144 Flyway migrations — serving an Angular 21 frontend. The standout feature is protocol breadth: one catalog is exposed simultaneously as OPDS feeds, a Komga-compatible API, a native Kobo sync endpoint (including KEPUB and CBX conversion), and a KOReader sync API, so a Kobo, an OPDS app, and a comic reader all sync against the same library instead of their vendor clouds.

The backend (~91K lines of Java) is deliberately tuned for self-hosted scale: virtual threads, a Hikari pool capped at 5 connections, and book recommendations computed from hand-rolled 128-dimension feature-hashed vectors with brute-force cosine similarity stored in MariaDB — no vector database. BookDrop watches a folder for new files, fingerprints them with a sparse MD5, and queues them for import; a "pending deletion pool" debounces file moves so a rename doesn't read as a delete-plus-add.
