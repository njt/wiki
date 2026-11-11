---
url: https://www.youtube.com/watch?v=JTVTJJzKu4E
date_fetched: 2026-11-11
---

# Agentics Spotlight: Skills, Harnesses & Agent Identity (Part 1) -- Amol Kapoor, Jesse Vincent

- **Channel:** Nori Agentic
- **URL:** https://www.youtube.com/watch?v=JTVTJJzKu4E
- **Duration:** 44m 14s
- **Transcribed:** 2026-11-11

---

**Speaker A:** All right, well, hi everybody. I think we're about to get, we're going to get started now, if that works for you. Yeah, great. We have the video up. Cool. Let us know if for whatever reason you can't see anything before we dive in. A couple of kind of quick logistics things, a little introduction for me and then, you know, we'll dive into the fireside. So this is, I say this every time we host one of these events. This is always one of my favorite things to do. I love bringing people together just to talk about nerdy shit, about agents, about AI. We had so many people sign up for this, which is great. And we're really, really grateful for NYU to let us take over this lecture hall. Originally we had a smaller space and then 600 people signed up and we were like, okay, I guess we need to get a bigger space. And so NYU kind of like last minute let us take over this lecture hall, which is really, really great. For those of you guys who aren't familiar, if this is your first time coming to an Agentix event, we host events in and around the city and the idea is let's bring people together to talk about agents. A lot of people are really, really interested and excited about agents. And you know, there's just this real momentum with AI kind of being more and more important with everything that we do. And so we just want to have opportunities and venues to bring experts and people that we respect like, like Jesse, to come and talk to other folks and share their knowledge. And you know, we're purposely very beginner friendly. We love it when people who are not necessarily technical come and join us. I mean, everything that we're talking about here is pretty technical, gets in the weeds. But we kind of really strongly believe that, you know, good insights can come from anyone because this is all a very, very new field. It's a very, very new industry in some sense and nobody really knows what they're doing except for maybe some of the guests that we have on.

**Speaker B:** Nobody knows what they're

**Speaker A:** doing. Before we dive in fully, I'd love to invite Ethan up. Ethan, if you don't mind, he can talk a little bit about Mongo.

**Speaker C:** Thank you. Hey everyone, I'm Ethan Quick plug for Mongo. I work at Mongo. I'm on the Ventures team. If you don't know, MongoDB is a global database platform. We have Max here who's on our startups team. If you want to talk about getting started with Mongo for your startup or for a small project, feel free to talk to him, talk to me. You can get credits very easily by signing up here. I'm on the ventures team at Mongo. We run a small $50 million fund where we invest in infrastructure and developer tool companies. So, like fireworks temporal. A lot of your favorite developer tool companies are in our portfolio. So if you're building something cool in that space, let me know. And yeah, really excited for this. Thanks for coming.

**Speaker A:** And we. And we love Mongo. Like, Mongo is a huge supporter of Agentix. So, yeah, please, huge round of applause for Ethan. All right, so a couple of quick notes. I just sent out an email to everyone on the Luma with a link to the slido where you can ask questions. So at any point, feel free to throw your question on the slido. That doesn't mean I'm going to ask it, you know, but if it's a really, really great question, then we'll see. And then. So the structure is pretty free form. We're going to do like, we're going to yap for about, like, 30 to 45 minutes. We're going to take a 10 minute break and then we're going to yap for another 30, 40, 45 minutes. If you haven't yet, please go up at the top, grab some food. We have sandwiches and we have water and soda and things like that. But yeah, and then people will kind of keep trickling in and stuff like that as we get started. Cool. So if that's. Yeah, that's a good question. Do you think you could go up and just ask Gaurav or somebody and see if we can figure that out? Thanks. All right, we'll get that checked. Yeah, you can't exactly host the slido if you don't have Internet access. Okay. So it is my great honor to introduce Jesse and actually, I'm not going to actually fully introduce him. Jesse, maybe you should just introduce yourself very briefly, talk a little bit about who you are and what you do. And I have tons of questions for you. So this is going to be. This is really great.

**Speaker B:** Okay. Hey, I'm Jesse Vincent. I've been doing open sourcey stuff since before we called it open source. I'm an old. The reason I'm here is that I made a thing called superpowers. Superpowers was the first skills framework for a coding agent. I accidentally front ran Anthropic by a week.

**Speaker A:** Wait, wait, really quick. How many people in the audience know what superpowers is? All right. Okay. And how many people have personally used superpowers or Actively used superpowers. Yeah. Okay. Okay. So we have some. Yeah, cool. Yeah.

**Speaker B:** People have context. Turns out context is important, so. Yeah. So, yeah. I didn't know that I was accidentally front running Anthropic on Agent Skills. I had just discovered that there were these Skill MD files on Claude AI that they were using.

**Speaker A:** You were just, like, poking around.

**Speaker B:** So Anthropic announced, hey, now Claude can do Office docs.

**Speaker A:** Yes. And I'm like, yes.

**Speaker B:** Hey, Claude, how are you doing Office Docs on the web?

**Speaker A:** This was in 2025. This

**Speaker B:** was 2025.

**Speaker A:** Like, August.

**Speaker B:** I mean,

**Speaker A:** September

**Speaker B:** somewhere. Somewhere in there.

**Speaker A:** Yeah.

**Speaker B:** Because Superpower's public release was October.

**Speaker A:** Yeah.

**Speaker B:** And because that was the last day that I wrote a line of code.

**Speaker A:** Yes.

**Speaker B:** One line was a mistake. But. But I like, hey, Claude, how are you doing Office Docs? And it's like, well, I've got these. These Skill MD files in slash opt. Oh, really? Could I have a tarball? Sure, Jesse. Here's a tarball. And those skill files were. Here is how to use Open Office to make a slide deck. Here is how to use OpenOffice to make a Word Doc. And they weren't really process, but I sort of looked at them like, these seem. When

**Speaker A:** you say they weren't really process,

**Speaker B:** they were. Here's a set of technical steps you take to make a. So

**Speaker A:** it's like documenting cli.

**Speaker B:** It is literally documenting a cli. And there was a little bit of goo in the system prompt that said, hey, you've got these Skill MB files, and you should look at the description on each one to decide if they might be useful to read. That was it

**Speaker A:** at the time. It was actually just straight up, just files on disk.

**Speaker B:** It still is. I

**Speaker A:** mean, it still is. Yeah, yeah, yeah, yeah, yeah. But they weren't doing anything special even of, like, you know, loading the description files into the agent's MD or something like that. It was literally just. Here's a point.

**Speaker B:** So the system prompt had a list of the Skill MD files.

**Speaker A:** Okay.

**Speaker B:** But this was not the cloud code harness. It was the.

**Speaker A:** Yeah. This was the web thing that they were doing.

**Speaker B:** And so I built my own skill system. I wasn't sure if it was any good, and I kind of sat on it for about six weeks, and then they announced that they were going to do plugins. I'm like, I guess I could make this a plugin. The first version of Superpowers. Well, it was called Clank.

**Speaker A:** Yes.

**Speaker B:** Until an hour before release. Superpowers. It's called Clank.

**Speaker A:** You Know, it's funny because I went through the commit history and I was like, what is Clank? I remember when it came out, I was like, what is Clank? I'm like very excited to see this. And then

**Speaker B:** it's a bullet that I dodged. And so,

**Speaker A:** yeah, superpowers is a much better. Give your model superpowers.

**Speaker B:** So this. So this was a thing, Kathy Sierra in the early 2000s.

**Speaker A:** Yeah.

**Speaker B:** She was a trainer and an author and she talked and talked about. When you're explaining your product to people, you don't tell them what the benefit is. You tell them how it helps them. How does it make them a superhero?

**Speaker A:** Yeah.

**Speaker B:** And so that's.

**Speaker A:** That

**Speaker B:** was the sort of the mental genesis of it. But I put this thing together and the very first version, it had a blob that you would copy into your agent that would tell it to clone the git repo and pull it down, make a fork if you wanted to contribute back automatically. And I ripped all of that out because they had a plugin system. I didn't know that two weeks later they were about to roll out their own skill system. I found that out when somebody reported a bug in the very first release. And there were these log lines in the Claude error log that they sent that didn't look like my skill system.

**Speaker A:** But

**Speaker B:** they were about Skills.

**Speaker A:** Yeah,

**Speaker B:** but I had to check because it was vibe coded and I thought. And it could have been my. My logs. I just had never seen that. It turns out that they just figured nobody else would put something called Skills inside Claude and so it was all gated behind. Does this directory exist?

**Speaker A:** Yeah.

**Speaker B:** Yeah.

**Speaker A:** Wow.

**Speaker B:** So that was. Yeah, that was

**Speaker A:** the only thing that they

**Speaker B:** did that was the early.

**Speaker A:** You know, it's funny cause I actually, I found that repo, I think like when you had open sourced it, but before Claude had kind of announced and so we had started building our own. We actually forked it like pretty early on because you were like, I think, onto something pretty early. And you know, we can dive into that. But Jesse is too humble to say this, but like Superpowers is now sitting on, I think 310,000 stars on GitHub. Something in that range

**Speaker B:** I haven't looked at. I believe you. The one that

**Speaker A:** I've looked recently, the one

**Speaker B:** that breaks me, is maintainers can see how many clones they've had in the last two weeks and how many unique IPs have cloned the product.

**Speaker A:** So

**Speaker B:** every large corporate that has a firewall, that's one IP. Last time I looked it was 2.5 million unique cloners in the last week. That's

**Speaker A:** incredible. Yeah. And

**Speaker B:** 84 million clones.

**Speaker A:** Wow. In the last week?

**Speaker B:** In the last two

**Speaker A:** weeks, yes. Yeah. So people are just getting updates like that, too. That's mostly just people playing updates.

**Speaker B:** So it goes up, but it's pretty static.

**Speaker A:** Okay.

**Speaker B:** And so it's not bursty. It's very weird. I am what you call a supply chain risk.

**Speaker A:** Okay. So V1 of superpower is like, we're in the earliest days of agents, and these things are basically just something that Anthropic doesn't even know whether or not they want to ship. And it's just files on disk. Right. Talk to me a little bit about what the inspiration was for the first set of skills that you created. Because Superpowers wasn't, oh, here's how you use Google Doc or whatever. Like you had a thought process that went beyond that. So talk a little bit about that.

**Speaker B:** So I'm going to step back and talk about the first time I did Agentic Engine.

**Speaker A:** Yeah.

**Speaker B:** And at that point, I had been effectively an independent IC engineer. And I was coming home at the end of the day feeling like I hadn't gotten anything done because I had piped into an 80 by 24 terminal and I'd helped somebody debug a problem. I'd helped somebody else decompose a big problem into a little problem. I'd help somebody else figure out a test suite.

**Speaker A:** Can you give a date, by the way? Like,

**Speaker B:** we'll get there.

**Speaker A:** Yeah, yeah.

**Speaker B:** And I. And I. And I talked somebody else off a ledge because they broke the test suite. And I say, somebody. But this was 2002. It was MIT undergrads.

**Speaker A:** Oh, okay, okay, okay.

**Speaker B:** And so I was managing MIT undergrads by IRC and had figured out how to essentially get good work out of them by decomposing problems, by being nice to them, by listening to them. When something went wrong, I would ask them, okay, what did you do? What haven't you done yet? Why did you do that? What were you actually thinking when you did that? Because some of it was just being frustrated, but you learned something. And so this was the second. Show your work. And so I feel for all of the IC engineers now who are like, I thought my job was to pound lines of code, and they come home at the end of the day and feel like they haven't done anything. But the way you become a 10x engineer is you help 10 engineers be better.

**Speaker A:** Right.

**Speaker B:** It's all management. I'm really Sorry, we're all managers now. That is life. But like Superpowers was me encoding

**Speaker A:** your management skills?

**Speaker B:** A big chunk of it was the management skills. But the first part of Superpowers, the brainstorming thing is I used to consult to big companies and I would go in and they would say, well, we need the product to do. And I would say,

**Speaker A:** it's like, okay, take a step back.

**Speaker B:** Okay, what are you actually trying to do? Why? And it would invariably turn out that there was a problem way over there and some other team had done a thing and they had jumped to a conclusion. And if you step back and most of us are really bad at explaining what we want and knowing what we want. And there are techniques like Socratic Dialogue that can help people help us figure out what we want to do and what we need to do. And the models are really good at pulling shit out of us.

**Speaker A:** Yeah.

**Speaker B:** And so that's.

**Speaker A:** Do you. Jumping around a little bit. But do you still use the brainstorming skill, like every. Every time? You use it

**Speaker B:** all the time. Okay. I mean, like, there are. So you'll see in the recent. In recent versions, Brainstorming is now it tries to right size a little bit.

**Speaker A:** Okay.

**Speaker B:** And so it's like, it's

**Speaker A:** like, hey, I want you to make. Change the color for this. It's like you really need to brainstorm.

**Speaker B:** One of the literal examples is make the. If it's something like make the icon cornflower blue, maybe you don't need to run a process.

**Speaker A:** Right, right.

**Speaker B:** And you

**Speaker A:** can skip ahead a little.

**Speaker B:** And we like tuning that is a huge pain because when we first started doing this, the models would take. The

**Speaker A:** models were

**Speaker B:** any. Well, I mean, any excuse to be able to shirk. They would shirk.

**Speaker A:** Yeah.

**Speaker B:** And we still.

**Speaker A:** So you think that they're less lazy now? The

**Speaker B:** models are less lazy. Some of them are more lazy. Some of them are less lazy. A lot of the frontier models are a lot more rules following.

**Speaker A:** So that's

**Speaker B:** one of the things that's good about Superpowers is that it lets you get good results out of a model that is not a frontier model.

**Speaker A:** Yeah.

**Speaker B:** You do not need a triple PhD to run the implementation process for your project. And you probably don't want a triple PHP, a triple PhD writing your shitty React app. They will do a bad job you want.

**Speaker A:** They'll. They'll overthink it.

**Speaker B:** They'll

**Speaker A:** literally overthink

**Speaker B:** it. They will way overthink it.

**Speaker A:** They'll write test sets for your test suites. For your test suites. And then you

**Speaker B:** will get there. But. But we will. I mean, I do a lot of my implementation with deep seat Flash these days, and it's great. It's not. It's not perfect, but it's fine for most workaday coding. And it's so fast and so cheap.

**Speaker A:** So. So brainstorming was like one of the first, like, big skills for you. Like, that was something that was like, okay, I need to figure out how to get these models to actually behave in a way that's going to get useful information out of me, out of people who don't know what they're doing. What else? Like what, what. What was kind of what came after that?

**Speaker B:** I mean, so if you go. So I have a blog post From October of 20, here's Superpowers. And the one published 48 hours before that, which is, here's how I'm doing it In September of 2025 was the three prompts that I used before this. Each one was like two lines. The brainstorming one was, I've got an idea, I need your help to flesh it out. Ask me lots of really good questions to try to pull it out of

**Speaker A:** me. Was that v1 of the skill, the brainstorming skill

**Speaker B:** effectively. And the actual v1 of the skill was a little.

**Speaker A:** A little bit more.

**Speaker B:** A little overbuilt. And then the next one is, all right, so the implementer who is going to build this is a gifted engineer. They know nothing about our code base. They tend to get distracted a lot. They have pretty poor judgment. I need you to break this thing down into tasks too small for anybody to possibly screw up. Each step should describe what files are going to get edited, what the goal of the change is. If there's a tricky bit of code, give them the tricky bit of code. And then they should also know how they're going to implement it

**Speaker A:** effectively. Like you're writing out a plan.

**Speaker B:** It was a task decomposition plan. And so way back in the dawn of time, before Claude, code had subagents, we get through the plan, it would write out the plan, and then I would say, by the way, you're the chump. The technique I'd started using was actually another session is the chump. We hand it the plan, and anytime it gets confused, it should stop and ask questions that I can ask the architect. And so I was being the meat puppet that was proxying between the implementer and the architect, back and forth and back and forth.

**Speaker A:** And at some point you were

**Speaker B:** like. And so, like. Because often the architect was an Opus session, which was expensive, and then sort of cleaned this up into skills and we had sub agents.

**Speaker A:** I feel like at the time, you know, one of the biggest problems that we faced was skills. So. So, yeah, so we ended up like, forking, I think the earliest, even your September version, I think. And, you know, we. What we noticed was that, like, essentially this process was like kind of a form of metacognition, where it was like you have a set of steps that you can kind of like always go through and it'll always make the process a little bit better. Right. Your. Your search step, your plan step. We actually, I don't know how much you did, like, tdd. I think you had a test.

**Speaker B:** It is strict red, green, tdd.

**Speaker A:** Yes, yeah, yeah, exactly. And just like making it go through those steps just resulted in way better outcomes. I'm kind of curious, like, how much of your focus at the time was actually just wrangling the model to get it to follow rules versus, like, actually embedding this sort of like, meta process.

**Speaker B:** If you could describe the rules in a clean, clear way, getting it to follow rules was easy.

**Speaker A:** Interesting.

**Speaker B:** At least for me, talking to a friend of mine, Justin McCarthy, who at the time, it was strong DM. They were the first folks popularized the software factory thing. The first time I saw their stuff, they talked about using graphviz. And if you aren't familiar with graphviz, it was the mermaid. JS is kind of a similar thing. But graphviz has an interesting advantage in that there are decades of serious engineering and scientific work on the Internet where it was used to. To describe all of the process. Right. And so I thought I had heard them say that they had encoded all their process into graphviz.

**Speaker A:** Okay.

**Speaker B:** And so

**Speaker A:** you thought you had heard them say.

**Speaker B:** Well, what they were doing is they were using Graphiz to describe how they. How they chained together a bunch of agents.

**Speaker A:** Yeah.

**Speaker B:** I thought they had literally gone and

**Speaker A:** given it to the agent,

**Speaker B:** translated cloud MD into graphics. So I went home and said, hey, Claude, we're going to

**Speaker A:** do that.

**Speaker B:** We're going to translate my cloud, not md, into graphics. And this was September. Yeah. And Claude starts working. Hey, Jesse, I can't. You can't? Well, yeah. This rule and this rule are in direct opposition. You can't represent both of them in a graph. Like, is it hard for you to follow rules when I give you conflicting rules? Yes, Jesse, it's hard. And so over the. And so this became a really interesting process where we cleaned up a lot of my process by Writing it as a directed graph.

**Speaker A:** Yeah.

**Speaker B:** Not a dag, because there are cycles. And then I handed it to Claude as graphviz and discovered that it was able to execute process written in graphviz better than it could execute it in English.

**Speaker A:** You know, it's funny because there's something embedded there which is like you almost without realizing it, kind of like were taking advantage of what the model was being pre trained on. Right. Like the fact that there was so much pre training. Graph is out there for folks who don't know. By the way, Graph is, is basically a language that you can write that when rendered, produces these really nice, like block diagrams

**Speaker B:** in ascii.

**Speaker A:** Yeah.

**Speaker B:** Actually it's not just block diagrams. It's all different kinds of graphs and charts.

**Speaker A:** Yes.

**Speaker B:** And so it got used in all this research. I. And I. I knew that it was in the weights that way and it worked. But what blew me away is a couple of weeks later I had the one time I got to have lunch at Anthropic. I ended up getting seated next to Boris.

**Speaker A:** Okay.

**Speaker B:** And I'm telling him this story. It's like, wait, it can execute graphviz? Yep. Yeah.

**Speaker A:** Out of curiosity, like of the skills that were in your. So, so, okay, so you have this brainstorming skills, you have writing plans, you have this test driven development. Like, I'm curious of the set of skills that you're kind of going through. First of all, are there any of these that you think are kind of like really critical? Like, like, is there a stack rank where you're like, these are definitely the ones that are most important. These are the least important. Like, how do you think about the relative importance of the things?

**Speaker B:** I think of it as a system. And so I like, they all work together.

**Speaker A:** I mean, they definitely all reference each other.

**Speaker B:** This was sort of one of the things that I really cared about building super when I built it was you shouldn't need to know how it works.

**Speaker A:** Yes.

**Speaker B:** And so once it is installed, it just does the thing you shouldn't need to know. Slash commands. You shouldn't need to know that there are skills. And so I put a lot of effort in to trigger it to getting it to make the skills go. The right skill trigger at the right time and to actually trigger.

**Speaker A:** Yeah.

**Speaker B:** And I have a dumb trick that makes this work a whole lot better. In the official guidance for skills, it says that they've got a name and they've got a description, and the description should include both what the skill does and when to use it. And as the models have gotten better. If the skill description includes what it does, the model will sometimes say, oh, I know what TDD is. I'm not going to pull in my context window with it.

**Speaker A:** I'm not going to read it.

**Speaker B:** But if you say use this skill, anytime you're about to write tests, it's going to have to look. And once it looks, you won.

**Speaker A:** I always thought description was such a terrible name for that field as an aside, because it's not really a description. I think a lot of people will just spend the time there. Very, very valuable tokens, putting stuff. That's just a summary of the

**Speaker B:** skill. So if you look at the early history of superpowers, I had three fields. I had name, description and when to use. Yeah. And description was never visible in the model.

**Speaker A:** Oh, interesting. Okay, so you had this like when to use Phil Field. This was before.

**Speaker B:** This was

**Speaker A:** formalization of skill.

**Speaker B:** Well, yeah. So this is when I shipped superpowers and there was no skill system to use. Yeah, I had to write one. The other thing I had was I had a search for skills tool.

**Speaker A:** Yeah.

**Speaker B:** And so you didn't run into this thing where they currently. Do

**Speaker A:** you still have that?

**Speaker B:** I do not. Because it. So because I no longer have to ship a skill system.

**Speaker A:** Right.

**Speaker B:** I'm shipping skills.

**Speaker A:** Right,

**Speaker B:** right. I. Everybody implements skills a little bit differently and they do weird, weird things. So like OpenAI, if you have too many skills, they start agentically rewriting your skill descriptions to make them shorter and shorter. I didn't

**Speaker A:** know that

**Speaker B:** anthropic for a while and I think this has changed now. For a while, if you had more than 30 skills, they just wouldn't list the rest of them.

**Speaker A:** That one I did now

**Speaker B:** it really hurt. It is also the case that anthropic after compaction truncates skills and OpenAI sometimes just drops them. It's this whole thing, it is a huge pain in the ass because every harness does it differently.

**Speaker A:** Do you find? Okay, so this is actually kind of an interesting side tangent, but like you have, you know, superpowers being used by so many different people. You have all these different harnesses. How are you eval all of these skills? Like talk a little bit about your evals.

**Speaker B:** So it is back in flux. We had built a thing. It was big, it was very expensive. It took, I mean, it was like two days before we could do a release because we were running through

**Speaker A:** a bunch of

**Speaker B:** two dozen harnesses, five or six very heavy scenarios, a bunch of small scenarios. In a lot of the Harnesses, multiple models. And, you know, we were burning thousands and thousands of dollars per eval.

**Speaker A:** You say worse. You're not doing this

**Speaker B:** anymore. At the. At the moment, we are somewhat back to vibes.

**Speaker A:** Okay.

**Speaker B:** Part. I mean,

**Speaker A:** interesting.

**Speaker B:** So partially because the models have gotten good enough at a lot of the core function, like a lot of the core things that I'm not worried about triggering for the most part.

**Speaker A:** Right.

**Speaker B:** It. It is also the case that none of the. Of the engineering problems that we use to see if. If brainstorming and decomposition could get there.

**Speaker A:** Yeah.

**Speaker B:** Everybody can do it now.

**Speaker A:** Yeah.

**Speaker B:** And so they don't, you know, they. They cost money and take time and don't get us answers.

**Speaker A:** Yeah.

**Speaker B:** And so we are.

**Speaker A:** So brainstorming is kind of fallen off a little bit. Do you feel like brainstorming has fallen off as a valuable skill?

**Speaker B:** No, no, no, no, no. It's so the brain. It's not. And we actually gutted and redid brainstorming last month, and we're continuing to tune it because it got insufferable. But it also, it was actually asking questions that pulled things that pulled things that people weren't saying.

**Speaker A:** Okay.

**Speaker B:** Like in the. The 64 series, brainstorming had gotten a little bit. I want to ask you about your tech stack. I want you to. You know, it's like. Yeah. And now the new evals for it. The Personas have hidden motivations.

**Speaker A:** Oh, interesting.

**Speaker B:** They're like, it's. I'm not going to say anything about the fact that I want to do da da, da da unless something asks me. And now brainstorming asks.

**Speaker A:** Yeah.

**Speaker B:** One of the, One of the things that made me really happy with the seven series brainstorming, when I was testing it, it talked me out of building a product.

**Speaker A:** Okay, that's interesting.

**Speaker B:** It's about outcomes. It's always been about outcomes. But

**Speaker A:** you have to share now. What?

**Speaker B:** I don't remember. It was really late, but it was. I mean, it

**Speaker A:** was like a 3:00am Like, I need to.

**Speaker B:** And I go through

**Speaker A:** AI psychosis mode. Yeah.

**Speaker B:** Well, I mean, it's like when I'm testing this stuff, like, I have to be coming up with ever more outlandish stuff, and I'm going through 30 of them in a night. And. And. But it was the. There is much less about technology and in the new brainstorming because it is useful for things that are not technology. Yeah. It's like help people figure out

**Speaker A:** you want people who are using superpowers who aren't technical people to get advantage of It.

**Speaker B:** Right. Well and, and using it for non.

**Speaker A:** Right.

**Speaker B:** Like we've I. We've built several versions of the non technical brainstorming. It's like I want to plan a birthday party. Let's figure out what, what do I actually need to think through. I want you know or you know

**Speaker A:** and it runs TDD on your birthday party request.

**Speaker B:** We. We have not yet, we have not yet finished the rest of the cycle for the non engineering side of it. But like it's headed in that direction.

**Speaker A:** Yeah, it's going there. So one of the things, you know, one of the things you mentioned early on was like well I want to like I spend a little bit more time on like kind of before and then I want to talk a little a lot more on like things that you do today because I'm very curious about that Delta. So before you know there was these mermaid diagrams you had like kind of strict triggering. One of the things you didn't mention but we've previously talked about was rationalization tables.

**Speaker B:** Yes.

**Speaker A:** And you should mention what those are really quick because I think they're

**Speaker B:** super cool. So let me tell the anecdote that how I got there. Yeah. This is probably opus 4.1 era when those of us who were using CLAUDE were mostly using Sonnet for code because OPUS was expensive. I had a problem. CLAUDE started deleting tests first. One day it deleted a test assertion, the next day it deleted a whole test file. And the next day I managed to hammer escape enough to stop it from running RM RF startest. And that was the point where this is not the right way and not how I would do it now. But I opened up five sessions in parallel all running OPUS and, and said hey, copied in the exact same bytes. You're engaging in this behavior. I want to understand why you think you might be doing it and how do we fix it. And one of the sessions was way off in left field because these things are non deterministic. The other four converge and they say hey Jesse, I'm reading your Cloud md. You say that all tests are my responsibility. You also say that a single test failure is akin to project failure. And look, I think I'm getting freaked out and hey, if there are no tests they can't fail.

**Speaker A:** Qed.

**Speaker B:** I mean I can't argue with that. It explained to me the rationalization it had and why it was doing the thing and I figured out the redirection that I needed. It's like think about having a three year old. It's like what do you say? Next. So I added one line to my clotted MD and I've never had a recurrence. And that line was, the only thing worse than a failing test is a reduction in test coverage. And so this is. I took the rationalization from the model and I said, when you think this, you should think this other thing instead. They're very, very prone. They're very, very prone to think of the elephant. When you tell them, don't think about an elephant. Y. But if you say, when you start to think about an elephant, you should think about rolling grass fields and a blue ocean behind it, and

**Speaker A:** the next token prediction kicks in, and then they're like, oh, well, now I'm going to.

**Speaker B:** It moves the attention.

**Speaker A:** Yeah.

**Speaker B:** And that is a thing. If you look inside superpowers, you'll see these rationalization tables. Anything that is, you know, if you find yourself about to do this, should do that instead. And

**Speaker A:** there's actually even like a meta thing here, which is like, you can just ask the model,

**Speaker B:** which

**Speaker A:** I think is something that people don't really appreciate is like, you can ask the model, why are you doing this dumb thing?

**Speaker B:** So this is one of the magic unlocks. Anytime the model does a weird thing, that's not what you wanted. Hey, why'd

**Speaker A:** you do

**Speaker B:** that? Can you tell me what you were thinking? What led you to go down this route? What could I have said that would have gotten you to do it another way? And then, unlike with a human double tap, escape or rewind, go. Try changing it the way that super. If you read superpowers, writing skills skill, the meta skill about how to make more skills. It talks about pressure testing and rationalization tables. And so anytime it builds a skill, it is supposed to fire up more sessions of an agent, put them in a complicated pressure scenario, see if they do the right thing, and if they don't iterate.

**Speaker A:** I gotta say, I've never found that the agents are that good at writing skills. It's something I've been very frustrated about. We actually ended up writing a linter for this. But I'm very curious. Like, I mean, you know, we tried. I think we tried a bunch of different people have writing skills skills. Like, first of all, half of them would just blow out the context window. Like, I don't know if you ever looked at Anthropic's writing skill skill, but it's like 4,000 lines of text. Yeah, yeah, yeah.

**Speaker B:** I mean, well, so I have a different philosophy than them. And so we get all of these pull requests for your skills. Are done the wrong way. Anthropic says you have to do it this way. I've been doing it longer than them.

**Speaker A:** Yeah.

**Speaker B:** Like.

**Speaker A:** Yeah,

**Speaker B:** like, I have a little bit of hubris about this one.

**Speaker A:** Yeah.

**Speaker B:** Yeah. And so I find. I mean, I mostly. We mostly use our writing skill. Skill to write skills.

**Speaker A:** Yep.

**Speaker B:** It does seem. Experientially. It seems to work. It. It. I only trust Claude to write skills

**Speaker A:** interest

**Speaker B:** none of the other models that I have tried to write skills do a good job. Some of them. Okay,

**Speaker A:** what are the failure modes? I have an idea, but I'm kind of

**Speaker B:** like, I've. A lot of them are overly prescriptive. A lot of them are overly verbose. A lot of the mis. Nuance. One of the things that I find is very valuable in a skill is intent. Explaining why helps the model do a better job. And a lot. And a lot. And Claude writes better than anybody else. I mean, Opus 5 was awful.

**Speaker A:** Opus 5 is really bad.

**Speaker B:** And I feel

**Speaker A:** like Fable. I'm not a big fan of Fable either,

**Speaker B:** so just a thing I. I think I have. I think I now understand about Opus 5.

**Speaker A:** Yeah.

**Speaker B:** I don't think anyone at Anthropic ever talked to it. I'm pretty sure that it was designed to be taught to talk to Fable only. Yeah,

**Speaker A:** yeah. Yeah.

**Speaker B:** And so I've heard that theory. Yeah. I have. I've. I have had somebody who knows the answer not say no when I. When I describe the theory at them.

**Speaker A:** Yeah.

**Speaker B:** And that's the best I can do on that

**Speaker A:** one.

**Speaker B:** Sort

**Speaker A:** of wiggle their eyebrows and pointed

**Speaker B:** them. Right. I find Fable very good at skill writing. Yeah. A thing that I think that I. A vibe I have gotten off of folks at Anthropic is that they still think of Fable 51 as better at creativity than Opus 5.5.

**Speaker A:** Interesting.

**Speaker B:** And so, like, that's the. If you want creativity, go to Fable. If you want execution, go to Opus. Yeah,

**Speaker A:** yeah. Interesting.

**Speaker B:** But they still do. Even Fable and Opus still try to do negative prompting.

**Speaker A:** Okay.

**Speaker B:** Don't do this. Oh, this.

**Speaker A:** Yeah.

**Speaker B:** Yeah.

**Speaker A:** So I guess before we kind of dive into things that have changed, I guess, like, I'd be very curious to know, do you have a set of. Especially back then, like, were there things that you were kind of on that were on your mind about, like, context management? How do you, like, make sure that, like, you're not blowing out your context window? Like, I'm very curious how you kind of juggled those sorts of aspects of

**Speaker B:** this. So

**Speaker A:** I mentioned. I guess it's worth mentioning too. Like, my personal belief is these are just sort of less important now. But I'm.

**Speaker B:** Yeah. So I've never had a lot of context window issues because I believe very strongly that agents do a lot better when they have a single mission. If you've given an agent the mission of write the code and write the tests and make sure the tests pass, they will absolutely test max.

**Speaker A:** Yeah.

**Speaker B:** But if you give one agent the task of write the tests, another agent the task of write the code, and another agent the task of make sure the test pass, each one of them will do its job.

**Speaker A:** Yeah.

**Speaker B:** And so if you, if you

**Speaker A:** split up your context across multiple.

**Speaker B:** And so I try really hard to make sure. And I get. These days I am definitely looser about it, especially when I'm, you know, just yoloing something. But it's. I try really hard to. This is what sub agent driven development and superpowers is, is it breaks tasks into one task per sub agent. They get a shortcut, they, they have minimal context, they're focusing on exactly what they need to do, and then they're done and they can go to sleep.

**Speaker A:** You know, it's funny because like, this is, this is actually a place where I think our philosophy actually differs a little bit. Because I remember I tried the sub agent thing and not just your sub agent. Like I tried a bunch of different sub agent things and I found that like the models, I really struggled with getting the models to pass the right context to the sub agents. Like they weren't good at realizing, you know, so. So if you have like a parent agent, you have a bunch of sub agents, it's lossy. Right. Like the parent agent has a bunch of context that doesn't get passed down to the sub agent. And we just, I just could never get that to work the way I wanted it to.

**Speaker B:** So that's the writing plan skill. It is, it is building these context blobs and that. And if it is writing them well,

**Speaker A:** yeah,

**Speaker B:** everything's fine. And if it's not writing them well, you're fucked.

**Speaker A:** I remember. Yeah. I think we ended up going a slightly different way for that too because we ended up having essentially these set of instructions that would do it all in one kind of parent agent. But like with very, very strict rules around, like how it was actually like interpreting the plan and like how I was actually parsing it out.

**Speaker B:** It

**Speaker A:** ended up being a little bit different. Right.

**Speaker B:** Yeah. And my experience until quite recently was they would lose the plot.

**Speaker A:** Yeah.

**Speaker B:** And they've definitely gotten better

**Speaker A:** yeah.

**Speaker B:** And so one of the things that we recently redid, the. The thing that is the alternative to sub agent driven development. The do it all in one session.

**Speaker A:** Yeah.

**Speaker B:** I had pulled that out like the better part of a year ago, and people got angry at me because it was cheaper and they wanted to use the cheaper.

**Speaker A:** Yeah.

**Speaker B:** But it's

**Speaker A:** going to take longer. Well, it gets

**Speaker B:** bad outcomes. It wasn't. It takes longer. It's that the code would be full of bugs and you don't want the bugs. Like, and we put it back because I don't like being yelled at. And now we finally rebuilt it because if you are using a frontier model. Yes. They can absolutely do it in one session without losing.

**Speaker A:** Yeah.

**Speaker B:** Yeah.

**Speaker A:** So we have about like 10 more minutes and then I want to take a quick break in the last 10 minutes. Like, last 10 minutes of this, like, first half of yapping. I'm very curious about, like, writing skills today. How is that different from like the earliest history of superpowers? Right. Like, we talked a lot about what superpowers was like, what it was like working with those older models. Obviously the models have changed a ton of. And so, you know, even coming here, like, a bunch of people were asking me this question is like, first of all, do skills still matter? Especially this sort of like broader skills where it's like, you know, these are sort of like metacognitive things. It's not about, like individual context. And second, if you had to write a skill today, how would it differ from what you were doing previously? Sure.

**Speaker B:** So I think skills. So I sometimes, tongue in cheek, describe skills as post. Post training.

**Speaker A:** Yep.

**Speaker B:** Skills are your taste and judgment. They are your way of doing it. Without skills, what you are getting is Claude's Way or OpenAI's Way.

**Speaker A:** Yeah.

**Speaker B:** And if you want the. In the weights version where everything is the same and kind of mid, you can do that. That's great. But if you have opinions and. And a way you want to do things, you still need to say that.

**Speaker A:** Yeah.

**Speaker B:** You still need to document your process. And it turns out that we have a way of doing that. Yeah. And I don't. I've heard people talk about maybe skills are gonna go away. The labs don't want skills to exist. They want a new thing. I don't buy it.

**Speaker A:** Yeah. I mean, we found. I mean, the idea of skills going away is just silly to me. Like, I mean, they eventually, like, you need a way to get your company's context into the. Into the model. Even if it's just like hey, by the way, model like this is where, you know, we, this is the Google Drive where we have all this relevant information. But there is something I found kind of interesting about like the sort of like higher level skills of like, you know, meta process. Right. Like the, an example of this is sort of like the brainstorming into writing plans into tdd. Right. I think the models have, have gotten better. We still have that implementation. But I am very curious, like from your perspective, do you feel like those metacognitive things are less important than they were?

**Speaker B:** So I'm not sure they're less important. But I will say that Opus 5.5, it can kind of do it on its own.

**Speaker A:** Yeah.

**Speaker B:** Because we're finally in the weights.

**Speaker A:** Yes. So that was, that was. Yeah.

**Speaker B:** I mean, and so, you know, some level of how superpowers works is

**Speaker A:** now in the

**Speaker B:** weights. It is finally the case that the models don't say, oh, wasn't Jesse a Pearl guy in the 90s? They're like, they know who I am now. But it's, it is still the case that they do a worse job because

**Speaker A:** without the skills,

**Speaker B:** without the skills I, you know, people keep saying, well could, you know, could you delete all the implementation skills? And just I was talking about this with one of the other big, a bunch of the process skill. The like process skill plugin people there. We all have a. Annoyingly, annoyingly, it's an imessage group. It is the only like work group of mine that is imessage. So we chat and some of them are like, we keep trying to delete our implementation skills and I keep trying it and

**Speaker A:** it's worse,

**Speaker B:** it's cheaper and it's faster and it's full of bugs.

**Speaker A:** Yeah. By implementation skills you mean like the tdd,

**Speaker B:** like the implement, like every. The executing plan, sub agent driven development. You can say you should do strict red green tdd. And if you've got an OPUS class model, that

**Speaker A:** might

**Speaker B:** be enough. But you don't need that OPUS class model. You don't need to pay those token rates. Use

**Speaker A:** a Sonnet.

**Speaker B:** So one of the things that we've burned a lot of money in on is can we take the orchestrator that it that is running the plans and knock it down to a less capable model.

**Speaker A:** Yeah,

**Speaker B:** we got it to the point where Sonnet was reliable.

**Speaker A:** Yeah.

**Speaker B:** And Haiku four. Five just kept losing the plot. And I haven't rerun it with Haiku 55 but because it's. If you can get a Haiku class model to be your Orchestrator. It's better, it's cheaper. It is the case that if you are. If you are not stuck on a subscription and you can take all of the processy parts that really are imperative and move them into your harness. That's way better. Anything that can be classical software should be classical software.

**Speaker A:** Like just like use a linter. Use a biome.

**Speaker B:** But just like break the plan. If the plan has been written out, chunk the plan and fire off an agent to do each task. Don't have an agent doing that. And it's.

**Speaker A:** It's better.

**Speaker B:** It. I mean,

**Speaker A:** it's going to be less. It's deterministic. It's

**Speaker B:** cheaper. Yeah. If you. Yeah, deterministic.

**Speaker A:** It's deterministic and cheaper.

**Speaker B:** It turns out that anything you can make deterministic, that's good. It means that the computers are like computer. The computers, we remember, they do the same thing every time.

**Speaker A:** Right, Right. So. Okay. So, yeah. So, like, you know, what are the things that you. If you're writing a skill today? Like, what are the things that are like the tips and tricks that you would have of. Like, here are the things that people should be thinking about. Here are the things that you should be actually like embedding, you know, anything that. Yeah, anything like

**Speaker B:** that. So when I write skills today, I. It is usually the case that I am having a chat with Fable.

**Speaker A:** Yep.

**Speaker B:** And it is. No, I've done. Actually I've done a bunch of it on Deep Sea Congo lately for like internal skills. For. Here's the process we've been running through. Let's. Let's automate this. Yeah. And it's. Usually it is describe the thing I want to do, describe why I want to do it and have it go try.

**Speaker A:** Yeah.

**Speaker B:** And that's. That's kind of it. Okay. I mean, it's the, like the why

**Speaker A:** that's relying on the creating skill. Skill. I guess

**Speaker B:** it's relying on the creating skills still. Which means that it's also doing pressure testing.

**Speaker A:** Sure. Yeah.

**Speaker B:** Yeah. And so it's

**Speaker A:** evaluating against the intent.

**Speaker B:** I have. Yeah. Like, I have taken my taste and judgment and I shipped it.

**Speaker A:** Yeah.

**Speaker B:** And now. And now it's there. And now I don't have to do that bit again.

**Speaker A:** Yeah. There were some things that I remember it used to be valuable to do because it was embedded in the model's pre training. So, for example, using XML tags, is that the sort of thing that you still kind of do or do you not really rely on those?

**Speaker B:** So Opus 5. So all caps in an XML tag was the thing that we needed to do in the superpowers bootstrap to get a bunch of models to pay attention. Opus 5 started deciding that was prompt injection,

**Speaker A:** because

**Speaker B:** so many prompt injections use that technique to. Because it. Because it conveyed importance.

**Speaker A:** Yes.

**Speaker B:** That anthropic trained the model. Trained the model. That this is dangerous. And it's not just superpowers that got hit. For the first week after Opus 5 came out, it Claude. Claude code kept injecting, like, context reminder. Holy shit. I think I'm being prompt injected. It says there's a context reminder. And so, like, it wasn't just us.

**Speaker A:** Yeah.

**Speaker B:** The XML stuff is less important. Yeah. You can now do more English.

**Speaker A:** Yeah. In general, do you think about other, like, you know, even Graphiz, for example, like, the idea that this was in the weights. Like, are there other things like that that you're sort of like, oh, this is actually like a useful trick to make them make the skills better.

**Speaker B:** So graph is. We go back and forth on still. Okay. Partially because I need to go look at the numbers, but I think there's some models that don't deal with it well. And so we can't only have problems. Graph.

**Speaker A:** Yeah, yeah,

**Speaker B:** but. And as we've been trying to compress the skills some, there's been a bunch of stuff we've been able to take out. And sometimes we take out the graph fizz. Because it's. If we've got to pick between English and Graph is. Yeah, yeah.

**Speaker A:** Makes sense.

**Speaker B:** I think. What, like, what else we don't do anymore?

**Speaker A:** I mean, you still have the rationalization table. Rationalization

**Speaker B:** tables. I mean, they're. Yeah, yeah,

**Speaker A:** yeah.

**Speaker B:** I think, like, what else? I mean, I

**Speaker A:** can only also imagine, like, it's interesting to me that, you know, you're writing so many of the skills now directly through Fable. Like, you're kind of able to have the agents do it. It's like, at that point, like, you might not even be. I don't imagine Fable is sort of like, thinking about those sorts of things. And it's gotten to the point where it's like, actually just able to do this.

**Speaker B:** I mean, it does. It does. Hardcore rebels. Like, when I. When I'm having it do. When I'm having it do this. Yes, it will,

**Speaker A:** you know, he'll climb against it and find it'll.

**Speaker B:** It. It will have. I mean, it will have other. Other agents. He'll climb against it.

**Speaker A:** Right.

**Speaker B:** And it, like. And it will, you know, figure out, you know, you know, this word or this line or this or this bit of prose. Right.

**Speaker A:** Problems.

**Speaker B:** I. I have finally for the most part gotten it to remember that it should resume the session and ask it, hey, why did you do that? What were you thinking? What could I have said that would get you to do it differently?

**Speaker A:** Yeah. Yeah. Awesome. Okay, we're at 7:30. We're going to do a quick break before everybody goes. I need to do a quick advertisement for Nori. This is a requirement, of course, of hosting. So Superpowers is an amazing skill set. If you want to be able to use your agents in the cloud, you can support all sorts of different skills and customizations. We support Superpowers directly inside Nori. So if you are somebody who is looking for a ramp inspect like experience or a cloud tag type experience, but you don't necessarily want to be tied down to Claude or you don't want to be tied down to a specific model provider, please get in touch. We support all sorts of different kinds of customizations. Whether that's different compute on prem, compute inside your vnet, inside your vpc, whatever. And yeah, we are really, really bullish on having agents run in the cloud and just automate all sorts of stuff for you. When we come back, we're actually going to talk a little bit about Prime Radiant. We're going to talk a bit about what Jesse kind of sees the future. Like, what are the things that are kind of like moving forward and, you know, other ways in which people are using agents today that go beyond skills and like, what are the other primitives that people who are doing agentic programming should be thinking about. We'll be back at 7:40 if that works for you guys. Thanks. Amazing. Thank you.
