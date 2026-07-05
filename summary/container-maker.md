---
url: https://github.com/UPwith-me/Container-Maker
title: "Container-Maker: The Ultimate Developer Experience Platform for the Container Era"
author: JIPEI HE (UPwith-me), Renmin University of China
date_fetched: 2026-05-14
date_published: 2026-01-10 (v3.1.0)
---

# Container-Maker (cm)

A Go CLI tool that fuses "the speed of Makefiles, the isolation of Docker, and the intelligence of VS Code DevContainers" into a zero-config developer experience platform. 23 stars, 78 commits, dual-licensed under AGPL-3.0 and a commercial license. Latest release: v3.1.0 (Jan 10, 2026). Single developer: JIPEI HE, Renmin University of China.

## Core Concept

The `devcontainer.json` file is the single source of truth — eliminating separate Dockerfiles, Makefiles, and shell scripts. Built on Docker BuildKit for aggressive layer caching.

## Key Features (v3.0+)

- **Zero-config onboarding:** `cm setup` auto-installs Docker/Podman per OS
- **Environment diagnostics:** `cm doctor` checks runtime, GPU, network, disk
- **Project initialization:** `cm init --template <name>` from 30+ templates
- **AI config generation:** `cm ai generate` analyzes project files, suggests configs
- **AI build debugging:** `cm ai debug` and `cm ai optimize`
- **Local AI:** `cm ai local generate` via Ollama (no API key required)
- **Template marketplace:** `cm marketplace search/install`
- **VS Code integration:** `cm code` launches with DevContainer support
- **Multi-service workspaces:** `cm workspace` orchestrates microservices
- **Brownfield migration:** `cm import docker-compose.yml`
- **GPU management:** `cm gpu list/status/allocate`
- **Cloud control plane:** 14+ providers, GPU instances (T4, A10, A100), web dashboard, billing
- **Policy as Code:** `cm policy check` with built-in security rules
- **SBOM generation:** `cm sbom` (CycloneDX format)
- **Resource profiling:** `cm profile start/stop --report` suggests P95-based limits
- **Security scanning:** `cm scan` (Trivy integration)
- **File watching:** `cm watch --run "<command>"`
- **Service mocking:** `cm mock quick/serve`

## v3.1.0 (Industrial Edition)
- **Plugins:** `cm plugin install`
- **Environment Snapshots:** `cm snapshot create/restore`
- **Offline Delivery:** `cm export/load` for air-gapped environments

## Template Library

30+ templates: pytorch, tensorflow, huggingface, jupyter, miniconda, python-poetry, python-pipenv, cpp-conan, cpp-vcpkg, cpp-cmake, java-maven, java-gradle, dotnet, php-composer, node, react, nextjs, python-web, go, rust, cpp, terraform, kubernetes, ansible.

## Caching Claims

Go (up to 10x), Node.js (up to 5x), Rust (up to 8x), Python (up to 3x), Java (up to 4x).

## Team/Enterprise Features

Repository management, auth, version pinning, global variables, audit logging.

## Feature Maturity

Most features "Stable"; Beta: `cm plugin`, `cm scan`, `cm cloud`, `cm marketplace`.

## Installation

Prebuilt binaries (Windows/Linux/macOS), `go install github.com/UPwith-me/Container-Maker/cmd/cm@latest`, or build from source. Requires Go 1.21+.
