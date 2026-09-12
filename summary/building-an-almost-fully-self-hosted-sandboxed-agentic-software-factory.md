---
url: https://blog.jakesaunders.dev/building-an-almost-fully-self-hosted-sandboxed-agentic-software-factory/
title: Building an almost fully self-hosted sandboxed agentic software factory
site: blog.jakesaunders.dev
author: Jake Saunders
date_fetched: 2026-09-04
date_published: null
topics:
  - agent-coding-workflow
---

Jake Saunders set out to build a fully remote agentic development environment where the LLM is *structurally contained* rather than merely trusted — give it one instruction and have it autonomously walk the whole SDLC (research, code, tests, commit, CI, deploy) on his home server, at no ongoing cost beyond a £20/month Codex subscription. The tl;dr is that it worked: from a single prompt it created a repo, wrote an app and its tests, got CI green, provisioned Postgres, and deployed the finished app behind HTTPS without another message.

The stack is glued together with Coolify (a self-hosted, Heroku-style PaaS over Docker): Pi-hole for local DNS, Tailscale so the home network "follows him around," Forgejo (with runners) for self-hosted Git and CI, Hermes (an OpenClaw-style assistant using Codex for inference) as the agent, Telegram for chatting from a phone, self-hosted Firecrawl for web scraping/SERP access, and Porkbun + Let's Encrypt for a domain and on-the-fly SSL. He chose Forgejo over GitHub partly because handing the box a GitHub token would undermine the isolation.

Isolation is the point, and it's layered. First, the agent runs on its own sacrificial metal — a second-hand eBay box (2021 i7, 32 GB RAM) that could be `rm -rf`'d at the cost of a couple hours' rebuild. Second, the network: no port-forwarded external ingress, so there's no attack surface or internet background radiation. Services are reached only over the tailnet, and SSL certs are issued via DNS-01 through the Porkbun API — so a "ghost service" gets a valid HTTPS URL with *no public A/AAAA record* pointing at it (the hostname may still appear in certificate-transparency logs). Coolify does this on the fly for any subdomain the agent creates.

The demo prompt asked for a MyFitnessPal-like calorie tracker — full-stack SvelteKit with Drizzle and Postgres, Tailwind, mobile-first, deployed via Docker Compose at `calories.internal.jakeshomelab.me`. The agent bootstrapped the repo, wrote app and tests in sensible commits, built a CI pipeline, worked through failures until green, containerised the app plus its own Postgres, and deployed — no nudging, no copy-pasting error messages. One follow-up prompt fixed a CSRF bug (adding regression tests and redeploying).

Saunders is candid that this isn't harmless: Hermes can still nuke the box, delete repos and databases, leak or abuse any credentials it's been given, burn tokens, make arbitrary outbound requests, or poke anything else on the network the firewall allows. What he's done is make the machine sacrificial and sharply limit what he cares about that's within reach — the failure mode becomes "rebuild the eBay box and rotate a handful of keys" rather than "discover an LLM has reorganised my laptop." Next steps: put the box on its own VLAN, scope and rotate every credential, automate backups and rebuilds, and require approval before anything genuinely public or hard to undo — while noting that enough approval gates turn the magical factory back into forms.
