---
url: https://qntm.org/ratchet
date_fetched: 2026-07-05
backfilled: true
---

So there's a thing we use at work which I call a *ratchet*.

In our codebase, there are "patterns" which we used to use all the time, but we decided to stop using them, but removing all of the existing instances at once is too much work. We want to remove all of these instances eventually, and in the meantime we want to make absolutely sure that they don't proliferate via copy-and-paste. So what we have is a ratchet, a script which runs at source code linting time and counts all of these "pattern" instances across the codebase. If the script counts too many instances, it raises an error, explaining why we don't want more of that "pattern". If it counts too few, it also raises an error, this time congratulating you and prompting you to lower the expected number.

This script is intentionally extremely simple. The expected numbers are hard-coded in the script itself. The "patterns" for which it scans our code are not advanced, abstract Gang of Four-style software design patterns but plain text strings.

At the time of writing, the strings are mostly the names of methods whose usage we frown upon. The methods in question aren't our own. They are public methods of first- and third-party libraries we use. We can't apply deprecation warnings upstream. Nor would we. These are perfectly acceptable and normal methods for people in general to use. It's only within the scope of our own specific codebase that we've decided to try to quit using them.

The script carries out *extremely* basic string matching. There is no source code parsing. So there are some obvious edge cases. What if someone wants to talk about `THE FORBIDDEN METHOD` in a comment, say? What if it shows up in a string literal? Answer:

One important observation is that this technique does nothing to actively encourage the removal of these old "patterns". Those remaining 67 or so calls to `THE FORBIDDEN METHOD` have been kind of lingering. But perhaps that's a different problem.

Sometimes, due to extenuating circumstances, we have had to manually raise the count again. This is something we try to avoid, though.

In the very near future I plan to upgrade the script to support regular expression matches as well as simple string matches. It currently isn't very good at raising sensible error messages — you have to read the explanatory comments in the script's source code — and it might be nice if it could ratchet the expected counts downwards automatically instead of demanding that the developer do it. In theory, this ratchet script is not a long walk from a conventional source code linter, and it might be nice if it had a more consistent, flexible interface for adding new heuristics, configuring which source files to inspect or ignore, automatically suggesting fixed code, and so on, and so on...

But on the other hand, I am as conscious as anybody that it would be incredibly easy to divert a huge amount of pointless time and energy into maintaining and improving a "simple" tool of this kind. I think the specifics of our ratchet script (whose content, no, I will not be sharing) are less important here than the generic technique of using basic text scans at linting time to prevent the proliferation of deprecated practices through a codebase.

In general, I dislike having bad practice left over in our codebase. This can be difficult to avoid, but is very misleading for newcomers, and for interlopers — that is, experienced developers from other teams who open pull requests with the best of intentions. "Ah, I see you have diligently followed the example set by your predecessors. Well done, and bad luck. Changes requested." What this technique does is automate what was previously a manual process of me saying "don't do this, we've stopped doing this" in code review. Or forgetting to say it. Or missing the changes entirely, due to the newcomer having the audacity to request their review from someone else.

Another pitfall which I've spotted is that it would be easy to abuse this technique to enforce unnecessarily strict "standards" on a development team who really ought to be allowed some creative freedom. Sometimes it's okay to say "No" to adding a new rule.

This felt like a really basic technique when we first adopted it, but on the other hand it's not a standard practice I've heard discussed elsewhere in the same way that, say, linting, unit testing, code coverage measurement and other techniques are. When I asked people about this online, quite a few people found the idea to be novel and expressed an interest in adopting it. Meanwhile, an equal number of people said that they already do something almost exactly like this, or applied a similar technique to the domains of code coverage or performance.

Anyway, it seems to work okay.

## Discussion (16)

## 2021-11-21 03:30:13 by Josh:

## 2021-11-21 03:30:19 by Max:

## 2021-11-21 04:31:02 by Emily:

## 2021-11-21 08:52:34 by skztr:

## 2021-11-21 13:20:33 by Tim McCormack:

## 2021-11-22 10:30:51 by Spwack:

## 2021-11-24 09:39:13 by LiterallyMechanical:

## 2021-11-28 04:17:40 by Jake:

## 2021-11-29 02:56:29 by g:

## 2021-12-17 05:29:55 by FeepingCreature:

## 2021-12-17 07:31:42 by Ingvar:

## 2022-01-08 01:19:57 by callmesalticidae:

## 2022-03-15 06:00:28 by Curt J. Sampson:

## 2023-06-09 20:28:53 by J:

## 2026-02-02 16:51:11 by Mike:

## 2026-02-02 17:02:26 by tim:
