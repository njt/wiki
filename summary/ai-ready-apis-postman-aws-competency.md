---
url: https://blog.postman.com/ai-ready-apis-agentic-ai-aws-competency/
title: "AI is ready. Your APIs probably aren't."
author: Matt Gray
date_fetched: 2026-07-18
date_published: 2026-07-09
topics:
  - mcp-and-tool-protocols
---

Matt Gray announces Postman's achievement of the AWS AI Competency in Agentic AI Tools and uses the milestone to argue that API readiness — not model selection — is the real bottleneck for enterprise AI adoption.

The core thesis: the model problem is largely solved, but the infrastructure problem is not. AI agents depend on APIs to interact with real systems and production data, yet most organizations' APIs aren't designed for agentic consumption. A human developer can infer intent from ambiguous documentation; an agent cannot. If an OpenAPI spec omits authentication scopes, misrepresents response schemas, or fails to document error codes, the agent simply fails. Gray cites Gartner's June 2025 prediction that over 40% of agentic AI projects will be canceled by end of 2027 due to escalating costs, unclear business value, and inadequate risk controls.

The AWS competency is not self-certification — it requires technical validation and demonstrated customer implementations. It recognizes platforms that support the full agentic development lifecycle (design, testing, governance, production operations) across varying regulatory environments.

PayPal serves as the marquee case study. Their public Postman workspace accumulated over 100,000 forks, with time-to-first-call dropping from 60 minutes to 1 minute. Their MCP server, published as a Postman Collection covering payments, invoices, disputes, and more, is designed so "humans and agents read them the same way."

Three AWS integrations are detailed: the Postman MCP Server in Kiro (AWS's spec-driven agentic IDE), Agent Mode on Amazon Bedrock (powered by Claude, supporting in-region and cross-region inference), and API Catalog integration with Amazon API Gateway for bidirectional spec sync. Gray frames these as building blocks for closing the gap between AI experimentation and production readiness, concluding that preparing APIs for agents is an engineering problem — and one as important as model selection itself.
