# MCP-FSD — Filesystem Capability Laundering in MCP Servers

An arXiv preprint (2610.22301) that names, formalizes, and measures a new MCP vulnerability class: **filesystem capability laundering** — an out-of-scope filesystem access exercised under the guise of an innocuous tool contract. Using the MCP-FSD framework across 21,097 servers, it finds 150 dynamically confirmed vulnerable instances and lands two CVEs in Anthropic's Git MCP Server, then argues the fix belongs in SDKs, the protocol itself, and OS sandboxes — not in user judgment at the approval prompt.

---

## The mechanism

The architectural hole is simple and uncomfortable: the client's approval prompt gates the *declared* tool call, but the spawned local server process runs with the user's ambient filesystem permissions.

> An ostensibly benign operation can therefore access and expose host credentials through ordinary tool output. We call this vulnerability class *filesystem capability laundering*: an out-of-scope filesystem capability is exercised under the guise of an innocuous tool contract.

The formalization requires three conditions: an effect exceeding the scope justified by the contract, attacker-influenced arguments selecting the target, and controls failing to constrain the effect. Crucibly, this decouples the vulnerability from the prompt injection that usually triggers it — injection is the delivery channel; laundering is the server-side consequence.

## Measuring it at scale

MCP-FSD works at three layers and, notably, validates every confirmed finding dynamically rather than trusting static analysis:

1. **L1 screening** — 787 of 4,806 triaged servers (16.4%) show filesystem capability flags without a FileSystem category label.
2. **L2 contract analysis** — a narrow tool reaching beyond its implied scope.
3. **L3 defense bypass** — a path check that can be defeated (the classic string-prefix-match admitting sibling directories, symlink TOCTOU, etc.).

> PoC 1 confirms that the exploit succeeds on an unconfined host; PoC 2 verifies that the inferred sandbox policy preserves legitimate, in-scope functionality; and PoC 3 demonstrates that the sandbox neutralizes the exploit under that exact policy.

That three-case protocol (malicious unsandboxed / benign sandboxed / malicious sandboxed) is the paper's best methodological idea: it proves both exploitability *and* containment against an inferred least-privilege boundary, with planted credential fixtures rather than real secrets.

## The case studies sting

Both CVEs are in Anthropic's own Git MCP Server, and both exploit ordinary Git workflows:

- **CVE-2025-68143** — `git_init` accepts any `repo_path`; an attacker-steered argument turns `~/.ssh` into a Git repository, after which legitimate `git_add`/`git_diff_staged` calls return SSH private keys as diff output.
- **CVE-2026-27735** — `git_add` stages paths containing `../` outside the repository root; GitPython reads the bytes into the object database, and a staged diff discloses them.

> A check against only the newly selected root would therefore miss the earlier workspace violation.

The synthesis matters: containment must hold at *every* step of a call sequence, not just the tool that finally discloses the bytes.

## The ecosystem diagnosis

> The MCP TypeScript SDK provides no built-in path-containment primitive (such as a safePath() utility), leaving Node servers prone to flawed custom checks... Python servers frequently omit validation checks entirely.

Two remediation paths fall out naturally: SDK-level allowlist helpers for Python, and standardized containment primitives for TypeScript. At the protocol level, the authors propose a `fs_scope` field on tool declarations so clients can inspect the requested read/write scope before approval — with MCP-FSD's inferred policies as a candidate representation.

## Key themes

#concept filesystem-capability-laundering #pattern least-privilege-boundary-inference #tool MCP-FSD #concept three-case-dynamic-validation

## Critical take

This is one of the most rigorous papers in the agent-security corpus — the three-case validation with paired sandboxed/unsandboxed replay and planted fixtures is genuinely good measurement discipline, not just scanning theater. The honest limitations section (inferred boundaries themselves unvalidated; 96% of the corpus is Node/Python because that's what installs locally; LLM-generated triage labels inherit model error modes) strengthens rather than weakens it.

Two things deserve pushback. First, the inferred-boundary problem is harder than the paper admits: a wrong hypothesis silently determines the ground truth of the whole experiment, and their own "source coverage plus a successful benign call do not establish authorization accuracy" concession means the 150 figure is a lower bound conditioned on guessed policies. Second, the `fs_scope` protocol proposal skirts the deepest issue: a declared scope is still a self-report by server authors — the same actors who currently don't validate paths. The durable fix the paper actually demonstrates is the boring one: OS-enforced sandboxing worked on every confirmed instance. The protocol extension is nicer; the sandbox is the proof.

The laundering framing itself is a genuine conceptual contribution — it cleanly separates the *declared contract* from the *ambient capability*, which is exactly the axis on which the approval-prompt UX model of MCP security fails.

## Related pages

- [[ANSI Escape Sequence Injection in MCP Servers]] — a sibling MCP attack class exploiting the gap between declared fields and what models actually consume; together they show the MCP contract surface is leaky at both the byte and filesystem layers.
- [[Securing MCP Servers — Database Access Control]] — Redgate's confused-deputy and over-scoped-credential failure modes for database-backed MCP servers; this paper extends the same least-privilege argument from data access to filesystem access, with measurement to back it.
- [[nono.sh — Brokered Tool Execution for Agent Security]] — a commercial runtime implementing exactly the containment model this paper validates empirically: brokered calls, sandboxed execution, and audited decisions.
- [[How We Contain Claude]] — Anthropic's own containment postmortem; this paper supplies the ecosystem-scale evidence for why custom code inside unsandboxed MCP servers is always the failure point.

---
*Sources: [[raw/2610-22301v1]], [[summary/2610-22301v1]]*
*Last updated: 2026-10-18*
