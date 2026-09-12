---
url: https://medium.com/airbnb-engineering/flexible-authentication-reimagining-authentication-for-millions-of-users-at-airbnb-3a8a4c917137
title: "Flexible Authentication: Reimagining authentication for millions of users at Airbnb"
author: Jose Santos, Mike Barry
site: Airbnb Engineering (Medium)
date_published: 2026-08-13
date_fetched: 2026-08-14
topics:
  - software-engineering-craft
---

Airbnb rebuilt its login and signup flows around a paradigm called **Flexible Authentication**, reframing authentication as a product problem rather than a purely technical one. The trigger: on a two-sided marketplace, logins happen at irregular intervals — a guest books in January and may not return until summer — so a failed login means a lost booking, not a minor inconvenience.

The core insight is **Identify first, then Challenge**. Instead of asking "can this person prove who they are?" as one monolithic question, the flow splits into two stages. First the person tells Airbnb who they are (email, phone, or social login). Then a server-side policy engine picks the challenge most likely to succeed for that person's context — a WhatsApp OTP for a Brazilian traveler, Naver login for a South Korean host — and leads with it, offering everything else as fallback.

The load-bearing architectural decision is that **the client never decides which challenge to present**. The server makes that call and the client renders whatever screen it receives. This lets Airbnb tune authentication per-region without shipping new client code.

A second hard requirement came from product: **no dead ends**. Every challenge screen carries a "Try another way" button backed by a server-driven **Challenge Picker** — a ranked list of alternatives ordered by predicted success rate for that person. When someone taps it, the flow adapts rather than restarts.

The whole experience became **server-driven**, with the screen as the unit of abstraction: each step (identifier input, challenge, account picker, error recovery) is a screen defined entirely by server response, and clients become thin renderers using generated type definitions from the server schema.

Results: 60% less code, a 100KB smaller web bundle, and — because flow rules now live behind a server response — 20+ experiments run in the three months after launch, with idea-to-measured-result turnaround collapsing from weeks to days. Share of visitors who successfully authenticate rose 2.6%, duplicate-account creation fell 27%, and OTP costs fell ~11%.

The authors frame Flexible Authentication as a framework for continuous product improvement, not a finished product. The generalizable lesson: for any flow where context determines the right experience, moving the decision boundary off the client unlocks the iteration speed needed to act on what you learn.

---
*Source: [[raw/flexible-authentication-reimagining-authentication-for-millions-of-users-at-airbnb-3a8a4c917137]]*
*Last updated: 2026-08-14*
