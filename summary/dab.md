---
title: "DAB"
url: https://learn.microsoft.com/en-us/azure/data-api-builder/
date_fetched: 2026-05-14
section: "Databases and Data"
topics:
  - databases-and-data
  - mcp-and-tool-protocols
---

# Data API Builder (DAB) - Microsoft

## Summary
Data API builder generates REST and GraphQL endpoints for databases including SQL Server, Azure SQL, Azure Cosmos DB, PostgreSQL, and MySQL. Also includes an MCP server for AI agent integration. Open source, secure, and production-ready.

## Key Features

**API Generation:**
- Auto-generated REST endpoints with OpenAPI/Swagger UI
- Schema-driven GraphQL queries, mutations, relationships, and aggregation
- Multiple mutations as transactions
- Database views and stored procedures support

**SQL MCP Server:**
- Connect AI agents to databases through Model Context Protocol
- DML tools for agents
- Custom MCP tool configuration
- Entity descriptions for agent comprehension
- MCP authentication support

**Query Capabilities:**
- Record filtering, field projection, sorting, pagination
- In-memory (Level 1) and Redis (Level 2) caching
- Cache-Control headers

**Security:**
- Microsoft Entra ID, custom JWT (Okta, Auth0, Keycloak), App Service EasyAuth
- Authorization roles
- Row-level security and database policies
- On-Behalf-Of (OBO) flow

**Configuration:**
- CLI for init, add, update, configure, validate, start, export
- Environment-specific configuration files
- Azure Key Vault integration
- Multiple data sources
- Auto-configuration

## Notable
The MCP server integration makes this particularly interesting for AI agent workflows -- agents can query and manipulate databases through a standardized protocol.
