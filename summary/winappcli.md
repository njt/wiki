---
url: https://github.com/microsoft/winappCli
title: "winapp CLI — Windows App Development CLI"
author: Microsoft
date_fetched: 2026-10-03
date_published: 2026 (public preview, v0.7.x)
topics:
  - developer-tools
  - agent-coding-workflow
---

Microsoft's winapp CLI is a single command-line tool for Windows app development: SDK setup, package identity, MSIX packaging, signing, manifests, and certificates — for any framework (.NET/WinUI, Electron, C++, Rust, Flutter, Tauri). The pitch is collapsing the ~12 manual steps of getting Windows-native capabilities (notifications, OS integration, on-device AI) into a few commands, and it is explicitly designed agent-first: several commands exist primarily so AI coding agents can ground their output in real project metadata instead of hallucinating APIs.

Three architectural layers: a large C#/.NET CLI (~108k lines in `WinApp.Cli`, 64 command classes, Spectre.Console + System.CommandLine), a Node.js/npm wrapper (`@microsoft/winappcli`) that intercepts `init`/`restore` to add JS-native-addon generation hooks and shells out to the native binary, and a `plugins/winapp` agent plugin shipping 11 SKILL.md files plus a Copilot agent definition — the skills are structured prompts encoding operational judgment (workflow-ID desktop arbitration, sandbox consent rules, batch-your-API-lookups).

The most interesting parts are agent-first: `winapp find-api` searches the project's actual .winmd/.dll API surface and instructs agents to treat it as the authority over training-data recall; `winapp ui` wraps Windows UI Automation with stable selectors and a cooperative turn-taking protocol (`WINAPP_UI_WORKFLOW_ID`, 4-second grace, `yield`) so concurrent agent workflows don't fight over one desktop; and `--on sandbox` runs apps in Windows Sandbox with file push/pull, screenshots, and video capture — with frank warnings that builds still run on the host and one Sandbox is a shared, not isolated, environment.
