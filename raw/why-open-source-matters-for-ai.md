---
url: https://oreillyradar.substack.com/p/why-open-source-matters-for-ai
date_fetched: 2026-08-14
---

# Why Open Source Matters for AI

#### By Tim O’Reilly

In 1995, the question in the media was whether Netscape or Microsoft would control the web. The answer, it turned out, was neither.

Both Netscape and Microsoft aimed to dominate the web server and browser market, reasoning that whoever controlled both ends of the connection would have an internet “platform” to rival the deathgrip that Microsoft had enjoyed on the personal computer. The two companies raced to build every feature they could think of directly into the product, on the theory that whoever built the most integrated and full featured web server would win.

The open source Apache web server took the opposite bet. It stayed a web server with a clean extension layer, so anyone could bolt something new onto it without asking permission or waiting for the next release cycle. Within a few years, Apache was far and away the most popular web server, and Netscape’s server and Microsoft’s Internet Information Server (IIS) were history. People started talking about the LAMP stack: Linux, Apache, MySQL, and (Perl | Python | PHP) as a legitimate platform. Modularity, not features, was the moat. The fact that major elements of that stack survive while others have been swapped out or extended is a testament to the power of composability and distributed innovation.

I called that pattern the architecture of participation when I wrote about it in 2004. I was trying to explain an inconvenient fact that the licensing debates of that era ignored. I had started working with Unix in the System III days, and saw how it had succeeded as a collaborative project even though AT&T offered Unix under a proprietary license. A few years later, I observed that nominally open source projects like OpenOffice with monolithic architectures never built much of a community. I realized that open source wasn’t just about licenses, but about architecture. A small kernel with standard interfaces that lets people extend your work without asking for permission is an important part of the secret sauce.

Swap out Netscape and Microsoft for OpenAI and Anthropic in this story, and perhaps you can see the echoes.

A model’s personality, its defaults, and its history used to live where you could, with a little effort, see them and edit them. Increasingly, they don’t. As Drew Breunig pointed out to me the other day, each new version of the frontier models moves a little more of the product’s behavior out of an editable layer and into the weights themselves, where nobody outside the lab can see it, let alone change it. The model stops being a component you build with and can adjust to your liking and starts being an appliance you rent. Post-training is important but Drew points out that it is also “trading diversity for reliability.” That’s a good trade for many people, but it is the same kind of trade that gives us highly processed foods when we know that “real food” is better.

**Open weights are just table stakes**

The public debate about open source AI seems devoted far too much to model weights, their national security implications, and whether a lab releases weights and under what license. But that covers only a fraction of what actually makes open source matter. Apache was never competing with Netscape and Microsoft (and Linux was never competing with Windows) over whose source was more available. They were competing over something more important. I remember talking with Bob Young, the founder of Red Hat, about his business model, and he said “What we really sell to our customers is control.” Open source meant that the platform your application depended on was no longer a sealed box you licensed from one company but a layer you could extend and build a business on top of without asking anyone’s permission. It sparked an explosion of innovation. It enabled companies like Google and Amazon to grow up free from Microsoft’s dominant paradigm.

Every wave of computing, from mainframes to PCs to the internet, has run through the same cycle: distributed innovation at the start, with the eventual winner gradually closing down its offerings to build a moat. What keeps a market open isn’t the license on any single component. It’s how easy it is to swap out one component for another when a better one appears.

The protocols connecting the pieces are an important part of that picture. Unix utilities expected stdin and stdout, and the shell acted as a kind of harness to connect them, so it was easy to build a new tool that worked seamlessly with existing ones. A testament to the power of that approach is just how much the shell and Unix utilities are the lingua franca of agentic tooling today, more than 50 years after they were invented! TCP/IP, HTTP, and other internet protocols played a similar role in keeping the internet open and composable.

Fortunately, so far, we are seeing some wins for composable, protocol-centric architectures in AI. Anthropic’s Model Context Protocol was a disruptive move in that direction, an open standard for letting any application reach any tool or data source without a custom integration for each pairing. Along with other open protocols, MCP also now has a home outside of Anthropic at the Agentic AI Foundation (a subproject of the Linux Foundation), which is at least a partial guarantee of its independence.

Isobel Moure, Ilan Strauss, and I made the case earlier this year in Protocols and Power that as models commoditize, competition moves up the stack to context. Opening the means of accessing that context opens the market, regardless of whether open or closed weights sit underneath it. That’s an unbundling, model from harness from context, done the way Apache unbundled web server from web application.

Agentic skills may also be a critical element of the open source AI future, though as the history of the LAMP stack shows, they may fall by the wayside in the same way that Perl and PHP did. And that’s just fine. Composability means that it’s easy to switch to something better when it comes along, or when more people agree on it.

There’s also a lot of great work going on in portable memory from players like Letta, Nous Research, and others. Open source agentic harnesses like Goose and Pi are also a big part of giving power back to the people. Pi in particular is optimized to be modifiable. There’s a fun story told about Mario Zechner’s decision to give Pi a “/quit” command rather than an “/exit” command like Claude or Codex. Countless issues and PRs have been submitted to Pi’s repo, asking for or implementing “/exit”, but Zechner is stubborn. His retort is that you should just ask Pi to add it to your install.

But the projects I listed above are just the tip of the iceberg when it comes to the scale and scope of open source AI. Current AI’s Open Source Gap Map covers more than 24,600 open source AI projects!!, with 421 of them scored in depth across openness, capability, and adoption. The map organizes the stack into three layers: 1) models and associated elements including data sets, fine tuning tools, inference frameworks like VLLM, and evals; 2) the product and UX layer, including harnesses and personal agents; and 3) the infrastructure underneath, including core ML frameworks like PyTorch, deployment tools like Ollama, and edge hardware.

Current AI itself is a public-private partnership that came out of the AI Action Summit in Paris last year. This summer they announced AI Potluck, which they describe as “a public project to build a vertically integrated AI product assembled entirely from open source components… a viable alternative to proprietary AI that isn’t owned by any one company or country.” It is backed so far by roughly $400 million of a five-year, $2.5 billion commitment from the French government, tech companies including DeepMind and Salesforce, and major philanthropies including Omidyar’s AI Collaborative, the Macarthur Foundation, and the Ford Foundation.

The fact that this organization exists, along with others like the Agentic AI Foundation, is a testament to the rising tide of interest in open source AI. The coalition of interested parties also says a lot about the underlying motivations that are driving that interest: AI sovereignty, corporate independence from the overweening ambition of the major labs, and an interest in technology for the public good.

**Keeping it weird**

There’s another element, which Drew Breunig put his finger on in our conversation the other day. The problem with having one or two big closed models dominating AI, and having those models increasingly locking their desired personality, business goals, and guardrails into the weights themselves, is that they will reduce the diversity that is at the heart of innovation.

It’s our job, Drew said, to make it weird, to push a model deliberately out of distribution rather than to settle for whatever the labs have made the default outcome. He described how his team chose not to build in React for a recent project for exactly that reason: every model already knows React too well, so building in it means shipping the average of what everyone else was doing instead of something genuinely their own. He has started using GLM and Kimi not to save money but because they are more malleable and take direction better inside a custom harness. And he wants the open-weight ecosystem to survive precisely so that models stay infrastructure rather than becoming appliances.

That’s what an architecture of participation is actually for. We need real separation between the model, the harness, and the application, so that someone who wants to build something weird can still do it without a lab’s roadmap and guardrails deciding whether they’re allowed to.

“Weird” may make it sound like something that not all developers might want. But we’re really talking about something intensely practical. In his short essay on trading reliability for diversity, linked above, Drew Breunig put it this way:

Labs have to ship a product that delivers “good enough” results when a layperson gives a model a lazy prompt. Without direction, the model must return something decent. (If it’s a website it’ll use the Inter font, cards with a single colored border, gradients, implemented with ReAct and Tailwind). Anthropic named this default output “distribution convergent.” At CAIS, @trq212 put it well, roughly, “If it’s not in your prompt, you’re getting what’s in-distribution” …. Less diverse models make for more reliable coding agents, but they encourage a monoculture of output.


Addy Osmani, my co-chair of the O’Reilly AI Codecon, took this point beyond model diversity after reading a draft of this piece: “Almost nobody I work with is tinkering with weights, but they’re rewriting the harness and what sits around it pretty constantly—skills, subagents, hooks, context files etc etc. That’s where participation is currently happening.” Addy went on to note that forking a skill instead of adopting the default, memory and constitution files that travel with the agent instead of living in a vendor account, or picking the unfashionable framework on purpose are all areas where ease of modifiability matters to everyone.

I want to end by returning to the Apache story. I believe that the big labs are making the same strategic mistake that Netscape and Microsoft made in the mid 90s. Yes, make the models more reliable for ordinary users. But don’t shut down the options for developers who don’t work for you to push the state of the art forward. As Bill Joy put it decades ago, “No matter who you are, most of the smartest people work for someone else.” No one should have a monopoly on innovation, and no one should be building a moat to hold it back.

*And be sure to join us at *AI Codecon: Building with Open Source AI* on August 31, a free half-day virtual conference. You’ll hear from leading developers and technical experts working with open-weight models, self-hosted infrastructure, and real-world AI workflows, and learn how building in the open gives teams more control over costs, data privacy, and what they ship. Register today to save your spot.*

The healthcare translation I would add is that composability needs authority metadata, not only connectivity.

For an OR-adjacent agent, the chart, imaging, preference card, huddle, and room state should not enter the harness as interchangeable context. Each source needs a visible owner, freshness signal, allowed action, and stop rule before the system turns retrieved context into readiness or closure.

Open protocols matter most when they let the workflow inspect what a component was allowed to see and what it was not allowed to change.
