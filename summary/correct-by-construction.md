---
title: "Correct by Construction"
url: https://minimalmodeling.substack.com/p/my-take-on-data-quality
date_fetched: 2026-05-14
section: "Databases and Data"
topics:
  - software-engineering-craft
  - databases-and-data
---

# Data Quality Framework: A Whitelist Approach

## Core Argument

Alexey Makhotkin proposes a data quality methodology based on "Minimal Modeling" that emphasizes building datasets "correct by construction" rather than attempting to validate flawed data retroactively. The approach relies on establishing clean input data through a systematic whitelist method across three tiers.

## Three-Tier Framework

**Tier 1:** Establish input data quality for specific queries

**Tier 2:** Maintain data currency for particular queries

**Tier 3:** Manage all data across all queries (existing and new)

## Key Components

### Anchors (Entity IDs)
- Represent countable entities (users, orders, employees)
- Require: complete unique ID lists, uniqueness verification, and clarity on archived vs. current data
- Example: `SELECT id FROM orders`

### Attributes (Data Values)
- Store actual information (numbers, strings, dates, monetary values)
- Must filter NULL values and "sentinel values" (empty strings, "UNKNOWN", nonsensical dates)
- Include canonicalization to fix typos and standardize spelling
- Delivered as two-column datasets: anchor ID + value

### Links (Relationships)
- Connect two anchors, representing relationships between entities
- Require unique ID pairs with no NULLs or sentinel values
- Support M:N, 1:N, and 1:1 cardinalities only
- Multiple distinct links permitted between same anchors

## Quality Requirements Summary

The whitelist methodology prevents common data problems by eliminating:
- Unclear entity identity
- Cardinality ambiguity
- Duplicate rows
- NULLs and sentinel values
- Spelling inconsistencies
- Unrestricted JSON containers

## Notable Quote

The author acknowledges: "I understand that this is not how things are commonly done, but I assume that you have a problem with data quality, and you want to try a different approach."

## Conclusion

By treating queries as functions requiring verified input quality, this approach prioritizes establishing correct foundational datasets before building downstream analytics or reports.
