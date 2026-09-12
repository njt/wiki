---
url: https://qntm.org/ratchet
title: Ratchets in Software Development
author: qntm (Sam Hughes)
date_fetched: 2026-05-15
date_published: 2021-11-21
topics:
  - guardrails-and-feedback-loops
---

So there's a thing we use at work which I call a ratchet.

In our codebase, there are "patterns" which we used to use all the time, but we decided to stop using them, but removing all of the existing instances at once is too much work. We want to remove all of these instances eventually, and in the meantime we want to make absolutely sure that they don't proliferate via copy-and-paste. So what we have is a ratchet, a script which runs at source code linting time and counts all of these "pattern" instances across the codebase. If the script counts too many instances, it raises an error, explaining why we don't want more of that "pattern". If it counts too few, it also raises an error, this time congratulating you and prompting you to lower the expected number.

This script is intentionally extremely simple. The expected numbers are hard-coded in the script itself. The "patterns" for which it scans our code are not advanced, abstract Gang of Four-style software design patterns but plain text strings.

At the time of writing, the strings are mostly the names of methods whose usage we frown upon. The methods in question aren't our own. They are public methods of first- and third-party libraries we use. We can't apply deprecation warnings upstream. Nor would we. These are perfectly acceptable and normal methods for people in general to use. It's only within the scope of our own specific codebase that we've decided to try to quit using them.

The script carries out extremely basic string matching. There is no source code parsing. So there are some obvious edge cases. What if someone wants to talk about THE FORBIDDEN METHOD in a comment, say? What if it shows up in a string literal? Answer:

1. [shrug]
2. It hasn't come up
3. I guess we'd just raise the ratchet by 1
4. Oh, but make sure the scanning script doesn't scan itself

One important observation is that this technique does nothing to actively encourage the removal of these old "patterns". Those remaining 67 or so calls to THE FORBIDDEN METHOD have been kind of lingering. But perhaps that's a different problem.

Sometimes, due to extenuating circumstances, we have had to manually raise the count again. This is something we try to avoid, though.

In the very near future I plan to upgrade the script to support regular expression matches as well as simple string matches. It currently isn't very good at raising sensible error messages — you have to read the explanatory comments in the script's source code — and it might be nice if it could ratchet the expected counts downwards automatically instead of demanding that the developer do it. In theory, this ratchet script is not a long walk from a conventional source code linter, and it might be nice if it had a more consistent, flexible interface for adding new heuristics, configuring which source files to inspect or ignore, automatically suggesting fixed code, and so on, and so on...

But on the other hand, I am as conscious as anybody that it would be incredibly easy to divert a huge amount of pointless time and energy into maintaining and improving a "simple" tool of this kind. I think the specifics of our ratchet script (whose content, no, I will not be sharing) are less important here than the generic technique of using basic text scans at linting time to prevent the proliferation of deprecated practices through a codebase.

In general, I dislike having bad practice left over in our codebase. This can be difficult to avoid, but is very misleading for newcomers, and for interlopers — that is, experienced developers from other teams who open pull requests with the best of intentions. "Ah, I see you have diligently followed the example set by your predecessors. Well done, and bad luck. Changes requested." What this technique does is automate what was previously a manual process of me saying "don't do this, we've stopped doing this" in code review. Or forgetting to say it. Or missing the changes entirely, due to the newcomer having the audacity to request their review from someone else.

Another pitfall which I've spotted is that it would be easy to abuse this technique to enforce unnecessarily strict "standards" on a development team who really ought to be allowed some creative freedom. Sometimes it's okay to say "No" to adding a new rule.

This felt like a really basic technique when we first adopted it, but on the other hand it's not a standard practice I've heard discussed elsewhere in the same way that, say, linting, unit testing, code coverage measurement and other techniques are. When I asked people about this online, quite a few people found the idea to be novel and expressed an interest in adopting it. Meanwhile, an equal number of people said that they already do something almost exactly like this, or applied a similar technique to the domains of code coverage or performance.

Anyway, it seems to work okay.
