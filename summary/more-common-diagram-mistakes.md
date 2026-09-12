---
title: "7 More Common Mistakes in Architecture Diagrams"
author: "Billy Pilger"
source: "https://www.ilograph.com/blog/posts/more-common-diagram-mistakes/"
date_published: 2026-03-12
date_fetched: 2026-05-14
type: blog post
topics:
  - software-engineering-craft
---

# 7 More Common Mistakes in Architecture Diagrams

**Author:** Billy Pilger (Ilograph)
**Date:** March 12, 2026
**Read Time:** 6 minutes

Follow-up to an earlier piece on common diagram mistakes.

## Mistake #1: Not Including Resource Names

Resources should be labeled by both type and name. Types describe what kind of thing a resource is (databases, VM instances, services). Names differentiate resources from others of the same type and reveal purpose.

When space permits, diagrams benefit from showing both elements. A practical approach involves adding type suffixes to names, such as "Orders Table" or "Results Bucket." Since icons typically indicate type, prioritizing resource names becomes especially important.

## Mistake #2: Unconnected Resources

Every resource in a diagram should connect to other resources somehow. Including isolated elements undermines the diagram's purpose of showing relationships. This problem frequently arises when creators attempt to include too much information in one diagram.

## Mistake #3: Making a "Master" Diagram

Attempting to display an entire system in one diagram typically overwhelms viewers. Such diagrams often combine runtime dependencies, DNS configuration, CDN setup, source code, and deployment dependencies simultaneously.

Solution: Break diagrams into multiple perspectives, each telling a cohesive story. Model-based diagramming allows perspectives to share resources while maintaining connections.

## Mistake #4: Conveyor Belt Syndrome

This occurs in behavioral diagrams (showing specific interactions) when creators over-simplify by omitting real-world round-trips and orchestrations. The result misleads viewers about actual system operations.

Example: A diagram showing data flowing linearly through resources suggests each one simply processes and passes along input -- rarely accurate.

Solution: Use sequence diagrams instead, which are specifically designed to show detailed back-and-forth interactions between components. These provide both greater detail and fidelity.

## Mistake #5: Meaningless Animations

Animated diagrams with moving arrows that add no technical information primarily serve marketing purposes. These animations often prove redundant, simply indicating directions already shown by the arrows themselves.

Recommendation: Avoid unnecessary animations unless creating diagrams exclusively for marketing.

## Mistake #6: Fan Traps

Fan traps occur when relation information between resources gets lost through intermediate resources. This happens in event-based systems when specific communications between edge resources collapse onto a shared message broker.

Solution: Add more specific resources (such as topics) within intermediate components and reroute relations through them, restoring visibility of communication paths.

## Mistake #7: Assuming AI Can Create Quality Diagrams from Source Code

While AI can assist during interactive whiteboarding, automatically generating diagrams from source code remains problematic. AI-generated diagrams often suffer from vagueness, hallucinations, and exhibit many issues discussed in this article.

Challenges include:
- Almost complete lack of training data
- Difficulties analyzing dense source code
- Inability to strategically choose what to include or omit

System diagramming remains primarily a human endeavor, though this may evolve as AI improves.
