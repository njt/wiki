---
url: https://gist.github.com/njt/85282efdbb67eb9e1219bdf1078d184a
date_fetched: 2026-08-07
---

# ytx: AI-Powered Gamification for the Web - Courtney Yatteau - NDC Copenhagen 2026 — NDC Conferences

Source gist: https://gist.github.com/njt/85282efdbb67eb9e1219bdf1078d184a



======================================================================
## FILE: 20260807-ai-powered-gamification-for-the-web-courtney-yatteau-ndc-copenhagen-2026.md
======================================================================

### Key points
- **Shallow gamification is hollow and counterproductive.** Points for trivial actions, meaningless badges, generic feedback, and random difficulty feel manipulative and erode trust. Good gamification must be layered on an already valuable experience.
- **Good gamification rests on four pillars: visible progress, relevant feedback, appropriate challenge, and meaningful reward.** Examples: Duolingo’s streaks (progress), Khan Academy’s mastery badges (feedback), adaptive difficulty (challenge), and Starbucks’ tangible rewards (reward). These make the user feel seen and motivated.
- **AI should be a small, focused enhancement, not the whole feature.** The speaker repeatedly emphasizes that AI is just one decision in a loop—generating a fact, classifying sentiment, or predicting difficulty—and the app itself must be solid without it. “Not bigger, not better… not tons of AI everywhere, just AI in places that we want it to feel more engaging.”
- **Three practical AI patterns for gamification:**
  1. **Dynamic content** – Use hosted AI (OpenAI) to generate fresh, structured content (facts, prompts) tied to user context, creating replayability and lightweight personalization.
  2. **Sentiment-aware feedback** – Use cloud sentiment analysis (Azure) on a single user reflection to tailor the tone of the app’s response, making the user feel heard.
  3. **Adaptive challenge tuning** – Use a tiny browser-side model (TensorFlow.js) to predict the next difficulty based on correctness and response time, keeping pacing tight without server calls.
- **The reusable mental model: Signal → Decision → Response.** Collect a simple signal (click, answer, timing, reflection), let AI make one focused decision (generate, classify, predict), and then visibly change the UI (reward, feedback, difficulty shift). This loop keeps AI’s role constrained and the experience intentional.
- **Combining patterns creates a cohesive gamified experience.** The final geo-explorer app weaves together dynamic quiz questions, adaptive difficulty, sentiment check-ins, badges, map unlocks, and a session recap—all driven by the same signal-decision-response pattern, with AI handling only the parts where personalization or generation adds value.

### Pithy and provocative quotes
- “Sometimes you get points just for, say, breathing. You basically just open up an application, you tap around for a few seconds and maybe it'll just hand you some points like you just accomplished something, but you really didn't do much of anything.” — *On shallow gamification.*
- “AI does not need to be the whole feature… It can just play a small role in something that already exists. And that thing that already exists should be great on its own.” — *Core philosophy of the talk.*
- “Not bigger, not better… Not tons of AI everywhere, just AI in places that we want it to feel more engaging for the users.” — *Reinforcing the minimalist AI approach.*
- “The user really does not care that there's some kind of model happening in the background. It's just the experience is all it is for the user itself.” — *On the adaptive difficulty demo using TensorFlow.js.*
- “The response shape really matches what the interface actually needs. That's the main reason this demo really feels usable. And it's not just the model is good at writing or whatever.” — *On the importance of structured output from AI for dynamic content.*
- “If you are trying to gamify some kind of experience or application, of course the thing that experience or application on its own should have great, meaningful behind the scenes itself without needing that gamification.” — *Precondition for any gamification effort.*

### Tools, practices, and methodologies
- **OpenAI Responses API** – Generates structured, multi-field output (headline, fact, reward label) for dynamic content. The speaker uses it to create location-based facts with a strict schema so the UI can consume the response directly without parsing guesswork.
- **Azure AI Sentiment Analysis** – Cloud service that returns overall sentiment, sentence-level sentiment, and confidence scores (positive/neutral/negative percentages). Used to classify a player’s free-text reflection after a round, enabling the app to choose a supportive or celebratory response tone.
- **TensorFlow.js** – Browser-side machine learning library for small, local predictions. The speaker trains a tiny model on correctness, response time, and current difficulty to predict the next challenge level, avoiding any server round-trip.
- **ArcGIS Geocoding & Reverse Geocoding** – Grounds user-selected or current-location inputs to real places, providing accurate coordinates and place names. Used in the dynamic content and combined app to ensure generated facts are tied to a specific location.
- **ArcGIS Maps SDK for JavaScript** – Renders interactive maps and static basemap tiles. In the combined app, map style changes serve as a visual reward unlocked by streaks of correct answers.
- **Signal → Decision → Response loop** – A design pattern: capture a simple, observable behavior (signal), let AI make one narrow decision (generate/classify/predict), and then update the UI in a way the user can feel (response). This keeps AI’s role constrained and the experience intentional.
- **Structured output schemas** – When using generative AI for gamification, define the exact fields the UI needs (e.g., headline, fact, reward label) and enforce them via the API. This prevents the model from returning unusable free text.
- **Small, focused training data for browser models** – For adaptive difficulty, the speaker used a very small dataset with just a few features (correctness, time, current difficulty) and tuned thresholds manually to get the right “feel.” More data would improve accuracy, but the pattern works with minimal inputs.

### Unanswered questions and omissions
- **Cost and latency at scale.** The talk uses hosted APIs (OpenAI, Azure) without discussing pricing, rate limits, or latency implications for real-world apps with many users. How would these patterns hold up under load?
- **Privacy and data handling.** Sentiment analysis sends user reflections to Azure; dynamic content sends location context to OpenAI. The talk doesn’t address user consent, data retention, or GDPR considerations—especially relevant for a European conference.
- **Bias and appropriateness of AI-generated content.** The dynamic fact generation and postcard images (DALL·E) could produce inaccurate, culturally insensitive, or inappropriate content. No mention of guardrails, content moderation, or fallback strategies.
- **Failure modes and error handling.** What happens when the AI call fails, returns malformed JSON, or times out? The demos appear to assume happy-path responses. Production apps need graceful degradation.
- **Measuring effectiveness.** The speaker asserts these patterns create more engaging experiences, but provides no metrics, A/B test results, or user research to back the claim. How do we know the gamification actually improves retention or satisfaction?
- **Accessibility and inclusivity.** No discussion of how dynamic content, sentiment-aware feedback, or adaptive difficulty might affect users with disabilities, non-native speakers, or those with different cultural expectations around feedback.
- **Over-gamification and ethical concerns.** While the talk distinguishes shallow from good gamification, it doesn’t explore the line where even “good” gamification becomes manipulative or addictive. The Duolingo streak example is mentioned positively, but its psychological pressure is not examined.
- **Production readiness of browser-side ML.** TensorFlow.js model training happens on page load; the talk doesn’t address model size, loading time, or how to update the model without redeploying the entire app. The demo’s training data and thresholds are hand-tuned—how would this be maintained?
- **Combining multiple AI services.** The combined app uses three different AI providers. The talk doesn’t discuss orchestration, dependency management, or how to keep the user experience coherent when multiple AI decisions interact.
- **When *not* to use these patterns.** The speaker says AI should only be used where it adds value, but doesn’t give criteria for deciding when a feature is better off without AI. What signals indicate that a simple rule-based system would suffice?


======================================================================
## FILE: transcript.md
======================================================================

# AI-Powered Gamification for the Web - Courtney Yatteau - NDC Copenhagen 2026

- **Channel:** NDC Conferences
- **URL:** https://www.youtube.com/watch?v=GOXxAUtrB8c
- **Duration:** 52m 15s
- **Transcribed:** 2026-08-07

---

All right, well, hello everyone. I'm really excited to be here today. This is actually my first time at NDC Copenhagen, so I'm very happy to be here. Yeah, thank you guys for joining me. I'm really excited to talk to you guys about AI powered gamification for the web and really some practical ways that web developers can use AI to create more engaging experiences.

So the demos today, I will say they're all going to be web based, but really the patterns behind them can be used really in any type of application across many different types of apps. So real quick, show of hands, how many of us have ever gamified an application before? Nice. Awesome. Okay, so we've got a few of us.

Great. All right, well, hopefully some of these patterns will be familiar to you As I go through all of these different. I have a few mini demos and stuff that we'll look at. Okay, so now before I get into all of the good content, I just want to briefly introduce myself. My name is Courtney Yato.

I am a developer advocate at esri who's heard of ESRI before. Hey, we got a few. Okay, awesome. They are a GIS company, Geographic Information Systems. So you can think of it as like maps, location intelligence kind of stuff.

So yeah, I've been there for about three and a half years previously. Before that I was actually a high school math and computer science teacher and I did that for 10 years. And honestly, that is really where a lot of this topic comes from, is my experience in the classroom. I use gamification quite a bit to help get create more engaging experiences for my students. And that was before the big AI boom and everything.

This was from 2012 to 2022. So having the ability to actually then utilize AI inside of web applications has really been pretty cool to create even more, say, personalized experiences. So we will get into that. Oh, and one other little note, I do live in the Washington D.C. area, if you're curious.

All right, so here is going to be our roadmap. And right away I went ahead and gave you guys the opportunity to pull up the demos and slides if you would like. I will also have this again up at the end. Yeah. So if you want to follow along.

So here is our agenda. So we're going to start with the gamification side of things. So just to really kind of make sure we're all on the same page about what gamification actually means. And then we'll look at three practical AI patterns that are each useful on their own. But what we'll do is we'll take each three of those different patterns and combine them into one full fledged application.

And then finally after we get to that app, we will zoom out and we will actually talk about how I would actually build something like this in practice. So a few extra little wrap up tips there at the end. Okay, so first, before I get into why maybe you should gamify things or you know, why gamification is awesome, I do want to address kind of elephant in the room of like maybe why gamification might feel a bit shallow. Some people a bit skeptical sometimes. So sometimes it is just for.

Sometimes you get points just for say, breathing. You basically just open up an application, you tap around for a few seconds and maybe it'll just hand you some points like you just accomplished something, but you really didn't do much of anything. Right. I'm sure we've all seen this in certain kinds of apps. Maybe fitness apps or learning apps, productivity apps, whatever, whatever it may be.

So something like seeing a score go up but not really feeling any kind of earning, that could just feel kind of silly. All right. The next is badges that really feel kind of disconnected and they're not really anything meaningful, right? So maybe you could get a badge for just getting started in an app or just showing up for doing some kind of tiny action. So really the app is trying to make you feel like it's a meaningful moment, but it doesn't really mean anything.

Or maybe feedback is a little bit too generic. Right? So let's say you will see some dynamic difficulty in a bit. But let's say you miss a question, you struggle through maybe around, and the app still gives you exactly the same kind of feedback as if you had been doing really well. Right?

So there's really maybe no personalization there in that example. And then finally again, the difficulty level, sometimes maybe a challenge might feel kind of random. So if you're working through especially something like learning application, if it's not working with you to progress, then what's kind of the point? You'll probably give up if it's not keeping you engaged and challenged enough. And honestly, AI features in general can just feel this way.

So really I want, I'm hoping that these examples that we go through, they don't seem too gimmicky and not like fluff really. Hopefully you can see how these features will help make an application better. And another thing I'll say is if you are trying to gamify some kind of experience or application, of course the thing that experience or application on its own should have great, meaningful behind the scenes itself without needing that gamification. Right. So it should actually be a good app and then you just add a little bit extra to maybe hooked on some users.

All right, so if shallow gamification is the version that we're all kind of rolling our eyes at, what is good gamification? So the first thing about good gamification is it does actually make your progress visible. Right. One good example of this is Duolingo. They do a really good job with streaks.

Those work pretty well in terms of. At least. I've never actually used it myself, but I've seen a lot of my friends use it. So yeah, Duolingo actually literally describes it as a tangible, measurable number that helps keep learners accountable day to day. Another example of this is also LinkedIn learning.

They do this pretty well too, with progress tracking and weekly goals as well. So it's nice because basically you're able to obviously keep track of. Am I actually getting anywhere with what I'm working through? The second thing is feedback. So good experiences do not just react, they react in a way that actually feels.

Feels relevant to the user. Right. So a good example of this is a learning app called Khan Academy. They do a really good job with having badges and a mastery system to work through. They're tied to actual learning milestones within the application.

All right, the third piece is a challenge or reward. Right? So this one really does matter because if something is a bit. Oh, sorry, actually I mixed those up. Challenge and reward are two separate things.

The third piece is going to be challenge. This one does matter because if something is too easy, like I was saying a minute ago, people can get bored. You're not going to want to keep working on something if, say, maybe you're not being challenged enough. Or it could be the opposite effect too. If something is too difficult, potentially you're going to want to quit.

So yeah, so you want to keep the pacing nice and catered to the user's actual experience. And then finally we do have the fourth piece, which is reward. And when I say reward, I'm not necessarily meaning little things like confetti. Really good example of this. At least more so within the US But Starbucks does a good job with their rewards program and they keep you coming back, right?

They keep saying like, hey, come make some more purchases and you'll gain so many bonus stars or whatever it might be. So actual, physical, tangible rewards and doesn't always have to be that way. It could be. That's more of an Extrinsic motivator, but it could be something that's more intrinsic as well. So we've all got those motivators in us.

So. So when we talk about gamification, we've got progress, feedback, challenges, and rewards. Okay, so AI gamification, it really is pretty practical now. And why is that? So one reason is that hosted AI, it has become obviously pretty easy to implement within your normal web apps.

So if you want to generate something that's dynamic, or maybe if you want to classify the tone like we'll see in a little bit, or get back some structured output you can actually work with, then it's pretty straightforward. And I'll prove that with our first example in the dev tools in a little bit. So, yeah, so it's become a lot easier to implement within your applications, and it doesn't take a huge ton of research either to make it happen. All right. The second reason is that browser side AI, which we'll see that with a TensorFlow demo in a little bit, it is quite useful for the right kinds of tasks.

Right. Not everything does actually have to point to a service or a backend. So if your task is nice and small, you can keep it local. You can do things like prediction or adaption just right there in the browser. All right, and then this really is an important part to point out.

So for a lot of apps, AI does not need to be the whole feature. And that's. I'm going to keep emphasizing that. Right. It can just play a small role in something that already exists.

And that thing that already exists should be great on its own. So AI really can just help with maybe like one little decision or one little extra portion of the experience. All right, so for this talk, you know, this is really the mindset. Not bigger, not better, or not bigger. Definitely a little bit better, though.

Not tons of AI everywhere, just AI in places that we want it to feel more engaging for the users. All right, we're getting close to the first demo, but there are a few pieces of vocabulary I want to point out as well, that I will be pointing through each one of these micro demos that we'll have, or mini demos. So the first thing is a signal. So a signal is just something that the app notices. This could be something like the tone of the user, the timing of which they're interacting, correctness of a question, or maybe even a streak.

It's really any kind of behavior. Number two is decision. So this is what AI really does with the information. So from that signal, so maybe it'll classify something or generate something, summarize something or maybe even adjust on the fly. Then we've got response.

And this is what obviously the user will actually see change in the full experience there. So this could be some form of feedback, reward, maybe even next challenge level, that kind of thing. All right, so, yeah, I like framing it like this because I think each one of these demos will be easy to see kind of the full picture from, you know, start to finish. You've got signal, decision and response. Okay, so pattern one, this is going to be associated with the first demo, is dynamic content.

And this really is where we can see gamification become alive right away. It's really important because a lot of experiences can kind of just say fall flat, maybe because they feel like they're too fixed. Right. So when you have the ability to create this dynamic content, it can feel way more alive. So the goal is not to generate more content for the sake of really is to create fresh content, fresh prompts, fresh facts, variations on things.

And so, yeah, variety is very important. Second, it creates that nice replayability and then third, it gives you that nice lightweight personalization experience. So you may be playing through or working through an application that's catered to your experience and your friend may be doing the exact same experience as well, but it feels completely different because it's being, you know, cater to you, that personalization element. Okay, so let's talk through what this we're going to see in the first demo for this dynamic content. So imagine that you're exploring somewhere new and the app is trying to make the experience feel just a little bit more surprising and game like.

So it's going to start with a few simple pieces of context. So here's our signal. Say, what did I click? What kind of fact do you want? And what place are we actually talking about?

So you're in a place, we're going to generate a fact about that place. All right? So then AI does that actual decision making. It's going to generate that fact for you about that place. And just a heads up, I will be after each one of these demos, I'll be saying, you know, what pieces like libraries and SDKs and various things we're using to build each one of these.

But I will go ahead and tell you right now that I've used OpenAI for this first one. All right? And then we have response. So this obviously is the payoff that you actually then get to see the fresh content. And there are going to be little gamified elements of Rewards in there too.

And of course, we'll be able to see our progress as well. As we generate more facts, there's going to be some things that pop up that allow us to see the experience grow. So let's check it out. Okay. So we are starting with this fun little demo.

Maybe I'll zoom in a little bit. Okay. All right, so it's pretty simple. We choose a place, we pick a vibe, which could be fun, spooky, or historical, and then we get to generate that fact. So that's the surface level so far.

So let's try it out. I'm going to go ahead and type Copenhagen Zoo. Okay. And notice right away we are getting some of these suggestions here. So it is using an ArcGIS geocoding service, just so you know.

So we got all these different suggestions here kind of coming through. All right, so great. We've got the zoo. And do we want to do fun, spooky or historical? I kind of lean towards the spooky.

So let's just try it out. I'm going to go ahead and click generate fact and let's see what we get. All right, nice. Okay, so it says, after dark at the zoo. Copenhagen Zoo is calm by day, but after closing, old enclosures and winding paths can feel strangely watchful in the mist.

In a place built around living creatures, every Russell seems to carry a hidden story. So not too spooky, but it's got a little bit of a spooky vibe to it. Now, some things I want to point out. We have generated one fact. We have 10 XP right here.

So for each one of the facts, we're going to gain 10 XP. We are on level one, and this little hint here says map locked. So that's interesting. If we keep reading, it says progress. All right, Nice start.

One more fact unlocks the map snapshot. So if we generate another fact, we realize, okay, we're going to get something called a map snapshot. So it gives you a little bit of that incentive there to kind of. Okay, let me try to generate another fact. We also have a badge earned first find.

So our first fact has been found here, and then it does actually keep track of all of your recent discoveries as well. All right, so before I continue with generating a new fact and showing you the map snapshot and everything, let me open the dev tools real quick and I will go to the network and I'm going to go ahead and generate. Well, I'll go ahead and generate another fact now so we can just see it come up right Away. Just use the same location. Okay, cool.

So I'm going to open up the responses now. The key thing here really is just we're seeing the response come back in a nice structured way in the preview. So that is under here. We've got our output and then we've got. Open this up a little bit more.

And then we've got our zero content zero. There it is. Okay, this is the key thing right here. So under the text of the output, you know, the content output, this really is the exact format that our app is expecting. Right.

So we've got a headline, we've got our fact, and then we've got our reward label. So again, thing I mentioned earlier makes it nice and easy because the structure comes back in a very simple way. All right. Okay. So continuing.

So I did when I went ahead and generated a second fact as well. So let's read a little bit about what happened here. We have map snapshot unlocked. One more fact unlocks postcards. So we'll have to do one more in a minute.

But we did get a badge earned. That was map reveal. And we scroll down and we see a fun little scroll screenshot of that exact location of where the zoo is. So just gives you a little bit more context to the area. Okay, I'm going to go ahead and generate one more fact.

We can. One other thing I'll point out. We could use my location as well. Let's see. I think I'll have to enable that real quick.

Oh, I guess I already enabled it. Okay. It determined exactly where we're at right here. So let's generate a fun fact about this exact location. All right.

It says we are in the city's oldest street grid, where it meets modern design energy. A short walk away, you can trace canals, bikes and architecture in one compact urban loop. So seems a little bit generic, but it's a fun little fact there. Okay, so we have now created our postcard mode. So I'm going to go ahead and generate the postcard to see what that looks like.

And just a heads up, this will be using the GPT 1.5 image generator. Yes. And it takes a minute. Well, usually comes through by now, so maybe it could be in and out. There it goes.

Yay. Okay, postcard created for current fact. Let's scroll down, take a look. A fun little postcard. Even put the little map snapshot in the top corner as well.

So fun little exploration app that you could use to learn more about a specific location. Let's get into a little bit of the code and then what powered this entire application? Okay, so a bit of the code first is the schema shaped output, like I was pointing out. Right. We're not asking the model to just say something interesting and then hoping it comes back usable.

It's very specifically structured to handle in our app. Right. So we've got the headline, the main fact, and the reward label. All right. The second piece is that the response is very strict, so it does matter.

Once you start building even some small gamified loops, you do need multiple pieces of content going to different parts of the interface. So one field might be the title, one might be the main card text, one might be the reward, etc. And then the third piece is that these fields map directly to what the UI actually does render. So that's the main reason this demo really feels usable. And it's not just the model is good at writing or whatever.

So again, the response shape really matches what the interface actually needs. All right. Okay. And what powered this demo? Well, there are a few things.

The first thing is we had a lot of location stuff going on there. So the first was ArcGIS was actually grounding the locations. We had our search box with auto suggest. We had a place resolution. So it actually basically grounded the location to make sure that it was tied to the actual place instead of something just kind of vague.

And then also we have reverse geocoding for when you click Use my location. The second thing was OpenAI responses API. And in that case, this gave us that structured output. And for the third thing, we have the fun little map that we have there. So that actually is a static basemap tile and it's centered directly on the specific latitude and longitude location.

And then for reward, we had several different things. We had xp, we had badges, we had levels, we had the fun postcard. So yeah, all sorts of things that we incorporated there to make the experience more interesting, more specific to the user's choice and memorable. All right, so lots of different pieces in a tiny little app. So that was our first pattern, dynamic content.

Now the second pattern is Sentiment Aware Feedback or Sentiment Aware Analysis. This is going to be where our app starts to feel a bit emotionally aware, a bit more in tune with what the user is feeling. So this is not about creating brand new content in this case. Right. It's really, it's not about changing the challenge itself.

It's more about just kind of checking the tone, the vibe, reinforcing that good stuff and then choosing a smart follow up to the tone. It kind of receives all right. And so yeah, all of this combined together, it's kind of a post round checkup here. So sentiment analysis could be used in many different experiences, but basically the main idea is the player gives some kind of feedback and then instead of just generating a generic message each time, that response is going to actually be tailored to how the user's feeling. Okay, so here's what our second mini demo is going to look like.

Basically, a player has just finished up a round of something. Maybe it's a trivia round or maybe it's like a learning checkpoint, whatever it may be. And now we ask one simple question. How did that feel? How are you feeling?

So the answer is going to be our signal. It's not a giant profile or anything, it's just just one little quick check in and then from there the app is going to make a decision. So it looks at the tone of that reflection and it's going to decide what kind of feedback route should be taken. And it uses confidence scores for this. Just a heads up, we are using Azure sentiment detection for this and then the response changes and the tone changes.

So some kind of support potentially will come based on how the user responds. All right, let's go ahead and check it out. Okay. All right, so, all right, it says, how did that feel? Well, I've got some pre built in options here, so I think I will go ahead and just pick one of those.

To start, I'm going to go ahead and just pick the positive example. It says, I really like this. It was smooth, helpful and way better than I expected. Let me go ahead and click analyze reflection and let's see what it says. Okay.

Right away it came back 99% positive. So very, very positive. The response tone that we've given here is celebratory and it says this reads clearly as positive. A good follow up here would reinforce probably progress, keep the tone upbeat and make the next step feel rewarding. So this app is more of just kind of showing the getting the point across of how you could utilize sentiment detection in your own applications.

Certainly not a user facing app, but it shows how the API actually works here. Yeah, so it also is able to, if you notice, it's able to break up the two different sentences there. So because it's able to do that, it is also able to actually handle, you know, mixed feelings as well. So let's try the mixed example. Since I was frustrated at first.

However, once I figured it out, I felt pretty good about it. All right, so let's analyze that. Okay, cool. And it says a mixed reflection. Right.

This reflection has both progress and friction. The best response would keep momentum visible while still making room for the rough patch. And then notice we got 46% positive, 51% negative, and 3% neutral. So it's able to handle this kind of nuance as well. Now, one other fun example I like to do when I show this demo is do something very simple, very kind of potentially have multiple meanings.

So if we use literally just the word mean, so very, very kind of nuanced in that, what does it mean? Right? Is it that meaning in that sentence of what does it mean? Or is it someone is being mean? Or is it the average of a set of numbers?

So it's multiple things. Let's go ahead and analyze and see what happens. Okay, so it definitely came back pretty mixed, right? It's 21% positive, 42% neutral, 37% negative. So if you give it a bit more context, of course it's going to be able to determine the situation much better.

So let's see. He was being mean. There we go. Okay. 88% negative.

So once you add a little bit more context, the Azure AI is able to actually analyze the feeling there. All right, let's go ahead and look at a bit of the code for this. Okay, so first we analyze the reflection. So we're telling Azure here that we want sentiment analysis, and we're sending in the player's reflection as the thing that we want analyzed. All right?

And then we actually choose a route. Right? So this is when where the app logic takes over. So the model gives you the signal, but the code actually decides what that means for the player. So if the reflection reads more negative, we can move into some kind of more supportive path.

And then, of course, we change the UI in a way that the player can actually feel the experience happening. So they understand that, hey, we were actually being listened to. Right? And that's obviously the whole point of this pattern is asking the user how they're feeling and giving them some kind of good response, good feedback for the user to have a great experience all around. So what powered this demo at the start, really, the app only has one thing to work with, and that's really just the player's reflection of how the round went.

Right. And so then what happens is Azure turns that into something much more useful. You get the overall sentiment. You can see sentence level sentiment, like I mentioned, and you do get these confidence scores back, which were those percentages. All right?

And then your app maps that into a checkpoint round and that's what changes the tone of the feedback, really, the support around the player and the recovery options that are available. So that was two. That was our second pattern. Okay. So our third pattern is going to be Adaptive challenge tuning.

So this one is one that you can feel immediately, even if you've never heard the term for it, really. So adaptive challenge tuning or dynamic difficulty is kind of another way you can put it. Basically, what we mean by this is the experience is not feeling random, right? It's not going to feel randomly, too hard or too boring. It's going to actually listen for how the user is working through a problem and determine, hey, maybe we should make this a little harder, a little bit easier.

So first it has to notice, obviously, the behavior. Then it has to adjust the experience experience, and then what the user feels kind of gives that better pacing from there. Okay, so for this demo, I want you to imagine a quiz app that is trying to keep you engaged. So it does not want to stay boring if you're cruising along, and it does not want to stay punishing if you are struggling. So it starts out by noticing a couple of simple things.

Did I get it right? And how long did it take? So these are the two key things for when you're answering questions in this application. So is it correct? And how long did you take to answer the question?

And then it will make a lightweight prediction based on that about what really should happen next. Right. So if you took, say, maybe a really long time to answer a question, but you got it right, maybe it'll keep you at the same difficulty. Maybe not, depending on how long that exact time is. And heads up, we are using TensorFlow for this, and that changes the obviously feel of the experience.

Right? Because the challenge can adjust, can change the difficulty level so you feel like you're having a better experience there. All right, let's check it out. Okay, There it is. All right, so first thing I want to do is point out at the very top, I'm going to refresh this page, but pay attention to this little part here.

Basically, when I refresh the page, TensorFlow is training itself on the fly, right? This is browser side analysis here. There it goes. This is training. And there it is.

Okay, so once it's ready, we can start our short little quiz session. So I'll go ahead and start the session, but before I click that real fast, it is going to start out on easy, just so you know. Cause that's. Yeah, that's just the starting point there. And that's what We've got.

Let's see a few other things to point out. We've got our. I went ahead and left this in here, actually, so you can see the signal decision and response flow there. But over here, this will tell us the prediction value of the confidence of how well confident the app is to move to a different difficulty level. So again, it'll be determined on correctness and response time.

So let's try. Okay, which ocean is the largest? Well, it's Pacific, so let's click it. Nice. Okay, it says you answered Pacific in four seconds.

The Pacific Ocean is the largest and it says 2% easy, 98% positive. So maybe if I had answered it just slightly faster, it would have potentially been 100% medium to the confidence level. There a bit more context within our signal decision and response up here. Just to really give you a clear view of everything, the we're 100% accurate so far. This actually, I did forget to completely clear out the previous tests of this.

So technically my best streak from testing out the application was four. And let's see, down here we get a compact session log of all of our results so we can see everything happening as we continue to answer these questions. Questions. Okay, so now I'll go to the next question and this time I'm going to intentionally miss it just so we can see how the app responds. So let's check it out.

Capital of Australia, I'm going to say. Oh, that was the right one. Okay. I don't know my capitals very well. Let's try it again.

Okay, I know this one definitely. What year did the first iPhone launch? No, it's 2007. I'm going to go ahead and answer 2008. And right away I took quite a bit of time because I was explaining that, but also I got it wrong.

So it said, okay, let's go back to easy. Now, all of this, you'll see a bit of the code. And obviously I've provided the code to you all, so you can see kind of how I trained the model itself. But all of this just took some tweaking of various different variables and values to train to get kind of the right feeling of what I thought worked. But in different situations, you would want to use potentially different variables and stuff along with the amount of training data that you have as well.

I use a very small amount of training data since this is a very simple application. But of course, the more you use, the more accurate it will be based on the feeling that you want the users to experience. All right, I Believe that is everything for that application. Yes. Okay, let's go back.

So let's look at a little bit of the code. Okay. The nice thing about this one is the code loop for this is really small. So first we collect a few simple features. Was the answer correct?

How long did it take? And what difficulty are we currently on right now? Right, that is our signal and then we pass it into TensorFlow, the model, and ask for one prediction. And we're not generating, you know, some kind of paragraph or anything, it's just the one focus question, what should the next difficulty be based on that response. And then of course, the UI updates, we update the difficulty level, move to the next question, and the user should feel that pacing kind of shift.

Right, of course, that's the most important part here. So the reusable pattern, it's not very big. It just captures the inputs small prediction and visibly change the experience. Like basically what we've been doing for all of these different kinds of applications. Yeah.

So TensorFlow, great for again, in browser experiences, when you are wanting to create some kind of small change in your application, potentially to help gamify a situation like this and where you don't want to have to touch a server to create that experience. Okay, and what powered this demo? So first, this is. I already have mentioned this several times. TensorFlow JS very nice and lightweight small local prediction loop here.

So nothing, nothing giant about this. All very small and local. And second, the inputs are very intentionally simple. Right. We're not feeding in some kind of giant like behavior profile or anything.

We're just using the correctness, response time and current difficulty. And then finally it makes that one decision. So it predicts the the next difficulty. Really, that is the whole point of this pattern. Again, small pacing is noticeable and the user really does not care that there's some kind of model happening in the background.

It's just the experience is all it is for the user itself. So again, under the hood, browser side model. Okay, so now this is my favorite app out of all of them. Because basically what we're doing is we're going to zoom out and we're going to put it all together. So every one of these features so far has been interesting enough by themselves, I think.

But what happens when you actually put them all together in one app? You get to see how the loops work together, how everything can complement each other all in one. Okay. All right, so this is our combined app version of the Talk. And basically what it is is it has a quiz mode with adaptive difficulty A sentiment mode with recap and a layer of badges, unlocks and visible stats so that the user can feel that progression over time.

So this one is actually going to be a fun little geo explored app. And let's take a look at that. Let's see here. Okay. All right, so similar idea as to earlier of where you are exploring a new place.

Let me go ahead and refresh fresh all of this. I'm going to click Start Fresh so it's all nice and fresh there. Perfect. Okay. All right, so let me point out a few things before I show this app in action.

So the first thing is it's obviously centered on the Copenhagen area here we are using an ArcGIS Maps SDK for JavaScript application here for the map itself and we have a lot of different stuff going on over here. So let me point out all these different things over to the side. That way you can understand what the power of this application. So the first thing is we have the experience mode and this lets you choose the full experience or just one part of it. So there are two different modes you could work in in this application.

You could work in a challenges mode or you could work in a mood check ins mode. So the challenges are going to be that dynamic content being generated by OpenAI or the Mood check ins. That's going to be your sentiment analysis. If you do the full experience it's going to be both. Right, so there's that.

The next thing is going to be your progress. So it's a simple view of how your session is going. So as we click through different places on the map, we're going to be asked different questions and the pacing itself is going to adjust based on how well we answer the questions. So we get to learn a little bit about an area and kind of challenge ourselves as well. So we see all of the attempts, accuracy, average time, current streak and mood tags and then we've got our challenge pace again that's just going to be the what the difficulty level prediction is.

For the next question again from TensorFlow, we have a recent challenge activity that is just complete a few place challenges to build your recent activity. So we'll have to do a few before we can see really much there Recent mood. So after we answer how we felt about a different location, we'll get to see that here. Whether it's positive, neutral or negative. We've got a bunch of badges here that are all currently locked that we will get to unlock as we do some fun exploring.

Another fun thing here is new looks for the Map unlock as you build momentum. So as we continue to click through through and answer questions, it says this one says unlock at 3, correct, this one says 6 correct, etc. Etc. So after we continue to get more questions right, we get to see different map experiences. And it really is mostly a matter of changing colors and labels and other things.

And finally, I thought this was kind of fun to add in here as well. So after you have rated a bunch of different places, you can generate a session recap, and that will tell you how you generally felt in your entire session experience after clicking through and rating places. All right, let's go ahead and try it out then. Okay, so I do have the ability to search for a place if I want to, or I can just click on the map. So I'm going to go ahead and just click click on the map and let's see this happen.

All right, so it says Emerco is in which city area, and it's one of these. Clearly it should be that first one there. Nice. Based on exactly where I clicked. All right, so we've got a simple question.

Right again, easy question. I answered it in 9.3 seconds. So based on that, even though I did get it correct, it still thinks that we should stay at an easy level for the next question. Now, what I can do is I can generate another question about that exact same clicked location, or I could just click somewhere else if I want to do a new location. And then the other thing I mentioned is that sentiment, my reaction so I could say I loved this place.

Okay. And one fun little thing that you'll see happen here is once I save my reaction, we get this positive little reaction here. And then we've got. It says upbeat. So it says we're in an upbeat mood.

It keeps track of all of our information over here in our recent mood. Okay, I'm going to do another one. I want to do just enough so we can see some of this stuff unlock. All right. Oh, one other thing I forgot to point out is that little dot there, it actually turned green after I had a positive sentiment.

So I'll go ahead and do a negative one for this next one. So you can see that happen. So we're at this blue dot right now. Clearly I'm not going to get this right. I'll just click something.

Okay. Oh, I should have known because it's literally up here. Anyways, easy question. But yeah, I got it wrong. It's all right.

We get to see the experience of getting it wrong. And I'm going to go ahead and say I hated this place. So we can see that change to red. So as you continue to explore the area, you can see all of your reactions kind of come together and see how that's happening through. Okay, now one thing I do want to do is I'm actually.

I have logged the answers in the console. That way I can get them right. So I'm going to go ahead and do a couple more so we can see the map unlock that next reward that I mentioned. All right, here we go. Okay, nice call.

And as we can see here, it is keeping track. It says unlock at three. Correct. And we've got two out of three, so. So let's do one more.

A again. All right. Boom. Okay, right away, if you didn't catch that, I will go ahead and change it back. But right away we actually did end up with a change in our map style.

So fun little reaction to kind of see happen. Here was the original map style and here is the new map style. So little fun reward to give the user in the entire overall experience here. Okay, so the only other thing that I do need to point out, I believe, is that final session recap. Right.

And again, that's going to be based off of my sentiment from all the different places that I have given my review to. Let me actually do one more real quick so that way we can see a bit more interesting of a recap here. So this place was okay, but I didn't like the people. I don't know. Let's see what it says there.

Okay. All right, so that one, it came out to be. It says negative. Let's look at our. Yeah, actually it thinks that I.

It's mostly negative, so. So a little bit of neutral there. This place was okay, but I think the. Didn't like the people part. It definitely took that as a negative thing.

All right, let's go ahead and generate our recap now to see what it says. All right, so here we go. It says you had a strong session, getting four out of five correct with a solid three win streak and quick average times. Your reflections suggest a clear split in reactions you liked. Emerico felt strongly negative about this other place.

We're lukewarm on another place. So overall it looked like a focused run with mostly good momentum and a few strong options along the way. And then it actually says next beat. So next you might keep the streak going with another round and see if you can, you know, push for a perfect run. So it gives you a little bit of nice feedback there as well, so overall experience, it told you about how you did with both modes, the sentiment and the challenge mode as well.

Okay, so again, you can see how all these pieces put together create a nice gamified user experience. Okay, so I've got just a few last things. Let's go ahead and recap our overall mental model here. So the first is you collect your signal, right? This could be a click, an answer, timing, a streak, really anything that the app has to observe the user doing.

And then second, AI gets one focus job. So you generate something, you can classify something, summarize something, maybe whatever it may be. We saw that in many different ways of using our AI libraries. And then third is make the UI respond, right? There's no point in having AI do something if you aren't going to show the user what happened.

Right? So that could be something like an unlock reward or whatever it may be. So that is the hopefully pattern that I want you to overall think about when you're trying to apply this to your own applications. Okay. And finally, so that's the build pattern.

But really for me, what makes this work is the following things, right? So first, the app has better context with all of this AI powered gamification, right? And knows something useful about what just happened. So maybe the user, you know, clicked on a place and then picked a fact type, like I was saying. Or maybe a they typed a reflection, that kind of thing.

So obviously it makes it feel less random. And then of course, AI allows for much better feedback loops. The app actually notices something and responds to it and is able to help with that interaction. And it's not just like a one trickle that every user is going to experience. Of course we have our visible UI like I've been emphasizing.

And fourth is, it's not magical, right? It's definitely, you do not want these features to feel like mysterious or anything. You want them to feel very intentional and having the user in mind for each one of these different experiences. All right, so again, I'll emphasize it one more time. Using AI gamification, it's not some miracle, it's just supposed to help an experience that's already existing without those elements to become much more worthwhile and relevant for the user.

Okay. All right, so I have the slides and code back up here. I would love to hear about. If you end up using any kind of these demo ideas in your future apps. You can connect with me on socials and yeah, thank you so much.

And it looks like we do have a little bit of time for questions if anyone would like to or I'm happy to talk with you up here as well. Cool. Sounds good.
