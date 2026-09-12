# AI, Tools and Transformation

Benedict Evans' September 2026 essay on why "AI will sweep away enterprise software" misunderstands where software comes from, how people use it, and how companies change. The intoxicating tool-builder fantasy — make the tool in five minutes, or just have the model do the task — collides with three realities: most people aren't tool-builders, the hard part was never writing code but knowing you need a tool, and enterprise software lives on a spectrum from institutionalised (SAP) to improvised (Excel) that AI doesn't collapse — it adds one more improvised substrate, the chatbot.

---

## Key Quotes

> "Most people are not tool builders, and most people don't instinctively think about how their job could be done in a different way."

The essay's load-bearing observation. Silicon Valley's blind spot is that it is full of people whose entire job is rethinking how work gets done. A great matrimonial lawyer thinks about cases and clients, not discovery software; a great enterprise salesperson thinks about deals and competitors, not sales-enablement tools. The task sits in plain sight and its owner doesn't see it — which is exactly what the "forward-deployed engineer" exists to fix.

> "None of this is solved by making it easier to write code… The hard part is knowing that you need a tool for this in the first place, and then knowing what the tool should do."

The direct counter to "AI makes software free." Most of what we've automated wasn't obvious and didn't have an obvious solution — half a dozen failed startups usually precede the one that got the problem right. This echoes [[The AI Productivity Paradox]]'s "knowing what to build is the hard part," but pushes further: even the *need* is invisible to the person doing the work.

> "You pave the desire path and pay someone to set it in stone."

The institutionalisation step. Edge cases start improvised in Excel and email; once a task is done the same way every time, by lots of people, with revenue and risk attached, the company has to institutionalise it — audit, security, maintenance, accountability. This is why the company has hundreds of apps, and why "just ask the model" can't replace them.

> "AI doesn't change the question: it creates new choices and moves the thresholds."

The thesis compressed to a sentence. A small firm hiring five people stays on Google Sheets longer because AI makes it more scalable; a team inside PwC still improvises around Workday because it's too inflexible. The chatbot is a new freeform substrate next to Excel and email — taking tasks from apps and losing tasks to them.

> "Yes, you gave everybody a web browser, but that wasn't how you rebuilt your supply chain management around the internet…"

The historical analogy that deflates "give everyone Copilot." Giving everyone Lotus 123 in 1983 or a browser in 1997 didn't transform invoice processing or supply chains — pilots and structural process change did. This lands the same point as [[Laura Tacho — Data vs Hype]]'s "adoption is not transformation," but from thirty years of platform-shift history.

## Key Themes

- **#concept Institutionalised vs improvised** — software sits on a spectrum from top-down/institutionalised (SAP, Carta, Rippling) to bottom-up/improvised (Excel, email, Tableau, now the chatbot). Tasks migrate between poles as they gain repetition, revenue, and risk.
- **#concept The tool-builder's blind spot** — the Silicon Valley assumption that everyone sees their job as a workflow to be automated. Most people see cases, clients, and competitors.
- **#pattern Pave the desire path** — the institutionalisation trigger: when a task becomes repetitive, cross-departmental, and risky, you hire someone to set it in stone. This is where SaaS apps come from.
- **#pattern Pilots don't scale** — the CIO runs five or ten pilots; the CEO asks why hundreds of workflows aren't transformed. Giving everyone a model scales theoretically, but most people don't find ways to use it.
- **#concept The three questions** — every company must ask buy/build/deploy, how-far-does-this-change-operations, and is-this-an-existential-threat. Professional services (Accenture, the Big Four, Bain/BCG/McKinsey, the labs' "deploycos") monetise all three.
- **#pattern New things, not old things faster** — the real payoff of a platform shift is work that wasn't possible before, not the existing work done faster.

## Critical Analysis

**The strongest card Evans plays is the spectrum, not the critique.** The institutionalised/improvised axis is a genuinely useful frame that most AI-adoption writing lacks. It explains simultaneously why the tool-builder fantasy is wrong (most tasks are already institutionalised in SAP, and the improvised remainder is invisible to its owners) and why the "AI kills SaaS" story is too simple ([[AI Killing B2B SaaS]]): Carta is a $4bn company that manages one spreadsheet, and tasks migrate both ways — a consultant told Evans half their jobs were telling Excel users to use a database, the other half the reverse.

**Where Evans is weakest is the thing he names at the end.** After an essay arguing that transformation is slow, contested, and organisational, he closes with "the stuff that actually mattered was the stuff that wasn't even possible before." True, and almost certainly right — but it's the moment the analysis stops and gestures instead: he never says how you'd spot the new-possible thing while it's happening, which is the actual question. [[AI as an Enterprise Operating System]] at least attempts the recipe; Evans only names the gap.

**The "nobody sees the problem" argument cuts both ways.** Evans uses it against the tool-builders ("you can't automate what you can't see"), but it's also the argument *for* the forward-deployed engineer — the person who can. The essay treats the FDE as a joke ("anyone OpenAI hired from a systems integrator"), yet the deploycos and the Big Four's AI practices are the market voting that the discovery problem is real and someone must be paid to solve it. [[AI Mania Is Eviscerating Global Decision-Making]] shows how badly that market currently malfunctions.

**The historical analogies do a lot of work, honestly.** Lotus 123 in 1983 and the browser in 1997 are the right instincts — per [[Four Time Scales for Technology Development and Deployment]], organisational reshaping runs decades, not quarters. But Evans doesn't contend with the possibility that AI's threshold-moving is faster this time, precisely because it attacks the improvised substrate (Excel, email, the spreadsheet) directly rather than requiring a new system of record first. That is the one place the analogy may understate the change.

**Compare to:** [[AI as an Enterprise Operating System]] (Dan Guido's recipe for the reorganisation Evans says is missing), [[The AI Productivity Paradox]] (knowing what to build is the hard part), [[Laura Tacho — Data vs Hype]] (adoption is not transformation, with data), [[AI Killing B2B SaaS]] (the SaaS side of the bundling/unbundling cycle), [[AI Mania Is Eviscerating Global Decision-Making]] (the pilots-don't-scale failure in the wild).

---

*Sources: [[raw/ai-tools-and-transformation]], [[summary/ai-tools-and-transformation]]*
*Last updated: 2026-09-04*
