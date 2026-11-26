---
url: https://ai.engineer/code/2026/talks/justin-mccarthy
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

6:34:30All right, my name is Justin McCarthy.

6:34:33I'm the founder and CEO at Diffusion,

6:34:36and we are diffusing the capabilities of the model into the global economy.

6:34:41It's a big place out there.

6:34:43There's $100 trillion of it.

6:34:44There's 8 billion of us, so there's a lot to do.

6:34:47And that's what we're doing.

6:34:48And one way we're doing it is by building software factories.

6:34:52This talk actually, you know, I guess it's got the factory thing in the name.

6:34:57It's not going to be super technical.

6:34:58There's a QR code at the end that is going to have some technical stuff for you.

6:35:02So I would encourage you to pick up or have your agents pick things up.

6:35:06I'm sending a lot of dual channel messages these days where I say the human stuff up here

6:35:10and then I attach something for the agent down there.

6:35:12So that's kind of this presentation.

6:35:14I'm going to say human things to you and then your agents are going to pick up the agent channel stuff at the end.

6:35:19okay so so this started for me in July of last year we were on the Claude four series of models

6:35:27and you know I had the privilege of being able to dive very very deep and really ask what happens

6:35:34when you begin looping these things so that was the story of the beginning of the software factory

6:35:41that I built together with the team and I'll say that you know even on the four series of models

6:35:46even as we were awaiting GPT-5 and the initial codex harness,

6:35:51even then we could see glimmers that this thing was going to work,

6:35:55but at tremendous token expense

6:35:57and only in the presence of a real Rube Goldberg contraption.

6:36:01So only when the loops were very duct taped together

6:36:05and very sort of handcrafted

6:36:08would you be able to see these glimmers of some kind of an output

6:36:12that would stabilize the input,

6:36:14that would lead to the kinds of reliability that we would ultimately want.

6:36:19So I'll just say that feeling, if you judge from the number of software factory talks

6:36:24that have been present here at the conference,

6:36:26that feeling I think is what we know now as software factories,

6:36:29and it's sort of accepted in the zeitgeist that at least some part of this

6:36:33just kind of happens now,

6:36:34and it requires substantially less of our creativity and input to achieve.

6:36:40So that's, of course, the time for me to say,

6:36:42please stop building software factories.

6:36:44Right. So so I and I'm, you know, obviously, you know, a little bit tongue in cheek here, but but also serious.

6:36:53I think you really should stop subjecting your ideas and your ambitions and your goals to a factory process, you know, because most of the time we don't know what we want, or at least a lot of the time we're creating something new.

6:37:09we don't yet know what we want. And if we're trying to industrialize something where we don't

6:37:14know what we want, we're certainly, we're not going to get the right thing or we're going to

6:37:18take too long or we're going to spend too much. We're going to have all, it's going to be sad in

6:37:22multiple ways. Okay. So what I would say is let's begin to use some discernment about whether we're

6:37:29discovering product market fit, we're breaking new ground, we're conquering new territory,

6:37:34or whether we are doing something that, you know, already has a precedent and is already working, okay?

6:37:40And that discernment about sort of what way, what mode we're working in, you know, I find it really helpful

6:37:48because the great thing about a brownfield factory is it's something that we're already being paid for.

6:37:54It's something where customers are saying with their dollars and their utilization, like, yeah, this works for me.

6:38:00I like it. I want more of this, okay?

6:38:01It's the existing things that keeping lights on, paying the bills, keeping customers happy.

6:38:07And you don't need to discover anything about it because everything you need to know to loop the stability of that system is just built into the runtime.

6:38:16So everything that's happening, all the ways your customers are using it, everything they're saying about it, maybe even the support tickets,

6:38:22those are reinforcing the integrity and the vitality and the utility of whatever you've got in that brownfield.

6:38:29Okay, and then of course we're gonna talk about this but

6:38:33Another exciting thing as we heard from many of our other speakers

6:38:37You know Jared just talked all about rewriting bun and cloud code

6:38:43You can rewrite a lot of that brownfield of course

6:38:45Okay, so this becomes all of that technical debt all those sort of boat anchors that are slowing us down historically

6:38:52All that, you know impossibly long backlog

6:38:55We can see those drying up now

6:38:56and we don't have to spend much energy asking what the customer wants because

6:39:01the customer is arriving every day and they're saying what I'm using is what I

6:39:05want.

6:39:12Okay so we know what to build, we build the existing thing and a couple

6:39:21of notes about just ways of thinking about working with your brownfield and

6:39:26cleaning it up and making it self-sustaining. At Diffusion, we use the word promise to represent

6:39:34some guarantee, some aspect, some priority, and some dimension within the product that we value.

6:39:41Feature is a promise. The cost and price of a thing is a promise. The security of a thing is

6:39:48promise and it's rare that you want to revert and regress on your promises.

6:39:53Okay so if you can start talking about the ways that we have expectations that

6:39:59are

6:40:00against an existing product, you can begin to encode those in loops.

6:40:05And a loop, just like any of the factory schemes we've been discussing, whether represented in a YAML file or some Python or graph, a loop has the characteristic of somehow taking an input and an output and then somehow feeding it back.

6:40:21Okay, as many of you know, this is the sort of idealized block diagram of an amplifier.

6:40:26And if you cut that feedback leg, you're just going to amplify noise. Okay, and I think we all have this feeling now that we've been working with software factories and working with loops.

6:40:36There has to be some way of feeding back that fidelity into the input of the system. But if you do it and you tune it right,

6:40:45it can work, right? So this is what we should strive for, especially in a brownfield. We're

6:40:51getting all the signal we need to cure, stabilize, and improve those loops.

6:40:58The next attribute of a brownfield factory is, you know, it really should be continuous.

6:41:05If you look at the history and the before times of when commits would arrive,

6:41:10there's some seasonality to those commits. There would be a daily and hourly seasonality,

6:41:14there'd be a weekly cycle, and you would see those commits come in at intervals.

6:41:20It can happen continuously now. When your loops are being maintained by your agents in a factory,

6:41:26I don't know, like it might look like a straight line. Just a solid list of activity that's

6:41:32happening 24-7 because there's always something to worry about. Is it a little bit slow? Is a

6:41:38a little bit, whatever, all those worries apply around the clock.

6:41:42And if you're building your brownfield factory the right way,

6:41:45it should be compatible with continuous intelligence,

6:41:49continuous application of this question, and it should just run.

6:41:54So you've got an existing code estate, you've got an existing set of promises,

6:41:59just make it continuous, make it loop, and make it keep those promises.

6:42:03So another way to summarize that is, my brownfield factory

6:42:07on an existing set of promises.

6:42:09It should just be continuous.

6:42:11It should be background

6:42:12and should be keeping those promises for me.

6:42:15Okay?

6:42:16So I'm a strong advocate

6:42:18of building a system like this.

6:42:21And it's going to give you

6:42:23the freedom and the creativity

6:42:25to begin asking

6:42:26what you should be doing in the green field.

6:42:28You know, probably not with the same level

6:42:30of looping rigor,

6:42:31probably with more true product market fit

6:42:34and discovery sort of energy.

6:42:35Okay?

6:42:36All right.

6:42:37One other really fascinating property, this is certainly something that I've experienced over the last year.

6:42:43I mentioned earlier, you know, a year ago it would work, but there was a lot of componentry.

6:42:49Getting it to work was still very much engineering.

6:42:52As we heard from some others, from Zach and from Potato, these systems are getting a little bit easier,

6:42:58and a lot of credit goes to the model itself, as well as the harnesses.

6:43:02But one thing you can expect from your factory is that it is going to simplify over time.

6:43:11Okay?

6:43:11This is something that we've seen in our skills.

6:43:14The skills are getting shorter, of course.

6:43:15The harnesses are getting more general, of course.

6:43:18And a lot of this, again, is credit to the models themselves.

6:43:22I'll note that there's a famous AI company that also makes these widgets.

6:43:28And this is kind of a famous progression.

6:43:31So the Raptor 1 engine from that AI company was really fiddly, but it worked.

6:43:36Okay, it looked like this. There were a lot of tubes, but it definitely propelled

6:43:42the rocket. Okay, in that same package, right, that same approximate external

6:43:47dimension, the second version had higher performance and fewer moving parts. Okay,

6:43:52the Delta between this one and this one was a tremendous application of

6:43:57intelligence, okay, like literal rocket scientists and engineers, focusing on how can we make this

6:44:02thing simpler, okay. You know, that's hard. It's hard to get those people, and it's hard to

6:44:07concentrate them on a difficult problem. We have tremendous allies in this simplification,

6:44:12of course, because we have Astra and Fable and Carbon, right, and so they are trying to take our

6:44:18factory from version one to two to three, so that by the time, you know, we come back, we come back

6:44:24and start our jobs on Monday or whatever,

6:44:26maybe we have a much simpler factory

6:44:29than when we started, right?

6:44:30Or we wait one more model version

6:44:32and our factory has fewer moving parts

6:44:35yet twice the performance, okay?

6:44:38So this is a really interesting feature of this moment

6:44:40that hasn't existed before.

6:44:41So if you can just barely get the thing to work,

6:44:45you can expect a smoother, laminar flow,

6:44:50higher performance soon, okay?

6:44:52without much of your creative contribution in between.

6:44:58Okay, so let's make a brownfield factory.

6:45:02A great instruction to give that factory

6:45:05is the same but faster.

6:45:09I've been in very few meetings

6:45:10where someone debated, you know,

6:45:13increasing the performance of a system, right?

6:45:15As long as you are at some sort of a Pareto equilibrium

6:45:20where you're not trading faster for cheaper or trading safer for faster, okay, then this is just good, okay?

6:45:28So it should be faster, it should be cheaper, it should be safer.

6:45:34Note that sometimes we include better in this mix, and better may be difficult to achieve in a brownfield factory.

6:45:40Better may involve more of your intent, better may involve more of your judgment and taste.

6:45:45So maybe that's where you go into discovery and use different techniques.

6:45:48but gosh, faster, cheaper, safer,

6:45:50especially on the other side of the vulnpocalypse.

6:45:52Sorry, other side, still at the beginning

6:45:53of the vulnpocalypse.

6:45:54Like we definitely need these things

6:45:56to just autonomously get safer, right?

6:45:58Okay, so you should have that.

6:46:01Okay, so I think everyone should have

6:46:04the brownfield factory.

6:46:05You should have it for your existing commitments.

6:46:07You should have it.

6:46:07It should become better, faster, and cheaper over time.

6:46:10I find that this takes like a week.

6:46:12So if you haven't spent that week yet, just do it.

6:46:15Just like whenever you get home,

6:46:16like go, go, spend a week.

6:46:18say you're on vacation next week and just stand up your brownfield factory on all your existing

6:46:23commitments. If you don't have a way to do that or you feel like you need help, you can always

6:46:27reach out to me and we're standing by to help. So brownfield factories, you should do them, but also

6:46:33too easy. Okay. Like, come on, too easy. Okay. You need to elevate your ambition as well. Okay.

6:46:42So in my remaining couple minutes here, we're going to talk about how to elevate our ambition.

6:46:48Okay, so one of the things that I think we are all struggling with at this moment is, gosh, there's a lot of code coming out.

6:46:55Even worse, it's working.

6:46:57That's a lot of stuff.

6:47:00It's maybe too much stuff.

6:47:02We maybe have a glut of stuff.

6:47:05And it could be the case that our customers didn't want all this software.

6:47:09They actually weren't asking for software.

6:47:12They were asking for more gross margin.

6:47:13They were asking for more customers.

6:47:15They were asking for more brand differentiation.

6:47:17Okay?

6:47:18And just shipping them more windows to click on and buttons to click on, that's not doing it for them anymore.

6:47:25Okay?

6:47:26And this is where all of us are beginning to see the emergence of our new job.

6:47:33Okay?

6:47:34We can communicate with the models quite well.

6:47:36That's why you're here.

6:47:37Right?

6:47:38There are other people in your firms who are not as fluent in communicating with these things.

6:47:43Okay?

6:47:44So that means, like, we have a new job now.

6:47:48we have to figure out how to build a demand factory because our brownfield factory can't

6:47:55build the moat, right? All it can do is keep our existing promises running well. That's great

6:48:00because it allows us to focus on sort of outside the building and look toward the market and look

6:48:04toward the future. But gosh, we need to start working on the demand factory, okay? I don't

6:48:11know what a demand factory is, but it probably looks something like this. It probably looks like

6:48:18the thing that we need so that when we boost performance in the back end here in the software

6:48:22factory, we don't end up spilling software all over the floor. And we rate match the demand

6:48:27systems to our production systems, okay? But there's a lot of discovery that needs to go into

6:48:34the demand factory. This is a new idea, okay? But I think you are absolutely the ones who can begin

6:48:40pushing your organizations to ask these questions.

6:48:43Don't wait for sales and marketing to come to you on this question.

6:48:47You can see that within the envelope of your brand,

6:48:52there's something more you can ship,

6:48:54but you're going to need to explain that to the world,

6:48:56and you're going to need to explain it using new words and new inventory

6:48:58and new product lines and so forth.

6:49:01I think your demand factory helps you ask better questions of your market

6:49:06because the agents can help prioritize what to ask next.

6:49:10I think it can help with better listening

6:49:11because the agents can synthesize the answers

6:49:14you get back from those questions

6:49:15and can help you re-rank,

6:49:17and they can help you re-rank overnight,

6:49:18and they can compare it to everything else you've heard.

6:49:21And therefore, I think they can schedule better experiments.

6:49:23How do we get the price elasticity of demand

6:49:25on this particular offering?

6:49:26You have a finite amount of experimental bandwidth.

6:49:29How do you allocate it?

6:49:30The models are going to help you with that.

6:49:31I think this could lead to lower latency responses

6:49:34where we are saying yes, faster,

6:49:36okay and customers begin uncensoring themselves and they start asking for

6:49:40more that would be great I think like weirdly you know even though our

6:49:45competitors have the same models there's some version of some new equilibrium

6:49:49where we're all doing game theory optimal competition and yet still if we

6:49:53start first there'll be some edge okay so get started now and then figure out

6:49:59what is the maximum

6:50:00extent of what I can possibly promise to the market. And that's my brand envelope. And let's fill it up. And just quick digression on brand envelope. Let's say we're Spindrift. We're selling some awesome sparkling water with a little fruit juice. I propose, hey, you know what, let's throw some caffeine in there. Like, maybe that gets by the committee. Like, maybe we ship that. And then the next week I come in and I say, like, and motor oil. Okay?

6:50:24You know, that's going to get shut down.

6:50:26But energy drink, that's a good expansion of our brand envelope.

6:50:29For every one of your companies, you have the same thing.

6:50:31Maybe you can't sell motor oil, but maybe you can sell an energy drink.

6:50:34So figure out what your energy drink is and start selling it now.

6:50:38Okay?

6:50:39So let's build your demand factory.

6:50:42It's going to have a high volume of high-quality signal coming in from customers,

6:50:46coming in from the market, coming in from any place where you can measure demand.

6:50:50Okay?

6:50:51And it's going to have a high volume of flow.

6:50:55But in order for that flow to not overwhelm change management, desire, it can't blow out the needs of your customer.

6:51:03So therefore, you need to have more customers.

6:51:06You in fact need to have more of everything, but you need to have more wallet within those customers.

6:51:11To get more share of wallet within those customers, you're going to have to sell more SKUs.

6:51:16In addition to more SKUs, you're going to have to organize them into coherent product lines.

6:51:19You're going to have more product lines.

6:51:21Okay.

6:51:22And once you've maxed out all of that, you're going to need to just acquire your competitor.

6:51:26Okay.

6:51:27And if you get started now, you're going to be the one doing the acquiring.

6:51:34Okay.

6:51:35So, uh, so this is it.

6:51:37This is Brownfield factories.

6:51:38I think you can build your Brownfield factory.

6:51:41I think you can absolutely build your demand factory beginning now.

6:51:44Uh, I also obviously think you can reboot your product management and reboot, and you

6:51:48you could wake up your executive team.

6:51:50And we are standing by at Diffusion,

6:51:52so we do have some technical resources attached there

6:51:55on some brownfield techniques that I really like.

6:51:58And yeah, I look forward to connecting with many of you.

6:52:01Okay, take care, thanks.
