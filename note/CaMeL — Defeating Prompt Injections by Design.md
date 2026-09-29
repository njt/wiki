# CaMeL — Defeating Prompt Injections by Design

CaMeL (Debenedetti et al., Google DeepMind/Google + ETH Zurich, arXiv 2503.18813) is the most serious attempt yet to answer prompt injection with *system design* rather than model training: a privileged planner compiles the user's query into Python, a quarantined tool-less model parses untrusted data into structured form, and a custom interpreter enforces capability-based security policies on every tool call — giving provable guarantees that untrusted data can never redirect the program flow or leak private data.

---

## The core move: treat the LLM as an untrusted component

CaMeL's thesis is a category shift. Every prior defense tries to make the model more robust — delimiters, prompt sandwiching, fine-tuning, instruction hierarchies. The authors observe that none of these give *guarantees*, and rebuild the agent from software security's playbook instead: Control Flow Integrity, access control, information-flow control.

> "CaMeL does not rely on model behavior modification. Instead, it provides an environment where predefined policies prevent unintended consequences caused by prompt injection attacks."

The architecture instantiates Simon Willison's Dual LLM pattern for the first time concretely: the **P-LLM** sees only the trusted user query and writes Python expressing intent; the **Q-LLM** has no tools and reduces unstructured (possibly hostile) tool output to a Pydantic-typed schema. The P-LLM never sees tool outputs — it writes code over variables whose contents it cannot read. That isolation alone kills the control-flow hijack.

## The second half: data flow, not just control flow

The paper's sharpest insight is that Dual LLM is *insufficient*: even when the plan can't be hijacked, the *arguments* to tool calls can be. A prompt injection in meeting notes can make the Q-LLM return an attacker's email and a different document — an SQL-injection-style attack on parameters, not structure. CaMeL's answer is capabilities: every value carries provenance and an allowed-readers set, a data-flow graph tracks dependencies through the interpreter, and Python-function security policies gate each tool call (e.g. a document may only be emailed to its existing readers). Exfiltration becomes impossible *by construction*, with user approval as the fallback prompt.

## Honest about the edges

The paper is unusually candid about limits, which is part of why it matters:

> "No, prompt injections are not fully solved."

Explicit non-goals: text-to-text attacks that don't touch the data flow (misleading summaries, phishing shown to the user), for which fine-grained provenance can at least power a "this came from an untrusted source" UI. Side channels survive: indirect-inference leaks, exception-triggered one-bit leaks, timing channels — mitigated by a STRICT interpreter mode but not eliminated. And the classic capability-system costs reappear: implementation effort, the ecosystem-participation problem for third-party tools, and user fatigue from de-classification prompts. The CFI/ROP analogy is apt — CaMeL expects gadget-style attacks that compose individually-allowed flows, as their own "data becomes control flow" case study demonstrates.

## Results

On AgentDojo, CaMeL roughly preserves utility (except Travel, where underdocumented APIs hurt; newer models largely fixed that by using the Q-LLM to parse) and eliminates nearly all of 949 attacks — GPT-4o-mini falls to 276 successful attacks via its instruction-hierarchy tool-calling API, to zero with CaMeL. A cheap Q-LLM (Haiku-class) costs almost nothing in utility while cutting cost, and moves sensitive-data parsing on-device.

## Analysis

This is the paper to point at when someone claims prompt injection is unsolvable or, worse, solved by better prompting. Its real contribution is methodological: it imports sixty years of software security (CFI, capabilities, IFC) into agent design and — crucially — *formalizes the threat as a game* (PI-SEC) so defenses can be judged against guarantees rather than benchmark scores. The trade it accepts is real, though: CaMeL requires someone to write policies, tools to cooperate on tagging, and users to approve downgrades — all the adoption friction that killed capability systems the first time. The "data requires action" failure mode (tasks where the *plan itself* depends on untrusted data) is the structural limit of the whole dual-LLM family, and the paper admits no current model solves it. Note also the quiet irony: the defense works because it *shrinks* what the model does — the P-LLM becomes a code generator that can't see data, which is precisely the verification-over-generation posture the rest of this wiki keeps rediscovering empirically.

**Themes:** #concept #pattern #security

**Related:** this strengthens [[LLM01 Prompt Injection (OWASP)]]'s mitigation list by replacing "may not exist" heuristic defenses with the first design that offers guarantees; it sharpens [[Bounding the Blast Radius — Prompt Injection Defenses]]' taxonomy with a fourth layer that isn't heuristic at all; it is the enforcement half of the access-control story told in [[The Agent Access Model]] and [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]], turning task-scoped credentials into per-value dataflow policy.

---
*Sources: [[raw/2503-18813v2]], [[summary/2503-18813v2]]*
*Last updated: 2026-09-29*
