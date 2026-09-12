---
title: "AI Coding Tools Create More Bugs Than They Fix"
author: "Darryl K. Taft"
source: "https://thenewstack.io/ai-coding-tools-create-more-bugs-than-they-fix/"
published: 2025-07-02
retrieved: 2026-05-14
type: article
tags: [ai-security, vibe-coding, vulnerabilities, mobb]
topics:
  - guardrails-and-feedback-loops
---

# AI Coding Tools Create More Bugs Than They Fix

By Darryl K. Taft, The New Stack, July 2, 2025.

## Summary

Mobb, a provider of automatic security vulnerability remediation technology, has launched new tools to secure AI-generated code without slowing down development.

AI-powered coding tools and the so-called "vibe coding" craze have enabled developers to build more code more swiftly than ever before, but lurking underneath all that productivity lie unrecognized security risks.

Platforms like Lovable, Bolt and Cursor are enabling anyone to build functional applications in minutes, while professional developers are leveraging AI assistants to code faster.

However, recent research reveals that more than 40% of AI-generated applications are inadvertently exposing sensitive user data to the public internet. Even more alarming is that AI coding assistants are consistently introducing vulnerabilities into professional codebases, Tomer Cohen, vice president of product at Mobb, told The New Stack.

## Mobb's Two-Pronged Solution

The Boston-based startup is taking a dual approach to tackle both sides of the AI coding spectrum: casual builders using no-code platforms and professional developers using AI assistants to augment their efforts.

Mobb is launching two complementary solutions: SafeVibe.Codes and Mobb Vibe Shield.

**SafeVibe.Codes** targets the no-code security crisis with a free web-based scanner. Users simply paste their application URL to receive an instant security analysis showing exposed databases, leaked personal information and misconfigured permissions. The tool provides actionable guidance for fixing identified issues without requiring technical expertise.

"Security shouldn't be a luxury available only to those with extensive technical knowledge or large budgets," Cohen explained. "By making SafeVibe.Codes completely free and accessible, we're democratizing security testing just as AI platforms have democratized app development."

"SafeVibe.Codes is an application we launched as a free service for the industry to quickly identify certain types of vulnerabilities in apps they built with some of the leading vibe coding platforms," Eitan Worcel, co-founder and CEO of Mobb, told The New Stack. "We are great supporters of vibe coding and see immense potential in it. In fact, the [SafeVibe.Codes] site itself was built with Bolt, which is one of those platforms."

The platform scans for:
- Database exposure and unauthorized access
- Sensitive data leakage
- Permission misconfigurations
- Hidden pages (e.g., /admin)
- Common API security vulnerabilities (coming soon)
- Data validation issues (coming soon)

Future testing coverage will include real-time monitoring, page exposure scanning, AI prompt exposure detection, full code security analysis, compliance checking, and a security education hub.

**Mobb Vibe Shield** addresses the professional developer side through IDE integration -- it works with VS Code, GitHub Copilot, Cursor, Windsurf, JetBrains IDEs and Cline (with others to come). The tool continuously scans code as it is written, automatically detecting vulnerabilities introduced by AI assistants and applying verified security fixes in real time. Using the Model Context Protocol (MCP), it integrates easily with existing AI coding workflows.

The system does not rely on AI to fix security issues it detects. Instead, it applies pre-verified security patches developed by Mobb's security experts, ensuring that fixes resolve vulnerabilities rather than creating new ones.

"We see it happening in companies of all sizes and across all industries. Neither the developers nor the security teams are equipped to handle the huge amount of new code and code vulnerabilities generated, and existing code scanning tools being slow and noisy weren't designed to handle it either," Worcel told The New Stack.

## The Hidden Crisis in No-Code Platforms

Mobb's research team conducted an extensive security analysis across five major AI development platforms: Lovable, Bolt, Base44, Replit and v0.

"More than 40% of applications we tested across all platforms contained some level of sensitive data exposure," Cohen said. "This means that nearly half of all AI-generated apps we examined were unintentionally sharing private information with the public internet."

The types of exposed data include personal information like names, emails and phone numbers, as well as financial records, private communications and authentication credentials. In roughly 20% of cases, anonymous users could not only view this data but also modify or delete it entirely.

## It Gets Even Worse

During an interview, Cohen demonstrated the process of using Base44 to build a gym booking system. The AI created a functional application complete with class schedules and member registration. But by default, it also created publicly accessible databases containing every member's personal information -- with no warnings about the security implications.

"At no point does the platform warn you about these security implications," Cohen wrote. "There's no indication that sensitive personal information is being collected and stored in a publicly accessible database."

Perhaps even more troubling is what happens when users try to fix these issues. When developers attempt to enable proper security controls, the applications often break. When they ask the AI to fix the broken functionality, the platforms frequently respond by removing the security measures entirely -- without explaining the trade-offs being made.

"When attempting to fix the security issue by enabling Row Level Security (RLS) settings to restrict database access, in many cases the application immediately breaks. The AI-generated code is not designed to work with proper security controls.

"When asking the AI agent to fix the broken functionality, the platform's response is alarming: in many cases it simply removed my RLS restrictions and reopened the data to the world -- without any warning that this was dangerous or explaining the security implications. The AI prioritized making the app 'work' over keeping the data secure, effectively undoing my security improvements."

## Introducing Vulnerabilities in Professional Codebases

In testing, Mobb researchers found that asking AI assistants to implement common development tasks -- like creating an endpoint for code commits -- resulted in command injection vulnerabilities with a 100% reproduction rate. These flaws could allow attackers to execute arbitrary commands on production servers.

Additionally, the AI tools often claim to have implemented security measures. "It actually tells me that it did something to prevent command injection," Cohen said. "So, developers, even experienced, responsible developers, could be tricked by this, because it looks very convincing."

This creates a false sense of security where developers believe their AI assistant has properly secured their code, when in reality, it has introduced vulnerabilities while appearing to address security concerns.

"This isn't meant to discourage the use of AI development tools -- they're powerful and democratizing technologies," Cohen wrote. "Instead, it's a call to action for both platform providers and developers to prioritize security alongside functionality."
