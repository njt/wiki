---
title: "floci"
url: https://github.com/hectorvent/floci
date_fetched: 2026-05-14
section: "Random"
---

# Floci: Free Local AWS Emulator

Free, open-source local AWS emulator. Alternative to LocalStack Community Edition (which sunset in March 2026, requiring auth tokens and freezing security updates).

Performance vs LocalStack Community:
- Startup: ~24ms vs ~3.3 seconds
- Idle memory: ~13 MiB vs ~143 MiB
- Docker image: ~90 MB vs ~1.0 GB

Supports 47 AWS services including API Gateway v2, Cognito, ElastiCache, RDS, MSK, Athena with DuckDB -- many unavailable in LocalStack Community.

Architecture:
- Stateless services (IAM, SQS, SNS, KMS) run in-process
- Stateful services (S3, DynamoDB) with configurable storage backends
- Container services (Lambda, RDS, EC2) use real Docker containers with IAM/SigV4 auth

"Wire-compatible behavior against the actual engine, not a simplified approximation."

Four storage modes: memory (default), persistent, hybrid, write-ahead logging.

MIT licensed, perpetually free.
