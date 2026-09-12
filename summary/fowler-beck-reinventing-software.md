---
url: https://gist.github.com/6576b007d7c3f21554435b2b2923492e
source_url: https://www.youtube.com/watch?v=CZs8J1ZD0CE
title: "Martin Fowler & Kent Beck: Frameworks for reinventing software, again and again"
channel: The Pragmatic Engineer
author: Martin Fowler, Kent Beck
date_fetched: 2026-07-04
date_published: 2026-05
transcript_date: 2026-05-18
duration: "32m 33s"
tags:
  - ai
  - software-engineering
  - craft
  - agile
  - pair-programming
  - agents
topics:
  - ideas-and-culture
  - software-engineering-craft
---

# Summary: Martin Fowler & Kent Beck on The Pragmatic Engineer

Martin Fowler and Kent Beck — two of the 17 authors of the Agile Manifesto — sit down at the Future of Software Engineering conference to reflect on 25 years of agile, the AI revolution, and what happens when nobody has the answers anymore. The conversation covers skepticism as a discipline, AI as an amplifier not a replacement, the Agile Industrial Complex as a warning for AI snake oil, organizational incentives that punish "faster, cheaper, better," and why the re-soloing of programming is a dangerous illusion.

## Key points

- **AI is a change of unprecedented magnitude and speed.** Nothing in Fowler's career—not objects, the internet, or agile—compares to the sheer scale and velocity of AI's impact. The industry has no choice but to pay attention.
- **Nobody has the answers right now, and that's the point.** For 25 years, experienced developers could press play on known solutions. Today, the answers change week to week. The critical skill is not knowing, but figuring out how to know—running the smallest possible experiment to validate a claim to your own satisfaction.
- **Skepticism must be total, including skepticism of your own skepticism.** Fowler's rule: be absolutely skeptical of every new technology, but stay curious enough to probe for real signals. His early dismissive reaction to AI autocomplete would have been a mistake if he hadn't kept listening to trusted, balanced voices like Simon Willison.
- **AI is an amplifier, not a replacement.** Beck frames it like the circular saw for carpenters: it doesn't end the craft, it removes drudgery. Juniors who learn fast will learn faster; experienced developers who work effectively will work more effectively. The "middle"—people who entered programming just for money—faces the biggest risk of displacement, echoing the dot-com crash but on a larger scale.
- **The Agile Industrial Complex will repeat itself with AI.** The core ideas of agile were solid, but a massive snake-oil industry grew around them. The same is already happening with AI. Distinguishing real value from hype requires constant probing and wariness.
- **Incentives inside companies are often misaligned with "faster, cheaper, better."** Beck notes that people inside organizations will punish you for delivering improvements that don't align with their personal incentives. AI promising the same things will hit the same wall.
- **The "re-soloing" of programming is a dangerous illusion.** Beck sees a trend of replacing team collaboration with one programmer managing multiple agents. That's not the same as the messy, social, high-bandwidth interaction of an XP team, which produced good results precisely because it was uncomfortable. Two humans pairing with genies can be powerful—the slowness of models even creates space for valuable conversation about design.
- **Developer experience and agent experience overlap.** Fowler quotes an insight from the Future of Software Engineering conference: the Venn diagram of DX and agent experience is a circle. Modular code, good tests, and precise domain language help both humans and AI agents. This suggests existing craft practices retain leverage.
- **Let go of the OCD satisfaction of perfecting a single function.** Beck says, with sadness, that the deep craft pleasure of getting one function just right no longer has leverage. The shift is toward enjoying overall understanding of the domain and its connection to the program.
- **Large enterprises are in confusion and panic, and security disasters are looming.** Fowler reports that big companies with million-line codebases are struggling to apply AI safely. He's alarmed by groups wanting to give LLMs full control over email, predicting serious security incidents this year due to inattention.

## Pithy and provocative quotes

- **Kent Beck on TDD feedback:** "Thank you so much for Test Driven Development. I also get this Test Driven development ruined my life, my dog left me, my house burned down, and it's all your fault."
- **Kent Beck on the current state of knowledge:** "People want the answer and the answer is changing. So you can't possibly, in this environment have the answer now. That's the bad news. The good news is nobody else has the answer either. So you're just as smart as everybody else because you're just as ignorant as everybody else."
- **Kent Beck on organizational incentives:** "It turns out that people don't want faster, cheaper, better. … Inside a company, the incentives are so misaligned with actually achieving that. … People will punish you for that if that doesn't align with their incentives inside of organizations."
- **Kent Beck on junior developers:** "AI is an amplifier and if you're young and learning quickly, AI is going to amplify that or can amplify that. So I personally think this is the golden age of the junior programmer."
- **Kent Beck on losing the craft high:** "I need to let go of that because that satisfaction of getting this one function just right just doesn't make a difference anymore. … I can't do that anymore. But I can still develop an overall understanding of what I'm doing."
- **Martin Fowler on skepticism:** "My skepticism has to be absolute and total, which means I have to be skeptical about my skepticism."
- **Martin Fowler on security risks:** "I'm very much concerned we're going to have some really bad security incidents over this year because people are just not paying attention."
- **Martin Fowler quoting an insight on DX and agents:** "The Venn diagram of developer experience and agent experience is a circle."
- **Kent Beck on the value of small experiments:** "What's the smallest experiment I can run to verify to my own satisfaction … whether this claim is true? That's the skill that is suddenly in the last year become a thousand times more valuable."
- **Kent Beck on the illusion of managing agents:** "Instead of having 50 people on my team, I have five people on my team. They don't have to talk to each other, and they can each have 10 agents. And that's the same. It's not the same."

## Tools, practices, and methodologies

- **Test-Driven Development (TDD):** Writing tests before production code to specify and verify behavior. Beck and Fowler note that TDD is now critical for verifying AI-generated code—you need tests to ensure the "genie" does the right thing, and the discipline of TDD over 20 years prepared the ground for this.
- **Smallest possible experiment:** Running the least-effort test to validate a claim about a tool or technology to your own satisfaction. Beck says this skill has become a thousand times more valuable in the last year because answers change constantly; it's how you navigate uncertainty without waiting for someone else's answer.
- **Skepticism with curiosity:** Maintaining absolute skepticism toward any new technology while simultaneously being skeptical of that skepticism—probing for real signals even when initial impressions are negative. Fowler used this to overcome his early dismissal of AI autocomplete by reading balanced sources like Simon Willison's blog.
- **Pair programming with AI (two humans + n genies):** Two developers working together while interacting with one or more AI agents. Beck reports positive experiences: the slowness of current models creates gaps for human discussion about naming, conditionals, and next steps, preserving the social benefits of pairing while leveraging AI.
- **Precise domain language for agents:** Developing a rigorous, shared language to communicate about the domain with AI agents. Fowler cites colleague Unmish Joshi's practice: building a precise vocabulary makes interactions with the genie more efficient, echoing domain-driven design and model-building techniques.
- **Modular code and good tests as agent enablers:** Structuring code into well-separated modules and maintaining a strong test suite. Fowler reports feedback that these practices make AI agents more effective, reinforcing that what's good for humans is good for agents.
- **Extreme Programming (XP) social practices:** Creating a safe, high-interaction social environment where programmers talk to each other hours a day. Beck warns against abandoning this for solo agent management; the messy, social, complicated process produced good results, and replacing it with isolated programmers and agents is "not the same."
- **Refactoring (implied):** Improving internal code structure without changing behavior. Not discussed directly as an AI practice, but Fowler's emphasis on modular code and tests aligns with refactoring as a foundation for agent-friendly codebases.
- **Mob programming with genies (speculative):** The whole team working together on one task, potentially combined with AI agents. Fowler wonders if this could be effective but offers no reports or conclusions.

## Unanswered questions and omissions

- **How do you actually integrate AI agents into pair or mob programming?** The speakers mention the idea positively but provide no concrete patterns, workflows, or pitfalls.
- **What should the "middle" of programmers do to avoid displacement?** Beck raises the concern that the large cohort who entered programming for money may be flushed out, but offers no guidance on how they can adapt or transition.
- **How do you mitigate the security risks of giving LLMs control over sensitive systems?** Fowler flags the danger of AI reading and replying to email as "mind boggling" and predicts serious incidents, but no defensive strategies or design principles are discussed.
- **How do you distinguish real AI value from snake oil in practice?** Both acknowledge the problem and the need for probing, but no heuristics, questions, or evaluation frameworks are offered.
- **What does "code" become when prompting and agent interaction replace traditional writing?** Fowler says the nature of code may radically change, but doesn't speculate on what form it might take or what skills will replace coding.
- **How should engineering leaders manage AI adoption?** Advice is aimed at individual engineers; the leadership perspective—budgeting, team structure, risk management, cultural change—is absent.
- **How should CS education and training adapt?** Beck calls this a golden age for juniors, but the implications for curricula, mentoring, and skill progression are unexplored.
- **What are the ethical, bias, and intellectual property implications of AI-generated code?** The conversation stays entirely within the bounds of developer effectiveness and organizational dynamics, ignoring these broader concerns.
- **How do you preserve collaboration and social safety in an agent-heavy world?** Beck criticizes the "re-soloing" trend but doesn't offer a counter-strategy beyond mentioning pair programming with AI; the systemic forces pushing isolation are not addressed.
- **What happens to open-source and the concentration of power in AI providers?** Not mentioned, despite the potential for AI to reshape how software is built and who controls the tools.
- **How do you measure productivity or effectiveness with AI?** The talk assumes AI will make people faster and better, but doesn't question how to measure that or whether current metrics break.

## Full transcript

Available at: https://gist.github.com/6576b007d7c3f21554435b2b2923492e (transcript.md)
Original video: https://www.youtube.com/watch?v=CZs8J1ZD0CE
