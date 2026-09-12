---
url: https://github.com/darkmentorllc/bt-re-mad-skillz
title: "bt-re-controller"
author: darkmentorllc
date_fetched: 2026-09-04
site: github.com
topics:
  - security-and-sandboxing
---

# bt-re-controller

By darkmentorllc, publication date not stated on the repo (internal "Measured behaviour" notes are dated 2026-07-22). A collection of two agent skills — plus companion skills referenced in their docs — for reverse-engineering Bluetooth Controller firmware at the HCI layer and below.

## Summary

The repo ships a pair of skills that turn an unnamed Bluetooth Controller firmware binary loaded in Ghidra into spec-named decompilation. The flagship `bt-re-controller` skill runs a five-phase DAG of ~35 prompt "phases" over a `pyghidra-mcp` server (a fork of the Ghidra MCP bridge), discovering and renaming HCI Command/Event handlers, LMP (BR/EDR) and LLCP (BLE) packet send/recv functions, per-connection structs, and spec crypto/channel-selection algorithms — all normalized to the Bluetooth Core Specification 6.3 naming convention (`hci_ctrl_cmd_<OGF>_<OCF>__<Name>`, `lmp_send_<OP>__<PDU>`, `llcp_recv_<OP>__<PDU>`). It is parameterized by four session tokens (`CHIP_NAME`, `DECOMPS_PATH`, `BT_SUPPORT` = DUAL_MODE/BLE_ONLY/CLASSIC_ONLY, `STACK_MODE` = BELOW_HCI/FULL_STACK), which template away whole prompt families and prune the reference tables.

The second skill, `bt-re-controller-merge-ghidra-files`, takes two independently RE'd Ghidra projects of the same firmware (typically Claude Code vs. Codex output) and merges them into one model-consensus project. It dumps both inventories, aligns them by `(program, address)`, then **drives both models' CLIs headlessly** so each grades the other's contested findings — the merge is `agreed ∪ CONFIRMed − REJECTed`, with rejected names reset to `FUN_<addr>` and protected from later passes.

The same skill tree ships twice — once under `ClaudeCode/` and once under `ChatGPTCodex/` — because the two agent hosts invoke different CLI instances of themselves. The core logic is identical; only the launcher/peer-invocation plumbing differs.

## Key Points

- **Not a decompiler** — Ghidra (with the `ghidrecomp` fork) does the decompiling; the skill is the orchestration + naming intelligence layered on top.
- **Everything is self-encoding names + CSVs.** The "intelligence" is carried by reference tables (opcode→name, HCI→event mappings, feature-bit→PDU gating) sourced from the spec, not by the model's memory.
- **Deterministic/mechanical + model split.** Discovery, alignment, and merge arithmetic are pure Python; only the rename *decisions* and the cross-model audit are LLM work.
- **Dependencies are forks** (`XenoKovah/pyghidra-mcp`, `XenoKovah/ghidrecomp`), run straight from source via `uvx` — no pip install — because upstream lacks features the skill needs (e.g. `--existing-project`).
- **Loopback-only by design**: the MCP server exposes `run_inline_script` (arbitrary host code execution), so the launcher forces `--host 127.0.0.1` rather than inheriting a possibly-overridden `MCP_HOST`.
