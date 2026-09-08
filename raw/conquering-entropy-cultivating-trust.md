---
url: https://kaeruct.github.io/posts/2026/09/06/conquering-entropy-cultivating-trust/
date_fetched: 2026-09-08
---

# Conquering Entropy: Cultivating Trust

 2026-09-06

Part of “Conquering Entropy”


Most of the issues I have with AI-generated code are related to trust. Do I trust the person who wrote this ticket? Do I trust that the engineer who opened this PR understood the ticket and guided the coding agent to implement it properly? Do I trust the coding agent’s implementation? Do I trust our test suite to catch regressions before they hit production? Do I trust our CI/CD to properly build, test, and deploy our change? Do I trust our observability setup to alert us when the ai-generated code breaks production? Do I trust the AI SRE (Site Reliability Engineer) to properly diagnose the issue and help us mitigate it? Do I trust GitHub not to have an incident when we need it the most?

Trust is hard to earn and easy to lose. So I think it’s crucial to foster a culture of trust within the engineering team. You must trust engineers to do the right thing.

## Encouraging Accountability

I find it helpful to clearly communicate something like this: “You are accountable for what you ship. If this breaks production and you authored the PR, you should be there to fix it.”

If engineers are made accountable for the code they ship, they should be given the agency to produce it with their preferred methods. Even if you’re not AI-pilled, you have to admit that AI agents do generate a lot of code very fast. Code still needs to be generated, and the expectation is that now it’s cheap to generate a lot of it. To cope with this, engineers must be allowed to put some measures in place to ensure the quality of the codebase does not degrade.

In a healthy organization, the engineers should trust each other to only push code of reasonable quality. I say reasonable because it’s not pragmatic to obsess over quality and try to ship always 100% perfect code. Even before coding agents most code was already a buggy mess! So it’s understandable when a “good enough” solution is delivered. Often, we trade speed of delivery for quality, and incur some technical debt.

Here are some practices I have found useful to facilitate engineering accountability in this new era of coding:

### Coding Guidelines

Have a clear technology strategy: take some time to decide what matters for your codebase and invest in clear guidelines. Both for humans and for the coding agents. Even if the humans don’t read the guidelines, their coding agents will, and will follow them (mostly).

### Deterministic Tooling

Make heavy use of deterministic tooling to ensure code quality. Typed languages, linters, dead code detection, security scans, CI/CD, and so on. All of these tools existed before coding agents and help us in the fight against slop. (Stay tuned for a follow-up post with specific recommendations!)

### Enforce Small PRs

Empower the engineers to reject unreviewable PRs. If possible, codify this criteria so any unreviewable PRs are immediately rejected. Of course, make sure to leave room for exceptions.

### Own the Tests

Write test cases by hand. This is similar to the job of a business analyst. Deeply think about the feature and define proper test scenarios. Discuss this within the team. Coding agents can implement the tests, but they should be defined by humans.

### Taste in Product

Work in tandem with product to have a coherent product vision. It’s very easy to go crazy with AI and implement whatever feature comes to mind. Make sure you only implement useful features that actually bring value to users!

### Prototype

Throwaway code. Since code is very easy to generate now, it’s a good opportunity to try different approaches. Don’t just let the coding agent generate one solution. For example, try three radically different approaches and pick the one that best fits the problem and existing system.

### Focus on the Outcome

Be pragmatic about the result. Sometimes the code is not the result. The result is a report, or a tool that helps you accomplish something else. For cases like this the quality of the code is not that relevant as long as the result is useful.
