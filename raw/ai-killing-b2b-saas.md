---
url: https://nmn.gl/blog/ai-killing-b2b-saas
date_fetched: 2026-07-05
backfilled: true
---

# AI is Killing B2B SaaS

SaaS is the most profitable business model on Earth.1 It’s easy to understand why: build once, sell the same thing again ad infinitum, and don’t suffer any marginal costs on more sales.

I have been writing software for more than half my life. In the last year itself, I’ve talked to hundreds of founders and operators in SF, from preseed to Series E companies.

AI is bringing an existential threat to a lot of B2B SaaS executives: How to keep asking customers for renewal, when every customer *feels* they can get something better built with vibe-coded AI products?

And the market is pricing it in. Morgan Stanley’s SaaS basket has lagged the Nasdaq by 40 points since December. HubSpot and Klaviyo are down ~30%. Analysts are writing notes titled “No Reasons to Own” software stocks.

## The relation between vibe coding and B2B SaaS sales

The new problem for B2B SaaS is that with AI, customers can get *something* working with vibe coding. There are tens of vibe coding “internal tool” services that promise to connect to every integration in the world to pump out CRUD and workflow apps.

Whatever they build *simply works*. It takes some wrangling to get there (one Series C VP listed **eleven different** vibe coding tools they’ve tried and the pros and cons between each on a phone call once), but productivity gains are immediate.

And vibe coding is fun. You feel like a mad wizard using the right incantation 2 to get this magical new silicon intelligence to do exactly what you want.

What they don’t know, though, is that a poorly architected system will fail, eventually. As every senior programmer (eventually) understands, our job is complex because we have to understand the relationships in the real world, the processes involved, and the workflows needed, and representing it in a robust way to create a stable system. AI can’t do that.

Non-programmers don’t know any of this nuance. One Series E CEO told me that they’re re-evaluating the quarterly renewal of their engineering productivity software because they along with an engineer reimplemented something using Github and Notion APIs. They were paying $30,000 to a popular tool3 **and they were not going to renew anymore.**

## How does it impact B2B sales?

If customers feel like they aren’t being served exactly like they want to, they are more likely to churn. The reason behind all this is that customers are demanding more from their B2B vendors, because they know what’s possible.

Previously, you would change *your company* to fit what your ERP and pay them hundreds of thousands of dollars. Now, everyone can see that agentic coding makes an unprecedented level of flexibility possible. And customers are demanding that flexibility, and if they don’t get it, they’ll leave.

This week itself I was on a phone call with a Series B AE talking about how they’re potentially losing an $X00,000 account just because the customer can’t use a specific failure reporting workflow in the SaaS. They’re now working with me to build what the customer needs and retain them.

## How to survive

### 1. Be a System of Record

If the entire company’s workflows operates on your platform, i.e. you’re a line-of-business SaaS, you are integrated into their existing team already. They know your UI and rely on you on the day to day.

For example, to create a data visualization I won’t seek any SaaS. I’ll just code one myself using many of the popular vibe coding tools *(my team actually did that and it’s vastly more flexible than what we’d get off-the-shelf).*

Being a “System of Record” means you’re embedded so deeply that there’s no choice but to win. My prediction is that we’ll see more SaaS companies go from the application layer to offering their robust SoR as their primary selling point.

### 2. Security, authentication, and robustness

This is where vibe-coded apps silently fail — and where established SaaS platforms earn their keep.

When a non-technical team vibe-codes an internal tool, they’re not thinking about environment keys, XSS vulnerabilities or API keys hardcoded in client-side JavaScript. They’re not implementing rate limiting, audit logs, or proper session management. They’re definitely not thinking about SOC 2 compliance, GDPR data residency requirements, or HIPAA audit trails.

I’ve seen it firsthand: a finance team built a “quick” expense approval tool that stored unencrypted reports in a public S3 bucket. A sales ops team created a commission calculator that anyone with the URL could access — no auth required. These aren’t edge cases. They’re the norm when software is built without security as a foundational concern.

Enterprise SaaS platforms have spent years (and millions) solving these problems: role-based access control, encryption at rest and in transit, penetration testing, compliance certifications, incident response procedures. Your customers may not consciously value this — until something breaks.

The challenge is that security is invisible when it works. You need to communicate this value proactively: remind customers that the “simple” tool they could vibe-code themselves would require them to also handle auth, permissions, backups, uptime, and compliance.

### 3. Adapt to the customer, not the other way around

The times of asking customers to change how they work are gone. Now, SaaS vendors that differentiate by being ultra customizable win the hearts of customers.

How? It’s the most powerful secret to increase usage. We’ve all heard the classic SaaS problem where the software is sold at the beginning of the year, but no one actually ends up using it because of how inflexible it is and the amount of training needed.

And if a SaaS is underutilized, it gets noticed. And that leads to churn. And just adding random AI features to your SaaS won’t cut it.

This is the case with one of my customers, they have a complex SaaS for maintenance operations. But turns out, this was not being used at the technician level because they found the UI too complex4.

How I’m solving this is essentially a whitelabelled vibe-coding platform with in-built distribution and secure deployments. When they heard of my solution they were immediately onboard. Their customer success teams quickly coded a very specific mobile webapp for the technicians to use and deployed it in a few days.

Now, the IC technician is exposed to just those parts of the SaaS that they care about i.e. creating maintenance work orders. The executives get what they want too, vibe coding custom reports exactly the way they want vs going through complicated BI config. They are able to build exactly what they want and feel like digital gods while doing it.

**Usage for that account was under 35%, and is now over 70%.** They are now working closely with me to vibe code new “micro-apps” that work according to all of their customer workflows. And the best part? This is all on top of their existing SaaS which works as a system of record and handles security, authentication, and supports lock-in by being a data and a UI moat.

This is exactly what I’m building: a way for SaaS companies to let their end-users vibe code on top of their platform (More on that below). My customers tell me it’s the best thing they’ve done for retention, engagement, and expansion in 2026 – because when your users are building on your platform, they’re not evaluating your competitors.

## The Real Shift

Here’s what I’ve realized after hundreds of conversations with founders and operators: AI isn’t killing B2B SaaS. It’s killing B2B SaaS **that refuses to evolve**.

The SaaS model was built on a simple premise: we build it once, you pay forever. That worked when building software was hard. But now your customers have tasted what’s possible. They’ve seen their finance team whip up a custom dashboard in an afternoon. They’ve watched a non-technical PM build an internal tool that actually fits their workflow.

You can’t unsee that. You can’t go back to paying $X0,000/year for software that almost does what you need.

The survivors won’t be the SaaS companies with the best features. They’ll be the ones who become platforms – who let customers build *on top of* them instead of *instead of* them. When I showed a well-known VC what I was building to help SaaS companies do exactly this, he said: “This is the future of marketplaces and software companies.”

Maybe. Or maybe this is just another cycle and traditional SaaS will adapt like it always has. But I know this: the companies I’m talking to aren’t waiting around to find out. They’re already rebuilding their relationship with customers from “use our product” to “build on our platform.”

The question isn’t whether AI will eat your SaaS.

It’s whether you’ll be the one holding the fork.

**I’m solving exactly this problem with a whitelabelled AI platform for B2B SaaS companies**, so your users can vibe code customized workflows on top of their existing system of record.

My customers tell me this is the **best way to support retention, engagement, and expansion** in 2026. If this sounds interesting to you or someone you know, I can reach out with a custom demo or you can learn more about Gigacatalyst.

- 
      Whenever I bring a new friend to the Salesforce Park, they are in absolute awe. And, the meme remains true that no one even knows what Salesforce does. Whatever they’re doing, they’re clearly earning enough revenue to purchase multiple blocks in SF. ↩ 
- 
      a.k.a. “prompt engineering” which is not engineering at all but that’s a different blog post. ↩ 
- 
      I won’t name any names, but the company’s named after an invertebrate animal. ↩ 
- 
      And who can blame them – I still feel a pang of anxiety when I look at my sales CRM. ↩ 

## Give your sales team superpowers

Gigacatalyst allows your sales team to build features that prospects ask for, so they never lose a deal to product gaps.

My customers say that this is the best way to **increase sales** in 2026.
