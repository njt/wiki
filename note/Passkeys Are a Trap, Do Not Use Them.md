# Passkeys Are a Trap, Do Not Use Them

A polemic against the passkeys push whose most durable contribution is its taxonomy: "passkey" is one brand over many unlike credentials (copyable/non-copyable, transferable/non-transferable, hardware/software-backed, user-present/verified, attested/unattested), and the terminal option — remote attestation — is less about login security than about websites controlling which software you may run. The conclusion ("do not use them") is weaker than the anatomy that supports it.

---

The essay's structure is a deliberate trap of its own: it presents every passkey flavour in its best light, then reveals that attestation exists to make your software unable to lie about which flavour it is. Once attestation is in the standard, a website can — at a moment of its choosing — reject everything the FIDO Alliance hasn't approved, including your custom ROM, your self-vibecoded passkey manager, and your unapproved browser.

## The taxonomy (the part worth keeping)

The author's central and best move is insisting that WebAuthn's spec "reads less like a standard and more like a parts catalogue" — the word *passkey* appears almost not at all in it. The dimensions, each independently optional and independently enforceable by the server:

- **Copyable vs non-copyable** — can the credential be duplicated across devices, or is it pinned to one?
- **Transferable vs non-transferable** — a non-copyable key can still *move* (die with the old phone, live on the new one). This trips intuition: digital objects that vanish from the source when sent.
- **Hardware-backed vs software-backed** — a file you own and store, versus a secret sealed in a cryptoprocessor (TPM, Secure Enclave) that not even you can extract.
- **User-present vs non-user-present** — must a human press something, or can software authenticate autonomously?
- **User-verified vs non-user-verified** — does it additionally demand biometrics or a PIN?
- **Attested vs non-attested** — does the cryptoprocessor, OS, and firmware chain-of-trust cryptographically prove you're running *approved* passkey software?

Everything before attestation is a dial *you* set. Attestation is the dial the *website* sets — via authenticator data flags, which grew from WebAuthn L1/L2 to L3 in 2022, with two flags still "reserved for future use."

## Key quotes

> "Unlike with any other kind of login credential, **you do not have ultimate control over your passkeys**."

The thesis, and it's a genuine insight about the credential's *design* rather than its math: passwords, TOTP seeds, and even physical keys are all yours to mishandle. Passkeys embed a negotiation with the relying party into the credential itself.

> "Your passkey software can lie about whatever passkey features you like… Unless we're talking about an Attested… passkey. In which case, no. That can't be faked."

The pivot of the whole essay. Every earlier dimension is a client-side claim the server must trust; attestation is the one dimension the hardware chain-of-trust guarantees. That's a real technical distinction — and also exactly the property that makes attestation the control point.

> "They can use passkeys to prevent you from running unapproved software."

The ulterior-motive claim. This is where the essay overreaches and also lands its most interesting point: the infrastructure needed for "sorry, your passkey isn't secure enough — download our app" is the same infrastructure that could enforce mandatory client-side scanning, ID verification, or extension-lockdown browsers. The Chrome/uBlock Origin example makes it concrete rather than hypothetical.

> "It's very unfortunate that the FIDO Alliance has ruined a good thing by slipping in remote attestation and by making no attempt to let the difference between these various kinds of passkeys be evident to the user."

The addendum, which quietly retracts the headline. The author concedes passkeys are "incredible," domain-bound, and MIT-resistant — that he *wants* to recommend them — and locates the villainy in attestation plus deliberately confusing branding. His own escape hatch (a YubiKey is a self-contained passkey whose attestation "doesn't have to spill out and lock down your whole operating system") concedes that WebAuthn itself is fine.

## Critical take

Nat's steer is right: the taxonomy is the valuable part, the conclusion is the weak part. "Do not use passkeys" doesn't survive the essay's own addendum — the author ends up recommending a passkey in hardware form. The stronger, honest version of the argument is: *attestation is a bundled trojan inside an otherwise good standard, and the naming deliberately hides the bundle*. That's a spec-governance critique (worth raising with the W3C/WebAuthn process), not a reason to keep typing `solarwinds123`-class passwords or SMS codes, which the essay itself dismantles effectively. The FIDO-membership guilt-by-association section (intelligence agencies! banks!) is the weakest part — no consumer groups at the table is a fair observation, but it doesn't follow that the flags were designed to catch dissidents. Slippery-slope risk is real (two "reserved" flags) but speculative.

The essay is also a mirror of debates the wiki tracks elsewhere: hardware trust anchors as an enforcement point, and the perennial tension between open standards and platform control of clients.

## Connections

This essay directly complicates [[Authentication Is Largely Solved]]: Windley files passkeys and FIDO under "quietly closed," treating authentication as done — this author argues the *standard that closed it* smuggles in a platform-control mechanism, so "solved" is true for the math and contested for the politics. It nuances [[Understanding Device Bound Session Credentials]]: DBSC is the same hardware-anchored pattern (TPM-held keys, browser-level enforcement) proposed by the platform vendors, and this essay is the sharpest articulation of the civil-liberties counterargument that pattern invites — hardware binding is a security win whose second use is client attestation. It strengthens [[Flexible Authentication (Airbnb)]]: Airbnb's server-driven, "try another way" auth design assumes the *server* chooses the challenge — exactly the server-side discretion this author warns becomes a ratchet ("unfortunately your passkey isn't secure enough; download our app").

---
*Sources: [[raw/passkeys-are-a-trap-do-not-use-them]], [[summary/passkeys-are-a-trap-do-not-use-them]]*
*Last updated: 2026-10-12*
