---
url: https://thorstenball.com/blog/2026/09/19/what-i-believe-about-the-future-of-software-development/
date_fetched: 2026-09-29
---

# What I believe about the future of software development

*This was originally posted on
X and blew up. To plant
my flag, to say that these are things I believed in September 2026, here it is
on the blog, non-ephemeral. Some of these predictions are just observations,
they’ve long been true at companies like Amp. Others will take some time to play
out.*

**Code review will die.** I mean: it’s already dead. But in the future, humans
won’t find a bug or an issue with the code produced by a model, at least not in
a reasonable time. Humans will only review the system and its composition, but
it won’t be in PRs and it won’t be by looking through every line of the code.

**Unit tests might die too.** Why have training wheels if you never fall over?
I’ve had models write 900 lines of Arduino C, compile it without *a single
error*, and send it to the device, where the program ran perfectly. 900 lines
will be nothing in the future.

**The craft of writing code will disappear.** Yes, there are still Italian shoe
makers around. But look at your feet.

**The craft of building software will be more important than ever.** Knowing
how to solve business problems with software, how other software did it and why
and why not, when and how to ship it, how to get feedback on it – that’s the
new game.

**Most bugs won’t be “coding” bugs.** They’ll be “you asked for the wrong thing”
bugs.

**Open source in its current form doesn’t make sense anymore.** “Given enough
eyeballs, all bugs are shallow” is still true but now we have artificial
eyeballs.

**Performance critical contributions by humans will stay what they are: an edge
case.** In 99% of software it does not matter that you could’ve written a
faster algorithm or picked a better data structure. You have no customers, no
users, no one’s executing the code. It does not matter. When it matters, the
models can fix it. Do not compare the top of the top 1% of software
(developers) to 99%.

**The terminal is dead.** Most developer tooling will be washed away by tokens.
Shells, text editors, CLI tools won’t be used by humans anymore. There’s no
need to know command line flags and jq invocations anymore. It’ll be seen as
arcane as knowing how to write everything as a Perl one-liner. (I’m saying this
as a lover of the terminal & dev tools.)

**Tokens are the new computing paradigm.** Everything will be re-made on top of
it. We’ve had deterministic computers for 80 years, so we confuse “how
computers have worked” with “how computers must work.” We’re entering the
post-binary era.

**The triad of PM/Design/Eng will disappear.** It does not make any bit of
sense anymore. Agile, SCRUM, whatever – dead. “Engineers” who act as “meat
proxies” and shove tickets into agents and report back to humans will no longer
be valuable.

**The difference between software engineering at large corporations and small
ones will increase.** Start-ups adopting practices of Google will look even
sillier than before, because they can now build so much faster and with so much
less constraints.

**There’s no proof that “good code” will matter in the future.** The notion of
“good code” itself is mostly based on the idea that it’s easy/cheap/efficient
for humans to work with. But humans won’t modify the majority of code. Think of
how dumb it is to assume that “only 80 columns wide” or “but newlines here and
there” matters for agents – now consider all the other properties you have in
mind for “good code”. Yes.

**Some people will be priced out of producing software.** Just like only some
people could afford a personal computer in the 80s and 90s, for the next few
years, if you can’t get enough tokens, you’re playing second league. You need
to get to the tokens.

**It’s questionable whether cheaper models will be used.** Tokens will be
everywhere and we’ll swim in tokens. And what we consider a smart model today
will be considered very dumb in the future. But a smarter model makes less
mistakes, needs fewer turns. When do you really think “I’m okay with it being
wrong a few times?”

**Models will become so fast that UI will be generated on the fly.** A lot of
UI exists because software can’t understand what you want. Menus, settings
screens, dashboards, filters: much of it is a human-accessible API to a dumb
machine. Smart machines need much less UI.

**It’ll take a while for this to play out.** It’ll take a generation for the
“new software” to replace the “old software”. Just like there are people
happily employed as ASP developers today, there will be people employed to
write code in 10 years. But do you want to have that job?
