---
url: https://commandline.microsoft.com/kantar-link-ai-content-optimizer-advertising-learning-loop/
date_fetched: 2026-10-08
---

## The Kantar LINK AI Content Optimizer

*Kantar has spent three decades building metrics and frameworks to calculate a signal for whether an ad works. Turning that signal into a system that improves ads meant building graders, evals, and a learning loop—with the realization that every component inside the loop is its own optimization challenge.*

## The scenario: A long history of rich data and domain expertise

Kantar is a leading brand intelligence company. For more than three decades, it has helped brands answer a deceptively simple question: will this ad work? Before AI, that kind of ad testing meant recruiting consumers, showing them the creative, running focus groups, and interpreting the results. And no company did it—or still does it—better than Kantar.

Asking real people how an ad influenced them—and doing it at scale for 35 years—produced the LINK database of 35 million consumer interactions and a testing database of over 260,000 ads. LINK AI, the ML-powered version, was developed as an ensemble of ML models, operationalizing the data and knowledge into multiple metrics of predictive scoring models, so a creative asset can be assigned scores in minutes instead of undertaking more involved survey research and feedback.

Enter social media, fractured audiences, and generative AI with multimodal LLMs. Many of Kantar’s business customers use LINK AI only for ad effectiveness scoring, but increasingly, they want to know whether an ad works on several metrics. They want to improve it for specific audiences and produce multiple variants.

This is the problem Microsoft Frontier Company and our FDE studio set out to solve with Kantar: build an LLM-powered LINK AI system that scales to larger volumes and can optimize the content by specifying what to change, why it matters, and how the new asset would score. The result is a learning loop wrapped around Kantar’s IP and a new product: **Kantar LINK AI Content Optimizer**.

The system is grounded in proprietary knowledge of what makes advertising effective and turns it into a repeatable optimization loop. Rubrics define what good ads mean across new user-defined dimensions (based on the original Kantar metrics), and LINK AI scores are decomposed across these dimensions to apply those rubrics as graders. A general reasoning agent uses skills following these workflows and produces targeted recommendations and new asset generation for revised scoring, so the same graders measure whether the asset improved. Expert feedback and experimentations tune the components around that loop. These are the main steps of our learning loop:

- Drive the agent with specific user inputs (evaluation sets) in the development environment and collect traces from the agent
- Assign scores to traces with LINK AI, with validation by human SMEs
- Make changes to the agent and repeat until scores are acceptable

### From IP to agentic system

The Kantar LINK AI Content Optimizer system has several inner loops inside the main product learning loop. While the outer loop improves the creative asset (score, recommend, generate, and re-evaluate), those on the inside improve the graders, tools, recommendations, generation choices, skills, orchestration, and evaluation process. LINK AI supplies Kantar’s proprietary grading signal and the scalable services needed to apply it repeatedly, making it a critical enabler of the loop. The remaining gap was actionability: scoring could identify how an asset scores, but not what should change next.

### Turn IP into actionable optimization signal

To support optimization, we simplified the LINK AI output to six key dimensions and extended grading beyond the whole asset to individual scenes and images. That made Kantar IP usable at each stage of the creative workflow: an asset score could show overall position, while component-level scores could indicate where a change might have the greatest effect.

This finer granularity also reduced a key measurement problem. A six-second improvement inside a 30-second video can be diluted in the overall score, whereas scoring the affected scene directly exposes the undiluted change. Scene- and image-level grading therefore created a tighter signal for both recommendation generation and later evaluation.

The LINK AI engineering work enabled different levels of granularity. And under production load with standardized ML services, cloud-native orchestration, queues, observability, and elastic scaling, it reduced the time and operational friction between iterations. These capabilities enabled running the learning loop frequently and reliably.

### Standardize ML services

LINK AI is an ensemble of scoring workflows built from open-source models, proprietary models, and LLMs. Without an optimized inference engine, the same hardware ran roughly eight times slower than it should—moving a GPU-bound embedding model from FastAPI to vLLM removed GPU dependency across several services. A single service template (manifest, standard endpoints, checklist, smoke test) brought unified telemetry across the estate and let the team—and its coding agents—ship a new ML service in hours from weeks previously.

### Orchestrate the services on Azure

We rebuilt the orchestration of the 20 ML services powering LINK AI on Dapr Workflow, with helpers that template most workflow code, so a new step lands in hours instead of weeks. Compute runs on Azure Kubernetes Service, observability on OpenTelemetry with Azure Monitor, and configuration and secrets in Azure App Configuration and Key Vault.

### Scale reliably under load

Scale was a tuning problem. The first load test failed about 80% of requests—prioritized queues and implementing advanced communication patterns with Dapr Workflow components took us to a 95% pass rate across thousands of requests. Requests were tiered, and the platform scaled through queues powered by the elasticity of Azure.

Repeated grading turns a scoring service into part of an optimization loop, so latency and scale were product requirements rather than background infrastructure concerns. A single video took about 15 minutes to score, and large batches could take days. Moving toward minutes per asset and hours for large batches, while controlling cost through elastic Azure scaling, made it practical to evaluate scenes, whole videos, and generated candidates repeatedly.

## Turn scoring into content improvement

Once Kantar graders could evaluate keyframes, scenes, and whole videos quickly enough to be called repeatedly, the scoring signal could move from reporting into optimization. LINK AI makes repeated grading operationally feasible, while the experimentation loops improve how reliably the full system performs each step.

SCORE → RECOMMEND → GENERATE → RE-EVALUATE

## The loops Inside the loop: Improving the system that improves the asset

Within the outer optimization loop, a reasoning agent coordinates the skills and tools used to analyze the video, apply Kantar’s rubrics, and generate recommendations. Each skill packages a distinct concern into reusable instructions, procedures, resources, and tool use. The reasoning agent decides which capabilities to invoke and in what order as the task unfolds, while carrying forward relevant history and outputs from earlier executions. This architecture is one tuning surface inside the larger learning loop, alongside LINK AI graders, content-understanding tools, recommendation logic, generation choices, orchestration, and evaluations.

Skills keep proprietary knowledge in dedicated, versioned components rather than embedding the full method in the agent’s general instructions. When loaded, those skills operate within the reasoning agent context; their value here is modularity, reuse, and governance rather than context isolation.

### Experimentation connects the inner tuning loops

Experimentation is the human-driven optimization that improves every component inside the outer loop. Kantar SME feedback and automated evaluations provide evidence about recommendation quality, while traces expose how the system reached each result. The team uses that evidence in a controlled cycle: establish a baseline; identify a failure mode; change one configuration in a prompt, skill, tool, model choice, grader, or orchestration decision; evaluate again; and promote or reject the change. The learning comes not from any single component, but from repeatedly improving the system around the task.

### Evaluations

A recommendation can be well grounded and still be useless. A generated candidate may also fail to capture the recommended change accurately. Neither failure is reliably exposed by the final asset score alone, so we built an evaluation framework along the stages of product releases pre-production, combining automated metrics with rounds of SME reviews. Kantar experts assessed recommendations and supplied qualitative and quantitative feedback while the team analyzed coverage, failure modes, and differences between configurations.

Collecting expert evaluation is expensive and needed its own tooling. The quantitative and qualitive feedback SMEs share across a full experiment run to a volume no one can hold in their head and compare fairly.

This is where the automated side earns its place. Modified videos went out, expert responses came back, and LLM-based judging was used to read across those comments and answer whether a configuration was improving. The work that would have required a person to read and reconcile hundreds of individual assessments became something that could run every time a configuration changed.

Neither kind of eval is sufficient alone. Automated judging scales and drifts. Expert judgment is the calibration and doesn’t scale. Running both and using each to check the other is critical.

A tunable harness creates possible changes. Evaluations decide which of them are improvements. Without the second half, the first half is a churn.

#### Let production evidence guide the next tuning experiment

As the system moves into broader production use, usage telemetry adds another source of evidence. Data on which recommendations users select and how revised assets score reveals where prompts, skills, tools, and orchestration need adjustment.

## The learning loop takeaways

Write the rubrics first. Turn rubrics into graders and expect to need more than one level. A whole-asset score locates you. A component-level score tells you what to change. Make grading fast. If a turn is too slow, there is no loop, only a report. Evaluate the optimizer separately from the asset.

Budget for expert evaluations and build the tooling for it. Expert evals are a key signal to collect and calibrate everything else. Use automated judging to scale expert judgment, not to replace it. Read the two against each other, and use expert-labeled examples to calibrate, evaluate, and refine the automated judges over time. In our work, expert assessments helped tune the automated evaluation, so it better reflected Kantar standards.

Decide how proprietary knowledge will be packaged and governed before the skills multiply. Skills create separation of concerns, while requirements for stronger isolation may call for sub-agents, access controls, or separate execution contexts. Keep telemetry on everything. Traces are what makes experimentation possible later.

A competitor can use the same foundation model. It can adopt similar infrastructure. It can draw the same architectural diagram. What is hard to reproduce is a scoring system built on decades of proprietary data with rubrics engineered to scale, graders calibrated by people who know the domain, and an experimentation process that converts that evidence into a better system.
