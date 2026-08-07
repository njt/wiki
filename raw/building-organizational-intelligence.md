---
url: https://www.oreilly.com/radar/building-organizational-intelligence/
date_fetched: 2026-08-07
---

## Introduction

Not long ago, one of my engineering directors came to me with a request: His team seemed overloaded, and he wanted to hire another engineer. I decided to test a research assistant I had been building—an AI agent connected to our internal systems via MCP—by asking it to analyze the team’s workload and write a hiring case.

What came back was thorough. Headcount, service ownership, sprint velocity, ticket backlog, and capacity allocation, all of it neatly summarized. But reading through the document, I felt the same frustration I’d felt with every AI-generated organizational report that’s come across my desk. It told me *what was happening* without helping me understand *why*, or what I should actually do. It was organized around the data rather than around the decision. In short, it was the kind of response that’s easy to agree with and difficult to act on.

Then I added one more thing to the configuration: the O’Reilly Expert MCP server. I reran the same analysis and asked a slightly different question: “How would the experts on O’Reilly review this request?”

Instead of leading with headcount and ticket counts, the output now opened with a finding: “The operational overhead problem is structural, not a staffing deficiency.” Citing the Google SRE framework’s concept of operational toil, it noted that the team was operating at approximately 67% toil, well above the threshold at which the SRE literature recommends structural intervention, and made specific, concrete recommendations: run a toil audit, set explicit reduction targets, and assign operational runbook ownership. This wasn’t a recommendation for whether to hire or not. It was a grounded, traceable argument for doing something else instead.

That difference—between a data summary and an expert-grounded recommendation—is what this paper is about.

What follows is a case study of how we built an organizational intelligence system at O’Reilly, using our own platform as a core component. The approach I describe is grounded in engineering because that’s where I work, but it generalizes to any function where important knowledge is scattered across multiple systems and important decisions require synthesizing all of it. The recipe has four steps: map your information hierarchy; connect those systems to an LLM via MCP and write a skill file that defines how it should reason; add the O’Reilly Expert MCP as an expert review layer that grounds the analysis in established frameworks; and build a lightweight system for human-in-the-loop review. I’ll explain each step in detail and make the case for why the third step is the one that changes everything.

## Why organizational intelligence is getting harder

To understand the problem this approach solves, it helps to look briefly at how engineering has changed over the past three decades. These forces have played out first and fastest in engineering, but as AI tools proliferate beyond the engineering team, the underlying dynamic of more output, more decisions, and more scattered information is spreading to every part of the organization.

In the waterfall era of the 1990s, software organizations ran on central plans. Everything was specified up front, and leaders maintained visibility precisely because all information flowed through a single coordinating document. The plans were brittle and often fictional by the time they were executed, but at least everyone knew what was supposed to be happening.

Agile replaced central plans with small, autonomous teams working in short sprints, and this solved the reliability problem while creating a visibility problem. Important decisions began happening locally and quickly—the right teams making the right calls—but the information needed to see across all of those decisions splintered into dozens of separate tools. Product strategy lived in one system, project execution in another, code in a third, and service ownership in a fourth. More things got shipped, but the big-picture view got harder to maintain.

The agentic era has intensified this dynamic dramatically. Individual engineers today can ship in a day what used to take a full sprint team. The output is extraordinary, but the visibility is nearly gone.

Any effort that spans multiple teams, such as a platform migration, a shared infrastructure change, or a reorganization, now requires enormous coordination overhead simply because the information decision-makers need to understand the full picture is distributed across too many places. And this isn’t a problem unique to engineering. It exists in any function that runs on data spread across multiple systems.

Faced with this visibility problem, I wanted to build something I could ask big-picture questions and get synthesized answers back quickly. Things like:

- What is the status of this cross-team migration effort, and which teams are behind?
- A team seems overloaded. Do they actually need another engineer, or is something else going on?
- What are the trade-offs of adopting this new infrastructure technology?
- Help me produce a scope statement from this product brief.

Building something that could answer these well took two foundational steps, and getting it to provide recommendations based on my specific business context took two more. While my specific tools are from engineering, the structure applies equally to a sales team synthesizing CRM data and market research, or a finance team working across an ERP, a planning tool, and external benchmarks.

## Step 1: Map your information hierarchy

Every organization has a set of systems where important knowledge lives, and those systems form a natural hierarchy that spans from strategic intent at the top to operational detail at the bottom. Before you can build a useful research assistant, you need to make that hierarchy explicit, because it’s the map of how decisions get made, which sources carry the most authority, and how different kinds of questions should be approached.

At O’Reilly, our engineering hierarchy looks like this:

| Layer | System | Purpose | 
|---|---|---|
| Roadmap | Productboard | Strategic goals, initiatives, and feature prioritization | 
| Execution | Jira | Epics, stories, sprints, and contributor tracking | 
| Implementation | GitHub | Source code, PR history, and event instrumentation | 
| Service catalog | Cortex | Service ownership, dependencies, on-call, and Slack channels | 
| Observability | Datadog | System performance, errors, and incidents | 

Your organization will have a different set of tools. A sales organization might place Salesforce at the top, followed by a revenue intelligence platform, marketing automation, and market research. A legal team might start with a contract management system, followed by a regulatory tracker, internal policy documentation, and a research database. The specific systems matter less than the act of mapping them: understanding which layer answers which kind of question, and which sources take precedence when they conflict.

## Step 2: Connect your systems via MCP and write a skill that describes how to reason

This step has two parts that must work together. First, you need to connect your systems to your AI tools via MCP. Then you have to write a skill file that tells the model what to do with that access. At O’Reilly, we call this complete grounding layer Expert Intelligence.

Configuring MCP is straightforward. Most major tools now offer MCP connectors, and connecting them is typically a matter of routine JSON configuration. For systems without MCP connectors, a bash-capable agent with `curl` and `jq` can often reach a REST API directly. MCP just makes it cleaner and more reliable.

But MCP connections alone aren’t enough, and this is the part most implementations get wrong. MCP gives the agent access to your data, but it doesn’t tell the agent how to use it effectively. Without explicit guidance, the agent retrieves information and organizes it the way the underlying systems organize it, which produces a data dump, not an analysis.

The skill file—a CLAUDE.md or SKILLS.md document that provides specific reasoning instructions—transforms retrieval into analysis. Mine defines the reasoning hierarchy (which systems to consult for which types of questions, and how to weigh them), the output format (this is not a coding agent—it produces reports and recommendations, not code), epistemic standards (show your work, name gaps, surface assumptions for human verification), and tone. On that last point, I borrowed one of the most useful instructions from Ted Lasso: “be curious, not judgmental.” Adding it meaningfully improved the quality of the output.

The skill is a codified version of how a skilled analyst would approach these questions. It encodes your organization’s reasoning process and makes it repeatable.

## Step 3: Add the expert layer

With the research assistant connected to our internal systems, I had something genuinely useful: fast, synthesized answers to questions that previously would have taken days to research. But I kept running into the same problem: The reports felt generic, and people didn’t trust them. This challenge points to a fundamental limitation of AI-generated organizational analysis that goes beyond any particular implementation.

### The generic analysis problem

General-purpose AI assistants tend to produce a recognizable kind of organizational analysis: technically reasonable, balanced, cautious, and ultimately not very useful. This isn’t primarily a failure of knowledge—every major LLM has absorbed an enormous amount of management and organizational thinking. It’s a failure of grounding. When an AI assistant has no specific framework anchoring its response, it tends to produce recommendations broad enough to apply to almost any situation: consider the trade-offs, weigh your options, and ensure alignment across stakeholders. These responses are hard to disagree with and just as hard to act on.

When a report says, “The team appears overloaded. Consider adding headcount,” it’s not wrong. But that recommendation could apply to almost any team in almost any company! It won’t make a director change their mind, and it’s not one a leadership team can debate, refine, and act on.

### What happened when I added the expert layer

Calling on the O’Reilly Expert MCP didn’t provide the model with new facts—most of the information was technically available already. However, without the Expert MCP and associated skills, the model couldn’t use that information for anything but the broadest analyses. Incorporating the Expert MCP and associated skills changed the character of the analyses by grounding them in specific frameworks, citing named authors and thresholds, and organizing their conclusions around established bodies of practitioner knowledge rather than general principles.

To make this concrete, here’s the kind of output the research assistant produced before adding the Expert MCP:

The team appears overloaded. The backlog is large and the migration project is consuming significant sprint capacity. Consider adding headcount or reducing scope.


And here’s what it produced after:

According to Google’s SRE guidance, sustained operational toil above approximately 50% indicates structural inefficiency rather than a staffing shortage. This team’s telemetry suggests approximately 67% operational toil. Hiring another engineer would likely increase total toil unless operational ownership is first reduced. Recommended actions: run a structured toil audit, set an explicit toil-reduction target below 50%, and assign runbook ownership for recurring operational tasks.


The second report cites a framework by name, references the specific threshold that framework establishes, applies it to the team’s actual data, reaches a different conclusion than the obvious one, and makes actionable recommendations. It’s the kind of analysis that changes a conversation because the director can see where the conclusions came from, engage with the reasoning, push back on the framework if they disagree, or accept it with confidence that it was reasoned rather than pattern-matched.

When I shared this version with my engineering director, their reaction was immediate: *This is defensible*.

### Frameworks aren’t facts

The most underappreciated aspect of O’Reilly’s content library is that the value isn’t primarily informational. Most of the facts in an O’Reilly book are available on the internet, and LLMs have already read much of the internet.

The deeper value of O’Reilly’s catalog is that it’s organized around *coherent frameworks*—complete mental models built by practitioners who spent years or decades developing them. Google SRE. Team topologies. *Accelerate*. Domain-driven design. *The Manager’s Path*. Wardley mapping. *Designing Data-Intensive Applications*. These are structured ways of thinking about specific classes of problems, developed with enough rigor that they can actually guide decisions.

Frameworks are distinct from facts in a critical way: They tell you not just what’s true but what’s relevant, what to measure, what threshold matters, and what to do when you exceed it. A model with access to the SRE framework as an organized body of practitioner knowledge is more likely to surface it explicitly, apply it to the specific question at hand, and use it to anchor its recommendations, producing output that human reviewers can actually interrogate.

This points to the organizing principle behind the approach described in this paper:


Organizational data provides local evidence about what is happening in your specific context. Expert frameworks provide accumulated practitioner knowledge about how to think about problems of that kind. Good organizational judgment requires both.

The Expert MCP is the bridge between your specific business context and practitioner insights. It connects the AI’s access to your internal systems with a curated body of expertise relevant to the decisions your organization needs to make.

### Why use MCP rather than uploading your own documents

The natural objection at this point is “Couldn’t I get the same effect by dumping relevant PDFs into Claude, or using Claude Projects, or NotebookLM?”

The short answer is not quite, and the reasons are practical as much as they are technical.

Uploading documents gives you retrieval from those specific documents. The O’Reilly Expert MCP differs in several operationally significant ways. First, the corpus is editorially curated around coherent practitioner frameworks. Unlike a collection of PDFs, which tends to reflect whatever you happened to find, the Expert MCP offers a sustained curatorial perspective: The authors are vetted, the content has been through editorial review, and it’s organized around established bodies of knowledge rather than assembled ad hoc. This is a much more expansive kind of evidence base. Second, the corpus is maintained and updated by O’Reilly. New titles are added, new editions replace old ones, and the content stays current without any management on your part. Third, the Expert MCP is configured once and works consistently across your entire organization and toolchain rather than being tied to a single user’s Claude Project or a document upload that expires. Finally, accessing content through a proper API respects the appropriate usage terms in a way that uploading copyrighted texts doesn’t.

And when paired with a well-written skill, the agent can be directed to look explicitly for competing frameworks, surface cases where the literature disagrees, and name gaps in the available evidence, providing a meaningful check against the common tendency of AI tools to quietly favor whatever framework first seems to fit. That’s something you can encourage with any retrieval setup, but it works more reliably when the underlying corpus is organized around coherent bodies of thought rather than a heterogeneous collection of documents.

### What we’re not claiming

I want to be clear about the limits of what Expert MCP does today. O’Reilly doesn’t claim that Expert MCP automatically selects the single correct framework for every situation, or that adding it to your configuration produces consultant-quality analysis without thoughtful prompting and human review.

The results described in this paper were the outcome of all four elements—the internal organizational data, the carefully designed skill architecture, the Expert MCP, and human review—in combination working together.

The Expert MCP is an important differentiator, but it’s not a magic layer you can add to an otherwise generic setup and expect to reproduce these results. The system works because each element does something the others cannot. The skill defines the reasoning process, the internal MCP connections provide the organizational evidence, the Expert MCP provides the expert frameworks, and human review supplies the judgment and context that no AI system can generate on its own.

What the Expert MCP reliably contributes to that system is access to a curated body of practitioner knowledge: technical and managerial frameworks that are editorially organized around coherent bodies of thought and difficult to reconstruct from scattered web content or assembled document collections. Your organizational data still tells you what’s happening, while the O’Reilly Expert MCP helps interpret what it means. That’s a meaningful and concrete improvement over an ungrounded AI assistant, and it’s something you can put in production and build on today.

### A note on hallucinations

No AI system eliminates the risk of hallucination. The Expert MCP doesn’t make the model infallible.

What it does is change the burden of proof. When every recommendation is grounded in a named framework, a named author, and a traceable citation, a human reviewer can check the reasoning rather than simply accepting or rejecting a conclusion. The question shifts from “Is this right?” (unanswerable in isolation) to “Does this framework actually say this, does it apply here, and do I agree with the conclusion?” That’s a question humans can engage with productively, which is exactly what you want from a decision-support tool.

## Step 4: Human review is nonnegotiable

Organizational systems rarely contain the full context behind a decision. The meeting that changed everything happened last Tuesday and hasn’t been written up yet. A key person is quietly planning to leave. A strategic direction shifted in a conversation that was never documented. AI can synthesize everything in your systems with remarkable fidelity, but it can’t know what isn’t there, and organizational reality changes faster than documentation does.

More fundamentally: AI can identify trade-offs, but it can’t decide which trade-offs matter. That judgment requires human knowledge of context, priorities, and risk tolerance that can’t be fully encoded in any system. The goal isn’t to remove humans from the loop but to give them better-structured input to reason from.

### Extend the expert layer by solving collaboration

As I started sharing analyses more broadly, I ran into a new set of limitations in the collaboration layer. The research assistant produced documents. I shared them in Google Docs, and people added comments, but when the AI updated a document based on reviewer feedback, I had to paste in a new version, which wiped out the existing comments. Documents proliferated without clear relationships between them, and the AI had no visibility into the discussions in the comments, which was where the most important context and pushback lived.

To solve the collaboration problem, I worked with one of our engineering directors to build what we call Superanswers, a system that uses GitHub as the source of truth for AI-generated research documents and their associated discussions.

The architecture is straightforward: Documents are stored as Markdown files in a GitHub repository, a GitHub Pages site renders them with a clean interface that supports inline commenting, and all discussion happens in GitHub Discussions, meaning every comment, question, and revision is versioned and traceable. Because the documents and their discussions live in GitHub, Claude Code has full access to both. It can read the document content plus the entire conversation that’s developed around it.


This enables a qualitatively different kind of AI participation. Instead of generating a document and stepping back, we can now ask:

What is the consensus around this project based on the discussion so far? What questions remain unresolved? Incorporate the reviewer comments and produce an updated version.


The AI becomes a participant in an ongoing conversation rather than a one-shot report generator, which meaningfully shifts how organizational knowledge gets built and refined.

### What teams are using Superanswers for

As Superanswers has spread across our engineering organization, the range of questions people bring to it has been broader than I expected:

| Theme | Typical questions | 
|---|---|
| Architecture and infrastructure | Should we make this change? What will it cost? What might break? | 
| Operational effectiveness | Where is our toil coming from? What should we automate, simplify, or retire? | 
| Team health and capacity | Where is the team’s time going? What’s limiting execution? | 
| Organization and strategy | How should we organize, prioritize, and invest? | 
| Engineering measurement | How do we know if we’re healthy and improving? | 
| AI and organizational learning | How do we build better systems for reasoning and decision-making? | 

None of these questions is about writing code. They are about understanding an organization, making decisions, and coordinating work, and most of them would map naturally onto the concerns of leaders in other functions. The same questions arise in any organization navigating rapid change with information scattered across too many places.

## How to use the recipe

The AI conversation to date has been dominated by a particular set of questions. But there are more interesting questions we should be asking.

| We’ve spent a lot of time asking… | What else might be possible? | 
|---|---|
| How do we make people more productive? | How do we make organizations more effective? | 
| How do we produce faster? | How do we make faster decisions? | 
| How do we generate output? | How do we generate understanding? | 
| How do we accelerate execution? | How do we improve outcomes? | 
| How do we gather data? | How do we build institutional knowledge? | 
| How do we automate tasks? | How do we improve organizational learning? | 

The challenges outlined in this chart aren’t unique to engineering. They exist wherever important information is scattered across multiple systems and important decisions require synthesizing all of it.

Individual productivity matters, but organizations don’t succeed by having contributors go faster in arbitrary directions. They do so by making good decisions about where to invest, allocating resources well, surfacing problems before they compound, and building institutional knowledge that persists over time.

The recipe I’ve described can help organizations make those decisions and build that knowledge.

## The recipe for building an organizational intelligence system:


Map your information hierarchy.Identify the systems where important knowledge lives in your organization, from strategic intent down to operational detail. This is an organizational task, not a technical one, and doing it well requires understanding how decisions actually get made.

Connect those systems via MCP and write a skill that describes how to reason.The MCP connections give the AI access to your data; the skill file tells it how to think with that data. Without the skill, you get retrieval. With it, you get analysis.

Add the O’Reilly Expert MCP as an expert review layer.Organizational data provides local evidence about what is happening in your specific context; expert frameworks provide accumulated practitioner knowledge about how to reason about problems of that kind. This step bridges the two. The O’Reilly library spans engineering, management, data science, security, finance, product, and more, organized not as a collection of facts but as coherent frameworks developed by practitioners who spent careers building them. The result is analysis grounded in named frameworks with traceable citations, something human reviewers can engage with and question, rather than generic advice they can only accept or reject.
Build a lightweight system for human-in-the-loop consensus.AI-generated analysis is a starting point, not an end point. You need a mechanism for people to review, challenge, and refine what the AI surfaces, one where those discussions become part of the context the AI can learn from in subsequent iterations.

The biggest practical lesson I took from this work is reframing what AI is actually for in an organizational context. The difference between a useful AI research assistant and a generic one isn’t primarily about which model you use or how much data you feed it. It’s about whether the reasoning combines local organizational evidence with established expert frameworks. Your data tells you what happened. Expert frameworks help interpret what it means. That combination, with human judgment applied at the end, is what makes the difference between a report that gets read (maybe) and filed away and a recommendation that changes a decision.
