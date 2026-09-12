---
url: https://gist.github.com/b7d09e2d36ba4636bd48091e7fe9d13e
date_fetched: 2026-09-13
---

You signed in with another tab or window. Reload to refresh your session.You signed out in another tab or window. Reload to refresh your session.You switched accounts on another tab or window. Reload to refresh your session.Dismiss alert

SQLite was born from a broken dependency. A flaky Informix server caused user-facing failures that the speaker couldn’t control, so he eliminated the server entirely — a database as a single file, directly accessed.

Reliability is designed in, not bolted on. The product was re-architected to be testable from the start: pluggable OS interfaces, fault-injection hooks, and a public test-control API that ships in every production build.

100% MCDC is the non-negotiable floor. Every machine-code branch is exercised in both directions, and every bit in a bitmask test must independently affect the outcome. Achieving this in 2009 stopped the flood of external bug reports almost overnight.

Test code dwarfs source code — and that’s the point. The TH3 test harness alone is over six times larger than the SQLite source. About 15–20% of the source code itself exists only for testing. This is not waste; it’s what makes a three-committer team viable.

Testing is not a one-time event. MCDC coverage, fuzzing, and semantic checks require constant maintenance. Every new feature, every bug fix, every compiler quirk demands updates to the test suite.

Comments and asserts are executable, testable artifacts. Human-language comments engage different neural pathways and catch design flaws early. Assert macros (never, always, testcase) serve as both documentation and runtime invariant checks, with behavior that changes depending on build mode.

Fuzzing and AI expose gaps that MCDC misses. Profile-guided fuzzers found creative ways to flip branches. Semantic fuzzing uncovered logic errors. AI-generated inputs recently found a stack-overflow in a hand-rolled quicksort — a bug that no traditional coverage would catch.

Necessary, but not sufficient. All these practices make software that works, but the speaker attributes SQLite’s extraordinary adoption to “providence” — forces outside anyone’s control.

2. Pithy and provocative quotes

“SQLite exists because Informix did not work.”

“I thought I can write code that is practically bug free. It just runs. It doesn’t ever have any problems. … I was beginning to realize maybe I can’t write perfect software after all.”

“The rule at SQLite: if it hasn’t been tested, it doesn’t work.”

“Don’t be afraid of making your test code 10 times bigger than your actual product code. … Don’t be afraid of making 10 to 20% of your source code be useful only for testing purposes.”

“A young programmer … said ‘I’m just a tester.’ And I thought for a second, that’s really all I do.”

“I claim absolutely that in order for a project to become like SQLite, it requires an element of providence.”

“We do not trust compilers. Just because the source code is correct does not mean that the compiled code is correct.” (SQLite has found bugs in GCC, Clang, and MSVC.)

“The AI found a pathological 1 million entry input that caused quicksort to overrun the CPU stack. … Correct me if I’m wrong, but I don’t think that Zig or Filc or Rust or Go or anybody else is going to help me here.”

“Write your own code, but include that prompt as your comment. … Your comment should be a sufficient prompt for an AI to reproduce the code.”

3. Tools, practices, and methodologies

100% MCDC (Modified Condition/Decision Coverage) — Test coverage that requires every branch at the machine-code level to be taken both ways, and every bit in a bitmask test to independently influence the outcome. Measured via GCC’s gcov and custom scripts. The speaker says it’s the foundation that made bug reports “just kind of stop.”

TH3 test harness — A custom C-based test harness that tests the exact deliverable object code (not a special debug build). Achieved 100% MCDC in 2009. It is maintained continuously and is over six times larger than the SQLite source.

Pluggable VFS (Virtual File System) — An internal interface that lets tests substitute fake OS file operations. Used to inject deterministic I/O errors and to simulate power-loss crashes by recording, snapshotting, and corrupting file operations.

Fault simulation (faultsim) — A function that always returns false in production but can be rigged via sqlite3_test_control to fail on specific calls (e.g., pthread_create). Used to test error-recovery paths that are otherwise nearly impossible to trigger.

Rigged memory allocator — A substitute malloc that deliberately fails on the Nth allocation. Tests loop through every allocation point to verify graceful unwinding and no memory leaks on OOM.

Assert macros (never, always, testcase) — Macros that behave differently depending on build mode: in coverage measurement they force branches; in debug they panic on invariant violation; in production they pass through. Over 7,500 asserts in the codebase.

Custom fuzzer (based on libFuzzer) — Fuzzes both the database file and SQL input simultaneously with a custom mutator. Later extended with semantic fuzzing to check that a query and its equivalent subquery return the same result.

Fossil version control — A DVCS with cryptographic artifact hashing, contemporaneous with Git. The speaker uses it for configuration management and situational awareness, inspired by DO-178B.

DO-178B (avionics software standard) — A concise 84-page document that shaped the speaker’s thinking on requirements tracking, configuration management, and testing. The core takeaway: “if it hasn’t been tested, it doesn’t work.”

Checklists — Used heavily to manage the enormous testing surface; “you can’t remember it all.”

Design for testability — The product was retrofitted with hooks (sqlite3_test_control, pluggable subsystems) to enable the testing described above. The speaker insists this must be done from the start, not after the fact.

4. Unanswered questions and omissions

Mutation testing remains unsolved. The speaker admits he cannot get 100% reliable mutation testing because some branches (e.g., a hash function that always returns zero) still produce correct results, so tests don’t break. No solution is offered.

How transferable is this to other domains? SQLite is a single-file embedded C library. The talk doesn’t address how these practices apply to distributed systems, networked services, or projects in languages without the same low-level control.

The cost of maintenance is acknowledged but not quantified. The speaker mentions “80-hour days” and a “long year” to reach MCDC, and shows ongoing check-in activity, but doesn’t discuss the ongoing resource burden or how a small team sustains it without burning out.

The Fossil vs. Git argument is teased but not made. He says “Git is okay, but there are better” and invites private discussion, leaving the audience with no concrete comparison or criteria.

AI’s role in testing and fixing is left ambiguous. AI finds bugs, but its fixes are poor. The speaker doesn’t explore whether AI could be integrated into the test-generation pipeline or how to close the loop.

“Providence” as a success factor is a hand-wave. After a rigorous technical talk, the final claim that widespread adoption requires luck/providence is stated without evidence or exploration. It sidesteps questions about marketing, community building, licensing strategy, or platform bundling deals that likely played a role.

No discussion of security testing. Despite the emphasis on reliability, there’s no mention of fuzzing for security vulnerabilities, threat modeling, or supply-chain integrity beyond compiler bugs.

The talk doesn’t address how to introduce these practices incrementally into an existing, untestable codebase. The advice is “design it in from the start,” which may not help teams with legacy systems.

Thank you Isaac, you give me too much credit. Seriously, Seriously. I've got a lot of slides here, you might want to follow along, I might go quickly, I don't know but this is a very technical talk. I love giving technical talks and so we're going to Talk Technical here. SQLite exists because Informix did not work.

We're back in the 1990s I was a subcontractor I was two major corporations removed from the end customer and my job was to write some client programs that would talk to an Informix database, pull some data off of them, find good approximate solutions to an NP complete problem and present a compelling graphical display of the solution. And that was a fun problem and enjoyed working on it. The two major corporations in between me and the customer took all the profit. I did all the work but I had a lot of fun doing it. But along the way Informix was supplied by the customer, specified to the customer.

I didn't have any choice in it, I didn't have any control over it. And sometimes Informix would go down. This was ultimately traced to a configuration problem the system administrator didn't set it up right or something but when the informant server would go down my program had to paint a dialog box and before you hate on me for this dialog box that was the state of the art dialog box in the 1990s. And so what else can you do? I mean I can't talk to the date, I can't get to the data I need, I have to put it up.

But the end user, they would see this and then they would send me the bug report that made me very sad, that caused me pain to use Andrew's words. So I thought do we really need that server? Why can't we talk directly to the data on disk and get rid of the server in the middle? That's going to cause me this pain, why do I need that? And so I thought, well I looked around, I couldn't find anything like that.

I thought well I'll just, I'll write my own database engine. How hard can that be, right? Turns turns out it's harder than you'd think but so that was, that was. That's kind of the origin story of SQLite. It's built because the software was not working and it's grown over the years.

It took me a while to get it going but in case you haven't heard of it, when I made these slides I didn't realize that everybody knew what it was but in case you haven't Heard it. It is a full featured SQL implementation. It's power save asset transactions, a C library. It's not a system, it's not something that can go wrong. A database is a single file on disk.

You just move the file around, you can email it to your friends, which is pretty cool. Well defined file format. You might not know that the U.S. library of Congress has designated SQLITE database files as a preferred storage medium for long term archival storage of binary data. I did not know that myself until somebody on social media pointed it, brought it to my attention.

But it's backwards, it's in the public domain. We release everything freely so there's a team of us working on it. It's a small team. Right now there are three committers including myself. All the code is in a single file of C.

So it's really easy to install in your project. It's a big file, you know, over a quarter million, quarter million lines long. But we don't actually edit that one big file usually we've got a bunch of individual source files and our build process is to kind of concatenate C code together to make one big file. It's all handwritten. Well I say that we do write programs that write C code.

We're not using agents to write code, but we do write our own programs to write code. And so some of it's built that way. So anyway, this, the project Design lifespan is 50 years. First code was on May 29th of 2000 and all of my friends are retiring and you know, they're you know, spending time with their grandkids and they're Richard, when are you going to retire? Well, in 2050.

I've got 24 years to go, so at least that's the plan. We'll see how that goes. And to my great surprise, it has become possibly the most widely used bit of software in the world. As I see pointed out at the beginning. He stole a slide from my talk.

It's used in all your mobile devices right now sitting around you where you're sitting within probably five or six feet of you, there's probably hundreds of instances of SQLite quietly running and making your life better. And that's just amazing. There's no way to know because it's free and most people use it and don't even tell me so. But good estimates are well over a trillion SQLite databases in active use right now. Responsible the people at Apple and Google tell me that about half of The File System IO on your phone is SQLite.

Most of the other Half is media files, big media files that are going back and forth and people do write XML and whatever and other database engines, but Most of it's SQLite. So how did this happen? Well, a few years ago, this cartoon appeared and, oh, they're talking about SQLite. I'm not sure. I've never actually been to Nebraska and I do have people helping me, but that kind of gives you the idea of what's been going on here.

So why is it used so much? What is it about SQLite that it has become so popular that it went so viral? Well, one, it's useful. It solves a problem. People need to store their data.

The saying is that without data, you're either lucky or you're wrong. So you need to store your data and everybody wants to do that. It's easy to deploy. It's just one file of C code. You just drop it in your product program and compile it and away you go.

It's open source, public domain, it's out there. You can do anything you want with it. And people do do very unnatural things with it, but that's the freedom that comes with it. And you don't have to pay anything, which improves its adoption. There's no low dependency.

I mean, it just uses the standard C library. It doesn't have a big supply chain to worry about, doesn't use much in terms of resources. It runs on your iWatch, it runs on small embedded devices and it usually doesn't crash. So, you know, so the reason is that it became popular, I think, is because it solves more problems than it creates. Which is kind of a good life philosophy in general.

I mean, if you want to be successful in your career, if you want to have a good career, if you want to do things, solve more problems than you create another way thinking, another way of thinking that is, it just works. And as I've discovered since I've been here, there's like a, a huge community of people who just enthusiastically embrace SQLite. I mean, I saved this tweet from a few years ago. I don't know what they were talking about. I don't know who Mr.

Krishna cough is. I agree, all modern tech is inherently bad, except for SQLite. He may have an overly positive opinion of SQLite, but I do appreciate his enthusiasm. Thank you very much, sir. So anyway, it took me a while to kind of get this going.

There was, we were on SQLite version three and it came out in 2004 and there was a version one and a version two. And see I've never had. I've never had any formal training in database technology. And Dan, who's contributed maybe almost half the code, has had no computer science background whatsoever. He's a mechanical engineer.

And so we had to learn a lot for a few years. But when we came out with version three, it really began being adopted by the tech titans of that age. Does anybody remember America Online? They were a big customer and they paid. They gave us some money to work on some things.

Things. And the world's largest manufacturer of cell phones at the time was Motorola. Nokia got involved with this. But before this I thought I can write code that is practically bug free. It just runs.

It doesn't ever have any problems. But about this time it was going viral and an obscure startup company in Mountain View was starting to use it. You might have heard of it called Android. And they were sending me like a bug report daily. And this was frustrating and I was beginning to realize maybe I can't write perfect software after all and what can I do about this?

And I was also doing some work for Rockwell Collins and that's an avionics manufacturer. And they told me about this. This document called DO178B. Does anybody ever heard who's familiar with this? A few people.

It's obsolete now. Apparently. There's do 178C. I never purchased that one. What I like about this is that it's very succinct.

It's only 84 pages long. Most of the quality assurance documents are like bookshelves full of stuff. This one's very short, little brief thing you can go online and buy. Costs $6.50 per page. I'm not making that up.

They think very highly of their PDF but it gives you a really cool way of thinking about software from the avionics point of view. And there's three main elements they look at. They look at the requirements analysis and tracking. We do that at SQL. I'm not going to talk about it today.

They talk about configuration and lifecycle management. And I've got one slide on that. Most of the rest of this talk is going to be about testing and how we do it and how that makes it more reliable and all the problems we've had and how difficult that is. But why, that's really what you ought to be doing. So my one slide on configuration management is that DO178B inspired me to run my own version control system for SQLite.

It's called Fossil. And people say that I'm crazy for that. I Should be doing everything in Git, but this was contemporaneous with Git. Git and Fossil started this. They're both derived from another version control system you probably never heard of, called Monotone.

Anybody heard of Monotone? A few people have heard of that. Monotone had the original idea of identifying the artifacts using cryptographic hash. And so Git uses that. Fossil does that.

It's the same basic idea, it's just different implementations. This is a whole nother talk on Fossil. And if you want me to give you that talk in private after this, let me know. But the big thing to learn from D0178B is that the idea, if it hasn't been tested, it doesn't work. And so the rule at Sglite is that we follow 100% MCDC or 100% modified condition decision test coverage.

And that's a mouthful, MCDC. And it's a very technical meaning. And you can look up the details of the technical meaning on Wikipedia if you want to for this Talk. And in SQLite, I'm going to narrow it down to two things. With this test coverage, that means that every branch operation at the machine code level has been tested in both directions.

And two, every bit in a bit mask test makes a difference in the outcome. So for example, if you've got a piece of code that says this is made up code, Obviously A equals 5 and B is 7 or C is 11, you've got at least four test cases. To verify that this works, you've got to say that you've got to have a case where A is not equal to 5 and then by the virtue of short circuit evaluation, you don't need to deal with anything with B and C in that case. But if A is equal to 5, you have to do B equals 7, B not equals 7, C 11 and B not equals 7, C not equal 11. Do you see how that goes?

So you've got to verify that you have all those cases covered. You have to have test cases that cover all those conditions. And if you're doing a bit mask test like x and 7 not equals, you need to make sure that each bit has been tested separately in that test. And there are other things about this, but this is the idea of what you have to do. So how do we actually measure this and our product?

It turns out that GCC allows you to compile things in a special way and then you run the program and it gives you the program called GCOV, generates a big file. In the case of SQLite, it's about 15 and a half megabytes, 320,000 lines of output. That shows you how many times every branch operation was taken. And then you develop a script or something to process that file and it shows you how many times every branch is taken. And in this case, we've got 100% MCD coverage on that particular line of code because all the branches were exercised.

But it's looking for cases where the number of times taken is zero. And then. But how do you do the bitmaps thing? So we've got a macro called TestCase. And when we're testing coverage, TestCase actually puts another branch in the code and we have to give it something to do.

I wrote no OP there, but it does something to the optimizer, doesn't optimize it out. You have to give it something to do. And then you put these test case macros throughout the code that verify in this case that every one of those bits that we care about in that bit mask has been tested when it's both set and clear. But notice that in a deliverable test case is nothing. It only runs when we're, when we're checking to see if our test values are right.

You can also use test case for checking boundary conditions. This is also very useful for making sure you don't have off by one errors that you check that make sure you're at a test case. We're on both sides of an inequality. But the thing to realize here, and D0178B goes into great detail on this, is that you've really got. You're testing three different times.

There's three different kinds of tests that you're interested in. You've got. First of all, you have to verify that your test cases are actually testing what you think they are. You have to make sure your test cases are correct. They're testing all the requirements to make sure all the requirements are fulfilled.

And did you have full test coverage? So that's number one. And then you're testing the source code. And there's lots of ways to do this. You might make sure the source code is the right answer, but also you've got sanitizer, various static analyzers, this sort of thing.

But finally, we're also testing the deliverable object code in DO178B. We do not trust compilers. Just because the source code is correct does not mean that the compiled code is correct. And as it happens, SQLite in its history has found bugs in each of GCC, Clang and MSVC and you can still go look in the source code and find bug workarounds. I mean, all the bugs have been fixed now in the newest versions, but people still use older versions of the compiler.

So we have workarounds for these bugs and you can find them in the SQLite source code. So how do we actually test the product? It is a C library. So normally an application would talk to the C library and the C library then talks to your file system to get the answer. But SQLite's not in the monolith.

We've got these plugin, there's various ways of, there's various subcomponents that we can use and control. So when we write a test program, it's just like another application, right? I mean it just talks to the library because we're testing it exactly the way we fly. And what the test program can do is we have special interfaces that we can go in and we can substitute variations on these pluggable interfaces that allow us to do interesting things. For example, we've got this module called vfs, which is what we use to talk to the operating system.

And we can plug in alternative vfs that do that, inject errors in the vfs that, that we do this with the ones we write. The operating system interfaces are called by pointer. In other words, we don't call the operating system interfaces directly. We have a table of pointers to the operating system entry point and we have a published API that we can change those pointers. I'll show you, this is actual code.

We have a table of pointers of pointers to the APIs. And so if we need to, you know, how do you make the, how do you make the open system call fail in a predictable way? That's hard to do. But what we can do, we can test the product by substituting an alternative pointer there using a well known API. And we've got a wonky version of Open that will raising error for us in a predictable way.

And then we can verify that the product deals with that error correctly out of memory testing is a big deal. I know a lot of y' all program on Linux and Linux processes never. Malloc never fails. No, no, seriously it doesn't. Because if you allocate too much memory, it, it just kills some other random process on your machine and steals its memory, right?

I mean that's the way things go now. And even like, correct me if I'm wrong, but I think most, you know, if you're programming in Rust and you run out of memory, it just kills the whole process off, right? So. But SQLite runs on things like your watch and stuff where running out of memory is a bad thing and they don't have much memory to begin with. So we have to have to deal with the case of what happens if Malloc fails.

What happens when you run out of memory and we want to unwind the stack gracefully, return an error and not leak memory in the process. So we do that by. You can substitute an alternative memory allocator. You don't have to. The default is to just use system Malloc.

Right. But you can substitute an alternative at start time. And we do that and we rig this, we substitute a rigged alternative that will deliberately fail memory allocations. And we do this in a loop. We have a test test module and some code we want to test and we do this in a loop.

We said fail on the first allocation, make sure you get a message, and you fail on the second allocation, and so forth. And you keep doing this until you get all the way through the test without ever failing. In other words, you've done all the allocations. And in this way we find a lot of errors in out of memory paths. Same thing with IO testing.

We can plug in alternative I O mechanisms and instead of using the real I O that's provided by your operating system, we can intercept those calls and inject errors. And we put it in a loop and we inject errors at various parts of the test and make sure that they're all captured and handled appropriately. So I don't know if you remember one of my early slides, one of the requirements of SQLite is that power safe acid transactions. That term power safe. I made that up really, when an acid transaction supposed to be power safe.

By power safe, I'm emphasizing the fact that you can be in the middle of a transaction, part of it's been written to the file system, part of it's still in memory pending the write and you lose power or the operating system crashes or whatever. And it doesn't all get into disk. The idea, the spec in SQLite is that even after you power back up, either the entire transaction made it to disk and is complete, or the whole thing is rolled back, which is a very important property for databases. How do you test that? I know in Cupertino, California, Apple has, I don't know if it's a room or a building just full of machines and they're hooked up to switches that automatically kill their power every now and then and reboot them looking for this kind of error.

I mean, and if you have millions and millions of dollars to devote to loggery, you can do stuff like that. I don't have that kind of money. So how do we do that? Same idea with crash testing. We can plug in an alternative backend, a fake file system that records all of the file operations.

And it takes snapshots of this file system after every system call and then it pulls these snapshots and it corrupts the file system in ways that might happen if you lost power. It's assuming out of order rights, so it puts the writes in a random order and forgets half of them or some of them. It does things out of order and this scrambles things around, does this over and over again and make sure that the transaction either made it to disk completely incorrectly or that it was completely rolled back. That's a lot of testing. All of this goes into, oh, here's the diagram of how we do that.

So all of this goes into. What I was going to say is that when you're writing tests, it's not sufficient. You don't have a complete product and then you write tests for your completed product. The product has to be designed to be testable from the get go. And so when I was developing teach, or when I was developing the test for SQLite, I had to go in and change SQLite itself radically.

And one of the interfaces I had to add was SQLite 3 test control. It's just a published interface. It's available in every version of SQLite that you use in your application. Probably you're not using it, you shouldn't be using it. In fact, the documentation for this says this is for testing purposes only, you ought not be using it.

But it's included in the builds because otherwise we wouldn't be testing what we're flying. So it does things like turn query optimizations on and off or just query parameters. I'll talk about the fault SIM thing, which is really cool in just a moment, but there's a pseudo random number generator in SQLite that it uses. You can use this interface, for example, to seed it to a known value so that your tests are reproducible. There are some modules within SQLite that are just inherently difficult to test.

They have their own unit tests and they're built in. And we can run those unit tests using Test Control and much, much more. There's this internal function in SQLite. The design rule is that functions that begin with SQLite 3/ underscore are published APIs and those that begin with SQLite 3 and don't have an underscore after them are internally use only and subject to change. So this is internally use only and this function faultsim or for fault simulation in production, it always returns false.

It should never be true in a production build. But using the test control API, we can make it return non zero and we can do that. We can make it return whatever value we want. And it doesn't have to do this every time. It can just do that sometimes, like on every nth call or something like that.

And so here's an example of where it's actually used. I mean, we need to make sure that SQLite doesn't commonly produce kickoff threads and in fact it never does unless you ask it to. But. But how would you test the error recovery for pthread create not working. How would you do this otherwise?

So here we set a fault simulation that causes it to pretend that pthread create was unable to create a thread. And how are you gonna recover from that? There are 24 instances of that in the latest code, which you're welcome to go check out for yourself. So just a little bit of background. Some of you might be familiar with the TCL language.

Does anybody know tcl? Had experience with that. It's a really great language. John wrote it about 1988. This is.

Reading his code really taught me to program in C. I mean, I knew C before. You can get some of my code from before I read his code and afterwards and it's night and day. It's a totally different world. I really appreciate that.

And I was actually part of the TCL core team for a while. Anyway, what you might not know is SQLite really started out as a TCL extension. Excuse me. You know, it used to be called tcl and then there was this thing called me too. And it was determined that TCL was too risque.

And so now they've rebranded as tcl. So there's that. I'm still. Old habits die hard. But anyway, SQLite was a TCL extension that escaped into the wild.

It really started out as a tcl. That was its whole purpose. So naturally, all of the testing in SQLite is done in TCL. At least the initial testing for the first few years. That's all the tests we had currently, and they're still there.

If you pull the source code and type make test, it's going to run the TCL tests. And if you do a make test, I think it's. Well, I checked the other day and it's 14 and a half million different tests now. There's really only like 20,000 cases but a lot of them are in loops with the parameters varying. So it comes out to 14 and a half million to run the whole thing.

It seems like it was, it's in my backup slides, I think it was 12 CPU hours or something like that. It's part of the source tree. Six and a half times larger than SQLite itself is the code to do all of this. But it's a custom build. In other words, in order to use the TCL tests, you have to give SQLite some special compile time options that you would not normally give it in production.

It's really a source code test, it's not an object code test because it's not testing exactly the same code that you're delivering. And we wanted, because of DO178B, we wanted to test exactly what was being delivered. And so I had this idea of writing a new test harness which we call TH3. Yes, there was a TH1 and TH2. They didn't work out and I started that in 2008 and it achieved 100% MCDC in 2009 and that was a really hard year.

A long year of 80 hour days and thinking that why am I tormenting myself this way? But really it was very interesting. Once we got to 100% MCGC, those bugs that we were getting from Android and elsewhere, they just kind of stopped. They really did. I was shocked.

I didn't really believe stepped off the edge, I didn't believe that this was really going to happen. But it worked very, very well. And then for several years after that we really didn't have any bugs coming into SQLite. We'd achieved bug free code or so I thought. Yeah, those were happy days.

We could focus on making the product better but understand that th3 was not a one and done. When you think oh well, I've got my 100% MCDC testing now I'm done, I'm finished with that. No, it's an ongoing maintenance thing. This is a snapshot I just did this morning of all the number of check ins to th3 non merged check ins to th3. And as you see there was a lot of work in the early two years but I mean it's ongoing even to this day.

There's a lot of maintenance work. Every time we make a change, every time we add a new feature, every time a new bug is found, we have to update TH3 as well. So there is a maintenance burden involved in this. But it, but it pays good dividends now. So 100% MCDC testing is not the only kind of testing we do.

You know, I'm going to say, I'm going to make the bold claim that your code comments, it's part of your testing code comments are very important. This is, I hear so many people saying, oh, you know, if you write your code right, you don't need to put any comments because it's self explanatory and I don't think that is correct. I believe very strongly that you should have human readable human language comments on all of your functions and procedures and variables. Concise, not boilerplate, not boilerplate comments. Don't fall in that trap of you've got a particular format and this is so that people not yet born can understand what you've written.

Also so that you, at least in my case, so that I can understand why I did something a week after I'd written it. But also because as humans we have different neural pathways for writing formal language, writing code and spoken language. And you may have experienced this, you may have some experience with this. Who's ever experienced something like this where you're working on some problem. Hey Carl, look at this problem with me.

I've been working on this for hours. X clearly cannot be zero because. Wait, wait, no. Oh, I got it now. Never mind.

Thank you, Carl. You've all experienced this, haven't you? And it's because, it's because you know, you were thinking in a formal language, you were writing code and you were not using human language and you're using different neural pathways. And this works the other way as well. If you're thinking in human language, you're in meetings designing something or you're prompting your AI or whatever it is you're doing in the human language and you think, oh, this is a great design, this is going to work out great.

And then you stand to write the code and you immediately realize, oh, that's never going to work. It works both ways. You need to do both. Use your whole brain. It's a very important feature.

About one third of the SQLite source code is comment and it's not useless. It's very important for it being there. We used heavy use of assert and this was a controversial thing. I never thought assert would be so controversial. Apparently, in fact we had a big fight with go about this.

So I. So the idea of assert, of course is that it's an invariant. If an assert ever fails, that's a bug in your program. Okay? But it turns out that and think of assert as an executable comment.

I'm reading code and I see a comment and I see some code. I'm not going to assume that the comment and the code agree, but if I see an Assert and I know this thing is passing tests, then I, then I believe the assert. So, but the developers of Go, they were thinking, look, we've seen a lot of people and they use assert to validate inputs, and that's wrong. And so we're going to forbid Assert from being in Go. And because of that, I had some sharp criticism of Go on my website and the Go people took umbrage at this and they contacted me and we had a meeting and they agreed to tone down their rhetoric on the cert, and I agreed to tone down my rhetoric against Go.

And that worked out great and everybody came away happy. Escrow makes heavy use of Assert. We really lean on it heavily. There's over 7,500 of them in the code right now, and that's a very important factor. In fact, Assert, normally it's on by default.

We've done some if deferee magic to make it off by default, because when asserts are on, the code runs four times slower, and nobody wants that. You have to compile with SQLite debug in order to enable asserts. And it's more than just simple a search statement. We have entire subroutines or procedures for checking invariants that are omitted from production builds. So, you know, here's something in the page cache we're gonna, we're checking all of our invariants for an object out of the page cache.

And this runs when you do a debugging build. And so that we know that our page cache doesn't have some subtle problems that aren't even showing up in our tests, We have Assert, like macros never and always, that we can put around test cases. And in production, code never is a no op and always is no op. They just go straight through. But they identify branches that we believe, or in this case, it never identifies a branch or a condition that is always false.

So we never expect that branch to be taken. And always, of course, is the inverse of that. And this is a very powerful way of doing error checking. Think of it like if you're writing Zig or Filc or something, and you want to do bounds checking on an array or something, you're going to insert branches that should never be taken, because if there's no bugs in your program, then you're never going to read outside the array, right? So never and always or that way.

And we do have never and always macros that we make use of. And whether or not they, the way they function depends on how you're testing, if you're doing, if you're measuring coverage, never, always is just a hard coded to one and never is hard coded to zero. If you're debugging, they raise, they panic if you, if the condition is wrong, but for production, they just pass through. The idea is that maybe, maybe you've missed something, maybe there's some bug and maybe the never or the always is going to save you. So there's a lot of these macros, there's test case, always, never, insert, always.

We use them all over the place and their behavior depends on what you're doing. If you're testing your test cases, they behave one way. If you're testing source code, they behave another way. And then in production, they behave a third way. This also, I think conflicts with a core GO doctrine that your code should only ever compile in a single way.

But I don't know of any other way to do the kind of testing that we do at SQLite. Now, we talked about MCDC testing. You can go even further and do mutation testing. By mutation testing, I mean you make a small change to the source code or the object code and then you verify that your test break. This is a very powerful thing.

But understand, it's not testing that your program is correct, it's testing that your test procedures are really checking your program really hard. It's problematic though, because this is the actual hash function used for symbols inside of SQLite. And, and what if the branch instruction that implements that not equal to operator. What if it's always an unconditional jump? Then the hash function always returns zero.

Okay, that still works. It might not be quite as fast, but it still works. So how do you write a test case to verify that that branch is doing the right thing? That's a hard thing to get right. In general, there's a lot of places in SQLite where we're checking some condition that's very common.

And then we have some very fast algorithm that we can do it and that contains. But sometimes we need to fall out to a more general case that's a lot slower. If we take a false positive, if we take a false negative and shortcut works, we're going to do the slower case. It's still going to get the correct answer, but we can't really, we're going to miss the fact that the code was corrupted. So I haven't yet figured out how to do a reliable mutation testing requirement.

I can get code to be 100% MCDC. I can't get code to be 100% mutation testing reliably we do a bunch of other testing with test on different platforms, different compilers, different architectures. Notice we Big Indian vs Little Indian. Back when I first started this big Indian processors were still relatively common. These days you can't buy them really.

I have a. I went on ebay and bought a circa 19 or 2004 MacBook so that I could test on a power PC. And finally we disable optimizations. There's a bazillion tests. We keep track of it all with checklists.

We make heavy use of checklists because you can't remember it all. And so we spend all of our time testing some. A few years ago I was talking to a young programmer right out of school, young woman. And she knew who I was. And I asked her what did you do?

And she felt really bad and she said I'm just a tester. And I thought for a second that's really all I do. And really the anthropic thinks that's what everybody should. You write the tests and Claude writes the code. That's their goal, right?

I'm not sure if that's right or not, but there are I came up with for doing all these testing I came across a lot of benefits that I did not expect. I just wanted to get rid of the bugs. But there are a lot of other benefits that come into this. For one, we can maintain this code base which the world depends on, with just three committers. How many committers are there in postgres?

I don't know, 100, 200? There's a bunch. Are there any other projects that have three committers that are this big? I don't think so. And the only way we can do that is that we we have this coverage.

We can refactor big punk sections of the code, strip out entire subsystems and rewrite them from scratch. And do that in a point release with high confidence that we haven't broken anything. That's very important. It makes it a lot easier to find and fix bugs and it makes the code faster. Honest engine.

What? Here's a graph of actual performance measurements. Billions of x64 CPU cycles needed to run a particular performance Benchmark. Starting in 2009 when we first got 100% McD. Because we have this high level of test coverage, we can make small micro optimizations.

No one optimization is is even Measurable. But you put in thousands and thousands of them and we have tripled the performance of the, of the product over the course of years. It now runs three times faster than it did in 2009. More than 3,000 reach more than three times faster. You know, recently I've seen on social media a fallacy or an unspoken assumption that source code is distinct from test code, that these are two different things.

And I've been seeing that there was a big argument that they, because somebody rewrote, they used the test cases for postgres to rewrite Postgres in Rust. Well, their agent did it. They just used the test cases and all the arguments back and forth on either side. We're assuming that the test cases and the code were distinct. But I've showed you that at least in the case of SQLite, there's a significant overlap between the test cases and the code.

Even about 15 to 20%. You know, in the chip design community in integrated circuits, about 15 to 20% of the transistors on your new CPU are only used during manufacturing testing. They're not used after the device is packaged and put in your computer. So 15 to 20% of your code, maybe that should be your target for how much of your code is actually used for testing. And it seems like dead weight, but no, it's there because to make it faster, they're smaller once you take out the source code testing part, but still it's there.

And as I was writing this, if we scale this closer to the way SQLite does it, we see that the test code is significantly larger than the source code. And I was thinking, so many of these people are telling me that I need to be writing just write the tests and then get an agent to write the code. And this will vastly improve my productivity. And maybe I'm being dense, but I'm not seeing how that's really going to be that helpful. Somebody can maybe help me understand better why that works.

Okay, so everything's going along peachy for a while, but then we came across profile guided fuzzing. You're familiar with this. You're familiar with American Fuzzylop and Live Fuzzer. This happened in about 2013 and we didn't have any bugs. 2009 to 2013, really.

But boy, that sure started happening here because it turns out that code that is MCDC 100% MCD tested is very vulnerable to fuzzers. Because when you do MCDC testing, you're taking out a lot of branches that you couldn't reach or you're at least making them nevers or alwayss or certs or something. But these fuzzers are remarkably good at finding very creative ways to make branches flip. And so this gave us a lot of grief for a couple of years. We eventually wrote our own fuzzer based on Live Fuzzer with a custom mutator.

This is specific to SQLite. It fuzzes both the database file and the SQL at the same time. And once I got that dialed in and figured out the fuzzer bugs kind of stopped. And that was great. In other words, we were finding the bugs ourselves before the external bug finders were, and that was great.

And then a few years later, manual rigor came up with this idea of the original fuzzers were just looking for memory errors or search and faults or something like that. He came up with the idea we can do fuzzing ideas to test for inconsistencies in SQL. Like if you have a query that returns a tuple at the top, you can use that as a subquery and you should get the same tuple out, right? And he found a lot of bugs that way. Now, these are not bugs that you would ever hit in code that you would write yourself.

But there were still bugs and they needed to be fixed. So I had to then go in and enhance our fuzzer to do this semantic fuzzing as well. We eventually got that worked out as well. There's a picture of Manuel. He's Austrian guy, he's in Singapore now, working, trying to get tenure.

Really smart, really nice. More recently, we've seen an avalanche of bugs from AIs and they find bugs that fuzzers and MCDC testing do not find. And here's a typical one that occurred the other day, and it's very interesting. I thought so. The way SQLite computes median, it's an aggregate function.

It collects all the values, all the inputs, puts them in an array, quicksorts the array, and then takes the one in the middle. Fair enough. And I wrote my own quicksort rather than use qsort because QSORT has to call out to a callback to do the comparison. And. And so if you just write it yourself and just do the floating point comparison yourself, it's about 3 times faster.

So I have my own quicksort and the AI found a pathological 1 million entry input that caused quicksort to overrun the CPU stack and hence seg fault. Now, correct me if I'm wrong, but I don't think that zig or Filc or Rust or Go or anybody else is going to help me here. This is, this is just a problem. And so I've scratched my head on this for a while. The AI, they're, they're very good at finding these bugs.

They're not very good at fixing them though. The suggested fix from the AI was we need to add a depth parameter to quicksort and if it goes too deep then fall back to some other slower algorithm. But thought about this for a while and eventually figured out that look, when you're doing the quicksort, if you only do recursion on the smaller of the two partitions, you're guaranteed to converge and log in recursions and then for the larger partition you can just tail recursion and just do it in a loop. And that solved the problem very quickly, actually made it faster. And in the post action analysis I did some research on this and this was actually well known in the literature.

It just was not known to me. It was discovered by Robert Sedgwick and published when I was in high school. Robert's a protege of Don Knuth, you know, so he published it in some academic journal that I've probably never read. So whatever. So we are, we're going to work through the AI problem as well.

But that is how we got to where we are. So what have I learned in all this? This is the summary slide. This is what this whole thing is about. 1.

In order to be testable, you have to design your product to be testable. You can't take a finished product and suddenly test it. You have to design testing into the product. You have to think, begin with testing in mind. You have to design the product so that you can make it testable.

100% MCDC testing does work. I'd like to say that mutation testing works, but I haven't figured out how to make that work yet. There is a lot of work for 100% MCD testing, but it does work. Don't be afraid of making your test code 10 times bigger than your, than your actual product code. That, that does work.

You can do that. It does be maintainable. Don't be afraid of making 10 to 20% of your source code be useful only for testing purposes. There's nothing wrong with that. Write clear and concise documentation.

A rule of thumb, if you're, if you're doing agentic or if you're doing vive coding, you know, you would give a human language description to the AI to go do your code. Just write your own code, but include that prompt as your comment. That's a good guide. If you're using it, your comment should be a sufficient prompt for an AI to reproduce the code and expect to spend most of your time questioning. And finally, situational awareness.

It's so important to know what's going on in your project. So you need good version control. Git is okay, but there are better and so I understand git is kind of. It has become embedded in so many things and if you want to, if you're doing modern development, you kind of have to use git, but we've managed to avoid it. And if you can find better, do I'm not saying that Fossil is best by any means, but I would encourage you, if you like version control, to go invent your own that uses the best ideas of all of them and write one that's better still.

Now these I'm going to claim without proof, that these are necessary conditions for having successful software, software that works. But they are not sufficient. Or at least they're not sufficient for having software that has gained the notoriety and widespread adoption of SQLite. Because as I reflect back over the history of SQLite, I recognize that so many things happened that I had no control over and didn't even realize were happening at the time. They just happened and they were good things and they caused the project to grow in popularity and use and to help so many people.

And I had no control over this. It just happened. And so I claim absolutely that in order for a project to be to become like SQLite, it requires an element of providence you really have to have. And I'm very serious about it. I mean, it's providential where we are.

So that is my story on SQLite. How much time did I actually use for that? I'm over time, aren't I? I'm sorry. Thank you.
