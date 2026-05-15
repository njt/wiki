# Data Engineering for Large Models

A comprehensive open-source textbook covering the complete data engineering pipeline for LLMs: from pre-training data cleaning through multimodal alignment, RAG retrieval, synthetic data generation, and enterprise DataOps. 28 chapters, 10 capstone projects with runnable code.

---

## Key Quotes

> "Data is the new oil, but only if you know how to refine it."

## Key Themes

#data-quality #knowledge-management #agent-architecture

This book fills a genuine gap. Most LLM resources focus on model architecture and training techniques; the data pipeline that feeds those models gets hand-waved as "just clean your data." This book treats data engineering as a first-class discipline with its own architecture, algorithms, and production concerns.

The coverage is broad: Common Crawl extraction, deduplication, decontamination, tokenization (pre-training); image-text alignment, video/audio processing, cross-modal fusion (multimodal); SFT instruction design, preference data construction, chain-of-thought engineering (fine-tuning); seed generation, knowledge distillation, quality control against model collapse (synthetic data); RAG pipeline architecture (application layer); and DataOps flywheel, versioning, experiment tracking, privacy-preserving techniques (enterprise).

The modern tech stack (Ray, Spark, DVC, Airflow, Milvus, Federated Learning) reflects real production choices rather than toy examples.

Connects to [[Correct by Construction]] (data quality philosophy that applies to training data), [[GraphRAG]] (RAG pipeline architecture), and [[Dolt]] (data versioning for experiment tracking).

## Critical Analysis

The scope is ambitious -- 28 chapters across 10 parts is a lot of ground. The risk with comprehensive references is that they become too broad to be deep. But the 10 capstone projects with runnable code suggest practical depth. The trilingual support (Chinese, English, Japanese) reflects the global nature of LLM development. The "data-centric AI" framing throughout is the right lens -- data quality improvements consistently outperform model architecture improvements at the margin, yet most engineering effort goes into the model side. This book pushes back on that imbalance.

---
*Sources: [[raw/data-engineering-for-large-models]]*
*Last updated: 2026-05-14*
