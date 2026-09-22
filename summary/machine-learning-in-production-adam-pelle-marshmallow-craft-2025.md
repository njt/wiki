---
url: https://gist.github.com/11c02e99ecb56a27626abd5af023d8c3
title: "Machine Learning in Production — Adam Pelle (Marshmallow), Craft 2025"
author: Adam Pelle
date_fetched: 2026-09-13
date_published: 2025
topics:
  - software-engineering-craft
  - databases-and-data
---

A ytx gist carrying the digest and full transcript of Adam Pelle's (staff engineer, Marshmallow insurance) Craft 2025 talk on the operational side of machine learning — "how we keep the lights on" for 14 SageMaker online-inference endpoints hosting ~25 models across claims, fraud, and pricing (busiest endpoint ~30 req/s at P99), with a goal of 100+ models in every major business domain.

The central reframe: real-time insurance intelligence (risk scoring, fraud detection, dynamic pricing) turned the old offline analytics loop — services → Snowflake → data scientists → insights — into a cycle, because "operational data equals to analytical data" once models consume the pipeline. Best-effort emission from services ("if it's sent, it's sent, if not, that's life") is dead: a missing field no longer ruins a dashboard, it degrades a live prediction. Data quality becomes the binding constraint ("garbage in, garbage out"), and the tension between services' need-to-know data posture and models' appetite for data "requires clear governance."

The engineering program treats ML endpoints exactly like microservices — latency SLAs, high availability, the four golden signals, Python project templates on a shared Docker base image, CI/CD, canary releases via SageMaker production variants — then adds ML-specific machinery: a Tecton feature store (versioning, cross-team consistency, automated backfill; features classed as on-demand, aggregational, or periodically updated), drift monitoring (data drift vs concept drift) with PSI and Kolmogorov–Smirnov tests and proactive scheduled retraining, bidirectional Pact contract testing (WireMock consumer pacts validated by a broker against Pydantic-generated OpenAPI contracts), Schemathesis/Dredd schema verification (including the decimals-as-strings saga), and backtesting — replaying historical production inputs through a PR-branch Docker image, re-scored with the current production model state, to confirm expected outcomes before release.

The honest core comes in Q&A: the biggest challenge was timing — models shipped "without any sort of considerations just to really get to the market very fast," the failures were eaten, and only then was an MLOps team and roadmap assembled. The gist's own digest flags what the talk skips: delayed ground truth, the feedback loop where models shape their own future training data, adversarial drift, prediction-quality evaluation during canary, what the promised "governance" actually is, and regulation invoked without explainability.
