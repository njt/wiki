# Tech Writers and AI (HN Discussion)

A 266-comment HN thread that became an accidental oral history of what technical writing actually is — and what's lost when you mistake the output (words on a page) for the job (observing, listening, understanding). The OP linked an essay arguing against replacing tech writers with AI; the top comment, by working tech writer nicbou, reframed the whole debate around empathy as the core competency.

---

## Key Quotes

> "I write documentation for a living. There is a difference between my output (writing) and my job (observing, listening, understanding). Empathy is the engine that powers my work." — nicbou

The thread's thesis statement. He goes on to describe revising a Berlin transit guide by riding every route himself, interviewing readers, and maintaining a trusted network of informants. The writing is the last 10% of the work.

> "To curate a list of the best cafés in your city, someone must eventually go out and try a few. A machine that cannot feel will never be a good curator of human experiences." — nicbou

Dubbed "the banana bread problem." You can scrape reviews, but you can't taste. This is the hardest challenge to the AI-replacement thesis and the one the thread never satisfactorily resolves.

> "AI writing is only as good as the data it feeds on. I hunt for my own data." — nicbou

A quiet indictment of the entire RAG paradigm. If your source material is whatever's already on the internet, your documentation can only ever be a remix of what someone else already wrote down. The things nobody wrote down — the edge cases, the undocumented behavior, the thing the senior engineer knows but never mentioned — don't exist in the training data.

> "Documentation is the cheaper form of customer service." — nicbou

The economic argument that management might actually hear. Support agents cost salary; documentation costs one writer. But it's an argument that only works if leadership can trace support costs to documentation gaps — and as jerf and kbelder point out, those failures are aggregate-visible but individually unattributable.

> "AI-made documentation has 0% of the quality." — marcosdumay

Because AI only documents what's already written down. Counterpoint from Calazon: "Most documentation is documenting things that somebody already wrote down." The thread never settles whether AI docs are 0%, 50%, or 90% as good — but the distribution matters more than the average. AI might be adequate for documenting the happy path and useless for the edge cases where documentation actually saves someone's week.

> "Traceable errors get corrected, untraceable errors don't." — kbelder

A management theory in one sentence. When documentation quality drops, nobody files a ticket that says "the docs are worse." They just struggle longer, ask colleagues, or give up. The cost is real but invisible to any dashboard.

> "I would not want to let anything an AI wrote out the door without heavy editing." — DeborahWrites

From a tech writer who's actually tried the tools. The editing takes as long as writing from scratch. This maps to Telemakhos's observation that catching AI errors "requires someone who might as well just write the documentation."

> "The whole article felt imprecise with language...it made me feel LESS confident in human writers." — entontoent

The thread's most uncomfortable moment. A human writer, arguing for human writers, wrote an article whose title was ambiguous enough that readers thought it meant the opposite. AI skepticism loses force when the human alternative isn't clearly better.

> "Replacement will be 80% worse, that's fine. As long as it's 90% cheaper." — ajuc

The thread's most honest summary of market logic. Quality isn't the variable being optimized.

---

## Key Themes

### #concept The Empathy Gap

The thread's central claim: technical writing's real value isn't wordcraft, it's the human work of noticing what users will find confusing. Tech writers function as "stand-ins for actual users" (drob518), "anthropologists bridging communication" (sehugg), "usability radar" (TimByte). AI can paraphrase a spec; it can't feel the confusion that makes someone close a tab.

Counterpoint from chiefalchemist: most people are terrible communicators, so AI-written docs might be an improvement over what those people would produce unassisted. The baseline isn't "professional tech writer," it's "whatever an overworked engineer typed into a wiki at 6pm."

### #pattern Traceable vs. Untraceable Errors

jerf and kbelder articulate a pattern that applies far beyond documentation: **metric-driven management systematically underinvests in quality that fails silently.** A doc bug is untraceable (nobody files a ticket saying "I was confused for 45 minutes"). A P0 outage is traceable. So you fix the outage and let the docs rot. This is the mechanism by which AI replacement happens: the quality drop is real but invisible to decision-makers.

### #concept The Printing Press Parallel

samiv argues that when production gets cheaper, "the volume of low quality production saturates the market" and economics "get destroyed." Telemakhos counters with the printing press: individual books got worse, but what average people owned got dramatically better. The question for AI docs is whether the floor is "acceptable" or "trash." The thread splits on the answer.

BrenBarn's refinement is sharper than either position: the problem isn't peak quality, it's that "the expected value of the quality you can access in practice" goes down because discoverability collapses. When everything looks the same, finding the good stuff gets harder.

### #concept The User as Resource

nicbou names a shift "from the user as a customer to the user as a resource." rkomorn traces it back 30 years to automated phone trees. The progression: human support → automated phone tree → chatbot → user expected to figure it out from AI-generated docs. Each step saves the company money and costs the user time. The "cartel of shitty treatment" (nicbou's phrase, praised by multiple commenters) describes an equilibrium where all competitors converge on the same bad experience because defecting (offering better docs/support) costs money without guaranteed return.

### #tool AI for Compliance Documentation

GuB-42's taxonomy of documentation-nobody-reads: glossaries defining CPU and RAM, UML diagrams that don't match code, screenshots from three development stages ago, signatures of departed team members. This is box-ticking documentation, and AI is genuinely good at producing it — because quality doesn't matter, only existence. The danger is that organizations start treating all documentation this way.

### #person The Bot in the Thread

A user copied another commenter's exact words 53 minutes later, got called out as a bot, and denied it ("im new to hackernews lol"). publicdebates: "I replied to a bot. I am officially retiring from social media." The irony — an AI discussion contaminated by AI-generated comments — went unremarked upon by most.

---

## Critical Analysis

**The thread is better than the article that spawned it.** The original essay at passo.uno apparently didn't make its case clearly enough to convince even sympathetic readers (entontoent found it imprecise). The HN comments — particularly nicbou's — made the case better through lived experience than the article did through argument. This is a recurring pattern: HN comment sections as distributed peer review that occasionally outclasses the primary source.

**The empathy argument is correct but insufficient.** nicbou is right about what technical writing actually is, and the thread's detractors (observationist, block_dagger) are mostly strawmanning him as a protectionist rather than engaging with his actual claim. But "machines can't feel" is a vulnerable position — it's a claim about AI capabilities that could be falsified by next year's model, or at least rendered irrelevant by AI that's good enough at *simulating* empathy that users can't tell the difference. See [[The Future of Everything is Lies I Guess]] on LLMs performing empathy "without meaning anything."

**The economics argument cuts both ways.** AI docs might be 80% worse at 10% the cost, and that's a winning business case. But nicbou's counter — that good documentation reduces support costs — is also an economic argument, and a stronger one than he gets credit for. The problem is measurement: support cost reduction from good docs is aggregate-visible but individually unattributable, exactly the kind of benefit that gets killed in spreadsheet-driven organizations. This is the same dynamic that makes [[Write Only Code]] dangerous: the costs are diffuse and delayed.

**The "banana bread problem" names something real but limited.** You need a human to taste banana bread and recommend cafés. But senordevnyc's counter — that AI monitoring social media, purchase data, and foot traffic could identify the top cafés without tasting anything — is also correct. The curator's value might be data capture, not subjective experience. The harder version of nicbou's claim is: for some domains, the thing being documented has no proxy signals, and subjective experience is the only path to useful knowledge. That's a narrower claim and a stronger one.

**The real loss isn't documentation quality — it's the human upscaling.** ainiriand's point is the one that stuck with me: writing well shapes your brain in ways useful across life. When you automate away the practice of clear thinking, you don't just lose the output, you lose the cognitive capacity the practice built. This connects to [[Cognitive Debt]]: AI makes it cheap to produce without understanding. The compounding effect across a career — across a generation — is the cost nobody's spreadsheet tracks.

---

*Sources: [[raw/hn-tech-writers-ai]]*
*Last updated: 2026-05-15*
