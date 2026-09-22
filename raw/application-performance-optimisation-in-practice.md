---
url: https://gist.github.com/njt/34460de457cb2f63069845db0e3f7d4d
date_fetched: 2026-08-07
---

# ytx: Application Performance Optimisation in Practice - Steve Gordon - NDC Copenhagen 2026 — NDC Conferences

Source gist: https://gist.github.com/njt/34460de457cb2f63069845db0e3f7d4d



======================================================================
## FILE: 20260807-application-performance-optimisation-in-practice-steve-gordon-ndc-copenhagen.md
======================================================================

**Key points**  
- Performance must be treated as a proactive engineering discipline, not a reactive crisis response. Most teams ignore it until users complain, then firefight under pressure.  
- Premature optimization is a mistake, but so is ignoring performance entirely. The famous quote actually says to forget small efficiencies 97% of the time, yet not pass up the critical 3%. Optimization is not evil when guided by data.  
- Performance is contextual. It always matters, but the bar varies by business. You need to know where your system falls on the scale to decide how much to invest.  
- Performance impacts three connected dimensions: user experience (latency, smoothness), cost (CPU/memory → infrastructure spend), and reliability (GC pauses, throttling, cascading failures). Improving one often helps the others, but the priority depends on context.  
- Production monitoring data is the ultimate trigger and validation. Local benchmarks and dev‑machine tests do not replicate real usage patterns, concurrency, GC modes, or latencies.  
- The optimization loop: start with production data (APM, traces, SLOs) → identify the problem → profile (CPU and memory) to find hotspots → ensure tests exist → benchmark to get a baseline → apply small, targeted changes → re‑benchmark and re‑test → document → repeat. Deploy and validate with production data again.  
- The goal is not the fastest possible code; it’s code that is fast enough, maintainable, and delivers the required business outcome. Stop when you hit diminishing returns or your production metrics improve sufficiently.  
- AI can assist in suggesting optimizations, but you must verify each change with benchmarks and tests.  
- Performance work should be shared ownership, part of normal delivery, with SLOs and alerts set before release, and regression detection built into the pipeline.

**Pithy and provocative quotes**  
- “Premature optimization is the root of all evil. … Yet we should not pass up our opportunities in that critical 3%.”  
- “Performance is a feature that users experience with every interaction.”  
- “Performance, unlike functional bugs, is quite insidious. It doesn’t necessarily rear its head immediately. It will degrade over time until a point where it hits you.”  
- “Never trust a user blindly. … Go back to your production data and ask it that kind of question.”  
- “I tend to see allocations as I’m reading code, which is both good and bad. I mean, it’s slightly better than seeing dead people.”  
- “The goal is not the fastest code possible. It’s code that’s fast enough, but easy to maintain and delivering good results for you.”  
- “It was fast locally” is a bias you must avoid.  
- “If you raise the issue of performance, it’s something we’ll get to later or we will fix if it becomes a problem. And those are often phrases that will bite you back.”

**Tools, practices, and methodologies**  
- **Production monitoring data (APM/observability)** – Logs, metrics, traces, alerts, SLOs. Use to detect degradation, validate user complaints, and confirm real‑world impact after optimization.  
- **Service Level Objectives (SLOs)** – Define error budgets and track whether you’re on track to meet latency/availability targets. Less noisy than raw alerts; warns when budget depletes.  
- **Profiling (CPU sampling)** – Use sampling mode (e.g., JetBrains dotTrace) to identify hot methods by call time. Run many iterations (100k+) to capture short‑lived methods.  
- **Profiling (line‑by‑line / high accuracy)** – Injects IL to count exact line executions. Gives accurate call counts but adds overhead; use to find methods called millions of times where tiny gains multiply.  
- **Memory profiling** – Collect full allocation data and take snapshots to see heap allocations, boxing, string builder expansions, and temporary strings. Use to spot unnecessary heap allocations.  
- **JetBrains Profiler API** – NuGet package that lets you start/stop profiling and trigger snapshots from code, enabling precise measurement of short code paths.  
- **Benchmark.NET** – The de‑facto .NET benchmarking library. Use to get baseline execution time and allocations, then compare after each small change. Include multiple input cases via `[Params]`.  
- **Performance optimization loop** – Structured process: monitoring → profiling → tests → baseline benchmarks → small targeted changes → re‑benchmark → document → repeat. Inner loop focuses on method‑level changes; outer loop validates in production.  
- **Pre‑sized StringBuilder / array renting** – Avoid internal StringBuilder expansion by initializing with capacity equal to input length. Later, replace StringBuilder entirely with a rented `char[]` and span APIs to eliminate allocation and locking overhead.  
- **Static reusable StringBuilder with atomic concurrency control** – Use a static instance guarded by `Interlocked.CompareExchange` to reuse a single builder per thread in mostly sequential scenarios, slashing allocations.  
- **Span<T> and Memory APIs** – Slice strings and arrays without allocating temporary strings. Use `Append(ReadOnlySpan<char>)` instead of `Substring` to avoid short‑lived string allocations.  
- **Ref structs and `ref`/`in` parameters** – Convert short‑lived state classes to `ref struct` to guarantee stack allocation. Pass by reference to avoid copying large structs.  
- **ConcurrentDictionary with approximate count** – Replace `Hashtable` to eliminate boxing/unboxing and leverage modern collection optimizations. Use a volatile `approxCount` to avoid the locking overhead of `.Count`.  
- **SearchValues<T>** – New .NET type for ultra‑fast searching in sequences. Use to efficiently scan for SQL keywords.  
- **Keyword metadata for algorithmic shortcuts** – Pre‑compute which keyword can follow the current one (e.g., after `SELECT` only `FROM` matters) to skip unnecessary character comparisons.  
- **Return original string when sanitization is a no‑op** – Compare sanitized output to input; if identical, return the original string to avoid a final allocation.  
- **AI‑assisted optimization (Copilot)** – Ask for 10 possible CPU/memory optimizations, filter out hallucinations, implement plausible ones, and verify with benchmarks and tests.  
- **Automated performance regression detection** – Run benchmarks or load tests in CI/CD to catch regressions before release (advanced, but valuable for the 3% scenarios).  
- **Documentation of optimized code** – Comment why the code is complex, what it does, and that it’s highly optimized. Warn future maintainers to re‑run benchmarks if they modify it.

**Unanswered questions and omissions**  
- How do you set appropriate SLO thresholds and error budgets for a given service? The talk says “set them before you deploy,” but offers no guidance on choosing numbers.  
- Balancing engineering cost vs. infrastructure savings: the speaker mentions that performance work can pay for itself, but provides no framework for deciding when the optimization effort is worth the cloud bill reduction.  
- The demo focuses on a single, self‑contained library method. How does the loop scale to large distributed systems with many services, databases, and network hops?  
- Cultural and organizational barriers: how do you convince management to invest in proactive performance work and monitoring when there’s pressure to ship features?  
- The “3%” critical scenarios are mentioned but not characterized. How do you objectively determine if you’re in that category?  
- The inner loop of small changes and re‑benchmarking is slow. Are there ways to speed up the feedback cycle without sacrificing reliability?  
- The talk briefly mentions AI assistance, but doesn’t explore its risks (e.g., subtle correctness bugs, over‑optimized unreadable code) or how to integrate it into the loop safely.  
- Memory profiling with dotMemory missed the boxed struct allocation due to a tool limitation. How do you systematically catch such hidden allocations across a codebase?  
- The optimization stopped at ~86% allocation reduction and ~46% CPU reduction. What heuristics signal “diminishing returns” and when to stop? The speaker says “when you start to see diminishing returns,” but doesn’t quantify that.  
- No discussion of performance testing under realistic load (e.g., load testing with varying concurrency) before production deployment, beyond “deploy and check observability data.”  
- The talk is .NET‑centric. How do the principles translate to other runtimes (JVM, Go, Node.js) where tooling and memory models differ?  
- The speaker mentions that “most of you probably aren’t working on those kinds of applications” (the 3%), but many enterprise systems have performance‑sensitive paths. How do you identify which parts of a large system deserve the full loop vs. a lighter touch?


======================================================================
## FILE: transcript.md
======================================================================

# Application Performance Optimisation in Practice - Steve Gordon - NDC Copenhagen 2026

- **Channel:** NDC Conferences
- **URL:** https://www.youtube.com/watch?v=jd8zvXEx40s
- **Duration:** 1h 2m 30s
- **Transcribed:** 2026-08-07

---

Good morning. I think there's still a few seats if you want to. Yeah, find them. Welcome. Thank you for coming.

It's great to see so many people here. My name's Steve. I'm a Microsoft MVP pluralsight author and engineer at Elastic. You can find me online. I'm evejgordan on most of the social platforms and I blog@stevejgordon.co.uk and the most important part of this slide is not about me.

It's this bitly link, this bitly link and this QR code which takes you to a copy of the slide deck and the demos I'm going to show you today. So if you want to kind of absorb the stuff again later on, you can go and do that. And today we're going to be talking about application performance optimization in practice. I'll give everyone a second to get a picture of that slide. I'll show it again at the end if you miss it.

I think most people have got it. Just to set expectations and make sure that everyone thinks they're in the right place. I'll just go through briefly what I'm going to cover today. So we're going to start with some foundational concepts, just thinking a little bit about what performance means to applications and more importantly, how do we make performance visible so that we can go in and choose when and what we should be optimizing in our code. We want to be doing that in a kind of repeatable way and a structured way.

The main bulk of this session is actually going to talk about this thing that I call the performance optimization loop. So this is the kind of the practicalities of, okay, we know what we want to optimize now, how do we do that? Using optimization triggers to guide us into what we're going to optimize and then real data to guide further work that we're going to be doing so that it's a repeatable and reliable process every time. Once we've gone through the theory steps, then the meat of this will be a 35 or so minute demo where we actually start seeing how I applied this in a real scenario. This isn't going to go through the nitty gritty of things like span of T and all of that kind of stuff.

I've done talks on that in the past. It's more about applying the loop. But we will talk and see about how we do profiling both of CPU and memory, how we apply benchmarking, and then how we actually take optimization through various steps. So looking at a real project that I worked on. And at the end there will be a few minutes left, so we'll just dive back into some theory and talk about how do we make all of this sustainable and something that you can actually take back to your work.

So the problem with performance is that a lot of us will face performance at some point, point or another on our teams. But what I tend to see all of the time is that most teams treat performance as a kind of reactive process rather than being proactive and finding problems before they hit your users. It's a case of you deploy stuff and then everything's fine, everything's fine, everything's fine. And then suddenly it's not. Suddenly you've got a user that's ringing up saying that this page is really, really slow.

And then you are in a situation where you are reactively trying to firefight what changed? When do we change it? And you're trying to do that under the pressure of this user that's obviously experiencing problems and maybe stakeholders internally that are pressuring you. And it's not really a nice process at that point. There are two, I would say, common mistakes around how engineering orgs tend to treat performance in their businesses.

The first, we'll talk about this one, because this is kind of the elephant in the room and it's the whole premature optimization problem. This is where we optimize code before we have data guiding us to do so. And this burns engineering cycles if there's no real direction to trigger this need. You're probably all thinking of that famous quote right now. Premature optimization is the root of all evil.

This quote is often sort of abbreviated to this form, but in its fuller context it says we should forget about small efficiencies, say about 97% of the time. Premature optimization is the root of all evil, yet we should not pass up our opportunities in that critical 3%. So there's a couple of important takeaways from this. Something this is saying and something this is importantly not saying. What it accepts is that there are a small number of situations that 3% where we might already know that we're in a highly optimized scenario.

We're building trading platforms or financial services or maybe health service applications. And in those situations, every nanosecond could influence your revenue for the business that you're working with or possibly cost lives. And those situations are rare. Most of you probably aren't working on those kinds of applications. But when you are, you're not prematurely optimizing.

You are just optimizing because you know that for your business, you need to have performance. So that's. That's okay, but it's rare. What it also doesn't mean, and this is how I hear, people kind of tend to throw this phrase out. It doesn't mean optimization is the root of all evil.

Optimization, when you're guided by your data to go in and optimize some areas of your code for various reasons, and we'll talk about those in a moment, is again, not evil as long as it's guided and we apply a structured, repeatable process to how we first establish that and then apply the techniques. The opposite mistake, probably more common, is that one we talked about before, this ignoring performance until it becomes a crisis. You probably have faced this situation internally where you're just under pressure to get features out. We need to ship this thing. Okay, great, we'll ship it, ship it, ship it.

And if you raise the issue of performance, it's something we'll get to later or we will fix if it becomes a problem. And those are often phrases that will bite you back at some point in your time. Because performance, unlike functional bugs, is quite insidious. It doesn't necessarily rear its head immediately. It will degrade over time until a point where it hits you.

So a functional bug, if you've got a code path that doesn't properly check nulls somewhere in your code base and you get a null reference exception, that will either crash the app or hopefully just trigger an error and send something to the user. But someone's going to be blocked at that point. But there's a cause and effect there that's easy to correlate. You can go in and you can fix that bug and release the fix with performance, which degrades gradually over time. Something that you release may be a new sort of endpoint on your web application, and the page loads in around 200 milliseconds, but that drifts.

And after maybe some other features get released elsewhere, it drifts to 250, then 300 and then 400. So each of these changes is small enough that no one's necessarily noticing them until the point that someone says, hey, this is really slow. And again, we're at that point now where if users are complaining, this is bad for our business potentially, and we're behind, right? And so we're no longer really in this formal process of optimizing. We're just back in crisis mode.

So what this talk is really about is just trying to bring us into treating performance as an engineering discipline and being proactive in how we apply this. Our goal is that we want to catch degradation early and then make informed, data driven decisions about when and what we're going to optimize to avoid that crisis point. Now, everything I've just said in the last six minutes or so can be summed up really as performance is contextual. Only you and your organizations and your teams will know to what degree performance matters to you. Performance, I would argue, always matters, but for some people, the bar is very, very low and you don't have particularly critical systems.

In other situations, performance really matters, and that's that 3%. Where you fall on that scale is contextual to your business. Only you can understand it exactly, but you should know roughly where you fit on there so that you can guide how much time you invest and where. Now, when we talk about performance, we tend to focus on raw speed. That's the thing people think about.

And speed is important, but performance affects more than one dimension. And so I kind of put it on this impact triangle here. And I put user experience at the top because I would say that this is the most important. Probably for most situations, this is where we're talking about things like response times, so the latency of pages. But it's also harder to measure things like the smoothness or how a page feels to use, or how an application feels to use.

But this is usually where the pain is noticed first, because performance is a feature that users experience with every interaction. And if they're having a bad experience, then there's risks involved. If they're paying customers, if this problem persists, they might just leave you. So we really want to focus on that as a general priority. But then there's also this factor of cost, right?

If we have an inefficient system, inefficient code, then it does mean that we might have higher CPU usage, higher memory usage, and that might mean that ultimately we need more instances or VMs or infrastructure behind this service to make it run well enough. And that will have cost implications. Maybe your cloud bills go up because you need to compensate. So cost is something to think about. Performance work can often pay for itself here, but you need to do the balancing act.

How much engineering cost are we going to put into optimizing versus what we can save on the end bills? And then the final corner I put is reliability, because systems under pressure, under resource pressure, tend to behave unpredictably. We don't necessarily see all of that exactly, but this is where we run into things like GC pauses, kicking, garbage collections running that can Trigger throttling and timeouts and other cascading type failures within systems. The important point is that all of these are connected. Changing and improving one of these areas typically will help the others, but not necessarily by the same amount.

And so what matters most to you depends on context. Again, if you're building a customer facing checkout flow, then your priority might be user experience. There's stats and data out there that show slow page load times during a checkout process can lead to higher drop offs, less conversion rates. And so for your business, it's more important to optimize for the user experience there. If you're building a back end processing system, then you might be balancing cost and reliability as your main drivers.

Ultimately, performance engineering is knowing at a given point in time for a given service which corner of this triangle you're focusing on most and prioritizing with that goal in mind. And as I'm going to stress for the next few slides, this is why I think production data is particularly important. So let's talk about what should trigger us to think about optimizing code. So this is avoiding premature optimization and using data to drive the right choices around when we optimize. And I put at the top here production monitoring data.

So this is APM solutions, observability data, things like logs, alerts, metrics, Alerts and metrics being. Sorry, logs, alerts and traces. Traces and metrics being probably the most two useful ones. Out of interest, how many people in the room can honestly say you've got an APM or observability solution in production that's pretty reliable for what you're doing that's less than I was hoping. Probably a few.

I would say it's definitely not half there. So the reason this data is so important is it's showing us what's actually happening in a real system under real loads. So alerts are useful because we can set a threshold for this page should load in X number of milliseconds or under and we get alerted when it doesn't. Problem with alerts is we tend to get into this problem of sort of noise and people start ignoring alerts. If they start triggering by a few milliseconds over your alert thresholds, everyone just goes oh, it's only a few milliseconds.

So we ignore it and ultimately people start sort of just pushing alerts to the side. Service level objectives is a nicer measure I would argue because it has a built in error budget. It doesn't trigger for every failure. But as your error budget depletes over time. It can see are we on track to meet our 99.9% of requests on under this threshold.

And if you start to deviate from that, then it warns you. So you can look into that as another opportunity to use metrics in what I would say a less noisy, more useful way. Trace data can be useful as well, because we can do ad hoc analysis of tracing in our telemetry data to see how pages are actually functioning, what calls they're making, what databases they're calling, what downstream services they're calling, and how long everything takes. You can use it when you've released a feature to understand how it's actually working when you get it out, and you can use it from time to time during incidents or analyzing further data that might drive you. Another critical trigger is user feedback, and we already touched on this.

But if your users are feeding back to you that a system is slow or unreliable, you're kind of a bit too late. And that's why we want the Alerts and the SLOs in place to be proactive and catch things before users see them. But user feedback's important. If you do have a support ticket comes in about slow system, you want to be fixing it. Never trust a user blindly, though.

This is where if a user tells you something slow, hopefully you've got your monitoring data. You go back to your production data and you can go and have a look at the traces. If they said it was okay yesterday or last week, we can compare the traces from yesterday and last week to what we're seeing. Is it always slower? Are they actually correct?

In what situations is it slower? Perhaps they're on a particular feature plan or category of customer. That means that certain customers are seeing this degradation. You should be able to go to your production data and ask it that kind of question and figure out what's going on. Another form of user feedback is more anecdotal.

So this is more for internal applications, I would argue, but perhaps you're just sort of in the office and from time to time you hear people maybe around the coffee machine just complaining to one another that System X is really slow. Just try not to use that because it's really crappy. That is actually useful feedback. Again, you need to treat it with a bit of skepticism. What you should do from that is go back to the users and get proper feedback.

Okay, why are you saying the system slow? Which bit are you trying to use? Figure out where the pain is for the user and then go back to your production monitoring data. To validate it once again. Sometimes it will be your own developer experience, your own intuition.

As you get more involved and more sort of used to doing performance work, you will start to feel how code might perform just as you're reading it. When I'm reading code now, I've done so much around optimization that I tend to see allocations as I'm reading code, which is both good and bad. I mean, it's slightly better than seeing dead people, but it's still. When you're reading that code, you kind of say, oh, there's an array allocation here, or there's an unnecessary string allocation there. This can be useful, but we really have to be careful here about premature optimization.

Just because you've seen something you think could, could be optimized slightly, you want to make sure that you don't just touch it for the sake of it and that you have a good reason to. Some really subtle, obvious changes might be okay if you're in an area of code and you're just cleaning up. But most of the time what we want to do is go, okay, well, this code will only get called if this request fires. Go back to your production data. Are those requests within our thresholds?

If they're really close to your thresholds and your alerts that you've set, then maybe that's a good opportunity to optimize that code. Otherwise, maybe it goes on to a future backlog or something. We also should be talking about performance within our teams from the point of designing a feature through to the acceptance criteria that we will check at the end. This isn't done often enough. I would say you might be tasked with a new feature and you might ask, well, how fast does this need to be?

What's the performance criteria? Most people might get back the answer. That doesn't really matter right now. We just need to ship this next week. We've got a deadline to hit for Customer X.

That's really not a great answer. Test it a little bit. Go, okay, well, if this new page loads in 10 seconds, is that good enough? And the answer might be, quite obviously, no, that's ridiculous. So for like, what is a reasonable threshold, even if you set that threshold quite high and you say, okay, well, 500 milliseconds should be enough for now, have that bar in place, have those metrics, have those SLOs set up before you deploy to production, and then you can see, are you at least achieving that?

You can always lower that threshold in time as you understand your customers and the feedback you're getting. But having something proactive in place before you deploy is really useful. And as you get more involved in this, this would be sort of more into the 3% scenarios, but you might have automated performance testing to catch regressions before you release. This would be load testing. This might be running sort of automated benchmarks before you merge PRs, that kind of stuff.

It's a little bit more complex. Doing these things repeatably and reliably is not easy. But you might find this as a trigger in some of your orgs. The important thing is we pretty much always go back to the top, back to the production data as the final proof that whatever we've seen elsewhere is a true trigger. So I've drummed in one last time.

Why does production data matter so much? Because your local benchmarks, your local testing, do not match real usage. It's very, very difficult to replicate real usage on a dev machine. And so the usage patterns you're seeing, the types of load, the types of concurrency, will not match. The GC mode might be different.

The caches that you're using might be set up differently. You might be using a Docker database instead of a real one with actual latency. None of that is actually going to tell you how does this thing work in production. It's great when you're doing the actual optimization work because you can compare like, for, like on your local machine. But when you want to actually see, does that translate to real users?

That's why we need production data. We need to avoid that. It was fast, locally biased that we can have that. So just to get into this performance optimization loop, then I'll describe the steps that I typically go through. There's actually several loops in here and we'll talk about each of them at the end.

Let's see what the steps are first. So we need that monitoring data. I always start with this one because this is what will tell us something's wrong. Even if a user has fed back something in a ticket, we want to be able to go to that monitoring data, look at the traces and figure out where specifically is the problem. Is it our service?

It might not even be our service. It might be a downstream one that suddenly got slower, or maybe a database is no longer returning as quickly and there's some new network latency we need to deal with. So it might not require you to do anything in your service. It might need notification of another team. So that observability data is that crucial point to figure out the starting point for your journey here.

Once you've identified it's in something, in an application, you own the code that you own. Then we need to go in and identify where in the code we need to focus. And this is where profiling is our friend because it will help us see where the pain is within that code. We profile the app, we exercise whatever it is the user was doing or the observability data tells us is slow, and then we get that profile data for CPU usage and memory usage to give us ideas of where we could optimize before we actually go and start any work around optimizing, have tests in place, unit tests, integration test, doesn't really matter. You just want to be sure that the methods and things that you've identified that you will optimize actually have good functional criteria around them.

Because the performance work that we do, as we get to the more and more nuanced code changes, the code gets harder to read, it gets harder to maintain and reason about. You might start doing sports span level byte parsing, and it's very easy to parse incorrectly by one byte here or there. So having tests in place first means we can assure that we're not going to break the actual functionality just by trying to optimize it. Then we're at a point where we can begin to measure the code. So we know now a few methods that we might want to focus on.

And this is where we use benchmarks to help us quantify the impact. We start with benchmarks before we do any work. These give us a baseline and then everything else we do can compare against those. And this is when we can now actually finally get into code optimization. And this is where I recommend that we apply small targeted changes as we go forward.

If you see five, six opportunities within a particular method that might make it faster, they might all be good ideas. But if you change them all at once, you can't really validate them individually. And you might find that overall you improve your performance. But maybe one of those actually regress things and you lost a bit of performance that you could have gained. So make the changes in small units, change one thing and do it as a sort of scientific approach.

And then once you're done, you can validate to make sure that you've got the gain that you expected. So you rerun your benchmarks and you rerun this test as well to make sure everything functionally still works. This does make the process quite slow. The feedback loop here isn't particularly quick, and that's why optimizing can have a performance cost, sorry, an engineering cost to your business and why premature optimization can be bad, but at least doing it this way, I feel that I get the best results. Another step that's often overlooked is documenting what you've done.

I do two forms of documenting, really. I track as I go through each of these little changes, how much gain I'm making. That's just really recording the benchmarking results in a notepad file with each set of changes. So I can kind of look back and figure out what's. What's giving me the biggest bang for buck in terms of what I'm doing.

But I also need to explain the complexity of the code I'm introducing. We might start with quite a readable method. It doesn't really require any description. But as we move to methods where we start parsing with spans or we start doing loops in an unusual way to get some slight CPU or algorithmic improvements, those can be really hard to understand just by looking at the code. So we want to comment it and say, why are we doing this?

And explain the what and why of that code and also flag that it's highly optimized for a particular reason. And if someone's touching this code, they should be doing so by running the benchmarks to validate what they're doing. So you don't want people regressing performance without really understanding where and why they're doing it. And finally, then we repeat, this is what makes these loops within this process. So the inner loop is this one.

This is where we're doing the method level optimization and we're basically doing a small change, validating it and then documenting what we've done and then looping back through. And we do this until we think we've made enough gains within that method. And it's not an exact science. You have to determine at what point you stop within here. And at some point you might think, well, we've done the obvious things, so maybe now we're going to look back at that profile data and target somewhere else and look for some other bigger wins in other methods on that hot path.

Once we've done all of the bits that we think might make a difference, and we think, okay, this is probably a good place to stop, then we drop it out to production and then we look at the observability data, as soon as we're deployed, we can start validating. Has this made the improvement we hoped? Most importantly, has it regressed anything because of some weird situation that in production there's a whole different scenario that we didn't factor into our code and then we just keep looping. So with that, we can kind of get into demo now, which, as I say, is about half an hour or so. And this is a real scenario.

So to give you the context on this, I've pulled this code out of the repository just to make it easy to share with you online. Ultimately, what I was working on here was I was working on the open telemetry repositories around the instrumentation code. So I work on the AP game team at Elastic. So my job a lot of the time now is looking at open telemetry stuff and where we can contribute there. And one of the things we wanted to contribute was some work around some features and also performance opportunities I saw in the instrumentation for SQL code.

So if you do a SQL call through Microsoft system data or one of those kind of APIs, then the instrumentation libraries of OpenTelemetry listen to events in those in order to add spans into your trace data that tell us, okay, yeah, the SQL call occurred here and it took this long. And so this code here gets invoked every time a SQL span is observed. And what its job is to do is to produce some attributes that will go out on that span. So attributes ultimately, sort of metadata that join the span. And the open telemetry semantic conventions define certain attributes that should be there.

And one of them is that you want the SQL statement on the span data, but most importantly, you want to make sure that you don't leak sensitive information. So there has to be a sanitization process of the statement that actually went to the SQL server to make sure you're not sharing literal data that might contain sensitive information, personally identifiable information, et cetera. You don't want that in your telemetry. Typically. The other thing that these conventions define is a way of abbreviating a SQL statement into a common shortened string that can be used for naming those spans.

We want span names to be quite low cardinality, and there's a guide for doing that. So ultimately, this is what this processor thing does. It's a functional method, takes in the original SQL statement and returns SQL statement info, which is basically just a wrapper around those two output strings that we're going to produce. This is the code as I came to it. So I worked on this end of last year at some point, and I came to this code.

This is exactly as it was at the time. So there's a cache involved. Because if you see the same SQL statement multiple times, it. It wants to not go through the parsing process each time. And then it has logic around skipping out comments from there, sanitizing the various literals and ultimately writing the tokens out to the final strings that it outputs.

And if we scroll down, there's 400 lines here, we're not going to go through the code in detail, but you can see the code is already reasonably optimized when I got to it, and it's not particularly well commented. So there's a lot of character level parsing and things here. So it's. It's not easy to read, but it's already fairly optimized because this is a scenario where we are kind of in that 3% observability tools, observability libraries should not affect your application performance any more than they have to. You don't want the observer effect, you don't want to turn on data collection for telemetry and find your memory usage doubles in your application.

So already this code is trying to parse that string once and be quite efficient in how it does it. That's pretty much what we need to look at in the code. As you can see, lots of code in there. So what do we need to do to begin optimizing this? Well, the first thing we talked about on the loop is making sure we've got test data.

Fortunately, there were tests in place already. So there was one theory test which takes in member data. It just runs that get sanitize SQL command and then asserts on the various outputs it's expecting. And the data for this comes from a JSON file. So we can see here for this particular statement, the sanitized version should strip the comments and the summarized version kind of just boils down to what was the main operation and what was it on?

So select from table and then in other scenarios here where we have these literals in the statement, they get sanitized to either one of these two forms based on semantic conventions. So we've got tests, so we don't need to worry about that too much. So the next thing is how do we profile this code in this SQL processor to understand when I call that sanitize method, what's actually happening, where's the time being spent and where are we allocating? So in the project here I've got this SQL Processor benchmarks project, which runs not only benchmarks, but also acts as like a profiler harness, which is kind of a nice way, I find, to just make profiling of my code repeatable. And it's not really that complicated.

So the project itself depends on benchmark.net, which is the the de facto benchmarking tool for. Net, and it uses also this JetBrains Profiler API. So I like the JetBrains profiling tools mostly because they have a NuGet package. I can drop in to control the profiling session. So in other profiling tools, typically what you will find is if you want to take a memory snapshot, you have to manually do it in the profiler while the app's running.

And triggering that particular point in time when you're measuring methods that take milliseconds or under is pretty difficult. With the API I can just say in code, okay, I want to profile this particular line of code. So the program for this is pretty straightforward. It's just a regular top level statement console app and it takes in one main argument which tells it which mode it's operating in. So these are the different profile modes I've set up for it.

And if it falls through to default that will be running benchmarks. And then this switch here basically handles the mode. So the one we'll look at first is CPU profiling. What this does is sets up this prepare for profiling, which is quite a low tech way of just warming up the code before we actually profile it. So I run five statements 100 times each through that method, force some GCs, force finalizers and things to clear out and just wait to let the system stabilize before we start measuring.

That's good enough for what I'm doing in this position. And then this is where the JetBrains APIs are. Great. So here I can say once I've warmed everything up, now I want to start collecting CPU data. So the profiler will have attached before to this point, but only when this method is called in code will it actually start profiling CPU cycles.

Then I call get sanitize SQL some number of iterations. What's important when you're doing CPU profiling is the most common way that you'll tend to do that is in some form of sampling mode. And when you're sampling every some number of milliseconds, the profiler will take a picture of all the call stacks and then use that data over time to work out which methods are calling each other and how long is being spent within them. Sampling is good for accurate call time measurements, but you need to run it quite a lot of times because if you have very short lived methods in your code, sometimes they fall between the samples and you could lose them. So to get good data, to be able to analyze this properly, we typically run sort of thousands, hundreds of thousands of iterations to get good overall averages.

And then when I'm done, I save the data so we can see what this looks like in trace. So in trace I set up a process that I want to run which is just my built benchmarks project. I'm passing in, I'm running in trace mode. I'm going to do 100,000 iterations. We use sampling mode to begin with because that gives the most accurate call time measurements.

But as I say, there are some trade offs with that. And importantly, let me zoom in in case you can't see it. We're using the API here which says that this application has Jetbrains API to trigger when things should run. So I'll click start, zoom out here and we'll let this run. So you can see after about three seconds there, it's done.

So 100,000 calls, pretty fast. And you'll see that actually profiling was quick. Building the data into a visualization is going to take longer. So it's done, we've got the results. And there's two main views here for cpu.

So there's hotspots which identifies which of the methods tend to take the most time from the code that we're actually profiling. And we can see that in here. The high percentage numbers here are code that we own. Our lookahead method is taking up quite a lot of time overall, as is our write token. Proportionally we can see how much time they take those methods specifically take on their own.

So that's the code within the method plus any code that they call down to. So write token itself is quite quick, but then it calls a bunch of methods that also add some milliseconds on afterwards. The other view is called tree, which is quite useful as well. So here I can jump into the main thread and program main and I can see that proportionally within that program main, how is that time broken down? And if I actually look here, this is my entry point that I'm really interested in.

So I'm going to scope to the that. And now I say, well treat that as my entry point that I'm profiling and caring about. So that's 100% of time. How do I break that down? Where's the time spent?

And again we can see that in here we call everything called Sanitize SQL and then write token is the big proportion of this, the write token and the code that it calls down into these look ahead methods are going to be the most valuable targets in terms of reducing execution Time. Now, there is another way that we can analyze profile data. So if I come back to dot trace this time, I'm going to do line by line. I'm going to use high accuracy. I'm still using the API, and I'm going to trigger this and we'll let that start profiling.

So line by line is different. It doesn't sample. What it does is inject IL code into the application it's about to profile so that it can count exactly how many times each line of code is getting invoked. As you'll see, this is now taking a lot longer. Previously it was about three seconds.

It's now going to take probably 22, 23 seconds to run. And that's because that's added a huge amount of overhead to the application we're measuring in order to give us those call counts. So we can't get accurate call time measurements because our profiling has affected them, but it will give us accurate call counts in there. And you'll see the main difference now is we have a huge number more methods in our hotspots. We can see all the getters being invoked and all the different things, both our own and system ones.

We'll see constructors being invoked. We'll see these little methods within system span helpers. So this gives us a much bigger overall picture. And the most important thing is now we also get call counts that are very accurate. So for 100,000 iterations, lookahead gets invoked 20 million times.

So about 200 times per statement on average. And that gives us an indication that if we can shave a few nanoseconds even off of that method, we can multiply the effect by 200 times because it's invoked so often on those call paths. So we can start to really understand where we might want to invest our time. Now, the other type of profiling will be memory profiling. And in the code here, this particular one is the memory one.

So again, I prepare for profiling. I then ask the profiler to start collecting full allocation data. So there's an API in the runtime that profiler's going to attach to to say, just let me know about every allocation as it occurs. That has some overhead and expense. So you typically only do this when you're on your development machine, not on a production system, as you profile it.

And then I also want to trigger some snapshots. Snapshots basically force a garbage collection and then can see, based on the GC info, what objects were still alive at the end of that garbage collection. Ultimately, which objects are living on which heap, etc. And in this case we just run this method once. We don't have number of iterations.

With memory data, this is generally okay. It's not always okay, and we'll see a case where it isn't in a minute. But what I really want to do is just for one method, call, see what objects are actually created and then see if any of those maybe could be removed from my code. So I'm going to come over to memory here. So this time I'm using this profile which runs the same thing with the correct argument using the API again and I'm just going to hit start and so we're only going to call that method once.

So done. That was really fast. I'm going to untick show unmanaged memory because we don't really care about that. We can't really massively influence it. And then this graph here shows over time the heap sizes.

So. Net's garbage collection and its management of the heap is built into this sort of generational model to optimize everything under the hood. And we can see how those grow over time. So this is all sort of startup stuff from the point we said start tracking allocations. Here is our before and after snapshot that we triggered.

So I'm not sure if you can really see it, but there's a small increase there in Gen 0 mainly, which is short lived allocations between those two points. So we know we're allocating a little bit. But these snapshots, this will tell us exactly what we're doing. So if I choose to add those both to the comparison, click View Memory allocations. I can see all the allocations that occurred here.

There's one missing that we'll come to in a moment. But the key thing is I can see there's not a lot of allocations, but I can see how many objects and their sizes. So let's start with SQL processor state 40 bytes, one object. Pretty straightforward and, and understanding why will be super easy because in our code we knew up an instance of SQL processing state or SQL processor state here it's a short lived allocation, it's just on the heap for this method and it's a class, so it's heap allocated. Ultimately what that stores is two string builders which are used to build up those output strings and some general state as it parses so it can track its position.

So that one makes sense. You may have some ideas about how you could remember remove that, but we'll come to that in A moment. The other ones here. So StringBuilder, that makes sense. We have new StringBuilder, so that's fine.

But we've got five and we could see that we created two, so that's a question mark. You can see there's 240 bytes there. We can equally see that we've got five char arrays as well. And that's because internally a stringbuilder is backed by a char array. So five stringbuilders need five char arrays, so.

So we're up to about 500 overall bytes there for those. So we want to understand why we've got five stringbuilders and we might expect only two. We can get a bit of a view by coming to the back traces if I click on StringBuilder, so if I click up here, I can change from byte view to object view. And I can see two of them are coming from my SQL processor state constructor, as I kind of expected. So that's fine.

But three of them are coming from this expand by a block. Where is that coming from? Well, that comes with append, with expansion, and that's on stringbuilder. Append character. So to understand these allocations, we need to understand how stringbuilders work internally.

I have a whole blog series on it if you want to go into the depths. The short summary is that when a stringbuilder is created with default constructor, it has a character array of 16 characters. A lot of strings you are going to be creating might go past 16 characters. So how does it handle that? The obvious design choice might be, well, we create a new character array that's bigger, we copy from array one to array two and we can just discard array one internally.

That would be a viable design choice. But copying as particularly as you get to bigger and bigger arrays can be expensive. So what the team did instead for rightly or wrongly is inside stringbuilder. What they do is create another stringbuilder that's double the capacity of the previous one. So the first one is 16 by default.

The next one would be 32 character array. There's a link or a property that stores which stringbuilder. The stringbuilder that you started with points to the next one. So ultimately a linked list of stringbuilders gets built up and then when you find the output, it just runs through all of those. So that's why we see these additional string builders, these additional char arrays, because of that process.

So understanding that gives us an idea of where we could Optimize, we have strings, so four of them. Now we know we want or we expect two, because we have two output strings that we're producing as part of our method. But we can go and see that. So those are coming from stringbuilder, but two more of those are coming from substring. So pretty much any function you call on string is going to create a new string because they're immutable types.

And so we know that maybe if we go and track down where the substring occurs and if we can avoid that, that's some other allocations we avoid. If I click back to byteview, you can see those are only going to take up 80 bytes. So it's not a huge saving. But in this sort of scenario, this 3% it matters/table I'm going to gloss over. This is a scenario where running this code one, this one line of code once is not sufficient.

Hash table has a lot of legacy overhead. It's an older collection type. It doesn't have this overhead per call of get Sanitize SQL, it sort of amortizes over time. So if I ran that method 100 times, I might not necessarily see those numbers go up. So you do have to profile under different conditions to understand everything about your application, but we gloss over that.

The one other known item that I expected to see and didn't comes from arguably a bug in dot memory. Arguably just the way it interacts with the APIs of the net sort of runtime. And it doesn't show one thing that I was expecting. So if I come into this other view, this is a bit hard to see at this zoom, but if I do compare, what I actually see is between snapshot one and two. It does those garbage collections and how many objects were created since snapshot one that also survive the garbage collection before snapshot two, which is ultimately the new longer lived objects.

So this tracks dead objects, new objects and survived objects, objects that existed before the first snapshot that are still there. So new objects will give me one additional thing. SQL statement info shows up here and if we go back to the code we can sort of. Where's my mouse gone? Mouse.

There it is. We can see. Oh, I think this table is causing me problems. We can see SQL statement info is our return type, so it maybe makes sense. But it's a struct and most people understand and appreciate that.

Typically structs are stack allocated, not heap allocated. So why am I seeing it on a profiler for heap data? And going back to the code we can we can see it, but it's not obvious. So we, we create the SQL statement info if we need to and ultimately at the end we're going to add it into the cache. But the cache is this hash table.

Hash table is an older collection type, it's non generic. So it's storing all of the things you store there in that cache as objects. To store a struct as an object, you box it onto your heap, basically wrap it in a heap allocated object. And that's why we're seeing the allocation. And it also means that when we pull it back out the cache, we have to unbox that struct, which is also a slightly expensive proportionally operation.

So this non generic type is also potentially something we could go after. So we've done all the analysis, we can now start to actually dig into some of the optimizations. But before we do that, we need our baseline data to go on. So the final part in this program file that's of real use to us today is that we in the default mode will run our benchmarks. So I pre create some config.

This isn't really important, I just hide some columns I don't really want to see right now. I tell it to run the benchmarks from the assembly of where this program class is defined, which ultimately is this benchmark one benchmark class here. Couple of things. Ignore this. This is for demo purposes.

These are commented out for demo purposes. So let's have a little quick look. So ultimately to write a benchmark we create a class. I've added memory diagnoser here to say I want memory allocation data for any benchmarks within this class and the actual benchmark is just the one I've attributed here. And all that does is execute the code we want to benchmark and it passes in the input string that we're going to test.

Now this params here on this property allows me to say to benchmark.net, run this multiple times with each of these different statements because each of these will have a different performance characteristic. Those with comments will go through a code path for stripping the comments. Those with literals will need to be sanitized. So there's different behaviors here that we want to measure. And now I can actually go and run the benchmarks.

So if I do a. Net run on here. Yeah, that's not too bad. Make it a bit bigger. So.netrun I'm using release mode.

I'm using no build here because I've already built it. I'm using I'm running my benchmarks project and I'm filtering it down to the benchmarks that we care about here. So I'm going to kick that off and benchmark.net is going to do a bunch of stuff to kind of start all this stuff in background processes and sort of low impact way as it can to measure the code and then we can just see as it goes on hopefully in a minute. This is slower than usual. Watch it.

Think for a minute. There we go. We're getting a. Eventually we get some data. So what this is showing us as it's working ultimately is it's doing a lot of steps that you don't need to really care about, but we'll just briefly touch on them.

So it does a bunch of work to prejip the code just in time. Compiler in. Net needs to take ultimately the IL code in our assembly and work out how it's going to prepare produce the machine code to run that most efficiently. So it does that to make sure that it's analyzing and benchmarking your most optimized code. It then does a pilot mode to measure its workload.

So what it's doing here is running that code multiple times with bigger and bigger numbers of operations until it gets to what it thinks is roughly a stable state. So it's worked out it needs to execute for each benchmark run, needs to run it 2 million times to get goods to statistical averages on the data it's going to give to you. It then warms up the overhead and measures its own overhead. So you can see there's only like less than 2Ns, but it's going to subtract its own sort of benchmarking overhead away from the code that you're actually measuring. So giving you the data about your code, not itself.

And then here, because we forced it in that attribute one run, it's now run the warm up for its workload, done the actual measurements and build the results. And it's done that for the two different statements that are in my code. And ultimately at the very end we're going to get a table for each of the two statements with the mean execution time and the allocations. And we can see larger inputs. It kind of makes sense, will take longer to run and in this case have larger allocations.

Now we're going to come back to this, but I'm just going to do a git reset, git reset on here and I'll just show you the actual date. So this is as the benchmarks were. I Didn't have that test attribute. I only did it for time here. And so the full results are here.

So we have all of the statements, their execution times and their allocations. So nothing's massively slow. Like the longest is 700 nanoseconds, which is pretty fast still. But it is allocating 1800 bytes. And this is overhead that we want to know if we can remove.

So in terms of how we are going to do that, I'm going to do show you the stages that I went through. I'm not going to actually do live coding because we wouldn't have time. This is days of work condensed down into now 10 minutes hopefully. So I'm going to check out stage two. So the first thing I wanted to go after, my first theory based on profiling was that that stringbuilder expansion is a problem that's costing me quite a lot of allocations.

So how can we improve that in the code? Well, the obvious way is that rather than creating a stringbuilder with the default constructor, I'm now going to new them up here with a specific pre sized char array. Internally, the size I'm telling it to use is the size of the input string because the outputs are never going to be bigger than the inputs. They're only either going to be sanitized shorter or condensed. So this guarantees that my stringbuilder is definitely big enough for what I want to produce at the end.

That's really the only real change in here. And if we then go to the results. That's not the results, the results. So here's the new data for stage two and then I've calculated myself the differences between stage two and stage one. So the key thing is that we've shaved off about 8% of the CPU time.

And the way we've done that is we're no longer calling append, we're no longer calling it spanned by a block and all of those internal string builder methods which execute on the cpu. So we're basically removing code that needs to run. So we've got a little bit of a gain there. The main thing we wanted was allocations and on average about 18% reduction. Most of these are positive gains, a negative number, but one didn't change at all.

And specifically one stands out as actually regressing. So for this particular statement at 69 characters in length, what this likely means is that the sanitized version was really short and fit into 16 characters originally and add with a summary as well. And so in this situation we've actually forced it to allocate 69 characters internally, but that's bigger than what it was doing before because it never needed to call, append and expand, etc. So there's one regression, but overall we've made a good balance. And as we'll see, this is going to become moot in a moment because we'll do something about that as well.

So if I come back and check out stage three, what the difference is here is again thinking about how this code gets invoked. Ultimately what we do is listen to kind of legacy events from the SQL libraries that tell us the SQL is being executed on the server. Most of those events will hit us on a single thread and ultimately be sequential. There will be subtle cases where there could be some concurrency, so we have to factor that in. But what it ultimately means is we don't really need a string builder per statement we process, we could reuse one.

So here I have a static stringbuilder for each of the two strings I want to produce pre sized to a thousand. I also have these integers here, which ultimately are how I'm going to manage concurrency. And down here, down here we just use atomic operations here to see if that value is already 0, and if it is 0, increment it to 1 within one thread. Safe Atomic operation. If that happens, then we know we can use the shared instance, that we've got, the static instance.

I clear it just to make sure I'm not going to be starting with any existing data and then we're going to use that. Otherwise the fallback path is what we did before, create the string builder. We do that for the two that we need and then right at the end it's going to flip that bit back basically and say I'm not using this anymore. So assuming all of these spans come into us roughly sequence, eventually we're always just going to use the shared instance, hopefully. And that's typically true under the profiling examples I ran into.

So we can go back to the results here with that new theory in mind and see, okay, execution time, very little difference. All we've done is trim a constructor call basically. But allocation wise, massive change because now we're not allocating those string builders, those char arrays that we don't need. So 57 odd percent allocation reduction. So these allocation numbers are already looking pretty good.

Step four is another thing we saw in the data is string substring was causing string allocations. That's an obvious quick win if we can avoid it. So I'll come into the processor here, I'm just going to show the diff here. It'll be a lot quicker. So we should compare that to previous.

Ah, sorry. No, I'm not doing. No, I'll show you that in a minute. Apologies. What I did first is tackle the cache because the cache was another obvious example.

I know I'm boxing, unboxing and boxing operations that cost time. So what I've done is basically changed to concurrent dictionary. So this is good because it's a generic collection type. We no longer have the need to box those structs. Also, it's a newer collection type, so it's ultimately been optimized by the.

NET team a lot more than hash table has. If you read Steven Taub's blog posts every year, they're always trimming bits of time off of these collection types. So I'm using that. The only oddity here is this approx count. So what I could do is say concurrent dictionary, give me the count of how many items are in here to limit us to our predefined capacity.

The problem with count on a concurrent dictionary is it has to be thread safe. And because of that there's a little bit of execution time, overhead locking and things that happen behind the scenes. What I do is track an approx count. And so all I do here is read that count with a volatile read to check if we're okay to if we're over capacity or not. If we're not over capacity, we try and add the item in and increment the count.

There's a subtle risk, if we had a lot of concurrency here, that we might, when we come into this, we might already have 1,000 items and so we might end up with 1,001. But I'm okay with that little minor trade off for the small optimization I get. So the benchmark results here, execution time improvement. So that's just really from coming from going to that more modern collection type and avoiding boxing and unboxing in the code paths, allocation difference. We're not seeing the reduction from that type.

We're no longer boxing. That immediately feels strange. But you have to then run this code and profile it and understand what's actually going on. So for the benchmarks to work, I disable the cache because if the cache is enabled, I'm never executing most of the profile code because it's going to run that code 200 or 2 million times, a number of operations per each iteration it does. So I turn the cache off so we don't actually go through the caching path.

Here I created a temporary benchmark to check that and I also used profiling to tell me what was going on. So in the profiler I could see that I removed that heap allocation, which is good. So I saved 32 bytes. I think I increased my allocations by 4 bytes because using concurrent dictionary, each entry takes 36 bytes of overhead. So there's actually a tiny, tiny 4 byte regression.

But the performance of moving to a more modern type still pays off overall. So now we can do stringbuilder as substring, which is, as I say, a super quick win. So if I now do history compare, it's basically a one liner. So the original code, when it needs to to trim a little part of the original statement to then append it uses substring. So this is a short lived string allocation ultimately and it gets appended and then we don't care, we throw it away.

But now in modern net we have things like the span APIs. Again we're not going to the detail of them, but spans. Let us look at the existing string memory and just trim down to the portion of that memory we care about by slicing it and then we can append that directly so we don't have to create the temporary string in order to append it. We can just say append this portion of some existing memory that you have. So that's all we need to do.

And then in the benchmarks we can see very little, some minor fluctuation, but at these level of nanoseconds you will see some fluctuation. But ultimately all the code paths that did end up needing to go through that, all of the input statements that went through that code path, we see another, on average 20% of reduction in allocations by removing the temporary strings. Now stage six, this is the exciting one. This now represents quite a few days of work and also a bunch of feature requirements. So I've kind of bundled it all together because we can't really.

I'd love to show each thing, but it's just going to take too long. So to demonstrate how different things are, I'll just show. And this is why I've put the code online. If you want to look at all of the individual differences and find probably some new ones that you can optimize, you can do that. But if we look at the diff, it's basically all new.

So I've kind of rewrote the method. At this stage I'm not going to go through everything, but I'll give you some highlights, some subtle differences so there's a whole bunch of stuff. There's new types in. NET called search values, which allow you to very efficiently search through sequences of data. There's a bunch of kind of meta information that I build in around the different keywords to avoid using strings when I can avoid them.

Most of this isn't that interesting. The key thing will be down here. So I've removed StringBuilder entirely now because that still has overhead and complexity around locking and all of that stuff. Instead I just rent an array of characters. Renting from an array pool is super cheap.

It's basically no overhead because that pool builds up over time and you're just borrowing something from the pool. Generally I rent double the length I need because rather than renting two arrays, one for each string, I just rent one array and I work within different portions of it. And then everything is done as span APIs now. So I get the span representing my buffer and then I process through char level sort of slicing and operations to manipulate that data. The main sort of functionality code is similar there, but you'll see that each of these methods now look quite a lot different because they're working with these slice APIs and things versus the original APIs that went through the array with characters directly.

At the end, we just slice the output data and produce the final strings. Down the bottom here we've got a few significant changes. The key one is SQL keyword inf. No, not that one. It's above that somewhere here.

Parse state. So previously it was a different name, I renamed it. This was that class that we had. So for every statement we had to create that temporary state, but it was short lived. I've changed it to be a struct, specifically a ref struct.

So it's guaranteed to never be heap allocated. And I've added all my state in here, so it tracks the temporary buffer and a bunch of additional data because of the new features. So it's quite big. Typically a struct over a few fields, over four fields or so, wouldn't be particularly efficient because structs are passed by value, not by reference, typically. And in that you have to copy all of the data on each method call you pass it into.

The way we avoid that here is we use ref or in keywords to avoid the copy and pass it by reference directly into the next method. And because I own all the code, I can do that. And then the only other real change in here very quickly is this keyword info is a bunch of metadata that allows me to algorithmically optimize how we process the code. So for each keyword I track, the most interesting one is which keyword I care about afterwards. So previously I parse the select keyword and then I'd go through my array of other possible keywords we care about and check each one and see if my next few characters match it.

Now I'm saying to the code, well, if you see the select, the only one I care about is from. So I only have to check the next character being F and then I can drop out of the loop more quickly. That's kind of all we have time to go into, but the results speak for themselves. So we now have another 35 or so percent of CPU time dropped by moving mostly to spam based APIs, removing string builders, et cetera, and on average another 15% or so of my allocation. So this is compared to stage five, not from the beginning, this is another new 15% reduction.

And then I thought I was done at this stage, but there was one thing that kind of popped into my mind. So I looked at the profiling data and I said I have got rid of all the allocations pretty much except these two strings which are my outputs, so I can't really do anything about them. But then I realized that sometimes the statement that I get in when I sanitize it won't change. If it doesn't have literals in it or comments, it's not going to change. So why create a new string to return that new string out?

I could just return the original input string that I got. And so that's what we basically do in stage seven. So in here probably use this just to jump up to here. Where are we here? So I slice to get the built up sanitized string.

I then use this sequence equal thing to compare my sanitized SQL data to the original characters that came in from the string that we got originally. And if they're the same, I just return the original strings value. Otherwise I now still need to call tostring. And so ultimately what that means is that these two columns in the middle are compared to previous stage. So execution time not very interesting.

But in the cool paths where the string was the same, another sort of 60, 70% of allocations reduced because of one extra string I didn't need. Overall from where we began, about 46% of our CPU time reduced and about 86.5% of the allocation data. So going through that loop gave us some quite good gains. So I'm going to try and wrap up Everything else quite quickly. You don't really want that view.

Why is that? Sorry. Quick recap. If you're really quick at reading right at the end, one thing I did want to call out. And when I first gave this talk in January, I was still quite AI skeptical.

Today I'm like, in an AI terminal most days, AI can help you with this stuff, particularly if you're new to it. So a lot of the stuff I did in stages one through five were ideas I just had. I could change the string builder to, you know, cache it, and I could switch to concurrent dictionary, etc. And I was implementing those ideas and testing them. But I got to a point in stage six where I was like, well, I can't see any obvious gains.

So I just said to copilot, give me 10 possible optimizations for CPU and memory I could do to this application. And ultimately it gave me those 10 answers. I then scanned through them. Two of them were insane. Disregarded those.

One of them hallucinated an API that doesn't exist in. Net. Disregard that one, but seven seems plausible. And so all I said to the AI was okay, implement option two, made the code changes. I then manually triggered the benchmarks and test runs.

And if the, you know, the numbers were improved and the test still passed, I was like, okay, that's a good idea. And then I just said, okay, now try three. And I can always get the AI to back out for ones that didn't make a difference or introduce too much complexity and didn't seem to offer the gain. Now. NET does in Visual Studio include this profiler agent?

When I used it eight or so months ago, it was a bit raw, it was a bit new. It might be better now. I found it didn't get everything perfect, but the idea of it is instead of running that loop yourself, you can just say, hey, here's some code that's slow. Can you profile it for me? It runs a CPU memory profiler, looks for the hotspots in those methods, creates benchmarks, runs them, and then checks the benchmarks at the end and then gives you the final answer.

As I say, a bit raw at the time, but something to look at. I'm going to go like one minute over, if you're okay with that. So where do we spend our time? Where users and machines are spending their time. That's what we want to optimize quick wins, hot paths and avoid anything that's just cold code, startup code, and avoid hypothetical improvements.

That's where we drift into premature optimization. We need data to govern what we should be doing. And the hardest problem, I find is knowing when to stop, because it's quite addictive. Watching those numbers go down is fun, but when you start to see diminishing returns, that's when you should start thinking, thinking about moving on. Ultimately, if you were trying to improve some SLO or some metric in production data, once you've made enough changes, deploy to production and see if those have improved enough.

If they have, you're done. If you made a clear user impact, you've closed the user ticket because you've improved. The thing they said was slow, you're done. If you were trying to reduce costs, hopefully you had a goal. But if you reduce 5% of costs, you're done.

If you were trying to improve reliability and you have, you're done. The goal is not the fastest code possible. It's code that's fast enough, but easy to maintain and delivering good results for you. I'm going to skip through this very quickly. Use monitoring data, dashboards, alerts, SLOs, et cetera to be able to see what's going on.

It's not about clever tricks when we're doing performance work, it's about just having those feedback loops in place. If you're really advanced, you might want to do regression detection before you release. Automated benchmarking, automated load testing. Make sure performance is part of normal delivery. So you set your Alerts and your SLOs before you release based on what you decided as acceptance criteria and you review that once you release.

Make this shared ownership. It isn't just an individual within your org or a team that should be your performance expert. It's all of you thinking about where performance matters and applying it to the right levels based on the data. It's ultimately continuous. Right?

Regressions will happen. Only tooling and discipline will help you. So takeaways. It's not a crisis response. You should be using this as a discipline.

Be data driven production data. Use the optimization loop aggressively, focusing on hotspots. Balance between pragmatic code changes to get the performance you need and build ownership. So I'll give you the links at the end there. Thanks.

Thank you for staying two minutes longer and that's it.
