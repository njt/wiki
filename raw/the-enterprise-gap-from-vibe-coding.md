---
url: https://wolkensteiner.substack.com/p/the-enterprise-gap-from-vibe-coding
date_fetched: 2026-08-06
---

# The Enterprise Gap: From "Vibe Coding" to Executable Architecture

### A non-technical colleague built an app in a single day. Then we popped the hood.

A non-technical colleague built a working app in a single day using an AI app builder.

On the screen, it looked remarkable. Sleek UI, clean inputs, and it actually solved a real internal problem. To leadership, it looked like magic. The question in every hallway was some version of: *why does engineering take three months to build what someone built between coffee breaks?*

Then we popped the hood.

Underneath the polished frontend was a system held together by duct tape and hope:

- **Half the data lived in a Supabase instance.**The other half was hardcoded as mock JSON directly inside frontend components.
- **Zero backend logic.**Every database read and write was called straight from the client.
- **Zero authentication.**No SSO, no roles, no audit logging.

## Patching the Frankenstein’s Monster

As the architect brought in to make it production-ready, I spent the next phase retrofitting reality onto a prototype that was never built to hold it.

We stood up a proper backend, wired in real authentication, built actual data models, and secured the endpoints. It’s deployed now. It works.

But it isn’t how anyone would design this system if you started from the actual requirements. It’s a prototype wearing compliance bandages.

That’s the gap between an AI demo and a system you can run a business on.

Tools like Lovable, Cursor, or Claude Code optimize for **speed to pixels**. They give you instant visual feedback by taking every shortcut available to get there. But nobody evaluates production software on how fast it renders a form. They evaluate it on whether it’s repeatable, maintainable, secure, and still holds together a year later.


Prompt and hope doesn’t scale past a demo. You can’t build a business on a model guessing its way through backend architecture.

## Decoupling Planning from Execution

The industry’s answer to this right now is to give the AI more agency inside the loop—the theory being that a smarter prompt or a longer agentic loop eventually figures out compliance on its own.

It won’t.

An LLM generating code in an unconstrained loop will always take the path of least resistance to make the UI work or pass a local test, even if that means dropping mock data into a frontend file or quietly stepping around an auth gate.

The only way out is to separate planning from execution, all the way:

- **The plan is a template:**A plain TOML file, not some model’s inscrutable inner monologue. An LLM can write one from a prompt. So can you, by hand, the same way you’d write a Makefile target or a CI config.
- **Whoever wrote it steps away:**The plan, however it was authored, gets checked against policy before anything executes.
- **Execution is deterministic:**A dedicated runner carries out the plan inside strict guardrails, with no model improvising its way through a runtime error.

## Introducing `rigorix-oss`

I didn’t write this to vent about a bad prototype. I built something instead.

I’ve open-sourced `rigorix-oss`—a deterministic execution engine for bounded autonomy (dual-licensed under MIT and Apache-2.0). It ships as a CLI, a GitHub Action, and an MCP integration, all running on the same core engine.

It takes plans—plain TOML templates, whether an LLM wrote them or you did—and runs them inside deterministic policy boundaries. It doesn’t matter what produced the intent: the resulting code can’t violate your database rules, bypass your auth layer, or slip in a phantom mock file.


The guarantee is simple: Quality issues get a shot at self-correcting through a validate-fix loop—run the tests, see what breaks, fix it, repeat. Policy violations don’t—if a step crosses a permission boundary, touches a file it shouldn’t, or blows through a budget, execution halts cold. It doesn’t patch around it or improvise a workaround. It stops, and it tells you exactly what crossed the line.

What we need isn’t a better prompt or a smarter chat window. It’s infrastructure that actually enforces what we intended to build.

`rigorix-oss` is that infrastructure. It’s on GitHub now, built to drop straight into your pipeline.

The colleague’s app taught me the difference between a demo and a deployed system. `rigorix-oss` is the difference between “prompt and hope” and something you can actually run in production.

*(Inspect the code or contribute on GitHub)*
