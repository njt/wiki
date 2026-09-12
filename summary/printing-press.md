---
url: https://printingpress.dev/
title: "Printing Press — Print the best agent-designed CLI of all time"
author: Matt Van Horn (@mvanhorn)
date_fetched: 2026-05-22
date_published: unknown
topics:
  - agent-architecture
---

# Printing Press

Printing Press generates agent-native CLIs from API specs, website HAR captures, or URLs. Each output includes a Go CLI, a Claude Code skill, an OpenClaw skill, and an MCP server. Two GitHub repos: the Press generator (2.4k stars) and the Library of 165 community-contributed CLIs (1.2k stars).

## Philosophy

Inspired by Peter Steinberger's work with discrawl and gogcli. Three design bets:
- Local SQLite mirrors over remote API calls
- Compound commands over multiple round trips
- Agent-native CLIs over raw HTTP

"Muscle memory for agents" is the framing. "Every API has a secret identity" — Discord as searchable knowledge base, Linear as team behavior observatory.

## Installation

Starter pack: `npx -y @mvanhorn/printing-press install starter-pack` (four CLIs: espn, flight-goat, movie-goat, recipe-goat)
Press binary: `go install github.com/mvanhorn/cli-printing-press/v4/cmd/printing-press@latest`
Skills: `git clone https://github.com/mvanhorn/cli-printing-press.git` (git pull for updates)

## Featured Magic Moments

- **flight-goat**: Non-stop flights over 8 hours from SEA, Dec 24 to Jan 1, cheapest first. Two sources stitched: Kayak nonstop search + sniffed Google Flights.
- **espn × flight-goat**: When OKC plays next + cheapest fly-in. Live ESPN context picks the date; FlightGoat books the route.
- **movie-goat**: Kelly Van Horn filmography sorted by Rotten Tomatoes. TMDb for filmography, OMDb for scores.
- **recipe-goat**: Chocolate cake search ranked by trust, 8 servings. Recipe-shaped output triggers Claude's cooking widget with scalable servings and timers.
- **linear**: Blocked issues whose blocker hasn't moved in 7 days. Raw SQL against local SQLite mirror. "50ms against the local SQLite mirror. Compound queries the Linear API can't answer."
- **contact-goat**: Find a verified email for someone you've never met. LinkedIn lookup → Happenstance for warm intros → Deepline for verified delivery.

## Key Quotes

- "Print the best agent-designed CLI of all time. From anything, or install and use the ones the community has made so far."
- "a local SQLite mirror beats a remote API call, compound commands beat ten round trips, and an agent-native CLI beats raw HTTP"
- "Two sources stitched into one query: Kayak nonstop search + sniffed Google Flights."
- "Compound queries the Linear API can't answer. 50ms against a local SQLite mirror."
- "Recipe-shaped output triggers Claude's cooking widget: scalable servings, in-step ingredient links, timers."

## Catalog — 165 CLIs by Category

Categories: AI (3), Cloud (4), Commerce (15), Developer Tools (20), Devices (4), Education (1), Food and Dining (12), Marketing (15), Media and Entertainment (27), Monitoring (1), Other (12), Payments (8), Productivity (20), Project Management (3), Sales and CRM (7), Social and Messaging (5), Travel (8).

Most CLIs include MCP server support (full or partial). Each is community-contributed with named authors. Auto-updates from the library repo when a README ships.
