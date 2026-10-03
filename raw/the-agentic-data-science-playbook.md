---
url: https://www.oreilly.com/radar/the-agentic-data-science-playbook/
date_fetched: 2026-10-03
---

The following article originally appeared onVanishing Gradientsand is being republished here with the authors’ permission

When an AI agent can explore a dataset, choose a modeling approach, run the analysis, and explain its findings, what should the data scientist do?

Traditionally, data scientists chose each step and implemented much of the analysis themselves. Agentic data science changes that division of work: we can delegate an investigation, including methodological choices, while shaping the question, supplying relevant expertise, and challenging the evidence it produces. For AI-native data scientists, choosing the runtime, writing reusable skills, and designing the workflows and feedback that guide the agent are part of the analytical work.

This article provides a playbook for working with data science agents, from setting up an investigation to reviewing its results and carrying lessons into the next assignment. To see why that requires more than a capable model and a business question, consider an experiment we deliberately started with too little guidance. We gave Claude Opus 5.0 a modified version of the public Elliptic dataset and asked, “Build me a model to detect fraudulent nodes.” The dataset is a graph of Bitcoin transactions: each node is a transaction, and an edge represents a flow of bitcoin between transactions. Some transaction nodes carry licit or illicit labels based on the entities that created them; the rest are unlabeled. Each node has a time step indicating when its transaction was broadcast, allowing us to train on earlier time steps and test on later ones. We also wanted separate results for transactions with many connections (high-degree nodes), which mattered most in the intended application. Importantly, we renamed the columns, changed several features, and reindexed the time steps while preserving their order, making the public dataset harder for Claude to recognize.

Claude wrote the code, trained a random forest, and reported an F1 of 0.87 and ROC AUC of 0.99. It had split transactions randomly, mixing earlier and later time steps in both the training and test sets. That test did not measure how the model would perform on transactions from later time steps. Moreover, Claude also used a feature we had planted as a proxy for the fraud label (yes, we tricked it!), giving the model leaked information it would not have when scoring a new transaction. So how do we avoid these situations?

**We then supplied the guidance missing from the initial prompt:**  We required a temporal holdout, removed the leaking feature, supplied context about how the model would be used in production, and asked for separate reporting on the high-degree nodes that mattered most. Under the corrected evaluation, F1 was 0.70, overall recall was 0.61, and recall on high-degree nodes was 0.21.

The key point is that “Build a fraud detector” left Claude to infer how the model would be used and what would count as success. **AI-native data scientists build and direct** an analytical process in which agents can investigate, receive feedback, and return evidence for review. The work begins with deciding how much of the investigation to delegate.  Agentic data science is doing data science work with AI agents as teammates. Crucially, the scope of their responsibility can extend well beyond code implementation. An agent can help frame a question, explore data, test a claim, or communicate a result, provided it has the context and tools to do the work, a way to assess its progress and validate its results.

Asking an agent to write a pandas transformation leaves you as the bottleneck, responsible for deciding every next operation. Asking it to investigate a change in customer behavior gives it larger analytical responsibility. It can inspect a result, form another question, choose a method, and continue. The interaction becomes a conversation about the investigation rather than a sequence of requests for code.

The question may be *descriptive* (what happened?), *diagnostic* (why did it happen?), *predictive* (what might happen next?), or *prescriptive* (what should we do?). The fraud model is predictive; the pricing investigation later in this article is diagnostic and causal. Across these kinds of work, we need to specify the question and intended use, then verify that the evidence supports the answer.

*The following five practices are key to agentic data science:*

- Frame the investigation.
- Equip the agent for the assignment.
- Organize the work through bounded experiments, competing analyses, or both, according to the question.
- Review the result independently.
- Preserve evidence and turn reviewed lessons into reusable expertise.

The first two practices set up the work. The third determines how the investigation proceeds; the fourth tests its claims. Evidence is captured throughout, and the fifth practice carries reviewed lessons into future assignments.

As in agentic software engineering, the agentic data scientist’s two central responsibilities are ** specification** and 

**. Specify the question, intended use, and evidence the agent should produce; then verify that its analysis supports the conclusion. Agents can help with both, while the data scientist remains responsible for judging the question and the evidence.**

*verification***1. Frame the investigation with the agent**

** Start by discussing the assignment with the agent.** Supply the intended use and organizational context, then let it inspect the data and propose an approach. Method selection can be part of its responsibility. Your intervention matters when a proposal changes the question, rests on a questionable assumption, or needs information the agent cannot obtain. Predicting fraud and deciding which flagged entities to investigate, for example, require different evidence about errors and their consequences. A brainstorming skill such as those in Superpowers can help structure that conversation before you turn it into a task prompt.

A useful specification records that shared understanding. It states the decision, relevant constraints, and evidence the investigation should produce. It need not prescribe every step. In the fraud example, “classify transactions from later time steps using only information available when each is scored” matters more than “use a random forest.” The former defines the analytical task, while the latter selects one possible implementation.

You can specify what the investigation must establish without specifying the answer you want. “Determine whether the data support a recommendation” leaves room for an inconclusive result. “Keep trying until you find an effect” does not.

Turn that discussion into a short analytical brief to give the agent as its task prompt. For an assignment like our fraud example, a starting version could read:

```
TASK PROMPT:
Question: Can we identify fraudulent nodes as they enter the network?
Use: Support investigation, with separate reporting on high-degree nodes.
Available information: Only inputs known when the node is scored.
Agent discretion: Explore data, propose eligible features, choose models.
Return to me: Unclear feature provenance, changes to the target or
population, or a trade-off that requires an operational decision.
Deliverable: Reproducible analysis, temporal evaluation, subgroup errors,
and a recommendation that states what the evidence cannot establish.
```
Review it with the agent before the investigation proceeds. If exploration reveals that the evidence cannot answer the question, revise the brief explicitly; do not quietly substitute an easier question.

The deliverable may still be a notebook, model, or report prepared outside a production service. You can begin in the workspace where you already do that work.

**2. Equip the agent for the assignment**

The task prompt tells the agent what to investigate. It also needs to know how the project works, reach the data, run the analysis, and check the result. The harness is the system around the language model that allows this: its tools, runtime, context, permissions, and feedback from its actions. Its runtime is the environment that executes those actions. A language model alone cannot inspect a warehouse, run a simulation, or recover an interrupted statistical model fit. The environment must make those operations possible and return useful evidence about what happened.

Runtime choices are analytical choices as well as engineering choices. Can the agent execute Python or R with the libraries the task needs? Can a long-running fit continue after an interactive session ends? Which scientific libraries should the agent use? Can the agent inspect plots, or does it only see the code that produced them? Can you reproduce the environment in which it reported a result?

An existing agent runtime may provide most of this. Configuring it means deciding what belongs in Markdown, what needs a tool, and what should be checked by a small script. In an investigation like the fraud example, Markdown can hold the brief and data definitions, while a Python script could check that the appropriate temporal validation split is executed. A CSV data extract may be enough for exploration; if the agent needs data warehouse access, a tool exposed through an MCP server can provide it with appropriately scoped, read-only credentials. A sentence in a prompt cannot enforce that access limit.

Take the same care with outputs. Ask the agent to preserve the data reference, code, environment, assumptions, and diagnostics behind its report. A chat transcript is a poor substitute for a runnable analysis. Review becomes much harder when the only surviving artifact is a confident paragraph about what the agent says it did.

Execution is only part of the problem. An agent may know how to fit a model and still misunderstand what the columns mean. It may find five revenue tables and choose the wrong one. A schema rarely explains which customers were eligible for an offer, when a measurement changed, or why the team stopped using an apparently reasonable metric. This is where **agent skills** and **domain knowledge** enter. A skill packages instructions and resources for a type of analytical work. It might contain a modeling approach, example code, required diagnostics, and guidance on when to ask for help. Data documentation supplies the organizational meaning: canonical definitions, table grain, known limitations, and the history needed to interpret a result.

A useful skill is specific enough to change the agent’s behavior. “Be rigorous” gives it little to work with. A fraud-modeling skill can require the agent to establish feature availability, evaluate on later observations, and report performance on operationally important subgroups. For example:

```
For fraud prediction:
Establish what information is available when a node is scored.
Exclude features derived from subsequent investigations or labels.
Fit preprocessing on training data only.
Evaluate on later-arriving nodes and report the required degree groups.
Flag uncertainty about feature provenance before claiming performance.
```
These instructions leave room to choose a model. They encode reasons that some apparently successful models should be rejected. Where a requirement can be checked reliably in code, the skill can call a script that performs the check and records its result.

Loading every method and every document into every assignment is unnecessary. Give the agent a way to find relevant expertise, including its scope and exceptions. A forecasting skill should not silently impose its evaluation rules on an unrelated retrospective analysis. Nor should a notebook from last year outrank an updated metric definition merely because it offers convenient code to copy.

To put these pieces together locally, begin with a file-and-code agent in a sandboxed project workspace, such as the following:

```
fraud-investigation/
  AGENTS.md                # Project instructions, where supported by the runtime
  brief.md                 # Agreed question and delegation boundaries
  data-notes.md            # Sources, column meaning, availability times
  skills/fraud.md          # The methodological guidance above
  environment.lock         # Dependency versions, in your tool’s format
  model/                   # Code the investigating agent may change
  results/                 # Experiment log, diagnostics, saved candidates
  review.md                # Acceptance decision and unresolved questions
```
Use a project instruction file, such as `AGENTS.md` in runtimes that support it, to explain which context files the agent should read and how to propose updates to them. In other runtimes, provide those instructions through the supported mechanism. Give the sandbox read access to the approved development data and write access to the model and results directories. Keep the final test data outside of the agent’s accessible workspace. The practitioner can run acceptance checks in a separate environment whose evaluator and data the investigating agent cannot modify. A different folder, or version control alone, is not an access boundary.

Now ask the agent to inspect the inputs, identify unresolved questions, and build a baseline. Before allowing repeated experiments, rerun that baseline and examine its feature-availability record, split dates, and subgroup report. This small rehearsal checks whether the setup works all the way from instructions to evidence. A missing subgroup report points to a different problem than a failed package installation. Resolve those problems before giving the agent a longer run.

**3. Organize the investigation**

*With the question framed and the agent equipped, the next choice is how to organize its work.* This depends on the intent of the data science problem. For descriptive work, exploratory data analysis may proceed one question and plot at a time. Building a predictive model may support repeated experiments against a fixed evaluator; a causal question may require comparing analyses built on different assumptions.

In a live exploratory analysis on *Show Us Your Agent Skills*, Eric Ma (Moderna) uses a marimo notebook as a shared workspace with an agent. He explains the protein mutation data, asks for one plot at a time, corrects a color scale that affects interpretation, and chooses the next question from what he sees. The agent edits the notebook and renders the plots; Eric supplies the domain context, checks the artifacts, and owns the interpretation. The reason Eric needed to be in the loop was that human understanding was part of the objective function here!

**Use a bounded experiment loop**

For predictive modeling, the autoresearcher pattern organizes the work into a repeatable loop: propose a hypothesis, change the model, evaluate it, and keep or revert the change. The agent records each result and uses it to choose the next attempt. Within the scope you give it, it can explore features and model structure as well as parameter values. This is an inner loop within a broader investigation: the data scientist frames the question and sets the evaluation, the agent searches within those boundaries, and the data scientist reviews the result (potentially using an independent agent) before deciding what to do next.

Define what the agent may change, protect the evaluator from those changes, and set a time or compute budget. This makes iteration a bounded task within the investigation.

A compact experiment contract could say:

```
Improve the supplied baseline within the agreed compute budget.
You may change model code and propose eligible features.
Keep the target definition, validation split, and evaluator fixed.
Record each hypothesis, change, result, and keep-or-revert decision.
Stop at the budget limit or escalate if the evaluation is unsuitable.
Return the best candidate and the experiment history for review.
```
In a separate exercise with its own baseline and evaluation, we used this pattern to improve a graph neural network trained on the network data from the opening example. We used lower validation loss as the rule for keeping a change; F1 for fraudulent transactions was a separate measure of the resulting classifier. The agent ran 41 experiments while the team slept and retained seven changes that reduced validation loss. On the validation set, loss fell by about 70%, and F1 for fraudulent transactions rose from about 0.72 to 0.82. The log preserved both successful changes and failed attempts, so we could examine how it reached the result.

One candidate had the highest F1 for fraudulent transactions, but the agent rejected it because its validation loss was higher. That followed the selection rule we had set. The experiment history records that choice for the subsequent review.

The autoresearcher pattern works when an objective gives the agent useful feedback on each attempt. But some investigations turn on which assumptions to make, not which candidate scores best. Those tasks need a different way to organize the agent’s work.

**Investigate competing explanations**

In causal work, no held-out outcome directly reveals what would have happened without an intervention. The agent needs to examine how different analyses construct and test that counterfactual.

In a demonstration from our Master Agentic Data Science course using simulated subscription-business data, we asked: “What did the price increase cost us?” The outcome is daily conversion rate: paid conversions divided by the pool of potential subscribers. Choices about the observation window, counterfactual, exclusions, and validation produce different analytical paths. A final memo usually shows only one.

Two agent runs estimated conversion roughly 16% below their no-price-increase counterfactuals, yet shipped opposing claims. Run A attributed its estimated drop to a changing pool of potential subscribers and concluded there was “no real effect,” but did not validate that explanation. Run B backtested its counterfactual, ran a placebo check, and compared six specifications. It reported a robust relative reduction of 15.6%.

The parallel-analysis pattern has independent agents test different choices in the same investigation: one examines the observation window, another tests seasonal assumptions, and we compare their estimates, uncertainty, and diagnostics. Our Decision Lab work extends this approach across analytical paths, using checks to identify unsuitable analyses and unresolved disagreements.

Both approaches give the agent feedback while it works. The fixed evaluator steers the model experiments; diagnostics help it compare causal analyses. The output is a candidate and experiment history, or a set of analyses with their assumptions and checks. Those are the materials for the next task: *verifying the claim*.

**4. Review the result independently**

The agentic data scientist now needs to check what that evidence supports, and a fresh agent can help. Give an **independent agent reviewer** the original brief, data context, code, diagnostics, and final claim. Ask it to reproduce decisive checks and challenge assumptions. In this adversarial review pattern, the agent raises objections it can substantiate; the data scientist judges whether they change the conclusion.

In the fraud exercise, the agent used the same validation data to guide 41 experiments, so the reported gains may partly reflect what worked on that set. Freeze the selected candidate and assess it on an untouched holdout chosen for the intended use, including errors in the groups that matter.

In the pricing exercise, a fresh reviewer challenged Run A’s conclusion. Run A attributed the estimated decline to a changing pool of potential subscribers but provided no evidence for that explanation. The reviewer found that conversion had been rising before the price increase and that placebo interventions in earlier periods did not reproduce the negative effect. Run B’s analysis, which included these validation checks, was selected in the final comparison.

Netflix’s agentic workflow for causal inference implements this division: an actor performs the analysis and diagnostics, while a critic challenges the reasoning and claims. Humans can inspect and rerun the artifacts.

A fresh agent session is not necessarily an independent review if it can read the investigator’s earlier attempts through the workspace or Git history, though! For a check meant to stand on its own, give the reviewer the original brief, final artifact, and data needed for that check, while limiting access to the prior path. The full experiment trail can be examined separately when auditing how the result was reached.

These examples call for different balances of human and agentic verification. In Eric Ma’s EDA, the agent makes plots while Eric checks them and chooses the next question. In the bounded experiment loop, a fixed evaluator checks each candidate before a person reviews the selected model. In the pricing analysis, agentic diagnostics and critique help a data scientist judge what the evidence supports.

The low-low quadrant leaves little basis for trusting a result (in fact, it’s “vibe data science!”). Repeatable checks can move some work toward more agentic verification, while questions that depend on domain understanding or consequential decisions continue to need human judgment.

**5. Turn reviewed experience into reusable expertise**

The experiment loop and adversarial review both depend on an evidence trail: the saved artifacts that show what the agent did and why a conclusion survived or changed. Preserve that trail throughout each investigation, including data references, code versions, analytical choices, experiment results, diagnostics, and review findings. Keep the failed alternatives and review findings as well as the final report.

This trail has a second use beyond inspecting the current result. Reviewing it with the agent can reveal missing context, recurring mistakes, or methods worth reusing. The next question is which of those lessons should change how the agent approaches a future assignment. Leaving them in a conversation makes that learning difficult to carry forward.

In the fraud example, we deliberately planted a feature that leaked the fraud label. Removing it corrected that analysis. The reusable lesson is to have the agent check proposed features for leakage: where did each feature come from, and would it be available when a new transaction is scored? That requirement can go into a fraud-modeling skill for future investigations.

Before making a lesson into standing guidance, we need to define where it applies. The planted feature was a problem because it carried information unavailable at scoring time, not because it predicted fraud well. The temporal split likewise fits a task involving transactions from later time steps; it is not a rule for every analysis. A skill should capture those conditions so the agent applies the lesson to the right task.

For example:

```
Lesson: a feature encoded information from the fraud label.
Scope: prospective fraud prediction.
Update: require a documented source and availability time for inputs.
Evaluation: test whether the agent detects outcome-derived inputs
without rejecting legitimate signals merely because they predict well.
```
This is where evals enter: repeatable tasks with explicit criteria for assessing the data science agent’s behavior. Here, we evaluate how the agent conducts the analysis, not only its model’s predictive performance. The evidence trail supplies concrete failures that can become test cases for proposed changes to its skills or workflow.

Keep the evals, skill versions, and results together. As reviewed assignments reveal new failure modes, expand the cases and rerun them when the agent’s setup changes. The aim is evidence that its analytical behavior improves, rather than a growing collection of instructions that merely sound sensible.

Workflow changes can accumulate in the same way. If a reviewer repeatedly catches a missing diagnostic, move that diagnostic earlier. If a separate reviewer adds cost but never changes the analysis, reconsider its role. If the agent repeatedly asks the same question about a table, improve the data context rather than supplying the answer again in chat.

A completed assignment need not always produce a new skill. A one-off constraint belongs in the project’s notes; a recurring methodological failure may justify standing guidance. That distinction keeps the next investigation from inheriting every exception encountered in the last one.

**When other people use the agents you build**

When colleagues use an agent without you mediating each request, your local knowledge has to become shared infrastructure. OpenAI’s internal data agent combines institutional context with query evaluations and existing user permissions. Meta’s Analytics Agent draws on prior analytical work and reusable guidance, exposing generated SQL alongside results. Both illustrate why earlier analyses and corrections belong in the system, not only in an analyst’s memory.

In your own work, you can explain an unfamiliar table or catch a misleading conclusion as it appears. When colleagues use the agent directly, that support must be built into the system. Try an assignment with a colleague and note where you need to step in. Missing context belongs in the agent’s guidance; recurring mistakes become evals; questions beyond its remit need a route to a qualified reviewer. Someone must maintain that guidance, and access controls must limit each user’s data access. The analytical principles stay the same, but the agent can no longer depend on you being present for every investigation.

**Put the playbook to work**

Choose a small investigation you understand well enough to challenge: a model you periodically retrain or a business metric you regularly explain. Give the agent the decision context and ask it to propose an approach. Agree on what it can decide, then let it carry the investigation far enough to produce evidence you can inspect.

At review, pay attention to where your intervention changes the work. Did the agent need a definition only your team knows? Did a diagnostic overturn its conclusion? If that intervention would help on another assignment, make the relevant context or check available there, and test whether it helps.

AI-native data scientists use their expertise to build and direct analytical agents. They turn lessons from reviewing an analysis into skills and checks, then test whether those changes help the agent on future tasks.

*The next cohort of our**Master Agentic Data Science**course starts Oct 6.*
