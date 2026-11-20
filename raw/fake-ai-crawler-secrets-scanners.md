---
url: https://tin.computer/blog/fake-ai-crawler-secrets-scanners
date_fetched: 2026-11-20
---

# Octopus, Gecko and Tortoise: the scanners hunting your AI keys

If GPTBot or ClaudeBot asks your site for `/.env`, it is not OpenAI or Anthropic. It is a scanner we call Octopus. It starts with `/z9x8c7v6b5-debug-trigger-` and your hostname. A second scanner, Gecko, sends the cut-off user agent `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36` and asks for the files where Claude Code, Codex, Cursor and Gemini CLI keep their keys. A third, Tortoise, is small and slow. We watched all three on ten websites we run, from late September to 18 November 2026.

## What we foundin short.

- **Found:**Octopus uses 32 AI crawler names and changes name on every request. It rents a new Google Cloud machine for each website: 97 machines in 15 regions. Most lived less than an hour.
- **Found:**before it asks for secrets, Octopus browses the site like Chrome. Then it picks a list of paths for the site’s framework. Static sites, Next.js sites and Vite sites get different lists.
- **Found:**Gecko is a .NET program. It runs mostly from Google Cloud’s Belgium region, and also from rented Windows machines at an offshore hoster. Its user agent is its own Chrome 128 user agent, cut after 60 characters. The cut removes the words “like Gecko”.
- **Found:**Gecko’s list of AI tool files grew from 4 to 26 paths in five weeks. On 12 November it tried a new list for one day, then kept only the token files.
- **Not proven:**who runs them. The evidence points to two different operators for Octopus and Gecko.

Want to know what agents are doing on your site? Book a free 30-minute call with Emre →

## What you seein your logs.

Each scanner has one sign you can search for:

### Octopus

/z9x8c7v6b5-debug-trigger-<your hostname>32 AI crawler names, a new one on each request

### Gecko

Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36Asks for /.claude/settings.json and /.cursor/mcp.json

### Tortoise

/nonexistent-debug-trigger-7x7Chrome/126 on Windows, one request every few seconds

## Octopuswears 32 costumes.

Every Octopus visit starts the same way:

- 01**The canary**/z9x8c7v6b5-debug-trigger-<your hostname>
- 02**Two random paths**/gubgyjrd34ipmzr98uwq
- 03**About 300 secret files**in about 10 seconds, each with a different crawler name

The two random paths always get a 404. Octopus probably uses them to learn what your site sends for a missing page. Then it can tell a real `.env` file from a page that says “not found” with a 200.

On each request, Octopus picks one of 32 user agents at random. Seven of them use names that no company documents:

The other 25 copy real crawlers, mostly letter for letter:

A typical one reads `Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko); compatible; GPTBot/1.4; +https://openai.com/gptbot`. All 32 come from one machine, on one connection, within seconds. No real crawler does that.

- Accept
- */*
- Accept-Encoding
- gzip
- TLS
- the same handshake as Chrome
- Redirects
- follows them, keeps the user agent, adds a Referer
- Network
- Google Cloud, AS396982, *.bc.googleusercontent.com

Its handshake is the same as Chrome’s. It follows redirects, keeps the user agent and adds a Referer. That is how Go’s standard HTTP client behaves. We think Octopus is a Go program with a Chrome-like TLS library.

Other site owners have seen it. GreyNoise, Fastly and Cequence reported scanners with fake AI crawler names on Google Cloud, and the canary path is in public logs from September. On our sites it started on 3 October. Until 16 October it used 23 or 24 names, including PanguBot/1.0, YiBot/1.0 and MistralAI-User. It then went to 32.

## A new machinefor each website.

We matched each Octopus address to Google Cloud’s published ranges. Every one is a Google Compute Engine machine. Octopus uses a new machine for each website and a different region almost every time. Of 82 machines in a row, only 3 used the same region as the one before.

On 24 October, five machines in five regions scanned the same site within 25 minutes. A block on one address does nothing. The next visit comes from a new one.

Every still-live Octopus machine we connected to was a stock Ubuntu 24.04 install with an open SSH port and nothing else in the top 1,000 ports. Two of them were gone within minutes of our connecting.

Octopus does not sweep the internet by address. Most of the machines we checked were unknown to GreyNoise, which watches mass scanning. It works from a list of hostnames. It reached three of our hostnames that have no certificate of their own, so the list does not come only from certificate logs.

## It browses first,then picks its tools.

Octopus has a second stage that the reports above do not mention. About 1.5 seconds before the canary, the same machine opens your home page as a normal browser. It sends one Chrome or Edge user agent, version 152 to 157, with full browser headers.

In that stage it reads `/robots.txt` and `/sitemap.xml` and follows links on your pages, a median of 68 per visit. It asks for build manifests, such as `/.next/build-manifest.json`, `/ngsw.json` and `/.vite/manifest.json`. It even asks for Next.js route templates with the brackets still in them, such as `/plays/[playId]`. It also tries `/admin`, `/settings/api-keys`, `/graphql` and `/swagger`.

The summary of Cequence’s report says the scanner ignored ordinary web pages. On our sites it did not. The page requests come with a browser user agent, so a filter on AI crawler names misses them.

Since late October, Octopus uses what it reads. It sends a different list to each kind of site:

### Static HTML sites

≈385PHP, Python, Laravel, Terraform, Docker and cgi-bin files, plus POST requests that try known code-execution bugs.

### Next.js sites

≈265No PHP or Python and no POST requests. Next.js build files and Kubernetes configs instead.

### The Vite site

≈28522 extra paths for an exposed Vite dev server, such as /@fs/var/task/.env and /.env::$DATA.

Before late October every site got the same list of about 275 paths. Our Next.js sites never got a POST request after that. We think Octopus looks for signs of the framework in the home page and the manifests.

## What Octopuswants.

Its full list has 457 paths. Most of them are secrets that live in files:

- .env files in many places
- Cloud keys
- AI gateways and builders
- AI model tokens
- Build and deploy files
- CI and secret stores
- Java and debug endpoints

The list changes. Octopus added the Langflow and Flowise code-execution requests on 20 October, two weeks after a run of advisories for both tools, and removed them a day later. It added the LiteLLM paths, which show a LiteLLM proxy’s keys and spending, on 12 November. It also drops paths: home folder and SSH files went on 24 October.

About half of the list is in public wordlists. The other half is not, and it is the half that looks designed: Vite paths aimed at cloud servers (`/@fs/var/task/.env`), product-specific AI endpoints and `.env` files in folders named `llm`, `agent` and `chatbot`.

## Geckodropped its tail.

Gecko’s user agent ends early. This is its own Chrome 128 user agent, cut after 60 characters:

Before each scan, Gecko opens your home page once with the full string. The scan uses only the first 60 characters. Everything from `(KHTML, like Gecko)` on is gone. Chrome 128 is from 2024, so the string is an old constant in the program.

- Order
- GET / with the full user agent, then the whole list 0.65 seconds later
- Speed
- about 200 requests in under one second, always in the same order
- Accept
- none
- Accept-Encoding
- gzip, deflate
- Referer
- never
- TLS
- the default cipher list of .NET on Linux

Gecko is a .NET program. Its cipher list is the default list that .NET uses on Linux. We made the same request with .NET 9’s HttpClient and got the same handshake, byte for byte. The home page fetch and the scan share one TLS session, so they come from one program.

Gecko sends its whole list at once: a median of 206 requests in 0.86 seconds. The order is always the same.

## Geckolives in Belgium.

Unlike Octopus, Gecko stays in one place. 54 of its addresses are in one Google Cloud region, europe-west1 in Belgium. It also used two small hosting companies:

| Network | ASN | Addresses | Dates (2026) | 
|---|---|---|---|
| Google Cloud europe-west1 (Belgium) | AS396982 | 54 addresses | 1 Oct to now | 
| TC DATACENTER | AS218785 | 45.138.12.23, .25, .27, .28 | 30 Sep to 8 Oct | 
| DMZHOST | AS48090 | 45.148.10.21, 195.178.110.242 | 5 Oct to 16 Nov; 23 Oct to 14 Nov | 

Both DMZHOST machines run Windows Server 2022 and answer only on remote desktop. Their hostnames are `D06S15` and `D10S18`. They are ordinary rented VPS at an offshore hoster that sells Windows KVM from €5 to €45 a month, and they stay up: 16 and 37 days when we connected on 18 November.

Gecko’s Google Cloud machines are stock Ubuntu 26.04 LTS now. Earlier in November the same fleet used a minimal OpenSSH 10.0 build with publickey-only login and a shorter key-exchange list. All 24 SSH host keys we collected differ. In October, Gecko kept some Google Cloud addresses for weeks. Since 10 November, almost every address is used for one day only.

Gecko visits sites in a fixed order. When one address scanned several of our sites, the second site came about 5 minutes after the first and the third about 17 minutes after that, in every case we saw.

## Gecko wants yourcoding agent’s keys.

Gecko was on our sites on 30 September at 07:33 UTC. It scanned four sites in 37 seconds and already asked for Claude Code’s settings files. Since then it added paths for more AI tools:

- **Claude Code settings and provider keys**
- **MCP server configs**
- **Coding agent logins**
- **LLM CLI keys and gateways**

`/.claude/.credentials.json` is where Claude Code keeps its login on Linux. `/.codex/auth.json` holds Codex’s login. `/.cursor/mcp.json` and `mcp_config.json` often hold API keys for MCP servers.

AI tool files were about 4% of Gecko’s requests in October. In the week of 16 November they were 10%.

On 12 November at 12:19 UTC, Gecko replaced about 100 paths with a new list. The next morning it went back to the old list. It kept 13 of the new paths, among them the three token files: `.claude/.credentials.json`, `.codex/auth.json` and `.gemini/settings.json`. It also corrected its Windsurf path to the real location.

Gecko was not the first. Other scanners asked our sites for `.claude/.credentials.json` and `.codex/auth.json` from 30 October. SANS and HoneyLabs have reported scanners with similar targets. We found no public report of Gecko’s user agent with this list.

## Tortoiseis slow.

The third scanner is small. It came twice, on 13 and 17 November:

- First request
- /nonexistent-debug-trigger-7x7
- Missing-page test
- /pscan-<8 hex digits>-nonexistent.txt
- User agent
- Chrome/126 on Windows
- Speed
- one request every 1 to 7 seconds, over HTTP/1.1
- Redirects
- does not follow them
- Network
- TC DATACENTER (AS218785): 45.138.12.180 and 45.138.12.35
- Seen
- 13 and 17 November, on two of our sites

It is a Go program with the standard Go TLS library. It asks for debug pages of Spring, PHP and Rails, reads your home page and asks for the scripts it finds. Its network is the same /24 that Gecko used in early October, but nothing else connects them. That network is a budget VPS host with dozens of tenants: 54 SSH servers, 27 Windows remote desktops, Debian, Ubuntu and Bitvise side by side. Tortoise’s two addresses are stock Ubuntu 24.04 and show only an RSA host key. We found nothing public about either of its paths.

## Two tools,two operators.

Octopus and Gecko started in the same week and share about 150 common paths, such as `/.env` and `/.git/config`. Everything else is different:

| Octopus | Gecko | |
|---|---|---|
| Language | Go, most likely | .NET | 
| TLS handshake | the same as Chrome | .NET’s default | 
| User agent | 32 crawler names, random | one cut-off Chrome string | 
| Redirects | follows, adds a Referer | never sends a Referer | 
| Hosting | 15 Google Cloud regions | Belgium, plus two VPS | 
| Machine life | under an hour, mostly | days to weeks | 
| Path list | chosen per framework | one list for every site | 
| Speed | ≈300 requests in 10 s | ≈200 requests in under 1 s | 

They never used the same address or /24. They did not scan at the same times more often than chance. We also connected to both fleets: they share no SSH host key (24 keys, all different), no OS image, no SSH build, no login policy and no exposed service beyond the admin port. We think two different operators run them.

Both read the home page before they scan. Both asked for the real, hashed names of a site’s JavaScript source maps, which they could only get from the page. Real AI crawlers asked for the same source maps too, so a source map request alone means nothing.

## Why nowand what changed.

AI tools put new keys in new places. A coding agent keeps its login and its MCP server keys in files in your home folder and your project. Those files reach a website when a build copies the whole project into its output, when a container serves the wrong folder, or when someone commits `.claude/settings.local.json`. Gecko’s list follows the tools: every new agent config file shows up in it.

Attackers also move faster. This week a researcher found a remote code execution bug in Java with an AI coding agent in about an hour, and had an exploit about two hours later. An OpenSSH flaw was exploited within hours of disclosure. Octopus adds new exploit paths one to three weeks after a run of advisories. The time between a public bug and a request for it on your site is short, and it is getting shorter.

We do not think this week’s news caused these scanners. Gecko added its newest AI paths 17 hours before the week’s big package compromise, and neither scanner has ever asked for `AGENTS.md` or `.mcp.json`. Both had their busiest day on 17 November. That is all we can say about the timing.

## A crawler nameis not proof.

Many site owners decide what to allow by user agent. Octopus uses that. Its names are in the blocklists and allowlists people copy: 29 of its 32 names are in the popular ai.robots.txt list. The pull request that added Hunyuan and MoonshotBot said itself that the traffic looked like vulnerability probes from Google Cloud. The names went into the list anyway.

A name in a list does not show that a company runs a crawler with that name. Check the address. OpenAI, Google, Microsoft and others publish the addresses of their crawlers. Cloudflare marks verified bots for you. A request with a real crawler’s name from an address that the company does not list is not that crawler.

## How to protectyour keys.

Blocking a scanner helps a little. Not having the files helps a lot. Start here:

- **Request your own secret paths.**Each of these should return 404:- `/.env`,- `/.env.production`,- `/.git/config`,- `/.claude/settings.local.json`,- `/.cursor/mcp.json`,- `/.codex/auth.json`,- `/.aws/credentials`. If one returns 200, open it. A copy of your home page is harmless. A real file is not.
- **Check your build output.**Static hosts publish everything in- `public/`or- `dist/`. Run- `find public dist out -name '.*'`before you deploy.
- **Keep agent files out of Git.**Add- `.claude/settings.local.json`,- `.claude/.credentials.json`,- `.codex/`and- `.env*`to- `.gitignore`. Put MCP server keys in environment variables, not in- `mcp.json`.
- **Never put a dev server on the internet.**Octopus has 22 paths for an exposed Vite dev server alone.
- **Keep AI gateways private.**LiteLLM, Flowise and Langflow admin APIs do not belong on a public address. Update them when an advisory comes out. Octopus follows advisories.
- **Check your source maps.**If you publish them, make sure they contain no keys.
- **Rotate anything that was ever public.**Scanners use keys within seconds.

To block the scanners too:

### Cloudflare: every dotfile except /.well-known/

`starts_with(http.request.uri.path, "/.") and not starts_with(http.request.uri.path, "/.well-known/")`Stops the requests for .env, .git and agent config files from all three.

### Cloudflare: Gecko

`http.user_agent eq "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"`Blocks the exact cut-off string. Full Chrome user agents do not match.

### Cloudflare: the canaries

`http.request.uri.path contains "debug-trigger"`Stops only the first request of Octopus and Tortoise.

### nginx

```
location ~ /\.(?!well-known/) { return 404; }
if ($http_user_agent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36") { return 403; }
```
The same two rules for nginx.

Do not block AI crawler names to stop these scanners. You will block real crawlers too, and the scanners will pick other names.

## Sourcesand other reports.

## Questions.

## What is /z9x8c7v6b5-debug-trigger in my logs?

It is the first request of a secrets scanner we call Octopus. The path ends with your own hostname, for example /z9x8c7v6b5-debug-trigger-www.example.com. A few hundred requests for .env files, cloud keys and AI tool configs follow within about ten seconds, each with the name of a different AI crawler.

## Is GPTBot or ClaudeBot asking for /.env really OpenAI or Anthropic?

On our sites, no. Real AI crawlers read pages. They do not ask for .env, .git/config or cloud credential files. Every request with an AI crawler name for a secret file that we checked came from scanners on rented machines. Check the address against the list the company publishes before you trust the name.

## Is "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" a real browser?

No. A real Chrome user agent continues with (KHTML, like Gecko) Chrome/… Safari/537.36. This one stops after 60 characters. A scanner we call Gecko sends it, together with a list of about 240 secret files. Some other scanners send the same string, so block it in any case.

## What is /nonexistent-debug-trigger-7x7?

It is the first request of a third, slow scanner we call Tortoise. It also asks for /pscan-<8 hex digits>-nonexistent.txt to learn how your site answers missing pages, then asks for debug pages of Spring, PHP and Rails, one request every few seconds.

## Does blocking these hurt my visibility in ChatGPT or Claude?

No, if you block by behaviour and not by name. Real AI crawlers do not ask for .env files. Block requests for paths that start with a dot (keep /.well-known/), block the exact Gecko user agent, and leave the AI crawler names alone or check them against the operator’s addresses.

## Did the scanners get any secrets from your sites?

No. From 22 September to 18 November 2026, every request for a secret path on our sites got a 404, 403 or 405. They did download public pages and public source maps, which any visitor can do.

## How do I check if my site exposes .env?

Request it yourself: curl -s -o /dev/null -w '%{http_code}' https://yoursite.com/.env. You want 404. If you get 200, open the response. Many single-page apps answer every path with the home page, which is harmless. A real file with keys is not, and you must rotate those keys.

## Not every bot is who it says.Know which ones are.

Book a free 30-minute call with Emre. We’ll look at which agents and scanners visit your site, which ones are real, and how to turn the real ones into customers.

Choose a time for your free 30-minute call.
