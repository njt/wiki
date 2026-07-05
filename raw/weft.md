---
url: https://github.com/jonesphillip/weft
date_fetched: 2026-07-05
backfilled: true
---

Task management, but AI agents do your tasks.

Weft is a personal task board where AI agents work on your tasks. Create a task, assign it to an agent, and it gets to work. Agents can read your emails, draft responses, update spreadsheets, create PRs, and write code.

Run as many agents in parallel as you want. Schedule recurring tasks to run daily or weekly. Get notified when tasks complete or need your approval. All actions that mutate state (sending emails, creating PRs, modifying documents) require your approval before committing.

**Built-in integrations:**

- Gmail (read, draft, send)
- Google Docs (create, edit)
- Google Sheets (create, update)
- GitHub (issues, PRs, code)
- Cloudflare Sandbox (isolated containers for code execution and coding agents)
- Remote MCP servers (bring your own tools)

**Self-hosted and private.** Deploy to your own Cloudflare account. Your data stays in your account.

Weft runs entirely on Cloudflare's Developer Platform:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   Browser ◄──────► Worker ◄──────► Durable Objects ◄──────► Workflows  │
│     │               (HTTP)          (BoardDO, UserDO)      (AgentWorkflow)
│     │                                     │                      │      │
│     │                                     │                      ▼      │
│     └─────── WebSocket ──────────────────►│              ┌────────────┐ │
│              (real-time updates)                         │  Anthropic │ │
│                                                          │  + Tools   │ │
│                                                          └────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```
| Component | Purpose | 
|---|---|
| Workers | HTTP routing, auth, serves React frontend | 
| Durable Objects | Persistent state (boards, tasks, credentials) and real-time WebSocket updates | 
| Workflows | Durable agent loop with automatic checkpointing and retry | 

- Node.js 18+
- Docker running (required for Cloudflare Sandboxes)
- Cloudflare account with Workers Paid plan
- OAuth credentials for integrations you want to use:

```
git clone https://github.com/jonesphillip/weft.git
cd weft
npm install
```
Copy the example environment file:

`cp .env.example .env`Edit `.env` with your secrets:

```
# Encryption key for stored credentials (generate with: openssl rand -base64 32)
ENCRYPTION_KEY=your-generated-key
# OAuth credentials (add the ones you need)
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...
```
See Setting up OAuth credentials for detailed instructions on creating Google and GitHub OAuth apps.

`npm run dev`This starts both the Vite dev server (frontend) and Wrangler (worker). Open http://localhost:5174.

By default, Weft runs without authentication. Boards and tasks are stored under the user defined by `USER_ID` in `wrangler.jsonc`.

After creating your first board, go to **Settings** and add your Anthropic API key. This key is stored per-board and is required for agents to run.

Set your secrets (only set the ones you need):

| Secret | Required | Description | 
|---|---|---|
| `ENCRYPTION_KEY` | Yes | Encrypts stored credentials. Generate with `openssl rand -base64 32` | 
| `GOOGLE_CLIENT_ID` | For Google integrations | From Google Cloud Console | 
| `GOOGLE_CLIENT_SECRET` | For Google integrations | From Google Cloud Console | 
| `GITHUB_CLIENT_ID` | For GitHub integration | From GitHub Developer Settings | 
| `GITHUB_CLIENT_SECRET` | For GitHub integration | From GitHub Developer Settings | 

```
npx wrangler secret put ENCRYPTION_KEY --env production
npx wrangler secret put GOOGLE_CLIENT_ID --env production
npx wrangler secret put GOOGLE_CLIENT_SECRET --env production
npx wrangler secret put GITHUB_CLIENT_ID --env production
npx wrangler secret put GITHUB_CLIENT_SECRET --env production
npm run deploy:prod
```
Authentication is controlled by the `AUTH_MODE` variable in `wrangler.jsonc`:

| AUTH_MODE | Description | Use case | 
|---|---|---|
| `none` | No authentication. Uses `USER_ID`/`USER_EMAIL`from config as singleton user. | Personal/single-user deployments | 
| `access` | Cloudflare Access JWT verification. Requires `ACCESS_AUD`/`ACCESS_TEAM`secrets. | Multi-user or login-protected deployments | 

By default, `AUTH_MODE` is set to `"none"` for both development and production.

For multi-user deployments or if you want login protection, enable Cloudflare Access:

- In the Cloudflare dashboard, go to **Zero Trust > Access > Applications**
- Create a new **Self-hosted application**with your Weft URL
- Add an **Access policy**(e.g., allow only your email address)
- After saving, find the **Application Audience (AUD) Tag**in the application settings
- Your **team name**is in your Zero Trust dashboard URL:`https://<team>.cloudflareaccess.com`

Set the secrets:

```
npx wrangler secret put ACCESS_AUD --env production
npx wrangler secret put ACCESS_TEAM --env production
```
Then in `wrangler.jsonc`, change `AUTH_MODE` to `"access"` in the production environment:

```
worker/
├── index.ts              # HTTP routing and auth
├── handlers/             # Request handlers
├── services/             # Business logic
├── workflows/            # Cloudflare Workflows (agent loop)
├── mcp/                  # MCP server registry
├── google/               # Google integrations (Gmail, Docs, Sheets)
└── github/               # GitHub integration
src/
├── components/           # React components
├── context/              # React context providers
├── hooks/                # Custom hooks
└── api/                  # API client
```
Tasks can be configured to run on a schedule (daily, weekly, or cron). When a scheduled task runs, it operates differently from regular tasks:

- A Durable Object alarm triggers at the scheduled time
- The parent agent runs with read-only tool access. It cannot send emails, create PRs, or modify anything directly.
- The parent agent calls `create_task`to fork child tasks for each piece of work it finds
- Each child task gets its own agent context with full tool access and runs until it needs human approval

This separation means scheduled agents can analyze your inbox or repos overnight, but actual changes still require your review.

| Schedule Type | Example | 
|---|---|
| Daily | 8:00 AM in your timezone | 
| Weekly | Monday and Thursday at 9:00 AM | 
| Custom | Cron expression ( `0 */6 * * *`) | 

Each board has its own configuration in **Settings**, organized into tabs:

- **Credentials**- Add your Anthropic API key (required for agents) and connect OAuth accounts like Google and GitHub.
- **Integrations**- Enable built-in tools (Gmail, Docs, Sheets, GitHub) or add any remote MCP server for custom tools.

To add a built-in integration, Weft uses a registry-driven architecture:

- Create your MCP server in `worker/`(see`worker/google/GmailMCP.ts`for example)
- Register it in `worker/mcp/AccountMCPRegistry.ts`
- Add OAuth handler if needed in `worker/handlers/oauth.ts`

The registry defines tool schemas, OAuth configuration, and workflow guidance—no changes needed to the agent workflow itself.

- Go to the Google Cloud Console
- Create a new project or select an existing one
- Go to **APIs & Services > Library**and enable the Gmail API, Google Docs API, and Google Sheets API
- Go to **APIs & Services > OAuth consent screen**and configure it (External is fine, add yourself as a test user)
- Go to **APIs & Services > Credentials**
- Click **Create Credentials > OAuth client ID**
- Select **Web application**
- Add authorized JavaScript origins:
- `http://localhost:5174`(development)
- `https://weft.<your-subdomain>.workers.dev`(production)
 
- Add authorized redirect URIs:
- `http://localhost:5174/google/callback`(development)
- `https://weft.<your-subdomain>.workers.dev/google/callback`(production)
 
- Copy the Client ID and Client Secret to your `.env`file

- Go to GitHub Developer Settings
- Click **New OAuth App**
- Set the homepage URL to your Weft deployment
- Set the authorization callback URL:
- `http://localhost:5174/github/callback`(development)
- `https://weft.<your-subdomain>.workers.dev/github/callback`(production)
 
- Copy the Client ID and Client Secret to your `.env`file

Apache License 2.0 - see LICENSE
