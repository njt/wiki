# AI-Assisted Database Work — The Machine Reads, The Human Decides

Pinal Dave's field report from nine consulting engagements where AI was deployed not to write SQL, but to read, classify, and extract from existing database artifacts at volume — with human verification as the structural keystone. The thesis: these aren't hard problems; they're tedious ones. The machine makes starting cheap; the human makes the output trustworthy. Invert that order and "the project produces a beautiful document full of confident fiction."

---

## Key Quotes

> "The most satisfying work of my career has not been tuning a query. It has been walking into a room where a team is quietly terrified of their own database, and walking out three days later with them arguing confidently about it."

The emotional core that Dave returns to throughout: the real output isn't a document or a test suite — it's a team that stops being frightened of the old system. "Being able to argue about your own database is an underrated form of wealth." This is a profoundly non-technical metric for technical work, and it's more honest than most.

> "Feeding whole packages produced vague summaries. We got useful output only after splitting by procedure and passing the referenced table DDL alongside the code."

The most actionable technical detail in the piece. Context about the *data* — that `P_CUST.TIER_CD` is a three-character code with a check constraint, not free text — changed reasoning quality more than any prompt wording. This is a concrete instance of [[Agent Memory and Context]]'s thesis that context engineering, not prompt engineering, is the real lever.

> "Requiring the reason changed everything. A match justified by 'both are named CUST_STATUS' is a weak match and now it looks weak on the page."

The M&A schema mapping insight applies far beyond databases. Forcing the AI to articulate *why* it made a match turned a confidence score (always high, always useless) into a triageable artifact. The reason column is what let humans process 900 columns in a week and catch that one company's Active was the other's Archived. This is the same pattern as [[AI Code Migration with Claude Code]]'s "make review adversarial and verification mechanical" — structure the output so that failures are visible, not buried in a number.

> "It knew the mechanism. It did not know the blast radius."

Dave's one-line summary of the entire technology, from the query plan regression case. The AI correctly identified that a cardinality estimator change caused join-order shifts, and correctly listed the options. It then recommended a database-wide legacy setting that would have fixed six queries and pessimized several hundred others. This is the central tension of AI-assisted work: the model understands *what* but not *how much*.

> "Not one of them is a hard problem. Every single one is a large, tedious, low judgment reading task that a competent person could do perfectly, given three months and no interruptions, which is a resource that has never existed."

The unifying thesis. These projects weren't blocked on skill — they were blocked on tedium. Reading 800 packages, 240 job definitions, 900 column pairs, 300 pages of compliance prose. The work was always possible; it was just never worth starting. What changed is that starting got cheap.

> "This is not a story about AI understanding your database, it is a story about AI reading it fast enough that you finally can."

The closing sentence, and the best summary of what distinguishes Dave's approach from the "AI will replace DBAs" narrative. The machine doesn't understand — it reads. Understanding remains the human's job.

## Key Themes

- **#pattern — Machine reads, human decides**: Every one of the nine cases follows the same architecture. AI reads at volume and proposes; a human verifies and decides. The moment anyone inverted that order, the project produced confident fiction. This is [[Smart Models Dumb Pipes]] applied to database work — the model owns the reading/classification layer, deterministic verification owns the decision layer, and the boundary is where the value lives.

- **#pattern — Structure the output for human triage**: The M&A mapping's "reason column," the compliance suite's section-number traceability in comments, the job graveyard's four-bucket classification — in each case, the AI's output was structured so a human could efficiently find the parts worth arguing about. "The machine narrows 900 columns to 40 worth arguing about, and the humans argue about the right 40."

- **#concept — Verification is not optional, and it's the part everyone wants to skip**: Dave repeats this across multiple cases. Every business rule verified against real data. 340 columns verified by changing one value in the application and watching which column moved. 19 dialect-drift test failures caught before they reached customers. The verification step is where the value is; skipping it is where the disasters live.

- **#concept — Context about the data beats clever prompts**: The most actionable finding across all nine cases. Feeding DDL, value distributions, FK graphs, and Extended Events traces alongside code produced dramatically better results than prompt iteration. This is a concrete validation of the context-engineering-over-prompt-engineering thesis that runs through [[Agent Memory and Context]] and [[New Rules of Context Engineering]].

- **#concept — The tedium bottleneck**: Dave names something that applies far beyond databases. Organisations are full of "possible but never worth starting" work — reading, classifying, mapping, auditing — that AI makes cheap to begin. The [[Software Engineering Craft]] implications are significant: a large fraction of what we call "legacy risk" is actually just "nobody had three uninterrupted months to read it."

- **#tool — SQL Server system views as AI feedstock**: `sys.sql_modules`, `sys.foreign_keys`, `sys.dm_sql_referencing_entities`, `msdb.dbo.sysjobsteps`, Extended Events — Dave treats these not as DBA tools but as structured input streams for AI. The pattern of assembling exactly the right system metadata per problem is transferable to any database platform.

## Critical Analysis

**Dave's piece is the most useful kind of AI field report: specific enough to steal from, honest enough to trust.** Every case names the failure mode alongside the success. It will state intent it cannot know (case 1). It will commit to one meaning of an ambiguous abbreviation with total confidence (case 3). It knows the mechanism but not the blast radius (case 8). It will happily propose queries for untestable requirements (case 5). This is not the usual "AI is amazing" blog post — it's an engineer documenting what worked, what broke, and what guardrails made the difference.

**The article is a direct counterpoint to [[Text-to-SQL in the Real World]]**, but not a contradiction. Stonebraker and Chen showed that LLMs fail catastrophically at *generating* queries for real enterprise schemas. Dave shows they succeed at *reading* them — not because they understand the schema, but because reading at volume is what language models do. The two pieces together draw a boundary: AI for database *generation* is not ready; AI for database *reading and classification* is here. The distinction matters enormously for anyone deciding where to invest. [[DeepSQL]] complicates the boundary slightly: it ships both capabilities in one product — fenced generation (EXPLAIN-validated, schema-whitelisted, read-only) alongside reading at volume — but still keeps a human as the only path to mutation.

**Compare with [[AI Code Migration with Claude Code]].** Both are about using AI on large-scale codebase work, but the architectures are different. Anthropic's approach is a rulebook-driven factory: fix the loop, regenerate, throw away the first output. Dave's approach is a reading-and-triage engine: AI proposes, human decides, nothing is thrown away because nothing was generated — it was extracted. The two approaches aren't competitors; they solve different problems. Rulebook-driven generation works when the target is known (Rust, TypeScript). Reading-and-triage works when the target is understanding (what does this code *mean*, what is this column, what does this job do).

**The "blast radius" problem is the deepest insight and the least developed.** Dave's example from case 8 — the AI recommended a database-wide legacy cardinality setting — is a specific instance of a general failure mode: AI can diagnose a local problem correctly and propose a local fix, but it cannot assess the systemic impact of applying that fix globally. This is the same problem that makes [[Constraint Decay]]'s findings so damning: every additional constraint (database, architecture, ORM) compounds the failure rate. The solution Dave arrives at — Query Store plan forcing for specific queries — is an instance of the general principle: contain the fix to the smallest possible scope. But he doesn't generalize the principle, and this is the article's one missed opportunity.

**What's quietly radical: the consulting model.** "None of these clients bought a product. There was no platform, no license, no vendor with a booth at the conference." The deliverable was a workflow, a verification step they weren't allowed to skip, and a written note about failure modes. Their own people run it all now. "I am not needed for any of it, which is exactly how a consulting engagement is supposed to end and almost never does." This is an honest description of what good AI consulting looks like: transfer the capability, document the failure modes, leave. The opposite of the vendor playbook.

**The article's limitation is that these are all one-off projects built by a deeply experienced SQL Server consultant.** Dave's 20+ years of SQL Server expertise is the hidden ingredient — he knows which system views to query, which Extended Events to capture, which DDL to feed alongside the code. Replicating any of these nine patterns requires someone who knows both the domain and the tool. The AI reduces the tedium; it doesn't replace the expertise. This is consistent with [[Laura Tacho — Data vs Hype]]'s finding that AI is an accelerator, not a fixer — it makes good teams better and dysfunctional teams worse.

---

## Connections

- [[Text-to-SQL in the Real World]] — The complementary finding: AI fails at generating queries for real schemas but succeeds at reading and classifying them. Together they bound the useful surface area.
- [[Bad Data in Production — Response Playbook]] — Another Pinal Dave piece on database operations. The new article extends his thinking from reactive data quality to proactive AI-assisted database understanding.
- [[AI Code Migration with Claude Code]] — A different AI-assisted migration architecture (rulebook-driven factory vs. reading-and-triage engine). Complementary approaches for different problems.
- [[Smart Models Dumb Pipes]] — Dave's "machine reads, human decides" is the smart-models-dumb-pipes pattern applied to database work. The model owns judgment; deterministic verification owns the decision.
- [[The Archaeologist's Copilot]] — Malykhin's "Tourist vs. Archaeologist" prompt distinction is the same structural insight as Dave's "reading at volume with mandatory human verification."
- [[Databases and Data]] — The hub page. Dave's article fills the "database migration patterns" gap with concrete, battle-tested workflows.
- [[Software Engineering Craft]] — The tedium-bottleneck thesis: a large fraction of legacy risk is just "nobody had three uninterrupted months to read it."
- [[Guardrails and Feedback Loops]] — Dave's mandatory verification step is a guardrail. The pattern of "disable with a note, delete after 90 quiet days" is a feedback loop.
- [[Why Are Databases So Hard]] — Dave's migration dialect-drift case (case 9) is the practical complement to the physics-of-consistency argument: even when row counts match, NULL ordering and collation silently produce wrong answers.
- [[SQL Pagination — Offset vs Seek Method]] — Winand's pagination patterns are the kind of non-obvious database expertise that Dave's AI-reading approach could surface from existing code (find every `OFFSET` query and flag it for seek-method conversion) but couldn't generate from scratch without explicit instruction. The "machine reads, human decides" architecture applied to query pattern auditing.
- [[Object-Relational Impedance Mismatch]] — The underlying structural problem: the nine fracture lines (type, transaction, identity, etc.) between OO and relational models that make "reading" a database schema tractable for AI but "generating" against it reliably out of reach. The machine can classify what exists; bridging the gap to produce correct new queries requires crossing the impedance boundary.
- [[Analytical AI]] — Sutro's term for exactly the work Dave's nine engagements perform: foundation models processing unstructured data into structured decisions ("machine reads, human decides"), where tasks are discriminative and measurable against expert-annotated ground truth. Dave's "the machine doesn't understand — it reads" is analytical AI's core assumption stated plainly.

---
*Sources: [[raw/nine-unusual-ways-my-clients-use-ai-with-sql-server]], [[summary/nine-unusual-ways-my-clients-use-ai-with-sql-server]]*
*Last updated: 2026-08-07*
