---
url: https://ai.engineer/code/2026/talks/online-dmitry-maintenance
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

0:00:00Hi, I'm Dimmitri. In this session, I

0:00:03will show how I work on maintaining

0:00:05large AI coding projects. Coding agents

0:00:08made code cheap. They made entropy even

0:00:12cheaper. In the Marvel movie, Thanos

0:00:15watched his home planet Titan collapse

0:00:18under its own growth. So, when the

0:00:20universe got crowded, he raised half of

0:00:23everything. This talk is about the one

0:00:26time I did the same in one project. Our

0:00:30code was growing the way Titan did. When

0:00:32I found what my agents had done, I

0:00:36deleted tests and boil the plate code

0:00:40nobody used. I had to rewrite the whole

0:00:43project and I lost some useful pieces.

0:00:46And that's the dilemma. Drown and chaos

0:00:49or break what people use. And here we'll

0:00:53build the third option for that project.

0:00:56The rewrite didn't hold. The agents kept

0:01:00writing and the mess grew back. So I

0:01:03drew a line above it. There is more

0:01:06change than anyone can check. The third

0:01:09option is to stay under it with small

0:01:12sweeps every week and feedback loops

0:01:15built into the system. Each chapter of

0:01:17this talk is one thing I had to learn to

0:01:19get there. Thanos needed six stones for

0:01:22one snap. Ours are six questions and

0:01:26they light up in the corner as we go.

0:01:29The first one is time. How do we stay

0:01:32under the line?

0:01:34Chapter one. To stay under the line. I

0:01:37first had to know what's right means

0:01:39here. I used to think that tests decide

0:01:42that. So what does right actually depend

0:01:46on?

0:01:48Let's keep one task in mind. Someone

0:01:50wants to change the amount on an order

0:01:53we already approved. Two kinds of agents

0:01:56meet here. One wrote the instructions

0:01:59and the other follows them in

0:02:01production.

0:02:04We run this in 110 countries with about

0:02:0710,000 scenarios of pros instructions.

0:02:11Every task lands in a slice. One process

0:02:14in one country. There's a second stone

0:02:18space. It asks where a change is allowed

0:02:21to go. By default, it stays inside its

0:02:25slice.

0:02:27Most of this system is pros about three

0:02:29quarters by my estimate. So pros needs

0:02:32engineering too. At start we borrowed

0:02:35mutation testing. You break one sentence

0:02:37on purpose and you watch for red test.

0:02:41Here almost none turned red. And if your

0:02:44wrapper is all code, your agent

0:02:47instructions and your tests are your

0:02:49pros. With agents, a rewrite is easy.

0:02:54The hard part is knowing what has to

0:02:56survive it. For code, tests can tell

0:02:59you. For pros, only people can. Agents

0:03:02can't build a complex system on their

0:03:05own. They fix what is easy to see and

0:03:08break consistency on the way. And people

0:03:11also get tired. So we spend their

0:03:13judgment only where it's needed. When

0:03:17business asked check with the team, it

0:03:20hides a real question. Is this a human

0:03:23step or can we automate it? Experts

0:03:25can't answer that. They don't see how

0:03:28the systems connect inside. So

0:03:30developers dig through the traces and

0:03:33logs with their coding agents and they

0:03:35decide which it is. In our task, the

0:03:38team leads decision stays human. The

0:03:41limit was data we could connect.

0:03:45Business experts see the rules.

0:03:47Developers see what's under the water.

0:03:50And that's where most bugs leave. They

0:03:53have their coding agents write those

0:03:55links down so the runtime can check

0:03:58them. Our guide for experts says it

0:04:00simply business tell us what we work out

0:04:04how. One document to readers. Agents

0:04:09read the blogs. So their blocks have to

0:04:11be exact. Experts read the text around

0:04:14them and they sign it. We keep one copy,

0:04:17a second one with drift and works both

0:04:20ways. Our code needs a plain description

0:04:22for experts too. Experts can only fix

0:04:26what they can see. Each of them reviews

0:04:29a task exactly the way the agent saw it.

0:04:33The node becomes the next change made by

0:04:35a coding agent and test it on the same

0:04:38task. So right depends on people. No

0:04:42single person knows it. A group of

0:04:45experts decides and that's why we bring

0:04:48them in from the start. You'll see the

0:04:51soul stone here a lot. We take it in

0:04:54chapter 4.

0:04:56Chapter two. Experts correct one task at

0:04:59a time while agents run the same pros

0:05:02all day. We track every outcome accepted

0:05:06or rejected. The accuracy shows us where

0:05:10people still need to look. I used to

0:05:13think the goal was to move everything

0:05:15into code. So where do we still need

0:05:18judgment? Here is our task again. The

0:05:21pro says the limit is 1,500. The code

0:05:25says a,000. The same order is fine in

0:05:28one place and too high in the other and

0:05:31nothing throws an error. Even a line

0:05:34like do not call X teaches the model

0:05:37that X exists. That's the third stone

0:05:41reality. Is it true or does it only look

0:05:44true for the model? Pros has no

0:05:47compiler. So we built one. Then the

0:05:50author lints every changed file into

0:05:53ledger just like code.

0:05:58Lind can't stop copies drifting aparts.

0:06:02So we watched the walks. A walk is the

0:06:04path one agent takes through the pros.

0:06:07When agents keep walking in the same

0:06:09path, we turn it into one workflow tool

0:06:12and we delete the pros that walked it.

0:06:16Our first workflow of tools made things

0:06:19worse. The pros still named the steps

0:06:22inside them. So agents run those steps

0:06:25again. It worked once. The pros kept

0:06:27only the decisions. Now a walk becomes

0:06:31code only when it repeats and agrees

0:06:34across countries otherwise it stays

0:06:37pros. What still varies stays in pros

0:06:41inside the frame. Code prepares the

0:06:44facts and carries out the decision.

0:06:47Experts own the decision table and the

0:06:50model only picks a row. If no row fits,

0:06:54a person takes over. We haven't built

0:06:56the next step yet. The model will take

0:06:58the small decisions and workflows will

0:07:01connect them. This boundary moves both

0:07:04ways. We moved hundreds of lines of

0:07:07expressions out of code and into pros

0:07:11because experts couldn't read them. A

0:07:14reax is still fine for spotting a

0:07:15critical signal. When there are many,

0:07:18they move into config. Judgment stays

0:07:21wherever the answer varies in a place an

0:07:24expert can read. Chapter three. Now the

0:07:28model decides and its decision becomes

0:07:31an action. I used to think control meant

0:07:35permissions on the agent. So what does

0:07:37it mean to stay in control?

0:07:41Control starts with what the model sees.

0:07:44That's what the fourth stone mind. What

0:07:47does the model know? When we let it

0:07:49choose what to load, it loaded

0:07:52everything. Then it invented a pricing

0:07:55check just because prices were visible.

0:07:58Whatever sits in its context becomes a

0:08:01reason to act. The runtime decides

0:08:04what's the model sees for our task.

0:08:07That's three facts from the order. The

0:08:09full record stays with the runtime and

0:08:12the model makes fewer wrong moves.

0:08:17Here how we minimize context blo for a

0:08:20big set. The model passes a handle, a

0:08:22short name for date that the runtime

0:08:24already holds. The runtime checks it

0:08:27before any write. It keeps the context

0:08:29small and still enough for the decision.

0:08:32At the same time, it uh validates

0:08:35context for consistency. A model with

0:08:39the right facts can still do damage.

0:08:41That's the fifth stone, power. What made

0:08:45change? We control a change twice.

0:08:47during development and again at the

0:08:50right at runtime only code opens the

0:08:53right path so we can change pros many

0:08:56times a day the open rings are the parts

0:08:59that still depend on people let's zoom

0:09:02in and run right five walls stand in

0:09:06front of it and the sixth checks that it

0:09:08landed if a right looks like it failed

0:09:12an agent retries and the journal makes

0:09:15sure it runs only once means much more

0:09:18than permissions on the agent.

0:09:24Chapter 4. Most days I glide over the

0:09:27system. From up here, everything looks

0:09:30fine, so I don't go down into every

0:09:32change my agents made. Then the surface

0:09:36gets rough. How do we know we're losing

0:09:39control?

0:09:41All those controls protect the system.

0:09:43Nothing protected my attention. At the

0:09:46start, I read every change and that's

0:09:48how the system picked up my habits. Then

0:09:52the volume grew past what one person can

0:09:55read. I was burning out and I stopped

0:09:58reading every change. That was the price

0:10:00of the last stone. So, who judges and

0:10:04what does it cost? With it, the gauntlet

0:10:07is complete. Every tech lead I know has

0:10:10been here. The one person every change

0:10:13waits on. agents just get you there in

0:10:16weeks instead of years. I needed a way

0:10:19of working that doesn't run through one

0:10:22head. And the first gauge was me. When

0:10:25I'm tired and I can't explain what

0:10:27changed, that's the signal. The

0:10:30dashboard can be green while that gauge

0:10:33is firing. So, I stop and fix the

0:10:36process first. I don't wait for a

0:10:38number. I couldn't read every div

0:10:41anymore. So I built instruments that

0:10:44watch how things change and their

0:10:46derivatives. One of them is blast

0:10:49radius. Before we built one new check,

0:10:52we replayed its rule over several

0:10:54thousand past sessions. It would have

0:10:56fired on less than 1%. Where the numbers

0:11:00jump, I go down and read. When something

0:11:04looks wrong, we wait. One signal waits

0:11:06until it comes back, even when it comes

0:11:09from an expert. A cluster across slices

0:11:12is different. Then we fix the concept

0:11:15behind it once. Left alone, an agent

0:11:18keeps adding conditions until today's

0:11:20tasks pass. The eval score drops when

0:11:23you take them out and you get an honest

0:11:26list of gaps.

0:11:31One red slice is rarely alone. The

0:11:34experts note becomes an eval and an

0:11:36agent searches the siblings for the same

0:11:39failure. Siblings differ on purpose by

0:11:42market and by local law. So we sweep

0:11:45them and the fix goes only where the

0:11:48failure is confirmed.

0:11:53At any time 10 to 20 of these loops are

0:11:57open. At one point I had become the

0:11:59message bus between users and my agents.

0:12:03At this scale, every user has to be an

0:12:06evaluator with a screen to give feedback

0:12:08without me. My job as the architect is

0:12:12to build those loops into the system.

0:12:15Each loop writes to a ledger. So nothing

0:12:18waits on my memory and nothing waits on

0:12:20me.

0:12:24Agents forget everything when a session

0:12:26ends. So the memory has to leave in

0:12:29files. We keep two kinds. what happened

0:12:32and what is true now. The next session

0:12:35starts from what is true now.

0:12:40Every issue has to connect to its root

0:12:43cause. Without that, the agent decides

0:12:46on wrong facts and the check has nothing

0:12:49to count. So we don't act without

0:12:51evidence.

0:12:54Reality comes back. In chapter two, it

0:12:58asked if the pros is true. Now it asks

0:13:01if the change is I used to trust green.

0:13:05So what would make us change our mind?

0:13:08Green was tempting because checking is

0:13:11where the times goes. My agent spend

0:13:14almost 9/10en of that time on a change

0:13:17just checking it and checking took a

0:13:19bigger share of my own week too.

0:13:22Verification speed is the limit.

0:13:27We rank evidence by what it proves. A

0:13:30replay on real tasks sits at the top.

0:13:34Before the data arrives, we write down

0:13:36what would prove us wrong. And every new

0:13:39check has to fail once on purpose before

0:13:42we trust it.

0:13:46AI refactors fail quietly. Every

0:13:50condition and every decision table row

0:13:53is a branch. And after a refactor, each

0:13:56one has to land exactly once. Here the

0:13:59branch for amounts over the limit got

0:14:01lost. Our [snorts] order would have gone

0:14:04through without the team lead. Real

0:14:07tasks decide what we test. We freeze

0:14:11them as a set and replay the old version

0:14:14against the new one. If no real task

0:14:17takes a branch, we call that no

0:14:19evidence.

0:14:21You can't trust one agent to judge its

0:14:24own work. A second agent checks it with

0:14:28a checklist and without the author's

0:14:30reasoning. It can't be talked into

0:14:32agreeing. Over 6 months, it grew to most

0:14:36of my checking.

0:14:39Coding agents finish most of a batch and

0:14:43quietly skipped the rest, especially

0:14:45when their context is full. The ledger

0:14:48shows what was skipped in one backlog.

0:14:51Most items looks covered, but only about

0:14:54a quarter could really be closed. So now

0:14:58evidence we can count is what changes

0:15:01our mind. And this is the third option

0:15:05from the start of the talk. I used to

0:15:08think there were two choices. Keep

0:15:11everything or snap. So what is worth

0:15:14carrying forward?

0:15:16This one process over a year. Some

0:15:19branches merged back, some still drift,

0:15:22and some are dead but still look alive.

0:15:25Which ones deserve to survive?

0:15:30A fix is only a moment. These forces

0:15:33keep pushing right after it. You've met

0:15:36most of them already. This chapter deals

0:15:39with regrowth and branching.

0:15:44The first gauge was me back in chapter

0:15:464. These are the gauges for the code.

0:15:50Time gets its answers here. The bridge

0:15:53line comes back as a number. A ratchet

0:15:56means the count can only go down. Two

0:15:58gauges run today and six more are next.

0:16:04Every week real use decides what stays.

0:16:07Anything no real task reaches gets a

0:16:10name and its evidence before it goes.

0:16:13Test the evidence I didn't have when I

0:16:16snapped. Tests kept that variants alive.

0:16:20Agents kept every option a test still

0:16:23mentioned. Now retired variant takes its

0:16:27test with it. Here's the whole method as

0:16:30one week. This is the third option as a

0:16:34process.

0:16:39together. That's the full gauntlet.

0:16:42Thanos used his for one snap. We use

0:16:45ours every week in small doses with

0:16:48evidence. I haven't had to snap again.

0:16:55Soul had a second price and I only saw

0:16:58it later. After we deliver it to

0:17:01production, I found my own engineering

0:17:03principles built into the systems

0:17:06design. Its skills and its memory carry

0:17:09my habits and my biases. Conway said the

0:17:12system copies the orc chart. Ours copies

0:17:16whoever corrects it. Here are 18

0:17:19questions, three for each stone. I'm

0:17:22using them when working on my projects.

0:17:26Pause here and pick one your own

0:17:29repository can't answer yet. That's

0:17:32where to start.

0:17:37I learned to use this process when I was

0:17:40already burning out. If you're the one

0:17:43your team's changes weight on today,

0:17:46this is for you. Use your stones while

0:17:49you can still read every change. In the

0:17:52end, Thanos retires to a garden. Maybe

0:17:55he should start using them daily. Thank

0:17:59you.
