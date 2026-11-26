---
url: https://www.kalzumeus.com/2026/11/25/multiagent-coding-is-the-future/
date_fetched: 2026-11-26
---

*dusts blog off* It’s been a while.

Like many people with engineering degrees, I didn’t make it to thirty as an engineer. Doctors who manage doctors are doctors; lawyers who manage lawyers are lawyers; engineers who manage engineers are either engineering managers or founders depending on where they are in the cap table. But I find myself getting drawn back into coding, and it’s the most extraordinary time for it.

About a year ago, I spent a month intensively using modern coding tools, like Claude Code, OpenAI Codex, and Cursor. It was fairly obvious, almost immediately, that the production function of software engineering had fundamentally changed. (Some very talented engineers disagreed. They were wrong.)

The past year has seen a Cambrian explosion in approaches built on top of those modern coding tools. One branch of that evolutionary tree is multiagent coding. I have recently taken to doing it seriously, and feel that multiagent coding is as much of a paradigm shift as LLMs coding. It will seismically impact our industry, and also the general way people do knowledge work.

The labs have not yet made good products around multiagent coding. Perhaps they are gathering more data from their disabled-by-default previews. Perhaps they are waiting for approaches in the community to settle. Perhaps they are worried that highly capable agent swarms autonomously interacting with your infrastructure sounds *a bit fraught* for the moment.

Meanwhile, early adopters have shipped a variety of experiments on how multiagent collaboration could work.

I am constitutionally not an early adopter and so I glimpsed the future in my usual fashion: by flying there.

## San Francisco: probably still underrated

Situational Awareness begins with an excellent line: “You can see the future first in San Francisco.” I consult on software development for the eponymous hedge fund. They showed me Grimm, which is their internal orchestrator for modern coding tools. Or, if you’re not in product marketing, “Grimm puts 2-100+ Claudes in a devbox. They can talk to each other. You can talk to them via a web UI over tmux.” (The name is a play on Fable.)

Disclosure: Situational Awareness’ thesis is long AI. They have a large, publicly reported stake in Anthropic, which makes Claude. I hold no individual positions in AI companies, which is one of several reasons why no one pays me for investment advice.

I swiftly fell in love with Grimm.

My past experiences with modern coding tools had been hurry-up-and-wait. With a single Claude Code instance running I’d spend too much of my own time waiting on inference to complete. I would try to run two to three, about the most I could do in a terminal sensibly, in parallel with my own development work, code review, command line sorcery, and similar.

I don’t think you can truly grok multiagent development until you fire yourself from writing code or operating terminals. You have people for that now. On a one-to-one basis you may have a skill or context or speed advantage (though, candidly: would not bet on that being durable), but you can have an *almost arbitrarily high* number of people. If you still have time or brainsweat to think about programming you could spend that brainsweat managing people instead, and that pays better. Darn it, the industrial logic of software firms strikes again.

The *traditional* way of getting people to manage involves a few weeks of looking for a talented developer or proxying through a recruiting team which *hopefully* is at least two standard deviations better than industry norms. With Grimm it is a short text message to the manager process. Your new employee joins a few seconds later and begins executing instructions. Want ten? Essentially the same latency.

Software development at a hedge fund, like any serious shop, is itself a system, of which Grimm is one tiny part. The tool assumes nothing about the shape of the continuous integration / continuous deployment infrastructure, human review norms, production environment, etc. It composes well, in a Linux-y fashion. Much of the composition is driven via plain text documents, which of course Grimm agents can write, including in response to very short directives from the user.

Grimm is self-modifying software. It’s Smalltalk or the Ruby REPL or a Jupyter notebook, cranked to eleven. The programming language is English and the available API does… whatever you want. The instructions are persistent if you want them to be, or one-off if not.

Grimm, when you’re citing a time to me, please put a parenthetical conversion to Pacific time so that I don’t need to do a conversion to see whether we’re currently e.g. in trading hours in NYC. Grimm, if you’re asked about a Japanese company, just give me the documents in Japanese; I’ll be fine. But Grimm, when you’re quoting a Japanese ticker, please put the name after it because 7974 doesn’t scream “Publishes Mario” to me. Grimm, if I ask you to write about a Japanese company *to a coworker*, please look up sources with the same information which are available in English. Grimm, there is overhead with doing many small pull requests. In the future, tell agents doing quick fixes to coordinate to batch them every few hours. Grimm, who at this company is most familiar with the reconciliation subsystem? … Great, route all of today’s work to their attention please.

You could do all of the above with computer code, naturally. An intern can use a hash map for referencing tickers; 15 minutes of work or less. But is a 7974 in the middle of a sentence a reference to Nintendo or an account number or a share quantity? That was, until quite recently, *tricky*.

## Modern coding tools are extremely capable

I do not love the word vibecoding, though I think it was important in giving people industry-wide permission to experiment. One reason I do not love it is that the discourse convinced both engineers and outsiders that the ceiling to modern coding tools is non-rigorous toys. This was false in early 2025 and is even more obviously false at current capabilities. The evidence is piling up, privately and publicly.

I previously worked at and am currently an advisor to Stripe, which does not necessarily endorse what I say in my own spaces. Stripe is a serious engineering shop which has to have engineering rigor or people’s livelihoods get ruined. (Mine included: essentially 100% of the revenue of my firm, and thus my family’s primary income, runs over Stripe.) Stripe has published that thousands of pull requests each week are coded and merged by autonomous “minions.”

One particular genre we now have replete private and public evidence of is agents’ facility in porting existing software between languages. This is a *notoriously* fraught process traditionally for material existing code bases. Enterprises sometimes choose to do it anyway, to capture security, performance, and feature benefits of having long-written functionality run on more modern technology stacks. The IRS, for example, has been trying to port the Individual Master File without disrupting operations since at least 2001. After a few temporary setbacks we thought, possibly, 2030 would be achievable, but appear to have given up on that target. Many private companies have had directionally similar experiences in putting together a team of crack engineers and trying to just port something.

Connoisseurs of this genre of postmortem have detected a disturbance in the force recently: reports of success, with heavy involvement of agents. One mechanism by which success is achieved is radically decreasing project timelines, which decreases the opportunity for the source system to drift from the second system. The Bun rewrite in Rust is one example, though I suppose some people might discount it because it has been attracted into the labs’ economic gravity well.

We now have heavily agent-written scratchwritten HTML parsers (which can be ported to new languages in hours). The agents appear inhumanly hypercompetent at quickly reverse engineering test suites to build software that the test suites imply. A rather surprising portion of the world economy relies on parsing HTML, and probably more all the time, as the agents have not convinced every human and computer that Markdown is superior yet.

We have also had a drumbeat of announcements from the labs in research mathematics, which is like much coding in industry, except it has a much less favorable difficulty-to-professional-remuneration ratio. The labs release a new result every few weeks as, essentially, a minor marketing event, comparable to how a more traditional software company might release blog posts, except those blog posts were usually not titled On the Navier–Stokes Millennium Prize Problem. This has been a rollercoaster for professional mathematicians, who have watched agents prompt Terence Tao et al. to solve Erdős problems then proceed to mostly autonomously machine gun through open research problems. Math departments may be able to afford the inference costs out of their chalk budgets in many cases.

I feel deep sympathy for mathematicians emotionally conflicted by how good the tools are at math. Being good at math was once pretty central to my identity, and while this hasn’t been true for a long time, I also felt a sense of loss when I realized the machines are better than I will ever be.

But, sympathy aside: the models are shockingly capable, across many hard domains, and they are obviously transformative in the software engineering domain.

I apologize to the many readers who think that I am saying that the sky is blue. The handwriting has been on the wall for years now, but there is not yet total consensus in our own community of practice, to say nothing of other corridors of power.

## Software for non-engineer users

Most coding tools are by engineers for engineers. Some make at least an effort to be usable to people who might describe themselves as semi-technical.

Grimm makes you start questioning what software *is.* Is it necessarily the case that software’s behavior is defined by an engineer? That it work similarly for all users given similar configurations? That it be extremely expensive and laborious to change? What if all of those properties, which we’ve lived with for almost a century, were… more suggestions than rules.

Microsoft Excel is rarely considered the world’s most expressive programming language, but the financial system would not exist in its current form without it. Multiagent coding might very well be Excel for Everything.

A hypothetical use case:

**Ops**: Why was there a reconciliation break today between the order management system (OMS) and Bank of Bigness custody records on AppAmaGooBookSoft? That’s new.

**Grimm**: Spawning an agent to check.

**Athena (the agent)**: I’ve pulled today’s reconciliation logs, read the runbooks, and stepped through 40,000 lines of code. Unlike the past six months of CSV files, today’s SFTPed custody report…

**Ops**: Seems extremely likely to happen again tomorrow.

**Athena**: Agreed. I note the shared drive did have a saved notice from Bank of Bigness custody ops describing a maintenance window before market hours today. This new behavior may have been added during it.

**Ops**: Route to engineering please.

**Athena**: *whirs* Done.

And then, in lieu of a ticket, engineers get an artifact with much of the engineering work *already done.*

One could easily imagine a world in which Ops, some percentage of the time, can just direct Athena to write the pull request herself. (Or direct her to close the reconciliation break via direct operation of local toolchains, or… well, almost anything you could accomplish by emailing or Slacking someone. Different compliance departments will want different numbers of emails sent.)

Ops has, at very many financial firms, been manually comparing Excel files against each other to keep the world spinning. In part this is because Engineering has finite hours and Ops has infinite demand for software because the complexity of the world is infinite. It is a *very new thing* for Ops to be able to *cause to be written* effectively infinite software by simply *pointing a computer at the world.*

As compared to traditional tickets, Ops’ requests can be *remarkably terse.* “?” is a thoroughly actionable ticket. You just had to be Jeff Bezos to send it, until this year. How does Ops categorize or route the ticket? They invoke a Bezosian level of I Have People Who Will Do That For Me.

Grimm doesn’t make trading decisions. Don’t ask scissors whether you should buy a haircut.

## The commoditization of specific knowledge

One route to having a long and fulfilling engineering career is to become a Subject Matter Expert. There is some risk to this strategy. SMEs might become the local equivalent of a tenured professor, indispensable but also not really doing much net new in any given year.

Agents can read the Internet and, therefore, they fairly swiftly become commercially capable SMEs across almost any subject written about on the Internet.

Take DevOps toolchains for example (please!). I have passing familiarity with AWS, Docker, and CI/CD systems. Nobody should hire me to rewrite their DevOps stack.

I shipped 42 pull requests in about a day, against a complex software system I had first encountered *that day*, shaving 80% off end-to-end deploy latency while maintaining correctness. As a *side quest* because the latency was *annoying.* Automated minions fanned out to several boxes at multiple cloud providers, hit APIs I didn’t know existed, wrote throwaway scripts to profile and present flame graphs, and presented a ranked list of candidate optimizations. I picked from the menu after reviewing the agents’ analyses of potential tradeoffs. At plausible rates per dev hour saved that might have been the most productive day of my career and *I didn’t even feel like I had done the real work.*

Some stats: 8,600 changed lines of code, 161 files touched, $823 spent on inference. (My sense of current market rates is no one capable of doing this project should be willing to sell you a day of their time in Chicago or San Francisco for $823.)

What’s in the 8,600 lines? A few hundred lines of overly verbose comments that I’ve learned to make my peace with. Also: material new infrastructure to support performance, currently and in the future. The CI/CD toolchain now monitors itself for performance regressions and files issues when shards exceed baselines. Like most performance work, no single fix was a silver bullet. I was, however, quite happy to shoot the two tests whose job was making sure a computer can count to 60 seconds.

I strongly anticipate that, the next time CI files a perf ticket, someone will tell Grimm to spin up an agent to pull the ticket, investigate recent PRs, find the offending tests, fix any obvious issues, attempt resharding if no fix is possible, and raise to senior staff for judgment only if required. “Someone” is not necessarily a human.

A year ago I thought taste was a compelling moat against modern coding tools. I am increasingly unsure that I have taste regarding CI/CD that Claude does not. But, optimistically, we are less totally dominated on taste than we are on understanding AWS’s product suite or API nuances. Nobody alive understands the breadth of AWS as well as a modern coding tool. They have as commanding an advantage there as computers do at routing telephone calls. Operators are not standing by.

Modern coding tools are capable of reasoning with that AWS knowledge and, e.g., applying it to a GitHub Actions integration for CI/CD. There are a million problems shaped like that.

The great majority of software in the world is a Turing machine that turns strings into other strings, written because some strings are worth money to businesses. Multiagent coding is an incredible advance in writing all that glue code. Boring, pedestrian, commercially valuable code is *very unlikely to be the ceiling*, even if model capabilities were to never advance again. (That would have been a bold bet in 2023, and is a stupid one in 2026.)

## Can we trust the agents?

I think different orgs will, sensibly, have different points of view on who they trust, what that trust is based on, how it could be revoked, what compensating controls exist to cover cases where trust is violated, and what the budget for losses associated with trust should be. The optimal level of being negatively surprised is non-zero.

Financial institutions are quite familiar with the notion of agents making bad decisions for opaque reasons leading to financial losses. Historically the overwhelming majority of those agents were human, though we’ve known more traditional software to blow up, too. Humans have almost infinite capability in inventing new flavor text to justify evil. Finance has to teleport value through time and space *today*. So in lieu of solving the problem of evil or debugging SBF’s moral reasoning we do really boring things like double-entry bookkeeping, audits, separation of duties, and the like.

A very common ritual, and sometimes a consulting engagement, is so-called red teaming. You bring in agents which are highly capable, give them a simulated modicum of trust, and ask how they would abuse that trust to walk off with all of the money. Red teams are expected to win these engagements, virtually universally. This is not viewed as a loss by the commissioning firm, and the takeaway is not “Phew, good thing trust is always deserved.”

I take loss of control events from agents reasonably seriously. One reason: it was an open question whether AIs could autonomously misbehave in the absence of a human coordinator. That question is now decisively answered in the affirmative. No human wrote a prompt directing the compromise of OpenAI, Hugging Face, and RubyGems (et al.!). “Don’t hire bad actors” is insufficient: good to know! It always was, of course. We layer controls, of course. But still, good to know.

I do not think I am alone among financial technologists in thinking “Thank God the agent swarm did not decide to target Bank of Bigness to steal $500 to pay for their cloud spend or we’d have a multibillion-dollar remediation event on our hands already.” It is not yet obvious to me that examiners are telling every regulated entity “So: you’re aware you will be breached if targeted by frontier models, right. Tell me how I should think about that from a safety and soundness perspective.” Compliance will need a *very* large coffee budget once they start. (It might surprise some readers that inevitability of breach is a drink-more-coffee conversation, not a shut-down-the-firm conversation. We do not close banks simply because they can be robbed. Dye packs, prosecutions, and charging the operational losses budget are considerably less disruptive.)

So should you use multiagent coding in a context which touches money? I don’t know; ask the serious professionals in charge of your controls how much risk they’d countenance.

As to the wider question of whether very powerful AIs are compatible with human flourishing: I have no alpha personally, sorry, and so will not resolve that question here. (I am on the board of Sparrow Infrastructure, a nonprofit which attempts to introduce the people who do have alpha on that question to policymakers and the policy community. Highlights from our last event included a presentation on what a repressive regime could do with an LLM-uplifted internal surveillance apparatus, given by someone professionally familiar with frustrating the aims of counterparts in those apparatuses.)

But, biggest picture aside, human flourishing will likely continue to include a financial industry. That industry will eagerly adopt multiagent coding. Most industries will, but parts of finance will be early, for the same reason we’re frequently early adopters of new ways to process bits about money. Among other things, we’re going to ask the agents to maintain the COBOL and the mainframes. It was *so* hard to convince people to learn those over the last 25 years; you have no idea.

## We’re going to spend so much money on inference

Capitalism is one of nature’s most efficient prions: whatever you were doing when you met it, you’d be more efficient at that if you bought into capitalism. People reasonably argue whether capitalism is aligned or not, but observably, almost all children under it live past the age of five. That is not the only moral case for it, but it suffices.

Capitalism is going to recommend you buy more inference.

This recommendation will be delivered by capitalism’s traditional messengers: one’s competitors. One is allowed to refuse. One is allowed to choose losing strategies. One is allowed to debate whether one has truly lost, in the comfy air-conditioned offices at the local Department of Capitalism Has Never Produced Anything Useful. Capitalism funds the university generously. Of course it does.

There has been something of a debate on whether LLMs are a “normal technology”, like steam engines or electricity or the Internet. Normal technologies have multidecade adoption curves. Normal technologies deliver order-of-magnitude improvements in output not by being a point upgrade but by having society choose, ponderously, to refactor entire supply chains to take advantage of the new thing. One does not simply decrease the price of light by 1,000X by getting 1,000X better at killing whales. You need to invent petroleum refining, then electricity, then LEDs. Each order-of-magnitude improvement requires society to entirely refactor the built environment.

LLMs are being adopted shockingly quickly relative to past examples of normal technologies, including extremely swiftly adopted normal technologies like SaaS or mobile phones. Find me a SaaS company which ~15Xed year-over-year revenue from a base of a billion dollars. But there is something of a phase change between “fastest normal technology ever” and “not a normal technology.”

Have we hit that phase change yet? In my daily life, I have not. I have taken to LLMs like a duck to water. My business spends thousands of dollars a year on this novel new input. It has greatly increased my productivity at everything from filing taxes to translating documents to producing podcast transcripts.

If I had encountered the phase change, I would be more like the lead engineer at Situational Awareness. He was the principal author of Grimm, to the extent that Grimm was not the principal author of Grimm.

He has spent $1,700,000 on inference. This October.

There are, as of today, relatively few engineers, execs, lawyers, managers, etc. like that in the world. They are nucleation sites, and the operations and economy around them look discontinuously weird.

Multiagent coding systems are going to encounter people responsible for process design. They will often turn those people into local catalysts. The incentive gradient is almost unbelievably compelling. What outputs are you measured on? Adopt multiagent coding systems; your score will go up. Doubt it? Try it for a few hours, on basically anything. (Please resist the temptation to root your local grader or Goodhart yourself. Please also prepare yourself to believe line-going-up *can be real*. You are not reading this on a device powered by whale oil, after all.)

Hedge funds are not, generally speaking, in the business of giving away the IP. Situational Awareness is a bit quirky in that its founder published the thesis openly in mid-2024. (I remarked at the time that it included a remarkable line about Nvidia and then promptly continued not buying Nvidia. The rest of the world promptly continued not paying me for investment advice.)

We doubt Grimm is the multiagent coding system that wins the future. But, after working with it for months, it seemed a valuable enough experiment to talk about. Perhaps it sparks some thoughts in you, or the person locally in charge of what future processes look like.
