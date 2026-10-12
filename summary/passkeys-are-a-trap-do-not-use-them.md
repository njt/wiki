---
url: https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them
title: "Passkeys are a Trap, Do Not Use Them"
author: Nonfunctional (Substack)
date_fetched: 2026-10-12
date_published: n.d.
topics:
  - security-and-sandboxing
  - ideas-and-culture
---

A polemical essay against the "just use passkeys" industry push, arguing that the WebAuthn standard hides a parts catalogue of very different credential types under one friendly name, and that the catalogue's endgame — remote attestation — is a mechanism for websites to control which software you may run, not merely how you log in.

## The argument

The author grants passkeys' headline virtues up front: credentials can be domain-bound (resistant to man-in-the-middle), hardware-backed keys can't be exfiltrated, and convenience is real. The problem is elsewhere: "passkey" is one word for many unlike things. The WebAuthn spec itself barely uses the term, preferring discoverable/undiscoverable, hardware/software-backed, attested/unattested credentials. The author walks the taxonomy — copyable vs non-copyable, transferable vs non-transferable, hardware-backed vs software-backed, user-present vs non-user-present, user-verified vs non-user-verified — presenting each option in its best light, and arrives at the crux: **attested** passkeys. Attestation cryptographically proves to the website which passkey manager, OS, and firmware you run, using the tamper-proof cryptoprocessor in every modern device. Your software can lie about every other flag; it cannot lie about attestation.

## The ulterior motive claim

The FIDO Alliance's membership (Big Tech, banks, governments, intelligence agencies, but no consumer advocacy groups) suggests the motive is not your convenience. Attested passkeys let platforms verify — and therefore eventually require — approved software: no custom ROMs like LineageOS, no unapproved browsers or extensions, no credential sharing (the running joke: Netflix does not want you to share your passkeys). The flags have grown from WebAuthn L1/L2 to L3, with two still "reserved for future use."

## The conclusion — and the addendum

The essay concludes: reject passkeys, keep using TOTP (an open standard you can implement yourself, even on a Game Boy), wait for something better-defined. But the addendum partially undercuts the headline: the author admits passkeys are "incredible" and domain-bound, that he *wants* to recommend them, and that the real grievance is remote attestation plus the deliberately confusing naming — and that a self-contained hardware security key (a YubiKey) gets you passkey security without attestation spilling into your whole OS.
