# Scaling Intent, Quality, and Artistry (Katie Dill)

Katie Dill, Stripe's head of design, argues that the AI building boom risks repeating the post-WWII building boom: intentionality lost to urgency, style copied without thought, and a landscape of "zombie" artifacts. Her prescription is fourfold — hold a point of view, encode standards into the machine, refuse to confuse done with good, and use AI as a creative catalyst rather than a monotony engine.

---

## The zombie-building thesis

Dill opens with architectural history: modernism had founders with real intentionality (the Bauhaus had principles), but the post-war boom copied the surface style without the thinking. Banks that once signaled trust gave way to generic boxes "ill suited to the context at hand." The analogy is direct — teams of three now do what teams of thirty did, and LLMs are "really good at telling you the most probable answer... what has been or is popular," and "less good at telling you what's original or specific to you."

> "If we're not careful, our building boom could end up a little too much like the post war building boom. The proliferation of patterns ill suited to the context at hand. Essentially Zombie ui. Zombie UI that's monotonous, vacant or uncared for."

The Korean-barbecue-site example lands the point: a "very fine website" with no character, no context — competent and dead. Her microwave-burrito metaphor for premature polish is the best writing in the talk: "I'm willing to overlook some pretty serious flaws... I'm so enamored by the speed of execution." AI output seduces the same way — corners and drop shadows that read as done before anyone asks whether it solves the problem.

## Four recommendations

**1. Have a point of view.** "If you don't, AI will give it for you. And... that's very likely to be generic and looking at the past." Her sharpest moment is a design-crit story: a cross-functional partner asked what the quality standard should be for something *made with AI*. Her answer: "It doesn't matter to the users how it was made. It matters to them if it's good or not." That's the correct stance and one the industry keeps fumbling — provenance is not a quality bar; output is. The crit still ended with 17 bullet points of craft improvements, now a verb at Stripe ("Pepsi bubbling").

**2. Encode your standards into the machine.** This is the most operationally meaty section. Stripe started with an MCP over design documentation: "Three different people could put in the same prompt and get three different results." They moved to a CLI built on the design system that "makes a harness that makes the AI far more obedient," consumes docs "at the right time and place to avoid context rot," and — crucially — ships full templates and flows, not just atomic components: "it's far more opinionated than it's been in the past." Her formulation is quotable: "The old system scaled consistency, but the new system needs to scale intent."

The Gutenberg example is well chosen — 290 characters including ligatures and abbreviations, built so machine-set type could feel "as good as handmade." And she quotes Christopher Alexander: "a system can satisfy every rule and still be dead" — which is exactly why components-and-tokens alone don't survive agentic building.

**3. Refuse to confuse done with good.** The filter that used to run throughout product development (20 ideas, staff one) now happens *post*-build, "when it is a lot harder to say no." Her fix is the editor role: "Who's looking at the end to end and understanding whether or not we're actually building a whole?" And the discipline: "experience it like a user would... a lot of things look good in isolation, but then when you pull it all together into the user journey, it feels a little disconnected." The 56-iteration animation story is the strongest concrete: AI "opened the possibility space" — the designer attempted a far more complex scene than he would have by hand — but the care came from 56 rounds of human scrutiny.

**4. Unleash creativity and artistry.** "Chats, charts, CLIs — no way is that the epitome of great interactions in the modern era." She frames AI like multitouch or the synthesizer: a technology that unlocks wholly new aesthetics, with "the best practices... not written yet." Tactically: brand-specific prompts with source material, stress the outputs, use adversarial agents as critics. Strategically: "If we only use AI to make the things that we already make just faster, then we are definitely missing out on the most interesting part. AI can help us raise the ceiling, not just the floor."

## Analysis

This is a design-leadership talk, but its core is an engineering claim: when the means of production get cheap and distributed, quality has to live in the system rather than in the room. That's the same thesis as [[Design Systems for AI Agents (Luke Wroblewski)]] — design systems rot until they become part of the agent steering layer — and Dill adds the Stripe-specific detail that components aren't enough; the system must encode *flows and behavior*, i.e., opinionated templates, or three prompts get three products.

It's also a sharper articulation of the taste problem that runs through this wiki. [[Turning Your AI Into a World-Class Designer (Chimala)]] gives the practitioner's loop for escaping slop; [[Design Language in Three Days (curiosity.ai)]] shows a spec built for a reader with no eyes; Dill supplies the executive framing — point of view first, or the model supplies its own — plus the honest admission that alignment is cultural, not technical: "It is easy to follow cookie cutter patterns, it's safe and cozy."

The editor recommendation is the one most teams skip, and it connects directly to the review-bottleneck literature: the filter moved post-build, which is precisely why [[Agentic Code Review]] and the verification-first crowd exist. Dill's "experience it like a user would" is the design-side twin of [[Agentic Verification]]'s claim that agents make checking cheap — but she'd add that some checking (does it feel coherent end to end?) still has no deterministic oracle.

One weakness: the talk leans on vocabulary — intent, soul, care — that resists operationalization, and Stripe's CLI harness is described only at gesture level. But the analogy discipline is genuinely useful: the zombie-building frame gives teams a shared image for the failure mode of scale-without-intent, and the Ruskin close (no two Gothic columns alike, each showing "the hand and mind of the maker") is the right counter-ideal to prompt-generated sameness.

**Key themes:** #concept (scaling intent vs consistency, zombie UI, the editor role) #pattern (standards encoded into the means of production) #person (Katie Dill) #tool (Stripe's design-system CLI harness)

---

*Sources: [[raw/how-to-scale-intent-quality-and-artistry-with-al-katie-dill-stripe]], [[summary/how-to-scale-intent-quality-and-artistry-with-al-katie-dill-stripe]]*
*Last updated: 2026-10-18*
