---
url: https://www.chinatalk.media/p/how-to-buy-cheap-claude-tokens-in
title: "How to Buy Cheap Claude Tokens in China"
author: Zilan Qian
date_fetched: 2026-05-14
date_published: 2026-05-05
---

# How to Buy Cheap Claude Tokens in China

By Zilan Qian, research associate at Oxford China Policy Lab. Published on ChinaTalk (Substack), May 5, 2026.

## Overview

Examines the "transfer station" (中转站) economy — a grey market infrastructure enabling Chinese developers to access Anthropic's Claude models at roughly 10% of official pricing through API proxy networks.

## Geo-blocking and KYC

Anthropic implements multiple access controls: phone verification, overseas credit cards, billing address matching, and since April 2026, live biometric KYC using government-issued photo IDs and selfies. Despite these measures, Chinese users circumvent restrictions through proxy networks.

## Transfer Stations Defined

"Transfer stations (中转站)" are API proxies — overseas servers sitting between users and Anthropic's infrastructure. Users redirect requests through these proxies and pay in Chinese currency (WeChat/Alipay), bypassing direct credit card requirements. Unlike legitimate Western aggregators, these operate explicitly for evasion purposes.

## Supply Chain Structure

Three layers:

**Upstream:** Account merchants bulk-registering accounts, SMS verification platforms providing foreign phone numbers, reverse engineers analyzing authentication systems, payment infrastructure enabling overseas billing.

**Middle:** Proxy interface accepting requests, payment integration, operational management (account cycling, load balancing).

**Downstream:** Individual developers, enterprises, application builders, Taobao resellers repackaging access.

## Three Monetization Methods ("One Fish, Three Meals")

**Meal 1 — Access Markup:**
- Farming Anthropic's $5 free credits through bulk registration
- Reselling unused quota
- Corporate/educational discount arbitrage
- "APImaxxing" — subdividing $200 Max plans among multiple users
- Accounts purchased with fraudulent credit cards

**Meal 2 — Model Swapping:**
Users cannot verify which model processes their requests. A German CISPA study of 17 proxies found "widespread model swapping" — accessing "Gemini-2.5" achieved only 37% on medical benchmarks versus 83.82% on official APIs. Proxies substitute premium models with inferior tiers or competitors' models (GLM, Qwen).

**Meal 3 — Log Harvesting:**
Every request — prompts, responses, tool calls, reasoning chains — resides on proxy servers. These datasets represent high-value training data for fine-tuning and distillation. Several Claude Opus reasoning datasets circulate on HuggingFace with unclear provenance.

## Safety and Monitoring Limitations

Anthropic's Clio system identifies coordinated misuse through cross-account patterns, but proxy routing obscures user IPs behind proxy addresses. Ban enforcement fails when upstream supply chains quickly establish replacement proxies. Distributed attacks staged across multiple proxy accounts remain harder to detect than obvious coordinated spam.

## Downstream Harms

- Biometric data harvested for KYC verification becomes commodities for fraudulent financial accounts, employment fabrication, or deepfakes
- SMS verification and fraudulent registrations feed spam call networks, phishing operations, credit card scams
- Model substitution fraud: users receive inferior outputs while paying premium prices
- Data leakage: developers' proprietary prompts and solutions exposed to blackmail or targeted scams

## Key Data Points

- Chinese developers access Claude at approximately 10% of official pricing (sometimes as low as 5%)
- April 2026 White House memo: Chinese entities running "industrial-scale" distillation using "tens of thousands of proxy accounts"
- February 2026 Anthropic report: Single proxy network managed over 20,000 fraudulent accounts
- Worldcoin biometric black market precedent: iris scans sold for under $30
- Singapore reports highest global per-capita Claude consumption despite smaller population than New York City

## Broader Implications

Access controls — whether geopolitical or safety-focused — create resilient evasion markets. The transfer station economy reveals fundamental limits of IP-based blocking and account-level monitoring as governance tools, with harms extending beyond US-China competition into consumer fraud and data exploitation in the Global South.

The author notes she utilized LLMs for preliminary research and copy-editing while accessing Claude through VPN and Singapore nodes.
