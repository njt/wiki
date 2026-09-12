---
url: https://textslashplain.com/2026/08/04/security-is-hard-yall/
title: "Security Is Hard, Y'all"
author: Eric Lawrence
site: text/plain
date_published: 2026-08-04
date_fetched: 2026-08-06
topics:
  - security-and-sandboxing
---

Eric Lawrence (former Microsoft Edge engineer, now at Cloudflare) narrates his experience trying to claim a handle on Cloudflare's new Wallet product — and being convinced, with good reason, that he was being phished. Every signal screamed attack: the product lived on `cloudflare.pay` (not `cloudflare.com`), the OAuth consent screen didn't recognize the company's own feature, a suspicious green checkmark looked like an emoji shoved into an attacker's display name, the Cloudflare dashboard had no link to the new product, and the AI chat agent asked for full account access. Lawrence reported it as a phish, only to discover it was a legitimate Cloudflare product. The post uses this experience to argue that security UX failures make it impossible for users to distinguish legitimate from malicious — and equally impossible for URL reputation services like SmartScreen and SafeBrowsing to block malicious sites without false positives. Concrete recommendations for developers: host under your trusted domain, show security info in trustworthy places, make scam reporting trivial, and test your reporting flows. The closing plea: never blame the victim — they've got an impossible job.
