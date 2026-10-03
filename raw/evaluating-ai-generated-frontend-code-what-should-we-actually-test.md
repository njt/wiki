---
url: https://www.oreilly.com/radar/evaluating-ai-generated-frontend-code-what-should-we-actually-test/
date_fetched: 2026-10-03
---

AI can now generate a surprising amount of frontend code from a short description. A developer can ask for a form, a table, a modal, a settings page, or a dashboard view and get something that looks usable almost immediately. It may compile, render, and even arrive with a few tests. That is useful, but it also creates a problem: The first version of the UI can look more complete than it really is.

Frontend code is often judged too quickly. If the build passes and the screen looks close to the design, it is tempting to treat the generated code as mostly done. But a user interface is not just a collection of components on a page. It is a path someone has to move through. It has to handle input, state, errors, loading, navigation, focus, responsiveness, and accessibility. Some of the most important failures are not visible in a screenshot.

This is why teams need better ways to evaluate AI-generated frontend code. They need evidence that the UI is ready for people to use.

**Build and render are only the starting line**

The easiest checks are usually the first ones teams run. Does the code compile? Does the page render? Are there obvious console errors? Does the component appear in the browser? Those checks matter, but they are only the starting line. A page can render while the form is difficult to complete. A modal can appear while focus remains behind it. A generated test can pass while the actual user flow is broken.

This is especially important with AI-generated code because the output often has a polished surface. The code may be formatted well, the component names may sound reasonable, and the test file may make the change look more complete than it is. That polish can make reviewers less likely to slow down and ask whether the interface actually works. A better evaluation process starts with a simple assumption: generated frontend code is a draft until the user behavior has been checked.

**Start with the structure of the page**

Before looking at more complex behavior, it is worth checking whether the generated UI has a sound structure. Frontend evaluation should include the basic semantics of the page, not just the visual layout.

That means checking whether the code uses native HTML where possible. A button should usually be a button, not a clickable div. A link should be used for navigation, not for actions that behave like buttons. A form field should have a label that is connected to it. These details are easy to overlook because the UI may look fine without them, but they affect how people navigate, how assistive technologies interpret the page, and how maintainable the code will be later.

AI tools sometimes choose generic containers where native elements would be better. They may also add ARIA without using it correctly. ARIA stands for Accessible Rich Internet Applications, a set of attributes defined by the W3C to help make web interfaces more accessible when native HTML is not enough. The W3C’s WAI-ARIA overview is a useful reference. ARIA can be important, but it should not be used as a substitute for the right HTML element. The first evaluation question should be: Did the generated code use the right building blocks?

**Check the keyboard path**

A useful frontend evaluation should include the keyboard path through the interface. Many users rely on keyboards or keyboard-like navigation, and keyboard testing also exposes problems in the interaction model.

The simplest test is often the most revealing: put the mouse aside and try to complete the task. If the flow becomes confusing, the generated code is not ready. Can you reach the important controls? Is the focus order logical? Can you open and close a dialog without a mouse? When the dialog closes, does focus return to a sensible place?

These checks are especially important for generated UI because AI can produce interactions that work for the most obvious mouse path but fail in less visible ways. A custom dropdown, for example, may open on click and look finished in a demo, but it may not respond correctly to keyboard input. That is not a small edge case. It is part of whether the interface is usable.

**Test focus, not just clicks**

Click-based tests are useful, but they can hide important problems. A test that clicks a button and waits for a success message may pass even when the same flow is frustrating for someone who is navigating by keyboard.

Focus behavior deserves its own attention, especially when the UI changes after the user takes an action. For example, when a form submission fails, the user should not be left guessing what happened. The error should be visible, connected to the relevant field when appropriate, and reachable in a way that makes recovery clear. In many cases, focus should move to the first error or to a summary that explains what needs attention.

The same idea applies to modals. When a modal opens, focus should move into it. When it closes, focus should return to the element that opened it. These are small details in code, but they make a large difference in whether the UI feels predictable.

**Evaluate what happens when things go wrong**

Generated frontend code often looks best in the happy path. The user fills everything in correctly, the network responds quickly, the data shape is exactly as expected, and nothing fails. Real interfaces spend a lot of time outside that path.

A practical evaluation should check what happens when data is missing, delayed, empty, invalid, or returned in an unexpected state. This is what I mean by loading, error, and empty states. They are the parts of the interface that explain what is happening when the ideal path breaks down. A loading state should help the user understand that something is in progress. An error state should explain what went wrong and what the user can do next. An empty state should make it clear whether there is nothing to show, whether the user needs to take action, or whether something failed quietly.

These cases are easy to leave for later because the happy path is usually enough to make the screen look finished. But users will eventually hit the less perfect paths. A generated component may include a spinner because the prompt asked for one, but that does not mean the loading experience is useful. An error message may say “Something went wrong,” but offer no recovery. Evaluation should include these cases because this is where many real user experiences break.

**Test the full user flow**

Component-level checks are helpful, but they do not always tell the full story. A component can work by itself and still fail when it is placed inside a larger flow.

That is why AI-generated frontend code should be evaluated through user tasks. Can someone start the flow, understand what is expected, recover from a mistake, submit successfully, and see what changed afterward? Does the interface still work on a smaller screen? Does the state remain consistent if the user goes back, edits something, or retries after a failure?

This is where Playwright-style tests or other end-to-end tests can be useful. The goal is not to automate every possible interaction. The goal is to protect the flows that matter most. A good test should determine whether the user can complete the task the component is supposed to support.

**Use accessibility checks, but do not stop there**

Automated accessibility checks are useful and should be part of the evaluation process. They can catch missing labels, invalid ARIA usage, some contrast issues, landmark problems, and other common mistakes. They are especially helpful when AI-generated code is moving quickly because they catch issues before they become repeated patterns.

But automated checks are not a complete accessibility review. They cannot fully judge whether a flow is understandable, whether focus movement feels natural, or whether instructions are clear. Passing an automated accessibility scan does not mean the UI is accessible. It means some common problems were not detected.

The best approach is to combine automated checks with behavior-based review. Run the tools, but also use the interface. Navigate by keyboard. Trigger an error. Try the empty state. Look at the generated code and ask whether native HTML could do more of the work. Accessibility evaluation is strongest when it is part of normal frontend quality, not a separate pass at the end.

**Review the generated tests too**

When AI generates code, it may also generate tests. That sounds helpful, but those tests need to be reviewed with the same care as the code.

Generated tests often reflect what the implementation already does. They may check that text appears, that a function was called, or that a component was rendered. Those checks are not useless, but they can create false confidence if they do not test meaningful behavior. A better review asks what the tests would catch if the UI broke. Would they fail if a validation error was unclear? Would they fail if the retry button did not work? Would they fail if keyboard navigation was broken?

If the answer is no, the tests may be documenting the implementation more than protecting the user experience. Teams can use AI to help write better tests, but the prompt matters. “Write tests for this component” is too vague. A better request explains the behavior that matters, such as validation recovery, loading behavior, successful submission, and focus movement. Even then, the generated tests still need human review.

**Decide what evidence is enough**

Not every UI change needs the same level of evaluation. A small copy update does not require the same review as a new checkout flow, onboarding flow, or account settings page. Teams need judgment.

A useful approach is to match the evaluation to the risk of the change. If the generated code affects a critical user flow, collects user input, changes navigation, introduces a custom interaction, or handles important status messages, it deserves deeper testing. If it reuses stable components in a familiar pattern, the review may be lighter.

There is no need to create a checklist for every pull request; clarity on what evidence is enough suffices. For some changes, a quick review and component test may be fine. For others, the team should expect keyboard testing, accessibility checks, error-state review, and a user-flow test. The point is to avoid treating all generated code as equally trustworthy just because it looks polished.

**Human review still matters**

AI can generate code and suggest tests, but it cannot fully understand the product, the users, or the trade-offs behind a frontend decision. It does not know which flows are most important, which interaction patterns users already rely on, or where inconsistency will cause confusion.

That is why human review remains central. The reviewer’s role is to ask whether the generated solution fits the system and supports the user’s task. Sometimes that means accepting the generated code. Sometimes it means asking for a simpler native element, reusing an existing component, improving the error recovery, or adding a test that reflects real behavior.

The more code AI generates, the more important this judgment becomes.

**What should we actually test?**

When AI writes frontend code, teams should test the parts of the interface that users depend on. That includes structure, keyboard access, focus behavior, loading and error states, form validation, responsive behavior, accessibility checks, and full user-flow completion. It also includes reviewing the generated tests themselves to make sure they protect behavior rather than merely confirming the current implementation.

The goal is not to slow down AI-assisted development. The goal is to make it safer to use. If AI reduces the time spent producing a first draft, teams have an opportunity to spend more time asking whether the software actually works properly.

That may be the real shift. In frontend development, the value of AI is not just faster code. It is the chance to move more engineering attention toward evaluation, user behavior, and quality.

AI-generated UI should not be trusted because it looks complete. It should be trusted because the team has checked the right things.

**AI use acknowledgment**

AI assistance was used lightly for phrasing, editing, and tightening parts of this draft. The article’s ideas, structure, examples, and final review are my own.

**Author’s note**

The views expressed are my own and do not represent those of my employer.
