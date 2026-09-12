# Reverse Engineering the Tesla Firmware Update Process

Pen Test Partners spent a couple of weeks with a real Tesla Model S and mapped, end to end, how the car updates its own firmware: the Tegra boot chain, the dual-bank flash layout, six distinct update mechanisms (legacy shell scripts, a multi-personality updater binary, kernel/bootloader flashes, map data, gateway-relayed ECU updates, USB modem firmware), and the MPC5668G gateway that programs every ECU over UDS. The security verdict: the one genuinely strong boundary is a per-vehicle OpenVPN tunnel, and inside it sits a soft centre — `install.sh` running as root, MD5 and CRC32 doing a signature's job, a self-integrity SHA-512 truncated to 8 bytes, and fixed UDS seeds. It is now also a pre-agent baseline for what this kind of reverse engineering used to cost.

---

## Key Quotes

> "Crucially, we found that it was possible to make requests for any VIN using the VPN for another car."

The authorization gap in one sentence. The VPN certificate authenticates *this* car — its subject is the VIN — while the handshake endpoint authorizes by VIN in the URL path, and nothing checks the two against each other. Authentication was solved; binding the authenticated identity to what it may do wasn't. Precisely the gap [[Authentication Is Largely Solved]] argues the industry still under-builds.

> "The install.sh file can contain arbitrary commands and the entire process runs as root. In order to take control, an attacker can place a valid upgrade package in dropbox to carry out arbitrary actions on the CID or IC."

The legacy update path, distilled. The package format "would be easy for an attacker to recreate"; the `md5sums` file "adds no security and is purely an integrity check"; the only user-visible barrier is a pop-up. Everything behind the VPN is trust.

> "We were surprised to find that, although sha512 outputs 64 bytes, only the first 8 bytes are retained. This means that it possible to find a hash collision using brute force."

The updater's *self*-integrity check — it hashes itself at startup and sends the result in its User-Agent — is truncated to 64 bits. A control that presents as cryptography, functions as a version string. The authors' dry verdict: "It is not a strong protection against malicious action."

> "Tesla upgrades are performed remotely. The gateway must implement the means to perform UDS Secure Access on all the ECUs in the vehicle because the programming device must be implemented in the gateway itself. This makes it easy for an attacker to get their hands on them, to recover and reverse engineer at their leisure."

The sharpest architectural insight in the piece. Dealer-world security assumed the programming laptop was scarce and tightly controlled; OTA moves that laptop into the car, so every seed/key algorithm ships to the customer. Remote updates quietly inverted a decades-old automotive trust model — and several ECUs answer Security Access with *fixed* seeds.

> "Far from being secure, this mechanism has few protections outside of the transport encryption provided by the VPN. This is probably the primary reason why it is deprecated. We do not know why it is still present on the system. Developers may be concerned that removing of one of the numerous scripts could cause unintended consequences."

Dead code as security debt. The dangerous legacy updater survived because nobody was confident removing it was safe — comprehension debt, automotive edition. [[Agents and Acquiring Debt]] makes the same argument about codebases nobody dares touch; this one drives a car.

---

## Key Themes

#reverse-engineering #firmware #embedded #automotive #secure-boot #supply-chain #threat-model #ota-updates

---

## Critical Analysis

**The crunchy shell, soft centre.** Tesla did the hard, novel things well: per-vehicle OpenVPN certificates on an outbound-only tunnel, a gateway bootloader (`noboot.img`) verified by a public key in gateway firmware, HMAC'd download URLs that expire after two weeks, Salsa20-encrypted firmware. Then the interior assumes the boundary holds forever: handshakes over plain HTTP (the updater "has no TLS functionality at all"), CRC32s labelled "signature" in the Redbend tooling, entropy analysis showing ECU `.hex` files carry no signatures, no integrity check at all on map data ("anyone with valid VPN keys can download the entire mapping dataset"), and a development mode that switches signature checking off entirely. This is the same shape [[Everything I Own, Owned]] found in 2025 peripherals — deep protection at one layer, nothing behind it — and it suggests the pattern is the *default state* of firmware engineering, not a recent regression. The single point of failure is the SBK: a symmetric AES key baked into the SoC, and the authors could not determine whether it is unique per vehicle or shared across a fleet.

**The A/B machinery deserves as much billing as the flaws.** Dual-bank everything — ROM → stage1 primary/recovery → stage2 primary/recovery → kernel_a/kernel_b → online/offline squashfs `/usr` — with updates patched onto the *offline* bank "at the raw flash level," signature-checked, then redeployed. That is why a car could survive two weeks of security researchers and still "(mostly) work," and it is the design everyone else has since copied. The escalation strategies named "INDIFFERENT" through "SUICIDE_BOMBER" are a small window into the internal culture of retry logic — the kind of artefact you only see when someone reads your strings.

**A baseline for what firmware RE used to cost.** Two expert researchers, weeks of work, one vehicle — and the open questions outnumber the closed ones: SBK uniqueness ("we would need access to multiple vehicles"), why the CID update refused to proceed, what unknown control blocked the USB-update bug ("we tried this many times, but could not trigger it"), how relay and redeploy actually behave ("we did not observe this during testing"). The stated limiting factor: "the lack of spare parts." Read against what this wiki tracks in 2026 — [[DeepSeek Reverse Engineers TeamSpeak Licensing]] at $3.88 of API credits, [[bt-re-controller — Bluetooth Firmware RE Skill]] encoding a complete firmware-RE methodology as an agent skill — this document measures the per-model human labour whose scarcity is exactly what kept vehicle firmware secret. The secrets stayed secret because the work was expensive.

**Read it as history, not as a Tesla vuln list.** The authors flag it themselves: "the process has recently changed with the Model 3." The extracted VPN keys expired 31 May 2018; the symmetric-SBK weakness predates Tegra's public-key boot support ("not in place before 2015"). The specific findings are period pieces. What survives is the methodology — strings, heavily-commented shell scripts, entropy analysis, QEMU emulation, and honest negative results — and the pattern-level lessons, which have not aged at all.

**The epistemic honesty is the model.** Every claim is tied to an observed script, string, or packet; every gap is named as a gap ("we could not ascertain," "we did not observe," "we do not know why"). No CVE-math, no "hackers could take over your car" paragraph. Security writeups that keep "we found," "we could not determine," and "we did not observe" rigorously separate are rare, and that discipline is what makes the piece citable years later.

---

## Connections

- [[Everything I Own, Owned]] — this piece is the pre-LLM, high-stakes version of schlarp's peripheral teardown, and it strengthens his thesis: the "update mechanism as attack surface, protection concentrated at one layer" pattern predates the agent era and shows up in a vehicle from a manufacturer that demonstrably invested in security. It also supplies the denominator for his cost-collapse argument: expert-weeks then, agent-evenings now.
- [[Transsion Telemetry — Embedded Mobile Surveillance]] — the same "your device is not your own" remote architecture, seen from the vendor's side of the table: Tesla's plumbing enabled feature enablement (enable-autopilot-after-purchase.sh) and daily token rotation as *product features*. The infrastructure that enables surveillance also enables monetizing hardware the customer already owns.
- [[bt-re-controller — Bluetooth Firmware RE Skill]] — this writeup is a catalogue of exactly the mechanical expert labour (spec-table lookups, function naming, string cross-referencing, consensus on contested findings) that darkmentorllc's ~35-phase DAG and cross-model merge automate. It reads like the manual that skill encodes.
- [[DeepSeek Reverse Engineers TeamSpeak Licensing]] — the "$3.88 vs expert-weeks" contrast, and the same failure shape: TeamSpeak's protection was deep but brittle; Tesla's was broad but inconsistent. Both fell to tracing the chain end-to-end, and both writeups conclude that the economics — not the cleverness — is the real story.

---
*Sources: [[raw/reverse-engineering-the-tesla-firmware-update-process]], [[summary/reverse-engineering-the-tesla-firmware-update-process]]*
*Last updated: 2026-09-13*
