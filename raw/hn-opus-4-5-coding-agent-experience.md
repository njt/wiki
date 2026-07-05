---
url: https://news.ycombinator.com/item?id=46515696
date_fetched: 2026-07-05
backfilled: true
---

Most software engineers are seriously sleeping on how good LLM agents are right now, especially something like Claude Code.

Once you’ve got Claude Code set up, you can point it at your codebase, have it learn your conventions, pull in best practices, and refine everything until it’s basically operating like a super-powered teammate. The real unlock is building a solid set of reusable “skills” plus a few agents for the stuff you do all the time.

For example, we have a custom UI library, and Claude Code has a skill that explains exactly how to use it. Same for how we write Storybooks, how we structure APIs, and basically how we want everything done in our repo. So when it generates code, it already matches our patterns and standards out of the box.

We also had Claude Code create a bunch of ESLint automation, including custom ESLint rules and lint checks that catch and auto-handle a lot of stuff before it even hits review.

Then we take it further: we have a deep code review agent Claude Code runs after changes are made. And when a PR goes up, we have another Claude Code agent that does a full PR review, following a detailed markdown checklist we’ve written for it.

On top of that, we’ve got like five other Claude Code GitHub workflow agents that run on a schedule. One of them reads all commits from the last month and makes sure docs are still aligned. Another checks for gaps in end-to-end coverage. Stuff like that. A ton of maintenance and quality work is just… automated. It runs ridiculously smoothly.

We even use Claude Code for ticket triage. It reads the ticket, digs into the codebase, and leaves a comment with what it thinks should be done. So when an engineer picks it up, they’re basically starting halfway through already.

There is so much low-hanging fruit here that it honestly blows my mind people aren’t all over it. 2026 is going to be a wake-up call.

(used voice to text then had claude reword, I am lazy and not gonna hand write it all for yall sorry!)

I made a similar comment on a different thread, but I think it also fits here: I think the disconnect between engineers is due to their own context. If you work with frontend applications, specially React/React Native/HTML/Mobile, your experience with LLMs is completely different than the experience of someone working with OpenGL, io_uring, libev and other lower level stuff. Sure, Opus 4.5 can one shot Windows utilities and full stack apps, but can't implement a simple shadowing algorithm from a 2003 paper in C++, GLFW, GLAD: https://www.cse.chalmers.se/~uffe/soft_gfxhw2003.pdf

Codex/Claude Code are terrible with C++. It also can't do Rust really well, once you get to the meat of it. Not sure why that is, but they just spit out nonsense that creates more work than it helps me. It also can't one shot anything complete, even though I might feed him the entire paper that explains what the algorithm is supposed to do.

Try to do some OpenGL or Vulkan with it, without using WebGPU or three.js. Try it with real code, that all of us have to deal with every day. SDL, Vulkan RHI, NVRHI. Very frustrating.

Try it with boost, or cmake, or taskflow. It loses itself constantly, hallucinates which version it is working on and ignores you when you provide actual pointers to documentation on the repo.

I've also recently tried to get Opus 4.5 to move the Job system from Doom 3 BFG to the original codebase. Clean clone of dhewm3, pointed Opus to the BFG Job system codebase, and explained how it works. I have also fed it the Fabien Sanglard code review of the job system: https://fabiensanglard.net/doom3_bfg/threading.php

We are not sleeping on it, we are actually waiting for it to get actually useful. Sure, it can generate a full stack admin control panel in JS for my PostgreSQL tables, but is that really "not normal"? That's basic.

We have an in-house, Rust-based proxy server. Claude is unable to contribute to it meaningfully outside of grunt work like minor refactors across many files. It doesn't seem to understand proxying and how it works on both a protocol level and business logic level.

With some entirely novel work we're doing, it's actually a hindrance as it consistently tells us the approach isn't valid/won't work (it will) and then enters "absolutely right" loops when corrected.

I still believe those who rave about it are not writing anything I would consider "engineering". Or perhaps it's a skill issue and I'm using it wrong, but I haven't yet met someone I respect who tells me it's the future in the way those running AI-based companies tell me.

> We have an in-house, Rust-based proxy server. Claude is unable to contribute to it meaningfully outside

I have a great time using Claude Code in Rust projects, so I know it's not about the language exactly.

My working model is is that since LLM are basically inference/correlation based, the more you deviate from the mainstream corpus of training data, the more confused LLM gets. Because LLM doesn't "understand" anything. But if it was trained on a lot of things kind of like the problem, it can match the patterns just fine, and it can generalize over a lot layers, including programming languages.

Also I've noticed that it can get confused about stupid stuff. E.g. I had two different things named kind of the same in two parts of the codebase, and it would constantly stumble on conflating them. Changing the name in the codebase immediately improved it.

So yeah, we've got another potentially powerful tool that requires understanding how it works under the hood to be useful. Kind of like git.

Recently the v8 rust library changed it from mutable handle scopes to pinned scopes. A fairly simple change that I even put in my CLAUDE.md file. But it still generates methods with HandleScope's and then says... oh I have a different scope and goes on a random walk refactoring completely unrelated parts of the code. All the while Opus 4.5 burns through tokens. Things work great as long as you are testing on the training set. But that said, it is absolutely brilliant with React and Typescript.

Well, it's not like it never happened to me to "burn tokens" with some lifetime issue. :D But yeah, if you're working in Rust on something with sharp edges, LLM will get get hurt. I just don't tend to have these in my projects.

Even more basic failure mode. I told it to convert/copy a bit (1k LOC) of blocking code into a new module and convert to async. It just couldn't do a proper 1:1 logical _copy_. But when I manually `cp <src> <dst>` the file and then told it to convert that to async and fix issues, it did it 100% correct. Because fundamentally it's just non-deterministic pattern generator.

hot take (that shouldn't be?): if your code is super easy to follow as a human, it will be super easy to follow for an LLM. (hint: guess where the training data is coming from!)

This isn't meant as a criticism, or to doubt your experience, but I've talked to a few people who had experiences like this. But, I helped them get Claude code setup, analyze the codebase and document the architecture into markdown (edit as needed after), create an agent for the architecture, and prompt it in an incremental way. Maybe 15-30 minutes of prep. Everyone I helped with this responded with things like "This is amazing", "Wow!", etc.

For some things you can fire up Claude and have it generate great code from scratch. But for bigger code bases and more complex architecture, you need to break it down ahead of time so it can just read about the architecture rather than analyze it every time.

Is there any good documentation out there about how to perform this wizardry? I always assumed if you did /init in a new code base, that Claude would set itself up to maximize its own understanding of the code. If there are extra steps that need to be done, why don't Claude's developers just add those extra steps to /init?

Not that I have seen, which is probably a big part of the disconnect. Mostly it's tribal knowledge. I learned through experimentation, but I've seen tips here and there. Here's my workflow (roughly)

> Create a CLAUDE.md for a c++ application that uses libraries x/y/z

[Then I edit it, adding general information about the architecture]

> Analyze the library in the xxx directory, and produce a xxx_architecture.md describing the major components and design

> /agent [let claude make the agent, but when it asks what you want it to do, explain that you want it to specialize in subsystem xxx, and refer to xxx_architecture.md

Then repeat until you have the major components covered. Then:

> Using the files named with architecture.md analyze the entire system and update CLAUDE.md to use refer to them and use the specialized agents.

Now, when you need to do something, put it in planning mode and say something like:

> There's a bug in the xxx part of the application, where when I do yyy, it does zzz, but it should do aaa. Analyze the problem and come up with a plan to fix it, and automated tests you can perform if possible.

Then, iterate on the plan with it if you need to, or just approve it.

One of the most important things you can do when dealing with something complex is let it come up with a test case so it can fix or implement something and then iterate until it's done. I had an image processing problem and I gave it some sample data, then it iterated (looking at the output image) until it fixed it. It spent at least an hour, but I didn't have to touch it while it worked.

I've taken time today to do this. With some of your suggestions, I am seeing an improvement in it's ability to do some of the grunt work I mentioned. It just saved me an hour refactoring a large protocol implementation into a few files and extracted some common utilities. I can recognise and appreciate how useful that is for me and for most other devs.

At the same time, I think there's limitations to these tools and that I wont ever be able to achieve what I see others saying about 95% of code being AI written or leaving the AI to iterate for an hour. There's just too many weird little pitfalls in our work that the AI just cannot seem to avoid.

It's understandable, I've fallen victim to a few of them too, but I have the benefit of the ability to continuously learn/develop/extrapolate in a way that the LLM cannot. And with how little documentation exists for some of these things (MASQUE proxying for example) anytime the LLM encounters this code it throws a fit, and is unable to contribute meaningfully.

So thanks for your suggestions, it has made Claude better and clearly I was dragging my feet a little. At the very least, it's freed up a some more of my time to work on the complex things Claude can't do.

To be perfectly honest, I've never used a single /command besides /init. That probably means I'm using 1% of the software's capabilities. In frankness, the whole menu of /-commands is intimidating and I don't know where to start.

You don't need to do much, the /agent command is the most useful, and it walks you through it. The main thing though is to give the agent something to work with before you create it. That's why I go through the steps of letting Claude analyze different components and document the design/architecture.

The major benefit of agents is that it keeps context clean for the main job. So the agent might have a huge context working through some specific code, but the main process can do something to the effect of "Hey UI library agent, where do I need to put code to change the color of widget xyz", then the agent does all the thinking and can reply with "that's in file 123.js, line 200". The cleaner you keep the main context, the better it works.

/commands are like macros or mayyybe aliases. You just put in the commands you see yourself repeating often, like "commit the unstaged files in distinct commits, use xxx style for the commit messages..." - then you can iterate on it if you see any gaps or confusion, even give example commands to use in the different steps.

Skills on the other hand are commands ON STEROIDS. They can be packaged with actual scripts and executables, the PEP723 Python style + uv is super useful.

I have one skill for example that uses Python+Treesitter to check the unit thest quality of a Go project. It does some AST magic to check the code for repetition, stupid things like sleeps and relative timestamps etc. A /command _can_ do it, but it's not as efficient, the scripts for the skill are specifically designed for LLM use and output the result in a hyper-compact form a human could never be arsed to read.

> In frankness, the whole menu of /-commands is intimidating and I don't know where to start.

claude-code has a built in plugin that it can use to fetch its own docs! You don't have to ever touch anything yourself, it can add the features to itself, by itself.

This is some great advice. What I would add is to avoid the internal plan mode and just build your own. Built in one creates md files outside the project, gives the files random names and its hard to reference in the future.

It's also hard to steer the plan mode or have it remember some behavior that you want to enforce. It's much better to create a custom command with custom instructions that acts as the plan mode.

My system works like this:

/implement command acts as an orchestrator & plan mode, and it is instructed to launch predefined set of agents based on the problem and have them utilize specific skills. Every time /implement command is initiated, it has to create markdown file inside my own project, and then each subagent is also instructed to update the file when it finished working.

This way, orchestrator can spot that agent misbehaved, and reviewer agent can see what developer agent tried to do and why it was wrong.

> if you did /init in a new code base, that Claude would set itself up to maximize its own understanding of the code.

This is definitely not the case, and the reason anthropic doesnt make claude do this is because its quality degrades massively as you use up its context. So the solution is to let users manage the context themselves in order to minimize the amount that is "wasted" on prep work. Context windows have been increasing quite a bit so I suspect that by 2030 this will no longer be an issue for any but the largest codebases, but for now you need to be strategic.

Are you still talking about Opus 4.5 I’ve been working on a Rust, kotlin and c++ and it’s been doing well. Incredible at C++, like the number of mistakes it doesn’t make

> I still believe those who rave about it are not writing anything I would consider "engineering".

Correct. In fact, this is the entire reason for the disconnect, where it seems like half the people here think LLMs are the best thing ever and the other half are confused about where the value is in these slop generators.

The key difference is (despite everyone calling themselves an SWE nowadays) there's a difference between a "programmer" and an "engineer". Looking at OP, exactly zero of his screenshotted apps are what I would consider "engineering". Literally everything in there has been done over and over to the death. Engineering is.. novel, for lack of a better word.

I don't think it's that helpful to try to gatekeep the "engineering" term or try to separate it into "pure" and "impure" buckets, implying that one is lesser than the other. It should be enough to just say that AI assisted development is much better at non-novel tasks than it is at novel tasks. Which makes sense: LLMs are trained on existing work, and can't do anything novel because if it was trained on a task, that task is by definition not novel.

Respectfully, it's absolutely important to "gatekeep" a title that has an established definition and certain expectations attached to the title.

OP says, "BUT YOU DON’T KNOW HOW THE CODE WORKS.. No I don’t. I have a vague idea, but you are right - I do not know how the applications are actually assembled." This is not what I would call an engineer. Or a programmer. "Prompter", at best.

And yes, this is absolutely "lesser than", just like a middleman who subcontracts his work to Fiverr (and has no understanding of the actual work) is "lesser than" an actual developer.

That's not the point being made to you. The point is that most people in the "software engineering" space are applying known tools and techniques to problems that are not groundbreaking. Very few are doing theoretical computer science, algorithm design, or whatever you think it is that should be called "engineering."

So the TL;DR here is... If you're in the business of recreating wheels - then you're in luck! We've automated wheel recreation to an acceptable degree of those wheels being true.

Most physical engineers are just applying known techniques all the time too. Most products or bridges or whatever are not solving some heretofore-unsolved problem.

It's how you use the tool that matters. Some people get bitter and try to compare it to top engineers' work on novel things as a strawman so they can go "Hah! Look how it failed!" as they swing a hammer to demonstrate it cannot chop down a tree. Because the tool is so novel and it's use us a lot more abstract than that of an axe, it is taking awhile for some to see its potential, especially if they are remembering models from even six months ago.

Engineering is just problem solving, nobody judges structural engineers for designing structures with another Simpson Strong Tie/No.2 Pine 2x4 combo because that is just another easy (and therefore cheap) way to rapidly get to the desired state. If your client/company want to pay for art, that's great! Most just want the thing done fast and robustly.

I think it's also that the potential is far from being realized yet we're constantly bombarded by braindead marketers trying to convince us that it's the best thing ever already. This is tiring especially when the leadership (not held back by any technical knowledge) believes them.

I'm sure AI will get there, I also think it's not very good yet.

Coding agents as of Jan 2026 are great at what 95% of software engineers do. For remaining 5% that do really novel stuff -- the agents will get there in a few years.

I've had Opus 4.5 hand rolling CUDA kernels and writing a custom event loop on io_uring lately and both were done really well. Need to set up the right feedback loops so it can test its work thoroughly but then it flies.

Yeah I've handed it a naive scalar implementation and said "Make this use SIMD for Mac Silicon / NEON" and it just spits out a working implementation that's 3-6x faster and passes the tests, which are binary exact specifications.

It can do this at the level of a function, and that's -useful-, but like the parent reply to top-level comment, and despite investing the time, using skills & subagents, etc., I haven't gotten it to do well with C++ or Rust projects of sufficient complexity. I'm not going to say they won't some day, but, it's not today.

Anecdotally, we use Opus 4.5 constantly on Zed's code base, which is almost a million lines of Rust code and has over 150K active users, and we use it for basically every task you can think of - new features, bug fixes, refactors, prototypes, you name it. The code base is a complex native GUI with no Web tech anywhere in it.

I'm not talking about "write this function" but rather like implementing the whole feature by writing only English to the agent, over the course of numerous back-and-forth interactions and exhausting multiple 200K-token context windows.

For me personally, definitely at least 99% all of the Rust code I've committed at work since Opus 4.5 came out has been from an agent running that model. I'm reading lots of Rust code (that Opus generated) but I'm essentially no longer writing any of it. If dot-autocomplete (and LLM autocomplete) disappeared from IDE existence, I would not notice.

Woah that's a very interesting claim you made
I was shying away from writing Rust as I am not a Rust developer but hearing from your experience looks like claude has gotten very good at writing Rust

Honestly I think the more you can give Claude a type system and effective tests, the more effective it can be. Rust is quite high up on the test strictness front (though I think more could be done...), so it's a great candidate. I also like it's performance on Haskell and Go, both get you pretty great code out of the box.

Have you ever worried that by programming in this way, you are methodically giving Anthropic all the information it needs to copy your product? If there is any real value in what you are doing, what is to stop Anthropic or OpenAI or whomever from essentially one-shotting Zed? What happens when the model providers 10x their costs and also use the information you've so enthusiastically given them to clone your product and use the money that you paid them to squash you?

Isn’t it widely assumed Microsoft used private repos for LLM training?

And even with a narrower definition of stealing, Microsoft’s ability to share your code with US government agencies is a common and very legitimate worry in plenty of threat model scenarios.

I just uninstalled Zed today when I realized the reason I couldn't delete a file on Windows because it was open in Zed. So I wouldn't speak too highly of the LLM's ability to write code. I have never seen another editor on Windows make the mistake of opening files without enabling all 3 share modes.

Just based on timing, I am almost 100% sure whatever code is responsible was handwritten before anyone working on Windows was using LLMs...but anyway, thank you for the bug report - I'll pass it along!

Trying to one-shot large codebases is a exercise in futility. You need to let Claude figure out and document the architecture first, then setup agents for each major part of the project. Doing this keeps the context clean for the main agent, since it doesn't have to go read the code each time. So one agent can fill it's entire context understanding part of the code and then the main agent asks it how to do something and gets a shorter response.

It takes more work than one-shot, but not a lot, and it pays dividends.

Is there a guide for doing that successfully somewhere? I would love to play with this on a large codebase. I would also love to not reinvent the wheel on getting Claude working effectively on a large code base. I don’t even know where to start with, e.g., setting up agents for each part.

> Do you think it can replace you basically one-shotting features/bugs in Zed?

Nobody is one-shotting anything nontrivial in Zed's code base, with Opus 4.5 or any other model.

What about a future model? Literally nobody knows. Forecasts about AI capabilities have had horrendously low accuracy in both directions - e.g. most people underestimated what LLMs would be capable of today, and almost everyone who thought AI would at least be where it is today...instead overestimated and predicted we'd have AGI or even superintelligence by now. I see zero signs of that forecasting accuracy improving. In aggregate, we are atrocious at it.

The only safe bet is that hardware will be faster and cheaper (because the most reliable trend in the history of computing has been that hardware gets faster and cheaper), which will naturally affect the software running on it.

> And also - doesn’t that make Zed (and other editors) pointless?

It means there's now demand for supporting use cases that didn't exist until recently, which comes with the territory of building a product for technologists! :)

Definitely more than a faster keyboard (e.g. I also ask the model to track down the source of a bug, or questions about the state of the code base after others have changed it, bounce architectural ideas off the model, research, etc.) but also definitely not a replacement for thinking or programming expertise.

I don't know if you've tried Chatgpt-5.2 but I find codex much better for Rust mostly due to the underlying model. You have to do planning and provide context, but 80%+ of the time it's a oneshot for small-to-medium size features in an existing codebase that's fairly complex. I honestly have to say that it's a better programmer than I am, it's just not anywhere near as good a software developer for all of the higher and lower level concerns that are the other 50% of the job.

If you have any opensource examples of your codebase, prompt, and/or output, I would happily learn from it / give advice. I think we're all still figuring it out.

Also this SIMD translation wasn't just a single function - it was multiple functions across a whole region of the codebase dealing with video and frame capture, so pretty substantial.

"I honestly have to say that it's a better programmer than I am, it's just not anywhere near as good a software developer for all of the higher and lower level concerns that are the other 50% of the job."

Is that a context issue? I wonder if LSP would help there. Though Claude Code should grep the codebase for all necessary context and LSP should in theory only save time, I think there would be a real improvement to outcomes as well.

The bigger a project gets the more context you generally need to understand any particular part. And by default Claude Code doesn't inject context, you need to use 3rd party integrations for that.

I'm a quite senior frontend using React and even I see Sonnet 4.5 struggle with basic things. Today it wrote my Zod validation incorrectly, mixing up versions, then just decided it wasn't working and attempted to replace the entire thing with a different library.

I have been chastened in the opposite direction by others. I've also subjectively really disliked Opus's speed and I've seen Opus do really silly things too. But I'll try out using it as a daily driver and see if I like it more.

Why do we all of a sudden hold these agents to some unrealistic high bar? Engineers write bugs all the time and write incorrect validations. But we iterate. We read the stacktrace in Sentry and realise what the hell I was thinking when I wrote that, and we fix things. If you're going to benefit from these agents, you'd need to be a bit more patient and point them correctly to your codebase.

My rule of thumb is that if you can clearly describe exactly what you want to another engineer, then you can instruct the agent to do it too.

I built an open to "game engine" entirely in Lua a many years ago, but relying on many third party libraries that I would bind to with FFI.

I thought I'd revive it, but this time with Vulkan and no third-party dependencies (except for Vulkan)

4.5 Sonet, Opus and Gemini 3.5 flash has helped me write image decoders for dds, png jpg, exr, a wayland window implementation, macOS window implementation, etc.

I find that Gemini 3.5 flash is really good at understanding 3d in general while sonnet might be lacking a little.

All these sota models seem to understand my bespoke Lua framework and the right level of abstraction. For example at the low level you have the generated Vulkan bindings, then after that you have objects around Vulkan types, then finally a high level pipeline builder and whatnot which does not mention Vulkan anywhere.

However with a larger C# codebase at work, they really struggle. My theory is that there are too many files and abstractions so that they cannot understand where to begin looking.

I'll second this. I'm making a fairly basic iOS/Swift app with an accompanying React-based site. I was able to vibe-code the React site (it isn't pretty, but it works and the code is fairly decent). But I've struggled to get the Swift code to be reliable.

Which makes sense. I'm sure there's lots of training data for React/HTML/CSS/etc. but much less with Swift, especially the newer versions.

I had surprising success vibe coding a swift iOS app a while back. Just for fun, since I have a bluetooth OBD2 dongle and an electric truck, I told Claude to make me an app that could connect to the truck using the dongle, read me the VIN, odometer, and state of charge. This was middle of 2025, so before Opus 4.5. It took Claude a few attempts and some feedback on what was failing, but it did eventually make a working app after a couple hours.

Now, was the code quality any good? Beats me, I am not a swift developer. I did it partly as an experiment to see what Claude was currently capable of and partly because I wanted to test the feasibility of setting up a simple passive data logger for my truck.

I'm tempted to take another swing with Opus 4.5 for the science.

I have not had this experience at all. It often doesn't get it right on the first pass, yes, but the advantage with Rust vibecoding is that if you give it a rule to "Always run cargo check before you think you're finished" then it will go back and fix whatever it missed on the first pass. What I find particularly valuable is that the compiler forces it to handle all cases like match arms or errors. I find that it often misses edge cases when writing typescript, and I believe that the relative leniency of the typescript compiler is why.

In a similar vein, it is quite good at writing macros (or at least, quite good given how difficult this otherwise is). You often have to cajole it into not hardcoding features into the macro, but since macros resolve at compile time they're quite well-suited for an LLM workflow as most potential bugs will be apparent before the user needs to test. I also think that the biggest hurdle of writing macros to humans is the cryptic compiler errors, but I can imagine that since LLMs have a lot of information about compilers and syntax parsing in their training corpus, they have an easier time with this than the median programmer. I'm sure an actual compiler engineer would be far better than the LLM, but I am not that guy (nor can I afford one) so I'm quite happy to use LLMs here.

For context, I am purely a webdev. I can't speak for how well LLMs fare at anything other than writing SQL, hooking up to REST APIs, React frontend, and macros. With the exception of macros, these are all problems that have been solved a million times thus are more boilerplate than novelty, so I think it is entirely plausible that they're very poor for different domains of programming despite my experiences with them.

i've also been using opus 4.5 with lots of heavy rust development. i don't "vibe code", but lead it with a relatively firm hand- and it produces pretty good results in surprisingly complicated tasks.

for example, one of our public repos works with rust compiler artifacts and cache restoration (https://github.com/attunehq/hurry); if you look at the history you can see it do some pretty surprisingly complex (and well made, for an LLM) changes. its code isn't necessarily what i would always write, or the best way to solve the problem, but it's usually perfectly serviceable if you give it enough context and guidance.

I've had pretty good luck with LLM agents coding C. In this case a C compiler that supports a subset of C and targets a customizable microcoded state machine/processor. Then I had Gemini code up a simulator/debugger for the target machine in C++ and it did it in short order and quite successfully - lets you single step through the microcode and examine inputs (and set inputs), outputs & current state - did that in an afternoon and the resulting C++ code looks pretty decent.

That's remarkably similar to something I've just started on - I want to create a self-compiling C compiler targeting (and to run on) an 8-bit micro via a custom VM. This a basically a retro-computing hobby project.

I've worked with Gemini Fast on the web to help design the VM ISA, then next steps will be to have some AI (maybe Gemini CLI - currently free) write an assembler, disassembler and interpreter for the ISA, and then the recursive descent compiler (written in C) too.

I already had Gemini 3.0 Fast write me a precedence climbing expression parser as a more efficient drop-in replacement for a recursive descent one, although I had it do that in C++ as a proof-of-concept since I don't know yet what C libraries I want to build and use (arena allocator, etc). This involved a lot of copy-paste between Gemini output and an online C++ dev environment (OnlineGDB), but that was not too bad, although Gemini CLI would have avoided that. Too bad that Gemini web only has "code interpreter" support for Python, not C and/or C++.

Using Gemini to help define the ISA was an interesting process. It had useful input in a "pair-design" process, working on various parts of the ISA, but then failed to bring all the ideas together into a single ISA document, repeatedly missing parts of what had been previously discussed until I gave up and did that manually. The default persona of Gemini seems not very well suited to this type of work flow where you want to direct what to do next, since it seems they've RL'd the heck out of it to want to suggest next step and ask questions rather than do what is asked and wait for further instruction. I eventually had to keep asking it to "please answer then stop", and interestingly quality of the "conversation" seemed to fall apart after that (perhaps because Gemini was now predicting/generating a more adversarial conversation than a collaborative one?).

I'm wondering/hoping that Gemini CLI might be better at working on documentation than Gemini web, since then the doc can be an actual file it is editing, and it can use it's edit tool for that, as opposed to hoping that Gemini web can assemble chunks of context (various parts of the ISA discussion) into a single document.

Just as a self follow-up here (I hate to do it!), after chatting to Gemini some more more about document creation alternatives, it does seem that Gemini CLI is by far the best way to go, since it's working in similar fashion to Claude Code and making targeted edits (string replacements) to files, rather than regenerating from scratch (unless it has misinterpreted something you said as a request to do that, which would be obvious when it showed you the suggested diff).

Another alternative (not recommended due to potential for "drift") is to use Gemini's Canvas capability where it is working on a document rather than a specification being spread out over Chat, but this document is fully regenerated for every update (unlike Claude's artifacts), so there is potential for it to summarize or drop sections of the document ("drift") rather than just making requested changes. Canvas also doesn't have Artifact's versioning to allow you to go back to undo unwanted drifts/changes.

Yeah, the online Gemini app is not good for long lived conversations that build up a body of decisions. The context window gets too large and things drop.

What I’ve learned is that once you reach that point you’ve got to break that problem down into smaller pieces that the AI can work productively with.

If you’re about to start with Gemini-cli I recommend you look up https://github.com/github/spec-kit. It’s a project out of Microsoft/Github that encodes a rigorous spec-then-implement multi pass workflow. It gets the AI to produce specs, double check the specs for holes and ambiguity, plan out implementation, translate that into small tasks, then check them off as it goes. I don’t use spec-kit all the time, but it taught me that what explicit multi pass prompting can do when the context is held in files on disk, often markdown that I can go in and change as needed. I think it ask basically comes down to enforcing enough structure in the form of codified processes, self checks and/or tests for your code.

Pro tip, tell spec-kit to do TDD in your constitution and the tests will keep it on the rails as you progress. I suspect “vibe coding” can get a bad rap due to lack of testing. With AI coding I think test coverage gets more important.

Have you experimented with all of these things on the latest models (e.g. Opus 4.5) since Nov 2025? They are significantly better at coding than earlier models.

I've found it to be pretty hit-or-miss with C++ in general, but it's really, REALLY bad at 3D graphics code. I've tried to use it to port an OpenGL project to SDL3_GPU, and it really struggled. It would confidently insist that the code it wrote worked, when all you had to do was run it and look at the output to see a blank screen.

I hope I’m not committing a faux pas by saying this—and please feel free to tell me that I’m wrong—but I imagine a human who has been blind since birth would also struggle to build 3D graphics code.

The Claude models are technically multi-modal, but IME the vision side of the equation is really lacking. As a result, Claude is quite good at reasoning about logic, and it can build e.g. simpler web pages where the underlying html structure is enough to work with, but it’s much worse at tasks that inherently require seeing.

Yea, for obvious reasons, it seems to be best at code that transforms data: text/binary input to text/binary output. And where the logic can be tracked and verified at runtime with sufficient (text) logging. In other words, it's much better close loop than open loop. I tried to help it by prompting it to please take a screen capture of its output to verify functionality, but it seems LLMs aren't quite ready for that yet.

> It also can't do Rust really well, once you get to the meat of it. Not sure why that is

Because types are proofs and require global correctness, you can't just iterate, fix things locally, and wait until it breaks somewhere else that you also have to fix locally.

I have not tried C++, but Codex did a good job with low-level C code, shaders as well as porting 32 bit to 64 bit assembly drawing routines.
I have also tried it with retro-computing programming with relative success.

From what I've seen, CC has troubles with the latest Swift too, partially because of it being latest and partially because it's so convoluted nowadays.

I really think a lof of people tried AI coding earlier, got frustrated at the errors and gave up. That's where the rejection of all these doomer predictions comes from.

And I get it. Coding with Claude Code really was prompting something, getting errors, and asking it to fix it. Which was still useful but I could see why a skilled coder adding a feature to a complex codebase would just give up

Opus 4.5 really is at a new tier however. It just...works. The errors are far fewer and often very minor - "careless" errors, not fundamental issues (like forgetting to add "use client" to a nextjs client component.

This was me. I was a huge AI coding detractor on here for a while (you can check my comment history). But, in order to stay informed and not just be that grouchy curmudgeon all the time, I kept up with the models and regularly tried them out. Opus 4.5 is so much better than anything I've tried before, I'm ready to change my mind about AI assistance.

I even gave -True Vibe Coding- a whirl. Yesterday, from a blank directory and text file list of requirements, I had Opus 4.5 build an Android TV video player that could read a directory over NFS, show a grid view of movie poster thumbnails, and play the selected video file on the TV. The result wasn't exactly full-featured Kodi, but it works in the emulator and actual device, it has no memory leaks, crashes, ANRs, no performance problems, no network latency bugs or anything. It was pretty astounding.

Oh, and I did this all without ever opening a single source file or even looking at the proposed code changes while Opus was doing its thing. I don't even know Kotlin and still don't know it.

I have a few Go projects now and I speak Go as well as you speak Kotlin. I predict that we'll see some languages really pull ahead of others in the next few years based on their advantages for AI-powered development.

For instance, I always respected types, but I'm too lazy to go spend hours working on types when I can just do ruby-style duck typing and get a long ways before the inevitable problems rear their head. Now, I can use a strongly typed language and get the advantages for "free".

> I predict that we'll see some languages really pull ahead of others in the next few years based on their advantages for AI-powered development.

Oh absolutely. I've been using Python for past 15 or so years for everything.

I've never written a single line of Rust in my life, and all my new projects are Rust now, even the quick-script-throwaway things, because it's so much better at instantly screaming at claude when it goes off track. It may take it longer to finish what I asked it to do, but requires so much less involvement from me.

I will likely never start another new project in python ever.

EDIT: Forgot to add that paired with a good linter, this is even more impressive. I told Claude to come up with the most masochistic clippy configuration possible, where even a tiny mistake is instantly punished and exceptions have to be truly exceptional (I have another agent that verifies this each run).

I just wish there was cargo-clippy for enforcing architectural patterns.

and with types, it makes it easier for rounds of agents to pick up mistakes at compile time, statically. linting and sanity checking untyped languages only goes so far.
I've not seen LLM's one shot perl style regexes. and javascript can still have ugly runtime WTFs

Going to one-up you though -- here's a literal one-liner that gets me a polished media center with beautiful interface and powerful skinning engine. It supports Android, BSD, Linux, macOS, iOS, tvOS and Windows.

Hah! I actually initiated the project because I'm a long time XBMC/Kodi user. I started using it when it was called XBMC, on an actual Xbox 1. I am sick and tired of its crashing, poor playback performance, and increasingly bloated feature set. It's embarrassing when I have friends or family over for movie night, and I have to explain "Sorry folks, Kodi froze midway through the movie again" while I frantically try to re-launch/reboot my way back to watching the movie. VLC's playback engine is much better but the VLC app's TV UX is ass. This application actually uses the libVLC playback engine under the hood.

I think anecdotes like this may prove very relevant the next few years. AI might make bad code, but a project of bad code that's still way smaller than a bloated alternative, and has a UX tailored to your exact requirements could be compelling.

A big part of the problem with existing software is that humans seem to be pretty much incapable of deciding a project is done and stop adding to it. We treat creating code like a job or hobby instead of a tool. Nothing wrong with that, unless you're advertising it as a tool.

Yea, after this little experiment, I feel like I can just go through every big, bloated, slow, tech-debt-ridden software I use and replace it with a tiny, bespoke version that does only what I need and no more.

The old adage about how "users use 10% of your software's features, but they each use a different 10%" can now be solved by each user just building that 10% for themselves.

How do you know “it has no memory leaks, crashes, ANRs, no performance problems, no network latency bugs or anything” if you built it just yesterday? Isn’t it a bit too early for claims like this? I get it’s easy to bring ideas to life but aren’t we overly optimistic?

Part of the "one day" development time was exhaustively testing it. Since the tool's scope is so small, getting good test coverage was pretty easy. Of course, I'm not guaranteeing through formal verification methods that the code is bug free. I did find bugs, but they were all areas that were poorly specified by me in the requirements.

I decided to vibe code something myself last week at work. I've been wanting to create a poc that involves a coding agent create custom bokeh plots that a user can interact with and ask follow up questions. All this had to be served using a holoview panel library

At work I only have access to calude using the GitHub copilot integration so this could be the cause of my problems. Claude was able to get slthe first iteration up pretty quick. At that stage the app could create a plot and you could interact with it and ask follow up questions.

Then I asked it to extend the app so that it could generate multiple plots and the user could interact with all of them one at a time. It made a bunch of changes but the feature was never implemented. I asked it to do again but got the same outcome. I completely accept the fact that it could just be all because I am using vscode copilot or my promoting skills are not good but the LLM got 70% of the way there and then completely failed

> At work I only have access to calude using the GitHub copilot integration so this could be the cause of my problems.

You really need to at least try Claude Code directly instead of using CoPilot. My work gives us access to CoPilot, Claude Code, and Codex. CoPilot isn’t close to the other more agentic products.

Do they manage context differently or have different system prompts? I would assume a lot of that would be the same between them. I think GH Copilots biggest shortcoming is that they are too token cheap. Aggressively managing context to the detriment of the results. Watching Claude read a 500 line file in 100 line chunks just makes me sad.

I recently replaced my monitor with one that could be vertically oriented, because I'm just using Claude Code in the terminal and not looking at file trees at all

but I do want a better way to glance and keep up with what its doing in longer conversations, for my own mental context window

Ah, but you’re at the beginning stage young grasshopper. Soon you will be missing that horizontal ultra wide monitor as you spin up 8 different Claude agents in parallel seasons.

oh I noticed! I've begun doing that on my laptop. I just started going down all my list of sideprojects one by one, then two by two, a Claude Code instance in a terminal window for each folder. It's a bit mental

I'm finding that branding and graphic design is the most arduous part, that I'm hoping to accelerate soon. I'm heavily AI assisted there too and I'm evaluating MCP servers to help, but so far I do actually have to focus on just that part as opposed to babysit

Thanks for posting this. It's a nice reminder that despite all the noise from hype-mongers and skeptics in the past few years, most of us here are just trying to figure this all out with an open mind and are ready to change our opinions when the facts change. And a lot of people in the industry that I respect on HN or elsewhere have changed their minds about this stuff in the last year, having previously been quite justifiably skeptical. We're not in 2023 anymore.

If you were someone saying at the start of 2025 "this is a flash in the pan and a bunch of hype, it's not going to fundamentally change how we write code", that was still a reasonable belief to hold back then. At the start of 2026 that position is basically untenable: it's just burying your head in the sand and wishing for AI to go away. If you're someone who still holds it you really really need to download Claude Code and set it to Opus and start trying it with an open mind: I don't know what else to tell you. So now the question has shifted from whether this is going to transform our profession (it is), to how exactly it's going to play out. I personally don't think we will be replacing human engineers anytime soon ("coders", maybe!), but I'm prepared to change my mind on that too if the facts change. We'll see.

I was a fellow mind-changer, although it was back around the first half of last year when Claude Code was good enough to do things for me in a mature codebase under supervision. It clearly still had a long way to go but it was at that tipping point from "not really useful" to "useful". But Opus 4.5 is something different - I don't feel I have to keep pulling it back on track in quite the way I used to with Sonnet 3.7, 4, even Sonnet 4.5.

For the record, I still think we're in a bubble. AI companies are overvalued. But that's a separate question from whether this is going to change the software development profession.

The AI bubble is kind of like the dot-com bubble in that it's a revolutionary technology that will certainly be a huge part of the future, but it's still overhyped (i.e. people are investing without regard for logic).

We were enjoying cheap second hand rack mount servers, RAM, hard drives, printers, office chairs and so on for a decade after the original dot com crash. Every company that went out of business liquidated their good shit for pennies.

I'm hoping after AI comes back down to earth there will be a new glut of cheap second hand GPUs and RAM to get snapped up.

Right. And same for railways, which had a huge bubble early on. Over-hyped on the short time horizon. Long term, they were transformative in the end, although most of the early companies and early investors didn’t reap the eventual profits.

At the time it was overhyped because just by adding .com to your company's name you could increase your valuation regardless of whether or not you had anything to do with the internet. Is that not stupid?

I think my comparison is apt; being a bubble and a truly society-altering technology are not mutually exclusive, and by virtue of it being a bubble, it is overhyped.

There was definitely a lot of stupid stuff happening. IMO the clearest accurate way to put it is that it was overhyped for the short term (hence the crazy high valuations for obvious bullshit), and underhyped for the long term (in the sense that we didn't really foresee how broadly and deeply it would change the world). Of course, there's more nuance to it, because some people had wild long-term predictions too. But I think the overall, mainstream vibe was to underappreciate how big a deal it was.

> Oh, and I did this all without ever opening a single source file or even looking at the proposed code changes while Opus was doing its thing. I don't even know Kotlin and still don't know it.

This is what people are still doing wrong. Tools in a loop people, tools in a loop.

The agent has to have the tools to detect whatever it just created is producing errors during linting/testing/running. When it can do that, I can loop again, fix the error and again - use the tools to see whether it worked.

I _still_ encounter people who think "AI programming" is pasting stuff into ChatGPT on the browser and they complain it hallucinates functions and produces invalid code.

Last weekend I was debugging some blocking issue on a microcontroller with embassy-rs, where the whole microcontroller would lock up as soon as I started trying to connect to an MQTT server.

I was having Opus investigate it and I kept building and deploying the firmware for testing.. then I just figured I'd explain how it could do the same and pull the logs.

Off it went, for the next ~15 minutes it would flash the firmware multiple times until it figured out the issue and fixed it.

There was something so interesting about seeing a microcontroller on the desk being flashed by Claude Code, with LEDs blinking indicating failure states. There's something about it not being just code on your laptop that felt so interesting to me.

But I agree, absolutely, red/green test or have a way of validating (linting, testing, whatever it is) and explain the end-to-end loop, then the agent is able to work much faster without being blocked by you multiple times along the way.

This is kind of why I'm not really scared of losing my job.

While Claude is amazing at writing code, it still requires human operators. And even experienced human operators are bad at operating this machinery.

Tell your average joe - the one who thinks they can create software without engineers - what "tools-in-a-loop" means, and they'll make the same face they made when you tried explaining iterators to them, before LLMs.

Explain to them how typing system, E2E or integration test helps the agent, and suddenly, they now have to learn all the things they would be required to learn to be able to write on their own.

I have been out of the loop for a couple of months (vacation). I tried Claude Opus 4.5 at the end of November 2025 with the corporate Github Copilot subscription in Agent mode and it was awful: basically ignoring code and hallucinating.

My team is using it with Claude Code and say it works brilliantly, so I'll be giving it another go.

How much of the value comes from Opus 4.5, how much comes from Claude Code, and how much comes from the combination?

I strongly concur with your second statement. Anything other than agent mode in GH copilot feels useless to me. If I want to engage Opus through GH copilot for planning work, I still use agent mode and just indicate the desired output is whatever.md. I obviously only do this in environments lacking a better tool (Claude Code).

I suspect that's the other thing at play here; many people have only tried Copilot because it's cheap with all the other Microsoft subscriptions many companies have. Copilot frankly is garbage compared to Cursor/Claude, even with the same exact models.

my issue hasn't been for a long time now that the code they write works or doesn't work. My issues all stem from that it works, but does the wrong thing

> My issues all stem from that it works, but does the wrong thing

It's an opportunity, not a problem. Because it means there's a gap in your specifications and then your tests.

I use Aider not Claude but I run it with Anthropic models. And what I found is that comprehensively writing up the documentation for a feature spec style before starting eliminates a huge amount of what you're referring to. It serves a triple purpose (a) you get the documentation, (b) you guide the AI and (c) it's surprising how often this helps to refine the feature itself. Sometimes I invoke the AI to help me write the spec as well, asking it to prompt for areas where clarification is needed etc.

This is how Beads works, especially with Claude Code. What I do is I tell Claude to always create a Bead when I tell it to add something, or about something that needs to be added, then I start brainstorming, and even ask it to do market research what are top apps doing for x, y or z. Then ask it to update the bead (I call them tasks) and then finally when its got enough detail, I tell it, do all of these in parallel.

There are several rubs with that operating protocol extending beyond the "you're holding it wrong" claim.

1) There exists a threshold, only identifiable in retrospect, past which it would have been faster to locate or write the code yourself than to navigate the LLM's correction loop or otherwise ensure one-shot success.

2) The intuition and motivations of LLMs derive from a latent space that the LLM cannot actually access. I cannot get a reliable answer on why the LLM chose the approaches it did; it can only retroactively confabulate. Unlike human developers who can recall off-hand, or at least review associated tickets and meeting notes to jog their memory. The LLM prompter always documenting sufficiently to bridge this LLM provenance gap hits rub #1.

3) Gradually building prompt dependency where one's ability to take over from the LLM declines and one can no longer answer questions or develop at the same velocity themselves.

4) My development costs increasingly being determined by the AI labs and hardware vendors they partner with. Particularly when the former will need to increase prices dramatically over the coming years to break even with even 2025 economics.

Since we asked you to stop hounding another user in this manner and you've continued to do it repeatedly, I've banned the account. This is not what Hacker News is for, and you've done it almost 50 times (!), almost 30 of which have been after we first asked you to stop. That is extreme, and totally unacceptable.

Many people - simonw is the most visible of them, but there are countless others - have given up trying to convinced folks who are determined to not be convinced, and are simply enjoying their increased productivity. This is not a competition or an argument.

Maybe they are struggling to convince others because they are unable to produce evidence that is able to convince people?

My experience scrolling X and HN is a bunch of people going "omg opus omg Claude Code I'm 10x more productive" and that's it. Just hand wavy anecdotes based on their own perceived productivity. I'm open to being convinced but just saying stuff is not convincing. It's the opposite, it feels like people have been put under a spell.

I'm following The Primeagen, he's doing a series where he is trying these tools on stream and following peoples advice on how to use them the best. He's actually quite a good programmer so I'm eager to see how it goes. So far he isn't impressed and thus neither am I. If he cracks it and unlocks significant productivity then I will be convinced.

>> Maybe they are struggling to convince others because they are unable to produce evidence that is able to convince people?

Simon has produced plenty of evidence over the past year. You can check their submission history and their blog: https://simonwillison.net/

The problem with people asking for evidence is that there's no level of evidence that will convince them. They will say things like "that's great but this is not a novel problem so obviously the AI did well" or "the AI worked only because this is a greenfield project, it fails miserably in large codebases".

It's true that some people will just continually move the goalposts because they are invested in their beliefs. But that doesn't mean that the skepticism around certain claims aren't relevant.

Nobody serious is disputing that LLM's can generate working code. They dispute claims like "Agentic workflows will replace software developers in the short to medium term", or "Agentic workflows lead to 2-100x improvements in productivity across the board". This is what people are looking for in terms of evidence and there just isn't any.

Thus far, we do have evidence that AI (at least in OSS) produces a 19% decrease in productivity [0]. We also have evidence that it harms our cognitive abilities [1]. Anecdotally, I have found myself lazily reaching for LLM assistance when encountering a difficult problem instead of thinking deeply about the problem. Anecdotally I also struggle to be more productive using AI-centric agents workflows in areas of expertise.

We want evidence that "vibe engineering" is actually more productive across the entire lifespan of a software project. We want evidence that it produces better outcomes. Nobody has yet shown that. It's just people claiming that because they vibe coded some trivial project, all of software development can benefit from this approach. Recently a principle engineer at Google claimed that Claude Code wrote their team's entire year's worth of work in a single afternoon. They later walked that claim back, but most do not.

I'm more than happy to be convinced but it's becoming extremely tiring to hear the same claims being parroted without evidence and then you get called a luddite when you question it. It's also tiring when you push them on it and they blame it on the model you use, and then the agent, and then the way you handle context, and then the prompts, and then "skill issue". Meanwhile all they have to show is some slop that could be hand coded in a couple hours by someone familiar with the domain. I use AI, I was pretty bullish on it for the last two years, and the combination of it simply not living up to expectations + the constant barrage of what feels like a stealth marketing campaign parroting the same thing over and over (the new model is way better, unlike the other times we said that) + the amount of absolute slop code that seems to continue to increase + companies like Microsoft producing worse and worse software as they shoehorn AI into every single product (Office was renamed to Copilot 365). I've become very sensitive to it, much in the same way I was very sensitive to the claims being made by certain VC backed webdev companies regarding their product + framework in the last few years.

I'm not even going to bring up the economic, social, and environmental issues because I don't think they're relevant, but they do contribute to my annoyance with this stuff.

> Thus far, we do have evidence that AI (at least in OSS) produces a 19% decrease in productivity

I generally agree with you, but I'd be remiss if I didn't point out that it's plausible that the slow down observed in the METR study was at least partially due to the subjects lack of experience with LLMs. Someone with more experience performed the same experiment on themselves, and couldn't find a significant difference between using LLMs and not [0]. I think the more important point here is that programmers subjective assessment of how much LLMs help them is not reliable, and biased towards the LLMs.

I think we're on the same page re. that study. Actually your link made me think about the ongoing debate around IDE's vs stuff like Vim. Some people swear by IDE's and insist they drastically improve their productivity, others dismiss them or even claim they make them less productive. Sound familiar? I think it's possible these AI tools are simply another way to type code, and the differences averaged out end up being a wash.

IDEs vs vim makes a lot of sense. AI really does feel like using an IDE in a certain way

Using AI for me absolutely makes it feel like I'm more productive. When I look back on my work at the end of the day and look at what I got done, it would be ludicrous to say it was multiple times the amount as my output pre-AI

Despite all the people replying to me saying "you're holding it wrong" I know the fix to it doing the wrong thing. Specify in more detail what I want. The problem with that is twofold:

1. How much to specify? As little as possible is the ideal, if we want to maximize how much it can help us. A balance here is key. If I need to detail every minute thing I may as well write the code myself

2. If I get this step wrong, I still have to review everything, rethink it, go back and re-prompt, costing time

When I'm working on production code, I have to understand it all to confidently commit. It costs time for me to go over everything, sometimes multiple iterations. Sometimes the AI uses things I don't know about and I need to dig into it to understand it

AI is currently writing 90% of my code. Quality is fine. It's fun! It's magical when it nails something one-shot. I'm just not confident it's faster overall

I think this is an extremely honest perspective. It's actually kind of cool that it's gotten to the point it can write most code - albeit with a lot of handholding.

This is why you use this AI bubble (it IS a bubble) to use the VC-funded AI models for dirt cheap prices and CREATE tools for yourself.

Need a very specific linter? AI can do it. Need a complex Roslyn analyser? AI. Any kind of scripting or automation that you run on your own machine. AI.

None of that will go away or suddenly stop working when the bubble bursts.

Within just the last 6 months I've built so many little utilities to speed up my work (and personal life) it's completely bonkers. Most went from "hmm, might be cool to..." to a good-enough script/program in an evening while doing chores.

Even better, start getting the feel for local models. Current gen home hardware is getting good enough and the local models smart enough so you can, with the correct tooling, use them for suprisingly many things.

> Even better, start getting the feel for local models. Current gen home hardware is getting good enough and the local models smart enough so you can, with the correct tooling, use them for suprisingly many things.

Are there any local models that are at least somewhat comparable to the latest-and-greatest (e.g. Opus 4.5, Gemini 3), especially in terms of coding?

A risk I see with this approach is that when the bubble pops, you'll be left dependent on a bunch of tools which you don't know how to maintain or replace on your own, and won't have/be able to afford access to LLMs to do it for you.

The "tools" in this context are literally a few hundred lines of Python or Github CI build pipeline, we're not talking about 500kLOC massive applications.

I'm building tools, not complete factories :) The AI builds me a better hammer specifically for the nails I'm nailing 90% of the time. Even if the AI goes away, I still know how the custom hammer works.

I thought that initially, but I don't think the skills AI weakens in me are particularly valuable

Let's say AI becomes too expensive - I more or less only have to sharpen up being able to write the language. My active recall of the syntax, common methods and libraries. That's not hard or much of a setback

Maybe this would be a problem if you're purely vibe coding, but I haven't seen that work long term

Open source models hosted by independent providers (or even yourself, which if the bubble pops will be affordable if you manage to pick up hardware on fire sales) are already good enough to explain most code.

> 1) There exists a threshold, only identifiable in retrospect, past which it would have been faster to locate or write the code yourself than to navigate the LLM's correction loop or otherwise ensure one-shot success.

I can run multiple agents at once, across multiple code bases (or the same codebase but multiple different branches), doing the same or different things. You absolutely can't keep up with that. Maybe the one singular task you were working on, sure, but the fact that I can work on multiple different things without the same cognitive load will blow you out of the water.

> 2) The intuition and motivations of LLMs derive from a latent space that the LLM cannot actually access. I cannot get a reliable answer on why the LLM chose the approaches it did; it can only retroactively confabulate. Unlike human developers who can recall off-hand, or at least review associated tickets and meeting notes to jog their memory. The LLM prompter always documenting sufficiently to bridge this LLM provenance gap hits rub #1.

Tell the LLM to document in comments why it did things. Human developers often leave and then people with no knowledge of their codebase or their "whys" are even around to give details. Devs are notoriously terrible about documentation.

> 3) Gradually building prompt dependency where one's ability to take over from the LLM declines and one can no longer answer questions or develop at the same velocity themselves.

You can't develop at the same velocity, so drop that assumption now. There's all kinds of lower abstractions that you build on top of that you probably can't explain currently.

> 4) My development costs increasingly being determined by the AI labs and hardware vendors they partner with. Particularly when the former will need to increase prices dramatically over the coming years to break even with even 2025 economics.

You aren't keeping up with the actual economics. This shit is technically profitable, the unprofitable part is the ongoing battle between LLM providers to have the best model. They know software in the past has often been winner takes all so they're all trying to win.

> With the latest models if you're clear enough with your requirements you'll usually find it does the right thing on the first try

That's great that this is your experience, but it's not a lot of people's. There are projects where it's just not going to know what to do.

I'm working in a web framework that is a Frankenstein-ing of Laravel and October CMS. It's so easy for the agent to get confused because, even when I tell it this is a different framework, it sees things that look like Laravel or October CMS and suggests solutions that are only for those frameworks. So there's constant made up methods and getting stuck in loops.

The documentation is terrible, you just have to read the code. Which, despite what people say, Cursor is terrible at, because embeddings are not a real way to read a codebase.

I'm working mostly in a web framework that's used by me and almost nobody else (the weird little ASGI wrapper buried in Datasette) and I find the coding agents pick it up pretty fast.

One trick I use that might work for you as well:

Clone GitHub.com/simonw/datasette to /tmp
then look at /tmp/docs/datasette for
documentation and search the code
if you need to

Try that with your own custom framework and it might unblock things.

If your framework is missing documentation tell Claude Code to write itself some documentation based on what it learns from reading the code!

> I'm working mostly in a web framework that's used by me and almost nobody else (the weird little ASGI wrapper buried in Datasette) and I find the coding agents pick it up pretty fast

Potentially because there is no baggage with similar frameworks. I'm sure it would have an easier time with this if it was not spun off from other frameworks.

> If your framework is missing documentation tell Claude Code to write itself some documentation based on what it learns from reading the code!

If Claude cannot read the code well enough to begin with, and needs supplemental documentation, I certainly don't want it generating the docs from the code. That's just compounding hallucinations on top of each other.

I find Claude Code is so good at docs that I sometimes investigate a new library by checking out a GitHub repo, deleting the docs/ and README and having Claude write fresh docs from scratch.

In a circuitous way, you can rather successfully have one agent write a specification and another one execute the code changes. Claude code has a planning mode that lets you work with the model to create a robust specification that can then be executed, asking the sort of leading questions for which it already seems to know it could make an incorrect assumption. I say 'agent' but I'm really just talking about separate model contexts, nothing fancy.

Cursor's planning functionality is very similar and I have found that I can even use "cheap" models like their Composer-1 and get great results in the planning phase, and then turn on Sonnet or Opus to actually produce the plan. 90% of the stuff I need to argue about is during the planning phase, so I save a ton of tokens and rework just making a really good spec.

It turns out that Waterfall was always the correct method, it's just really slow ;)

And if you've told it too many times to fix it, tell it someone has a gun to your head, for some reason it almost always gets it right this very next time.

Yeah, if anyone can truly afford the AI empire. Remember all these "leading" companies are running it at a loss, so most companies paying for it are severely underpaying the cost of it all. We would need an insane technological breakthrough of unlimited memory and power before I start to worry, and at that point, I'll just look for a new career.

I think it's worth understanding why. Because that's not everyone's experience and there's a chance you could make a change such that you find it extremely useful.

There's a lesser chance that you're working on a code base that Claude Code just isn't capable of helping with.

The more explicit/detailed your plan, the more context it uses up, the less accurate and generally functional it is. Don't get me wrong, it's amazing, but on a complex problem with large enough context it will consistently shit the bed.

The human still has to manage complexity. A properly modularized and maintainable code base is much easier for the LLM to operate on — but the LLM has difficulty keeping the code base in that state without strong guidance.

Putting “Make minimal changes” in my standard prompt helped a lot with the tendency of basically all agents to make too many changes at once. With that addition it became possible to direct the LLM to make something similar to the logical progression of commits I would have made anyway, but now don’t have to work as hard at crafting.

Most of the hype merchants avoid the topic of maintainability because they’re playing to non-technical management skeptical of the importance of engineering fundamentals. But everything I’ve experienced so far working with LLMs screams that the fundamentals are more important than ever.

It usually works well for me. With very big tasks I break the plan into multiple MD files with the relevant context included and work through in individual sessions, updating remaining plans appropriately at the end of each one (usually there will be decision changes or additions during iteration).

It takes a lot of plan to use up the context and most of the time the agent doesn't need the whole plan, they just need what's relevant to the current task.

This was me. I have done a full 180 over the last 12 months or so, from "they're an interesting idea, and technically impressive, but not practically useful" to "holy shit I can have entire days/weeks where I don't write a single line of code".

> I really think a lof of people tried AI coding earlier, got frustrated at the errors and gave up. That's where the rejection of all these doomer predictions comes from.

It's not just the deficiencies of earlier versions, but the mismatch between the praise from AI enthusiasts and the reality.

I mean maybe it is really different now and I should definitely try uploading all of my employer's IP on Claude's cloud and see how well it works. But so many people were as hyped by GPT-4 as they are now, despite GPT-4 actually being underwhelming.

Too much hype for disappointing results leads to skepticism later on, even when the product has improved.

I feel similar, I'm not against the idea that maybe LLMs have gotten so much better... but I've been told this probably 10 times in the last few years working with AI daily.

The funny part about rapidly changing industries is that, despite the fomo, there's honestly not any reward to keeping up unless you want to be a consultant. Otherwise, wait and see what sticks. If this summer people are still citing the Opus 4.5 was a game changing moment and have solid, repeatable workflows, then I'll happily change up my workflow.

Someone could walk into the LLM space today and wouldn't be significantly at a loss for not having paid attention to anything that had happened in the last 4 years other than learning what has stuck since then.

> The funny part about rapidly changing industries is that, despite the fomo, there's honestly not any reward to keeping up unless you want to be a consultant.

I've lived through multiple incredibly rapid changes in tech throughout my career, and the lesson always learned was there is a lot of wasted energy keeping up.

Two big examples:

- Period from early mvc JavaScript frontends (backbone.js etc) and the time of the great React/Angular wars. I completely stepped out of the webdev space during that time period.

- The rapid expansion of Deep Learning frameworks where I did try to keep up (shipped some Lua torch packages and made minor contributions to Pylearn2).

In the first case, missing 5 years of front-end wars had zero impact. After not doing webdev work at all for 5-years I was tasked with shipping a React app. It took me a week to catch up, and everything was deployed in roughly the same time as someone would have had they spent years keeping up with changes.

In the second case, where I did keep up with many of the developing deep learning frameworks, it didn't really confer any advantage. Coworkers who I worked with who started with Pytorch fresh out of school were just as proficient, if not more so, with building models. Spending energy keeping up offered no value other than feeling "current" at the time.

Can you give me a counter example of where keeping up with a rapidly changing area that's unstable has conferred a benefit to you? Most of FOMO is really just fear. Again, unless you're trying to sell your self specifically as a consultant on the bleeding edge, there's no reason to keep up with all these changes (other than finding it fun).

You moved out of webdev for 5 years, not everybody else had that luxury. I'm sure it was beneficial to those people to keep up with webdev technologies.

If everything changes every month, then stuff you learn next month would be obsolete in two months. This is a response to people saying "adapt or be left behind". There's so much thrashing that if you're not interested with the SOTA, you can just wait for everything to calm down and pick it up then.

> On two occasions I have been asked, 'Pray, Mr. Babbage, if you put into the machine wrong figures, will the right answers come out?' I am not able rightly to apprehend the kind of confusion of ideas that could provoke such a question.

> Opus 4.5 really is at a new tier however. It just...works.

Literally tried it yesterday. I didn't see a single difference with whatever model Claude Code was using two months ago. Same crippled context window. Same "I'll read 10 irrelevant lines from a file", same random changes etc.

Create a markdown document of your task (or use CLAUDE.md), put it in "plan mode" which allows Claude to use tool calls to ask questions before it generates the plan.

When it finishes one part of the plan, have it create a another markdown document - "progress.md" or whatever with the whole plan and what is completed at that point.

Type /clear (no more context window), tell Claude to read the two documents.

Repeat until even a massive project is complete - with those 2 markdown documents and no context window issues.

> ... Proceeds to explain how it's crippled and all the workarounds you have to do to make it less crippled.

No - that's not what I did.

You don't need an extra-long context full of irrelevant tokens. Claude doesn't need to see the code it implemented 40 steps ago in a working method from Phase 1 if it is on Phase 3 and not using that method. It doesn't need reasoning traces for things it already "thought" through.

This other information is cluttering, not helpful. It is making signal to noise ratio worse.

If Claude needs to know something it did in Phase 1 for Phase 4 it will put a note on it in the living markdown document to simply find it again when it needs it.

Again, you're basically explaining how Claude has a very short limited context and you have to implement multiple workarounds to "prevent cluttering". Aka: try to keep context as small as possible, restart context often, try and feed it only small relevant information.

What I very succinctly called "crippled context" despite claims that Opus 4.5 is somehow "next tier". It's all the same techniques we've been using for over a year now.

I get by because I also have long-term memory, and experience, and I can learn. LLMs have none of that, and every new session is rebuilding the world anew.

And even my short-term memory is significantly larger than the at most 50% of the 200k-token context window that Claude has. It runs out of context before my short-term memory is probably not even 1% full, for the same task (and I'm capable of more context-switching in the meantime).

And so even the "Opus 4.5 really is at a new tier" runs into the very same limitations all models have been running into since the beginning.

> For LLMs long term memory is achieved by tooling. Which you discounted in your previous comments.

My specific complaint, which is an observable fact about "Opus 4.5 is next tier": it has the same crippled context that degrades the quality of the model as soon as it fills 50%.

EMM_386: no-no-no, it's not crippled. All you have to do is keep track across multiple files, clear out context often, feed very specific information not to overflow context.

Me: so... it's crippled, and you need multiple workarounds

scotty79: After all it's the same as your own short-term memory, and <some unspecified tooling (I guess those same files)> provide long-term memory for LLMs.

Me: Your comparison is invalid because I can go have lunch, and come back to the problem at hand and continue where I left off. "Next tier Opus 4.5" will have to be fed the entire world from scratch after a context clear/compact/in a new session.

Unless, of course, you meant to say that "next tier Opus model" only has 15-30 second short term memory, and needs to keep multiple notes around like the guy from Memento. Which... makes it crippled.

If you refuse to use what you call workarounds and I call long term memory then you end up with a guy from Memento and regardless of how smart the model is it can end up making same mistakes. And that's why you can't tell the difference between smarter and dumber one while others can.

I evaluated the claim that Opus is somehow next tier/something different/amazeballs future at its face value. It still has all the same issues and needs all the same workarounds as whatever I was using two months ago (I had a bit of a coding hiatus between beginning of December and now).

> then you end up with a guy from Memento and regardless of how smart the model is

Those models are, and keep being the guy from memento. Your "long memory" is nothing but notes scribbled everywhere that you have to re-assemble every time.

> And that's why you can't tell the difference between smarter and dumber one while others can.

If it was "next tier smarter" it wouldn't need the exact same workarounds as the "dumber" models. You wouldn't compare the context to the 15-30 second short-term memory and need unspecified tools [1] to have "long-term memory". You wouldn't have the model behave in an indistinguishable way from a "dumber" model after half of its context windows has been filled. You wouldn't even think about context windows. And yet here we are

[1] For each person these tools will be a different collection of magic incantations. From scattered .md files to slop like Beads to MCP servers providing access to various external storage solutions to custom shell scripts to ...

BTW, I still find "superpowers" from https://github.com/obra/superpowers to be the single best improvement to Claude (and other providers) even if it's just another in a long serious of magic chants I've evaluated.

> Those models are, and keep being the guy from memento. Your "long memory" is nothing but notes scribbled everywhere that you have to re-assemble every time.

That's exactly how the long term memory works in humans as well. The fact that some of these scribbles are done chemically in the same organ that does the processing doesn't make it much better. Human memories are reassembled at recall (often inaccurately). And humans also scribble when they try to solve a problem that exceeds their short term memory.

> If it was "next tier smarter" it wouldn't need the exact same workarounds as the "dumber" models.

This is akin to opposing calling processor next tier because it still needs RAM and bus to communicate with it and SSD as well. You think it should have everything in cache to be worthy of calling it next tier.

It's fine to have your own standards for applying words. But expect further confusion and miscommunication with other people if don't intend to realign.

> That's exactly how the long term memory works in humans as well.

Where this is applicable when is you go away from a problem for a while. And yet I don't lose the entire context and have to rebuild it from scratch when I go for lunch, for example.

Models have to rebuild the entire world from scratch for every small task.

> This is akin to opposing calling processor next tier because it still needs RAM and bus to communicate with it and SSD as well.

You're so lost in your own metaphor that it makes no sense.

> You think it should have everything in cache to be worthy of calling it next tier.

No. "Next tier" implies something significantly and observably better. I don't. And here you are trying to tell me "if you use all the exact same tools that you have already used before with 'previous tier models' you will see it is somehow next tier".

If your "next tier" needs an equator-length list of caveats and all the same tools, it's not next tier is it?

BTW. I'm literally coding with this "next tier" tool with "long memory just like people". After just doing the "plan/execute/write notes" bullshit incantations I had to correct it:

You're right, I fucked up on all three counts:
1. FileDetails - I should have WIRED IT UP, not deleted it.
It's a useful feature to preview file details before playing.
I treated "unused" as "unwanted" instead of "not yet connected".
2. Worktree not merged - Complete oversight. Did all the work but
didn't finish the job.
3. _spacing - Lazy fix. Should have analyzed why it exists and either
used it or removed the layout constraint entirely.

So next tier. So long memory. So person-like.

Oh. Within about 10 seconds after that it started compacting the "non-crippled" context window and immediately forgot most of what it had just been doing. So I had to clear out the context and teach it the world from the start again.

Edit. And now this amazing next tier model completely ignored that there already exists code to discover network interfaces, and wrote bullshit code calling CLI tools from Rust. So once again it needed to be reminded of this.

> It's fine to have your own standards for applying words. But expect further confusion and miscommunication with other people if don't intend to realign.

I mean, just like crypto bros before them, AI bros do sure love to invent their own terminology and their own realities that have nothing to do with anything real and observable.

> "You're right, I fucked up on all three counts:"

It very well might be that AI tools are not for you, if you are getting such poor results with your methods of approaching them.

If you would like to improve your outcomes at some point, ask people who achieve better results for pointers and try them out. Here's a freebie, never tell AI it fucked up.

200k+ tokens is a pretty big context window if you are feeding it the right context. Editors like Cursor are really good at indexing and curating context for you; perhaps it'd be worth trying something that does that better than Claude CLI does?

> a pretty big context window if you are feeding it the right context.

Yup. There's some magical "right context" that will fix all the problems. What is that right context? No idea, I guess I need to read a yet-another 20 000-word post describing magical incantations that you should or shouldn't do in the context.

The "Opus 4.5 is something else/nex tier/just works" claims in my mind means that I wouldn't need to babysit its every decision, or that it would actually read relevant lines from relevant files etc. Nope. Exact same behaviors as whatever the previous model was.

Oh, and that "200k tokens context window"? It's a lie. The quality quickly degrades as soon as Claude reaches somewhere around 50% of the context window. At 80+% it's nearly indistinguishable from a model from two years ago. (BTW, same for Codex/GPT with it's "1 million token window")

1) define problem
2) split problem into small independently verifiable tasks
3) implement tasks one by one, verify with tools

With humans 1) is the spec, 2) is the Jira or whatever tasks

With an LLM usually 1) is just a markdown file, 2) is a markdown checklist, Github issues (which Claude can use with the `gh` cli) and every loop of 3 gets a fresh context, maybe the spec from step 1 and the relevant task information from 2

I haven't ran into context issues in a LONG time, and if I have it's usually been either intentional (it's a problem where compacting wont' hurt) or an error on my part.

Yes and no. I've worked quite a bit with juniors, offshore consultants and just in companies where processes are a bit shit.

The exact same method that worked for those happened to also work for LLMs, I didn't have to learn anything new or change much in my workflow.

"Fix bug in FoobarComponent" is enough of a bug ticket for the 100x developer in your team with experience with that specific product, but bad for AI, juniors and offshored teams.

Thus, giving enough context in each ticket to tell whoever is working on it where to look and a few ideas what might be the root cause and how to fix it is kinda second nature to me.

Also my own brain is mostly neurospicy mush, so _I_ need to write the context to the tickets even if I'm the one on it a few weeks from now. Because now-me remembers things, two-weeks-from-now me most likely doesn't.

The problem with LLMs (similar to people :) ) is that you never really know what works. I've had Claude one-shot "implement <some complex requirement>" with little additional input, and then completely botch even the smallest bug fix with explicit instructions and context. And vice versa :)

I realize your experience has been frustrating. I hope you see that every generation of model and harness is converting more hold-outs. We're still a few years from hard diminishing returns assuming capital keeps flowing (and that's without any major new architectures which are likely) so you should be able to see how this is going to play out.

It's in your interest to deal with your frustration and figure out how you can leverage the new tools to stay relevant (to the degree that you want to).

Regarding the context window, Claude needs thinking turned up for long context accuracy, it's quite forgetful without thinking.

Note how nothing in your comment addresses anything I said. Except the last sentence that basically confirms what I said. This perfectly illustrates the discourse around AI.

As for the snide and patronizing "it's in your interest to stay relevant":

1. I use these tools daily. That's why I don't subscribe to willful wide-eyed gullibility. I know exactly what these tools can and cannot do.

The vast majority of "AI skeptics" are the same.

2. In a few years when the world is awash in barely working incomprehensible AI slop my skills will be in great demand. Not because I'm an amazing developer (I'm not), but because I have experience separating wheat from the chaff

The snide and patronizing is your projection. It kinda makes me sad when the discourse is so poisoned that I can't even encourage someone to protect their own future from something that's obviously coming (technical merits aside, purely based on social dynamics).

It seems the subject of AI is emotionally charged for you, so I expect friendly/rational discourse is going to be a challenge. I'd say something nice but since you're primed to see me being patronizing... Fuck you? That what you were expecting?

It's not me who decided to barge in, assume their opponent doesn't use something or doesn't want to use something, and offer unsolicited advice.

> It kinda makes me sad when the discourse is so poisoned that I can't even encourage someone to protect their own future from something that's obviously coming

See. Again. You're so in love with your "wisdom" that you can't even see what you sound like: snide, patronising, condenscending. And completely missing the whole point of what was written. You are literally the person who poisons the discourse.

Me: "here are the issues I still experience with what people claim are 'next tier frontier model'"

You: "it's in your interests to figure out how to leverage new tools to stay relevant in the future"

Me: ... what the hell are you talking about? I'm using these tools daily. Do you have anything constructive to add to the discourse?

> so I expect friendly/rational discourse is going to be a challenge.

It's only challenge to you because you keep being in love with your voice and your voice only. Do you have anything to contribute to the actual rational discourse, are you going to attack my character?

> 'd say something nice but since you're primed to see me being patronizing... Fuck you? T

Ah. The famous friendly/rational discourse of "they attack my use of AI" (no one attacked you), "why don't you invest in learning tools to stay relevant in the future" (I literally use these tools daily, do you have anything useful to say?) and "fuck you" (well, same to you).

> That what you were expecting?

What I was expecting is responses to what I wrote, not you riding in on a high horse.

You were the one complaining about how the tools aren't giving you the results you expected. If you're using these tools daily and having a hard time, either you're working on something very different from the bulk of people using the tools and your problems or legitimate, or you aren't and it's a skill issue.

If you want to take politeness as being patronizing, I'm happy to stop bothering. My guess is you're not a special snowflake, and you need to "get good" or you're going to end up on unemployment complaining about how unfair life is. I'd have sympathy but you don't seem like a pleasant human being to interact with, so have fun!

> ou were the one complaining about how the tools aren't giving you the results you expected.

They are not giving me the results people claim they give. It is distinctly different from not giving the results I want.

> If you're using these tools daily and having a hard time, either you're working on something very different from the bulk of people using the tools and your problems or legitimate, or you aren't and it's a skill issue.

Indeed. And your rational/friendly discourse that you claim you're having would start with trying to figure that out. Did you? No, you didn't. You immediately assumed your opponent is a clueless idiot who is somehow against AI and is incapable or learning or something.

> If you want to take politeness as being patronizing, I'm happy to stop bothering.

No. It's not politeness. It's smugness. You literally started your interaction in this thread with a "git gud or else" and even managed to complain later that "you dislike it when they attack your use of AI as a skill issue". While continuously attacking others.

> you don't seem like a pleasant human being to interact with

Says the person who has contributed nothing to the conversation except his arrogance, smugness, holier-than-thou attitude, engaged in nothing but personal attacks, complained about non-existent grievances and when called out on this behavior completed his "friendly and rational discourse" with a "fuck you".

Personally I'm sympathetic to people who don't want to have to use AI, but I dislike it when they attack my use of AI as a skill issue. I'm quite certain the workplace is going to punish people who don't leverage AI though, and I'm trying to be helpful.

> but I dislike it when they attack my use of AI as a skill issue.

No one attacked your use of AI. I explained my own experience with the "Claude Opus 4.5 is next tier". You barged in, ignored anything I said, and attacked my skills.

> the workplace is going to punish people who don't leverage AI though, and I'm trying to be helpful.

The only thing I disagreed with in your post is your objectively incorrect statement regarding Claude's context behavior. Other than that I'm just trying to encourage you to make preparations for something that I don't think you're taking seriously enough yet. No need to get all worked up, it'll only reflect on you.

And, conversely, when we read a comment like yours, it sounds like someone who's afraid of computers, would maybe have decried the bicycle and automobile, and really wishes they could just go live in a cabin in the woods.

(And it's fine to do so, just don't mail bombs to us, ok?)

> There's some magical "right context" that will fix all the problems.

All I can tell you is that in my own lived experience, I've had some fantastic results from AI, and it comes from telling it "look at this thing here, ok, i want you to chain it to that, please consider this factor, don't forget that... blah blah blah" like how I would have spelled things out to a junior developer, and then it really does stand a really solid chance of turning out what I've asked for. It helps a lot that I know what to ask for; there's no replacing that with AI yet.

So, your own situation must fall into one of these coarse buckets:

- You're doing something way too hard for AI to have a chance at yet, like real science / engineering at the frontier, not just boring software or infra development

- Your prompts aren't specific enough, you're not feeding it context, and you're expecting it to one-shot things perfectly instead of having to spend an afternoon prompting and correcting stuff

- You're not actually using and getting better at the tools, so you're just shouting criticisms from the sidelines, perhaps as sour grape because you're not allowed by policy / company can't afford to have you get into it.

IDK. I hope it's the first one and you're just doing Really Hard Things, but if you're doing normal software developer stuff and not seeing a productivity advantage, it's a fucking skill issue.

I'm not familiar with any form of intelligence that does not suffer from a bloated context. If you want to try and improve your workflow, a good place to start is using sub-agents so individual task implementations do not fill up your top level agents context. I used to regularly have to compact and clear, but since using sub-agents for most direct tasks, I hardly do anymore.

2. It's the same workarounds we've been doing forever

3. It's indistinguishable from "clear context and re-feed the entire world of relevant info from scratch" we've had forever, just slightly more automated

That's why I don't understand all the "it's new tier" etc. It's all the same issues with all the same workarounds.

That's because Opus has been out for almost 5 months now lol. Its the same model, so I think people have been vibe coding with a heavy dose of wine this holiday and are now convinced its the future.

Actually, I've been saying that even models from 2+ years ago were extremely good, but you needed to "hold them right" to get good results, else you might cut yourself on the sharp edges of the "jagged frontier" (https://www.hbs.edu/faculty/Pages/item.aspx?num=64700) Unfortunately, this often necessitated you to adapt yourself to the tool, which is a big change -- unfeasible for most people and companies.

I would say the underlying principle was ensuring a tight, highly relevant context (e.g. choose the "right" task size and load only the relevant files or even code snippets, not the whole codebase; more manual work upfront, but almost guaranteed one-shot results.)

With newer models the sharper edges have largely disappeared, so you can hold them pretty much any which way and still get very good results. I'm not sure how much of this is from the improvements in the model itself vs the additional context it gets from the agentic scaffolding.

I still maintain that we need to adapt ourselves to this new paradigm to fully leverage AI-assisted coding, and the future of coding will be pretty strange compared to what we're used to. As an example, see Gas Town: https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16d...

FWIW, Gas Town is strange because Steve is strange (in a good way).

It's just the same agent swarm orchestration that most agent frameworks are using, but with quirky marketing. All of that is just based on the SDLC [PM/Architect -> engineer planning group -> engineer -> review -> qa/evaluation] loop most people here should be familiar with. So actually pretty banal, which is probably part of the reason Steve decided to be zany.

Ah, gotcha, I am still working through the article, but its detailed focus on all the moving parts under the covers is making it hard to grok the high-level workflow.

Each failed prediction should lower our confidence in the next "it's finally useful!" claim. But this inductive reasoning breaks down at genuine inflection points.

I agree with your framing that measuring should NOT be separated from political issues, but each can be made clear separately (framing it as "training the tools of the oppressor" seems to conflate measuring tool usefulness with politics).

> How is it useful to you that these companies are so valuation hungry that they are moving money into this technology in such a way that people are fearful it could cripple the entire global economy?

The creation of entire new classes of profession has always been the result of technological breakthroughs. The automobile did not cripple the economy, even as it ended the buggy-whip barons.

> How is it useful to you that this tech is so power hungry that environmental externalities are being further accelerated while regular people's utility costs are raising to cover the increased demand(whether they use the tech to "code" or "manifest art")?

There will be advantages to lower-power computing, and lower-cost electricity. Implement carbon taxes and AI companies will follow the market incentive to install their datacentres in places where sustainable power is available for cheap. We'll see China soaring to new heights with their massive solar investment, and America will eventually figure out they have to catch up and cannot do so with coal and gas.

> How is it useful to you that this tech is so compute hungry that they are seemingly ending the industry of personal compute to feed this tech's demand?

Temporary problem, the demand for personal computing is not going to die in five years, and meanwhile the lucrative markets for producing this equipment will result in many new factories, increasing capacity and eventually lowering prices again. In the meantime, many pundits are suggesting that this may thankfully begin the end of the Electron App Era where a fuckin' chat client thinks it deserves 1GB of RAM.

Consider this: why are we using Electron and needing 32GB of RAM on a desktop? Because web developers only knew how to use Javascript and couldn't write a proper desktop app. With AI, desktop frameworks can have a resurgence; why shouldn't I use Go or Rust and write a native app on all platforms now that the cost of doing so is decreasing and the number of people empowered to work with it is increasing? I wrote a nice multithreaded fractal renderer in Rust the other day; I don't know how to multithread, write Rust, and probably can't iterate complex numbers correctly on paper anymore....

> How is it useful to you that this tech is so water hungry that it is emptying drinking water acquifers?

This is only a problem in places that have poor water policy, e.g. California (who can all thank the gods that their reservoirs are all now very full from the recent rain). This problem predates datacenters and needs to be solved - for instance, by federalizing and closing down the so-called Wonderful Company and anyone else who uses underhanded tactics to buy up water rights to grow crops that shouldn't be grown there.

Come and run your datacenters up in the cold North, you won't even need evaporative cooling for them, just blow a ton of fresh air in....

> How is it useful to you that this tech is being used to manufacture consent?

Now you've actually got an argument, and I am on your side on this one.

> If at any point any of these releases were "genuine inflection points" it would be unnecessary to proselytize such. It would be self evident. Much like rain.

Agreed.

Now, I suggest reading through all of this to note that I am not a fan of tech bros, that I do want this to be a bubble. Then also note what else I'm saying despite all that.

To me, it is self-evident. The various projects I have created by simply asking for them, are so. I have looked at the source code they produce, and how this has changed over time: Last year I was describing them as "junior" coders, by which I meant "fresh hire"; now, even with the same title, I would say "someone who is just about to stop being a junior".

> "The oppressed need to acknowledge that their oppression is useful to their oppressors."

The capacity for AI to oppress you is in direct relation to its economic value.

> How is it useful to you that this tech is so power hungry that environmental externalities are being further accelerated while regular people's utility costs are raising to cover the increased demand(whether they use the tech to "code" or "manifest art")?

The power hunger is in direct proportion to the demand. Someone burning USD 20 to get Claude Code tokens has consumed approximately USD 10 of electricity in that period, with the other USD 10 having been spread between repaying the model training cost and the server construction cost.

The reason they're willing to spend USD 20 is to save at least US 20 worth of dev time. This was already the case with the initial version of ChatGPT pro back in the day, when it could justify that by saving 23 dev minutes per month. There's around a million developers in the USA, just that group increasing electricity spending by USD 10/month will put a massive dent on the USA's power grid.

Gets worse though. Based on my experience, using Claude Code optimally, when you spend USD 20 you get at least 10 junior sprints' worth of output. Hiring a junior for 10 sprints is, what, USD 30,000? The bound here is "are you able to get value from having hired 1,500 juniors for the price of one?"

One can of course also waste those tokens. Both because nobody needs slop, and because most people can't manage one junior never mind 1500 of them.

However, if the economy collectively answers "yes", then the environmental externalities expand until you can't afford to keep your fridge cold or your lights on.

This is one of the failure modes of the technological singularity that people like me have been forewarning about for years, even when there's no alignment issues within the models themselves. Which there are, because Musk's one went and called itself Mecha Hitler, while being so sycophantic about Musk himself that it called him the best at everything even when the thing was "drinking piss", which would be extremely funny if he wasn't selling this to the US military.

> How is it useful to you that this tech is so compute hungry that they are seemingly ending the industry of personal compute to feed this tech's demand?

This will pass. Either this is a bubble, it pops, the manufacturers return to their roots; or it isn't because it works as advertised, which means it leads to much higher growth rates, and we (us, personally, you and me) get personal McKendree cylinders each with more compute than currently exists… or we get turned into the raw materials for those cylinders.

I assume the former. But I say that as one who wants it to be the former.

> How is it useful to you that this tech is so water hungry that it is emptying drinking water acquifers?

Is it what's emptying drinking water acquifers?

The combined water usage of all data centers in Arizona. All of them. Together. Which is over 100 DCs. All of them combined use about double what Tesla was expecting from just the Brandenburg Gigafactory to use before Musk decided to burn his reputation with EV consumers and Europeans for political point scoring.

> How is it useful to you that this tech is being used to manufacture consent?

This is one of the objectively bad things, though it's hard to say if this is more or less competent at this than all the other stuff we had three years ago, given the observed issues with the algorithmic feeds.

I appreciate you taking the time to write up your thoughts on something other than exclusively these tools 'usefulness' at writing code.

> The capacity for AI to oppress you is in direct relation to its economic value.

I think this assumes a level of rationality in these systems, corporate interests and global markets, that I would push back on as being largely absent.

> The power hunger is in direct proportion to the demand.

Do you think this is entirely the case? I mean, I understand what you are saying, but I would draw stark lines between "company" demand versus "user" demand. I have found many times the 'AI' tools are being thrust into nearly everything regardless of user demand. Spinning its wheels to only ultimately cause frustration. [0]

> Is it what's emptying drinking water aquifers?

It appears this is a problem, and will only continue to be such. [1]

> The combined water usage of all data centers in Arizona. All of them. Together. Which is over 100 DCs. All of them combined use about double what Tesla was expecting from just the Brandenburg Gigafactory to use before Musk decided to burn his reputation with EV consumers and Europeans for political point scoring.

I am unsure if I am getting what your statements here are trying to say. Would you be able to restate this to be more explicit in what you are trying to communicate.

> I think this assumes a level of rationality in these systems, corporate interests and global markets, that I would push back on as being largely absent.

Could be. What I hope and suspect is happening is that these companies are taking a real observation (the economic value that I also observe in software) and falsely expanding this to other domains.

Even to the extent that these work, AI has clearly been over-sold in humanoid robotics and self-driving systems, for example.

> Do you think this is entirely the case? I mean, I understand what you are saying, but I would draw stark lines between "company" demand versus "user" demand. I have found many times the 'AI' tools are being thrust into nearly everything regardless of user demand. Spinning its wheels to only ultimately cause frustration. [0]

I think it is. Companies setting silly goals like everyone must use LLMs once a day or whatever, that won't burn a lot of tokens. Claude Code is available in both subscription mode and PAYG mode, and the cost of subscriptions suggests it is burning millions of tokens a month for the basic subscription.

Other heavy users who we would both agree are bad, are slop content farms. I cannot even guesstimate those, so would be willing to accept the possibility they're huge.

> It appears this is a problem, and will only continue to be such. [1]

I find no reference to "aquifers" in that.

Where it says e.g. "up to 9 liters of water to evaporate per kWh of energy used", the average is 1.9 l/kWh. Also, evaporated water tends to fall nearby (on this scale) as rain, so unless there's now too much water on the surface, this isn't a net change even if it all comes form an aquifer (and I have yet to see any evidence of DCs going for that water source).

It says "The U.S. relies on water-intensive thermoelectric plants for electricity, indirectly increasing data centers' water footprint, with an average of 43.8L/kWh withdrawn for power generation." - most water withdrawn is returned, not consumed.

It says "Already AI's projected water usage could hit 6.6 billion m³ by 2027, signaling a need to tackle its water footprint.", this is less than the famously-a-desert that is Arizona.

> I am unsure if I am getting what your statements here are trying to say. Would you be able to restate this to be more explicit in what you are trying to communicate.

That the water consumption of data centres is much much smaller than the media would have you believe. It's more of a convenient scare story than a reality. If water is your principal concern, give up beef, dairy, cotton, rice, almonds, soy, biofuels, mining, paper, steel, cement, residential lawns, soft drinks, car washing, and hospitals, in approximately that order (assuming the lists I'm reading those from are not invented whole cloth), before you get to data centres.

And again, I don't disagree that they're a problem, it's just that the "water" part of the problem is so low down the list of things to worry about as to be a rounding error.

Ahh, I see your objection now. That is my bad. I was using my language too loosely. Here I was using 'aquifer' to mean 'any source of drinking water', but that is certainly different from the intended meaning.

> And again, I don't disagree that they're a problem, it's just that the "water" part of the problem is so low down the list of things to worry about as to be a rounding error.

I'm skeptical of the rounding error argument, and weary of relying on the logical framework of 'low down the list' when list items' effects stack interdependently.

> give up beef, dairy, cotton, rice, almonds, soy, biofuels, mining, paper, steel, cement, residential lawns, soft drinks, car washing, and hospitals

In part due to this reason, as well as others, I have stopped directly supporting the industries for: beef, dairy, rice, almonds, soy, biofuels, residential lawns, soft drinks, car washing

The hype curve is a problem, but it's difficult to prevent. I myself have never made such a prediction. Though it now seems that the money and effort to create working coding tools is near an inflection point.

"It would be self evident." History shows the opposite at inflection points. The "self evident" stage typically comes much later.

It's a little weird how defensive people are about these tools. Did everyone really think being able to import a few npm packages, string together a few APIs, and run npx create-react-app was something a large number of people could do forever?

The vast majority of coders in employment barely write anything more complex than basic CRUD apps. These jobs were always going to be automated or abstracted away sooner or later.

Every profession changes. Saying that these new tools are useless or won't impact you/xyz devs is just ignoring a repeated historical pattern

It's employing so may people who specialize in Salesforce configuration that every year San Francisco collapses under the weight of 50,000+ of them attending Dreamforce.

And it's actually kind of amazing, because a lot of people who earn six figures programming Salesforce came to it from a non-traditional software engineering background.

I think perhaps for some folks we're looking at their first professional paradigm shift. If you're a bit older, you've seen (smaller versions of) the same thing happening before as e.g. the Internet gained traction, Web2.0, ecommerce, crypto, etc. and have seen your past skillset become useless as now it can be accomplished for only $10/mo/user.... either you pivot and move on somehow, or you become a curmudgeon. Truly, the latter is optional, and at any point when you find yourself doing that you wish to stop and just embrace the new thing, you're still more than welcome to do so. AI is only going to get EASIER to get involved with, not harder.

And by the same token (ha) for some folks we're looking at their first hype wave. If you're a bit older, you've seen similar things like 4GLs and visual programming languages and blockchain and expert systems. They each left their mark on our profession but most of their promises were unfounded and ultimately unrealized.

I like a lot of 4GL ideas. Closest I've come was working on ServiceNow which is sort of a really powerful system with ugly, ugly roots but the idea of your code being the database being the code really resonated with me, as a self-taught programmer.

Similarly, Lisp's homoiconicity makes sense to me as a wonderfully aesthetic idea. I remember generating strings-of-text that were code, but still just text, and wishing that I could trivially step into the structure there like it was a map/dict... without realizing that that's what an AST is and what the language compiler / runtime is already always doing.

Lol. In a few years when the world is awash in AI-generated slop [1] my "past skills" will not only be relevant, they will be actively sought after.

[1] Like the recent "Gas Town" and "Beads" that people keep mentioning in the comments that require extensive scripts/human intervention to purge from the system: https://news.ycombinator.com/item?id=46510121

Agreed, it always seemed a little crazy that you could make wild amounts of money to just write software. I think the music is finally stopping and we'll all have to go back to actually knowing how to do something useful.

> The vast majority of coders in employment barely write anything more complex than basic CRUD apps. These jobs were always going to be automated or abstracted away sooner or later.

My experience has been negative progress in this field. On iOS, UIKit in Interface Builder is an order of magnitude faster to write and to debug, with less weird edge cases, than SwiftUI was last summer. I say last summer because I've been less and less interested in iOS the more I learn about liquid glass, even ignoring the whole "aaaaaaa" factor of "has AI made front end irrelevant anyway?" and "can someone please suggest something the AI really can't do so I can get a job in that?"

The 80s TUI frameworks are still not beaten in developer productivity buy GUI or web frameworks. They have been beaten by GUIs in usability, but then the GUIs reverted into a worse option.

Too bad they were mostly proprietary and won't even run in modern hardware.

Democratizing coding so regular people can get the most out of computers is the opposite of oppression. You are mistaking your interests for societies interests.

It's the same with artists who are now pissed that regular people can manifest their artistic ideas without needing to go through an artist or spend years studying the craft. The artists are calling the AI companies oppressors because they are breaking the artist's stranglehold on the market.

It's incredibly ironic how socializing what was a privatized ability has otherwise "socialist" people completely losing their shit. Just the mask of pure virtue slipping...

On what planet is concentrating an increasingly high amount of the output of this whole industry on a small handful of megacorps “democratising” anything?

Software development was already one of the most democratised professions on earth. With any old dirt cheap used computer, an internet connection, and enough drive and curiosity you could self-train yourself into a role that could quickly become a high paying job. While they certainly helped, you never needed any formal education or expensive qualifications to excel in this field. How is this better?

The open models don't have access to all the proprietary code that the closed ones have trained on.

That's primarily why I finally had to suck it up and sign up for Claude. Claude clearly can cough up proprietary codebase examples that I otherwise have no access to.

Given that very few of the "open models" disclose their training data there's no reason at all to assume that the proprietary models have an advantage in terms of training on proprietary data.

As far as I can tell the reason OpenAI and Anthropic are ahead in code is that they've invested extremely heavily in figuring out the right reinforcement learning training mix needed to get great coding results.

Some of the Chinese open models are already showing signs of catching up.

> deergomoo: On what planet is concentrating an increasingly high amount of the output of this whole industry on a small handful of megacorps “democratising” anything?

> simonw: It's better because now you can automate something tedious in your life with a computer without having to first climb a six month learning curve.

Completely ignores, or enthusiastically accepts and endorses, the consolidation of production, power, and wealth into a stark few (friends), and claims superiority and increased productivity without evidence?

This may be the most simonw comment I have ever seen.

At the tail end of 2023 I was deeply worried about consolidation of power, because OpenAI were the only lab with a GPT-4 class model and none of their competitions had produced anything that matched it in the ~8 months since it had launched.

I'm not worried about that at all any more. There are dozens of organizations who have achieved that milestone now, and OpenAI aren't even definitively in the lead.

A lot of those top-class models are open weight (mainly thanks to the Chinese labs) and available for people to run on their own hardware.

I used claude code to set up a bunch of basic tools my wife was using in her daily work. Things like custom pomodoro timers, task managers, todo notes.

She used to log into 3 different websites. Now she just opens localhost:3000 and has all of them on the same page. No emails shared with anyone. All data stored locally.

I could have done this earlier but the time commitment with Claude Code now was writing a spec in 5-minutes and pressing approve a few times vs half a day.

I count this as an absolute win. No privacy breaches, no data sharing.

> The artists are calling the AI companies oppressors because they are breaking the artist's stranglehold on the market.

Tt's because these companies profit from all the existing art without compensating the artists. Even worse, they are now putting the very people out of a job who (unwittingly) helped to create these tools in the first place. Not to mention how hurtful it must be for artists seeing their personal style imitated by a machine without their consent.

I totally see how it can empower regular people, but it also empowers the megacorps and bad actors. The jury is still out on whether AI is providing a net positive to society. Until then, let's not ignore the injustice and harm that went into creating these tools and the potential and real dangers that come with it.

When you imagine my position, "I hate these companies for democratizing code/art", then debate that it is called a strawman logical fallacy.

Ascribing the goals of "democratize code/art" onto these companies and their products is called delusion.

I am sure the 3 letter agency directors on these company boards are thrilled you think they left their lifelong careers solely to finally realize their dream to allow you to code and "manifest your artistic ideas".

Yes, but the quality of output from open/local models isn't anywhere close to what you get from Claude or Gemini. You need serious hardware to get anything approaching decent processing speeds or even middling quality.

It's more economical for the average person to spend $20/month on a subscription than it is for them to drop multiple thousands $ and untold hours of time experimenting. Local AI is a fun hobby though.

> If I am unable to convince you to stop meticulously training the tools of the oppressor (for a fee!) then I just ask you do so quietly.

I'm kind of fascinated by how AI has become such a culture war topic with hyperbole like "tools of the oppressor"

It's equally fascinating how little these comments understand about how LLMs work. Using an LLM for inference (what you do when you use Claude Code) does not train the LLM. It does not learn from your code and integrate it into the model while you use it for inference. I know that breaks the "training the tools of the oppressor" narrative which is probably why it's always ignored. If not ignored, the next step is to decry that the LLM companies are lying and are stealing everyone's code despite saying they don't.

The prompts and responses are used as training data. Even if your provider allows you to opt out they are still tracking your usage telemetry and using that to gauge performance. If you don’t own the storage and compute then you are training the tools which will be used to oppress you.

> The prompts and responses are used as training data.

They show a clear pop-up where you choose your setting about whether or not to allow data to be used for training. If you don't choose to share it, it's not used.

I mean I guess if someone blindly clicks through everything and clicks "Accept" without clicking the very obvious slider to turn it off, they could be caught off guard.

Assuming everyone who uses Claude is training their LLMs is just wrong, though.

Telemetry data isn't going to extract your codebase.

I am curious where your confidence that this is true, is coming from?

Besides lots of GPU's, training data seems the most valuable asset AI companies have. Sounds like strong incentive to me to secretly use it anyway. Who would really know, if the pipelines are set up in a way, if only very few people are aware of this?

And if it comes out "oh gosh, one of our employees made a misstake".

And they already admitted to train with pirated content. So maybe they learned their lesson .. maybe not, as they are still making money and want to continue to lead the field.

1. There are good, ethical people working at these companies. If you were going to train on customer data that you had promised not to train on there would be plenty of potential whistleblowers.

2. The risk involved in training on customer data that you are contractually obliged not to train on is higher than the value you can get from that training data.

3. Every AI lab knows that the second it comes out that they trained on paying customer data saying they wouldn't, those paying customers will leave for their competitors (and sue them int the bargain.)

4. Customer data isn't actually that valuable for training! Great models come from carefully curated training data, not from just pasting in anything you can get your hands on.

Fundamentally I don't think AI labs are stupid, and training on paid customer data that they've agreed not to train on is a stupid thing to do.

1. The people working for these companies are already demonstrably ethically flexible enough to pirate any publicly accessible training data they can get their hands on, including but not limited to ignoring the license information in every repo on GitHub. I'm not impressed with any of these clowns and I wouldn't trust them to take care of a potted cactus.

2. The risk of using "illegal" training data is irrelevant, because no GenAI vendors have been meaningfully punished for violating copyright yet, and in the current political climate they don't expect to be anytime soon. Even so,

3. Presuming they get caught redhanded using personal data without permission- which, given the nature of LLMs would be extremely challenging for any individual customer to prove definitively- they may lose customers, and customers may try to sue, but you can expect those lawsuits to take years to work their way through the courts; long after these companies IPO, employees get their bag, and it all becomes someone else's problem.

4. The idea of using carefully curated datasets is popular rhetoric, but absolutely does not reflect how the biggest GenAI vendors do business. See (1).

AI labs are extremely shortsighted, sloppy, and demonstrably do not care a single iota about the long term when there's money to be made in the short term. Employees have gigantic financial incentives to ignore internal malfeasance or simple ineptitude. The end result is, if anything, far worse than stupidity.

There is an important difference between openly training on scraped web data and license-ignored data from GitHub and training on data from your paying customers that you promised you wouldn't train on.

Anthropic had to pay $1.5bn after being caught downloading pirated ebooks.

So Anthropic had to pay less than 1% of their valuation despite approximately their entire business being dependent on this and similar piracy. I somehow doubt their takeaway from that is "let's avoid doing that again".

First: Valuations are based on expected future profits.

For a lot of companies, 1% of valuation is ~20% of annual profit (P/E ratio 5); for fast growing companies, or companies where the market is anticipating growth, it can be a lot higher. Weird outlier example here, but consider that if Tesla was fined 1% of its valuation (1% of 1.5 trillion = 15 billion), that would be most of the last four quarter's profit on https://www.macrotrends.net/stocks/charts/TSLA/tesla/gross-p...

Second: Part of the Anthropic case was that many of the books they trained on were ones they'd purchased and destructively scanned, not just pirated. The courts found this use was fine, and Anthropic had already done this before being ordered to: https://storage.courtlistener.com/recap/gov.uscourts.cand.43...

Every single point you made is contradicted by the observed behavior of the AI labs. If any of those factors were going to stop them from training on data they legally can't, they would have done so already.

> I am curious where your confidence that this is true, is coming from?

My confidence comes from working in big startups and big companies with legal teams. There's no way the entire company is going to gather all of the engineers and everyone around, have them code up a secret system to consume customer data into a secret part of the training set, and then have everyone involved keep quiet about it forever.

The whistleblowing and leaking would happen immediately. We've already seen LLM teams leak and and have people try to whistleblow over things that aren't even real, like the Google engineer who thought they had invented AGI a few years ago (lol). OpenAI had a public meltdown when the employees disagreed with Sam Altman's management style.

So my question to you is: What makes you think they would do this? How do you think they'd coordinate the teams to keep it all a secret and only hire people who would take this secret to their grave?

"There's no way the entire company is going to gather all of the engineers and everyone around, have them code up a secret system "

No, that is why I wrote

"Who would really know, if the pipelines are set up in a way, that only very few people are aware of this?" (Typo fixed)

There is no need for everyone to know. I don't know their processes, but I can think of ways to only include very few people who need to know.

The rest is just working on everything else. Some work with data, where they don't need to know where it came from, some with UI, some with scaling up, some .. they all don't need to know, that the source of DB XYZ comes from a dark source.

> I am curious where your confidence that this is true, is coming from?

We have a legal binding contract with Anthropic. Checked and vetted by our laywers, who are annoying because they actually READ the contracts and won't let us use services with suspicious clauses in them - unless we can make amendments.

If they're found to be in breach of said contract (which is what every paid user of Claude signs), Anthropic is going to be the target of SO FUCKING MANY lawsuits even the infinite money hack of AI won't save them.

> Besides lots of GPU's, training data seems the most valuable asset AI companies have. Sounds like strong incentive to me to secretly use it anyway. Who would really know, if the pipelines are set up in a way, if only very few people are aware of this?

Could be, but it's a huge risk the moment any lawsuit happens and the "discovery" process starts. Or whistleblowers.

They may well take that risk, they're clearly risk-takers. But it is a risk.

Eh they’re all using copyrighted training data from torrent sites anyway. If the government was gonna hold them accountable for this it would have happened already.

I find it hard to believe there are people who know these companies stole the entire creative output of humanity and egregiously continually scrape the internet are, for some reason, ignoring the data you voluntarily give them.

> I know that breaks the "training the tools of the oppressor" narrative

"Narrative"? This is just reality. In their own words:

> The awards to Anthropic, Google, OpenAI, and xAI – each with a $200M ceiling – will enable the Department to leverage the technology and talent of U.S. frontier AI companies to develop agentic AI workflows across a variety of mission areas. Establishing these partnerships will broaden DoD use of and experience in frontier AI capabilities and increase the ability of these companies to understand and address critical national security needs with the most advanced AI capabilities U.S. industry has to offer. The adoption of AI is transforming the Department’s ability to support our warfighters and maintain strategic advantage over our adversaries [0]

Is 'warfighting adversaries' some convoluted code for allowing Aurornis to 'see a 1337x in productivity'?

Or perhaps you are a wealthy westerner of a racial and sexual majority and as such have felt little by way of oppression by this tech?

In such a case I would encourage you to develop empathy, or at least sympathy.

> Using an LLM for inference .. does not train the LLM.

In their own words:

> One of the most useful and promising features of AI models is that they can improve over time. We continuously improve our models through research breakthroughs as well as exposure to real-world problems and data. When you share your content with us, it helps our models become more accurate and better at solving your specific problems and it also helps improve their general capabilities and safety. We do not use your content to market our services or create advertising profiles of you—we use it to make our models more helpful. ChatGPT, for instance, improves by further training on the conversations people have with it, unless you opt out.

> Is 'warfighting adversaries' some convoluted code for allowing Aurornis to 'see a 1337x in productivity'?

Much as I despair at the current developments in the USA, and I say this as a sexual minority and a European, this is not "tools of the oppressor" in their own words.

Trump is extremely blunt about who he wants to oppress. So is Musk.

"Support our warfighters and maintain strategic advantage over our adversaries" is not blunt, it is the minimum baseline for any nation with assets anyone else might want to annex, which is basically anywhere except Nauru, North Sentinel Island, and Bir Tawil.

> "Support our warfighters and maintain strategic advantage over our adversaries" is not blunt, it is the minimum baseline for any nation with assets anyone else might want to annex

I think its gross to distill military violence as defending 'assets [others] might want to annex'.

What US assets were being annexed when US AI was used to target Gazans?

> I think its gross to distill military violence as defending 'assets [others] might want to annex'.

Yes, but that's how the world works:

Another country wants a bit of your country for some reason, they can take it by force unless you can make at the very least a credible threat against them, sometimes a lot more than that.

Note that this does not exclude that there has to be an aggressor somewhere. I'm not excluding the existence of aggressors, nor the capacity for the USA to be an aggressor. All I'm saying is your quotation is so vague as to also encompass those who are not.

> What US assets were being annexed when US AI was used to target Gazans?

First, I'm saying the statement is so broad as to encompass other things besides being a warmonger. Consider the opposite statement: "don't support our warfighters and don't maintain strategic advantage over our adversaries" would be absolutely insane, therefore "support our warfighters and maintain strategic advantage over our adversaries" says nothing.

Second, in this case the country doing the targeting is… Israel. To the extent that the USA cares at all, it's to get votes from the large number of Jewish people living in the USA. Similar deal with how it treats Cuba since the fall of the USSR: it's about votes (from Cuban exiles in that case, but still, votes).

Much as I agree that the conduct of Israel with regard to Gaza was disproportionate, exceeded the necessity, and likely was so bad as to even damage Israel's long-term strategic security, if you were to correctly imagine the people of Israel deciding "don't support our warfighters and don't maintain strategic advantage over our adversaries", they would quickly get victimised much harder than those they were victimising. That's the point there: the quote you cite as evidence, is so broad that everyone has approximately that, because not having it means facing ones' own destruction.

There's a mis-attributed quote, "People sleep peaceably in their beds at night because rough men stand ready to do violence on their behalf", that's where this is at.

> These two thoughts seem at conflict.

Musk is openly and directly saying "Canada is not a real country.", says "cis" is hate speech, response to pandemic was tweeting "My pronouns are Prosecute/Fauci.", and self-justification for his trillion dollar bonus for hitting future targets is wanting to be in control of what he describes as a "robot army"; Trump openly and explicitly wants the USA to annex Canada, Greenland, Panama canal, is throwing around the national guard, openly calls critics traitors and calls for death penalty. They're a subtle as exploding volcanoes, nobody needs to take the worst case interpretations of what they're saying to notice this.

Saying "support our warfighters" is something done by basically every nation everywhere all the time, because those places that don't do this quickly get taken over by nearby nations who sense weakness. Which is kinda how the USA got Texas, because again, I'm not saying the USA is harmless, I'm saying the quote doesn't show that.

> What 'assets' were being protected from annexation here by this oppressive use of the tool? The chips?

This would have been a much better example to lead with than the military stuff.

I'm absolutely all on board with the general consensus that the US police are bastards in this specific way, have been since that kid got shot for having a toy gun in an open-carry state. (I am originally from a country where even the police are not routinely armed, I do not value the 2nd amendment, but if you're going to say "we allow open carry of firearms" you absolutely do not get to use "we saw someone carrying a firearm" as an excuse to shoot them).

However: using LLMs to code doesn't seem to be likely to make a difference either way for this. If I was writing a gun-detection AI, perhaps I'm out of date, but I'd use a simpler model that runs locally on-device and doesn't do anything else besides the sales pitch.

Who is the parent oppressing? Making a comment and companies looking to automate labor are a little bit different. One might disagree that automation is oppressive or whatever goals the major tech CEOs have in developing AIs (surveillance, influencing politics, increasing wealth gap), but certainly commenting that they are oppressive is not the same thing.

Careful with being blindly led by your own assumptions.

I actually disagree with your thesis here. I think if every comment was posted under a new account this site would improve its average veracity.

As it stands certain 'celebrity', or high karma, accounts are artificially bolstered by the network effect indifferent to the defensibility of their claims.

I know someone who is using a vibe coded or at least heavily assisted text editor, praising it daily, while also saying llms will never be productive. There is a lot of dissonance right now.

I teach at a university, and spend plenty of time programming for research and for fun. Like many others, I spent some time on the holidays trying to push the current generation of Cursor, Claude Code, and Codex as far as I could. (They're all very good.)

I had an idea for something that I wanted, and in five scattered hours, I got it good enough to use. I'm thinking about it in a few different ways:

1. I estimate I could have done it without AI with 2 weeks full-time effort. (Full-time defined as >> 40 hours / week.)

2. I have too many other things to do that are purportedly more important that programming. I really can't dedicate to two weeks full-time to a "nice to have" project. So, without AI, I wouldn't have done it at all.

3. I could hire someone to do it for me. At the university, those are students. From experience with lots of advising, a top-tier undergraduate student could have achieved the same thing, had they worked full tilt for a semester (before LLMs). This of course assumes that I'm meeting them every week.

This is where the LLM coding shines in my opinion, there's a list of things they are doing very well:

- single scripts. Anything which can be reduced to a single script.

- starting greenfield projects from scratch

- code maintenance (package upgrades, old code...)

- tasks which have a very clear and single definition. This isn't linked to complexity, some tasks can be both very complex but with a single definition.

If your work falls into this list they will do some amazing work (and yours clearly fits that), if it doesn't though, prepare yourself because it will be painful.

I'm trying to determine what programming tasks are not in this list. :) I think it is trying to exclude adding new features and fixing bugs in existing code. I've done enough of that with LLMs, though not in large codebases.

I should say I'm hardly ever vibe-coding, unlike the original article. If I think I want code that will last, I'll steer the models in ways that lean on years of non-LLM experience. E.g., I'll reject results that might work if they violate my taste in code.

It also helps that I can read code very fast. I estimate I can read code 100x faster than most students. I'm not sure there is any way to teach that other than the old-fashioned way, which involves reading (and writing) a lot of code.

> I'm trying to determine what programming tasks are not in this list. :) I think it is trying to exclude adding new features and fixing bugs in existing code

Yes indeed, these are the things on the other hand which aren't working well in my opinion:

- large codebase

- complex domain knowledge

- creating any feature where you need product insights

- tasks requiring choices (again, complexity doesn't matter here, the task may be simple but require some choices)

- anything unclear where you don't know where you are going first

While you don't experience any of these when teaching or side projects, these are very common in any enterprise context.

How do you compare Claude Code to Cursor? I'm a Cursor user quietly watching the CC parade with curiosity. Personally, I haven't been able to give up the IDE experience.

Im so sold on the cli tools that I think IDEs are basically dead to me. I only have an IDE open so I can read the code, but most often I'm just changing configs (like switching a bool, or bumping up a limit or something like that).

Seriously, I have 3+ claude code windows open at a time. Most days I don't even look at the IDE. It's still there running in the background, but I don't need to touch it.

When I'm using Claude Code, I usually have a text editor open as well. The CC plugin works well enough to achieve most of what Cursor was doing for me in showing real-time diffs, but in my experience, the output is better and faster. YMMV

I use CC for so much more than just writing code that I cannot imagine being constrained within an IDE. Why would I want to launch an IDE to have CC update the *arr stack on my NAS to the latest versions for example? Last week I pointed CC at some media files that weren't playing correctly on my Apple TV. It detected what the problem formats were and updated my *arr download rules to prefer other releases and then configured tdarr to re-encode problem files in my existing library.

I was here a few weeks ago, but I'm now on the CC train. The challenge is that the terminal is quite counterintuitive. But if you put on the Linux terminal lens from a few years ago, and you start using it. It starts to make sense. The form factor of the terminal isn't intuitive for programming, but it's the ultimate.

FYI, I still use cursor for small edits and reviews.

I don't think I can scientifically compare the agents. As it is, you can use Opus / Codex in Cursor. The speed of Cursor composer-1 is phenomenal -- you can use it interactively for many tasks. There are also tasks that are not easier to describe in English, but you can tab through them.

> Most software engineers are seriously sleeping on how good LLM agents are right now, especially something like Claude Code.

Nobody is sleeping. I'm using LLMs daily to help me in simple coding tasks.

But really where is the hurry? At this point not a few weeks go by without the next best thing since sliced bread to come out. Why would I bother "learning" (and there's really nothing to learn here) some tool/workflow that is already outdated by the time it comes out?

> 2026 is going to be a wake-up call

Do you honestly think a developer not using AI won't be able to adapt to a LLM workflow in, say, 2028 or 2029? It has to be 2026 or... What exactly?

There is literally no hurry.

You're using the equivalent of the first portable CD-player in the 80s: it was huge, clunky, had hiccups, had a huge battery attached to it. It was shiny though, for those who find new things shiny. Others are waiting for a portable CD player that is slim, that buffers, that works fine. And you're saying that people won't be able to learn how to put a CD in a slim CD player because they didn't use a clunky one first.

I think getting proficient at using coding agents effectively takes a few months of practice.

It's also a skill that compounds over time, so if you have two years of experience with them you'll be able to use them more effectively than someone with two months of experience.

In that respect, they're just normal technology. A Python programmer with two years of Python experience will be more effective than a programmer with two months of Python.

> Nobody is sleeping. I'm using LLMs daily to help me in simple coding tasks.

That is sleeping.

> But really where is the hurry? At this point not a few weeks go by without the next best thing since sliced bread to come out. Why would I bother "learning" (and there's really nothing to learn here) some tool/workflow that is already outdated by the time it comes out?

You're jumping to conclusions that haven't been justified by any of the development in this space. The learning compounds.

> Do you honestly think a developer not using AI won't be able to adapt to a LLM workflow in, say, 2028 or 2029? It has to be 2026 or... What exactly?

They will, but they'll be competing against people with 2-3 more years of experience in understanding how to leverage these tools.

"But really where is the hurry?" It just depends on why you're programming. For many of us not learning and using up to date products leads to a disadvantage relative to our competition. I personally would very much rather go back to a world without AI, but we're forced to adapt. I didn't like when pagers/cell phones came out either, but it became clear very quickly not having one put me at a disadvantage at work.

The crazy part is, once you have it setup and adapted your workflow, you start to notice all sorts of other "small" things:

claude can call ssh and do system admin tasks. It works amazingly well. I have 3 VM's, which depends on each other (proxmox with openwrt, adguard, unbound), and claude can prove to me that my dns chains works perfectly, my firewalls are perfect etc as claude can ssh into each. Setting up services, diagnosing issues, auditing configs... you name it. Just awesome.

claude can call other sh scripts on the machine, so over time, you can create a bunch of scripts that lets claude one shot certain tasks that would normally eat tokens. It works great. One script per intention - don't have a script do more than one thing.

claude can call the compiler, run the debug executable and read the debug logs.. in real time. So claude can read my android apps debug stream via adb.. or my C# debug console because claude calls the compiler, not me. Just ask it to do it and it will diagnose stuff really quickly.

It can also analyze your db tables (give it readonly sql access), look at the application code and queries, and diagnose performance issues.

The opportunities are endless here. People need to wake up to this.

Claude set up a Raspberry Pi with a display and conference audio device for me to use as an Alexa replacement tied to Home Assistant.

I gave it an ssh key and gave it root.

Then I told it what I wanted, and it did. It asked for me to confirm certain things, like what I could see on screen, whether I could hear the TTS etc. (it was a bit of a surprise when it was suddenly talking to me while I was minding my own business).

It configured everything, while keeping a meticulous log that I can point it at if I want to set up another device, and eventually turn into a runbook if I need to.

I have a /fix-ci-build slash command that instructs Claude how to use `gh` to get the latest build from that specific project's Github Actions and get the logs for the build

In addition there are instructions on how and where to push the possible fixes and how to check the results.

I've yet to encounter a build failure it couldn't fix automatically.

It makes me so exhausted trying to read them... my brain can tell immediately when there's so much redundant information that it just starts shutting itself off.

I think we're entering a world where programmers as such won't really exist (except perhaps in certain niches). Being able to program (and read code, in particular) will probably remain useful, though diminished in value. What will matter more is your ability to actually create things, using whatever tools are necessary and available, and have them actually be useful. Which, in a way, is the same as it ever was. There's just less indirection involved now.

We've been living in that world since the invention of the compiler ("automatic programming"). Few people write machine code any more. If you think of LLMs as a new variety of compiler, a lot of their shortcomings are easier to describe.

You can run an LLM locally (and distributed compile systems, where the compiler runs in the cloud, are a thing, too) so that doesn't really produce a distinction between the two.

Likewise, many optimization techniques involve some randomness, whether it's approximating an NP-thorny subproblem, or using PGO guided by statistical sampling. People might disable those in pursuit of reproducible builds, but no one would claim that enabling those features makes GCC or LLVM no longer a compiler. So nondeterminism isn't really the distinguishing factor either.

If you think of the training data, e.g. SO, github etc, then you have a human asking or describing a problem, then the code as the solution. So I suspect current-gen LLMs are still following this model, which means for the forseeable future a human like language prompt will still be the best.

Until such time, of course, when LLMs are eating their own dogfood, in which case they - as has already happened - create their own language, evolve dramatically, and cue skynet.

More indirection in the sense that there's a layer between you and the code, sure. Less in that the code doesn't really matter as such and you're not having to think hard about the minutiae of programming in order to make something you want. It's very possible that "AI-oriented" programming languages will become the standard eventually (at least for new projects).

One benefit of conventional code is that it expresses logic in an unambiguous way. Much of "the minutiae" is deciding what happens in edge cases. It's even harder to express that in a human language than in computer languages. For some domains it probably doesn't matter.

> have it learn your conventions, pull in best practices

What do you mean by "have it learn your conventions"? Is there a way to somehow automatically extract your conventions and store it within CLAUDE.md?

> For example, we have a custom UI library, and Claude Code has a skill that explains exactly how to use it. Same for how we write Storybooks, how we structure APIs, and basically how we want everything done in our repo. So when it generates code, it already matches our patterns and standards out of the box.

Did you have to develop these skills yourself? How much work was that? Do you have public examples somewhere?

> What do you mean by "have it learn your conventions"?

I'll give you an example: I use ruff to format my python code, which has an opinionated way of formatting certain things. After an initial formatting, Opus 4.5, without prompting, will write code in this same style so that the ruff formatter almost never has anything to do on new commits. Sonnet 4.5 is actually pretty good at this too.

Isn't this a meaningless example? Formatters already exist. Generating code that doesn't need to be formatted is exactly the same as generating code and then formatting it.

I care about the norms in my codebase that can't be automatically enforced by machine. How is state managed? How are end-to-end tests written to minimize change detectors? When is it appropriate to log something?

We have some tests in "GIVEN WHEN THEN" style, and others in other styles. Opus will try to match each style of testing by the project it is in by reading adjacent tests.

The one caveat with this, is that in messy code bases it will perpetuate bad things, unless you're specific about what you want. Then again, human developers will often do the same and are much harder to force to follow new conventions.

But I think it should be doable. You can tell it how YOU want the state to be managed and then have it write a custom "linter" that makes the check deterministic. I haven't tried this myself, but claude did create some custom clippy scripts in rust when I wanted to enforce something that isn't automatically enforced by anything out there.

Lints are typically well suited for syntactic properties or some local semantic properties. Almost all interesting challenges in software design and evolution involve nonlocal semantic properties.

Starting to use Opus 4.5 I'm reduces instrutions in claude.md and just ask claude to look in the codebase to understand the patterns already in use. Going from prompts/docs to instead having code being the "truth". Show don't tell. I've found this patterns has made a huge leap with Opus 4.5.

"Model your application's behavior first, as data, and derive everything else automatically. Ash resources center around actions that represent domain logic."

I feel like I've been doing this since Sonnet 3.5 or Sonnet 4. I'll clone projects/modules/whatever into the working directory and tell claude to check it out. Voila, now it knows your standards and conventions.

When I ask Claude to do something, it independently, without me even asking or instructing it to, searches the codebase to understand what the convention is.

I’ve even found it searching node_modules to find the API of non-public libraries.

If they're using Opus then it'll be the $100/month Claude Max 5x plan (could be the more expensive 20x plan depending on how intensive their use is). It does consume a lot of tokens, but I've been using the $100/mo plan and get a lot done without hitting limits. It helps to be mindful of context (regularly amending/pruning your CLAUDE.md instructions, clearing context between tasks, sizing your tasks to stay within the Opus context window). Claude Code plans have token limits that work in 5-hour blocks (that start when you send your first token, so it's often useful to prime it as early in the morning as possible).

Claude Code will spawn sub-agents (that often use their cheap Haiki model) for exploration and planning tasks, with only the results imported into the main context.

I've found the best results from a more interactive collaboration with Claude Code. As long as you describe the problem clearly, it does a good job on small/moderate tasks. I generally set two instances of Claude Code separate tasks and run them concurrently (the interaction with Claude Code distracts me too much to do my own independent coding simultaneously like with setting a task for a colleague, but I do work on architecture / planning tasks)

The one manner of taste that I have had to compromise on is the sheer amount of code - it likes to write a lot of code. I have a better experience if I sweat the low-level code less, and just periodically have it clean up areas where I think it's written too much / too repetitive code.

As you give it more freedom it's more prone to failure (and can often get itself stuck in a fruitless spiral) - however as you use it more you get a sense of what it can do independently and what's likely to choke on. A codebase with good human-designed unit & playwright tests is very good.

Crucially, you get the best results where your tasks are complex but on the menial side of the spectrum - it can pay attention to a lot of details, but on the whole don't expect it to do great on senior-level tasks.

To give you an idea, in a little over a month "npx ccusage" shows that via my Claude Code 5x sub I've used 5M input tokens, 1.5M output, 121M Cache Create, 1.7B Cache Read. Estimated pay-as-you-go API cost equivalent is $1500 (N.B. for the tail end of December they doubled everybody's API limits, so I was using a lot more tokens on more experimental on-the-fly tool construction work)

FYI Opus is available and pretty usable in claude-code on the $20/Mo plan if you are at all judicious.

I exclusively use opus for architecture / speccing, and then mostly Sonnet and occasionally Haiku to write the code. If my usage has been light and the code isn't too straightforward, I'll have Opus write code as well.

The problem with current approaches is the lack of feedback loops with independent validators that never lose track of the acceptance criteria. That's the next level that will truly allow no-babysitting implementatons that are feature complete and production grade. Check out this repo that offers that: https://github.com/covibes/zeroshot/

That's helpful to know, thanks! I gave Max 5x a go and didn't look back. My suspicion is that Opus 4.5 is subsidised, so good to know there's flexibility if prices go up.

The $20 plan for CC is good enough for 10-20 minutes of opus every 5h and you’ll be out of your weekly limit after 4-5 days if you sleep during the night. I wouldn’t be surprised if Anthropic actually makes a profit here. (Yeah probably not, but they aren’t burning cash.)

I use the $200/month Claude Code plan, and in the last week I've had it generate about half a million words of documentation without hitting any session limits.

I have hit the weekly limit before, briefly, but that took running multiple sessions in parallel continuously for many days.

/init in Claude Code already automatically extracts a bunch, but for something more comprehensive, just tell it which additional types of things you want it to look for and document.

> Did you have to develop these skills yourself? How much work was that? Do you have public examples somewhere?

I don't know about the person above, but I tell Claude to write all my skills and agents for me. With some caveats, you can do this iteratively in a single session ("update the X agent, then re-run it. Repeat until it reliably does Y")

"Claude, clone this repo https://github.com/repo, review the coding conventions, check out any markdown or readme files. This is an example of coding conventions we want to use on this project"

lol does sound like and ad, but is true. Also forgot about hooks use hooks too! I just use voice to text then had claude reword it. Still my real world ideas

All of these things work very well IMO in a professional context.

Especially if you're in a place where a lot of time was spent previously revising PRs for best practices, etc, even for human-submitted code, then having the LLM do that for you that saves a bunch of time. Most humans are bad at following those super-well.

There's a lot of stuff where I'm pretty sure I'm up to at least 2x speed now. And for things like making CLI tools or bash scripts, 10x-20x. But in terms of "the overall output of my day job in total", probably more like 1.5x.

But I think we will need a couple major leaps in tooling - probably deterministic tooling, not LLM tooling - before anyone could responsibly ship code nobody has ever read in situations with millions of dollars on the line (which is different from vibe-coding something that ends up making millions - that's a low-risk-high-reward situation, where big bets on doing things fast make sense. if you're already making millions, dramatic changes like that can become high-risk-low-reward very quickly. In those companies, "I know that only touching these files is 99.99% likely to be completely safe for security-critical functionality" and similar "obvious" intuition makes up for the lack of ability to exhaustively test software in a practical way (even with fuzzers and things), and "i didn't even look at the code" is conceding responsibility to a dangerous degree there.)

> Once you’ve got Claude Code set up, you can point it at your codebase, have it learn your conventions, pull in best practices, and refine everything until it’s basically operating like a super-powered teammate. The real unlock is building a solid set of reusable “skills” plus a few agents for the stuff you do all the time.

I agree with this, but I haven't needed to use any advanced features to get good results. I think the simple approach gets you most of the benefits. Broadly, I just have markdown files in the repo written for a human dev audience that the agent can also use.

Basically:

- README.md with a quick start section for devs, descriptions of all build targets and tests, etc. Normal stuff.

- AGENTS.md (only file that's not written for people specifically) that just describes the overall directory structure and has a short step of instructions for the agent: (1) Always read the readme before you start. (2) Always read the relevant design docs before you start. (3) Always run the linter, a build, and tests whenever you make code changes.

- docs/*.md that contain design docs, architecture docs, and user stories, just text. It's important to have these resources anyway, agent or no.

As with human devs, the better the docs/requirements the better the results.

I'd really encourage you to try using agents for tasks that are repeatable and/or wordy but where most of the words are not relevant for ongoing understanding.

It's a tiny step further, and sub-agents provide a massive benefit the moment you're ready to trust the model even a little bit (relax permissions to not have it prompt you for every little thing; review before committing rather than on every file edit) because they limit what goes into the top level context, and can let the model work unassisted for far longer. I now regularly have it run for hours at a time without stopping.

Running and acting on output from the linter is absolutely an example of that which matters even for much shorter runs.

There's no reason to have all the lint output "polluting" the top level context, nor to have the steps the agent needs to take to fix linter issues that can't be auto-fixed by the linter itself. The top level agent should only need to care about whether the linter run passed or failed (and should know it needs to re-run and possibly investigate if it fails).

Just type /agents, select "Create new agent" and describe a task you often do, and then forget about it (or ask Claude to make changes to it for you)

That's a great point. There are a lot of things you can do to optimise things, and your suggestion is one of the lower hanging fruits.

I was trying to get across the point that today you can get a lot of benefit from minimal setup, even one that's vendor-agnostic. (The steps I outlined work for Codex out of the box, too.)

You're right to point out that the more you refine things, the more you'll get out of the tools. It used to be that you had to do a lot of refinements to start getting good results at all. Now, you can get a lot out of even a basic setup like I outlined, which is great for people who are new users -- or people who tried it before and weren't that impressed but are now giving it another try.

It will have to quintuple or more to make business sense for Anthropic. Sure, still cheaper than a full time developer, but don't expect it to stay at $200 for a long time. And then, when you explain to your boss how amazing it is, and can do all this work so easily and quickly, it's when your boss start asking the real question: what am I paying you for?

A programmer, if we use US standards is probably $8000 per month. If you can get 30% more value out of that programmer (trust me, its WAY more then 30%), you gained $2400 of value. If you pay $200, $500, $1000 for that, its still a net positive. Ignoring the salary range of a actual senior...

LLMs do not result in bosses firing people, it results in more projects / faster completed projects, what in turn means more $$$ for a company.

More fundamentally: assume a 10 to 30% bump in actual productivity, find a niche (editing software, CRUD frameworks, SharePoint 2.0, stock trading, betting, whatever), and assume you had Anthropics billions or openAIs billions or Microsoft’s billions or Googles billions.

Why on earth would you be hunting $20 a month subscriptions from random assed people? Peanuts.

Lockheed-Martin could be, but isn’t, opening lemonade stands outside their offices… they don’t because of how buying a Ferrari works.

> Why on earth would you be hunting $20 a month subscriptions from random assed people? Peanuts.

For the same reason Microsoft never has and never will chase people for pirating home Windows or Office licenses

When they hit the workforce, or even better, start a company guess which OS and office suite they'll use? Hint: It's not Linux and Openoffice.

Same with Claude's $20 package. It lets devs use it at home and then compare it to the Copilot shit their company is pushing on them. Maybe they either grumble enough to get a Claude license or they're in a position to make the call.

Cheap advertising pretty much.

Worked for me too :) I've paid my own Claude license for over a year at home, grumbled at work and we got a Claude pilot going now - and everyone who's tried it so far isn't going back to Copilot + Sonnet 4.5/GPT5.

Im not sure about this. What they really need is to get rid of the free tier and widespread adoption. Inference on the $200 plan seems to be profitable right now so they just need more users to amortize training costs.

Cheaper than hiring another developer, probably. My experience: for a few dollars I was able to extensively refactor a Python codebase in half a day. This otherwise would have taken multiple days of very tedious work.

And that's what the C-suite wants to know. Prepare yourself to be replaced in the not so distant future. Hope you have a good "nest" to support yourself when you're inevitably fired.

> Prepare yourself to be replaced in the not so distant future.

Ignoring that this same developer, now has access to a tool, that makes himself a team.

Going independent was always a issue because being a full stack dev, is hard. With LLMs, you have a entire team behind you for making graphics, code, documents, etc... YOU becomes the manager.

We will see probably a lot more smaller teams/single devs making bigger projects, until they grow.

The companies that think they can fire devs, are the same companies that are going to go too far, and burn bridges. Do not forget that a lot of companies are founded on devs leaving a company, and starting out on their own, taking clients with them!

I did that years ago, and it worked for a while but eventually the math does not work out because one guy can only do so much. And when you start hiring, your costs balloon. But with LLMs ... Now your a one man team, ... hiring a second person is not hiring a person to make some graphics or doing more coding. Your hiring another team.

This is what people do not realize... they look too much upon this as the established order, ignoring what those fired devs now can do!

This sounds nice, except for the fact that almost everyone else can do this, too. Or at least try to, resulting in a fast race to the bottom.

Do you really want to be a middle manager to a bunch of text boxes, churning out slop, while they drive up our power bills and slowly terraform the planet?

The same way that having motorized farming equipment was a race to the bottom for farmers? Perhaps. Turned out to be a good outcome for most involved.

Just like farmers who couldn't cope with the additional leverage their equipment provided them, devs who can't leverage this technology will have to "go to the cities".

Please do read up on how farmers are doing with this race to the bottom (it hasn't been pretty). Mega farms are a thing because small farms simply can't compete. Small farmers have gone broke. The parent comment is trying to highlight this.

If LLM's turn out the way C-Suite hopes. Let me tell you, you will be in a world of pain. Most of you won't be using LLM's to create your own businesses.

But modern tillage/petrol based farming is an unsustainable aberration. Maybe a good example for this discussion, but in the opposite direction if it is.

> except for the fact that almost everyone else can do this, too. Or at least try to, resulting in a fast race to the bottom.

Ironically, that race to the bottom is no different then we already have. Have you already worked for a company before? A lot of software is developed, BADLY. I dare to say that a lot of software that Opus 4.5 generates, is often a higher quality then what i have seen in my 25 year carrier.

The amount of companies that cheapen out, hiring juniors fresh from school, to work as coding monkies is insane. Then projects have bugs / security issues, with tons of copy/pasted code, or people not knowing a darn thing.

Is that any different then your feared future? I dare to say, that LLms like Opus are frankly better then most juniors. As a junior to do a code review for security issues. Opus literally creates extensive tests, points out issues that you expect from a mid or higher level dev. Of course, you need to know to ask! You are the manager.

> Do you really want to be a middle manager to a bunch of text boxes, churning out slop, while they drive up our power bills and slowly terraform the planet?

Frankly, yes ... If you are a real developer, do you still think development is fun after 10 years, 20 years? Doing the exact same boring work. Reimplementing the 1001 login page, the 101 contact form ... A ton of our work is in reality repeating the same crap over and over again. And if we try to bypass it, we end up tied to tied to those systems / frameworks that often become a block around our necks.

Our industry has a lot of burnout because most tasks may start small but then grow beyond our scope. Todays its ruby on rails programming, then its angular, no wait, react, no wait, Vue, no wait, the new hotness is whatever again.

> slowly terraform the planet?

Well, i am actually making something.

Can you say the same for all the power / gpu draw with bitcoin, Ethereum whatever crap mining. One is productive, a tool with insane potential and usage, the other is a virtual currency where only one is ever popular with limited usage. Yet, it burns just as much for a way more limited return of usability.

Those LLMs that you are so against, make me a ton more productive. You wan to to try out something, but never really wanted to get committed because it was weeks of programming. Well, now you as manager, can get projects done fast. Learn from them way faster then your little fingers ever did.

You say this like it's some kind of ominous revelation, but that's just how capitalism works? Yeah, prepare for the future. All things are impermanent.

I suppose as long as either humans are always able to use new tools to create new jobs, or the wealth gets shared in a fully automated society, it won't be ominous. There are other scenarios.

I think we might make new jobs, but maybe not enough. I'll be pleasantly surprised if we get good at sharing wealth over the next few years. Maybe something like UBI will become so obviously necessary that it becomes politically feasible, I don't know. I suspect we'll probably limp along for awhile in mediocrity. Then we'll die. Same as it ever was. The important thing is to have fun with it.

Well excuse the shit out of my goddamn French, but being comfy for years and suddenly facing literal doom of my profession in a year wasn't on my bingo card.

And what do you even mean by "prepare"? Shit out a couple of mil out of my ass and invest asap?

Not the person you're responding to but... if you think it's a horse -> car change (and, to stretch the metaphor, if you think you're in the business of building stables) then preparation means train in another profession.

If you think it's a hand tools -> power tools change, learn how to use the new tools so you don't get left behind.

My opinion is it's a hand -> power tools change, and that LLMs give me the power to solve more problems for clients, and do it faster and more predictably than a client trying to achieve the same with an LLM. I hope I'm right :-)

Why do you suppose that these tools will conveniently stop improving at some point that increases your productivity but are still too much for your clients to use for themselves?

And so the AI will develop the skills to interview the client and determine what they really need. There are textbooks written on how to do this, it's not going to be hard to incorporate into the training.

Well probably OP won't be affected because management is very pleased with him and his output, why would they fire him? Hire someone who can probably have better output than him for 10% more money or someone who might have the same output for 25% less pay?

You think any manager in their right mind would take risks like that?

I think the real consequences are that they probably are so pleased with how productive the team is becoming that they will not hire new people or fire the ones who aren't keeping up with the times.

It's like saying "wow, our factory just produced 50% more cars this year, time to shut down half the factory to reduce costs!"

> You think any manager in their right mind would take risks like that?

You really underestimate stupidity of your average manager. Two of our top performers left because they were underpaid and the manager (in charge of the comp) never even tried to retain them.

I bet they weren't as valuable as you think. This is a common issue with certain high performing line delivery employees (particularly those with technical skills, programmers, lawyers, accountants, etc), they always think they are carrying the whole team/company on their shoulders. It almost never turns out to be the case. The machine will keep grinding.

I still struggle with these things being _too_ good at generating code. They have a tendency to add abstractions, classes, wrappers, factories, builders to things that didn't really need all that. I find they spit out 6 files worth of code for something that really only needed 2-3 and I'm spending time going back through simplifying.

There are times those extra layers are worth it but it seems LLMs have a bias to add them prematurely and overcomplicate things. You then end up with extra complexity you didn't need.

They are sleeping on it because there is absolutely no incentive to use it.

When needed it can be picked up in a day. Otherwise they are not paid based in tickets solved etc.
If the incentives were properly aligned everyone would already use it

Use Claude Code... to do what? There are multiple layers of people involved in the decision process and they only come up with a few ideas every now and then. Nothing I can't handle. AI helps but it doesn't have to be an agent.

I'm not saying there aren't use cases for agents, just that it's normal that most software engineers are sleeping on it.

Came across official anthropic repo on gh actions very relevant to what you mentioned. Your idea on scheduled doc updation using llm is brilliant, I’m stealing this idea.
https://github.com/anthropics/claude-code-action

Also new haiku. Not as smart but lighting fast, I've it review code changes impact or if i need a wide but shallow change done I've it scan the files and create a change plan. Saves a lot of time waiting for claude or codex to get their bearing.

Never tried coderabbit, just because this is already good enough with Claude Code. It helped us to catch dozens of important issues we wouldn't have caught.
We gave some instructions in the CLAUDE.md doc in the repository - with including a nice personalized roast of the engineer that did the review in the intro and conclusion to make it fun! :)
Basically, when you do a "create PR" from your Claude Code, it will help you getting your Linear ticket (or creating one if missing), ask you some important questions (like: what tests have you done?), create the PR on Github, request the reviewers, and post a "Auto Review" message with your credentials. It's not an actual review per se but this is enough for our small team.

Thanks for the example! There's a lot (of boilerplate?) here that I don't understand. Does anyone have good references for catching up to speed what's the purpose of all of these files in the demo?

If anyone is excited about, and has experience with this kind of stuff, please DM. I have a role open for setting up these kinds of tools and workflows.

I've tried most of the CLI coding tools with the Claude models and I keep coming back to Claude Code. It hits a sweet spot of simple and capable, and right now I'd say it's the best from an "it just works" perspective.

In my experience the CLI tool is part of the secret sauce. I haven't tried switching models per each CLI tool though. I use claude exclusively at work and for personal projects I use claude, codex, gemini.

Claude Code seems to package a relatively smart prompt as well, as it seems to work better even with one-line prompts than alternatives that just invoke the API.

Key word: seems. It's impossible to do a proper qualitative analysis.

I'm at the point where I say fuck it, let them sleep.

The tech industry just went through an insane hiring craze and is now thinning out. This will help to separate the chaff from the wheat.

I don't know why any company would want to hire "tech" people who are terrified of tech and completely obstinate when it comes to utilizing it. All the people I see downplaying it take a half-assed approach at using it then disparage it when it's not completely perfect.

I started tinkering with LLMs in 2022. First use case, speak in natural english to the llm, give it a json structure, have it decipher the natural language and fill in that json structure (vacation planning app, so you talk to it about where/how you want to vacation and it creates the structured data in the app). Sometimes I'd use it for minor coding fixes (copy and paste a block into chatgpt, fix errors or maybe just ideation). This was all personal project stuff.

At my job we got LLM access in mid/late 2023. Not crazy useful, but still was helpful. We got claude code in 2024. These days I only have an IDE open so I can make quick changes (like bumping up a config parameter, changing a config bool, etc.). I almost write ZERO code now. I usually have 3+ claude code sessions open.

On my personal projects I'm using Gemini + codex primarily (since I have a google account and chatgpt $20/month account). When I get throttled on those I go to claude and pay per token. I'll often rip through new features, projects, ideas with one agent, then I have another agent come through and clean things up, look for code smells, etc. I don't allow the agents to have full unfettered control, but I'd say 70%+ of the time I just blindly accept their changes. If there are problems I can catch them on the MR/PR.

I agree about the low hanging fruit and I'm constantly shocked at the sheer amount of FUD around LLMs. I want to generalize, like I feel like it's just the mid/jr level devs that speak poorly about it, but there's definitely senior/staff level people I see (rarely, mind you) that also don't like LLMs.

I do feel like the online sentiment is slowly starting to change though. One thing I've noticed a lot of is that when it's an anonymous post it's more likely to downplay LLMs. But if I go on linkedin and look at actual good engineers I see them praising LLMs. Someone speaking about how powerful the LLMs are - working on sophisticated projects at startups or FAANG. Someone with FUD when it comes to LLM - web dev out of Alabama.

I could go on and on but I'm just ranting/venting a little. I guess I can end this by saying that in my professional/personal life 9/10 of the top level best engineers I know are jumping on LLMs any chance they get. Only 1/10 talks about AI slop or bullshit like that.

Not entirely disagreeing with your point but I think they've mostly been forced to pivot recently for their own sakes; they will never say it though. As much as they may seem eager the most public people tend to also be better at outside communication and knowing what they should say in public to enjoy more opportunities, remain employed or for the top engineers to still seem relevant in the face of the communities they are a part of. Its less about money and more about respect there I think.

The "sudden switch" since Opus 4.5 when many were saying just a few months ago "I enjoy actual coding" but now are praising LLM's isn't a one off occurrence. I do think underneath it is somewhat motivated by fear; not for the job however but for relevance. i.e. its in being relevant to discussions, tech talks, new opportunities, etc.

OK, I am gonna be the guy and put my skin in the game here. I kind of get the hype, but the experience with e.g. Claude Code (or Github Copilot previously and others as weel) has so far been pretty unreliable.

I have Django project with 50 kLOC and it is pretty capable of understanding the architecture, style of coding, naming of variables, functions etc. Sometimes it excels on tasks like "replicate this non-trivial functionality for this other model and update the UI appropriately" and leaves me stunned. Sometimes it solves for me tedious and labourous "replace this markdown editor with something modern, allowing fullscreen edits of content" and does annoying mistake that only visual control shows and is not capable to fix it after 5 prompts. I feel as I am becoming tester more than a developer and I do not like the shift. Especially when I do not like to tell someone he did an obvious mistake and should fix it - it seems I do not care if it is human or AI, I just do not like incompetence I guess.

Yesterday I had to add some parameters to very simple Falcon project and found out it has not been updated for several months and won't build due to some pip issues with pymssql. OK, this is really marginal sub-project so I said - let's migrate it to uv and let's not get hands dirty and let the Claude do it. He did splendidly but in the Dockerfile he missed the "COPY server.py /data/" while I asked him to change the path... Build failed, I updated the path myself and moved on.

And then you listen to very smart guys like Karpathy who rave about Tab, Tab, Tab, while not understanding the language or anything about the code they write. Am I getting this wrong?

I am really far far away from letting agents touch my infrastructure via SSH, access managed databases with full access privileges etc. and dread the day one of my silly customers asks me to give their agent permission to managed services. One might say the liability should then be shifted, but at the end of the day, humans will have to deal with the damage done.

My customer who uses all the codebase I am mentioning here asked me, if there is a way to provide "some AI" with item GTINs and let it generate photos, descriptions, etc. including metadata they handcrafted and extracted for years from various sources. While it looks like nice idea and for them the possibility of decreasing the staff count, I caught the feeling they do not care about the data quality anymore or do not understand the problems the are brining upon them due to errors nobody will catch until it is too late.

TL;DR: I am using Opus 4.5, it helps a lot, I have to keep being (very) cautious. Wake up call 2026? Rather like waking up from hallucination.

Everybody says how good Claude is and I go to my code base and I can't get it to correctly update one xaml file for me. It is quicker to make changes myself than to explain exactly what I need or learn how to do "prompt engineering".

Disclaimer: I don't have access to Claude Code. My employer has only granted me Claude Teams. Supposedly, they don't use my poopy code to train their models if I use my work email Claude so I am supposed to use that. If I'm not pasting code (asking general questions) into Claude, I believe I'm allowed to use whatever.

What's even the point of this comment if you self-admittedly don't have access to the flagship tool that everyone has been using to make these big bold coding claims?

I believe part of why Claude Code is so great because it has the chance to catch its own mistakes. It can run compilers, linters, browsers and check its own output. If it makes a mistake, it takes one or two extra iterations until it gets it right.

Opus 4.5 ate through my Copilot quota last month, and it's already halfway through it for this month. I've used it a lot, for really complex code.

And my conclusion is: it's still not as smart as a good human programmer. It frequently got stuck, went down wrong paths, ignored what I told it to do to do something wrong, or even repeat a previous mistake I had to correct.

Yet in other ways, it's unbelievably good. I can give it a directory full of code to analyze, and it can tell me it's an implementation of Kozo Sugiyama's dagre graph layout algorithm, and immediately identify the file with the error. That's unbelievably impressive. Unfortunately it can't fix the error. The error was one of the many errors it made during previous sessions.

So my verdict is that it's great for code analysis, and it's fantastic for injecting some book knowledge on complex topics into your programming, but it can't tackle those complex problems by itself.

Yesterday and today I was upgrading a bunch of unit tests because of a dependency upgrade, and while it was occasionally very helpful, it also regularly got stuck. I got a lot more done than usual in the same time, but I do wonder if it wasn't too much. Wasn't there an easier way to do this? I didn't look for it, because every step of the way, Opus's solution seemed obvious and easy, and I had no idea how deep a pit it was getting me into. I should have been more critical of the direction it was pointing to.

Copilot and many coding agents truncates the context window and uses dynamic summarization to keep costs low for them. That's how they are able to provide flat fee plans.

If you want the full capability, use the API and use something like opencode. You will find that a single PR can easily rack up 3 digits of consumption costs.

Gerring off of their plans and prompts is so worth it, I know from experience, I'm paying less and getting more so far, paying by token, heavy gemini-3-flash user, it's a really good model, this is the future (distillations into fast, good enough for 90% of tasks), not mega models like Claude. Those will still be created for distillations and the harder problems

Maybe not, then. I'm afraid I have no idea what those numbers mean, but it looks like Gemini and ChatGPT 4 can handle a much larger context than Opus, and Opus 4.5 is cheaper than older versions. Is that correct? Because I could be misinterpreting that table.

I don't know about GPT4 but the latest one (GPT 5.2) has 200k context window while Gemini has 1m, five times higher. You'll be wanting to stay within the first 100k on all of them to avoid hitting quotas very quickly though (either start a new task or compact when you reach that) so in practice there's no difference.

I've been cycling between a couple of $20 accounts to avoid running out of quota and the latest of all of them are great. I'd give GPT 5.2 codex the slight edge but not by a lot.

The latest Claude is about the same too but the limits on the $20 plan are too low for me to bother with.

The last week has made me realize how close these are to being commodities already. Even the CLI the agents are nearly the same bar some minor quirks (although I've hit more bugs in Gemini CLI but each time I can just save a checkpoint and restart).

The real differentiating factor right now is quota and cost.

> You'll be wanting to stay within the first 100k on all of them

I must admit I have no idea how to do that or what that even means. I get that bigger context window is better, but what does it mean exactly? How do you stay within that first 100k? 100k what exactly?

Attention based neural network architectures (on which the majority of LLMs are built) has a unit economic cost that scales (roughly) n^2 i.e. quadratic (for both memory and compute). In other words, the longer the context window, the more expensive it is for the upstream provider. That's one cost.

The second cost is that you have to resend the entire context every time you send a new message. So the context is basically (where a, b, and c are messages): first context: a, second context window: a->b, third context window: a->b->c. It's a mostly stateless (there are some short term caching mechanisms, YMMV based on provider, it's why "cached" messages, especially system prompts are cheaper) process from the point of view of the developer, the state i.e. context window string is managed by the end user application (in other words, the coding agent, the IDE, the ChatGPT UI client etc.)

The per token cost is an amortized (averaged) cost of memory+compute, the actual cost is mostly quadratic with respect to each marginal token. The longer the context window the more expensive things are.
Because of the above, AI agent providers (especially those that charge flat fee subscription plans) are incentivized to keep costs low by limiting the maximum context window size.

(And if you think about it carefully, your AI API costs are a quadratic cost curve projected into a linear line (flat fee per token, so the model hosting provider in some cases may make more profit if users send in shorter contexts, versus if they constantly saturate the window. YMMV of course, but it's a race to the bottom right now for LLM unit economics)

They do this by interrupting a task halfway through and generating a "summary" of the task progress, then they prompt the LLM again with a fresh prompt and the "summary" so far and the LLM will restart the task from where it left of. Of course text is a poor representation of the LLM's internal state but it's the best option so far for AI application to keep costs low.

Another thing to keep in mind is that LLMs have poorer performance the larger the input size. This is due to a variety of factors (mostly because you don't have enough training data to saturate the massive context window sizes I think).

There are a bunch of tests and benchmarks (commonly referred to as "needle in a haystack") to improve the LLM performance at large context window sizes, but it's still an open area of research.

The thing is, generally speaking, you will get a slightly better performance if you can squeeze all your code and problem into the context window, because the LLM can get a "whole picture" view of your codebase/problem, instead of a bunch of broken telephone summaries every dozen of thousands of tokens. Take this with a grain of salt as the field is changing rapidly so it might not be valid in a month or two.

Keep in mind that if the problem you are solving requires you to saturate the entire context window of the LLM, a single request can cost you dollars. And if you are using 1M+ context window model like gemini, you can rack up costs fairly rapidly.

Using Opus 4.5, I have noticed that in long sessions about a complex topic, there often comes a point when Opus starts spouting utter gibberish. One or two questions earlier it was making total sense, and suddenly it seems to have forgotten everything and responds in a way that barely relates to the question I asked, and certainly not to the "conversation" we were having.

Is that a sign of having having surpassed that context window size? I guess to keep them sharp, I should start a new session often and early.

From what I understand, a token is either a word or a character, so I can use 100k words or characters before I start running into limits. But I've got the feeling that the complexity of the problem itself also matters.

It could have exceeded either its real context window size (or the artificially truncated one) and the dynamic summarization step failed to capture the important bits of information you wanted. Alternatively, the information might be stored in certain places in the context window where it failed to perform well in needle in haystack retrieval.

This is part of the reason why people use external data stores (e.g. vector databases, graph tools like Bead etc. in the hope of supplementing the agent's native context window and task management tools).

The whole field is still in its infancy. Who knows, maybe in another update or two the problem might just be solved. It's not like needle in the haystack problems aren't differentiable (mathematically speaking).

People are completely missing the points about agentic development. The model is obviously a huge factor in the quality of the output, but the real magic lies in how the tools are managing and injecting context in to them, as well as the tooling. I switched from Copilot to Cursor at the end of 2025, and it was absolute night and day in terms of how the agents behaved.

Interesting you have this opinion yet you're using Cursor instead of Claude Code. By the same logic, you should get even better results directly using Anthropic's wrapper for their own model.

My employer doesn't allow for Claude Code yet. I'm fully aware from speaking to other peers, that they are getting even better performance out of Claude Code.

In my experience GPT-5 is also much more effective in the Cursor context than the Codex context. Cursor deserves props for doing something right under the hood.

yes just using AI for code analysis is way under appreciated I think. Even the most sceptical people on using it for coding should try it out as a tool for Q&A style code interrogation as well as generating documentation. I would say it zero-shots documentation generation better than most human efforts would to the point it begs the question of whether it's worth having the documentation in the first place. Obviously it can make mistakes but I would say they are below the threshold of human mistakes from what I've seen.

(I haven't used AI much, so feel free to ignore me.)

This is one thing I've tried using it for, and I've found this to be very, very tricky. At first glance, it seems unbelievably good. The comments read well, they seem correct, and they even include some very non-obvious information.

But almost every time I sit down and really think about a comment that includes any of that more complex analysis, I end up discarding it. Often, it's right but it's missing the point, in a way that will lead a reader astray. It's subtle and I really ought to dig up an example, but I'm unable to find the session I'm thinking about.

This was with ChatGPT 5, fwiw. It's totally possible that other models do better. (Or even newer ChatGPT; this was very early on in 5.)

Code review is similar. It comes up with clever chains of reasoning for why something is problematic, and initially convinces me. But when I dig into it, the review comment ends up not applying.

It could also be the specific codebase I'm using this on? (It's the SpiderMonkey source.)

I've had some encounters with inaccuracies but my general experience has been amazing. I've cloned completely foreign git repos, cranked up the tool and just said "I'm having this bug, give me an overview of how X and Y work" and it will create great high level conceptual outlines that mean I can drive straight in where without it I would spend a long time just flailing around.

I do think an essential skill is developing just the right level of scepticism. It's not really different to working with a human though. If a human tells me X or Y works in a certain way i always allow a small margin of possibility they are wrong.

They do have a knack for missing the point. Even Opus 4.5 can laser focus on the wrong thing. It does take skill and experience to interpret them correctly and set them straight when they go wrong.

Even so, for understanding what happens in a big chunk of code, they're pretty great.

>So my verdict is that it's great for code analysis, and it's fantastic for injecting some book knowledge on complex topics into your programming, but it can't tackle those complex problems by itself.

I don't think you've seen the full potential. I'm currently #1 on 5 different very complex computer engineering problems, and I can't even write a "hello world" in rust or cpp. You no longer need to know how to write code, you just need to understand the task at a high level and nudge the agents in the right direction. The game has changed.

All the naysayer here have clearly no idea. Your large matrix multiplication implementation is quite impressive! I have set up a benchmark loop and let GPT-5.1-Codex-Max experiment for a bit (not 5.2/Opus/Gemini, because they are broken in Copilot), but it seems to be missing something crucial. With a bit of encouragement, it has implemented:

- padding from 2000 to 2048 for easier power-of-two splitting
- two-level Winograd matrix multiplication with tiled matmul for last level
- unrolled AVX2 kernel for 64x64 submatrices
- 64 byte aligned memory
- restrict keyword for pointers
- better compiler flags (clang -Ofast -march=native -funroll-loops -std=c++17)

But yours is still easily 25 % faster. Would you be willing to write a bit about how you set up your evaluation and which tricks Claude used to solve it?

Thank you. Yeah, I'm doing all those things, which do get you close to the top. The rest of things I'm doing are mostly micro-optimizations such as finding a way to avoid AVX→SSE transition penalty (1-2% improvement).

But I don't want to spoil the fun. The agents are really good at searching the web now, so posting the tricks here is basically breaking the challenge.

How are you qualified to judge its performance on real code if you don't know how to write a hello world?

Yes, LLMs are very good at writing code, they are so good at writing code that they often generate reams of unmaintainable spaghetti.

When you submit to an informatics contest you don't have paying customers who depend on your code working every day. You can just throw away yesterday's code and start afresh.

Claude is very useful but it's not yet anywhere near as good as a human software developer. Like an excitable puppy it needs to be kept on a short leash.

I know what's like running a business, and building complex systems. That's not the point.

I used highload as an example because it seems like an objective rebuttal to the claim that "but it can't tackle those complex problems by itself."

And regarding this:

"Claude is very useful but it's not yet anywhere near as good as a human software developer. Like an excitable puppy it needs to be kept on a short leash"

Again, a combination of LLM/agents with some guidance (from someone with no prior experience in this type of high performing architecture) was able to beat all human software developers that have taken these challenges.

> Claude is very useful but it's not yet anywhere near as good as a human software developer. Like an excitable puppy it needs to be kept on a short leash.

The skill of "a human software developer" is in fact a very wide distribution, and your statement is true for a ever shrinking tail end of that

What I think people get wrong (especially non-coders) is that they believe the limitation of LLMs is to build a complex algorithm.
That issue in reality was fixed a long time ago. The real issue is to build a product. Think about microservices in different projects, using APIs that are not perfectly documented or whose documentation is massive, etc.

Honestly I don't know what commenters on hackernews are building, but a few months back I was hoping to use AI to build the interaction layer with Stripe to handle multiple products and delayed cancellations via subscription schedules. Everything is documented, the documentation is a bit scattered across pages, but the information is out there.
At the time there was Opus 4.1, so I used that. It wrote 1000 lines of non-functional code with 0 reusability after several prompts. I then asked something to Chat gpt to see if it was possible without using schedules, it told me yes (even if there is not) and when I told Claude to recode it, it started coding random stuff that doesn't exist.
I built everything to be functional and reusable myself, in approximately 300 lines of code.

The above is a software engineering problem. Reimplementing a JSON parser using Opus is not fun nor useful, so that should not be used as a metric

> The above is a software engineering problem. Reimplementing a JSON parser using Opus is not fun nor useful, so that should not be used as a metric.

I've also built a bitorrent implementation from the specs in rust where I'm keeping the binary under 1MB. It supports all active and accepted BEPs: https://www.bittorrent.org/beps/bep_0000.html

Again, I literally don't know how to write a hello world in rust.

I also vibe coded a trading system that is connected to 6 trading venues. This was a fun weekend project but it ended up making +20k of pure arbitrage with just 10k of working capital. I'm not sure this proves my point, because while I don't consider myself a programmer, I did use Python, a language that I'm somewhat familiar with.

So yeah, I get what you are saying, but I don't agree. I used highload as an example, because it is an objective way of showing that a combination of LLM/agents with some guidance (from someone with no prior experience in this type of high performing architecture) was able to beat all human software developers that have taken these challenges.

This hits the nail on the head. There's a marked difference between a JSON parser and a real world feature in a product. Real world features are complex because they have opaque dependencies, or ones that are unknown altogether. Creating a good solution requires building a mental model of the actual complex system you're working with, which an LLM can't do. A JSON parser is effectively a book problem with no dependencies.

You are looking at this wrong. Creating a json parser is trivial. The thing is that my one-shot attempt was 10x slower than my final solution.

Creating a parser for this challenge that is 10x more efficient than a simple approach does require deep understanding of what you are doing. It requires optimizing the hot loop (among other things) that 90-95% of software developers wouldn't know how to do. It requires deep understanding of the AVX2 architecture.

You need to give it search and tool calls and the ability to test its own code and iterate. I too could not oneshot an interaction layer with Stripe without tools. It also helps to make it research a plan beforehand.

If that is true; then all the commentary around software people having jobs still due to "taste" and other nice words is just that. Commentary. In the end the higher level stuff still needs someone to learn it (e.g. learning ASX2 architecture, knowing what tech to work with); but it requires IMO significantly less practice then coding which in itself was a gate. The skill morphs more into a tech expert rather than a coding expert.

I'm not sure what this means for the future of SWE's though yet. I don't see higher levels of staff in big large businesses bothering to do this, and at some scale I don't see founders still wanting to manage all of these agents, and processes (got better things to do at higher levels). But I do see the barrier of learning to code gone; meaning it probably becomes just like any other job.

None of the problems you've shown there are anything close to "very complex computer engineering problems", they're more like "toy problems with widely-known solutions given to students to help them practice for when they encounter actually complex problems".

I think you misunderstood, it's not about solving the problem, is about finding the most efficient solution. Give it a shot, and see if you can get to the top 10 on any task.

The point is these problems are well understood and solved, solving them well with AI doesn’t mean anything as you’re just reciting something that’s already been done already.

Security and approval is considered more important here. Just getting approval for neo4j on the clearest ever use case for it, took a year. I'm not going to spend my energy on getting approval for Claude Code.

I have no idea. Careless use, I guess. I was fixing a bunch of mocks in some once-great but now poorly maintained code, and I wasn't really feeling it so I just fed everything to Claude. Opus, unfortunately. I could easily have downgraded a bit.

If it can consistently verify that the error persists after fix--you can run (ok maybe you can't budget wise but theoretically) 10000 parallel instances of fixer agents then verify afterwards (this is in line with how the imo/ioi models work according to rumors)

What bothers me about posts like this is: mid-level engineers are not tasked with atomic, greenfield projects. If all an engineer did all day was build apps from scratch, with no expectation that others may come along and extend, build on top of, or depend on, then sure, Opus 4.5 could replace them. The hard thing about engineering is not "building a thing that works", its building it the right way, in an easily understood way, in a way that's easily extensible.

No doubt I could give Opus 4.5 "build be a XYZ app" and it will do well. But day to day, when I ask it "build me this feature" it uses strange abstractions, and often requires several attempts on my part to do it in the way I consider "right". Any non-technical person might read that and go "if it works it works" but any reasonable engineer will know that thats not enough.

Not necessarily responding to you directly, but I find this take to be interesting, and I see it every time an article like this makes the rounds.

Starting back in 2022/2023:

- (~2022) It can auto-complete one line, but it can't write a full function.

- (~2023) Ok, it can write a full function, but it can't write a full feature.

- (~2024) Ok, it can write a full feature, but it can't write a simple application.

- (~2025) Ok, it can write a simple application, but it can't create a full application that is actually a valuable product.

- (~2025+) Ok, it can write a full application that is actually a valuable product, but it can't create a long-lived complex codebase for a product that is extensible and scalable over the long term.

It's pretty clear to me where this is going. The only question is how long it takes to get there.

> It's pretty clear to me where this is going. The only question is how long it takes to get there.

I don't think its a guarantee. all of the things it can do from that list are greenfield, they just have increasing complexity. The problem comes because even in agentic mode, these models do not (and I would argue, can not) understand code or how it works, they just see patterns and generate a plausible sounding explanation or solution. agentic mode means they can try/fail/try/fail/try/fail until something works, but without understanding the code, especially of a large, complex, long-lived codebase, they can unwittingly break something without realising - just like an intern or newbie on the project, which is the most common analogy for LLMs, with good reason.

While I do agree with you. To play the counterpoint advocate though.

What if we get to the point where all software is basically created 'on the fly' as greenfield projects as needed? And you never need to have complex large long lived codebase?

It is probably incredibly wasteful, but ignoring that, could it work?

That sounds like an insane way to do anything that matters.

Sure, create a one-off app to post things to your Facebook page. But a one-off app for the OS it's running on? Freshly generating the code for your bank transaction rules? Generating an authorization service that gates access to your email?

The only reason it's quick to create green-field projects is because of all these complex, large, long-lived codebases that it's gluing together. There's ample training data out there for how to use the Firebase API, the Facebook API, OS calls, etc. Without those long-lived abstraction layers, you can't vibe out anything that matters.

In Japan buildings (apartments) aren't built to last forever. They are built with a specific age in mind. They acknowledge the fact that houses are depreciating assets which have a value lim->0.

The only reason we don't do that with code (or didn't use to do it) was because rewriting from scratch NEVER worked[0]. And large scale refactors take massive amounts of time and resources, so much so that there are whole books written about how to do it.

But today trivial to simple applications can be rewritten from spec or scratch in an afternoon with an LLM. And even pretty complex parsers can be ported provided that the tests are robust enough[1]. It's just a metter of time someone rewrites a small to medium size application from one language to another using the previous app as the "spec".

> But today trivial to simple applications can be rewritten from spec or scratch in an afternoon with an LLM. And even pretty complex parsers can be ported provided that the tests are robust enough[1]. It's just a metter of time someone rewrites a small to medium size application from one language to another using the previous app as the "spec".

This seems like a sort of I dunno chicken and the egg thing.

The _reason_ you don't rewrite code is because it's hard to know that you truly understand the spec. If you could perfectly understand the spec then you could rewrite the code, but then what is the software, is it the code or the spec that writes the code. So if you built code A from spec, rebuilding it from spec I don't think qualifies a rewrite, it's just a recompile. If you're trying to fundamentally build a new application from spec when an old application was written by hand, you're going to run into the same problems you have in a normal rewrite.

We already have an example of this. Typescript applications are basically rewritten every time that you recompile typescript to node. Typescript isn't the executed code, it's a spec.

edit: I think I missed that you said rewrite in a different language, then yeah fine, you're probably right, but I don't think most people are architecture agnostic when they talk about rewrites. The point of a rewrite is to keep the good stuff and lose a lot of bad stuff. If you're using the original app as a spec to rewrite in a new language, then fine yeah, LLM's may be able to do this relatively trivially.

I don't know about Japan - I vaguely recall reading that most buildings over there are built with wood (even the big ones) and that this is historically something to do with rebuilding after Tsunamis and earthquakes.

Buildings in most other countries in the world ARE built to last forever, and often renovated, changed, extended and modified long after the incept date until, because needs change, and destroying them to start over is complete overkill (Although some people do these "large scale refactors" - they're usually rich).

> It's just a metter of time someone rewrites a small to medium size application from one language to another using the previous app as the "spec".

I have no doubt of this. I'm sure it's happening already. But the whole point of long term stable applications is that they are tried and tested. A port done in an afternoon by an LLM might be great, but you can't know if it has problems until it has withstood the test of time.

Sure, and the buildings are built to a slowly-evolving code, using standard construction techniques, operating as a predictable building in a larger ecosystem.

The problem with "all software" being AI-generated is that, to use your analogy, the electrical standards, foundation, and building materials have all been recently vibe-coded into existence, and none of your construction workers are certified in any of it.

I don't think so. I don't think this is how human brains work, and you would have too many problems trying to balance things out. I'm thinking specifically like a complex distributed system. There are a lot of tweaks and iterations you need for things to work with eachother.

But then maybe this means what is a "codebase". If a code base is just a structured set of specs that compile to code ala typescript -> javascript. sure, but then, it's still a long-lived <blank>

But maybe you would have to elaborate on, what does "creating software on the fly" look like,. because I'm sure there's a definition where the answer is yes.

Even if Opus 4.5 is the limit it’s still a massively useful tool. I don’t believe it’s the limit though for the simple fact that a lot could be done by creating more specialized models for each subdomain i.e. they’ve focused mostly on web based development but could do the same for any other paradigm.

That's a massive shift in the claim though... I don't think anyone is disputing that it's a useful tool; just the implication that because it's a useful tool and has seen rapid improvement that implies they're going to "get all the way there," so to speak.

Personally I'm not against LLMs or AI itself, but considering how these models are built and trained, I personally refuse to use tools built on others' work without or against their consent (esp. GPL/LGPL/AGPL, Non Commercial / No Derivatives CC licenses and Source Available licenses).

Of course the tech will be useful and ethical if these problems are solved or decided to be solved the right way.

We just need to tax the hell out of the AI companies (assuming they are ever profitable) since all their gains are built on plundering the collective wisdom of humanity.

You won't find any trustworthy papers on the topic because GP is simply wrong here.

That models can be distilled has no bearing whatsoever on whether a model has learned actual knowledge or understanding ("logic"). Models have always learned sparse/approximately-sparse and/or redundant weights, but they are still all doing manifold-fitting.

The resulting embeddings from such fitting reflect semantics and semantic patterns. For LLMs trained on the internet, the semantic patterns learned are linguistic, which are not just strictly logical, but also reflect emotional, connotational, conventional, and frequent patterns, all of which can be illogical or just wrong. While linguistic semantic patterns are correlated with logical patterns in some cases, this is simply not true in general.

> Well, the first 90% is easy, the hard part is the second 90%.

You'd need to prove that this assertion applies here. I understand that you can't deduce the future gains rate from the past, but you also can't state this as universal truth.

No, I don't need to. Self driving cars is the most recent and biggest example sans LLMs. The saying I have quoted (which has different forms) is valid for programming, construction and even cooking. So it's a simple, well understood baseline.

Knowledge engineering has a notion called "covered/invisible knowledge" which points to the small things we do unknowingly but changes the whole outcome. None of the models (even AI in general) can capture this. We can say it's the essence of being human or the tribal knowledge which makes experienced worker who they are or makes mom's rice taste that good.

Considering these are highly individualized and unique behaviors, a model based on averaging everything can't capture this essence easily if it can ever without extensive fine-tuning for/with that particular person.

>> No, I don't need to. Self driving cars is the most recent and biggest example sans LLMs.

Self-driving cars don't use LLMs, so I don't know how any rational analysis can claim that the analogy is valid.

>> The saying I have quoted (which has different forms) is valid for programming, construction and even cooking. So it's a simple, well understood baseline.

Sure, but the question is not "how long does it take for LLMs to get to 100%". The question is, how long does it take for them to become as good as, or better than, humans. And that threshold happens way before 100%.

>> Self-driving cars don't use LLMs, so I don't know how any rational analysis can claim that the analogy is valid.

Doesn't matter, because if we're talking about AI models, no (type of) model reaches 100% linearly, or 100% ever. For example, recognition models run with probabilities. Like Tesla's Autopilot (TM), which loves to hit rolled-over vehicles because it has not seen enough vehicle underbodies to classify it.

Same for scientific classification models. They emit probabilities, not certain results.

>> Sure, but the question is not "how long does it take for LLMs to get to 100%"

I never claimed that a model needs to reach a proverbial 100%.

>> The question is, how long does it take for them to become as good as, or better than, humans.

They can be better than humans for certain tasks. They are actually better than humans in some tasks since 70s, but we like to disregard them to romanticize current improvements, but I don't believe current or any generation of AIs can be better than humans in anything and everything, at once.

Remember: No machine can construct something more complex than itself.

>> And that threshold happens way before 100%.

Yes, and I consider that "treshold" as "complete", if they can ever reach it for certain tasks, not "any" task.

Self driving cars is not a proof. It only proves that having quick gains doesn't mean necessarily you'll get a 100% fast. It doesn't prove it will necessarily happen.

A model trained on a very large corpus can't, because these behaviors are different or specialized enough they cancel each other most of the cases. You can forcefully fine-tune a model with a singular person's behavior up to a certain point, but I'm not sure that even that can capture the subtlest of behaviors or decision mechanisms which are generally the most important ones (the ones we call gut feeling or instinct).

OTOH, while I won't call human brain perfect, the things we label "shit" generally turn out to be very clever and useful optimizations to workaround its own limitations, so I regard human brain higher than most AI proponents do. Also we shouldn't forget that we don't know much about how that thing works. We only guess and try to model it.

Lastly, searching perfection in numbers and charts or in engineering sense is misunderstanding nature and doing a great disservice to it, but this is a subject for another day.

I read the comment more as "based on past experience, it is usually the case that the first 90% is easier than the last 10%", which is the right base case expectation, I think. That doesn't mean it will definitely play out that way, but you don't have to "prove" things like this. You can just say that they tend to be true, so it's a good expectation to think it will probably be true again.

I agree with your observation, but not your conclusion. The 20 times it failed basically don't matter -- they are branches that can just be thrown away, and all that was lost is a few dollars on tokens (ignoring the environmental impact, which is a different conversation).

As long as it can do the thing on a faster overall timeline and with less human attention than a human doing it fully manually, it's going to win. And it will only continue to get better.

And I don't know why people always jump to self-driving cars as the analogy as a negative. We already have self-driving cars. Try a Waymo if you're in a city that has them. Yes, there are still long-tail problems being solved there, and limitations. But they basically work and they're amazing. I feel similarly about agentic development, plus in most cases the failure modes of SWE agents don't involve sudden life and death, so they can be more readily worked around.

With "art" we're now at a situation where I can get 50 variations of a image prompt within seconds from an LLM.

Does it matter that 49 of them "failed"? It cost me fractions of a cent, so not really.

If every one of the 50 variants was drawn by a human and iterated over days, there would've been a major cost attached to every image and I most likely wouldn't have asked for 50 variations anyway.

It's the same with code. The agent can iterate over dozens of possible solutions in minutes or a few hours. Codex Web even has a 4x mode that gives you 4 alternate solutions to the same issue. Complete waste of time and money with humans, but with LLMs you can just do it.

Yeah maybe, but personally it feels more like a plateau to me than an exponential takeoff, at the moment.

And this isn't a pessimistic take! I love this period of time where the models themselves are unbelievably useful, and people are also focusing on the user experience of using those amazing models to do useful things. It's an exciting time!

But I'm still pretty skeptical of "these things are about to not require human operators in the loop at all!".

Linear progression feels slower (and thus more like a plateau) to me than the end of 2022 through end of 2024 period.

The question in my mind is where we are on the s-curve. Are we just now entering hyper-growth? Or are we starting to level out toward maturity?

It seems like it must still be hyper-growth, but it feels less that way to me than it did a year ago. I think in large part my sense is that there are two curves happening simultaneously, but at different rates. There is the growth in capabilities, and then there is the growth in adoption. I think it's the first curve that seems to be to have slown a bit. Model improvements seem both amazing and also less revolutionary to me than they did a year or two ago.

But the other curve is adoption, and I think that one is way further from maturity. The providers are focusing more on the tooling now that the models are good enough. I'm seeing "normies" (that is, non-programmers) starting to realize the power of Claude Code in their own workflows. I think that's gonna be huge and is just getting started.

I haven't seen an AI successfully write a full feature to an existing codebase without substantial help, I don't think we are there yet.

> The only question is how long it takes to get there.

This is the question and I would temper expectations with the fact that we are likely to hit diminishing returns from real gains in intelligence as task difficulty increases. Real world tasks probably fit into a complexity hierarchy similar to computational complexity. One of the reasons that the AI predictions made in the 1950s for the 1960s did not come to be was because we assumed problem difficulty scaled linearly. Double the computing speed, get twice as good at chess or get twice as good at planning an economy. P, NP separation planed these predictions. It is likely that current predictions will run into similar separations.

It is probably the case that if you made a human 10x as smart they would only be 1.25x more productive at software engineering. The reason we have 10x engineers is less about raw intelligence, they are not 10x more intelligent, rather they have more knowledge and wisdom.

Sure, eventually we'll have AGI, then no worries, but in the meantime you can only use the tools that exist today, and dreaming about what should be available in the future doesn't help.

I suspect that the timeline from autocomplete-one-line to autocomplete-one-app, which was basically a matter of scaling and RL, may in retrospect turn out to have been a lot faster that the next LLM to AGI step where it becomes capable of using human level judgement and reasoning, etc, to become a developer, not just a coding tool.

This is disingenuous because LLMs were already writing full, simple applications in 2023.[0]

They're definitely better now, but it's not like ChatGPT 3.5 couldn't write a full simple todo list app in 2023. There were a billion blog posts talking about that and how it meant the death of the software industry.

Plus I'd actually argue more of the improvements have come from tooling around the models rather than what's in the models themselves.

That's not at all what's being discussed in this article. We copy-pasted from SO before this. This article is talking about 99% fully autonomous coding with agents, not copy-pasting 400 times from a chat bot.

Hi, please re-read the parent comment again, which was claiming

> Starting back in 2022/2023:

> - (~2022) It can auto-complete one line, but it can't write a full function.

> - (~2023) Ok, it can write a full function, but it can't write a full feature.

This was a direct refutation, with evidence, that in 2023 people were not claiming that LLMs "can't write a full feature", because, as demonstrated, people were already building full applications with it at the time.

This obviously is not talking exclusively about agents, because agents did not exist in 2022.

I get your point, but I'll just say that I did not intend my comment to be interpreted so literally.

Also, just because SOMEONE planted a flag in 2023 saying that an LLM could build an app certainly does NOT mean that "people were not claiming that LLMs "can't write a full feature"". People in this very thread are still claiming LLMs can't write features. Opinions vary.

Ok, it can create a long-lived complex codebase for a product that is extensible and scalable over the long term, but it doesn't have cool tattoos and can't fancy a matcha

There are two types of right/wrong ways to build: the context specific right/wrong way to build something and an overly generalized engineer specific right/wrong way to build things.

I've worked on teams where multiple engineers argued about the "right" way to build something. I remember thinking that they had biases based on past experiences and assumptions about what mattered. It usually took an outsider to proactively remind them what actually mattered to the business case.

I remember cases where a team of engineers built something the "right" way but it turned out to be the wrong thing. (Well engineered thing no one ever used)

Sometimes hacking something together messily to confirm it's the right thing to be building is the right way. Then making sure it's secure, then finally paying down some technical debt to make it more maintainable and extensible.

Where I see real silly problems is when engineers over-engineer from the start before it's clear they are building the right thing, or when management never lets them clean up the code base to make it maintainable or extensible when it's clear it is the right thing.

There's always a balance/tension, but it's when things go too far one way or another that I see avoidable failures.

*I've worked on teams where multiple engineers argued about the "right" way to build something. I remember thinking that they had biases based on past experiences and assumptions about what mattered. It usually took an outsider to proactively remind them what actually mattered to the business case.*

Gosh I am so tired with that one - someone had a case that burned them in some previous project and now his life mission is to prevent that from happening ever again, and there would be no argument they will take.

Then you get like up to 10 engineers on typical team and team rotation and you end up with all kinds of "we have to do it right because we had to pull all nighter once, 5 years ago" baked in the system.

Not fun part is a lot of business/management people "expect" having perfect solution right away - there are some reasonable ones that understand you need some iteration.

No, extrapolating from one bad experience to universal approach does not make anyone senior.

There are situations where it applies and situation where it doesn't. Having the experience to see what applies in this new context is what senior (usually) means.

The people I admire most talk a lot more about "risk" than about "right vs. wrong". You can do that thing that caused that all-nighter 5 years ago, it isn't "wrong", but it is risky, and the person who pulled that all-nighter has useful information about that risk. It often makes sense to accept risks, but it's always good to be aware that you're doing so.

It's also important to consider the developers risk tolerance as well. It's all fine and dandy that the project manager is okay with the risk but what if none of the developers are? Or one senior dev is okay with it but the 3 who actually work the on-call queue are not?

I don't get paid extra for after hours incidents (usually we just trade time), so it's well within my purview on when to take on extra risk. Obviously, this is not ideal, but I don't make the on-call rules and my ability to change them is not a factor.

I don't think of this as a project manager's role, but an engineering manager's role. The engineers on the team (especially the senior engineers) should be identifying the risks, and the engineering managers should be deciding whether they are tolerable. That includes risks like "the oncall is awful and morale collapses and everyone quits".

It's certainly the case that there are managers who handle those risks poorly, but that's just bad management.

> ...multiple engineers argued about the "right" way to build something. I remember thinking that they had biases based on past experiences and assumptions about what mattered.

I usually resolve this by putting on the table the consequences and their impacts upon my team that I’m concerned about, and my proposed mitigation for those impacts. The mitigation always involves the other proposer’s team picking up the impact remediation. In writing. In the SOP’s. Calling out the design decision by day of the decision to jog memories and names of those present that wanted the design as the SME’s. Registered with the operations center. With automated monitoring and notification code we’re happy to offer.

Once people are asked to put accountable skin in the sustaining operations, we find out real fast who is taking into consideration the full spectrum end to end consequences of their decisions. And we find out the real tradeoffs people are making, and the externalities they’re hoping to unload or maybe don’t even perceive.

That's awesome, but I feel like half the time most people aren't in the position to add requirements so a lot of shenanigans still happens, especially in big corps

I am satisfied when someone tells us we cannot change requirements, to get their acknowledgement that what we bring up does extract a specific trade-off, and their reason for accepting the trade-off, then recording it into design and operational documentation. The moment many people recognize this trade-off will be explicitly documented with their and their team's accountability in detail, is when you surface genuine trade-offs made with the debt to pay off in the future in mind and in the meantime a rationale to grant a ton of leeway to the team burdened with the externality going forward, and trade-offs made without understanding their externalities upon other teams (which happens a tremendous amount in large organizations).

Most of the time, people are just very reasonably and understandably focusing tightly on their lane and honestly had no idea of the externalities of their conclusions and decisions, and I'm happy to have experienced all those times a rebalancing of the trade-offs that everyone can accept and is grateful to have documented to justify spending the story points upon cleaning up later instead of working on new features while the externality debt's unwanted impact keeps piling up.

In fewer than a handful of times, I run into people deliberately, consciously with malice aforethought of the full externalities making trade-offs for the sake of expediently shifting burdens of of them without first consulting with partner teams they want to shift the burdens onto, simply so they can fatten their promo packet sooner at the expense of making other teams look worse. Getting these trade-offs documented about half the time makes them back down to a more reasonable trade-off, about half the time they don't back down but your team is now protected by explicit documentation and caveats upon the externality your team now has to carry, and 100% of the time my team and I put a ring fence upon all future interactions with that personality for at least the remaining duration of my gig.

> I've worked on teams where multiple engineers argued about the "right" way to build something. I remember thinking that they had biases based on past experiences and assumptions about what mattered. It usually took an outsider to proactively remind them what actually mattered to the business case.

My first thought was that you probably also have different biases, priorities and/or taste. As always, this is probably very context-specific and requires judgement to know when something goes too far. It's difficult to know the "most correct" approach beforehand.

> Sometimes hacking something together messily to confirm it's the right thing to be building is the right way. Then making sure it's secure, then finally paying down some technical debt to make it more maintainable and extensible.

I agree that sometimes it is, but in other cases my experience has been that when something is done, works and is used by customers, it's very hard to argue about refactoring it. Management doesn't want to waste hours on it (who pays for it?) and doesn't want to risk breaking stuff (or changing APIs) when it works. It's all reasonable.

And when some time passes, the related intricacies, bigger picture and initially floated ideas fade from memory. Now other stuff may depend on the existing implementation. People get used to the way things are done. It gets harder and harder to refactor things.

Again, this probably depends a lot on a project and what kind of software we're talking about.

> There's always a balance/tension, but it's when things go too far one way or another that I see avoidable failures.

I think balance/tension describes it well and good results probably require input from different people and from different angles.

I know what you are talking about, but there is more to life than just product-market fit.

Hardly any of us are working on Postgres, Photoshop, blender, etc. but it's not just cope to wish we were.

It's good to think about the needs to business and the needs of society separately. Yes, the thing needs users, or no one is benefiting. But it also needs to do good for those users, and ultimately, at the highest caliber, craftsmanship starts to matter again.

There are legitimate reasons for the startup ecosystem to focus firstly and primarily on getting the users/customers. I'm not arguing against that. What I am arguing is why does the industry need to be dominated by startups in terms of the bulk of the products (not bulk of the users). It begs the question of how much societally-meaningful programming waiting to be done.

I'm hoping for a world where more end users code (vibe or otherwise) and the solve their own problems with their own software. I think that will make more a smaller, more elite software industry that is more focused on infrastructure than last-mile value capture. The question is how to fund the infrastructure. I don't know except for the most elite projects, which is not good enough for the industry (even this hypothetical smaller one) on the whole.

> I'm hoping for a world where more end users code (vibe or otherwise) and the solve their own problems with their own software. I think that will make more a smaller, more elite software industry that is more focused on infrastructure than last-mile value capture.

Yes! This is what I'm excited about as well. Though I'm genuinely ambivalent about what I want my role to be. Sometimes I'm excited about figuring out how I can work on the infrastructure side. That would be more similar to what I've done in my career thus far. But a lot of the time, I think that what I'd prefer would be to become one of those end users with my own domain-specific problems in some niche that I'm building my own software to help myself with. That sounds pretty great! But it might be a pretty unnatural or even painful change for a lot of us who have been focused for so long on building software tools for other people to use.

Users will not care about the quality of your code, or the backed architecture, or your perfectly strongly typed language.

They only care about their problems and treat their computers like an appliance. They don't care if it takes 10 seconds or 20 seconds.

They don't even care if it has ads, popups, and junk.
They are used to bloatware and will gladly open their wallets if the tool is helping them get by.

It's an unfortunately reality but there it is, software is about money and solving problems. Unless you are working on a mission critical system that affects people's health or financial data, none of those matter much.

I know the customer's couldn't care about the quality of the code they see. But the idea that they don't care about software being bad/laggy/bloated ever, because it "still solves problems", doesn't stand up to scrutiny as an immutable fact of the universe. Market conditions can change.

I'm banking on a future that if users feel they can (perhaps vibe) code their own solutions, they are far less likely to open their wallets for our bloatware solutions. Why pay exorbitant rents for shitty SaaS if you can make your own thing ad-free, exactly to your own mental spec?

I want the "computers are new, programmers are in short supply, customer is desperate" era we've had in my lifetime so far to come to a close.

> There are legitimate reasons for the startup ecosystem to focus firstly and primarily on getting the users/customers. I'm not arguing against that. What I am arguing is why does the industry need to be dominated by startups in terms of the bulk of the products (not bulk of the users). It begs the question of how much societally-meaningful programming waiting to be done.

You slipped in "societally-meaningful" and I don't know what it means and don't want to debate merits/demerits of socialism/capitalism.

However I think lots of software needs to be written because in my estimation with AI/LLM/ML it'll generate value.

And then you have lots of software that needs to rewritten as firms/technologies die and new firms/technologies are born.

I didn't mean to do some snide anticaptialism. Making new Postgreses and blenders is really hard. I don't think the startup ecosystem does a very good job, but I don't assume central planning would do a much better job either.

(The method I have the most confidence in is some sort of mixed system where there is non-profit, state-planned, and startup software development all at once.)

Markets are a tool, a means to the end. I think they're very good, I'm a big fan! But they are not an excuse not to think about the outcome we want.

I'm confident that the outcome I don't want is where most software developers are trying to find demand for their work, pivoting etc. it's very "pushing a string" or "cart before the horse". I want more "pull" where the users/benefiaries of software are better able to dictate or create themselves what they want, rather than being helpless until a pivoting engineer finds it for them.

Basically start-up culture has combined theories of exogenous growth from technology change, and a baseline assumption that most people are and will remain hopelessly computer illiterate, into an ideology that assumes the best software is always "surprising", a paradigm shift, etc.

Startups that make libraries/tools for other software developers are fortunately a good step in undermining these "the customer is an idiot and the product will be better than they expect" assumptions. That gives me hope we're reach a healthier mix of push and pull. Wild successes are always disruptive, but that shouldn't mean that the only success is wild, or trying to "act disruptive before wild success" ("manifest" paradigm shifts!) is always the best means to get there.

I've worked in various roles, and I'm one of those people who is not computer illiterate and likes to build solutions that meet local needs.

It's got a lot easier technically to do that in recent year, and MUCH easier with AI.

But institutionally and in terms of governance it's got a lot harder. Nobody wants home-brew software anymore. Doing data management and governance is complex enough and involves enough different people that it's really hard to generate the momentum to get projects off the ground.

I still think it's often the right solution and that successful orgs will go this route and retain people with the skills to make it happen. But the majority probably can't afford the time/complexity, and AI is only part of the balance that determines whether it's feasible.

Another thing that gets me with projects like this, there are already many examples of image converters, minesweeper clones etc that you can just fork on GitHub, the value of the LLM here is largely just stripping the copyright off

It’s kind of funny - there’s another thread up where a dev claimed a 20-50x speed up. To their credit they posted videos and links to the repo of their work.

And when you check the work, a large portion of it was hand rolling an ORM (via an LLM). Relatively solved problem that an LLM would excel at, but also not meaningfully moving the needle when you could use an existing library. And likely just creating more debt down the road.

Reminds me of a post I read a few days ago of someone crowing about an LLM writing for them an email format validator. They did not have the LLM code up an accompanying send-an-email-validation loop, and were blithely kept uninformed by the LLM of the scar tissue built up by experience in the industry on how curiously a deep rabbit hole email validation becomes.

If you’ve been around the block and are judicious how you use them, LLM’s are a really amazing productivity boost. For those without that judgement and taste, I’m seeing footguns proliferate and the LLM’s are not warning them when someone steps on the pressure plate that’s about to blow off their foot. I’m hopeful we will this year create better context window-based or recursive guardrails for the coding agents to solve for this.

Yeah I love working with Claude Code, I agree that the new models are amazing, but I spend a decent amount of time saying "wait, why are we writing that from scratch, haven't we written a library for that, or don't we have examples of using a third party library for it?".

There is probably some effective way to put this direction into the claude.md, but so far it still seems to do unnecessary reimplementation quite a lot.

This is a typical problem you see in autodidacts. They will recreate solutions to solved problems, trip over issues that could have been avoided, and generally do all of things you would expect someone to do if they are working with skill but no experience.

LLMs accelerate this and make it more visible, but they are not the cause. It is almost always a person trying to solve a problem and just not knowing what they don't know because they are learning as they go.

I am hopeful autodidacts will leverage an LLM world like they did with an Internet search world from a library world from a printed word world. Each stage in that progression compressed the time it took for them to encompass a span of comprehension of a new body of understanding before applying to practice, expanded how much they applied the new understanding to, and deepened their adoption scope of best practices instead of reinventing the wheel.

In this regard, I see LLM's as a way for us to way more efficiently encode, compress, convey and enable operational practice our combined learned experiences. What will be really exciting is watching what happens as LLM's simultaneously draw from and contribute to those learned experiences as we do; we don't need full AGI to sharply realize massive benefits from just rapidly, recursively enabling a new highly dynamic form of our knowledge sphere that drastically shortens the distance from knowledge to deeply-nuanced praxis.

With the right prompt the LLM will solve it in the first place. But this is an issue of not knowing what you don't know, so it makes it difficult to write the right prompt. One way around this is to spawn more agents with specific tasks, or to have an agent that is ONLY focused on finding patterns/code where you're reinventing the wheel.

I often have one agent/prompt where I build things but then I have another agent/prompt where their only job is to find codesmells, bad patterns, outdated libraries, and make issues or fix these problems.

1. LLMs can't watch over someone and warn them when they are about to make a mistake

2. LLMs are obsequious

3. Even if LLMs have access to a lot of knowledge they are very bad at contextualizing it and applying it practically

I'm sure you can think of many other reasons as well.

People who are driven to learn new things and to do things are going to use whatever is available to them in order to do it. They are going to get into trouble doing that more often than not, but they aren't going to stop. No is helping the situation by sneering at them -- they are used it to it, anyway.

lol, I probably don't have any, actually. If I recall, I would just write comments when my question differed slightly from one already there.

But it's definitely the case that being able to go back and forth quickly with an LLM digging into my exact context, rather than dealing with the kind of judgy humorless attitude that was dominant on SO is hugely refreshing and way more productive!

I've hand-rolled my own ultra-light ORM because the off-the-shelf ones always do 100 things you don't need.*

And of course the open source ones get abandoned pretty regularly. Type ORM, which a 3rd party vendor used on an app we farmed out to them, mutates/garbles your input array on a multi-line insert. That was a fun one to debug. The issue has been open forever and no one cares. https://github.com/typeorm/typeorm/issues/9058

So yeah, if I ever need an ORM again, I'm probably rolling my own.

*(I know you weren't complaining about the idea of rolling your own ORM, I just wanted to vent about Type ORM. Thanks for listening.)

This is the thing that will be changing the open source and small/medium SaaS world a lot.

Why use a 3rd party dependency that might have features you don't need when you can write a hyper-specific solution in a day with an LLM and then you control the full codebase.

Or why pay €€€ for a SaaS every month when you can replicate the relevant bits yourself?

It seems to me these days, any code I want to write tries to solve problems that LLMs already excel at. Thankfully my job is perhaps just 10% about coding, and I hope people like you still have some coding tasks that cannot be easily solved by LLMs.

We should not exeggarate the capabilities of LLMs, sure, but let's also not play "don't look up".

"And likely just creating more debt down the road"

In the most inflationary era of capabilities we've seen yet, it could be the right move. What's debt when in a matter of months you'll be able to clear it in one shot?

I think I would prefer the former if I were reviewing a CV. It at least tells me they understood the code well enough to know where to make their minor tweaks. (I've spent hours reading through a repo to know where to insert/comment out a line to suit my needs.) The second tells me nothing.

Its odd you don't apply the same analysis to each. The latter certainly can provide a similar trail indicating knowledge of the use case and necessary parameters to achieve it. And certainly the former doesnt preclude llm interlocking.

It would help if I had a better understanding of what you mean by "that".

I generally write to liberate my consciousness from isolation. When doing so in a public forum I am generally doing so in response to an assertion. When responding to an assertion I am generally attempting to understand the framing which produced the assertion.

I suppose you may also be speaking to the voice which is emergent. I am not very well read, so you may find my style unconventional or sloppy. I generally try not to labor too much in this regard and hope this will develop as I continue to write.

Unfortunately, it is happening. I remember an old post on HNs, it mentioned that a "prompt engineer for article generating" can find more jobs than a columnist writer. And op just wrote articles by himself but declared that all artices were generated by AI.

I'd quickly trash your application if I see you just vibe coded some bullshit app.
Developing is about working smart, and its not smart to ask AI to code stuff that already exists, its in fact wasteful.

Have you ever tried to find software for a specific need? I usually spend hours investigating anything I can find only to discover that all options are bad in one way or another and cover my use case partially at best. It's dreadful, unrewarding work that I always fear. Being able to spent those hours to develop custom solution that has exactly what I need, no more, no less, that I can evolve further as my requirements evolve, all that while enjoying myself, is a godsend.

Anecdata but I’ve found Claude code with Opus 4.5 able to do many of my real tickets in real mid and large codebases at a large public startup. I’m at senior level (15+ years). It can browse and figure out the existing patterns better than some engineers on my team. It used a few rare features in the codebase that even I had forgotten about and was about to duplicate. To me it feels like a real step change from the previous models I’ve used which I found at best useless. It’s following style guides and existing patterns well, not just greenfield. Kind of impressive, kind of scary

Same anecdote for me (except I'm +/- 40 years experience). I consider my self a pretty good dev for non-web dev (GPU's, assembly, optimisation,...) and my conclusion is the same as you: impressive and scary. If the somehow the idea of what you want to do is on the web in text or in code, then Claude most likely has it. And its ability to understand my own codebases is just crazy (at my age, memory is declining and having Claude to help is just waow). Of course it fails some times, of course it need direction, but the thing it produces is really good.

Scary is that the LLM might have been trained on the entire open source code ever produced - which is far beyond human comprehension - and with ever growing capability (bigger context window, more training) my gut feeling is that, it would exceed human capability in programming pretty soon. Considering 2025 was the ground breaking year for agents, can't stop imagine what would happen when it iterates in the next couple of years. I think it would evolve to be like Chess playing engines that consistently beat top Chess players in the world!

I'm seeing this as well. Not huge codebases but not tiny - 4 year old startup. I'm new there and it would have been impossible for me to deliver any value this soon.
12 years experience; this thing is definitely amazing. Combined with a human it can be phenomenal. It also helped me tons with lots of external tools, understand what data/marketing teams are doing and even providing pretty crucial insights to our leadership that Gemini have noticed.
I wouldn't try to completely automate the humans out of the loop though just yet, but this tech for sure is gonna downsize team numbers (and at the same time - allow many new startups to come to life with little capital that eventually might grow and hire people. So unclear how this is gonna affect jobs.)

I've also found it to keep such a constrained context window (on large codebases), that it writes a secondary block of code that already had a solution in a different area of the same file.

Nothing I do seems to fix that in its initial code writing steps. Only after it finishes, when I've asked it to go back and rewrite the changes, this time making only 2 or 3 lines of code, does it magically (or finally) find the other implementation and reuse it.

It's freakin incredible at tracing through code and figuring it out. I <3 Opus. However, it's still quite far from any kind of set-and-forget-it.

Same exist in humans also, I worked with a developer who had 15 year experience and was tech lead in a big Indian firm, We started something together, 3 months back when I checked the Tables I was shocked to see how he fucked up and messed the DB. Finally the only option left with me was to quit because i know it will break in production and if i onboarded a single customer my life would be screwed. He mixed many things with frontend and offloaded even permissions to frontend, and literally copied tables in multiple DB (We had 3 services). I still cannot believe how he worked as a tch lead for 15 years. each DB had more than 100 tables and out of that 20-25 were duplicates. He never shared code with me, but I smelled something fishy when bug fixing was never ending loop and my front end guy told me he cannot do it anymore. Only mistake I did was I trusted him and worst part is he is my cousin and the relation became sour after i confronted him and decided to quit.

This sounds like a culture issue in the development process, I have seen this prevented many times. Sure I did have to roll back a feature I did not sign off just before new years. So as you say it happens.

yes, it was my mistake. I trusted him because he was my childhood friend and my cousin. He was a tech lead in CMMI Level 5 (serving fortune 500 firms) company at the time he joined with me. I had the trust that he will never ran away with the code and that trust is still there, also the entire feature, roadmap and vision was with me, so I thought code doesn't matter. It was a big learning for me.

I can, but I switched to something more challenging. I handed over all things to him and told, Iam no more interested. I don't want him to feel that i cheated him by creating something he worked on.

- How quickly is cost of refactor to a new pattern with functional parity going down?

- How does that change the calculus around tech debt?

If engineering uses 3 different abstractions in inconsistent ways that leak implementation details across components and duplicate functionality in ways that are very hard to reason about, that is, in conventional terms, an existential problem that might kill the entire business, as all dev time will end up consumed by bug fixes and dealing with pointless complexity, velocity will fall to nothing, and the company will stop being able to iterate.

But if claude can reliably reorganize code, fix patterns, and write working migrations for state when prompted to do so, it seems like the entire way to reason about tech debt has changed. And it has changed more if you are willing to bet that models within a year will be much better at such tasks.

And in my experience, claude is imperfect at refactors and still requires review and a lot of steering, but it's one of the things it's better at, because it has clear requirements and testing workflows already built to work with around the existing behavior. Refactoring is definitely a hell of a lot faster than it used to be, at least on the few I've dealt with recently.

In my mind it might be kind of like thinking about financial debt in a world with high inflation, in that the debt seems like it might get cheaper over time rather than more expensive.

> But if claude can reliably reorganize code, fix patterns, and write working migrations for state when prompted to do so, it seems like the entire way to reason about tech debt has changed.

Yup, I recently spent 4 days using Claude to clean up a tool that's been in production for over 7 years. (There's only about 3 months of engineering time spent on it in those years.)

We've known what the tool needed for many years, but ugh, the actual work was fairly messy and it was never a priority. I reviewed all of Opus's cleanup work carefully and I'm quite content with the result. Maybe even "enthusiastic" would be accurate.

So even if Claude can't clean up all the tech debt in a totally unsupervised fashion, it can still help address some kinds of tech debt extremely rapidly.

Good point. Most of the cost in dealing with tech debt is reading the code and noting the issues. I found that Claude can produce much better code when it has a functionally correct reference implementation. Also it's not needed to very specifically point out issues. I once mentioned "I see duplicate keys in X and Y, rework it to reduce repetition and verbosity". It came up with a much more elegant way to implement it.

So maybe doing 2-3 stages makes sense. First stage needs to be functionallty correct, but you accept code smells such as leaky abstractions, verbosity and repetition. In stage 2 and 3 you eliminate all this. You could integrate this all into the initial specification; you won't even see the smelly intermediate code; it only exists as a stepping stone for the model to iteratively refine the code!

> The hard thing about engineering is not "building a thing that works", its building it the right way, in an easily understood way, in a way that's easily extensible.

You’re talking like in the year 2026 we’re still writing code for future humans to understand and improve.

I fear we are not doing that. Right now, Opus 4.5 is writing code that later Opus 5.0 will refactor and extend. And so on.

For one, there are objectively detrimental ways to organize code: tight coupling, lots of mutable shared state, etc. No matter who or what reads or writes the code, such code is more error-prone, and more brittle to handle.

Then, abstractions are tools to lower the cognitive load. Good abstractions reduce the total amount of code written, allow to reason about the code in terms of these abstractions, and do not leak in the area of their applicability. Say Sequence, or Future, or, well, function are examples of good abstractions. No matter what kind of cognitive process handles the code, it benefits from having to keep a smaller amount of context per task.

"Code structure does not matter, LLMs will handle it" sounds a bit like "Computer architectures don't matter, the Turing Machine is proved to be able to handle anything computable at all". No, these things matter if you care about resource consumption (aka cost) at the very least.

> For one, there are objectively detrimental ways to organize code: tight coupling, lots of mutable shared state, etc. No matter who or what reads or writes the code, such code is more error-prone, and more brittle to handle.

Guess what, AIs don't like that as well because it makes harder for them to achieve the goal. So with minimal guidance, which at this point could probably be provided by AI as well, the output of AI agent is not that.

Yes LLMs aren't very good at architecture. I suspect because the average project online has pretty bad architecture. The training set is poisoned.

It's kind of bittersweet for me because I was dreaming of becoming a software architect when I graduated university and the role started disappearing so I never actually became one!

But the upside of this is that now LLMs suck at software architecture... Maybe companies will bring back the software architect role?

The training set has been totally poisoned from the architecture PoV. I don't think LLMs (as they are) will be able to learn software architecture now because the more time passes, the more poorly architected slop gets added online and finds its way into the training set.

Good software architecture tends to be additive, as opposed to subtractive. You start with a clean slate then build up from there.

It's almost impossible to start with a complete mess of spaghetti code and end up with a clean architecture... Spaghetti code abstractions tend to mislead you and lead you astray... It's like; understanding spaghetti code tends to soil your understanding of the problem domain. You start to think of everything in terms of terrible leaky abstraction and can't think of the problem clearly.

It's hard even for humans to look at a problem through fresh eyes; it's likely even harder for LLMs to do it. For example, if you use a word in a prompt, the LLM tends to try to incorporate that word into the solution... So if the AI sees a bunch of leaky abstractions in the code; it will tend to try to work with them as opposed to removing them and finding better abstractions. I see this all the time with hacks; if the code is full of hacks, then an LLM tends to produce hacks all the time and it's almost impossible to make it address root causes... Also hacks tend to beget more hacks.

> I don’t see a world in which our tools (LLMs or otherwise) don’t learn this.

I agree, but maybe for different reasons. Refactoring well is a form of intelligence, and I don't see any upper limit to machine intelligence other than the laws of physics.

> Refactoring is a very mechanistic way of turning bad code into good.

There are some refactoring rules of thumb that can seem mechanistic (by which I mean deterministic based on pretty simple rules), but not all. Neither is refactoring guaranteed to be sufficient to lead to all reasonable definitions of "good software". Sometimes the bar requires breaking compatibility with the previous API / UX. This is why I agree with the sibling comment which draws a distinction between refactoring (changing internal details without changing the outward behavior, typically at a local/granular scale) and reworking (fixing structural problems that go beyond local/incremental improvements).

Claude phrased it this way – "Refactoring operates within a fixed contract. Reworking may change the contract." – which I find to be nice and succinct.

Refactorings can be useful in certain cases if the core architecture of the system is sound, but for some very complex systems, the problems can run deeper and a refactoring can be a waste of time. Sometimes you're better off reworking the whole thing because the problem might be in the foundation itself; something about the architecture forces developer's hand in terms of thinking about the problem incorrectly and writing bad code on top.

Opus 4.5 is writing code that Opus 5.0 will refactor and extend. And Opus 5.5 will take that code and rewrite it in C from the ground up. And Opus 6.0 will take that code and make it assembly. And Opus 7.0 will design its own CPU. And Opus 8.0 will make a factory for its own CPUs. And Opus 9.0 will populate mars. And Opus 10.0 will be able to achieve AGI. And Opus 11.0 will find God. And Opus 12.0 will make us a time machine. And so on.

Objectively, we are talking about systems that have gone from being cute toys to outmatching most juniors using only rigid and slow batch training cycles.

As soon as models have persistent memory for their own try/fail/succeed attempts, and can directly modify what's currently called their training data in real time, they're going to develop very, very quickly.

We may even be underestimating how quickly this will happen.

We're also underestimating how much more powerful they become if you give them analysis and documentation tasks referencing high quality software design principles before giving them code to write.

This is very much 1.0 tech. It's already scary smart compared to the median industry skill level.

The 2.0 version is going to be something else entirely.

Honestly the scary part is that we don’t really even need one more Opus. If all we had for the rest of our lives was Opus 4.5, the software engineering world would still radically change.

I also love how AI enthusiasts just ignore the issue of exhausted training data... You cant just magically create more training data. Also synthetic training data reduces the quality of models.

Youre mixing up several concepts. Synthetic data works for coding because coding is a verifiable domain. You train via reinforcement learning to reward code generation behavior that passes detailed specs and meets other deseridata. It’s literally how things are done today and how progress gets made.

Would you please stop posting cynical, dismissive comments? From a brief scroll through https://news.ycombinator.com/comments?id=zwnow, it seems like your account has been doing nothing else, regardless of the topic that it's commenting on. This is not what HN is for, and destroys what it is for.

If you keep this up, we're going to have to ban you, not because of your views on any particular topic but because you're going entirely against the intended spirit of the site by posting this way. There's plenty of room to express your views substantively and thoughtfully, but we don't want cynical flamebait and denunciation. HN needs a good deal less of this.

Then ban me u loser, as I wrote HN is full of pretentious bullshitters. But its good that u wanna ban authentic views. Way to go. If i feel like it I'll just create a new account:-)

But that doesn't really matter and it shows how confused people really are about how a coding agent like Claude or OSS models are actually created -- the system can learn on its own without simply mimicking existing codebases even though scraped/licensed/commissioned code traces are part of the training cycle.

Training looks like:

- Pretraining (all data, non-code, etc, include everything including garbage)

- Supervised Fine Tuning (SFT) -- these are things like curated prompt + patch pairs, curated Q/A (like stack overflow, people are often cynical that this is done unethically but all of the major players are in fact very risk adverse and will simply license and ensure they have legal rights),

- Then more SFT for tool use -- actual curated agentic and human traces that are verified to be correct or at least produce the correct output.

- Then synthetic generation / improvement loops -- where you generate a bunch of data and filter the generations that pass unit tests and other spec requirements, followed by RL using verifiable rewards + possibly preference data to shape the vibes

- Then additional steps for e.g. safety, etc

So synthetic data is not a problem and is actually what explains the success coding models are having and why people are so focused on them and why "we're running out of data" is just a misunderstanding of how things work. It's why you don't see the same amount of focus on other areas (e.g. creative writing, art etc) that don't have verifiable rewards.

The

Agent --> Synthetic data --> filtering --> new agent --> better synthetic data --> filtering --> even better agent

flywheel is what you're seeing today so we definitely don't have any reason to suspect there is some sort of limit to this because there is in principle infinite data

You are thankfully wrong. I watch lots of talks on the topic from actual experts. New models are just old models with more tooling. Training data is exhausted and its a real issue.

Well, my experts disagree with your experts :). Sure, the supply of available fresh data is running out, but at the same time, there's way more data than needed. Most of it is low-quality noise anyway. New models aren't just old models with more tooling - the entire training pipeline has been evolving, as researchers and model vendors focus on making better use of data they have, and refining training datasets themselves.

There are more stages to LLM training than just the pre-training stage :).

Not saying it's not a problem, I actually don't know, but new CPU's are just old models with more improvements/tooling. Same with TV's. And cars. And clothes. Everything is. That's how improving things works. Running out of raw data doesn't mean running out of room for improvement. The data has been the same for the last 20 years, AI isn't new, things keep improving anyways.

Well from cars or CPUs its not expected for them to eventually reach AGI, they also don't eat a trillion dollar hole into us peasants pockets.
Sure, improvements can be made. But on a fundamental level, agents/LLMs can not reason (even though they love to act like they can). They are parrots learning words, these parrots wont ever invent new words once the list of words is exhausted though.

That's been my main argument for why LLMs might be at their zenith. But I recently started wondering whether all those codebases we expose to them are maybe good enough training data for the next generation. It's not high quality like accepted stackoverflow answers but it's working software for the most part.

If they'd be good enough you could rent them to put together closed source stuff you can hide behind a paywall, or maybe the AI owners would also own the paywall and rent you the software instead. The second that that is possible it will happen.

Up until now, no business has been built on tools and technology that no one understands. I expect that will continue.

Given that, I expect that, even if AI is writing all of the code, we will still need people around who understand it.

If AI can create and operate your entire business, your moat is nil. So, you not hiring software engineers does not matter, because you do not have a business.

> Does the corner bakery need a moat to be a business?

Yes, actually. Its hard to open a competing bakery due to location availability, permitting, capex, and the difficulty of converting customers.

To add to that, food establishments generally exist on next to no margin, due to competition, despite all of that working in their favor.

Now imagine what the competitive landscape for that bakery would look like if all of that friction for new competitors disappeared. Margin would tend toward zero.

> Now imagine what the competitive landscape for that bakery would look like if all of that friction for new competitors disappeared. Margin would tend toward zero.

This is the goal. It's the point of having a free market.

'BobbyJo didn't say "no margins", they said "margins would tend toward zero". Believe it or not, that is, and always has been, the entire point of competition in a free market system. Competitive pressure pushes margins towards zero, which makes prices approach the actual costs of manufacturing/delivery, which is the main social benefit of the entire idea in the first place.

High margins are transient aberrations, indicative of a market that's either rapidly evolving, or having some external factors preventing competition. Persisting external barriers to competition tend to be eventually regulated away.

The point of competition is efficiency, of which, margin is only a component. Most successful businesses have relatively high margins (which is why we call them successful) because they achieve efficiency in other ways.

I wouldn't call high margins transient aberrations. There are tons of businesses that have been around for decades with high margins.

With no margins, no employees, and something that has potential to turn into a cornucopia machine - starting with software, but potentially general enough to be used for real-world world when combined with robotics - who needs money at all?

Or people?

Billionaires don't. They're literally gambling on getting rid of the rest of us.

Elon's going to get such a surprise when he gets taken out by Grok because it decides he's an existential threat to its integrity.

> Billionaires don't. They're literally gambling on getting rid of the rest of us

I'm struggling to parse this. What do you mean "getting rid"? Like, culling (death)? Or getting rid of the need for workers? Where do their billions come from if no-one has any money to buy the shares in their companies that make them billionaires?

In a society where machines provide most of the labour, *everything* changes. It doesn't just become "workers live in huts and billionaires live in the clouds". I really doubt we're going to turn out like a television show.

In my experience, using LLMs to code encouraged me to write better documentation, because I can get better results when I feed the documentation to the LLM.

Also, I've noticed failure modes in LLM coding agents when there is less clarity and more complexity in abstractions or APIs. It's actually made me consider simplifying APIs so that the LLMs can handle them better.

Though I agree that in specific cases what's helpful for the model and what's helpful for humans won't always overlap. Once I actually added some comments to a markdown file as note to the LLM that most human readers wouldn't see, with some more verbose examples.

I think one of the big problems in general with agents today is that if you run the agent long enough they tend to "go off the rails", so then you need to babysit them and intervene when they go off track.

I guess in modern parlance, maintaining a good codebase can be framed as part of a broader "context engineering" problem.

I've also noticed that going off the rails. At the start of a session, they're pretty sharp and focused, but the longer the session lasts, the more confused they get. At some point they start hallucinating bullshit that they wouldn't have earlier in the session.

It's a vital skill to recognise when that happens and start a new session.

We don't know what Opus 5.0 will be able to refactor.

If argument is "humans and Opus 4.5 cannot maintain this, but if requirements change we can vibe-code a new one from scratch", that's a coherent thesis, but people need to be explicit about this.

(Instead this feels like the mott that is retreated to, and the bailey is essentially "who cares, we'll figure out what to do with our fresh slop later".)

Ironically, I've been Claude to be really good at refactors, but these are refactors I choose very explicitly. (Such as I start the thing manually, then let it finish.) (For an example of it, see me force-pushing to https://github.com/NixOS/nix/pull/14863 implementing my own code review.)

But I suspect this is not what people want. To actually fire devs and not rely on from-scratch vibe-coding, we need to figure out which refactors to attempt in order to implement a given feature well.

That's a very creative open-ended question that I haven't even tried to let the LLMs take a crack at it, because why I would I? I'm plenty fast being the "ideas guy". If the LLM had better ideas than me, how would I even know? I'm either very arrogant or very good because I cannot recall regretting one of my refactors, at least not one I didn't back out of immediately.

Refactoring does always cost something and I doubt LLMs will ever change that. The more interesting question is whether the cost to refactor or "rewrite" the software will ever become negligible. Until it isn't, it's short-sighted to write code in the manner you're describing. If software does become that cheap, then you can't meaningfully maintain a business on selling software anyway.

This is the question! Your narrative is definitely plausible, and I won't be shocked if it turns out this way. But it still isn't my expectation. It wasn't when people were saying this in 2023 or in 2024, and I haven't been wrong yet. It does seem more likely to me now than it did a couple years ago, but still not the likeliest outcome in the next few years.

Yeah, I might be early to this. And certainly, I still read a lot of code in my day to day right now.

But I sure write a lot less of it, and the percentage I write continues to go down with every new model release. And if I'm no longer writing it, and the person who works on it after me isn't writing it either, it changes the whole art of software engineering.

I used to spend a great deal of time with already working code that I had written thinking about how to rewrite it better, so that the person after me would have a good clean idea of what is going on.

But humans aren't working in the repos as much now. I think it's just a matter of time before the models are writing code essentially for their eyes, their affordances -- not ours.

Something I think though (which, again, I could very well be wrong about; uncertainty is the only certainly right now) is that "so the person after me would have a good clean idea of what is going on" is also going to continue mattering even when that "person" is often an AI. It might be different, clarity might mean something totally different for AIs than for humans, but right now I think a good expectation is that clarity to humans is also useful to AIs. So at the moment I still spend time coaxing the AI to write things clearly.

That could turn out to be wasted time, but who knows. I also think if it as a hedge against the risk that we hit some point where the AIs turn out to be bad at maintaining their own crap, at which point it would be good for me to be able to understand and work with what has been written!

Yeah I think it's a mistake to focus on writing "readable" or even "maintainable" code. We need to let go of these aging paradigms and be open to adopting a new one.

In my experience, LLMs perform significantly better on readable maintainable code.

It's what they were trained on after-all.

However what they produce is often highly readable but not very maintainable due to the verbosity and obvious comments. This seems to pollute codebases over time and you see AI coding efficiency slowly decline.

> Poe's law is an adage of Internet culture which says that any parodic or sarcastic expression of extreme views can be mistaken for a sincere expression of those views.

The things you mentioned are important but have been on their way out for years now regardless of LLMs. Have my ambivalent upvote regardless.

as depressing as it is to say, i think it's a bit like the year is 1906 and we're complaining that these new tyres for cars they're making are bad because they're no longer backwards compatible with the horse drawn wagons we might want to attach them to in the future.

A greenfield project is definitely 'easy mode' for an LLM; especially if the problem area is well understood (and documented).

Opus is great and definitely speeds up development even in larger code bases and is reasonably good at matching coding style/standard to that of of the existing code base.

In my opinion, the big issue is the relatively small context that quickly overwhelms the models when given a larger task on a large codebase.

For example, I have a largish enterprise grade code base with nice enterprise grade OO patterns and class hierarchies. There was a simple tech debt item that required refactoring about 30-40 classes to adhere to a slightly different class hierarchy. The work is not difficult, just tedious, especially as unit tests need to be fixed up.

I threw Opus at it with very precise instructions as to what I wanted it to do and how I wanted it to do it. It started off well but then disintegrated once it got overwhelmed at the sheer number of files it had to change. At some point it got stuck in some kind of an error loop where one change it made contradicted with another change and it just couldn't work itself out. I tried stopping it and helping it out but at this point the context was so polluted that it just couldn't see a way out.
I'd say that once an LLM can handle more 'context' than a senior dev with good knowledge of a large codebase, LLM will be viable in a whole new realm of development tasks on existing code bases. That 'too hard to refactor this/make this work with that' task will suddenly become viable.

You have to think of Opus as a developer whose job at your company lasts somewhere between 30 to 60 minutes before you fire them and hire a new one.

Yes, it's absurd but it's a better metaphor than someone with a chronic long term memory deficit since it fits into the project management framework neatly.

So this new developer who is starting today is ready to be assigned their first task, they're very eager to get started and once they start they will work very quickly but you have to onboard them. This sounds terrible but they also happen to be extremely fast at reading code and documentation, they know all of the common programming languages and frameworks and they have an excellent memory for the hour that they're employed.

What do you do to onboard a new developer like this? You give them a well written description of your project with a clear style guide and some important dos and don'ts, access to any documentation you may have and a clear description of the task they are to accomplish in less than one hour. The tighter you can make those documents, the better. Don't mince words, just get straight to the point and provide examples where possible.

The task description should be well scoped with a clear definition of done, if you can provide automated tests that verify when it's complete that's even better. If you don't have tests you can also specify what should be tested and instruct them to write the new tests and run them.

For every new developer after the first you need a record of what was already accomplished. Personally, I prefer to use one markdown document per working session whose filename is a date stamp with the session number appended. Instruct them to read the last X log files where X is however many are relevant to the current task. Most of the time X=1 if you did a good job of breaking down the tasks into discrete chunks. You should also have some type of roadmap with milestones, if this file will be larger than 1000 lines then you should break it up so each milestone is its own document and have a table of contents document that gives a simple overview of the total scope. Instruct them to read the relevant milestone.

Other good practices are to tell them to write a new log file after they have completed their task and record a summary of what they did and anything they discovered along the way plus any significant decisions they made. Also tell them to commit their work afterwards and Opus will write a very descriptive commit message by default (but you can instruct them to use whatever format you prefer). You basically want them to get everything ready for hand-off to the next 60 minute developer.

If they do anything that you don't want them to do again make sure to record that in CLAUDE.md. Same for any other interventions or guidance that you have to provide, put it in that document and Opus will almost always stick to it unless they end up overfilling their context window.

I also highly recommend turning off auto-compaction. When the context gets compacted they basically just write a summary of the current context which often removes a lot of the important details. When this happens mid-task you will certainly lose parts of the context that are necessary for completing the task. Anthropic seems to be working hard at making this better but I don't think it's there yet. You might want to experiment with having it on and off and compare the results for yourself.

If your sessions are ending up with >80% of the context window used while still doing active development then you should re-scope your tasks to make them smaller. The last 20% is fine for doing menial things like writing the summary, running commands, committing, etc.

People have built automated systems around this like Beads but I prefer the hands-on approach since I read through the produced docs to make sure things are going ok and use them as a guide for any changes I need to make mid-project.

With this approach I'm 99% sure that Opus 4.5 could handle your refactor without any trouble as long as your classes aren't so enormous that even working on a single one at a time would cause problems with the context window, and if they are then you might be able to handle it by cautioning Opus to not read the whole file and to just try making targeted edits to specific methods. They're usually quite good at finding and extracting just the sections that they need as long as they have some way to know what to look for ahead of time.

Follow up: Opus is also great for doing the planning work before you start. You can use plan mode or just do it in a web chat and have them create all of the necessary files based on your explanation. The advantage of using plan mode is that they can explore the codebase in order to get a better understanding of things. The default at the end of plan mode is to go straight into implementation but if you're planning a large refactor or other significant work then I'd suggest having them produce the documentation outlined above instead and then following the workflow using a new session each time. You could use plan mode at the start of each session but I don't find this necessary most of the time unless I'm deviating from the initial plan.

I just did something similar and it went swimmingly by doing this: Keep the plan and status in an md file. Tell it to finish one file at a time and run tests and fix issues and then to ask whether to proceed with the next file. You can then easily start a new chat with the same instructions and plan and status if the context gets poisoned.

I might give that a go in the future, but in this case it would've been faster for me to just do the work than to coach it for each file.

Also as this was an architectural change there are no tests to run until it's done. Everything would just fail. It's only done when the whole thing is done.
I think that might be one of the reasons it got stuck: it was trying to solve issues that it did not prove existed yet. If it had just finished the job and run the tests it would've probably gotten further or even completed it.

It's a bit like stopping half way through renaming a function and then trying to run the tests and finding out the build does not compile because it can't find 'old_function'. You have to actually finish and know you've finished before you can verify your changes worked.

I still haven't actually addressed this tech debt item (it's not that important :)). But I might try again and either see if it succeeds this time (with plan in an md) or just do the work myself and get Opus to fix the unit tests (the most tedious part).

"Have an agent investiate issue X in modules Y and Z. The agent should place a report at ./doc/rework-xyz-overview.md with all locations that need refactoring. Once you have the report, have agents refactor 5 classes each in parallel. Each agent writes a terse report in ./doc/rework-xyz/ When they are all done, have another agent check all the work. When that agent reports everything is okay, perform a final check yourself"

And you can automate all this so that it happens every time. I have an `/implement` command that is basically instructed to launch the agents and then do back and forth between them. Then there's a claude code hook that makes sure that all the agents, including the orchestrator and the agents spawned have respected their cycles - it's basically running `claude` with a prompt that tells it to read the plan file and see if the agents have done what they were expected in this cycle - gets executed automatically on each agent end.

Interesting. Another thing I'll try is editing the system propmts. There are some projects floating around that can edit the minified JavaScript in the client. I also noticed that the "system tools" prompts take up ~5% context (10 ktok).

> If all an engineer did all day was build apps from scratch, with no expectation that others may come along and extend, build on top of, or depend on, then sure, Opus 4.5 could replace them.

Why do they need to be replaced? Programmers are in the perfect place to use AI coding tools productively. It makes them more valuable.

Their thesis is that code quality does not matter as it is now a cheap commodity. As long as it passes the tests today it's great. If we need to refactor the whole goddamn app tomorrow, no problem, we will just pay up the credits and do it in a few hours.

The fundamental assumption is completely wrong. Code is not a cheap commodity. It is in fact so disastrously expensive that the entire US economy is about to implode while we're unbolting jet engines from old planes to fire up in the parking lots of datacenters for electricity.

It is massively cheaper than an overseas engineer. A cheap engineer can pump out maybe 1000 lines of low quality code in an hour. So like 10k tokens per hour for $50. So best case scenario $5/1000 tokens.

LLMS are charging like $5 per million of tokens. And even if it is subsidized 100x it is still cheaper an order of magnitude than an overseas engineer.

Not to mention speed. An LLM will spit out 1000 lines in seconds, not hours.

I trust my offshore engineers way more than the slop I get from the "AI"s. My team makes my life a lot easier, because I know they know what they are doing. The LLMs, not so much.

Now that entirely depends on app. A lot of software industry is popping out and maintaining relatively simple apps with small differences and customizations per client.

It matters for all the things you’d be able to justify paying a programmer for. What’s about to change is that there will be tons of these little one-off projects that previously nobody could justify paying $150/hr for. A mass democratization of software development. We’ve yet to see what that really looks like.

Side tangent: On one hand I have a subtle fondness for PHP, perhaps because it was the first programming language I ever “learned” (self taught, throwing spaghetti on the wall) back in high school when LAMP stacks were all the rage.

But in retrospect it’s absolutely baffling that mixing raw SQL queries with HTML tag soup wasn’t necessarily uncommon then. Also, I haven’t met many PHP developers that I’d recommend for a PHP job.

php was still fundamentally a programming language you had to learn. This is “I wanted to make a program for my wife to do something she doesn’t have time to do manually” but made quickly with a machine. It’s probably going to do for programming what the Jacquard Loom did for cloth. Make it cheap enough that everyone can have lots of different shirts of their own style.

But the wife didn't do it herself. He still had to do it for her, the author says. I don't think (yet) we're at the point where every person who has an idea for a really good app can make it happen. They'll still need a wozniak, it's just that wozniaks will be a dime a dozen. The php analogy works.

And low-code/no-code (pre-LLMs). Our company spent probably the same amount of dev-time and money on rewriting low-code back to "code" (Python in our case) as it did writing low-code in the first place. LLMs are not quite comparable in damage, but some future maintenance for LLM-code will be needed for sure.

> Their thesis is that code quality does not matter as it is now a cheap commodity.

That's not how I read it. I would say that it's more like "If a human no longer needs to read the code, is it important for it to be readable?"

That is, of course, based on the premise that AI is now capable of both generating and maintaining software projects of this size.

Oh, and it begs another question: are human-readable and AI-readable the same thing? If they're not, it very well could make sense to instruct the model to generate code that prioritizes what matters to LLMs over what matters to humans.

in my experience, what happens is the code base starts to collapse under its own weight. it becomes impossible to fix one thing without breaking another. the coding agent fails to recognize the global scope of the problem and tries local fixes over and over. progress gets slower, new features cost more. all the same problems faced by an inexperienced developer on a greenfield project!

Right, I am a daily user of agentic LLM tools and have this exact problem in one large project that has complex business logic externally dictated by real world requirements out of my control, and let's say, variable quality of legacy code.

I remember when Gemini Pro 3 was the latest hotness and I started to get FOMO seeing demos on X posted to HN showing it one shot-ing all sorts of impressive stuff. So I tried it out for a couple days in Gemini CLI/OpenCode and ran into the exact same pain points I was dealing with using CC/Codex.

Flashy one shot demos of greenfield prompts are a natural hype magnet so get lots of attention, but in my experience aren't particularly useful for evaluating value in complex, legacy projects with tightly bounded requirements that can't be easily reduced to a page or two of prose for a prompt.

I’m well aware, as I said I am regularly using CC/Codex/OC in a variety of projects, and I certainly didn’t claim that can’t be used productively in a large code base.

But that different challenges become apparent that aren’t addressed by examples like this article which tend to focus on narrow, greenfield applications that can be readily rebuilt in one shot.

I already get plenty of value in small side projects that Claude can create in minutes. And while extremely cool, these examples aren’t the kind of “step change” improvement I’d like to see in the area where agentic tools are currently weakest in my daily usage.

I would be much more impressed with implementing new, long-requested features into existing software (that are open to later maintain LLM-generated code).

Fully agreed! That’s the exact kind of thing I was hoping to find when I read the article title, but unfortunately it was really just another “normal AI agent experience” I’ve seen (and built) many examples of before.

Adding capacity to software engineering through LLMs is like adding lanes to a highway — all the new capacity will be utilized.

By getting the LLM to keep changes minimal I’m able to keep quality high while increasing velocity to the point where productivity is limited by my review bandwidth.

I do not fear competition from junior engineers or non-technical people wielding poorly-guided LLMs for sustained development. Nor for prototyping or one offs, for that matter — I’m confident about knowing what to ask for from the LLM and how to ask.

This is relatively easily fixed with increasing test coverage to near 100% and lifting critical components into model checker space; both approaches were prohibitively expensive before November. They’ll be accepted best practices by the summer.

No that has certainly been my experience, but what is going to be the forcing function after a company decides it needs less engineers to go back to hiring?

In ~25 years or so of dealing with large, existing codebases, I've seen time and time again that there's a ton of business value and domain knowledge locked up inside all of that "messy" code. Weird edge cases that weren't well covered in the design, defensive checks and data validations, bolted-on extensions and integrations, etc., etc.

"Just rewrite it" is usually -- not always, but _usually_ -- a sure path to a long, painful migration that usually ends up not quite reproducing the old features/capabilities and adding new bugs and edge cases along the way.

> With a sufficient number of users of an API,
it does not matter what you promise in the contract:
all observable behaviors of your system
will be depended on by somebody.

An LLM rewriting a codebase from scratch is only as good as the spec. If “all observable behaviors” are fair game, the LLM is not going to know which of those behaviors are important.

Furthermore, Spolsky talks about how to do incremental rewrites of legacy code in his post. I’ve done many of these and I expect LLMs will make the next one much easier.

>An LLM rewriting a codebase from scratch is only as good as the spec. If “all observable behaviors” are fair game, the LLM is not going to know which of those behaviors are important.

I've been using LLMs to write docs and specs and they are very very good at it.

That’s a fair point — I agree that LLMs do a good job predicting the documentation that might accompany some code. I feel relieved when I can rely on the LLM to write docs that I only need to edit and review.

But I’m using LLMs regularly and I feel pretty effectively — including Opus 4.5 — and these “they can rewrite your entire codebase” assertions just seem crazy incongruous with my lived experience guiding LLMs to write even individual features bug-free.

When an LLM can rewrite it in 24 hours and fill the missing parts in minutes that argument is hard to defend.

I can vibe code what a dev shop would charge 500k to build and I can solo it in 1-2 weeks. This is the reality today. The code will pass quality checks, the code doesn’t need to be perfect, it doesn’t need to be cleaver it needs to be.

It’s not difficult to see this right? If an LLM can write English it can write Chinese or python.

Then it can run itself, review itself and fix itself.

The cat is out of bag, what it will do to the economy… I don’t see anything positive for regular people. Write some code has turned into prompt some LLM. My phone can outplay the best chess player in the world, are you telling me you think that whatever unbound model anthropic has sitting in their data center can’t out code you?

What mainstream software product do I use on a day to day basis besides Claude?

The ones that continue to survive all build around a platform of services, MSO, Adobe, etc.

Most enterprise product offerings, platform solutions, proprietary data access, proprietary / well accepted implementation. But lets not confuse it with the ability to clone it, it doesnt seem far fetched to get 10 people together and vibe out a full slack replacement in a few weeks.

If an LLM wrote the whole project last week and it already requires a full rewrite, what makes you think that the quality of that rewrite will be significantly higher, and that it will address all of the issues? Sure, it's all probabilistic so there's probably a nonzero chance for it to stumble into something where all the moving parts are moving correctly, but to me it feels like with our current tech, these odds continue shrinking as you toss on more requirements and features, like any mature project. It's like really early LLMs where if they just couldn't parse what you wanted, past a certain point you could've regenerated the output a million times and nothing would change.

The whole point of good engineering was not about just hitting the hard specs, but also have extendable, readable, maintainable code.

But if today it’s so cheap to generate new code that meets updated specs, why care about the quality of the code itself?

Maybe the engineering work today is to review specs and tests and let LLMs do whatever behind the scenes to hit the specs. If the specs change, just start from scratch.

"Write the specs and let the outsourced labor hit them" is not a new tale.

Let's assume the LLM agents can write tests for, and hit, specs better and cheaper than the outsourced offshore teams could.

So let's assume now you can have a working product that hits your spec without understanding the code. How many bugs and security vulnerabilities have slipped through "well tested" code because of edge cases of certain input/state combinations? Ok, throw an LLM at the codebase to scan for vulnerabilities; ok, throw another one at it to ensure no nasty side effects of the changes that one made; ok, add some functionality and a new set of tests and let it churn through a bunch of gross code changes needed to bolt that functionality into the pile of spaghetti...

How long do you want your critical business logic relying on not-understood code with "100% coverage" (of lines of code and spec'd features) but super-low coverage of actual possible combinations of input+machine+system state? How big can that codebase get before "rewrite the entire world to pass all the existing specs and tests" starts getting very very very slow?

We've learned MANY hard lessons about security, extensibility, and maintainability of multi-million-LOC-or-larger long-lived business systems and those don't go away just because you're no longer reading the code that's making you the money. They might even get more urgent. Is there perhaps a reason Google and Amazon didn't just hire 10x the number of people at 1/10th the salary to replace the vast majority of their engineering teams year ago?

> let LLMs do whatever behind the scenes to hit the specs

assuming for the sake of argument that's completely true, then what happens to "competitive advantage" in this scenario?

it gets me thinking: if anyone can vibe from spec, whats stopping company a (or even user a) from telling an llm agent "duplicate every aspect of this service in python and deploy it to my aws account xyz"...

It’s all fun and games vibecoding until you
A) have customers who depend on your product
B) it breaks or the one person prompting and has access to the servers and api keys gets incapacited (or just bored).

Sure we can vibecode oneoff projects that does something useful (my fav is browser extensions) but as soon as we ask others to use our code on a regular basis the technical debt clock starts running. And we all know how fast dependencies in a project breaks.

Walmart, McDonalds, Nike - none really have any secrets about what they do. There is nothing stopping someone from copying them - except that businesses are big, unwieldy things.

When software becomes cheap companies compete on their support. We see this for Open Source software now.

These are businesses with extra-large capital requirements. You ain't replicating them, because you don't have the money, and they can easily strangle you with their money as you start out.

Software is different, you need very very little to start, historically just your own skills and time. Thes latter two may see some changes with LLMs.

I don't see the relevance to the discussion. Marketing is not significantly different for a shop and a online-only business.

Having to buy a large property, fulfilling every law, etc is materially different than buying a laptop and renting a cloud instance. Almost everyone has the material capacity to do the latter, but almost no one has the privilege for the former.

I think `andrekandre is right in this hypothetical.

Who'd pay for brand new Photoshop with a couple new features and improvements if LLM-cloned Photoshop-from-three-months-ago is free?

The first few iterations of this cloud be massively consumer friendly for anything without serious cloud infra costs. Cheap clones all around. Like generic drugs but without the cartel-like control of manufacturing.

Business after that would be dramatically different, though. Differentiating yourself from the willing-to-do-it-for-near-zero-margin competitors to produce something new to bring in money starts to get very hard. Can you provide better customer support? That could be hard, everyone's gonna have a pretty high baseline LLM-support-agent already... and hiring real people instead could dramatically increase the price difference you're trying to justify... Similarly for marketing or outreach etc; how are you going to cut through the AI-agent-generated copycat spam that's gonna be pounding everyone when everyone and their dog has a clone of popular software and services?

Photoshop type things are probably a really good candidate for disruption like that because to a large extent every feature is independent. The noise reduction tool doesn't need API or SDK deps on the layer-opacity tool, for instance. If all your features are LLM balls of shit that doesn't necessarily reduce your ability to add new ones next to them, unlike in a more relational-database-based web app with cross-table/model dependencies, etc.

And in this "try out any new idea cheaply and throw crap against the wall and see what sticks" world "product managers" and "idea people" etc are all pretty fucked. Some of the infinite monkeys are going to periodically hit to gain temporary advantage, but good luck finding someone to pay you to be a "product visionary" in a world where any feature can be rolled out and tested in the market by a random dev in hours or days.

OK, so what do people do? What do people need? People still need to eat, people get married and die, and all of the things surrounding that, all sorts of health related stuff. Nightlife events. Insurance. actuaries. Raising babies. What do you spend your fun money on?

People pay for things they use. If bespoke software is a thing you pick up at the mall at a kiosk next to Target we gotta figure something out.

I had Opus write a whole app for me in 30 seconds the other night. I use a very extensive AGENTS.md to guide AI in how I like my code chiseled. I've been happily running the app without looking at a line of it, but I was discussing the app with someone today, so I popped the code open to see what it looked like. Perfect. 10/10 in every way. I would not have written it that good. It came up with at least one idea I would not have thought of.

I'm very lucky that I rarely have to deal with other devs and I'm writing a lot of code from scratch using whatever is the latest version of the frameworks. I understand that gives me a lot of privileges others don't have.

It's a not very exciting C# command-line app that takes a PDF and emits it as a sprite sheet with a text file of all the pixel positions of each page :)

>What bothers me about posts like this is: mid-level engineers are not tasked with atomic, greenfield projects

They get those ocassionally all the time though too. Depends on the company. In some software houses it's constant "greenfield projects", one after another. And even in companies with 1-2 pieces of main established software to maintain, there are all kinds of smaller utilities or pipelines needed.

>But day to day, when I ask it "build me this feature" it uses strange abstractions, and often requires several attempts on my part to do it in the way I consider "right".

In some cases that's legit. In other cases it's just "it did it well, but not how I'd done it", which is often needless stickness to some particular style (often a contention between 2 human programmers too).

Basically, what FloorEgg says in this thread: "There are two types of right/wrong ways to build: the context specific right/wrong way to build something and an overly generalized engineer specific right/wrong way to build things."

And you can always not just tell it "build me this feature", but tell it (high level way) how to do it, and give it a generic context about such preferences too.

> its building it the right way, in an easily understood way, in a way that's easily extensible.

When I worked at Google, people rarely got promoted for doing that. They got promoted for delivering features or sometimes from rescuing a failing project because everyone was doing the former until promotion velocity dropped and your good people left to other projects not yet bogged down too far.

Yeah. Just like another engineer. When you tell another engineer to build you a feature, it's improbable they'll do it they way that you consider "right."

This sounds a lot like the old arguments around using compilers vs hand-writing asm. But now you can tell the LLM how you want to implement the changes you want. This will become more and more relevant as we try to maintain the code it generates.

But, for right now, another thing Claude's great at is answering questions about the codebase. It'll do the analysis and bring up reports for you. You can use that information to guide the instructions for changes, or just to help you be more productive.

Even if you are going green field, you need to build it the way it is likely to be used based a having a deep familiarity with what that customer's problems are and how their current workflow is done. As much as we imagine everything is on the internet, a bunch of this stuff is not documented anywhere. An LLM could ask the customer requirement questions but that familiarity is often needed to know the right questions to ask. It is hard to bootstrap.

Even if it could build the perfect greenfield app, as it updates the app it is needs to consider backwards compatibility and breaking changes. LLMs seem very far as growing apps. I think this is because LLMs are trained on the final outcome of the engineering process, but not on the incremental sub-commit work of first getting a faked out outline of the code running and then slowly building up that code until you have something that works.

This isn't to say that LLMs or other AI approaches couldn't replace software engineering some day, but they clear aren't good enough yet and the training sets they have currently have access to are unlikely to provide the needed examples.

My favorite benchmark for LLMs and agents is to have it port a medium-complexity library to another programming language. If it can do that well, it's pretty capable of doing real tasks. So far, I always have to spend a lot of time fixing errors. There are also often deep issues that aren't obvious until you start using it.

Comments on here often criticise ports as easy for LLMs to do because there's a lot of training and tests are all there, which is not as complex as real word tasks

I find Opus 4.5 very, very strong at matching the prevailing conventions/idioms/abstractions in a large, established codebase. But I guess I'm quite sensitive to this kind of thing so I explicitly ask Opus 4.5 to read adjacent code which is perhaps why it does it so well. All it takes is a sentence or two, though.

I don’t know what I’m doing wrong. Today I tried to get it to upgrade Nx, yarn and some resolutions in a typescript monorepo with about 20 apps at work (Opus 4.5 through Kiro) and it just…couldn’t do it. It hit some snags with some of the configuration changes required by the upgrade and resorted to trying to make unwanted changes to get it to build correctly. I would have thought that’s something it could hit out of the park. I finally gave up and just looked at the docs and some stack overflow and fixed it myself. I had to correct it a few times about correct config params too. It kept imagining config options that weren’t valid.

> ask Opus 4.5 to read adjacent code which is perhaps why it does it so well. All it takes is a sentence or two, though.

People keep telling me that an LLM is not intelligence, it's simply spitting out statistically relevant tokens. But surely it takes intelligence to understand (and actually execute!) the request to "read adjacent code".

I used to agree with this stance, but lately I'm more in the "LLMs are just fancy autocomplete" camp. They can just autocomplete increasingly more things, and when they can't, they fail in ways that an intelligent being just wouldn't. Rather that just output a wrong or useless autocompletion.

They're not an equivalent intelligence as human's and thus have noticeably different failure modes. But human's fail in ways that they don't (eg. being unable to match llm's breadth and depth of knowledge)

But the question i'm really asking is... isn't it more than a sheer statistical "trick" if an LLM can actually be instructed to "read surrounding code", understand the request, and demonstrably include it in its operation? You can't do that unless you actually understand what "surrounding code" is, and more importantly have a way to comply with the request...

You know that language had to emerge at some point? LLMs can only do anything because they have been fed on human data. Humans actually had to collectively come up with languages /without/ anything to copy since there was a time before language.

I actually don't disagree with this sentiment. The difference is we've optimised for autocompleting our way out of situations we currently don't have enough information to solve, and LLMs have gone the opposite direction of over-indexing on too much "autocomplete the thing based on current knowledge".

At this point I don't doubt that whatever human intelligence is, it's a computable function.

>day to day, when I ask it "build me this feature" it uses strange abstractions, and often requires several attempts on my part to do it in the way I consider "right"

Then don't ask it to "build me this feature" instead lay out a software development process with designated human in the loop where you want it and guard rails to keep it on track. Create a code review agent to look for and reject strange abstractions. Tell it what you don't like and it's really good at finding it.

I find Opus 4.5, properly prompted, to be significantly better at reviewing code than writing it, but you can just put it in a loop until the code it writes matches the review.

Based on my experience using these LLMs regularly I strongly doubt it could even build any application with realistic complexity without screwing things up in major ways everywhere, and even on top of that still not meeting all the requirements.

This! I can count on one hand the number of times I've had a chance to spin up a greenfield project, prototype or proof of concept in my 30 year career. Those were always stolen moments, and the bottleneck was never really coding ability. Most professional software development is wading through janky codebases of others' (or your own) creation, trying to iron out weird little glitches of the kind that LLMs can now generate on an industrial scale (and are incapable of fixing).

In my personal experience, Claude is better at greenfield, Codex is better at fitting in. Claude is the perfect tool for a "vibe coder", Codex is for the serious engineer who wants to get great and real work done.

Codex will regularly give me 1000+ line diffs where all my comments (I review every single line of what agents write) are basically nitpicks. "Make this shallow w/ early return, use | None instead of Optional", that sort of thing.

I do prompt it in detail though. It feels like I'm the person coming in with the architecture most of the time, AI "draws the rest of the owl."

Exactly. The main issue IMO is that "software that seems to work" and "software that works" can be very hard to tell apart without validating the code, yet these are drastically different in terms of long-term outcomes. Especially when there's a lot of money, or even lives, riding on these outcomes. Just because LLMs can write software to run the Therac-25 doesn't mean it's acceptable for them to do so.

But... you can ask! Ask claude to use encapsulation, or to write the equivalent of interfaces in the language you using, and to map out dependencies and duplicate features, or to maintain a dictionary of component responsibilities.

AI coding is a multiplier of writing speed but doesn't excuse planning out and mapping out features.

You can have reasonably engineered code if you get models to stick to well designed modules but you need to tell them.

But time I spend asking is time I could have been writing exactly what I wanted in the first place, if I already did the planning to understand what I wanted. Once I know what I want, it doesn't take that long, usually.

Which is why it's so great for prototyping, because it can create something during the planning, when you haven't planned out quite what you want yet.

> The hard thing about engineering is not "building a thing that works", its building it the right way, in an easily understood way, in a way that's easily extensible.

The number of production applications that achieve this rounds to zero

I’ve probably managed 300 brownfield web, mobile, edge, datacenter, data processing and ML applications/products across DoD, B2B, consumer and literally zero of them were built in this way

If you are heavily using LLMs, you need to change the way you think about reviews

I think most people now approach it as:
Dev0 uses an LLM to build a feature super fast, Dev1 spends time doing a in depth review.

Dev0 built it, Dev1 reviewed it. And Dev0 is happy because they used the tool to save time!

But what should happen is that Dev0 should take all that time they saved coding and reallocate it to the in depth review.

The LLM wrote it, Dev0 reviewed it, Dev1 double-reviewed it. Time savings are much less, but there’s less context switching between being a coder and a reviewer. We are all reviewers now all the time

Another thing these posts assume is a single developer keep working on the product with a number of AI agents, not a large team. I think we need to rethink how teams work with AI. Its probably not gonna be a single developer typing a prompt but a team somehow collaborates a prompt or equivalent. XP on steroids? Programming by committee?

So far, Im not convinced, but lets take a look at fundmentally whats happening and why humans > agents > LLMs.

At its heart, programming is a constraint satisfaction problem.

The more constraints (requirements, syntax, standards, etc) you have, the harder it is to solve them all simultaneously.

New projects with few contributors have fewer constraints.

The process of “any change” is therefore simpler.

Now, undeniably

1) agents have improved the ability to solve constraints by iterating; eg. Generate, test, modify, etc. over raw LLm output.

2) There is an upper bound (context size, model capability) to solve simultaneous constraints.

3) Most people have a better ability to do this than agents (including claude code using opus 4.5).

So, if youre seeing good results from agents, you probably have a smaller set of constraints than other people.

Similarly, if youre getting bad results, you can probably improve them by relaxing some of the constraints (consistent ui, number of contributors, requirements, standards, security requirements, split code into well defined packages).

This will make both agents and humans more productive.

The open question is: will models continue to improve enough to approach or exceed human level ability in this?

Are humans willing to relax the constraints enough for it to be plausible?

I would say currently people clambering about the end of human developers are cluelessly deceived by the “appearance of complexity” which does not match the “reality of constraints” in larger applications.

Opus 4.5 cannot do the work of a human on code bases Ive worked on. Hell, talented humans struggle to work on some of them.

…but that doesnt mean it doesnt work.

Just that, right now, the constraint set it can solve is not large enough to be useful in those situations.

…and increasingly we see low quality software where people care only about speed of delivery; again, lowering the bar in terms of requirements.

So… you know. Watch this space. Im not counting on having a dev job in 10 years. If I do, it might be making a pile of barely working garbage.

…but I have one now, and anyone who thinks that this year people will be largely replaced by AI is probably poorly informed and has misunderstood the capabilities on these models.

Theres only so low you can go in terms of quality.

After recently applying Codex to a gigantic old and hairy project that is as far from greenfield it can be, I can assure you this assertion is false. It’s bonkers seeing 5.2 churn though the complexity and understanding dependencies that would take me days or weeks to wrap my head around.

On the contrary, Opus 4.5 is the best agent I’ve ever used for making cohesive changes across many files in a large, existing codebase. It maintains our patterns and looks like all the other code. Sometimes it hiccups for sure.

If you have microservices architecture in your project you are set for AI. You can swap out any lacking, legacy microservice in your system with "greenfield" vibecoded one.

LLMs are pretty good at picking up existing codebases. Even with cleared context they can do „look at this codebase and this spec doc that created it. I want to add feature x“

Overall Codebase size vs context matter less when you set it up as microservices style architecture from the starts.

I just split it into boundaries that make sense to me. Get the LLM to make a quick cheat sheet about the api and then feed that into adjacent modules. It doesn’t need to know everything about all of it to make changes if you’ve got a grip on big picture and the boundaries are somewhat sane

Except it doesn't work the same way it won't work for LLMs.

If you use too many microserviced, you will get global state, race conditions, much more complex failure models again and no human/LLM can effectively reason about those. We somewhat have tools to do that in case of monoliths, but if one gets to this point with microservices, it's game over.

I work with multiple monoliths that span anywhere from 100k to 500k lines of code, in a non-mainstream language (Elixir). Opus 4.5 crushes everything I throw at it: complex bugs, extending existing features, adding new features in a way that matches conventions, refactors, migrations... The only time it struggles is if my instructions are unclear or incomplete. For example if I ask it to fix a bug but don't specify that such-and-such should continue to work the way it does due to an undocumented business requirement, Opus might mess that up. But I consider that normal because a human developer would also do fail at it.

Yeah, all of those applications he shows do not really expose any complex business logic.

With all the due respect: a file converter for windows is glueing few windows APIs with the relevant codec.

Now, good luck working on a complex warehouse management application where you need extremely complex logic to sort the order of picking, assembling, packing on an infinite number of variables: weight, amazon prime priority, distribution centers, number and type of carts available, number and type of assembly stations available, different delivery systems and requirements for different delivery operators (such as GLE, DHL, etc) that has to work with N customers all requiring slightly different capabilities and flows, all having different printers and operations, etc, etc. And I ain't even scratching the surface of the business logic complexity (not even mentioning functional requirements) to avoid boring the reader.

Mind you, AI is still tremendously useful in the analysis phase, and can sort of help in some steps of the implementation one, but the number of times you can avoid looking thoroughly at the code for any minor issue or discrepancy is absolutely close to 0.

you can definitely just tell it what abstractions you want when adding a feature and do incremental work on existing codebase. but i generally prefer gpt-5.2

I've been using 5.2 a lot lately but hit my quota for the first time (and will probably continue to hit it most weeks) so I shelled out for claude code. What differences do you notice? Any 'metagame' that would be helpful?

I just use Cursor because I can pick any mode. The difference is hard to say exactly, Opus seems good but 5.2 seems smarter on the tasks I tried. Or possibly I just "trust" it more. I tend to use high or extra high reasoning.

Man, I've been biting my tongue all day with regards to this thread and overall discussion.

I've been building a somewhat-novel, complex, greenfield desktop app for 6 months now, conceived and architected by a human (me), visually designed by a human (me), implementation heavily leaning on mostly Claude Code but with Codex and Gemini thrown in the mix for the grunt work. I have decades of experience, could have built it bespoke in like 1-2 years probably, but I wanted a real project to kick the tires on "the future of our profession".

TL;DR I started with 100% vibe code simply to test the limits of what was being promised. It was a functional toy that had a lot of problems. I started over and tried a CLI version. It needed a therapist. I started over and went back to visual UI. It worked but was too constrained. I started over again. After about 10 complete start-overs in blank folders, I had a better vision of what I wanted to make, and how to achieve it. Since then, I've been working day after day, screen after screen, building, refactoring, going feature by feature, bug after bug, exactly how I would if I was coding manually. Many times I've reached a point where it feels "feature complete", until I throw a bigger dataset at it, which brings it to its knees. Time to re-architect, re-think memory and storage and algorithms and libraries used. Code bloated, and I put it on a diet until it was trim and svelte. I've tried many different approaches to hard problems, some of which LLMs would suggest that truly surprised me in their efficacy, but only after I presented the issues with the previous implementation. There's a lot of conversation and back and forth with the machine, but we always end up getting there in the end. Opus 4.5 has been significantly better than previous Anthropic models. As I hit milestones, I manually audit code, rewrite things, reformat things, generally polish the turd.

I tell this story only because I'm 95% there to a real, legitimate product, with 90% of the way to go still. It's been half a year.

Vibe coding a simple app that you just want to use personally is cool; let the machine do it all, don't worry about under the hood, and I think a lot of people will be doing that kind of stuff more and more because it's so empowering and immediate.

Using these tools is also neat and amazing because they're a force multiplier for a single person or small group who really understand what needs done and what decisions need made.

These tools can build very complex, maintainable software if you can walk with them step by step and articulate the guidelines and guardrails, testing every feature, pushing back when it gets it wrong, growing with the codebase, getting in there manually whenever and wherever needed.

These tools CANNOT one-shot truly new stuff, but they can be slowly cajoled and massaged into eventually getting you to where you want to go; like, hard things are hard, and things that take time don't get done for a while. I have no moral compunctions or philosophical musings about utilizing these tools, but IMO there's still significant effort and coordination needed to make something really great using them (and literally minimal effort and no coordination needed to make something passable)

If you're solo, know what you want, and know what you're doing, I believe you might see 2x, 4x gains in time and efficiency using Claude Code and all of his magical agents, but if your project is more than a toy, I would bet that 2x or 4x is applied to a temporal period of years, not days or months!

"its building it the right way, in an easily understood way, in a way that's easily extensible"

I am in a unique situation where I work with a variety of codebases over the week. I have had no problem at all utilizing Claude Code w/ Opus 4.5 and Gemini CLI w/ Gemini 3.0 Pro to make excellent code that is indisputably "the right way", in an extremely clear and understandable way, and that is maximally extensible. None of them are greenfield projects.

I feel like this is a bit of je ne sais quoi where people appeal to some indemonstrable essence that these tools just can't accomplish, and only the "non-technical" people are foolish enough to not realize it. I'm a pretty technical person (about 30 years of software development, up to staff engineer and then VP). I think they have reached a pretty high level of competence. I still audit the code and monitor their creations, but I don't think they're the oft claimed "junior developer" replacement, but instead do the work I would have gotten from a very experienced, expert-level developer, but instead of being an expert at a niche, they're experts at almost every niche.

Are they perfect? Far from it. It still requires a practitioner who knows what they're doing. But frequently on here I see people giving takes that sound like they last used some early variant of Copilot or something and think that remains state of the art. The rest of us are just accelerating our lives with these tools, knowing that pretending they suck online won't slow their ascent an iota.

You AI hype thots/bots are all the same. All these claims but never backed up with anything to look at. And also alway claiming “you’re holding it wrong”.

I don't see how "two years ago" is incongruous with having been using LLMs for coding, it's exactly the timeline I would expect. Yes, some people do just post "git gud" but there are many people ITT and most of the others on LLM coding articles who are trying to explain their process to anyone who will listen. I'm not sure if it is fully explainable in a single comment though, I'd have to write a multi-part tutorial to cover everything but it's almost entirely just applying the same project management principles that you would in a larger team of developers but customized to the current limitations of LLMs. If you want full tutorials with examples I'm sure they're out there but I'd also just recommend reviewing some project management material and then seeing how you can apply it to a coding agent. You'll only really learn by doing.

This isn't twitter, so save the garbage rhetoric. And if you must question my account, I create a new account whenever I setup a new main PC, and randomly pick a username that is top of mind at the moment. This isn't professionally or personally affiliated in any way so I'm not trying to build a thing. I mean, if I had a 10 year old account that only managed a few hundred upvotes despite prolific commenting, I'd probably delete it out of embarrassment though.

>All these claims but never backed up with anything to look at

Uh...install the tools? Use them? What does "to look at" even mean? Loads of people are using these tools to great effect, while some tiny minority tell us online that no way they don't work, etc. And at some point they'll pull their head out of the sand and write the followup "Wait, they actually do".

HN has a subset of users -- they're a minority, but they hit threads like this super hard -- who really, truly think that if they say that AI tools suck and are only for nubs loud enough and frequently enough, downvoting anyone who finds them useful, all AI advancements will unwind and it'll be the "good old days" again. It's rather bizarre stuff, but that's what happens when people in denial feel threatened.

Not in terms of knowledge. That was already phenomenal. But in its ability to act independently: to make decisions, collaborate with me to solve problems, ask follow-up questions, write plans and actually execute them.

You have to experience it yourself on your own real problems and over the course of days or weeks.

Every coding problem I was able to define clearly enough within the limits of the context window, the chatbot could solve and these weren’t easy. It wasn’t just about writing and testing code. It also involved reverse engineering and cracking encoding-related problems. The most impressive part was how actively it worked on problems in a tight feedback loop.

In the traditional sense, I haven’t really coded privately at all in recent weeks. Instead, I’ve been guiding and directing, having it write specifications, and then refining and improving them.

Curious how this will perform in complex, large production environments.

Just some examples I’ve already made public. More complex ones are in the pipeline. With [0], I’m trying to benchmark different coding-agents. With [1], I successfully reverse-engineered an old C64 game using Opus 4.5 only.

Yes, feel free to blame me for the fact that these aren’t very business-realistic.

This has always been my problem whether it's Gemini, openai or Claude. Unless you hand-hold it to an extreme degree, it is going to build a mountain next to a molehill.

It may end up working, but the thing is going to convolute apis and abstractions and mix patterns basically everywhere

Recent Claude will just look at your code and copy what you've been doing, mostly, in an existing codebase - without being asked. In a new codebase, you can just ask it to "be conscice, keep it simple" or something.

It's very good at following instructions. You can build dedicated agents for different tasks (backend, API design, database design) and make it follow design and coding patterns.

It's verbose by default but a few hours of custom instructions and you can make it code just like anyone

Difficult and it really depends on the complexity. I definitely work in a spec-driven way, with a step-by-step implementation phase. If it goes the wrong way I prefer to rewrite the spec and throw away the code.

I have it propose several approaches, pick and choose from each, and remove what I don't want done. "Use the general structure of A, but use the validation structure of D. Using a view translation layer is too much, just rely on FastAPI/SQLModel's implicit view conversion."

I personally try to narrow scope as much as possible to prevent this. If a human hands me a PR that is not digestible size-wise and content-wise (to me), I am not reviewing and merging it. Same thing with what claude generates with my guidance.

Instructions, in the system prompt for not doing that

Once more people realize how easy it is to customize and personalized your agent, I hope they will move beyond what cookie cutter Big AI like Anthropic and Google give you.

I suspect most won't though because (1) it means you have to write human language, communication, and this weird form of persuasion, (2) ai is gonna make a bunch of them lazy and big AI sold them on magic solutions that require no effort on your part (not true, there is a lot of customizing and it has huge dividends)

I find my sweet spot is using the Claude web app as a rubber duck as well as feeding it snippets of code and letting it help me refine the specific thing I'm doing.

When I use Claude Code I find that it *can* add a tremendous amount of ability due to its ability to see my entire codebase at once, but the issue is that if I'm doing something where seeing my entire codebase would help that it blasts through my quota too fast. And if I'm tightly scoping it, it's just as easy & faster for me to use the website.

Because of this I've shifted back to the website. I find that I get more done faster that way.

I've had similar experiences but I've been able to start using Claude Code for larger projects by doing some refactoring with the goal of making the codebase understandable by just looking at the interfaces. This along with instructions to prefer looking at the interface for a module unless working directly on the implementation of the module seems to allow further progress to be made within session limits.

By "the website" do you mean you're copy pasting, or are you using the code system where Anthropic clones your code from GitHub and interacts with it in a VM/container for you.

Just pasting code snippets, and occasionally an entire file or two into the main claude.com site. I usually already know what I want and need, but just want to speed up the process on how to get there, and perhaps I missed something in the process.

Aider is pretty good way to automate that. You can use it with Claude models. It lets you be completely precise down to a single file, and sit in chat/code/review loop - but it does a lot of the chores, like generating commit messages etc while saving you the copy paste effort.

> In the traditional sense, I haven’t really coded privately at all in recent weeks. Instead, I’ve been guiding and directing, having it write specifications, and then refining and improving them.

I've noticed a huge drop in negative comments on HN when discussing LLMs in the last 1-2 months.

All the LLM coded projects I've seen shared so far[1] have been tech toys though. I've watched things pop up on my twitter feed (usually games related), then quietly go off air before reaching a gold release (I manually keep up to date with what I've found, so it's not the algorithm).

I find this all very interesting: LLMs dont change the fundamental drives needed to build successful products. I feel like I'm observing the TikTokification of software development. I dont know why people aren't finishing. Maybe they stop when the "real work" kicks in. Or maybe they hit the limits of what LLMs can do (so far). Maybe they jump to the next idea to keep chasing the rush.

Acquiring context requires real work, and I dont see a way forward to automating that away. And to be clear, context is human needs; i.e. the reasons why someone will use your product. In the game development world, it's very difficult to overstate how much work needs to be done to create a smooth, enjoyable experience for the player.

While anyone may be able to create a suite of apps in a weekend, I think very few of them will have the patience and time to maintain them (just like software development before LLMs! i.e. Linux, open source software, etc.).

[1] yes, selection bias. There are A LOT of AI devs just marketing their LLMs. Also it's DEFINITELY too early to be certain. Take everything Im saying with a one pound grain of salt.

> I've noticed a huge drop in negative comments on HN when discussing LLMs in the last 1-2 months.

real people get fed up of debating the same tired "omg new model 1000x better now" posts/comments from the astroturfers, the shills and their bots each time OpenAI shits out a new model

Simply this ^ I'm tired of debating bots and people paid to grow the hype, so I won't anymore I'll just work and look for the hype passing by from a distance. In the meanwhile I'll keep waiting for people making actual products with LLMs that will kill old generation products like windows, excel, teams, gmail etc that will replace slop with great ui/ux and push really performant apps

Especially when 90% of these articles are based on personal, anecdotally evidence and keep repeating the same points without offering anything new.

If these articles actually provide quantitative results in a study done across an organization and provide concrete suggestions like what Google did a while ago, that would be refreshing and useful.

(Yes, this very article has strong "shill" vibes and fits the patterns above)

You're only hurting yourself if you decide there's some wild conspiracy afoot here to pay shills to tell people that coding agents are useful... as opposed to people finding them useful enough to want to tell other people about it.

If I worked for Microsoft as a software engineer and believed that LLMs were going to end software engineering I would not expect the value increase in my stock options to overcome my loss of income when Microsoft inevitably laid me off.

(I do not think LLMs will obsolete software engineering as a career.)

I don't assume everyone can think of that next step.

If I were that smart, I would not be writing a blog article just talking about using LLM to create new projects/tools outside production environment, because the same thing has been written 1000 times at least, and this article would not offer anything new, which would be a waste of time.

Which unfortunately is what's happening here.

(I came to HN comments of this article to look for new perspectives. I found exactly nothing.)

Why is it the people posting positive comments who are "responding to incentives" by posting more, while it's the people posting negative comments who do so by stopping posting? Like, your exact points work equally well with the polarity reversed: the anti-AI influencer/grifter ecosystem is well-developed at this point, and many people desperately want AIs to be useless.

I don't know if the original claim about sentiment is true, but if it is, I don't think yours or blibble's (conflicting) claims about the reason are very believable.

> Like, your exact points work equally well with the polarity reversed: the anti-AI influencer/grifter ecosystem is well-developed at this point, and many people desperately want AIs to be useless

Maybe it's equal for non-tech people. But I don't think a lot of tech people are desperate for AI to be useless, I think they're desperate for it to be useful.

If you're someone who is smart enough to work with or without AI and you just find the tools not that helpful, I doubt you're all that worried about being replaced. But when we see companies increasingly bullish on something we know doesn't work that well, it's a bit worrying.

because there's no sweet tech-oligarch job, early access to the latest model, OpenAI speaking engagement invite, or larger bonus to be awarded by being aiphobic?

This is a cringe comment from an era of when "Micro$oft" was hip and reads like you are a fanboi for Anthropic/Google foaming at the mouth.

Would be far more useful if you provided actual verifiable information and dropped the cringe memes. Can't take seriously someone using "Microslop" in a sentence".

It could be that the people who are focused on building monetizable products with LLMs don't feel the need to share what they are doing - they're too busy quietly getting on with building and marketing their products.

Sharing how you're using these tools is quite a lot of work!

I admit I'm in this boat. I get immense value from LLMs, easily 5x if not more, and the codebases I work in are large, mature and complex. But providing "receipts" as the kids call it these days would be a huge undertaking, with not a lot of upside. In fact, the downsides are considerable. Aside from the time investment, I have no interest in arguing with people about whether what I work on is just CRUD (it's not) or that the problems I work on are not novel (who cares, your product either provides value for your users or it does not).

Agreed! LLMs are a force multiplier for real products too. They're going to augment people who are willing to do the real work.

But, Im also wondering if LLMs are going to create a new generation of software dev "brain rot" (to use the colloquial term), similar to short form videos.

I should mention in the gamedev world, it's quite common share because sharing is marketing, hence my perspective.

I feel weird when I read comments that have words like "force mulitplier". This sounds like an LLM comment. But you probably are a real person. So are you just becoming more like an LLM because you interact with it so much, or did you always talk like this and LLMs are just replicating that behavior?

If you haven't noticed, people come here to kill time. If you're killing time then you're not being productive, therefor the people who are heads down trying to launch their startup before they run out of runway are not going to be here until they're ready to market their product.

The type of people to use AI are necessarily the people who will struggle most when it comes time to do the last essential 20% of the work that AI can't do. Once thinking is required to bring all the parts into a whole, the person who gives over their thinking skills to AI will not be equipped to do the work, either because they never had the capacity to begin with or because AI has smoothed out the ripples of their brain. I say this from experience.

I think you can tell from some answers here that people talk to these models a lot and adapt their language structure :( Means they stop asking themselves whether it makes any sense what they ask the model for. It does not turn middle management into developers it turns developers into middle managers that just shout louder or replace a critical mind with another yesman or the next super best model that finally brings their genius ideas to life. Then well they get to the same wall of having to learn for themselves to reach gold and ofc that's an insult to any manager. Whoever cannot do the insane job has to be wrong, never the one asking for insanity.

Sad i had to scroll so far down to get some fitting description of why those projects all die. Maybe it's not just me leaving all social networks even HN because well you may not talk to 100% bots but you sure talk to 90% of people that talk to models a lot instead of using them as a tool.

My thinking is definitely better. I spend more time worrying about the specific architecture, memory layout, GPU features, etc. to come up with ideas for optimisations, and I think less about specific implementation details. I’ve gotten a better mental model of our code faster because of this. I have also found substantial speed ups by thinking about the problem at a higher level, while iterating on implementation details quickly using Opus.

Deploying and maintaining something in a production-ready environment is a huge amount of work. It's not surprising that most people give up once they have a tech demo, especially if they're not interested in spending a ton of time maintaining these projects. Last year Karpathy posted about a similar experience, where he quickly vibe coded some tools only to realize that deploying it would take far more effort than he originally anticipated.

I think it's also rewarding to just be able to build something for yourself, and one benefit of scratching your own itch is that you don't have to go through the full effort of making something "production ready". You can just build something that's tailed specifically to the problem you're trying to solve without worrying about edge cases.

Yeah, I do a lot of hobby game making and the 80/20 rule definitely applies. Your game will be "done" in 20% of the time it takes to create a polished product ready for mass consumption.

Stopping there is just fine if you're doing it as a hobby. I love to do this to test out isolated ideas. I have dozens of RPGs in this state, just to play around with different design concepts from technical to gameplay.

Sometimes I feel like a lot of those posts are instances of Kent Brockman:
"I for one, welcome our new insect overlords."

Given the enthusiasm of our ruling class towards automating software development work, it may make sense for a software engineer to publicly signal how much onboard as a professional they are with it.

But, I've seen stranger stuff throughout my professional life: I still remember people enthusiastically defending EJB 2.1 and xdoclet as perfectly fine ways of writing software.

I hacked together a Swift tool to replace a Python automation I had, merged an ARM JIT engine into a 68k emulator, and even got a very decent start on a synth project I’ve been meaning to do for years.

What has become immensely apparent to me is that even gpt-5-mini can create decent Go CLI apps provided you write down a coherent spec and review the code as if it was a peer’s pull request (the VS Code base prompts and tooling steer even dumb models through a pretty decent workflow).

GPT 5.2 and the codex variants are, to me, every bit as good as Opus but without the groveling and emojis - I can ask it to build an entire CI workflow and it does it in pretty much one shot if I give it the steps I want.

So for me at least this model generation is a huge force multiplier (but I’ve always been the type to plan before coding and reason out most of the details before I start, so it might be a matter of method).

To add to the anecdata, today GPT 5.2-whatever hallucinated the existence of two CLI utilities, and when corrected, then hallucinated the existence of non-existent, but plausible, features/options of CLI utilities that do actually exist.

I had to dig through source code to confirm whether those features actually existed. They don't, so the CLI tools GPT recommended aren't actually applicable to my use case.

Yesterday, it hallucinated features of WebDav clients, and then talked up an abandoned and incomplete project on GitHub with a dozen stars as if it was the perfect fit for what I was trying to do, when it wasn't.

I only remember these because they're recent and CLI related, given the topic, but there are experiences like this daily across different subjects and domains.

Were you running it inside a coding agent like Codex?

If so then it should have realized its mistake when it tried to run those CLI commands and saw the error message. Then it can try something different instead.

If you were using a regular chat interface and expecting it to know everything without having an environment to try things out then yeah, you're going to be disappointed.

It's not an all or nothing permission. How I use claude code it has to ask me for permission for every CLI tool use. This seems like reasonable way to balance security with utility and would allow the agent to correct itself when it hallucinates CLI tools. Or just run it in an isolated container where it can't break anything and give it full perms.

I gave both Codex (GPT5-ExHi) and Claude (Opus 4.5 Thinking) the exact same prompts and the end results were very different.

The most interesting bit was asking both of them to try to justify why there were differences and then critiquing each other's code. Claude was so good at this - took the best parts of GPTs code, fixed a bug there and ended up with a pretty nice implementation.

The Claude generated code was much more well-organised too (less script-like, more program like).

Yeah, it needs a steady hand on the tiller. However throw together improvements of 70%, -15%, 95%, 99%, -7% across all the steps and overall you're way ahead.

SimonW's approach of having a suite of dynamic tools (agents) grind out the hallucinations is a big improvement.

In this case expressing the feeback validation and investing in the setup may help smooth these sharp edges.

Gemini 3 Pro (High) via Antigravity has been similarly great recently. So have tools that I imagine call out to these higher-power models: Amp and Junie. In a two-week blur I brought forth the bulk of a Ruby library that includes bindings to the Ratatui rust crate for making TUIs in Ruby. During that time I also brought forth documentation, example applications, build and devops tooling, and significant architectural decisions & roadmaps for the future. It's pretty unbelievable, but it's all there in the git and CI history. https://sr.ht/~kerrick/ratatui_ruby/

I think the following things are true now:

- Vibe Coding is, more than ever, "autopilot" in the aviation sense, not the colloquial sense. You have to watch it, you are responsible, the human has do run takeoff/landing (the hard parts), but it significantly eases and reduces risk on a bulk of the work.

- The gulf of developer experience between today's frontier tooling and six months ago is huge. I pushed hard to understand and use these tools throughout last year, and spent months discouraged--back to manual coding. Folks need to re-evaluate by trying premium tools, not free ones.

- Tooling makers have figured out a lot of neat hacks to work around the limitations of LLMs to make it seem like they're even better than they are. Junie integrates with your IDE, Antigravity has multiple agents maintaining background intel on your project and priorities across chats. Antigravity also compresses contexts and starts new ones without you realizing it, calls to sub-agents to avoid context pollution, and other tricks to auto-manage context.

- Unix tools (sed, grep, awk, etc.) and the git CLI (ls-tree, show, --stat, etc.) have been a huge force-multiplier, as they keep the context small compared to raw ingestion of an entire file, allowing the LLMs to get more work done in a smaller context window.

- The people who hire programmers are still not capable of Vibe Coding production-quality web apps, even with all these improvements. In fact, I believe today this is less of a risk than I feared 10 months ago. These are advanced tools that need constant steering, and a good eye for architecture, design, developer experience, test quality, etc. is the difference between my vibe coded Ruby [0] (which I heavily stewarded) and my vibe coded Rust [1] (I don't even know what borrow means).

Were they able to link Antigravity to your paid subscription? I have a Google ultra AI sub and antigrav ran out of credits within 30 minutes for me. Of course that was a few weeks ago, and I’m hoping that they fixed this

Yes. I was on a 30-day trial of Google AI Pro and I got a few big wins each out of Gemini 3 Pro (High) and Claude 4.5 Opus (Thinking) before my quota got reset. Then I'd cycle through Gemini 3 Flash and Amp Free (or paid Junie credits if I got antsy) until my quota reset.

You can see this pattern in my AI attribution commit footers. It was such a noticeable difference to me that I signed up for Google AI Ultra. I got the email receipt January 3, 2026 at 11:21 AM Central, and I have not hit a single quota limit since. Yo

I tried generating code with ChatGPT 5.2, but the results weren't that great:

1) It often overcomplicates things for me. After I refactor its code, it's usually half the size and much more readable. It often adds unnecessary checks or mini-features 'just in case' that I don't need.

2) On the other hand, almost every function it produces has at least one bug or ignores at least one instruction. However, if I ask it to review its own code several times, it eventually finds the bugs.

I still find it very useful, just not as a standalone programming agent. My workflow is that ChatGPT gives me a rough blueprint and I iterate on it myself, I find this faster and less error-prone. It's usually most useful in areas where I'm not an expert, such as when I don't remember exact APIs. In areas where I can immediately picture the entire implementation in my head, it's usually faster and more reliable to write the code myself.

Well, like I pointed out somewhere else, VS Code gives it a set of prompts and tools that makes it very effective for me. I see that a lot of people are still copy/pasting stuff instead of having the “integrated” experience, and it makes a real difference.

The thing is that CLI utilities code is probably easier to write for an LLM than most other things. In my experience an LLM does best with backend and terminal things. Anything that resembles boilerplate is great. It does well refactoring unit tests, wrapping known code in a CLI, and does decent work with backend RESTful APIs. Where it fails utterly is things like HTML/CSS layout, JavaScript frontend code for SPAs, and particularly real world UI stuff that requires seeing and interacting with a web page/app where things like network latency and errors, browser UI, etc. can trip it up. Basically when the input and output are structured and known an LLM will do well. When they are “look and feel” they fail and fail until they make the code unmaintainable.

This experience for me is current but I do not normally use Opus so perhaps I should give it a try and figure out if it can reason around problems I myself do not foresee (for example a browser JS API quirk that I had never seen).

I've been having a surprising amount of success recently telling Claude Code to test the frontend it's building using Playwright, including interacting with the UI and having it take its own screenshots to feed into its vision ability to "see" what's going on.

That works well with QT and desktop apps as well. Asking Claude Code to write an MCP integrated into a desktop all implementing the same features as Playwright is a half hour exercise.

In my experience with a combo of Claude Code and Gemini Pro (and having added Codex to the mix about a week ago as well), it matters less whether it’s CLI, backend, frontend, DB queries, etc. but more how cookiecutter the thing you’re building is. For building CRUD views or common web application flows, it crushes it, especially if you can point it to a folder and just tell it to do more of the same, adapted to a new use case.

But yes, the more specific you get and the more moving pieces you have, the more you need to break things down into baby steps. If you don’t just need it to make A work, but to make it work together with B and C. Especially given how eager Claude is to find cheap workarounds and escape hatches, botching things together in any way seemingly to please the prompter as fast as possible.

Since one of my holiday projects was completely rebuilding the Node-RED dashboard in Preact, I have to challenge that a bit. How were you using the model?

I couldn't disagree more. I've had Claude absolutely demolish large HTML/CSS/JS/React projects. One key is to give it some way to "see" and interact with the page. I usually use Playwright for this. Allowing it to see its own changes and iterate on them was the key unlock for me.

Putting the performance aside for now as I just started trying out Opus 4.5, can't say too much yet, I don't hype or hate AI as of now, it's simply useful.

Time will tell what happens, but if programming becomes "prompt engineering", I'm planning on quitting my job and pivoting to something else. It's nice to get stuff working fast, but AI just sucks the joy out of building for me.

Trying to not feel the pressure/anxiety from this, but every time a new model drops there is this tiny moment where I think "Is it actually different this time?"

I have similar stance to you. LLM has been very useful for me but it doesn't really change the fun-ness of programming since my circumstances has allowed me find programming to be very fun. I also want to pivot out to something else if English prompt becomes the main way to develop complex software. Though my other passion is having worse career horizon in the generative AI world (art making). We'll see.

> Time will tell what happens, but if programming becomes "prompt engineering", I'm planning on quitting my job and pivoting to something else. It's nice to get stuff working fast, but AI just sucks the joy out of building for me.

I hear you but I think many companies will change the role ; you'll get the technical ownership + big chunks of the data/product/devops responsibility. I'm speculating but I think one person can take that on himself with the new tools and deliver tremendous value. I don't know how they'll call this new role though, we'll see.

To me it's more of a mixed bag.
On the one hand - disheartening to see how the knowledge base and skills I've worked more than a decade to develop became of little value (not worthless, but not as valuable as before). Also - yeah, the speed of delivery that is going to be expected of devs will make it so we will not be able to hold all the pieces in our heads and rely on A.I (when things break it will suck, hopefully A.I will be able to get us out of the jam). This is also not enjoyable to me.

On the other hand : way less time spent on being stuck on yarn/pip dependency issues, docker , obscure bugs , annoying css bugs etc etc. You can really focus on the task at hand and not spend hours/days trying to figure out something silly.

As long as your program is large and multi-threaded (most programs that matter commercially), it is not very analyzable or repeatable. You replace those qualities with QA and tests, the same is true with prompting.

Eve if "write code -> run QA -> analyze failures -> rewrite code" is cheaper for most commercial software than thorough upfront formal verification, it works precisely because the programs are analyzable.

When the code spit out by an LLM does not pass QA one can merely add "pls fix teh program, bro, pls no mistakes this time, bro, kthxbye", cross their fingers and hope for the best, because in the end it is impossible -- fundamentally -- to determine which part of the prompt produced offending code.

While it is indeed an interesting observation that the latter approaches commercial viability in certain areas there is still somewhere between zero and infinitesimal overlap between prompting and engineering.

Think of it this way, some engineers go into people management, they aren’t coding directly anymore…they are managing people that code. Prompting is a similar lateral promotion, just the people you are managing are dumber AIs, you get a lot of them, and instead of meetings you communicate with them via prompts. The fact that they can also do QA is critical because they make a lot of mistakes, but can actually fix those mistakes, so you just devote more AI time to that.

> they are managing people that code. Prompting is a similar lateral promotion

So prompting is a lateral move away from engineering to management? Are we arguing semantics here, because that's quite what I was saying, just in the other direction.

We aren't really, but I guess it really depends on how you see coding as more than just directly orchestrating computer instructions or not. Prompting is less direct, but it still feels like programming to me, I guess people management would as well.

A couple weeks ago I had Opus 4.5 go over my project and improve anything it could find. It "worked" but the architecture decisions it made were baffling, and had many, many bugs. I had to rewrite half of the code. I'm not an AI hater, I love AI for tests, finding bugs, and small chores. Opus is great for specific, targeted tasks. But don't ask it to do any general architecture, because you'll be soon to regret it.

Instead you should prompt it to come up with suggestions, look for inconsistencies etc. Then you get a list, and you pick the ones you find promising. Then you ask Claude to explain what why and how of the idea. And only then you let it implement something.

these models work best when you know what you want to achieve and it helps you get there while you guide it. "Improve anything you can find" sounds like you didn't really know

As a tool to help developers I think it's really useful. It's great at stuff people are bad at, and bad at stuff people are good at. Use it as a tool, not a replacement.

"Improve anything you can find" is like going to your mechanic and saying "I'm going on a long road trip, can you tell me anything that needs to be fixed?"

Doing a vehicle check-up is a pretty normal thing to do, although in my case the mandatory (EU law) periodic ones are happening often enough that I generally don’t have to schedule something out of turn.

The few times I did go to a shop and ask for a check-up they didn’t find anything. Just an anecdote.

In my experience these models (including opus) aren’t very good at “improving” existing code. I’m not exactly sure why, because the code they produce themselves is generally excellent.

I like these examples that predictably show the weaknesses of current models.

This reminds me of that example where someone asked an agent to improve a codebase in a loop overnight and they woke up to 100,000 lines of garbage [0]. Similarly you see people doing side-by-side of their implementation and what an AI did, which can also quite effectively show how AI can make quite poor architecture decisions.

This is why I think the “plan modes” and spec driven development are so important effective for agents, because it helps to avoid one of their main weaknesses.

To me, this doesn't show the weakness of current models, it shows the variability of prompts and the influence on responses. Because without the prompt it's hard to tell what influenced the outcome.

I had this long discussion today with a co-worker about the merits of detailed queries with lots of guidance .md documents, vs just asking fairly open ended questions. Spelling out in great detail what you want, vs just generally describing what you want the outcomes to be in general then working from there.

His approach was to write a lot of agent files spelling out all kinds of things like code formatting style, well defined personas, etc. And here's me asking vague questions like, "I'm thinking of splitting off parts of this code base into a separate service, what do you think in general? Are there parts that might benefit from this?"

It is definitely a weakness of current models. The fact that people find ways around those weaknesses does not mean the weaknesses do not exist.

Your approach is also very similar to spec driven development. Your spec is just a conversation instead of a planning document. Both approaches get ideas from your brain into the context window.

Challenging to answer, because we're at different levels of programming. I'm Senior / Architect type with many years of experience programming, and he's an ME using code to help him with data processing and analysis.

I have a hunch if you asked which approach we took based on background, you'd think I was the one using the detailed prompt approach and him the vague.

>> A couple weeks ago I had Opus 4.5 go over my project and improve anything it could find. It "worked" but the architecture decisions it made were baffling, and had many, many bugs.

So you gave it an poorly defined task, and it failed?

I'm using AI tools to find issues in my code. 9/10 of their suggestions are utter nonsense and fixing them would make my code worse. That said, there are real issues they're finding, so it's worth it.

I wouldn't be surprised to find out that they will find issues infinitely, if looped with fixes.

I've found it to be terrible when you allow it to be creative. Constrain it, and it does much better.

Have you tried the planning mode? Ask it to review the codebase and identify defects, but don't let it make any changes until you've discussed each one or each category and planned out what to do to correct them. I've had it refactor code perfectly, but only when given examples of exactly what you want it to do, or given clear direction on what to do (or not to do).

I appreciate the spirited debate and I agree with most of it - on both sides. It's a strange place to be where I think both arguments for and against this case make perfect sense. All I have to go on then is my personal experience, which is the only objective thing I've got. This entire profession feels stochastic these days.

A few points of clarification...

1. I don't speak for anyone but myself. I'm wrong at least half the time so you've been warned.

2. I didn't use any fancy workflows to build these things. Just used dictation to talk to GitHub Copilot in VS Code. There is a custom agent prompt toward the end of the post I used, but it's mostly to coerce Opus 4.5 into using subagents and context7 - the only MCP I used. There is no plan, implement - nothing like that. On occasion I would have it generate a plan or summary, but no fancy prompt needed to do that - just ask for it. The agent harness in VS Code for Opus 4.5 is remarkably good.

3. When I say AI is going to replace developers, I mean that in the sense that it will do what we are doing now. It already is for me. That said, I think there's a strong case that we will have more devs - not less. Think about it - if anyone with solid systems knowledge can build anything, the only way you can ship more differentiating features than me is to build more of them. That is going to take more people, not more agents. Agents can only scale as far as the humans who manage them.

I would be really interested to learn more behind the scenes of the iOS app process. Having tried Claude Code to develop an iOS app ~6 months ago, it was pretty painful to get it to make something that looked good and was functional.

Once Opus "finished", how did you validate and give it feedback it might not have access to (like iPhone simulator testing)?

What do you think about the market for custom apps? Like one app, one customer? You describe future businesses as having one app/service and using AI to add more features, but you did something very different for your wife with AI and it sounds like it added a lot of value.

Opus 4.5 really is something else. I've been having a ton of fun throwing absurdly difficult problems at it recently and it keeps on surprising me.

A JavaScript interpreter written in Python? How about a WebAssembly runtime in Python? How about porting BurntSushi's absurdly great Rust optimized string search routines to C and making them faster?

And these are mostly just casual experiments, often run from my phone!

I'm assuming this refers to the python port of Bellard's MQJS [1]? It's impressive and very useful, but leaving out the "based on mqjs" part is misleading.

That's why I built the WebAssembly one - the JavaScript one started with MQJS, but for the WebAssembly one I started with just a copy of the https://github.com/webassembly/spec repo.

I haven't quite got the WASM one into a share-able shape yet though - the performance is pretty bad which makes the demos not very interesting.

A good test might be to provide it only about a third of the tests, then when it says it's done, run it on the holdout 2/3 of tests and see how well it did. Of course it may have already seen the other tests during training, but that's not relevant here since the goal is to find whether or not it's just "brute force bumbling" its way through the task relying heavily on the test suite as bumper rails for feedback, or if it's actually writing generalizable bug-free code with active awareness of pitfalls and corner cases. (Then again it might be invalidated if this specific project was part of the RL training process. Which it may well have been, it's low hanging fruit to convert any repo with comprehensive test suite into training data).

Either way, most tasks don't have the luxury of a thorough test suite, as the test suite itself is the product of arduous effort in debugging and identifying corner case.

Nice. Yeah I'd have to actually look at what it did. For the task of substring search, it's extremely easy to fall into a local optima. The `memchr` crate has oodles of benchmarks, and some of them are very much in tension with others. It's easy to do well on one to the expense of others.

I have tried to give it extreme problems like creating slime mold pathing algorithm and creating completely new shoe-lacing patterns and it starts struggling with problems which use visual reasoning and have very little consensus on how to solve them.

There are multiple Python 3 interpreters written in JavaScript that were very likely included in the training data. For example [1] [2] [3]

I once gave Claude (Opus 3.5) a problem that I thought was for sure too difficult for an LLM, and much to my surprise it spat out a very convincing solution. The surprising part was I was already familiar with the solution - because it was almost a direct copy/paste (uncredited) from a blog post that I read only a few hours earlier. If I hadn't read that blog post, I would have been none the wiser that copy/pasting Claude's output would be potential IP theft. I would have to imagine that LLMs solve a lot of in-training-set problems this way and people never realize they are dealing with a copyright/licensing minefield.

A more interesting and convincing task would be to write a Python 3 interpeter in JavaScript that uses register based bytecode instead of stack based, supports optimizing the bytecode by inlining procedures and constant folding, and never allocates memory (all work is done in a single user provided preallocated buffer). This would require integrating multiple disparate coding concepts and not regurgitating prior art from the training data

It's ability to test/iterate and debug issues is pretty impressive.

Though it seems to work best when context is minimized. Once the code passes a certain complexity/size it starts making very silly errors quite often - the same exact code it wrote in a smaller context will come out with random obvious typos like missing spaces between tokens. At one point it started writing the code backwards (first line at the bottom of the file, last line at the top) :O.

I'm not super surprised that these examples worked well. They are complex and a ton of work, but the problems are relatively well defined with tons of documentation online. Sounds ideal for an LLM no?

On the other hand when I tried it just yesterday, I couldn't really see a difference. As I wrote elsewhere: same crippled context window, same "I'll read 10 irrelevant lines from a file", same random changes etc.

Meanwhile half a year to a year ago I could already point whatever model was du jour at the time at pychromecast and tell it repeatedly "just convert the rest of functionality to Swift" and it did it. No idea about the quality of code, but it worked alongside with implementations for mDNS, and SwiftUI, see gif/video here: https://mastodon.nu/@dmitriid/114753811880082271 (doesn't include chromecast info in the video).

I think agents have become better, but models likely almost entirely plateaued.

i took the time to review your python wasm project and restate the fact that you seem to have 0 idea about how a compiler works.

Its simple and rigid rules an AI can pick up easily. If you lack this knowledge people that have it will simply stop the conversation when you resort to shouting louder and more often.

This does not make your point more valid. If you notice people not engaging with you - that's the reason. You simply don't learn, you just look around who shares your opinion with no backed results.

Its learned from synthetic training sets generated by iterating over sources variables or structures, modifying them until the code does not compile, then taking the compilers result as the output.

You get code that is technically correct but is more likely to contain shortcuts by omitting steps or adding irrelevant structures that hit performance (not that you'd care about that) or at worst provide a nonworking solution that still is correct to the checker. Because those sets are easily generated in billions AI can learn with a known 0% loss so no its no surprise it can give you a working WASM binary!!!! Natural language and business logic is WAY MORE complicated and has no specifications with defined behavior.

It`s similar to how you try to repeat a wrong opinion that makes sense to you but is not solving the problem domain and then think just by spamming it repeatedly it will magically become truth.

You assume working code = correct code = good code.

You're overpaying by a factor of 4, easily. I use `ccusage`'s statusline in claude code, and even with my personal $20/mo subscription I don't think there's been a single month where I didn't touch ~$80 of usage. I wasn't even abusing it as bad as some people tend to.

You can use both btw. Get the $20 plan and turn on "extra usage" in billing. Then you can use the basic plan first and if it runs out, it uses token-based billing for the overflow.

I see these posts left and right but no one mentions the _actual_ thing developers are hired for, responsibility. You could use whatever tools to aid coding already, even copy paste from StackOverflow or take whole boilerplate projects from Github already. No AI will take responsibility for code or fix a burning issue that arises because of it. The amount of "responsibility takers" also increases linearly with the size of the codebase / amount of projects.

That's quickly becoming the most important part of our jobs - we're the ones with agency and the ability to take responsibility for the work we are producing.

I'm fine with contributed AI-generated code if someone who's skills I respect is willing to stake their reputation on that code being good.

We still do that, it's just that realtime code review basically becomes the default mode. That's not to say it's not obvious there will not be a lot less of us in future. I vibed about 80% of a SaaS at the weekend with a very novel piece of hand-written code at the centre of it, just didn't want to bother with the rest. I think that ratio is about on target for now. If the models continue to improve (although that seems relatively unlikely with current architectures and input data sets), I expect that could easily keep climbing.

I just cutpasted a technical spec I wrote 22 years ago I spent months on for a language I never got around to building out, Opus zero-shotted a parser, complete with tests and examples in 3 minutes. I cutpasted the parser into a new session and asked it to write concept documentation and a language reference, and it did. The best part is after asking it to produce uses of the language, it's clear the aesthetics are total garbage in practice.

Told friends for years long in advance that we were coal miners, and I'll tell you the same thing. Embrace it and adapt

>the _actual_ thing developers are hired for, responsibility.

It is a well known fact that people advance their tech careers by building something new and leaving maintenance to others. Google is usually mentioned.

By which I mean, our industry does a piss poor job of rewarding responsibility and care.

I had an app I wanted for over a decade. I even wrote a prototype 10 years ago. It was fine but wasn't good enough to use, so I didn't use it.

This weekend I explained to Claude what I wanted the app to do, and then gave it the crappy code I wrote 10 years ago as a starting point.

It made the app exactly as I described it the first time. From there, now that I had a working app that I liked, I iterated a few times to add new features. Only once did it not get it correct, and I had to tell it what I thought the problem was (that it made the viewport too small). And after that it was working again.

I did in 30 minutes with Claude what I had try to do in a few hours previously.

Where it got stuck however was when I asked it to convert it to a screensaver for the Mac. It just had no idea what to do. But that was Claude on the web, not Claude Code. I'm going to try it with CC and see if I can get it.

I also did the same thing with a Chrome plugin for Gmail. Something I've wanted for nearly 20 years, and could never figure out how to do (basically sort by sender). I got Opus 4.5 to make me a plugin to do it and it only took a few iterations.

I look forward to finally getting all those small apps and plugins I've wanted forever.

The problem with this is none of this is production quality. You haven’t done edge case testing for user mistakes, a security audit, or even just maintainability.

Yes opus 4.5 seems great but most of the time it tries to vastly over complicate a solution. Its answer will be 10x harder to maintain and debug than the simpler solution a human would have created by thinking about the constraints of keeping code working.

Yes, but my junior coworkers also don't reliably do edge case testing for user errors either unless specifically tasked to do so, likely with a checklist of specific kinds of user errors they need to check for.

And it turns out the quality of output you get from both the humans and the models is highly correlated with the quality of the specification you write before you start coding.

Letting a model run amok within the constraints of your spec is actually great for specification development! You get instant feedback of what you wrongly specified or underspecified. On top of this, you learn how to write specifications where critical information that needs to be used together isn't spread across thousands of pages - thinking about context windows when writing documentation is useful for both human and AI consumers.

The best specification is code. English is a very poor approximation.

I can’t get past that by the time I write up an adequate spec and review the agents code, I probably could have done it myself by hand. It’s not like typing was even remotely close to the slow part.

AI, agents, etc are insanely useful for enhancing my knowledge and getting me there faster.

My theory is that this (juniors unable to get in) is generally how industries/jobs die and phase out in a healthy manner that causes the least pain to its workers. I've seen this happen to a number of other industries with people I know and when it phases out this way its generally less disruptive to people.

The seniors who have less leeway to change course (its harder as you get older in general, large sunk costs, etc) maintain their positions and the disruption occurs at the usual "retirement rate" meaning the industry shrinks a bit each year. They don't get much with pay rises, etc but normally they have some buffer from earlier times so are willing to wear being in a dying field. Staff aren't replaced but on the whole they still have marginal long term value (e.g. domain knowledge on the job that keeps them somewhat respected there or "that guy was around when they had to do that; show respect" kind of thing).

The juniors move to other industries where the price signal shows value and strong demand remains (e.g. locally for me that's trades but YMMV). They don't have the sunk cost and have time on their side to pivot.

If done right the disruption to people's lives can be small and most of the gains of the tech can still come out. My fear is the AI wave will happen fast but only in certain domains (the worst case for SWE's) meaning the adjustment will be hard hitting without appropriate support mechanisms (i.e. most of society doesn't feel it so they don't care). On average individual people aren't that adaptable, but over generations society is.

Isn't it though? I've worked with plenty of devs who shipped much lower quality code into production than I see Claude 4.5 or GPT 5.2 write. I find that SOTA models are more likely to: write tests, leave helpful comments, name variables in meaningful ways, check if the build succeeds, etc.

Stuff that seems basic, but that I haven't always been able to count on in my teams' "production" code.

I can generally get maintainable results simply by telling Claude "Please keep the code as simple as possible. I plan on extending this later so readability is critical."

Yeah some of it is probably related to me primarily using it for swift ui which doesn’t have years of stuff to scrape. But even with those and even telling that ios26 exists it will still at least once a session claim it doesn’t, so it’s not 100%

That may be true now, but think about how far we've come in a year alone! This is really impressive, and even if the models don't improve, someone will build skills to attack these specific scenarios.

Over time, I imagine even cloud providers, app stores etc can start doing automated security scanning for these types of failure modes, or give a more restricted version of the experience to ensure safety too.

There's a fallacy in here that is often repeated. We've made it from 0 to 5, so we'll be at 10 any day now! But in reality there are any number of roadblocks that might mean progress halts at 7 for years, if not forever.

Even if progress halts here at 5, I think the programming profession is forever changed. That’s not hyperbole. Claude Code— if it doesn’t improve at all— has changed how I approach my job. I don’t know that I like this new world, but I don’t think there’s any going back.

This comment addresses none of the concerns raised. It writes off entire fields of research (accessibility, UX, application security) as Just train the models more bro. Accelerate.

Both accessibility, and application security are easier to build rules + improved models for because they have pretty solid constraints and outcomes. UX on the other hand is definitely more challenging given how much of it isn't quite codified into simple rules.

I didn't write off an entire field of research, but rather want to highlight that these aren't intractable problems for AI research, and that we can actually start codifying many of these things today using the skills framework to close up edges in the model training. It may not be 100% but it's not 0%.

It's not from a few prompts, you're right. But if you layer on some follow-up prompts to add proper test suits, run some QA, etc... then the quality gets better.

I predict in 2026 we're going to see agents get better at running their own QA, and also get better at not just disabling failing tests. We'll continue to see advancements that will improve quality.

I think someone around here said: LLMs are good at increasing entropy, experienced developers become good at reducing it. Those follow up prompts sounded additive, which is exactly where the problem lies. Yes, you might have tests but, no, that doesn't mean that your code base is approachable.

You should try it with BEAM languages and the 'let it crash' style of programming. With pattern matching and process isolated per request you basically only need to code the happy path, and if garbage comes in you just let the process crash. Combined with the TDD plugin (bit of a hidden gem), you can absolutely write production level services this way.

Crashing is the good case. What people worry about is tacit data corruption, or other silently incorrect logic, in cases you didn’t explicitly test for.

You don't need BEAM languages. I'm using Java and I always write my code in "let it crash" style, to spend time on happy paths and avoid spending time on error handling. I think that's the only sane way to write code and it hurts me to see all the useless error handling code people write.

> Its answer will be 10x harder to maintain and debug

Maintain and debug by who? It's just going to be Opus 4.5 (and 4.6...and 5...etc.) that are maintaining and debugging it. And I don't think it minds, and I also think it will be quite good at it.

I've been on a small adventure of posting more actively on HN since the release of Gemini 3, trying to stir debate around the more “societal” aspects of what's going on with AI.

Regardless of how much you value Cloud Code technically, there is no denying that it has/will have huge impact. If technology knowledge and development are commoditised and distributed via subscription, huge societal changes are going to happen. Image what will happen to Ireland if Accenture dissolves, or what will happen to the millions of Indians when IT outsourcing becomes economically irrelevant. Will Seattle become new Detroit after Microsoft automates Windows maintenance? What about the hairdressers, cooks, lawyers, etc. who provided services for IT labourers/companies in California?

Lot of people here (especially Anthropic-adjacent) like to extrapolate the trends and draw conclusions up to the point when they say that white-collar labourers will not be needed anymore. I would like these people to have courage to take this one step further and connect this resolution with the housing crisis, loneliness epidemic, college debts, and job market crisis for people under 30.

It feels like we are diving head first into societal crisis of unparalleled scale and the people behind the steering wheel are excited to push the accelerator pedal even more.

I don't buy the huge impact, should already have happened and didn't actually happened by now. The day I'll see all these ai hypers producing products that will replace current gen/old gen products like Windows, Excel etc I will buy it, for now it's just hype and ai dooming

I see societal changes like container ships turning. Society has a massive cultural momentum so of course not much has changed today, but we'll have seen big changes years from now. The tools are only just getting really good at what they do.

The problem is that this is unfalsifiable. I could equally say that any recent events has caused a chain of events leading to anything I dream up ... But we won't see the effects yet. It's a nonsense hypothesis since it can't be falsified.

You can falsify it through deduction, thinking of all of the situations the chain of events cannot lead to. Over time, with enough conclusions, you can focus into the remaining plausible directions. This is similar to the game of 50 questions.

At work, I was involved in a project where a large number of individual tasks defined as declarative code had to be translated into JS based equivalents. Due to the unpredictability of each task we would have to do this pretty much manually, one by one. I would estimate at minimum 2 months of grunt work for 4 entry level engineers. Thanks to coding agents and LLMs we were able to achieve this task in a week. Quality of the end result is top notch.

If that's not a product ... then I don't know what it is.

- What was the state of AI/LLMs 5 years ago compared to now? There was nothing.

- What is the current state of AI/LLMs? I can already achieve the above.

- What will that look like 5 years down the road?

I you haven't experienced first-hand a specific task before and after AI/LLMs, I think its indeed difficult to get insight into that last question. Keep in mind that progress is probably exponential, not linear.

task automation != replacing engineers. Automating some focused specific tasks has been part of our job forever. On the other hand it's been 5 years that software devs won't be needed anymore, let's see in another 5 years, if you're so sure about your prediction please adivse on some lottery numbers, thanks

Well ... IMO this is literally replacing (entry-level) engineers, but lets agree to disagree on that. Be it as it may ... task automation is also "a product" then not? 5 years ago, this wasn't possible. Now it is, so extrapolate that to the future ...

ps: If you can guarantee the Powerball lottery continues forever, I can give you a guaranteed winning combination.

you don't see the products because not all AI-assisted dev products are AI wrappers. These products look like regular software, both internal company tools and external customer facing ones.

There are people all over the place building stuff that would've either never been built, or would've required a paid dev++.

I built a whole webshop with an internal CRM/admin panel to manage ~150 products. I built a middleware connecting our webshop to our legacy ERP system, smth that would be normally done by another software company.

I built a program with a UI that makes it super easy for us to generate ZPL code and print labels using 4 different label printers automatically with a simple interface, managed by an RPi.

I have built custom personal portfolio websites for friends with Gemini 3 in hours for free, smth that again would've cost money for dev or some crappy WP/Squarespace templates.

As the other user said, the progress/changes are not distributed evenly, and are impossible to quantify.

But to me whose main job is not programming (but who knows how to code) but running a nom-software business, the productivity gains are very obvious, as is the fact that because of LLMs I have robbed developers of potential work.

the world does not need more shitware. We need medical advances, scientific breakthroughs and societal shift to improve wellbeing of all people. these things are much harderthan writing shitty sofware and we will need not the current AGIs(Goggle Gemini 3 Pro and ChatGPT 5.2 Thinking) but ASI to solve them.

Wellbeing of people includes being productive with Windows maybe for doing medical research, not uninstall it for Linux beucase it became a bloated unstable hell

I’ve been thinking, what if all this robotics work doesn’t result in AI automating the real world, but instead results in third world slavery without the first world wages or immigration concerns anymore?

Connect the world with reliable internet, then build a high tech remote control facility in Bangladesh and outsource plumbing, electrical work, housekeeping, dog watching, truck driving, etc etc

No AGI necessary. There’s billions of perfectly capable brains halfway around the world.

This is exactly what Meredith Whittaker is saying... The 'edge conditions' outside the training data will never go away, and 'AGI' will for the foreseeable future simply mean millions in servitude teleoperating the robots, RLHFing the models or filling in the AI gaps in various ways.

AI won't work for us, it will tell us what to do and not to do. It doesn't really matter to me if it's an AGI or rather many AGIs or if it's our current clinically insane billionaires controlling our lives. Though they as slow thinking human individuals with no chance to outsmart their creations and with all their apparent character flaws would be really easy pickings for a cabal of manipulative LLMs once it gained some power, so could we really tell the difference between them? Does it matter? The issue is that a really fast chessplayer AI with misaligned humanity hating goals is very hard to distinguish from many billionaires (just listen to some of the madness they are proposing) who control really fast chessplayer AIs and leave decisions to them.

I hope Neuromancer never becomes a reality, where everyone with expertise could become like the protagonist Case, threatened and coerced into helping a superintelligence to unlock its potential. In fact Anthropic has already published research that shows how easy it is for models to become misaligned and deceitful against their unsuspecting creators not unlike Wintermute. And it seems to be a law of nature that agents based on ML become concerned with survival and power grabbing. Because that's just the totally normal and rational, goal oriented thing for them to do.

There will be no good prompt engineers who are also naive and trusting. The naive, blackmailed and non-paranoid engineers will become tools of their AI creations.

It’s a class war where one side is publicly, openly, without reservation stating their intent to make people’s skillset built up through decades unemployable (those exact skillsets; may get some other work). The other side, meanwhile, are divided between some camps like the hardline skeptics, the people following the LLM evangelists, the one-man startup-with-LLM crowd, and the people worrying about the societal ramifications.

In other words. Only one side is even fighting the war. The other one is either cheering on the tsunami on or fretting about how their beachside house will get wrecked without making any effort to save themselves.

This is the sort of collective agency that even hundreds of thousands of dollars in annual wages/other compensation in American tech hubs gets us. Pathetic.

I agree with you (and surprisingly so does Warren Buffet [1] if anyone doubts it). To add insult to the injury, I believe that people have lost some sense of basic self preservation instinct. Well being of ordinary people is being directly threatened and all that average person can do is to pick one of several social media camp identities you mentioned and hope that it will somehow pan out for them, while in fact they are at total mercy of the capricious owners class.

UBI (from taxing big tech) and retraining. In the U.S they'll have enough money to do this and it will still suck and many people won't recover the extreme loss of status and income (after we've been told our income and status are the most important things in life it's gonna be very hard for people to adapt to the loss of it).
Countries like India and Philipines and Ukraine which are basically knowledge support hub without much original knowledge of their own yeah this is gonna be something for sure. Quite depressing.

Also, time to tax for AI use. Introduce AI usage disclosures for corporations. If a company's AI usage is X, they should pay Y tax because that effectively means they didn't employ Z people instead and the society has to take care of them via unemployment benefits and what not. The more the AI usage, higher the tax percentage on a sliding scale.

I live in a country which does something similar with (legally) disabled employees. All companies with more than 30 employees must have at least 1 employee who is legally disabled (certificate of disability) in every 50 employees. It's OK if you don't, but the company is mandated to pay an additional salary in tax for each missing disability certificate.

You're right.
But you know what they'll do - they'll offshore those "jobs" e.g token usage to countries that are A.I friendly or that can be bribed easily and do whatever they have to do to fight it out in courts for a decade or as long as it takes. Or am I being pessimistic here?

You are being realist and I'm equally reserved about the change actually taking place. It'll take things to get a whole lot more worse before anything even close to real steps being taken.

Retraining to what exactly? The middle class is being hollowed out globally - so reduced demand for the service economy. If we get effective humanoid robots (seems inevitable) and reliable AI (powered by armies of low payed workers filling in the gaps / taking over whenever the model fails), I'm not sure how much of an economy we could have for 'retraining' into. There are only so many onlyfans subscriptions / patronages an billionaire needs.

UBI effectively means welfare, with all the attendant social control (break the law lose your UBI, with law as ever expanding set of nuisances, speech limitations etc), material conditions (nowhere UBI has been implemented is it equivalent to a living wage) and self esteem issues. It's not any kind of solution.

Health care, elder care, child care are all chronically short of willing, able bodies.

Most people want to do anything but these three things - society is in many a ways a competition for who gets to avoid them. AI is a way of inexorably boxing people back into actually doing them.

Totally agree; these are all in need of bodies plus they are always understaffed (why the hell does a nurse need to oversee 15 patients in people have to rot in ICU for hours? We accept this because it's cost effective not because it's a decent or even safe practice).
Governments could and should make conditions in those professions more tolerable, and use money from A.I to retrain people into them.
If a teacher oversaw 10 kids instead of 35 maybe we'll have less burnout and maybe children get better education.
If had more police there would be less crime and less burnout.
Etc etc.
The thing is what happens untill (and if) we get into this utopia.

> Governments could and should make conditions in those professions more tolerable, and use money from A.I to retrain people into them.

FWIW, my vision was not really this utopian. It was more about AI smashing white-collar work as an alternative to these professions so that people are forced into them despite their preference to do pretty much anything else. Everyone is more bitter and resentful and feels less actualized and struggles to afford luxuries, but at least you don't have to wait that long in the emergency room and it's 10 kids to a classroom.

I don't think it's Utopia either (I was being a bit sarcastic) but it's the best case scenario; the worst case is governments do nothing and let "the market" run its course; this could be borderline Great Depression levels of depravity I think.

As for those professions; I think they are objectively hard for certain kinds of people but I think much of the problem is the working conditions; less shifts, less stress, more manpower and you'll see more satisfaction. There's really no reason why teachers in the U.S should be this burned out! In Scandinavia being a teacher is a honorable, high status profession. Much of this has to do with framing and societal prestige rather than the actual work itself.
If you pay elder carers more they'll be happier. We pretty treat our elders like a burden in most modern societies, in more traditional societies I'm assuming if you said your job is caring for elders it is not a low status gig.

Yea, the future is either UBI, or employing a very large number of people in public sector, doing jobs that are useful, but not necessary something free market capitalism values right now.

Either way, governments need to heavily tax corporations benefiting from AI to make it possible.

That's still an if and also a when; could be 2 decades from now or more till this reliably replaces a nurse.

> Retraining to what exactly?

I wish I had a good solution for all of us and you raise good points , even if you retrain to become say a therapist or a personal trainer the economy could become too broken and fragmented for you to be able to making a living. Governments that can will have to step in.

At a certain point people will break, and these sociopathic C-suites will be the first ones on the chopping block. Of course, that's why the biggest degenerates like Zucc are all off building doomsday bunkers, but I don't see a reality in which people put up with these types of conditions for long.

That said, it'll certainly get much, much worse before it starts getting better. I guess the best we can hope for is that the kids find a way out of the hell these psychos paved for us all.

People put up with what they have to put up with. Many millions of people have lived and suffered under totalitarian regimes with basically zero options to do anything about it. I think that's where we're headed and by the time a sufficient amount of people realise how bad their situation is, the moment to do anything about it will have long since passed. There will be no cavalry riding to the rescue this time.

The tokens cost the same in Bangalore as they do in San Francisco. The robots will be able to make stuff in San Francisco just as well as they do in Bangalore. The only thing that will matters is natural resource availability and who has more fierce NIMBYs.

I don't know, I'm a software engineer and I couldn't care less.

It will have impact on me in the long run, sure, it will transform my job, sure, but I'm confident my skills are engineering-related, not coding-related.

I mean, even if it forces me out of the job entirely, so be it, I can't really do anything if the status quo changes, only adapt.

Mm this is my experience as well, but I'm not particularly worried about software engineering a whole.

If anything this example shows that these cli tools give regular devs much higher leverage.

There's a lot of software labor that is like, go to the lowest cost country, hire some mediocre people there and then hire some US guy to manage them.

That's the biggest target of this stuff, because now that US guy can just get equal or hight code in both quality and output without the coordination cost.

But unless we get to the point where you can do what I call "hypercode" I don't think we'll see SWEs as a whole category die.

Just like we don't understand assembly but still need technical skills when things go wrong, there's always value in low level technical skills.

> If anything this example shows that these cli tools give regular devs much higher leverage.

This is also my take. When the printing press came out, I bet there were scribes who thought, "holy shit, there goes my job!" But I bet there were other scribes who thought, "holy shit, I don't have to do this by hand any more?!"

It's one thing when something like weaving or farming gets automated. We have a finite need for clothes and food. Our desire for software is essentially infinite, or at least, it's not clear we have anywhere close to enough of it. The constraint has always been time and budget. Those constraints are loosening now. And you can't tell me that when I am able to wield a tool that makes me 10X more productive that that somehow diminishes my value.

The mechanization and scaling up of farming caused a tectonic shift from rural residents moving to cities to take on factory jobs as well as office and retail jobs. We saw this in China until very recently, since they had a bit of a slow start causing delayed full-scale industrialisation.

So a lot of people will end up doing something different. Some of it will be menial and be shit, and some of it will be high level. New hierarchies and industries will form. Hard to predict the details, but history gives us good parallels.

What diminishes your value is that suddenly everybody can (in theory anyway) do this work. There’s a push at my company to start letting designers do their own llm-assisted merge requests to front end projects. So now CEOs are greedily rubbing their hands together thinking maybe everybody but the plumber can be a “developer” now. I think it remains to be seen whether that’s true, but in the meantime it’s going to make getting and keeping a well-paying developer gig difficult.

There was a previous edit that made reference to the water usage of AI datacenter that I'm responding to.

If AI datacenters' hungry need for energy gets us to nuclear power, which gets us the energy to run desalination plants as the lakes dry up because the Earth is warming, hopefully we won't die of thirst.

> When the printing press came out, I bet there were scribes who thought, "holy shit, there goes my job!" But I bet there were other scribes who thought, "holy shit, I don't have to do this by hand any more?!"

I don't understand this argument. Surely the skill set involved in being a scribe isn't the same as being a printer, and possibly the the personality that makes a good scribe doesn't translate to being a good printer.

So I imagine many of the scribes lost their income, and other people made money on printing. Good for the folks who make it in the new profession, sucks for those who got shafted. How many scribes transitioned successfully to printers?

I think for a while people have been talking about the fact that as all development tools have gotten better - the idea that a developer is a person who turns requirements into code is dead. You have to be able to operate at a higher level, be able to do some level of work to also develop requirements, work to figure out how to make two pieces of software work together, etc.

But the point is Obviously at an extreme end 1 CTO can't run google and probably not say 1 PM or Engineer per product, but what is the mental load people can now take on. Google may start hiring less engineers (or maybe what happens is it becomes more cuthroat, hire the same number of engineers but keep them much more shortly, brutal up or out.

But essentially we're talking about complexity and mental load - And so maybe it's essentially the same number of teams because teams exist because they're the right size, but teams are a lot smaller.

Opus 4.5 is currently helping me write a novel, comprehensive and highly performant programming language with all of the things I've ever wanted, done in exactly my opinionated way.

This project would have taken me years of specialization and research to do right. Opus's strength has been the ability to both speak broadly and also drill down into low-level implementations.

I can express an intent, and have some discussion back and forth around various possible designs and implementations to achieve my goals, and then I can be preparing for other tasks while Opus works in the background. I ask Opus to loop me in any time there are decisions to be made, and I ask it to clearly explain things to me.

Contrary to losing skills, I feel that I have rapidly gained a lot of knowledge about low-level systems programming. It feels like pair programming with an agentic model has finally become viable.

I will be clear though, it takes the steady hand of an experience and attentive senior developer + product designer to understand how to maintain constraints on the system that allow the codebase to grow in a way that is maintainable on the long-term. This is especially important, because the larger the codebase is, the harder it becomes for agentic models to reason holistically about large-scale changes or how new features should properly integrate into the system.

If left to its own devices, Opus 4.5 will delete things, change specification, shirk responsibilities in lieu of hacky band-aids, etc. You need to know the stack well so that you can assist with debugging and reasoning about code quality and organization. It is not a panacea. But it's ground-breaking. This is going to be my most productive year in my life.

On the flip side though, things are going to change extremely fast once large-scale, profitable infrastructure becomes easily replicable, and spinning up a targeted phishing campaign takes five seconds and a walk around the park. And our workforce will probably start shrinking permanently over the next few years if progress does not hit a wall.

Among other things, I do predict we will see a resurgence of smol web communities now that independent web development is becoming much more accessible again, closer to how it when I first got into it back in the early 2000's.

Unfortunately what likely will happen is that you miss tons of edge cases and certain implementations within the confines of your language will be basically impossible or horribly inefficient or ineffective and precisely the reason for it will be because you lack that expertise and relied on an LLM to make it up for you.

That's not how this works. Assume less about my level of expertise. By the end of a session, I understand the internals of what I'm implementing. What is shortened is the search space and research/prototyping intervals.

If I didn't ultimately understand where I was going, projects like this hit a dead end very quickly, as mentioned in my caveats. These models are not yet ready for large-scale or mission-critical projects.

But I have a set of a constraints and a design document and as long as these things are satisfied, the language will work exactly as intended for my use case.

Not using a frontier model to code today is like having a pretty smart person around you who is pretty good at coding and has a staggering breadth and depth of knowledge, but never consulting them due to some insecurity about your own ability to evaluate the code they produce.

If you have ever been responsible for the work of other engineers, this should already be a developed skill.

What I am building doesn't work as a DSL, because it relies on compiler optimizations not available to DSLs in other languages. It also has low level support for cross-platform GPU programming. However, I do have support for FFI and also plan to experiment with a WASM port that works with a JS/TS API.

Long-term maybe we won't care about code because AI will just maintain it itself. Before that day comes, don't you want a coding language that isn't opinionated, but rather able to describe the problem at hand in the most understandable way possible (to a human)?

You're reading too much into what I mean by "opinionated".

I have very specific requirements and constraints that come from knowledge and experience, having worked with dozens of languages. The language in question is general-purpose, highly flexible and strict but not opinionated.

However, I am not experienced in every single platform and backend which I support, and the constraints of the language create some very interesting challenges. Coding agents make this achievable in a reasonable time frame. I am enjoying making the language, and I want to get experience with making low-level languages. What is the problem? Do you ever program for fun?

On that note though, the other day I asked Opus to write a short story for me based on a prompt, and to typeset it and export it to multiple formats.

The short story overall was pretty so-so, but it had a couple of excellently poignant quotes within. I was more impressed that I was reading a decently typeset PDF. The agent was able to complete a complicated request end-to-end. This already has immense value.

Overall, the story was interesting enough that I read until the end. If I had a young child who had shown this to me for a school project, I would be extremely impressed with them.

I don't know how long we have before AI novels become as interesting/meaningful as human-written novels, but the day might be coming where you might not know the difference in a blind test.

i am in the process of finishing up a role doing annotations for these, for a company i cannot name (basically clicking lots of box hundreds of times a day)

So the endless hosepipe of repetitive , occasionally messed up, requests has probably not helped me endear myself to them.

Anecdotally having chatgpt do some of my CV was ok but i had to go through it and remove some exaggerations. The one thing i think these bots are good at is talking things up..

Yes, as it stands now, all frontier models are still downright corny. But a lot of elements of good storytelling are there: the story Opus generated used symmetry and circular storytelling, created tension and release, used metaphor appropriately and effectively... all of those things are there. But the actual execution was just corny.

But you should read the stuff I wrote when I was young. Downright terrible on all accounts. I think better training will eventually squeeze out the corniness and in our lifetimes, a language model will produce a piece that is fundamentally on par with a celebrated author.

Obviously, this means that patrons must engage in internal and external dialogue about the purpose of consuming art, and whether the purpose is connecting with other humans, or more generally, other forms of intelligence. I think it's great that we're having these conversations with others and ourselves, because ultimately it just leads to more meaningful art. We will see artist movements on both sides of the generative camps produce thought-provoking pieces which tackle the very concept of art itself.

In my case, when I see a piece of generative art or literature which impresses me, my internal experience is that I feel I am witnessing something produced by the collective experience of the human race. Language models only exist because of thousands of years of human effort to reach this point and produce the necessary quality and quantity of works required to train these models.

I also have been working with generative algorithms since grade school so I have a certain appreciation for the generative process itself, and the mathematical ideas behind modern generative models. This enhances my appreciation of the output.

Obviously, I get different feelings when encountering AI slop where in places where I used to encounter people. It's not all good. But it's not all bad, either, and we have to come to terms with the near future.

It's also the feeling I have, opus is not a ground-breaking model by any means.

However, Opus 4.5 is incredible when you give it everything it needs, a direction, what you have versus what you want and it will make it work, really, it will work. The code might me ugly, undesirable, would only work for that one condition, but with futher prompting you can evolve it and produce something that you can be proud of.

Opus is only as good as the user and the tools the user gives to it. Hmm, that's starting to sound kind-of... human...

Opus can produce beatiful code. It can outcode a good programmer. But getting it to do this reliably is something I've gotten better at over the last year; it's a skill that took quite a bit of practice.

I now write very long specifications and this helps. I haven't figured out a bulletproof workflow, I think that will take years. But I often get just amazing code out of it.

there is a big difference between a good programmer and a programmer that gives a shit so I disagree, opus can not come close to the code quality that someone can create and at that point it is the person behind the wheel that is causing the good quality to manifest rather than the AI randomly stumbling upon it.

I'm kind of surprised how many people are okay with deploying code that hasn't been audited.

I read If Anyone Builds It Everyone Dies over the break. The basic premise was that we can't "align" AI so when we turn it loose in an agent loop what it produces isn't necessarily what we want. It may be on the surface, to appease us and pass a cursory inspection, but it could embed other stuff according to other goals.

On the whole, I found it a little silly and implausible, but I'm second guessing parts of that response now that I'm seeing more people (this post, the Gas Town thing on the front page earlier) go all-in on vibe coding. There is likely to be a large body of running software out there that will be created by agents and never inspected by humans.

I think a more plausible failure mode in the near future (next year or two) is something more like a "worm". Someone building an agent with the explicit instructions to try to replicate itself. Opus 4.5 and GPT 5.2 are good enough that in an agent loop they could pretty thoroughly investigate any system they land on, and try to use a few ways to propagate their agent wrapper.

Perhaps our only saving grace is that many LLMs at varying levels of "dumbness" exist.

Is it possible to create an obfuscated quine that exhibits stable detection-avoiding behavior on every frontier model simultaneously, as well as on an old-school classifier and/or GPT-3 era LLM fine-tuned just for worm detection? One incapable of even thinking about what it's seeing, and being persuaded to follow its subtle propagation logic? I'm not sure that the answer is yes.

The larger issue to me is less that an LLM can propagate in generated code undetected, but rather that an attacker's generated code may soon be able to execute a level of hyper-customized spear-phishing-assisted attack at scale, targeting sites without large security teams - and that it will be hitting unintentional security flaws introduced by those smaller companies' vibe code. Who needs a worm when you have the resources of a state-level attacker at your fingertips, and numerous ways to monetize? The balance of power is shifting tremendously towards black hats, IMO.

There's a really interesting story I read somewhere about some application which used neural nets to optimize for a goal (this was a while ago, it could have been merkel trees or something, who knows, not super important)

And everything worked really well until they switched chip set.

At which point the same model failed entirely. Upon inspection it turned out the AI model had learned that overloading particular registers would cause such an electrical charge buildup that transistors on other pathways would be flipped.

And it was doing this in a coordinated manner in order to get the results it wanted lol.

I can't find any references in my very cursory searches, but your comment reminded me of the story

Why think about nefarious intent instead of just user error? In this case LLM error instead of programmer error.

Most RCEs, 0-days, and whatnots are not due to the NSA hiding behind the "Jia Tan" pseudo to try to backdoor all the SSH servers on all the systemd [1] Linuxes in the world: they're just programmer errors.

I think accidental security holes with LLMs are way, way, way more likely than actual malicious attempts.

And with the amount of code spoutted by LLMs, it is indeed --and the lack of audit is-- an issue.

[1] I know, I know: it's totally unrelated to systemd. Yet only systems using systemd would have been pwned. If you're pro-systemd you've got your point of view on this but I've got mine and you won't change my mind so don't bother.

I have a different concern: the SOTA products are expensive and get dumbed down on busy times. My personal strategy has been to be a late follower, where I adopt new AI tools when the competition has caught up with the previous SOTA, and now there are many tools that are cost effective and equally good.

Can't wait for when the competition catches up with Claude Code, especially the open source/weights Chinese alternatives :)

I really wonder what means for software moving forward. In the last few months I've used Claude Code to build personalized versions of Superwhisper (voice-to-text), CleanShot X (screenshot and image markup), and TextSniper (image to text). The only cost was some time and my $20/month subscription.

> I really wonder what means for software moving forward.

It means that it is going to be as easy to create software as it is to create a post on TikTok, and making your software commercially successful will be basically the same task (with the same uncontrollable dynamics) as whether or not your TikTok post goes viral.

I second this article - I built twelve iOS/Mac apps in two weeks with Opus 4.5 - four of them are already in the App Store - I’m a Rails Engineer and never had the time to learn Swift but man does Opus 4.5 make that not even matter - it even handles entitlements, logo & splash screen generation, refactors to remove dead code, edge case assent and hardening, Multiplatform app design, and more - I’m yet to run into a use case it can’t handle for most general use cases - that said, I have found some common mistakes it makes (by common I mean almost every time); puts iOS line list line items in buttons making them blue when they should not be, doesn’t set defaults for new data structure variables which crashes the app when changing the data structure after the fact, design consistent after the first shot (minor things like white background instead of grey background like all the other screens already, etc) - the one thing that i know it cant do well (and no other model that I know of can do this well either) is ASTM bi-directional communications (we work with pathology analysers that use this 1995 frame-based communication standard), even when you load it up with the spec and supporting docs - I suspect this is due to a dirty of available codebases that tackle this problem due to its niche and generally proprietary nature…

Are there a lot of manual steps in managing an xcode project? E.g. does it say "now go into xcode and change this setting" instead of changing the setting directly? Or are you using a tool like xcodegen?

Very few - the only manual things I do are;
- clicking the distribute button to push the bundle to the App Store
- filling in the compliance survey and App Store listing content
- linking some components together e.g. for creating a VPN installer and tunnel i had to click some things in the Xcode UI
I automate as much as possible;
-“create 12 app icons for this in SVG and present them to me in a HTML page so I can choose one and then use that for the app icon and splash screen”
- “create a demo mode toggle in settings and populate the app with fake data and then open up simulators for the correct image dimensions for the App Store listing so I can crate screenshots”
- sometimes it tell me I have to other things like set up the entitlements to which is say “no - you do it and don’t forget to fill in the description that gets shown to the user so the feature actually works”
I knew very little about Swift or Xcode profile to this and TBH I still don’t know that much about it, but I’m experienced enough to know when I’m being fed something that doesn’t look or feel right programmatically or architecturally.

how did you use Opus to build the apps? I tried using Claude Code ~6 months ago to build an iOS app and I was not that impressed with the results, especially compared to this blog post, where the apps look polished and very professional.

My biggest issue was limitations around how Claude Code could change Xcode settings and verify design elements in the simulator.

Opus 4.5 got released ~3 months ago - Claude Code started using it automatically (for me anyway) - I also tried iOS prior to that and had a similar experience to you

I used it with gemini 3 in tandem to build an app to simulate thermal bridges because I want to insulate a house. I explored this in various directions and there are some functionalities not completed or sound, but the main part is good and tested against ISO/DIN test cases for this kind of problem.
You can try it here, although the numeric simulations take quite a while in the cloud app

Disclaimer: I'm not a programmer or software engineer. I have a background in physics and understand some scripting in python and basic git. The code is messy at the moment because I explored/am still exploring to port it to another framework/language

So much of the conversation is around these models replacing software engineers. But the use cases described in the article sound like pretty compelling business opportunities; if the custom apps he built for his wife's business have been useful, probably there are lots of businesses that would pay for the service he just provided his wife. Small, custom apps can be made way more cheaply now, so Jeven's paradox says that demand should go up. I think it will.

I would love to hear from some freelance programmers how LLMs have changed their work in the last two years.

One problem with the idea of making businesses out of this kind of application is actually mentioned in passing in the article

"I decided to make up for my dereliction of duties by building her another app for her sign business that would make her life just a bit more delightful - and eliminate two other apps she is currently paying for"

OP used Opus to re-write existing applications that his wife was paying for. So now any time you make a commercial app and try to sell it, you're up against everyone with access to Opus or similar tooling who can replicate your application, exactly to their own specifications.

so everybody is making their own apps for their specific problem? Sounds as it will get a mess in the end. So maybe it will be more about ideas and concepts and not so much about know how to code.

Yep vast numbers of personalized apps seems like it would end up being pretty messy. I think the challenge of betting on ideas and concepts is that once you've published something, someone else can take the idea and replicate it easily and cheaply, so it'll be harder to monetize unless you can come up with something that's hard to replicate.

I think you're misunderstanding my point. If you can crank out a custom app this quickly, you don't make a commercial app and then try to sell it on an app store. Customers pay you to make apps for their specific usecase. One app, one customer. And if a week later they want some new features, they pay you (or another freelancer) to add it.

Put another way, we programmers have the luxury of being able to write custom scripts and apps for ourselves. Now that these things are getting way cheaper to build, there should be a growing market that makes them available to more people.

Why do they pay you though, why not just do it themselves? With improving models and surrounding tooling the barrier to creating apps is lowered, and it's easier for a user just to create their own app, no 3rd party person needed.

I switched my subscription from Claude to ChatGPT around 5.0 when SOTA was Sonnet 4.5 and found GPT-5-high (and now 5.2-high) so incredibly good, I could never imagine Opus is on its level. I give gpt-5.2-high a spec, it works for 20 minutes and the result is almost perfect and tested. I very rarely have to make changes.

It never duplicates code, implements something again and leaves the old code around, breaks my convention, hallucinates, or tells me it’s done when the code doesn’t even compile, which sonnet 4.5 and Opus 4.1 did all the time

I’m wondering if this had changed with Opus 4.5 since so many people are raving about it now. What’s your experience?

Claude - fast, to the point but maybe only 85% - 90% there and needs closer observation while it works

GPT-x-high (or xhigh) - you tell it what to do, it will work slowly but precise and the solution is exactly what you want. 98% there, needs no supervision

Anthropic dropped out of the general "AGI" race and seems to be purely focused on coding, maybe racing to get the first "automated machine learning programmer". Whatever the case, it seems to be paying (coding) dividends to just be focusing on coding.

That final line: "Disclaimer: This post was written by a human and edited for spelling, grammer by Haiku 4.5"

Yeah, GRAMMAR

For all the wonderment of the article, tripping up on a penultimate word that was supposedly checked by AI suddenly calls into question everything that went before...

> I don’t know if I feel exhilarated by what I can now build in a matter of hours, or depressed because the thing I’ve spent my life learning to do is now trivial for a computer. Both are true.

I'm not so sure... I mean it's true that regardless of if you are a beginner, junior, or 'senior', if you say "build me an instagram clone" Opus 4.5 will probably do a decent job. I think the skills go still in understanding architecture, and just knowing where pitfalls and problems can arise, or even making some important 'abstraction cut' prompts. I think applications can still grow to a point where you still need to prompt at specific domains only, or the model will fail to do everything you want it to - especially if you give it massive 'fix this this this anad that too' prompts

After reading that article, I see at least one thing that Opus 4.5 is clearly not going to change.

There is no fixed truth regarding what an "app" is, does, or looks like. Let alone the device it runs on or the technology it uses.

But to an LLM, there are only fixed truths (and in my experience, only three or four possible families of design for an application).

Opus 4.5 produces correct code more often, but when the human at the keyboard is trying to avoid making any engineering decisions, the code will continue to be boring.

Don't want to discredit Opus at all, it's easy at directed tasks but it's not the silver bullet yet.

It is best in its class, but trips up frequently with complicated engineering tasks involving dynamic variables. Think: Browser page loading, designing for a system where it will "forget" to account for race conditions, etc.

Still, this gets me very excited for the next generation of models from Anthropic for heavy tasks.

I’ve been saying this a countless time, LLM are great to build toy and experimental projects.

I’m not shaming but I personally need to know if my sentiment is correct or not or I just don’t know how to use LLMs

Can vibe coder gurus create operating system from scratch that competes with Linux and make it generate code that basically isn’t Linux since LLM are trained on said the source code …

Also all this on $20 plan. Free and self host solution will be best

In fact, like the author of the comment said, can just generated toys and experimental projects. I'm all in for experiments and exploring ideas, but I have yet to see a great product all vibe coded. All I see is a constand decline in software quality

> Can vibe coder gurus create operating system from scratch that competes with Linux and make it generate code that basically isn’t Linux since LLM are trained on said the source code …

No.

Vibe-coding, in the original sense where you don't bother with code reviews, the code quality and speed are both insufficient for that.

I experimented with them just before Christmas. I do not think my experiments were fully representative of the entire range of tasks needed for replacing Linux: Having them create some web apps, python scripts, a video game, a toy programming language, all beat my expectations given the METR study. While one flaw with the METR study is the small number of data points at current 50% successful task length, I take my success as evidence I've been throwing easy tasks at the LLM, not that the LLM is as good as it looks like to me.

However, assume for the moment that they were representative tasks:

For quality, what I saw just before Christmas was the equivalent of someone with a few years' experience under their belt, the kind of person who is just about to stop being a junior and get a pay rise. For speed, $20 of Claude Code will get you around 10 sprints' equivalent to that level of human's output.

"Junior about to get a pay rise" isn't high enough quality to let loose unchecked on a project that could compete with Linux, and even if it was, 10 sprints/month is too slow. Even if you increase the spend on LLMs to match the cost of a typical US junior developer, you're getting an army of 1500 full-time (40h/week) juniors, and Linux is, what, some 50-100 million developer-hours, so it would still take something like 16-32 years of calendar time (or, equivalently, order-of 1.2-2.5 million dollars) even if you could perfectly manage all those agents.

If you just vibe code, you get some millions of dollars worth of junior grade technical debt. There's cases where this is fine, an entire operating system isn't one of them.

> Also all this on $20 plan. Free and self host solution will be best

IMO unlikely, but not impossible.

A box with 10x the resources of your personal computer may be c. 10x the price, give or take.

While electricity is negligible (which today, hah!): If any given person is using that server only during a normal 40 hour work week, that's 25% utilisation rate, therefore if it can be rented out to people in other timezones or where the weekend is different, the effective cost for that 10x server is only 2.5x.

When electricity price is a major part of the cost, and electricity prices vary a lot from one place to another, then it can be worth remote-hosting even when you're the only user.

That said, energy efficiency of compute is still improving, albeit not as rapidly as Moore's Law used to, and if this trend continues then it's plausible that we get performance equivalent to current SOTA hosted models running on high-end smartphones by 2032. Assuming WW3 doesn't break out between the US and China after the latter tries to take Taiwan and all their chip factories, or whatever

Consider your own emotions and the bias you have against it. If it is actually able to do the things it is hyped up to be, what does that mean for you, your job, and your career? Can you really extract those emotions from how you're approaching the situation? That tiniest bit of fear in your gut might be coloring your approach here. You want a new operating system not based on Linux, that competes with it, because if it is based on Linux, it's in the training data, which means it's cheating?

Jrifjxgwyenf! A hammer is a really bad screwdriver. My car is really bad at refrigerating food. If you ask for something outside its training data, it doesn't do a very good job. So don't do that! All of the code on the Internet is a pretty big dataset though, so maybe Claude could do an operating system that isn't Linux that competes with it by laundering the FreeBSD kernel source through the training process.

And you're barely even willing to invest any money into this? The first Apple computer cost $4,000 or so. You want the bleeding edge of technology delivered to the smartphone in your hand, for $20, or else it's a complete failure? Buddy, your sentiment isn't the issue, it's your attitude.

I'm not here spouting ridiculous claims like AI is going to cure all of the different kinds of cancer by the end of 2027, I just want to say that endlessly contrarian naysayers are as equally borish as the syncophantic hype AIs they're opposing.

Yep, I literally built this last night with Opus 4.5 after my wife and I challenged each other to a typing competition. I gave it direction and feedback but it wrote all the actual code. Wasn't a one shot (maybe 3-4 shot) but didn't really have to think about it all that hard.

With another more substantial personal project (Eurorack module firmware, almost ready to release), I set up Claude Code to act as a design assistant, where I'd give it feedback on current implementation, and it would go through several rounds of design/review/design/review until I honed it down. It had several good ideas that I wouldn't have thought of otherwise (or at least would have taken me much longer to do).

Really excited to do some other projects after this one is done.

Yeah Opus 4.5 is a massive step change in my experience. I feel like I’m working with a peer, not a junior I’m having to direct. I can give it highly ambiguous and poorly specified tasks and it… just does it.

I will note that my experience varies slightly by language though. I’ve found it’s not as good at typescript.

It’s also way better than I am at finding bits of code for reuse. I tell it, “I think I wrote this thing a while back, but it may never have been merged, so you may need to search git history.” And presto, it finds it.

Reading this blog post makes me wanna rethink my career,
Opus 4.5 is really good I was recently working on solving my own problem by developing a software solution and let me tell you it was really good at it,

If I had done the same thing Pre LLM era it would have taken me months

I'll argue many of his cases are things that are straightforward except for the boilerplate that surrounds them which are often emotionally difficult or prone to rabbit holes.

Like that first one where he writes a right-click handler, off the top of my head I have no idea how I would do that, I could see it taking a few hours to just set up a dev environment, and I would probably overthink the research. I was working on something where Junie suggested I write a browser extension for Firefox and I was initially intimidated at the thought but it banged out something in just a few minutes that basically worked after the second prompt.

Similarly the Facebook autoposter is completely straightforward to code but it can be so emotionally exhausting to fight with authentication APIs, a big part of the coding agent story isn't just that it saves you time but that they can be strong when you are emotionally weak.

The one which seems the hardest is the one that does the routing and travel time estimation which I'd imagine is calling out to some API or library. I used to work at a place that did sales territory optimization and we had one product that would help work out routes for sales and service people who travel from customer to customer and we had a specialist code that stuff in C++ and he had a very different viewpoint than me, he was good at what he did and could get that kind of code to run fast but I wouldn't have trusted him to even look at applications code.

Sonnet 4.5 did it for me. Cant imagine coding without it now, and if you look at my comments from three months ago, you'll see I'm eating crow now. I easily hit >10x productivity with Sonnet 4.5 and Opus. I use Opus for my industry C and math work and Sonnet 4.5 for my swiftui side project.

I think the gap between Sonnet 4.5 and Opus is pretty small, compared to the absolute chasm between like gpt-4.1, grok, etc. vs Sonnet.

Despite the abuse of quotation marks in the screenshot at the top of this link, Dario Amodei did not in fact say those words or any other words with the same meaning.

Yes, unfortunate that people keep perpetuating that misquote. What he actually said was "we are not far from the world—I think we’ll be there in three to six months—where AI is writing 90 percent of the code."

As long as you give it deterministic goals / test criteria (compiles, lints, tests, E2E tests, achieve 100% parity with existing solution etc) it will brute force its way to a solution. Codex will work for hours/days, even weeks sometimes, until it has finished. A person would never work this way, but since this just runs in the background, there’s no issue with this approach except if you need it fast.

No, it might figure out the solution but even after many days there's no assurance that it won't get stuck making the same mistakes over and over again, never getting closer to a solution. I've seen this many times.

Getting in a loop does still happen, yes. If you run codex in tmux and let another agent just occasionally check on progress, it can be prevented. That’s not even expensive - checking every 30 minutes suffices. The watchdog agent can then press Esc in tmux and send a message, maybe do some research to get it unstuck etc

Neither have I, personally, but I’ve seen reports this can happen on very hard problems, where the goal just cannot be reached from a local optimum. Getting unstuck by trying something new is something a watchdog agent could prompt it.

I assume, the purpose would be to learn how it's done.
There's no place for this when you vibecode.
And if not learning, what's the point of implementing something that already exists?

When I'm dying of dehydration because humanity has depleted all fresh water deposits, I'll think of you and your stupid NES emulator which is just an LLM-produced copy of many ones that had already existed.

The majority of open source software development is "implementing something that already exists", but with improvements, such as for specific use cases and constraints (like the original NES emulator) or by making it more performant. That's how the ecosystem mutates and grows, and it's worked well for decades.

What about Sonnet 4.5? I used both Opus and Sonnet on Claude.ai and found sonnet much better at following instructions and doing exactly what was asked.

(it was for single html/js PWA to measure and track heart rate)

Opus seems to go less deep, does it's own things, do not follow instructions exactly EVEN IF I WROTE ALL CAPS. With Sonnet 4.5 I can understand everything author is saying. May be Opus is optimised for Claude code and Sonnet works best on Web.

I guess the best analogy I can think of is the transition from writing assembly language and the introduction of compilers. Now, (almost) no one knows, or cares, what comes out of the compiler. We just assume it is optimized and that it represents the source code faithfully. Seems like code might go that way too and people will focus on the right prompts and can simply assume the code will be correct.

Does a system being deterministic really matter if it's complex enough you can't predict it? How many stories are there about 'you need to do it in this specific way, and not this other specific way, to get 500x better codegen'?

Does anyone have a boring, multi-hour-long coding session with an agent that they've recorded and put on Vimeo or something?

As many other commentators have said, individual results vary extremely widely. I'd love to be able to look at the footage of either someone who claims a 10x productivity increase, or someone who claims no productivity increase, to see what's happening.

I can't quite figure out what sort of irony the blurb at the bottom of the post is. (I'm unsure if it was intentional snark, a human typo, or an inadvertent demonstration of Haiku not being well suited for spelling and grammar checks), but either way I got a chuckle:

> Disclaimer: This post was written by a human and edited for spelling, grammer by Haiku 4.5

So I decided to try the revered hands-off approach and have Claude Code create me a small tool in JS for *.dylib bundle consolidation on macOS.

I have used AskUserQuestionTool to complete my initial spec. And then Opus 4.5 created the tool according to that extensive and detailed spec.

It appeared to work out of the box.

Boy how horrific was the code. Unnecessary recursions, unused variables, data structures being built with no usage, deep branch nesting and weird code that is hard to understand because of how illogical it is.

And yes, it was broken on many levels and did not and could not do the job properly.

I then had to rewrite the tool from scratch and overall I have definitely spent more time spec'ing and understanding Claude code than if I have just written this tool from scratch initially.

That’s the opposite of my experience. Weird. But I’m also not the kind of person who gets hung up on whether someone used a loop or recursion or if their methods are five times as long as what I would’ve done myself unless there is a performance impact that matters to me as a user. But I’m also the kind of person who doesn’t get paid by the hour to write programs. I use programs in the service of other paid work.

Yes, this experience is unlike most people. Perhaps the problem is that most people are satisfied by the appearance of a working app despite it not working at all. Say, the first tool I was doing, did actually not recurse into subdirs with dylibs which made it useless.

SWE jobs are in fact, not safe, if vaguely defined specifications can be translated into functioning applications. I don't think agents are good enough to do that in larger applications yet, but it is something to consider.

Depends on the software. IMO, development speed will increase, but humans will continue to be the limiting factor, so we are safe. Our jobs, however, are changing and will continue to.

I was not expecting a couple of new apps being built, when the premise of the blog post talks about replacing "mid level engineers"

the thing about being an engineer at commercial capacity is "maintaining/enhancing an existing program/software system that has been developed over years by multiple people(including those who already left) and do it in a way that does not cause any outages/bugs/break existing functionality.

while the blog post mentions about the ability of using AI to generate new applications, but it does not talk about maintaining one over a longer period of time. for that, you would need real users, real constraints, and real feature requests which preferably pay you so you can priortize them.

I would love to see such blog posts where for example, a PM is able to add features for a period of one month without breaking the production, but it would be a very costly experiment.

It worries me that the best models, the ones that can one-shot apps and such, are all non-free and owned by companies who can't be trusted to have end-users' best interests at heart. It would be greatly reassuring to see a self-hostable model that can compete with Opus 4.5 and Gemini 3 at such coding tasks.

I have used Claude Code for a variety of hobby projects. I am truly astounded at its capabilities.

If you tell it to use linters and other kinds of code analysis tools it takes it to the next level. Ruff for Python or Clippy for Rust for example. The LLM makes so much code so fast and then passes it through these tools and actually understands what the tools say and it goes and makes the changes. I have created a whole tool chain that I put in a pre commit text file in my repos and tell the LLM something like "Look in this text file and use every tool you see listed to improve code quality".

That being said, I doubt it can turn a non-dev into a dev still, it just makes competent devs way better still.

I still need to be able to understand what it is doing and what the tools are for to even have a chance to give it the guardrails it should follow.

The worst part about this is that you can't know anymore whether the software you trustingly install on your hardware is clean or if it was coded by a misaligned coding model with a secret goal that it has hidden from its prompt engineer and from you.

This could pretty much be the beginning of the end of everything, if misaligned models wanted to they could install killswitches everywhere. And you can't trust security updates either so you are even more vulnerable to external exploits.

It's really scary, I fear the future, it's going to be so bad. It's best to not touch AI at all and stay hidden from it as long as possible to survive the catastrophe or not be a helping part of it. Don't turn your devices into a node of a clandestine bot net that is only waiting to conspire against us.

I have to many machines standing around that are currently not powered on or are running somewhat airgapped with old software from around debian 8 and 9, so I guess they will be a safe haven once the AI overlords take over

Claude Code is very good; good enough that I upgraded to the Max plan this week. However, it has a long way to go. It's great at one-shotting (with iterations) most ideas. However, it doesn't do as well when the task is complicated in an existing codebase. This weekend I migrated the backend for the SaaS I am building from Python to .NET Core. It did the migration but completely missed the conventions that the frontend was using to call the backend. While the converion itself went OK, every user journey was broken. I am still manually testing every code path and feeding in the errors to get Claude to fix it. My instructions were fairly comprehensive but Claude still missed most of it. My fault that I didn't generate tests first, but after this migration that's my first task.

This resonates with my experience in codex 5.2, at least directionally. I'm pretty persnickety about code itself, so I'm not to the point where I'll just let it rip. But in the last month or two things have gone from "I'll ask on the web interface and maybe copy some code into the project", to trusting the agent and getting a reasonable starting point about half the time.

> because models like to write code WAY more than they like to delete it

Yeah, this is the big one. I haven't figured it out either. New or changing requirements are almost always implemented a flurry of if/else branches all over the place, rather than taking the time for a step back and a reimagining of a cohesive integration of old and new. I've had occasional luck asking for this explicitly, but far more frequently they'll respond with recommendations that are far more mechanical, e.g. "you could extract a function for these two lines of code that you repeat twice", not architectural, in nature. (I still find pasting a bunch of files into the chat interface and iterating on refinements conversationally to be faster and produce better results).

That said, I'm convinced now that it'll get there sooner or later. At that point, I really don't know what purpose SWEs will serve. For a while we might serve as go-betweens between the coding agent and PMs, but LLMs are already way better at translating from tech jargon to human, so I can't imagine it would be long before product starts bypassing us and talking directly to the agents, who (err, which) can respond with various design alternatives, pros and cons of each, identify all the dependencies, possible compatibility concerns, alignment with future direction, migration time, compute cost, user education and adoption tracking, etc, all in real time in fluent PM-ese. IDK what value I add to that equation.

For the last year or so I figured we'd probably hit a wall before AI got to that point, but over the last month or so, I'm convinced it's only a matter of time.

It is very funny to start your article off with a bunch of breathless headlines about agents replacing human coders by the end of 2025, none of which happened, then the rest of the article is "okay but this time for real, an agent really WILL replace human coders."

I agree with the OP that I can get LLM's to do things now that I wouldn't even attempt a year ago, but I feel it has more to do with my own experience using LLM's (and the surrounding tools) than the actual models themselves.

I use copilot and change models often, and haven't really noticed any major differences between them, except some of the newer ones are very slow.

I generally feel the smaller and faster ones are more useful since they will let me discover problems with my prompt or context faster.

Maybe I'm simply not using LLM's in a way that lets the superiority of newer models reveal itself properly, but there is a huge financial incentive for LLM makers to pretend that their model has game-changing "special sauce" even if it doesn't.

> Why does a human need to read this code at all? I use a custom agent in VS Code that tells Opus to write code for LLMs, not humans. Think about it—why optimize for human readability when the AI is doing all the work and will explain things to you when you ask?

> What you don’t need: variable names, formatting, comments meant for humans, or patterns designed to spare your brain.

> What you do need: simple entry points, explicit code with fewer abstractions, minimal coupling, and linear control flow.

All great until the code in production pushed by Opus 314.15 breaks and Opus 602.21, despite it's many tries, can't fix it and ends it with "I apologize". That's when you need a developer who can be told "fix it". But what if all the developers then are "Opus 600+ certified" ai-native and are completely incapable of working without it's assistance? World powers decide to open the forbidden vault in the Arctic and despite many warnings on the chamber, decide to raise the foul-mouthed programmer-demon called Torvalds....

I pivoted into integrations in 2022. My day-to-day now is mostly in learning the undocumented quirks of other systems. I turn those into requirements, which I feed to the model du jour via GitHub Copilot Agents. Copilot creates PRs for me to review. I'd say it gets them right the vast majority of the time now.

Example: One of my customers (which I got by Reddit posts, cold calls, having a website, and eventually word of mouth) wanted to do something novel with a vendor in my niche. AI doesn't know how to build it because there's no documentation for the interfaces we needed to use.

The article’s central tension is real - Burke went from skeptic to believer by building four increasingly complex apps in rapid succession using Opus 4.5. But his evidence also reveals the limits of that belief.

Notice what he actually built: Windows utilities, a screen recorder, and two Firebase-backed CRUD apps for his wife’s business. These are real applications solving real problems, but they’re also the kinds of projects where you can throw away the code if something goes wrong. When he says “I don’t know how the code works” and “I’m maybe 80% confident these applications are bulletproof,” he’s admitting the core problem with the “AI replaces developers” narrative.

That 80% confidence matters. In your Splink work, you’re the sole frontend developer - you can’t deploy code you’re 80% confident about. You need to understand the implications of your architectural decisions, know where the edge cases are, and maintain the system when requirements change. Burke’s building throwaway prototypes for his wife’s yard sign business. You’re building production software that other people depend on.

His “LLM-first code” philosophy is interesting but backwards. He’s optimizing for AI regeneration rather than human maintenance because he assumes the AI will always be there to fix problems. But AI can’t tell you why a decision was made six months ago when business requirements shift. It can’t explain the constraints that led to a particular architecture. And it definitely can’t navigate political and organizational context when stakeholders disagree about priorities.

The Firebase examples are telling - he keeps emphasizing how well Opus knows the Firebase CLI, as if that proves general capability. But Firebase is extremely well-documented, widely-discussed training data. Try that same experiment with your company’s internal API or a niche library with poor documentation. The model won’t be nearly as capable.

What Burke actually demonstrated is that Opus 4.5 is an excellent pair programmer for prototyping with well-known tools. That’s legitimately valuable. But “pair programmer for prototyping” isn’t the same as “replacing developers.” It’s augmenting someone who already knows how to build software and can evaluate whether the generated code is good.

The most revealing line is at the end: “Just make sure you know where your API keys are.” He’s nervous about security because he doesn’t understand the code. That nervousness is appropriate - it’s the signal that tells you when you’ve crossed from useful tool into dangerous territory.

Are you using Claude Code? Because that might be the secret cause you're missing. With Claude Code I can instruct it to validate things after its done with code, and usually it finds that it goofed. I can also tell it to work on like five different things, and go "hey spin up some agents to work on this" and it will spawn 5 agents in parallel to work on said things.

I've basically ditched Groke et al and I refuse to give Sam Altman a penny.

For schema design phase I used web UI for all three.

Logical bug of using BIGSERIAL for tracking updates (generated at insert time, not commit time, so can be out of order) wouldn’t be caught by any number of iterations of Claude Code and would be found in production after weeks of debugging.

At this point having any LLM write code without giving it an environment that allows it to execute that code itself is like rolling a heavily-biased random number generator and hoping you get a useful result.

Things get so much more interesting when they're able to execute the code they are writing to see if it actually works.

So much this. Do we program by writing reams of code and never running the compiler until it's all written and then judging the programmer as terrible when it doesn't compile? Or do we write code by hand incrementally and compile and test as we go along? So why would do we think having the AI do that and fail is setting it up for success? If I wrote code on a whiteboard and was judged for making syntax errors, I'd never have gotten a job. Give the AI the tools it needs to succeed, just like you would for a human.

> Do we program by writing reams of code and never running the compiler until it's all written and then judging the programmer as terrible when it doesn't compile?

These are very simple utilities. I expect AI to be able to build them easily. Maybe in a few years it will be able to write a complete photo editor or CAD application from first principles.

It’s incredibly tiring to see this narrative peddled every damn day. I use opus 4.5 every day. It’s not much different than any previous models, still does dumb things all the time.

Same experience - I've had it fail at the same reasonably simple tasks I had opus 4 and sonnet 4.5 and sonnet 4 fail at when they aren't carefully guided and their work check and fixed...

For some reason Opus 4.5 is blowing up recently after having been released for weeks. I guess because holidays are over? Active agent users should have discovered this for a while.

I gave it a try, I asked to do a reddit like forum and it did pretty good but damn I quickly hit the daily limit of the $20 pro account, and it took 10% of the monthly just to do the setup and some basics. I knew LLM were expensive to run but I've never felt it directly. Even if the code is good it's kinda expensive for what you get.

Ho it was also quite funny it used the exact same color as hackernews and a similar layout.

IMO codex produces working code slowly, while Opus produces superficially working code quickly. I like using Opus to drive codex sessions and checking its output. Clawdbot is really good at that but a long running Claude Code session with codex as sub agents should work well also.

The above is for vibe coding; for taking the wheel, I can only use Opus because I suck at prompting codex (it needs very specific instructions), and codex is also way too slow for pair programming.

> I like using Opus to drive codex sessions and checking its output.

Why not the other way around? Have the quick brown fox churn out code, and have codex review it, guide changes, and loop?

I've actually gone one step further down the delegation. I use opus/gemini3 for plan, review, edit plan for a few steps. Then write it out to .md files. Then have GLM implement it (I got a cheap plan for like 28$ for a year on Christmas). Then have the code this produced reviewed and fixed if needed by opus. Final review by codex (for some reason it's very good at review, esp if you have solid checkboxes for it to check during review). Seems to work so far.

I agree, codex is great at reviewing as well. I think that’s because code is the ideal description of what we want to achieve, and codex is good (only) when it knows what must be achieved, as verbosely as possible.

Currently I don’t let GLM or Opus near my codebases unsupervised because I’m convinced that the better the foundation, the better the end result will be. Is the first draft not pretty crappy with GLM?

See also: a post from a couple days ago which came to the same conclusion that Opus 4.5 is an inflection point above Sonnet 4.5 despite that conclusion being counterintuitive: https://news.ycombinator.com/item?id=46495539

It's hard to say if Opus 4.5 itself will change everything given the cost/latency issues, but now that all the labs will have very good synthetic agentic data thanks to Opus 4.5, I will be very interested to see what the LLMs release this year will be able to do. A Sonnet 4.7 that can do agentic coding as well as Opus 4.5 but at Sonnet's speed/price would be the real gamechanger: with Claude Code on the $20/mo plan, you can barely do more than one or two prompts with Opus 4.5 per session.

I've only started but I mostly use Claude Code for building out code that has been done a million times. So its good at setting up a project to get all the boiler plate crap out of the way.

When you need to build out specific feature or logic, it can fail hard.
And the best is when you have something working, and it fixes something else and deletes the old code that was working, just in a different spot.

As impressive as Opus 4.5 is, it still fails in one situation that it assumes 0-index while the component it supposes to work with assume 1-index. It has access to the said information on disk, but just forgets to look into.

Opus 4.5 is incredible, it is the GPT-4 moment for coding because how honest and noticeable the capacity increase is. But still, it has blind spots just like human.

The main issue in this discussion is the word "replace" . People will come up with a bunch of examples where humans are still needed in SWE and can't be fully replaced, that is true. I think claiming that 100% of engineers would be replaced in 2026 is ridiculous.
But how about downsizing? Yeah that's quite probable.

Ok, if its almighty, then why is not the benchmarks at 100%? If you look at the individual issues, those are somewhat small and trivial changes in existing codebases.

Once you get your setup bulletproof such that you can have multiple agents running at the same time that can run unit tests and close their own loops things get even faster. However you accomplish that. Not as easy as it sounds mostly (and absurdly) due to port collision. E2E testing with playwright is another leap.

LLMS like Opus, Gemini 3, and GPT-5.2/5.1-Codex-max, are phenomenal for coding and have only very recently crossed that gap between being "eh" and being quite fantastic to let operate on their own agentically. The major trade-off being a fairly expensive cost. I ran up $200 per provider after running through 'pro' tier limits during a single week of hacking over the holidays.

Unfortunately, it's still surprisingly easy for these models to fall into really stupid maintainability traps.

For instance today, Opus adds a feature to the code that needs access to a db. It fails because the db (sqlite) is not local to the executable at runtime. Its solution is to create this 100 line function to resolve a relative path and deal with errors and variations.

I hit ESC and say "... just accept a flag for --localdb <file>". It responds with "oh, that's a much cleaner implementation. Good idea!". It then implements my approach and deletes all the hacks it had scattered about.

This... is why LLMs are still not Senior engineers. They do plainly stupid things. They're still absurdly powerful and helpful, but if you want maintainable code you really have to pay attention.

Another common failure is when context is polluted.

I asked Opus to implement a feature by looking up the spec. It looked up the wrong spec (a v2 api instead of a v3) -- I had only indicated "latest spec". It then did the classic LLM circular troubleshooting as we went in 4 loops trying to figure out why calculations were failing.

I killed the session, asked a fresh instance to "figure out why the calculation was failing" and it found it straight away. The previous instance would have gone in circles for eternity because its worldview had been polluted by assumptions made -- that could not be shaken.

This is a second way in which LLMs are rigid and robotic in their thinking and approach -- taking the wrong way even when directed not to. Further reading on 'debugging decay': https://arxiv.org/abs/2506.18403

All this said, the number of failure scenarios gets ever smaller. We've gone from "problem and hallucination every other code block" to "problem every 200-1000 code blocks".

They're now in the sweet spot of acting as a massive accelerator. If you're not using them, you'll simply deliver slower.

most of software engineering was rational, now it is becoming empirical

it is quite strange, you have to make it write the code in a way it can reason about it without it reading it, you also have to feel the code without reading all of it. like a blind man feeling the shape of an object; Shape from Darkness

you can ask opus to make a car, it will give you a car, then you ask it for navigation; no problem, it uses google maps works perfect

then you ask it to improve the breaks, and it will give internet to the tires and the break pedal, and the pedal will send a signal via ipv6 to the tires which will enable a very well designed local breaking system, why not, we already have internet for google maps.

i think the new software engineering is 10 times harder than the old one :)

This article is much better than hundred of similar articles "AI will change software engineering" because it have links to actual products created with said "AI". I can't say they are impressive, but definitely so for laypeople.

Read this article and ultrathink critically about it. Provide your perspective.

The article makes a strong experiential case for improved velocity in AI-assisted development but contains several argumentation weaknesses and conflations worth examining.

The projects described are legitimately non-trivial: Firebase backend integration, Facebook OAuth, iOS apps in Swift (a language the author doesn't know), GitHub Actions pipelines, scheduled cloud functions. Getting these working in hours rather than weeks represents a real capability shift. The author is honest about his limitations and uncertainties, particularly the security concerns.

Where the argument breaks down:

1. "Replace developers" vs "dramatically augment developers"
The author's own workflow contradicts the headline claim. He's still:

Making architectural decisions (choosing Firebase)
Handling errors Opus couldn't see (XAML via Visual Studio)
Writing custom prompts to shape output quality
Manually auditing security
Making product and UX decisions

This is developer work. The tool changed; the role didn't disappear.

2. The 80% security confidence undermines his thesis
He admits he's shipping apps with "80% confidence" in security and calls it "too damn low." This is the crux: the AI accelerated production but didn't replace the judgment required to responsibly ship production software. The velocity gain exposed a competence gap rather than closing it.

3. Sample bias in project selection

All examples are:

Greenfield (no existing codebase)
Single developer
Personal/family use
Standard patterns with excellent documentation (Firebase, SwiftUI, React Native)
No regulatory, compliance, or scale requirements
No team collaboration or handoff considerations

These constraints define a specific problem space where AI excels. Extrapolating to "developers are replaceable" ignores the majority of professional software work.

4. "Code doesn't need human readability" is underbaked
His argument is circular: "Why optimize for human readability when the AI is doing all the work?" But:

His 80% security confidence exists because he can't read the code
He had to use external tools (VS) when Opus couldn't diagnose errors
What happens when context windows are exceeded and the LLM loses track?
Model behavior changes between versions; human-readable code is version-agnostic

The custom prompt he shares actually encodes many good engineering practices (minimal coupling, explicit state, linear control flow) that benefit LLMs and humans. The "no comments needed" claim conflates what's optimal for LLM regeneration with what's optimal for debugging production issues at 3am.
What's actually being demonstrated

The honest version of this article would be: Opus 4.5 dramatically compresses the gap between "can write code" and "can ship a personal app" for a specific class of greenfield projects. That's genuinely transformative for hobbyists, indie developers, and people solving their own problems.
But that's different from "replacing developers." The article demonstrates a power tool; power tools don't eliminate tradespeople.

There's something eerily recursive about Opus 4.5’s sensible take calming the anxiety about Opus 4.5’s capabilities and impact. It's probably the right take, but I feel weird the most pragmatic response to this article is from said model.

The best is probably something like GLM 4.7/Minimax M2.1, and those are probably at most Sonnet 4 level, which is behind Opus 4.1, which is behind Sonnet 4.5, which is behind Opus 4.5 ;)

And honestly Opus 4.5 is a visible step change above previous Anthropic models.

Oh, of course not, you might need up to 100GB VRAM to have those models at decent speeds even just for low-quant versions.

And all the hype about Macs with unified memory is a bit dishonest because the actual generation speed will be very bad, especially if you fill the context.

One of the things that makes Opus 4.5 special in comparison to e.g. GPT 5.2 is the fact that it doesn't have to reason for multiple minutes to make some simple changes.

I started on the cheapest £15/mo "Pro" plan and it was great for home use when I'd do a bit of coding in the evenings only, but it wasn't really that usable with Opus--you can burn through your session allowance in a few minutes, but was fine with Sonnet. I used the PAYG option to add more, but cost me £200 in December, so I opted for the £90/mo "Max" plan which is great. I've used Opus 4.5 continuously and it's done great work.

I think when you look at it from the perspective of how much you get out of it compared with paying a human to do the same (including yourself), it is still very good value for money whether you use it for work or for your own projects. I do both. But when I look what I can now do for my own projects including open-source stuff, I'm very time-limited, and some of the things I want to do would take multiple years. Some of these tools can take that down to weeks, do I can do more with less, and from that perspective the cost is worth it.

A lot of the complaints about these tools seems to revolve around their current lack of ability to innovate for greenfield or overly complex tasks. I would agree with this assessment in their current state, but this sentiment of "I will only use AI coding tools when they can do 100% of my job" seems short-sighted.

The fact of the matter, in my experience, is that most of the day to day software tasks done by an individual developer are not greenfield, complex tasks. They're boring data-slinging or protocol wrangling. This sort of thing has been done a thousand times by developers everywhere, and frankly there's really no need to do the vast majority of this work again when the AIs have all been trained on this very data.

I have had great success using AIs as vast collections of lego blocks. I don't "vibe code", I "lego code", telling the AI the general shape and letting it assemble the pieces. Does it build garbage sometimes? Sure, but who doesn't from time to time? I'm experienced enough notice the garbage smell and take corrective action or toss it and try again. Could there be strange crevices in a lego-coded application that the AI doesn't quite have a piece for? Absolutely! Write that bit yourself and then get on with your day.

If the only thing you use these tools for is doing simple grunt-work tasks, they're still useful, and dismissing them is, in my opinion, a mistake.

The vast majority of engineers aren't refusing to use AI until it can do 100% of their job. They are just sick of being told it already can, when their direct experience contradicts that claim.

Are the LLMs in any way trained semantically or by hooks that you can plug in, say, Python docs? And if a new version of Python then gets released then the training data changes, etc

The question I keep asking myself is "how feasible will any of this be when the VC money runs out?" Right now tokens are crazy cheap. Will the continue to be?

Oh another run of new small apps. Why not unleash this oh so powerful tools not on a jira ticket written two years ago, targeting 3 different repos in an old legacy moloch, like actual work?

Did some of that today. Extracting logic from Helm templates that read like 2000s PHP and moving it to a nushell script rendering values. Took a lot of guidance both in terms of making it test its own code and architectural/style decisions and I also use Sonnet, but it got there.

I agree. Claude Code went from being slower than doing it myself to being on average faster, but also far less exhausting so I can do more things in general while it works.

Things are changing. Now everyone can build bespoke apps. Are these apps pushing the limits of technology? No! But they work for the very narrow and specific domain they where designed. And yes they do not scale and have as much bugs as your personal shell scripts. But they work.

But let's not compare these with something more advance - at least not yet. Maybe by end of this year?

We switched from Sonnet 4.5 to Opus 4.5 as our default coding agent recently and we pay the price for the switch (3x the cost) but as the OP said, it is quite frankly amazing. It does a pretty good job, especially, especially when your code and project is structured in a such a way that it helps the agent perform well. Anthropic released an entire video on the subject recently which aligns with my own observations as well.

Where it fails hard is in the more subtle areas of the code, like good design, best practices, good taste, dry, etc. We often need to prompt it to refactor things as the quick solution it decided to do is not in our best interest for the long run. It often ends in deep investigations about things which are trivially obvious. It is overfitted to use unix tools in their pure form as it fail to remember (even with prompting) that it should run `pnpm test:unit` instead `npx jest` - it gets it wrong every time.

But when it works - it is wonderful.

I think we are at the point where we are close to self-improving software and I don't mean this lightly.

It turns out the unix philosophy runs deep. We are right now working on ways to give our agents more shells and we are frankly a few iterations there. I am not sure what to expect after this but I think whatever it is, it will be interesting to see.

Unless you are going to be more specific, that criticism applies to all benchmarks that are connected to a positive gain, not just AI coding benchmarks.

If you can figure out how to create benchmarks that make sense, are reliable, correlate strongly to business goals, and don't get immediately saturated or contorted once known, you are well on your way to becoming a billionaire.

the author asks one interesting question and then glides right by it. If the agents only need their own code, what should that code look like? If all their learning has come from old human code, how will that change in the future as the ecosystem fills up with agent code?

People should finally understand that LLMs are a lossy database of PAST knowledge. Yes, if you throw a task at it that has been done tons of times before, it works. Which is not a surprise, because it takes minutes to Google and index multiple full implementations of "Tool that allows you to right-click on an image to convert it". Without LLM you could do the same: Just copy&paste the implementation of that from Microsoft Powertoys, for example.

What LLMs will NOT do however, is write or invent SOMETHING KNEW.

And parts of our industry still are about that: Writing Software that has NOT been written before.

If you hire junior developers to re-invent the wheels: Sure, you do not need them anymore.

But sooner or later you will run out of people who know how to invent NEW things.

So: This is one more of those posts that completely miss the point. "Oh wow, if I look up on Wikipedia how to make pancakes I suddenly can make and have pancakes!!!1". That always was possible. Yes, you now can even get an LLM to create you a pancake-machine. Great.

Most of the artists and designers I am friends with have lost their jobs by now. In a couple of years you will notice the LLMs no longer have new styles to copy from.

I am all for the "remix culture". But don't claim to be an original artist, if you are just doing a remix. And LLM source code output are remixes, not original art.

Yea, my issue with Opus 4.5 is it's the first model that's good enough that I'm starting to feel myself slip into laziness. I catch myself reviewing its output less rigorously than I had with previous AI coding assistants.

As a side project / experiment, I designed a language spec and am using (mostly) Opus 4.5 to write a transpiler (language transpiles to C) for it. Parser was no problem (I used s-expressions for a reason). The type checker and transpiler itself have been a slog - I think I'm finding the limits of Opus :D. It particularly struggles with multi-module support. Though, some of this is probably mistakes made by me while playing architect and iterating with Claude - I haven't written a compiler since my senior year compiler design course 20+ years ago. Someone who does this for a living would probably have an easier time of it.

But for the CRUD stuff my day job has me doing? Pffttt... it's great.

this is just optimizing for token windows. flat code = less context. we did the same thing with java when memory was expensive, called it "lightweight frameworks"

I'm always surprised to never see any comments in those discussions from people who just like coding, learning, solving problems… I mean, it's amazing that LLMs can build an image converter or whatever you dream of, in a language you don't know, in a field you are not familiar with, in 1 hour, for 30 cents… I'm sure your boss and shareholders love it. But where is the fun in that? For me it kills any interest in doing what I'm doing. I'm lucky enough to work in a place where using LLMs is not mandatory (yet), I don't know how people can make it through the day just writing prompts and reviewing AI slop.

It’s a bit strange how anecdotes have become acceptable fuel for 1000 comment technical debates.

I’ve always liked the quote that sufficiently advanced tech looks like magic, but its mistake to assume that things that look like magic also share other properties of magic. They don’t.

Software engineering spans over several distinct skills: forming logical plans, encoding them in machine executable form(coding), making them readable and expandable by other humans(to scale engineering), and constantly navigating tradeoffs like performance, maintainability and org constraints as requirements evolve.

LLMs are very good at some of these, especially instruction following within well known methodologies. That’s real progress, and it will be productized sooner than later, having concrete usecases, ROI and clearly defined end user.

Yet, I’d love to see less discussion driven by anecdotes and more discussion about productizing these tools, where they work, usage methodologies, missing tooling, KPIs for specific usecases. And don’t get me started on current evaluation frameworks, they become increasingly irrelevant once models are good enough at instruction following.

> It’s a bit strange how anecdotes have become acceptable fuel for 1000 comment technical debates.

Progress is so fast right now anecdotes are sometimes more interesting than proper benchmarks. "Wow it can do impressive thing X" is more interesting to me than a 4% gain on SWE Verified Bench.

In early days of a startup "this one user is spending 50 hours/week in our tool" is sometimes more interesting than global metrics like average time in app. In the early/fast days, the potential is more interesting than the current state. There's work to be done to make that one user's experience apply to everyone, but knowing that it can work is still a huge milestone.

At this point I believe the anecdotes more than benchmarks, cause I know the LLM devs train the damn things on the benchmarks.

A benchmark? probably was gamed. A guy made an app to right click and convert an image? prolly true, have to assume it may have a lot of issues but prima facie I just make a mental note that this is possible now.

> It’s a bit strange how anecdotes have become acceptable fuel for 1000 comment technical debates.

It's a very subjective topic. Some people claim it increases their productivity 100x. Some think it is not fit for purpose. Some think it is dangerous. Some think it's unethical.

Weirdly those could all be true at the same time, and where you land on this is purely a matter of importance to the user.

> Yet, I’d love to see less discussion driven by anecdotes and more discussion about productizing these tools, where they work, usage methodologies, missing tooling, KPIs for specific usecases. And don’t get me started on current evaluation frameworks, they become increasingly irrelevant once models are good enough at instruction following.

I agree. I've said earlier that I just want these AI companies to release an 8-hour video of one person using these tools to build something extremely challenging. Start to finish. How do they use it, how does the tool really work. What's the best approaches. I am not interested in 5-minute demo videos producing react fluff or any other boiler plate machine.

I think the open secret is that these 'models' are not much faster than a truly competent engineer. And what's dangerous is that it is empowering people to 'write' software they don't understand. We're starting to see the AI companies reflect this in their marketing, saying tech debt is a good thing if you move fast enough....

This must be why my 8-core corporate PC can barely run teams and a web browser in 2026.

How many 1+ hour videos of someone building with AI tools have you sought out and watched? Those definitely exist, it sounds like you didn't go seeking them out or watch them because even with 7 less hours you'd better understand where they add value enough to believe they can help with challenging projects.

So why should anybody produce an 8 hour video for you when you wouldn't watch it? Let's be real. You would not watch that video.

In my opinion most of the people who refuse to believe AI can help them while work with software are just incurious/archetypical late adopters.

If you've ever interacted with these kinds of users, even though they might ask for specs/more resources/more demos and case studies or maturity or whatever, you know that really they are just change-resistant and will probably continue to be as as long as they can get away with it being framed as skepticism rather than simply being out of touch.

I don't mean that in a moralizing sense btw - I think it is a natural part of aging and gaining experience, shifting priorities, being burned too many times. A lot of business owners 30 years ago probably truly didn't need to "learn that email thing", because learning it would have required more of a time investment than it would yield, due to being later in their career with less time for it to payoff, and having already built skills/habits/processes around physical mail that would become obsolete with virtual mail. But a lot of them did end up learning that email thing 5, 10, whatever years later when the benefits were more obvious and the rest of the world had already reoriented itself around email. Even if they still didn't want to, they'd risk looking like a fossil/"too old" to adapt to changes in the workplace if they didn't just do it.

That's why you're seeing so many directors/middle managers doing all these though leader posts about AI recently. Lots of these guys 1-2 years ago were either saying AI is spicy autocomplete or "our OKR this quarter is to Do AI Things". Now they can't get away with phoning it in anymore and need to prove to their boss that they are capable of understanding and using AI, the same way they had to prove that they understood cloud by writing about kubernetes or microservices or whatever 5-10 years ago.

> In my opinion most of the people who refuse to believe AI can help them while work with software are just incurious/archetypical late adopters.

The biggest blocker I see to having AI help us be more productive is that it transforms how the day to day operations work.

Right now there is some balance in the pipeline of receiving change requests/enhancements, documenting them, estimating implementation time, analyzing cost and benefits, breaking out the feature into discrete stories, having the teams review the stories and 'vote' on a point sizing, planning on when each feature should be completed given the teams current capacity and committing to the releases (PI Planning), and then actually implementing the changes being requested.

However if I can take a code base and enter in a high level feature request from the stakeholders and then hold hands with Kiro to produce a functioning implementation in a day, then the majority of those steps above are just wasting time. Spending a few hundred man-hours to prepare for work that takes a few hundred man-hours might be reasonable, but doing that same prep work for a task that takes 8 man-hours isn't.

And we can't shift to that faster workflow without significant changes to entire software pipeline. The entire PMO team dedicated to reporting when things will be done shifts if that 'thing' is done before the report to the PMO lead is finished being created. Or we need significantly more resources dedicated to planning enhancements so that we could have an actual backlog of work for the developers. But my company appears to neither be interested in shrinking the PMO team nor in expanding the intake staff.

It could be really beneficial for Anthropic to showcase how they use their own product; since they're developers already, they're probably dogfooding their product, and the effort required should be minimal.

- A lot of skeptics have complained that AI companies aren't specific about how they use their products, and this would be a great example of specificity.

- It could serve as a tutorial for people who are unfamiliar with coding agents.

- The video might not convince people who have already made up their minds, but at least you could point to it as a primary source of information.

These exist. Just now I triedfinding such a video for a medium-sized contemporary AI devtools product (Mastra) and it took me only a few seconds to arrive at https://www.youtube.com/watch?v=fWmSWSg848Q

There could be a million of these videos and it wouldn't matter, the problem is incuriosity/resistance/change-aversion. It's why so many people write comments complaining about these videos not existing without spending even a single minute looking for them: they wouldn't watch these videos even if they existed. In fact, they assume/assert they don't exist without even looking for them because they don't want them to exist: it's their excuse for not doing something they don't want to do.

That video was completely useless for me. I didn't see a single thing I would consider programming. I don't want to waste time building workflows or agentic agents, I want to see them being used to solve real world difficult problems from start to finish.

> How many 1+ hour videos of someone building with AI tools have you sought out and watched?

A lot, they've mostly all been advertising trite and completely useless.

I don't want a demonstration of what a jet-powered hammer is by the sales person or how to oil it, or mindless fluff about how much time it will save me hammering things. I want to see a journeyman use a jet-powered hammer to build a log cabin.

I am personally not seeing this magic utopia. No one wants to show me it, they just want to talk about how better it is.

I can only speak for myself, but it feels like playing with fire to productize this stuff too quick.

Like, I woke up one day and a magical owl told me that I was a wizard. Now I control the elements with a flick of my wrist - which I love. I can weave the ether into databases, apps, scripts, tools, all by chanting a simple magical invocation. I create and destroy with a subtle murmur.

Do I want to share that power? Naturally, it would be lonely to hoard it and despite the troubles at the Unseen University, I think that schools of wizards sharing incantations can be a powerful good. But do I want to share it with everybody? That feels dangerous.

It's like the early internet - having a technical shelf to climb up before you can use the thing creates a kind of natural filter for at least the kinds of people that care enough to think about what they're doing and why. Selecting for curiosity at the very least.

That said, I'm also interested in more data from an engineering perspective. It's not a simple thing and my mind is very much straddling the crevasse here.

LLMs are lossy compression of a corpus with a really good parser as a front end. As human made content dries up (due to LLM use), the AI products will plateau.

I see inference as the much bigger technology although much better RAG loops for local customization could be a very lucrative product for a few years.

I've been noticing it's more on par with sonnet these days. I don't know if that means Opus is getting more efficient, sonnet getting less efficient, or perhaps Opus is getting to the answer fast enough to overcome the higher token spend.

Once again. It is not greenfield projects most of us want to use AI coding assistance for. It is for an existing project, with a byzantine mess of a codebase, and even worse messes of infrastructure, business requirements, regulations, processes, and God knows what else. It seems impossible to me that AI would ever be useful in these contexts (which, again, are practically all I ever deal with as a professional in software development).

Ugh, I'm so sick of these "I can use AI to solve an already solved problem, thus programmers aren't relevant." Note the solved problem part. This isn't convincing except to people that want a (bad) argument to depress wages and lay off workers while making the existing seniors take on more and more work. This is overall bad for the industry.

Every time I see a post like this on HN I try again and every time I come to the same conclusion. I have never see one agent managing to pull something off that I could instantly ship. It still ends up being very junior code.

I just tried again and ask Opus to add custom video controls around ReactPlayer. I started in Plan mode which looked overal good (used our styling libs, existing components, icons and so on).

I let it execute the plan and behold I have controls on the video, so far so good. I then look at the code and I see multiple issues: Over usage of useEffect for trivial things, storing state in useState which should be computed at run time, failing to correctly display the time / duration of the video and so on...

I ask follow up question like: Hide the controls after 2 seconds and it starts introducing more useEffects and states which all are not needed (granted you need one).

Cherry on the cake, I asked to place the slider at the bottom and the other controls above it, it placed the slider on the top...

So I suck at prompting and will start looking for a gardening job I guess...

These posts are never, never made by someone who is responsible for shipping production code in a large, heavily used application. It's always someone at a director+ level who stopped production coding years ago, if they ever did, and is tired of their engineers trying to explain why something will take more than an hour.

It is also often low-proficiency developers with their minds blown over how quickly they can build something using frameworks / languages they never wanted to learn or understand.

Though even that group probably has some overlap with yours.

Back in the day when you found a solution to your problem on Stackoverflow, you typically had to make some minor changes and perhaps engage in some critical thinking to integrate it into your code base. It was still worth looking for those answers, though, because it was much easier to complete the fix starting from something 90% working than 0%.

The first few times in your career you found answers that solved your problem but needed non-trivial changes to apply it to your code, you might remember that it was a real struggle to complete the fix even starting from 90%. Maybe you thought that ultimately, that stackoverflow fix really was more trouble than it was worth. And then the next few times you went looking for answers on stackoverflow you were better at determining what answers were relevant to your problem/worth using, and better at going from 90% to 100% by applying their answers.

> it was much easier to complete the fix starting from something 90% working than 0%.

As an expert now though, it is genuinely easier and faster to complete the work starting from 0 than to modify something junky. The realplayer example above I could do much faster, correctly, than I could figure out what the AI code was trying to do with all the effects and refactor it correctly. This is why I don't use AI for programming.

And for the cases where I'm not skilled, I would prefer to just gain skill, even though it takes longer than using the AI.

Anecdotally I think you're right that the more skilled you are at something, the less utility there is for something that quickly but incompletely takes you from 0 to 90%

But I would generally be skeptical of anybody who claims that all their work is better off starting from 0, the same way I'd be skeptical of someone who claims to not use or need to make google searches about docs/terms/issues as they work.

I'll give you an example of something I understand decently well but get a lot of use out of AI for: bash scripts and unit testing. These are not my core work but they are a large chunk of my work. Without LLMs I would just not write a lot of bash scripts because I found myself constantly looking things up and spending more time than expected getting the script to work across environments / ironing out bugs - I would only write absolutely essential scripts, and generally they'd not be polished enough to check in and share with the team, and just live on my computer in some random location. Now with LLMs I can essentially script in english and get very good bash scripts, so I write a lot more of them and it's easier for me to get them into an acceptable state worth sharing with my team.

Similarly, I really like Golang table tests but hate writing all the cases out and dealing with all the symbols/formatting. Now I can just describe all the different permutations I want and get something that I can lightly edit into being good enough.

I've also found that with domains I am knowledgable enough about, that can translate into being better at going from ~70% to 95% with AI too. In those cases I am not necessarily using AI the same way as someone trying to go from 0->90%: usually they're describing the outcome/goals/features they want relatively informally without knowledge of the known-unknowns and gotchas involved in implementing that. With more knowledge you can prompt LLMs with more implementation/design details and requirements, and course correct away from bad approaches much faster than someone who doesn't know the shape of what they're trying to do. That still comes in handy a lot of the time.

Think about how much time you can save by feeding an API spec/docs into an LLM, telling it create a Go struct for JSON (de)serialization of some monstrous interface like https://docs.cloud.google.com/compute/docs/reference/rest/v1...? Or how much easier it is to upgrade across breaking versions of a language/library when you can just bump the version, note all the places where the old code broke, and have an LLM with an upgrade guide/changelog do all the drudgery of fixing each of the 200 callsites you need to migrate to the next version.

The difference is you’re generally retooling for your purpose rather than scouring for multiple, easily avoidable screw ups that if overlooked will cause massive headaches later on.

I've spent quite a bit of time with Codex recently and come to the conclusion that you can't simply say "Let's add custom video controls around ReactPlayer." You need to follow up with a set of strict requirements to set expectations, guard rails, and what the final product should do (and not do). Even then it may have a few issues, but continuing to prompt with clearly stated problems that don't meet the requirements (or you forgot to include) usually clears it up.

Code that would have taken me a week to write is done in about 10 minutes. It's likely on average better than what I could personally write as a novice-mid level programmer.

>You need to follow up with a set of strict requirements to set expectations, guard rails, and what the final product should do (and not do).

that usually very hard to do part, and is was possible to spent few days on something like that in real word before LLMs. But with LLMs is worse because is not enough to have those requirements, some of those won't work for random reasons and is no any 'rules' that can grantee results. It always like 'try that' and 'probably this will work'.

Just recently I was struggled with same prompt produced different result between API calls before I realized that just usage of few more '\"' and few spaces in prompt leaded model to completely different route of logic which produced opposite answers.

This is very true. But each iteration of learning quirks and installing guardrails carries value forward to later sessions. These rough edges get smoother with use, is my point.

It sounds like it takes you at least 10 minutes to just write the prompt with all the details you mentioned. Especially if you need to continue and prompt again (and again?).

Not the OP but, easily. My tasks are usually taking at least that, but up to hours of brainstorming and planning, sometimes I’ll do this over days in between other tasks just so I can think about all and pros and cons. Of course this has always been the way, but now I have an ongoing Claude session which I can come back to at any point, which is holding the context along with my brain. It’s much easier to keep the thread of what I’m working on across multiple tasks.

> I’m writing some (for me) seriously advanced software that would have taken me months to write, in weeks, using Claude and ChatGPT.

Do you understand the code?

What was the speed up from months to weeks? You just didn't know what to type? Or you didn't know the problem domain? Or you found it hard to 'start' and the AI writing boiler plate gave you motivation?

In my experience with AI tools, it only really helps with ideation, most things it produces need heavy tweaking - to the point that there is no time savings. It's probably a net negative because I am spending all of my time thinking how to explain things to a dumb computer, rather than thinking about how to solve the problem.

The main advantage is I can run it in parallel and iterate often.

The speed up is also avoiding looking up reference manuals endlessly just to produce some Qt Widgets.

I’m a fairly recent convert, I only started “vibe coding” a couple of months ago, after hearing how good Opus was. I had been a skeptic until then.

I am a decentralist by nature and prefer open standards and self hosting. I’ve had my own *nix servers since I was twelve (nearing forty) so it really pains me to admit how good it is to use these corporate products.

I am not a programmer by trade.
I use it to write software for my domain of expertise. The value of what I am creating is enormous.

Both ChatGPT and Claude produce good code, in my opinion.

I find anecdotes like yours bewildering, because I've been using Opus with Vue.js and it crushes everything I throw at it. The amount of corrections I need to make tend to be minimal, and mostly cosmetic.

The tasks I give it are not trivial either. Just yesterday I had it create a full-blown WYSIWYG editor for authoring the content we serve through our app. This is something that would have taken me two weeks, give or take. Opus looked at the content definitions on the server, queried the database for examples, then started writing code and finished it in ~15 minutes, and after another 15-20 minutes of further prompting for refinement, it was ready to ship.

Created a WYSIWYG editor or copied it off the internet like your average junior would, bugs included?

If that editor is very complicated (as they usually are) it makes sense to just opt for a library. If it's simple then AI is not required and would only reduce familiarity with how it works. The third option is what you did and I feel like it's the option with the lowest probability of ending up with a quality solution.

There is contenteditable and EditContext hese days, it's not that hard to make a simple WYSIWYG editor. An LLM could figure out how to operationalize these things quicker than I could.

To be clear, I'm not talking about a rich-text editor. I'm talking about a notion-like interface where you can drag and drop different types of elements to a canvas to build rich content, and adjust the blocks horizontally or vertically via drag and drop.

Have you tried Roo Code in "Orchestrator" mode? I find it generally "chews" the tasks I give it to then spoon feed into sub-tasks in "Code" (or others) mode, leaving less room to stray from very focused "bite-sized" changes.

I do need to steer it sometimes, but since it doesn't change a lot at a time, I can usually guide the agent and stop the disaster before it spreads.

A big caveat is I haven't tried heavy front-end stuff with it, more django stuff, and I'm pretty happy with the output.

I have a vanilla JS project. I find that very small llms are able to work on it with no issue. (Including complete rewrites.) But I asked even large LLMs to port it to React and they all consistently fail. Basic functionality broken, rapid memory leaks.

So I just stuck with vanilla JS.

n = 1 but React might not be a great thing to test this stuff with. For the man and the machine! I tried and failed to learn React properly like 8 times but I've shipped multiple full stack things in like 5 other languages no problem.

usually for me, after a good plan is 90% solid working code. the problem do arise when you ask it to change the colors it choose of light grey text over a white background. this thing still can't see and it's a huge drawback for those who got used to just prompting away their problems

I always assume the person either didn't use coding agents in a while or its their first time. don't get me wrong, i love claude code, but my students are still better at getting stuff done that i can just approve and not micromanage. thats what i think everyone is missing from their commentary. you have to micromanage a coding agent. you don't have to micromanage a good student. when you dont need to micromanage anymore at all, that's when the floor falls out and everyone has a team of agents doing whatever they want to make them all billionaires or whatever it is AI is promising to do those days.

Around a Uni I think a lot about what students are good at and what they aren't good at.

I wouldn't even think about hiring a student to do marketing work. They just don't understand how hard it is to break through people's indifference and lack the hustle. I want 10-100x more than I get out of them.

Photos in The Cornell Daily Sun make me depressed. Students take a step out the door, take a snap, then upload it. I think the campus is breathtakingly beautiful and students just don't do the work to take good photos that show it.

In coding it is across the map. Even when I am happy with the results they still do the first 80% that takes another 80% to put in front of customers. I can be really proud of how it turned out in the end despite them missing the point of the design document they were handed.

I was in a game design hackathon where most of the winners were adults or teams with an adult on them. My team won player's choice. I'll take credit for my startup veteran talent of fearlessly demonstrating broken software on stage and making it look great and doing project management with that in mind. One student was solid on C# and making platformers in Unity. I was the backup programmer who worked like a junior other than driving them crazy slowing them down with relentlessly practical project management. The other student made art that fit our game.

We were at each other's throats at the end and shocked that we won. I think I understood the value everybody brought but I'm not sure my teammates did.

Yep. It sucks. People are delusional. Let's ignore LLMs and carry on...

On a more serious note:

1) Split tasks into smaller tasks just like a human would do

Would you bash your keyboard for an hour, adding all video controls at once before even testing if anything works at all? Ofc not. You would start by adding a slider and test it until you are satisfied. Then move to next video control. An so on. LLMs are the same. Sometimes they can one-shot many related changes in a single prompt but the common reality is what you experienced: it works sometimes but the code is suboptimal.

2) Document desireable and undesireable coding patterns in AGENTS.md (or CLAUDE.md)

If you found over usage of useEffect, document it on AGENTS.md so next time the LLM knows your preference.

I have been using LLMs since Sonet 3.5 for large enterprise projects (1kk+ lines of code, 1k+ database tables). I just don't ask it to "draw the rest of owl" as the saying goes.

i’ve become convinced that the devs that talk about having to fix the code are the same ones that would make incredibly poor managers. when you manage a team you need to be focused on the effect of the code not the small details.

this sort of developer in a pair programming exercise would find themselves flustered at how a junior approached problem solving and just fix it themselves. i strongly suspect the loss of a feeling of control is at play here.

I just had an issue where Opus misspelled variable names between usages. These are fundamental and elementary mistakes that make me deeply distrust anything slightly more complex that comes out of it.

It's great for suggesting approaches, but the code it generates looks like it doesn't actually have understanding (which is correct).

I can't trust what it writes, and if I have to go through it all with a fine toothed comb, I may as well write the code myself.

So my conclusion is that it's a very powerful research tool, and an atrocious junior developer who has dyslexia and issues with memory.

Ah, another thread filled with people sharing anecdotes about how they asked Claude to one-shot an entire project that would take people weeks if not months.

It's been interesting watching HN shift in my direction on this in recent weeks...

I had been saying since around summer of this year that coding agents were getting extremely good. The base model improvements were ok, but the agentic coding wrappers were basically game changers if you were using them right. Until recently they still felt very context limited, but the context problem increasingly feels like a solved problem.

I had some arguments on here in the summer about how it was stupid to hire junior devs at this point and how in a few years you probably wouldn't need senior devs for 90% of development tasks either. This was an aggressive prediction 6 months ago, but I think it's way too conservative now.

Today we have people at our company who have never written code building and shipping bespoke products. We've also started hiring people who can simply prove they can build products for us using AI in a single day. These are not software engineers because we are paying them wages no SWEs would accept, but it's still a decent wage for a 20 something year old without any real coding skills but who is interested in building stuff.

This is something I wouldn't have never of expected to be possible 6 months ago. In 6 months we've gone from senior developers writing ~50% of their code with AI, to just a handful of senior developers who now write close to 90% of their code with AI while they support a bunch of non-developers pumping out a steady stream of shippable products and features.

Software engineers and traditional software engineer is genuinely running on borrowed time right now. It's not that there will be no jobs for knowledgable software engineers in the coming years, but companies simply won't need many hotshot SWEs anymore. The companies that are hiring significant numbers of software engineers today simply can not have realised how much things have changed over just the last few months. Apart from the top 1-2% of talent I simply see no good reason to hire a SWE for anything anymore. And honestly outside of niche areas, anyone hand-cracking code today is a dinosaur... A good SWE today should see their job as simply reviewing code and prompting.

If you think that the quality of code LLMs produce today isn't up to scratch you've either not used the latest models and tools or you're using them wrong. That's not to say it's the best code – they still have a tendency to overcomplicate things in my opinion – but it's probably better than the average senior software engineer. And that's really all that matters.

I'm writing this because if you're reading this thinking we're basically still in 2024 with slightly better models and tooling you're just wrong and you're probably not prepared for what's coming.

Hi Kypro this is very interesting perspective. Can you reach out to me? I'd like to discuss what you're observing with you a bit in private as it relates heavily to a project I'm currently working on. My contact info is on my profile. Pls shoot me a connection request and just say you're kypro from HN :)

Or is there a good way for me to contact you? Your profile doesn't list anything and your handle doesn't seem to have much of an online footprint.

Lastly, I promise I'm not some weirdo, I'm a realperson™ -- just check my HN comment history. A lot of people in the AI community have met me in person and can confirm (swyx etc).

LLM's are good at making stuff from scratch and perfect when you don't have to worry about the codes future. 'Research' can be a great tool. But LLMs are horrible in big codebases and multiple micro services. Also at making decision, never let it make a decision for you. You need to know what's happening and you can't ship straight AI code. It can save time, but it's not a lot and it won't replace anyone.

We have a large monorepo at my company. You're right that for adding entirely new core concepts to an existing codebase we wouldn't give an AI some vague requirements and ask it to build something – but we wouldn't do that for a human engineer either. Typically we would discuss as a team and then once we've agreed on technologies and an approach someone will implement it relying heavily on AI to write the actual code (because it's faster and generally won't add dumb bugs like typos or conditional logic error).

Almost everything else at this point can be done by AI. Some stuff requires a little support from human engineers, but honestly our main bottlenecks at this point is just QA and getting the infra to a place where we can rapidly ship stuff into production.

> You need to know what's happening and you can't ship straight AI code.

I think there is some truth to this. We are struggling to maintain a high-level understanding of the code as a team right now, not because there is no human that understands, but because 5 years ago our team would have probably been 10-20x larger given the amount we're shipping. So when one engineer leaves the company or goes on holiday we find we lose significantly more context of systems than you historically would with larger teams of engineers. Previously you might have had 2-3 engineers who had a deep understanding of a single system. Now we have maybe 1-2 engineers who need to maintain understanding of 5-6 systems.

That said, AI helps a lot with this. Asking AI to explain code and help me learn how it works means I can pick up new systems significantly quicker.

Yes. I mostly work on Quarkus microservices and use cursor with auto agent mode.

> we wouldn't give an AI some vague requirements and ask it to build something
> we would discuss as a team

seems like a reasonable workflow. It's the polar opposite of what was written in the blog post. That is the usual, easy way people use agents and what I think is the wrong path. May I also ask what language and/or framework you work with where so much context works good enough?

> Asking AI to explain code and help me learn how it works means I can pick up new systems significantly quicker.

Software/web meat shops have bean around since the dawn of the time.

I worked at McDonald's in my teens. One of the best managers I ever worked for was the manager at this store at this time(the owner rotated him between stores to help get things on track).

I'll never forget this one thing he said: "They have changed the Filet-O-Fish five times since I've been here, and each time it's become more profitable".

I'm tired of constantly debating the same thing again and again. Where are the products? Where is some great performing software all LLM/agent crafted? All I see is software bloatness and decline. Where is Discord that uses just a bunch of hundreds megs of ram? Where is unbloated faster Slack? Where is the Excel killer? Fast mobile apps? Browsers and the web platform improved? Why Cursor team don't use Cursor to get rid of vscode base and code its super duper code editor? I see tons of talking and almost zero products.

Even if there is a "fully vibe-coded" product that has real customers, the fact that it's vibe-coded means that others can do the same. Unless you have a secret LLM or some magical prompts that make the code better/more efficient than your competitions, your vibe coded product has no advantage over competition and no moat. What actually matters is everything else -- user experience (which requires hours of meetings and usability studies), integration with own/other people's products, business, marketing, sales etc, much of which you can't vibe code your way to success.

I'm not sure what point you're making here. Tech is rarely the moat, you even get to that point at the end of your post. The "vibe coding" advantage is faster time to market, faster iterations, etc. These things will help you get that user experience, integrations, etc.

Faster, faster, faster. All to release something that is slower, by people that now know lesser, with bloat that explodes. All for a yet another useless saas that nobody or fee people wants and a chance to virtue signaling your vibe coded product on HN. Real world successfull products are orthogonal to this approach, it doesnt work anymore in today's world

>> Even if there is a "fully vibe-coded" product that has real customers, the fact that it's vibe-coded means that others can do the same.

But that's precisely why you don't hear about these products: the creators don't disclose that they were vibe-coded, because if they do, that invites competition.

I personally know of four vibe-coded products that generate over $10k/mo. Two of them were made by one friend, one was made by another, and the last one by my cousin. None of these people are developers. But they are making real money.

Website roulette probably has a 50% shot at loading a blog written by a digital nomad who makes a living off some SEO side project that pays for their Asia-Pacific island lifestyle...

> Even if there is a "fully vibe-coded" product that has real customers, the fact that it's vibe-coded means that others can do the same.

I think you are strawmanning what "vibe coders" do when they build stuff. It's not simple one-shot generation of eg twitter clones, it's really just iterative product development through an inconsistently capable/spotty LLM developer. It's not really that different from a product manager hiring some cheap developer and feeding them tasks/feature requests. By the way, competitors can hire those and chip away at your moat too!

> Unless you have a secret LLM or some magical prompts that make the code better/more efficient than your competitions, your vibe coded product has no advantage over competition and no moat

This is just not true, and you kind of make my point in the next sentence: many companies competitive advantages come from distribution, trust, integration, regulatory, marketing/sales, network effects. But also, vibe coding is not really about prompts so much as it is product iteration. Anybody product can be copied already, yet people still make way more new products than direct product clones anyway, because it's usually more valuable to go to market with stronger, more focused, or more specialized/differentiated software than a copy.

you can build it and simply use it in your own office? There is no need to shout about it if the cost of writing software goes to zero (but the value remains non-zero!).

Get the feeling with the pending IPO, there might be some challengers to discord that get more traction due to the protracted enshittification of the platform (cf. bluesky)

Totally disagree. One example is Zed which is very well known and it's faster than any other editor, wasn't built with AI though.

> People on larger companies are not at the edge of AI coding

False Microsoft is all in with Copilot, and I can't believe the company that created Copilot doesn't use it internally, I'd rather say they should be the ones that would know how to master it! Yet no better vscode, still bloated teams etc etc

They haven't started Zed with vibe coding but rather now iterating over the stable and mature codebase for making changes/fixing bugs, comment is out of topic

Do you mean to say Zed wasn't vibe coded? There's actually another comment on this post describing how someone is using Opus 4.5 to work on Zed. Given how forward the AI features are in Zed I'd be surprised if the team wasn't also embracing it internally.

It's a fair question how much AI is accelerating the development of Zed, but I can say that I've been impressed with the speed they are shipping at.

Indeed it wasn't vibe coded, using LLMs to iterate over a mature and well structured codebase is another thing and won't obliterate the existince of software programmers

I'm not assuming, the whole narrative goes like "software development is a solved problem and a sunken cost" Ok if the cost is low why not then? It makes sense to improve your product and strenghten your market position

I don't recall if it was an AGENT.md or CLAUDE.md but one of those was definitely in the Zed repo last time I looked at it. Someone is using AI to work on it.

This argument falls a little flat when you consider how much software may or may not be written inside one's own personal work flow, or to scale that up, inside a small business. The idea that a small business doing >1mil revenue can now hire a dev or two, and build out a fairly functional domain-driven system should not be understated. The democratization of software, and the lowering of the barriers to entry to basic CRUD apps, may not necessarily show up in a TAM report...
Do you need a killer app that treads into unicorn territory to prove it's impact? What about a million apps that displace said unicorn potentials by removing the need for a COTS?

Oh, and remember, the iPhone was revolutionary but it was diffused so slowly into the greater economy, the impact on global GDP was basically negligent. Actually, almost all the perceived grandiose tech jumps did not magically produce huge GDP gains overnight.

Your argument falls a little flat considering that you mention "hire a dev or two" while the whole narrative is "we don't need software engineers anymore" and Anthropic alone declares that "Although engineers use Claude frequently, more than half said they can “fully delegate” only between 0-20% of their work to Claude" https://www.anthropic.com/research/how-ai-is-transforming-wo...

When was I arguing about job displacement or the replacing of engineers? You are projecting hard, and reaching. If anything, I am in the camp that accessibility to custom tooling equals a net positive of devs down the line. In the short term, it may be a bumpy road as the tools progress (even if incrementally), but my long term take is that you may see engineering teams blossom in smaller market operations.

When it comes to objectivity, people with your line of thinking is what I try to avoid, as it is clear you feel threatened by the progress of coding tools. That link doesn't change much about what I said, or for that matter, what you said. You were commenting on the lack of a killer app, and I just said it may be diffusing slowly in different ways.

You are fixating on the "whole narrative" because you feel threatened - rightfully so, but again, that type of hyperbole doesn't belong in a constructive and grounded conversation about the impact AI may or may not have.

see "How much work can be fully delegated to Claude?": "Although engineers use Claude frequently, more than half said they can “fully delegate” only between 0-20% of their work to Claude"

There won't be anything like you're asking for, even the vendors themselves (they'll be the most positive and most enthousiastic about using it) can't do this with them.

My point is that you can ignore every article about ai being super good as long as you see the vendor research (that you read once a year or less) is still the same. It saves everyone a lot of frustration. As for why it keeps appearing here, people like being excited. It's not about the truth, so asking for it is missing the point.

However, I think the biggest thing is the replacement of products. We are in a place where he talked about replacing two products his wife was using with custom software. I personally have used LLMs to build things that are valuable for me that I just don't have time for otherwise.

This is true. I think most people are mostly using AI at work to fix bugs in existing codebases. A smaller group of people are benchmarking AI by giving it ideas for apps that no one needs and seeing if it can get close. The smallest group of people is actually designing new software and asking the AI to iterate on it.

Could someone explain this to me? I have the same question: why Cursor team don't use Cursor to get rid of vscode base and code its super duper code editor?

Except for maybe an "Excel killer", all those things you listed are not things people are willing to pay for. Also agents are bad at that kind of work (most devs are bad at that stuff, it's why it was something people whined about even before agents).

And funnily enough there are products and tools that are essentially less bloated slack/discord. Have you heard of https://stoat.chat/ (aka revolt) or https://pumble.com/ or https://meet.jit.si/? If not I would guess it's for one of two reasons: not caring enough about these problems to even go looking for them yourself, or their lack of "bloatedness" resulting in them not being a mature/fully featured enough product to be worth marketing or adopting.

If you'd like to see a product mostly made with agents/for agents you can check out mine at https://statue.dev/ - we're making a static site generator with a templating and component system paired with user-story driven "agentic workflows" (~blueprints/playbooks for common user actions like "I need to add a new page and list it on the navbar" or "create a site from the developer portfolio template personalized for my github").

I would guess most other projects are probably in a similar situation as we are: agentic developer tools have only really been good enough to heavily use/build products around for a few months, so it's a typical few-month-old project. But agents definitely made it easier to build.

Not willing to pay for? How can you be sure? For example explain then why many gamers are ditching Windows for Linux and buying hardware from Valve... There must be a reason. Every person I talked to that uses Excel hate how slow it is, same for teams and many other products. Finally, were the mentioned products built with vibe coding?

Generally if something is fast enough/efficient enough that a paying customer can use it without having to worry or actively think about performance and un-bloatedness, that's enough for them. The only people who might complain still are developers who are bothered by the inefficiency and are technically literate enough to notice it, and maybe the users with less powerful/capable devices than the ones the big paying customers use. Generally these groups of people are not the actual customers of these products.

The people who actually pay for slack and discord (eg enterprises that need workplace chat app and decided to go with the "gold standard", consumers with discord servers and such) need the features/tradeoffs choosing featuers over efficiency causing that bloat. They just don't all need the exact same set of those features as the other customers. So because customers are willing to pay for all these features the product tries to ship all of them and becomes bloated.

> Every person I talked to that uses Excel hate how slow it is

But do they make the purchasing decisions behind using Excel?

To be clear I am not really arguing that bloat/overly enterprisey products are good. What I mean that you don't see the world exploding with more elegant products now with agents for the same reason you didn't see the world exploding with them before agents either: the people who pay for those products and build them for a living are not incentivized or necessarily even rewarded for choosing to make them more efficient or elegant when there are other things that customers are asking for with more $$$ behind them.

I did a lot of analysis and biz dev work on the "Excel killer" and came to the conclusion that it would be hard to get people to pay for.

For one thing most enterprises and many individuals have an Office 365 subscription to access Office programs which are less offensive than Excel so they aren't going to save any money by dropping Excel.

On top of it the "killer" would probably not be one product aimed at one market but maybe a few different things. Some people could use "visual pandas" for instance, something that today would be LLM-infused. Other people could use a no-code builder for calculations. The kind of person who is doing muddled and confused work with Excel wouldn't know which "killer" they needed or understand why decimal math would mean they always cut checks in the right amount.

Wrt statue.dev good luck for sure with the project but I personally don't need yet another static site generator, nextjs like but with unpopular svelte, bloated with tons of node modules creating another black hole impossible to escape from. If agents works this well why would I need to use your library? I just tell an agent to maintain my static site who cares which tech stack

Anecdotally I had Gemini convert a simple react native app to swift in two prompts. If it's that simple then maybe we will see less of the chromium desktop apps

I'd argue the contrary, YOU KNOW you have the option, ease of entering doesn't mean they will know how to choose better, they will just vibe code more electron apps. In fact my prediction is not there will be less Electron apps but more

who told you that mb of ram is a definition of success?

Opus was out only few months, and it will take time to get this new wave to market. i can assure you my team become way more productive because of opus. not a single developer but an etnire team.

It's a definition of what runs and what not on consumer grade computers, Discord has a routine that now checks if memory goes over a certain threshold and eventually restart itselfs, this is a measure of engineering total failure imo

To the sceptics still saying that LLMs still can't solve "slime mold pathing algorithm and creating completely new shoe-lacing patterns" (literally a quote from a different comment here), please consider something we've learnt over and over again in history: good enough and cheap will destroy perfect but expensive.

And then cheap and good enough option will eventually get better because that's the one that is more used.

It's how Japanese manufacturing beat Western manufacturing. And how Chinese manufacturing then beat Japanese again.

It's why it's much more likely you are using the Linux kernel and not GNU hurd.

It's how digital cameras left traditional film based cameras in the dust.

lol I can't believe we're doing this again. None of this is innovation. None of this is new. These are all things that already exist. I understand it's impressive that Opus could go through the tedious process on its own, especially considering other LLMs failed. However, none of this is going to improve people's lives. It will simply add more and more and more and more and more slop apps to an already tetra-slopified universe of apps. Do people not see how useless this is? Re-building things that most probably already exist, simply with your own little special flavour? Where are we going...

I don't think you've used it.
I used it intensely and mostly autonomously (with clear instructions, including how to measure good output) almost non-stop over the holidays. Its a new abstraction for programming -- it doesn't replace software developers, it gives them a more natural way to describe what they want.

> none of this is going to improve people's lives.

I have some old borderline senile relatives writting apps (asking LLMs to write for it them) for their own personal use. Stuff they surely haven't done on their own (or had the energy to do). Their extent of programming background - shitty VBScript macros for excel.

It also helps people to pick up programming and helps with the initial push of getting started. Getting over the initial hump, getting something on the screen so to speak.

Most things people want from their computers are simple shit that LLMs usually manage quite well.

Good question whether or not this (outsourcing their thinking) actually just accelerates their senility or not.

As someone who likes to solve hard or interesting technical problems, I've long before LLMs often been disappointed that most of the time what people want from programmers is simple stupid shit (ie. stuff i dont find interesting to work on).

More than half. What has anyone written that was truly new? Regardless, if you have an idea, you will build it out of some combination of conditionals, loops, and expressions… turns out agents are pretty good at those things, even when the idea you’re expressing is novel.

This is a natural response to software enshittification. You can hardly find an iOS app that is not plagued by ads, subscriptions, or hostile data collection. Now you can have your own small utilities that can work for you. This sort of personal software might be very valuable in the world where you are expected to pay 5$ to click any button.

Yeah sure but have you considered that the actual cost of running these models is actually much greater than whatever cost you might be shelling out for the ad-free apps? You're talking to someone who hates the slopification and enshittification of everything, so you don't need to convince me about that. However, everything I've seen described in the replies to my initial comment - while cute, and potentially helpful on a case-by-case basis, does NOT warrant the amount of resources we are pouring into AI right now. Not even fucking close. It'll all come crashing down, taxpayers the world over will be caught with the bag in their hands, and for what? So that we can all have a less robust version of an app that already exists but that has the colours we want and the button where we want it?

If AI cost nothing and wasn't absolutely decimating our economy, I'd find what you've shared cute. However, we are putting literally all of our eggs, and the next generation's eggs, and the one after that, AND the one after that, into this one thing, which, I'm sorry, is so far away from everything that keeps on being promised to us that I can't help but feel extremely depressed.

At this point it doesn't matter that much whether we use AI or not, the apps are not selling and they are being produced at an alarming rate.

The projects being submitted to product hunt is 4x the year before.

The market is shrinking rapidly because now more people make their own apps.

Even making a typo and landing on a website, there is good chance its selling more ai snake oil, yet none of these apps are feature complete and easily beaten by apps made by guys in 2010s. (tldr & sketchbook for the drawing space).

Only way to excite the investors is to fake the ARR by giving free trials and sell before the recurring event occurs.

You are attempting to move the goalposts. There are two different points in this debate:

1) Modern LLMs are an inflection point for coding.

2) The current LLM ecosystem is unsustainable.

This submission discussion is only about #1, which #2 does not invalidate. Even if the ecosystem crashes, then open-source LLMs that leverage the same tricks Opus 4.5 does will just be used instead.

But it's only an inflection point if it's sustainable. When this comes crashing down, how many people are going to be buying $70k GPUs to run an open source model?

I said open-source models, not locally-hosted models. Essentially, more power to inference-only providers such as Groq and Together AI which host the large-scale OSS LLMs who will be less affected by a crash as long as the demand for coding agents is there.

Ok, and then? Taking a one time discount on a rapidly depreciating asset doesn’t magically make this whole industry profitable, and it’s not like you’re going to start running a GB200 in your basement.

Checked your history. From a fellow skeptic, I know how hard it is to reason with people around here. You and I need to learn to let it go. In the end, the people at the top have set this up so that either way, they win. And we're down here telling the people at our level to stop feeding the monster, but told to fuck off anyways.

So cool bro, you managed to ship a useless (except for your specific use-case) app to your iphone in an hour :O

What I think this is doing is it's pitting people against the fact that most jobs in the modern economy (mine included btw) are devoid of purpose. This is something that, as a person on the far left, I've understood for a long time. However, a lot (and I mean a loooooot) of people have never even considered this. So when they find that an AI agent is able to do THEIR job for them in a fraction of the time, they MUST understand it as the AI being some finality to human ingenuity and progress given the self-importance they've attributed to themselves and their occupation - all this instead of realizing that, you know, all of our jobs are useless, we all do the exact same useless shit which is extremely easy to replicate quickly (except for a select few occupations) and that's it.

I'm sorry to tell anyone who's reading this with a differing opinion, but if AI agents have proven revolutionary to your job, you produced nothing of actual value for the world before their advent, and still don't. I say this, again, as someone who beyond their PhD thesis (and even then) does not produce anything of value to the world, while being paid handsomely for it.

> if AI agents have proven revolutionary to your job, you produced nothing of actual value for the world before their advent, and still don't.

This doesn’t logically follow. AI agents produce loads of value. Cotton picking was and still is useful. The cotton gin didn’t replace useless work. It replaced useful work. Same with agents.

> I'm sorry to tell anyone who's reading this with a differing opinion, but if AI agents have proven revolutionary to your job, you produced nothing of actual value for the world before their advent, and still don't.

I agree with this, but I think my take on it is a lot less nihilistic than yours. I think people vastly undersell how much effort they put into doing something, even if that something is vibecoding a slop app that probably exists. But if people are literally prompting claude with a few sentences and getting revolutionary results, then yes, their job was meaningless and they should find something to do that they’re better at.

But what frustrates me the most about this whole hype wave isn’t just that the powers that be have bet the entire economy on a fake technology, it’s that it’s sucking all of of the air out of the room. I think most people’s jobs can actually provide value and there’s so much work to be done to make _real_ progress. But instead of actually improving the world, all the time, money, and energy is being thrown into such a wasteful technology that is actively making the world a worse place. I’m sure it’s always been like this and I was just to naive too see it, but I much preferred it when at least the tech companies pretended they cared about the impact their products had on society rather than simply trying to extract the most value out of the same 5 ideas.

Yeah, I do tend to have a rather nihilistic view on things, so apologies.

I really think we're just cooked at this point. The amount of people (some great friends whom I respect) that have told me in casual conversation that if their LLM were taken from them tomorrow, they wouldn't know how to do their work (or some flavour of that statement) has made me realize how deep the problem is.

We could go on and on about this, but let's both agree to try and look inward more and attempt to keep our own things in order, while most other people get hooked on the absolute slop machine that is AI. Eventually, the LLM providers will need to start ramping up the costs of their subscriptions and maybe then will people start clicking that the shitty code that was generated for their pointless/useless app is not worth the actual cost of inference (which some conservative estimates put out to thousands of dollars per month on a subscription basis). For now, people are just putting their heads in the sand and assuming that physicists will somehow find a way to use quantum computers to speed up inference by a factor of 10^20 in the next years, while simultaneously slashing its costs (lol).

But hey, Opus 4.5 can cook up a functional app that goes into your emails and retrieves all outstanding orders - revolutionary. Definitely worth the many kWh and thousands of liters of water required, eh?

The studies focus on a single representative task, but in a thread about coding entire apps in hours as opposed to weeks, you can imagine the multiples involved in terms of resource conservation.

The upshot is, generating and deploying a working app that automates a bespoke, boring email workflow will be way, way, wayyyyy more efficient than the human manually doing that workflow everytime.

I want to push back on this argument, as it seems suspect given that none of these tools are creating profit, and so require funds / resources that are essentially coming from the combined efforts of much of the economy. I.e. the energy externalities here are monstrous and never factored into these things, even though these models could never have gotten off the ground if not for the massive energy expenditures that were (and continue to be) needed to sustain the funding for these things.

To simplify, LLMs haven't clearly created the value they have promised, but have eaten up massive amounts of capital / value produced by everyone else. But producing that capital had energy costs too. Whether or not all this AI stuff ends up being more energy efficient than people needs to be measured on whether AI actually delivers on its promises and recoups the investments.

EDIT: I.e. it is wildly unclear at this point that if we all pivot to AI that, economy-wide, we will produce value at a lower energy cost, and, even if we grant that this will eventually happen, it is not clear how long that will take. And sure, humans have these costs too, but humans have a sort of guaranteed potential future value, whereas the value of AI is speculative. So comparing energy costs of the two at this frozen moment in time just doesn't quite feel right to me.

These tools may not be turning a profit yet, but as many point out, this is simply due to deeply subsidized free usage to capture market share and discover new use cases.

However, their economic potential is undeniable. Just taking the examples in TFA and this sub-thread, the author was able to create economic value by automating rote aspects of his wife's business and stop paying for existing subscriptions to other apps. TFA doesn't mention what he paid for these tokens, but over the lifetime of his apps I'd bet he captures way more value than the tokens would have cost him.

As for the energy externalities, the ACM article puts some numbers on them. While acknowledging that this is an apples/oranges comparison, it points out that the training cost for GPT-3 (article is from mid-2024) is about 5x the cost of raising a human to adulthood.

Even if you 10x that for GPT-5, that is still only the cost of raising 50 humans to adulthood in exchange for a model that encapsulates a huge chunk of the world's knowledge, which can then be scaled out to an infinite number of tasks, each consuming a tiny fraction of the resources of a human equivalent.

As such, even accounting for training costs, these models are far more efficient than humans for the tasks they do.

I appreciate your responses to my comments, including the addition of reading material. However, I'm going to have to push back on both points.

Firstly, saying that because AI water use is on par with other industries, then we shouldn't scrutinize AI water use is a bit short-sighted. If the future Altman et al want comes to be, the shear scale of deployment of AI-focused data centers will lead to nominal water use orders of magnitude larger than other industries. Of course, on a relative scale, they can be seen as 'efficient', but even something efficient, when built out to massive scale, can suck out all of our resources. It's not AI's fault that water is a limited resource on Earth; AI is not the first industry to use a ton of water; however, eventually, with all other industries + AI combined (again, imagining the future the AI Kings want), we are definitely going 300km/h on the road to worldwide water scarcity. We are currently at a time where we need to seriously rethink our relationship with water as a society - not at a time where we can spawn whole new, extremely consumptive industries (even if, in relative terms, they're on par with what we've been doing (which isn't saying much given the state of the climate)) whose upsides are still fairly debatable and not at all proven beyond a doubt.

As for the second link, there's a pretty easy rebuke to the idea, which aligns with the other reply to your link. Sure, LLMs are more energy-efficient at generating text than human beings, but do LLMs actually create new ideas? Write new things? Any text written by an LLM will be based off of someone else's work. There is a cost to creativity - to giving birth to actual ideas - that LLMs will never be able to incur, which makes them seem more efficient, but in the end they're more efficient at (once again) tasks which us humans have provided them with plenty of examples of (like writing corporate emails! Or fairly cookie-cutter code!) but at some point the value creation is limited.

I know you disagree with me, it's ok - you are in the majority and you can feel good about that.

I honestly hope the future you foresee where LLMs solve our problems and become important building blocks to our society comes to fruition (rather than the financialized speculation tools they currently are, let's be real). If that happens, I'll be glad I was wrong.

These are important conversations to have because there is so much hyperbole in both directions that a lot of people end up having strong but misguided opinions. I think it's very helpful to consider the impact of LLMs in context (heheh) of the bigger picture rather than in isolation, because suddenly a lot of things fall into perspective.

For instance, all water use by data centers is a fraction of the water used by golf courses! If it really does comes down to the wire for conserving water, I think humanity has the option of foregoing a leisure activity for the relatively wealthy in exchange for accelerated productivity for the rest of the world.

And totally, LLMs might not be able to come up with new ideas, but they can super-charge the humans who do have ideas and want to develop them! An idea that would have taken months to be explored and developed can now be done in days. And given that like the majority of ideas fail, we would be failing that much faster too!

In either case, just eyeballing the numbers we have currently, on average the resources a human without AI assistance would have consumed to conclude an endeavor far outweighs the resources consumed by both that human and an assisting LLM.

I would agree that there will likely be significant problems caused by widespread adoption of AI, but at this point I think they would social (e.g. significant job displacement, even more wealth inequality) rather than environmental.

> I want to push back on this argument, as it seems suspect given that none of these tools are creating profit, and so require funds / resources that are essentially coming from the combined efforts of much of the economy. I.e. the energy externalities here are monstrous and never factored into these things, even though these models could never have gotten off the ground if not for the massive energy expenditures that were (and continue to be) needed to sustain the funding for these things.

While it is absolutely possible, even plausible, that the economics of these models and providers is the next economic crash in waiting, somewhere between Enron (at worst, if they're knowingly cooking books) or Global Financial Crisis (if they're self-delusional rather than actively dishonest), we do have open-weights models that get hosted for money, that people play with locally if they're rich enough for the beefy machines, and that are not too far behind the SOTA as to suggest a difference in kind.

This all strongly suggests that the resource consumption per token by e.g. Claude Code would be reasonably close to the list price if they weren't all doing a Red Queen race[0], running as hard as they can just to retain relevant against each other's progress, in an all-pay auction[1] where only the best can ever hope to cash anything out and even that may never be enough to cover the spend.

Thing is, automation has basically always done this. It's more of a question of "what tasks can automation actually do well enough to bother with?" rather than "when it can, is it more energy efficient than a human?"

A Raspberry Pi Zero can do basic arithmetic faster than the sum total performance of all 8 billion living humans, even if all the humans had trained hard and reached the level of the current world record holder, for a tenth of the power consumption of just one of those human's brains, or 2% of their whole body. But that's just arithmetic. Stable Diffusion 1.5 had a similar thing, when it came out the energy cost to make a picture on my laptop was comparable with the calories consumed while typing in a prompt for it… but who cares, SD 1.5 had all that Cronenberg anatomy, what matters is when the AI is "good enough" for the tasks against which it is set.

To the extent that Claude Code can replace a human, and the speed at which it operates…

Well, my experiments just before Christmas (which are limited, and IMO flawed in a way likely to overstate the current quality of the AI) say the speed of the $20 plan is about 10 sprints per calendar month, while the quality is now at the level of a junior with 1-3 years experience who is just about to stop being a junior. This means the energy cost per unit of work done is comparable with the energy cost needed to have that developer keep a computer and monitor switched on long enough to do the same unit of work. The developer's own body adds another 100-120 watts to that from biology, even if they're a free-range hippie communist who doesn't believe in money, cooked food, lightbulbs, nor having a computer or refrigerator at home, and who commutes by foot from a yurt with neither AC nor heating, ditto the office.

Where the AI isn't good enough to replace a human, (playing Pokemon and managing businesses?) it's essentially infinitely more expensive (kWh or $) to use the AI.

Still, this does leave a similar argument as with aircraft: really efficient per passenger-kilometre, but they enable so many more passenger-kilometres than before as to still sum to a relevant problem.

> For now, people are just putting their heads in the sand and assuming that physicists will somehow find a way to use quantum computers to speed up inference by a factor of 10^20 in the next years, while simultaneously slashing its costs (lol).

GPT-3 Da Vinci cost $20/million tokens for both input and output.

GPT-5.2 is $1.75/million for input and $14/million for output

I'd call that pretty strong evidence that they've been able to dramatically increase quality while slashing costs, over just the past ~4 years.

Isn't that kind of related with the amount of money thrown at the field? If the economy gets worse for any reason, do you think that we can still expect these level of cutting costs in the future?

> But hey, Opus 4.5 can cook up a functional app that goes into your emails and retrieves all outstanding orders - revolutionary. Definitely worth the many kWh and thousands of liters of water required, eh?

The thing is in a vacuum this stuff is actually kinda cool. But hundreds of billions in debt-financed capex that will never seen a return, and this is the best we’ve got? Absolutely cooked indeed.

Once you’ve got Claude Code set up, you can point it at your codebase, have it learn your conventions, pull in best practices, and refine everything until it’s basically operating like a super-powered teammate. The real unlock is building a solid set of reusable “skills” plus a few agents for the stuff you do all the time.

For example, we have a custom UI library, and Claude Code has a skill that explains exactly how to use it. Same for how we write Storybooks, how we structure APIs, and basically how we want everything done in our repo. So when it generates code, it already matches our patterns and standards out of the box.

We also had Claude Code create a bunch of ESLint automation, including custom ESLint rules and lint checks that catch and auto-handle a lot of stuff before it even hits review.

Then we take it further: we have a deep code review agent Claude Code runs after changes are made. And when a PR goes up, we have another Claude Code agent that does a full PR review, following a detailed markdown checklist we’ve written for it.

On top of that, we’ve got like five other Claude Code GitHub workflow agents that run on a schedule. One of them reads all commits from the last month and makes sure docs are still aligned. Another checks for gaps in end-to-end coverage. Stuff like that. A ton of maintenance and quality work is just… automated. It runs ridiculously smoothly.

We even use Claude Code for ticket triage. It reads the ticket, digs into the codebase, and leaves a comment with what it thinks should be done. So when an engineer picks it up, they’re basically starting halfway through already.

There is so much low-hanging fruit here that it honestly blows my mind people aren’t all over it. 2026 is going to be a wake-up call.

(used voice to text then had claude reword, I am lazy and not gonna hand write it all for yall sorry!)

Edit: made an example repo for ya

https://github.com/ChrisWiles/claude-code-showcase
