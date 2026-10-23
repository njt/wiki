---
url: https://apron.chat/
date_fetched: 2026-10-24
---

# Apron Chat Protocol

Apron is a minimal protocol for self-hosted chats. Simple enough to write a server in ~100 lines, powerful enough for rooms, threads, bots, reactions, and more.

## 1.The Protocol

JSON over any transport, like WebSocket. Small core specification and optional incremental capabilities with graceful fallbacks.

{"method":"server", "params": {"apron": 9, "auth": ["guest"]}} {"method":"auth", "id": "c1", "params": {"scheme":"guest", "name":"Ada"}} {"id": "c1", "result": {"you": {"user_id": "guest_1234", "name":"Ada"}}} {"method":"message", "id": "c2", "params": {"body": {"text":"Hello"}}} {"method":"message", "params": {"room_id": "general", "from": {"user_id": "guest_1234"}, ...}} {"id": "c2", "result": {"message_id": "1724803200042"}}

## 2.Live Demo

Chat on the public demo server with restricted functionality, or point the web frontend at your own server.

## 3.Servers

Run one, read one, or write your own.

Goapron-server-goReference server for self-hosting. Every capability.
TypeScript · Workerapron-server-cloudflareRuns the public demo on the Cloudflare free tier.
Python · 1 filepython-minimalOne room, guests, history. uv run apron_server.py
Bun · 1 filejs-minimalThe same, no dependencies. bun apron-server.js

## 4.Clients

Any client works with any server.

SvelteKitapron-webThe web client behind the live demo.
botapron-pr-botPosts GitHub pull request activity to the demo server.

your bot
Two frames to post

{"method":"auth", "id": "c1", "params": {"scheme":"token", "token": "..."}} {"method":"message", "id": "c2", "params": {"body": {"text":"Deployed v1.4.2"}}}
