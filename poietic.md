# poietic

Vaughn Tan and Erik Garrison's vehicle for making human-machine collaboration legible. Their core tool, wg, is a Rust-based task coordination system where humans and AI agents work within a shared dependency graph -- claims, handoffs, execution logs, and completions all tracked in one inspectable structure. The thesis: AI can now do the work, but the durable problem is organizational -- how do you coordinate people and machines across extended timeframes while preserving judgment and exposing evidence?

---

## Key Quotes

> "AI agents can now search literature, write code, analyze data, and design molecules. But the durable problem is organizational."

> "Tool work, research practice, and organizational design should all make human and machine collaboration legible and responsive to its participants."

## Key Themes

#coordination #human-machine #workflow #transparency #organizational-design

The three-layer model (tool, practice, institution) is more thoughtful than most agent framework designs. wg isn't just software; it's embedded in a methodology ("graph work") and an organizational form (public-benefit corporation that eats its own dogfood). Every process -- including incorporating the company itself -- runs through wg and is publicly inspectable.

The dependency graph approach to task coordination is a different paradigm from the planner/worker/judge pattern in [[Scaling Long-Running Agents]]. Cursor's agents coordinate through hierarchical decomposition; wg coordinates through explicit dependency tracking. The wg approach is more transparent and auditable, which matters for research and compliance-heavy domains.

The founders bring unusual credibility: Garrison built pangenome graph tools (the Human Pangenome Reference Consortium), Pinello builds genome editing tools (CRISPResso), and Tan studies organizational behavior. This isn't a startup selling agent infrastructure; it's researchers building tools they need for their own work.

## Critical Analysis

The public-benefit corporation structure and the commitment to radical transparency are admirable but constrain growth. Most organizations won't expose their work processes publicly. The immediate applications (grant writing, genomic analysis) suggest this is pitched at academia and research labs, not enterprise.

The "graph work" methodology is the most interesting and least developed part. Dependency graphs for task coordination are well-understood in project management (see: every build system ever), but extending them to human-agent hybrid workflows raises new questions: Who resolves blocked dependencies? How do you handle the asymmetry between human and agent work speeds? What happens when an agent's output doesn't meet the quality bar for a handoff?

wg being MIT-licensed and Rust-based is a good foundation for adoption, but the coordination methodology might be harder to export than the software.

---
*Sources: [[raw/poietic]]*
*Last updated: 2026-05-14*
