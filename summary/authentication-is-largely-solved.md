---
url: https://www.technometria.com/p/authentication-is-largely-solved
title: Authentication Is Largely Solved
author: Phil Windley
date_fetched: 2026-09-11
---

Phil Windley — CIO of Utah in 2001, co-founder of the Internet Identity Workshop, author of *Digital Identity*, *The Live Web*, and *Learning Digital Identity* — announces his fourth book, *Authorization in Action* (Manning), with a thesis he says he didn't expect to be making: authentication is largely a solved problem, and authorization is not.

The solved half: passkeys and FIDO have quietly closed most of the gap that "kept him up at night in 2005." The unsolved half: deciding what someone is allowed to do is still improvised, buried in application code, and reinvented badly on every team. Knowing who someone is tells you almost nothing about what they should be able to do — the interesting, unsolved work all lives on the far side of authentication.

The second question is harder because it is actually several questions: *what* can this person do, *under what conditions*, *on whose behalf*, and *in which context*? Miss any one and the rest stop meaning much — the same request can be right for a manager at noon from the office and wrong for a contractor at midnight from an unknown device.

What convinced him the book was needed is AI agents. An agent that can read your calendar, spend your money, or send mail as you is only as trustworthy as the boundaries around it, and those boundaries are authorization. Without the ability to say precisely what an agent may do on our behalf, we are left choosing between agents that can't do anything useful and agents we have to trust blindly. Authorization is the infrastructure that lets us delegate real authority to software while keeping it bounded, accountable, and revocable.

Working at AWS Identity (with the Amazon Verified Permissions and Cedar policy language team) taught him something theory hadn't: fine-grained authorization doesn't only make a system safer, it makes it more usable — explicit, externalized rules let you hand people exactly the access they need without drowning them in prompts or handing over the keys. The book works this out through a fictional company, ACME, moving from tangled access-control code to policy-based authorization, aimed at the developers and architects who have to make these decisions every day. A commenter notes that Cedar creator Sarah Cecchetti — who demonstrated Cedar for AI-agent guardrails on her standalone "Clawdrey Hepburn" Mac mini — wrote the book's foreword.
