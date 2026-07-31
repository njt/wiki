# Make Better Documents

Anil Dash's field manual for the ordinary documents that run organisations — resumes, reports, budgets, slide decks — and the craft we never teach anyone. Ten rules that are obvious once stated and ignored almost everywhere, anchored on a single insight: the person reading your document is not you, and almost everything you do to a document makes it harder for them.

---

## The Core Argument

Dash opens with a diagnosis that rings true for anyone who's sat through a bad slide deck or waded through an unreadable report: "we almost never actually teach people how to use the ordinary tools of business communication." We teach people to code, to analyse data, to manage projects — and then we hand them PowerPoint and Word and assume the communication part takes care of itself. It doesn't.

The ten rules that follow are not a system. They're a pattern language for documents — small, composable pieces of advice that work together. The unifying thread is empathy for the reader: everything that makes a document _feel_ professional to its author (dense text, elaborate formatting, dramatic reveal structure) makes it worse for the person who has to extract meaning from it.

## Key Advice, With Commentary

> "Start from common ground, identify the shared goal."

Dash's first rule is the hardest to follow because it requires you to actually know who you're writing for and what they need — not what you want to tell them. The diagnostic is the opening paragraph: if it's about you (your process, your context, your caveats), you've failed. The reader doesn't care about your insecurities; they care about what this document means for them. This converges with [[How to Write an Effective Software Design Document]]'s central question ("what's the penalty for being wrong?") — both are asking you to write for the decision the reader needs to make, not for the work you did.

> "Something that is bold, italicized, underlined, and brightly colored means you don't know what's important."

The formatting rule that should be tattooed on every knowledge worker. Dash credits Joey Cherdarchuk's "Clear off the table" post for the visual-minimalism argument. The insight is structural, not aesthetic: formatting is a signal, and when everything is signaled, nothing is. If you bold the key phrase, italicize the caveat, and leave everything else plain, the hierarchy is legible. If you bold-italic-underline-highlight the whole paragraph, you've outsourced the hard work of deciding what matters to the reader — which is exactly what you were paid to do.

> "Spray them with bullets."

Dash on white space: you cannot have too much of it. Bullet points aren't lazy; they're a gift to the skimmer, and everyone is a skimmer on the first pass. The companion point — avoid meaningless clip art and stock photos — is subtler: "visual noise has a huge cost." Every decorative element the reader processes is cognitive load not spent on your actual argument. The stock photo of people shaking hands doesn't add meaning; it adds milliseconds of processing that compound across a 40-page deck.

> "It's not a murder mystery."

The rule that most directly contradicts how people actually write. Academics, consultants, and engineers are all trained to build toward conclusions — show your work, earn the reveal. Dash says this is backwards for business documents: state the conclusion first, then provide the supporting evidence. The reader who disagrees can scrutinize your reasoning; the reader who agrees can stop reading. Both outcomes are better than making everyone wade through the buildup. The companion point — "put the important points on the page" rather than in speaker notes or appendices — is the same argument applied to slide decks: if it matters, it goes on the slide. The appendix is where arguments go to die.

> "How can we do better?" is a philosophical debate, not a prompt for an organization to make a choice.

Dash's advice on asking answerable questions is borrowed from parenting ("Do you want spaghetti or chicken nuggets?") and applies it to organisations. Open-ended questions feel collaborative and empowering; in practice, they're abdication. If you want a decision, constrain the options. If you want a discussion, be explicit that you're not asking for a decision. Conflating the two is how meetings become unproductive and documents become circular.

> "just pretend that the underline button in your apps doesn't even exist anymore."

The closing line is Dash at his best: a specific, actionable rule delivered as a joke you'll actually remember. The underline button is a synecdoche for everything wrong with how people use formatting — a holdover from the typewriter era that survives only because it's there, not because it helps. Kill it, and you've killed the mindset that more formatting equals more clarity.

## Key Themes

**#pattern — Reader-first design.** Every rule traces back to one question: what does the reader need? This isn't about dumbing things down; it's about respecting the reader's time and cognitive capacity. Dash's advice converges with the [[Command Line Interface Guidelines]]'s principle that CLIs are conversations, not atomic invocations — both treat communication as a designed experience where the recipient's context matters more than the author's intent.

**#pattern — Signal discipline.** The formatting rules, the sequencing rules, and the naming rules are all versions of the same idea: every element of a document communicates something, whether you meant it to or not. Inconsistent formatting makes readers "deduce the semantic meaning of a change that you didn't even make on purpose." A list ordered by when you thought of things implies a priority you didn't intend. A file named `Meeting with Sam` tells the recipient nothing about what's inside. Signal discipline is the craft of making sure your document's implicit messages align with its explicit ones.

**#pattern — Constraint as kindness.** The parenting metaphor in rule 8 generalises: constraining choices is doing the reader a favour. An outline constrains what they need to hold in their head. Bullet points constrain the parsing effort. A conclusion stated upfront constrains the uncertainty. Dash is arguing, implicitly, that most business communication fails because it offers too much — too many words, too many formatting signals, too many open questions — and the fix is almost always removal.

## Critical Analysis

Dash's advice is excellent and also incomplete in ways that reveal its context. These are rules for _upward_ communication — documents written by someone who needs something from a more powerful reader (a hiring manager, a budget committee, an executive audience). The advice assumes the reader is busy, sceptical, and doing you a favour by reading at all. That's the right default for most business communication, but it's worth naming the assumption: if you're writing to peers who share your context, or to a team you lead, the calculus changes. You can afford more build-up, more caveats, more collaborative open-endedness — because the reader isn't scanning for a reason to stop reading.

The article also predates the LLM era in an interesting way. Written in March 2024, it addresses a world where humans write documents and other humans read them. In 2026, documents are increasingly written by agents and read by agents — or written by agents and read by humans who know they were written by agents. Dash's rules about signal discipline become _more_ important in this world, not less: an LLM that doesn't know what's important will bold everything, and a document written by an agent with no visual restraint will trigger every "this was AI-generated" alarm in the reader's brain. The [[Why Does AI Write Like That]] problem is partly a prose problem and partly a formatting problem — and Dash's rules are a formatting style guide that would make AI-generated documents less obviously synthetic.

One gap: Dash doesn't address the organisational incentives that make bad documents persist. The reason people write "lengthy, off-putting, deeply insular" openings isn't just that nobody taught them better — it's that in many organisations, looking like you did a lot of work is more career-advancing than being clear. The 40-page deck with elaborate formatting signals effort; the 5-page deck with bullet points and a clear conclusion signals confidence, which reads as laziness to a boss who measures input rather than output. Dash's advice works if your organisation rewards clarity. If it rewards performative thoroughness, following these rules will hurt you. That's not a flaw in the advice — it's a flaw in the organisation — but it's the real reason these rules aren't more widely followed.

---

*Sources: [[raw/make-better-documents]]*
*Last updated: 2026-08-01*
