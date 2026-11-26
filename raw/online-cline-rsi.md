---
url: https://ai.engineer/code/2026/talks/online-cline-rsi
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

3:59:46Uh, hey guys, this is Era. I'm a

3:59:49founding engineer at Klein and today I'm

3:59:51going to be talking about recursively

3:59:52self-improving coding agents. So this is

3:59:55a very interesting conversation in the

3:59:59sense that everyone wants to understand

4:00:01what RSI is, how we accomplish it, how

4:00:03we go through it and how it affects our

4:00:04career. So I'm going to start with

4:00:06giving you just three goals of this

4:00:08entire conversation. Just three simple

4:00:10goals and that would help you understand

4:00:13like what what we get out of it and what

4:00:14the future uh for this looks like. So

4:00:16the goal number first is that

4:00:18understanding the intent of Frontier

4:00:19Labs, right? Everyone who's outside of

4:00:21the frontier lab is always trying to

4:00:22speculate like hey what what is the

4:00:24topic in open or what is deep mind going

4:00:26to come out with or what what are they

4:00:27trying to make and what's the future

4:00:29going to look like who has cracked the

4:00:31RSI so like this kind of helps you get

4:00:34get a rough outline of what their road

4:00:36map looks like and the second thing is

4:00:37that like if the models are improving so

4:00:40fast like how do you think about your

4:00:41career or the next few years right that

4:00:44is really what we want to understand we

4:00:45want to understand like okay things are

4:00:48changing so fast like the ground

4:00:50underneath us is shifting so quickly.

4:00:52Right now like it's like uh

4:00:55there used to be a time when you could

4:00:56like demonstrate your side projects on

4:01:00your personal website or GitHub and now

4:01:02it's just like you could literally

4:01:03oneshot them using any Frontier Coding

4:01:05Engine and with that it's like it's like

4:01:08oh well how how do we demonstrate proof

4:01:10of work? How do we demonstrate you know

4:01:12that that that we know what we're doing

4:01:14and how do we think of our career for

4:01:15the next coming years, right? So it's a

4:01:18very interesting times that we're living

4:01:20in. And then the third and the most

4:01:21important goal of the conversation is

4:01:22escaping the permanent underclass,

4:01:25right? So I know this seems like a joke,

4:01:27but this really is a very valid

4:01:29possibility that people are

4:01:31contemplating, everyone's like, "Oh

4:01:32well, if everything that we want to be

4:01:35doing can potentially easily done by a

4:01:37frontier model, like what are we what

4:01:38are we good for?" And like where does

4:01:40software go? Where where does the future

4:01:42of technology going to go? And what are

4:01:43we going to do as as things evolve so

4:01:45fast, right? So I think that there's

4:01:48like a lot of like fear-mongering but

4:01:50also there's like some very valid merge

4:01:52where people are concerned that like if

4:01:54all the revenue of the planet is going

4:01:56to go to um a few hardware companies

4:01:59that make frontier GPUs and then a few

4:02:01few uh AI companies that that make the

4:02:05best models like how are we how are the

4:02:07rest of us going to throw in what what

4:02:08are we going to do here? So, I think

4:02:11I'll I'll walk you through a few

4:02:12examples and a few ideas that will like

4:02:14really really help you reconsider the

4:02:17career choices and and and the life

4:02:19choices you're making and what your

4:02:20problems you're picking up. And then and

4:02:22then that will help you like realign

4:02:25what it is that we should be working on

4:02:27and what it is that you you know would

4:02:29would that you could work on that will

4:02:31stand the test of time and help you both

4:02:33just not just like on a psychological

4:02:35but like financial and other levels as

4:02:36well. So uh to give you a very precise

4:02:38example, let's start with something very

4:02:40precise that I had done. So a couple

4:02:42months ago I uh came up with recursive

4:02:47improve improvements on a coding

4:02:49harness. So this was the first probably

4:02:51public example of uh an automatic eval

4:02:54improvement. So to give you an example u

4:02:57we [snorts] used to do this thing where

4:02:58we would measure the quality of a coding

4:03:01agent using evaluation scores.

4:03:04on terminal bench 2.1 Kim K3 with client

4:03:07would score um usually like you know um

4:03:1060 70% and then depending on like how

4:03:14good the model is like you get better

4:03:15scores on on terminal bench 2.1 so we

4:03:18noticed that when Kim K3 came out our

4:03:19score was 77.5% which is a good score

4:03:22but we were like oh is this is this

4:03:24truly is it truly the best score that

4:03:26this model can get on this benchmark and

4:03:28also like what does the model release

4:03:31paper claim about what their best score

4:03:33is so the model release paper said that

4:03:34they could score up to 85% or even more.

4:03:37So we were just like okay okay so we are

4:03:40not getting the sort of score and often

4:03:41what we do with model releases in client

4:03:43is that we try to like optimize your

4:03:45score and try to get the best out of the

4:03:46model. So when we started hill climbing

4:03:50on the on the model like we we realized

4:03:53is we realized that like you know

4:03:55there's a lot of steps that we do says

4:03:57that we do it gracefully says that we

4:03:58pursue the right kind of problems and do

4:04:00it the right way and we started thinking

4:04:03like okay if we there's a lot of ground

4:04:05work here how do we automate it and this

4:04:07is a demonstration of like how we

4:04:08automated it. So I'll I'll just leave

4:04:10that as a placeholder for now and we'll

4:04:12get get back to it later where we talk

4:04:13about in detail of like how does like

4:04:15this automatic improvement loop of a

4:04:18client harness work. So this basically

4:04:20means that like you have a coding a

4:04:21harness that could just like basically

4:04:22improve itself and become better on on

4:04:25the same set of problems. So let's start

4:04:27with the ground terms. What is recursive

4:04:29self? Recursive self-improvement is the

4:04:31idea that an AI system that helps

4:04:33[snorts] build a more capable successor

4:04:36which repeats the process you know until

4:04:37you hit artificial general intelligence

4:04:40right you have better systems you have

4:04:41better improvements you have a better

4:04:42system and then you know you have

4:04:43singularity so a classic example is like

4:04:46a self-reing coding agent right

4:04:48self-reving coding agent is just like a

4:04:50coding agent that can improve itself so

4:04:52you know this this is all like a pie in

4:04:54the sky idea like it's like it's like

4:04:55you know yeah sure this looks cool but

4:04:57how how did this even come out to be

4:04:58like how we even get to here. So if you

4:05:02notice in topics road map in 2021 in

4:05:052021's road map they very specifically

4:05:07said that uh uh they wanted to develop a

4:05:10general AI assistant that can interact

4:05:12conversationally with a user to create

4:05:14edit and explain software. It would

4:05:16display all the capabilities of a

4:05:18software engineer colleague. It would

4:05:19write functions. It would identify bugs.

4:05:21It would design an API discuss style and

4:05:23best practices. Right? So we don't

4:05:25expect to reach this goal right away but

4:05:26it's in the direction which we're

4:05:27striving. It's interesting how this was

4:05:29more than five years ago. More than five

4:05:31years ago, people had predicted that

4:05:33there exist a day where you will have

4:05:36like a full-blown journal assistant that

4:05:37can basically act like a colleague,

4:05:39which we do have right now. Um, then in

4:05:412022, Chad Deputy got released. We all

4:05:43know how that happened. It was it was a

4:05:45fun moment. It was like the first time

4:05:47things got sparked for us. Um, then in

4:05:502023, 2024, there was the first time

4:05:53chat models could call functions. These

4:05:56functions would be simple functions like

4:05:57hey what's the weather or you know

4:05:59what's the what's you know what's this

4:06:02competition over this computations

4:06:04result in Python. Um and then finally in

4:06:062024 when s 345 was released that was

4:06:09when the agents could like truly call

4:06:11like actual tools the way we call like

4:06:14think of tools. So it's like reading a

4:06:15file editing a file you know reading

4:06:18searching something on web uh listing

4:06:20listing a folder searching and grapping

4:06:22through you know uh a file system things

4:06:24like that. So and in 2024 we had the

4:06:27intuition that like chatbot what if

4:06:29chatbox could actually talk to each

4:06:31other you know so then that's was the

4:06:33first time when we had like you know uh

4:06:35multi-agent conversations now at the

4:06:37time these things were very janky right

4:06:38like they were just like taped together

4:06:40in this like weird weird uh bizarre ways

4:06:44where you have just like these classes

4:06:46and these links and then like these

4:06:47hierarchies of classes and it was like

4:06:49it was all like kind of like a very

4:06:50janky attempt to like make us think that

4:06:53like these these models to like have

4:06:55these pipeline of flows of responses

4:06:56that can all communicate with each

4:06:58other. [snorts] A lot of it was like

4:07:00mostly deterministic and it was like you

4:07:02know it was not as free flowing as it is

4:07:04right now. So and you know we started

4:07:07getting the intention. So this was the

4:07:08autogen project but like in only in 2025

4:07:11we we started [snorts] getting the like

4:07:12what if the agents just kept getting

4:07:14better and faster and cheaper like what

4:07:15what if that just kept happening and

4:07:17then in 2026 uh we we had the intuition

4:07:21that like what if what if the agent

4:07:23could improve itself which is like the

4:07:25final you know the final part of the

4:07:27whole process. So let me give an example

4:07:29like what does it mean for an agent to

4:07:30improve itself right? So like it's like

4:07:32a lot of times when we think of like

4:07:34evaluating an agent, we initially used

4:07:36to think of evaluating an agent in the

4:07:38form of like just a single turn

4:07:40conversation. So we would tell we would

4:07:42tell like um um agents that like ask a

4:07:45simple question like how many toes does

4:07:48a dash animal has? It's like it could be

4:07:50let's say how many toes does a cat have

4:07:51and then al would go through and the

4:07:53element would like prepare response and

4:07:55then we would get the response. So like

4:07:56a cat has maybe 10 toes, 18 toes,

4:07:58however many. I don't I actually don't

4:08:00know how many toes a cat has. But

4:08:01anyway, um yeah. So then we create the

4:08:05response as like a yes or no. But like

4:08:07when you work with an agent, there's a

4:08:09ton of things that you're doing. So let

4:08:10me give an example. Let me give an

4:08:12example. So for example, you're going to

4:08:13ask an agent like hey set up this MCP

4:08:15for me. So now it's not just like it's

4:08:18not just like you know a trivial

4:08:20oneshotable like thing like it will like

4:08:22search online and go what's the MCP?

4:08:24what is the config? So, I'll like edit

4:08:26and download some soft files. Like maybe

4:08:28I need to add database update the MCB

4:08:30tool. Maybe I need to read and edit some

4:08:32files. Maybe I need to run some bash

4:08:34commands. Right? So, you do all of that

4:08:36and then the agent response like, hey, I

4:08:38did it. And then you grade that with a

4:08:40bunch of tests to gauge like, hey, did

4:08:42the agent actually accomplish the thing

4:08:44that it said it would do? So to do that

4:08:47thing the second part which is like more

4:08:48intricate where you're actually like

4:08:51truly you know testing agents like uh to

4:08:54their best extent like it's like you

4:08:56know um we use this thing called

4:08:58terminal bench. terminal bench is a

4:09:00benchmark for you know AI agents in

4:09:01terminal environments and that is

4:09:03something that we are we have a lot of

4:09:05like experience with at client and uh

4:09:08it's like a it's like one of the best

4:09:09ways to like really gauge like is your

4:09:11coding agent accomplishing uh in general

4:09:13the task that you're doing uh that you

4:09:15want to be doing with age model. So

4:09:18there's a couple there's a many

4:09:19different parts of evaluation suit. So

4:09:21ideally you want that to run in um you

4:09:25know [snorts] you you want that to run

4:09:27in um individual sandboxes. Then you

4:09:29have like independent graders that like

4:09:31check everything like how many turns it

4:09:33took, how many tokens it took, how much

4:09:35latency it took, how many how many how

4:09:37many tool calls did it make, what was

4:09:39the trajectory looking like and then

4:09:40grading all the results for things like

4:09:42hey did you not reward hack? Did you not

4:09:43cheat through the solution? Did you not

4:09:45just like look up the solution online?

4:09:47Things like that. Um so we we basically

4:09:50like a lot of these uh data sets of

4:09:52evaluation problems have a set of tasks.

4:09:55So when a terminal bench 2.1 they have

4:09:5789 tasks. So what we do is like we spin

4:10:00up 89 parallel containers where they all

4:10:03run online and they run in parallel and

4:10:05they're sandbox and then they each try

4:10:07to solve a problem. So then they're all

4:10:10uh in parallel and independent of each

4:10:11other. And then at the end we get get a

4:10:14score. So our score could like give us

4:10:16measures like is is a is this is the

4:10:19model performing well and what kind of

4:10:21failures it has right so like you run

4:10:23the evaluation you get a score so you

4:10:24get an aggregate score and there

4:10:26aggregate score could reveal a set of

4:10:28failures okay failures could be of any

4:10:30kind failures could be hey the read file

4:10:32tool is not working sometimes the

4:10:34entrance is so slow that like you know

4:10:36the agent's working it's doing it's

4:10:37making progress maybe it's running a

4:10:39circle but it's so slow that it

4:10:40literally like runs out of time it runs

4:10:42out of like 45 minutes or where that

4:10:44were given for the task and just like

4:10:46stop working. So once you do that once

4:10:48you understand hey this is what we ran

4:10:50this is what the failures we got we

4:10:52bucket the failures in different

4:10:53categories. So like 10 failures of you

4:10:55know this kind of tool called 10 10 20

4:10:57failures of like say inference not

4:10:58working or something else right and then

4:11:00we once we bucket the failures into

4:11:02different categories where like we try

4:11:03to make an improvement to the harness

4:11:04because like what if there's like

4:11:06something wrong with the harness that we

4:11:08could change such that the model score

4:11:09would improve right and on and on and on

4:11:11and you do this iteratively it takes a

4:11:13while it's just like you know hill

4:11:14climbing is like a very groundworky kind

4:11:16of process and you know usually usually

4:11:19we we took we used to take a lot of time

4:11:22trying to do this but it was a great way

4:11:23for us to like learn how to improve

4:11:25models. So [snorts] one way to hill

4:11:28climb on top is that like you know we we

4:11:32could have a thing where you know we

4:11:34started doing this thing where we could

4:11:36just like what if we could have like a

4:11:39longunning agent that runs for a while

4:11:40and automatically improves the model

4:11:42score and that's what we did. So we ran

4:11:44GPD 5.6 soul that led the work and that

4:11:47was like the main agent and then uh Ky

4:11:50K3 was just running eval. So what it was

4:11:52doing is that like GBD 5.6 was like hey

4:11:54run these events get these cores let me

4:11:57look at the failures let me look at

4:11:58where where it went wrong and then I'll

4:12:01make changes to client's harness and

4:12:03then I'll run these eval again and then

4:12:05I'll keep trying. So again, you can see

4:12:07how like very quickly this can like go

4:12:09off the rails because this is supposed

4:12:10to be like the running and evaluation

4:12:11itself takes like an hour. Then you have

4:12:13to read the feedback. Then you have to

4:12:15like see if you're properly interpreting

4:12:17the feedback to make reasonable changes

4:12:19to the harness that are like not reward

4:12:21hacking that are just like actual

4:12:22genuine improvements and then run the

4:12:23eval again. So we ran that for 17 hours

4:12:26and uh when we ran that for 17 hours

4:12:28there was like a legitimate hell

4:12:30climbing and what we found that was very

4:12:32interesting. This happened with a very

4:12:34detailed prompt that accomplished this

4:12:36is that like we had baseline score of

4:12:3927% and then you know we we found issues

4:12:43like loop detection loop detection is

4:12:44when a harness just runs in circles

4:12:46around a problem right uh we found like

4:12:48background process grinding where

4:12:50background processes would run too long

4:12:52uh we found some file index issues and

4:12:54other things as well I mean when I say

4:12:56we what I actually me 5.6 solve right so

4:12:59that found those issues that found and

4:13:01bucketed those issues it's like hey how

4:13:03about I make those changes in the

4:13:05harness and I run the eval again and it

4:13:07iteratively built the hill climbing

4:13:08process completely autonomously on its

4:13:10own without my interference and 13 hours

4:13:12later when I come back to it like it has

4:13:14improved the harness's performance in so

4:13:16many ways so see what does that mean

4:13:19what does that mean it just it just

4:13:20basically means that like we're at a

4:13:23point where the experience could make

4:13:26the harness better that like there's

4:13:27like a tool level learning loop where

4:13:29the harness does some action, interacts

4:13:31with the environment, understands

4:13:33whether or not it did the right thing or

4:13:34the wrong thing and gets some reward and

4:13:37and like almost and then has there's

4:13:39like a meta agent that's observing the

4:13:41main harness and finding the changes to

4:13:44it and just like incorporating those

4:13:46changes and improving. Now it's I think

4:13:48in my personal opinion it's much harder

4:13:49to do in real time because if you do it

4:13:51in real time you overindex on solving

4:13:54certain problems or you're just like too

4:13:56cartel to a specific thing. But I do

4:13:59think that this is something that works

4:14:00like on a broader scale of like working

4:14:02on a wider range of problems [snorts]

4:14:04and you want to be able to like you want

4:14:06you want to be able to justify the

4:14:08changes you're making on a broader

4:14:10applicability so that the changes you

4:14:12make here will apply to a broader range

4:14:14of problems as well. So what does that

4:14:18mean? Like what am I trying to get at

4:14:21here? The point I'm trying to get at is

4:14:23that the models are so good right now

4:14:24that they can if they can improve the

4:14:25harness, they can improve a lot of other

4:14:27things. So then what else can they

4:14:28improve? So this is this is a this is a

4:14:31post uh primarily from OpenAI where GP

4:14:345.6 autonomously rewrote and optimized

4:14:37its own production kernels and those

4:14:39production kernel improvements remove

4:14:41the end to-end serving costs by 20%. you

4:14:44know it also had 15% higher to token

4:14:47generation efficiency and this was like

4:14:49completely done by 5.6 soul on itself,

4:14:52right? So what that means is that like

4:14:55inference is becoming much cheaper and

4:14:57that means that like if on if if for a

4:14:59company like OpenAI if you're having 20%

4:15:02low lower serving cost for a frontier

4:15:04model that could be a billions of

4:15:06dollars of improvement in price and you

4:15:08could you could do that as well for your

4:15:09own models. Um another interesting thing

4:15:12is that like when people have kernel

4:15:14writing uh competitions so core auto is

4:15:16a company which has a kernel writing

4:15:18competitions and you'll notice that like

4:15:20they they mentioned that like the person

4:15:23who won the kernel writing competition

4:15:25the explic said that I have never

4:15:27handwritten a GPU operator it's all ALM

4:15:29and agent driven so notice that like how

4:15:32like the best uh inference improvements

4:15:35kernel improvements are at this point

4:15:36completely exclusively done by a swarm

4:15:39of agents or generally like agents doing

4:15:41some magic to you know magically improve

4:15:43like the kernel kernel code and have

4:15:45faster inference cheaper inference. Um

4:15:48yeah so basically like the improvement

4:15:51surface that you think of is not just

4:15:54like the harness improvement it's

4:15:55literally the whole surface stack right

4:15:57so the whole stack from like evaluation

4:16:00to harness improvements to training data

4:16:02to post training to world interaction to

4:16:04inference like literally the entire pip

4:16:06part of the pipeline like all the way

4:16:08from like electricity going through your

4:16:11machine to like making a network request

4:16:14going through the model models doing

4:16:16inference um making a response and

4:16:18getting back to you like every single

4:16:19part of the pipeline we can optimize it.

4:16:21So the whole RSI loop has like um

4:16:24there's a couple different parts so it's

4:16:25like pre-training um uh neatron examples

4:16:28of like you know AI curated uh

4:16:30pre-training data that was good um

4:16:32there's like post- training examples

4:16:33which are you know where there's like

4:16:36constitutional AI examples of like post-

4:16:38training uh improvements then inference

4:16:40I gave you the example of GP 5.6 So for

4:16:43eval generation, I give you our own

4:16:46example of how we improved ourselves on

4:16:47our own eval. Um, in general, what you

4:16:50want to think of is that like anytime a

4:16:52model is trying to do something and it's

4:16:54like not able to do it or not able to

4:16:56accomplish it. So for example, like

4:16:57imagine you're having a long

4:16:58conversation where you ask the model

4:17:00something obvious. So like it's like or

4:17:03something that you think is not

4:17:05necessarily obvious, but it's like it's

4:17:07hard, but you would assume that the

4:17:08model could potentially do it. So to

4:17:10give an example, let's say you want to

4:17:12make add a new column to a database and

4:17:15then fill that column, right? And then

4:17:17put a cache in front of it. So what

4:17:19happens is it does the whole thing, but

4:17:21for some reason the way it pushes the

4:17:23cache in front of the whole uh database

4:17:24like it kind of doesn't do it right. So

4:17:26at that point you're like, "Oh no, set

4:17:28this cache differently. Set this as the

4:17:30TTL." And then you give like a

4:17:31correction signal and then that kind of

4:17:33like fixes your problem and you kind of

4:17:35go about your day.

4:17:36The way I want to think of it is that I

4:17:38want to think of like every single time

4:17:40like a human correction to the digital

4:17:42landscape is a part of a delta that

4:17:47needs to be covered. And what that means

4:17:49is that like every single time you're

4:17:51finding like the edge of model

4:17:52capabilities of like how far it can push

4:17:54for infants, how much it can push for

4:17:56this, how much it can push for, you

4:17:58know, solving your problems. You want to

4:18:00find that delta and we want to like push

4:18:02that frontier a little bit further. a

4:18:04little bit further and that's what post

4:18:06training does, right? So post training

4:18:08is is this is in that post training we

4:18:11find these like really interesting and

4:18:13really hard long horizon problems for uh

4:18:15for coding agents or for other kinds of

4:18:17problems and then just like slowly hill

4:18:18climb the models on on those kind of

4:18:21problems. So anything that has a

4:18:22verifiable loop can be optimized but is

4:18:26this is not just limited to software

4:18:29like literally like the physical world

4:18:30itself could have a verifiable loop. So

4:18:33to give an example like let me let me

4:18:35give you a very funny kind of bizarre

4:18:37example where like it's like you you

4:18:40look at this and uh you um

4:18:44um look at this uh like robot and it's

4:18:46it's typing something and it's like

4:18:49really really like interesting what the

4:18:51robot is typing.

4:19:01So notice how like notice how the robot

4:19:03is itself typing like how it wants to

4:19:05like solve robotics and imagine if the

4:19:08robot itself could like interpret the

4:19:10results of of like the answers and just

4:19:13like keep improving then that means that

4:19:15like this is you know basically like an

4:19:17ASI robot. Another example that I could

4:19:19give you for my uh personal life is that

4:19:22like um I have this um chip that has the

4:19:25ability to see and has a speaker and has

4:19:28a microphone inside of it. So I was

4:19:29trying to set up this uh with my machine

4:19:32and it was um a gift that I had for my

4:19:36girlfriend. And uh what I found very

4:19:38interesting about this is as I was

4:19:40trying to set it up, it was very slow.

4:19:42It was slow to respond. It had higher

4:19:43latency. It had an agent in and of

4:19:45itself. Right? So what I did was I

4:19:47connected the step to my um computer and

4:19:51I let um I let um putting agents run on

4:19:55this hardware and basically run on this

4:19:58hardware automatically make tests to it.

4:20:01So like my computer would talk to my

4:20:04hardware give it some tests like hey do

4:20:06this do that and just like what do you

4:20:08see right now? So then it would notice

4:20:11the end to end latency of like the

4:20:12inference response of like how fast uh

4:20:15my hardware is responding and then um

4:20:18what I found is that like it was able to

4:20:20do the breakdown of like every single

4:20:22case and scenario where there was like

4:20:24high latency um slower response or or

4:20:28just the algorithm wasn't right and I

4:20:30could then set up a recursive loop where

4:20:33both the firmware could be updated the

4:20:36um the tool calls the inference model

4:20:38used um the speech to text model used

4:20:41all of those could be improved

4:20:42automatically with like an open router

4:20:44key which I'd given to a coding agent

4:20:46and then overnight with self-running

4:20:48experiments with the hardware uh my um

4:20:51this tiny little piece of beautiful

4:20:52hardware that I have was like deeply

4:20:54optimized so this also like even

4:20:57hardware could be a part of the RSI loop

4:20:59so what happens when you connect these

4:21:00loops right that's the next question

4:21:01like you could have cheaper inference

4:21:03you could have like more experiments

4:21:05budget you have like you know better

4:21:07data and better models. So, is it

4:21:09singularity? It's a very question to ask

4:21:12and I think that the part of the parts

4:21:13of the loop are already weird, right?

4:21:14Like we're we we are really we really

4:21:17are very close to parts of the RSI loop,

4:21:20right? And what do we do like what do we

4:21:22do about this and it's like how do we

4:21:24think of that in terms of like our

4:21:26career and how do you think of that in

4:21:27terms of you know our lives and what do

4:21:29we want to do next? So here's my advice

4:21:31to everyone who is watching this like

4:21:33please please please please move closer

4:21:35to the RS group right so it's like right

4:21:38now like people are working on all these

4:21:39like different kinds of software which

4:21:41is like you know a agents for you know

4:21:43construction agents for this agents for

4:21:44that and it's like I think that you're

4:21:47you know what the way I see those things

4:21:50is that like I think of those as like

4:21:51you're missing the forest for the for

4:21:52the trees you really are and it's like

4:21:55you have this like very interesting

4:21:58godlike technology that can change parts

4:22:00of humanity. And then we're not

4:22:02ambitious enough to pursue the best

4:22:04problems. We really are not. And we're

4:22:06just like I think that it's not just

4:22:08like the problem isn't just that we're

4:22:10not ambitious and we're also like what

4:22:12we're pursuing. A lot of those things

4:22:14are just like one model release away

4:22:16from like being completely like wiped

4:22:18out, right? So like it's also like it's

4:22:20also like it may not be that

4:22:22intellectually simulating because like

4:22:23all you're essentially all we're

4:22:25essentially doing is like sitting all

4:22:26day. It's like hey cloud do this for me.

4:22:27Hey cloud do this for me. And I don't

4:22:28know if that's like I don't know if

4:22:29that's like the most intellectual

4:22:31stimulating or like or like um

4:22:33thoughtful use of a person's time like

4:22:35maybe maybe you know we're supposed to

4:22:37be solving harder and more uh more

4:22:39frontier problems and I think that like

4:22:40it's like the ceiling is like our

4:22:42ambition. So instead of like working on

4:22:44these kind of things my suggestion to

4:22:46people is to like you know do things

4:22:48that of course like goes without saying

4:22:50but you should do things you believe in

4:22:52and you know I'm just a guy on the

4:22:53internet but do things that help you

4:22:55move closer to the RSI loop right? So

4:22:58like anything that just like the end to

4:22:59end pipeline of like hey this is where

4:23:03like the models are lacking like it's

4:23:05like whether that's inference whether

4:23:06that's post training whether that's

4:23:07literally like physical interaction

4:23:09whether that could be you know send

4:23:12sending end to-end pipelines of truly

4:23:13any kind you know whether that's

4:23:15hardware problems whether that's um

4:23:17networking problems whether that's um

4:23:19building a sense community things that

4:23:20just you know um things that just like

4:23:23help um you as an engineer get closer to

4:23:27the benefit benefits of the improvements

4:23:29um in uh in uh um in in the model. So

4:23:33this is a very interesting slide from

4:23:35Sasha topic. So um um they mentioned

4:23:39that like you want to have like this

4:23:43you're in this space where it's like you

4:23:44know you only you only have like one to

4:23:46five years left to contribute to luxury

4:23:47to the world. What do you do? Right? So

4:23:49it's like um they they gave great

4:23:51suggestions were like prefer ambitious

4:23:53targeted and past collaborations right

4:23:55and you know in general like the time

4:23:58scale like all the timelines that you

4:23:59give to yourself like they should be

4:24:00just like completely squeezed out should

4:24:02be completely squeezed out because it's

4:24:03like a lot of times and you also have to

4:24:05think through that like if if the model

4:24:07if the problem that you're working on is

4:24:08too ambitious like it's like is it

4:24:10probably is it probably better to wait

4:24:12for just like the next next frontier

4:24:14model to come out or not like that's

4:24:16that's something you also have to weigh

4:24:17through Right. And uh you you ideally

4:24:21want to work on problems that benefit

4:24:23from scale and you know you also at the

4:24:25same time want to avoid problems that

4:24:26can be solved by sale alone because then

4:24:28you know if even if you like it's like

4:24:29if you're working on a problem and then

4:24:31the next model release comes out it

4:24:32solves the problem. So that's not that's

4:24:33probably not a good thing to do. So it's

4:24:35very like you have to like think

4:24:36philosophically of like what is the

4:24:39endeavor that benefits uh greatly from

4:24:41auto improvements but like isn't also

4:24:43completely wiped out from it. And then

4:24:45ideally like it's like if the if the

4:24:47endeavor could also uh be a part of the

4:24:50the like the improvement pipeline uh

4:24:53that could be amazing. So you see a lot

4:24:55of companies raising on say um making

4:24:58data pipelines and there's a lot of

4:25:00money there. There's a a lot of

4:25:02companies working on physical hardware

4:25:03and stuff. So I think if you're a

4:25:05software engineer watching these, it's

4:25:06like I think that if you're if you feel

4:25:08like you're like here the delta of your

4:25:11intellect and ability in your

4:25:13day-to-day, it's like not contributing

4:25:15as in like you have the intellect, you

4:25:17can make a difference. But like if cloud

4:25:19is just like oneshotting all those

4:25:21problems like in your day-to-day and

4:25:22you're just like kind of like becoming a

4:25:23me proxy, it's worth to think through.

4:25:25Take a step back and think through. like

4:25:27okay where are the problems where where

4:25:29you're actually truly uh contributing

4:25:31and that they will stand the test of

4:25:32time and then that kind of reflection

4:25:34and realignment right now would I think

4:25:36like have like very interesting

4:25:39consequences for you um in the long term

4:25:42um so yeah so thank you so much you know

4:25:44thank you so much for your time like I

4:25:46think that uh I think that if you're

4:25:48watching this and if you've gone this

4:25:49far I think that you're definitely

4:25:51contemplating like one of the most

4:25:53interesting improvements in the course

4:25:55of humanity And uh you know there's many

4:25:58philosophical arguments here. Some

4:25:59people might say hey you know I I think

4:26:02I'm maybe I'm off on the timelines or

4:26:04maybe maybe I'm I'm just being too

4:26:06pessimistic or maybe I'm being too

4:26:07optimistic. But I would love to hear all

4:26:10those thoughts. So this is you know my

4:26:11QR code. Please feel free to scan it and

4:26:13let me know all your thoughts. Uh thank

4:26:15you so much. Very kind of you to give me

4:26:16your time.
