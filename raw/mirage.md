---
url: https://github.com/strukto-ai/mirage
date_fetched: 2026-07-05
backfilled: true
---

Mirage is **a Unified Virtual File System for AI Agents**: it mounts services and data sources like S3, Google Drive, Slack, Gmail, and Redis side-by-side as one filesystem. Any LLM that already knows bash can read, grep, and pipe across every backend out of the box, with zero new vocabulary.

```
const ws = new Workspace({
  '/data':  new RAMResource(),
  '/s3':    new S3Resource({ bucket: 'logs' }),
  '/slack': new SlackResource({ token: process.env.SLACK_BOT_TOKEN! }),
})
await ws.execute('grep -r alert /slack/channels/general__C04QX/ | wc -l')
await ws.execute('cp /s3/report.csv /data/local.csv')
await ws.execute('wc -l $(find /s3/data -name "*.jsonl")')
// Commands are extensible: register new commands, or override one per
// resource + filetype, e.g. `cat` on S3 Parquet renders rows as JSON.
ws.command('summarize', ...)
ws.command('cat', { resource: 's3', filetype: 'parquet' }, ...)
await ws.execute('summarize /data/local.csv')
await ws.execute('cat /s3/events/2026-05-06.parquet | jq .user')
```
- **One interface instead of N SDKs and M MCPs.**Every service speaks the same filesystem semantics, and pipelines compose across services as naturally as on a local disk.
- **Around 50 built-in backends:**RAM, Disk, Redis, S3 / R2 / OCI / Supabase / GCS, Gmail / GDrive / GDocs / GSheets / GSlides, GitHub / Linear / Notion / Trello, Slack / Discord / Email, MongoDB / Postgres / LanceDB / Qdrant, SSH, and more, mounted side-by-side under a single root.
- **Portable workspaces:**clone, snapshot, and version a workspace; agent runs move between machines without restarting or reconfiguring the system.
- **Embeddable:**the Python and TypeScript SDKs run in-process inside FastAPI, Express, browser apps, or any async runtime; no separate process required.
- **Agent integrations:**OpenAI Agents SDK, Vercel AI SDK, LangChain, Pydantic AI, CAMEL, and OpenHands via the SDKs; coding agents like Claude Code and Codex via the lightweight CLI + daemon.

  

- **Python**≥ 3.11 for the- `mirage-ai`package and the- `mirage`CLI
- **Node.js**≥ 20 for the TypeScript SDK
- **macOS**or- **Linux**(FUSE-based mounts require platform support)

`uv add mirage-ai    # installs the `mirage` library and the `mirage` CLI binary````
npm install @struktoai/mirage-node      # Node.js servers and CLIs
npm install @struktoai/mirage-browser   # browser / edge runtimes
npm install @struktoai/mirage-agents    # OpenAI / Vercel AI / LangChain / Mastra adapters
```
Both runtime packages pull in `@struktoai/mirage-core` automatically.

```
curl -fsSL https://strukto.ai/mirage/install.sh | sh
# or
npm install -g @struktoai/mirage-cli
# or
uvx mirage-ai
# or
npx @struktoai/mirage-cli
```
```
from mirage import Workspace
from mirage.resource.ram import RAMResource
from mirage.resource.s3 import S3Config, S3Resource
ws = Workspace({
    "/data": RAMResource(),
    "/s3":   S3Resource(S3Config(bucket="my-bucket")),
})
await ws.execute("cp /s3/report.csv /data/report.csv")
await ws.execute("grep alert /s3/data/log.jsonl | wc -l")
await ws.snapshot("demo.tar")
```
```
import { Workspace, RAMResource, S3Resource } from '@struktoai/mirage-node'
const ws = new Workspace({
  '/data': new RAMResource(),
  '/s3':   new S3Resource({ bucket: 'my-bucket' }),
})
await ws.execute('cp /s3/report.csv /data/report.csv')
await ws.execute('grep alert /s3/data/log.jsonl | wc -l')
await ws.snapshot('demo.tar')
```
```
mirage workspace create ws.yaml --id demo
mirage execute   --workspace_id demo --command "cp /s3/report.csv /data/report.csv"
mirage provision --workspace_id demo --command "cat /s3/data/large.jsonl"
mirage workspace snapshot demo demo.tar
mirage workspace load demo.tar --id demo-restored
```
Mirage plugs into agent frameworks as a sandbox or tool layer. POSIX operations such as `read` can also be customized per resource and filetype, e.g. reading a PDF returns parsed pages instead of raw bytes.

| Integrations | |
|---|---|
| Python | OpenAI Agents SDK, LangChain, Pydantic AI, CAMEL, OpenHands, Agno | 
| TypeScript | Vercel AI SDK, OpenAI Agents SDK, LangChain, Mastra | 
| Coding agents | Claude Code, Codex, OpenCode, Pi | 

Every `Workspace` has a two-layer cache so repeated work against remote backends hits local state instead of the network:

- **Index cache:**listings and metadata. The first directory walk hits the API; later ones serve from the index until the TTL expires (default 10 minutes).
- **File cache:**object bytes. The first read streams from origin; later pipelines read from cache (default 512 MB).

Both layers default to in-process RAM with zero setup. A Redis store shares cache state across workers, processes, and machines:

```
import { RedisFileCacheStore, S3Resource, Workspace } from '@struktoai/mirage-node'
const ws = new Workspace(
  { '/s3': new S3Resource({ bucket: 'my-bucket' }) },
  {
    cache: new RedisFileCacheStore({ url: 'redis://localhost:6379/0', cacheLimit: '8GB' }),
    index: { type: 'redis', url: 'redis://localhost:6379/0', ttl: 600 },
  },
)
```
See the cache docs for the full miss/hit lifecycle.

Thanks to everyone who has contributed to Mirage.
