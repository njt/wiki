---
url: https://code.visualstudio.com/docs/devcontainers/containers
title: "Developing inside a Container"
author: Microsoft / Visual Studio Code documentation team
date_fetched: 2026-05-14
date_published: 2026-05-13
topics:
  - developer-tools
---

# Developing inside a Container

The Dev Containers extension enables using a container as a full-featured development environment. A `devcontainer.json` file instructs VS Code on how to access or create a "development container" with a specific tool and runtime stack. Workspace files are mounted from the local file system or copied/cloned into the container. Extensions install and run inside the container, giving them full access to tools and platform.

Two operating models: using a container as a full-time dev environment, or attaching to a running container for inspection. The extension supports the open Dev Containers Specification.

## System Requirements

- **Windows:** Docker Desktop 2.0+ on Windows 10 Pro/Enterprise. Windows 10 Home (2004+) needs Docker Desktop 2.3+ with WSL 2 back-end. Docker Toolbox and Windows container images unsupported.
- **macOS:** Docker Desktop 2.0+
- **Linux:** Docker CE/EE 18.06+ and Docker Compose 1.21+. Ubuntu snap package unsupported.
- **Remote hosts:** 1 GB RAM minimum; 2+ GB and 2-core CPU recommended.
- **Containers:** x86_64/ARMv7l/ARMv8l Debian 9+, Ubuntu 16.04+, CentOS/RHEL 7+, x86_64 Alpine Linux 3.9+.

## Installation

Install Docker, VS Code or VS Code Insiders, and the Dev Containers extension (or Remote Development extension pack).

Git tips: consistent line endings across platforms; SSH keys can be shared with the container.

## Quick Starts

1. **Try a sample:** Via the Containers tutorial, pick a sample, walk through Docker setup.
2. **Open existing folder:** Run "Dev Containers: Open Folder in Container..." from Command Palette, pick a starting point (Template, Dockerfile, or Docker Compose file). VS Code adds `.devcontainer/devcontainer.json`, reloads, builds the container.
3. **Clone repo in volume:** Uses isolated Docker volumes instead of bind-mounts for better Windows/macOS performance. Works with GitHub PRs directly -- the PR is checked out and the GitHub Pull Requests extension auto-installs.

Templates come from the first-party and community index at `containers.dev`.

## devcontainer.json

Configuration stored in `.devcontainer/devcontainer.json` or `.devcontainer.json`. Command "Add Dev Container Configuration Files" offers pre-defined configurations.

```json
{
  "image": "mcr.microsoft.com/devcontainers/typescript-node",
  "forwardPorts": [3000],
  "customizations": {
    "vscode": {
      "extensions": ["streetsidesoftware.code-spell-checker"]
    }
  }
}
```

## Dev Container Features

Features are self-contained, shareable units of installation code and dev container configuration. Appear as options when adding configuration files (e.g., Git, Azure CLI). After rebuild, Features appear in `devcontainer.json`:

```json
"features": {
    "ghcr.io/devcontainers/features/github-cli:1": {
        "version": "latest"
    }
}
```

"Always installed" Features via `dev.containers.defaultFeatures` user setting. Features can be created and distributed as OCI Artifacts. Part of the open-source Development Containers Specification.

## Pre-Building Dev Container Images

Pre-building is recommended for faster startup, simpler config, and supply-chain security. Dev Container CLI, GitHub Action, or Azure DevOps task can build. Images can be pushed to registries (ACR, GitHub Container Registry, Docker Hub).

Metadata can be embedded in prebuilt images via image labels, allowing simplified `devcontainer.json`:

```json
{
  "image": "mcr.microsoft.com/devcontainers/go:1"
}
```

Dockerfile label:
```
LABEL devcontainer.metadata='[{"capAdd":["SYS_PTRACE"],"remoteUser":"devcontainer","postCreateCommand":"yarn install"}]'
```

## Extension Management

Extensions run locally (UI/client side) or in the container. Themes and snippets are local; most others reside inside the container. Extensions can be added to devcontainer.json by right-clicking. Opt out with minus prefix: `"-dbaeumer.vscode-eslint"`. "Always installed" extensions via `dev.containers.defaultExtensions` setting.

## Port Forwarding and Publishing

- **Always forward:** `"forwardPorts": [3000, 3001]` in devcontainer.json
- **Temporarily forward:** "Forward a Port" command; remembered with `"remote.restoreForwardedPorts": true`
- **Publish:** `"appPort": [3000, "8921:5000"]` (requires rebuild)

## Container Lifecycle

Containers start automatically when opening the folder and shut down when VS Code closes. Change with `"shutdownAction": "none"` in devcontainer.json. Remote Explorer provides container management (stop, start, remove).

## Dotfile Repositories

Dotfiles can be auto-copied from a GitHub repository into containers:

```json
{
  "dotfiles.repository": "your-github-id/your-dotfiles-repo",
  "dotfiles.targetPath": "~/dotfiles",
  "dotfiles.installCommand": "install.sh"
}
```

## Known Limitations

- Windows container images unsupported
- Multi-root workspaces all go in one container
- Ubuntu Docker snap unsupported; Docker Toolbox unsupported
- SSH keys with passphrases may cause hangs
- Local proxy settings not reused
- Alpine containers may have issues with glibc dependencies in native extension code
