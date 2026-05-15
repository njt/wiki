# Gambit

An agent evaluation framework from Bolt Foundry that generates synthetic test scenarios, runs agents against them, grades behavior from traces, and converts failures into regression suites. Think of it as a test harness purpose-built for the non-deterministic world of AI agents, where you need to test behavior rather than exact outputs.

---

## Key Quotes

> Gambit is "the test-data engine, grader loop, local reproduction harness, and CI behavior check" for agent systems.

## Key Themes

#evals #agents #testing #harness #CI-CD

Gambit occupies the space between "run the agent and hope" and the comprehensive eval framework described in [[Demystifying Evals for AI Agents]]. It's practical tooling: generate scenarios, run agents, grade outputs, feed failures back into regression suites. The PR gate workflow -- run behavior checks on every pull request -- is where this becomes genuinely useful for teams shipping agents.

The three agent definition patterns (markdown-based, TypeScript compute, composite) reflect the reality that agent systems are heterogeneous. You need to test model-powered agents, deterministic code paths, and composite orchestrations differently.

The bring-your-own-agent support (Mastra, LangGraph, OpenAI) is smart -- it means Gambit doesn't lock you into its own agent framework. You can use it purely as an evaluation layer over whatever you're already building.

## Critical Analysis

The framework is TypeScript-first, which limits its appeal for Python-heavy ML teams. The dependency on OpenRouter for model-powered agents is pragmatic but adds a third-party dependency to your CI pipeline.

The scenario generation approach -- synthetic rather than production-derived -- is both a strength and a weakness. Synthetic scenarios are reproducible and controllable, but they miss the long tail of weird real-world inputs. The best eval suites (as Anthropic notes in [[Demystifying Evals for AI Agents]]) start from real failures, not synthetic ones.

Still, this fills a real gap. Most teams building agents have no test infrastructure at all. Having something that can run in CI and catch behavioral regressions is vastly better than nothing.

---
*Sources: [[raw/gambit]]*
*Last updated: 2026-05-14*
