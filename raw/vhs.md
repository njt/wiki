---
title: "VHS"
url: https://github.com/charmbracelet/vhs
date_fetched: 2026-05-14
section: "Random"
---

# VHS: CLI Home Video Recorder

## Overview

VHS is a terminal recording tool that generates GIFs and videos from scripted commands. As stated in the repository, it allows you to "Write terminal GIFs as code for integration testing and demoing your CLI tools."

## Key Features

- **Script-based recording**: Create `.tape` files containing sequences of terminal commands
- **Multiple output formats**: Generate GIFs, MP4 videos, WebM files, or PNG frame sequences
- **Customizable terminal styling**: Control font size, family, colors, themes, window decorations, and dimensions
- **Recording automation**: The tool includes a `record` subcommand to capture terminal sessions and convert them to tape files
- **SSH server capability**: Self-host VHS to run recordings on remote machines
- **Publishing functionality**: Share generated GIFs via the built-in `publish` command
- **CI/CD integration**: Works with GitHub Actions for automated GIF generation

## How It Works

Users create a `.tape` file defining terminal interactions through commands like `Type`, `Enter`, `Sleep`, and `Wait`. VHS then executes these commands in a virtual terminal, capturing frames at specified intervals, and renders them into the desired output format.

## Installation Methods

- Package managers: Homebrew, Arch Linux, Nix, Scoop
- Docker: Pre-packaged with all dependencies
- Binary downloads for Linux, macOS, and Windows
- Go installation: `go install github.com/charmbracelet/vhs@latest`
- Repository packages in Debian and RPM formats

**Dependencies**: Requires `ttyd` and `ffmpeg` to be installed and accessible in PATH.

## Main Use Cases

1. **Integration testing**: Generate golden ASCII files for comparing terminal outputs
2. **CLI tool demonstrations**: Create professional demo videos for documentation
3. **Tutorial content**: Record step-by-step terminal walkthroughs
4. **Documentation**: Visual examples for command-line tool manuals
