---
url: https://paulhammant.com/2013/03/28/interface-builders-alternative-lisp-timeline/
date_fetched: 2026-11-11
---

# Paul Hammant's Blog: Interface Builder's Alternative Lisp timeline

Denison “Denny” Bollay is one of the guys at ExperTelligence that crafted the first Interface Builder (IB) and showed Steve Jobs and Bill Gates. Jeff Wishnie introduced me (thanks dude) recently, and I’ve been talking to Denny as I have an agenda - Pseudo-declarative UIs - that is in complete opposition to my patronage of AngularJS, but still UI markup related. As part of that agenda I’ve been trying to get to the bottom of a rumored version of Interface Builder (IB) that loaded and saved a Lisp ‘source’. I heard about this thing in the 90’s (can’t remember who told me) and I’ve been asking around about it ever since. ThoughtWorks has a number of Apple/NeXT alumni and coders active for 20+ years in the Mac space, so I’ve been able to explore more avenues.

Denny wrote “Action!” that was sold by Texas Instruments after his involvement with the precursor Lisp Interface Builder in 1986. Read on..

# Action!

Here is a 15 minute ‘sales’ style video uploaded by Denny (and featuring him):

That’s: http://vimeo.com/62618532.

Denny has sorted through boxes in his garage looking for it, and I think it is a fun quarter of an hour for all.

This Texas Instruments video runs through the rationale for the genus of tool, then a demonstration of its use, a quicker list of features, and lastly some selling points.

Denny’s blurb that goes with the video (for those that did not click):

ExperTelligence introduced IB in 1986. We took it up to neXt to show Steve Jobs - the rest is history. In 1988, Denison Bollay built a much more dynamic interface tool, in which the interface was fully modifiable **AS** the program was running. Since it was built in incrementally compiled LISP, all other functions and methods were also modifiable on the fly. Denny took it to Seattle to show Bill Gates, but MicroSoft wanted a version written in basic (no objects, no methods, etc back then). I explained one couldn’t do that without OO. They built Visual Basic.

Sadly, it seems no one has duplicated Action!’s sophistication and interactivity even today, 25 years later!

Denny, Gray Clossman and (after relocation from France) JMH were all at ExperTelligence together and working on that IB. There was an intention to license Interface Builder as they had it, to NeXT. JMH going to NeXT and starting over on the same product was the end of that strategy. One of the realities of Californian IP law, much like Europe is that the ideas in your head, and your ability to rewrite something from concept, isn’t restrainable by your current/previous employer.

# Markup formats

## Binary Resource Forks (precursor IB)

The ExperTelligence team showed the first ExperLisp version of IB to Steve Jobs. Jobs was initially not sure whether the technology was an advance over HyperCard (also unreleased at that stage). Thirty minutes later, Jobs was a convert. Bill Gates was also shown (as mentioned) and Denny actually did that demo. This initial version of IB saved “resource forks” in binary in a Mac specified format. There’s a possibility that there was some brittleness to that over time, if library/framework/package/OS upgrades were considered. Binary was not inspectable or editable outside the IB toolchain.

## NeXT and Apple’s versions

The first shipping version of IB from NeXT saved in .nib format. Some yeas later Apple changes this to .xib which is what we have today. The latter is human readable, source-control compatible, and if you’re insane - editable outside of the IB application.

## Lisp (Action!)

Denny’s Action! saved in Lisp, and **interpreted the same** on load of the project. That was not just at design-time, but at runtime too. That was interpretation into a context, and a far cry from the parsing of HTML, and XAML today. Ignoring visual markup languages, it is also a far cry from JSON, TOML, and XML.

This Action! derivative was made outside Apple/NeXT, and in conjunction with Texas Instruments. I can’t help but feel that Lisp is a better markup language that any of (chronological order) ‘serialized binary resource forks’, plists (.nib), XML (.xib), or even today’s HTML.

What’s not shown in the video is the markup code outside of the design tool. Specifically that Lisp DSL that could be round-trip edited in Vi/Emacs etc. Given this was a hopefully pretty-printed format of Lisp, I’ll claim it was source-control compatible from the outset. I hope Denny finds some example source and discusses it some more. To me, Action! is one the “holy grail” items from the history of user interfaces, if not Information Technology more generally.

## Putting those together on a timeline

# Versus the DOM?

I recently compared the DOM-mutating current cutting edge with the leased-rectangle types of UI layout/rendering. This Action! product from 1988, seems like something again that could speak directly to that space:

- A live mutable model of the objects within the UI (equiv of the DOM)
- An ability to flip from design mode to run quickly
- Considerations as to the types of affordances that were supportable
- Bindings to High level Languages (at least C++ back then)

Some differences too:

- No forward/back button
- No XPath style DOM traversal (only Lisp)
- no URL concept
- no lazy-loading page-by-page concept
- Not client server in the way the web is today

So the DOM of the modern HTML/CSS/JavaScript space is much more advanced given 25 more years, but you can’t help but contrast to the elegant beauty of a unified Lisp markup for the same (with a WYSIWYG editor). Not only contrast, but we could ponder ‘what if?’

# 1990 or thereabouts

Tim Berners-Lee, off in Switzerland, used the NeXT version of Interface Builder to design the first web browser called WorldWideWeb initially, but later renamed to ‘Nexus’, before itself being obsoleted by Mosaic.

Denny was asked by the CIA to look into the forthcoming ‘web’. This was divulged after, I drew a parallel to the DOM as we have it today:

Tim Berners Lee only really spec’d HTTP and HTML in 91-93 [sic]. It was the Netscape guys that reverse engineered a DOM from that by 94-95. You might have been “there” earlier :)

Denny responded (I quote from his email):

Actually, that is another whole story. After seeing Action!, the CIA approached us in 1990(?) to discuss their work in the new fangled thing call “hypertext”. They had a contract with Xerox PARC, and had built “NoteCards” in LISP on a Xerox LISP machine. I re-architected the hypertext concepts to OO, with many classes of links unidirectional, bi-directional, contextual, searchable links and stored everything in an object database (written in LISP of course). We aptly called it “Dynamic Documents”. Another “lesson learned” was that people at Xerox had been hesitant to re-author all their docs, just so they could add links. We vastly improved this by allowing links between arbitrary portions of general purpose docs built in word processing apps, drawings, and databases. And, the links were “out of band” - so the docs & apps didn’t need to be changed. One day the CIA officer mentioned a project called the “web” in Switzerland, and gave us a copy of an HTML spec. We reviewed it with Xerox PARC at a “hypertext summit”, and came to the conclusion it was dumb from almost every angle:

1) Every document had to be re-written in HTML with only links as an added value

2) The links cannot be contextual - because they MUST be the same for all users

3) Only the dumbest unidirectional links were supported (and no mechanism for dead link checking)

4) Completely static

5) Mangled the notion of SGML by adding style tags like `<b>` mixed in with semantics (style was supposed to be like CSS today)

6) Even HTTP was stupid

Obviously we misjudged…

# Agile

A lot of the language of the video is the same as the language of the modern Agile community, which I find interesting too.

# 2023 Prompting ChatGPT to speculate

**Me**

If the NeXT operating system had used Lisp for the Interface Builder save format instead of ‘nibs’, may the later creation of the web on a NeXT machine by Time Berners-Lee have utilized Lisp too instead of Tim creating HTML?

**Chat GPT**

It’s an interesting hypothetical scenario, but it’s important to remember that the development of technologies and programming languages is influenced by a complex interplay of factors. While the choice of the Interface Builder save format on the NeXT operating system could have had some impact on the future development of web technologies, it’s difficult to say with certainty how things would have unfolded.

If the NeXT operating system had used Lisp for the Interface Builder save format instead of ‘nibs’, it might have made Lisp more prominent within the NeXT developer community. Lisp’s strengths, such as its expressiveness and flexibility, could have led to innovative user interfaces and applications being built on NeXT machines. However, the NeXT operating system was not as widely adopted as other platforms at the time, and this could have limited the reach of such applications.

As for the creation of the web by Tim Berners-Lee, it’s important to recognize that the World Wide Web was developed as a means of sharing and linking research documents at CERN, a high-energy physics research organization. Tim Berners-Lee’s main goal was to find a way to share information easily and efficiently among scientists. He proposed the concept of HTML as a simple markup language for structuring documents with hyperlinks, which eventually evolved into the web we know today.

Even if the NeXT operating system had used Lisp extensively, it’s not guaranteed that Tim Berners-Lee would have chosen Lisp as the language for the web. Several factors led to the adoption of HTML and the development of the web:

1) Simplicity: HTML was designed to be easy to use and understand, making it accessible to a wide range of users.

2) Platform Independence: HTML’s platform-independent nature allowed it to work across different operating systems and browsers, which was crucial for the web’s widespread adoption.

3) Incremental Development: The web’s development was incremental, starting from simple hypertext documents and gradually evolving into a more complex system. This approach aligned well with the initial goals of the project.

4) Openness and Collaboration: The World Wide Web was built on principles of openness and collaboration, encouraging widespread adoption and participation from the global community.

While Lisp is a powerful language, it might not have aligned with the specific goals and requirements of the early web. Additionally, the existing momentum and network effects around HTML played a significant role in its widespread adoption and evolution.

In conclusion, while alternative choices in the development of technologies can have far-reaching effects, it’s challenging to predict how specific changes in history would have impacted the course of events. The development of the web was shaped by multiple factors, and HTML’s simplicity, platform independence, and incremental growth were among the key reasons for its success and widespread adoption.

So maybe it would all have been the same: an SGML-inspired markup would have happened anyway, and we have no time machine to go back and nudge Tim towards Lisp, either.
