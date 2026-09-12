---
url: https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/
title: What Color is Your Function?
author: Bob Nystrom
site: journal.stuffwithstuff.com
date_published: 2015-02-01
date_fetched: 2026-08-08
topics:
  - software-engineering-craft
---

# What Color is Your Function? — Summary

Bob Nystrom's classic 2015 programming language rant uses an allegorical language with "red" and "blue" functions to explain why async/await doesn't actually solve the fundamental problem of asynchronous programming. The metaphor is devastatingly effective: in his strawman language, every function has a color (red or blue), you must use the correct calling syntax for each color, red functions can only be called from other red functions, and red functions are more painful to call. The reveal: red functions are asynchronous ones.

The core argument: async/await, promises, and callbacks are all band-aids on the same wound. The real problem is that asynchronous IO requires unwinding the entire callstack back to the event loop, forcing every function in the chain to become async — a "color" that infects everything above it. This is why "red functions can only be called from red functions."

Nystrom identifies the only languages that avoid this problem — Go, Lua, Ruby, and (pre-async) Java — and what they share: threads, or more precisely, multiple independent callstacks that can be switched between. Goroutines, coroutines, and fibers let you suspend the entire thread without unwinding the callstack, eliminating the sync/async distinction entirely. Go does this most beautifully: IO operations *appear* synchronous but don't block other goroutines.

The article traces the CPS (continuation-passing style) transformation that compilers do under the hood for async/await, noting the irony that programmers are now manually writing code in a style invented as a compiler intermediate representation in the 1970s. The 2021 update notes that JavaScript's async library count had grown to 15,118 — evidence that library-level patches can't fix a language-level problem.
