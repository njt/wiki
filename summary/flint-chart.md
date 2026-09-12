---
url: https://github.com/microsoft/flint-chart
title: Flint — A Visualization Language for the AI Era
author: Microsoft Research (with IDEAS Lab, Renmin University of China)
date_published: 2026-07
topics:
  - developer-tools
---

Flint is a visualization intermediate language (IL) that lets AI agents turn compact, human-editable chart specifications into polished visualizations across five rendering backends. Rather than requiring verbose, library-specific configuration for scales, axes, spacing, labels, and layout, Flint derives those decisions from the data, a 70+-entry semantic type registry, chart type, encodings, and an optional visual theme. The same input compiles to Vega-Lite, ECharts, Chart.js, Plotly, or native Excel charts.

The project ships as two npm packages: `flint-chart` (the TypeScript library, ~78K lines) and `flint-chart-mcp` (an MCP server that lets agents create, validate, and render charts directly from chat). A Python port is in preview. Flint is built by Microsoft Research and licensed under MIT.

Core architectural metaphor: a traditional three-stage compiler. Stage 1 (semantic resolution) maps field types and data characteristics to per-channel visualization decisions. Stage 2 (optimizer) fits the resulting chart to the available canvas using physics-inspired models — spring mechanics for discrete axes, gas pressure for continuous density. Stage 3 (code generator) instantiates backend-native specs through dynamic templates that adapt to data cardinality and semantics. Templates and themes are separate, interchangeable concerns: `chart_spec` says what to draw, `theme_spec` says how it should look, and neither names a backend property.

The library includes ten visual theme presets (Economist, Swiss, Nature, NYT, McKinsey, etc.), a named-view pivot system that computes alternative chart orientations from a single encoding, and an overflow strategy that gracefully truncates rather than rendering unreadable charts. v0.5.0, released August 2026, introduced the formal ThemeSpec specification for portable, inheritable design languages.

The MCP server provides tools for listing chart types, validating inputs, compiling specs, and opening interactive chart previews in MCP-capable clients. Agent skills for chart and theme authoring ship alongside the code.
