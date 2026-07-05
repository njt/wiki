---
url: https://willybrauner.com/journal/signal-the-push-pull-based-algorithm
date_fetched: 2026-07-05
backfilled: true
---

We have been using Signals in production for years via several modern front-end frameworks like Solid, Vue, and others, but few of us are able to explain how they work internally. I wanted to dig into it, especially diving deep into the push-pull based algorithm, the core mechanism behind their reactivity. The subject is fascinating.

## The state of the world

Imagine an application as a world where we describe the set of rules that govern it. Once a rule is defined, our program will no longer be able to change it.

For example, we decide that in our world, any `y` value must be equal to `2 * x`. We define this rule, and from then on, whenever `x` changes, `y` will automatically adjust. We can define as many rules as we want. They can even depend on each other by deciding that `z` must be equal to `y + 1`, and so on.

Now we press the play button, our program starts, the world is running, and the rules we have defined are now in effect over time. (think of it as our runtime).

And then, we just have to observe. We can modify `x` and see how `y` and `z` automatically adjust to comply with the rules we have established. It's like a spreadsheet where dependent cells automatically update when their sources change. In other words, derived values are **reactive** to changes in their dependencies.

These derived values behave like pure functions: no side effects, no mutable state. In the next example, `time` is a source that changes continuously while `rotation` is derived from it. The square simply reflects the result of this transformation that is declared once.

This "reactive world" didn't come out of nowhere. The idea emerged in the 1970s and was formalized as Reactive Programming, a paradigm that describes systems where changes in data sources automatically propagate through a graph of dependent computations, which is exactly what Signals do.

Signals are thus heirs to the Reactive Programming paradigm, whose first JavaScript implementations came with libraries like Knockout.js (2010) and then RxJS (2012), which brought reactive ideas to the browser.

Now that we have more context on what Signals are, let's dive into the push-pull based algorithm that is at the core of this system.

## Signals: Push-based

A **Signal** is an abstraction that represents a reactive value that can be read and modified. When a signal changes, all parts of the application that depend on this signal are automatically updated. I went through the exercise of implementing a very basic version:

```
const signal = <T>(initial: T) => {
  let value: T = initial
  const subs = new Set<(state: T) => void>()
  return {
    get value(): T {
      return value
    },
    set value(v: T) {
      if (value === v) return
      value = v
      for (const fn of Array.from(subs)) fn(v)
    },
    subscribe(fn: (state: T) => void) {
      subs.add(fn)
      return () => subs.delete(fn)
    }
  }
}
```
We can imagine Signals as the starting point of the rules of our world, the primitive entry points of targeted mutations.

My first thought was "Ok, it's just a simple publish–subscriber pattern with a getter and a setter." The Signal itself works like that, except the function keeps a reference to the current state that can be read and modified. If you have ever used an event emitter, this pattern will seem familiar to you:

```
const count = signal(0)
// Somewhere in the application
count.subscribe((newValue) => {
  console.log("Count changed to:", newValue)
})
// Anytime and anywhere in the application
count.value += 1
// "Count changed to: 1"
```
This is what we call the **push** approach, also known as **eager** evaluation. A notification is immediately *pushed* to its subscribers when the signal is updated. Updating the signal dispatches a notification to all its subscribers.

I deliberately use the term "notification" and not "state" because **Signals, using the push-pull based algorithm, don't dispatch a state value; they notify that their own state has changed**; this is not the same. We will talk about *cache invalidation* in detail in the next section. Keep in mind that the dot moving between "nodes" is only a notification. (In the next modules, you can click the Signal to see the notifications being dispatched to its subscribers).

In this more complex example, we have multiple "nodes" that depend on each other. All of them can notify their own subscribers that their state has changed.

At this point, we understand that the push-based approach propagates downward through notifications, and now we have to explore how the pull-based approach propagates upward through re-evaluation. What does that mean?

## Computed: Pull-based

One of the most important aspects of Signals may not be the `signal` function itself, but the `computed`. They are **reactive derived functions** that compute values based on signals or computeds. We can imagine them as signals without a setter.

First, the main difference between signals and computeds is that computeds are **lazy**. They are **invalidated** (not updated) whenever one of their dependencies changes. Furthermore, they are updated only when they are read, if they have been invalidated first (our cache system). This is what we call the **pull-based algorithm**.

Secondly, computeds **automatically track their dependencies**. They subscribe to changes in the signals/computeds they access during their execution. It's one of the most "magical" aspects of this system that developers love, compared to React where we have to manually specify the dependencies of a `useEffect` or `useMemo` with the dependency array. Let's see how we can implement a simple version:

```
const computed = <T>(fn: () => T) => {
  let cachedValue: T
  // ...
  const _internalCompute = (): void => {
    // ...
    cachedValue = fn()
  }
  return {
    get value() {
      _internalCompute()
      return cachedValue
    }
  }
}
```
The thing to note here is that accessing the `value` property of the computed object triggers the `_internalCompute` function, which re-evaluates the computation and updates the cached value (not actually cached for now, but we will address this later).

```
const count = signal(1)
const doubleCount = computed(() => count.value * 2)
const plusOne = computed(() => doubleCount.value + 1)
// Update the signal…
count.value = 5
console.log(doubleCount.value) // 10
console.log(plusOne.value) // 11
```
We know this code, right? Now, look at the dependency tree of this program and focus on the "pull" aspect of the algorithm. You can click the computed to see how the dot moves up the tree when we read its value.

We can observe that **the computed being read has no knowledge of the entire tree**. It only knows what its sources (dependencies) are and what its subscribers (dependents) are.

Check the same module with a more complex dependency tree. We can see what happens when a `computed` function has multiple dependencies at the same time (this is the case for the lowest node in the tree):

Some questions remain about the implementation of this system at this point, and this is where Signals become more complex and interesting:

- How does the `computed`function process the link between its sources and itself? (the auto-tracking of dependencies)
- How does the cache system work, allowing re-evaluation of the computation only when necessary?

## The magic link

The link between signals and computeds is somewhat *magical*. As mentioned before, no need to explicitly declare the dependencies of a computed value on signals, as we do in React (with the damned dependency array). The system automatically tracks which signals are accessed during the execution of the computed function. This is what we will discover in this section.

Back to our previous example with the `count` signal, `doubleCount` and `plusOne` computeds.

```
const count = signal(1)
const doubleCount = computed(() => count.value * 2)
const plusOne = computed(() => doubleCount.value + 1)
// Update the signal…
count.value = 5
console.log(doubleCount.value) // 10
console.log(plusOne.value) // 11
```
Keep this program in mind. To understand the mechanism of the auto-tracking, the best way is to look at the implementation of a Signal library in detail:

We start with the update of `count`, which is a `signal`. The flow of our program starts here.

`count.value = 5`

When we call the setter `value` of the `count` , it updates the internal value of the signal and notifies all its subscribers. **But for now, it has nobody to notify**. Computeds have been created but links between signals and computeds are not established yet.

`console.log(doubleCount.value)`

Now, `computed` enters the game.`doubleCount.value` is invoked in `console.log`.

So the program wants to access the getter of this factory function, and multiple things happen here. We will not cover all of them in the next step, let's go directly to the `dirty` flag.

The getter looks at the internal `dirty` flag. This is a **cache flag** to say: *"If I'm dirty, I need to be recomputed"*.

`dirty = true` by default, because we need to compute the value of this `computed` the first time we access it. So, we execute the internal function `_internalCompute`.

Inside this function, we will push information into the global `STACK` to say: *"I'm the one currently running this computed".*

We push two things:

- the ability to mark this computed as dirty again (`setDirty`), plus mark as dirty all its subscribers.
- the ability to register sources (dependencies) for this computed (`addSource`)

`STACK` is a **global array** that allows us to keep track of the currently running `computed` functions. It will be available globally for all signals and computeds of the application. Imagine it as a store of "temp data" that any `signal` or `computed` can access when they are read.

(Note that it could be another data structure, this is a detail of implementation).

Then, the `fn` function that we passed as parameter to the `computed` is called and its result registered internally as `cachedValue`. This is the function that computes the value of this `computed`. According to our program, it will store `count.value * 2` so `10`.

But wait! By executing `fn`, we read `count.value`, a `signal`! Let's go to this getter.

In this `signal` getter, we have access to the global `STACK` with some fresh and precious data that we can use to create the **magic link** between `doubleCount` and `count`. (between the `computed` and the `signal`)

Because, there is temp data inside the `STACK`, the `signal` can say:

*"There is data inside the  STACK, so it means that the computed who has pushed inside it, depends on me. I need to add the function that invalidates its cache in my subscribers."*

Then, the next time that `count.value` will be updated, it will notify its subscribers, and set `doubleCount` as `dirty = true` again. We have a cache system! 🎉

`computed` has also pushed an `addSource` function that registers the signal as a source of this computed and stores a cleanup function to remove the `setDirty` subscription from that source when the computed is recomputed.

It means that if tomorrow, the `computed` needs to be recomputed and that during this recomputation, it doesn't depend anymore on `count` but on another `signal`, it can clean up the previous link and create a new one with the new source.

And this cleanup of all the sources is processed each time we start to recompute the `computed`. With that, we are sure to not keep dead `signal` subscribers and to always have an up-to-date graph of dependencies between these functions.

Back to the end of the `_internalCompute`. After the value is computed (the full process seen just before), we mark this `computed` as clean by setting the `dirty` flag to `false`. It means that we have a cached value that is up to date!

Finally, we remove this "temp data“ from the global `STACK` to say: *"I'm not running anymore".*

`console.log(plusOne.value)`

Now, what about `plusOne` that depends on `doubleCount`? We read its getter, and the same process happens as for `doubleCount`.

The `computed` looks at the `STACK` if there is temp data registered in it, exactly like the `signal` getter does. If it is flagged as dirty, it is recomputed. And we restart the whole process before returning the cached value.

```
type Sub<T> = (s: T) => void
type ComputeContext = {
  setDirty: () => void
  addSource: (cleanup: () => void) => void
}
const STACK: Array<ComputeContext> = []
export const signal = <T>(
  initial: T
): {
  value: T
  suscribe: (fn: Sub<T>) => () => void
} => {
  let value: T = initial
  const subs = new Set<Sub<T>>()
  return {
    get value(): T {
      const currentComputed = STACK[STACK.length - 1]
      if (currentComputed) {
        subs.add(currentComputed.setDirty)
        currentComputed.addSource(() => {
          subs.delete(currentComputed.setDirty)
        })
      }
      return value
    },
    set value(v: T) {
      if (value === v) return
      value = v
      for (const fn of Array.from(subs)) fn(v)
    },
    suscribe(fn: Sub<T>) {
      subs.add(fn)
      return () => subs.delete(fn)
    }
  }
}
export const computed = <T>(fn: () => T) => {
  const subs = new Set<ComputeContext>()
  const sources = new Set<() => void>()
  let cachedValue: T
  let dirty = true
  const _internalCompute = (): void => {
    sources.forEach((cleanup) => cleanup())
    sources.clear()
    STACK.push({
      setDirty: () => {
        if (dirty) return
        dirty = true
        for (const sub of Array.from(subs)) sub.setDirty()
      },
      addSource: (unsubscribe) => sources.add(unsubscribe)
    })
    cachedValue = fn()
    dirty = false
    STACK.pop()
  }
  return {
    get value() {
      const currentComputed = STACK[STACK.length - 1]
      if (currentComputed) {
        subs.add(currentComputed)
        currentComputed.addSource(() => {
          subs.delete(currentComputed)
        })
      }
      if (dirty) _internalCompute()
      return cachedValue
    }
  }
}
```
We have demystified the **auto-tracking dependency system**, using the global `STACK`, which enables communication between the currently executing computed and the signals/computeds it accesses during its execution.

We have also seen how **the cache system** works in the pull-based algorithm, using the `dirty` flag to know when a computed is invalidated.

## Final flow

As described above, the final flow of signals is now possible by combining both push and pull mechanisms! You can click the signal or a computed to observe the invalidation and re-evaluation of nodes in the tree. Let's play with it!

Note that all

`setDirty`calls are synchronous; each node is invalidated when the dot passes through it. The delay is purely for visual purposes.

And that's it! We now have a complete picture of the push-pull algorithm at the core of Signals. I will not cover it here, but I still need to mention that most signal libraries also expose an `effect` function on top of the same tracking mechanism, but that belongs more to API design than to the algorithm itself.

## Conclusion

The article focused on the algorithm, so what makes Signals interesting is not just that they update some UI, but how they propagate change through a reactive graph:

- **push**for eagerly propagating invalidation;
- **pull**for lazily re-evaluating only when necessary.

This combination gives us a **fine-grained reactivity** system already adopted by many frameworks like Solid, Vue, Preact, Angular, Svelte, and others. Each comes with its own API surface, but shares the same underlying logic.

The Signals topic has already been covered in a large number of publications that greatly helped me understand the subject, but none that I found offered an in-depth analysis of implementing the push-pull based algorithm from scratch. To explore this subject in depth, I implemented my own version of the Signal system, certainly very naive compared to the great alien-signals, preact-signals or solidjs-signals, but functional enough to understand the concept.

Note that we may "soon" (maybe?) no longer need to implement this system manually, as this model is being standardized natively in JavaScript: TC39 proposal-signals (currently at Stage 1). This would be a major advancement for the entire JavaScript ecosystem, as it would allow each framework to rely on a common foundation, while retaining the freedom to choose the API that best suits them.

I greatly enjoyed writing this article and building interactive modules for it. If you learned something new or enjoyed reading it, consider supporting my work ☕️ or feel free to connect with me on Bluesky or LinkedIn 👋

## Sources

*I highly recommend taking the time to listen to this podcast episode, which helped me dive deep into this subject: How signals work by Con Tejas Code w/ Kristen Maevyn and Daniel Ehrenberg*

**Articles**

- Reactivity, Milo Mighdoll
- Introducing Signals Preact, Marvin Hagemeister and Jason Miller
- Signal Boosting, Joachim Viide
- The evolution of signals in JavaScript, Ryan Carniato
- State-based vs Signal-based rendering, Jovi De Croock
- Push-pull functional reactive programming, Conal Elliott

**Videos & podcasts**

- Beyond Signals, Ryan Carniato
- How signals work, Con Tejas Code w/ Kristen Maevyn and Daniel Ehrenberg
- Controlling Time and Space: understanding the many formulations of FRP, Evan Czaplicki

**Libraries**
