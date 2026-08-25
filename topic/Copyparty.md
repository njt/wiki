# Copyparty

Copyparty is a single-file Python file server that turns a folder on almost any machine into a multi-protocol sharing hub — HTTP(S), WebDAV, FTP(S), SFTP, TFTP, and a minimal SMB server all at once, with resumable parallel uploads, content dedup, and in-browser file management. It is the anti-Nextcloud: no database imports, no blobs, no sync daemon — it serves your existing files as-is, with the stated philosophy of "do all the things, and do an *okay* job."

---

## Architecture

Copyparty is a **hub-and-workers** process design, not a web framework in the usual sense.

- **`SvcHub`** is the single hub process. It owns the shared, process-unsafe resources: the auth/permission model (`AuthSrv`), the SQLite file index, thumbnail generation (`ThumbSrv`), and the listening sockets (`TcpSrv`).
- **One worker process per CPU core** (`MpWorker`) each runs its own `HttpSrv` plus a thread pool (`HttpConn`). The hub itself never touches HTTP; workers never touch the shared state directly.
- Workers and hub communicate over a **homemade RPC**: messages are `(retq_id, dest, args)` tuples put on a `multiprocessing.Queue`. `dest` is a dotted attribute path resolved with `getattr` (e.g. `"httpsrv.listen"`), and replies come back on a per-message return queue keyed by `retq_id`. Exceptions are serialized and re-raised on the caller's side, so a crash in a worker looks like a normal exception at the call site.
- **Socket handoff instead of `fork`**: `TcpSrv` binds each socket once, then passes the file descriptor through the queue to every worker with `broker.say("httpsrv.listen", srv)`. Each worker runs an `accept()` loop on the shared socket; the kernel load-balances connections across workers. This is a pre-fork-style fan-out that works even on Windows, where the start method is `spawn`.

A single-threaded fallback (`BrokerThr`, `num_workers=1`) exists for small machines, and multiprocessing is auto-disabled on macOS and single-CPU boxes unless forced. This one codebase runs the whole spectrum: from a Raspberry Pi to a many-core server, with the same `HttpSrv`/`HttpConn` logic either way.

## Key techniques

- **`up2k` upload protocol** — resumable, segmented, checksummed, parallel uploads. Files are sliced adaptively (1 MiB base chunk, grown until a file has ≤256 chunks — or ≤4096 chunks once it exceeds 32 MiB), each chunk hashed, and the chunk-hash list + size + a per-server salt folds into a 44-char **`wark`** (content address). Chunks write in place to a `.<name>.PARTIAL` file and atomic-rename on completion.
- **"Race the beam"** — because chunks land in the final file as they arrive, a downloader can start reading a file *while it's still uploading*, with `sendfile`-based download throttled to stay just behind the write cursor. This is a genuine throughput win for large files.
- **Content dedup** — identical content (same `wark`) is stored once; duplicates become symlinks or reflinks (`FICLONE`) rather than second copies. Combined with the `.PARTIAL` dotpart convention, this also enables unpost/filekey links keyed to content rather than location.
- **`bos/` — a bytes-oriented filesystem layer** — every path operation is wrapped in `fsenc`/`fsdec` that round-trips *arbitrary bytes*, not Unicode. On Unix that's `surrogateescape`; on Windows it's the `\\?\` long-path prefix with UTF-16 encoding. This is what lets copyparty serve mojibake or invalid-UTF-8 filenames that would crash a naive `os.listdir`.
- **Sendfile done right** — `sendfile_kern` uses `os.sendfile` with `select`/`poll` and a sleep-based throttle for shaping; a pure-Python `sendfile_py` falls back where the syscall is unavailable.
- **Per-volume SQLite index** — `.hist/up2k.db` (schema `DB_VER=6`) stores wark/path/mtime/size per file for fast search (`U2idx`) and dedup decisions, kept in-memory as an upload registry.
- **Zeroconf** — mDNS/SSDP/UPnP announce the server on the LAN, with QR-code URLs and a full-featured browser client (upload progress, thumbnails, audio player).

## Design decisions

- **Serve files as-is, no import step.** Nextcloud/Seafile swallow files into a store and a database; copyparty serves whatever is already in your folders. The trade-off is that it can't do full bidirectional sync — the author is explicit that sync is *never* going to be supported, and that S3-style object storage is reached by mounting it with rclone rather than reimplemented.
- **"Inverse Unix philosophy."** One binary, one process tree, many protocols. The README embraces being a jack-of-all-trades rather than a purist tool. Individual features (SMB, SFTP, TFTP) are admitted to be weaker than their dedicated counterparts.
- **Python + one dependency.** Jinja2 is the only mandatory dep; everything else degrades gracefully. This is what makes the 0.6 MB SFX / 6 MiB `.exe` packaging viable and the whole thing drop-in portable.
- **Multiprocessing over asyncio.** Rather than adopt `asyncio` (and rewrite everything), copyparty gets parallelism by running the *same* threaded code in N processes and bolting a queue RPC on top. It sidesteps the GIL while keeping the codebase readable to a thread-model Python programmer.
- **Bytes-first, everywhere.** The `bos`/`fsenc` layer is a deliberate stance that filenames are bytes, not text — a correctness decision most servers punt on until a user hits a broken filename.

## Comparison notes

- **vs. [[The Limits of Generalized Sync]]** — copyparty is an existence proof *for* that page's thesis: its author rejects generalized sync outright ("sync will never be a feature") and instead makes uploads so resumable, segmented, and deduped that one-directional transfer covers the common case. It's the "specialized, local, verifiable" end of the spectrum.
- **vs. [[Stash — Conflict-Free Folder Sync]]** — the two are nearly inverse. Stash wants *eventual identical copies everywhere* via a merge algorithm; copyparty wants *one canonical copy served everywhere* and treats the folder as the source of truth, with no replication model at all.
- **vs. [[Petals — Decentralized LLM Inference]]** — the shared idea is *segmenting content and naming it by content* (copyparty's `wark` chunk-hash-list is the same trick as Petals' block-hash naming), and both use a coordinator-and-workers topology. Copyparty's coordinator is a trusted local hub; Petals' is a decentralized swarm.
- **vs. [[What Color is Your Function]]** — copyparty deliberately *rejects* the async-colored world. Instead of `asyncio` it pays the multiprocessing tax (queue RPC, `spawn` start) to keep its handler code synchronous and threaded, an architectural trade the coloring essay doesn't address directly.

## Tags

#tool #project #self-hosted #file-server #sync #networking #python

---

*Sources: [[raw/copyparty]], [[summary/copyparty]]*
*Last updated: 2026-08-25*
