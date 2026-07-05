# BFId — WiFi Identity Inference via Beamforming Feedback

A 2025 ACM CCS paper demonstrating that WiFi beamforming feedback information (BFI) can identify individuals with 99.5% accuracy — *better* than the more-studied CSI approach, while requiring a *weaker* adversary model. This inverts the conventional wisdom that BFI, being lossy-compressed CSI at lower temporal resolution, would be inferior for sensing. The authors explicitly frame their work as a privacy *attack*, not a "privacy-preserving" system.

---

## What's New

WiFi sensing has been known for years, but prior work focused almost exclusively on **Channel State Information (CSI)** — raw physical-layer data that requires custom firmware and specific (often ancient) hardware like the Intel 5300 NIC from 2008. BFI was introduced with WiFi 5 (802.11ac) beamforming and has a crucial difference: **it's broadcast unencrypted over the air**, meaning any device within range can passively record it without network credentials.

The paper makes three core contributions:

1. **First BFI-based identity inference attack** (BFId) — an LSTM-based model with no domain-specific pre-processing
2. **Largest-ever WiFi sensing dataset** — 197 participants, simultaneous CSI + BFI from four perspectives across five walking styles
3. **BFI beats CSI** — higher accuracy (99.5% vs 82.4%) with a weaker adversary model

## Key Quotes

> "While much of the extant literature focuses on the utility of WiFi sensing, the privacy threats appear clear: with WiFi networks in the most private places and inferences such as activity recognition shown to have high accuracy, adversaries may be able to infer very sensitive information."

The paper's framing is refreshingly direct. Most prior work sells WiFi sensing as a feature ("privacy-preserving identification!"). The authors call bullshit:

> "Even though related work may consider their identification systems to be 'privacy-preserving' [...], 'less privacy intrusive' [...], 'not cause any privacy concerns' [...] or 'avoid[ing] [...] personal privacy invasion' [...], we consider this to be an attack on privacy."

> "In fact, BFI-based identification even performs significantly better than CSI. Possible explanations for this include that the compression of the feedback matrix into the angles [...] work as a form of pre-processing and noise removal."

This is the paper's most surprising empirical finding. The compression inherent in BFI isn't a weakness — it's a feature that filters noise and retains only the signal relevant to identification. Spatial resolution matters more than temporal resolution.

> "Our results can be found in Figure 6. We find that, as expected, identification accuracies are very low. For those cases where we trained with perspective 1, all accuracies are below 5%."

Identification doesn't transfer across perspectives. An adversary can't train on recordings from one physical location and identify people from a different angle. This is both a limitation and a rare bright spot for privacy.

> "With this hardware making its way into millions of homes, the privacy concerns are severe."

## Threat Model

The adversary is **passive** — they don't need to transmit, authenticate, or join the network. They just listen:

- **CSI adversary**: needs custom firmware, specific NICs, and is limited to one perspective (the channel between AP and malicious node)
- **BFI adversary**: any WiFi NIC in monitor mode, records *every* client's beamforming reports from a single position, multiple perspectives for free

The paper draws an evocative scenario: an oppressive state records protesters walking past a coffee shop's WiFi, then identifies them later in benign situations. The "inverse panopticon" — seemingly harmless WiFi infrastructure as a surveillance mesh.

## Key Findings

| Experiment | BFI Accuracy | CSI Accuracy |
|---|---|---|
| Normal walking (baseline) | 99.5% ± 0.38 | 82.4% ± 0.62 |
| Cross-walking-style (train normal, test backpack) | ~88% | ~55% |
| Cross-walking-style (train normal, test fast) | ~65% | ~10% |
| Cross-perspective | <5% (most) | <5% (most) |
| Empty room → participant matching | 2.34% (top-2) | — |

- **Scaling with population**: BFId on BFI matches the best CSI-specific method (LW-WiID) even with simple architecture and no pre-processing
- **Time-coarsening robustness**: Reducing BFI sample rate has minimal impact on accuracy — a straightforward mitigation (fewer beamforming probes) doesn't work
- **Perspective robustness**: All four perspectives (parallel, high-across, low-across, non-LOS) achieve high accuracy individually; only cross-perspective matching fails

## Critical Analysis

**This paper is important because it changes the privacy calculus.** Most WiFi sensing discourse has been utility-first ("look what we can detect!") with privacy hand-waves. This paper says: no, the weaker adversary model + higher accuracy means BFI-based tracking is the *real* threat, and it's already deployed in millions of devices.

**The 99.5% number deserves scrutiny.** The study was controlled — no baggy clothes, no skirts, no heeled shoes. In the real world with winter coats and backpacks and crowds, accuracy would drop. The authors acknowledge this. But the controlled conditions also made identification *harder* in one way (less clothing diversity across participants), so the net effect is unclear.

**The cross-perspective failure is both good and bad news.** Good: an adversary can't build a universal tracking system from one listening post. Bad: they don't need to — a single device records *all* beamformees' perspectives simultaneously. They can just train per-perspective models.

**The paper's weakness is the same as all WiFi sensing work**: we don't understand *why* it works. The authors are honest about this — the semantics of BFI angles are opaque, and identification succeeds through brute-force ML pattern matching rather than domain understanding. This makes defenses harder to design.

**The mitigation landscape is bleak.** Encrypting BFI would require a WiFi standard revision and break compatibility. Adding noise risks degrading beamforming itself (packet loss). Reducing channel sounding frequency barely affects accuracy. The paper is essentially saying: this attack vector exists, is easy to exploit, and has no obvious fix. Standardization of WiFi sensing in 802.11bf, without privacy protections, looks reckless in this light.

**Connection to the AI safety discourse**: This is a concrete example of dual-use technology where the "legitimate" application (better WiFi throughput) accidentally created a surveillance capability. The ML model is almost beside the point — a simple LSTM, no fancy architecture — the danger is in the signal, not the model.

## Themes

#privacy #surveillance #wireless #security-research #biometrics #dual-use

---
*Sources: [[raw/bfid-wifi-identity-inference]]*
*Last updated: 2026-07-05*
