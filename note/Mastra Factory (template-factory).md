# Mastra Factory (template-factory)

Mastra Factory is the AI company behind the Mastra agent framework shipping its own opinionated **software factory**: an open-source server you scaffold with `npm create factory`, point at a repository, and let turn GitHub issues (or Linear/Jira/Slack requests) into plans, implementations, and reviewed pull requests. The repo is worth reading not for its code — it's a thin template — but for what the single entry file reveals about how Mastra has factored the factory problem into composable platform services.

---

The repository itself contains almost no code: a README, an `AGENTS.md` telling coding agents to load the `mastra` skill before touching anything, and one entry file. Everything else is pulled from `@mastra/factory`, `@mastra/core`, and `@mastra/pg` at `latest`. This is a template-as-product move — the repo's job is to pin a deployable configuration, not to contain logic.

## Architecture

`src/mastra/index.ts` (~100 lines) is the entire repo's code, and its comments are unusually honest about *why* it's shaped that way:

- **One entry, one env read.** The module is "the ONE place deployment env is read", mapping env vars onto explicit `MastraFactory` config — instances for behaviors (pubsub, storage, vector), plain values for config (publicUrl, origins). Anyone reading the entry sees exactly which env var feeds which slot.
- **`prepare()` / `finalize()` split.** `mastra build`'s deployer validates that a literal `new Mastra(...)` exists in the entry file (a Babel plugin, `checkConfigExport`), so `factory.prepare()` returns *constructor args* rather than an instance, and the literal call happens in the entry with storage, pubsub, workers, and workflows spread explicitly — static analysis can then detect the worker topology. `finalize()` boots the controller (which inherits the Mastra instance's storage) and starts its workers via top-level await, so the deployer imports a fully-booted instance.
- **One database, many roles.** A single `PgFactoryStorage` powers agent storage, the factory app tables, a *distributed project lock*, and better-auth; the paired `PgVector` rides the same `DATABASE_URL` for recall search. Consolidating auth, state, locking, and embeddings on one Postgres is a deliberate small-team simplification.
- **Boards, not global config.** "Lifecycle, transition-policy, and tool-result rules are owned by the installed boards (the built-in Work and Review boards here); nothing is configured globally." The issue→investigate→plan→implement→PR-review pipeline is packaged as two boards you install, not YAML you write.
- **Fail-fast integration config.** Partial Slack credentials throw at startup with a message naming exactly which of the four tokens is missing, and account linking requires HTTPS — incomplete configuration fails loudly rather than silently disabling Slack.
- **Auditable deployments.** A `configVersion` string is stamped onto audit rows and deferred decisions so behavior can be traced back to the deployment that produced it; you bump it when handler behavior changes.

## Key techniques

- **Credential encryption at rest.** `createFactorySecretEncryption` with a versioned primary key (`id: 'v1'`) encrypts stored model-provider keys and integration secrets; the README insists the key survive restarts with a protected backup because losing it makes stored credentials unreadable.
- **Sandbox swapping as a constructor argument.** Omit `sandbox` and the factory picks local sandboxes under `mastra dev` and platform sandboxes in deployment (gated on `MASTRA_PLATFORM_SECRET_KEY`/`MASTRA_PROJECT_ID`/`MASTRA_ENVIRONMENT_ID`); pass `new LocalFactorySandbox()` to pin one. `sandboxStart: 'eager'` pre-warms them.
- **GitHub App + per-user intake.** Repository access flows through a GitHub App, and each *teammate* chooses their own issue sources under Work Intake — intake is personal, not just org-level.
- **AGENTS.md as onboarding.** The repo's own `AGENTS.md` mandates loading the live `mastra` skill because "APIs change between versions" — the project is built by its own kind of agent and encodes that discipline in its own config.

## Design decisions

The template optimizes for **deployability and auditability over hackability**. Logic lives in versioned npm packages at `latest` (with `minimumReleaseAgeExclude` exempting `@mastra/*` from supply-chain delay), so the repo trades control for always-current behavior. The strict entry-file contract — literal `new Mastra`, explicit worker properties, top-level-await boot — sacrifices conventional module structure so a build tool can statically verify deployment topology. Consolidating four concerns on one Postgres sacrifices scale headroom for operational simplicity, which matches the product's framing (one factory server per team, on platform or one VM).

The notable bet is that the factory pattern has matured enough to be *scaffolded*: gates, boards, intake, sandboxes, and review come pre-built, and your configuration choices are auth/storage/sandbox/integrations — exactly the swap-independently axis the README's configuration table is organized around.

## Comparison notes

- Unlike [[How to Build an AI Software Factory]], which synthesizes industry factories into a five-stage control plane you assemble yourself, Mastra Factory ships those stages as installable boards — the buy-line crossing Paul Iusztin predicted in [[Inside a Software Factory]].
- It complicates [[The Case Against Building Your Own Agent Platform]]: this is a vendor arguing you *shouldn't* build the platform — and packaging memory (pgvector recall), governance (audit rows, configVersion), and orchestration (controller + workers) as the four underestimated components, ready-made.
- Compared to [[TEHAS — A Pi Software Factory]] or [[Self-Hosted Sandboxed Agentic Software Factory]], the self-hosted single-binary factories, Mastra's template pushes sandboxes and auth to a hosted platform by default, with local execution as the explicit fallback.
- Against [[Paperclip]]'s heterogeneous-CLI control plane, Mastra controls the whole stack (agents, storage, sandboxes) rather than normalizing external agent CLIs — more integration, less flexibility.

#tool #project #agents #software-factory #orchestration

---
*Sources: [[raw/template-factory]], [[summary/template-factory]]*
*Last updated: 2026-11-20*
