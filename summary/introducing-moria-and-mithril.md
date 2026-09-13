---
url: https://brownfinesecurity.com/blog/introducing-moria-and-mithril
title: "Moria and Mithril: IoT Firmware Extraction and Analysis Without External Dependencies"
author: Brown Fine Security (nmatt0)
date_fetched: 2026-09-13
date_published: unknown
topics:
  - developer-tools
  - mcp-and-tool-protocols
---

Brown Fine Security (nmatt0) announces two MIT-licensed tools that replace the standard IoT firmware analysis pipeline: **moria** identifies and unpacks firmware images, **mithril** reads what moria unpacked. The motivation is dependency archaeology — binwalk and unblob both shell out to "a small army of external extractors" (sasquatch, jefferson, ubi_reader, yaffshiv, 7z, cramfsck), many unmaintained mid-2010s Python, some needing patched forks for vendor quirks. Both new tools are single C++ binaries that parse every supported filesystem in-process; the only shared libraries are four compression codecs (zlib, lzma, lz4, zstd) plus the C/C++ runtime.

moria is identify-first: by default it prints a map of the image — offset, size, type, and a **confidence tier** per finding (`verified` means a real structural check passed, e.g. a uImage header CRC32; `consistent` means cross-field checks agree; `structural` and below are weaker) — instead of dumping files. `-e` unpacks recursively in under a second, with a `manifest.json` recording exactly what happened including partial extractions. The aging formats (JFFS2, YAFFS2, cramfs, romfs, UBIFS) are reimplemented from the on-disk format up — JFFS2 by walking and replaying the node log directly, no MTD emulation. Extraction treats firmware as hostile input: writes go through `openat` with `O_NOFOLLOW`, no root needed, with `--depth`, `--max-files`, and a decompression-ratio cap bounding malicious images.

mithril runs four passes over the extracted tree. **Secrets** ride their own ladder — `pattern`, `structural` (parsed PEM/JWT), `validated` (recomputed checksum, e.g. crypt hash or GitHub token) — surfacing empty root/default password hashes, embedded RSA keys, and wifi configs on real camera images. **SBOM** is built from package databases, language manifests, ELF version banners, versioned libc filenames, and the kernel banner, so it works on stripped images; output is CycloneDX and SPDX keyed on purls. **CVEs** join that SBOM against a local mirror of OSV, NVD, CISA KEV, and FIRST EPSS (no network calls), recording each match's basis and annotating with EPSS; a curated kernel-CVE checklist is version- and kconfig-gated (config inferred from kallsyms), cutting kernel noise to what actually applies. **Licenses** is the fourth pass.

The agent angle is a first-class design goal, not an afterthought: both tools print a human table by default and full JSON with `-j`, and every object carries `schema_version`, offsets, confidence, and sourced references, because the author's firmware triage now runs through Claude Code skills and "an LLM agent parses JSON far more reliably than it scrapes a table." The whole pipeline is two commands: `moria -e firmware.bin` then `mithril firmware.bin.extracted/`. Both build with cmake and a C++20 compiler; contributions and real-world firmware samples that break the tools are explicitly requested.
