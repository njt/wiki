---
url: https://leerob.com/agents
title: "Coding Agents & Complexity Budgets"
author: Lee Robinson
date_fetched: 2026-05-14
date_published: 2025-12
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

Lee Robinson recounts how he migrated cursor.com from a headless CMS back to raw code and Markdown. What he expected to take weeks with an agency was "able to finish the migration in three days with $260 in tokens and hundreds of agents."

## Content is just code

Robinson describes a lunch conversation with Roman and Eric at Cursor where the team aired grievances about their recently redesigned site now running on a headless CMS. While the design was beautiful, shipping new content had become harder.

Previously they could "@cursor and ask it to modify the code and content," but the CMS added friction, forcing "clicking through UI menus versus asking agents to do things for us."

He argues that "with AI and coding agents, the cost of an abstraction has never been higher" and questioned whether a CMS was truly necessary if people could use a chatbot instead of a GUI.

Roman asked for a timeline estimate. Robinson guessed 1-2 weeks, possibly with an agency. After creating a migration plan in Cursor, he was "surprised by how far it got" and was nerdsniped into tackling the migration over a weekend.

## Removing complexity

The cursor.com site is a standard Next.js/React app. The CMS was added so non-developers could build marketing pages and write blog posts -- a standard pattern. But Robinson notes the hidden technical complexity of integrating a headless CMS well.

### 1. User management

User management existed in two places. If "designers are developers," marketing teams can live in GitHub too -- the team already had GitHub SSO and RBAC. New members would ask to "be added to the CMS please." Instead, they added marketing to GitHub: "One account management system."

### 2. Previewing changes

A fast marketing site ideally prerenders statically, which removes "a ton of operational complexity" and avoids downtime when the CMS has availability issues. Next.js draft mode with Vercel's toolbar works but consumes a lot of the "complexity budget." Viewing draft URLs requires Vercel accounts and SCIM management. When "content is code, you can create a PR, get a link with your changes, and share it with anyone. No login is required."

### 3. Internationalization

i18n support requires significant effort. Robinson's team previously used AI during the build step for automatic localized translations of their docs, with locking mechanisms. The CMS complicated this: they "had to hire contractors to help build a plugin system" for localization. A blog post like `/blog/2-0` needed a variation per language, creating "a bunch of items in the CMS" and a complex publishing process. His conclusion: "Define the source in code, and then use compilers and AI."

### 4. CDN and asset delivery

Blogs and web pages need images and videos, so CMS providers also serve as CDNs for static assets -- and a major revenue source. After launching the CMS-backed site, they incurred costs for bandwidth, API requests, and CDN requests. The total: **$56,848 on CDN usage** since September. Robinson notes there are "plenty of ways to serve assets at more affordable prices" and that teams "pay a hefty markup for the convenience of the GUI." They moved assets to their own object storage and built a small upload GUI in just a few prompts.

### 5. Dependency and abstraction bloat

CMS usage requires proprietary content storage and rendering formats, but "it all turns into the same DOM elements and images." The codebase became bloated and harder to maintain. He gives the example of a `Navbar` component fetching navigation data from the CMS over the network. The problem: "Agents can't use their tools to grep and edit the code. That network boundary is costly."

He shows three iterations: first fetching from CMS, then inlining as a mapped array, then rendering plain JSX with Tailwind classes. He voices a personal React gripe about "turning everything into an array and then mapping over it," arguing that with Tailwind, "copy-paste is better than the wrong abstraction." Changing navigation is now "a single coding agent prompt away."

## Doing the migration

Robinson created a plan with Opus 4.5, which suggested using the existing CMS API key to write export scripts rather than manually downloading through menus. The plan: export content, validate structure, convert to Markdown and repo files, and upload images to object storage.

He "got 80% of the way there with probably 10 agent runs." Cursor installed and removed dependencies, ran scripts, and built pages of content. The last 20% took most of the time, but he was already nerdsniped. For pages that weren't an exact match, he ran agents with instructions to compare local against production via screenshots and "keep iterating until the local version is a perfect match."

As he kicked off agents, he broadened the refactor. He used subagents with Opus to create a plan for the desired API shape, then ran subagents in parallel across many call sites.

## Happy little accidents

In his enthusiasm to remove complexity, Robinson "also deleted Storybook entirely." It had nice features but was barely used, and with Cursor's browser for visual editing, he didn't see the value. Storybook meant "a lot of dependencies being downloaded and installed on every machine and CI run."

He built a simple component playground alternative and a GUI for managing assets on object storage -- "3 or 4 prompts with the agent to get something decent and workable."

A bonus of content-in-code: all changes flow through git, which is "incredibly helpful for coding agents to dig through autonomously."

## Results

Cursor calculated the usage via its own APIs:

- **$260.32** and 297.4M tokens (mostly cached)
- **344 agent requests**
- 66 manual Tab changes
- 67 commits (+43K / -322K lines)

What Robinson thought would take weeks and an agency was done for $260 in tokens (or one $200/month Cursor plan).

The migration paid off immediately. The first day after, he "merged a fix to the website from a cloud agent on my phone." The next day, an engineer shipped a cross-product feature in a single PR. They're "saving thousands of dollars in CDN usage" with lower-cost object storage, and build times are "2x faster by cutting out network I/O going to the CMS."

Robinson closes by reflecting that "the cost of abstractions with AI is very high" and that "over abstraction was always annoying and a code smell but now there's an easy solution: spend tokens." The migration "already paid for itself," and he envisions coding agents helping teams "try their wildest ideas, and fix tech debt that was buried deep in the backlog," leading to "a world of abundant, high-quality software."
