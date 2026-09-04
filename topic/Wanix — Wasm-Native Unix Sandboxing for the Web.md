# Wanix — Wasm-Native Unix Sandboxing for the Web

Wanix is a browser-native framework that runs real Wasm and x86 programs in a sandboxed, serverless Unix environment you assemble from custom HTML elements. It is Plan 9's "everything is a file, per-process namespaces" model transplanted to the browser — and an argument, alongside [[Smolbox]], that the local-first web needs a Unix substrate, not just a storage layer.

---

## Key Quotes

> "Run and interact with real Wasm and x86 programs entirely sandboxed in the browser. No server. Inspired by Plan 9."

The pitch in one sentence. The "no server" is doing the same work it does in [[Smolbox]]: the sandbox boundary is *constructed* by the browser, not provisioned by a cloud. There is nothing outside the tab to reach, because the Unix system itself lives inside it.

> "The spirit of Plan 9, in Wasm. Per-process namespaces, everything-is-a-file, and why a research OS from the 90s turns out to be the right model for the local-first web."

The thesis. Wanix isn't borrowing Plan 9's aesthetics — it's borrowing its architecture: namespaces as first-class, composable, bindable objects rather than a single global filesystem. The wager is that the web's real problem isn't "how do we run code in a tab" (Wasm solved that) but "how do we give that code a *world* to run in that is legible and isolatable."

> "Using `type="import"` you can import a remote namespace using 9P over WebSocket or as an embedded iframe."

The most consequential line in the page, and the one that makes this more than a toy. Plan 9's 9P protocol — the "everything is a file server" wire format — is what lets a namespace cross page and origin boundaries. A namespace with an `id` and `allow-origins` becomes importable by another page. That is capability-passing between sandboxes, expressed as a file protocol rather than a permission system.

## Key Themes

#concept #tool #pattern

**Per-process namespaces as the isolation primitive.** Where a conventional sandbox draws one boundary around *the whole program*, Plan 9 namespaces give every task its own private view of the filesystem, assembled from binds. Wanix makes that the HTML author's job: `<wanix-bind>` mounts files, archives, and remote namespaces into a `<wanix-namespace>`, and a task sees only what's been bound in. This is the same capability intuition as [[Extensible Software in the Age of LLMs]] — narrow, explicit references instead of ambient I/O — but done with Unix mount semantics instead of function references.

**Everything-is-a-file as the universal interface.** The Plan 9 move Wanix inherits is that storage, programs, devices, and even a VM's guest filesystem all look like files under one namespace. `#ramfs` is memory, `#web/opfs` is browser persistent storage, `#vm/1/guest` is the exported Linux VM filesystem. Compare [[Mirage (VFS)]], which does the same for *services* — S3, Slack, Postgres as a POSIX tree agents navigate with bash. Wanix does it for *execution environments*; both are betting that "files and a shell" is the interface LLMs and humans are already fluent in.

**Sandbox-as-browser-tab, taken further than Smolbox.** [[Smolbox]] compiles a full x86_64 Alpine VM to Wasm and runs an LLM against it. Wanix is the more general substrate: it can *also* run an x86 VM (`<wanix-vm>` boots Linux via v86), but the VM is one component among many, not the whole system. A Wasm binary, a JS task, an archive, and a remote namespace are all first-class peers. Where Smolbox is a proof-of-concept argument, Wanix is a *library* — a `<script>` tag and some elements, meant to be embedded in pages.

## Critical Analysis

The interesting claim is architectural, not incremental. A decade of "web sandboxing" has meant running one WebAssembly blob inside one iframe and calling it isolated. Wanix's counter-proposal is that isolation isn't a wall you build — it's a *namespace you compose*. A task's entire environment is whatever its author bound in, and nothing else. That is a genuinely different frame, and it's why the 9P import bind matters: it's the mechanism for two sandboxes to share *parts* of their world without sharing everything, which is the exact problem [[Security and Sandboxing]] keeps circling around (blast-radius shrinking, not wall-building).

But the page is a landing page, not a proof. It shows recipes and demos, not the hard cases: what happens when an imported namespace tries to write to its importer, how `allow-origins` is actually enforced, whether the v86 VM's escape surface is any better than the browser's own. The "no server" claim is also the standard client-side caveat — the *runtime* is local, but the CDN that serves `wanix.min.js` is still an observable, mutable distribution channel, the same point [[Smolbox]] concedes about its own privacy claim. The version number (`0.4.0-rc2`) and the typo on the page ("In the futre") both say this is early.

What Wanix adds to the wiki is a concrete instance of a thread that has otherwise only appeared in passing: **Plan 9's model as a live, shipping answer to local-first web isolation.** [[Extensible Software in the Age of LLMs]] names WASM as a sandbox primitive in the abstract; Wanix is someone actually shipping the Unix-namespace version. If it holds, the takeaway is that the browser is not a security boundary to be escaped but a namespace server to be extended.

---

*Sources: [[raw/wanix-dev]], [[summary/wanix-dev]]*
*Last updated: 2026-09-04*
