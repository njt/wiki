# GitHub Runners Self-Hosted Fleet

`bastosmichael/github-runners` is a ~900-line Terraform + Docker Compose project that turns one Ubuntu box into a self-hosted GitHub Actions runner fleet with a single `terraform apply`. Its interest is not scale — ten containers on one host — but the density of operational judgment per line: credential hygiene that treats the runner's own job environment as a threat surface, resource limits derived from measured cgroup throttling counters, and a deployment designed to fail loudly instead of reporting success over a dead fleet.

---

## Architecture

Three layers, each small:

- **Terraform (`infra/main.tf`, 434 lines)** — two `null_resource`s. `bootstrap_docker` ships one remote bash script that installs Docker, zRAM (zstd, priority 100), a 16 GiB swap file, a journald cap, a `disk-guard` cron, and `daemon.json` with log rotation. `deploy_stacks` scp's the compose files to the host, resolves the GitHub credential locally, and runs a post-deploy **registration health gate** polling the GitHub runners API until every expected runner reports `online`. A destroy-time provisioner brings the stacks down so containers can deregister before the credential file is deleted.
- **Compose stack (`infra/stacks/github-runner/docker-compose.yml`)** — one shared image, two services: `github-runner-heavy` (replicas 2, `cpus: 4`, `mem_limit: 8g`) and `github-runner-light` (replicas 8, 2 CPUs, 4 GB). Replica counts are fed from `.env` into `deploy.replicas`, so a manual `docker compose up -d` on the host doesn't silently collapse each tier to one runner.
- **Container (`Dockerfile` + `start.sh`, 145 lines)** — the whole lifecycle lives in one bash entrypoint: mint registration token → `config.sh --ephemeral` → run listener → on exit, mint a remove token and `config.sh remove`.

## Key techniques

- **Credentials never enter the job environment.** `start.sh` copies `GITHUB_PAT` into a plain shell variable and `unset`s it *before* launching the listener, because the Actions listener passes its environment down to every job step — a step running `env` would otherwise print an org-scoped credential into the workflow log. The PAT is exported only command-scoped to `gh api` inside `mint_token()`. The long-lived credential stays in `/opt/github-runner/.env`, mode 600, never committed and never in Terraform state (only its sha256 appears in triggers).
- **Self-minting short-lived tokens.** Registration tokens expire in an hour, so each container mints one fresh at startup and a remove token at shutdown. There is no manually generated token to go stale, so re-applies always work.
- **Fail-loudly health gate.** The apply polls the runners API with a Python one-liner counting online runners matching the name prefix, timing out after 900 s and dumping compose logs on failure. The README's rationale is exactly right: `docker compose up -d` exits 0 even when every container is crash-looping.
- **Measured resource limits.** The light tier's 2 CPU / 4 GB defaults came from reading `/sys/fs/cgroup/.../cpu.stat` during a real node build: 362 throttle events, 26.8 s of throttled time against 78.5 s consumed — a quarter of CPU demand denied on a 96%-idle host — and a 1.35 GB peak too close to a 2 GB OOM-kill line. The README ships the diagnostic loop for readers to repeat on their own workloads.
- **Idempotent bootstrap.** `daemon.json` and zram config are rewritten only when content differs (`cmp -s`), so a re-apply doesn't restart dockerd and kill in-flight jobs. Swap is rebuilt when the size variable changes.
- **Self-healing versioning.** `--disableupdate` is deliberately omitted: GitHub refuses connections from deprecated runner versions, which would kill pinned-version listeners on startup. `RUNNER_VERSION` only sets the initial build-time download; the fleet self-updates afterward.
- **Baked-in binary + workdir wipes.** The ~215 MB runner tarball is downloaded once at image build (not once per container: 215 MB vs 2.1 GB for ten runners), and `_work`/`_diag` are wiped at startup because an ephemeral container's writable layer persists across jobs.

## Design decisions

- **Oversubscription on purpose.** `cpus` and `mem_limit` are ceilings, not reservations; ten runners summing to 36 CPUs / 56 GB on a 16 GB / 16-thread box means any single job can burst while the fleet is mostly idle, with zRAM (`vm.swappiness=100`) and disk swap absorbing contention. The sacrifice is predictability under simultaneous load — the README states the tradeoff plainly rather than hiding it.
- **Deliberate disk-reclaim layering.** Root LV grown into free VG space (Ubuntu defaults to 100G even on bigger disks), journald capped, container logs rotated, and a 15-minute `disk-guard` cron pruning at 80% and aggressively at 90% — but never touching running containers or checked-out jobs.
- **Terraform-as-orchestrator honesty.** Everything is `null_resource` + local-exec/SSH; no Docker or GitHub provider. The author accepted Terraform's awkwardness as a remote-execution tool in exchange for one deployable unit and state-tracked change detection (file hashes in triggers re-run the deploy when any compose file changes). Destroy provisioners mirror values into `self.triggers` and use `lookup()` defaults because destroy evaluates against old state.
- **Honest trust boundary.** Injecting the host's docker group gid so jobs get the Docker socket grants every job root-equivalent control of the host. The README says so and confines the project to trusted workflows rather than pretending otherwise.

## Comparison notes

- [[Self-Hosted Sandboxed Agentic Software Factory]] builds a homelab automation factory on a sacrificial box with no external ingress; this project is the CI counterpart — same "one beefy home machine, infrastructure-as-code, everything self-heals" philosophy, but for Actions runners rather than agent sandboxes.
- [[How Nango Runs Untrusted Customer Code at Scale]] treats job code as hostile and builds layers of isolation; this project explicitly assumes the opposite — trusted org workflows — and its security story is about credential hygiene (environment unsetting, mode-600, short-lived tokens) rather than containment. The two define opposite ends of the self-hosted-runner trust spectrum.
- [[We Built a Scalable Agent Sandbox]] optimizes for running unknown agent code safely in cloud infrastructure; github-runners shows what the trusted-code version looks like when you care more about idle-fleet burst capacity than isolation.
- [[Why I Run Enterprise-Grade CI at Solo-Founder Scale]] makes the argument for heavy self-hosted CI practice at tiny scale; this repo is the concrete Terraform-able artifact behind that argument.

---
*Sources: [[raw/github-runners]], [[summary/github-runners]]*
*Last updated: 2026-10-10*
