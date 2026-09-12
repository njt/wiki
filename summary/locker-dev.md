---
url: https://locker.dev/
title: Locker
author: zmeyer44
date_fetched: 2026-05-15
date_published: 2026
topics:
  - misc
---

# Locker

Open-source, self-hostable file storage and knowledge platform. A Dropbox/Google Drive alternative where you bring your own storage (local disk, S3, R2, or Vercel Blob) and switch providers with a single environment variable.

Tagline: "Your files, Your cloud, Your rules."

GitHub: https://github.com/zmeyer44/Locker

## Key Features

- File Explorer with grid/list views, in-app previews for PDFs, images, Markdown, CSV, audio, video, and plain text
- Tags: color-coded, workspace-scoped file tags with filtering
- Share Links: password-protected, expiring links with download limits. Recipients don't need an account
- Upload Links: non-users can send files to your storage
- Command Palette: Cmd+K search across file names and content
- Knowledge Base: AI-powered wiki that ingests tagged documents, supports chat, and visualizes page relationships in an interactive graph view
- Plugins: extensible plugin system with built-in search (QMD, FTS), document transcription, Google Drive sync, and knowledge base
- Document Transcription: AI-powered OCR/transcription for images and PDFs
- Multi-Store Storage: attach multiple backends per workspace with primary, replica, and read-only configurations
- Read-Only Ingest: scan external buckets/directories and import without moving originals
- Storage Quotas: per-user limits with usage tracking
- Virtual Bash Shell: navigate files with ls, cd, cat, grep, find via just-bash
- Workspace teams with role-based access, email/password and Google OAuth auth, API keys with tRPC type safety

## Tech Stack

Next.js 16 (App Router, Turbopack), Turborepo + pnpm workspaces, PostgreSQL 16 + Drizzle ORM, tRPC 11, BetterAuth, Vercel AI SDK, Tailwind CSS 4 + Radix UI, Playwright E2E tests, TypeScript strict mode.

## Self-Hosting

Docker Compose (recommended), Railway, or Fly.io. Auto-detects platform and adjusts capabilities. Serverless (Vercel) disables long-running ops. Needs PostgreSQL 16+, BETTER_AUTH_SECRET, and a storage provider.

## Pricing

Completely free and open source. You pay only for your infrastructure.
