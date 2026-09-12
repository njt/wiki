---
url: https://surfingcomplexity.blog/2026/08/19/github-autoscaling-and-the-component-substitution-fallacy/
title: Autoscaling and the Component Substitution Fallacy
author: Lorin Hochstein
date_fetched: 2026-09-04
date_published: 2026-08-19
topics:
  - software-engineering-craft
---

# Autoscaling and the Component Substitution Fallacy

Lorin Hochstein digs into one detail from GitHub's recent outage writeup: the autoscaling policy on the service whose Istio sidecar saturated. The policy watched load on the host service but not on the sidecar, so the sidecar hit its concurrency limits without ever triggering a scale-up.

Hochstein uses the case to make two points. First, autoscaling is hard to get right: a service can saturate while CPU stays low (Slack's 2021 thread-pool incident is the canonical example), and because every service behaves differently under load, every autoscaling policy is effectively bespoke — a custom control system that service owners, who are almost never autoscaling experts, must tune and can only verify through load testing.

Second, and more important, fixating on the broken component is a trap. Hochstein invokes David Woods's **component substitution fallacy**: the belief that reliability improves by identifying and fixing defective components. But every production system is full of latent defects and yet is not constantly failing over — so component defects alone can't be what takes a system down. The real failure lives in the *interactions*: in GitHub's case, changing traffic (scrapers), the autoscaling policy, sidecar saturation, retry logic, HAProxy node saturation, and authentication traffic, all colliding. Public incident writeups don't give you the history and traffic detail needed to trace those interactions — but internal writeups at your own organization can, if you ask the questions.
