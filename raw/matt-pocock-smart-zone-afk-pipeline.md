---
url: https://gist.github.com/a80e60a6ff03bd5a853c91b130d22bd0
date_fetched: 2026-07-05
backfilled: true
---

Yeah. We good? Okay, folks, we're at capacity. Let's kick off. I don't want you waiting here for 25 more minutes before we some arbitrary deadline.

So welcome. My name's Matt, I'm a teacher and I suppose now I teach AI. We have a link up here if you've not already been to this, which is has the exercises for the stuff we're going to do today. This is going to be around two hours, so we might just sort of kick off two hours from now. Is that right, Mike?

Yeah, perfect. And the theory behind this talk, or at least the thesis under which I've been operating for the last kind of six months or so, is that we all think that AI is a new paradigm. Right? AI is obviously changing a lot of things. You guys are obviously interested in this and that's why you've come to this talk.

And I feel that when we talk about AI being a new paradigm, we forget that actually software engineering, fundamentals, the stuff that's really crucial to working with humans, also works super well with AI. And this is what my keynote is on tomorrow. Really, I'm going to sort of be fleshing that out a lot more. And in this workshop I'm hopefully going to be able to direct your attention to those things and hopefully show you that I'm right. But we'll see.

Can I get a quick heads up first? How many of you guys our coding have ever coded with AI? Raise your hand if you've ever coded with AI. Perfect. Okay, keep your hand raised.

Let's all share those armpits with the world. How many of you code every day with AI? Cool. Okay, keep your hand raised if you've ever been frustrated with AI. Okay, very good.

You can put your hands down. Thank you for that show of obedience. I really appreciate that. And we are also being live streamed to the Gielgud room as well. I've not.

Did we send someone up to the Gielgud Room to just check they're ok? Don't know. But I see you and there is a way that you can participate, which is we have a Q and A we're going to be doing. I have a sort of hatred of Q&As because they're not very democratic. Mostly the sort of most talkative people get to get to participate and share.

And so we're going to be going through this QA here. So why do we have to wait till 3:45? The room is packed, the doors are closed. 100% agree. And so if you want to ask a question.

We're going to be. I would like you to pile into this async and then we can vote on each other questions and hopefully get the best question surface for the entire room to enjoy. So I want to talk about first the kind of weird constraints that LLMs have. And those weird constraints are sort of what we have to base a lot of our work around. Now, there's a guy called Dex Horthy who runs a company called Human Layer, and he came up with this idea, which is that when you're working with LLMs, they have a smart zone and a dumb zone.

When you're first kind of like working with an LLM and it's like you just started a new conversation, you start from nothing. That's when the LLM is going to do its best work. Because in that situation, the attention relationships are the least strained. Every time you add a token to an LLM, it's kind of like you're adding a team to a football league. You think of the number of matches that get added.

Every time you add a team to a football league, you just go. It scales quadratically. And that's because you have attention relationships going from essentially each token to the other, but are positional and the sort of meaning of the individual token. And so this means that by around sort of 40% or around, I would say around 100k is kind of my new marker for this. Because it doesn't matter Whether you're using 1 million context window or 200K, it's always going to be about this.

It starts to just get dumber. So as you continually keep adding stuff to the same context window, it just gets dumber and dumber until it's making kind of stupid decisions. Raise your hand if that feels familiar to you. Yeah. Cool.

So this means that we kind of want to size our tasks in a way that sticks within the smart zone. We don't want the AI to bite off more than it can chew. And this goes back to old advice, like Martin Fowler in Refactoring, like the pragmatic programmer talks about this. Don't bite off more than you can chew. Keep your task small so that you, as a developer, a human developer, don't freak out and don't start acting and going into the dumb zone.

But how do you tackle big tasks? How do you take a large task like, I don't know, cloning a company or something or just doing something crazy? And how do you break it into small tasks so they all fit into the dumb zone? One way, of course you could do is, I mean, kind of what the AI companies maybe want you to do, or the natural way of doing it is just keep going and going and going. You end up in the dumb zone charging you tons of tokens per request.

You then compact back down. We'll talk about compacting properly in a minute. And you keep going, keep going, keep going. Compact back down, keep going, keep going, keep going. And I think that doesn't really work very well because the more sediment.

We'll talk about that in a minute. So the theory here is then, and, and this is what I was doing for a while is I would use these kind of multi phase plans where I would say, ok, we have this sort of number four thing here, this large, large task. Let's break it down into small sections so that we can then kind of chunk it up and do each little bit of work in the Smart zone. Raise your hand if you've ever used a multi phase plan before. Yep, really common practice, right?

This is kind of how we've been doing it. Certainly. This is how I was doing it up until December last year, really. And any developer worth their salt will look at this and go, this is a loop, right? This is a loop.

We just got phase one, phase two, phase three, phase four. Why don't we just have phase N, right? Phase N where we essentially just say, okay, we have let's say a plan operating in the background and then we just loop over the top of it and we go through until it's complete. And this is where. Raise your hand if you've heard of Ralph Wiggum as a software practice.

Okay, cool. Raise your hand if you've not heard of Ralph Wiggum as a software practice. Actually, that's more like it. Okay, so there's this idea called Ralph Wiggum, which is kind of sort of based on this, which is essentially all you need to do is sort of specify the end of the journey where you just say, okay, we create a prd, a product requirements document to say, okay, let's describe where we're going. And then we just say to the AI, just make a small change.

Make a small change. That gets us closer and closer to that. And RALPH works okay, but I prefer a little bit more structure. So that's kind of where we got to in terms of thinking about the Smart zone. And that's kind of where I want you to first start thinking about here.

Another weird constraint of LLMs is LLMs are kind of like the guy from Memento, right? They just continually Forget, they could just keep resetting back to the base state. Let me pull up this diagram. I really should use slides, but I just prefer just like randomly scrolling around an infinite tldraw canvas. Thank you, Steve.

So let's say another concept I want you to have is that every session with an LLM kind of goes through the same stages. You have. First of all, the system prompt here, this gray box here, is essentially the stuff that's always in your context. You want this to be as small as possible, because if you have a ton of stuff in here, if you have 250k tokens like I have seen people put in there, then that you're just going to go straight into the dumb zone without even being able to do anything. So you want this to be tiny.

You then go into a kind of exploratory phase. This blue is sort of where the coding agent is going out and exploring the code base. Then you go into implementation, and then you go into testing and making sure that it works, running your feedback loops and things like this. Raise your hand if that feels familiar based on what you've done. Yep.

Sort of like the main cornerstones of any session. And when you clear the context, you go right back to the system prompt, you go right back there. So you delete everything that's come before and raise your hand if you've heard of compacting as well. Yeah, okay. There are some people who've not heard of compacting, so let's just quickly show what that means.

For instance, I've just been having a little chat with my LLM. I want to make sure we sort of, you know, just cover the basics. So we're all sort of on the same wavelength here. I've just been having a chat with my LLM. I've been talking about a thing that I want to build.

How's the font size? Shall I bump it up, folks in the back? Bump, bump, bump, bump, bump, bump. Oh, I'm using Claude code for this session, but you don't need to use Claude code. In fact, it's often nice not to use Claud code.

So I've been having a chat with the LLM, just sort of planning out what I'm going to do next. It's asking me a bunch of questions and I can. I highly recommend you do this. There's this tiny little status line here that tells me how many tokens I'm using, the exact number of tokens I'm using. I have a article on my website, AI Hero, if you want to copy this.

This is. Oh, wow, that is. That shakes, doesn't it? This is essential information on every coding session because you need to know exactly how many tokens you're using so that you know how close you are to the dumb zone. Absolutely essential.

And so let's watch it. So I've got two options. I can either clear wrong and go back to nothing, or. Or I can compact. And when I compact, then it's going to squeeze all of that conversation, which admittedly isn't very much, into a much smaller space.

And this, in diagram terms, kind of looks like this, where you take all of the information from the session and you essentially create a history out of it, a written record of what happened. And devs love compacting for some reason, but I hate it. I much prefer my AI to behave like the guy from Memento, because this state is always the same, always the same. Every time you do it, you clear and you go back to the beginning. And so if you're able to do that and you're able to optimize for that, then you're in a great spot.

So that's kind of the two things I want you to think about with LLMs. The two constraints that we're working with. They have a smart zone and a dumb zone, and they're like the guy from Memento. So let's take a look at the first exercise. And while I'm doing this, the way I want this to work is I'm going to sort of show you how I'm going to be sort of walking through it up here.

And I want you folks to be kind of like tapping away and doing things as well. So that was just a little lecture bit. Let's now actually get into some coding. For anyone who arrived late or anyone in the Gilguid room, go to this link, this link up here to see the exercises and clone the repo. You absolutely do not have to.

You can just watch me do it if you fancy it. But let's go there myself and let's see what exercises await us. So essentially, I've built a. This is from my course. This is a course management platform, essentially a kind of CMS for instructors, for students.

And this is what we're going to be building a feature in. So I'm going to take you from essentially the idea for the feature all the way up to building a PRD for the feature, all the way up to implementing the feature. And hopefully you can take inspiration from this process and use it in your own work. So let's kick Off. So we're going to start by using a skill which is very close to my heart.

It's the grill me skill. And this grill me skill is wonderfully small, wonderfully tiny, and it helps prevent one of, I think, the main issues when you're working with an AI, which is misalignment. The. The sort of silent idea that I'm talking against here, that I'm arguing against is the specs to code movement. Has anyone heard of the specs to code movement?

Raise your hand. It's not really a movement, I suppose it's just sort of people saying specs to code. What it is, is people say, ok, you can write a program or you want to build an app. The best way to build that app is to take some specifications, so to write some sort of, like, document and then turn that document into code. So just turn it into code.

How do you do that? You pass it to AI. If there's something wrong with the resulting code, you don't look at the code. You look back at the specs. You change the specs and you sort of just keep going like this.

This is kind of like vibe coding by another name, where you're essentially ignoring the code. You don't need to worry about the code. You just sort of keep editing the specs and eventually you just keep going. And I tried this, I really tried it, and it sucks. It doesn't work because you need to keep a handle on the code.

You need to understand what's in it. You need to shape it because the code is your battleground. And so this again is where we're going. Let's get some exercises. So what I'd like you to do is go to this page, the grill me skill.

And inside the repo here, we have a slack message from our pal. Where is it? It's in the root of the repo, and it's under where is it? Client brief md. It's a slack message from Sarah Chen.

For some reason, Claude always chooses Sarah Chen as the name. I don't know why it's saying that in cadence. Our course platform, our retention numbers are not great. Students sign up to a few lessons, then they drop off. I'd love to add some gamification to the platform.

And so when you're presented with an idea like this, you need to find some way of turning it into reality. Let's say Sarah Chen is your client. You're on a tight budget. You need to get this done fast. How do you go and do it?

Raise your hand if you would enter plan mode when you're doing this. Anyone a big user of plan mode? Yep. Let's actually shout out quickly any other ideas about what you would do with this? Or raise your hand if you.

What would be your first port of call? Yeah, sorry, what is the purpose and where our current statement. Yes, exactly. Let's imagine that Sarah Chen's gone on Holder. You have no idea.

Right. She's just posted this thing. You need to action it before you go. Well, my first protocol is I go for this particular skill. I'm going to clear my context, I'm going to get rid of you.

You don't need to be there. And I'm going to say I'm going to invoke a skill, which is the grill me skill. Let's quickly check. Raise your hands if you don't know what this is. Cool.

Oh, sorry, sorry. Let me be more specific. Raise your hands if you don't know what I'm doing here. When I do a forward slash and then type something, everyone kind of understand what that is. I'm invoking a skill.

I'm invoking the grill me skill. And what I'm going to do is I'm going to say grill me and I'm going to pass in the client brief. So now the LLM really has only a couple of things here. It just has the skill and it has the description of what I want to do. And this is virtually how I start every piece of work with AI.

And while it's exploring the code base, I'm just going to show you what the grill me skill does. So this is inside the repo, so you can check it out. It's extremely short. Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies one by one for each question.

Provide your recommended answer. Ask the questions one at a time, blah, blah, blah. What this does and what I noticed when I was working with AI, especially in plan mode, actually, is it would really eagerly try to produce a plan for me. It would say, okay, I think I've got enough. I'm just gonna poof.

Plan, plan. And what I found was that I was really trying to find the words for this, for what I wanted instead of that. And Frederick P. Brooks in the Design of Design, he has a great quote talking about the design concept. When you're working on something new with someone, when you're all trying to build something together, then there's this shared idea that's shared between all participants and that is the design concept.

And that's what I realized I needed with Claude. I needed to reach a shared understanding. I didn't need an asset, I didn't need a plan. I, I needed to be on the same wavelength as the AI, as my agent. And this is an extremely effective way of doing it.

So hopefully. There we go. Nice. It has done its exploration. First of all, it's invoked a sub agent which spent 93,7k tokens on Opus and it's asked me the first question.

Cool. We can see that even though the sub agent burned a ton of tokens, I haven't actually increase my token usage that much. Raise your hand if you don't know what subagents are. It's an important question. Everyone kind of clear what sub agents are?

Okay, I'll give a brief definition which is that this, this subagents thing here, this Explore subagent, it has essentially gone and called another LLM which has an isolated context window and then that LLM has reported a summary back. So a sub agent is kind of like a delegation. You're delegating a task to a sub agent. It's, it goes eagerly does all the thing, explores a ton of stuff and then just drip feeds the important stuff back up to the orchestrator agent, to the parents agent. So ok, so hopefully you guys have seen the same thing it's done on Explore.

And we now have our first question, Points economy. What actions earn points and how much? Ok, at this point you can ask it, by the way, questions to deepen your understanding of the repo. I obviously know this repo really well because I wrote it, but you might not know what's going on. So let's say my recommendation, keep it simple.

Two point sources to start. What's so nice about this is that not only does it give us a question that kind of aligns us here, we get a recommendation too. And often what I'll find is the AI's recommendations are really good and so I'll just say skip video watch events, they're noisy and gamble. I agree. Sarah's asked, well, keep lessons in the bread and butter.

Yeah, looks good, pal. Now what I usually do is I usually dictate to the AI. I'm usually actually chatting to the AI instead of typing here. But this is a relatively new laptop and I couldn't get my dictation software working on it because Windows is crap. So should points be retroactive?

There are existing lesson progress records we've completed at timestamps. This is a really Nasty question. Right? Should we actually go back and backfill all of the lesson progress events? This is a kind of question that you need to be aligned on if you're going to fulfill the feature properly.

This is not something I considered and Sarah Chen certainly didn't consider. Do I want it to be retroactive? Let's actually do a vote inside here. Should we go back and backfill all the records? Raise your hand if you think we should backfill all the records.

Raise your hand if you think we shouldn't backfill all the records. There are a lot of fence sitters in the room. I'm going to say, you know, this is the kind of discussion you're sort of having with the AI. You're getting further aligned. Yes.

I'm just going to go with this recommendation because I'm lazy. Notice too, how I'm able to keep in the loop here with AI. I'm not, you know, it's. It's pinging me these questions pretty quickly. I'm not having to go off and check Twitter or something.

Levels. What's the progression curve? Yeah, that looks about right. For instance. Yes.

Okay, so hopefully you should be able to go and kind of work through this with the AI and essentially try to reach an alignment. And this grill me skill, this can last a long time. I've had it ask me 40 questions. I've had it ask me 80 questions. Had some people it asks 100 questions to.

Literally you're sat there for an hour chatting to the AI, and what you end up with is essentially this conversation history that works really nicely and works really nicely as an asset of the design concept that you're creating. This can also function like this. You can have a meeting with someone who's maybe a domain expert. Maybe I have a meeting with Sarah. I feed that meeting transcript into, I don't know, Gemini meetings or whatever you guys are using.

You take that, you feed it into a grilling session and you grill through the assumptions that you didn't have. So this ends up being a really nice, kind of a really nice way of just taking inputs from the world and then just turning and validating them. So, okay, let's see. I really want to get to the end of this, but I also don't want to just like be sat here talking to the AI in front of you for a thousand days. So I'm just going to say, yes, let's see what happens.

So I tell you what, while you guys sort of have a little fiddle with this locally, let's start a little Q and A session now and let's see how's this going to work? Can we keep the door closed? I'll turn up the microphone. It's quite noisy. Let's see.

Mike, can we. Door closed? Oh, it has been closed. Markers aren't so beautiful. So what I'd like you to do, is there any air con?

Yeah, there is some air con. I think there is some air con. You guys aren't being lit here. I'm being fried alive here. So what I'd like to do is go on to the slido, which you can join here.

If you're not taking the exercise, go onto the slido, have a little fiddle and vote on some good questions. I'm just going to chat to the AI for a second until we reach a stopping point. So do streaks earn points? Streaks are standalone. Let's see what else it comes up with.

Where does Gamification UI live? Let's have it in the dashboard. I'm just going to scan these and blast through them, basically. So how are we doing with our slido? Okay.

Have I tried Spec Kit, Open Spec or Taskmaster instead of the Grill me skill, do I find them more verbose or a structured alternative? This is a great question. So there are a ton of different frameworks out there that allow you to sort of build up this planning process for you. I personally believe you. At this stage, when there's no clear winner, when there's no kind of like one true way, and when things are changing all the time, you need to own as much of your planning stack as you possibly can.

What I've noticed, and a lot of my students, is they tend to overuse a certain stack. They get into trouble and they. Because they don't own the stack and they don't have observability over the whole thing, they just go, this isn't working. This sucks. Whereas if.

If you have control over the whole thing, then at least you know how to fix it or potentially know how to fix it. So I'm even though I'm sort of giving you a stack, basically, I believe in inversion of control and you should be in control of the stack. So can I plus it zero please? Sorry, sorry. That was a lot of sort of mumbling feedback.

You have four options on the bottom of Claude and they want you to hit dismiss message. Thank you. I'm so sorry. Well, you didn't want to give Claude good feedback. What's wrong with you?

Ok, cool. Many of the questions asked by the Grill Me skill are not necessarily appropriate for a developer, rather a PO in larger teams who should use it in. Yeah. Raise your hand if you've ever done pair programming. Anyone ever done pair programming?

Right. Put your hands down and raise your hand again if you've ever done a pair programming session with an AI. Right. How did it go? Was it good?

You enjoy it? I think pair programming sessions with AI is a great idea because you've got a third person in the room who will relentlessly quiz you and ask you questions. If you don't know the answer, it should be you, the domain expert and the AI in the same room. If you have a question about implementation, it should be you, a fellow developer, and the AI in the same room. You can be sort of working through these questions in your team.

And I think actually we're going to look at implementation in a bit and we're going to see how you can make implementation so much faster. But I think the really crucial decisions, the ones you need humans for, you actually need a lot of humans. And it doesn't really matter how many humans are in there. You can actually throw a bunch like a kind of like mob programming with AI, essentially. What's my favorite metaprompting tool?

I think I kind of answered that there's no air con. Let's just live with it. How do I use the conversation as an asset after the grill me session? Well, we're going to get there. Ok, So I really want to.

I want to speed this up sort of artificially. Just this is the thing. So someone just said, okay, ralph, loop this. But this is crucial because I can't loop over this, right? I can't.

I think of there as being two types of tasks in the AI age where you have human in the loop. Tasks where a human needs to sit there and do it, which is this. We are the human in the loop with multiple humans in the loop. And there are AFK tasks. There are tasks where the human can be away from the keyboard and.

And it doesn't matter. Implementation, as we'll see, can be turned into an AFK task. But planning this alignment phase has to be human in the loop. Has to be. So I've got to do it, unfortunately, I don't know.

Give me a long list of all your recommendations. I'm running a workshop right now, so I artificially need you to pull more weight. So let's see what it does. Let's answer a couple more questions while it's doing its thing. What is my opinion on PMs or other non Dev roles, vibe coding tasks.

I'm going to return to this later. I think I'm going to leave this unanswered. A bit of mystery. I notice I'm not using the Ask User Questions UI for Grill me. Why?

There's a specific UI that you can bring up in Claude code, which I'll answer this. Just quickly ask me a question. Using the Ask User Question tool. And this UI is just sort of broken in Claude and I really hate it. You notice I'm using Claude, but I don't like Claude very much like you.

You really are free with this method to choose any system you like. And this is what the UI looks like. It's very pleasing when you first encounter it, but then you realize it is actually broken in a ton of different ways. All right, what did it come back with? Oh, blimey.

Oh, no. So while this is doing its thing, let me do some teaching in the meantime. The plan here is that we take our Grill Me skill and we need to essentially find some way of turning it into a destination. We need to go down to the. We essentially need to.

We're figuring out the shape of this. That's what we're doing. We're figuring out the shape of the tasks during the grilling session. And in order to turn it into a bunch of actionable actions for the AI, we essentially need to figure out the destination, we need to know where we're going, we need to know the shape of this entire thing. So I think of there as being two essential documents that we need.

We need a document that documents the destination. Oh, no, it's not bright enough. There we go. Still not bright enough. There we go.

We need something to document the destination and we need something to document the journey. In other words, we need something, a document that's going to figure out what this even looks like in all of its user stories and figure out a definition of done. And then we need to figure out what the split looks like. So that's where we're going to go to next. So once we finish with the grilling session.

Yeah, it looks great. Fantastic. I love answered 22 of its own questions. There you go. That's quite representative of what a grilling session looks like.

So at this point now, I have used 25k tokens and all of that, or loads of that stuff is gold. I want to keep that around. I've got 25k great tokens there. And what I want to do is kind of summarize it in some kind of destination document. So this is the next exercise where we're going to.

We're going to write a product requirements document. And the product requirements document or the PRD is essentially. That's its function, it's the destination document. And it sort of doesn't matter what shape it is. I've got a shape that I prefer and I quite like, but you can just choose your own shape or whatever your company uses.

And all we're really doing is too worried about that. All we're really doing is summarizing the design concept that we have so far. And the. So let's try this. So I'm going to initiate this.

I'm going to say zoom all the way to the bottom. All I'm going to do is just say write a prd. And we can take a look at that skill now. Write a prd. So this skill, it does a few things.

It first asks the user for a long, detailed description of the problem. You can use writerprd without grilling first, but I just like to grill first and then write the PRD afterwards. Then you can get it to explore the repo, which we've kind of already done. Then we get it to interview the user relentlessly. So have a kind of grilling session again.

And then we start putting together a PRD template. So this is available in the repo, if you want to check it out. And essentially this is what it looks like. We've got some problem statements, the problem the user is facing, the solution to the problem, and a set of user stories. And these user stories sort of define what this is.

You know, as you guys have probably seen, things like this, if you've been a developer at all, you know, there are. Cucumber is a language you can use to write these in, or we just sort of write them ourselves, essentially. Then we have a list of implementation decisions that were made and list of crucially testing decisions too. So I'm going to run this. Okay.

And so it's finished its thing. Ah, Windows. Let me close the thing. Thank you. I don't know why I bought a Windows laptop.

I think I just. I like the challenge. So the first thing that it's going to give me are a set of proposed modules it wants to modify. Now, there's a deep reason why I'm thinking about this. So this is.

At this stage, we have an idea. We have sort of specced out the idea. We've reached a sort of understanding of what we're trying to do. And then we need to start thinking about the code, because at this Point we need to. This is not specs to code.

This is not where we're ignoring the code. We actually keep the code in mind throughout the whole process. And the way I like to do this is I like to just sort of think about a set of proposed modules to modify. We're going to return to this. This idea of continually designing your system and keeping your system in mind.

So it's saying recommend test for the gamification service. The only deep module with meaningful logic. These modules look right. Yeah, looks good. And it's going to ping out a prd.

Now, for ease of setup, I've got it so that it creates a set of issues locally. So it's just going to create essentially a PRD inside this issues directory. But the way I usually do it, and you can check this out yourself is you can go to my. Essentially what I consider my work repo, which is GitHub.com mattpoc course video manager up here and in here. This is essentially a app that I create that I use all the time to record my videos and things like this.

I think I've recorded like I pulled down the sets. I think I've recorded like a thousand videos in here or something. Nuts. And you can see here that it's got 744 closed issues. And this is essentially all of the PRDs and all of the implementation issues that I've put into here.

So this is how I usually like to do it. So that's what I'm doing with the. There we go. Yeah, I'm just going to say yes and get that issue out. Let's see, it is inside here.

So we've got the problem statement. People sign up for courses. The solution, the user stories. 18 user stories looks nice. Some implementation decisions, level thresholds, etc.

This is enough information. We've kind of clarified where we're going and what we're doing. So that's what we do. We essentially have a grilling session and we've created an asset out of it. Now raise your hand should I be reviewing this document?

Raise your hand if you think I should be reviewing the documents. Yeah, I don't look at these. I don't look at these. The reason I don't look at these is because what am I testing at this point? When I read it?

What am I testing? What are the failure modes I'm trying to test for? I know that LLMs are great at summarization because they are. They're really good at summarization. I have reached the same Wavelength as the LLM.

Right. Using the Grill Me skill, we have a shared design concept. So if I have a shared design concept, all I'm doing is I'm just essentially checking the LLM's ability to summarize. So I don't tend to read these. Let's have.

Let's have a Q and A, because I can feel you guys are reaching for it. Then I think we might have like, I don't know, just a five minute comfort break just to rest my voice and so you can catch up with the exercises for a minute, if that's all right. So let's have a little Q and a sesh. If I don't like Claude code, which one do I actually like? Have you ever heard the phrase democracy is the worst way to run a country?

Apart from all the other ways. That's how I feel about Claude code. We've answered that one. What's your thoughts on developers needing to very deeply understand Typescript now that fix the ts, make no mistakes exist. I don't understand the phrasing of this, but I think I understand the meaning, which is that I believe that code is very important and this is kind of going to feed through the whole session and that bad code bases make bad agents.

If you have a garbage code base, you're going to get garbage out of the agent that's working in that code base. We'll talk more about that in a bit. And so I think understanding these tools very deeply, understanding code deeply, is going to make you a much, much better developer and get more out of AI. And that answers that question too. Sweet.

Get out of here. There you are. Now that we have 1 million tokens available, do we ever actually want to take advantage of that? I've noticed that the Dumb Zone has become less dumb lately. Okay, great question.

This goes back to our kind of initial idea on the Dumb Zone. I recorded my Claude code course using a 200k context window. And on the day that I launched the course, they announced the 1 million context window. My take on this is that what Claud Code did is they essentially just did this wee. They shipped a lot more Dumb Zone to you, essentially.

Now, this is good for tasks where you want to retrieve things from a large context window. If you want to pass five copies of War and Peace or something to it and you want to find out all the things that I can't remember a character from War and Peace. Why did I start with that? It's good for retrieval, it's less good for coding. So I consider that it is about 100k at the moment is the Smart Zone.

The Smart zone will get bigger and that will be a really nice improvement. So folks, we're going to take it like a five minute comfort break, if that's all right, just for my voice and so maybe you can have a little move around or something or grab a drink. I just noticed some sleepy eyes and I want to make sure that we're awake for the next bit, if that's all right. So we'll take five minutes and I'll see you back here then. All right, so we have our prd, which I'm not going to read our kind of destination document.

Let's quickly scan for any good questions before we zoom ahead and rediscovering the role of software engineer in today's world. Top three disciplines you recommend. Taekwondo is good, I've heard. I have no idea how to answer this question. Thank you for asking it though.

Top three disciplines recommend. Sorry. Plumbing. Plumbing is a good one. Yeah, yeah, yeah.

I don't know if that's a discipline. The plumbers I've hired are not usually very disciplined. Right. So, okay, we now have our destination. Okay, perfect.

So how do we actually get to our destination? How do we. We have a sort of vague prd. How do we spend. Split it so that we don't put things into the dumb zone.

In other words, we have our number four. How do we split it into this kind of multiphase plan? Well, probably what you would do at this point is you would say, okay, Claude, give me a multiphase plan that gets me to this destination. Right. That sort of makes sense.

This is what we've been doing before, but I have a sort of better way of doing it now, which is that I like creating a Kanban board out of this. Raise your hand if you don't know what a Kanban board is. Cool. Okay. A Kanban board is essentially just a set of tickets that you put on the wall that have blocking relationships to each other.

So we're going to see what it kind of looks like here. This is how we've worked as developers for a long time, really, since Agile came around. And what it does. We can see it here. It has proposed that we split this setup into five different tasks.

Here we have the first one, which is the schema and the gamification service. Yeah, that looks pretty good. This is blocked by nothing. And we can even see here that it's a. It's given it a type of afk.

Remember I talked about human in the loop and AFK earlier. This is an AFK task. This is something we can just pass off to an agent to do its thing. Streak tracking. Okay, that looks good.

Then wire points and streaks into lessons. Quiz completion. This is blocked by 1 and 2. Retroactive backfill. This is blocked only by 1.

And then this one here is blocked by all of the tasks. Cool. Hmm. Now, I consider this, you could say, why don't we just make this sort of generation of the issues. Why don't we just hand that over to the AI?

Why do I need to be involved here? Right, because it's given us quite a good selection of tools here. Why do I need to review this and sort of figure out what's next? Now, my take here is that this is really cheap to do, very quick to do, once I've done the pr and I can immediately see some issues here. There's a really, really important technique when you're kind of figuring out what the shape of this journey should look like.

And it sort of comes to this very classic idea, which comes from pragmatic programmer called tracer bullets or vertical slices. And tracer bullets really transform the way I think about actually getting AI to pick its own tasks. Systems have layers, right? There are layers in your system. These might be different deployable units.

You might have a database that lives somewhere. You might have an API that lives maybe close to the database, but in a separate bit, you. You might have a front end that lives somewhere totally different, like a cdn. Or within these deployable units, you might have different layers within those. In, for instance, the code base that we're working in, we have a ton of different services.

We have a quiz service, a team service, user service, coupon service, core service. And these services have dependencies on each other, so they're kind of like individual layers. Well, what I noticed is, is the AI loves to code horizontally, so it loves to code layer by layer. So in other words, in phase one, it will do all of the database stuff, all of the schema, all of the, you know, all the stuff related to that unit. Then it will go into phase two and do all the API stuff.

Then it will add the front end on top of that. Does. Can anyone tell me what's wrong with that picture? Why is that not a good thing to do? Raise your hand if you.

You have an answer. Yeah, exactly. You don't get feedback on your work until you've really started or completed phase three. So what you really need to do is until you get to phase three, you're not actually testing that all the layers work together. You haven't got an integrated system that you can test against.

And so instead you need to think about vertical layers. You need to think about thin slices of functionality that cross all of the layers that you need to. And this is a much better way to work, much better way for the AI to work too, because it means at the end of phase one or during phase one, it can get feedback on its entire flow. So what this means to me is inside the PID to issues skill. Up here I have got breaker PRD into independently grabbable issues.

Using vertical slices traceable, it's written as local markdown files. We first locate the PRD again, explore the code base. If this is a fresh session, we draft vertical slices. So we break the PRD into tracer bullet issues. A tracer bullet, by the way, is essentially when you're like an anti aircraft gunner, it's quite a violent idea actually.

And you're looking up in the sky and it's night. If you're just shooting normal bullets, you have no idea what you're firing at, right? You could just be, you know, you see the plane but you don't see where your bullets are going. Tracer bullets is they attach a tiny bit of phosphorescence or phosphor or something to make it glow as it goes. So this means that every sixth bullet or something, you actually see a line in the sky.

So you have feedback on where you're aiming. So this is what this is. The idea here is, is that we increase our level of feedback and we get near instant feedback on what we're building. Because without that, the AI is kind of coding blind until it reaches the later phases. We've got some vertical slice rules, we quiz the user and then we create the issue files.

So what I see here is that even though I've told it to do vertical slices, it's proposing to create the gamification service first on its own. That's just one slice there and that to me feels like a horizontal slice. What I want to see in the first vertical slice especially is I want to see the schema changes or some schema changes. I want to see some new service being created and I want a minimal representation of that on the front end. So I want it to go through the vertical slices, not just the horizontal.

Does that make sense? Okay, so I'm going to give the AI a rollicking bad boy. No, I'm not going to waste tokens just being just memeing. So the first slice is too horizontal. I'll just start with that and see if it picks it up.

So that makes sense as a concept and I think having that. What I really like about going back to those old books is that we are really trying to in this day and age, like get verbalize best software practices in English. And these books, 20 year old books have already done that and it's an absolute gold mine if you want to throw that into prompts. But even with that, it's not going to do a perfect job each time. So Award points for lesson completion visible on Dashboard.

Yes, that's a beautiful vertical slice because it's definitely a big chunk of stuff. It's doing a lot of stories there. But we're going to see something visible at the end and the AI will then just be able to add to that. You see why that's preferable to the first one. Cool.

Looks great. So we're getting closer now and anyone following at home as well. You're not at home, but you get the idea. Will hopefully see the same thing too and start developing the same instincts. Let's open up for questions just while I'm sort of creating these GitHub issues or not GitHub issues, local issues.

When will I stop using Windows? Never. What is your. Okay, we'll get to that later. How does AI decide when to stop grilling?

Because AI can ask incessantly. Can we have a smarter way to decide to stop pointing? Yeah, it does tend to really. Those grilling sessions can be super intense. And the thing about these skills is you can tune them if you want to.

If you feel like the AI is just absolutely hammering you, hammering you, hammering you, then you can just tell it to just pull back a little bit or get it to do stop points and that kind of thing. So if that's a failure mode that you run into a lot, then you just change the skill. Do I still use Be extremely concise. Sacrifice grammar for the sake of concision. There was a tip that I gave folks five months ago, which is to basically increase the readability of your plans.

So when you're using plan mode, then you can put it in your Claude MD and you can say, okay, yeah, approve that. Let's open up Claude md. Do I have a Claude md? Maybe I don't. I really don't use Claude MD very much.

I'm just gonna put a dummy inside here when. No, when talking to me. Sacrifice grammar for the sake of concision. And this prompt was really useful to me when I was reading the plans because it meant that the plans would come out and they would be very concise. Really nice, easy to read, often very concise.

But I've since dropped this idea in preference to a grilling session because what I noticed was it just. I didn't want to read the plans. I wanted to get on the same wavelength as the LLM. I wanted it to ask aggressive question questions to me. And when I stopped reading the plans, I stopped needing them to be concise.

So I think of the plans really in the destination document as the end state. And I don't need that end state to be concise. Hopefully that answers your question. What do I think will be the outcome of the Mexican standoff of future roles of PMs and other roles converging? I have no idea.

I'm not a pundit. I have no idea. Okay, so we should, after a couple of approvals, end up with a set of issues. Now these issues that we're creating, they're designed to be independently grabbable, which means that this Kanban board ends up looking kind of like this, where you have essentially a set of tickets with a whole load of independent relationships. So this one needs to be done before this one, this one needs to be done before this one and this one, let's say we've got another one over here.

This one needs to be done before this one. This means that you can start to parallelize. You can start to get agents working at the same time on these tasks because, yeah, this one needs to be done first. And then these two can be grabbed at the same time by independent agents. Raise your hand if you've done any kind of parallelization work with agents.

Okay, cool. So this allows you to turn those plans into optimally kind of like into directed acyclic graphs essentially, where you just are able to essentially have three phases here. Where you have phase one, let me grab my. That above this line here, you do this one. Then phase two, you do the two below it.

And then phase three, you do this third one and add it onto that. And when you think about there could be. This could. This is a relatively simple plan, but you could have many different plans operating all at once. It means that you can do really nice parallelization.

And we'll talk more about that in a bit. But that's why I prefer a Kanban board set up like this to a sequential plan, because a sequential plan can really only be picked up by one agent. So this. Where did it go over here? Yeah, this plan here, this is really only one loop, right?

Only one agent can work on these because we have numbered phases and they're not parallelizable. Does that make sense? Cool. So we've got our issues. Ah, come on, stop asking me for oh no, it's creating them on GitHub.

I really don't want that. Oh no, you fool. Create them in issues instead. No, that's not precise enough, you fool. Create them in local markdown files instead referencing the local version.

Sorry about this. So once we get to this point, we have a bunch of issues locally that we can start looping over and implementing. And it's at this point that the human leaves the loop. So so far, let me pull up a proper overview of this kind of flow that we're explaining exploring here. So far we have taken an idea, zoom this in a bit for the folks at the back and we've grilled ourselves about the idea.

We can skip over research and prototype, but we turn that into a prd, into a destination document. We've then turned that PRD into a Kanban board and all of those steps are human reviewed. And now the implementation stage, we step back and we let an agent work through that Kanban board, or multiple agents work through the Kanban board. Now what this means is that, yeah, we've spent a lot of time planning here, but it means that we've queued up a lot of work for the agent. We can think of this as kind of like the day shift and the night shift.

This is the day shift for the human. Right. Planning everything, getting all the, all the stuff ready. And then once we kick it over to the night shift, the AI can just work afk. But what does that look like?

Well, so I'm just gonna. Oh yeah, just allow it. It's perfect. So this looks like if we head to the next exercise, which is in fact the last exercise here, running your AFK agent. Now, I've called this ralph, really, because it is essentially a RALPH loop.

And this prompt here, I want to walk through this really closely. The first thing it's doing here is we're essentially going to run Claude and we're going to basically try to encourage it to work completely afk. I'll show you what the script for this looks like in a minute, but you say, okay, okay. Local issue files from issues are provided at the start of context. The way we do that is if you look inside, once sh here, inside the repo we have, it's essentially just a bash script where we grab all of the issues which are inside Markdown files and we cap them into a local variable.

So that issues variable contains all of the issues that are in our entire backlog. Then we grab the last five commits. I'll explain why in a minute. And then we grab the prompt and we just run claude code with permission mode, accept edits, and then just, essentially just pass it all of the information. This is what the implementer looks like.

So that's what a very, very simple version of this sort of loop looks like. And of course, this is not a loop, this is just running it once the loop is in the AFK version up here, which is a fair bit more complicated. And the crucial part here is we're running it in Docker Sandbox as well. So I. I don't want you to install Docker on your laptops because we're just going to be like, you need to download a special image and we're going to tank the conference WI Fi if we do that.

So I am going to demo this to you, but you won't need to run this yourself. But I'll talk through this in a minute. But essentially, this once loop here, We're just essentially running one version of the thing that we're going to loop again and again and again. So this is kind of like the human in the loop version. And this is essential.

Running this again and again is essential because you're going to see what the agent does and see how it ends up working. And any tuning that you need to add to the prompt, then you can do that, go into the prompt. So local issue files are being passed in. You're going to work on the AFK issues only if that makes sense. If all AFK tasks are complete, output this no more tasks thing, and then the next thing, pick the next task.

So what we're doing here is we're essentially running a backlog or curating a backlog that. That our AFK agent is going to pick up. That's the purpose of all of these setups. In the beginning, in this, all the way to this Kanban board here, we're just essentially creating a backlog of tasks for the night shift to pick up. And the night shift, this sort of RALPH prompt here, it's got its own idea about what a good task looks like.

So next pickup. I did talk about parallelization. I will show you this later. But this is essentially a sequential loop here. We're just going to run one coding agent at a time.

This is a good way to just sort of get your feet wet, essentially. So it's prioritizing critical bug fixes, development infrastructure, then tracer bullets, then polishing quick wins and refactors. And then we just have a very simple kind of instruction on how to complete the task. So we explore the repo use TDD to complete the task. I'll get to that later.

And we then run some feedback loops. So let's just try this and let's just see what happens. So good. It's created the issue files. We should be good to go.

I'm going to cancel out of this clear. And I'm going to run where is it Ralph once sh and you can feel free if you're following along to do the same thing. So we can see it's just running Claude inside here with the prompt and with all of the issues that have been passed in. And while it's doing its thing, you probably have some questions about this setup and about the decisions that I've made to essentially delegate all of my coding to AI. Right.

So let's do a quick Q and A while it's getting its feet under. Okay, I'm going to just remove those. How do you retain negative decisions, things that you decided against and rationales when persisting the results from the grill me session? Great question. That's a very simple answer which is that in the PRD write a PRD section, there is a stuff at the bottom, a section of the things that are out of scope.

So the things we're not going to tackle in this prd, which is very important for giving a definition of done. Feel free ping on the slido if you've got any more questions. What's my front end workflow? Okay, that's a great question. I'm going to answer that in a minute.

I think how to deal with agents producing more code that we can review. How to properly parallelize and use multiple agents in a separate way. Ok, there's two questions there. Raise your hand if you feel like you're doing more code review now than you used to. Yeah, definitely.

I don't think there's a way to avoid this. If we delegate all of our coding to agents, you notice that the implementation here is really the only AFK bit. We then also need to QA the work and code review the work. And if we are running these loops where it's essentially going to implement four issues in one, it it's hard to pair that with the dictum that you should keep pull requests small and self contained. Right.

Like small self contained pull requests means you're needing to do fewer Loops or shorter loops or something. Or maybe you do like a big stack of PRs, but that seems horrible as well. That's still just more separated code to review. I don't honestly know what the answer to this yet. I think we just need to be ready to be doing more code review, essentially, which is not fun.

That's not fun thing to say. That's not like. I don't know, I don't feel good saying that. But I do think it's probably the way things are going. It's a great question.

Can we grab a couple of questions from the room as well? Let's not. We won't do the mic, but raise your hand if you've got a question for me immediately. Yeah. So the approach looks very linear for from an idea to a QA code review.

Of course, the real world is a lot more messy, so you have all these ideas that are in parallel and nobody has the full picture. And while you're working on something, something else comes in. Like a bug. Yeah. How do you deal with the messiness?

How do you tighten the feedback loop? Great question. So the question was, this all looks great if you're a solo developer, but actually, how do you implement this in a team? How do you gather team feedback on this? And my answer to that is that if you have an idea up there, and essentially the sort of journey from the idea to the destination is something you need to figure out with the team.

Right. So all of this stuff up here, this is kind of like team stuff, you know what I mean? So if you have an idea and you do a grilling session on it and you have a question that you don't know how to answer, then you need to loop in your team as we described before. Then you might need to go, okay, we just need to build a prototype of this. We need to actually hash this out.

We need something that the domain experts can fiddle with. Oh, okay. We might need to integrate a third party library into this. We might need to do some research. We might need to actually kind of like ping this back and forth and find a third party service that we can get the most out of.

We might need to go back with the information that we gathered there to the idea phase. So all the way up to the sort of PRD and the journey, that's something you need to involve your team with. That's something where these assets are going to be shared and argued over and you're going to have requests for comments on them and that loop is going to just keep grinding and grinding until you figure out where you're going. Once you figure out where you're going, then you can start doing the Kan on board the implementation. But this is essentially super arguable and you'll be bouncing back and forth between the phases.

Does that make sense? Yeah. A PRD for your prototype. Say again? Sorry, Would you not want to have a PRD for your prototype?

The question was, do you want to go through this whole session just to sort of create a prototype? Do you not need a PRD for your prototype as well? Let's just quickly talk about prototypes for a second. There was a question about how do you make this work for front end. Like, how do you.

Because front end is like really sensitive to human eyes. You need human eyes looking at the front end all the time to make sure that it looks good. AI doesn't really have any eyes. It can look at code, but front end is multimodal. And so my experiences with trying to plug AI into, let's say agent, browser or playwright, mcp to give it.

You can give it tools to allow it to look through a front end and sort of look at images, but in my experience it's not very good at that yet. And it can't create a nice front end in a mature code base. It can sort of spit one out. But what it can do is you say, okay, I want some ideas on how this front end might look. Give me three prototypes that I can click between in a throwaway route that I can decide which one looks best.

And you take the asset of that prototype and you then feed it back into the grilling session or you get feedback on it, blah, blah, blah, blah, blah, answer your question kind of thing. The prototype is just, you know, it's messy. It's supposed to give you feedback early on in the process. So that's a great way of working with front end code. Great way of looking at software architecture in general.

Let's go. One more question, Jan. In your system, how do you integrate respecting an architecture, a design with API contracts and fitting with a large assistant? Security constraints, all kinds of constraints like that? Yeah, there's a lot in that question.

The question was how do you conform with existing architecture? How do you do, how do you make it conform to the code standards, like of your code base or architecture design, APIs, security rules that constrains your design? Yeah, I'm going to answer that in a bit, if that's okay. So hopefully we have started to get some stuff cooking. It's just pinging on the explore Phase here.

Tempted to just start running at afk. Maybe I will, maybe I won't. What it's essentially doing is it's exploring the repo. It's going to then start implementing based on what we wanted. Let's actually have one more question just while it's running.

Yeah. Why are there AI qa? Yep. So question was, why do you not get AI to qa? AI to qa?

I just got jargon overload for a second. Why do you not get AI to test its own code now? Of course you absolutely can. And I think while it's doing. While it's cooking here, okay, it's got a clear picture of the code base.

It's assessing the issues, it's doing issue 002 as next task. I'm again going to show you that in a bit. I think the sort of. Because you definitely should do an automated review step as part of implementation. So you have your implementation.

You should then because tokens are pretty cheap and AI is actually really good at reviewing stuff, you should get it to review its own code before you then QA it. I found that that catches a ton of different bugs. And the way that works is I will just do a little diagram is if you have, let's say, an implementation that sort of like used up a bunch of tokens in the Smart zone, if you get it to sort of try to do its reviewing, it's going to be doing the reviewing in the dumb zone. And so the reviewer will be dumber than the thing that actually implements it. If we imagine this is the.

Let's be consistent. That's the review, that's the implementation. Whereas if you clear the context, then you're essentially going to be able to just review in the Smart zone, which is where you want to be. Let's see how our implementation is doing. Okay, good.

It's generating a migration that looks pretty nice. We're getting some code spitting out and while I'm sort of like, aha, here we go, tdd. Let's talk about tdd and then I think we'll have a little. Another little break. TDD I've found, is absolutely essential for getting the most out of agents.

Raise your hand if you know what TDD is. Cool. Okay. TDD is test driven development. What it's essentially doing is it's doing a something called Red Green Refactor.

And if you look in the code base, you'll be able to find a skill which really describes how to do Red Green Refactor and teaches the AI how to do it. So what it's doing is it's writing a failing test fail first. So it's saying, okay, I've broken down the idea of what I'm doing and I'm just going to write a single test that fails and then I need to make the implementation pass. I have found that first of all, this adds tests to the code base and this tends to add good tests to the code base. And so we've got this kind of gamification service.

It looks like it's using some existing stuff to create a test database. Test fails because the module doesn't exist yet. Okay, we've confirmed red and then it goes and hopefully runs it and it passes. I found that. Raise your hand if you've ever had AI write bad tests.

Yeah, it tends to try to cheat at the tests because it's sort of doing it in layers. It will do the entire implementation and then it will do the entire test layer just below it. I'm just going to say, yes, you're allowed to use NPXVText and using this technique, it generally is a lot harder to cheat because it's sort of instrumenting the code before it's then writing the code. So I find that TDD is so, so good for places where you can pull it off. And in fact, it's so good that I sort of warp my whole technique around getting TDD to work better.

I can see some drooping eyes. It is so hot in here. You cannot imagine how hot it is up here. Let's take another five minute comfort break. Let's come back at quarter two, I think, have a nice generous one and we'll be back in about six, seven minutes.

And I'll talk about how I think about modules, think about constructing a code base to make this possible. I've just been sort of fiddling with the AI here and we have ended up with some with a commit. So we have something to test. Issue number two is complete. Here's what was done.

This is kind of what it looks like when a RALPH loop completes is you end up with a little summary. And we have now something we can QA because we did the feedback loops, because we did the tracer bullets, because we were said, okay, give us something reviewable. At the end of this, we can immediately go and QA it. Now there's nothing less exciting than watching someone else QA something, but hopefully we can have a little play. Let's just check that it works at all.

In fact, before I go there, I just want to sort of Work through what just happened, which is we see that it's created some stuff on the dashboard and it then ran the feedback loops. So it then ran the tests and the types. Now, TDD is obviously really important, and it's really important because these feedback loops are essential to AI. Essential to get AI to produce anything reasonable. Because without this, AI is totally coding blind.

Right? You have to, have to. If your code base doesn't have feedback loops, you're never, ever, ever going to get decent AI, decent output out of AI. And often what you'll find is that the quality of your feedback loops influences how good your AI can code. Essentially, that is the ceiling.

So if you're getting bad outputs from your AI, you often need to increase the quality of your feedback loops. We'll talk about how to do that in a minute now. So it ran NPM run test, NPM run type check. It got one type error and it needed to fix it with a nice bit of Typescript magic. Very good.

Yeah. Type A level, thresholds, number. Okay. You see why I stopped teaching Typescript? Because just AI knows everything now.

So. And it ran the tests and it passed and it's looking good. So we now end up with 284 tests in this repo. Pretty good. I do find front end really hard to test here.

We're essentially just testing the service. So we've created a gamification service. If we look up here and then we have a test for that service, you can see the service and the test itself. Now, if I was doing code review here, I would then go to. I would first go to review the tests, make sure the tests were testing reasonable things, and then go and kind of review the code itself just to make sure that it's not doing anything too crazy.

The essential thing is I need to actually look at the dashboard. I'm going to log in as a student, if it'll let me. Maybe it won't let me. Come on, son. There we go.

Let's log in. As Emma Wilson, head into courses. Let's say I've got an introduction to Typescript. Continue learning. Yes, I completed this lesson.

Something went wrong. I imagine it's because I don't have SQLite error. I don't have the right table. So I need a table Point Events. Point Events is a strange table name.

I'm not sure quite what it was thinking there. Let's suspend. Let's run npmdb Migrate or push, I think can't remember which one it was, but you kind of get the idea right? I'm not going to subject you to watching me do QA because it's so dull. But at this point, I would essentially go back in, I would let me open the project back up and I would.

This, this is a crucial moment and it's so important to QA it manually here, because qa, oh dear, oh dear, what's going wrong? There we go. QA is how I then impose my opinions back onto the code base, how I impose my taste. What you'll often find is that there are teams out there who are trying to automate everything, like every part of this process. And they will tend to.

If you try to, like, automate the sort of creation of the idea, automate the qa, automate the research, automate the prototype, you end up with apps that I feel just lack taste and are bad. Maybe they just don't work, or they don't even work as intended, or there's just no, you need a human touch when you're building this stuff, because without that, you just end up with slop. And we are not producing slop here. We're trying to produce high quality stuff. And so that's what the QA is for.

So I'm going to do two things in this final section, which is I'm going to first tell you how to. There's probably a question in your mind here, which is, let's say I have a code base that I'm working on and it's a bad code base. It's a code base that's like really complicated, that AI just never does good work in. And maybe actually most humans that go into that code base don't do good work. How do I improve that code base?

And the second thing is, I'll show you my setup for parallelization. So let's go with bad code first. Now, where is it? Where's the diagram? Here it is.

In his book the Philosophy of Software Design, John Ousterhout talks about the ideal type of module. And let's imagine that you have a code base that looks like this. Each of these blocks here are individual files, and these files export things from them. You know, they have things that you pull from the files that you then use in other things. And so you might have these weird dependencies where this file over here might rely on this file or might rely on that file, for instance.

Now, if these files are small and they don't kind of like export many things, then John Osterhaut would call these shallow modules, essentially, where they're not very. They kind of look like this if I actually. No, I can't make a good diagram of it. They're essentially lots and lots of small chunks. Now, this is hard for the AI to navigate because it doesn't really understand the dependencies between everything.

It can't work out where everything is. You know, it has to sort of manually track through the entire graph and go, okay, this relies on this, this one relies on this one, this one relies on this one. And it's then also hard to test this as well, because where do you draw your test boundaries here? Do you test each module individually? Like just literally draw a test boundary?

No, don't do that around this one and then maybe another test boundary around the next one and then the next one? Or should you sort of do big groups of it? Should you say, okay, we're going to test all of these related modules together and just sort of, you know, hope and pray that they work? Now this means that if I think that bad tests mostly look like that, where the AI essentially tries to sort of wrap every tiny function in its own test boundary and then just sort of test that those individually work. But what that does is it means that when, let's say this module over here calls those two, so it depends on both of these, then this module might misorder the functions, or there might be sort of stuff inside that poor module that's worth testing on its own.

And if you then wrap this in a test boundary, what do you do? Do you mock the other two modules? How does that work? So actually, figuring out how to build a code base that is easy to test is essential here, because if our code base is easy to test, then our code, our feedback loops are going to be better and the AI is going to do better work in our code base. Does that make sense?

So what does a good code base looks like, look like? Well, not like that. It looks like this, where you have what John Ousterhout calls deep modules, modules that have a little interface on there that expose a small, simple interface that have a lot of functionality inside them. Now, what this means is that these are easy to test because you just. Let's say that there's a dependency between this one and this one.

My arrow working. Yeah, there we go. Then what you do is you just wrap a big test boundary around that one module, around this one up here, and you're going to catch a lot of good stuff because there's lots of functionality that you're testing. And really the caller, the person calling the module is going to have A simple interface to work from, so it's not too tricky. That makes sense.

Deep modules versus shallow modules. This is good. This shallow version is bad. And what I find is that unaided, or if you don't. If you don't watch AI carefully, it's going to produce a code base that looks like this.

So you need to be really, really careful when you're directing it. And that's why too, is that if we look inside the prd, where is the PRD gone? It's inside the issues. It's inside the gamification system. Not found.

Of course it's not. Here it is. Then I have inside here data model the modules. So it's specifically saying, okay, this gamification service is a new deep module which we're going to test around. It's going to have this particular interface and it's going to have.

Okay, we're modifying the progress service too. We're modifying the lesson route, we're modifying the dashboard route, etc. So I'm being really specific about the modules that I'm editing, and I'm making sure that I keep that module map in my mind at all times throughout the planning and then throughout the implementation. That makes sense. Very, very useful.

It's useful for one other reason, too. Not only does it make your app more testable, but you get to do a little mental trick. And I'm going to refill my water while you wait for what that is. Let me. Let me get a question from you guys.

Or raise your hands if you feel like. If you feel like you're working harder than ever before with AI. Yeah. Raise your hands if you feel like you know your code base less well than you used to. Yeah, this is a real thing.

Because we're moving fast, because we're delegating more things, we end up losing a sense of our code base. And if we lose the sense of our code base, we're not going to be able to improve it. And we're essentially delegating the shape of it to AI. I don't think that's good. But then how do we make it so that we can move fast while still keeping enough space in our brains?

I think that this is a way to do it because what you're doing here is not only are you thinking about creating big shapes in your code base, big services, what I think you should do is design the interface for these modules, but then delegate the implementation. In other words, these modules can become like gray boxes where you just need to know the shape of Them, you need to know what they do and sort of how they behave and. But you can delegate the implementation of those modules. I found this is really nice. I don't necessarily need to code review everything inside that module.

I don't necessarily need to know everything of what it's doing. I just need to know that it behaves a certain way under certain conditions and that it does its thing. So it's kind of like, okay, I've got a big overview of my code base and I understand kind of the shapes inside it, understand what the interfaces will do, but I can delegate what's inside. I found that has been a really nice way to retain my sense of the code base while preserving my sanity. Makes sense.

And so you might ask, how do I take a code base that looks like this and then turn it into a code base that looks like this? How do I deepen the modules? Well, we have. Hopefully it's in here. Pretty sure it is.

We have, we have a skill and that skill is called Improve code base architecture. Nice and direct. Let's run it. What this skill is going to do is it's essentially just going to do a scan of our code base and looking for what's available here. And feel free to run this yourself if you're running the exercises and it's exploring the architecture, exploring essentially how to work within this code base.

And. And it's going to attempt to find places to deepen the modules. Pretty simple. One really cool thing that it found here is part of my course Video manager app is a video editor. A video editor built in the browser, which is really hardcore.

It's a decent bit of engineering and I wanted a way that I could wrap the entire front end all the way to the back end in a single big module so that I could test the. The fact that I press something on the front end and it goes all the way to the back end. And so I found a way essentially by using a kind of discriminated union between the two types here, by sort of, I was able to use this skill to essentially have a huge great big module that just tested from the outside or was testable from the outside, this video editor infrastructure. And it meant that AI could see the entire flow, could act on the entire flow and test on the entire. And honestly, it was just night and day in terms of the ability of AI to actually make changes, because AI working on a video editor is pretty brutal if you don't give it good tests.

So that is, honestly, if you take one thing away from today, Just try running this skill on your repo and see what happens. Let's go to slido. Let's check a couple of questions just while this is running. So let's see. Have you tried Claude's auto mode?

With Claude, enable auto mode. That way you can avoid many of the obvious permission checks. We'll talk about permission checks in a second. Do I keep the markdown plans and issues for later reference? Okay, this is a great question.

So let's say that you have a great idea, you turn it into a PRD and you then implement that PRD and the PRD is essentially done. Raise your hand if you can keep that information in the repo. So you turn it into a markdown file. Raise your hand if you want to keep that around. Cool.

Okay. And raise your hand if you don't want to keep it around. If you want to get rid of it as soon as possible. Yeah, this is, I think, a question that doesn't have a clear answer. What I'm really scared of with any documentation decision is that let's say that we have a PRD for this gamification system.

We, we keep it in the repo, we go on, go on, go on. Let's say a month later we want some edits to the gamification system and we go in with Claude and it finds this old PRD and says, yes, I found the original documentation for the PRD system. Well, it turns out that the actual code has changed so much from the original PRD that it's almost unrecognizable. The names of things have changed, the file structure has changed, even the requirements may have changed. We might have actually tested it with users.

This is Doc Rot, where the documentation for something is rotting away in your repo and influencing Claude badly or Claude agents badly. So I tend to not keep it around, I tend to get rid of it. And for me, because my setup uses GitHub issues, I just mark it as closed. It can fetch it if it wants to, but it's got a visual indicator that it's done. So I tend to prefer ditching these thoughts on the BEADS framework from Steve.

I've not tested it, but it seems like sort of another way to manage Kanban boards and issues. Seems very good, but I've not tried it. Let me just quickly check the setup here. Let's take a couple of questions from the room. Anybody got any questions at this point about anything that we've covered so far, especially this last week?

Yes, They will create like codes, rough. How about migrations like with migration files, Would you also squash them often then, like database migrations? Yeah. I don't know. I hope that answers your question.

I'm so sorry. No, no, I think database migrations are a different thing because you have a sort of running record of exactly what changed and it's more deterministic. And I think. Yeah, it's an interesting analogy. I'm not sure.

Let's talk about it afterwards. That's a good way of saying I have no idea. Yeah, yeah. So you mentioned that we normally use the prd. You mentioned you don't review the PRD once it does.

Sorry, guys, I'm just trying to listen to this guy's question. You mentioned you don't read the PRD once it does. Have you considered using a deep. Think, like, look at this PRD and tell me. Yeah, the question here is, should I, in the sort of early planning stage, be trying to optimize the plan?

This is something I actually see a lot of people doing and it's a really good idea. So when you go back to the phases. So let's say that you have all of these phases here and you get to the point where you've sort of figured out everything with the LLM, you understand where you're going. You've created this sort of journey, destination, document here. How do you then, like, should you then try to optimize and optimize and optimize that PRD until it's the perfect PRD you can possibly imagine?

I don't think there's a lot of value in that because I think the journey is really just sort of a hint of where you want to go and the place that you need to be putting the work is in qa. And you can sort of do that afk, I suppose, but in my experience, you're not going to get a lot of juice out of it. Like, it's the. The thing that really matters is getting alignment with the AI, which is you do in the grilling session initially. Let's have one more question.

Anyone got any more? Yeah. Do you get in your workflow to get it to code the way you want it to code? So by the time you get to code review, it's at least familiar with the libraries you want to use? Yeah, we had this question before, actually, which was like, how do you enforce your coding standards on the agents, essentially, how do you get it to code how you want it to code?

Now, there's essentially two different ways of doing it. You've got, come on, push and You've got pull. What do I mean by push and pull? Push is where you push instructions to the LLM. So you say, okay, if you put something in Claude MD talk like a pirate, that instruction is always going to be sent to the agent, right?

So that is a push action. You're pushing tokens to it. Pull is where you give the agent an opportunity to pull more information. And that's for instance, like skills. So a skill is something that can sit in the repo and it has a little description header that says, okay, agent, you may pull this when you want to.

My thinking, my current thinking about code review and about coding standards looks like this. When you have an implementer, what's going on? There we go, implementer. I'm going to make this less red in a second. Then you want the coding standards to be available via poll.

If it has a question, you want it to be able to sort of answer it. But if you then have an automated reviewer afterwards, then you want it to push. You want to push that information to the reviewer. You want to say, these are our coding standards and make sure that this code follows them. So if you have skills, for instance, then you want to push that stuff to the reviewer.

So the reviewer has both the code that's written and the coding standards to compare to. Hopefully that answers your question. I can show you an automated version of this as well. Actually, let's do that now, just while it's fresh in my mind. I recently spent maybe a week or so building this thing called Sandcastle.

And Sandcastle is a. I was sort of unhappy with the options out there for running agents afk. And what this does is it's essentially a typescript library for running these loops. So you have a run function that creates a work tree, sandboxes it in a Docker container, and then allows you to run a prompt inside there and in that work tree. Then it's just a git branch and you have that code and you can then merge it later if I open up.

There are some really, really nice ways of viewing this. And it essentially allows you to run these kind of automated loops and allows you to parallelize across multiple different agents really simply. So I'll go into my sandcastle file, go into main TS here, and let's just walk through this. So this is kind of like. I showed you a sort of version of the RALPH loop earlier.

This is where we take it from sequential into parallel. We have here, first of all, a planner that takes in. It has a plan prompt here that looks at the backlog and chooses a certain number of issues to work on in parallel. Remember I showed you that Kanban board where it had all the blocking relationships, it works out all of the phases. So this one will say, okay, let's say we have.

You can ignore all this glue code here. This is essentially just a set of issues, GitHub issues with a title and with a branch for you to work on. And then for each issue we create a sandbox and then we run an implementer in that sandbox pipe passing in the issue number, issue title and the branch. This is like the loop that we ran just before. Then if it created some commits, we then review those commits.

This is essentially the loop. What do we do with those commits? We pass those into a merger agent which takes in a merge prompt, takes in the branches that were created, takes in the issues, and it just merges them in. If there are any issues with the merge, with the types and tests and that kind of thing, it solves them. And this has been my flow for quite a while now for working on most projects.

It works super, super well. And I recommend you check out Sandcastle if you want to learn more. And to answer your question properly is that in the reviewer, I would push the coding standards in the implementer, I would allow it to pull. And I'm actually using Sonnet for implementation and Opus for reviewing, because I consider reviewing sort of. I need the smarts then.

Any question? Actually, let me. Before we do more questions, let's go back here. Okay, where are we at? Okay, we're sort of zooming everywhere in this talk because I'm kind of having to run things in parallel.

So let's go back to the improve code base architecture. It has finally finished running and it's found a bunch of architectural improvement candidates. So it's got essentially a cluster of different modules that are all kind of related that could probably be tested as a unit. Got number one, the quiz scoring service. There's some reordering logic extraction as well.

It has arguments for why they're coupled and it has a dependency category as well. So local substitutable in SQLite within memory. TestDB quiz scoring service currently has zero test. This is the biggest gap. So this is what it looks like when we come back of improved code base architecture.

Okay, so we have nominally kind of 17 minutes left. I don't know about you, but I'm knackered. I want to let me kind of sum up for you because I think we're sort of reaching the end of our stamina I'm going to be available for the full time if you want to come and ask me questions. I might do one more check of the slide here, but let's kind of sum up where we've got to. So this is essentially the flow where throughout this whole process, we're bearing in mind the shape of our code base.

This is not a Spectacode compiler, this is not an AI that's sort of just like churning out code. We are being very intentional with the kind of modules and the shape of the code base that we want. We are making sure that we are as aligned as possible by using the grilling session, by really hammering out our idea. We're not over indexing into the prd, we're not trying to read every part of it, we're not thinking too much about it even. We are then just turning that into a set of parallelizable issues which can be worked on by agents in parallel.

We implement it and we QA and code review the hell out of it and then keep going back to that implementation. One thing I didn't really mention is that in the QA phase, what the QA phase is for is creating more issues for that Kanban board. So while it's implementing, even you can be QA ing the stuff and going back adding more issues. And the Kanban board just allows you to add blocking issues kind of sort of infinitely really. And then once that's all done, once you've got code that you're happy with, once you've got work that you're happy with, then you can share it with your team and you can get a full review.

So this is kind of like once you get here, this is kind of one developer or maybe a couple of developers sort of managing this. And then it's kind of up to you to figure out how to merge it back in. Of course, all of this can be customized by you. This is just something that I have found works. I'm not trying to sell you on a kind of approach here.

What I recommend, if you take one thing away from this session, is that you should head back. You should head to Amazon and just buy a ton of those old books because, I mean, I just found it so enlightening reading them. Pre AI writing is always really fun to read anyway. And on every single page I found that there was something useful and something interesting to read. So thank you so much.

Thank you for putting up with the heat. Hopefully your body temperatures will reset soon. Thank you very much.
