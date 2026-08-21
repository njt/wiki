# Organizational Intelligence Systems

A four-step recipe for building AI systems that produce actionable, defensible organizational analysis by combining internal data with expert practitioner frameworks — as developed and deployed at O'Reilly's own engineering organization.

---

## Key Quotes

> "It told me *what was happening* without helping me understand *why*, or what I should actually do. It was organized around the data rather than around the decision."

The diagnostic that launched the whole investigation. An AI agent connected to internal systems produced a thorough report — headcount, velocity, backlog — that was "easy to agree with and difficult to act on." This is the organizational equivalent of the generic-analysis problem: technically correct, broadly applicable, and useless for actually changing anyone's mind. The fix wasn't better data or a better model — it was grounding the analysis in named expert frameworks.

> "The operational overhead problem is structural, not a staffing deficiency."

The output after adding the O'Reilly Expert MCP. Citing Google SRE's toil threshold (~50%), the system identified that the team was at ~67% operational toil and recommended a toil audit and structural intervention rather than hiring. The key property: the recommendation was *traceable* — the director could see where the conclusion came from, engage with the reasoning, and push back on the framework if they disagreed. "This is defensible," was the director's reaction.

> "MCP gives the agent access to your data, but it doesn't tell the agent how to use it effectively. Without explicit guidance, the agent retrieves information and organizes it the way the underlying systems organize it, which produces a data dump, not an analysis."

The part most implementations get wrong. MCP connections without a skill file give you retrieval; the skill file (a CLAUDE.md or similar) transforms retrieval into analysis by encoding your organization's reasoning process — which systems to consult for which questions, how to weigh conflicting sources, what output format to use, epistemic standards for surfacing assumptions. This maps directly onto [[Claude Code Mastery]]'s insight that CLAUDE.md is "compounding infrastructure" and [[Steering Claude Code]]'s taxonomy of instruction-delivery mechanisms.

> "Organizational data provides local evidence about what is happening in your specific context. Expert frameworks provide accumulated practitioner knowledge about how to think about problems of that kind. Good organizational judgment requires both."

The organizing principle. It's a clean separation of concerns: your data tells you *what happened*; expert frameworks (Google SRE, Team Topologies, *Accelerate*, Wardley mapping) tell you *what it means*. Neither is sufficient alone. This echoes [[Smart Models Dumb Pipes]]' end-to-end principle applied to organizational analysis — the model owns the judgment, the data infrastructure owns the evidence, and neither substitutes for the other.

> "The AI becomes a participant in an ongoing conversation rather than a one-shot report generator, which meaningfully shifts how organizational knowledge gets built and refined."

On Superanswers, the GitHub-based collaboration system that stores AI-generated documents as Markdown with inline discussion in GitHub Discussions. Because the AI has access to both the documents and the conversation around them, it can answer questions like "What is the consensus around this project based on the discussion so far?" This is AI participating in organizational sense-making, not just dumping reports into a Google Doc. The architecture echoes [[Team-Wide Agentic Harness]]'s argument for version-controlling agent infrastructure as team infrastructure.

## Key Themes

- **#pattern Data + Frameworks = Defensible Analysis**: The core recipe. Organizational data alone produces generic reports; expert frameworks alone produce context-free advice. Together, with the data anchoring the analysis in local reality and the frameworks providing interpretive structure, the output becomes something a human can interrogate, challenge, and act on. This is a concrete implementation of the principle that [[Loop Engineering]] names: designing the system that produces analysis, not producing analysis directly.

- **#pattern Information Hierarchy as Decision Map**: The act of mapping which systems carry which kinds of knowledge (strategic → operational) is "an organizational task, not a technical one." The hierarchy is the map of how decisions get made — which sources carry authority, which types of questions each layer answers, and what to do when sources conflict. This is the explicitly organizational dimension that purely technical MCP integration misses.

- **#pattern Skill Files as Codified Reasoning**: The skill file doesn't just list tools — it encodes the organization's reasoning process. It defines the epistemic standards (show your work, name gaps, surface assumptions), output formats, tone ("be curious, not judgmental" — borrowed from Ted Lasso), and source priority. This is an organizational capability, not a technical configuration. Once written, it makes the reasoning process repeatable across questions and teams.

- **#tool O'Reilly Expert MCP**: An MCP server providing access to O'Reilly's editorially curated corpus of practitioner frameworks. The value proposition: the corpus is organized around *coherent frameworks* developed by practitioners over years, not scattered facts; it's maintained and updated by O'Reilly; it works across the organization rather than being tied to a single user's document upload; and it respects copyright through proper API access. The claim is not that it automatically selects the right framework — it's that it provides a curated evidence base that's difficult to reconstruct from web content alone.

- **#concept Superanswers**: GitHub-as-collaboration-layer for AI-generated organizational analysis. Markdown documents + GitHub Discussions = versioned, traceable, AI-accessible conversation. This solves the Google Docs problem where pasting a new AI revision wipes out comments, and the AI can't see the discussion. Because the documents and their discussions live in git, the AI can read both and participate in an ongoing conversation rather than producing one-shot reports.

- **#pattern Human-in-the-Loop as Judgment Layer, Not Approval Layer**: The article is clear: organizational systems rarely contain the full context (the meeting that changed everything hasn't been written up; the key person planning to leave hasn't told anyone). Human review supplies the context and judgment that no AI can generate. But critically, the AI's role is to produce *better-structured input* for human judgment — grounded, traceable, framework-anchored recommendations that humans can engage with and question, rather than generic advice they can only accept or reject. This is a more constructive framing than [[Human-in-the-Loop is Tired]]'s diagnosis of supervision fatigue — the article is describing a system designed to make the human's judgment work *easier*, not just more frequent.

## Critical Analysis

**What's genuinely new here.** The article's core contribution is the *integration pattern*: MCP-connected internal data + skill-file reasoning + expert-framework grounding + human review. None of these pieces is novel alone, but the article makes a compelling case that they compound — each element does something the others can't, and the system breaks if any piece is missing. The "expert layer" as a distinct architectural component (not just "upload some PDFs") is the most interesting claim, and the before/after comparison of AI outputs makes it concrete rather than theoretical.

**What's undersold.** The article presents a success story from someone who has both access to O'Reilly's entire content library and the engineering resources to build MCP connectors to five internal systems. The "map your information hierarchy" step is described as "an organizational task, not a technical one," but anyone who's tried to get a consistent picture across Jira, GitHub, and Datadog knows it's both — and the technical part is often the harder one because the data models don't align. The article's recipe assumes a level of internal system integration that most organizations don't have.

**The expert layer question.** The article makes a strong case for curated, framework-organized content over ad-hoc document uploads, but it's also a product pitch for O'Reilly's platform. The honest admission that "the results described in this paper were the outcome of all four elements in combination" is welcome, but the practical question for someone without an O'Reilly subscription is: can you approximate the expert layer with well-chosen books and papers uploaded to a project? The article says "not quite" and gives reasons (editorial curation, maintenance, organization-wide consistency), but doesn't engage with the counter-argument that a carefully selected set of 5-10 foundational texts might get you 80% of the way.

**The collaboration problem is real but underdeveloped.** The Superanswers description is interesting but thin — it's a GitHub repo with Markdown files and Discussions. The real insight (using GitHub as the source of truth so AI can read both documents and discussion context) is strong, but the article doesn't address the adoption problem: GitHub Discussions is not where most directors and VPs naturally go to review and comment on organizational analysis. The social friction of getting non-engineers into GitHub is a significant barrier the article doesn't acknowledge.

**The hallucination framing is sharp.** "The question shifts from 'Is this right?' (unanswerable in isolation) to 'Does this framework actually say this, does it apply here, and do I agree with the conclusion?'" — this is the best single sentence in the article. It reframes the hallucination problem from a liability to be eliminated (impossible) to a burden-of-proof shift (practical). When every recommendation cites a named framework and author, the human reviewer has something to check. This is a genuinely useful way to think about AI reliability in decision-support contexts.

**Connection to the broader agentic landscape.** The article sits at an interesting intersection. It's about using AI for organizational decision-making (not code generation), which is an underexplored application relative to the flood of coding-agent content. But it uses the same primitives the coding-agent world has developed — MCP, skill files, CLAUDE.md, human-in-the-loop review, GitHub-based collaboration — just applied to a different domain. The recipe generalizes to any function (sales, legal, finance) where information is scattered across systems and decisions require synthesis. This domain-transfer argument is the most scalable contribution. [[CEOS (Claude + EOS)]] pushes the same recipe one level deeper — grounding a named practitioner framework (EOS) in skill files and markdown to *run* a company's weekly L10 meetings and quarterly Rocks, not just analyze them.

---

*Sources: [[raw/building-organizational-intelligence]], [[summary/building-organizational-intelligence]]*
*Last updated: 2026-08-07*
