---
url: https://www.spacebar.news/powershell-is-actually-good/
title: "I can't believe PowerShell is actually good"
author: Corbin Davenport
date_fetched: 2026-10-24
date_published: 2026
topics:
  - developer-tools
  - software-engineering-craft
---

Davenport's conversion essay: after years of assuming PowerShell was Windows-only cruft, using it for real projects on a Mac convinced him it's one of the best scripting tools available — cross-platform via PowerShell 7 (the modern .NET Core version; Windows PowerShell 5.1 is deprecated and what ships on Windows 11).

The argument rests on what PowerShell bundles that Bash doesn't: first-class data types (integers, strings, arrays, hash tables), native JSON conversion, real parameter blocks (`Param` with mandatory-typed inputs replaces `getopts` boilerplate), `Invoke-WebRequest` instead of hoping curl/wget exists, and a consistent verb-noun cmdlet naming scheme (`Set-Location`, `Remove-Item`). The killer design idea is the virtual drive: the Windows Registry, certificate store, or any path you like appear as filesystems, so `Test-Path "HKLM:\SOFTWARE\..."` works identically on files and registry keys — one mental model instead of two.

Two project walk-throughs carry the case: Just the Browser (a Windows installer using .NET `List<T>`, registry cmdlets, and `Invoke-WebRequest` + `reg.exe`), and a Tumblr-to-Eleventy migration script mapping an API response to `PSCustomObject` and emitting JSON — the work he'd otherwise have handed to Node.js, minus the dependency roulette and `await` confetti.

He closes with a deliberate side-swipe: AI chatbots could write these scripts for him, but he'd rather keep his brain from atrophying — scripting as a craft he chooses to keep doing by hand, with a local Gemma model only for troubleshooting.
