---
title: "trailmark"
url: https://github.com/trailofbits/trailmark
date_fetched: 2026-05-14
section: "Databases and Data"
---

# Trailmark: Source Code Graph Analysis Tool

## Main Purpose
Trailmark parses source code into queryable graph representations of functions, classes, calls, and semantic annotations for security analysis and vulnerability identification. Built by Trail of Bits.

## Key Features

**Core Capabilities:**
- Language-agnostic AST parsing using tree-sitter
- High-performance graph traversal via rustworkx
- Support for 25+ programming languages (Python, JavaScript, TypeScript, Rust, Go, Java, C++, Solidity, Cairo, and more)
- Three-phase operation: parse -> index -> query

**Query Operations:**
- `callers_of()` and `callees_of()` for direct neighbors
- `ancestors_of()` and `reachable_from()` for transitive slicing
- `paths_between()` for call path analysis
- `entrypoint_paths_to()` for attack surface analysis
- `complexity_hotspots()` for identifying high-complexity functions

**Semantic Annotation:**
Users can add and query custom annotations to nodes, enabling security researchers to mark assumptions, findings, and other metadata.

**External Integration:**
Augments graphs with findings from SARIF-formatted static analyzer results and weAudit findings.

## Supported Constructs
Extracts nodes (functions, classes, structs, interfaces, traits, enums) and edges (calls, inheritance, implementation, containment, imports) with metadata including type annotations, cyclomatic complexity, branches, docstrings, and exception types.

## Installation
Requires Python >= 3.12. Install via PyPI (`uv pip install trailmark`) or from development checkout.

Basic command: `trailmark analyze path/to/project`

Apache-2.0 licensed, 378 GitHub stars.
