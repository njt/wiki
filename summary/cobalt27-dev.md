---
url: https://cobalt27.dev/
title: "COBALT — GraphQL from TypeScript, without writing GraphQL"
author: "Cobalt team (uncredited)"
date_fetched: 2026-11-26
date_published: unknown
topics:
  - coding-agents-and-frameworks
  - agent-memory-and-context
---

COBALT is a TypeScript web framework that generates a GraphQL API from your TypeScript code — "your TypeScript code IS your schema" — and ships a typed client SDK with no codegen step. The pitch page positions it against REST (endpoint fatigue, over-fetching), pure GraphQL (the double-declaration problem of drifting interfaces, schemas, and resolvers), and tRPC (tight coupling: backend types leak into the client bundle, awkward for microservices or public APIs).

The most interesting section for this wiki is the explicit "LLM Friendly" pitch: "LLMs struggle with large, scattered codebases. Providing context across 50 files degrades performance and increases cost." Cobalt's answer is dense, collocated code — no controllers, serializers, or manual schemas — so a smaller context window footprint yields "smarter AI assistance and lower token costs." The tagline sells it outright: "The last framework that actually makes sense, in the age of AI."

This is marketing copy for an early framework, so take the claims as a manifesto rather than a benchmark. But it's a clear data point in the trend of frameworks designed backwards from what an LLM can hold in context, alongside training-data-dense stack choices.
