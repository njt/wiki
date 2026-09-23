---
url: https://arinco.com.au/blog/vibe-coding-has-arrived-in-power-platform-and-technical-debt-has-never-been-easier-to-create/
date_fetched: 2026-09-23
---

*AI has dramatically reduced the cost of creating software. It has not reduced the cost of owning software.*

For years, one of Power Platform’s biggest promises has been democratisation. Give more people the ability to solve problems. Reduce dependency on traditional software development. Let the people closest to a business problem build the solution, and I’m not here to complain about how much trouble that mission has given us power platform developer.

Now AI has taken that idea and put it on steroids. We are quickly reaching a point where someone can describe an application in natural language, have an AI agent generate much of it, connect it to enterprise data, iterate on it conversationally and get something working without understanding much of what is happening underneath.

And I think that’s incredible. I also think it creates one of the biggest governance challenges Power Platform has faced.

## Building was never the hardest part

There is a dangerous moment in almost every technology project. It is the moment when something works. The screen loads. The button does what it is supposed to do. The API returns data. The demo goes well. Everyone gets excited, I GET EXCITED!

From then on things fast track, management knows what you have built, End users are becoming interested, the spotlight turns on you and suddenly, before you know it the prototype becomes a production application.

However, saying “it works” has never been the same thing as “it is production-ready.” Behind even a relatively simple enterprise application are questions that have very little to do with how quickly someone can generate its interface:

- Who owns it?
- How is it deployed?
- What identity does it use?
- Where are the credentials?
- Which APIs does it call?
- What happens when one of those APIs changes?
- What data can leave the environment?
- How do we know when it fails?
- Who receives the incident?
- How do we roll it back?
- Who supports it when its creator moves to another role?
- How was the solution tested?
- Where is the architecture documented?
- What happens six months from now when someone else needs to change it?


AI can help write code. It cannot make those questions disappear. In fact, by allowing us to create considerably more software, considerably faster, it arguably makes them more important.

## Complexity doesn't disappear, it just becomes invisible

And if I’m being honest with you, watching all the vibe coders skip past the complexity has only brought me anxiety.

Traditional low-code already created an interesting problem: a solution could look simple from the outside while hiding significant complexity underneath. That issue has always been more prominent with power platform and PowerApps. If only I had a penny every time someone told me “It’s just a drag and drop” I would be typing this blog on my personal yacht.

Even the simplest canvas apps might depend on several flows. Those flows might use multiple connection references. Those connections might belong to individual users. A custom connector might call an API. That API might depend on another system. The application might exist in an environment nobody originally intended to become production.

And eventually somebody from the platform team gets a message:

“The app has stopped working. Can you fix it?”

AI accelerates this. A maker no longer necessarily has to understand each component in order to create it, and that is part of the value. But abstraction doesn’t remove complexity. It relocates it and eventually somebody has to understand what was created.


### "But Copilot built it"

Is a dangerous sentence and I suspect this is going to become the new version of: *“It worked on my machine.”*

When something fails, saying that an AI generated it doesn’t change the organisation’s responsibility for the solution.

If an application processes sensitive information, integrates critical systems, or supports an important business process, somebody still needs to understand its architecture. If an AI generates a questionable authentication pattern, someone needs to recognise it. If it introduces a dependency that nobody realised existed, someone needs to find it. If it creates inefficient queries, unnecessary API calls or brittle logic, someone eventually has to maintain it.

AI-generated software is still software. And enterprise software inherits enterprise consequences regardless of who or what wrote it.

### The problem isn't citizen developers

This is where I think the conversation sometimes goes wrong. The answer isn’t:

*Stop non-technical people from building things.*

That would throw away one of the greatest strengths of the Power Platform. Some of the best solutions originate from people who understand the business problem deeply but aren’t professional developers. AI can make those people dramatically more capable. That is a good thing.

The real problem is treating every application as though it carries the same level of risk. An app someone creates to organise their personal workload is not the same as an application used by 500 employees. A departmental workflow is not the same as an integration updating a financial system. A prototype is not the same as a business-critical application.

The governance model therefore shouldn’t simply ask:

*“Are you allowed to build?”*

It should ask:

*“What are you building, what does it touch, and what happens if it fails?”*

## The barrier should move, not disappear

Historically, organisations placed most of the barrier at the beginning of development:

- Who is allowed to build?
- Who gets a developer environment?
- Who gets access to the platform?
- Who gets access to connectors?


AI makes that increasingly difficult and in many cases undesirable. Instead, I think the barrier needs to move further down the lifecycle.

### Experimentation should be easy. Production should be deliberate.

Build whatever you like in an appropriate development space. Experiment. Prototype. Use AI. Break things. Learn. I would not want to stop others from learning but the moment that solution starts becoming important to the organisation, the requirements should change. Now we need to know:

- Who owns the solution?
- What environments does it move through?
- Is it packaged correctly?
- Are connection references being used appropriately?
- Are personal connections involved?
- Has Solution Checker identified issues?
- Does the application comply with data policies?
- Have integrations been reviewed?
- Is there monitoring?
- Is there support documentation?
- Is there a rollback path?
- Does the support team know the solution exists?


That isn’t bureaucracy for the sake of bureaucracy. That’s the difference between building an application and running a service.

## Governance has to become technical

There is another uncomfortable consequence of AI-assisted development: *governance documentation alone is not going to be enough.*

You can publish a beautiful 40-page governance framework explaining exactly how applications should be built. AI will still happily help someone build something at 11 PM without ever opening it.

Modern governance needs to increasingly exist inside the platform itself. That means using capabilities such as environment strategies, data policies, Managed Environments, deployment pipelines, security controls, sharing restrictions, monitoring and automated quality gates.

Microsoft is already moving further in this direction. Managed Environments brings together controls including environment groups, data policies, pipelines, usage insights and Solution Checker, while Solution Checker enforcement can now warn about or block problematic solutions during import.

The important shift is this:

*Governance should increasingly become something the platform does, not something we hope people remember.*

That becomes especially important when the person building the solution may not know which questions they are supposed to ask.

### AI changes the role of the developer

There is a popular question at the moment:

*Will AI replace developers?*

In Power Platform, I think the more interesting question is:

*Which parts of development become less valuable when AI can perform them easily?*

If producing screens, formulas, components and basic application logic becomes dramatically faster, then the differentiating skills move upward:

- Architecture
- Security
- Integration design
- ALM
- Observability
- Performance
- Governance
- Understanding business processes
- Recognising bad patterns
- Designing for failure
- Knowing what should not be built


And, perhaps most importantly, being able to look at something generated in five minutes and determine whether the organisation should trust it for the next five years.

The Power Platform developer doesn’t disappear. The job moves further up the stack.

### AI doesn't eliminate technical debt it manufactures it at scale

Technical debt traditionally accumulated because teams were under pressure to deliver quickly and nowadays most people seem to think AI has somehow removed technical debt. I wish!

Now imagine giving every employee an incredibly capable junior developer who works instantly, never gets tired and enthusiastically implements almost anything you ask. That is an extraordinary productivity opportunity. It is also an extraordinary technical-debt-generation machine if there are no boundaries around it.

The organisations that succeed with AI-assisted development won’t necessarily be the ones that allow the most people to build the most applications. They will be the organisations that create the safest path from:

Idea → Experiment → Application → Production → Operations → Retirement

Because the future problem probably won’t be:

“How do we build enough applications?”

It will be:

“How do we responsibly manage everything we can now build?”

## The uncomfortable conclusion

Vibe coding isn’t going away. Nor should it. AI-assisted development is going to make Power Platform dramatically more accessible and significantly more powerful, and trying to stop that wave would be pointless.

But we need to stop pretending that making software easier to create automatically makes software easier to operate.* It doesn’t.* A five-minute build can still become a five-year responsibility.

And as the barrier to creating applications approaches zero, the discipline around deciding what enters production needs to become stronger not weaker.

*So by all means, let AI build the app.*

*Just make sure somebody knows who is going to own what it leaves behind.*
