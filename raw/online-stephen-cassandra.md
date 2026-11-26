---
url: https://ai.engineer/code/2026/talks/online-stephen-cassandra
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

6:57:39Hello and welcome to our AIA Engineer San Francisco talk.

6:57:44My name is Stephen Chin.

6:57:45I'm joined by Cassandra Chen, who's the creator of Tracetune framework.

6:57:51Hi, I'm Cassandra Chen. I built Tracetune because I wanted to improve PRs. I see a lot of developers

6:57:59doing great things. The speed of innovation is as fast as it has ever been with AI technologies,

6:58:05but a lot of pull requests are getting rejected. And it's not because the code is bad, it's because

6:58:11because the PRs often contain incorrect information

6:58:14or are written poorly.

6:58:17So hopefully with Tracetune, we can improve that.

6:58:20It also has the option to add a fun comic

6:58:22to help explain your pull requests in a funny way.

6:58:28What we're gonna do today

6:58:29is we're gonna go through a little bit

6:58:31about kind of the common issues of PRs,

6:58:34how to improve the quality,

6:58:35how to improve token efficiency on this,

6:58:37and hopefully help everyone to be a better AI engineer for your organization,

6:58:43like our main character, Alex, who all he wants in life is to get an AI engineer hoodie.

6:58:53That's a really neat hoodie.

6:58:54I have a lot of AI engineer t-shirts, but I don't have the hoodie yet.

6:59:00Get Swix to make this AI engineer hoodie, everyone.

6:59:06And this is enough motivation that hopefully he's going to do an amazing talk at the conference about his use case study on what he's doing in his company to improve the quality of PRs.

6:59:20Here is a picture of the token factory where Alex works.

6:59:25So Alex works together with a bunch of agents.

6:59:30He has, of course, finance looking over his shoulder

6:59:33Because we all spend too much tokens

6:59:35And they want to make sure we're keeping up with costs

6:59:38His buddy there, Pip, he's walking with

6:59:40Carries all the evidence

6:59:42And looks into the archives

6:59:44And what's actually happening

6:59:46And then Mira does all the reviews

6:59:48So she's keeping track of things

6:59:51It looks like Pip left his station empty

6:59:58Part of the factory

7:00:02Until he sees what Alex produced for a PR.

7:00:05Oh, no.

7:00:07That's a very long ledger.

7:00:10A lot to review.

7:00:14So it's hard.

7:00:16And Mira's going through everything for the fine-tuned comb.

7:00:19She actually is red.

7:00:22Found some issues with the PR.

7:00:26And this is a real PR that I think is actually one of your Tracetune PRs where it needs to have exact preview links for the URLs, command attribution, make sure the earlier work is qualified as well, and explain all of this in detail in the PR record to be accepted by Mira's very critical review.

7:00:47This was interesting because as I was building Tracetune, I was actually, like, as I was building it, I was using Tracetune to test it.

7:00:56So I would have Tracetune build the PRs.

7:00:58And as I improved the PRs, it would, like, get better.

7:01:03You got amazing comics along the way, too, just to have fun while you were building stuff.

7:01:08Yeah.

7:01:09Effective.

7:01:15So you can see that the repeated PRs, which Alex is doing to try to get good quality results, are not working out.

7:01:23Well, it's working out for his token billionaire card.

7:01:26Do you know anything about the token billionaire program?

7:01:29Yeah, so actually I am a token billionaire.

7:01:32I spent lots and lots of tokens to build Tracetune.

7:01:39That's kind of unfortunate. Was that all my money?

7:01:44Don't sweat the details.

7:01:47Okay. So I think there's a lot of good uses for tokens and for spending more AI work. Maybe doing repeated PRs is not where you want to spend your token.

7:01:58And definitely the finance department is not happy with this.

7:02:01So he wants a result, and he wants to make sure that the PRs are accurate, are working, and they don't have to keep getting repeated again and again by reviewing the logs and reconstructing the evidence repeatedly.

7:02:15And it's interesting because normally when you're just coding on your agent, you're in your session and you ask it to write a PR.

7:02:25it just reads your session log to try to figure out what it should have done using its inflated

7:02:34context window. It actually often like as it's churning it compresses its context window

7:02:40so it gets more and more inaccurate as it goes along. So instead what we're doing here is with

7:02:47Tracetune we're having it read through different sessions on your computer to try to really figure

7:02:54out what you built and what happened. Yeah, and I think like that concept of

7:03:01looking through the audit trails, looking through like what the agents have actually done

7:03:05is a lot of what you were doing with the Showtale project you were doing for education.

7:03:11Yeah, I actually started this project with this Showtale project because there was this like

7:03:17academic research study I was supporting, but basically the issue they hit is students were

7:03:24creating programs, but there was no way to trace back the actual work and effort they did.

7:03:29Because with AI coding, what you submit is the final work, the program, but most of the work

7:03:36is the prompting and everything that came before, and that doesn't get submitted with the program.

7:03:43And I use the technology and theory behind that to be like, well, I can also improve pull requests with all this data collection.

7:03:54Cool. And I think that improvement in pull requests is something that benefits everybody because now you can find the evidence, like logging the change directory or the CWD before the CD runs, explaining why maybe other test data is leaking into other repos.

7:04:11and explaining what's happening behind the scenes on the PR

7:04:16rather than what most AI PRs read like,

7:04:19which is a line-by-line explanation of the diffs

7:04:21since that's what they build the PRs off of.

7:04:28But incorrect facts can still pollute the information,

7:04:32can cause a problem with conflicting PRs,

7:04:35and then still give you back inaccurate results.

7:04:40so maybe we have a solution for this what do you what do you think you have a solution cassandra

7:04:45yeah everyone should use trace tune because of the comics so because the comics not because

7:04:53it gives you better prs right well i started with the comics and i kind of went backwards and like

7:04:59oh i need to actually make the prs better well hopefully better prs better comics and

7:05:08and maybe better token savings by the end of this.

7:05:12Maybe.

7:05:13It should be.

7:05:16So TraceDude is an open source project.

7:05:18It lets you improve the quality of your PRs by pulling in agent context,

7:05:23by pulling in information about the code,

7:05:26and then having an independent writer, independent reviewer,

7:05:30build better quality PRs, plus optionally giving you an amazing comic,

7:05:35which, what do you think the best feature is here, Cassandra?

7:05:39I think it's obviously the comics.

7:05:42This is one of the earlier comics from when I was using Trace Tune to build it.

7:05:50That's good.

7:05:51And then you made sure that there's new characters every episode,

7:05:54none of this repeating characters.

7:05:56Yeah, that's what this PR was about.

7:06:01Okay, and then this is a description of the original trace student flow.

7:06:06So it finds all the traces for a particular branch.

7:06:09It researches them, reviews the description, adds more context,

7:06:13and kind of goes in a loop until it gets all of the evidence it needs to make a quality PR.

7:06:19And subjectively but also through PRBench, which is a testing framework,

7:06:26gives you better quality, better research,

7:06:29lower rate of inaccuracies in the PRs.

7:06:33I think at this point, the PRs were already better

7:06:36than if you don't use Tracedoon.

7:06:38But I found that it still makes some mistakes occasionally.

7:06:42You can see at the end that it tries to revise with more context.

7:06:46I think that's where the mistakes come in.

7:06:50Okay, well, Alex thinks it's good enough.

7:06:52He's going to try to solve the token crisis.

7:06:54the factory with Trace2 and see if this will solve all of his problems.

7:06:58Polish checked. Alex looks very, very happy. But Mira doesn't look that happy.

7:07:06It looks like we still hit a mistake here, like I was saying earlier.

7:07:11So the original PR claim was no later passing run is recorded.

7:07:16And the problem here is in the actual Trace, it shows it running a button test

7:07:24at the same time as the

7:07:25bun x tsc

7:07:28and you can't run those two

7:07:30at the same time they have to be

7:07:31run sequentially for the test to pass

7:07:34yeah exactly

7:07:36so he claims test passing

7:07:38and all the checks were run

7:07:39and actually that's not what the

7:07:42code did so

7:07:44poor Alex another

7:07:45failed PR very angry

7:07:48badger with all the token cost

7:07:51kind of makes me feel like

7:07:53Monday morning with the Neo4j finance department looking at my token spend each week.

7:07:57But they're very forgiving, especially when it's an important project like this.

7:08:06So maybe Alex can save some costs by using cheaper writers.

7:08:09This is the number one thing finance has me to do.

7:08:12They're like, hey, no more opus, sonnet, haiku, do something cheaper, please.

7:08:17Does this work?

7:08:19I feel like the quality is a bit lower

7:08:22when you do it like that

7:08:23just reducing the cost doesn't quite help

7:08:26yeah so we were trying to get better quality

7:08:28and we ended up getting worse quality

7:08:30by just using cheaper writers

7:08:34and obviously Alex is in a bind here right

7:08:37he's spent too much on revisions

7:08:39he doesn't have a solution for reducing token costs

7:08:42he's not getting the results he wants out of his PRs

7:08:45but

7:08:47maybe there's

7:08:48something Pip can help us with.

7:08:50Pip has a friend.

7:08:52I wonder who this friend could be.

7:08:55Oh.

7:08:56Well, I think you know from

7:08:58my previous talks that this friend

7:09:00is definitely going to be

7:09:02a graph.

7:09:04You're the graph guy. We're doing another

7:09:06graph again.

7:09:08Yeah, there's always a new graph.

7:09:10But this is not the same old graph.

7:09:12This is not the Neo4j

7:09:15database. This is our new friend, Archie,

7:09:19who is a embedded database written in Rust

7:09:23called Relate that actually is being launched this week

7:09:29at the Nodes Conference and also this talk at A Engineer

7:09:32in San Francisco. And it's an open source

7:09:37database which allows us to get a lot of the benefits

7:09:40of more accuracy, multi-hop queries,

7:09:44more explainable traces,

7:09:46without having the heavyweight approach.

7:09:49So it's kind of an agent-focused graph database

7:09:52specifically for these sort of embedded use cases.

7:09:58Okay, so let's see how Archie can

7:10:00help us out. So before we were relying a lot on agents and loops of agents to resolve the problem, but graph databases can store relationships, can find different things which are clustered, build a brief which connects all this information together, and then give the agents, even a cheaper agent, better information and more context to give a good drafting decision

7:10:24And the good thing about Archie is he has zero LLM costs.

7:10:28It's all local compute costs and storage,

7:10:32and it's a fully in-memory database and REST,

7:10:34so it's highly optimized.

7:10:35So we get this providence about all these submitted changes

7:10:40and this evidence graph basically for free

7:10:43compared to all the LLM and token costs,

7:10:45which we were doing before.

7:10:47And this means our buddy Alex doesn't need to research.

7:10:50He doesn't need to go through and find all the facts again

7:10:53and go through the sources.

7:10:55because relate is going to do all that hard work for him

7:10:59and build an evidence packet that he can use for trace tune.

7:11:03It all sounds really great.

7:11:04Maybe we do need more graphs.

7:11:08I'm surprised to hear that from you

7:11:10after all the graph talks you've had to put up with.

7:11:12Okay, but we can do this

7:11:15and make it even better with a neighborhood brief.

7:11:18So basically taking all of the why, the code, the checks,

7:11:21putting attributions and relationships between those

7:11:24and then returning back a connected set of information as a packet.

7:11:28by doing this, then when the agent is drafting up the PR,

7:11:33it has all the connected facts with their sources together in a single source.

7:11:38And that makes the writer draft more accurate with fewer passes.

7:11:44And it also makes the reviewer's job much easier.

7:11:47So we don't need an expensive reviewer.

7:11:49We can save by using a few haiku reviewers instead of a sonnet reviewer or a Luna in the case of Codex to come up with the findings, check those against the draft, redo the writing, and then we retain the higher quality writer, a sonnet or tarot class writer to make sure we have a high quality brief that will pass human approval.

7:12:13When we were building this, it felt very counterintuitive to have the writer be higher quality than the reviewers.

7:12:20Actually, it's common that people always want to have the planner, the reviewer, the checker to be the highest quality model.

7:12:28But if you're doing high quality writing, a lower model, no matter how many iterations and how much feedback you give it, can't do it.

7:12:36It just turns things into a checklist and makes it very mechanical.

7:12:39So to give writing which feels human-like, you do need a higher quality model.

7:12:44And it's actually the reviewing and the cross-checking step, which is more mechanical.

7:12:48So it's a little counterintuitive, but that works out quite well.

7:12:52It's also a little counterintuitive that our badger has a thumbs up coming out of his chest, but let's ignore the high-quality AI artwork.

7:13:05Okay, so we've checked all the claims against the evidence.

7:13:08We've looked for missing findings with our cheaper reviewer.

7:13:11Did the focus work?

7:13:13And then found the things which were connected information

7:13:17to give a better draft and review for Mira.

7:13:23And here's the output.

7:13:25So it has the Y.

7:13:27So log CWD, and that describes the shell before the TD runs.

7:13:31What's the exception?

7:13:33So the CD targets in repo, or the command mentions a root.

7:13:37what passed. It has type check plus 71 tests, checked on submitted content, not before.

7:13:44And then it starts in correlate.ts inspecting the leading CD exception.

7:13:49And Mira looks pretty happy with these results, right?

7:13:54Yeah, this might be the first scene where she's genuinely happy.

7:13:59I think she was kind of happy with the AI engineer sweatshirt before she realized that Alex actually

7:14:05was a was a goof okay so here are some of the actual data and results from the side-by-side

7:14:12analysis using PR bench which is a pull request benchmark which uses audit traces plus code and

7:14:21LM judges to review the quality of the work of course reviewed by human as well for accuracy

7:14:27and you can see that on both Claude and Codex you get a significant quality improvements on a scale

7:14:33one to 100 the judges by and large almost unilaterally liked the trace tune plus relate

7:14:41quality better um on a one to 100 scale it was like 74 to 76 82 to 86 so plus two plus four point

7:14:49these tests contributed to my token billionaire card because we were running lots and lots of

7:14:56tests we created this side project called pr bench just for testing and we would have different

7:15:02agents test over different pull requests and compare the results. Initially, I had it running

7:15:10sequentially. It'll do one test and finish it, then do the next. But then we decided to run them

7:15:16all in parallel. It's a very quick way to reach high spend. Yeah, that's kind of funny. At the

7:15:24current testing rate with the number of use cases and tests on the benchmark, we can fully saturate

7:15:30my high-end gaming computer with as many threads as Linux can handle simultaneously.

7:15:37And of course, the token spend is slightly higher than what we'd like.

7:15:43But it's all for the good of improving the quality of pull requests for the world.

7:15:47So you all get the benefit of all these test runs and having the highest quality improvements.

7:15:52And you save on tokens.

7:15:55So we've spent the tokens for you in advance because Tracetune with the graph solution takes about half as many tokens as it did previously, which is an amazing result from the graph because now you're pre-computing.

7:16:10You're getting all this evidence graph constructed in advance.

7:16:14The writing's cheaper.

7:16:16The reviewing's cheaper with a higher quality result.

7:16:20I wasn't sure about the whole graph thing initially, but after seeing the cost tank almost in half, like it's not even a question to making this program work.

7:16:33and this is something we did for the TraceTune project

7:16:36but you can do this for pretty much any project

7:16:38where you have data, you have relationships

7:16:41you have evidence, you have things which you're trying to compare

7:16:44because agents are not that good

7:16:47at doing the data analysis

7:16:49at doing all of the evaluation and the curation

7:16:51so if you give them a help at the front end

7:16:54by creating a custom graph layer

7:16:57with an in-memory database like Relate

7:17:00suddenly your costs go down, your quality goes up, and

7:17:04you too can be a token billionaire like our friend Alex.

7:17:09Well, I guess now he's using it as a bookmark.

7:17:12I think that's a good use for it. It's a really high-quality bookmark.

7:17:16It's heavier than a credit card.

7:17:20It is a nice prize for spending all those tokens on PR bench.

7:17:26so finance is happy

7:17:27lower drafting costs that they can reasonably fund

7:17:31Alex is happy he has his new bookmark

7:17:34he now has a great case study

7:17:37with the connections doing neighborhood retrievable facts

7:17:41so that it's a better brief

7:17:43focusing the reviews so the reviewers get the same compact brief

7:17:46Mira gets exactly the evidence she needs

7:17:49and then here's Alex on stage at

7:17:52AI engineer. Somewhere in the world,

7:17:55better PRs, lower drafting

7:17:57costs with

7:17:58a graph plus open

7:18:01source. Trace to

7:18:02build, which also gives you

7:18:05amazing comics.

7:18:10Okay.

7:18:11So, big round of applause for our

7:18:13friend Alex, who's now

7:18:14earned the AI engineer

7:18:16hoodie.

7:18:18I think that's definitely a

7:18:20wardrobe upgrade for him. Yeah, I don't think the pocket protector look was that great for him. So

7:18:27good for him. I think we'll see Swix on stage with one of these hoodies soon. He can't not do it now

7:18:33that we shamed into doing it at his own conference. The token O'Billion program is amazing. A big

7:18:39kudos to the team for coming up with ways to incentivize and reward people who are in the

7:18:44leading edge of AI usage and learning how to make it really effective for the rest of us.

7:18:52And now you too can have better PRs impress your coworkers by having well-researched, effectively written, and lower-cost PRs.

7:19:03With a comic.

7:19:05With a comic.

7:19:06So for more information, go to tracetune.com.

7:19:09Learn about Trace2 and give it a try.

7:19:11It works with Quad.

7:19:12It works with Codex.

7:19:13It works with all your favorite AI tools.

7:19:15and if you want to do these same sort of amazing results

7:19:18on your own project, try out Relate,

7:19:21our new Rust-based embedded open source graph database,

7:19:25which brings context to your agents

7:19:27and helps you build more efficient applications

7:19:29that just work better or cheaper

7:19:32and give you better results.

7:19:34So thank you all for joining our

7:19:36AI Engineer San Francisco talk

7:19:38and if you're watching this talk this week

7:19:41at San Francisco,

7:19:43what should you do this weekend with your kids on Saturday, Cassandra?

7:19:48We are running an AI Engineer Kids Day.

7:19:51So anyone who's available, just sign your kids up.

7:19:55We're teaching kids workshops.

7:19:56Kids will be learning how to do AI programming to make their own

7:20:00games.

7:20:00And I think we're also doing a, no, it's just the gaming one.

7:20:06No, we'll do the robots too.

7:20:08Okay. Yeah, we'll also be doing robots. So kids will get to control these little soda can robots using AI.

7:20:15Yeah. So a lot of fun. Physical AI, gaming AI, all in one place.

7:20:21sign up at AI Engineer Kids Day

7:20:23and bring your kids, bring your nieces, bring your nephews

7:20:27and let's train the next generation of AI engineers.

7:20:32All right, thanks so much and enjoy the rest of the conference.
