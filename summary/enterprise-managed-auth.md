---
url: https://claude.com/blog/enterprise-managed-auth
title: Centrally manage authorization for MCP connectors
author: Anthropic (uncredited)
date_fetched: 2026-06-21
date_published: 2026-06-18
topics:
  - security-and-sandboxing
---

# Centrally manage authorization for MCP connectors

Source: https://claude.com/blog/enterprise-managed-auth
Published: June 18, 2026
Categories: Enterprise AI, Product announcements
Products: Claude Enterprise, Claude apps
Reading time: 5 minutes

## Core Announcement

Admins can now provision MCP connectors organization-wide through their identity provider, starting with Okta. Users get connector access automatically on first login, with authorization configured centrally.

**The problem it solves:** Previously, enabling connectors required two steps — an admin enabled the connector, then every individual user had to authorize it themselves. This new feature eliminates that second step for end users.

**The result:** "zero-touch connector setup for the end user."

## Technical Details

- Built on the **Enterprise-Managed Authorization extension** to the Model Context Protocol — an open standard so any connector can support it, including custom connectors.
- Access stays consistent across Claude chat, Claude Code, and Cowork.
- For admins: provision once, scope by group, manage revocation through the IdP.
- Admins can shorten access token lifetimes since IdP checks are frictionless.
- Admins can require that a connector only connects through the IdP, keeping work and personal use separated.

## Ecosystem at Launch

**Identity providers:** Okta at launch; more coming soon.

**MCP providers supporting at launch:** Asana, Atlassian, Canva, Figma, Granola, Linear, and Supabase. Slack coming soon.

**Claude customers rolling it out:** Hubspot, Ramp, and Webflow.

## Key Quotes

- **Arnab Bose, Asana CPO:** "Enterprise-managed auth is a foundational milestone in realizing Asana's vision as the operating system for human-agent teams."

- **Brendan Haire, Atlassian VP of Engineering, Rovo and AI:** Enterprise-managed auth "makes Atlassian Rovo MCP easier for Claude Enterprise customers to adopt at scale, giving employees a simple way to connect..."

- **Aaron Parecki, Okta Director of Identity Standards:** "The momentum around MCP is incredible, but as we move toward an interconnected AI workforce, security can't be an afterthought."

- **Cameron Leavenworth, Ramp Staff IT Engineer, AI:** "Now they log in to Claude on day one already connected — 2,000 employees, provisioned through Okta, zero extra steps."

- **Tom Moor, Linear Head of Engineering:** "Logging in once and automatically having all your MCP connectors automatically set up is pretty magical."

- **Andrew Meinert, Hubspot Director, System Operations & AI:** "Enterprise-managed auth is the security and user experience that we've been looking for with MCP connections."

- **Bil Harmer, Supabase CISO:** "The only way to use Supabase through Claude was to be an org owner or hand out Personal Access Tokens..."

- **Rod García, Slack VP of Engineering:** "Slack is the place where humans and agents are working side by side, in the same conversation..."

- **Reed Shackelford, Webflow Senior Manager, Enterprise AI Operations:** "Enterprise-managed auth turned AI into something people use instead of request..."

- **Chris Pedregal, Granola CEO & co-founder:** "It's great to see Anthropic and Okta make it easier for enterprises to connect to MCP servers securely..."

- **Devdatta Akhawe, Figma VP of Engineering:** "The Figma MCP brings the power of code and canvas together so teams can move faster..."

- **Anwar Haneef, Canva GM & Head of Ecosystem:** "Enterprise-managed auth with Okta makes it clear and simple for enterprises to manage AI access with a system they already trust..."

## Availability

Available in **beta** for customers on Claude Team and Enterprise plans.
