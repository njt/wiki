---
url: https://blog.cloudflare.com/search-for-agents/
date_fetched: 2026-11-26
---

# Introducing Search for Agents

__AI Search__, our service that indexes your content and lets you add search to any application or agent without having to build an entire retrieval infrastructure, has supported adding your __website__ as a data source for a while now, automatically crawling, converting and indexing its pages.

Still, setting up an AI Search instance for your website requires manually creating and configuring it initially, and then doing all the integration work to get the search endpoints available from your domain and to make sure that agents know where to find the index and know how to use it.

Today we're introducing **Search for Agents**, which puts that setup behind a toggle in your Cloudflare dashboard. Turn it on for your zone, and our service will create your AI Search instance, start indexing your content, and make search discoverable and available from your own domain.

## How it works

The idea follows naturally from __Markdown for Agents__, where Cloudflare's network converts any HTML page to Markdown in real time for enabled zones using content negotiation headers. Similarly, Search for Agents leverages the power of AI Search by automatically creating an index for your zone and exposing APIs that any AI agent can discover and use to search your website, under your domain, all done at the network level.

You can enable Search for Agents using the API, or by going to the dashboard in your zone page, under **Quick Actions**:

Search for Agents is available to all customers, including Free (with __limits__). Once it’s enabled, Cloudflare will automatically create an AI Search instance, start crawling and indexing the content in your zone. Refer to our __developer documentation__  to understand how the crawling process works.

You can access your instance directly from this page, or by going to directly to your account’s **AI** > **AI Search** instances view:

The AI Search instance for your zone is created with default settings, but you can enable reranking or change configuration options like cache settings at any time. Refer to our __documentation__ for the full list of options.

Once AI Search indexes your content, you or any agent on the Internet can start using Search for Agents, subject to your domain settings (e.g. your bot management settings, __AI Crawl Control__ and web application firewall (__WAF__) configurations).

### Discoverability

Search isn't much use if the agents don’t know it's there. To facilitate discoverability, Search for Agents automatically advertises the search endpoint on every page of your site.

For example, for HTML responses, we add metadata to the page's <head>:

```
<meta name="agent:instructions" content="Use this search API to find relevant content across this site with keywords or natural-language queries, instead of crawling individual pages. Results are returned in Markdown.">
<meta name="agent:endpoint" content="https://developers.cloudflare.com/.well-known/search?q={query}">
<meta name="agent:method" content="GET">
<meta name="agent:documentation" content="https://developers.cloudflare.com/ai-search/how-to/search-for-agents/">
```
If you also have __Markdown for Agents__ enabled, an agent requesting a page with **Accept: text/markdown** receives the same information in YAML front matter:

```
---
agent:
  instructions: "Use this search API to find relevant content across this site with keywords or natural-language queries, instead of crawling individual pages. Results are returned in Markdown."
  endpoint: "https://developers.cloudflare.com/.well-known/search?q={query}"
  method: "GET"
  documentation: "https://developers.cloudflare.com/ai-search/how-to/search-for-agents/"
---
```
The agent will easily pick up these instructions and know what to do next to search your site.

### Searching content

Once the agent knows that your site has a search endpoint, it can start using it:

```
$ curl "https://developers.cloudflare.com/.well-known/search?q=cloudflare+artifacts"
# Search results for: "cloudflare artifacts"
## docs/artifacts/api/workers-binding/index.md (score: 1.000)
# Workers binding · Artifacts · Cloudflare Artifacts docs
Call Artifacts from a Worker binding.
---
## docs/artifacts/index.md (score: 0.951)
# Artifacts · Cloudflare Artifacts docs
Store, version, and share filesystem artifacts across Workers, APIs, and Git-compatible tools.
...
```
Depending on your AI Search configuration, the agent can also use natural language and ask questions to the search endpoint:

```
$ curl --get "https://developers.cloudflare.com/.well-known/search" --data-urlencode "q=What is the maximum push size for Cloudflare Artifacts?"
# Search results for: "What is the maximum push size for Cloudflare Artifacts?"
## docs/artifacts/platform/limits/index.limits-cloudflare-artifacts-docs.md (score: 1.000)
# Limits · Cloudflare Artifacts docs
Section: Limits · Cloudflare Artifacts docs
title: Limits
description: Review Artifacts platform limits.
pcx_content_type: reference
products:
  - artifacts
Limits that apply to creating, importing, cloning, and pushing Artifacts are detailed below.
These limits cover naming rules, storage, request rates for control-plane and Git operations, and the size of each push.
FeatureLimitControl-plane request rate2,000 requests per 10 seconds per Artifacts namespaceGit request rate, per artifact2,000 requests per 10 seconds per artifactMaximum storage per repository1 GB for repos created on or after 2026-09-29. Repos created earlier have a 10 GB limit. See size limit change.Maximum storage per account1 TB (can be raised on request)Maximum number of repositoriesUnlimitedMaximum number of namespacesUnlimitedNamespace name length2–63 charactersNamespace and repo namesStart with a letter or digit. Remaining characters may include letters, digits, ., _, and -.Maximum push size64 MiB per push. Includes all data sent in the push. Split larger work into several smaller pushes.Maximum objects per push50,000 objects, such as file versions, folders, and commits.Maximum file size32 MiB per file. Already-compressed files, like images, video, or zip archives, may be rejected slightly below 32 MiB.Maximum ref updates per push1 MiB of branch and tag updates per push.Maximum entries per folder100,000 files and subfolders in a single folder.
...
```
Note that this search endpoint returns retrieved content, in Markdown, not a generated answer. The agent already has an AI model and a conversation with the user; all it needs is the relevant information from your site, not another AI model's attempt to answer on its behalf.

### Supporting a new HTTP method: QUERY

The new HTTP QUERY method is an IETF Proposed Standard, incidentally coauthored by our own __James Snell__, published in June 2026 as __RFC 10008__. A QUERY allows a client to send search criteria in a request body safely and idempotently without modifying server state. This makes it a good fit for AI search.

The adoption of QUERY is underway with major browser vendors __discussing__ the RFC and multiple frameworks and clients either adopting (__curl__ and __undici__, for example), or __discussing__ adoption.

Search for Agents supports QUERY natively either using a plain text or JSON request body:

```
# Plain text
curl -X QUERY 'https://developers.cloudflare.com/.well-known/search' \
  --data 'What is the maximum payload size per ingestion request in Basin Pipelines?'
# JSON
curl -X QUERY 'https://developers.cloudflare.com/.well-known/search' \
  --header 'Content-Type: application/json' \
  --data '{"q":"How many bulk import requests per minute does Email Service allow?"}'
```
### See it in action

Let’s see it in action during an agentic session. Search for Agents is enabled for our __Developer Documentation__. When we open an OpenCode session and ask “What is the maximum push size for Cloudflare Artifacts?”, in the logs we can see the agent discovering the search endpoint on the homepage and then fetching **/.well-known/search** with the question for the search results.

The answer comes back a few seconds later and it’s correct.

Once the agent discovers the search API, it will likely reuse it multiple times within the same session whenever it needs to search for content in the same site.

We enabled Search for Agents for our developer documentation ahead of this announcement and we’ve found that agents do a good job of discovering the search API and using it during their tasks.

### It’s still an AI Search instance

Because Search for Agents is powered by an AI Search instance in your account, you get all the standard configuration options and features as well. For example, you can enable public __REST API__ or __MCP__ endpoints, or use it from __Workers__, or even __embed__ AI Search on your website for human visitors.

### Start using Search for Agents

Starting today, Search for Agents is available to all customers. The usage follows the __AI Search pricing__; that includes ingesting content, storing the index's content, and serving queries, subject to the account's included usage and limits for both the Workers Free and Paid plans.

We also recommend you check our __Agent Readiness__ page (see __announcement__) and understand how you can make your sites optimized for agents, from guiding agents on how to authenticate, to controlling what content agents can see, the format they receive it in, and how they pay for it.

Check out the Search for Agents __developer docs__ for more information. __Follow us on X__ or come and talk to us in the __Cloudflare Developer Discord__. We'd like to hear from site owners making their content searchable to agents.
