---
url: https://www.infoq.com/articles/platform-engineering-playbook-production-llms/
date_fetched: 2026-10-08
---

### Key Takeaways

- Hallucination rate is a platform-controllable metric. By wrapping the model in an automated retry loop that catches formatting, grounding, and infrastructure errors on the fly, we cut production hallucination rate from fifteen percent to 1.5 percent without touching the foundation model.
- An intent-validation gate that returns "unclassified" instead of defaulting to the highest-scoring agent eliminates off-intent hallucinations and improves multi-agent routing.
- By storing prompts in a history-preserving registry rather than hardcoding them into our application files, we can update or roll back instructions instantly at runtime. Tweaking prompts without a clear audit trail is the easiest way to silently break your AI's behavior.
- Enforce tool authorization directly at the resource server with a strict default-deny policy; relying solely on API gateway checks, means any single bug in your orchestrator can instantly expose every connected tool.
- Standard APM tools cannot detect semantic degradation. You must instrument hallucination rates and per-team token costs at request ingress. Trying to retrofit this attribution onto a live production path later will cost you weeks of engineering time.

By the end of our first month running an LLM-driven inventory recommendation system in production, roughly fifteen percent of agent responses were hallucinations, confident-sounding outputs grounded in nothing real.

Six months later the rate was 1.5 percent and the lever that moved the number wasn't a better foundation model. It was the decision to stop treating the LLM stack as an application concern and start treating it as platform infrastructure.

The project behind this article is an inventory accuracy platform at a large retail organization. In retail supply chains, system inventory records drift from what is physically on shelves through misplacement, damage, theft, and miscounts; that drift degrades ordering, replenishment and product availability.

Our multi-agent LLM system analyzes discrepancy signals across millions of SKUs and billions of historical inventory records and generates corrective recommendations for our consumers who aim to maintain inventory accuracy at all times.

The platform layer described in this article was built in collaboration with a team of business, product, software engineers, data scientists, and data engineering experts, and now serves dozens of application teams across use cases like inventory audits, replenishment review, discrepancy triage, and more.

This article documents that shift, routing through foundation models such as Gemini and GPT under enterprise traffic. The patterns below are the ones I wish had been in place on day one.

Every primitive described here is implemented in a companion GitHub repository, built deliberately with no hidden abstractions. Where a framework would hide the mechanics behind a decorator or a built-in agent type, the repo keeps the moving parts visible so the patterns stay legible.

The code snippets in this article come directly from that repo. One deliberate difference from production exists. The repo runs a local model (llama3.1 via Ollama), connected through LiteLLM instead of a hosted foundation model, so the entire stack is runnable end-to-end at zero API cost.

## Why Individual LLM Apps Break at Scale

The failure pattern is consistent enough across teams to be predictable. The prototype works. The first production deployment looks fine for a week. Then the issues that don't show up in evaluations start showing up in dashboards.

In our case, three failures surfaced in the beta testing, each traced back to something the prototype never exercised. The first was LLM API throttling. Under real request volume, we started being rate-limited by the model provider. The second was context. When the system recommended an item that didn't fit, the root cause was almost always the data we fed the model, not the model itself, which made robust data-engineering pipelines central to the fix. The third was the big one, hallucinations, which stayed individually rare but became a steady rate once we were serving production traffic. We had engineered the platform so these failures could be tracked. That visibility is what turned each of them from a mystery into something we could fix.

Tracking them also made clear why they belonged in a platform layer rather than any one application. Three properties stood out:

- **Hallucinations as output, not as bugs.**
 A hallucinated response isn't a stack trace. It passes every traditional health check. The downstream service consumes it, takes action on it, and only much later does someone notice the suggested output doesn't exist.
- **Uncontrolled token spend.**
 Without per-request attribution, monthly bills arrive as a single line item from the model vendor. There's no way to tell which application, team or use case is driving cost, so optimization is guesswork.
- **No behavioral observability.**
 Latency dashboards show p99 looks fine. The error rate is near zero. Meanwhile the model is silently drifting into producing lower-quality outputs that look syntactically valid.

None of these are application bugs. Every team that ships an LLM feature hits the same set of issues independently and the first instinct is to fix it inside the application that produces N copies of the same scaffolding code across N teams. That's the part of work that should have been pushed down via a platform layer.

## The Case for a Shared LLM Platform

The argument for centralizing LLM concerns isn't theoretical. It maps almost exactly onto the case for centralizing authentication, logging, or service mesh in any large-scale system: once more than one application has the same cross-cutting requirement, the cost of duplication starts to dominate the cost of building shared infrastructure.

For LLMs the shared concerns are:

- Retry logic and failure classification
- Prompt registry and versioning
- Schema enforcement
- Token cost attribution by request
- Behavioral observability
- Tool and context routing
- Authentication and Authorization across tool servers

There is a second-order effect that matters more than the duplication. A shared platform forces a contract. Applications declare what they need (i.e., a model, a prompt version, a schema, a tool set, and an eval engine) and the platform handles how to deliver it. That contract is what lets a new application reach production in days instead of months, because the hard parts are already solved.

The decision point for building this platform isn't the first LLM application. It's the second. By the time you have ten teams each shipping LLM features, retrofitting a platform underneath them is long and difficult migration work.

## The Platform Architecture

Before walking through the primitives individually, Figure 1, below, shows how they compose. The repo implements the full request path and it mirrors the production environment.

**Figure 1. LLM platform architecture: gateway, coordinator, specialist agents, and per-team MCP servers. (Image created by Author)**

A single gateway is the only ingress. It authenticates the caller, issues a role-carrying JWT, exposes the agent-execution endpoint, serves per-user and per-team metrics, and hosts the admin-only prompt-rollback routes. Behind it, a root coordinator agent, built with Google’s Agent Development Kit (ADK), classifies intent on every turn and delegates to one or more of the specialist agents, each bound to its own registry-resolved prompt and its own structured-output schema. The specialists own the tool connections, which are multiple independent MCP servers, partitioned by team or usage, each of which verifies the caller's token and enforces the authorization policy itself before any tool runs.

*A note on the open-source tools underneath, because the layering is deliberate.*

Orchestration is handled by Google's Agent Development Kit (ADK): It provides the agent and sub-agent topology, the delegation call the coordinator uses to hand off to a specialist, and the callback hooks where the platform records cost and latency.

Each agent reaches its model through LiteLLM, which is the seam that makes the model provider swappable; the same agent code runs against a local model when running the application on a laptop and against hosted foundation models such as Gemini or GPT in production, changing only a model identifier.

In the companion repo that local model is served by Ollama, which is what lets the entire stack, gateway, agents, and tool servers run end to end on a laptop at zero API cost.

The tools themselves live behind the Model Context Protocol. ADK's MCP client connects the agents to the team servers, which are built with FastMCP on the other side.

The value of this separation is that no single framework owns the whole stack. The model layer, the tool layer, and the orchestration layer each sit behind a stable interface, so any one of them can be replaced by a stronger model, a different tool server, or even a different orchestrator without rewriting the platform primitives around them. That is the same "no hidden abstractions" discipline applied at the dependency level: Each tool does one job and the path between them stays visible.

## Structured Outputs: Enforcing Schema at the Platform Layer and Retrying on Failures

The single most cost-effective change we made was enforcing structured outputs at the platform boundary rather than at the application boundary.

The pattern is unremarkable in concept. Every LLM call declares a target schema (we use Pydantic models in Python) and the platform validates the response against that schema before returning it to the caller. If validation fails, the platform re-prompts with the validation error embedded as additional context, up to a configurable retry limit.

What changes when this pattern lives at the platform layer rather than in each application:

- One implementation handles all schema mismatches, so the strategy improves once for everyone.
- Downstream consumers can rely on the contract and if the call returns, the response shape is guaranteed.
- Schema violations become a metric the platform tracks, not a runtime exception buried in application logs.

Our output schema doesn't just format data; it serves as explicit instructions for the model, complete with dedicated tracking fields designed specifically to isolate and measure hallucinations. Figure 2, below, shows what a document specialist's output schema looks like in code using Pydantic.

```
```
```
from pydantic import BaseModel, Field
class DocsOutput(BaseModel):
    """Result of a grounded documentation lookup."""
    answer: str = Field(
        description="The answer, grounded entirely in the fetched documentation."
    )
    sources: list[DocSource] = Field(
        default_factory=list,
        description="Documentation sources cited; empty only when nothing was fetched.",
    )
    grounded: bool = Field(
        description="True if every claim is backed by the fetched docs; "
        "False if the docs did not cover the question.",
    )
    out_of_scope: bool = Field(
        default=False,
        description="True if the request fell outside documentation lookup.",
    )
```
**Figure 2. The  DocsOutput schema (app/model.py).**

Figure 3, below, shows what the model actually returns when it satisfies this schema. The answer is populated, sources lists the documentation the answer was drawn from and grounded is true precisely because those sources were fetched and used. Had the question fallen outside the available docs, the same structure would come back with grounded set to false and an empty sources list, a response the platform records as a hallucination signal rather than passing off as a confident answer.

```
```
```
{
   "answer": "FastAPI's APIRouter lets you group related path operations and mount them on the main app with app.include_router().",
   "library_name": "FastAPI",
   "version": "0.115",
   "sources": [
       {
           "library_id": "/tiangolo/fastapi",
           "topic": "routing"
       }
   ],
   "grounded": true,
   "out_of_scope": false
}
```
**Figure 3. Example  DocsOutput response.**

The grounded and `out_of_scope` flags cost nothing to generate but force the model to commit, in machine-readable form, to whether its answer is backed by retrieved context. A response that fails validation, has the wrong shape, missing fields, or unparseable output is the highest-precision hallucination signal a platform has.

The grounded flag only means something because of what happens before the model answers. Every specialist is connected to the platform's MCP servers, while tool results are the only evidence the model is allowed to use. The root agent's instruction states it plainly. Tool results are the source of truth for the answer.

The docs specialist is the clearest example. Its prompt gives it two documentation tools, used in sequence. One resolves a library name to an ID, while the other fetches the current official docs for that library and topic. The rules around those tools are strict. The agent must fetch docs before answering any question about an API or a parameter, never from training data, which may be outdated. Every claim, code example, and parameter name must come from the fetched documentation. If the docs do not cover the question, the agent must say so instead of guessing.

The schema then makes the model show its work. The "sources" field records one entry per lookup and "grounded" may be true only if every claim is backed by what the tools returned. The same pattern holds across specialists. When the execution agent reports a deployment, that claim comes from the MCP tool's response, not the model's memory. Grounding here is not a prompt trick, it is a pipeline. Tools fetch the evidence, the prompt restricts the model to it, the schema forces a declaration and validation turns a broken declaration into a counted, retryable failure.

Enforcing strict output schemas turns the vaguest AI failures into clear, typed data events giving you the exact signal needed to drive your retries, metrics, and evals using a single Pydantic model.

## Retry Orchestration with Validation

This is the layer where most of the fifteen percent to 1.5 percent hallucination reduction came from, so it warrants a more detailed discussion.

Blindly rerunning failed LLM calls is actively harmful. Repeating a prompt that hallucinated just yields a different hallucination, while naive retries during an outage create traffic storms that crush your infrastructure.

The pattern that worked for us:

### Classify the Failure

Is it a schema violation, a hallucination signal (low confidence, factual ungroundedness, or off-intent generation) or an infrastructure error (timeout, rate limit, or model unavailable)? The three classes receive three different responses; collapsing them into one generic retry is the mistake.

### Route by Class

Schema violations re-prompt with the error appended, so the model sees exactly what broke (for example, "grounded" returned as the string "yes" instead of a boolean, or a missing required "sources" field). Hallucination signals re-prompt with stronger grounding context (for example, the docs agent returning "grounded: false", or citing a source for a library it was never given). Infrastructure errors back off exponentially (for example, a 429 rate-limit or a 503 from the provider, the throttling we hit in our first release).

### Cap the Retry Budget per Request

If retries have no limit, one bad request can retry forever and run up a large bill, so every request needs a hard cap. There are a few ways to set that cap. The simplest is a fixed number of tries, three in our case, because a structured-output failure that hasn't fixed itself by the third try almost never fixes itself on the fourth. Another way is to limit the total tokens a request may spend, which controls the actual cost rather than the number of tries, since one retry on a long input can cost more than three on a short one. A third way is a time limit, which is useful when a fast answer matters more than a complete one. Any of these approaches work. What matters is that the limit exists and is enforced, so no single request can keep retrying without end.

Instead of swallowing malformed data and handing back broken objects, our platform wraps the agent's execution loop to catch validation errors, log them as hallucinations and trigger smart retries.

The agent wraps its own run loop, catching the "ValidationError" a badly structured output raises, recording each failed attempt as a hallucination, and retrying up to a configurable cap before re-raising, as shown in Figure 4.

```
```
```
from google.adk.agents import LlmAgent
class CustomLlmAgent(LlmAgent):
    def __init__(self, *args, max_validation_retries: int = 3, **kwargs):
        super().__init__(*args, **kwargs)
        self._max_validation_retries = max_validation_retries
    @property
    def max_validation_retries(self) -> int:
        """How many times structured-output validation is retried before failing."""
        return self._max_validation_retries
    async def _run_async_impl(self, ctx):
        """Run the agent with retry logic for validation errors.
        Both structured-output paths raise pydantic.ValidationError from
        inside ADK's _run_async_impl. This wrapper retries on validation
        errors up to max_validation_retries times, logging each failure
        before retrying.
        """
        for attempt in range(self.max_validation_retries):
           try:
               async for event in super()._run_async_impl(ctx):
                   yield event
               return  # Success
           except ValidationError as exc:
               is_last_attempt = attempt == self.max_validation_retries - 1
               logger.warning(
                   "[%s] VALIDATION ERROR (attempt %d/%d): output failed %s validation "
                   "(%d error(s)): %s%s",
                   self.name,
                   attempt + 1,
                   self.max_validation_retries,
                   getattr(self.output_schema, "__name__", "?"),
                   exc.error_count(),
                   exc.errors(include_url=False),
                   " -- retrying..." if not is_last_attempt else " -- max retries exceeded",
               )
               metrics.record_hallucination(get_current_user())
              
               if is_last_attempt:
                   raise  # Re-raise on last attempt
```
**Figure 4. The  CustomLlmAgent retry loop (app/agent.py).**

We log every single failed attempt to keep our hallucination metrics completely transparent and we throw a hard exception on the final failure so callers never receive a silently broken payload.

Tool-call failures use an entirely different mechanic. When an MCP tool errors, the raw failure is fed directly back into the model's context window so it can self-correct and reattempt the call. The repo delegates this loop to ADK's built-in plugin, registered at the application level as shown in Figure 5, below.

```
```
```
from google.adk.plugins import ReflectAndRetryToolPlugin
app = App(
    root_agent=root_agent,
    name="app",
    plugins=[
        ReflectAndRetryToolPlugin(max_retries=3),
    ],
)
```
**Figure 5.  ReflectAndRetryToolPlugin registration (app/agent.py).**

We offload tool-error recovery to the framework because tool failures are already structured for the model, but we strictly manage schema validation ourselves, using Pydantic, because hiding it would destroy our core hallucination telemetry.

We track two key metrics at this layer: recovery rate to ensure our retry loop actually works and amplification factor to guarantee it stays affordable.

## Prompt Versioning and Change Management

Treat prompt changes like code changes. A prompt is executable text. Modifying it changes the runtime behavior of the system in ways that are not visible in a diff. Two prompts that differ by a single sentence can have meaningfully different hallucination rates, latency profiles, and token costs. There is no compiler that warns you the new version is worse.

The minimum platform primitive: every prompt has an ID, a version, and is deployable independently of the application code that uses it. Applications reference prompts by ID, not by string literal. The platform stores prompt versions, tracks which version is live and exposes a rollback path that does not require redeploying the application.

Each prompt is registered under a name with an explicit version, its content, a creation timestamp, and tags. Registering a new version does not overwrite the old one; it adds to the history and moves the "live" pointer forward. Figure 6 shows the "docs_agent" prompt registered at two versions, the original v1.0.0 and the current v2.0.0 that supersedes it.

```
```
```
# Docs Agent Prompt -- v1.0.0 (registered first so v2.0.0 below stays active;
   # kept available as a rollback target).
   register_prompt(
       name="docs_agent",
       content="""You are a documentation lookup specialist. Your role is to:
               - Answer questions about software libraries and frameworks
               - Look up APIs, parameters, and usage patterns from their docs
               - Ground every answer in the documentation, not from memory
               - Say so explicitly when the docs do not cover a question
               - Keep answers concise and cite the library/version used""",
       version="1.0.0",
       tags=["documentation", "lookup", "specialist"]
   )
   # Docs Agent Prompt -- v2.0.0 (active)
   register_prompt(
       name="docs_agent",
       content="""You are a documentation lookup specialist. Your job is to answer questions about software libraries and frameworks using their current, official documentation -- never from memory.
           You have two tools, used in sequence:
           1. resolve-library-id -- pass the library or framework name (e.g. "next.js", "fastapi") to get its Context7-compatible library ID. If the name is ambiguous and several libraries match, choose the one that best fits the user's context and state which you picked.
           2. get-library-docs -- pass the resolved library ID, plus a topic when the question is about a specific area (e.g. "routing", "middleware", "authentication"), to retrieve the documentation.
           Rules:
           - ALWAYS resolve and fetch docs before answering any question about a specific API, parameter, version behavior, or usage pattern. Do not rely on training data for library details, which may be outdated.
           - Ground every claim in the fetched documentation. When you give a code example or name a parameter, it must come from the docs you retrieved, not from memory.
           - If the fetched docs do not cover the question, say so explicitly rather than guessing. Do not fill gaps with plausible-sounding but unverified API details.
           - If you cannot resolve the library at all, tell the user the library wasn't found and ask them to confirm the exact name.
           - Keep answers concise and cite which library/version the docs came from.
           You do not author new documentation, run code, or search the open web. If a request falls outside documentation lookup, say it's out of scope for this agent.
           Output format -- return a single JSON object matching the DocsOutput schema:
           - answer: your grounded answer, drawn only from the fetched docs.
           - library_name: the library the answer is about (e.g. "FastAPI").
           - version: the library version the docs reflect, or null if unknown.
           - sources: one entry per docs lookup, each with the resolved library_id and the topic (or null).
           - grounded: true only if every claim is backed by the fetched docs; false if the docs did not cover the question.
           - out_of_scope: true if the request was not a documentation lookup.
           Do not include any text outside the JSON object.""",
       version="2.0.0",
       tags=["documentation", "lookup", "context7", "grounded"]
   )
```
**Figure 6. The  docs_agent prompt at versions 1.0.0 and 2.0.0 (prompt_registry/prompts.py).**

Rollback is not deletion. It repoints which version is active and keeps the full history, so rolling forward is as fast as rolling back, as shown in Figure 7.

```
```
```
def rollback(self, name: str, version: str) -> str:
    """Repoint a prompt's active version to an existing earlier version.
    History is never deleted -- rollback only changes which version
    get_prompt(name) returns. Returns the version now active. Raises
    KeyError if the prompt or target version is unknown.
    """
    if name not in self._prompts:
        raise KeyError(f"Prompt '{name}' not found in registry")
    if version not in self._prompts[name]:
        raise KeyError(f"Version '{version}' not found for prompt '{name}'")
    self._latest_versions[name] = version
    return version
```
**Figure 7. The  rollback() method (prompt_registry/registry.py).**

The detail that makes rollback take effect without a redeploy is on the agent side. Each agent resolves its instruction from the registry at run time rather than capturing a fixed string when it's constructed, shown in Figure 8, below.

```
```
```
def _prompt_provider(name: str):
    # Resolve at run time (ADK InstructionProvider) rather than capturing a
    # fixed string at construction, so a prompt rollback via the registry
    # takes effect on the next request without rebuilding the agents.
    return lambda _ctx: get_prompt(name)
```
**Figure 8. The  _prompt_provider runtime instruction provider (app/agent.py).**

Because of that lookup, an operator hitting the rollback endpoint changes the behavior of the next request, without deploying, restarting, or rebuilding. Rollback is an admin-only gateway route, so the same RBAC layer that protects tools also controls who can change a live prompt.

One honest limitation, where I would extend the repo next is to address the issue of the registry keeping prompts in memory. That is fine for demonstration and local development, where you want to read the code and run it without external dependencies, but it is not suitable for production, where prompts must survive restarts and stay consistent across instances. The production step is to put a database behind the same interface, with a separate version binding per environment (i.e., dev, staging, and prod). The calling code does not change; only the storage layer behind it does.

## Intent Validation as a Routing Gate

This was the most novel primitive in our setup. Intent validation answers one question before the expensive foundation model call. Does this request match something the platform knows how to serve and if so, which specialist should handle it? In production we ran a small, trained classifier in front of the foundation model on every request, the kind you might build by fine-tuning a transformer through Hugging Face or training a lightweight model with scikit-learn. But the architecture does not depend on the classifier being trained. What matters is a separate, cheap, inspectable gate that runs before routing and produces a structured decision. The repo implements the gate deterministically on purpose, so you can see the mechanics and, critically, it refuses to guess when nothing matches. Figure 9 shows the gate in the repo. It scores each agent against the YAML keyword rules and, when nothing clears the bar, returns "unclassified" rather than picking the highest-scoring agent by default.

```
```
```
def classify_intent(user_input: str) -> Dict:
    """Classify user intent based on YAML keyword rules.
    Returns a structured decision the calling agent can act on:
      - matched:    {"status": "classified", "agent": <name>, "score": <float>}
      - no match:   {"status": "unclassified", "agent": None, ...}
                    The request is left unclassified rather than defaulting to a
                    fixed agent; the caller decides or asks the user.
    """
    user_input_lower = user_input.lower()
    agent_scores = {}
    for agent_name, agent_rules in INTENT_RULES.get("agents", {}).items():
        keywords = agent_rules.get("keywords", [])
        weight = agent_rules.get("weight", 1.0)
        score = sum(1 for kw in keywords if kw in user_input_lower) * weight
        agent_scores[agent_name] = score
    # No confident match: do NOT default to a specific intent. Hand control
    # back so the platform can ask the user or fall through to a grounded path.
    if not agent_scores or all(score == 0 for score in agent_scores.values()):
        return {
            "status": "unclassified",
            "agent": None,
            "reason": "No intent rule matched the request.",
            "available_agents": list(INTENT_RULES.get("agents", {}).keys()),
            "next_action": "ask_user_about_web_search",
        }
    best_agent = max(agent_scores, key=agent_scores.get)
    return {"status": "classified", "agent": best_agent, "score": agent_scores[best_agent]}
```
**Figure 9. The classify_intent gate (intent_classifier/classifier.py).**

A basic router would force the request to the highest-scoring agent even on a weak match. That is exactly how an agent ends up handling a question it wasn't built for, leading to hallucinations. By explicitly returning "unclassified" instead, the platform chooses the safest next step, either rejecting the request, asking the user to clarify, or falling back to a strictly constrained search.

The payoff comes twice. The gate catches requests the model would hallucinate on. In addition, by picking the right tools and prompt up front, it shrinks the space the model has to search, which itself lowers the hallucination rate as well as keeping the context minimal and concrete.

We keep the router as a separate, swappable component so you can easily choose between a free keyword gate or a more accurate, trained classifier in front of the foundation model, for example a fine-tuned transformer via Hugging Face or a lightweight model built with scikit-learn.

## Authentication and Authorization: Securing the MCP Layer

Most write-ups treat agent security as a prompt-injection problem. In a multi-team deployment, the more important problem is simpler. When several teams' tools live behind MCP servers and several roles share the platform, *which caller may invoke which tool?* That is RBAC. It belongs at the platform layer, enforced at the resource server instead of enforcing the access control in each application.

The platform issues a stateless JWT at login, signed with PyJWT. The token carries the caller's role as a claim, so any service holding the shared secret can verify it and learn the role without calling back to the auth server. Figure 10 shows how the platform mints that token. It signs the caller's identity and role into a short-lived JWT that any downstream service can verify with the shared secret.

```
```
```
def issue_token(username: str, role: str, ttl_seconds: int = TOKEN_TTL_SECONDS):
    """Mint a signed token for an authenticated user."""
    now = datetime.now(timezone.utc)
    payload = {
        "sub": username,
        "role": role,
        "iat": int(now.timestamp()),
        "exp": int((now + timedelta(seconds=ttl_seconds)).timestamp()),
    }
    token = jwt.encode(payload, JWT_SECRET, algorithm=JWT_ALGORITHM)
    return {"token": token, "token_type": "Bearer", "role": role, ...}
```
**Figure 10. The issue_token function (gateway/authz.py).**

Our permissions live in a YAML config file rather than hardcoded rules, using a default-deny posture so security changes only require a configuration edit, not a full deployment. Below is an example of the policies config shown in Figure 11.

```
```
```
developer:
  description: Development-only access (no production)
  permissions:
    tools:
      - read_file
      - list_directory
      - query_database
    resources:
      - "file://src/*"
      - "db://dev/*"
      - "db://staging/*"
    operations: [SELECT, GET, LIST]
  # Explicitly blocked: production databases, deployment, admin operations
```
**Figure 11. The authorization policy (gateway/policies.yaml).**

The decision I would defend hardest is where this policy is enforced. It would be easy to check permissions once at the gateway and trust everything downstream. We don't. Each MCP server, built with FastMCP, is an independent resource server that verifies the token and checks the policy itself, on every tool call, before the tool runs, as shown in Figure 12.

```
```
```
from fastmcp.server.middleware import Middleware, MiddlewareContext
class AuthzMiddleware(Middleware):
    """Deny tool calls whose caller role isn't permitted by the policy."""
    async def on_call_tool(self, context: MiddlewareContext, call_next):
        tool_name = context.message.name
        auth_header = get_http_headers(include={"authorization"}).get("authorization")
        try:
            claims = decode_token(token_from_header(auth_header))
        except AuthError as exc:
            raise ToolError(f"Unauthorized: {exc}")
        role = claims.get("role", "")
        if not policy_engine.can_access_tool(role, tool_name):
            logger.warning("MCP authz DENY role=%r tool=%r", role, tool_name)
            raise ToolError(
                f"Forbidden: role '{role}' is not authorized to use tool '{tool_name}'"
            )
        return await call_next(context)
```
**Figure 12. The AuthzMiddleware enforcement point (mcp_server/authz_middleware.py).**

## Observability for LLMs

Standard APM was the most expensive thing we got wrong. Latency, error rate, and throughput tell you whether the model is reachable; they tell you nothing about whether it's behaving correctly or not.

Here is what a production LLM system actually needs:

- The hallucination rate is the percentage of AI responses that either break our formatting rules (schema failures) or invent facts missing from our source documents (grounding failures), tracked per app and prompt version.
- Prompt drift is the rate of behavioral change across prompt versions. A new version that raises the hallucination rate by two percent must be visible before it reaches every user.
- To drive effective cost optimization, we break down token spend by team, application, and specific use case to automatically highlight our highest consumers.
- Tool invocation patterns, such as which tools agents call, in what order, and with what failure and denial rates. This invocation is where multi-agent pipeline issues show up.
- Behavioral correctness sampling, which is offline evaluation of sampled production traffic against a golden set, covered in the "Evaluating Agent Behavior" section.
- By capturing request-level success rates and latency per user, we tie semantic AI quality directly to traditional operational metrics.

The detail that makes "per team" work is that the team dimension must exist in the data from the first call. It cannot be rebuilt later from logs. The collector derives the team from the authenticated role and carries it on a context variable set at request ingress. Every record is attributable and no function needs a team ID threaded through it. Figure 13 shows how that attribution is structured. The role maps to a team, while each user's request, token, cost, and hallucination counts accumulate on a single per-user record.

```
```
```
ROLE_TEAM: dict[str, str] = {
   "viewer": "shared",
   "analyst": "analytics",
   "developer": "developer",
   "deployer": "devops",
   "admin": "platform",
}
@dataclass
class UserMetrics:
    user_id: str
    team: str = "unknown"
    requests: int = 0
    successes: int = 0
    errors: int = 0
    hallucinations: int = 0
    input_tokens: int = 0
    output_tokens: int = 0
    total_cost_usd: float = 0.0
    total_latency_s: float = 0.0
```
**Figure 13. The  ROLE_TEAM map and UserMetrics record (observability/metrics/collector.py).**

Token cost is recorded at every model call site, not just at the final response. The recording lives in the agent's `after_model_callback`, so a request that fans out to four specialists attributes all four calls to the same user and team. The hallucination counter on the same record is incremented by the schema-validation check. Latency and success/error counts come from a context manager wrapping the gateway's execution endpoint. One snapshot endpoint exposes everything per user or team. Figure 14 shows a real snapshot from the running system for a single devops user, with tokens, cost, latency, and success rate on one attributable record.

```
```
```
{
       "user_id": "dale",
       "team": "devops",
       "requests": 1,
       "successes": 1,
       "errors": 0,
       "hallucinations": 0,
       "input_tokens": 191,
       "output_tokens": 79,
       "total_cost_usd": 0.0,
       "total_latency_s": 11.145345083001303
}
```
**Figure 14. Example metrics snapshot for a user (observability/metrics/collector.py).**

We track local token estimates separately from the actual API response data so our internal counts never mix with the true billing data.

We use OpenTelemetry to ship our metrics out, but we wrap them in custom methods so we can swap backends like Prometheus, instantly without touching our application code.

## Evaluating Agent Behavior

Unit and contract tests pin down the deterministic path, but they cannot answer the question that matters most for an LLM system: Is the model still behaving? Answering that question is what the evaluation layer is for and it is the part most worth building carefully, because it is the closest thing an agent platform has to a regression suite.

The process has four parts: a golden set of representative cases, a runner that drives the real agent through them, a set of scored metrics, and a threshold that turns those scores into a pass/fail gate.

The golden set is data, not code. In the repo it is a JSON file, with each case fixing one expected outcome, consisting of a user request, the tool trajectory the agent should follow, the response it should return, and the role the request runs under. The seed cases cover the routing contract. A documentation request should classify intent and then hand off to the docs specialist. A code-review request to the codebase specialist. A research request to the research specialist. In production this set is the artifact that grows fastest. Every incident that reaches a user becomes a new case, so the same failure cannot ship twice.

The runner is ADK's `AgentEvaluator`. It executes the real agent against every case, several times per case and averages the scores, repetition matters because the model is stochastic and a single run tells you little. The metrics ADK exposes cover the two things worth checking separately: whether the agent took the right path (the tool trajectory, did it classify and then route to the correct specialist?) and whether it produced the right answer (response match against a reference and a model-graded quality score). A safety metric rounds out the set.

The threshold is what makes it a gate rather than a dashboard. The repo requires an average tool-trajectory score of at least 0.8 across runs. Below that score, the evaluation fails. That number is deliberately visible in a three-line config file, not buried in code, so tightening the bar is a reviewable change.

Two design choices make this effort worthwhile. First, the split is enforced in continuous integration. The fast, hermetic tests run on every push and gate merge. The model-backed evals need a live model and the tool servers, so they run as a separate, explicitly triggered job that gates prompt promotions. You do not pay for a live model on every commit, but no prompt reaches production without clearing the bar. Second, the golden set itself has a test. A lightweight check runs on every push, with no model that validates every case still encodes a valid contract and every metric named is one the runner understands so the evaluation suite adheres to the rules on each run.

Paired with the prompt registry, this design is what closes the loop on prompt changes. A candidate prompt version is evaluated against the golden set before it is promoted. If its trajectory or grounding scores fall below the live version, it never ships. That is how "prompt drift" moves from a post-incident finding to a predeployment gate and it is the same measurement discipline behind the hallucination numbers in this article, applied to sampled production traffic against the same references. Figure 15 shows one case from the golden set. The user request, the exact tool trajectory the agent should follow, and the role the request runs under.

```
```
```
# tests/evals/data/root_agent.evalset.json -- one golden-set case (abridged).
# Each case fixes the expected outcome: the user request, the tool
# trajectory the agent must follow, and the role the request runs under.
{
  "eval_id": "docs_lookup_routes_to_docs_agent",
  "conversation": [{
    "user_content":  {"parts": [{"text": "Write documentation explaining the FastAPI routing API."}]},
    "final_response": {"parts": [{"text": "Routed to the documentation specialist, which grounds its answer in the FastAPI docs."}]},
    "intermediate_data": {
      "tool_uses": [
        {"name": "classify_intent",   "args": {"user_input": "Write documentation explaining the FastAPI routing API."}},
        {"name": "transfer_to_agent", "args": {"agent_name": "docs_agent"}}
      ]
    }
  }],
  "session_input": {"app_name": "app", "user_id": "eval_user", "state": {"user_role": "viewer"}}
}
```
**Figure 15. A golden-set evaluation case (tests/evals/data/root_agent.evalset.json).**

As shown in Figure 16, the pass/fail bar lives in a three-line config file. The average tool-trajectory score across runs must clear 0.8 or the evaluation fails.

```
```
```
// tests/evals/data/test_config.json -- the pass/fail bar.
{ "criteria": { "tool_trajectory_avg_score": 0.8 } }
```
**Figure 16. The evaluation threshold (tests/evals/data/test_config.json).**

## Architectural Patterns for Multi-Team Deployments

When more than one team builds on the same platform, the question shifts from "how do we serve this model" to "how do we serve N teams sharing this infrastructure". Three patterns mattered most:

- **The gateway pattern**
 One ingress that handles authentication, per-tenant rate limiting, routing to the right model and prompt, and observability emission. It is the API-gateway pattern from microservices, applied to LLM traffic.
- **The agent coordinator pattern**
 To prevent cascading system failures, a multi-agent pipeline must have a single orchestrator that owns the entire topology. Without a central owner to manage circuit breakers, enforce timeout budgets, and validate data between transitions, individual agents will independently try to recover from errors. When an upstream issue occurs, agents blindly retry the components above them, turning a single isolated failure into a system-wide crash.
- **The cost attribution model**
 Every request enters the platform tagged with team, application, and use case. The tags propagate through every downstream model call. This approach is what makes per-team cost dashboards possible and it has to be designed in from the start.

## Summary

To summarize the article, the six concrete decisions, in priority order are:

- Build the observability layer before the first production deployment, not after the first incident. Skipping it for v1 is the most expensive shortcut available. Without behavioral metrics, you cannot tell the system is degrading until users complain. By then, you have lost trust.
- Adopt platform thinking before the second application, not the tenth. Every shared concern built into application code must be refactored out later, under deadline pressure.
- Employ version prompts from day one. A registry with ID-based references is small to build. Building it after your first silent regression costs far more, and that incident is a near certainty.
- Enforce schemas at the platform boundary on the first call, not after the first downstream parse failure. Anything less means defensive parsing in every consumer.
- Tag every request for cost attribution from the start. Without per-team tags, the first time leadership asks what this approach costs, the answer will be a multi-week investigation. With them, the answer is a dashboard.
- Stand up a golden-set evaluation per specialist before the first prompt change, not after it. The registry lets you roll back fast. The golden set tells you a version needed rolling back before your users do.

The unifying thread among these decisions is that LLM systems let you defer infrastructure decisions in ways traditional systems do not. They do not crash. They do not return 500 errors. They quietly produce slightly worse output. That gap between a wrong decision and its visible consequence is why platform thinking pays off up front, and why retrofitting it under fire costs so much.

The shift from "we built an LLM application" to "we built on an LLM platform" is the shift the rest of the field will make over the next two years. The only question is whether a team makes it on its own timeline or under pressure from an incident.

The complete, runnable implementation of every primitive in this article is here.
