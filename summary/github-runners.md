---
url: https://github.com/bastosmichael/github-runners
title: "github-runners"
author: Michael Bastos
date_fetched: 2026-10-10
date_published: 
topics:
  - developer-tools
  - security-and-sandboxing
---

Michael Bastos' `github-runners` is a Terraform + Docker Compose project that turns a single Ubuntu box into a self-hosted GitHub Actions runner fleet. One `terraform apply` bootstraps the host (Docker install, zRAM + swap file, disk-guard cron, log rotation), ships a runner container image, and starts ten runner containers split into two tiers — 2 "heavy" (4 CPUs / 8 GB) and 8 "light" (2 CPUs / 4 GB) — with a Portainer UI alongside.

The signature move is credential handling: no token is ever copy-pasted or stored in Terraform state. Terraform pulls the local `gh auth token`, verifies it can mint registration tokens, and writes it to a mode-600 `.env` on the host. Each runner container mints its own short-lived registration token at startup via `gh api`, and unsets the credential from the environment before launching the listener so workflow jobs can never read it. Runners register as ephemeral (one clean job each, auto-removed by GitHub) and deregister themselves on shutdown with a freshly minted remove token.

The repo is unusually opinionated about operational failure modes: a registration health gate fails the apply if runners don't come online (because `docker compose up -d` exits 0 even when every container is crash-looping); replica counts live in `deploy.replicas` fed from `.env` so a manual `docker compose up` doesn't collapse the fleet; `daemon.json` is only rewritten when content changes so re-applies don't bounce dockerd and kill in-flight jobs; the runner binary is baked into the image (215 MB once instead of 2.1 GB for ten runners); and `--disableupdate` is deliberately not passed so an aging runner version self-updates instead of going dark when GitHub deprecates it. Resource limits are chosen from measured cgroup throttling data, and the README is candid that the defaults oversubscribe the host deliberately — ceilings, not reservations.
