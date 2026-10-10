---
url: https://www.uber.com/in/en/blog/designing-mcp-gateway/
title: "Designing MCP Gateway: Uber's MCP Management Platform"
author: "Alok Srivastava, Deepanshu Mehndiratta, Gaurav Gill, Prashik Sahare, Vaibhav Tayal, Shiven Tripathi, Uday Kiran Medisetty, Abhishek Bhatia"
date_fetched: 2026-10-10
date_published: 2026-10-10
topics:
  - mcp-and-tool-protocols
  - agent-architecture
---

Uber's MCP Gateway is a centralized platform serving all MCP interactions across the company — over 800 MCP servers and 5,000 tools — with two planes: an MCP Registry control plane (discovery, ownership, enablement) and a Proxy Gateway data plane (runtime execution, protocol translation).

The control plane is largely automated. AutoCrawler, a Cadence-powered workflow system, continuously scans Uber's IDL registry, parses Protobuf/Thrift definitions, uses an LLM to generate agent-friendly tool descriptions, translates schemas to JSON-RPC 2.0, and registers everything as "virtual MCP servers" disabled-by-default. Native MCP servers (built with Uber's MCPFx framework) are discovered via heartbeat metrics and `listTools` calls. Third-party servers like Jira are supported through token exchange.

A core design principle: **discovery doesn't imply exposure**. Every tool starts disabled; owning teams must review, enable, and approve every config-change diff, with rollback. The data plane provides built-in tool-level authorization via Uber's Access Control System (charter policies for humans, services, and agents), automatic PII redaction, and protocol translation executed through Uber's Muttley service-mesh sidecar — so downstream services need zero changes.

At scale, new problems emerged: MCP has no native cross-server search, and wiring hundreds of servers eats the model context limit. Uber's answers: **Omni MCP**, a single proxy with gradual discovery tools (`discover_server`, `discover_tools`, `get_tool_schema`, `invoke_tool`); **Response Projection**, a GraphQL-like pattern where the LLM injects required field paths and the gateway trims responses at runtime; and **Code Mode**, routed through Uber's `aifx` CLI so agents list, search, and call tools in a shell and write output to files for selective grepping — now the company default for MCP tool use in coding agents.

The conclusion: "the hardest part isn't the AI. It's building the connective tissue — the discovery, the security, the reliability" — and the core insight is that existing APIs are the fastest way to provide tools to an agent, so meet teams where they are rather than asking them to rewrite services for an agentic world.
