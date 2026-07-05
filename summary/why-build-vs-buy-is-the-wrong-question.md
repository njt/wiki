---
url: https://quii.dev/Why_%22Build_vs_Buy%22_is_the_Wrong_Question
title: Why "Build vs Buy" is the Wrong Question
author: Chris James
date_fetched: 2026-07-05
date_published: 2026-07-02
---

# Why "Build vs Buy" is the Wrong Question

**Author:** Chris James
**Published:** 02 July 2026

## Opening

James opens with a familiar enterprise IT scenario: a well-meaning leader questions whether talented, expensive engineers should build something that doesn't seem like a "competitive advantage," suggesting an off-the-shelf solution instead. He argues the intent is good but misses important nuance.

## What is our core?

He introduces Eric Evans's 2003 book *Domain Driven Design*, describing it as much about business architecture as technical software modeling. Evans's concept of a **business domain** is central — James's current company operates in scientific publishing. Within any domain lie smaller "subdomains," which he classifies into three types that should guide build-vs-buy decisions.

### Generic subdomains (buy)

These are necessary but not competitively differentiating — payroll, email infrastructure, etc. He calls it foolish to build these in-house: "You would be foolish to invest the time, risk and so on of building these systems yourself." The expertise already exists elsewhere, and building internally creates massive opportunity cost.

### Core subdomains (build)

The opposite: these are where your business differentiates and succeeds. He notes that if you could buy your core, so could competitors, eroding your advantage. Building in-house gives you control over knowledge, expertise, and the ability to evolve with the market.

### The third subdomain, "supporting"

James argues the build-vs-buy debate typically becomes a binary fight between generic and core, missing Evans's identified gap. Supporting subdomains are "not generic enough to buy off the shelf, nor valuable enough to be core." They support core domains without improving the business — a cost of doing business, often invisible to senior leadership.

#### The problem with vendoring a supporting domain

The buy argument — "Don't invest engineering effort in things that aren't core" — sounds reasonable. But when you vendor a supporting domain, you're not buying expertise. These subdomains tend to be "boring" (uninteresting, easy), and James notes others have suggested these systems make good training grounds for newer engineers.

#### The hidden cost of integration

Build-vs-buy comparisons consistently "downplay the cost of integration, because the cost feels invisible down the line." It shows up as friction — "death by a thousand cuts" — in the form of mapping complexity, lead time, vendor upgrades, misleading documentation. It doesn't appear as a line item but as frustration with engineering capacity. Additionally, teams build anticorruption layers around vendors to manage failure modes and breaking changes, preserving the option to switch. This is sound practice, but it's not free and not counted in the purchase price.

#### The gravitational pull of vendors

A further issue: supporting subdomains are typically too small and specific to have a dedicated vendor — "Nobody builds their business on solving your exact problem." So you buy something far larger that covers your use case only incidentally, and "pay for everything else indefinitely." That excess feature surface then gets retroactively justified. James questions whether those extras are real opportunity or "a sunk cost you're rationalising before you've even signed the contract."

#### JFDI (Just f*cking do it)

The irony, he writes, is that trying to avoid investing in a supporting subdomain leads to investing more — just in things with zero value to your core business. Building it yourself is "simple and finite" — beyond care and maintenance, you're done. The anticorruption layer vanishes: "When you own the supporting subdomain, there is nothing to protect your core from. You are the vendor."

#### The obligatory AI angle

Using Evans's subdomain lens clarifies where AI fits:

- **Generic:** The more bullish AI proponents argue SaaS is doomed because AI can build these things, but James doesn't see CIOs rushing to replace HR software with vibe-coded solutions. "AI does not replace expertise, it augments it" — you're buying expertise, not just code.
- **Core:** AI can accelerate delivery but "cant substitute for the deep domain understanding that makes your core valuable in the first place." Problem understanding matters more than coding speed.
- **Supporting:** Clear win for AI — requirements are simple, stable, and resemble problems the AI was trained on.

He notes the irony: the same people pushing "use AI to go faster" are often the same ones saying "buy don't build" for supporting subdomains — where AI would most reduce the cost of building.

## Wrapping up

James's closing advice for future build-vs-buy discussions: change the frame. Ask "What kind of subdomain is this?" The answer won't always be obvious, and people will disagree — but without asking the question, the discussion will lack the nuance needed for a good decision.
