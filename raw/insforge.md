---
url: https://github.com/InsForge/InsForge
date_fetched: 2026-07-05
backfilled: true
---

⭐ *Help us reach more developers and grow the InsForge community. Star this repo!*

The all-in-one, open-source backend platform for agentic coding. InsForge gives your coding agent database, auth, storage, compute, hosting, and AI gateway to ship full-stack apps end-to-end.

## read-me.mp4

Coding agents interact with InsForge through one of two interfaces:

- **MCP Server**(self-hosted and cloud): exposes InsForge's operations as tools any MCP-compatible agent can call.
- **CLI + Skills**(cloud only): a command-line interface paired with Skills that agents invoke directly from the terminal.

Both interfaces let coding agents operate the backend like backend engineers:

- **Read backend context and state**: Pull documentation, schemas, metadata (deployed functions, bucket contents, auth config), and runtime logs, so the agent has what it needs to write code, verify what it built, and debug when something breaks.
- **Configure primitives**: Deploy edge functions, run database migrations, create storage buckets, set up auth providers, and configure other backend resources directly.

```
graph TB
    subgraph TOP[" "]
        AG[AI Coding Agents]
    end
    subgraph MID[" "]
        SL[InsForge]
    end
    AG --> SL
    SL --> AUTH[Authentication]
    SL --> DB[Database]
    SL --> ST[Storage]
    SL --> EF[Edge Functions]
    SL --> MG[Model Gateway]
    SL --> CP[Compute]
    SL --> DEP[Deployment]
    classDef bar fill:#0b0f14,stroke:#30363d,stroke-width:1px,color:#ffffff
    classDef card fill:#161b22,stroke:#30363d,stroke-width:1px,color:#ffffff
    class AG,SL bar
    class AUTH,DB,ST,EF,MG,CP,DEP card
    style TOP fill:transparent,stroke:transparent
    style MID fill:transparent,stroke:transparent
    linkStyle default stroke:#30363d,stroke-width:1px
```
    - **Authentication**: User management, authentication, and sessions
- **Database**: Postgres relational database
- **Storage**: S3 compatible file storage
- **Model Gateway**: OpenAI compatible API across multiple LLM providers
- **Edge Functions**: Serverless code running on the edge
- **Compute**(private preview): Long-running container services
- **Site Deployment**: Site build and deployment

If you find InsForge useful or interesting, a GitHub Star ⭐️ would be greatly appreciated.

### Cloud-hosted: insforge.dev

Prerequisites: Docker + Node.js

You can run InsForge locally using Docker Compose. This will start a local InsForge instance on your machine.

Or run from source:

```
# Run with Docker
git clone https://github.com/InsForge/InsForge.git
cd insforge
cp .env.example .env
docker compose -f docker-compose.prod.yml up
```
Follow the steps to connect InsForge MCP Server

To verify the connection, send the following prompt to your agent:

```
I'm using InsForge as my backend platform, call InsForge MCP's fetch-docs tool to learn about InsForge instructions.
```
You can run multiple InsForge projects on the same host by using different ports and project names.

```
# Create a separate env file for each project
cp .env.example .env.project1
cp .env.example .env.project2
```
Edit `.env.project2` with different ports:

```
POSTGRES_PORT=5442
POSTGREST_PORT=5440
APP_PORT=7230
AUTH_PORT=7231
DENO_PORT=7233
```
Start each project with a unique name:

```
docker compose -f docker-compose.prod.yml --env-file .env.project1 -p project1 up -d
docker compose -f docker-compose.prod.yml --env-file .env.project2 -p project2 up -d
```
Each project gets its own isolated database, storage, and configuration. Manage them with:

```
docker compose -f docker-compose.prod.yml --env-file .env.project1 -p project1 ps      # status
docker compose -f docker-compose.prod.yml --env-file .env.project1 -p project1 logs -f  # logs
docker compose -f docker-compose.prod.yml --env-file .env.project1 -p project1 down     # stop
```
In addition to running InsForge locally, you can also launch InsForge using a pre-configured setup. This allows you to get up and running quickly with InsForge without installing Docker on your local machine.

| Railway | Zeabur | Sealos | 
|---|---|---|

**Contributing**: If you're interested in contributing, you can check our guide here CONTRIBUTING.md. We truly appreciate pull requests, all types of help are appreciated!

**Support**: If you need any help or support, we're responsive on our Discord channel, and also feel free to email us info@insforge.dev too!

- **Official Docs**- Comprehensive guides and API references

- **Email**: info@insforge.dev

This project is licensed under the Apache License 2.0 - see the LICENSE file for details.

Show your project is built with InsForge.

**Markdown:**

`[![Made with InsForge](https://insforge.dev/badge-made-with-insforge.svg)](https://insforge.dev)`**HTML:**

```
<a href="https://insforge.dev">
  <img
    width="168"
    height="30"
    src="https://insforge.dev/badge-made-with-insforge.svg"
    alt="Made with InsForge"
  />
</a>
```
**Markdown:**

`[![Made with InsForge](https://insforge.dev/badge-made-with-insforge-dark.svg)](https://insforge.dev)`**HTML:**

```
<a href="https://insforge.dev">
  <img
    width="168"
    height="30"
    src="https://insforge.dev/badge-made-with-insforge-dark.svg"
    alt="Made with InsForge"
  />
</a>
```
⭐ **Star us on GitHub** to get notified about new releases!
