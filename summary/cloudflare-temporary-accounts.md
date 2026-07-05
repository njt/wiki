---
url: https://blog.cloudflare.com/temporary-accounts/
title: Temporary Cloudflare Accounts for AI agents
author: Sid Chatterjee, Celso Martinho, Brendan Irvine-Broque
date_fetched: 2026-06-22
date_published: 2026-06-19
source_domain: blog.cloudflare.com
---

# Temporary Cloudflare Accounts for AI agents

Published June 19, 2026. Authors: Sid Chatterjee, Celso Martinho, Brendan Irvine-Broque. 4 min read.

## Summary

Cloudflare is rolling out Temporary Cloudflare Accounts for Agents, allowing AI agents to deploy websites, APIs, and agents without first signing up. Agents run `wrangler deploy --temporary` to deploy a Worker with a 60-minute lifetime, during which the human can claim the account. Unclaimed accounts auto-expire. This is one step in a broader push to eliminate the signup barrier for agents — cited alongside a Stripe partnership for agent-driven account provisioning and a WorkOS collaboration on OAuth-based agent auth.

## Why frictionless deployments matter

Three reasons:
1. Background AI sessions have no human in the loop — browser-dependent auth steps cause agents to get stuck or choose other platforms.
2. Agents need cheap, throwaway deployment targets for their write→deploy→verify loop.
3. Agent platforms are building deployment to "just work" without extra credentials, and users now expect this.

## How it works

Wrangler was updated to inform agents about the `--temporary` flag. When an agent runs `wrangler deploy` without auth, Wrangler's output tells the agent about the flag. On rerun with `--temporary`, Cloudflare provisions a temporary account, issues an API token, and provides a claim URL for the human.

### Full flow

1. User prompts a coding agent: "Make a hello world Cloudflare Worker in TypeScript and deploy it using wrangler"
2. Agent attempts deploy, Wrangler outputs instructions about `--temporary`
3. Agent reruns with `--temporary`, gets a temporary account + API token + claim URL
4. Agent deploys, curls the preview URL, verifies — all with no human involvement
5. For iteration: "Now change hello world to 'hello cloudflare' and redeploy" — agent modifies source, redeploys using the same temporary account
6. Human claims the account via the claim link (sign-up or sign-in), which includes Workers plus databases and other bindings
7. If unclaimed within 60 minutes, account auto-deletes

### Claiming

The claim link goes to a sign-up/sign-in page. Claiming preserves not just Workers but also resources like databases and other bindings created during the session.

## Broader context

The post references:
- A partnership with Stripe on a co-designed protocol allowing agents to provision Cloudflare on behalf of users (create accounts, start subscriptions, register domains, get API tokens) without copy-pasting tokens or entering credit card details.
- A collaboration with WorkOS on auth.md, letting agents provision accounts using existing OAuth standards.
- "Temporary accounts have some limitations, and their capabilities may change over time" — caveat directing readers to developer docs.

## Tags

Agents, Wrangler, AI, Cloudflare Workers
