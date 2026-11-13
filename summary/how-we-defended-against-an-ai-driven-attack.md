---
url: https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
title: "How We Defended Against an AI-Driven Attack"
author: Eye Security Labs
date_fetched: 2026-11-13
date_published: 2026-11 (series part 1)
topics:
  - security-and-sandboxing
  - agent-orchestration
---

Eye Security's incident-response write-up of what they present as one of the first documented LLM-agent-driven attacks against a real company. The attacker used ARTEX, an autonomous LLM-driven pentesting framework: many agents running in parallel, sharing findings, and adapting within minutes. Entry was a vulnerable custom PHP upload function; from there the agents pivoted through environment variables, a cloud secret store, the internal network (Neo-reGeorg, modified within minutes to survive pod restarts), an engineer's Git identity (leading to a malicious deploy on Kubernetes), and CI runners re-registered every eight minutes.

The defining pattern is what the authors call "machine speed": a stolen API key was in use minutes after discovery, and when the rate limiter blocked them, the agents had a header-spoofing rewrite in about three minutes, peaking at 9,000+ requests a minute before falling back to 17,000+ residential proxies. The attack is noisy (agents label their own work: `artx-pod`, `artx-probe-qa-1`) but relentless — when responders deleted the attacker's workloads, the victim's own deployment automation recreated them within a second.

The defenders fought back with their own agents, run read-only behind a hard guardrail hook that blocked any state-changing command, coordinating through a shared private Git repo with findings synced as pull requests. The agents proved good at exactly the triage that eats human analysts' nights (walking process trees to dismiss false EDR alerts), though findings drifted out of sync and one guess ("probably guest Wi-Fi") propagated into five documents before live data corrected it. The authors' four conclusions: the gap between finding and using a credential has collapsed; the agents are loud where defenders have visibility; breadth means containment is now about finding *all* ways in; and equality of arms — "defending at machine speed means defending with machines, and keeping people in charge of them." Includes a non-exhaustive IOC list.
