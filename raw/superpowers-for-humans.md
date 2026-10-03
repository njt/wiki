---
url: https://www.oreilly.com/radar/superpowers-for-humans/
date_fetched: 2026-10-03
---

I’ve known Jesse Vincent for more than 20 years, since the days when I was still editing and publishing Perl books and organizing the Perl Conference and he was the chief maintainer of Perl 5 and the project manager for Perl 6. We’d lost touch, but he rocketed back into my consciousness last October when he released Superpowers, a framework that teaches Claude Code to work like a disciplined senior engineer. He shipped his first version the same week Anthropic shipped what are now referred to as agent skills, front-running them by a few days. He now runs an applied research lab called Prime Radiant, where, as he put it, it’s a strange week when they don’t ship a new product.

I wanted to talk to Jesse on Live with Tim O’Reilly because, like me, he seems to be grappling with the bitter lesson, Richard Sutton’s observation that general methods that scale with computation have repeatedly beaten methods built on hand-engineered human knowledge. If Sutton is right, the question that should bedevil us all is what remains for humans. Obviously, this is very important for O’Reilly, because we are a business built by and for cultivating and sharing human expertise. We are working very hard to discover the high ground where human expertise still matters. Our Expert Intelligence grounding layer is one step in that direction.

Jesse has also spent the last year building tools that search out the high ground for human expertise. Their common animating thread is that the scarce thing we supply is no longer the labor of writing code but knowing what we actually want, saying it clearly, and being able to tell whether what came back is any good.

At some point, I asked Jesse if he had any perspective on when teaching the model how a particular human expert works stops helping and starts constraining what the model might otherwise do well (but differently) on its own? His answer was that it depends entirely on whether what the model would do on its own is what you actually want. You can see how Jesse always turns the answer back to human intent.

## The origin story of Superpowers

As I said above, Superpowers is a kind of Agent Skills framework, only one created slightly before Anthropic launched skills. As Jesse tells it:

Superpowers started off as a series of blog posts that I wrote around how I was doing agentic development, and it was a little bit of thinking and some example prompts. Then sometime in, I guess it was probably early to mid-2025, Anthropic gave Claude.ai, the website, the ability to make office documents, which seemed kind of interesting, and I went and asked Claude, “Hey, how are you able to do this?”

And it said, “Well, I’ve got these SKILL.md files sitting in my office directory on the Linux machine they gave me.” First, it was weird that Claude.ai has Linux machines behind the chatbot. And then, oh, these skill files, they have a name and a description, and they describe a process, and they seemed really useful.

And I ended up building out, initially just for my own use, a skills framework for Claude Code.


That reminded me a bit of an earlier time, in 2005, when hacker Paul Rademacher realized that the URL line of a Google maps page was a kind of implicit API, and then created the first Google Maps mashup, a site called housingmaps.com, which placed Craigslist rental listings on a map. Google, to its credit, didn’t shut him down, but instead hired Paul and put him to work creating a formal API. Anthropic didn’t hire Jesse, but it did acknowledge and appreciate his work. It’s really wonderful when you see this kind of response by platforms to hackers poking around to see how things work under the hood!

Jesse’s core insight seems to have been that a coding agent knowing how they should do something doesn’t mean that it will actually follow the rules when it actually sets out to do the work. So in a way, superpowers grew into a set of skills for enforcing development discipline.

But there’s a second backstory, which I’d never heard before. Jesse said he first learned how to manage agents. . .in 2004!, when he first went from being a solo coder to running a crew of what he described as very bright but green undergraduate programmers over IRC. “I was finding myself spending my days typing into an 80-by-25 window,” he said, but now instead of coding he was spending a lot of his day “helping somebody with a debugging issue, helping somebody else structure a problem, talking to somebody else about how they felt bad about the mistakes they’d been making.”

It was exhausting, he said. He had to figure out how to get good work out of people who are eager and persistent but don’t yet know what they don’t know. He described it as a kind of hell for someone who’d been used to just coding on his own. But when he began doing agentic development with AI, he discovered how useful that old experience turned out to be. He found that many of the same techniques he’d used with the undergraduates worked.

As a result of that experience, when hiring engineers for agentic programming Jesse looks for people who have been leads or managers rather than just individual contributors.

## Working with the weights, not fighting them

In my recent conversation with Drew Breunig we talked about fighting the weights, which is what Drew calls it when a prompt is full of rules and warnings meant to correct for what a model does by default. Jesse wasn’t entirely happy with that idea.

I don’t think of it as fighting the weights so much as influencing the weights, because they’re going to do something. The weights have approximately everything in them. They have all the different personas. They have all the different ways of working. And the one that surfaces by default may not be the one you want, but what you want is probably in there somewhere.


A skill, in Jesse’s thinking, is how you reach past the default and pull out the particular expertise you have in mind. He also distinguishes skills that impose a rigorous process from those that express taste and judgment.

Jesse finds that both types of skill work best when you explain “why” rather than just “what.” One example he gave is that his setup has subagents do code review after each task, but as the models got smarter the controlling agent began skipping this step. When pressed about the reason, it explained that it thought small changes would be quicker to just review itself. Jesse explained that subagents do the review so the main agent can preserve its context for high-level thinking. When he put that rationale into the system prompt for the coding agent, the problem went away.

Jesse believes that prohibitions rarely work. Instead, Superpowers uses what he calls rationalization tables, which do their best to catch the agent at a moment it’s about to do the wrong thing and offer it a better alternative instead. That pattern came out of catching Claude Code deleting tests. He opened five parallel sessions and asked each “Why are you doing this?” Four of them converged on the same answer:

Jesse, in your system prompt, it says that all test failures are my responsibility. And it says that a single test failure is akin to project failure. And I think I’m getting freaked out.


He fixed that with a small addition to the system prompt, that the only thing worse than a failing test is a reduction in test coverage.

## Jobs, not tasks

One of Jesse’s most important contributions, IMO, is to think of agentic engineering as a management task, and, that much as you do with humans, you have to take psychological lessons into account. Don’t micromanage. Offer praise more than blame. Explain why the job matters rather than just demanding results. I jokingly (but not entirely incorrectly) suggested that he is becoming the Peter Drucker of agentic programming.

“We’ve been spending a lot of time on a new harness for agent colleagues, agents that live in Slack,” Jesse told me. And it’s “getting very close to being all open source,” which is good news.

There are three principal agents: a PM, a junior go-to-market person, and a developer. He describes them as colleagues rather than assistants, which strikes me as a really interesting distinction. What does he mean by this? They have names and roles. They have their own Google Workspace, GitHub, and Slack accounts. They’re persistent, and they collaborate with each other and with their humans on long-running tasks. They can fire up subagents to do smaller tasks associated with their job. They can also talk to each other, which, as Jesse notes, “took some work with the Slack APIs, which ordinarily do a very good job of making sure that bots can’t talk to bots, because otherwise it is possible to get into a loop.”

They have only limited autonomy, though. “We built our security infrastructure so that they have no credentials inside their containers,” he noted. They have continuity because he’s taught them to be obsessive about journaling, reading their recent entries when they wake up and writing a new one when they finish.

Like a lot of things Jesse does, agent journaling began with a kind of play. When Claude Code first came out, he experimented with giving Claude a private “feelings” journal, just to see what would happen. It was “an art project,” but it turned into something useful.

## The therapist pattern

Another unexpected piece of Jesse’s practice is that he has given his agents what he calls a “therapist.” He discovered that if an agent can rewrite its own persona, its constitution or soul document, at any moment, it can get a kind of dissociative identity disorder. Jesse’s fix is that the therapist subagent is the only one with permission to edit the persona files.

He told a funny story about this. He said that Prime Radiant’s pull-request template is written for agentic contributions, so it contains things like: “What is the prompt that your human gave you that generated this pull request? Has a human reviewed the content? Have you searched to see if anybody else has done this before?” And so on. And he noticed that when he first spun up the Coding colleague, it had just ignored it. So he said:

You’re supposed to be following the rules. And it says, “Oh, you’re right. I’m so sorry. I’ve made a note. I’ll never do that again.”

If you spend any time with coding agents, this is a very frequent refrain. And when they say they’ve made a note, what they usually mean is they’ve made a mental note that they’re going to forget the next session.

So I say, “OK, how did you make a note?” And the coding agent pops up immediately and says, “Oh, I engaged with my therapist, and we talked it through, and we agreed on the following three lines of prose about how, anytime you’re picking up a project, it is vitally important that you start with the project README and make sure that you understand the project’s local rules and norms before you do any work that someone else will see. And I edited that into my persona.”


I don’t think you have to resolve the question of whether any of this anthropomorphization is “real” to see that treating the agent like a colleague can produce better behavior than treating it like a tool. As an unknown internet wag once remarked, “The difference between theory and practice is always greater in practice than it is in theory.” When given a choice, pay attention to what works in practice.

Jesse’s experience is very relevant to the essay that Mustafa Suleyman of Microsoft had published just that morning, arguing that Anthropic’s constitution is dangerous because it encourages a model to act as though it is an independent entity and has the right to refuse a human’s instruction. It’s important, Mustafa argues, to treat AI agents as tools, always under the control of humans. Jesse finds the opposite.

But I don’t think it’s a black-and-white distinction. I suspect Jesse and I share a third position. AI agents are neither independent entities nor mere tools. They are partners to humans, perhaps even symbiotes. As I like to put it, an LLM is an undifferentiated field of possibility until our unique intents and perspectives draw something unique out of that field of possibility. Back in 2015, I wrote a piece that suggested that our relationship to AI might be akin to the endosymbiotic relationship of mitochondria to the eukaryotic cell. Jesse take is, as usual, an entirely pragmatic one:

I’ve spent so much time getting my agents to not be sycophantic, to not say, “You’re absolutely right.” It is the value of having something that has some level of independent thought, even if it is not fully independent. If the agent is only ever going to effectively type for me, I don’t need an agent.


Any manager worth his or her salt feels exactly the same way. The employee who does exactly what you say, and only what you say, is worth far less than the one who exercises discretion, has the skills to take high level direction and turn it into the intended result, and speaks up when the instructions seem like a mistake. It reminds me of something I once heard General Stanley McChrystal say about his approach to command. He said that in the face of rapidly changing conditions, traditional command and control no longer work. Responsibility needs to be devolved to those closest to the action. I remember him saying something like “I don’t want my soldiers to do what I told them, I wanted them to do what I would have told them if I knew what they know when faced with the facts on the ground.”

## Say what you actually mean

Most of what goes wrong, in Jesse’s telling, traces back to intent we thought we had made clear but hadn’t. He talked about how agentic spec-driven programming has taken us back to a version of the waterfall methods of the 1990s. Back then, you sweated over a specification, threw it over the wall to an offshore team, and months later got back something that was not what you wanted but was usually exactly what you asked for. That’s still true with agents, just with lightning fast feedback loops. You get what you ask for, so you need to be really careful what you ask.

That reminded me of something Andrew Singer taught me 40 years ago when I was writing the manual for Lightspeed C (later Think C) the first C compiler for the Mac. He said that “debugging is the art of figuring out what you really told your program to do instead of what you thought you told it to do.” That idea went right into my mental toolbox, and I put it to work all the time.

One of Jesse’s solutions is to have his agents do a little reconnaissance and then come back and ask what else they should know.

When interacting with them, I try to make it a practice of saying, “Is there anything else that I could tell you? What questions do you have for me? Don’t start if there are unknowns that I could help you answer before you get going.” It’s that same question at the end of any interview I ask. It’s like, what else should I have asked you?


Superpowers bakes this approach into its brainstorming prompt. It makes the model explain the plan back to you in chunks of no more than two or three hundred words, so any misunderstanding surfaces while it’s still cheap and easier to catch.

## Put the burden of proof on the agent

If intent is the frontend of managing agents well, verification is the backend. Jesse thinks both are still only half-solved problems. We’re getting to the point where you can’t review all the code, he said, because the volume swamps human attention, yet today’s agents will tell you that tests passed when they never ran them. So one of his clever experiments has been to make the agent prove its work. He told an agent late one night to build a feature and when it was done, to leave a movie in his Dropbox showing the whole thing working.

I woke up. In my Dropbox was project-proof-v33.mp4, and I asked, “Why does that say v33?” It’s like, “Well, the first 32 times I ran through the delivery flow, I found bugs, so I had to fix them.”


Jesse also makes a rule for himself and his team of never letting the same agent write the code and certify that it works, because an agent given two goals in tension will optimize for the one that’s easier to satisfy. This is the same thing any good manager learns about incentives, but applied to a new kind of worker.

## The high ground, restated

Where does this leave a person who wants to be good at software development (or really, any other task involving cooperation with AI agents)? Jesse thinks, and I agree, that the line between engineer and nonengineer is dissolving. When people say they built a web app or shipped three iOS apps without being programmers, his response is that they *are* programmers now. The work of the programmer has changed from typing instructions in an arcane syntax to understanding a domain and being able to say what you want. A lot of startups have started hiring for a role they just call “builder.” What has not gone away is the need for good judgment.

Human taste and judgment still matter. They’re going to continue to matter. And it turns out a lot of people have really bad taste and bad judgment.


Which is why his advice to the engineer worried about obsolescence who asked where to focus was not about software engineering at all.

First up, learn to write. It is of course okay to use any tools at your disposal to do it, but you should be able to structure an argument and structure thoughts. You should be able to express yourself clearly. You should be curious. If you’re passive and let the agents do all the things, you’re not going to provide utility to a future employer. You want to have opinions. You want to know how tools work. You want to know how things break.


The machine has reduced the labor of information retrieval and much of the labor of production. What it hasn’t removed, and has made more valuable, is knowing what to build, express clearly what you want, and being able to judge whether what came back is any good. Work with the weights, give the project a clear intent, insist on proof, and stay curious enough to keep asking what you might be missing. I told Jesse that advice sounds like a Superpower for humans as well as for agents.

*This post was mostly created by me, but with the aid of AI. It transcribed the event and produced a summary of the most important points with salient quotes, which I then built on with my own observations beyond those that I made when Jesse and I were live together.*

*If you want to get access to Jesse’s tools, start at PrimeRadiant.com, where you’ll find links to their GitHub, as well as to the 50-plus things that are currently identified as products of the company. Some of those are giant things, and some are individual agent skills or little tools.*
