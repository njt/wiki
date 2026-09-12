# Portless

A Vercel Labs tool that replaces `localhost:3000` with `https://myapp.localhost` -- stable, named URLs for local development. Designed explicitly for both humans and AI agents.

---

## Key Quotes

> "Replace port numbers with stable, named .localhost URLs. For humans and agents."

## Key Themes

#tool #developer-experience #local-development

The "for agents" part is the interesting bit. When an AI coding agent spins up a dev server, it gets a random port number. If it needs to interact with that server later (or hand it off to another agent), the port number is ephemeral context that gets lost between sessions. A stable name like `myapp.localhost` is a fixed reference point.

The git worktree support is clever -- it auto-prepends branch names as subdomains, so `feature-x.myapp.localhost` and `main.myapp.localhost` can coexist. This maps directly to the parallel-branch workflows in [[Agentic Coding]] where multiple agents work on different branches simultaneously.

HTTPS by default with auto-generated certificates eliminates a whole class of "works in dev, breaks in prod" bugs around secure cookies, CORS, and mixed content.

## Critical Analysis

This solves a real paper-cut. Port numbers are cognitive overhead that compounds in microservice architectures. The Tailscale integration for LAN sharing is a nice touch -- share your dev server with a colleague without ngrok.

The limitation: it's another proxy layer. When something breaks, you're now debugging through one more hop. And the `.localhost` TLD trick requires modern browser support (RFC 6761). Most developers won't hit issues, but edge cases in corporate proxy environments could be painful.

For Nat's setup specifically, the Tailscale integration is interesting -- his server already uses Tailscale for internal services. [[Tailport]] attacks the same paper-cut from the exposure side rather than the naming side: it keeps the port number and toggles tailnet (`serve`) or public (`funnel`/Caddy) reachability per port instead of minting a stable name.

---
*Sources: [[summary/portless]]*
*Last updated: 2026-05-14*
