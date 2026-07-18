---
url: https://blog.postman.com/ai-ready-apis-agentic-ai-aws-competency/
title: "AI is ready. Your APIs probably aren't."
author: Matt Gray
date_fetched: 2026-07-18
date_published: 2026-07-09
---

# AI is ready. Your APIs probably aren't.

**Author:** Matt Gray
**Published:** July 9, 2026
**Publication:** Postman Blog

---

Postman has achieved the AWS AI Competency in Agentic AI Tools. Matt Gray argues this says as much about readers' APIs as it does about Postman itself.

Gray reflects on conversations with enterprise customers and partners about AI, observing a consistent pattern: "The model isn't the hard part. The challenge is everything that comes after the prompt."

Organizations can access powerful foundation models, but problems arise when those models need to interact with real systems and production data. Gray cites a June 2025 Gartner prediction that "over 40% of agentic AI projects will be canceled by the end of 2027," citing escalating costs, unclear business value, and inadequate risk controls. Gartner also estimated that of thousands of vendors claiming agentic AI capabilities, only around 130 represent genuinely agentic products.

Gray states: "The model selection problem is largely solved. The infrastructure problem is not."

He notes that getting APIs ready for how agents discover, understand, test, and call them reliably at scale is where most AI initiatives encounter friction, with governance and compliance adding further complexity for regulated enterprises.

## What the AWS AI Competency Actually Validates

Gray explains this isn't a self-certification. AWS created the Agentic AI Tools category to recognize platforms that help teams build, deploy, and manage agentic AI systems while meeting enterprise requirements for governance, security, and compliance. Partners undergo technical validation and must demonstrate successful customer implementations.

The designation covers platforms supporting the full development lifecycle — from design and specification through testing, governance, and production operations — across varying regulatory environments. Gray stresses this "reflects the capabilities enterprises need to move AI from experimentation to production."

## AI Agents Run on APIs, and API Quality Determines Agent Reliability

Gray writes that most enterprises are questioning whether their infrastructure is ready for AI. AI agents depend on APIs, but there's a specificity problem: a human developer can infer intent from ambiguous documentation, but "an agent can't." If an OpenAPI specification omits authentication scopes, misrepresents response schemas, or fails to document error codes, the agent fails.

He argues that as agents become a distinct buyer class, "the quality, discoverability, and governance of an organization's APIs will directly determine whether that organization can take part in an agent-driven economy."

Postman helps teams design well-specified APIs, enforce consistency through governance rules powered by Spectral, and validate behavior with automated test suites. The Postman AI Agent Builder lets teams turn any existing collection into a production-ready MCP server.

## Building Agent-Ready APIs at Scale: The PayPal Example

PayPal serves as a case study. Since publishing their public Postman workspace, their collections have accumulated more than 100,000 forks. Business impact includes time to first API call dropping from 60 minutes to 1 minute, and testing cycles that used to take hours now finishing in minutes. Engineers on the Postman Enterprise plan save roughly one hour per week.

PayPal published its MCP server as a Postman Collection, presented at POST/CON 2025, covering payments, invoices, disputes, shipment tracking, subscriptions, and more. The collection uses Postman's built-in OAuth support.

A quote from Mark Lummus, Head of Product, Developer Tools at PayPal:

"Postman has become a front door to PayPal, and increasingly the developer walking through it is an AI agent."

He adds that they built collections and Flows so "humans and agents read them the same way," turning discovery into a first API call in under a minute.

## Bringing Agentic AI to AWS Builders

Gray outlines three integrations as part of Postman's collaboration with AWS:

**Postman MCP Server in Kiro:** Kiro is AWS's spec-driven agentic IDE built on Code OSS. It produces three structured artifacts before writing code: `requirements.md`, `design.md`, and `tasks.md`. Through the Postman MCP server, developers can access Postman workspaces, collections, environments, and tests from within Kiro.

**Postman Agent Mode on Amazon Bedrock:** Agent Mode is powered by Anthropic's Claude models through Amazon Bedrock. Bedrock offers In-Region and Geo Cross-Region inference profiles for different compliance requirements. Agent Mode helps generate test suites, troubleshoot failing requests, and synchronize collections with Git repositories.

**API Catalog Integration with Amazon API Gateway:** Teams can import OpenAPI specs from API Gateway into the Postman API Catalog, view deployment status and CloudWatch metrics, and export specifications back to API Gateway — creating "a continuous loop between design, testing, and deployment."

## Moving AI from Pilot to Production

Gray confirms the Postman MCP server is available in Kiro, Agent Mode on Amazon Bedrock is live, and the API Catalog integration with Amazon API Gateway is available now.

He concludes by returning to Gartner's 40% cancellation prediction, calling it "a warning about the gap between experimentation and production-readiness." The foundations underneath AI systems — APIs, governance, testing, and operations — decide whether those systems perform reliably at scale. He describes this as "an engineering problem," not a model problem.

Gray ends: "As organizations move from AI experimentation to enterprise-scale deployment, preparing APIs for agents will become just as important as selecting the model itself."

## Learn More

- Postman's AWS AI Competency announcement on BusinessWire
- Postman on AWS Marketplace
- PayPal customer story
- Postman Partners program

---

**Tags:** AI, API, API-First, Company News, Partners
