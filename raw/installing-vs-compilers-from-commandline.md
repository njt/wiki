---
title: "Installing VS Compilers From Commandline"
url: https://marler8997.github.io/blog/fixed-windows/
date_fetched: 2026-05-14
section: "C# and .NET"
---

# I Fixed Windows Native Development

## Problem
Windows native development tooling conflates the editor, compiler, and SDK into the Visual Studio monolith. Users navigate a "maze of checkboxes" to find necessary workloads, with cryptic error messages if they miss components. Hours-long waits downloading 15GB for a 50MB compiler. No transparency, no version control, no clean uninstall.

## Solution: msvcup
The author developed msvcup, an open-source CLI tool that:
- Parses Microsoft's official JSON component manifests
- Downloads only essential compilation packages directly from Microsoft's CDN
- Installs into versioned, isolated directories
- Enables reproducible builds across machines
- Operates idempotently with millisecond execution times after initial setup

"It's a small CLI program. On good network/hardware, it can install the toolchain/SDK in a few minutes."

## Real-World Usage
Integrated at Tuple (pair-programming app), eliminating pre-installation requirements and enabling consistent ARM and x86_64 builds across CI systems.

## Conclusion
msvcup transforms Windows native development from a dependency-management nightmare into a reproducible, portable, modern workflow -- making Visual Studio installation optional for command-line compilation.
