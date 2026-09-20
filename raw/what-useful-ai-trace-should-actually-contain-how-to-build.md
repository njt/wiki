---
url: https://www.telerik.com/blogs/what-useful-ai-trace-should-actually-contain-how-to-build
date_fetched: 2026-09-20
---

Summarize with AI:

Learn how to produce the six trace elements using the Progress AI Observability Platform with the architecture choices and tradeoffs you’d want in production.

If your AI trace doesn’t explain decisions, it’s just noise. A span that says your LLM call succeeded in 1.3 seconds confirms the infrastructure worked, but it says nothing about what the model received, what it returned, what tools it called, what that cost or whether the output was correct. Useful AI traces contain the prompt sent, the completion returned, every tool invocation with its inputs and outputs, token counts, estimated cost and a quality signal.

If you’re new to the topic, start with The Six Questions Every AI Trace Should Be Able to Answer for the conceptual foundation. This post picks up where that one leaves off.

This post is an implementation walk-through: it shows how to produce those six trace elements using the Progress AI Observability Platform Python SDK (`progress-observability`), with the architecture choices and tradeoffs you’d want in production. The platform also supports .NET and JavaScript/TypeScript—see telerik.com/ai-observability-platform—free to start, 5-minute setup.

Picture this: your dashboard shows HTTP 200, p95 latency looks normal, error rate is zero. Your agent ran 4,000 times this morning. Then a customer emails saying it gave them completely wrong product information. Where do you even start?

When something goes wrong with an AI agent, you need to answer six questions fast. The previous post covers what those questions are and why standard observability can’t address them. Briefly: what context did the agent have, what did tools return, what did the model receive, what did it produce, what did it cost and was the output good.

Standard OpenTelemetry answers none of these. It captures span timing, parent-child relationships and error codes—necessary, but not sufficient for debugging agent behavior.

The gap is significant. A request can return HTTP 200, produce no errors and still have:

Without AI-specific context baked into the trace, every one of these failures is invisible until a user reports it.


This post covers trace design—what to put into a trace and how to instrument it. For a deep dive on detecting agents that silently return wrong answers despite showing HTTP 200, see When Status OK Is Still a Failure. For tracking and optimizing token spend, see AI Cost Visibility Before the Invoice.

The Progress Observability Platform sits between your application code and the observability backend. It uses standard OpenTelemetry plumbing but enriches spans at the collector level with AI-specific data: cost calculations, token attribution and tag enrichment.

```
flowchart LR  
App["Your App\n(Python + SDK)"] -->|"OTel spans\n(OTLP/gRPC)"| Collector["Progress Collector\n(Auth + Cost enrichment)"]  
Collector -->|enriched spans| Backend["Platform Backend\n(Storage + Processing)"]  
Backend --> Dashboard["Platform Dashboard\n(Observations, Costs, Scores)"]
```
The SDK handles auto-instrumentation. Once initialized, it patches supported libraries—OpenAI, LangChain, Anthropic, LlamaIndex and others—so every LLM call and framework operation creates a span automatically. You don’t touch your LLM client code.

Cost calculation happens in the collector, not in your application. This matters more than it sounds: model providers change pricing with a blog post and a short notice period. If cost logic lives in your app, that’s a code change, a PR, a deploy and potentially a rollout across multiple services. If it lives in the collector, it’s a configuration update in one place.

The AI observability community has a specific name for the gap between having many spans and having useful ones: the **trace noise problem**.

Standard OpenTelemetry auto-instrumentation was designed to instrument everything. In practice, that means LLM calls appear alongside HTTP client requests, authentication flows, database queries and framework internals. That’s all technically valid telemetry, but none of it is useful when you need to understand *why* an agent produced a wrong answer.

This shows up repeatedly across AI observability tooling: a single agent trace can contain the real LangChain or LLM spans right next to Microsoft Graph authentication calls, generic HTTP events and other infrastructure telemetry because OTel does not distinguish AI-relevant spans from infrastructure spans by default. The usual fix is some combination of instrumentation-scope blocklists, span filters or name-based selection, but that filtering surface tends to expand as more libraries get added. Across the market, vendors have added span kind filters, annotation-based filtering, name-based span selection and similar narrowing controls precisely because isolating relevant spans from a noisy store is an ongoing operational challenge.

**Trace volume** is easy to produce. **Trace hygiene**—capturing the right signals, in the right structure, with the right context—requires deliberate design.

As the companion awareness piece The Six Questions Every AI Trace Should Be Able to Answer describes, trace hygiene rests on five implementation principles: **Scope** (instrument AI operations, not all infrastructure), **Structure** (spans that reflect agent logic), **Content** (prompt and completion text captured), **Attribution** (tags for filtering) and **Quality** (evaluation scores tied to traces). The four steps below implement those principles in practice—each one closes a gap that a timing-only OTel span leaves open.

```
pip install progress-observability
```
Initialize once at the very start of your process, before importing LLM libraries. The SDK patches libraries at initialization time. If you import `openai` first, those calls won’t be traced.

```
import os  
from progress.observability import Observability  
  
Observability.instrument(  
  app_name="support-agent",  
  api_key=os.environ["OBSERVABILITY_API_KEY"],  
  trace_content=True, # captures actual prompt and   completion text  
)
```
`trace_content=True` is the most important configuration choice here. Without it, you get timing spans with no content—which is fine for production environments with PII concerns, but makes debugging nearly impossible. Use environment variables to toggle this per deployment.

Tags let you filter observations in the platform by environment, team, experiment or any dimension that matters. Set global tags at initialization:

```
Observability.instrument(  
  app_name="support-agent",  
  api_key=os.environ["OBSERVABILITY_API_KEY"],  
  trace_content=True,  
  additional_tags=[  
    "environment:staging",  
    "team:support",  
    "release:2.4.1",  
  ],  
)
```
For tags that vary per request—tenant ID, experiment variant—use `propagate_attributes` to scope them to a specific block of code:

```
from progress.observability import propagate_attributes  
  
async  def handle_request(user_message: str, tenant_id: str):  
  with propagate_attributes(tags=[f"tenant:{tenant_id}", "experiment-prompt-v3"]):  
    # Every span created inside this block inherits these tags
    result =  await run_agent(user_message)  
  return result
```
Tags from `propagate_attributes` merge with global tags and are deduplicated automatically. Nesting is supported—tags accumulate from outer to inner context managers.

The practical payoff: when a customer reports an issue, filter by `tenant:acme-corp` combined with `release:2.4.1` and you’ve narrowed a million traces down to the 20that matter.

Auto-instrumentation captures LLM calls, but it doesn’t know your agent’s structure. Without decorators, the trace shows a flat list of LLM calls with no context about which workflow triggered them or why a particular tool was invoked.

The SDK provides four decorator types:

| Decorator | Use for | 
|---|---|
| `@agent` | The AI agent entry point | 
| `@workflow` | Multi-step orchestration that coordinates tasks | 
| `@tool` | Functions the agent can invoke | 
| `@task` | Discrete units of work: retrieval, preprocessing, API calls | 

Here’s a complete instrumented agent:

```
from progress.observability import agent, workflow, tool, taskfrom openai import AsyncOpenAI  
import os  
  
client = AsyncOpenAI()  
  
@task(name="retrieve-context", attributes={"source": "vector-db"})  
async  def retrieve_relevant_docs(query: str) ->  list[str]:  
  """Retrieves relevant documents — span captures latency and custom attributes."""
  results =  await vector_db.search(query, top_k=5)  
  return [r.text for r in results]  
  
@tool(name="fetch-product-info")  
async  def fetch_product(product_id: str) ->  dict:  
  """Tool span captures the product_id input and the dict return value."""  
  return  await product_service.get(product_id)  
  
@workflow(name="answer-product-question", version=2)  
async  def answer_question(user_query: str, product_id: str) ->  str:
  context =  await retrieve_relevant_docs(user_query)
  product =  await fetch_product(product_id)
  context_text =  "\n".join(context)
  messages = [  
    {"role": "system", "content": "You are a helpful product support agent."},  
    {  
      "role": "user",  
      "content": f"Context:\n{context_text}\n\nProduct: {product}\n\nQuestion: {user_query}",  
    },  
  ]  
  
  # Auto-instrumented: span captures model, messages, completion, token counts
  response =  await client.chat.completions.create(  
    model="gpt-4.1-mini",  
    messages=messages,  
  )  
  return response.choices[0].message.content  
  
@agent(name="product-support-agent")  
async  def handle_request(user_message: str, tenant_id: str) ->  str:  
  with propagate_attributes(tags=[f"tenant:{tenant_id}"]):
    product_id = extract_product_id(user_message)  
    return  await answer_question(user_query=user_message, product_id=product_id)
```
In the platform, this renders as:

```
agent: product-support-agent  
└── workflow: answer-product-question (v2)  
  ├── task: retrieve-context [source: vector-db]  
  ├── tool: fetch-product-info [input: product_id, output: {...}]  
  └── llm: gpt-4.1-mini [tokens: 847, cost: $0.00042]
```
That single view replaced hours of log grepping. Without the decorators, this same run would show a flat list of anonymous `llm: gpt-4.1-mini` spans with no structural context—no way to tell which call is the retrieval prompt and which is the final answer generation. You’d count spans and guess.

Always flush pending telemetry before the process exits. Without this, buffered spans are lost—especially in scripts, short-lived containers and serverless functions.

```
import asyncio  
from progress.observability import Observability  
  
async  def main():  
  try:  
    await handle_request(  
      user_message="What's the return policy for order #12345?",  
      tenant_id="acme-corp",  
    )  
  finally:  
    Observability.shutdown()  
  
asyncio.run(main())
```
In long-running services, hook `Observability.shutdown()` into your SIGTERM handler or application lifecycle teardown, not into individual request handlers.

This is the most common mistake, and the most disorienting to debug. Your agent runs, returns results, no exceptions anywhere. And zero traces appear in the platform. Not errors, not partial traces: silence.

The SDK patches libraries at the time `Observability.instrument()` is called. Any library imported before that call won’t be patched.

```
# Wrong — openai is imported before instrumentation  
import openai  
from progress.observability import Observability  
Observability.instrument(...)  
  
# Correct — instrumentation first  
from progress.observability import Observability  
Observability.instrument(...)  
import openai
```
Setting `trace_content=False` everywhere eliminates your ability to debug output quality issues. Use the `OBSERVABILITY_TRACE_CONTENT` environment variable to control this per deployment without changing code.

```
# .env.production  
OBSERVABILITY_TRACE_CONTENT=false  
  
# .env.staging  
OBSERVABILITY_TRACE_CONTENT=true
```
Auto-instrumentation alone gives you a flat trace of LLM calls. Decorators give you structure. Without them, a three-step agent looks like this in the platform:

```
llm: gpt-4.1-mini [tokens: 312]  
llm: gpt-4.1-mini [tokens: 847]  
llm: gpt-4.1-mini [tokens: 203]
```
No workflow context, no tool inputs, no parent-child relationships. You’re left counting spans and guessing which one is the retrieval call and which is the final answer.

Short-lived scripts that don’t call `Observability.shutdown()` frequently lose the last batch of spans. This makes traces appear truncated or missing in the platform. In AWS Lambda and other serverless runtimes, the environment can be frozen or terminated between invocations before the buffer flushes—which means you often lose the most recent trace completely, exactly when you’re trying to debug a one-off failure.

| Choice | When to use it | What you give up | 
|---|---|---|
| `trace_content=True` | Development, staging, debugging | Prompt and completion text in trace storage—PII exposure risk in production | 
| `trace_content=False` | Production with PII | Can’t inspect what the model received or returned | 
| Global tags only | Simple single-environment apps | No per-tenant or per-experiment filtering | 
| `propagate_attributes` | Multi-tenant or A/B testing | Context manager overhead in async code | 
| Full decorator coverage | Complex agent workflows | Some boilerplate per function | 
| Real-time evaluation tasks | Ongoing production quality monitoring | 2 units consumed per evaluated span | 

Starting fresh? Default to `trace_content=True` in staging, `propagate_attributes` for anything multi-tenant and full decorator coverage from day one. These are easy to set up upfront and painful to retrofit when you’re trying to debug something at 2 a.m.

**Clone the demo first:** NickIliev/support-agent-demo is a fully runnable version of the agent from this post. Add your API keys to `.env` and run python `support_agent.py`. All six trace elements will appear in your dashboard immediately. This is the fastest way to see a complete, well-instrumented trace before building your own.

**Instrument one agent:** `pip install progress-observability`, add `Observability.instrument()` with `trace_content=True`, apply the decorators from Step 3. You’ll have a working trace in under 15 minutes.

**Open the Observations view** and inspect the span tree. Filter by a tag, drill into a span, verify the exact prompt the model received.

**Set up one evaluation task** to start tracking output quality automatically.

**Want a guided path from zero?** Work through the Log Your First AI Trace quickstart *(coming soon)* for a structured first-trace experience in 10–15 minutes. In the meantime, the getting started guide covers the same ground—free to start, 5-minute setup.

**AI Observability Reading Path**

Nikolay Iliev is a senior technical support engineer and, as such, is a part of the Fiddler family. He joined the support team in 2016 and has been striving to deliver customer satisfaction ever since. Nick usually rests with a console game or a sci-fi book.
