---
url: https://docs.eventsourcingdb.io/blog/2026/07/02/use-model-with-domain-an-interview-on-domain-storytelling/
date_fetched: 2026-07-05
backfilled: true
---

# Use Model With Domain: An Interview on Domain Storytelling¶

**Golo:** Henning, Stefan, you are both working at **WPS – Workplace Solutions** and you are the creators of Domain Storytelling, a collaborative modeling technique that has gained a lot of attention in the Domain-Driven Design community. Before we dive into the method itself, can you tell us a bit about your backgrounds and how Domain Storytelling came to life?

**Stefan:** The predecessor of Domain Storytelling had been developed at the University of Hamburg and WPS, a university spin-off. When I joined WPS in 2005, my colleagues had been using the method in requirements workshops with clients. It went without saying that the diagrams were the backbone of our requirements and domain models. Since this was my first job as a professional software developer, it took me some time to realize that this was something special. It seemed like no one else on the planet worked like we did – until DDD started to gain adoption around 2015.

**Henning:** And that's when Domain Storytelling came to life. We simplified the academic method, gave it a catchy name and started talking about it at meetups and smaller conferences. 2018 was our break-out year with talks and workshops at all DDD conferences and the release of our open-source modeling tool **Egon.io**.

## The Origins of Domain Storytelling¶

**Golo:** Domain Storytelling has a very distinctive visual language – pictographic icons, arrows, and numbered sentences that read almost like a storyboard. It reminds me of the classic LucasArts point-and-click adventures like *Monkey Island* or *Indiana Jones and the Fate of Atlantis*, where every interaction was built around actors, objects, and activities. Was there any such inspiration behind the visual style, or how did you arrive at this particular notation?

**Stefan:** The pictographic language was already part of Domain Storytelling's academic predecessor. It was inspired by **Rich Pictures**.

**Henning:** We strengthened the "storytelling" aspect: To us, the notation is not just icons and arrows – it consists of *sentences* with a *subject*, *verb*, and *objects*. The basic structure is *who* (an *actor*) does *what* (*activity*) with *what* (*work object*) with *whom* (another *actor*). A *story* consists of several sentences that are ordered by *sequence numbers*.

**Golo:** For those who haven't used Domain Storytelling yet – can you walk us through the basic mechanics? What does a typical Domain Storytelling workshop look like?

**Henning:** The workshop brings people together: people with questions (like developers and business analysts) and people with domain knowledge – our *storytellers*. They learn from each other in a structured conversation, led by a *moderator*. The moderator visualizes the conversation as a diagram – live, while the conversation takes place.

**Stefan:** The storytellers tell one scenario at a time. A scenario is a concrete, meaningful example of a business process. Usually very few domain stories are enough to grasp a business process.

## In Practice¶

**Golo:** Many people in the DDD community are familiar with Event Storming as a collaborative modeling technique. How does Domain Storytelling differ from Event Storming, and when would you recommend one over the other? Or do they complement each other?

**Stefan:** People are surprised when I tell them that I use Event Storming almost as often as Domain Storytelling. In some cases, it does not even matter which modeling technique is used as long as it facilitates the conversation and a shared visualization is created. Yet, we recommend Domain Storytelling over Event Storming if the domain involves many actors (people or software) and you want to investigate how these actors cooperate with each other. Also, in my experience it is easier to design to-be processes with Domain Storytelling because a common perspective emerges more easily from a moderated, sentence-by-sentence approach.

**Henning:** If you know Event Storming, the first difference that you will notice is the layout of the model. Domain stories do not follow a left-to-right timeline, because they need to show cooperation between actors. That works better with arrows. Another difference is that each domain story is about one scenario.

**Stefan:** The workshop is also different. In Domain Storytelling, iterations of storming (discussing the next sentence) and consolidation (recording that sentence) are much smaller.

**Golo:** Can you walk us through a concrete example of how Domain Storytelling played out in a real project? What was the domain, what did the workshop look like, and what insights emerged that might not have surfaced otherwise?

**Stefan:** Some time ago, I worked on a project which implemented a requirements specification: 80 use cases, 300 pages in total. Our customer's business analyst had done a great job, but as soon as we started working on the software, my colleagues and I realized that the use cases were not sufficient to really understand the domain. We did not know enough about the business processes and the context of the requirements. At our first meeting with the client's team I modeled three scenarios that covered the existing solution. This helped us to gain a better understanding of the shortcomings of the existing solution, which motivated several of the requirements. When we all met again about a month later, we used the same three scenarios as we did the first time, except that we had the domain experts tell us about the to-be processes. Now, we understood the interplay of the use cases. Another three months and several implemented use cases later, we had reached a point where we had to think about going live with the first release.

Our customer had planned an incremental rollout. Some users would switch to our new software, and others would continue to use the old software. To make that possible, we had to enable collaboration between the two software systems. We needed to design a collaborative, software-supported workflow, and again we chose Domain Storytelling for that purpose. During the workshop, we realized that the newly designed, software-supported workflow was not backed by the requirements document. However, it was easy to derive new requirements from the domain story. Basically, we just had to transcribe the activities into text form.

## Domain Storytelling Meets DDD and Event Sourcing¶

**Golo:** One of the big challenges in DDD is finding the right Bounded Contexts. How can Domain Storytelling help teams identify and validate their Bounded Context boundaries? Do you have a systematic approach for this?

**Henning:** Existing boundaries (systems, teams, and departments) confine people's ability to design good Bounded Contexts. We recommend that you first model a "pure" version of your business processes, i.e., leave out existing systems and just focus on what needs to be done to make the process work. Then, we ask the domain experts which activities belong together because they serve a common goal. We then visually group the activities that belong together. The result is not a Bounded Context yet, but a *Subdomain* – and a good starting point for Bounded Contexts. There's more to the design of Bounded Contexts, but that would go beyond what we can discuss in this article.

**Golo:** Event Sourcing is all about capturing what happened in a domain as a sequence of events. Do you see a natural bridge between Domain Storytelling and Event Sourcing? For example, can the stories that emerge from a workshop help teams discover their domain events and event flows?

**Stefan:** You *can* model events, commands, and views as *work objects* with Domain Storytelling. But it does not have "native" support for Event Sourcing, like Event Storming (on the design level) or Event Modeling do. However, it makes sense to combine one of these techniques with Domain Storytelling. Here is an example from the insurance domain: a self-service web app that allows companies to sign their employees up for a corporate insurance policy.

We used Domain Storytelling for the "broad strokes". What is the purpose of the software that we are building? Who is supposed to use it? Does it require or provide data from/to other systems? What domain language(s) do stakeholders use? It also helped us define what is within the scope of the project (e.g., simple plausibility checks) and what is out of scope (e.g., elaborate data verification).

Next, we decided how to slice the story for development. Usually, one to three sentences make a good slice. Then, we did a design-level Event Storming for the first slice and implemented it. Then, we modeled and implemented the next slice, etc.

## Getting Started and Looking Ahead¶

**Golo:** You have built **Egon.io**, an open-source tool for Domain Storytelling. Can you tell us a bit about it and how it fits into the workflow? Are there other tools or resources you recommend for teams getting started?

**Stefan:** You can model domain stories with sticky notes and pen, digital drawing tools, and digital whiteboards. That's okay for occasional modeling. But eventually, you want a tool that makes modeling easy and fast. Egon.io is that tool because it implements the notation and grammar of Domain Storytelling. And it runs in your browser, is free, does not require an account, does not track you, and does not process or store your data.

**Henning:** A good resource for getting started is **domainstorytelling.org** and of course our **book** published by Addison-Wesley.

**Golo:** If a team wanted to try Domain Storytelling for the first time next week, what would be your top three tips to make their first workshop a success?

**Henning:** First, invite the right people. Real domain experts (not just proxies). Second, agree on a scenario. If the discussion drifts off, come back to the scenario. Third, agree if you model an as-is or to-be scenario and on a granularity (coarse-grained overview, fine-grained details, or something in between).

**Golo:** And looking ahead – what's next for Domain Storytelling? Are there new ideas, tools, or directions you're currently exploring?

**Stefan:** Egon is actively being developed. The community has already contributed PlantUML support for Domain Stories, and an Egon-based extension for VS Code is in early development. Rumor has it that Mermaid might be next.

**Henning:** Obviously, AI is a big driver of ideas. Domain stories are used to feed domain knowledge into LLMs, e.g., to generate mock-ups and APIs.

**Golo:** Henning, Stefan, thank you so much for sharing your insights on Domain Storytelling with us. It's been a fascinating conversation, and I'm sure our readers will find plenty of inspiration to try it out in their own projects.

*Henning Schwentner is a coder, coach, and consultant at **WPS – Workplace Solutions** in Hamburg, Germany, and one of the creators of Domain Storytelling. For more information or to get in touch, visit the **WPS website**.*

*Dr. Stefan Hofer is a software developer and consultant at **WPS – Workplace Solutions** in Hamburg, Germany, and one of the creators of Domain Storytelling. For more information or to get in touch, visit the **WPS website**.*
