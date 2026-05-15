---
title: "zvec"
url: https://github.com/alibaba/zvec
date_fetched: 2026-05-14
section: "Databases and Data"
---

# Zvec: In-Process Vector Database (Alibaba)

## Purpose
Lightweight, open-source vector database engineered for embedding directly into applications. Developed and battle-tested within Alibaba Group.

## Key Features

- **Performance**: Searches billions of vectors in milliseconds
- **Simplicity**: Install and begin searching within seconds; pure local operation without servers
- **Vector Support**: Handles both dense and sparse embeddings with native multi-vector query capabilities
- **Hybrid Search**: Combines semantic similarity with structured filtering
- **Data Durability**: Write-ahead logging (WAL) ensures persistence even during crashes
- **Concurrent Access**: Multiple processes can simultaneously read collections; writes maintain single-process exclusivity
- **Portability**: Functions as in-process library, compatible with notebooks, servers, CLI tools, and edge devices

## Installation
Available for Python 3.10-3.14 (`pip install zvec`) and Node.js (`npm install @zvec/zvec`). Supports Linux (x86_64, ARM64), macOS (ARM64), and Windows (x86_64).

## Technical Stack
Primary implementation in C++ (79.5%), with Python bindings (7.8%) and SWIG language integration (7.7%).
