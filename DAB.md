# DAB

Microsoft's Data API Builder: point it at a database, get REST and GraphQL endpoints with authentication, caching, and row-level security. Now includes an MCP server for connecting AI agents directly to your data.

---

## Key Themes

#api-design #database #agent-architecture

DAB automates the boring-but-essential layer between your database and your application. REST endpoints with OpenAPI/Swagger, GraphQL with schema-driven queries and mutations, multi-level caching (in-memory + Redis), and a security model that includes Entra ID, custom JWT, row-level security, and database policies.

The MCP server integration is the most interesting recent addition. AI agents can query and manipulate databases through the Model Context Protocol, with custom tool configuration and entity descriptions that help agents understand the data model. This is the "give agents database access safely" problem that many teams are solving ad hoc.

Supports SQL Server, Azure SQL, Cosmos DB, PostgreSQL, and MySQL. The CLI (`dab init`, `dab add`, `dab start`) makes it easy to get running, and the auto-configuration feature can introspect an existing database and generate the configuration.

## Critical Analysis

DAB is exactly the kind of infrastructure that Microsoft does well: unglamorous but essential middleware that saves teams from building the same REST/GraphQL layer for the hundredth time. The MCP server is the forward-looking feature -- as agents become common consumers of APIs, having a standardized way to expose database operations to them matters. The Azure-centric security model (Entra ID, Key Vault) is both a strength (deep integration) and a limitation (less useful outside Azure). The open-source nature and support for PostgreSQL and MySQL broaden the appeal beyond the Azure ecosystem.

---
*Sources: [[raw/dab]]*
*Last updated: 2026-05-14*
