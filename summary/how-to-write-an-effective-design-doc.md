---
url: https://refactoringenglish.com/excerpts/write-an-effective-design-doc/
title: "How to Write an Effective Software Design Document"
author: "Michael Lynch"
date_fetched: 2026-07-03
date_published: 2026-06-24
publication: "Refactoring English (refactoringenglish.com)"
book: "Refactoring English: Effective Writing for Software Developers"
---

# How to Write an Effective Software Design Document

Michael Lynch, excerpt from his forthcoming book *Refactoring English: Effective Writing for Software Developers*. Lynch has written design docs as a developer at Google, Microsoft, and his own companies.

## When Should You Write a Design Doc?

Six yes/no questions:
1. Does the project require coordination among >1 person?
2. Will it take >3 months of full-time work?
3. Will the software remain in production beyond 6 months?
4. Will the project require cross-team coordination?
5. Are there ambiguous or unclear requirements?
6. Would time spent designing prevent a catastrophic future outcome?

Answering yes to any one suggests it's likely worth it; two or more makes it "almost certainly" worthwhile.

## How Much Should You Invest?

No universal rule. A design doc can range from a single page to 50 pages requiring multi-team signoff. "Sometimes, the right amount to invest in a design doc is zero."

## The Guiding Question

**"What's the penalty for being wrong?"** — Language/framework choices are high-penalty decisions worth documenting; UI details like "load more" vs. displaying all are trivially changeable and not design-level concerns.

## Components of a Design Doc

1. **Title** — Short, distinctive, evocative. "RecencyBank" good; "Project Flying Silver Horse" bad.
2. **Metadata** — Author (name + email), creation date, authoritative URL.
3. **Objective** — Single-sentence purpose statement in plain language on the first page.
4. **Background** — Why the project, what problem, prior attempts. Must make sense to someone who wasn't in the verbal discussions.
5. **Related Documents** — Links to test plans, functional specs, related system designs, prior iterations.
6. **Goals** — Impact on users/team/company, not implementation details. Bad: "Add Kubernetes." Good: "Minimize outages related to deploying new app versions."
7. **Non-goals** — Explicitly states what is out of scope to prevent mistaken assumptions.
8. **Scenarios** — Concrete walkthroughs of how the completed system works in practice.
9. **Diagrams** — "Tremendously valuable" because reviewers lack the author's mental picture. Recommends Excalidraw, draw.io, Mermaid, D2, Graphviz. Warns against whiteboard photos.
10. **Glossary** — Defines internal tools and systems for newer team members and external readers.
11. **Constraints** — Budget, client, infrastructure, or dependency limitations that shape design choices.
12. **Service Level Objectives (SLOs)** — Measurable, concrete goals for uptime/availability, latency, and scale. Example: 50th percentile latency <=200ms for user-facing HTTP requests.
13. **Monitoring / Alerting** — How SLOs will be measured in production, progressing from manual testing to automated alerts.
14. **Timeline** — Milestone-based schedule producing useful stakeholder artifacts. Recommends starting with dummy data. References Joel Spolsky's "Painless Software Schedules."
15. **Interfaces** — Sketches of UI, API/CLI semantics, or file formats.
16. **Dependencies / Infrastructure** — Language, hardware/service, and data persistence decisions.
17. **Security** — Threats considered, attack surface, trust boundaries. Even documenting why threats are unlikely is helpful.
18. **Privacy** — Sensitive data handled, retention periods, access controls, encryption at rest and in transit.
19. **Legal Considerations** — Relevant in regulated domains; also covers open-source license choices.
20. **Logging** — Critical events to log, log levels, storage location, retention, access, sensitive data exclusion.
21. **Open Issues** — Appendix documenting unresolved flaws, trade-offs, information gaps. Each entry: problem, possible solutions, immediate next step.
22. **Resolved Issues** — When an open issue is decided, the entry moves here with decision summary + full original discussion retained.
23. **Alternatives Considered** — Proactively addresses "why didn't you do X?" Explains rejected options, especially appealing ones or those requiring extensive research.

## Key Quotes

- "A good design doc can save you years of development time."
- The guiding question: "what's the penalty for being wrong?"
- "Once you've completed your design doc, the next step is to share it with your team and gather feedback."
- On diagrams: "reviewers don't have your mental picture of the system"

## Example Design Doc

Lynch created a complete example design doc for a real web app called "Little Moments" — written before any code, adhered to during implementation. Available on Codeberg.

## Related Resources

- Example design doc: Little Moments (Codeberg)
- Related article: How to Get Meaningful Feedback on Your Design Document
- Book: Refactoring English: Effective Writing for Software Developers (early access)
