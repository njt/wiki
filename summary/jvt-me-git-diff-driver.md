---
url: https://www.jvt.me/posts/2026/04/11/how-git-diff-driver/
title: "How to build a `git diff` driver"
author: Jamie Tanna
date_fetched: 2026-05-15
date_published: 2026-04-11
topics:
  - developer-tools
---

# How to build a `git diff` driver

Jamie Tanna's guide to writing an external diff tool for git, written after discovering a gap in discoverable documentation. Covers the 7-argument interface git passes to external diff drivers, the `/dev/null` convention for file creation/deletion, and a worked example using `oasdiff` for OpenAPI spec diffs.

## Summary

> "How to write an external tool for `git diff` to delegate complex diffs to."

## Motivation

Tanna had meant to write this "since November 2024." Working on `renovate-packagedata-diff`, he found "there seemed to be a lack of documentation around how to do it." While "some documentation in the Git Diffs man page" exists, it "wasn't exactly well discoverable."

A nudge came from Andrew Nesbitt's post on Git Diff Drivers, plus Tanna's own work diffing OpenAPI specs with `oasdiff`.

## Arguments

Git passes 7 arguments to the external tool:

1. Filename in the repo
2. Path to the "before" file
3. SHA-1 hash of the "before" file
4. Octal mode of the "before" file
5. Path to the "after" file
6. SHA-1 hash of the "after" file
7. Octal mode of the "after" file

### Special cases

- **File created** (net-new): argument 2 is `/dev/null`, arguments 3 and 4 are `.`
- **File deleted**: argument 5 is `/dev/null`, arguments 6 and 7 are `.`

Also: check for `GIT_PAGER_IN_USE` environment variable if your command needs to handle both regular args and git diff args.

## Example: oasdiff driver

```bash
#!/usr/bin/env bash

if [[ "$2" == "/dev/null" ]]; then
	echo "$1 was added"
	exit 0
elif [[ "$5" == "/dev/null" ]]; then
	echo "$1 was deleted"
	exit 0
fi

oasdiff changelog "$2" "$5" --color always
```

The author notes this "doesn't handle any changes in permissions" and suggests it "may be worth using the SHA-1 checksums of the files to cache the resulting diffs."

## Related links

- Git Diffs man page: https://git-scm.com/docs/git#_git_diffs
- Andrew Nesbitt's Git Diff Drivers post: https://nesbitt.io/2026/03/30/git-diff-drivers.html
- oasdiff tool: https://github.com/oasdiff/oasdiff
- Renovate packagedata-diff: https://www.jvt.me/posts/2024/12/08/renovate-packagedata-diff/
- Separate oasdiff driver post: https://www.jvt.me/posts/2026/04/11/oasdiff-driver/
