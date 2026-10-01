---
url: https://www.youtube.com/watch?v=0GzwuYGvKA4
date_fetched: 2026-10-02
---

# Distributed databases with Peter Mattis

- **Channel:** The Pragmatic Engineer
- **URL:** https://www.youtube.com/watch?v=0GzwuYGvKA4
- **Duration:** 1h 42m 31s
- **Transcribed:** 2026-10-02

---

**Speaker A:** I met Larry and Sergey from Google. Somehow one of them hooked me up, asked me to come interview at Google

**Speaker B:** when Google was three years old.

**Speaker A:** Three years old. And I said no, you did not. I've always been a very prolific coder. I look back at my GitHub output. So like peak years, maybe a hundred thousand lines of code in a year. Pre AI, right? This is when you're back doing this manually. I kind of look at that. It's like there's kind of a max that you can hold in your head at a time. We did eraser coding classes. That was kind of a big breakthrough. You have nine chunks of data, but any five of those chunks can be used to reconstruct it. And what this means is you can lose any four copies and you can still reconstruct your data.

**Speaker B:** What do you think good software engineering looks today compared to four years ago

**Speaker A:** before we had a. I think the ambition has to increase.

**Speaker C:** He built a GIMP image editor while at college, designed the original storage system behind Gmail, and built many more large, complex and widely used systems. This is Peter Mattis, co founder and CTO of Cockroach Labs. Before we sat down to talk, Peter told me for the last 30 years I've always been a prolific coder, but my current output is a bit insane. And this isn't vive coded junk, but database worthy, high quality, high performance code thanks to working with strong coding models. Today we cover why B trees are so important when building databases and why Peter kept reaching for the data structure over and over throughout his career. How he wrote 100,000 lines of production code per year. Pre AI stop coding between 2022 and 2024 and why he is back now, why he thinks AI agents are lazy about testing and how this is an easier fix than it looks and many more. If you're interested in distributed databases, distributed storage systems, or knowing how Peter manages to use AI to produce unusually high quality and production ready code. This episode is for you. This episode is presented by TurboBuffer, a ridiculously scalable, fast and cheap hybrid search engine built on top of object storage by an engineering team I've really gone to like. After spending time with them, the Turbo Buffer engineering team is doing something really, really cool. They're completely redesigning their storage architecture from first principles to make storage faster, cheaper and more reliable at scale. If you follow Turbo Buffer, you know that their storage architecture was a massive part of their early success. Redesigning a winning architecture is a big deal. It's one thing for your query plans to pass unit tests. It's another to bring each query plan to performance parity or better, while maintaining correctness and reliability in production. Here's the cool part. TurboPuffer is documenting the whole thing. Their new storage architecture, which they're calling Tpuff V3, demands some hardcore systems engineering, and they're building it in public, sharing design decisions and benchmark results as they should. They're keeping a work log of their journey and the first post just dropped today. Follow along@turbobuffer.com V3 that is Turbobuffer.com V3

**Speaker B:** Peter welcome to the podcast.

**Speaker A:** Oh, I'm happy to be here. This is awesome.

**Speaker B:** I wanted to get into. How did you get into tech originally? When did you figure out computers are interesting?

**Speaker A:** I figured that out in kind of elementary school, high school. Gaming was a little bit of a gateway drug for me as a software engineer, as many other people I remember. Like early on my mom did programming at IBM at some point. I'm never even quite sure what she did, but we had computers always around our house like Apple II plus Apple iigs. I'm from that era on up and you know, go to the bookstore, I'd find a book on BASIC or magazine type in programs. No idea what it was doing, but you know, just kind of like I was addicted. Like you could produce, put stuff into these computers and you get interesting stuff out. And then I got to college and I was like, I didn't think there was any money in computers. I didn't know anything about it. I started as a mechanical engineer following the footsteps of my dad.

**Speaker B:** You started mechanical engineering as your specialization?

**Speaker A:** Yeah, as my major, yeah. And I got in there and I was like doing like I'd done some computer stuff before. It was a real foolish move to do this, but first semester of doing these homework assignments in mechanical engineering, they were God awful. Like six pages for a single problem. And I happened to take a CS course at the same time and it was so easy and then everybody almost was failing it. I'm like, I'm in the wrong, I'm in the wrong field, let me switch.

**Speaker B:** And then you switched.

**Speaker A:** Then I switched, yeah.

**Speaker B:** What was the first software that you built either at college? It must have been a college that you were like, all right, this is a piece of software that I'm kind of proud of. That's a complete piece of software.

**Speaker A:** Well, I mean, the big thing that I did, along with my roommate in college, we had this course. Was it a compiler's course? I'm not even quite sure Anymore. This is like 30 years ago, and we were kind of bored with it, so we wanted to do something, like, kind of fun on the side. And I'd done journalism in high school my senior, junior and senior year, I knew stuff about kind of computer graphics and wanted to do something like Adobe Photoshop, so started just kicking the tires. And we built up this program that a lot of people know of called the gimp. And along with the gimp, I did a lot of the graphics library, gtk. This has since evolved just massively since then. It's kind of interesting because after college I kind of stepped away from it. Didn't really stay involved much past my first year out of college, but definitely people still know me. And it. It led to other. Some interesting events in my career.

**Speaker B:** And. And it. It was like starting gimp, it was literally just you saying, all right, I want to do something like Photoshop. How hard could it be? And then you just. This wasn't through what you learned in college, right? This was like you figuring out how to build, you know, like a graphical, I guess, engine rendering, drawing, data structures, all of that stuff, right?

**Speaker A:** All that stuff figured it all out. I remember trying to look at some papers back then. My roommate was looking at papers. We're just it all out. And it's one of these things. You almost kind of need to be a little bit naive to start anything like that, because if you know how hard it's going to be, you would never start it. So I think there's this, like, you have to have this level of, like, if you're really, truly wise about the effort, you, like, you never get into it. But then when, as you get going, it just keeps on snowballing. It gets far further and further along. And then at some point you're like, wow, this is really awesome. But there's an interesting little tidbit associated with our work on the gimp, which is kind of a little bit known. I think we've talked about this before, but it's just kind of fascinating where we were getting to the point where we're like, we should release this to the public. And back then there was Usenet newsgroups where people would post stuff, and there's one on graphics. And I remember, like, just a couple weeks before we were gonna release our first version of the gimp, someone else came on there. Like, I've been working on this graphics program, and it did everything the GIMP did, every last thing and then some. And we're just like, oh, that sucks. I guess we'll just keep on working on it. It's been fun. And then we release the gimp. Never heard from this other guy again. And I just took a little lesson with that. There's always gonna be someone else working on your idea. You can't get dissuaded if they pre announce it. Nothing ever comes of it. And a lot of the marketing behind some of that stuff that you might hear is like, I mean, I don't know if we, like, we stole his thunder. I'm not even sure what happened. He never traced down what happened. But there's a real lesson there.

**Speaker B:** So there's a possible future where you read this announcement where someone's saying, I'm gonna build all of this thing and you go like, you know someone did it and you kind of go back and just do something else and the game never happens.

**Speaker A:** That's right.

**Speaker B:** Wow. Yeah, I guess especially with today with startups, sounds like just do your thing, put it out there at the very least. Right.

**Speaker A:** The advice I give people is like, you might have a unique idea, but most likely there's like a dozen people out in the world who've had the same idea. And there might be a couple of them working on it, but a lot of people don't even work on it. They're just like, whatever. They can't get going on their idea. So I wouldn't be concerned at all if you hear someone else working on your same idea. That's probably the case. You know, we're working on some cool stuff now at my current company. I guarantee you there's other competitors out there working on the same thing. And just know it's like it's a competition. You got to enjoy that aspect of it, not be afraid of it.

**Speaker B:** And what were some fun things that the GIMP led to?

**Speaker A:** Well, you know, I got kind of tired with graphics and that's why I kind of moved away from it. After college. I got into storage systems and I bounced around. You know, I worked at one of the early search engines, synced me, and then I got over to another startup and I met through that time period in that first startup I was doing, met Larry and Sergey from Google, because it turns out that Larry, I think it was maybe Sergey, I'm not quite sure. The very first version of the Google logo was down in the gimp. So they knew of us somehow. One of them, you know, looked me up, asked me to come interview at Google. I did back in 2001.

**Speaker B:** And this is when Google was three years old.

**Speaker A:** Yeah. Three years old. And I said, no, you did not. I said, no.

**Speaker B:** Yeah, no.

**Speaker A:** Here's the calculation in my mind. I was like, google's down in Mountain View. I was living up in San Francisco, and I didn't want to do the commute. So I went and worked at another startup for a year. And at that moment, after a year, I, like, I saw it wasn't going anywhere, and they called me back up and they said, hey, do you want to interview again? I was like, sure. Like, we're not going to be able to offer you the same stock options you got before. I'm like, okay. I can't remember what it was, and I still can't remember what they offered me the first time, but I probably would have made a lot more money if I'd taken that first. But I did.

**Speaker B:** All right.

**Speaker A:** I'm not, like, complaining, but it's one of these things. Like, yeah, Now I think back about that, I'm like, oh, a lot of good stuff came out of it. Some point along the line, Red Hat was going public. They actually offered friends and family stock during their IPO to a lot of people. We got offered friends. I made a little bit of money off that. Not a lot, but it was like, just after college. It was actually quite significant at the time, you know. So, I mean, there's a number of good things that kind of came out of that.

**Speaker B:** That, and I also use the GIMP as a. You know, I was looking for, like, oh, I can't afford Photoshop, and here's the gimp. And it did so many things. So, like, I'm sure there's so many people like, you had a positive impact of being able to use this thing. And the nice thing that I really liked about it is it was free, but I wasn't, like, stealing any software from anyone. You see what I mean? This was at a time where free software was not as common as today. Free and open source was not as mainstream as it is today.

**Speaker A:** The other cool thing I've heard from a lot of people, because people still, like, I mentioned this, like, oh, I've used it. And then some people, you know, software engineers, like, learn how to program by looking at your code. And I'm just like, wow, that was the code I wrote 30 years ago, and I wasn't nearly as good as software engineer as I am now.

**Speaker B:** And then you got into Google second time around. You said, yes. What did you start to work on?

**Speaker A:** Yeah, yeah. Well, I got in there and they said like, hey, you know, this was early days. 2002 is April 1st. Auspicious date, your start date. Yeah, April 1st, 2002. And I got in there and they're like, oh, well, you know, we're going to actually build email, Google email. And it wasn't called Gmail at the time. There's a code name internally it's called Caribou. Caribou, yeah. And got in there, started working on it, and I was kind of tasked with working on the backend threading and message storage and indexing system and worked on that, you know, pretty hardcore for the first year and a half, actually. I was on to maybe it was three years on it through the launch, which actually happened to be April 1, 2004. Do you remember this? There was a. It was like considering April's full joke.

**Speaker B:** Yeah, I remember. So correct me if I'm wrong, but the. The launch said Gmail with 1 gigabyte of storage or something. Infinite storage. And like Star 1 gigabyte. I'm not sure what it was, but back then most email providers would give you about 10 megabyte of storage, the free email providers, and then you could pay for maybe 50 megabytes or a hundred megabytes. But that was really expensive. This launch, it looked like an April Fool's joke, because who could possibly give you a free email service with 20 to 50 times or a hundred times more storage?

**Speaker A:** We were talking about this internally. It was like kind of a shock and awe campaign for the industry. I think it was actually four megabytes for like Hotmail or Yahoo Mail. And then you had this. And not only that, we had so much more storage, but it was indexed really fast. So, you know, you could do a search and it would come back almost instantaneously.

**Speaker B:** But can you tell me internally, like, when the project started, okay, we're going to do email for Google. How did you and the team arrive to the point of like, okay, we will offer all this storage and we'll do it fast because this was at a time if you can take us back. But what I remember is hard drives were still expensive. They were relatively slow. We're talking hdd. We're not talking necessarily about SSD, if I remember, but if you can take us, can you take us back of what, what it was like, what the constraints were like, and then how you innovated, like to actually, like, do something that's never been done before.

**Speaker A:** Yeah, yeah. So at the time, Google internally had this large distributed file system called gfs. Google Filesys. Yeah. And we were like, basically looking at numbers and being like, yeah, we think we can build on top of this. They have a lot of knowledge about how to do search and retrieval. It ended up being that like some of the existing systems they had for search and retrieval, they started building the prototype on there and then that was completely rewritten. That was actually what I got involved in because I got in there and there's already a prototype and then it was completely rewritten. So I was on that, you know, threading side. We decided to have message threading right from the get go, which is also kind of innovative. That wasn't common in email systems and threading.

**Speaker B:** Do you have the message thread which I assume needed data structures on the back and it needed like, you know, storage and, and figuring out the read intensity of those kinds of things.

**Speaker A:** Yeah, you know, there's some B trees involved in that. I think that might've been the second time I implement B trees and I think I've implemented them like a dozen times now.

**Speaker B:** How do B trees relate to email threads?

**Speaker A:** It's just like you have to, you know, have some storage there where you have a thread id, a message comes in, you have to look up. It was using the search index to match actually the subject of the message ID into the thread. And there were some other things taking place in the B tree as well because we were keeping track of the unread counts of, of threads and whatnot. So, you know, I can't even remember all the details now. This is ancient history. This is 2004. What are we in 20, 26, 22 years ago? So it's like left my memory. But there was definitely B trees. There was also this, you know, inverted index taking place there. All this code has since been completely rewritten.

**Speaker B:** What were the economics like in the team? You must have done the economics of being able to offer free email, which of course I'm sure you did some maths of like how it could be subsidized but to like not make a terrible, terrible loss.

**Speaker A:** Yeah, yeah, yeah, yeah. No, I mean we were kind of kicking around ideas and you know, one of the ideas that had come up at some point was, hey, maybe we can put ads on this. And it was one of these crazy things. So I didn't. Wasn't involved in this. This is Paul Buchheit went on to do some cool stuff at friend feed and Facebook and ended up, I think, a partner Y combinator at some point. But he just like one night he's like, no, I think I can just take out some of our existing ad functionality and incorporate it in there.

**Speaker B:** And.

**Speaker A:** And it just went like gangbusters. And there's a huge business that, you know, kind of grew up from that, which is kind of incredible. So, I mean, I think there's a lot of things that it's just like you kind of have a sense of what can be done, like, oh, we think this is going to cost, you know, per user a couple of dollars per year. How do we monetize that? And you know, we didn't want to charge for it. We eventually they did charge for it because you have the whole Google workspace stuff. But at the time it was more like, can we do this for free? Oh, it looks like it can be economical. And it took a little bit of a leap of faith.

**Speaker B:** And do you remember the launch? The reason I'm asking because I remember that it was an invite based system, like not everyone could get in. And I assumed that must have been to control the expected demand because again, you were offering something for free that was paid before and it was kind of pretty obvious that there would be massive demand. How did you think about it? Kind of monitor demand, decide how many people to onboard.

**Speaker A:** Honestly, this is a team effort and I wasn't involved in that. I mean, I do remember being worried about the load that would happen. And then someone came up the idea of like, hey, we should do an invite based system. And this also had like, it played dual roles. So it kind of constrained the growth, but it also kind of drummed up this excitement. Oh, can you get me a Gmail invite? I remember people asking me at the time, I was like, yeah, I can get you as many Gmail invites as you want.

**Speaker B:** And then after, after, after you built the system, where did you move on to? What was your next project? What was, was it the build system?

**Speaker A:** Well, there was the build system and that was kind of all muddled in my mind because I was kind of doing that part time when I was doing Gmail as well. At some point, you know, Google actually had this large Monorepo. I think they actually still have the Monorepo.

**Speaker B:** They still have the Monorepo.

**Speaker A:** They started out with the Google One repo. That was before my time. Then they moved on to Google 2. That was what was there when I got to Google. And at some point we saw the strains of Google 2 and Google 2 was just one single monolithic make file. I think it actually had some submake files, but it was this like really large, unwieldy make file. And someone came to me and was like, I think we could do something. I'd had some interest in the build systems and I, you know, kind of put the foundation in for Google3. There was a bunch of people involved. I was kind of doing like kind of the initial work on Google 3. And Google 3, the initial insight was like, hey, make files kind of suck to write. We introduced this thing called build files and I decided to do it. It's just this stripped down kind of Python language, but it was still Python at that point. And what G Config spit out at the end was a monolithic make file. But when you didn't have to write over time, this evolved. It became Blaze internally. There's some other systems that are associated with now. I don't even know how complicated it's got. I haven't seen it for a long, long time. But then that became Basil. Externally, it became Buck. There's some other, you know, people went to Facebook and they like, that was awesome.

**Speaker B:** At Google, what was the reason that make make files or make didn't really work was.

**Speaker C:** Was it.

**Speaker B:** Are we talking about build performance? Are we talking about maintainability or readability?

**Speaker A:** Yeah. So like, I think make itself is like, it's okay in declaring dependencies, but it's a little bit like kind of assembly language. And you didn't want to write, you know, all your dependencies in assembly language. And if you didn't know what you were doing, it was easy to make a mistake and miss dependencies and whatnot. And so the kind of my. The thought behind the build files is like, hey, you need to express these dependencies, but at a higher level and just like kind of cleaner semantics associated with it. And from that you can compile down into the assembly language. And then eventually folks were like, oh, you don't have to compile down to that. We can just kind of implement, you know, the dependency kind of update engine directly.

**Speaker B:** And then that's where the performance improvements were able to come in. So like, because at Google scale, like when you have a large repo in general, like, my understanding is that the reason Bazel and Buck are so popular for large code bases is it can help you improve your build performance. It gives you a lot more lever levers to play around with, from caching, from being smart about cache generation to obviously just raw performance.

**Speaker A:** Yeah, that's exactly it. Yeah.

**Speaker B:** So you kind of just dabbled and like, okay, I'll make this build file. Did that. People took that over. But what was your next main focus?

**Speaker A:** Well, so I mentioned GFS earlier Google file system. And at some point we realized that there was some limitations in GFS scalability bottlenecks. Because I've been working on the storage system for Gmail, I actually dabbled in another storage system. You know, kind of a research thing that never went anywhere. But because we're working on that, got invited to participate in like the founding team of Colossus, which was the successor to gfs. And as far as I know, Colossus still ex. It's gone through multiple iterations at Google, but it's like the second generation, you know, distributed file system.

**Speaker B:** So what is Colossus?

**Speaker A:** Yeah, so when I say distributed file system externally you might think of something like S3 kind of blob storage. It had a flat namespace, kind of like S3. You give names and there's like a minor hierarchy there, but it's like very limited. But it's not like a POSIX file system. So you don't have the full directory hierarchy. You didn't even have the full kind of permission system. A lot of that stuff got it added later. But the files are not stored on your local machine. There is a fleet, you know, kind of a service out there that has all the files. They're writing it down to their hard drives or no SSDs and your client can access that and it's all replicated. So if there's any crashes and whatnot, you're not losing your data. I don't know when at some point S3 got erasure coding. We did erasure coding in classes. That was kind of a big breakthrough, which coding we used Rees Solomon. So for the audience who's not familiar with erasure coding, you might think of like, I want to have replicas and there's a. This thing in hard disks called raid where it's like, I don't actually have to have full replicas. I can actually, you know, if you take A plus A and B, you can xor them together and then you can have this kind of third kind of version and there's more and more complicated versions of that. Reed Solomon is kind of, you know, I think there's actually better codes now, but it's like one of the known ways to change.

**Speaker B:** One of the known ways.

**Speaker A:** Yeah, yeah. And we, we had to, you know, kind of pioneer internally. Like, oh, how are we going to actually make this work? In distributed file system?

**Speaker B:** Were you focused on latency, on being able to store data more efficiently?

**Speaker A:** It's storing it more efficiently. So in GFS and There's varying costs where it's like, well, you're not storing two copies or one copy of your data or two copies, you're storing three triplication. Three times as much storage you're having to use. And with Reed Solomon, you can get that down quite a bit lower. I can't remember offhand exactly what our. What we used for Reed Solomon, but I think it was like, essentially 2x. But you get that with also the redundancy, too. So it's like, it's smaller and the redundancy is higher. So.

**Speaker B:** Because I guess the name, the naive thing, if you're saying, all right, I want my data to be replicated at three places, you take three nodes, three machines, physical machines, and you say, like, copy one, copy one, copy one. I have it three places.

**Speaker A:** Great.

**Speaker B:** If one explodes, I still have two. Wonderful. And then you're saying that the algorithm here is you could take not 3x the data, but 2x the data, split it smartly across machines, or maybe you could take it and take it lower and you still have the thing where, like, oh, one of them explodes. I still have all my data because it's split in a.

**Speaker A:** That's exactly right. And like, kind of the mental model, you know, if you just want to understand at a high level, which is essentially, you might want to say, like, I want to have eight replicas of this data, or I think. I can't remember offhand, I think S3 might use nine. They've actually talked about this publicly. You have kind of nine chunks of data, but any five of those chunks can be used to reconstruct it. And what this means is you can lose any four copies and you can still reconstruct your data. And oftentimes it's more like, you know, the first five chunks are exact replicas and the other four are kind of parody ones. I've kind of forgotten some of the details and stuff escape my mind. But it's along this line. But.

**Speaker B:** But when you come up with an algorithm, you can then prove that this algorithm will work, right? Like, this is a little bit like. I know in software engineering, like, maths and algorithms is a bit out of fashion, but in this case, this is really important because once you. Once you can prove it that this algorithm works, it will work.

**Speaker A:** It will work. And, you know, it's like the math behind here is like, kawa fields over, like, GF2, something like that. I don't even know the. I never actually understood the full math behind it. I always regretted not doing more Math in college, but you didn't have to like Reed and Solomon proved how this worked. I think it's like back in the 1970s something associated with like communication networks. So you just take that and you know, kind of use that expertise, but leverage that and have to do all the engineering behind it to make it work in a storage system.

**Speaker B:** And then when building a distributed storage system like Colossus, what were other things beyond okay, you want to store data in a resilient but efficient way. What were other things that problems that you needed to solve? I'm thinking things potentially like sharding and resharding or metadata being important, those kind of things.

**Speaker A:** Particularly for the scale that Google wanted to operate at. GFS kind of had this scalability limit. I believe it was like, you know, you could have a thousand machines in a GFS cluster and it kind of.

**Speaker B:** Which I guess sounds big until like today it's kind of ridiculously small. Right.

**Speaker A:** It sounds big for 2004. And then Google was like no, we need to have this scale up to 10,000 machines. And there's some bottlenecks. There was this GFS master as a single node. It was a bit of a bottlenecks. We're like ah, we need to have a distributed master to store the metadata for all the objects. And Google at the time happened to have this system called BigTable. And so Colossus stored its metadata inside BigTable. And one of the things that I'm kind of proud and like, kind of like also you know, a little bit embarrassed by one of the design choices I went down but actually worked out is we Want to use BigTable for the metadata for classes and the metadata

**Speaker B:** just for those of us not as into distributed system. What is the metadata in a distributed file system?

**Speaker A:** Yeah, it's like the names of the files and for each of the files the files are broken into chunks. What were they? 64 megabyte chunks. And then you have to have the list of chunks for each file.

**Speaker B:** Yeah.

**Speaker A:** And then you have to periodically, you know, the master has to be scanning over this and doing repair, repair work and you know, but there's more metadata in that. But that's it in a nutshell. So we wanted Colossus to have this, you know, kind of scalable, you know, service BigTable to store its metadata. The biggest user of GFS is BigTable. So we want BigTable to work on top of Colossus.

**Speaker B:** So you wrap your head around the circle dependency.

**Speaker A:** Yeah, well there's a bootstrapping thing, right?

**Speaker B:** Bootstrapping yeah.

**Speaker C:** Oh yes.

**Speaker A:** Yeah.

**Speaker B:** Which one starts up like you need to mock something somewhere, right?

**Speaker A:** Yeah, no, I mean the way it actually worked at the time and they've since replaced this, you know because this is like the way to get started and leverage what you have and eventually you kind of get rid of it. But there is the kind of a foundational big table. That big table didn't use Colossus. There's the Colossus using the big table and then there's normal big table sitting on top. Top of Colossus. I'm talking about all work for years. So I don't even know when they got rid of that. They got rid of it at some point but that was.

**Speaker B:** But I guess sounds like you can make like hacks that go really long knowing that they're hacks and they get you off the ground, right?

**Speaker A:** They get you off the ground, right. Because if we had to like you know, implement that kind of big table layer from the get go or just delayed how long it took to get, you know, Colossus built.

**Speaker B:** One of the things that strike me about a system like Colossus is it promises or this was internal to Google but. But even distributed file systems that are external they will promise high throughput, high availability and low latency. And to me it's always a bit conflicting of like well it's pretty easy to I guess build a distributed file system. With my limited knowledge I could probably do something where I have either high throughput but high latency because whenever I write something I write it out to all the replicas. You already mentioned one technique of doing it but how did you kind of reconcile how did you get low latency while you have high throughput while you also have replication going on on the file system?

**Speaker A:** Well I mean these distributed file systems, I mean the latency isn't super low in particular when they're running on hard disks, which Colossus was doing at the time, which S3 does you actually notice the latency? So like S3 is a high performance system. Google GCS, the competitor from Google which is buil on top of classes. Its high performance has incredible throughput but the latencies are like 20 to 30 milliseconds for first read and that is bounded by your hard disk latency. If you put it on SSD it gets down to closer to SSD latencies but not actually kind of the state of art SSD latencies which is kind of this crazy thing that's been happening in our industry. It's just like how Much faster. The hardware has been getting reading from a hard drive maybe 5 to 10 milliseconds nowadays. And we never touch hard drives reading from an SSD over NVMe 30 microseconds, 50 microseconds. So that's a microseconds. There's a thousand microseconds and one millisecond. So we're talking like a huge, huge difference.

**Speaker B:** Well, there are now startups or infrastructure companies that are starting to take advantage of the fact that they can have an NVME layer and they pull things up either predictively or not. But as you say, when the physical reality changes, you can build systems on top of it that should take advantage of it.

**Speaker A:** Yeah, absolutely. He knows the, the disks are so much faster with SSDs. The networks are so much faster. I mean, just kind of crazy how fast like the intra zone latencies are at a Google or Amazon center. And it wasn't like that. But when we were building colossus, you know, I can't even remember what the numbers were, but milliseconds to do a network round trip and now it's down in, you know, 100 microseconds within a zone. I mean, I just look at these things, I'm like, holy crap. You know, the hardware guys have really done a good job.

**Speaker B:** Yeah. Sometimes I feel that our software should be, feel way more snappy and there are some snappy software. But sometimes I almost wonder if we're getting too complacent with all the abstractions or we're not even doing this like napkin maths. Simon Erickson at turbobuffer talks about this napkin math where like you, it was like, all right, here's the theoretical limitation of the hardware read from SSD might be, I don't know, 30 microseconds. And then like, how can I build a system that is as close to this as possible as opposed to the other way around, saying, okay, like, you know, a human will notice like 20 milliseconds, like 100 milliseconds. Let's build around.

**Speaker A:** Yeah, yeah. And sometimes like when you're architecting something, you need to think about the human kind of perceptible latencies. But oftentimes when you're dealing with the storage system layer, you know, I know Simon working at turbopuffer, they're doing great stuff over there. You have to think about the machine scale and the machine speed, which is a lot faster than human perception. Like a human can tolerate maybe a hundred milliseconds of delay. Or if you're playing a game, maybe you need to have like, you know, frame rates of like every 4 milliseconds. But the machine wants much, much faster than that. Simon says napkin math. I call this speed of light numbers. And like, sometimes it's literally the speed of light bottlenecking you. The cross zone latencies between zones, cross region latencies is speed of light and fiber. You want to hear a kind of crazy fact?

**Speaker B:** I love hearing crazy facts.

**Speaker A:** The fastest way to send packed data across the globe is to send into space.

**Speaker B:** Is it because the speed of light is faster in vacuum?

**Speaker A:** Quite a bit faster or.

**Speaker B:** Well, it's not fuel vacuum. No way.

**Speaker A:** So.

**Speaker B:** So you cover a higher distance.

**Speaker A:** Yeah, well, you actually, I believe the, the way to do this, you send it straight up. And then even with Starlink, you send it straight up, you bounce around between Starlink, you send it down to the other side. So you want to get it out of the atmosphere as quickly as possible.

**Speaker B:** But now if we're ever talking with speed of light, this will also go. There's like a digital transformation happening. So you would need to calculate how long it takes for that system to, to process and do it. And, and. But you're saying that even if you do this, like, really well, it will be faster than beaming it through an optical cable. And an optical cable slows down the speed of light, right?

**Speaker A:** Yeah, yeah, yeah. The speed of light is only the speed of light and vacuum, in every other medium, it's slower.

**Speaker B:** I was talking, I did a deep dive on the hedge fund industry and they didn't tell me exactly. They said that they do use satellites and microwaves and some of these things they will not tell you because, you know, this is their thing. But I had a suspicion that they might have found a faster way. And I think this is like somewhat well known, but the details they're not going to get into because again, that. But like. Yeah, so they're probably bouncing stuff in space.

**Speaker A:** Yeah, yeah. So I mean, one of the things that they do in the high frequency train is like between New York and Chicago, it's actually not far enough distance wise to make it worthwhile to send it in space. So they were doing microwave beams there. Yeah, but I mean, if you really want to get faster, you need to build like a vacuum tube between them and just like send a. Maybe they're doing that. I don't know. Maybe they're doing that. Yeah, but I mean, the thing that I think is like just kind of awesome about performance nowadays is, I mean, there's so Many layers you have to be paying attention to in terms of performance. The rabbit hole just goes so deep there. You're almost certainly running on multi threaded systems, right? And you're like, oh, I need to have a multi threaded program, I need to have synchronization in there. Well, you get the best performance if you just kind of avoid the synchronization. And part of this is lock free programming, but part of it is arranging so you don't need locks at all. You need to carry about your processor caches and there's this whole like kind of setup of caches above the cpu. Like you can think of registers as cache, then you have L1, L2, L3, even your memory and onto disk and there's just. You have to pay attention to all those levels. And if you do, your performance gets way better. And if you ignore it and you're like, oh, I'm not worrying about kind of, kind of, you know, kind of the cache accesses, I'm just accessing data all over. Your program will just be way, way slower. And some folks pay a ton of attention to this, the high frequency trading. I mean they do this all day long and in so many other places like no one pays attention to it. And you get this kind of gradual degradation in the performance of the hardware or the software.

**Speaker B:** I did want to talk a bit more about low level stuff, but not about the speed of light, but low level data structures and programming language features. You made some contributions to the standard library, right?

**Speaker A:** I've done a couple not quite standard libraries. So I mean I just like have been always fascinated by data structures. It's kind of awesome, I mean I think just algorithms in general and you're just like sorting algorithms. They're just kind of awesome. You could probably explain, you know, insertion sort and you know, to kind of.

**Speaker B:** Could you explain quicksort as well?

**Speaker A:** Quicksort's a little bit harder. I mean that's the thing.

**Speaker B:** Yeah, no, but it's a smart one.

**Speaker A:** Yeah. And then you get to these levels of like, you know, it's like, oh wait, someone really smart came up with this. So one of the things I worked on just as a little bit of a side at Google at some point was a colleague came to me and he was like, you know what? We're using the STL map structure all over the place. And the STL map structure is the balanced binary tree. I can't remember if it was red black trees or one of these other bouncing algorithms that many CS college students implement. And he Came to me, he's like, I think we could do better. Because, you know, there's actually a cache problem here. Every time, every node you're traversing down, you're going to a different cache line. And he was thinking about using something else, a skip list to do this, which. That's another awesome, awesome data structure everybody should kind of look at. But at some point I was like, actually this feels more like a B tree. So I'd implemented B trees a couple times before and figured out, like, how to implement a B tree that implemented almost all the semantics of the stlmap. It couldn't quite do it perfectly. And the reason is when you insert into a B tree node, you have to shift stuff around so you don't get pointer stability. This is just kind of, of fundamental. But if you can, you know, you don't need that for your use case. You actually can pack more data in. So the thing about B tree, like, the real easy way to describe this is like, if you just have a small list of items, like eight items. The best way to store that, if you want to, you know, kind of fast access in sorted order, is just to sort the items. Right. Literally not to have a tree at all. Yeah. For like eight items.

**Speaker B:** Just have it in the very simple list.

**Speaker A:** Very, very. Just an array. Sort the array. And then you can either do a linear scan over it, you can do a binary search, and oftentimes linear scan is faster. And then you think about that. I'm like, well, if I want to store 18 items, I could just have one node that has 8 items, and then I have another node, and then you have a parent node that connects them together. And that is essentially like the. You know, you build it. You think about building it. Bottom up, you start with just one node of eight items. Oh, I need to insert the. The ninth thing. I'll split into two chunks. And the two chunks, one will have four, the other will have five, and then you have a parent node that points to them. And then you just kind of recurse on that. The. That's the B tree algorithm in a nutshell. Everybody go implement it. Actually, nobody should implement this anymore because nowadays we have something else that will implement this in all the optimizations, because there's a crap ton of optimizations that you can do on a B tree.

**Speaker B:** But I just want to go back to this. There was already an existing implementation for Maps. And then so your colleague looked at the code and said, I think we can do better. What I want to figure out is in my mind, against someone sitting outside of, you know, I'm not involved in how some of these libraries are or data structures are built. I always thought, and again, this might be naive, but really smart people sit down, they kind of look at the state of the art, they implement it, and there's no way it can be faster. In fact, I've had arguments in the past saying like, oh, let's write a faster sorting thing. Like, it's surely it is the fastest, but if you could bring us a little bit of like, how, like you've been inside Google, how it actually how it happens and how other people like yourself and your colleague can say, like,

**Speaker C:** oh, what if we, what if we try something else?

**Speaker A:** Yeah, yeah. So, I mean, my recollection here is he was working on this kind of the big internal system, I think it was called Gaia, that actually had the mapping from, you know, you log in, you have your user ID and you have to look this up. And they were storing, you know, all like this, the map from user ID and email to whatnot, to the metadata about the user in STL maps. And you just know, it's like, well, there's a lot of memory usage here and it shows up on profiles and then we're like, well, what can we do to do better? And that was kind of the genesis of it. And you know, he happened to be working on it and he happened to be working with me and like, we just started kind of noodling on this problem like, oh, can we do something better? And it's not one of these things. Like, I think now with Google, they have a whole team working on their kind of internal libraries. At the time it was more of like, you know, everybody working on their own systems and contributing to a shared base.

**Speaker B:** But I guess it still goes back to what you were just saying of just go down the layers, try to understand. And if something just doesn't add up, like suddenly like, oh, there's this big explosion, memory usage, just ask the questions, why is this? And if you're able to, or you happen to be like, oh, can we do something about it?

**Speaker A:** Right? And one of the things that he observed earlier on, I think part of one of the things was it was like a map from integer ID to something else. And if you look at red black tree, every node, you have your kind of value that you're storing the map and then you have two pointers, you might have an integer ID that's like 4, 8 bytes and then 2.

**Speaker C:** It's a waste.

**Speaker A:** Yeah, and you look at that and you're like, ooh, that seems like a lot of overhead.

**Speaker B:** And you're like, you could use a bit.

**Speaker A:** Well, the B tree actually has a lot better. It has better spatial locality and that's what made it faster. But it was actually smaller as well at the same time because you had less pointers involved.

**Speaker B:** You also contributed to go. Right?

**Speaker A:** Yeah, well, that came later.

**Speaker B:** Yeah, it came a lot later. But can we talk about that?

**Speaker A:** Yeah, just one of these other things, you know, I pay attention to, like, you know, when there's research papers coming out about new data structures and like hash tables. Hash tables are like the, the. One of the earliest things you learn about in college and data structures, like how do I map keys to values where the ordering is unimportant? That's when hash tables come in and there is like, you know, the, the very earliest ways to do this I've implemented hash tables multiple times, is like you take your key, it might be a string, and you, you put through a function and it spits out an integer. And then you map that into an array of buckets. And if multiple things map to the same bucket, you have to have a link. Link list.

**Speaker B:** You have a link list. Yeah. This is a naive implementation.

**Speaker A:** Naive implementation used quite frequently. And over time people discovered like a lot better ways to do hash tables. There's very like, that's called chaining of your hash. There's another technique called open addressing, where instead of actually having a linked list, you just kind of hash it again and move on or kind of walk down to subsequent buckets to find out like. Or subsequent slots to find out where you should be. And I remember reading about this new technique and it came out of some folks at Google, I believe it came out of their Swiss office because it was called Swiss tables. I believe that's where the. Yeah, yeah, I believe where that's where the naming came from. Not 100% sure about that, but I remember reading about it and then I was working on GO for a long period of time and GO has this built in map structure and it's a hash table. It's a very highly optimized hash table because the GO team is very competent. The GO runtime team and various folks had taken an attempt at like, you know, putting together a Swiss table implementation for go and I tested some of them and I was like, this is kind of fascinating what Swiss tables do, and I'll explain how it works in just a second. But I looked at it, it's like well, it's really hard to beat the performance of the runtime. The runtime was really good and I kept on, I knew, lit on this for a little while and eventually I ended up having to take this business trip to, to India, to Bangalore.

**Speaker B:** And so I was on a long flight.

**Speaker A:** No.

**Speaker B:** Yeah, it always starts like this.

**Speaker A:** And I just, like, I'm just going to try to pull on this. I pulled on it sufficiently that I could get some of the benchmarks to be faster.

**Speaker B:** Wow.

**Speaker A:** And then I'm like, you know, that like, is kind of like catnip for an engineer. Like, can I make it all faster? Figure all the rest of it, you know, got some help from the Runtime folks. There's a, an issue on the Go issue tracker that, you know, where other people have been attempting this because people propose like, hey, let's use Swiss tables. And like the GO forks were like, well, you know, we don't quite know all the details. You're gonna have to navigate this and that. And there were some ideas there that combined them together and got to the point where we had kind of a complete implementation that was faster on, on most benchmarks. Not quite all of them, but most of them. And then the Go folks eventually picked this up and pushed over to finish line.

**Speaker B:** And then so you, you know, like, you came with the idea, you, you got to the point where you were able to show an implementation that showed how some of the benchmarks were faster. And then you started to work with some folks on the GO team too.

**Speaker A:** Well, I didn't. I, it wasn't quite that I, I came up with that we ended up using at my company. It was good for our use case, but actually putting it into the runtime is a whole other, you know, kind of level.

**Speaker B:** But, but then you, you just showed like, here is this implementation. And then they, they, they kind of took the inspiration and the ideas.

**Speaker A:** Yeah, they're like, well, this is great. We want to make all. They, they always are looking for ways to make the runtime faster. And there's like, you know, oh, wait.

**Speaker C:** And then, so, so you did this

**Speaker B:** in, in this contribution. This was a lot of years after you left Google, right?

**Speaker A:** Yeah.

**Speaker B:** So this was from, from the outside.

**Speaker A:** This is from the outside.

**Speaker B:** That's awesome.

**Speaker A:** Yeah. And other people contribute stuff to the outside as well. You know, we had another colleague, he contributed one of the CRC implementations, you know, adapting some stuff. CRC Cyclic Redundancy Checksum. Intel published some papers about here's how to do the CRC and assembly. Very Fast and he contributed one of the implementations. You see a number of those things where you know, people just like are contributing externally. It's not a lot, honestly. I mean I actually don't know the full details but you know, people are regularly contributing to these things.

**Speaker C:** Peter just described how the Swiss table work came together using an issue track in the Go tracker with different people contributing to the work and then the Go team pushing all of this over the finish line. This is where I need to mention our season sponsor, Linear, which is a place to coordinate work between humans as well as agents. One thing I've noticed about how most of us work with agents is how it's a pretty single player thing. You open a terminal ui, go back and forth with an agent and it usually produces a pr. But the rest of your team has no idea what happened in that chat that unless you tell them or copy the whole history. And when everyone in the team works like this, a lot of work happens that's invisible to the rest of the team. Linear's take is that agent work should be teamwork. Even today teams already use Linear to define the work to be done. Now Linear can already delegate an issue to a coding agent. This agent could be an AI agent that Linear integrates with like Codex or Cursor or Linear's own agent or a custom agent. Either way, the endure in delegating stays responsible for the outcome. What I really like about how Linear works is how the work stays visible. Your teammates can follow the session of the agent, check out the PRF produces and join the review. We've gone from single player work to multiplayer engineering work with agents. Oh, and one more thing I like about Linear, a focus on costs. Linear agents autorouting chooses a model that is the best suited for the task. Teams can also inspect usage and set limits track usage so you can use capable agents without having cost. Balloon out of control control hop on board at Linear app Pragmatic. Peter also previously mentioned Gaia, Google's internal system that map logins to users. It's not surprising that Google custom built all systems including this one. But most of us won't build our internal Gaia. This brings us to our seasoned sponsor WorkOS. You can think of WorkOS as something like Gaia for the rest of us identity infrastructure you've otherwise spent quarters building yourself. WorkOS include single sign on SCIM, directory sync, audit logs, role based access control, basically everything a big customer security team asks for delivered as a handful of clean APIs. It's how companies go from we have A login to we can sell to a Fortune 500 without standing up their own internal identity platform. And Workos is already building for the next version of the Agentic authorization problem. Their newest product is Airlock, the authorization layer for AI agents. Think about what happens when you hand an agent a task like clean up the sale opportunities in our pipeline.

**Speaker B:** The last thing you want is for this thing to have standing permission to delete whatever it likes.

**Speaker C:** Airlock sits between your agent and the tools they call it, evaluates every request against the agent's intent and your rules and then it allows, it denies it or routes it to human for approval. You write down the policies in plain language, the agent never sees your credentials and every call and verdict gets logged. It works with coding agents like cloud code and codecs and with MCP gateways. So if you're working out how to let agents do real work in production without over permissioning them, take a look at workos airlock@workos.com airlock and with this, let's get back to Peter and why he left Google after building Colossus.

**Speaker B:** You were at Google, you're building Colossus distributed file systems. You're at this point probably working on probably the largest system on the planet. Honestly, why did you even consider leaving?

**Speaker A:** Yeah, yeah, well, after Colossus, I kind of dabbled in this other project called Google Goggles for a little while. Remember the glass holes? Yeah.

**Speaker B:** He keeps coming back the idea by the way.

**Speaker A:** So yeah, yeah, now it's still here, present.

**Speaker B:** Seems like Google's early.

**Speaker A:** Yeah, yeah. And you know, I think that was a technology before its time. I don't think it was ready to do at that point. It looks like the actually doing the glasses is quite a bit harder. You know, the Android phones we were trying to power it on were, you know, not powerful enough. And then, you know, I just kind of got wanderlust, you know, like, you know, am I just kind of stagnating here? Cool. Which is a strange thing to say, but you know, some other people feel it as well and decided to go off and try my hand in another startup that didn't work out. We got Acqua hired by Square.

**Speaker B:** So I want to pause for a second. So this company, what was the company name?

**Speaker A:** The company that we founded is called Viewfinder.

**Speaker B:** Viewfinderfinder, yeah.

**Speaker A:** It was in the mobile photo sharing space, which should sound familiar. This is like Instagram. This is like Snapchat.

**Speaker B:** This was in 2012, right as Instagram and all the more we're taking off.

**Speaker A:** Yeah, yeah, no, we were right there in the play and we just didn't have the right go to market. Kind of like how to attract the users, how to get buyers growth kind of thing.

**Speaker B:** Because from the outside, like what I read when I check, you know the story just like, oh, you know, like you, you, you co founded a startup, it got acquired by Square. Hooray. Like, sounds like you had bigger ambitions and this was a decent outcome but not, not the, the dream, right?

**Speaker A:** Yeah, yeah, no, it wasn't the dream at all. So the term I used was acquihired. So sometimes a company will get acquired, get bought for, you know, their ip, for their product, for their business and other times they get bought just for the talent, the people, the people. And we got bought just for the talent. They acquired the ip, but I don't think Square ever did anything with it. It wasn't kind of like where they were working, but we built up a kind of strong technical team and that's what we were hired for. And you know, like, I can't remember the details. We'd raised a small amount of money. We were able to pay our investors back, make them whole. Maybe they got a little bit of a haircut, but maybe they actually got a little bit of a. But it was, it was okay. Essentially they got their money back back, which is like, like you know, as a founder, you, you know, investors are big boys. They're used to losing their money, but you kind of feel bad if you lose a lot of money for them. So you know, getting them paid back kind of makes you feel a little bit better.

**Speaker B:** Yeah, yeah. And, and then you, you spent a little time at the company that acquired. You were just square.

**Speaker A:** Yeah.

**Speaker B:** And then you started itching it a little bit.

**Speaker A:** Yeah, yeah, yeah. Because you know, we've been working on these, you know, distributed file systems and storage systems. You know, Colossus. One of the kind of sister projects to Colossus is Spanner.

**Speaker B:** And how is Spanner different to Colossus?

**Speaker A:** Well, Spanner is essentially a distributed database. Colossus is a distributed storage system. And the way I think about the difference between a distributed storage system and a distributed database, you might think, oh, they're both storing data.

**Speaker B:** I was about to ask because a distributed database will at some point be a storage system.

**Speaker A:** Right.

**Speaker C:** So what's the difference?

**Speaker A:** Yeah, so for Colossus was cursed. Start targeting large files. Large append only files you can't update.

**Speaker C:** Append only?

**Speaker A:** Yeah, yeah, large append only files. So, you know, 64 megabytes, maybe up to gigabytes in size. But if you're a database, you want to be storing like kind of small, like, you know, kind of, you know, if you're using SQL or like the relational data, you might have a table with you know, billions or billions of, you know, rows. Those rows are broken up into columns, the columns are typed. That just has a very different nature to the engineering challenge for database than it does for a distributed storage system. And usually distributed store databases are implemented on top of some kind of distributed storage system. And that was the relationship. So Spanner was implemented on top of Colossus and some of the design decisions in Spanner kind of directly fell out of the append only nature of the files. In Colossus you can't update a file in place, so you have to make the files immutable in your database. And this is where like you know, log structure, merge trees, you know, come into play. And they weren't invented at Google, but Google really popularized them with Level DB, which emerged out of the work on Bigtable and Spanner that got popularized into RocksDB. I subsequently re implemented one of these things and this is what we use at Cockroach Labs, it's called Pebble. So I'm very familiar with the internals of that. But it's kind of all based on this idea that the data is kind of stored in the immutable files.

**Speaker B:** So how did you decide to found Cockrell Schlabs?

**Speaker A:** While we were working at Square, my co founder and I, it was actually three of us, we were all at Square and one of them, Spencer, like I mentioned earlier, he was working on the Gimp. He's my college roommate. And he was also at Google. He was also at Google. This other guy, Ben Darnell, who's also at Google, he joined us at Viewfinder, ended up at Square. We, you know, like we're kind of just noodling on a project to do and we'd actually had this design back in Viewfinder. We're like ah, we didn't. We looked around for data, this base to be using in order to build Viewfinder on top of. We didn't really like the things that were out there. The technologies inside Google looked better. We had the bigtable, we had Spanner and whatnot. We were looking around and you know like hbase existed but I wasn't quite happy. And there's some other systems like React and others and you know, at one point we're just kind of like, you know, came up with a design for Cockroachdb initial design and I was like, no, no guys, we're doing a mobile photo sharing site. We shouldn't build distributed, you know, database. So we put it on the back burner, which I think was absolutely the right thing. Maybe, or maybe we should just pivoted away from doing the mobile photo sharing site given the way things worked out. And then we got to square and we saw some of the same problems that they were experiencing with data storage systems. And Spencer is very convincing. Myers convinced some of the management that like, hey, you can just work on this part time, you know, to see if it had life behind the design and then kind of conscripted Ben and I into it and eventually started getting, you know, attention externally and we're like, like, hey, can we go and spin this off into a company? And that's what ended up happening.

**Speaker B:** And so you started a company but I understand you didn't raise VC funding initially. Right?

**Speaker A:** That was the case at Viewfinder. At. We did it differently at Viewfinder. We kind of eschewed the VC money. And you know, in hindsight I wouldn't recommend that.

**Speaker B:** You wouldn't recommend?

**Speaker A:** Yeah, I would Recommend Taking the VC money because my experience, the VCs are very, very intelligent and they can help you navigate a lot of challenges. You know, I think sometimes there's this perception, you know, the VCs will push you into various areas and maybe there's some bad ones out there that do. The VCs I've had experience with just like some of the sharpest people, you know, I've ever met.

**Speaker B:** And so you kind of have like an extra like, like person helping you

**Speaker A:** on the team, pretty much mentoring you, giving you guidance, telling you what they're seeing, they give you advice, seeing what they're seeing in the market, where things are going. That is very hard for sometimes for

**Speaker B:** a founder, especially as a technical founder. Right. That you're, you're focused on the engineering part.

**Speaker A:** Yeah, yeah, no, we actually took money right away for Cockroach Labs. It was almost like as soon as we left we got and did a little road show, you know, kind of in the Bay Area and got some interest and you know, got an investor right away.

**Speaker B:** I have to ask about the name though.

**Speaker A:** Yeah.

**Speaker B:** How did the name of Cockroach come?

**Speaker A:** So, you know, we named the gimp. Yes, that was mine. Pulp Fiction had come out in college and like, what should we name this thing? Oh, the New Image Manipulation Program. I think we're thinking Image Manipulation Program initially. And I'm like, oh yeah, it's obvious it just stuck at some point. You know, we're newly on this new database and you kind of want to give things a name. You can't just say, oh, we're, we're working on this distributed database. You kind of need something. And this master was like, oh cockroachdb. Like cockroaches are unkillable. I want these things, this database to be unkillable, you know, and Cockroach is going to survive the nuclear apocalypse. So that was where the genesis was and just stuck.

**Speaker B:** Yeah. Whenever a bunch of your nodes goes down, this thing will still be up.

**Speaker A:** Yeah, yeah. And you know, this is where we're at today with Cockroach. It's like one of the things that I point out. I'm like, holy crap, this is awesome. You kill a node. We did this whole campaign last year, which was really just to prove out something that had already been present. The campaign was performance under adversity. But just like you can run a workload against it, you can kill a node, you can sometimes kill a whole region and the system keeps on going. And it's like stories like that, you know, like what we did on the marketing side there. But also we hear this from our customers too. They've had fires in data centers and all the other data systems go down and Cockroach TV keeps on going. I'm like, like that's awesome.

**Speaker B:** When you started out, who, who were companies, startups that who wanted to use cockroachdb. And how has it changed since? Because you know, like just making the case, like, okay, I'm starting a startup, it's a small startup. Like I will need a database and I'll. I don't know, I'll spit up. I'll typically choose a postgres, right. Because it's free, everyone's using it. I'm running it on node. At what point did you see that typical, typically tech companies are like, okay, like this is not enough for me that it's running on a node either because it can go down or because I'm outgrowing it. What was it they outgrowing? Like, I'm, I'm trying to get a sense of like, at what point did companies say like tell themselves like we need something distributed in a database?

**Speaker A:** Yeah. I mean oftentimes we have companies calling us up after they've had a disaster.

**Speaker B:** So, so, so like a node went up or a hard drive failed, that kind of stuff.

**Speaker A:** You know, we, it's not quite like we're ambulance chasers, but if you see an outage, like a big outage from some, you know, company, you know, like we'll sometime trying to knock them up but also they, they will call us, you know, and being like there's a very big bank who's now a customer, don't think I can name them, but you can go read. They had a very serious outage due to a weather event.

**Speaker B:** And after that, which probably knocked down, I'm assuming a region or database or a networking cable or tree fell on something.

**Speaker A:** I think it knocked down a whole region. You know, it was a region wide power outage. It knocked down the region. And there's a mandate from the CEO. It's like, no, we just have to, you know, be able to survive these things. And that gets pushed down all the way. And you see this in other places where you know, one of our early customers, they were running on AWS and they just got to the maximum size you can run in a roar instance on. And then what would typically happen at that point is then you have to shard your database. You know, this is very standard practice. You, you take your single node database, you create 10 or 20 or 100 shards. And this is what Google have done for some period of time. And that's a heavy burden on the application developer. And the way we always phrase this is like, I mean application developers becoming a database developer at that point and they're doing it poorly. You know, they're trying to implement distributed transactions or indexes and whatnot. We felt the burden for that belongs on the database developer.

**Speaker B:** Can we talk about automatic sharding? I think it's safe to some. Most of us will know what sharding is when, when you're. But actually let's start from like manual sharding and then how you can implement automatic sharding and if you can tell us like, you know, tactics that a database like CockroachDB can, can do to actually just take that load off of.

**Speaker A:** Yeah, yeah. So I think the very basic form of sharding is a little bit like the hash table. Let's say, you know, you have a fixed number of shards. Like let's say it's just a hundred shards. Your data model is a user with a lot of data associated with the user. You just take the user and you say like oh, they map them to one of the shards and you know, you kind of just rely on the hash function to get like fairly even distribution. The problem with this is at some point, you know, one of your shards will get full and you have to kind of reshard. And that's a very, very onerous resharding, I guess.

**Speaker B:** Simple way to do is like, if it's just a hard drive, I don't know, per. Per node, where you write the user data gets. It gets full, and you're like, okay, well, I now need to split it somehow. I need to move it, I need to remap it, I need to reject my metadata, which knows where this data lives, that kind of stuff.

**Speaker A:** Yeah, yeah. And depends on exactly how you're doing that mapping from like, you know, the user ID or whatever your shard key is to the shard, you might have to remap them. All right, this is very typical.

**Speaker B:** Exactly.

**Speaker A:** And this happens in hash tables where, you know, oftentimes in order to grow the hash table, you just have to essentially actually create a new hash table, double the size and copy all the data over. Now, that's kind of like the very basic, straightforward way. And there's various levels of complexity you add on it. One of them is called consistent hashing. And there's various techniques to do this. It's kind of fascinating, like how they all work. But in consistent hashing, you can add an additional node and then it only moves a fraction of the data from each shard over there. There's various systems to do that. I believe this is like 100 lies, Cassandra. The way CockroachDB does it is more akin to BigTable, more akin to Spanner, more akin to HBase, where instead of actually hashing, we actually take. You know, you can imagine all your keys in a system, and this is always true in any system that you can imagine them just in one big contiguous key space. And then you kind of partition contiguous spans of that, and then you have to build up an index on top of those contiguous spans. And what I just described there actually sounds a lot like a B tree. So there's this index on top of that is like. That maps you from. You know, like, I need to have this range. Which node is it on? And this is a little bit like a B tree. You know, he kind of squint. You know, it's like. I think you squint and everything's either B tree or it's a hash table. And, you know, that index structure.

**Speaker B:** But it's. Now I'm starting to make sense because when I. I remember when I read it might have been the Wikipedia article on B trees, it's. It said, this is a data structure that is frequently used in databases. It's now coming Back to me because I didn't think too much of it. I'm not a data, I'm not not someone who builds databases. But now that we're talking, we just like organically keep touching on B trees again and again.

**Speaker A:** And the other place that it comes up in databases, so this is where it kind of comes up in distributed databases and no one ever really calls it a B tree. I just kind of squint sometimes and I see like it's actually kind of a B tree. But the other place it comes up in databases is for your indexes. So if you have a table and you have like an index on your email address and you want to be able to scan over those email addresses in order, that's a B tree under the hood in a database. And you know, like any kind of index you have, it usually provides sorted order. There are hash indexes, but oftentimes it's the B tree index and they are ubiquitous in databases. There's actually a paper called the Ubiquitous B tree. And you know, basically I just identified that. I think that paper was written back in the 80s and they're still ubiquitous today. They're the foundations of single node databases. And pretty much every data system I'm worked on has had B trees at some point place in them.

**Speaker B:** I want to ask about strong consistency. So CockroachDB offers strong consistency. Now for people who are a bit more newcomers to distributed systems, can we talk about the consistency models and then why strong consistency is important and why it's hard to implement it.

**Speaker A:** So I mean, there's multiple ways to kind of approach this. But I mean, if you've used a database, you probably heard of transactions, and transactions are a way to perform a whole bunch of mutations atomically. So databases talk about atomicity, consistency, isolation, durability. The durability is really easy. It's like when I write database, it has to be durably written, so if anything crashes, it comes back. The atomicity is just referring to the fact I want to do a whole bunch of changes. I want them all committed or all aborted at the same time. I don't want to have like some kind of partial operation. And why is this important? Why is the atomicity important? Well, the atomicity is what gets you to the point where it's like as an application I can do a bunch of operations and if there's an error hand, an error occurs, it all kind of gets rolled back and it's a much simpler development model to work within. And then there's the Consistency and isolation, which kind of gets. Get, you know, kind of muddled. The isolation is referring to isolation between transactions. I don't just want to one run one transaction at a time. That's easy to do. Right. I want to run a lot in parallel.

**Speaker B:** Oh yeah.

**Speaker A:** And make it so that when they're running in parallel, they are running as concurrently as possible. But you want to have the appearance when they're running as concurrently as possible that there is kind of a serial order to them. So this is like kind of the whole trick. The kind of. The gold standard for isolation is called linearializability. Don't worry about that. There's a. The step down from that is called serializability. And that literally refers to having a serial order of your transactions. But you're having to construct that in a way that you're doing everything as, as concurrently as possible. And the benefit of this, the serializability is again, it's a very simple model for the application program. They don't have to worry about weird kind of defects occurring in their program. And some of the ones that can occur. It's like the classic description is of a bank, right, Where I might want to read, you know, like, have an operation that reads and says like, do I have a hundred dollars in my bank account to transfer somewhere else? And you could like arrange for lesser isolation levels that you might be able to subtract that a hundred dollars twice. And that's bad, right? You know, we want to keep accurate, you know, track of your bank account or, you know, within your shopping cart or, you know, it's kind of like the, the use cases are endless there and you do this wrong and you have very egregious bugs.

**Speaker B:** But now going back to weak consistency

**Speaker A:** and strong consistency, anything less than linearizability or serializability might be considered kind of weak consistency. But there's also like, you know, kind of strong consistency and eventual consistency. So the eventual consistency is like, sometimes I can do an operation and it might not be immediately. I might not be able to see all the updates.

**Speaker B:** The read result will not necessarily be the current update, but they'll come back,

**Speaker A:** you know, at some point, you know, I've written part of it and the rest of it will show up at some point. And oftentimes when you're doing it, easiest

**Speaker B:** thing is your credit card balance, right?

**Speaker A:** Yeah, yeah. And it's faster to do it that way. Faster in terms of just what the performance you can get out of the system. But again, it's a little bit harder for the application to deal with. One of the places this often comes up in distributed databases or databases that have any sort of replication is that I could write to the primary replica and then I read from the secondary and it's not there yet.

**Speaker B:** That's eventual consistency.

**Speaker A:** Yeah, that's eventual consistency. And you can often work around this. But just like it puts bigger burden on the application developer because you have to pay attention to that.

**Speaker B:** And then with CockroachDB you have strong consistency, meaning as soon as you're writing it, when you're reading from the database, you already get the written value back.

**Speaker A:** You get the written value back. You know, it's like you, you read whatever you wrote. It doesn't matter if you're reading from the same, you know, node you wrote it to. If you read it from another node, you still actually get the data you just wrote.

**Speaker B:** Is the trade off logically enough that you would have higher latency? Because clearly to implement strong consistency, you would somehow need to, in the naive approach, you would need to write all replicas, right?

**Speaker A:** Yeah.

**Speaker B:** What are you doing inside CockroachDB?

**Speaker A:** Yeah, well, we are writing to all the replicas so well.

**Speaker B:** Yeah, yeah, yeah, you're doing it fast.

**Speaker A:** You were just doing it fast. You're making that efficient. I think this is one of the things that kind of also fascinates me about the software industry is we keep on finding ways to be more and more sophisticated in order to provide, you know, like, do, do things that make it easy to write the applications, but do it in a very high performance way. And we've gotten some, you know, very, very good at this over, over the years. And this is the area I know about, the databases, this is happening everywhere. Like, I'm just fascinated by how fast graphics have gotten where when I entered the industry, like you were literally like writing out each individual pixel and now you have these GPUs that are doing like billions of triangles per second, whatever the current numbers are. And just like kind of astounded, like how much sophistication I've got has gotten into every area of computer science, wherever you look at it.

**Speaker B:** One more thing on cockroachdb I want to ask about is RAFT consensus? What is the RAFT consensus?

**Speaker A:** Yeah, I mean consensus protocols. The original consensus protocol is called Paxos and it was famously hard to implement. Raft was, you might think of it as a variant of Paxos. It was kind of like an alternative to Paxos, but in my mind today,

**Speaker B:** and then, because this is algorithm being that you have like A number of nodes, like three to a lot more. And then how do you get them to agree on? What do you typically agree on?

**Speaker A:** Yeah, yeah. So you agree that the writer occurred. So like the way to think about the write occurred. Yeah, consensus. So you might think I want to replicate data and I write it to a primary and write it to a secondary. And you can't actually have consensus when you only have two replicas. And the reason you can't have consensus is if there's a crash and I come up like, if I'm on the secondary, how do I know if something was written to the primary? If I'm on the primary, how do I know it was written on the secondary? Right. And you're always going to be in this kind of confusing place where it's like you either have to roll back a little bit or like, like, you know, you lose some data. And consensus requires at least three. But you can have consensus across more than three replicas. And the idea with consensus is I'm going to write to three places. And normally you don't actually, when you're doing a read, you don't read from multiple of them. But if there's a crash, I have to do recovery, then I'm reading, oh, I can read from two of the, any two of the three. And I know I can like kind of determine what had happened previously. And it's usually just on the recovery time that you're actually doing that consensus read. So reads are typically just happening from one replica. You have to write to all three. And on a crash during that kind of failure is when the consensus read occurs.

**Speaker B:** And inside CockroachDB, how many replicas do you choose? For the consensus, it's typically 3.

**Speaker A:** For some system tables it can be 5. And then customers also have control of this at the database level. You can write to 5. 7. 5 is like, you know, if you're really concerned, you know, about kind of the data durability, you might use five. But there's a slowdown. You know, the more you're writing to, it's like the more storage space and

**Speaker B:** obviously like we're talking like of slow down with nodes. But of course if they're like between regions, there's now you have a lot more resilience for let's say an earthquake or a power hours or whatever. But now you will have additional latency, just speed of light basics. Right?

**Speaker A:** Speed of light latency. Right. And it's, you know, tens, you know, or up to hundreds of milliseconds or even Higher if you're going across the globe. So you know, you have to be very careful with that in terms of how you architect your queries. One of the things that it's just a general truth, some of distributed databases is you don't want to have a lot of back and forth. You want to kind of like do all your reads in one kind of parallel read set, get them back, then do your writes. Right. But if you're doing like kind of serial operations where I read a row, I write a row, I read a row, I write row. I mean the latencies just add up.

**Speaker B:** We talked about founding CockroachDB, but how has the company grown and where are you today?

**Speaker A:** Yeah, yeah, I mean we're being used in like we're powering mission critical applications. This is our bread and butter.

**Speaker B:** But by the way, can you elaborate on mission critical? Because it's like, yeah, if you're not, you are in the industry where you know what this means, but from the outside it can feel hard to put a thumb on.

**Speaker A:** What is mission critical.

**Speaker B:** Is my SaaS that is like showing as mission critical. Probably not.

**Speaker A:** Yeah. So mission critical to my mind is like these kind of. The other term of art is tier 0 applications. The ones that are like just the core crown jewels of what's running a company. You know, like a trading system, you know, your trading system can't go down. If the trading system goes down, this is kind of a critical problem. Or the firm that is running the training system, you know, banking systems. But also, you know, like we work with doordash. Some people might think that delivering burrito is a kind of a mission critical system and it certainly is for doordash. Right. You know, if that goes down, it's problematic. We power shopping carts, you know, other stuff like this where it's like, well, if the shopping cart goes down, you know, that company is losing, you know, hundreds of thousands, millions of dollars per hour. So that's kind of the critical. You think about it.

**Speaker B:** Yeah, I guess. Of course they're losing, but, but this is like when. Yeah, their customers are also like, they're used to this just working like running water. And then when it's not the same thing as when your utility breaks, right. Your water, electricity is out, you'll survive, but it's not what you expected.

**Speaker A:** Yeah, yeah, yeah. And everybody's like, what, how, what age are we living in that the electricity goes out? We were kind of had this ingrained into our heads at Google. It's like Gmail cannot go down. People Are, you know, relying on it. Search cannot go down. You know, if it goes down too long, people are going to move to other system. And it's not like, you know, like, in some ways search isn't as mission critical. Except, oh, wait, every single search is ad dollars behind it and you can actually notice the blip in the revenue. And it's not just the blip in the revenue, it's the blip in reputation as well. I mean, I think that's the one that really poisons companies, is like if your bank is down for a serious amount of time, the reputational damage there will be horrifically bad. And, you know, like, we often talk about, like, oh, and then grandma won't be able to pay her rent and she'll get evicted. You have to take this, like, responsibility really, really seriously.

**Speaker B:** No, but also like just Gmail being mission critical. Just on the way here, we only exchanged numbers later, but we were communicating over email, like, oh, I was telling you that. You were telling me that you're here. I emailed and I never for a second thought that it could go down. And I think we were like responding within 30 seconds. Right. And it's just, I just know it's there. Like, I didn't like bother setting up a secondary communication line. So.

**Speaker A:** Yeah, yeah, it's like when people just like when you have that kind of level of trust with your users, you got to maintain it and invest in it it. But then it leads to this kind of freedom for the user as well, where you just don't think about like, oh, I don't have to worry about it, it's just going to work.

**Speaker B:** And then in terms of the company, like, how many engineers do you have roughly?

**Speaker A:** We have some hundreds. I don't actually know the Precise engineering number, 110, but there might be 150 in R& D. Overall. There's other folks besides just engineers. I'm clearly engineering managers. Those are engineers as well. And then, you know, like, we've been growing steadily. It takes quite a while to build a distributed database. Not for the faint of heart. So it took a couple years.

**Speaker B:** You've done it a couple times.

**Speaker A:** Yeah, yeah. Well, I did distributed storage system. Now I did a distributed database. It's not for the faint of heart. Right. So there's a lot of work getting it to a level of stability, then a level of kind of quality beyond that level of stability, getting all the bugs out and then continuing to innovate and put more performance into the system, adding functionality to integrate better within Enterprises. Our revenue's been, you know, kind of steadily growing over the years. And, you know, it's at this place now where we. We see a path to future success as well.

**Speaker B:** And I want to ask about your coding habits. So when you co founded the company, how much code did you write? For the first few years, I wrote a lot.

**Speaker A:** So I've always been a very prolific coder. There were a lot of code. Early days and early days. I mean, I was kind of. We were all technical co founders, Ben Spencer and I, and we were already a lot of code, and I was no exception. But, you know, if, like, I look back at my GitHub output and, you know, it's like kind of peak years, maybe a hundred thousand lines of code in a year, which is. Which is a lot. Yeah, yeah. No, so, I mean, we're talking pre AI. Pre AI, right? This is when you're back doing this manually. Right. You know, at some point, you know, we started out using a system called RocksDB, which is an LSM. At some point, I think it's back in 2019, you ran into limitations with it. I decided we wanted to rewrite it and did a big push to, you know, write it. That might have been like 40, 50,000 lines of code. And then a bunch of other people have come up and helped. And like, you kind of look at that output, I'm just like, oh, my goodness. That was a lot to keep in your head. It's a lot just to type, you know, a hundred thousand lines of code. The average kind of like that the industry talks about is 3000 lines of code from an engineer in a month. And so if you multiply that out, maybe 36,000 in a year. That's good, right? So I was doing a lot. I kind of look at that. It's like there's kind of a max that you can hold in your head at a time. The tools have gotten a lot better since I first entered the industry. We gotten better debugging techniques, better testing techniques, but still quite significant.

**Speaker B:** And you were CTO from the beginning. Co founder of CTO. But there was a time sometime around like 2022, when you decided to kind of be a bit more hands off. Right?

**Speaker A:** Yeah. Yeah.

**Speaker B:** Can you tell me about that?

**Speaker A:** I mean, the general rule of thumb for engineering leaders is, well, you got to have your team, you got to manage your team. And we had a VP of engineering. But I was kind of getting to the point of like, okay, are my coding days done? You know, like, can I just direct from A higher level. And you know, I got this advice for a long period of time and I pushed back on it, but you know, I kind of acquiesced at some point. And I think it was the right advice. I'm not saying that the advice was wrong at the time. Time, but there's a time period from about 2022 to 2024. Whereas like my, my output declined. I think I actually did the Swiss tables thing in that time period, but I wasn't doing much on the core.

**Speaker B:** The business.

**Speaker A:** Yeah, the core business. You know, I would get in there and do some work, but at times, like it's really hard that if you're in meetings all day to also do coding. I think this is the fundamental tension. If you're like.

**Speaker B:** So you kind of took on the kind of the meeting burden, the coordination burn and the, the stuff that was, if I'm reading correctly, before you spent a lot of your head in the code and now you're spending a lot of your head like above the code, the business, the engineering, the. Or the whatever. Customers. That kind of stuff.

**Speaker A:** The customers. And just being an executive as well. Yeah. So very hard to wear all those hats simultaneously. And then I got back into it because AI started to emerge.

**Speaker B:** So when did you start using AI? When did you start to find it useful or in terms of coding?

**Speaker A:** Yeah. Well, it's interesting because, you know, those initial versions of like kind of glorified autocomplete came out and we're Talking about

**Speaker B:** the GitHub copilot of the cursor, the early version.

**Speaker A:** That was the one I had the first exposure to. We dabbled with cursor at the time, but they're all like kind of glorified autocomplete. It was kind of crazy that you could just start typing something and like fill the rest of the function. You look at it, I'm like, wait, kind of got that right. This is crazy, right? And you know, we're trying to encourage our engineers to use this. And at some point, you know, I can't remember if this is my idea or my co founders or someone basically like, you know, like, in order to like really guide people about how to use it, you have to be a user yourself. You know, I think this is true of like engineering management in general. Like you want to like manage engineers, you have to know how to be an engineer. Like if you, if you don't know how to be a good engineer, it's like really hard to manage other engineers.

**Speaker B:** I feel like you'll have A hard time, like just relating to them at the very least.

**Speaker A:** Exactly, exactly. So. So kind of took it on being like, no, I mean, this is like, it was clear very early on, like, this is probably gonna go somewhere, but it wasn't quite clear how far, how fast it would go. And you start dabbling this. And I was like, oh, okay, well, it's not quite good enough. It's not quite good enough. But, you know, let me start getting back into the coding very rapidly. You know, you start seeing the signs of life. Like, you know, kind of the Opus models coming out. It was Sonnet first, then the Opus. And you're like, you're looking at these and you're like, oh, wow, okay. Well, they seem to be able to do quite a lot, but, you know, the code is still not great. But then it was just like, just this cadence of continuing improvements. And, you know, I was starting to do a lot with Sonnet and then starting to use Opus. And you know, I had that same moment everybody else did. And this was last year, Last November.

**Speaker B:** November, December. Winter break, right?

**Speaker A:** Yeah, it was Thanksgiving. I distinctly remember it because, you know,

**Speaker B:** you were not chilling. You were coding, weren't you?

**Speaker A:** I was.

**Speaker B:** You were agenting.

**Speaker A:** I was a agent just like everyone else. I had this thing I'd wanted to do for a long period of time on CockroachDB, which is like, I wanted to like test like all these configurations of cockroachdb across like, you know, different vertical scaling, like how many CPUs you have on a node, how many stores you disks you have on a node, how many nodes you have in the system, and test across all this huge matrix of it. This is one of these things that I couldn't ever quite get prioritized appropriately because it never seemed like, kind of critical. But I've like, I always had this intuition there was something there. And then over this like four day span, the code just like materialized as I was using. I think that was opus 4:7. Or is it 4:5, whatever the number was. Like, the recollection I had is like a pretty fast typer. And then I just remember having this feeling of like, well, the code is just like materializing before my eyes. You know, you would ask for these things. I got out of the habit of actually typing it. You just kind of like, like ask for something Materializes. If you've ever seen like some of those, you know, a movie where they had this kind of archetype of a software engineer gets in front of the keyboard and they start typing, show the screen, it's just like it's going way beyond human speed. And then it was, it was beyond that. Right? Like it wasn't.

**Speaker B:** I think as software engineers, like we used to laugh at these. Like, I still remember Swartfish, the thing when they're visualizing the things are going or the code appearing, you know, you're seeing that, that the person's typing and then it's a big line. And a software engineer is like, we're laughing because it's not how it is. But. But it's crazy that like that effect, right?

**Speaker A:** Yeah, yeah. But now, now it's even better than that effect. Right?

**Speaker B:** Because it actually works this time.

**Speaker A:** And it's. That's slow in comparison. It materializes faster than that. Like, you can literally go like. We've been talking about B trees a whole bunch. I implemented another B tree in, in the past month.

**Speaker B:** Of course you did.

**Speaker A:** Yeah. And it took about out 30 minutes to implement probably 10,000 lines of highly optimized rust. I mean, it's just like, it boggles the mind. I mean, I. We should look up later. 10,000 lines, it's like a crazy amount. You physically cannot type that fast.

**Speaker B:** You, you start getting back to coding. Like, are we talking about kind of vibe coding, prototyping, or are we actually talking. You start to contribute like proper production ready code that is up the level of what you're doing at home.

**Speaker A:** Well, it started with this tool that was like this kind of benchmarking tool that tested this matrix. I'm a CTO. We have an office of the CTO. The office of the CTO's mandate is to innovate. And I was looking for places like we can have innovation. And one of the things that we want to innovate in was better auto scaling of a cockroachdb cluster. And kind of in the January timeframe, I came out with what I would think is kind of a. A research breakthrough, you might say. And it came about because I was dabbling in this area and just working with the models, trying to understand it. And they're not just good at coding, they're also good at helping you explore design ideas. And I think this is kind of the fascinating thing where you have to get out of the mindset of I know exactly what I'm going to build, but more like, hey, we have this problem. Talk through it. Be a partner with me.

**Speaker B:** Be a sparring partner.

**Speaker A:** Be a sparring partner. There's this advice you might have heard that if you're stuck on a problem, you should go yellow duck it.

**Speaker B:** Go rubber duck.

**Speaker A:** Rubber duck it. I think I've heard is yellow duck. You know, go. Just go talk to something. You don't even need it to respond. Now you can talk to this system, this intelligence, and it will give you back stuff. And it's not always right. I mean, this is the thing. Even today, these models are fantastic. Fable's fantastic, Astra's fantastic. And they're not always right, but they kind of like, they know so much. It's just encyclopedia. And you can explore ideas super, super fast and then be pointing out, well, that doesn't sound right to me. And it'll be like, oh, yeah, you're absolutely right. You know, like, I hate that sick fancy. It's like it kills me.

**Speaker B:** Or you're right to push on. Back on. You were right to push back on that. That's a new. Absolutely right.

**Speaker A:** Yeah. And yet, like, just able to make such fast progress, and I find it absolutely incredible. So it quickly moved from just doing kind of side things, building up some tools and whatnot, to building, you know, essentially over the last eight months, it's January, we've been building towards a new launch. And, you know, like, a lot of that has been produced gently powered engineers. And like the entire company's got on board with this now, where I think it's like probably a hundred percent of engineers are using it to a greater or lesser degree and a lot of code is being produced. And I think it's very high quality code. These models will not test things adequately. They don't look quite intently enough about performance. But if you're like, you can guide them in the right way. And I think there's actually a huge advantage to anybody who's done management before. It's a little bit like being a manager of people, where you're a manager of a large group, you're not looking at every line of code, but you're definitely kind of helping architect the system. I think there's a very strong analogy there.

**Speaker B:** I sometimes push back on this analogy. The reason being I was an engineering manager. My take is that working with these agents and it's not like management, because management has so much of the human stuff. Like, as a manager, when I think of all this stuff I dealt with, which was the people side of things, the conflicts between people, the performance reviews, the meeting, et cetera, and you have none of that. You do have the orchestration again. This is almost like, I guess, a really naive way of management where they don't push back, they start to do it. Sometimes they're unreliable. But I almost like to use orchestration a bit more because I feel management is so much more involved. Like I think this is the thing where like a tech lead who has no management responsibilities but they have a group of insurance, but they don't need to deal with their performance with their anything. Like it's a lot closer to that, if you know what I mean.

**Speaker A:** Yeah, I do agree. I use the engineering manager shorthand, but it's really about being a tech lead for like a 30 or 40 person organization, you know, or being an architect for one. I think the term architect gives me a little bit of a distaste, but it's being an architect. He's also on the ground, a hands on architect. Hands on architecture and you don't have any of the management stuff, which is a blessing and curse. But it's kind of remarkable that you can spin these things up. If they make a mistake, you can keep on correcting them until they get it right. We've always been able to do that on the human side and now you can do it faster. I think the ultimate result for me is you just have to be more ambitious about everything you do. You can produce more higher performance, higher quality, more secure. So our ambitions have to raise up.

**Speaker B:** You also mentioned that with AI like your everyone's using and you're building stuff, but also with CockroachDB, you're now building something that is also related to AI. Can you talk about that?

**Speaker A:** Yeah, yeah. We're building multiple things. I mean AI is the future.

**Speaker B:** It's here to say. I think that's here to say.

**Speaker A:** Yeah, I mean like you know, one of our theses, which is not crazy. Every application in the future is going to be written by AI. You know, I think there will be some kind of bespoke software, you know, handcrafted software. I think we will see that continue to exist just as people write assembly still. But it's going to be diminishing in size. So asymptotically approaching 100% of software will be written by AI generated, right? Yeah. It's hard to say when it's going to end. Where the humans are the agent kind of supplying the agency and the vision behind it. I think that might exist for many, many years, but I think the code will fundamentally be written by the AI And I think we're going to just see this explosion of applications and we're seeing that inside Cockroach Labs. We're seeing it elsewhere. Earlier this year we kind of rolled out this internal platform where non engineers could write kind of mini applications. I mean this is like you're hearing this at other companies. We did the same thing and over the course of just a couple months, you know, 500 applications, a thousand applications by non engineers. By non engineers, primarily by non engineers. And I thought it was awesome. Like our HR team is building like these little applications. Like this is the thing I've always dreamed about doing and like they were never serviced. Like your CFOs all over the place and ours is no exception, producing dashboards that they never created before. And I think it's very empowering. I'm married, I have a wife. She needs software. She cannot produce that software on her own. And I have never actually helped her produce the software, which is my own failing. But I think there's a world in the future where she gets like everyone's getting custom software built for them and you're just going to also see greater and greater applications and higher quality systems being produced by every company as well today.

**Speaker B:** What is your stack? What do you work with in terms of harness model, how you run agents? Is it one agent? Is it multiple? What kind of terminal do you use?

**Speaker A:** Yeah, yeah, it's evolved over time. So, you know, like when I first started dabbling the AI stuff again, it was, you know, GitHub Copilot. I was an Emacs user for like 20 something years. I got convinced to move to VS code, but that's all gone now. At some point I moved to using Claude code. That was the thing that whatever reason I just got started using, it was Claud code in the terminal of late. I use a mixture so I'll just grab my current setup. I use the cloud desktop app. Cloud code via the cloud desktop app. It's fantastic. Good job. Anthropic. I also sometimes use the Codex desktop app just so I have an alternative model to turn to for some very critical things we're working on. I will get, you know, one of those models sometimes like the best model. Oftentimes I'm using the best model, like Fable, you know, I've recently started using that. Sometimes it's Astra, sometimes it's the other ones. But you, you like, you have one producer design, you have the other one being like, hey, my colleague produced this. Can you kind of tear it up? You know, adversarial review it. And it's not always perfect. Right. You know, but I think there is utility, especially for something that's super, super critical. We're pushing towards the launch really soon and kind of rushing towards the finish line. I'm telling folks on the team, it's like, you know, especially our very senior folks, like, like, you should use the best model. Right now. It's worthwhile to do that. I'm generally just using the best model. And part of the reason is I don't actually know that. I'm getting a lot more intelligence from it. But I don't want to have the cognitive overhead of deciding on a case by case basis. Yeah, should I use Sonnet? Should I use Opus? Should I use Fable? Should I use Sol or Astra? And I think, you know, maybe if you like, I really need the speed. I would make that decision. But oftentimes I'm like, I'm doing things in parallel. So you're asking how many agents I'm spinning up? Well, it depends. Like, oftentimes there's like a certain number of sessions you might be using. I find my kind of cognitive overhead is about five to ten sessions concurrently. But sometimes those sessions will have many subagents doing things. Yeah.

**Speaker B:** And they'll be long running.

**Speaker A:** Yeah. It'll depend on what I'm doing. So as I was coming here, I was on the train. I spun up something to do a little bit of research, and that was just probably using three sub agents. Agents. Other times it might be 20 subagents, you know, with anthropic, has dynamic workflows. I can't remember what the Codex thing is called, but they have various ways of spinning up kind of graphs of agents and, you know, prosecuting them. I mean, I think it's. Sometimes I'm probably using a hundred subagents, others it's only like five. It depends on what's happening. And sometimes I'm not doing any of that agentic stuff because I'm often thinking about something else.

**Speaker B:** Since you were an Emacs user for 20 years, you know, using the terminal. Right, Right. Or, well, Emacs. But how come you move to graphical interface with agents? A lot of people still use terminal UIs.

**Speaker A:** Yeah. Yeah. Well, so I moved from Emacs to VS Code. I was just like a colleague was like, hey, these ides are really good. You need to try them out. I was like, emacs is an ide. And then I kind of moved and like, you know, some of the stuff is just more seamless. And like, Emacs keeps on catching up. It always feels a little bit, you know, six months to a year behind. And like every time they have a new version, my. My setup would break and I just got tired of that. So I moved to VS code. But then I was like, I'd have many terminals open in VS code. My setup would always be like terminals in VS code, tabs. The terminals and tabs, you know, have like eight of them open. And then they started taking over. It used to be like just CLAUDE code in the terminals. Yeah. And then, you know, just some point recently, it's like six weeks ago, Colleague was like, well, if you tried out the cloud desktop app recently, it's really good. I was like, yeah, no, I'm very. I'm very happy. And I made the switch and it was a little bit awkward at first, and then suddenly I'm like, holy crap, this is awesome.

**Speaker B:** You can, like manage the agents a bit better, easier.

**Speaker A:** I mean, it's like you have the sessions there, the sessions down the side, you know, and it's like they're kind of like tabs in some regard and shit. But the whole integration, that anthropic's been deleted. OpenAI is doing the same thing now, by the way. Like, I'm not dissing cursor and factory and cognition, they're all pushing the same thing. It's incredible how fast these systems are innovating right now and evolving.

**Speaker B:** What do you think good software engineering looks today compared to like, you know, four years ago before we had a.

**Speaker C:** Has it. Has it changed?

**Speaker B:** Has good change or that's not really changed?

**Speaker A:** Yeah, I think the ambition has to increase. You know, I think quality has to be higher, security has to be higher.

**Speaker B:** I'm glad you mentioned quality. Can we talk about that? Because I'm seeing across the industry just quality declines, which you cannot fully put your finger on, AI. But oftentimes it is people pushing out more and more and just not paying attention to just small regressions here and there. Again, you're building a database. Have you noticed any, or have you gotten feedback of any quality regressions? Or if not, how come? Because this is just basic. We talk about the law of physics. This is an observed thing that when you start to have more output, you increase your deployment frequency. You often, not always, but you often have like more regressions and more bugs.

**Speaker A:** If you produce a certain number of lines of code, you're probably going to have a certain number of defects per line of code. And now you can produce more lines of code, so you probably would have more defects. Right. But the thing that pushes against that is that you can be telling these agents, you have to give them quite a firm hand at this. This is a thing That I hope the model providers are listening to. But you need to give them credit for a hand. They kind of get a little bit lazy on the testing side and you have to make sure the tests are comprehensive, but also they're using all the testing techniques and there's a lot of testing techniques out there. We have a lot of knowledge about how to do testing well. And you know what, the agents are lazy, humans are a little bit lazy. So getting the humans to actually be very disciplined about their testing is also challenging. And I think it's actually easier with agents. You know, you can kind of instill that you can get them set up. You have to give them the guidance, use property based testing, use metamorphic testing, use like these advanced testing techniques, deterministic simulation testing. You know, there, there's like technique after technique you can use. And humans have always like, I, I'm lazy myself. Like I'm pretty good about being disciplined about testing. At some point you're just like, okay, that was enough, we gotta ship. And like now you can get a little bit like, you know, a little bit stricter and firmer. I think the same thing applies over on the security side, the performance side. I mean we've always had at cockroach in the industry, we've known security coding practices. You can literally have every single line of code on every commit reviewed by

**Speaker B:** a security expert doing like an adversarial security review by an agent or multiple

**Speaker A:** agents or multiple agents. Right. And we're going through a tough time in the industry right now with like kind of hacks and security leaks and whatnot. There's only a limited number of bugs that can be in software. I think we'll get it, you know, added on the security side and on the quality side. And then also just on the, when I say quality, it's not just bugs, but it's also like, you know, just little things like oh, that UX element is wrong and there's no excuse for that. Right now, like fixing it is so, so easy. And what we're also starting to see evolve is like, you know, designers using Figma. Well, I think that should be a thing of the past. Right now our designer just deals with HTML and CSS and JavaScript directly and sometimes is even just producing pull requests, you know, PRs, which she loves, we all love. Everybody's happy with this. There's not any of this waterfall.

**Speaker B:** So she, she's producing pull requests for the production code base.

**Speaker A:** Yeah, yeah. And you know, this is on the UX side, not on the Core database, of course.

**Speaker B:** But this is the area that, that like she owns, right?

**Speaker A:** Yeah, yeah. It's just, I mean, she loves it, everybody loves it. I mean, there's no downside.

**Speaker B:** This is not new. Yeah, for sure.

**Speaker A:** Yeah.

**Speaker B:** What about code review? What's your take on code review? I think it's pretty, it's starting to get a bit controversial. Like, is it going to stay or not? Because it's been a practice that's like, think about, I mean, you, you worked at Google. Google has been so big on code review. As I understand, there used. There are two code reviews, right? The, the domain experts reviews it and then there's a language expert reviews it. So you went through that. How did you think of it and how are you thinking of it now?

**Speaker A:** I used to be part of the, I was one of the early members of the C coding reviews.

**Speaker B:** I'm asking the.

**Speaker A:** Right, okay.

**Speaker B:** You will have strong opinions.

**Speaker A:** Tell me. Yeah, yeah, I mean, I, I, I see that. Code review, I think was fairly essential before the age of AI. But now we're getting to the point where I'm reviewing and looking at code. I look at most of the code that the agent's producing still. And yet it's like you're not giving it the same level of scrutiny. Right. And this has always been the case. If you get a pull request from a junior engineer, you have to give it more scrutiny than if you get it from your most senior engineer. And you ask any tech lead, any engineer, manager, any software engineer, they'll say, yeah, yeah, so the junior engineer or someone new to the code base, you know, you just have to get more scrutiny. And what I'm finding is the agents are getting better, you're having to give less and less scrutiny, and yet you do still have to give scrutiny to some things. It's like I said, it's like the testing will be incomplete. Maybe they didn't follow the security stuff. Maybe, you know, like there was a performance regression. And I think it's like, you know, sharing. There's kind of constraints on the system, so it's hard for them to do the wrong thing. I suspect that, you know, I don't know if it's going to be this year or next that we're materially going to stop looking at the code in the same way that we don't look at assembly anymore.

**Speaker B:** We trust the compiler to produce pretty good assembly.

**Speaker A:** We trust the compiler to produce good assembly and we'll still do it.

**Speaker B:** Unless you're one of those people where it's really important. You're maybe a game developer or someone and you look at it, but there's fewer and fewer of those folks.

**Speaker A:** Yeah. And even then, I think what we'll be migrating to is why does the human have to look at the assembly? The AI should look at the assembly. Like, I'm doing something right now where I want to have a zero overhead abstraction, you know, for doing something that is done at test time. And when it's in production, it's compiled out. Fable set up for me a little system where he actually looks at the decompiled code to verify like that there's only a few extra instructions put in place. I never would have put that in place. But now it has this guardrail that every time it changes this, it can verify that there's no regression. And I think you're going to see more and more of this where you kind of put guardrails in place. I think the model kind of likes it because it can work within that guardrail.

**Speaker B:** Now for what, like 20 plus years? You were writing so much code. Like, I'm sure you were in the zone. You remember being in the zone and just churning out, you wrote, wrote a lot of like production ready code. Now that you're coding with AI, do you get into the zone?

**Speaker A:** Yeah, absolutely.

**Speaker B:** And how is that zone? Is it the same? Is it different?

**Speaker A:** It definitely feels a little bit different. It's maybe a little bit less intense, but you're managing more things cognitively.

**Speaker B:** Like, can you describe me? Like, what is it right now when you're in the zone?

**Speaker A:** Yeah, well, I'm thinking of ideas that normally would have taken me a week to experiment with. And I think of multiple of these experiments and then I fired them off all simultaneously. And then I'm kind of like reviewing, like, what else should I do while that's what's being complete. And sometimes I'm kind of reviewing like the. They'll be giving me progress updates of like, oh, hey, this is coming in. We're seeing this stuff. And I'm being like, well, that doesn't sound right. Hey, what about this? Or did we do this, you know, correctly? You know, maybe I had design and it's not implementing the design quite perfectly, but I have this little feeling that, you know, I haven't been a college professor, but maybe I'm a. I was a, you know, if I was a college professor and I had a whole swarm of research assistants and they're all got off doing things and it's coming back, but it's coming Back just really rapidly.

**Speaker B:** You're not waiting months or weeks.

**Speaker A:** I'm not waiting months or weeks.

**Speaker C:** Weeks.

**Speaker A:** And then I'm iterating. I'm like, oh, that one failed. That's fine. You know, you just gotta let go. And this, this is the nature of the. The software I build. I think there's other pieces where it's like you just whip out a website. You can whip out something that doesn't have this level of kind of scrutiny, you know, very quickly. But I talk about this in a. I have a whole bunch of analogies about, like the. Let's.

**Speaker B:** Let's talk about analogies. What analogies do you have about using AI or AI?

**Speaker A:** The one I was, you know, advocating for, I had kind of two that I was advocating for, like late last year and this year, which is, you know, AI is coming for us. It's here, right? And it's like, you know, you're producing software. You're like, walking down the road, and sometimes, you know, someone will pass you. They're running, they got an efficient gate and whatnot. But everyone's under their own locomotion. And the. These agenti coding agents came and it was like a car pulled up next to you. You get into the car, you don't know how to drive. You don't understand the controls, but you got to get in and you start, you can figure out the gas pedal, and it takes off and crashes into a tree. But you have to learn how to drive. You know, I think using all these coding tools, it just isn't. Doesn't just happen naturally. It's learning how to drive. I think we might be in the era of F1 driver right now, which is like, the really good people can drive these systems a lot harder, a lot faster than the people who are just picking them up. If you've never used an agent coding tool, there's a vast difference between someone who's like, really expert in them, knows where they break, can pay attention to that, versus someone who's just picking up for the first time.

**Speaker B:** One analogy I've heard is we used to talk about 10x engineer. You remember? Like, this used to be a debate for a very long time. Is it or is it not? But now what I'm hearing is the 100x engineer.

**Speaker A:** Yeah.

**Speaker B:** And so you're saying that you are seeing some folks who maybe, let's not use the term 100x engineering, but like this, like, F1 driver who is just really, really good at it. Like, how would you describe a person who you've seen this is it just like rock solid engineering basics and they picked up, up, they lean into using these tools or what are they like?

**Speaker A:** Yeah, yeah. I mean, there is quite a bit of a, it feels like, you know, it's directional, that if they're good at software engineering before. I mean, I sometimes think it's like, you know, everyone's in this kind of spectrum of capability of software engineering and this is just like, you know, taking that line and spread it out and it's not quite true. You know, I think it's helped some people more than others, but, you know, it feels like it's just stretched it out. So, you know, your ability before is now amplified.

**Speaker B:** I'm always interested to learn that. For example, Boris Journey, Thibault OpenAI, they both have been really, really good software engineers. Boris wrote one of the first typescript books, the first typescript book for O'Reilly. He built some massive systems. Same with Thibault, who built it. And now you're kind of seeing, oh, these people are building all these tools and innovating, like, yeah, they look really good before.

**Speaker A:** Well, I mean, this is, I mean, I have two other analogies to give you about, like the, what it feels like in AI. You've entirely heard about the term pair programming and.

**Speaker B:** Pair programming.

**Speaker A:** Yeah. You know, and the idea behind pair programming is it's good to just like, you know, have one keyboard, one monitor and two engineers at it, one of the keyboard and the other one sitting beside them, kind of like looking over the shoulder and giving guidance. And I think there's an aspect of that feeling where I actually got ChatGPT to do a little image of this where, you know, it's like the Android is at the computer typing, and you're just there giving an instruction. But that was maybe the way it felt like a year ago. I think it feels a little bit different now. The one I, I, I've just started recently saying is that I feel like the domain experts, the people who were really strong before are now massively amplified. Have you seen all this mathematical stuff coming out, like the crazy proofs?

**Speaker B:** Well, I don't understand it, but I,

**Speaker A:** I don't understand them either. Yeah, yeah. So I told you I wasn't very good at math. You know, it's not completely, I, I just was like, I, I stopped in the freshman year of college, right?

**Speaker B:** Yeah.

**Speaker A:** But, you know, I kind of like watch along with these advancements and you know, Terence Tao, he's like probably most famous living mathematician, you know, super genius. He actually Posted this session. There's this recent breakthrough. I think it's called the Jacobian Conjecture. I don't even know what it meant. But he posted this ChatGPT session where he's interacting, I think it was ChatGPT, and you could see him interacting with this intelligence and it was crazy because he's talking to it as a peer colleague and it's responding and literally it honestly looks like I'd encourage everyone to go look this up. It looks like, you know, almost a foreign language. It's like his domain expertise is getting amplified by the system he's interacting with. And you can see how he's like kind of learning and exploring ideas just really rapidly. Now there's all this controversy about AI mathematics, but I feel like the, the analogy that comes to mind to me though is the domain experts, they're a little bit like sorcerers in their particular domain. You know, you have the earth sorcerers, the earth wizards, the water ones and whatnot. And if you know the magic incantations, the right words to say the right order, you actually get something kind of magical. And if you don't, you just get sparkle stuff that doesn't have anything there behind it. You know, you would probably know. You don't know much about distributed databases. If you're going to ask Fable or Astra, build me a distributed database like CockroachDB. You will get something out, but it'll kind of be ultimately hollow inside. But if you're an expert in databases and you ask to build a distributed database and you can point out all the various things you have to know about distributed database, here's what you have to worry about. The storage layer, the networking layer, here's the various data structures, runtime inside. You can actually get something quite magical very, very rapidly out of it.

**Speaker B:** So what would your advice be for? Advice be for engineers with like mid level to, to senior level who, you know, who have been figuring out how to do coding to become strong engineers in this, I guess, age of AI.

**Speaker A:** Well, the first off is you have the most amazing tutor kind of readily at hand. And I mean, one of the things that I would always do, you know, throughout my career and now I've kind of stopped doing it, but it's like the reason why is going to become obvious, which is like I would always look at other people's code. So I was at, you know, Google early on. You probably heard of Jeff Dean. Jeff Dean was an amazing coder. His colleague Sanjay Gamawat also just an incredible coder. And you know I would be looking at their pull requests, I'd be looking at their changes. It wasn't called pull requests at Google. There's a different name for it. But I'd be looking at my Cl, right? Yeah, Cl, this is P4. It's a different version control system. But I'd be looking at their changes, be like, how'd they do what they did? Right. Be looking at their code. Oh, my goodness. Sanjay's code is really always very elegant. You know, Jeff says high performance. How's he doing that? You know, like, how's he going about it? It's almost just like you acquire via osmot osmosis. But now what you can do is not just acquire v osmosis, but you can literally, like, I mean, I would be encouraging. If you're a junior engineer, you know, there's a senior engineer nearby. Like, you could ask him how they're doing, what they're doing, but you could just ask the AI to dissect what they've done and explain it to you and explain to you at various different levels, like, how does this code work? What is it doing? Give me a diagram. Explain it to me like I'm five. Explain it to me like I'm ten. Explain it to me in French, whatever, like you want. Like, like. I mean, fundamentally, to some degree, AI is a translation tool to translate it from whatever, you know, kind of language understanding it's in and keep on interrogating it until it gets it, you know, increases your understanding.

**Speaker B:** You were saying how domain experts are very much amplified. I guess one strategy as a software engineer is like, obviously become a great software engineer and use it as a tool. You can get a lot faster. You can good at distributed systems. Like, I'm not a distributed databases expert at all, but I use AI to explain a few things for me to understand upfront, which was very helpful. And it would have taken me a lot longer time beforehand. So I can use this. But I wonder if there's another part of like, as a software engineer, you can use it to become more of a domain expert wherever you're working. If it's a payments company, I mean, use it to learn about payments as well. So you can help the business, you can help your team and honestly, you'll just learn more, right?

**Speaker A:** Yeah, yeah, I think, you know, I would encourage everyone. You have to have a little curiosity, right? Don't be bound in by the area you're working on. Explore outside of it. I was working on Gmail, but I was fascinated about how the main Google Search engine worked. I was fascinated by how the internals of BigTable worked. Even though I wasn't directly working on BigTable. Just explore in look at those things. And now it's so much easier because you have this super advanced patient intelligence there to explain to it why do you think it was done this way? And then like, I mean, once you become an expert, you can be integrating. Well, I see it's done that way. Why don't we change this? Would this be helpful? And you know, that's where you go from just kind of learning to actually contributing back. I think everyone has to have the personal agency to do this. You know, if you're just sitting there waiting for someone to educate you on how to do this, it's going to be really hard right now because anyone who's coming in explaining how to use AI or explaining how to be a better software engineer, they're going to be out of date. Right. You just got to get in there, be using this tool all the time yourself and using it like, easy to learn. I feel like I've learned more in the past, probably even year, than the previous five years combined. Which is weird given, given your trajectory,

**Speaker B:** given the environment you, you were working in. Right.

**Speaker A:** I mean, like, everybody's been in the industry for a while. Like, I'm a definitely a better coder. I was a better coder 10 years ago than when I first got in the industry. It's like I could look back every decade and realize like, I got a lot better. And. And I feel like I just got a lot better over this past year.

**Speaker B:** And this was also one of the reasons I was really excited to talk to you because when we started to just exchange messages, the first thing you wrote to me when I asked you, like, hey, you know, how, how are things going? You said like, you wouldn't believe, but my coding output is insane. And it's high quality and it's database quality level. And those were things I don't really usually see it. I usually see, okay, I'm not producing more code, but it's slop. But again, like, to me, this is a bit of an inspiration. Like, look like you can use these tools to just like amplify yourself as a software engineer. Like, you are one example, right? Hopefully one of many.

**Speaker A:** Yeah, yeah, no. And I'm not the only one in cockroach labs. We have other people doing this as well. I find it very exciting. You know, it's a little bit exhausting right now, but it's very exciting. Like, I got into software engineering because I like building stuff now. You can build stuff faster, you know, the stuff you might have had compromise on in the past. And you can take away some of those compromises. I mean, you see this in the UX of software coming out. I think the UX is a lot higher. You see all the fancy like web animations and whatnot, but that's only just like the surface level. It just extends way, way below that.

**Speaker B:** Peter, this was awesome. Thanks for coming on a podcast.

**Speaker A:** Yeah, this is wonderful. Thanks for having me.

**Speaker C:** One reason I was excited to talk to Peter is because he's been a very high profile and productive engineer pre AI building some of the most resilient distributed systems in production. CockroachDB is known for its resilience and how even if several nodes are destroyed, the database still operates with that data loss. Basically it's as hard to get rid

**Speaker B:** of as cockroaches are.

**Speaker C:** Hence the name. One interesting part of our conversation was how Peter built more efficient data structures than the standard coding libraries had, thanks to him and colleagues paying attention to parts of the library that seemed slow. He did it for the C STL map and in Go for the Swiss table implementation. I found both stories a good reminder that you can improve the existing library or even the language, especially if you measure which parts feel slow. Another part of his conversation that I liked was how Peter came a bit of a full circle. He used to write 100,000 lines of code per year, being a very productive engineer and cto. He didn't stop writing code aiming to Coach engineers between 2022 and 2024. And then he started to code again because with AI tools he wanted to coach his engineers better, but it's hard to do if you don't use the tools yourself. And now he finds himself being extremely productive and this time the team around him is productive as well. And we're not talking about vibe coded software, but database worthy, high quality code generated and committed to production. Peter is convinced that AI amplifies existing expertise and this is one reason why he probably learned more this last year building with AI than the previous five years combined. And I find it a valuable reminder that learning and building deep expertise in software engineering this very valuable. And as closing, I appreciated that Peter said that not only is he excited, but he's also exhausted. There's a lot to learn, but it's tiring and neither him nor anyone I know is immune to this.

**Speaker B:** So if you're also exhausted with all

**Speaker C:** of the things going on with AI, know that you're not alone. Check the show notes for more of the Pragmatic Engineer deep dives on Google's engineering culture and on distributed systems. If you like this episode, please make sure you're subscribed in your podcast player and a special thank you if you leave a rating. Thanks and I'll see you in the next one.
