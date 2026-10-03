# Evaluating AI-Generated Frontend Code: What Should We Actually Test?

An O'Reilly Radar essay arguing that AI-generated frontend code is judged far too early because its polish — clean formatting, reasonable names, bundled passing tests — manufactures false confidence. The essay proposes a concrete evaluation ladder (semantic structure → keyboard path → focus → failure states → full user flows → accessibility → generated-test review) and reframes AI's value in frontend as a shift of engineering attention from generation to evaluation.

---

## Key Quotes

> "A better evaluation process starts with a simple assumption: generated frontend code is a draft until the user behavior has been checked."

The whole essay compressed into one sentence. The interesting word is *draft* — it's a status claim, not a quality claim. The code may be fine; the point is that you can't know yet, and the visual evidence that tempts you to call it done is precisely the evidence least correlated with whether the UI works. This is the frontend-specific version of the verification-over-generation thesis running through [[Agent Coding Workflow]].

> "That polish can make reviewers less likely to slow down and ask whether the interface actually works."

The essay's sharpest psychological observation, and the one most worth dwelling on. Generated code isn't just unreviewed because teams are rushed — it's unreviewed because it *looks* reviewed. Well-formatted code with sensible component names and a test file triggers the same heuristics that signal a careful human author. This is a source of comprehension debt the agent-era literature ([[Agents and Acquiring Debt]], [[Principal Drift]]) describes structurally but rarely pins to a concrete visual trigger the way this essay does.

> "A custom dropdown, for example, may open on click and look finished in a demo, but it may not respond correctly to keyboard input. That is not a small edge case. It is part of whether the interface is usable."

The keyboard path as the single most revealing test: put the mouse aside and try to complete the task. This is a cheap, unfakeable check — exactly the property that [[Software Factories, Light and Dark]] argues autonomy must be gated on. It's also the check most likely to be skipped, because the agent that generated the component tested the interaction model it imagined, and the imagined model is mouse-first.

> "Generated tests often reflect what the implementation already does... If the answer is no, the tests may be documenting the implementation more than protecting the user experience."

The review question for generated tests: *would they fail if a validation error was unclear? Would they fail if keyboard navigation was broken?* This converges exactly on [[Test Validation and the Trustworthiness of Tests]]' "who is validating the tests?" argument — AI-generated tests deserve the same scrutiny as AI-generated production code — and adds a frontend-specific failure mode: tests that assert text appears or a component rendered are snapshots of the happy path, which is the one path this essay says is least representative of real usage.

> "AI-generated UI should not be trusted because it looks complete. It should be trusted because the team has checked the right things."

The closing formula, and a direct counter to screenshot-driven evaluation. Note what it implies for tooling: the thing to build is not better generation but better evidence — keyboard walkthroughs, focus assertions, error-state coverage, flow-level end-to-end tests. That's an argument about what the verification harness for UI work should measure.

---

## Key Themes

- #concept — polish as a false-completeness signal; "draft until user behavior is checked"
- #pattern — the evaluation ladder: structure, keyboard, focus, failure states, flows, a11y, generated tests
- #concept — risk-matched evidence: not every change needs the same review depth
- #concept — generated tests as documentation of the implementation, not protection of the user

## Opinion

The essay is deliberately unglamorous and that's its strength. It names no framework, sells no product, and its checklist could have been written in 2015 — semantic HTML, focus management, error states. That's the point: almost nothing here is new knowledge; what's new is that the population of code authors no longer has the apprenticeship that used to transmit it. Models reproduce the training distribution, and the training distribution is happy-path-heavy and div-heavy, so AI-generated UI concentrates the exact failures that accessibility practitioners have spent decades correcting. The essay's quiet implication is that accessibility expertise just became a load-bearing review skill rather than a specialist afterthought.

Two things it understates. First, the economics: it gestures at "match the evaluation to the risk" but doesn't say who decides or how that judgment is recorded — a young team of agent-supervised juniors will not reliably self-assess risk. Second, it stops short of automation: the keyboard path and focus checks are described as manual review acts, but they're exactly the checks a Playwright agent could run on every generated change — the natural fusion with [[A New Era for Software Testing]]'s agentic QA and Slack's "agents verify goals" finding in [[Agentic Testing]].

Where this fits in the wiki: it is the most UI-specific statement in the corpus of the general claim that verification is the bottleneck of agentic development, and it supplies the concrete evidence categories (keyboard, focus, failure states) that harness-level treatments like [[Guardrails and Feedback Loops]] treat abstractly. It nuances [[Agentic Manual Testing]] — Willison's "never assume LLM code works until executed" — by pointing out that *executed* is not enough for UI: the code must be exercised along the paths users actually take, most of which are invisible in a click-through demo.

---
*Sources: [[raw/evaluating-ai-generated-frontend-code-what-should-we-actually-test]], [[summary/evaluating-ai-generated-frontend-code-what-should-we-actually-test]]*
*Last updated: 2026-10-03*
