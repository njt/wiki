---
url: https://www.dbreunig.com/2026/04/14/cybersecurity-is-proof-of-work-now.html
title: "Cybersecurity Looks Like Proof of Work Now"
author: David Breunig
date_fetched: 2026-05-14
date_published: 2026-04-14
topics:
  - security-and-sandboxing
---

# Cybersecurity Looks Like Proof of Work Now

**Author:** David Breunig
**Date:** April 14, 2026
**Categories:** AI, Development, Security, Mythos

Anthropic recently released Mythos, a large language model demonstrating exceptional capabilities in computer security tasks. Rather than releasing it publicly, the company granted access exclusively to critical software makers, allowing them time to strengthen their defenses.

The author initially approached the announcement with measured skepticism, noting that finding exploits represents "a clearly defined, verifiable search problem" well-suited to computational intensity rather than genuine innovation.

The AI Security Institute released its third-party analysis, largely validating Anthropic's claims. Their evaluation centered on "The Last Ones," a 32-step corporate network attack simulation requiring approximately 20 human hours to complete. Mythos succeeded in 3 of 10 attempts using 100 million tokens per try, while other frontier models showed no completion despite similar budgets.

This outcome reveals a fundamental shift in security economics: defending systems requires spending more computational resources discovering vulnerabilities than attackers invest in exploiting them. The author compares this to cryptocurrency's proof-of-work model, where success correlates directly with raw computational expenditure rather than clever engineering.

The analysis identifies three significant implications:

**Open source software gains renewed importance.** If security depends on token spending, widely-used open source projects benefit from community resources that individual implementations cannot match. This contradicts recent arguments from figures like Andrej Karpathy advocating for replacing dependencies with AI-generated code.

**Software development will likely adopt a three-phase model:**

1. Development — rapid feature implementation guided by human judgment
2. Review — refactoring and documentation applying established practices
3. Hardening — autonomous vulnerability discovery continuing until budget exhaustion

Human input constrains the first phase while financial resources limit the third, creating distinct workflow stages. Previously, security audits were sporadic and inconsistent; continuous application within defined budgets now becomes feasible.

The economic reality remains stark: code remains inexpensive unless security requirements apply. Even as inference costs decline, organizations must outspend potential attackers to maintain advantage. The expense reflects market value of discovered exploits rather than absolute computational costs.
