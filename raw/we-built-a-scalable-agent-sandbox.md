---
url: https://newsletter.cloudsquid.io/p/we-built-a-scalable-agent-sandbox
date_fetched: 2026-08-25
---

Most agents work well in a singular workflow. 

When you run 100 agents parallely across the same enterprise data, things start to get complex. There are structural limits one hits when you keep adding more agent-driven workflows. 

Additionally legacy systems don’t come with API or webhooks. 

Instead of building custom fixes, we built an agent sandbox to handle scale while preserving governance and access controls, which works with the outputs of legacy systems: PDFs, CSVs, Excel and SFTP.

###### Agent Sandbox Demo

**The first approach:**

Our first approach was a state machine: a graph of every possible action a user could take, with the agent allowed to choose from only a small, pre-defined set of responses at each step.

The belief was that agents perform effectively when their environment is tightly-controlled. Evidently, we approached the build with extreme constraint. 

In our v1, when you ask the agent to build a workflow, it could offer to create a trigger or ask a clarifying question. It was just a tightly controlled response structure.

It worked, until we scaled. The real costs started showing:

- **Double Build:**once as a UI for the human, once as a rigid graph for the agent.
- **Edge Cases**: you cannot fully anticipate every path a real user will take.
- **Unused model capability**: the ability to reason broadly and synthesize information from many sources was locked away behind a tightly constrained box.
- **Model Improvement didn’t help**: because the agent was so boxed in, improvements to the underlying model couldn’t express themselves.

**The second approach:**

Imagine 20 people editing the same Excel file at once. 

It crashes or produces conflicting versions. Changes made by people silently disappear, and nobody notices. This just happens when concurrent actors touch a shared resource with no coordination model.

Now put agents in those 20 seats instead of people. They move faster and don’t pause to ask about the previous content. At scale, the architecture really gets constrained.

Our second component, SQL access to agents, was built for exactly this moment. Instead of navigating a fixed graph, agents could query structured data directly and reason over it. Most of our data was already organized into structured extraction tables, so this was a natural evolution.

The next bottleneck was PDFs, emails, and unstructured documents were still out of reach in enterprise implementation due to privacy and data concerns. Agents could reason over more data, but only limited to the databases.

**The third approach**

At the scale of running hundreds of agents, the problem became a product design question.

When you run a reconciliation task with 100 rows in a table and each row its own agent, all pointed at the same underlying dataset: invoices, delivery notes, ERP exports, PDFs, it produces some lesser known effects.  

We watched this happen in a recent multi-agent research study at Anthropic: agents overwriting each other’s work, producing conflicting results and even starting a turf war. At scale, this happens frequently, unless the architecture is explicitly built to prevent it.

To resolve it, we built  a sandboxed terminal for our agents similar to claude code or codex. Our agents now can run scripts, query databases, and read or write files through a terminal. It solved the ceiling that v2 hit.

But it introduced a structural issue of the infrastructure. Anthropic’s hosted terminal environment for the API caps file count per session, with a limited toolset. 

That ceiling exists to avoid getting abused by users as free storage or an attack surface. But for a finance team juggling hundreds of documents per task and tens of parallel tasks, a few file uploads are not enough. 

At this scale, we needed an architecture that was not going to hit a ceiling proportional to agent count. 

**The virtual file system approach:**

One isolated file store in Google Cloud. And for every single agent task, an isolated and permissioned copy of the files that task is allowed to touch.

Each of the 100 agents in a reconciliation task works inside its own clean environment. It reads what it’s permitted to read, writes what it’s permitted to write, and cannot see what’s been hidden from it. 

When it finishes, its changes sync back to the true source in a controlled, ordered way with no race conditions and no silent overwrites, fully auditable.  

We successfully decoupled agent count from collision risk. A hundred agents no longer means a hundred chances to collide, because collision was never possible in the first place.

Sequential hand-offs between agents, the instinct that “Agent A must finish before Agent B starts”, turn out to be almost never necessary. When the workspace is configured correctly upfront with Standard Operation Procedures (SOPs), a hundred agents can complete a reconciliation task in a single pass, with no coordination overhead.

If you are interested in learning more about our agent sandbox and try it out, connect with me on LinkedIn.
