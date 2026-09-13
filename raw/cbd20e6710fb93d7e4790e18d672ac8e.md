---
url: https://gist.github.com/cbd20e6710fb93d7e4790e18d672ac8e
date_fetched: 2026-09-13
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

Cast: Daniel Vacanti (ProKanban.org, author of the books under discussion), Colleen Johnson (CEO, ProKanban.org), Nigel Turlow (lean consultant, skeptic), Gaia Becheri (trained statistician, delivery/risk manager in insurance), and Dan North as moderator — who repeatedly abandoned neutrality to argue his own "Monty Python Simulation" critique.

1. Key Points — The Fight Was Never About Forecasting; It Was About Definitions

Vacanti's thesis: risk management and software management are inseparable, and forecasting is inseparable from risk management. He also rejects the panel's own title: "all forecasting is probabilistic in nature" — "probabilistic forecasting" is a tautology.

The method on trial: Monte Carlo simulation run over your own historical throughput data, with no assumed parametric distribution — "it's just your data." The load-bearing assumption, conceded by Vacanti: "the future can be modeled with data from the past," which he calls safe "most of the time."

The statistician's strike (Gaia): sampling your own data means assuming an empirical distribution — non-parametric statistics that "only converge when you have enough data." The field's claim that "few weeks of data would be enough" doesn't survive contact with actual statistics. And Monte Carlo itself "requires to have a distribution or a stochastic process… as an assumption" — so "there is no distribution" is not a coherent stance.

The lean dilemma (Nigel): if the system is stable enough to model, ordinary lean averaging (cycle time, takt time, throughput, process cycle efficiency) already works and Monte Carlo is redundant; if it's not stable, the model is invalid. Either way, the variation that forces forecasting comes from "management, red tape, bureaucracy" — so interrogate why you need to forecast at all.

The moderator's misapplication charge (Dan North): Monte Carlo is a population tool being used as a single-case oracle. "What's the likelihood that 10 of a hundred projects make it?" is answerable; "will this project finish by June?" is not — that's asking about a single surgery outcome before the operation.

The communication problem: percentiles don't survive the trip to the executive floor. Forecasts become targets; when confidence drops, management asks "why didn't you succeed," not "what changed?" Gaia's insurance world translates the 99.5th percentile into "1 in 200 years" because that's what humans actually understand.

Colleen's defense: a forecast is really a risk conversation — "85% by June 15th" means "15% chance it's later," and a slide to 60% is an invitation to troubleshoot (throughput? capacity? scope?). Operationally: reforecast daily, sample like-for-like periods, manage WIP rather than shuffle people.

The actual conclusion, from Vacanti himself: "fundamentally where this panel has failed is before we can even have an argument, we have to agree what we're arguing about." Distribution, stability, predictability, and stationarity were all contested and none resolved. Dan North's salvage attempt — a system can be "super spiky in a predictable way," so predictable without stable — was dismissed rather than engaged.

2. Pithy and Provocative Quotes

"It is impossible to manage software development without managing risk and it's impossible to manage risk without some type of forecasting." — Vacanti's opening position statement; the moderator called the second half "a bit racy."

"Those are made up. Those are mathematical hacks." — Vacanti, on Nigel citing Gaussian/normal/log-normal distributions. Gaia's rebuttal: "a model is a made up construct to explain reality… The point is whether it's useful or not."

"There is always a distribution… you are assuming that the distribution is actually completely represented by the past data which is called the empirical distribution." — Gaia, dismantling "there is no distribution" in one sentence.

"They will just tell me why you didn't succeed. You told me it would be 85% of probability of success." — Gaia, on why percentiles fail even with risk-trained executives.

"We never talk about 99.5 percentile with management. We talk about 1 in the 200 years event because this they understand." — Gaia, importing insurance solvency practice into the debate.

"Asking about a single data point, it's like asking about a single surgery outcome. Is the patient going to die? We don't know yet. We haven't done the operation." — Dan North, on the most common misuse of Monte Carlo.

"I have never met an adult professional outside of the stochastic academic world that could tell you what the difference is between an 80% and an 85% confidence interval." — Dan North, on the communication gap.

"I think the focus shouldn't be on getting a probability of when this work will be done. I think the focus should be on why are we having to do this to get a probability of when the work should be done." — Nigel, the lean counter-position in one line.

"I don't think you keep saying that word. It doesn't mean what you think it means." — Vacanti going "all Princess Bride" on Nigel's use of "stable."

"What's your historical precedent for when you're going to retire? What's your historical precedent for if you're going to get cancer or not?" — Vacanti's answer to "what if the future doesn't look like the past?" (i.e., not an answer).

"If I have to choose between listening to Nigel or listening to Dr. Shewhart, I can tell you who I'm going to listen to." — Vacanti, on how the disagreement was settled: by authority, not argument.

"If really what we're trying to do is help our developers spend more time coding. That's really the goal, not to have perfect dates." — Colleen, the most pragmatic line of the session.

"I don't have to justify my professionalism in statistics. I mean, you can just look at my CV… Being taught that I should go and practice with Monte Carlo and I don't know what the distribution is." — Gaia's closing, after being told to "go practice" by people whose position she'd just statistically dismantled.

"Just for the record, I haven't got a clue what I'm doing here. So there we go. I'm here to learn." — the moderator's opening, which aged ironically given how much of the arguing he ended up doing.

3. Tools, Practices, and Methodologies

Monte Carlo simulation over throughput data — the central method: sample your own historical throughput, simulate forward, get percentile-based delivery dates. Colleen's pitch: "You can do this in a spreadsheet. You can randomize your data and start to see how much variability there is coming through your forecast" — no story pointing, no hours, no points required.

Daily reforecasting — "every day that goes by is a new data point"; rerun the forecast roughly daily to see whether risk is increasing or decreasing for the delivery date. Turns forecasting into a monitoring practice, not a one-off prediction.

Like-for-like data sampling — don't "use data from December maybe when everybody's on holiday to forecast a project that you're delivering in March." Match the sampling window to the forecast period.

WIP over headcount — "the amount of work in progress in your system… is going to probably impact your throughput more than shuffling people around." Manage the queue, not the team roster.

Percentile-as-risk framing — present "85% by June 15th" as "15% chance it will be later," and treat a falling confidence number as a trigger for the "what changed?" conversation (throughput, capacity, scope, features pulled away).

"1 in 200 years" translation — from insurance solvency capital calculation: express extreme percentiles (the 99.5th quantile) as return-period frequencies because that's the language executives actually process.

Shewhart's definition of stability — characterize variation with Shewhart's formulas; if all data falls within the accepted limits of variation, the process is stable and predictable. Crucially, per Vacanti: "It says nothing about team size, it says nothing about complexity, it says nothing about what management is doing."

Little's Law — requires "a long enough time period under observation so that those long term averages don't change" (a stationary distribution). Nigel invoked it as proof that forecasting methods presuppose stability.

The lean averaging toolkit — cycle time, lead time, takt time, throughput, process cycle efficiency — Nigel's alternative for stable systems, "things we use in the lean world [that] have served us pretty well."

Stable vs. predictable distinction (Dan North) — predictable means you know the system's characteristics and what makes it change, even if it's spiky (fractals as the example); stable means within Shewhart bounds. "The contrapositive is not true. It can be predictable without being stable." A genuinely useful analytical distinction, offered and left on the table.

Conditional predictability (Gaia) — a system can be non-stationary yet predictable if you know the dynamics of the variability — e.g., seasonal effects like holidays. You predict conditionally on the known change pattern.

Portfolio-level questions (Dan North) — apply probabilistic tools to populations ("10 of 100 projects"), never to single cases. The one methodological rule the panel's practitioners never rebutted.

4. Unanswered Questions and Omissions

"What happens when the future does not look like the past?" Nigel asked this repeatedly — new work with no historical precedent, cold-start teams, changed contexts. Vacanti's only response was rhetorical questions about retirement and cancer. The cold-start problem — arguably the most common real situation — was never addressed.

How much data is enough? Gaia directly challenged the "few weeks of data" claim as insufficient for non-parametric convergence. No answer. Worse: Vacanti, citing audio problems, said "I really did not understand any of that" — the strongest technical challenge of the session went unanswered on the record.

Scale. Nigel's opening question — many non-identical teams, communication complexity growing with scale, "find the long pole in the tent" — and Gaia's version (cross-team dependencies, moving capacity between projects, a statistically-driven KPI that handles complexity "given that you don't want to model the complexity") both got no real answer. Colleen's WIP point touched the edge and moved on.

What to do when the system is unstable. Nigel explicitly conceded the definitions and asked "so the system's stable. What happens when it's not?" The definitional fight resumed instead.

The independence problem. Dan North's core critique — Monte Carlo assumes you can identify independent variables and know each one's distribution — was never answered by the practitioners. Gaia's point that Monte Carlo requires a distribution/stochastic-process assumption was never reconciled with Vacanti's "no distribution" stance. The resampling-vs-modeling distinction was never made explicit.

No rebuttal to the misapplication charge. Dan North accused the field of using a population tool on single-case questions. Vacanti never responded.

The forecast-becomes-target pathology. Nigel raised it; Colleen reframed it as an opportunity for good conversations. But the organizational failure mode — management refusing to hear updated probabilities — was left hanging as "more of a management problem than a tooling problem."

Why 85%? The percentile choice was used throughout and never justified.

Validation. Nobody discussed backtesting or calibration — whether these forecasts have historically held. An entire debate about assumptions, with zero debate about evidence.

Item granularity. What counts as a "work item" for throughput? Nigel gestured at it ("we aren't necessarily writing all the requirements out and refining the hell out of them"); sizing/splitting was waved off as "a different panel on a different day." Cycle time vs. lead time was explicitly deferred too.

No named tooling. "A spreadsheet" was as concrete as the practical guidance got.

The commercial elephant. Vacanti and Colleen sell this method — books, training, ProKanban.org certification — and Nigel disclosed shared clients and that he'd prepped by feeding Vacanti's books to Grok and ChatGPT. Nobody acknowledged how incentives might shape the practitioners' certainty.

The meta-failure the panel named but didn't fix: Gaia asked "what are you trying to estimate?" and the moderator echoed it — and the question of what the forecast is for was never crisply answered. The session ended with the moderator assigning "homework" on distributions, stability, and predictability — a panel on probabilistic forecasting that couldn't establish its own terms, and a statistician who had to defend her CV instead of being heard.

Speaker A: So without further ado, I would like to call on stage our panelists led by Daniel Tanhorst North, Nigel Turlow, Colin Johnson, Daniel Vacanti and Gaia Becheri. A round of applause please.

Speaker B: My wife told me.

Speaker C: Oh, I don't mind.

Speaker B: I didn't know I was miked up. My wife told me I have to cross my legs when I sit on stage, especially when I'm elevated.

Speaker D: Suzurate.

Speaker E: Hello lovely panelists, Hello, Welcome Gaia, can you hear us? Fantastic. Okay, so we have everybody. That's good. Just for the record, I haven't got a clue what I'm doing here. So there we go. I'm here to learn. What I do know because I can read is that we're looking at probabilistic forecasting and methods of. Now my understanding a little bit. The genesis of this panel was Daniel something something probabilistic forecasting and Nigel something something. So I would like, I think a good way to start this is to go around, introduce ourselves and some kind of position statement about probabilistic forecasting in the context, I guess of software development or digital product development would be great. Do you feel like that's a sensible place to start? Let's start. Daniel's sitting nearest me. Let's start with Daniel. Me? Yes please.

Speaker A: Yeah. So I'm introducing myself and a position statement.

Speaker E: Yes please. That'd be great.

Speaker A: So good afternoon everyone. My name is Daniel Vacanti, I work with Colleen Colleen Johnson here@procombound.org I guess my position statement would be something along the lines of. It is im. Well, I'll just go all out. It is impossible to manage software development without managing risk and it's impossible to manage risk without some type of forecasting.

Speaker E: Oh, okay. That second statement's a bit racy. Totally with you on the first one. Let's see. Right. Okay, brilliant. Can we hop to Gaia? Hello, Gaia?

Speaker C: Hello. Hello. Hello. Yes, I'm Gaia. Be. I work as. I'm sorry, I have a lot of noise coming back.

Speaker E: Oh, you got feedback noise, have you?

Speaker C: Yes.

Speaker E: Oh, you can hear yourself?

Speaker D: Yes.

Speaker E: Okay, Av. We can. We're getting. Yeah, I'm getting a nod from the back.

Speaker C: Okay.

Speaker E: Because that's very off putting when you can hear your own voice offset by like half a second.

Speaker C: I'll try again. So I'm Gaia, I work in CDP insurance as a solution and delivery manager. So I work actually risk management. So thanks for bringing the risk management topic in this discussion. I'm a trained statistician so I offer data driven approach. But I also have been very painfully aware of the shortcoming of statistics and data driven approach. So I'm a bit hesitant embracing probabilistic forecasting when it comes to my deliveries.

Speaker E: Okay, fantastic. So if you didn't catch that. So guys, got an academic background as a statistician and then she's ended up in insurance, I guess, so been tempted out of academia. So this is brilliant. I love someone who's got like the academic chops, but who's also applying it as a day job is very cool. Okay, Colleen.

Speaker D: Hi everyone. My name is Colleen Johnson. I'm the CEO of proconbun.org and I guess my position statement for for probabilistic forecasting would be that I think it provides a better starting point for teams to understand when something will be done without investing a lot of overhead in having to define what is getting delivered or estimating out things in hours or points or insert anything after estimating without estimating, period.

Speaker B: Okay, so I make notes because I'm old, I don't actually disagree with anything that's been said. I may disagree with methods, I don't disagree with managing risk. And Gaia's a global, she's responsible for global risk, so I'm assuming that she'll help out. But I wrote a statement down from Deming and Dave's in the audience. They may disagree with me because he's a complexity guy, but a system must be understood as a whole, not in parts. Forecasting without understanding variation leads to tampering. So I'm a big guy, I'm a lean guy. I've got big, lean background. So I believe a lot about managing variation. And so we'll get into that, I hope a little bit in the conversation. But I think when we scale, because I know in some of our preamble we don't talk about scale. When we scale, the complexity of the work doesn't increase the complexity of the communications, the interactions, the organization increases significantly. So I'm really eager to understand how some of these methods solve for that. And of course, at scale, when we get many, many teams, each team isn't identical to the other team. So we need a way to be able to find the long pole in the tent. So it's just a little bit of an opening for sort of thought from me at the moment, but I want to learn as much as anything from all of you.

Speaker E: Fantastic. Okay, so then I guess a sensible place to start would be let's understand difference. So what is it about probabilistic forecasting, maybe some of the way Daniel describes it, that you are uncomfortable.

Speaker B: You might disappear.

Speaker E: My mic disappear.

Speaker B: Oh yeah, that's back. So you said what I'm uncomfortable with.

Speaker E: What are you uncomfortable with? So I guess what is irking you?

Speaker B: So I read Daniel's books. So I did go and read them and I went through and actually asked Grok and Chatgpt to give me summaries as well. And I started to look and I was. I'm not a statistician. I know absolutely nothing about stats and maths. That's why Guy is here. But when I started to look at it, I thought because it's been described as empirical sampling. Tell me if I'm wrong.

Speaker A: What has been described as empirical sampling?

Speaker B: So the method that's used to forecast a probable date or probable outcome,

Speaker A: you use the data that your system created. So if that's what you mean by empirical sampling.

Speaker B: Okay, so yeah. So we're looking at historical data to sample the future. Yes, I'm just asking if I'm correct. So I understood what. So now if we're looking at the previous data and we're using that to forecast the future, then my thought is that the future looks very much like the past. So why do we need to do this particular approach? Because if the distribution is changing and the variation is changing significantly, then we need to know what type of model to apply to that data to run a forecast on it. We wouldn't just be using. So that's what I'm trying to understand because maybe I just don't understand.

Speaker A: So I mean, do you want to go first? I'll be honest with you, I don't understand the question because.

Speaker E: Can I try and reframe it slightly?

Speaker A: Sorry?

Speaker E: Can I try and reframe the question slightly? And tell me if I'm slightly misrepresenting you. Okay, so we've got this model of. We're going to use some information, some evidence, some stuff that we're getting from our existing system of work, whatever it is, and we're going to use that. We're going to put that through some forecasting machine and it's going to give us some information, some projections or something about possible future state in our world. How do we know that any of that holds water? How do we know that the thing we're doing, the assumption we're like, what assumptions are we making? What assumptions we're making that we know about? What assumptions we're making that we don't even know about. How do we know that the machine's gonna tell us anything useful.

Speaker A: Yeah. So certainly. Well, that's a good place to start, is the assumptions. I mean, certainly the big assumption that we're making is that the future that we're trying to predict roughly looks like the past that we have data for. That's the essential. That's the essential assumption. Most of the time, that's a fairly safe assumption. There's a lot of times when it's not. But most of the time, whatever risk, whatever variation that you have incurred in the past, you can reasonably expect that to occur in the future. Now, what I don't understand about the question is because there is variation, there will always be variation in that past state. Data. How could you ever know without some type of. Some type of. I'm uncomfortable with the word probabilistic forecasting, because every forecast, all forecasting is probabilistic in nature. Right.

Speaker E: So it's a tautology, forecasting.

Speaker A: How would you ever be able to quantify the risk? If there's variation of data. Yeah, if there's variation of data in the past. So my team got zero things done on this day. Two things done on this day, one thing done on this day, nine things done on this day. If you've got that variation in the data and that variation can be reasonably expected to continue in the future, you need some way of being able to quantify what that looks like in the future.

Speaker B: But that's the question from my point I'm trying to understand because you're looking at the past and you made the statement, I maybe paraphrase it wrongly. Somebody will correct me if I'm wrong, that in most cases, the future looks like the past.

Speaker A: The future can be modeled with data from the past.

Speaker B: Okay, so we get variation in the past, but the variation is probably very small.

Speaker A: Not necessarily. Well, I guess we'd have to talk about what's small, what's large. You know, I mean, if you're a Deming fan, Deming makes it very clear, you know, how he characterizes variation. In fact, we could probably go back to Schuet would be even a better place to go for how to characterize that variation. So there's no really such thing as large or smaller.

Speaker B: What type of distribution are we talking about here in the work?

Speaker A: Because that's what I'm trying to understand. That's the fundamental. There is no distribution. There is no distribution.

Speaker E: At this point, I think I want to jump to is your data.

Speaker B: Maybe ask the brains on the team.

Speaker E: I heard the word distribution. I want to go to my statistician, Gaia, help us, help us make this make sense.

Speaker C: There is always a distribution. So what you're assuming is actually what is called empirical distribution. So it's not like there is no distribution. You are assuming that the distribution is actually completely represented by the past data which is called the empirical distribution. So you have a distribution.

Speaker A: So okay, yeah, so let me, let me, let me, let me, let me, let me correct. There is no well known statistical distribution. We're not talking about feeding it into a normal distribution. We're not talking about feeding it in a log normal distribution. We're not talking about. It's just your data. So yes, your data can be represented as it, but it's not a well known statistical distribution problem.

Speaker C: It's an empirical distribution. So you're actually making the assumption that the data are enough to estimate distribution, not some parameters, but the full distribution, which is already per se, a kind of a very strong assumption because basically you are using non parametric statistics which only converge when you have enough data. So I have read in some literature that you say few weeks of data would be enough, but I mean it's not enough for a non parametric metrics to converge to something statistically meaningful. I mean if an empirical distribution, if what you're using is an empirical distribution, you need enough data. So this would be my first, my first reaction to the reasonable distribution. The second one, I think it's, especially when we go at scale, my question would be what are you trying to estimate? And if what you are trying to estimate is giving you a clear KPI? Because I mean when you go at scale there are dependencies probably also across the teams. And I mean one of my biggest concern is can I for instance move capacity when I need it from one team to another or from one project to another. So when we go at scale, how can I identify a statistical driven KPI which is helping also modeling this complexity, given that you don't want to model the complexity. This is what I understood. I mean Colleen said we don't do any modeling, we don't spend any time sizing, giving points. So if you just use the data, how can you use this data to come up with a meaningful KPI to manage your teams?

Speaker A: So I'm sorry Guy, I don't know if it's the way the speaker is oriented here.

Speaker E: It's very muffled to me.

Speaker A: I really did not understand any of that. I'm really, really sorry.

Speaker E: So what I got, let me play it back. Okay, so Gaia and correct Me if I'm mishearing any of this because it is unfortunately, the way the speakers are set up in the room, we don't have fallback of you. So we're kind of hearing you bounce off a curtain. Two, three things. The first thing is the distribution we said is simply the empirical data. It's the data we've got and the shape of the data is the shape of the data we happen to measure. Right. So the first question, and it's what I want to get into as well, I have opinions, is let's think about the question we're trying to answer with these models. What is it we're trying to ask? What is it we're trying to estimate?

Speaker A: Can we say approximate instead of estimate?

Speaker E: Okay, what is it we're trying to learn about? Okay, okay, have an opinion about. And the second part was about the transferability or applicability. So can I take what I learn about the distribution or the data from one project and does it necessarily kind of apply to the next one? Is that sort of what you were saying, Gaia?

Speaker B: Not only.

Speaker C: I mean there is also the complexity of having the scale of big teams. And then basically my question is also when you are only using the data, you're not actually modeling complexity and the complexity may be the inering affecting your planning.

Speaker B: Right.

Speaker E: So there's a level of complexity of a project beyond which you've got too much emergence and too much dynamic behavior and it becomes just impossible to model that anymore.

Speaker D: Let's start with the data side of it. I think the question about the data sampling is a good place to start. I mean, I think you wouldn't want to, for you wouldn't want to use data from December maybe when everybody's on holiday to forecast a project that you're delivering in March. So we want to try to get like, for like sample sets of the data that we're using. I think the number of people on the team probably matter less in terms of shifting capacity. I think that was part of your question too, Gaia. Then how much work is in the system? So I think there's a direct relationship typically to the metrics that we're using. We're using throughput to run these Monte Carlo Carlo simulations and understanding the impact that the amount of work in progress in your system is going to have on your throughput, that's going to probably impact your throughput more than shuffling people around. But when you think about the data that you're feeding into these models, what becomes really important is every day that goes by is a new data point. So as you add somebody to that team, let's say the throughput starts to increase. That's probably what you're looking for by adding more capacity to the team, you want to keep reforecasting with that new data. So whether it's zero throughput or, you know, now we go to, we got nine items done, that's a new data point that you would want to feed into that model. So you're constantly rerunning these pretty much every day to check your forecast and see if your risk is either increasing or decreasing for that forecasted delivery day.

Speaker B: Can I ask a question on that? I wrote down a couple of things because I'm still confused on distribution, but that's probably because I'm just confused, period. But there's no distribution. But there must be a distribution because all mathematicians, statisticians look at data sets and they can see a type of distribution, Gaussian normal, log normal, something.

Speaker A: Those are made up. Those are mathematical hacks. Those are made up.

Speaker B: Oh really?

Speaker A: Yes.

Speaker B: I'm going to stop right there, Gaia. Apparently it's all made up. There's no type of distribution description.

Speaker C: I mean it may be to a certain extent true in the sense it's the same as seeing a model in physics being, I don't know, relative or Einstein kind of relative. Physics is a model. I mean of course as a model is a made up construct to explain reality. So to, to this extent, I would agree that is a, is a made up construct. The point is whether it's useful or not. So that's a different kind of question.

Speaker E: So before we get this, this is all been a bit abstract for me so far, let's talk about actual methods. So are we just idiot in the corner, Are we all talking about Monte Carlo simulation, building distributions, using that as a forecasting tool or is there some other bunch of things, set of tools that I'm missing completely? Is it mostly that?

Speaker A: I mean I'm probably, I'm talking about Monte Carlo simulation.

Speaker E: Okay. So because I, I wrote an article, I don't write many articles. I wrote an article a couple of years ago, I called it Monty Python Simulation. And it's basically, I just think we're doing an awful lot of this.

Speaker B: Maybe Dave west calls it Monty Python Syndrome. So.

Speaker E: Well, it could well be that. So the idea that a we use the wrong tool and also that we're using the tool wrong was where I was going with it. So partly, and this comes back to, I think Gaia's Question of like, what is it we're trying to measure here? What is it we're trying to understand? So my understanding with Monty Python Monte Carlo simulation is I have a bunch of ideally independent variables. I believe they're independent first assumption. And I believe that I know the distribution for each one of those variables. So that I could generate a random, a realistically random sequence of those and feed a model. I could then use that to generate a result. And based on the distribution of that result with that seed, I can then make some predictions. That's sort of my understanding of Monte Carlo simulation, right? At which point when we get to the assumptions, I'm like, there's so much of this that I'm not buying. Right. As a software guy, 30 odd years in software, so a, identifying which are the independent variables that genuinely affect whether or not how this is going to, this project is going to work or this sequence of work is going to happen. Also that even if we can identify them, that we can be reasonably confident about the probability distribution of each one of those things. So that I could generate a random sequence that was realistic. And then generally. So that's the using the tool wrong, using the wrong tool thing. And the using the tool wrong part is that then we try and answer not saying we, you don't. Because you do this. I'm sure you do this a lot more grown up than most of the people places I see it applied. But where I see it applied is people are trying to use this to solve a question. How will this next project work? What is the likelihood? And I'm like, that's not how this works. Right? I could say I've got a portfolio of a hundred projects and what's the likelihood that 10 of those are going to make it? That's a sensible question to ask with a probabilistic tool. But asking about a single data point, it's like asking about a single surgery outcome. Is the patient going to die? We don't know yet. We haven't done the operation. I need to know about this patient. That's not how the tool works. And I think that's a massive misapplication that I see time and again.

Speaker B: I like a lot of what you just said and I like what Colleen said about having to continuously sample. Now again, my experience is smaller than these guys who do it every day as a living. A lot of companies that I go into that are running the simulation, some that you've actually trained out, Daniel, because I know they're a client of mine and a client of yours. So I get to talk about the training.

Speaker E: Do you talk about each other behind your backs? Do you go to your clients to go that, Nigel? No.

Speaker B: Well, they may. I don't know. I mean, it's this professional respect. But the challenge that some of them have is that they generate a Probable forecast, the 85th percentile, whatever, it's probability of 85% probability being done on this period of time or this date or this range. And that becomes a target. It's no longer because then when. And I talk to clients daily on this, they go back and they rerun the forecast and they go, hey, a bunch of our assumptions were wrong that date. Now you're 85% has shifted a month. Now we're sort of more like 40% probable at this date or whatever the conversation is. But the management don't want to hear that now. And so that's more of a management problem than a tooling problem or a math problem. But my understanding from what you were just saying, correct me and please tell me if I'm wrong. And Gaia, the same Monte Carlo simulation seems to be used, stock markets, gambling maybe, that type of thing. Well, we've got a wide variance in that. So lots and lots of different values that are very random, potentially large data set. And we know the regardless of whether distributions are made up, and I'll stand on the experts on that, you know the type of distribution because the way you model the data depends upon the distribution. But if you've got a very small variance and a small amount of data, are we really doing Monte Carlo? Are we just doing a different form of averaging?

Speaker E: So I think I might be acting as a peacemaker here. I think that there's a disconnect. The way you're using the word variance, Nigel, is not necessarily the way variance, the same meaning of semantics of variance when you're doing this kind of modeling. Okay, so variance you're talking about is process variance and variance in a lean sense.

Speaker B: I mean, we use throughput in this model. So if the throughput is high, if the throughput is a day, two days, three days, a day, two days, two days, three days, that's a small variance. If the throughput is suddenly 5 weeks, 2 days, 27 weeks, this type of thing, that's a large variance.

Speaker D: That would be cycle time though, right? Duration?

Speaker B: No. Well, if you're talking about item, yeah, you're right. So. Well, we don't get on the cycle time, lead time thing, but if we're talking about throughput, so this many Items, two items like units per unit time. So this many items on this day, this many items on that day. So if I did a two week period of variance and I was working at 323-214-32321, that's a low variation. It's a reasonably stable system. And even read that in one of your articles in 2019 we talked about Little's Law because that assumes the system is stable. Little's Law doesn't, doesn't work. It isn't looking at a highly unstable system. It assumes that the system is stable. And some of your own writing says we assume the system is relatively stable. What I'm trying to get at here is if the system is relatively stable, why can't we just use a standard set of averaging which we do in lean all the time? I mean you've studied lean stuff. We use cycle time, lead time, tack time in proper pool systems, we use throughput, process, cycle efficiency. These are all the things we use in the lean world. They served us pretty well. Now the whole Deming thing is if you reduce the variance in the system so the system is predictable, then why do I need a simulation tool to give me that outcome? I think the problem isn't in the work. I think the problem is beyond the work in the greater system around it. And I think that variation comes from management, red tape, bureaucracy, all that stuff. And I think the focus is wrong. I think the focus shouldn't be on getting a probability of when this work will be done. I think the focus should be on why are we having to do this to get a probability of when the work should be done.

Speaker E: So I'm going to jump in here. I'm being a really bad MC because I've got opinions here. I'm going to go back to. Actually, let's go to Gaia. We haven't heard from Gaia for a moment. What's your take on all of this? So. So what Nigel's. I'm going to mansplain what Nigel just said. What is that? What I believe Nigel is saying is if we're talking about. If we. Right. There's a contrapositive thing going on here. If we believe the system is relatively stable, or at least if Monte Carlo simulation requires things to be stable enough that we can model them, then surely they're stable enough that you could use much simpler statistical tools.

Speaker B: Yeah, what I'm saying is.

Speaker E: So why would you bother with. The other thing is that sort of

Speaker B: don't want Monte Carlo. You wouldn't use it if you had a stable system is what I'm saying. So if we're assuming the data is relatively stable, why do we need Monte Carlo simulation? That's probably.

Speaker E: I have a thought about this, but I want to ask Gaia that question.

Speaker C: What you are doing is starting a non Monte Carlo simulation. Monte Carlo simulation actually requires to have a distribution or a stochastic process on the basically as an assumption to run a Monte Carlo method. I think there is nothing wrong with the variability. The point of variability is that you need way more data to make the model converge. So again you have an empirical distribution, you have. If you have large variability you need even more data. And I mean even if you're sampling on it on a daily basis, how many weeks and how many observations you would need for the model to converge. And maybe the last comment I have is again about what you are chasing after. Because I mean if I go to my management, the senior management level and I say there is 85% probability this project will succeed, they will not understand what I'm talking about. If I will be the 15% interval where I'm failing, they will not say you told me that it could have been failing. They will just tell me why you didn't succeed. You told me it would be 85% of probability of success. And I'm saying that because I'm a statistician. I understand confidence interval, I understand the contacts, etc. But I live in a world risk management when people are trained in this kind of topics and yet when we are for instance calculating capital for solvency reason in insurance, in those cases we look at the 99.5 quantile of the worst event etc, etc. We never talk about 99.5 percentile with management. We talk about 1 in the 200 years event because this they understand and even it's not correct, this is what they understand understand. They don't understand 99.5%. So I'm a bit confused on how this actually is even helping me getting to something that is measurable and also something that can handle my conversation with management in terms of managing expectation.

Speaker E: Right. This is the thing. So the underlying all of this is we're using this as a tool for communication, right? And so if we as practitioners are unable to articulate what this stuff means, we've already lost. It doesn't matter how good the tool is. So I think there's an element of that and I see this in the context of I have never met an adults professional outside of the stochastic academic world that could tell you what the difference is between an 80% and an 85% confidence interval or even an 85% and a 95% confidence interval. Because what I understand by that is if we run many, many statistically significant number of these, 15% of them will fail. But if I run a single sample, I know nothing about that single sample until I run that sample. It's one of a set of samples. And again, this is where we end up sort of misusing the tool and miscommunicating the tool. What's the likelihood of this project overrunning? That's we don't have a tool for that. What I can do. So I want to look at some success stories. We talked about bashing this and things. One of the places I've seen it used really successfully is when you're trying to get a distribution for things like throughput of features or stories or whatever work items in a software delivery team. And when they look at the historical throughput lead time, whatever it might be, how are we going to be academic about lead time and attack time and cycle time and stuff? The amount of time it takes to do an item and the amount of those items I can get done in a chunk of time and making predictions about how many I'm likely to do. Absolutely. This is a good statistical. I think it's a good statistical tool for that and a relevant statistical tool. And you can have kind of grown up conversations about ranges of things we're likely to ship in a period of time. I think that makes sense. The 85 versus 95% thing I think just confuses the heck out of exactly what Guy is saying. She's a statistician. She gets this right. It's her bread and butter. Sits in a room full of risk execs in a major insurance firm and they're like, no, sorry, none of that works for me. One thing I just want to follow up with Nigel, you're saying about why would you use something like Monte Carlo when we have little variants? My understanding. And again, Daniel, please jump in because I'm an amateur.

Speaker B: Stable system.

Speaker C: Let's be Dan, before we move off

Speaker D: of that, can I make a comment about the communication with executives?

Speaker E: Yes.

Speaker D: I think what you just described is a really important piece of this conversation though. Typically when we forecast something and we go to an executive team and say there's an 85% chance we'll deliver by June 15th or earlier. Right. We're giving them an idea of when we can have something done. What we're really telling them is there's a 15% chance it will be later than June 15th. So we're starting to talk to them about risk. And I think the way you described it, Nigel, of if I go back in two weeks and say we've dropped down to 60% confidence in hitting June 15th, I just invited them to ask me a lot of hard questions. What changed? Did our throughput rate change? Did our team capacity change? Did I add more scope? Did I add a different feature? Did I pull them away? There can be a million things, but we can't have that conversation if we just go back to them and change the date. And so I think by communicating with our stakeholders about the risk just increased on this project, we can start to troubleshoot the system that's contributing to that risk increasing.

Speaker B: I think the vocabulary aside, I think that that's a sensible conversation. I wrote down something because I keep coming back to this thing we're assuming we're in now I'm making this assumption. Tell me if I'm completely, completely wrong. But we're assuming we're in a relatively stable system because we're modeling historical work for future work and we aren't necessarily writing all the requirements out and refining the hell out of them. So they're all sort of, you know, small enough to be, to be counted. But I wrote down what happens when the future does not look like the past. How do we know when the future doesn't look like the past if we've not written some requirements, you know, so how do we. What do we do then? Because I don't know what we call that because apparently log norm things are made up.

Speaker E: I think in order to be able to have that conversation, which is kind of what Daniel is coming from, we need to have an opinion about what the future should look like to know that the future isn't looking like it. Otherwise, the future just more stuff, right? If we don't know what the future is going to look like, if we don't have a model for that, well,

Speaker B: how do you know what the future future is going to look like? Because you're going to take past data, stuff it in and say, if we have this range of throughput in the future, this is the. And you want to hit this date. That's the likelihood. I'm paraphrasing. But what if we haven't got a historical precedent for it? What happens then?

Speaker A: What's your historical precedent for when you're going to retire? What's your historical precedent for if you're Going to get cancer or not? What's your historical precedent for if I.

Speaker D: Or for,

Speaker A: you know, before you have children? How many children are you going to have? What's your historical precedent for any of that?

Speaker B: So let's talk context here, because they're very different contexts.

Speaker A: Okay, I asked you a question.

Speaker B: Yeah, but. All right, so I'll answer it in this way. The context matters. And the context of when we've been doing work on a sustained basis for a period of time and we have a relatively stable system because we understand.

Speaker A: Keep throwing out that stability term to go all Princess Bride on you. I don't think you keep saying that word. It doesn't mean what you think it means.

Speaker B: Okay, what do you think I think it means?

Speaker A: You wrote down in your little notepad?

Speaker C: Three, two.

Speaker A: One, two, one. Little. Dr. Little has a very clear definition of what system stability is. Sure. Has an even, I would argue, an even better version of what system stability means.

Speaker B: But the arrival rate and the departure rate are stable.

Speaker A: Do you know? Okay, and what does Little mean when he says stable? What does he mean when he says that?

Speaker B: So he's assuming the system, the person in the middle doing the work, which is the throughput. But remember that the Q is the actual part of when we start counting the time from when they arrive to when they exit the system. But when we start talking about.

Speaker A: No, no, no. What does Little mean by stable?

Speaker E: Hang on, gentlemen, I'm going to intervene here. This is brilliant. There's a bunch of folks here who won't know some of the technical details. So when you say Shoeheart has a great definition. Can you give me an elevated definition of what stable means? And when you're talking about Little's Law, just. It'd be great to kind of have a little.

Speaker A: Like.

Speaker E: I'm trying to get.

Speaker A: We've got somebody here who's speaking as an expert on Little's Law and can't even tell me what Dr. Little means by stability.

Speaker B: I'm not an expert on Little's Law at all. But everything I've read in your work and on. When I've read about Little's Law, it says it assumes the system.

Speaker A: What do we mean by stability?

Speaker B: So when I'm a lean guy and I'm talking about a stable system, I'm

Speaker A: talking about what does Dr. Little mean when he says stability?

Speaker B: System, from my perspective, is predictable.

Speaker A: And what does that mean? There's actually a real definition of what that means.

Speaker E: Could you give me the definition? That'd be great.

Speaker A: So in Little's case what we need is a long enough term, a long enough time period under observation so that those long term averages don't change. Essentially what he's saying, it's like a rolling window. The distribution of your data is stationary. It's not a. We gotta come back to the distribution conversation because doesn't seem like anybody on this panel understands what a distribution is either. But if what we're talking about is the long term average of that's what Littles is. Schuart has a different definition of stability. He says that all of the data in your process is going to be characterized by variation. And there's a way, using some formulas that Shewart came up with, there's a way to characterize that variation, such as all of that data is within acceptable ranges of variation versus all of that versus maybe some points are. If everything is within those normal limits of those accepted limits of variation, you have a stable system. It says nothing about team size, it says nothing about complexity, it says nothing about what management is doing. It says nothing about that.

Speaker B: So let's assume. Assume then. I mean I take your definitions as red and I have no reason to. You know more about this than me. So the system's stable. What happens when it's not?

Speaker E: Before we get into this, because again this is the. We keep using these words. I see a useful distinction between stable and predictable. Predictable means there are a number of factors. You don't.

Speaker A: I do not.

Speaker E: Can I offer you a distinction? And then you can please search pieces. Okay, so predictable means I can tell you the characteristics of the system. I can tell you the characteristics that make it change. And those characteristics themselves are well known. So those are my independent variables and those are the distributions of those independent variables. I can say this system is characterized by these things. And if I can tell you what those things are, I can, I can make predictions about that system. Now those things, the system itself might be super spiky, but it's super spiky in a predictable way. So it's not stable but it is predictable. And because it is predictable, I can make predictions about it. Even though I'm dealing with that spikiness. I think stability is where you don't have. Stability is where those things are within exactly what you say, within bounds

Speaker D: quite

Speaker E: often, certainly in any kind of complex adaptive system I might have very easy to describe. I mean fractals are a great example of this. A system that's easy to describe, I can describe its characteristics, but it's not stable.

Speaker A: So let's simplify it, let's just talk about, let's just talk about data and let's say specifically, let's talk about cycle time data. Okay, sure. Would say, I believe the way that I read Shewitt, I should say the way that I believe Shewart Schrute would say is that if your system, if your cycle time data is stable, it is predictable.

Speaker E: Yes, yes, but the contrapositive is not true. It can be predictable without being stable.

Speaker A: No.

Speaker E: I can give you a distribution for each of the inputs to a Monte Carlo simulation. I can then run that simulation and, and I can tell you that this many projects are going to come in within this period of time and I can be confident about that empirically. And the various cycle, the various times of those projects could be all over the place. And I'm still right.

Speaker A: If again, the way that I reach Shewart is if the data that you're feeding into that Monte Carlo simulation does not fit Schuart's definition of stability, then you can't trust if the input is

Speaker E: stable, the output isn't stable.

Speaker A: That makes no sense to me.

Speaker E: Right, makes sense to me. Daniel.

Speaker B: I'm conscious of time and I'm interested to know what Gaia, because we've heard some clear statements and I can't argue them, so I'm interested.

Speaker E: What I say, let's bring Gaia back in.

Speaker C: Yeah, I mean, I'm not sure about the concept of stability, but you said it means that something is stationary. But then you said it's when expectation is stable, but I mean positionatically time series, you also need the variance to be stable. So basically the distribution, the full distribution should be stable across time. And going back to Daniel's point, actually you can have something that is predictable, but it's not stable in the sense that maybe variability changes along the time, but you can conditionally on the changes of the variability, you can still have a prediction on how the future works because you know the dynamics of the variability. So let's put it in a, in a more concrete case, let's say that you, you know that the variability is changing during the year, for instance, because you have holidays or whatever, but you know the way that this variability is changing. So your, your system may not be stable, but it's not stable in a predictable good way. By stability you mean stationality, which is not clear to me because you, you gave a different definition than what station ideas as well.

Speaker A: And I think this is, for me, this is fundamentally, with all due respect, fundamentally where this panel has failed is before we can even have an argument. We have to agree what we're arguing about. And there is a fundamental disagreement about what a distribution means. There's a fundamental disagreement about what stability means. There's a fundamental disagreement about what predictability means. I know in all of your minds it makes sense, and in my mind it makes sense, but we are nowhere near any agreement about what these terms mean. So when I come at it from a short perspective. You don't have the short perspective in your head, so you have no idea whether or not you're like, well, predictable means this. Well, maybe in your mind it does. And I can't necessarily say that that's wrong. But I'm saying if we talk about,

Speaker E: you know, Monte Carlo simulation, Schuart did

Speaker A: not know

Speaker E: he's one person who has an opinion about stability. You're saying. Well, you're not using Schuert's definition. No, I'm not. No, I'm not. There are many definitions for these things.

Speaker A: Well, that's what I'm saying. That's why I gave you two. I gave you short and I gave you little. And I tell you what, and here's maybe this is fun to mentally it. If you can point me to somebody that I should trust more than, say, Dr. Little or Dr. Shewart, I'm all ears.

Speaker E: Sure.

Speaker A: But Nigel's opinion, I mean, if I have to choose between listening to Nigel or listening to Dr. Shuert, I can tell you who I'm going to listen to.

Speaker E: Right.

Speaker A: So, I mean, until you come, until you come at me with, you know, with something that is a little bit better and you even understand, you know, what we're talking about.

Speaker E: Hang on. So I'm going to take that as a closing remark. Thank you, Daniel. So the panel has failed. We didn't even get past the fundamentals. Sounds to me like a bunch of us have some reading to go and do. So that's kind of cool. Colleen, what's your closing remarks? Observations?

Speaker D: I think everybody should go practice a little bit with a Monte Carlo to understand some of the things we're talking about today, because I think we did jump right into some really heavy concepts here, and I don't think it has to be that hard to get started. You have the data you need. You can do this in a spreadsheet. You can randomize your data and start to see how much variability there is coming through your forecast and understand risk in a different way and not sit through hours of story pointing. And if really what we're trying to do is help Our developers spend more time coding. That's really the goal, not to have perfect dates. I think that we could probably simplify this for a lot of people.

Speaker E: Sure.

Speaker B: So I'm not advocating story pointing, by the way, but this is a different panel on a different day. I need to go study because I'm hearing there's no such thing as a distribution. All the statistics professors and mathematics professors have got all that wrong. I spent a ton of time reading stuff before I came here just to try and understand a little bit, talk to a lot of people I respect about this. But. So there's a lot of things that Daniel has stated that I'm not clear on and I'm not sure others who are professionals in that rail. Mark and I know Shuart's work not as detailed as you do. For sure he's the father of SQC if we want to know exactly what he's for. The Six Sigma guys took that and bastardized it. But from this panel's perspective, I think there is some fundamental distinction agreements which I'm at my depth on. You know, when it comes to modeling and distributions and statistics, I'm way out of my depth. But I heard that Daniel doesn't agree with some of the stuff that seems to be foundational teaching in the topic.

Speaker E: I want to give Gaia the last word. Gaia, could you bring this into land, please?

Speaker C: I don't know. Now I feel it comes personal, but I mean, I don't have to justify my professionalism in statistics. I mean, you can just look at my CV after the. I'm confident enough on that front at least. So I think it was still a good conversation. But I agree there is a disagreement in terminology. That's clear. Being taught that I should go and practice with Monte Carlo and I don't know what the distribution is. Go and check my. I think that will speak by itself.

Speaker E: Okay, so we have. We need to go and find out what distributions are, what was the other ones that we want? Stability, predictability and some other things. So we've all got some homework to do. Thank you very much indeed. Paddle. Appreciate all of your energy.
