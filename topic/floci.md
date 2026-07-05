# floci

A free, open-source local AWS emulator that replaces LocalStack Community Edition (which sunset in March 2026). Starts in 24ms, uses 13 MiB idle memory, supports 47 AWS services, and stays MIT-licensed forever. "Light, fluffy, and always free."

---

## Key Quotes

> "Wire-compatible behavior against the actual engine, not a simplified approximation."

## Key Themes

#tool #aws #local-development #testing #open-source

The timing is significant: LocalStack Community died in March 2026 (requiring auth tokens, freezing security updates), creating a vacuum for a truly free local AWS emulator. Floci fills that gap with dramatically better performance: 24ms startup vs 3.3 seconds, 13 MiB memory vs 143 MiB, 90 MB Docker image vs 1 GB.

The three-tier architecture is smart: stateless services (IAM, SQS, SNS, KMS) run in-process for speed, stateful services (S3, DynamoDB) use configurable storage backends for flexibility, and container services (Lambda, RDS, EC2) use real Docker containers for fidelity. The claim of "wire-compatible behavior against the actual engine" is ambitious -- LocalStack made similar claims and fell short on edge cases.

47 services including API Gateway v2, Cognito, ElastiCache, RDS, MSK, and Athena (backed by DuckDB) put it well beyond what LocalStack Community offered.

## Critical Analysis

The big question is production fidelity. Local AWS emulators are notorious for passing tests locally and failing in real AWS. The "real Docker containers" approach for Lambda, RDS, and EC2 with actual IAM/SigV4 authentication is the right design, but the devil is in behavioral edge cases.

The four storage modes (memory, persistent, hybrid, WAL) show thoughtfulness about CI/CD vs development use cases. Memory mode for fast test suites, persistent for development workflows, WAL for when you care about durability.

For anyone running agent-driven development with AWS dependencies, this is essential infrastructure. Agents can't test against real AWS without cost and latency penalties, so a fast local emulator is a force multiplier.

See also [[Windows in Docker]] for another approach to running complex environments locally for agent testing.

---
*Sources: [[summary/floci]]*
*Last updated: 2026-05-14*
