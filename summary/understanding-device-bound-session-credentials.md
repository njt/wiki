---
url: https://andrewlock.net/understanding-device-bound-session-credentials/
title: "Understanding Device Bound Session Credentials (DBSC)"
author: Andrew Lock
date_fetched: 2026-09-20
topics:
  - security-and-sandboxing
  - software-engineering-craft
---

Andrew Lock's explainer of Device Bound Session Credentials (DBSC), a browser protocol aimed at the oldest weakness in web authentication: session cookies are bearer tokens, so anyone who extracts one can replay it from any machine. DBSC closes that gap by binding the session to the device it was issued on — the browser generates a key pair inside the TPM/secure enclave, signs server challenges with the private key (which never leaves the hardware), and the server only accepts requests provably signed by the same machine.

The article walks the full protocol end to end: the server advertises DBSC via a `Secure-Session-Registration` header at login; the browser registers a TPM-held public key at a registration endpoint; the server swaps the long-lived auth cookie for a short-lived one (minutes, not months) plus a refresh URL; and the browser silently re-proves key possession at the refresh endpoint whenever the cookie expires. Crucially, DBSC is progressive enhancement — servers that adopt it keep their existing authentication code, and browsers without support simply ignore the header.

Lock's verdict is a qualified yes: implement it, since the fallback behaviour is harmless and it neutralises session hijacking, but wait for a canonical framework implementation first, because Scott Helme's production experience shows real-world edge cases, and ad blockers can break the flow outright. Only Chromium supports DBSC today.
