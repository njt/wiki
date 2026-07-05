# Dev Containers

VS Code's Dev Containers extension turns Docker containers into full-featured development environments defined by a `devcontainer.json` spec file. It's infrastructure-as-code for dev environments: the container specifies the runtime, tools, extensions, and settings, and VS Code mounts your source code into it. Extensions run *inside* the container with full access to tools and platform -- not locally with remote bridging. Three quick-start paths: try a sample, open an existing folder, or clone a Git repo/PR into an isolated Docker volume.

---

## Key Quotes

> "The Dev Containers extension lets you use a container as a full-featured development environment."

Commentary: The framing matters. This isn't "run code in a container" -- it's "the container IS the development environment." This inverts the traditional model where your laptop is the environment and containers are the deployment artifact. Here, the container becomes the source of truth for tooling, and your laptop is just a window into it.

> "Features are self-contained, shareable units of installation code and dev container configuration."

Commentary: Microsoft is quietly building an app store for development environment components, distributed as OCI Artifacts. The Features system (Git, Azure CLI, GitHub CLI, etc.) is composable infrastructure-as-code at the dev-tooling level. This is the `apt-get` model applied to dev environments -- but with versioning, distribution, and a registry.

> "Pre-building is highly recommended" for "faster startup, simpler configuration, and supply chain security."

Commentary: The most architecturally interesting decision in the whole system. Pre-built images can embed `devcontainer.json` metadata as Docker image labels, making the container **self-describing**. A single `"image": "mcr.microsoft.com/devcontainers/go:1"` line inherits capabilities, post-create commands, and remote user config. This is the [[Specifications as the Product]] principle applied to dev environments: the image carries its own spec.

---

## Key Themes

#tool #concept #pattern

This is **reproducible development environments as infrastructure-as-code.** The `devcontainer.json` is the spec; the container is the implementation. New team members go from zero to working dev environment in one command. No README-driven setup, no "works on my machine."

The **Features ecosystem** (#marketplace) turns dev environment components into a composable, distributable format. It's an extension marketplace, but for toolchains instead of editor features.

The **self-describing image pattern** (metadata embedded as Docker labels) is the clever bit. It means you can publish a base image that carries all the configuration new projects need. The image *is* the template.

---

## Critical Analysis

**What's actually new here isn't containers -- it's the spec.** Docker's been around for a decade. What VS Code added is a standardized configuration format that makes dev environments reproducible, composable, and self-describing. The `devcontainer.json` is doing for dev environments what `package.json` did for Node dependencies: making them declarative, versionable, and shareable.

**The "Features" system is a marketplace play.** Microsoft learned from the VS Code extension marketplace that if you make it easy to publish and consume small, composable components, you get a network effect. Features are the same play applied to dev toolchains. Expect Microsoft to build curation, discovery, and eventually monetization around this.

**Pre-built images with embedded metadata is the mature pattern.** Raw `devcontainer.json` with Dockerfiles means every new team member builds from scratch. Pre-built images with embedded metadata mean they pull and run. For enterprises, this is also a supply-chain security win: you scan and approve images once, not per-developer. This is [[Compound Engineering]] applied to dev environments -- add a system (pre-built images) rather than manual review (checking each dev's setup).

**The limitations are honest but telling.** No Windows container support, all multi-root workspaces go in one container, Alpine has glibc issues with native extensions. These aren't incidental -- they reveal that dev containers are fundamentally a Linux-first tool, and that Microsoft's "any language, any platform" story has real edges.

**Why this matters for agents:** A reproducible, self-describing dev environment is the ideal harness for coding agents. You can spin up an agent in a dev container and know exactly what tools, runtimes, and extensions it has access to. The [[claude-code-config (Trail of Bits)]] already uses a devcontainer option for complete isolation. As coding agents become production infrastructure, "give me a dev container spec" will be as standard as "give me a CI config."

**The self-describing image pattern is under-exploited.** Embedding config as Docker labels means the image can answer "what capabilities do I need?" without a separate config file. This is the same insight as [[Designing a Passively Safe API]]: make the thing self-documenting so you can't forget to configure it. More tools should do this.

**But it's still VS Code lock-in.** The spec is "open" (containers.dev) but the only serious implementation is VS Code. JetBrains has Gateway, GitHub has Codespaces (which uses devcontainer.json), but the editing experience is tightly coupled to VS Code's extension model. If you're standardizing on dev containers, you're standardizing on VS Code as the editor.

---

See also: [[Windows in Docker]] (headless Windows for agent testing), [[floci]] (local AWS emulation via Docker), [[Security and Sandboxing]] (broader container sandboxing context), [[yolo-cage]] (agent sandboxing in containers), [[Navaris]] (container/microVM abstraction), [[Specifications as the Product]] (specs as durable artifacts), [[Compound Engineering]] (systems over manual review), [[Feedback Loop is All You Need]] (deterministic enforcement beats instructions), [[Building Agents for Production Systems with MCP]] (MCP as the standard agent-to-production layer), [[claude-code-config (Trail of Bits)]] (devcontainer for agent isolation), [[PiClaw]] (self-hosted agent in Docker), [[Designing a Passively Safe API]] (self-describing systems), [[Harness Engineering]] (dev containers as harness).

---
*Sources: [[summary/devcontainers]]*
*Last updated: 2026-05-14*
