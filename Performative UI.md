# Performative UI

Nathaniel J. Smith's satirical React component library that catalogs the visual vocabulary of AI startup landing pages. Twenty-seven components, each one a named, packaged, and `npm install`-able version of a design trope that signals membership in the AI tribe without doing any actual work. The joke is that the joke is also the product.

---

## Key Quotes

> "Components that signal how oversubscribed your funding round is."

The site's tagline is also its thesis. Every component in the library is a form of status signaling dressed as UI. The gradient text, the animated sparkle, the glowing pricing card — none of these solve user problems. They solve founder problems: "do I look like a real AI company?"

> "We replaced the value prop with a text input." — PromptHero

The sharpest single line in the collection. The AI landing page pattern where a giant prompt textarea sits where the explanation of what the product actually does should be. It's the design equivalent of answering "what do you do?" with "what do you want me to do?" — which sounds profound until you realize it means the builder hasn't figured out the product yet.

> "The textarea every AI builder ships instead of explaining what their product does." — Prompt

The corollary to PromptHero. The prompt-as-hero isn't a design choice — it's an admission. When the core interaction is "type anything," the product is a blank check the user has to fill in.

> "Always green, even when it's not." — StatusDot

Three words that capture an entire genre of dishonest system-status communication. The green dot that says "operational" because the marketing site is decoupled from the actual infrastructure. The status page that's manually updated. The performative reliability that dissolves the moment you become a paying customer.

> "Backdrop-filter: ambition." — GlassCard

The glassmorphism trend reduced to its emotional payload. The frosted glass effect isn't about readability or information hierarchy — it's about looking like you belong to the future. The CSS property as cultural signifier.

> "Server-sent events (SSE) were added to the HTML5 spec in 2008 but never used until 2025." — TokenStream

The funniest historically-grounded joke in the collection. SSE sat unused for 17 years until AI chatbots needed a way to stream tokens to the browser. Now every landing page has a fake token stream demo that isn't connected to a real model.

> "Trusted by everyone you've heard of, including the ones that didn't sign." — LogoMarquee

The dark pattern that converted into a component. Every startup's logo wall includes companies that haven't actually paid for or consciously endorsed the product. The logos are social proof through adjacency — "these logos are here, therefore we're legitimate" — and the marquee animation prevents you from reading any of them closely.

> "The middle one is glowing. Choose accordingly." — PricingCard

Decoy pricing as a pre-built React component. The behavioral economics insight — people pick the middle option when it's visually distinguished — packaged for mass deployment. Smith's description treats this as so obvious it barely needs the sarcasm.

> "Demand we manufactured ourselves." — WaitlistForm

The waitlist as marketing theater. When the product isn't ready but you need to show traction, you launch a waitlist. The number of signups becomes a metric to put in your deck. The manufactured urgency ("limited spots!") creates FOMO for something that doesn't exist yet.

> "Built for conversion, not consent." — Popover

The privacy-violating popover that appears before you've read anything. It's not about user preference — it's about capturing email addresses before the visitor bounces. The honesty is refreshing: this was never about GDPR compliance.

---

## Key Themes

- **#concept** Performative design — UI elements whose primary function is signaling tribal membership rather than solving user problems. The gradient, the sparkle, the token stream: they say "we're an AI company" more loudly than any value proposition could.

- **#pattern** The startup landing page as a genre — Smith's catalog reveals that AI landing pages have congealed into a recognizable form with mandatory elements (hero prompt, logo marquee, glowing pricing card, chat bubble). What looks like variety across hundreds of startups is actually a shared template no one admits they're using.

- **#pattern** Vaporware componentization — the library packages things that normally require a product behind them (MockIDE, TokenStream, ChatBubble) into standalone components. The insight is that these UI elements already function independently of any real product on most landing pages. Smith just removes the pretense.

- **#concept** Honest naming as critique — each component's description states the quiet part out loud. "Always green, even when it's not." "Demand we manufactured ourselves." "Built for conversion, not consent." The humor comes from the gap between what these elements claim to be and what they actually are.

- **#concept** Design as class marker — the components aren't functional; they're sartorial. Using Aurora backgrounds and GradientText is the design equivalent of wearing a Patagonia vest to a VC meeting. It signals that you know the code.

---

## Critical Analysis

This is the funniest thing anyone has built about AI startup culture in 2026, and it landed because Smith understood something that most critics don't: **the UI is the message**. You can't satirize AI hype by writing thinkpieces about overvaluation. You have to build the thing they're building and make it absurd on its own terms.

The library works at three levels simultaneously. **Level one**: it's genuinely useful as a joke — you could theoretically `npm install performative-ui` and build the most AI-looking landing page in history in an afternoon. **Level two**: it's a taxonomy — by cataloging 27 distinct tropes, Smith has produced the most comprehensive map of startup design patterns since [Bennett Feely's "Make It Pop"](https://makeitpop.neocities.org/). **Level three**: it's an argument — the fact that these patterns can be componentized proves they're formulaic, and the fact that they're formulaic proves they're not about substance.

The closest parallel is [[The Behavioral Cost of Personalized Pricing]]: both pieces identify a system where performative behavior is incentivized over sincere behavior. Chen's essay is about consumers learning to game pricing algorithms; Smith's library is about founders learning to game visual trust signals. In both cases, the performance works until everyone does it, at which point it becomes table stakes and the arms race escalates.

What Smith doesn't say — but the library implies — is that **these components work**. They convert. The glowing middle pricing card sells more of the middle tier. The logo marquee increases trust. The token stream demo gets people to sign up. The critique isn't that these patterns are ineffective; it's that their effectiveness is independent of the product's quality. A landing page built entirely from performative-ui could generate a waitlist of 10,000 people for a product that doesn't exist. And many do.

The library's relationship to [[Intent Is the Interface]] is instructive. Marc argues that interfaces should be derived from capabilities and intents rather than designed for specific screens. Smith shows what happens when the opposite occurs: when the interface precedes the capability, when the prompt textarea ships before anyone knows what the model does, when the status dot is green before there's a system to monitor. Performative UI is what you build when you're designing an interface for a product you haven't figured out yet.

There's also a [[Creative Firewall]] dimension: these components are the visual equivalent of AI-generated text. They're the "delve" and the em dash of startup design — tells that signal you're optimizing for appearance rather than clarity. [[Why Does AI Write Like That]] mapped the linguistic tics of AI prose; Smith maps the visual tics of AI product design. The two catalogs are complementary.

The unstated question hanging over the library: **what would sincere UI look like?** If performative-ui is the satire, what's the alternative? Smith doesn't answer this, but the implication is that sincere UI would be boring. It would explain what the product does in plain text. It would show real code, not a MockIDE. It would have a status dot that sometimes goes red. It would make the pricing clear without glowing boxes. It would look less like an AI company and more like a company.

Which is, of course, the problem. Nobody funds boring.

---

## Cross-Links

- [[Intent Is the Interface]] — interfaces should derive from capabilities; performative UI is what happens when the interface precedes the capability
- [[The Behavioral Cost of Personalized Pricing]] — performative vs. sincere behavior as a systemic incentive problem
- [[Why Does AI Write Like That]] — Kriss's taxonomy of AI prose tics is the linguistic parallel to Smith's visual tic catalog
- [[Various LLM Smells]] — the user-side companion: recognizing your own writing becoming performative
- [[Creative Firewall]] — the boundary between authentic output and AI-optimized output; these components are on the wrong side
- [[AI Livestream Factories]] — another domain where performance exists without substance
- [[Vibe Coding and the Maker Movement]] — "evaluative anesthesia": the dopamine of making eclipses the ability to judge what was made
- [[Igor Schwarzmann Design Systems]] — design systems as organizational strategy, not component libraries; Smith's satire works because it's a real design system for performative purposes
- [[Experience Design for Agents]] — UX determines adoption; performative UI is UX optimized for investors, not users
- [[You Can Just Say It]] — "AI slop = form without discernible intent"; Smith catalogues the forms

---

*Sources: [[raw/performative-ui]]*
*Last updated: 2026-06-09*
