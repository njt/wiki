---
url: https://www.nextkicklabs.com/p/agentic-ai-security-stack-book-release
title: The Agentic AI Security Stack
author: Fernando Lucktemberg
site: Next Kick Labs (Substack)
date_fetched: 2026-07-05
date_published: 2026-07-01
---

# The Agentic AI Security Stack

**Subtitle:** Deploy secure agentic AI systems. This free 200+ page reference provides a unified threat model, traces kill chains, and maps every control to OWASP, MITRE ATLAS, & CSA MAESTRO.

**Author:** Fernando Lucktemberg
**Published:** July 1, 2026
**License:** Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 (CC BY-NC-ND 4.0)
**PDF:** https://www.nextkicklabs.com/api/v1/file/19237a18-eb0e-46ff-bde5-787ea98a911f.pdf

## Summary

The article announces the release of a free 200+ page reference book on agentic AI security. Available as a PDF download with no email or paywall required, under a Creative Commons license.

### For Security Leaders

Most agentic AI deployments lack a unified security reference grounded in a threat model, resulting in gaps attackers can exploit. The book covers twelve interception points mapped to OWASP, MITRE ATLAS, and CSA MAESTRO. Key concerns include structural gaps from missing threat models, framework alignment difficulties, and the cost of inaccessible security knowledge.

### From First Article to Free Reference

The book grew from a planned seven-module guide announced in February 2026 to twelve chapters. The author traces this to a threat-model-first approach: "start from the other direction: name the attacker's objective, trace the kill chain step by step." The prerequisite was three months of architecture writing (Oct–Dec 2025) on how agentic systems actually work.

### Building in Public

Between January 13 and March 31, 2026, the author published one article per security layer. Writing publicly surfaced gaps via reader questions. Examples: credential architecture questions pushed deeper into RFC 8693 token exchange semantics; MCP security questions revealed that "OAuth verifies that the client is connecting to the authorized server. It does not verify that the tool description that server delivers is free of injected instructions."

### The Framework Spine

Three frameworks anchor the book:
- **CSA MAESTRO** — architectural layer map
- **MITRE ATLAS** — attacker vocabulary (extending MITRE ATT&CK for AI systems)
- **OWASP** — vulnerability classification (Top 10 for LLM Applications and Top 10 for Agentic Applications 2026)

### The Structure the Threat Model Produced

Five chapters were added beyond the original seven:
1. **Agent-native identity (Ch. 8)** split from credential architecture (Ch. 2) because they address different failure modes
2. **Supply chain and skill verification (Ch. 10)** emerged from grounding the kill chain in a specific attack scenario

The book traces one kill chain through twelve interception points: a vendor intelligence agent attacked via prompt injection in a retrieved document, credential extraction, container escape, data exfiltration via disguised DNS queries, persistent memory manipulation, and multi-agent propagation.

### The Peer Review

Venkata Sai Kishore Modalavalasa (Chief Architect at Straiker, OWASP author contributing to Top 10 for LLM Applications, Top 10 for Agentic Applications 2026, OWASP AI Exchange, OWASP GenAI Security Project) reviewed the full manuscript. His review produced edits across all twelve chapters, including opening vignettes for each chapter.

### Why It's Free

The author argues that gating security knowledge behind signup forms or paywalls worsens the problem. The CC license allows internal sharing, educational use, and citation without legal friction. "The organizations that most need this reference are not the ones with dedicated AI red teams."

## Key Quotes

1. "name the attacker's objective, trace the kill chain step by step, and let the required controls fall out of that analysis"
2. "OAuth verifies that the client is connecting to the authorized server. It does not verify that the tool description that server delivers is free of injected instructions"
3. "The organizations that most need this reference are not the ones with dedicated AI red teams and enterprise security budgets"
4. "gaps can survive until final review. When the same content ships as a published article, the gaps come back as reader questions within 48 hours"

## Links & References

- PDF Download: https://www.nextkicklabs.com/api/v1/file/19237a18-eb0e-46ff-bde5-787ea98a911f.pdf
- Author's Substack: https://substack.com/@nextkicklabs
- Next Kick Labs: https://www.nextkicklabs.com/

**Frameworks referenced:** OWASP Top 10 for LLM Applications, OWASP Top 10 for Agentic Applications 2026 (ASI01–ASI10), MITRE ATLAS, CSA MAESTRO, RFC 8693 (token exchange)

**Reviewer affiliations:** Straiker (Chief Architect), Akamai (following Cyberfend acquisition)
