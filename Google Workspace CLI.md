# Google Workspace CLI

One CLI for all of Google Workspace -- Drive, Gmail, Calendar, Sheets, Docs, Chat, Admin, and more. Built in Rust. The key trick: it reads Google's Discovery Service at runtime and builds its command surface dynamically, so new API endpoints show up automatically without tool updates.

---

## Key Quotes

> "One CLI for all of Google Workspace -- built for humans and AI agents."

## Key Themes

#tool #cli #google-workspace #agent-tooling

This is architecturally interesting. Most CLI tools hardcode their command surface. This one generates it from Google's own API descriptions at runtime. That means when Google adds a new Calendar feature or Sheets endpoint, the CLI picks it up without a release. It's a meta-tool that wraps an API discovery protocol.

The 100+ agent skills shipped with the tool are designed for LLMs to use autonomously. Hand-crafted helper commands (prefixed with `+`) handle common workflows: `+send` for email, `+agenda` for calendar, `+standup-report` for generating reports. The timezone-aware calendar features are the kind of detail that separates usable tools from toy demos.

26.2k stars and 44 releases suggest serious traction. Written in Rust (98.8% of codebase) for performance and reliability.

## Critical Analysis

The "not an officially supported Google product" disclaimer is important. Google has a history of killing internal projects; an external one has even less protection. But the dynamic Discovery Service architecture means it's not tightly coupled to any specific API version -- it should degrade gracefully as long as Google's discovery endpoint exists.

The structured exit codes (0-5 for success, API error, auth error, validation error, discovery error, internal error) show production thinking. This is a tool designed for scripting and CI pipelines, not just interactive use.

For agent workflows specifically, this competes with MCP-based Google integrations. The CLI approach (invoke via terminal) is simpler to deploy but lacks the persistent connection and structured input/output of MCP. Both approaches work; the choice depends on your agent framework.

See also: [[Google Workspace CLI Skills]] — the structured skill catalog (services, helpers, recipes, personas)

---
*Sources: [[raw/googleworkspacecli]]*
*Last updated: 2026-05-15*
