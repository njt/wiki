---
url: https://www.thoughtworks.com/insights/blog/open-source/zero-cost-fallacy-open-source-agentic-era
title: "The zero-cost fallacy: Open source software in the agentic era"
authors: Chris Ford, Richard Gall
date_fetched: 2026-07-18
date_published: 2026-07-09
site: Thoughtworks Insights Blog
---

# The zero-cost fallacy: Open source software in the agentic era

The open source software movement birthed a utopian idea: software that is free to use, distribute, and inspect. But the long-standing perception that it requires little maintenance effort or investment is as dangerously flawed as ever. Generative AI has only sharpened existing tensions, placing financial and technical pressures on the ecosystem just as that same ecosystem is becoming more critical to the wider digital economy than ever before.

We've become accustomed to the magic of open source. You discover a library that does what you need, `npm install` it, and it works. The licensing is permissive, the code is high quality, the documentation is thorough. All tools you can use under write-once license terms without ever interacting with the people who maintain them, let alone supporting them. The magic trick papers over the cost: that someone, somewhere, is doing the work to make all of this happen.

Over the last year, we've been investigating how generative and agentic AI is transforming the practice of software engineering. We've spoken to maintainers of popular open source projects who've told us about the economic and practical challenges they face when it comes to keeping their projects going. In many cases, everyone benefits from the value of open source except the people who create it. And we're now asking those people to take on even more work — as the quality, quantity, and security threats introduced by generative AI-produced code demand their attention.

## Everything is free, but maintenance is priceless

When people think of a resource that costs nothing to use, open source software is one of the first things that spring to mind. While distributing digital assets indeed costs near zero, maintaining software still requires dedicated work embodied in real human beings. This disconnect between distribution and maintenance costs is proving to be a dangerous blind spot.

The most obvious result of this mismatch is financial exhaustion. Being an open source maintainer may cost you your will to live, but worse — it often doesn't pay the bills. The maintainers of load-bearing open source packages — libraries whose uncompensated labour underpins the global digital economy — are often burning out, suffering hostile demands from billion-dollar companies consuming their labour without any form of reciprocation. We've collectively confused permissive licensing with a license to exploit. And we've done nothing to address the massive asymmetry in value capture that we've allowed to flourish at the heart of the digital economy.

Maintainers of critical open source projects have provided us with first-hand accounts of financial destitution. They've reported intense pressure from corporate consumers of their software to provide support for outdated or deprecated versions — including, memorably, maintaining compatibility with Microsoft Internet Explorer. They've told us of being hounded and bullied to the point of closing their projects.

Importantly, economics aren't the only potential source of exhaustion. In an AI-fueled environment where reputation and quality signals are becoming increasingly unreliable — and where the noise-to-signal ratio is increasing — we're depleting the thing that keeps open source maintainers going: the sense of satisfaction that comes from doing important work as part of a community, of being a trusted source, of contributing.

## AI slop flooding the zone

Generative AI has intensified another corrosive dynamic. With prompting, developers can now produce plausible-looking code in seconds — including pull requests aimed at major open source projects. A flurry of low-quality, often literally AI-hallucinated contributions are flooding repositories, turning maintainers into unpaid code reviewers in the same way they were already — as Nadia Asparouhova Eghbal traces in *Working in Public* (2020) — unpaid support staff.

A maintainer told us that: "some LLM-created issues in my GitHub issues are well documented with reproduction steps and can at times look like an impressive root cause analysis. But since the large language model has no way of actually running and testing things, the issue is just plausible-sounding nonsense that wastes an enormous amount of time."

The result? Maintainers are at real risk of burn-out. Some may close their projects entirely to protect themselves. And in a vicious cycle, those that don't may respond by restricting community contributions — closing off the very ecosystem that might produce their successors.

## Star-struck? How open source signals have degraded

Coupled with that, the "signals" the open source ecosystem relies on for trust are degrading. The volume and accessibility of AI tools that put a "decent-enough" codebase together without any human effort, combined with the ability to mass-produce marketing that seems to prove genuine community engagement, means that it's now all but impossible to distinguish genuine adoption from AI-hyped launch spam.

GitHub stars — a traditional heuristic for quality and community health — have become borderline meaningless as a security signal. A project can skyrocket in popularity for a week, accumulate thousands of stars, all while having a practically non-existent commit history displaying only AI-generated code. As some maintainers told us: "Stars don't say something about the project, they say something about the marketing."

At the same time, the low cost of producing plausible code has made malicious pull requests a far more significant threat. This has the effect of degrading the overall trust in the commit history, even where no vulnerability exists. Open source's reputational commons, slowly accrued over decades, is at risk.

## Permissive licensing as a structural failure

Permissive licensing emerged from the belief that freedom and transparency in software development would yield an ecosystem that is collectively better off. But many open source maintainers now see this as a severe tactical error. One participant in a recent ThoughtWorks-hosted summit on the future of software delivery called permissive licensing "a profound collective mistake."

Restrictive licenses have emerged as a response, but the fix creates its own set of problems. Traditional open source business models were strained even before the AI boom — often built on a small pool of paying enterprise customers effectively subsidizing the rest. As AI coding tools reduce the friction of "swapping out" a dependency or replicating functional behaviors without the original library, those commercial models are further undermined.

The paradox is structural: permissive licensing enabled the vast ecosystem we have today — from Linux to Python — but it also created the conditions for the exploitation that leaves its maintainers exhausted and uncompensated. Adding commercial restrictions might capture revenue, but it forces the maintainer to become an enforcer — a skillset few possess and even fewer want. And procurement departments are now responding by blocking anything that's not standard open source licensing, gatekeeping maintainers from key revenue opportunities even when those maintainers are willing and able to pay.

## What happens when the money finally runs out

The emerging reality is one of asymmetrical tragedy of the commons: value is being rapidly extracted and then captured with no way of it being released back to the people who built and maintain the commons. The result threatens a systemic collapse that could far exceed the log4j vulnerability in scope and severity.

"Everything is going to burn to the ground": another participant in our summit laid the problem bare. Hostage to a handful of largely uncompensated maintainers, some of the world's most critical digital infrastructure is a single resignation away from making the CrowdStrike outage look like a productive afternoon. If the economics don't make sense, the people stop showing up. It's that simple.

In an immediate sense, the age of generative AI makes this situation worse by increasing the pressure on trusted repositories — more AI-generated code means more demands on maintainers. But the emergence of coding agents also offers a potential, longer-term escape route.

## Spec, not code: separating function from implementation

The debate within the software community has shifted from whether AI can write code to what "code" even means on a five-year horizon. An emerging thesis, present at our summit and gaining traction in the broader ecosystem, is that the medium of exchange between developers and machines may shift from code to specification.

There's a scenario where open source changes its fundamental character. If a coding agent can reimplement specific functional fragments from an open API specification — effectively creating a tailored, disposable library on demand — the economic calculus around dependencies would radically change. Rather than importing an entire library with its attendant supply chain risks and licensing constraints, you might import a behavioral specification and let your coding agent realize it in your own stack. This is fundamentally an inversion of the current model of shared maintenance.

However, there are still limits to this approach. Complex engineering shouldn't be distilled to PRD-style requirements: high-performance databases, cryptographic libraries, and other deeply technical systems are unlikely to be crowd-sourced into existence. And while reimplementation sidesteps licensing, it also denies maintainers the recognition and credit that, for many, is their primary compensation. It would amount to a silent, scalable refusal to acknowledge the intellectual labor that built the digital world.

## The end of the free lunch — and what to do about it

Open source as we've known it — freely maintained by a community of altruists — is breaking. AI didn't break it. It exacerbated an already unsustainable asymmetry. What changes is the urgency and the concrete steps we can take.

The era of the unvetted, un-patronized, completely permissive free lunch is coming to an end. The challenge for the software industry now — and more importantly, its leaders — is whether to acknowledge this and respond, or continue to extract value from a commons that everyone can see is under existential threat.

We suggest a few things that engineering teams can do now:

**Treat open source dependencies as active responsibilities, not passive gifts.** As one contributor to the summit told us: "You should treat every open-source dependency not as a free gift, but as code you have effectively hired." This shift in mindset — from passive consumption to active ownership — means funding projects you rely on. But it also means being ready and able to fork, patch, or even rewrite any critical dependency should the maintainer respond to the pressures we've detailed in this piece and step away.

**Rigid supply chain auditing.** As trust signals degrade, the quality of your auditing pipelines becomes the differentiator. Stop relying on metrics like GitHub stars as a signal of quality or security, and take the extra steps to know what you're consuming. This involves automated sandboxes to test new dependencies, but also tools like verified registries that provide cryptographically-signed attestations of provenance. Dependency management needs to become dependency verification.

**Formalize your contribution budget.** If you have money to spend on cloud infrastructure, security tooling, or proprietary software, you have money to contribute to the open source projects you depend on. This is not a charitable act, but a form of basic risk mitigation. Where possible, encourage and enable your engineers to contribute to core projects in work time. Ensure your legal department has a clear, light-weight process for approving upstream contributions. If that sounds like a hard sell, frame it this way: funding the projects that you rely on is as basic a risk mitigation strategy as you can have.

### About the Authors

**Chris Ford** is a Principal Technologist, Head of Technology, ThoughtWorks UK.

**Richard Gall** is a Lead Consultant and Writer, ThoughtWorks UK.
