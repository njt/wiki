---
url: https://github.com/taviso/matrix3
title: "matrix³ — An experimental content policy manager for Chrome MV3"
author: Tavis Ormandy (taviso)
date_fetched: 2026-05-18
date_published: unknown
---

# matrix³

Source: https://github.com/taviso/matrix3

## Overview

matrix³ is an experimental content policy manager inspired by uMatrix, built on Chrome's declarativeNetRequest API for Manifest V3 extensions. It provides a sidepanel interface for managing Content-Security-Policy (CSP) directives per site. The project is currently a prototype.

Author: Tavis Ormandy (taviso), a well-known security researcher.

## How It Works

The extension provides a sidepanel interface (opened via the toolbar icon) that lets users toggle web features per site. It leverages declarativeNetRequest rather than the older webRequest blocking API (which MV3 removed).

### Rules

- **Session rules** — ephemeral, discarded on browser restart (the default)
- **Dynamic rules** — persistent across restarts; users click **Commit** to promote a session rule to dynamic status
- The **Rules** panel displays all active rules with delete capability

### Report Tab

Shows which subresources were denied (highlighted in orange). Users toggle options until a site works, then **Reload** to refresh the tab and **Commit** to persist settings.

### Groups

Named bundles of origins users frequently want to trust together (e.g., CDNs, social media embeds). Defined on the **Groups** page, then applied via **Trust** or **Untrust** for a given host. The **Ignore** group hides irrelevant origins from the Report page.

### Suggested Policies

When a server proposes its own CSP, users can **Accept**, **Merge**, or ignore it.

## Default Policy Levels

| Policy | Behavior |
|---|---|
| **Permissive** | Nothing blocked by default |
| **First Party** | First-party allowed; third-party blocked unless permitted |
| **Sandbox** | Applied to every document by default — no scripts, forms, popups, downloads, or top navigation |
| **First Party Sandboxed** | Combines First Party with sandboxing |
| **Strict** | `default-src` set to `'none'` — effectively everything disabled |

## Technical Details

- Language breakdown: JavaScript 74.8%, HTML 18.8%, CSS 3.8%, Python 2.1%, Makefile 0.5%
- 100 commits on main branch
- Repository structure: `include/`, `panels/`, `tools/`, `vendor/` plus config files (`base.json`, `firstparty.json`, `sandbox.json`, `strict.json`, `permissive.json`), `manifest.json`, `server.js`, `service-worker.js`
- 24 stars, 0 forks, 1 watcher
- No releases published

## README (verbatim)

> matrix³ is an experimental content policy manager, inspired by umatrix, but built on declarativeNetRequest.

> This extension basically just provides an interface to Content-Security-Policy, if you're familiar with the CSP3 specification you'll be familiar with this extension.

> If sandbox mode is enabled, try disabling it and observe what resources the site requests.

> Any subresource that was denied by this extension is highlighted in orange.

> You can continue to enable things until the website works, then click Reload to refresh the tab. When you're happy with your settings, clicking Commit will make them persistent.

> A group is a named bundle of origins you frequently want to trust together.

> This is currently just a prototype.

From the README on installation:
1. Clone or download the repository
2. Navigate to `chrome://extensions`
3. Enable Developer mode
4. Click **Load Unpacked** and select the matrix3 directory
