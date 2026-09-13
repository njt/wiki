# Moria and Mithril

Brown Fine Security (nmatt0) releases two MIT-licensed single-binary C++ tools that replace the binwalk/unblob firmware-analysis pipeline: **moria** maps and unpacks IoT firmware images entirely in-process — no shelling out to sasquatch, jefferson, ubi_reader, or yaffshiv — and **mithril** reads the unpacked tree in four passes (secrets, SBOM, CVEs, licenses). What makes the pair interesting beyond the security domain is that agent consumption was a first-class design goal: schema-versioned JSON with confidence tiers on every object, because the author's own firmware triage now runs through Claude Code skills.

---

## Key Quotes

> "Half of them are unmaintained Python from the mid-2010s, several need patched forks to handle vendor quirks, and every one is another thing to install, pin, and keep working across machines. When an extraction fails, you are debugging someone else's decade-old script instead of looking at the firmware."

The motivation, and the clearest statement of dependency archaeology as a tax on the standard toolchain. The `ldd` output is the argument's proof: four compression codecs and the C/C++ runtime, nothing else. "Copy the binary to a box and it runs" is the operational payoff.

> "That confidence ladder is the point: moria tells you how sure it is instead of printing a magic-byte match and hoping."

The epistemically important design choice. `verified` (a real structural check, e.g. uImage header CRC32), `consistent` (cross-field checks agree), `structural`, and below — calibrated certainty instead of binwalk's pattern-match optimism. This matters double for agent consumers: an agent that can't gauge the certainty of a finding will act on it anyway.

> "This was a first-class design goal, not an afterthought, because a lot of my firmware triage now runs through Claude Code skills, and an LLM agent parses JSON far more reliably than it scrapes a table."

The agent-native thesis in one line, from a security practitioner rather than a CLI theorist. Human table by default, full JSON with `-j`, and every object carries `schema_version`, offsets, confidence, and sourced references — the output is auditable, not merely parseable.

> "[W]rites go through `openat` with `O_NOFOLLOW`, so a hostile archive cannot traverse out of the output directory or follow a symlink into your filesystem."

Extraction is parsing untrusted input, and moria treats it that way by default: no root, `O_NOFOLLOW` against symlink traversal, and `--depth`, `--max-files`, and a decompression-ratio cap bounding hostile images. The old pipeline never promised any of this. The xz-backdoor era makes "your extraction tool is an attack surface" a live concern, not paranoia.

> "That is the difference between a checklist you can act on and a wall of kernel CVEs you have to triage by hand."

On the kernel-CVE pass: a curated, version- and kconfig-gated checklist (config inferred from kallsyms when no `.config` exists), annotated with EPSS, with ruled-out CVEs shown *with their reasons*. Signal over noise as an explicit deliverable — 214 undifferentiated openssl CVEs become a sorted list where Dirty COW sits on top with its KEV badge.

## Key Themes

- **#tool** moria — identify-and-unpack firmware tool: offset/size/type/confidence map, recursive extraction, `manifest.json`, in-process parsers for JFFS2 (node-log replay), YAFFS2, cramfs, romfs, UBIFS, SquashFS, uImage
- **#tool** mithril — content analysis: secrets ladder (pattern → structural → validated), purl-keyed CycloneDX/SPDX SBOM, offline OSV/NVD/KEV/EPSS CVE join, licenses
- **#concept** Confidence tiers — calibrated output (`verified`/`consistent`/`structural`/`magic`) replacing magic-byte matching
- **#concept** Zero external extractors — the single-binary bet: reimplement formats from the on-disk spec up rather than orchestrate a museum of aging Python
- **#pattern** Agent-native CLI — human table by default, schema-versioned JSON with `-j`, sourced references on every object, two composable commands
- **#pattern** Hostile-input defaults — no-root extraction, `O_NOFOLLOW`, depth/file-count/decompression-ratio caps
- **#project** github.com/nmatt0/moria and github.com/nmatt0/mithril — C++20, cmake, MIT, signatures embedded in the binary

## Critical Analysis

**The zero-external-extractor bet is the whole article, and it's a good bet.** Reimplementing JFFS2, YAFFS2, cramfs, romfs, and UBIFS from the on-disk format up is a serious maintenance commitment, but it converts a sprawling, half-abandoned dependency graph into a bounded parsing surface the author owns. The performance claim is plausible for the right reason: one native binary with a single bounds-checked reader beats spawning a Python interpreter per region and scraping its stdout — the VStarcam image (U-Boot + MIPS kernel + SquashFS + JFFS2) mapped and unpacked in 0.3s. And the security properties (no root, `O_NOFOLLOW`, hostile-input caps) are things the shell-out pipeline structurally cannot promise.

**The confidence ladder is the most exportable idea.** Most identification tools report *shape* ("these bytes look like SquashFS"); moria reports *verification level* and puts the evidence ("header CRC32 ok") in the output object. For an agent consumer this is the difference between data and decision inputs. It's the same instinct as mithril's secrets ladder (pattern/structural/validated with a *recomputed checksum* at the top) — confidence is a first-class field, not a vibe.

**The agent section is short but load-bearing.** The pipeline is two commands with clean JSON at each boundary — `moria -e firmware.bin` then `mithril firmware.bin.extracted/` — which is exactly the shape an agent skill wants to drive. What's missing from the agent-native story is the compounding tier: no introspection surface, no feedback channel back to the maintainers. The Table Stakes are fully delivered; the compounding principles are not yet visible.

**Read the evidence base honestly.** This is a launch post by a company that sells IoT pentesting and training (the last two sections are a sales pitch), the benchmarks are self-reported on two camera images the tools demonstrably handle well, and "no false-positive noise" is a sample size of one tree. The CVE join rides `nvd-cpe-range` matching — the machinery known for both misses and false hits — though recording the match basis and EPSS score is the right mitigation, and the "22 undetermined" kernel CVEs on the VStarcam show the kconfig-inference gap is real rather than hidden. The call for real-world firmware that breaks the tools is the honest acknowledgment: the long tail of vendor firmware is where the museum of extractors earned its scars, and moria's corpus hasn't tested it yet.

**The cost curve this sits on.** A full image mapped and unpacked in under a second, secrets validated by recomputation, 94-component SBOM, 214 CVEs sorted by EPSS — all of it offline, all of it two commands. Read against what firmware analysis used to cost, this is the triage phase of firmware RE collapsing from expert-weeks to minutes, which moves the scarce human work further up the stack: to what you decide to *do* with the map.

---

## Related

- **[[10 Principles for Agent-Native CLIs]]** — This source is a shipped, security-domain instance of Chow's "design for agents first, humans benefit" thesis: human-readable table by default, full schema-versioned JSON with `-j`, confidence and sourced references on every object. It strengthens the Tier 1 (Table Stakes) half of that framework with a real tool, and its gaps (no introspection layer, no feedback channel) mark precisely where Chow's Tier 2 compounding principles are still rare in practice.
- **[[bt-re-controller — Bluetooth Firmware RE Skill]]** — moria/mithril is the deterministic mechanical layer that a firmware-RE skill pipeline like this one wants *underneath* it: unpack, secrets, SBOM, and CVEs as clean JSON before any decompiler opens. It also nuances this page's dependency story — where bt-re-controller deliberately accepted fragile forks of `pyghidra-mcp` and `ghidrecomp` as the price of capability, moria shows the alternative trade (own the parser, ship one binary). The two remain complements: moria maps bytes, bt-re-controller's model makes the semantic naming judgments no parser can.
- **[[Reverse Engineering the Tesla Firmware Update Process]]** — The pre-agent baseline: expert researchers, weeks of work, one vehicle, and the mapping phase was itself a major cost. Moria's sub-second full-image map and mithril's automated secrets/CVE passes collapse that first phase outright, which complicates the piece's economics — the secrets stayed secret because the work was expensive, and the per-image triage portion of that expense is now approaching zero.
- **[[Everything I Own, Owned]]** — Schlarp's threat model says firmware implants are no longer a state-actor activity because agentic RE collapsed the per-model labor cost. Mithril's secrets pass — empty root and default password hashes, embedded RSA keys, wifi configs on commodity cameras, found in two commands — is the defender's half of the same collapse, strengthening the warning while handing everyone the tooling to audit the devices on their own desk.

---
*Sources: [[raw/introducing-moria-and-mithril]], [[summary/introducing-moria-and-mithril]]*
*Last updated: 2026-09-13*
