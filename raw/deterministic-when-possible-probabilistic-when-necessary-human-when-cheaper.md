---
url: https://codemanship.wordpress.com/2026/09/30/deterministic-when-possible-probabilistic-when-necessary-human-when-cheaper/
date_fetched: 2026-10-03
---

One inescapable conclusion I’ve had to draw observing our industry over the last couple of years is that the AI thing isn’t about productivity. If it was, teams would be looking for ways to improve outcomes. But that’s not what I’ve been seeing.

I’ve seen dev teams setting out with the specific goal to use AI coding agents as much as possible, with the impact a secondary (or even a thirdary) concern.

I’ve watched developers prompting Claude Code to rename classes or methods when there’s a shortcut in their editor that will do it with a fraction of the keystrokes, using a tiny, tiny fraction of the compute and the energy, and do it more reliably.

I’ve watched them task Copilot with finding unused code when background compilation has already identified it.

I’ve watched them explain the code they want to Codex in more words than the code itself, and the model still gets it wrong.

Arguably these people have lost the plot.

And there are those other teams; the ones who use the best tool for the job. Need a refactoring? IntelliJ or Rider or PyCharm’s got you covered *most *of the time. Wanna know where the unused code is? Your IDE’s showing you. And if you don’t have that feature, a linter will do it lickety-split without the need for a £20,000 GPU and a terabyte of VRAM. If you know what code you need, maybe just write it. M’kay?

And then they hit a gap in their tooling. They need to move an instance method in Python. PyCharm doesn’t have that refactoring. So they go the agent window:

> Move the method calculateDiscount from the Order class to the Product class

And – 90% of the time – the model will do what they need. (And, annoyingly, sometimes more than they need.)

To perform the refactoring by hand would usually take longer, so they make a *rational choice* to throw the dice if it will save some time.

These are the teams whose outcomes have improved. Lead times and release cycles have shrunk a little, and it hasn’t been at the price of reliability.

And that’s what this is supposed to be about, right? *Better outcomes*. Bang for the buck. Or should I say “Bang for the token”?
