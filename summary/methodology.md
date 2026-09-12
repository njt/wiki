---
url: https://github.com/Chiaro-HQ/methodology
title: Chiaro Methodology
author: Y Assurance PLLC (Chiaro)
date_published: 2026-07-28
date_fetched: 2026-08-06
topics:
  - guardrails-and-feedback-loops
  - security-and-sandboxing
---

# Chiaro Methodology (Summary)

The Chiaro methodology is the complete open-source framework for SOC 2 readiness and audit, published by the CPA firm behind the Chiaro platform. It defines what gets tested (89 controls mapped to 61 Trust Services Criteria), how evidence is collected (an AI agent guided by deterministic rules), how judgments are calibrated (528 worked examples correcting AI decisions), and how a Type II examination works (complete populations by default, not sampling).

The key insight: the methodology is published openly while the platform software remains proprietary. This means anyone can fork the controls, run their own readiness against them, or challenge a specific judgment call — but only a licensed CPA firm can sign the opinion. The calibration examples (528 scenarios where an AI's verdict was corrected, with reasoning) are the most novel artifact: they encode the firm's accumulated judgment as machine-readable training data, 317 correcting over-strict AI and 211 correcting over-lenient AI.

The Type II method departs from industry practice by testing complete populations instead of sampling. Since most evidence is machine-produced, it can be verified at machine speed. Sampling survives only where full retrieval is genuinely impracticable, and even then selection is deterministic-seeded from a SHA-256 hash of the banked population so nobody can steer it. The HIPAA Security Rule crosswalk (v1.1, released same day) maps 64 provisions to the SOC 2 control library, adding 3 HIPAA-specific controls and 5 HIPAA-gated attributes to existing ones.
