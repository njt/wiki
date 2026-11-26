---
url: https://ai.engineer/code/2026/talks/philipp-schmid
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

2:21:16and harness and the less code we need.

2:21:18And if we look at um what we have been

2:21:21doing at Google deepmind for the last 6

2:21:22months. So at May we had Google IO IO

2:21:25and today it's uh November 11 and we

2:21:29have been shipping relentlessly.

2:21:30Basically just earlier today we

2:21:32announced our first decision model with

2:21:35Gemini decision one. Yesterday we made

2:21:38Gemini 4 carbon available and then if we

2:21:40just look at the last six months we

2:21:41basically ship now models every few

2:21:43weeks. Um as already said Gemini Carbon

2:21:46is now available for you. So if you are

2:21:48a Google AI ultra subscriber, you can

2:21:50use it in anti-gravity, you can use it

2:21:51in the Gemini app, you can use it in AI

2:21:53studio. Uh carbon is number one on the

2:21:56cursor bench uh eval and also number one

2:21:59on the perlexity benchmark and

2:22:01internally at Google. We have been using

2:22:03it already a lot and much more exciting

2:22:06is that I'm happy to share that we

2:22:08launched the Gemini decisions API

2:22:10roughly one hour ago with our new Gemini

2:22:13decision one model. So it's a similar

2:22:15model to Chef. You can provide all of

2:22:18the existing Gemini inputs, meaning it's

2:22:20multimodal. So you can provide text,

2:22:22audio, images, video, and also PDFs

2:22:25natively, and then provide up to 1,000

2:22:27questions in a single request. The

2:22:29average end to-end latency, including

2:22:31all of the network overhead, is between

2:22:33150 to 250 milliseconds, and it costs 10

2:22:37cents per 1 million token. So how do

2:22:40models get good at work? And that's a

2:22:43rough overview on how post- training

2:22:46works. So we have a task. We have our

2:22:48model. The model has a certain set of

2:22:50tools and then we have some kind of

2:22:52compute environment. And at the end we

2:22:54have some verifier. And the model tries

2:22:55to basically solve that task to use

2:22:57different tools. And then based on the

2:22:59score from the verifier, we update the

2:23:01model. But no one is telling the model

2:23:03which tools to use. And basically it

2:23:06only receives that reward feedback. So

2:23:10if we look at two examples on how a

2:23:12model can do a certain tasks with tools,

2:23:14we have the path A where we have

2:23:16dedicated tools for file system control.

2:23:19So you all have probably used uh coding

2:23:21agents and you have seen a read uh file

2:23:23call or an edit file call. So that's one

2:23:26way of doing it and the other way of

2:23:28doing it would be just using a single

2:23:30inline bash script for all of the

2:23:31changes. So on the left side we roughly

2:23:34need around 10 turns to make the same

2:23:36changes to a coding um repository

2:23:39compared to the two ch uh turns on the

2:23:41right side just using bash and it also

2:23:43reflects in the amount of tokens we

2:23:46need. So the left path takes around 10x

2:23:49more tokens than the life uh the right

2:23:51path and it's doing the exact same thing

2:23:54uh at the end for the model to achieve

2:23:56that task. So in my experiment every

2:23:59roughly 6 months I try to build a new

2:24:02coding agent harness from scratch and I

2:24:04always ask myself okay how much of the

2:24:06harness do I really need to achieve the

2:24:08same tasks and what is overhead to the

2:24:11model and we no longer need. So if we

2:24:13look at the coding agent harness which

2:24:16um from 2025 we had a roughly around 10

2:24:20to 15 to 20 tools. We had dedicated

2:24:22tools for um file editing. We had

2:24:26dedicated tools for planning, for to-do

2:24:28lists and also we had a lot of dedicated

2:24:31rules in our system prompt which told

2:24:33the model what to do and what not to do

2:24:36and that worked of course but since then

2:24:38we made a lot of changes. So in my

2:24:40experiments uh first one was done

2:24:42roughly now two years ago uh in February

2:24:46where we did not really had agents right

2:24:47we had workflows we basically had

2:24:49predefined steps rock applications back

2:24:51then were very popular we had a planner

2:24:53we had search we reflected on whatever

2:24:55was found and then we generated an

2:24:57answer then we started to see our first

2:24:59agents we had many many different tools

2:25:01for all sorts of fine cular actions um

2:25:04and then came the coding agents where we

2:25:06had those roughly 10 to 20 tools

2:25:09And now in my last experiment I tried to

2:25:11see okay what's going to happen if I

2:25:13provide only one tool to my agent and

2:25:16the outcome is was pretty exciting to

2:25:19me. So this one tool agent works the

2:25:21same way as every other agent except

2:25:23that it only has one tool which is bash

2:25:26and the model can use that tool for any

2:25:29sorts of task and the bash tool is very

2:25:31simple. So it runs in a se a se separate

2:25:33clean subshell and it supports the

2:25:36command argument. I also added a

2:25:38description so that it's nice and easier

2:25:40to display to the user what the model is

2:25:42actually doing and then like some async

2:25:43parameters. So basically when the model

2:25:45runs a bash command uh the parameter

2:25:47tells the model okay if it is uh the

2:25:50command longer running than this um

2:25:52milliseconds it's being um that the tool

2:25:55call resolves and the model receives uh

2:25:57a local file handle where it can look up

2:26:00later okay if the process is still

2:26:01running that's very interesting if you

2:26:03do like all sorts of like uh web

2:26:05development for example and the single

2:26:08batch tool actually works for all of the

2:26:09things we are using Asian so we can have

2:26:11longunning work where we can start on

2:26:13the foreground and then automatically

2:26:15move it to the background. The model

2:26:17receives some file handle and since it

2:26:20has bash available, if we store it to a

2:26:22file system, it can read the logs which

2:26:24are being streamed. If the agent itself

2:26:26is a CLI, it can call itself using bash.

2:26:30So there's practically no need to create

2:26:33sub aent tools for it because it can

2:26:34invoke itself. And then of course we

2:26:36don't need dedicated planning or memory

2:26:39tools because we have our file system

2:26:41available. We can provide some context

2:26:43to the model. So the model can keep

2:26:44track of its own work with the file

2:26:46system and using the bash tools. In

2:26:49addition of course to our tools we have

2:26:51system prompts and similar to tools uh

2:26:54system prompts evolved over time. Um

2:26:57every dot here is roughly around 100

2:26:59tokens and you can see that coding

2:27:01agents mid2025

2:27:03had a lot of context we needed to send

2:27:06in every request. Of course, this also

2:27:09improved and similar story here is like,

2:27:11okay, what do I really need to provide

2:27:13to my agent to make it successful and

2:27:16work and what of all of the system

2:27:18information and details we had in the

2:27:20past are now basically part of the

2:27:22model. So, how much of the harness helps

2:27:26and how much is in the way and to find

2:27:29that out I was running a lot of

2:27:31different evaluations of a lot of

2:27:32different agent harnesses with a lot of

2:27:34different tools. And one example I like

2:27:36to show here which was done with Gemini

2:27:384 carbon from an eval task from deep sv

2:27:42is like okay what happens if I provide

2:27:44my existing coding agent tools with read

2:27:47write search crap and my bash only tool

2:27:51and I think you can pretty clearly see

2:27:53that the top line is taking a lot of

2:27:55more tokens a lot of more turns and a

2:27:58lot of more times to achieve the same

2:28:00task and what stood out really well is

2:28:04that Okay, reading is done in parallel

2:28:06in the beginning, but as soon as we

2:28:08start going into making code changes, we

2:28:10see that same behavior that the model is

2:28:13making turn by turn a single edit, which

2:28:16of course consumes a lot of time and a

2:28:18lot of tokens. So how does this look at

2:28:23a protoscope? So I have my own eval set

2:28:26of around 350 tasks for coding knowledge

2:28:29work uh and terminal commands all run

2:28:33with Gemini 4 argan here and four

2:28:36different agent harnesses. One is the

2:28:38bash only harness and then okay with

2:28:40bash we still have that issue on how can

2:28:43I read images right if I'm I cannot cut

2:28:46screenshot.png it would return a binary

2:28:49and it will fill my context and the

2:28:51model cannot see the image. I added uh a

2:28:53view media tool which basically returns

2:28:55the image in a way the model can see.

2:28:58Then we have uh two different agent

2:29:00harnesses with structure tools uh for

2:29:02file system operations. And if we just

2:29:05look at the score column, all of them

2:29:08look roughly identical, right? So if we

2:29:11would just evaluate our model in ancient

2:29:14harness looking at the score, it would

2:29:18basically not really matter which one of

2:29:20those we would pick. Of course, there's

2:29:21a slight variance in the scores, but I

2:29:23only ran uh each task one time. So,

2:29:27might be just some noise or maybe

2:29:29something else. So, you would probably

2:29:30need to run a few more samples per per

2:29:32harness, but the score looks roughly the

2:29:35same. So, how would we now know which

2:29:38harness should I choose? If we just look

2:29:41all at all of the evol being shown, it's

2:29:44always mostly about score. But if we

2:29:46look at the token column or even more at

2:29:49a dollar per task, there's a picture

2:29:51which is changing. So the token per task

2:29:54is getting lower. Interestingly, if we

2:29:56just use bash and then also the cash hit

2:29:59rate increases. So if we just compare

2:30:01like the bash media as the the baseline,

2:30:04the tasks are almost 2x cheaper compared

2:30:07to more complex more tools agent

2:30:11harnesses. And we would not see it if we

2:30:14just look at the score. And to show you

2:30:18a few examples, I prepared some demos.

2:30:20So here I'm um using the PI harness,

2:30:23which makes it super simple to basically

2:30:25define what tools you want to use. I

2:30:28have the the same prompt for 3.5 flash,

2:30:31which we released roughly 6 months ago,

2:30:34uh for all tools. So like whatever pi

2:30:36ships out of the box. Then we have it

2:30:38for uh bash only and then we have it for

2:30:41carbon and uh sorry for carbon with the

2:30:45default tools and then with with um bash

2:30:47only. This runs now since we only have

2:30:50roughly around 20 minutes. I recorded

2:30:54all of that and clicked that very nice

2:30:56accelerate button here. And what we can

2:30:58see on the right side is that the same

2:31:01paradigma basically applies. when models

2:31:04get better, they roughly need less

2:31:07tools. So with 3.5 flash, we see with

2:31:10the default tools uh compared to bash

2:31:12only, we had the same amount of model

2:31:14calls. So we had the same amount of tool

2:31:15calls and we had roughly the same amount

2:31:17of tokens. And if we look at the traces,

2:31:20we can see for bash only, we have some

2:31:23cut commands to read different files. We

2:31:25have some inline scripts to make uh

2:31:27changes and then like some git changes.

2:31:30If we look at the default tools, we can

2:31:32see roughly the same. We had uh read

2:31:34file commands. We used the edit tool and

2:31:36then had some git commands to make those

2:31:38changes. If we now look at carbon with

2:31:42the default tools, it already used way

2:31:44less uh tool calls and model calls. So

2:31:48it it's basically doing work in parallel

2:31:51but also trying to reduce it. So we had

2:31:52for example the still same for write

2:31:56read calls and also the edit calls. But

2:31:58what starts to get interesting if we are

2:32:00going to look into bash and in bash we

2:32:03had even less tool calls and model calls

2:32:06and the interesting part here is that

2:32:08the model writes a single Python script

2:32:10to make all of that changes in one go

2:32:13and no longer needs to call multiple

2:32:15tools. So if we go further what does it

2:32:19means in terms for for carbon here and

2:32:21how bash pays off is that uh the better

2:32:24the model gets or the more reasoning and

2:32:27like capability it gets bash helps the

2:32:29model to be able to chain more commands

2:32:32in the single call for carbon it was

2:32:35roughly 2.2 to 3.7 uh commands per bash

2:32:39call and for flash it was only 1.3 to

2:32:411.8 eight. So we are seeing up to 3x

2:32:45less tokens uh by carbon to achieve the

2:32:50same task compared to having the same

2:32:52tools we need to use. So two more

2:32:57examples to look at it as well is like

2:32:59we have on the left side carbon with the

2:33:01default tools. So we see our coding

2:33:03agent tools with write with edit and we

2:33:06see okay it does some parallel work in

2:33:09turn two some parallel work in turn

2:33:11three and four and then we have all of

2:33:13the changes and on the right side where

2:33:15it just has the bash tool it uses Python

2:33:17to write an inline script to make all of

2:33:20that changes at the same time with much

2:33:23less token and the same counts for if we

2:33:26compare Gemini 3.5 flesh to Gemini 4

2:33:29carbon flesh with bash only is not

2:33:32chaining those commands. So here it it's

2:33:35not really it doesn't matter if I'm

2:33:36having a a retool a write tool edit tool

2:33:39and a bash tool or if I have just a bash

2:33:41tool because 3.5 flesh used to use bash

2:33:44for single turn command so it had a

2:33:48single turn for reading it had a single

2:33:50turn for writing but if we look at

2:33:52carbon which in here in this example is

2:33:54the exact same task it needs less weight

2:33:57turns so the reading is not needed to

2:33:59read all of the different files and uh

2:34:01completely it just uses rip trap and

2:34:03like some intelligent searching to find

2:34:05where we need to make changes and then

2:34:07it writes a single Python inline bash

2:34:09command to make all of those edits in

2:34:11one go saving us a lot of tokens. So

2:34:15what does it bring us? Models will keep

2:34:17eating the harness and I'm pretty sure

2:34:20that 2027 will also be a lot more about

2:34:23how many tokens do we need to solve a

2:34:25task. Even for the few uh improvements

2:34:28we made between Argan and Carbon is the

2:34:30the better model needs less tokens per

2:34:32task which of course is very exciting

2:34:34for all of us because then we can use it

2:34:36more and also it matters a lot what the

2:34:39harness provides and we will might not

2:34:41see it if we just look at the accuracy.

2:34:44It's also very important that we start

2:34:46measuring the token efficiency because

2:34:49harnesses can look the same on accuracy

2:34:51because our models are now very good. So

2:34:53if your harness is more complex to use

2:34:55the model will work around it will find

2:34:57solutions around it but needs more turns

2:35:00and more tokens to achieve the same

2:35:02results and then also what comes

2:35:04afterwards. So I will try to do the same

2:35:07experiment 6 months from now and

2:35:09currently the trend we see slowly um

2:35:13rising at least on like social media and

2:35:16like the industry is that we might go

2:35:18back to some code mode tool that we

2:35:21instead of writing a bash commands that

2:35:23we write TypeScript or Python directly

2:35:26that MCP will come back. I'm a big fan

2:35:29of MCP and like they are making great

2:35:31changes to make it more uh easy easily

2:35:35usable especially uh interesting for all

2:35:38of that code mode work is uh MCP servers

2:35:40now return allow you to return

2:35:42structured output so the model can write

2:35:44better scripts to take more actions in

2:35:46the single turn and it's also allows us

2:35:48to do some better proxy around the

2:35:50security because bash calls are very

2:35:52hard to understand but if I have some

2:35:55abstractive layer in typescript I can uh

2:35:58watch all of the commands being um

2:36:00executed and make sure those are secure

2:36:03and for you build to delete your code

2:36:07before the model does. give your model

2:36:09uh to a computer, not a toolbox. And

2:36:12when a new model ships, I definitely

2:36:14recommend you um trying to think a

2:36:17little bit outside of the box, be more

2:36:19ambitious, try to really see, okay, what

2:36:21if I remove my complete harness and just

2:36:23give it one tool and then also please

2:36:25scan the QR code. You will get free

2:36:28credits. Uh we will send them out after

2:36:30the AI engineering event to try out

2:36:32Carbon and also to try out the new

2:36:34Gemini decisions API. So if you have

2:36:37some time today, I really can recommend

2:36:39checking out the new decisions model

2:36:41carbon and thank you for your time.
