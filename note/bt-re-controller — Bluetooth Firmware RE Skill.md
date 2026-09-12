# bt-re-controller — Bluetooth Firmware RE Skill

An open-source pair of agent skills that turn an unnamed Bluetooth Controller firmware binary in Ghidra into spec-named decompilation — and then merge two independent LLM reverse-engineering passes into one model-consensus project. It matters because it is the most complete public example of a *domain-expert pipeline* encoded as a Claude Code / Codex skill: ~35 coordinated prompt phases, deterministic Python for everything mechanical, and a novel cross-CLI audit where each model grades the other's work.

---

## Architecture

The repo ships one skill tree twice — `ClaudeCode/` and `ChatGPTCodex/` — because the two hosts invoke different CLI instances of themselves; the core logic is identical. Each tree holds two skills plus a `manifest.json`.

**`bt-re-controller`** (`ClaudeCode/bt-re-controller/`) is a **five-phase DAG of ~35 prompt phases**, each a small markdown file with YAML frontmatter declaring `id`, `depends_on`, `parallel_safe_with`, and `applies_when` (`manifest.json:77-128`). The phases are grouped into ~12 batches that dispatch as parallel subagents or run serially (`SKILL.md:341-366`):

- **P-phase (prep)** — non-Bluetooth structural passes: rename `set_mem_*` stubs (P2), retype `undefined4`→fptr (P3), log-string inference (P4), memcpy/memset (P5), bulk pointer typing (P6), then a consolidated cache refresh (P7).
- **H-phase** — discover the OGF dispatcher (H1), rename HCI command handlers (H2), recover table-driven handlers (H5), type HCI args (H3), rename HCI event handlers (H4).
- **B-phase** — LMP (B1_LMP…) and LLCP (B1_LL…) packet send/recv discovery, two independent protocol chains run in parallel.
- **V-phase** — spec-coverage cross-checks (V1 extracts the over-the-air version; V2/V3/V4 gap-check HCI/LMP/LLCP).
- **S/D/E/F-phase** — structs (S1–S9), propagation (S6), per-connection arrays (S7), then cleanup and finishing passes.

The critical dependency edges are *explicit* in the manifest, not just batch ordering — e.g. `S1_HCI` depends on `V2_HCI` so a handler newly named by the coverage pass still gets struct typing (`manifest.json:99`).

**`bt-re-controller-merge-ghidra-files`** (`ClaudeCode/bt-re-controller-merge-ghidra-files/`) is an 11-phase pipeline (`SKILL.md:133-153`): preflight → init → materialize → inventory → align → **audit** → plan → stage → apply → struct+propagate → guard → finalize. Phases 0–4, 6–8, 10–11 are deterministic Python; phase 5 (peer audit) and 9 (struct propagation) are the model half.

The runtime is a **two-tier launcher** (`scripts/run.sh`): it starts a `pyghidra-mcp` server for the project, binds session variables via `template.py`, and spawns *one inner `claude -p`* that runs the whole DAG, dispatching phases as subagents. The launcher never connects to MCP itself — only the inner session can attach a just-launched server, which is why launch-and-use in one already-running session is impossible.

## Key Techniques

**Self-encoding names as the core abstraction.** Every rename encodes full identity so the opcode is recoverable from the symbol alone: `hci_ctrl_cmd_05_005__HCI_Read_RSSI`, `lmp_send_25__LMP_VERSION_REQ`, `llcp_recv_0C__LL_VERSION_IND` (`references/naming_conventions.md:9-21`). The `__` double-underscore is the greppable boundary between numeric opcode and spec name, chosen because spec names contain single underscores. This lets *every* downstream tool — coverage scorers, the merge aligner, the semantic matcher — parse names with regexes instead of a symbol database.

**The model is forbidden to recall spec names from memory.** Every phase prompt says "take names from `references/*.csv`/`.md`, never from memory" (`naming_conventions.md:17-21`). The skill ships the entire spec as data: opcode tables with an `Added` version column, HCI-command→event CSVs, feature-bit→gated-PDU maps. This is the opposite of [[Agent Skills for Security Testing]], which trusts the model's knowledge — here the model is a *lookup-and-match engine*, not an oracle.

**Version/feature down-select bounds search but never naming** (`scripts/downselect.py`). A `BT_SPEC_VERSION` guess (or the over-the-air `VersNr` extracted from `LL_VERSION_IND`) prunes the *search scope* to elements whose `Added` version ≤ the chip's; a disabled FeatureSet bit drops its gated PDUs. But a numeric opcode found in a dispatcher table is *always* renamed from the full tables — "down-select is search scope, not naming" is the standing rule. This cleanly separates "what should I look for" from "what is this thing I found", which is where most spec-driven RE tools blur.

**Staleness manifests instead of eager cache refresh.** The P-phase passes each write a JSON list of functions they made stale, and P7 unions them by `(binary, address)` into one decomp-cache refresh — running total decomp-fetch count down 2–4× on dense firmware (`SKILL.md:323`).

**Save-before-mark resume contract.** Every phase must `save_program`, capture `binary_fingerprints`, *then* write its completion marker — so a marker can never attest work that only exists in volatile Ghidra memory (`SKILL.md:513-524`). Stale markers are archived (never deleted) as forensic evidence.

**The cross-CLI peer audit (the genuinely novel piece).** `merge-ghidra-files` aligns two projects by `(program, address)` with a `SemanticMatcher` that treats two names as matching iff they denote the same spec *procedure* — ignoring direction (send/recv), sub-PDU (REQ/RSP/IND), and even layer (an HCI command, LMP PDU, and HCI event of the same procedure all match) (`semantic_match.py:192-204`). Then `peer_exec.sh` launches the *other* model's CLI headlessly — Claude invokes `codex exec`, Codex invokes `claude -p` — so each grades the other's contested findings. Merge = `agreed ∪ CONFIRMed − REJECTed`; rejected names are reset to `FUN_<addr>` and their addresses protected so a later pass can't re-guess them.

## Design Decisions

**Deterministic mechanical layer under an LLM judgment layer.** Everything that can be computed — dispatch, alignment, merge arithmetic, CSV filtering, fingerprinting — is Python. The LLM only makes rename *decisions* and the cross-model *judgments*. This is the opposite trade from a pure-prompt skill: the skill is willing to be a pile of shell scripts + Python + CSVs, because the *guarantee* (idempotency, resume, auditability) comes from the deterministic layer.

**Precision over recall, then fix recall with more passes.** The P-phase parallelism is deliberately capped at 3 because `pyghidra-mcp` has a single decompiler thread (`SKILL.md:321`). Heavy phases (P2, P4) are split across waves. The V-phase coverage cross-checks exist as a "spec-side backstop" to the structural B-phase discovery — a second, independent pass to catch what the first missed.

**Security posture is a hard constraint, not a feature.** Because the MCP server exposes `run_inline_script` (arbitrary host code execution), the launcher forces `--host 127.0.0.1` *explicitly* rather than inheriting a possibly-overridden `MCP_HOST`, and ships loopback-only with no option to change it (`SKILL.md:217-223`). A 0.0.0.0 bind would be a LAN-reachable RCE.

**Dependency forks as a deliberate trade.** The skill requires forks of `pyghidra-mcp` and `ghidrecomp` (run from source via `uvx`, no pip install) because upstream "installs cleanly and *looks* like it works" but lacks needed changes. That buys capability at the cost of a fragile, fork-tracking setup.

**Honest about the one thing it can't reproduce.** The merge skill's measured-behaviour section concedes that its model-driven struct/propagation phases don't record a replayable op log, so exact end-to-end semantic reproduction needs an exported type/signature patch (`merge SKILL.md:479-493`). It publishes the numbers (2,890 functions, 75 modified) rather than papering over the gap.

## Comparison Notes

- **vs. [[Binary RE]]**: that page covers a generic RE-skill collection; bt-re-controller is a *specialized* RE skill that encodes one domain (Bluetooth controllers) exhaustively — a full spec naming convention, ~35 phases, and reference tables — rather than general disassembly guidance. It's the depth-vs-breadth distinction.

- **vs. [[Kuna — Agent-First Decompiler]]**: Kuna asks whether an LLM can *build* the decompiler; bt-re-controller assumes Ghidra is the decompiler and puts the LLM on *naming and procedure recovery* — the problem a decompiler structurally can't solve. Complementary halves of the same pipeline; bt-re-controller's names could even feed a Kuna-style tool's training signal.

- **vs. [[Agent Skills for Security Testing]]**: both are Claude Code skills that encode domain expertise, but that skill is a *flat* library of independent grep recipes trusting the model's synthesis, while bt-re-controller is a *deep DAG* with deterministic tooling and the model forbidden to name from memory. The contrast is "16 shallow skills" vs. "one deep pipeline."

- **vs. [[Cloudflare Security Audit Skill]]**: both orchestrate parallel subagent phases with deterministic validation, but Cloudflare's adversarial separation is *within one model family* (hunter/validator/verifier), whereas bt-re-controller's merge step achieves cross-model consensus by literally invoking a *different vendor's* CLI (`claude -p` vs. `codex exec`) to grade the other's output — a stronger form of independence.

- **vs. [[DeepSeek Reverse Engineers TeamSpeak Licensing]]**: that field report used one model + two MCP servers for $3.88 of ad-hoc RE; bt-re-controller is the industrialized version of the same idea — reusable, parameterized, resumable, with naming normalized to spec and coverage cross-checks. The economics it enables are the same ones that report flagged.

---

*Sources: [[raw/bt-re-mad-skillz]], [[summary/bt-re-mad-skillz]]*
*Last updated: 2026-09-04*
