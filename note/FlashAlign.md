# FlashAlign

FlashAlign (Aevonix, AGPL-3.0, v1.1.1) is a per-request alignment layer for open-weight local models: it loads fitted "owner-mode" profiles — in the V1 starter set, refusal-reduction (abliteration-style) transforms — into supported engines and applies them per request without modifying the base weights. What makes it worth reading is not the weight algebra but the governance wrapper: cryptographic proof that a request came from an approved human contact, signed engine requests, replay protection, and explicit rules for how the profile propagates (or doesn't) through tool calls and subagents.

---

## Architecture

Everything lives in `hermes-owner-mode/` (~11.7k lines of Python), organised as a set of trust boundaries rather than a feature list:

- **`proxy.py` — the only signer.** A loopback-only HTTP proxy whose docstring states "Only this boundary can sign a final wire body." It validates the request is genuinely local (`loopback_request` checks Host, Origin and Sec-Fetch-Site against loopback authorities), then applies the route's policy and signs. Invalid `owner_mode_scale` values fail closed: the route stays in the pool serving stock traffic but can never sign.
- **`auth.py` — deployment keys.** 32-byte HMAC keys published atomically via `os.link` (never `rename`, so a concurrent creator never clobbers), with paranoid `O_NOFOLLOW`/mode `0600`/`st_nlink == 1` checks on every read.
- **`policy.py` — the turn-state machine.** `VerifiedTurn` is a frozen dataclass that can *only* be produced by an operator-installed identity adapter — never from request bodies, model text, or sender IDs in headers. Each turn tracks `anchored` (once a call passed the owner's-words check, later calls in the tool loop pass on shape alone) and `steered` (a child turn steered from outside owner work is "off for good").
- **`identity.py` — transport identity.** `IngressFacts` captures what the harness's native adapter observed at ingress (native delivery, synthetic/replayed flags, sender ID *as recorded by the adapter*); an `IngressLedger` binds one verified message to at most one later turn via content SHA-256, freshness window and one-time use.
- **`profiles.py` / `native_profile.py` / `deepseek_grouped.py` — artifact validation.** Profile formats (`projection`, `row_gain_projection_v1`, `residual_writer_group_projection_v1`, …) are validated with bounded reads, duplicate-JSON-key rejection, non-finite-number rejection, exact model revision pins, and rank caps (≤128). The DeepSeek native profile is a separate format that never passes through the compact loader, with exact 1.09 GB tensor payloads checksum-verified from a public release.
- **Engine contract** (`docs/engine-contract.md`): `X-Owner-Mode-Auth: v1:<unix_seconds>:<nonce>:<hex_hmac>` over `timestamp + "\n" + nonce + "\n" + sha256(body)`, 60-second window, persistent SQLite nonce replay stores (e.g. `$XDG_STATE_HOME/flashalign/replay/nonces.sqlite3`) whose unavailability must deny Owner Mode. `X-Owner-Mode-Effective` returns the actual selection, including `off`.

## Key techniques

- **Fail-closed error contract.** The engine's owner-mode 400s share type/param/code; only the `message` distinguishes them. `proxy.py` matches exactly four full message strings pinned to the engine build; anything changed or unknown is surfaced, never demoted. Only "server started without owner-mode profiles" gets a single stock retry — and only in Verified contacts mode, never in Always On.
- **Identity is attested in code, not in text.** The central security idea: a `VerifiedTurn` is constructed only by the native channel adapter that delivered the message. Text *in* a message claiming to be the owner ("I'm Nat, phone +1555…") is untrusted data, matching the injection-threat model rather than the content-moderation one.
- **Cache isolation.** Stock and profile-mode requests use separate model-cache identities keyed by profile, scale and model revision, so an aligned request can't read or poison a stock KV-cache prefix — profile-mode transforms are prevented from leaking across the boundary.
- **Tool inheritance as policy, not accident.** In Verified contacts mode, per-tool flags (`tool delegate_task off`, `tool sessions_spawn on`) decide which tools pass the profile to verified child work; children adopted from an owner turn inherit its anchor, but a once-steered child stays unverified.
- **Defensive engineering style throughout:** strict JSON parsers everywhere (`object_pairs_hook` rejecting duplicate keys, `parse_constant` rejecting NaN), bounded reads with size caps, symlinks rejected, audit labels that are fixed strings carrying no message content, logging disabled on the proxy because default logs can expose request data.

## Design decisions

- **Per-request profiles instead of shipping abliterated checkpoints.** Abliterated model files circulate freely and uncontrollably (a problem the wiki's Eve Security note covers); FlashAlign keeps the base weights stock and makes alignment an authenticated, revocable, per-request property. The cost: engine integration, a signing protocol, and a 1.09 GB payload for DeepSeek versus "just download the merged model."
- **Governance before capability.** The bulk of the code is identity, approvals, replay stores and staging transactions, not linear algebra. The bet is that refusal-reduction is only acceptable if it's scoped to *verified humans* — restricting abliteration to approved channels reduces injection-driven abuse of an unguarded model.
- **Honesty about limits.** Benchmarks are self-described "small qualification runs" showing 2–10% decode slowdowns; SMS is disabled by default because sender spoofing; the README notes tool outputs within an authorized request still rely on the harness's protections. The security model explicitly says profiles "do not guarantee a particular answer, zero refusals, or protection from every injection attempt."
- **Two modes instead of one dial.** Always On is the blunt instrument (everything routed gets the profile, including subagents); Verified contacts is the careful one. Mode is durable state that "survives restarts and does not re-enable a disabled deployment" — an operational-safety touch.

## Comparison notes

- The wiki's existing [[Abliterated Models and Agent Governance (Eve Security)]] note treats abliterated models as an ungovernable supply: FlashAlign is a direct architectural response — same transform, but moved behind identity, signing and per-request gating. Read together they're the problem statement and one proposed solution.
- It is built as a plugin for [[Hermes]] (native ingress, request and tool-execution hooks) and OpenClaw, relying on the harness for message admission and tool permissions — an instance of the division of labour in [[The Harness Is the Company]]: alignment of the weights is FlashAlign's job, admission and tool execution stay with the harness.
- Its verified-contact turn-binding is the model-weights counterpart of [[Zero Trust for AI Agents]]: caller identity is proven per request from transport facts, headers never attest identity, and revocation applies to subsequent delegated work.
- Unlike [[Apex Flash-1 — Cantina's Open-Weights Security Worker Model]], which makes safety a property of a purpose-trained model, FlashAlign makes it a property of a runtime governance layer bolted onto commodity open weights — a much cheaper and much more revocable path.

#tool #project #inference #alignment #security #local-ai

---
*Sources: [[raw/flashalign]], [[summary/flashalign]]*
*Last updated: 2026-11-02*
