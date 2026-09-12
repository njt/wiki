---
url: https://martinfowler.com/articles/reliable-llm-bayer.html
title: "Building Reliable Agentic AI Systems"
author: Sarang Sanjay Kulkarni (Principal Consultant, Thoughtworks)
date_fetched: 2026-06-21
date_published: 2026-06-16
publication: martinfowler.com
coauthors: Adam Zalewski, Annika Kreuchwig, Carlos Henrique Vieira-Vieira, Jobst Löffler, Jonas Münch (Bayer team); Bala Hari, Balu Saravanan (Thoughtworks team)
topics:
  - misc
---

# Building Reliable Agentic AI Systems

A field report on PRINCE (Preclinical Information Center), Bayer AG's cloud-hosted multi-agent RAG platform for pharmaceutical drug development, built with Thoughtworks. The article presents two engineering lenses: **context engineering** (how information is shaped and routed between specialized agents) and **harness engineering** (how orchestration, recovery, and observability are built around models).

## The Problem

Bayer researchers faced three core challenges in preclinical drug development:
1. **Data Silos** — information fragmented across disparate systems
2. **Limited Search** — keyword search couldn't handle complex preclinical terminology
3. **Manual Analysis** — extracting insights across documents diverted time from science

## PRINCE Evolution (Three Phases)

| Phase | Description |
|-------|-------------|
| **Search** | Unified gateway consolidating siloed structured study metadata |
| **Ask** | AI-powered Q&A using RAG over unstructured data including scanned PDFs |
| **Do** | Multi-agent system for complex queries and regulatory document drafting |

Key insight: structured metadata was often corrupted by system migrations, but "the authoritative 'gold standard' information was consistently present within the approved PDF study reports."

## Architecture

- **Orchestration**: LangGraph, served via FastAPI
- **Frontend**: React conversational UI
- **Vector store**: OpenSearch for knowledge base of study reports
- **Structured data**: Amazon Athena
- **State persistence**: PostgreSQL (LangGraph checkpointer) for agent state; DynamoDB for application state
- **LLMs**: Internal GenAI platform hosting models from OpenAI, Anthropic, Google, and open-source, via unified OpenAI-compatible endpoint

### Resilience Features
- Automatic retries on LLM failure, then fallback to alternative model/provider
- Retries at both individual call level and logical node level
- Agents receive error context to chart alternative trajectories
- User-initiated retries resume from persisted state, skipping completed steps

### Observability
- CloudWatch for system health
- Langfuse for detailed traces, evaluation datasets, debugging
- RAGAS evaluation framework for daily live traffic evaluation and dataset evaluation on significant changes

### Context Discipline
A core design principle: "different stages receive different context" — planning context for Think & Plan, retrieval context for Researcher, evidence context for Reflection Agent, synthesis context for Writer. This "reduces context pollution and makes the system easier to debug, evaluate, and improve."

## The Agentic RAG System

### 1. Clarify User Intent
First defense against ambiguity. Rather than expensive trial-and-error, asks clarifying questions to pinpoint domains (toxicology, pharmacology). Provides AI-assisted source recommendations; user retains final control. Constrains "which tools, domains, and data sources will be in scope before any retrieval begins."

### 2. Think & Plan: Process Reflection
A dedicated reasoning step before action, inspired by "Anthropic's Think tool." Performs **process reflection** — evaluating trajectory and progress, not data itself. When PRINCE's toolset grew from RAG+Text-to-SQL to many more, the LLM struggled with tool selection; adding this step "led to a dramatic improvement in the accuracy of tool selection."

### 3. Researcher Agent

#### RAG for Unstructured Data (8-step pipeline)
1. **Keyword Extraction** — LLM extracts search-relevant keywords
2. **Metadata Filter Generation** — LLM generates dynamic filters via few-shot prompting
3. **Query Expansion** — Smaller model generates n=5 semantically similar queries
4. **Hybrid Retriever** — Metadata filtering + semantic vector similarity (kNN) + keyword search in OpenSearch. Weighted scoring: 0.7 semantic, 0.3 keyword. Aggregates ~20 chunks from parallel searches.
5. **Reranking** — Cross-encoder model (bge-reranker-large) selects top 7 chunks
6. **Final LLM Prompt Generation** — Context + original question
7. **Response Generation with Citation** — Reasoning model with citations to chunks, page numbers, quotes
8. **Monitoring** — via Langfuse

**Ingestion Process:** PDFs → S3 data lake → extraction pipeline → structured JSON → chunking → enrichment with metadata from Athena (study ID, compound, species, route, page, parent section) → embedding and indexing in OpenSearch.

#### Text-to-SQL for Structured Data
1. Query analysis and intent recognition
2. Dynamic schema selection — only relevant schema injected
3. Dynamic few-shot prompting — relevant SQL examples from vector "semantic layer"
4. SQL generation with always-included columns (study ID, title)
5. Validation — only SELECT allowed; DELETE/INSERT/UPDATE blocked
6. Execution with 50-record limit
7. Error handling — up to 3 self-correction attempts with error messages fed back

Notably: an earlier LLM review step for SQL was removed because "the reviewing LLM sometimes incorrectly flagged valid queries as erroneous."

### 4. Reflection Agent: Data Validation
Performs **data reflection** (vs. process reflection): evaluates whether retrieved data is sufficient and relevant. If insufficient, generates "specific follow-up questions" that feed back to Think & Plan for further retrieval. Creates "an iterative process of data validation and subsequent information retrieval."

Key distinction: "an agent might execute a perfectly valid workflow (good process) but still retrieve insufficient data to answer the question, or conversely, might have access to sufficient data but fail to progress effectively."

### 5. Writer Agent: Answer Synthesis
Turns raw material into final answers. Non-negotiable rules:
- Every claim grounded in supplied context with accurate citations
- User-level formatting requirements honored
- Alignment with domain-specific answer standards
- Internal review loop: draft → check for missing sections → revise
- All regulatory drafting outputs intended for expert review

### Three Reflection Loops Summary

| Loop | Checks | Catches |
|------|--------|---------|
| **Process reflection** | Whether workflow is on the right path | Bad trajectory, wrong tool choice, poor sequencing |
| **Data reflection** | Whether gathered evidence is sufficient | Thin evidence, missing context, coverage gaps |
| **Draft reflection** | Whether output is complete | Missing sections, incomplete tables, synthesis gaps |

## Building Trust

- **Transparency**: Intermediate steps displayed — queries formulated, tools used, sources linked
- **Granular citations**: Users hover over any sentence to see citation, linking to source document including "page number and the exact quote from the report"
- **Evaluation**: Dataset evaluations on significant changes (Faithfulness, Answer Relevancy, Context Relevancy, Answer Accuracy, Semantic Similarity). Live traffic evaluations daily on real queries.
- **Human in the loop**: All regulatory outputs intended for expert review; final submissions authored and approved by qualified personnel

## Error Handling and Recovery

- Agent State in Postgres; application state (logs, intermediate steps, citations) in DynamoDB
- Framework-level support via LangGraph's built-in state and error management
- User-initiated retries: system "leverages the persisted state to continue the workflow directly from the point of failure, intelligently skipping the steps that were successfully completed in the previous attempt"

## NER Annotation System

A utility system uses Named Entity Recognition to extract accurate annotations from study PDFs to correct incomplete/corrupt metadata. High-confidence fields auto-update Athena; low-confidence fields are quarantined for human review.

## Iterative Development

PRINCE has been available to end-users since early 2024, with agentic integration added later that year. Key principle: "we don't wait for features to be absolutely perfect before seeking user feedback." Initial focus on accuracy and performance even at higher cost; cost optimization came after quality targets.

## Key Takeaway

> "production-ready agentic AI is not only about better models or better prompts. Reliability comes from engineering both the context the model sees and the harness within which the model acts."

As model capabilities improve, "some parts of today's harness may become thinner or move into native model capabilities. But in enterprise research systems, especially where trust, traceability, and reviewability matter, explicit control over context, workflow state, recovery, reflection, and verification remains essential."

## Related Publications

- Peer-reviewed paper in *Frontiers in Artificial Intelligence* (Volume 8, 2025): "From data silos to insights: the PRINCE multi-agent knowledge engine for preclinical drug development"
