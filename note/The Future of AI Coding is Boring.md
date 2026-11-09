# The Future of AI Coding is Boring

A pessimistic practitioner essay (packagemain.tech, November 2026) arguing that AI coding has won but created more problems than it solved, and that the future of software engineering is "boring": the engineer as caretaker of code nobody understands. Three named problems — how humans learn when the shortcut skips the brain, inevitable codebase erosion from accepted AI code, and AI slop in communication plus accelerated managerial pressure — with a comment section that adds developer apathy and predicts human-authored code shrinks to a hobby and high-criticality niches.

---

## What it argues

The author frames the shift as a change of profession rather than a change of tool: "the programmer profession has ended, or changed so much that it's just a new profession not fully defined yet." The chef analogy — years perfecting a craft, then finding your job is tasting machine-prepared meals — carries the emotional weight of the piece.

The three problems are deliberately left open, "without much answers too, I want to leave it open for discussion":

- **Learning.** The old causal chain `Problem → Research/Reading/Brain → Solution` has collapsed to `Problem → Solution`. Answers from Stack Overflow stuck "for years, because that's how our brain works when we 'use' it" — the struggle was the encoding. The scrape-and-generate economy took that corpus and gave nothing back.
- **Codebase erosion.** Review still works at first, but "no matter what you do, if you started accepting AI-generated code, your codebase will eventually erode." Even the models now struggle with the resulting mess. Review is skipped or delegated to AI stamping LGTM. And the feature request is now so cheap that codebases fill with unused and conflicting features nobody prunes.
- **Noise and management.** Slop isn't confined to code; AI-written docs and memos shift the deciphering burden onto the recipient. Managers, believing "generating code was ever a blocker," push for more — recursively feeding erosion and noise.

## Key quotes

> Give a reviewer a 10-line Pull Request and he will find 10 issues, give him a 500-line one, and he will say LGTM.

The author notes this was true of big PRs *before* AI — the review-bottleneck literature has since quantified it, but the aphorism is the mechanism by which agent-scale PR volume destroys human review.

> We used to spend time solving the problems with code, now we've become a caretaker of code that nobody really understands.

This is the essay's thesis in one line, and it names the least-discussed cost: not slower delivery, but a job that has lost its point.

> No one is passionate about babysitting Claude.

From the comments, and arguably the sharpest addition: developer apathy as a fourth problem. The "aha!" moment was the compensation for the hard parts; remove the hard parts and the motivation model breaks.

## Themes

#concept #comparison

## Analysis

This is not a novel argument — versions of it exist across the wiki — but it is a compact and unusually honest statement of it, and its strengths are structural rather than empirical. The "drug for the codebase" framing gets the timing right: the AI boost is front-loaded and the erosion is back-loaded, which is exactly why adoption decisions look rational in the quarter they're made and irrational two years later. It also correctly identifies that learning is a first-order casualty, not a sentimental footnote — the collapse of the research loop is a real mechanism, not nostalgia.

Its weaknesses: it offers no answers (by explicit choice, which is fine for a discussion piece but limits its utility), and its claims are unfalsifiable anecdotes. The erosion thesis in particular is stated as inevitable without engaging the counter-evidence — refactoring experiments and guardrail systems in this wiki suggest erosion is a rate to be managed, not a certainty. The "software engineering was never a field" closer in the quoted comment is more provocation than argument.

Still, as a record of how a mid-career engineer *feels* in late 2026, it is valuable precisely because it refuses optimism: prestige gone, review hollowed, slop everywhere, passion draining. That is a mood the productivity-survey literature measures but rarely voices.

## Relations

The erosion-and-caretaking diagnosis strengthens [[Agents and Acquiring Debt]] — the LGTM-stamping review path this essay blames is exactly the mechanism that article formalises as comprehension debt, the highest-interest debt in the agent era. It complicates [[The Joy and Power of Understanding]]: where Roztropiński argues understanding is still available to those who insist on it, this author observes the default path now skips the brain entirely — the two agree on the mechanism and disagree on whether escape is practical. It gives the emotional ground truth behind [[Human-in-the-Loop is Tired]]: Summers' supervision fatigue is the formal version of "no one is passionate about babysitting Claude." And its 500-line-LGTM quote is the same rubber-stamp collapse that [[The End of Code Review]] treats as the reason mandatory human review is already indefensible.

---
*Sources: [[raw/the-future-of-software-engineering]], [[summary/the-future-of-software-engineering]]*
*Last updated: 2026-11-09*
