---
url: https://github.com/9001/copyparty
title: "Copyparty"
author: ed (9001)
date: 2026-08-17
date_fetched: 2026-08-25
---

Copyparty is a portable file server that turns almost any device into a file-sharing hub reachable from any web browser. One self-contained Python file (a 0.6 MB SFX, or a 6 MiB Windows .exe) serves HTTP/HTTPS, WebDAV, FTP(S), SFTP, TFTP, and a minimal SMB/CIFS server simultaneously, announcing itself on the LAN via mDNS/SSDP. The only mandatory dependency is Jinja2 — every other feature (thumbnails, audio transcoding, SFTP, argon2 password hashing) degrades gracefully if its optional library is missing.

Its flagship feature is `up2k`, an upload protocol with resumable, segmented, checksummed, and parallel uploads. Files are sliced into adaptive chunks (1 MiB base, growing until the file has ≤256 chunks), each chunk hashed, and the chunk-hash list plus size and a per-server salt folds into a 44-character content address called a *wark*. Chunks are written in place to a `.<name>.PARTIAL` file and atomically renamed on completion, so a downloader can even read the file while it's still uploading ("race the beam"), and identical content can be deduplicated via symlink or reflink instead of stored twice.

Architecturally it's a hub-and-workers design: one `SvcHub` process owns the shared resources (config/auth, the SQLite file index, thumbnail generation, the listening sockets), while one worker process per CPU core — each running its own `HttpSrv` and thread pool — handles the actual HTTP traffic. Workers talk to the hub through a tiny homemade RPC over `multiprocessing.Queue` (dotted-path dispatch, serialized exceptions), which sidesteps Python's GIL to get genuinely parallel request handling while keeping the code written as if it were single-process. Its stated philosophy is "inverse unix philosophy — do all the things, and do an *okay* job," and unlike Nextcloud or Seafile it serves files as-is from your existing folders rather than importing them into a database.

See [[Copyparty]] for the architecture deep-dive.
