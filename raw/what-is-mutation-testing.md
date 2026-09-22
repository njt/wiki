---
url: https://stryker-mutator.io/docs/
date_fetched: 2026-09-13
---

# What is mutation testing?

**TL; DR: Mutation testing introduces changes to your code, then runs your unit tests against the changed code. It is expected that your unit tests will now fail. If they don't fail, it might indicate your tests do not sufficiently cover the code.**

Bugs, or *mutants*, are automatically inserted into your production code. Your tests are run for each mutant. If your tests *fail* then the mutant is *killed*. If your tests passed, the mutant *survived*. The higher the percentage of mutants killed, the more *effective* your tests are.

It's that simple.

Are you still confused? Why not take a look at our example page and try it out yourself?

## But wait, what about code coverage?

Well... code coverage doesn't tell you everything about the effectiveness of your tests. Think about it, when was the last time you saw a test without an assertion, purely to increase the code coverage?

Imagine a sandwich covered with paste. Code coverage would tell you the bread is 80% covered with paste. Mutation testing, on the other hand, would tell you it is actually *chocolate* paste and not... well... something else.

## Meet: Stryker

Sounds complicated? Don't worry! Stryker has your back. It uses one design mentality to implement mutation testing on three platforms. It's *easy to use* and *fast to run*. Stryker will only mutate *your source code*, making sure there are no false positives.

## An example

Say you're building an online casino. Users can only enter the casino when they're over 18. So you write the following piece of code to check if someone is allowed to use the website:

```
function isUserOldEnough(user) {
  return user.age >= 18;
}
```
Stryker will find the return statement and decide to change it in several ways:

```
/* 1 */ return user.age > 18;
/* 2 */ return user.age < 18;
/* 3 */ return false;
/* 4 */ return true;
```
We call those modifications mutants. After the mutants have been found, they are applied one by one, and your tests are executed against them.
If at least one of your tests fail, we say the mutant is *killed*. That's what we want! If no test fails, it *survived*.
The better your tests, the fewer mutants survive.

Stryker will output the results in various different formats. One of the easiest to read is the clear text reporter:

```
  Mutant killed: /yourPath/yourFile.js: line 10:27
  Mutator: BinaryOperator
  -                 return user.age >= 18;
  +                 return user.age > 18;
  Mutant survived: /yourPath/yourFile.js: line 10:27
  Mutator: RemoveConditionals
  -                 return user.age >= 18;
  +                 return true;
```
The clear text reporter will output how exactly your code was modified and which mutator was used. It will then tell us if a mutant was killed, meaning that at least one test failed, or if it survived. The second mutation in this example is marked as a survivor. This means there is probably a test missing that explicitly tests for age lower than 18.

For someone who hates mutants... you certainly keep some strange company.


*- Professor X*

Oh, they serve their purpose... as long as they can be controlled.


*- William Stryker*
