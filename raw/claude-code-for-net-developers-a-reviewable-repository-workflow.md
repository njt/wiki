---
url: https://www.devleader.ca/2026/10/12/claude-code-for-net-developers-a-reviewable-repository-workflow
date_fetched: 2026-10-14
---

Claude Code for .NET development is easier to understand when you start with a repository task, not a list of AI features. An existing endpoint needs a small behavior change. The solution already has conventions, authentication, tests, and a build environment. The useful question is how to make the agent work within those constraints and return something a developer can evaluate.

The answer is a reviewable loop: establish the environment, identify the relevant code, describe the contract, authorize bounded work, and inspect evidence. None of these steps makes generated code correct automatically. Together, they make mistakes easier to detect before the change becomes someone else's problem.

Information checked October 9, 2026. Product behavior below is grounded in freshly accessed official documentation; commands and examples illustrate the workflow and were not executed for this article.

## What Claude Code does in a .NET repository

The official Claude Code overview describes an agent that reads code, edits files, and runs commands. That makes it different from asking a chat interface to generate a class and copying the answer into an editor. Repository tools can discover surrounding code and modify several affected files.

It also makes Claude Code different from calling a model inside your application. In this article, the agent is a development tool working on source code. Your ASP.NET Core application does not acquire an AI dependency simply because the agent helped change it. An application that calls an external agent has another integration boundary, illustrated by an ASP.NET Core agent client.

Capabilities are not evidence of success. Reading a repository does not mean every constraint was found. Editing a handler does not mean its callers remain compatible. Running a test command does not mean the intended regression was covered. A useful workflow asks the agent to report those distinctions rather than flattening them into “done.”

Start with one reader-visible behavior. For example, an orders endpoint should reject a negative page-size argument instead of forwarding it to a query service. That is a concrete contract. “Improve the API” is not. It invites unrelated changes to routing, response models, dependencies, and architecture that make review harder.

## Establish which environment owns the work

A .NET solution can involve several environments: a Windows editor, a terminal, WSL, and a remote CI runner. The agent's environment determines which executables and files it can actually use. Do not infer its SDK from the SDK selected inside your IDE.

The official setup documentation supports native Windows and WSL. It currently distinguishes native Windows, where Claude Code sandboxing is not supported, from WSL 2, where it is supported. That is a product boundary, not a reason to change an existing repository's operating-system requirements.

Before delegating, identify the checkout, shell, SDK selection, and validation location. The following PowerShell example gathers a modest baseline from a fictional checkout:

```
Set-Location 'C:\repos\OrderService'
git status --short
Get-Command claude
Get-Command dotnet
claude --version
dotnet --info
```
These commands answer different questions. Git status establishes existing changes. Executable lookup shows what the shell resolves. Version information records the agent release and .NET environment. None proves that the application compiles or that its integration dependencies are available.

Keep existing changes visible. If another developer has edited a response model, the agent must not treat that work as disposable noise. A separate worktree can isolate new edits, but the baseline still matters: the commit and files being reviewed must correspond to the environment used for validation.

The official overview documents a VS Code extension and a JetBrains plugin. Visual Studio is not VS Code. Running a terminal beside Visual Studio does not establish a native Visual Studio extension. This workflow does not depend on an unverified IDE integration.

## Give the solution context before the implementation request

Repository context has two jobs. It supplies stable constraints, and it helps the agent find the right evidence. For .NET, useful context includes the solution entry point, target frameworks, SDK selection file, central package-management files, test projects, and where endpoint contracts live.

The official memory documentation explains `CLAUDE.md`, `AGENTS.md`, path-scoped rules, and auto memory. These are context mechanisms, not enforced access controls. A sentence saying “never modify migrations” guides behavior; it does not create an operating-system boundary.

Do not create another policy document just because the agent has a different name. If the repository already has maintained instructions, determine how the current Claude Code release loads them. The documented `AGENTS.md` behavior is version-sensitive, including interactions with `CLAUDE.md` and `CLAUDE.local.md`. Check the current page rather than copying an old workaround without understanding it.

The task prompt should then narrow the stable context to the change at hand. Here is an illustrative terminal launch:

```
Set-Location 'C:\repos\OrderService'
claude "Inspect the orders endpoint and its existing tests. Explain how pageSize reaches the query service and identify authentication, validation, and response contracts. Do not edit files, install packages, run builds or tests, contact databases, commit, or publish."
```
This is an initial discovery request, not permission enforcement. Its value is the requested output: a trace of the argument and the contracts around it. A reviewer can compare that trace with the actual handler, service, and tests before asking for edits.

The same discipline appears in broader repository-based .NET agent workflows, although product-specific configuration should not be transferred between tools. The reusable principle is to identify constraints before generating code.

## Trace one request through the affected surfaces

For the page-size example, begin at the route. Is the argument bound from the query string? Is it nullable? Does an existing filter validate it? Does a service apply a default? What happens when the client sends a nonnumeric string rather than a negative integer?

These are separate paths. Model binding can reject an unparseable value before a handler runs. A handler can reject a parsed but out-of-range integer. A service can enforce another domain constraint after the handler accepts input. The change needs to identify which boundary owns the new rule.

Then inspect response shaping. If neighboring endpoints return a standard problem response, returning an arbitrary anonymous object would create an inconsistent contract. If the route group requires authorization, moving the route outside that group would be a functional and security change even if the validation expression looked correct.

Middleware and endpoint code are also different layers. Understanding ASP.NET Core middleware and continuation helps explain why a small handler edit should not casually reorganize the request pipeline.

Ask for a proposed affected-file set, not an arbitrary one-file limit. The endpoint, regression test, and API documentation may all require changes. The goal is the smallest complete change, not the smallest diff that leaves consumers guessing.

## Make acceptance criteria observable

Acceptance criteria should describe what a client or collaborator can observe. “Add validation” is weaker than specifying the allowed range, status code, response shape, and whether the query service may be called.

For this hypothetical task, the request might say:

Reject pageSize values below 1 or above 100 using the existing API error format. Preserve omitted-parameter behavior, authentication, tenant scoping, ordering, and the successful response shape. Invalid values must not reach the query service. Add regression coverage using existing test conventions. Stop if implementing this requires a schema, package, or authentication change.


The omitted-parameter requirement is important. An optional query argument might currently default to 20. Changing it into a required argument would break clients even though tests for 0 and 101 passed. Requirements should name compatibility paths as well as the new error path.

Avoid asking for a number of tests as the primary objective. Five tests can assert the same thing. One parameterized test can cover several independent boundaries. The useful unit is behavior, not test count.

For larger systems, unit and integration test boundaries help separate isolated logic from interactions between real modules. The exact framework and fixture should come from the repository, not from the agent's favorite sample.

## Separate instructions from authority

Claude Code's current permissions documentation distinguishes allow, ask, and deny rules, with deny evaluated before ask and allow. It also distinguishes permission modes. These controls belong to the runtime, whereas prose instructions influence the model's choices.

That separation matters when a task crosses from reading to editing or from editing to executing. Installing a package, running a build, executing integration tests, applying migrations, and publishing are not interchangeable “developer commands.” They have different effects and may require different credentials.

A narrow settings example can deny both built-in shell tools while allowing a review-only session to retain non-shell discovery tools:

```
{
  "permissions": {
    "deny": [
      "Bash",
      "PowerShell"
    ]
  }
}
```
This demonstrates bare-tool denial in the documented syntax. It is not a complete security policy: file tools and any connected external tools still need their own review. Nor is it suitable for every development session, since denying shell execution removes useful command-based inspection.

The alternative is approved, narrowly scoped execution in a prepared validation environment. That supports stronger evidence but requires more setup. The right distinction is between authorized work and unauthorized work, not between inconvenient prompts and frictionless automation.

Do not describe a command prefix deny rule as protection against every equivalent command. The official permissions page explicitly explains Bash rule limits, including commands written in different forms. Credentials and environment isolation remain meaningful boundaries.

## Use code intelligence without treating it as a build

Claude Code's code intelligence documentation lists the official `csharp-lsp` plugin and the separate `csharp-ls` server binary. Installing one does not install the other. The documented plugin functionality includes symbol navigation and diagnostics after edits.

That can help the agent find a service interface, inspect its implementation, and follow references. It does not turn diagnostics into proof that every target framework or integration path works. The same documentation says plugin language servers do not start in cloud sessions, so a local navigation experience cannot be assumed there.

Text search remains useful for things that are not C# symbols: route strings, configuration keys, SQL, and documentation. A healthy discovery process combines the relevant evidence sources instead of treating one navigation tool as universal.

When the agent reports a diagnostic, ask which environment produced it. Editor inspections, a plugin language server, and CI compilation can disagree because they loaded different configurations. The disagreement is a reason to investigate, not permission to select the green result.

## Ask for evidence that matches the contract

A final handoff should connect each requirement to evidence. For a negative page size, that means a test that actually sends or constructs the negative input, asserts rejection, and checks that the downstream query was not performed where that is part of the contract.

A green test that only verifies a mock returned a configured object does not establish endpoint behavior. Likewise, a passing handler unit test cannot prove that query binding, routing, authorization, and serialization produce the intended HTTP response.

The processing-pipeline testing examples illustrate why rejection and failure paths deserve direct assertions. The broader lesson applies here: name the behavior, exercise it, and inspect the result rather than relying on confident test names.

Execution evidence should include the command, revision, environment, result, and any skipped coverage. If execution was prohibited or unavailable, the handoff must say so. “Sample authored” and “sample executed successfully” are different statuses.

Human review then checks the diff itself: preserved authorization, unchanged persistence behavior, compatible response shape, complete validation branches, and no unrelated dependency churn. A clean test run helps, but it does not replace that review.

## Frequently asked questions

### Is Claude Code a .NET library?

Not in the workflow described here. Claude Code is a development agent operating on a repository. Calling a model from a C# application is a separate integration task with different dependencies and runtime concerns.

### Does it automatically know the whole solution?

It can read and navigate repository files, but that is not proof it inspected every relevant contract. Give it the solution entry point and ask for a concrete trace through the affected code.

### Can repository instructions prevent database writes?

Instructions alone cannot enforce that boundary. Use runtime permissions, appropriately constrained tools, and credentials that do not grant unnecessary access. Review external tools as well as shell commands.

### Do C# language-server diagnostics replace compilation?

No. Diagnostics can provide early feedback for the workspace the server loaded. Compilation and tests in the intended environment answer additional questions about references, build configuration, and behavior.

### Should the agent change only one file?

Not necessarily. One behavior may involve an endpoint, tests, and documentation. Restrict scope to the complete behavior and review the affected-file explanation, rather than forcing an incomplete single-file fix.

### What does a useful completion report contain?

It identifies changed files, connects acceptance criteria to evidence, records executed checks, and states remaining uncertainty. It should not describe unexecuted commands as successful verification.

## A repository workflow is a chain of evidence

Claude Code for .NET is most useful to understand as a sequence of bounded decisions. The environment determines available tools. Context identifies constraints. A request defines behavior. Permissions establish authority. Verification and review determine what the result actually proves.

That model keeps the developer's attention on the application contract rather than the agent's confidence. A small, explained, evidence-backed change is easier to maintain than a broad rewrite whose only justification is that the tool completed it.
