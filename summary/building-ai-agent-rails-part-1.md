---
url: https://catalinionescu.dev/ai-agent/building-ai-agent-part-1/
title: "Building an AI agent inside a 7-year old Rails application"
author: Catalin Ionescu
date_fetched: 2026-05-14
date_published: 2025-12-26
tags: ruby-on-rails, ruby, llm, ai-agent, tool-calling
topics:
  - agent-architecture
  - mcp-and-tool-protocols
---

# Building an AI agent inside a 7-year old Rails application

Catalin Ionescu, Director of Engineering at Mon Ami (US-based startup building SaaS for Aging and Disability Case Workers), describes integrating an AI agent into a mature, multi-tenant Rails monolith with strict data access controls.

## Context

- Multi-tenant Rails monolith
- Sensitive data with layered Pundit authorization
- Algolia search index
- Initially believed there was "no low-hanging fruit that would work for our product and business"

## The Insight

At SF Ruby, a talk on RubyLLM's tool/function-calling approach made it click: encode authorization logic into specific function calls, giving the LLM data access "without having to give it unrestricted access."

## RubyLLM Setup

```ruby
RubyLLM.configure do |config|
  config.openai_api_key = Rails.application.credentials.dig(:openai_api_key)
  config.anthropic_api_key = Rails.application.credentials.dig(:anthropic_api_key)
  config.use_new_acts_as = true
  config.request_timeout = 600
  config.max_retries = 3
end

Dir[Rails.root.join('app/tools/**/*.rb')].each { |f| require f }
```

## Tool Definition Pattern

RubyLLM provides a DSL for defining tools with descriptions and typed parameters. The SearchTool:
1. Queries Algolia's search index with the user's natural language input
2. Retrieves matching record IDs from search hits
3. Passes results through a Pundit policy scope to enforce tenant/authorization boundaries
4. Returns only permitted data as a hash back to the LLM

"no model should ever know" a given user's private data from a SaaS service.

## Conversation & UI Architecture

- Conversations are RubyLLM `Conversation` objects with `broadcasts_refreshes` enabled for Hotwire/Turbo real-time updates
- A remote form submits messages, which enqueue an Active Job that calls `conversation.ask message`
- A Stimulus controller scrolls to new messages as they appear

## Model Selection

- GPT-5: Large context, but felt sluggish with 3+ consecutive tool calls
- GPT-4: "Very prone to hallucinations — rushing to respond with made-up data instead of calling the necessary tools"
- GPT-4o: Best balance of speed and correctness thus far
- Anthropic/Gemini: Not yet tested; flagged as future evaluation work

## Key Takeaways (author's own)

- The entire agent took 2-3 days of Claude-assisted development ("AIs building AIs")
- Surprised by the low complexity — the tool service object is essentially "an API controller action — pass inputs and get a JSON back"
- Compared ActiveAgent (which moves prompts to view files) but found it unsuitable since it "had no built-in support for defining tools or having long-running conversations"
- The same workflow pattern was later reused for internal publishing tooling
