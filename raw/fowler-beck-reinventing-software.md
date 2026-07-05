---
url: https://gist.github.com/6576b007d7c3f21554435b2b2923492e
date_fetched: 2026-07-05
backfilled: true
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

AI is a change of unprecedented magnitude and speed. Nothing in Fowler’s career—not objects, the internet, or agile—compares to the sheer scale and velocity of AI’s impact. The industry has no choice but to pay attention.

Nobody has the answers right now, and that’s the point. For 25 years, experienced developers could press play on known solutions. Today, the answers change week to week. The critical skill is not knowing, but figuring out how to know—running the smallest possible experiment to validate a claim to your own satisfaction.

Skepticism must be total, including skepticism of your own skepticism. Fowler’s rule: be absolutely skeptical of every new technology, but stay curious enough to probe for real signals. His early dismissive reaction to AI autocomplete would have been a mistake if he hadn’t kept listening to trusted, balanced voices like Simon Willison.

AI is an amplifier, not a replacement. Beck frames it like the circular saw for carpenters: it doesn’t end the craft, it removes drudgery. Juniors who learn fast will learn faster; experienced developers who work effectively will work more effectively. The “middle”—people who entered programming just for money—faces the biggest risk of displacement, echoing the dot-com crash but on a larger scale.

The Agile Industrial Complex will repeat itself with AI. The core ideas of agile were solid, but a massive snake-oil industry grew around them. The same is already happening with AI. Distinguishing real value from hype requires constant probing and wariness.

Incentives inside companies are often misaligned with “faster, cheaper, better.” Beck notes that people inside organizations will punish you for delivering improvements that don’t align with their personal incentives. AI promising the same things will hit the same wall.

The “re-soloing” of programming is a dangerous illusion. Beck sees a trend of replacing team collaboration with one programmer managing multiple agents. That’s not the same as the messy, social, high-bandwidth interaction of an XP team, which produced good results precisely because it was uncomfortable. Two humans pairing with genies can be powerful—the slowness of models even creates space for valuable conversation about design.

Developer experience and agent experience overlap. Fowler quotes an insight from the Future of Software Engineering conference: the Venn diagram of DX and agent experience is a circle. Modular code, good tests, and precise domain language help both humans and AI agents. This suggests existing craft practices retain leverage.

Let go of the OCD satisfaction of perfecting a single function. Beck says, with sadness, that the deep craft pleasure of getting one function just right no longer has leverage. The shift is toward enjoying overall understanding of the domain and its connection to the program.

Large enterprises are in confusion and panic, and security disasters are looming. Fowler reports that big companies with million-line codebases are struggling to apply AI safely. He’s alarmed by groups wanting to give LLMs full control over email, predicting serious security incidents this year due to inattention.

Pithy and provocative quotes

Kent Beck on TDD feedback: “Thank you so much for Test Driven Development. I also get this Test Driven development ruined my life, my dog left me, my house burned down, and it's all your fault.”

Kent Beck on the current state of knowledge: “People want the answer and the answer is changing. So you can't possibly, in this environment have the answer now. That's the bad news. The good news is nobody else has the answer either. So you're just as smart as everybody else because you're just as ignorant as everybody else.”

Kent Beck on organizational incentives: “It turns out that people don't want faster, cheaper, better. … Inside a company, the incentives are so misaligned with actually achieving that. … People will punish you for that if that doesn't align with their incentives inside of organizations.”

Kent Beck on junior developers: “AI is an amplifier and if you're young and learning quickly, AI is going to amplify that or can amplify that. So I personally think this is the golden age of the junior programmer.”

Kent Beck on losing the craft high: “I need to let go of that because that satisfaction of getting this one function just right just doesn't make a difference anymore. … I can't do that anymore. But I can still develop an overall understanding of what I'm doing.”

Martin Fowler on skepticism: “My skepticism has to be absolute and total, which means I have to be skeptical about my skepticism.”

Martin Fowler on security risks: “I'm very much concerned we're going to have some really bad security incidents over this year because people are just not paying attention.”

Martin Fowler quoting an insight on DX and agents: “The Venn diagram of developer experience and agent experience is a circle.”

Kent Beck on the value of small experiments: “What's the smallest experiment I can run to verify to my own satisfaction … whether this claim is true? That's the skill that is suddenly in the last year become a thousand times more valuable.”

Kent Beck on the illusion of managing agents: “Instead of having 50 people on my team, I have five people on my team. They don't have to talk to each other, and they can each have 10 agents. And that's the same. It's not the same.”

Tools, practices, and methodologies

Test-Driven Development (TDD): Writing tests before production code to specify and verify behavior. Beck and Fowler note that TDD is now critical for verifying AI-generated code—you need tests to ensure the “genie” does the right thing, and the discipline of TDD over 20 years prepared the ground for this.

Smallest possible experiment: Running the least-effort test to validate a claim about a tool or technology to your own satisfaction. Beck says this skill has become a thousand times more valuable in the last year because answers change constantly; it’s how you navigate uncertainty without waiting for someone else’s answer.

Skepticism with curiosity: Maintaining absolute skepticism toward any new technology while simultaneously being skeptical of that skepticism—probing for real signals even when initial impressions are negative. Fowler used this to overcome his early dismissal of AI autocomplete by reading balanced sources like Simon Willison’s blog.

Pair programming with AI (two humans + n genies): Two developers working together while interacting with one or more AI agents. Beck reports positive experiences: the slowness of current models creates gaps for human discussion about naming, conditionals, and next steps, preserving the social benefits of pairing while leveraging AI.

Precise domain language for agents: Developing a rigorous, shared language to communicate about the domain with AI agents. Fowler cites colleague Unmish Joshi’s practice: building a precise vocabulary makes interactions with the genie more efficient, echoing domain-driven design and model-building techniques.

Modular code and good tests as agent enablers: Structuring code into well-separated modules and maintaining a strong test suite. Fowler reports feedback that these practices make AI agents more effective, reinforcing that what’s good for humans is good for agents.

Extreme Programming (XP) social practices: Creating a safe, high-interaction social environment where programmers talk to each other hours a day. Beck warns against abandoning this for solo agent management; the messy, social, complicated process produced good results, and replacing it with isolated programmers and agents is “not the same.”

Refactoring (implied): Improving internal code structure without changing behavior. Not discussed directly as an AI practice, but Fowler’s emphasis on modular code and tests aligns with refactoring as a foundation for agent-friendly codebases.

Mob programming with genies (speculative): The whole team working together on one task, potentially combined with AI agents. Fowler wonders if this could be effective but offers no reports or conclusions.

Unanswered questions and omissions

How do you actually integrate AI agents into pair or mob programming? The speakers mention the idea positively but provide no concrete patterns, workflows, or pitfalls.

What should the “middle” of programmers do to avoid displacement? Beck raises the concern that the large cohort who entered programming for money may be flushed out, but offers no guidance on how they can adapt or transition.

How do you mitigate the security risks of giving LLMs control over sensitive systems? Fowler flags the danger of AI reading and replying to email as “mind boggling” and predicts serious incidents, but no defensive strategies or design principles are discussed.

How do you distinguish real AI value from snake oil in practice? Both acknowledge the problem and the need for probing, but no heuristics, questions, or evaluation frameworks are offered.

What does “code” become when prompting and agent interaction replace traditional writing? Fowler says the nature of code may radically change, but doesn’t speculate on what form it might take or what skills will replace coding.

How should engineering leaders manage AI adoption? Advice is aimed at individual engineers; the leadership perspective—budgeting, team structure, risk management, cultural change—is absent.

How should CS education and training adapt? Beck calls this a golden age for juniors, but the implications for curricula, mentoring, and skill progression are unexplored.

What are the ethical, bias, and intellectual property implications of AI-generated code? The conversation stays entirely within the bounds of developer effectiveness and organizational dynamics, ignoring these broader concerns.

How do you preserve collaboration and social safety in an agent-heavy world? Beck criticizes the “re-soloing” trend but doesn’t offer a counter-strategy beyond mentioning pair programming with AI; the systemic forces pushing isolation are not addressed.

What happens to open-source and the concentration of power in AI providers? Not mentioned, despite the potential for AI to reshape how software is built and who controls the tools.

How do you measure productivity or effectiveness with AI? The talk assumes AI will make people faster and better, but doesn’t question how to measure that or whether current metrics break.

Speaker B: Welcome, everyone. It's so nice to see all of you. It's so nice to see a lot of friendly faces. A lot of you said hi. And also just really good to meet Martin and Kent. And I was joking a little bit beforehand that I did not expect Martin Fowler and Kent Beck to walk into a place where it's all the kind of the hottest AI startups and all of them, but here we are. And we're here for a very, very good reason. What?

Speaker C: What? What the hell is that supposed to mean?

Speaker B: Oh, gosh, here we go. So Kent is gonna hate me for this because today I already called him old furniture once.

Speaker A: I think that's fair. Old furniture, yeah. You're the old guy in the crowd. I'm at least a year younger than him,

Speaker B: but I'm psyched that both of you are here. And a week ago, me, Martin, Kent and a bunch of other people were in Deer Valley, Utah in the Future of Software Engineering conference that Martin pulled together some very interesting thinkers. And we were talking about, like, it was nice to reflect that 25 years ago the Agile Manifesto was created,

Speaker A: There

Speaker B: were 17 people and two of you were there. Since then, you've really helped shape software engineering as you've helped influence and you've had major contributions. Could I ask both of you to recap what the feedback you've gotten over these years, these decades, what ideas really stuck with engineers? What do you hear? A lot they tell you like, thank you for this. I'm using a lot of this.

Speaker A: That's a really interesting question. I haven't really reflected on that. I mean, a lot of people talk about things in general that we've worked on, whether it's agile broadly or refactoring, particularly in my case, because that was a book I worked on. But I don't know anything, any of the pieces of that necessarily.

Speaker C: So I get. Thank you so much for Test Driven Development. I also get this Test Driven development ruined my life, my dog left me, my house burned down, and it's all your fault. So I don't know. Does that answer your question?

Speaker B: I think TDD has been very divisive. I've been a convert at some point and then I hated it. But it's interesting because I feel a lot of these are a bit like this. Right. They're meant to be provocative, they're meant to push you.

Speaker A: Yeah. I also got. Was actually chatting with somebody yesterday who's really pushing the AI envelope and his comment was, well, thank goodness for all of your pushing of TDD. For the last 20 years because it's really important that we've got AI agents. And it's interesting to hear that feedback because I'm always suspicious of it because I want it to be true. You know, I'm the kind of guy who, when, when I hear something I like, I'm kind of thinking, am I just making this up? But it does make sense to me that, you know, when we've got a big powerful genie, you really have to learn how to verify that it's doing the right thing for you.

Speaker C: Which we've been practicing for 25 years.

Speaker A: Yeah, well, I mean, I'm not a big powerful genie myself, but I still needed the tests to make sure I was doing the right thing.

Speaker B: So a lot of folks here, they will know you from the podcast that we did together. They might have, they probably read some of your books. Barton, you're really perfect writer, but can you tell us what are you up to these days? What does your day to day look like? How do you stay in touch with technology? I got a really rude comment that I'm still, I gotta fight into someone on LinkedIn about this. They said like, oh, your conference, like you're having authors here who are like out of touch with technology. And I'm like, do you even know who Martin, Valerie and Ken Beck is? But I just wanted to ask, like, these days, what are you up to? Seriously?

Speaker A: Well, when I finished the second edition of the refactoring book, which was five or six years ago, I toyed with the idea of writing another book. I've got several half written books out there to work on and I decided I should not do that. Instead, what I should do was work with people who are actually still doing real work on real projects, writing real code, as it were, and get them to get their ideas out and what they learn out. So that's been my main project ever since, primarily focused on the MartinFowler.com website because, hey, I control that. There's no big corporation that's going to sweep in and clobber it, at least not without my me selling out and getting lots of money out of it. And as the AI thing has come, I've been very keen to capture that. And what my big focus is on is trying to understand details of people's workflow and what exactly they are doing, what are the kind of conversations they're having with the genie, as Kent calls it. And if you're reviewing things, what are you looking for? If particularly what decisions are you? We, the humans, still making and how is that decision flow changing? So that's my interest is it's not what I'm doing, it's what my colleagues are doing and trying to spread that around. And that's, that's my focus.

Speaker C: So I've been. My personal mission in life is to help geeks feel safe in the world. And our people do not feel safe right now for some good reasons and some not good reasons. And one of the things I notice is for 25 years we've kind of had the answers. Somebody comes to us and says we have too many bugs, we're like, all right, well here's how you write tests. Oh, I can't write tests. Well, here's how you design, so you can write tests. Just kind of press play on the recorder. And the thing that's changed is at this moment, nobody knows the answers to anything. And so what I've been trying to do is both for my own geeky curiosity, satisfaction, let me go back into explore mode and find out as well as I can what you can do to be effective with these new tools and then demonstrate that to the next generation of people who've been used to getting answers. Oh, I'm having some trouble. Let me look up in the book what the solution is that worked for the last 20 years and it doesn't work for the last year and won't work for an extended period. So as seniors, I figure it behooves us to, to demonstrate not just how to use these tools effectively, but how to figure out how to use these tools effectively, because that's a whole different set of skills.

Speaker B: And you say, all right, we don't know what will come. No one has it figured out. I wanted to take you back into your professional journey. One thing that you share here, you have seen a lot more than a lot of us, myself included, a lot of people in the room as well. Do you remember a time where there was a technology change which looked similarly kind of unpredictable or scary like AI does right now? What was the thing that comes the closest in your career?

Speaker A: Well, nothing has hit with the magnitude of AI. That's. I mean, this is a whole size difference from anything that we've faced before. On a smaller scale, I would say. And we were very much involved in the growth of object oriented languages. I mean, and that scared a lot of people. It didn't scare us so much because we were part of it. I would say that the impact of the Internet had a huge impact upon us all. And of course, obviously we were spreading the challenge of agile software development. And that had a very big impact on a lot of organizations because you could tell by how hard they resisted it. But the thing about AI is that all of many of these things, we were talking about how important they were and how valuable were and trying to persuade people of the importance of them. Yes, even the Internet, that may sound surprising, but there were people who weren't thinking that was important. But I. There's kind of no argument about how important it is. People can't, I mean, you cannot put blinkers on to deny the importance of this thing.

Speaker C: So the, the other analogy that I have is to the introduction of the microprocessor. Before that, computers were a big box. You couldn't move them around. If you wanted another one, you'd mortgage your house. Again, it was a big deal. And I was a kid in Silicon Valley with my dad as a programmer when the Intel 4004 hit and we went, wait a minute, that's a computer. Oh my goodness, the possibilities suddenly expanded. If you can figure out how to write software, if you can figure out how to design hardware around this thing, you can suddenly do things we can't even imagine. And so I think part of AI is this expansion, expansion of imagination. So I'm writing projects that are ridiculously ambitious. I'm working on a persistent small talk. I'm writing library quality code for Rust. I'm just, you know, anything I can imagine to trying to do, I'm going to try and do it. And see now a bunch of those fail. And that's fine, that's part of this process. But it's not like this is the first time the heavens have opened and we've been brought tons of new opportunities

Speaker B: and back, either with object oriented spreading or with microprocessors. Do you remember what the feeling was in the industry and what was the difference between experienced professionals who, you know, just got thrived in this new world and ones who were just honestly left behind?

Speaker A: Yeah, with all of those, there was that sense of the mix between the people chasing the hype and the people who were saying, no, this is nothing special. I think you've always got to have that balance of skepticism and curiosity in order to be able to do it. And you are selective about it. I mean, I have been completely skeptical about some big changes. I mean, blockchain, for instance, I was extremely skeptical about that. But as I like to say about my skepticism about technologies, which is well rooted because I've seen so much snake oil projected out by the industry over the years my skepticism has to be absolute and total, which means I have to be skeptical about my skepticism. And that requires that curiosity. And I think that's where the thing is. You've got to be curious enough to say this looks like, but maybe it isn't. How do I probe in order to detect that there's signs of something coming out? Fair.

Speaker C: Yeah. What's the smallest experiment I can run to verify to my own satisfaction? And everybody's level of satisfaction is going to be different whether or not this claim is true. That's. That's the skill that is suddenly in the last year become a thousand times more valuable is that skill of saying, what's the least I can do to validate for my, to my own satisfaction whether this claim is true?

Speaker A: But there's also another step in there that you also got to be aware that your early interactions may not actually be a true signal. I mean, when I started playing around with AI, I guess it was the copiloty, like stuff about a year, year and a half ago. I was pretty unimpressed. Right? I mean I set up. I'm an Emacs guy. I set up Emacs, the one true editor. I set up Emacs so that I could just have it prompt and complete automatic. You know, Emacs is capable of doing that. And I used it for maybe three or four days before I just got. Because sometimes it would give you something wonderful, but most of the time it gave you such garbage that you would just control K right away. And if that had been my impression of AI and I said that's what I think of AI, I would have just immediately flipped the bozo switch on it, just like I did with Blockchain. But on the other hand, I'm also probing out there. So my most valuable discovery in all of this is in the room next door, Simon Willison's blog, which I read. And one of the things that I took from that was to use this tool well, you have to learn how to use it well. Which was also something very true of object orientation. People would say, oh, objects are. And you'd look at what they were doing. You're not using objects very well. In fact, we kind of were kind of at yeah, they were using C and Java. They didn't actually do the real stuff. But the point was you have to be also listening to the folks out there and being able to read with a critical eye and getting a sense of okay, if you do run across a Simon Willison, is he hyping everything Wonderful. Or does he seem to be recognizing real problems at the same time and giving me straight stuff? And that, I found was a really, when people give you that balance of good and bad, and also, most importantly, are prepared to say, I don't know, then that involves something to listen to. So him and also some of my colleagues in ThoughtWorks like Mike Mason and Boogie Taboukla, they really kind of showed me that, oh, I shouldn't be relying too much on my initial reactions.

Speaker C: Yeah. And it can change week to week. I'll, I'll try something with Gemini. One week fails miserably. This Gemini thing, cloud code, then that works pretty well and then it doesn't work well. And then I tried Gemini for the same thing and it works this week and it didn't work last week. That's a. You know, people want the answer and the answer is changing. So you can't possibly, in this environment have the answer now. That's the bad news. The good news is nobody else has the answer either. So you're just as smart as everybody else because you're just as ignorant as everybody else.

Speaker A: Is that reassuring?

Speaker C: Is that reassuring? By a show of hands,

Speaker B: One thing that struck me as like a bit of a similarity is back in 2001 when the, almost exactly 25 years ago, when the Agile Manifesto came out with that website with all the 17 names listed and Kent Beck being the first one. Why were you the first one?

Speaker C: Alphabetical. Strictly alphabetical, but it is a source of unending joy.

Speaker A: You can very much take a. That.

Speaker B: That kicked off some really interesting things in the industry because what my interpretation was like, well, use this Agile. Here's these four pretty simple, easy to understand and easy to identify with things to build better software, faster, cheaper, higher quality, you name it. Now, when I think of why so many companies are adopting AI, they're kind of expecting the same thing. Better, faster, cheaper, and so on. And so I wanted to. Can you reflect on how Agile actually went, Speaking of snake oil?

Speaker C: Well, it turns out that people don't want faster, cheaper, better.

Speaker B: Tell me more.

Speaker C: Inside a company, the incentives are so misaligned with actually achieving that. And so as geeks trying to achieve that and say, well, it's 40% better and it's 12% cheaper and it's less fattening. People will punish you for that if that doesn't align with their incentives inside of organizations. Yeah, in the ideal organization, everybody would care about the same things. And that's just not the way it works. And we haven't touched that problem. So if AI is coming along to promise the same things, we're going to see exactly the same reaction.

Speaker B: And this is what I wanted to ask, looking back from what you've seen for agile, now 25 years, and it played out at a loss, slow, slower pace. What similarities do you see right now with AI? How do you think the curve could fit? And also what is very different about that agile movement? And that took the industry by like a very slow storm. And now with AI, well, what's obviously

Speaker A: very different is the sheer magnitude and speed that is hitting with AI. So that is definitely different. I suspect one thing, I think there will still be some similarities. One of them I think is there will be a big difference between people who use it well and people who use it badly. And the trick is figuring out how to use it well and putting the effort in to learn to use it well. I think there will be a big distinction between those two groups. I think another similarity is, I mean the core notions behind Agile and extreme programming are solid and good, but a huge snake oil industry appeared around it, the Agile Industrial complex, as I like to refer to it. And that will happen, that is happening with AI right now. And it's often hard to see the difference between where is the snake oil and where is the real stuff. And so that's another thing that you've got to be constantly probing and be aware of and be wary of as you're looking at it.

Speaker C: Yeah, AI is an amplifier and if you're young and learning quickly, AI is going to amplify that or can amplify that. So I personally think this is the golden age of the junior programmer. I get people coming to me all the time. My son started his second year in CS and he wants to go into something more commercial like art history. And I'd say this is like if you're a carpenter and they just introduced the circular saw and you think, oh well, carpentry is over, anybody can build a house now. Well, no, you have more powerful tools, you have less time that you have to do, you know, kind of the crummy work. So I think that that's the young people who are learning fast are going to learn faster. The experienced people who are working effectively are going to work more quickly and more effectively. And my concern, and this is something I learned last week, that middle, if we look back at the dot com crash, there was also a middle of people who'd gotten into programming because it was a way to make money. And those people went into real estate more or less. And I don't know where the middle is going to go now because that middle is much bigger now than it was 25 years ago.

Speaker A: But that middle has also been flushed out to some degree by the retrenchment in the software industry, the end of the zero interest rate period. So that's an interesting difference because we've had these two things occurring at once, the AI boom and the economic headwinds that we've had in the last two or three years, which is an interesting kind of mix of things that wasn't the case back in the 90s with the dot com boom because that was pretty much all solid boom.

Speaker B: Yeah.

Speaker C: So another interesting confluence of factors is we have these periodic we get to get rid of all the programmers, woo

Speaker A: hoo, cobol, starting the programming, starting with

Speaker C: cobol, right when the business analysts were going to be able to write the programs and we didn't have to have programmers anymore. And so that comes back repeatedly. Agile was definitely not that we wanted programmers to be more effective in their jobs. And since we started it and were programmers, we were able to push that agenda pretty effectively. But now we have this repeating, hey, we get to get rid of all the programmers, which it behooves us as programmers to think about why they keep wanting to get rid of us. Some of that's about us and some of it's not, but some of it is. So we should think about that. But also that amps up the fear factor that everybody is experiencing.

Speaker A: And also one of the interesting things is when people say, oh, we're getting rid of code. I mean you hear people saying sessions, oh, no one's going to write code in six months time. I go to myself, well, yeah, but what do you mean by code? Because that kind of implies nobody's writing anything. Well, we're at least doing some prompting, we're having some interaction with the genie. What's that going to be if it's not some form of code in some way? I think the nature of what code is is going to be quite possibly very radically different. But I think there is still a need to produce it and be able to interact with it in some way.

Speaker B: One thing I really like and respect about both of you is you are two feet on the ground. We've heard from OpenAI, who are a leading lab, and of course they're building amazing technology, but they also have to talk their book. We've heard about Laura, who talks with so many people and still has so many good insights. In the end she will have a small bias to help sell some of the tools that help do this. Martin, you're talking with so many companies, especially like large skeptical companies as well as well as startups as to thoughtworks, consulting, advising, so do you Kent, what do you see on the ground, what is interest and what is surprising you about how these smaller and larger companies often the more enterprise y ones, the more traditional ones, what are they doing with the technology and how are they thinking about it

Speaker A: at the moment? Large scale confusion and panic is pretty much the order of a day right across the board.

Speaker C: So if that is your strategy, you're right in the middle.

Speaker A: I mean enterprise, I mean large enterprises have this thing where they just have enormous amounts of code and complex systems that fit together in difficult to give ways. Where someone says you know, can these tools handle a million lines of code? Because that's a smaller code base as far as many of these systems are concerned. And it's a very different picture to the startup world because no, you do not want to take a risk that's going to cause your airline to go offline for a day or two. That's not an acceptable thing to consider. And also there are other risks involved. I mean I've now run into several different groups including some at surprisingly large companies that are talking about let's have the LLM have complete control over my email. It can read all my emails and it can reply to most of the emails and I'm going no. What I mean the security risk of that is mind boggling but I am very much concerned we're going to have some really bad security incidents over this year because people are just not paying attention and those are the kinds of things that are out there as well. So there's a kind of blind rush to say let's grab these nice looking things at the same time as some real concerns that are coming across it as well.

Speaker C: So I see a big trend is the re soloing of programming where a big part of extreme programming is creating a safe social environment for basically antisocial people, not just asocial anti social. And when I think about the degree of interaction on an XP team, people are talking to each other hours a day and happy to be doing so because it's set up for that to be a positive experience. What I see now is well I'm a programmer and I've got six agents so really I'm managing a team. No you're not. You're using six tools at once which is fine, but that's very different than Having a conversation with somebody who believes things that are a little different than what you believe, or somebody who's got a different energy level today than you have. But I see the trend is. Oh, good. We used to have programmers, you remember, individual offices. We'd have offices and doors, and you shut the door and you slide the pizza under.

Speaker B: It was a thing.

Speaker C: And that was easy to manage and easy to control. And then along came this messy, social, complicated, chaotic process. It just happened to produce really good results, and that was uncomfortable. Oh, good. I can, you know, Instead of having 50 people on my team, I have five people on my team. They don't have to talk to each other, and they can each have 10 agents. And that's the same. It's not the same.

Speaker A: Yeah, that's the. I mean, it's part of the question that. And again, I think something that was discussed last week, you know, are we seeing that, you know, two pizza teams are going to become one pizza teams because agents don't eat pizza? Or do we see two pizza teams staying but just being able to be much more effective and capable?

Speaker C: Or do you create a genie that can eat pizza? That's the one.

Speaker A: My bet is on the more effective two pizza teams. And it's also some interesting feedback we're beginning to get in terms of pair programming. I mean, with pair programming, do you say pair programming is the human and the genie, or is it two humans and n Genies? Because if it's two of us, we can control the genies perhaps a little bit better. And we also have that same interaction. And I'm going to find it very interesting to hear reports of people trying that kind of route where they're saying, yes, we have those pairs controlling Genies, and possibly beyond pairs. I mean, there's also the whole mob programming thing and whether that will go a route with that combined with the genies, that combination. But I don't necessarily think one person, many genies is necessarily the right answer.

Speaker C: It's the simplest thing to understand. It's the simplest framing. But my experience pairing with two humans and a genie or multiple genies has been very positive. And the fact that they're kind of slow is really nice. So every time the models come out and they're faster, I'm like, oh, there's less time to talk. You give a prompt and it's like,

Speaker B: oh, well, blah, blah, blah.

Speaker C: And then it's gone for three minutes. And we can talk about our philosophy of naming or, you know, how do we express conditionals or what other you know, what should we be doing next? And if it pops back in 15 seconds, you don't have time to have that conversation.

Speaker B: So a lot of things are changing. As a closing question before we head over to Q and A for software engineers who really care about the craft, or engineering leaders who care about the craft, and they've learned to love this industry, they're seeing a lot of things are shifting. For example, you're not writing the code, you're losing a lot of control. What advice would you give to them to stay afloat and hopefully come out thriving from this change in abstraction? Basically,

Speaker A: I like to think of a comment again that came up last week and I can't remember who I shouldn't attribute it to, which is that the Venn diagram of developer experience and agent experience is a circle. And the point here is that what we do that's good for the agents is good for the humans and vice versa. I'm hearing a lot of feedback saying, yeah, actually if you have, well, modularized code, that actually makes it easy for the agents to work with. And we're already getting lots of reinforcement saying, actually focusing on tests, good tests, helps the agents as well as helps us. So I think there's a good bit of potential overlap here. And again, this could be me again, just wishful thinking because I want it to be true, but I'm going to run with it for a bit at least, and I think so focus on those craft things and focus on using that and teaching the agent, as it were, and working with the agent to find and find out how best to express that. One of the things that I found really fascinating was talking with another colleague of mine, Unmish Joshi, about how he was working in domains and he says the way he finds working often with an agent is to try and develop a language, a precise language to communicate about the domain with the agent. Which is basically the kind of model building, language building, domain driven design stuff that we're used to doing. But it makes him more efficient to talk to the agent. So those kinds of things give me a sense of. There's definitely a huge overlap between what is good about our practices and what will be good continuing to drive with AI.

Speaker C: So I think for me, I take a kind of OCD enjoyment in the, in the craft and I need to let go of that because that satisfaction of getting this one function just right just doesn't make a difference anymore. Getting an overall understanding of what's going on. And I say this with sadness because I really enjoyed getting in the zone, and you got some file, and it's a big mess, and you make tiny little safe steps, and you don't know quite where it's going, and then you start to get a glimmering, and then it's there, and then. Oh, pop. It just pops into focus and it. Oh, that feels so good. And I can't do that anymore. But I can still develop an overall understanding of what I'm doing. And I need to shift my focus to enjoying understanding the domain and its connection to my program in a way that I used to be focused on the program as the domain, and I could make that better and better. It just doesn't have leverage.
