---
url: https://devblogs.microsoft.com/commandline/wsl-container-is-now-available-for-public-preview/
title: WSL Container Is Now Available for Public Preview
author: Microsoft Developer Blogs (Command Line team)
date_published: 2026-05 (Microsoft Build 2026)
date_fetched: 2026-08-07
---

Microsoft announced at Build 2026 that WSL containers — a built-in, enterprise-ready way to create, run, and manage Linux containers on Windows through WSL — is now available as a public preview in the WSL pre-release channel.

The feature adds two components: a new CLI tool (`wslc.exe`, also aliased as `container.exe`) for full Linux container development workflows (run, debug, test), and a NuGet-packaged API (C, C++, C#) that lets Windows applications programmatically leverage Linux containers. The API integrates with MSBuild and CMake, so container build and deploy steps can be part of an application's build process.

Enterprise integrations include Microsoft Defender for Endpoint awareness of Linux container events (private preview), Intune management for controlling WSL distro vs. container usage plus container registry allowlists (GPO/ADMX today, Intune dashboard within weeks), and VS Code dev containers support via wslc in pre-release `0.462.0`.

Under the hood, WSL containers ship with a new default filesystem (`virtiofs`, claiming 2× faster Windows file access), a new experimental networking mode (`consomme`, which relays Linux network traffic through Windows for better VPN/proxy/enterprise compatibility), and improved memory reclaim. These improvements are currently container-only but planned for default WSL in the future. The team aims for GA in fall 2026.
