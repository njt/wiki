---
url: https://btmc.substack.com/p/transaction-oriented-programming
date_fetched: 2026-10-11
---

Small administrative note: I’d like to apologize for the long drought between posts. I’ve been working on an article related to UI, and in order to accurately discuss some topics I’ve spent quite a bit of time working on a prototype. The issue is that it appears I’ve bitten off more than I can chew, given my very limited free time. Hopefully I’ll be able to get that article out eventually, but it is not coming out any time soon, so I decided to make a smaller post in-between.


In this post I want to discuss some ideas I’ve had regarding software design. I’ll put a disclaimer right away that I have not put these ideas into practice in any serious capacity. These ideas are, however, based on other existing ideas that have been very successful in practice, but not necessarily described in these terms or placed in the same “conceptual bucket”, despite their commonalities.

The main idea I want to discuss is what I call **Transaction-Oriented Programming** (because I am horrible at naming things). Right away what probably comes to your mind are databases, and databases can be seen as an instance of the idea, but what I’m thinking of as a “transaction” is broader.

The idea of **Transaction-Oriented Programming** is to figure out “**user-relevant units of work**” (more on what those are below), and to implement those units as **transactions** in your application, following the usual ACID properties1.

My hypothesis is that structuring software this way leads to a pit of success. Figuring out these units plus the constraint of making them transactions provides clear and actionable guidelines regarding software architecture, error handling, etc., that I believe ultimately lead to better software.

# User-Relevant Units of Work

What constitutes a User-Relevant Unit of Work is somewhat fuzzy (or at least fuzzy in my head), and I don’t have a clear definition for it, so instead I’ll try explain it with examples.2

What is a **User-Relevant Unit of Work** for the `cd` (change directory) command in the terminal? Well, it’s the whole command. There’s no subset of work the cd command does that is user-relevant. You either changed directories, or you didn’t3. 

I believe this is why using `abort()` as an error handling solution on many batch applications doesn’t feel particularly wrong, not in the way it would for a more interactive application. There’s no point in propagating errors around nor retrying the operation nor anything of the sort if the only valid response is to terminate the application outright. Consider the following:

- If the error was reported by throwing an exception, then the exception would be caught, its message printed, and the application would exit with an error code. 
- If an error code was propagated through a chain of function calls, then main would check the error code, map it to an error message, print it, and exit the application with an error code ( - `Result<T,E>`and friends are the same).
- If the error was placed in some internal error log, and then the rest of the program operated on a bunch of “null-objects” (as in Ryan Fleury’s error handling approach), then eventually main will check the error log, print out the message, and exit the application with an error code. 

No matter which error handling strategy you use in this case, you arrive at the same destination, so the correct error handling solution is whichever is the simplest/most convenient/most efficient: in this case, calling abort.

In this example, the entire command forms a trivial **transaction**. It either does the task completely, or not at all, and since it does not do multiple tasks in parallel it is trivially isolated (durability is not relevant).

For other commands it’s less clear. What is a **User-Relevant Unit of Work** for the `cp` (copy) command? It’s actually not the whole command anymore, necessarily, because of directories. If you copy a directory, then each file being copied can be considered a user-relevant unit of work most of the time.

If an error occurs (e.g., one file could not be copied), then this should usually not lead to the complete command failing, it should just log the error and continue on with the larger task at hand. In this case, aborting the program outright on the first file that failed to copy would be a pretty bad error handling solution.

But what about partially copying a file? Partially copying an individual file is very rarely wanted (since it would end up corrupted), so copying part of a file is not normally a user-relevant unit of work*. *The copy of each file is a unit, and each unit is a transaction: either the file is copied completely, or not at all.

But writing part of a file can be a user-relevant unit of work when downloading a file via BitTorrent for example, so what constitutes a user-relevant unit of work varies depending on the context, but in all cases error handling happens around the unit, and each unit is handled as transaction.

# Guidelines for Error Handling

Whichever error handling mechanism (exceptions, error codes, etc.) you use, they will all ultimately have to accomplish the same thing: abort the current transaction and inform the user. If the user is presented with an option when this happens, that’s a *new* transaction, the other one already failed.

So it doesn’t really matter which error handling mechanism you use, so much as how you use it. For some problems error codes will require you to propagate errors unnecessarily4, leading to needlessly noisy code, while for other problems aborting the program outright is clearly not a valid option. All mechanisms have to do the same job in the end, so pick whichever works best for the task.

You might think that exceptions are the perfect solution here, since you can put a “try-catch” block around the block of code that implements the complete “user-relevant unit of work”, but you’d be wrong. What makes you think there’s such an obvious block of code? The software might be implemented using a task queue and a thread pool, with multiple tasks running per “user-relevant unit of work”, which greatly complicates things.

You need to keep in mind that you are running a transaction, and you need a plan on how you’re going to ensure it is ACID.

# Guidelines for Software Architecture

In the example above we already saw how these “units” provide some guidance on how to structure a software application in terms of error handling: When an error occurs, you need to abort the current transaction.

But that begs the question: how exactly does one abort a transaction correctly? As a reminder, these are the four ACID properties you need to ensure:

- **Atomicity**: How do you ensure the transaction either happens in its entirety, or fails in its entirety?
- **Consistency**: How do you ensure that there are no “leftovers” polluting the app state and its environment when the transaction is aborted?
- **Isolation**: How do you ensure that tasks can execute concurrently without interfering with each other?
- **Durability**: How do you make the results of the transaction persistent? This may or may not be relevant for the application, but is worth considering.

If you think about it, you need to ensure most of these properties for software to be reliable, specially consistency! If an error occurs and your software is now in a weird in-between state, that corruption will eventually lead to broken behavior and the user needing to restart the application.

So how do you ensure these properties? Well, there are many ways, and different software architectures can make it either very easy, or very hard.

Distributed software is obviously a nightmare in regard to the above, since you need a lot of additional coordination to ensure every process stays on the same page. But distributed software often simply stores the serious “app state” in a database anyway, and the database has those properties, so it works out in the end.

Object-oriented software (as in, software architected around communicating objects with isolated state), is just a local and synchronous form of distributed software. The same problem is there, but now there’s no database to fallback to, so it is a disaster in this regard. It’s well known in OOP circles that maintaining object invariants is extremely important (the private keyword is there for a reason), but a web of consistent objects is not necessarily consistent as a whole!

Smalltalk works around this problem by being really good at Durability. If you break the Smalltalk image, you can just revert to an older working version. If not for that, you’d be screwed, since you can very easily change the state of a large web of objects in a way that is essentially impossible to recover from.

Software with a “Functional Core, Imperative Shell” fares much better. If you think about it, the “functional core” is trivially a **transaction**! It only does compute, producing a command or list of commands that are then handled by the “shell”. If there’s an error, then there cannot have been any pollution of application state since it was never touched. While the shell itself might have issues performing the commands, the “surface area” where problems can occur has been greatly reduced.

Some very prominent software developers vouch for this software architecture, and this paradigm I’m proposing gives some answers as to *why* it works well. 

The fact that the core is functional actually matters very little. It might have benefits for testability and such, but the fact that the app state remains consistent is the important part. Similar to how functions with local mutation are not really all that different from 100% pure functions, what matters is the referential transparency.

The same outcome could be achieved with a “double buffered” application state, where the code imperatively writes to the new state and it is only switched with the old state if no error occurs. Or by being able to “undo” the work you did up to the point the error occurred. They all implement the same thing: a **transaction**.

Immediate mode UIs, Reactive UIs, etc., are also all ways of reducing the chances of the application entering an inconsistent state. The more derived state, the less the app state can go haywire. But having the state be derived is just a mechanism. If you can ensure that every transaction always sets both the app and UI states, or neither if it fails, you end up in the same place! Derived UI state is just the easiest way to achieve the intended outcome.

# Transaction-Oriented Programming

Paradigms aren’t nearly as important as the specific properties they embody that are beneficial in practice, so I don’t think a whole new paradigm matters5.

If you are doing “Functional Core, Imperative Shell”, you’re already reaping most of the benefits an imaginary “Transaction-Oriented Programming” paradigm would give you. Same is true if you’re using a database, since that’s already an implementation of the paradigm by nature.

But as a fun exercise, what would Transaction-Oriented Programming look like? And how would you implement a generic and composable “transaction” concept that programs can be built around?

I’m going to assume “just implement the relational model” isn’t a valid answer, that’s another more specific paradigm. Plus you can implement it in a way where the database state is consistent but the application overall isn’t (i.e., due to side-effects).

# A Python Prototype

There are two ways we can implement transactions: either the transaction collects all the work to be done, and then commits it at the end if everything succeeded, or the transaction does the work right away, but then undoes it if there is an error. Depending on the task one or the other is better. We’ll implement the undo version but you could also have the two kinds co-existing:

```
class Transaction(ABC):
  def __init__(self):
    self._error: Exception | None = None
  
  def run(self):
    self._execute()
    if self._error:
      raise self._error
  def _execute(self) -> bool:
    try:
      self._task()
      return true
    except Exception as e:
      self._error = e
      self._undo()
      return false
  def _abort(self):
    if self._error is None:
      self.undo()
      self._error = AbortedTransactionError()
  @abstractmethod
  def _task(self):
    pass
  @abstractmethod
  def _undo(self):
    pass
```
For simplicity, we’ll assume transactions cannot be canceled midway, which would require using generators or async.

The only public method of a Transaction is `run()`, which executes the transaction to completion, possibly throwing an exception.

The `run()` method is, however, implemented using a separate `_execute()` method, which traps any exception that occurs. This split is what will allow us to compose transactions into larger ones that can be handled atomically.

Each transaction must implement two methods: `_task()`, which does the actual work, and `_undo()`, which reverses the work that was done up to that point.

The `_abort()` method is used by composite tasks to revert sub tasks that already finished successfully within the larger transaction.

We can make our first primitive transaction now:

```
class CopyFile(Transaction):
  def __init__(self, source: str, dest: str):
     super().__init__()
     self._source = source
     self._dest = dest
  def _task(self):
    shutil.copyfile(self._source, self._dest)
  def _undo(self):
    if self._source != self._dest:
      try:
        os.remove(self._dest)
      except:
        pass
t = CopyFile("hello.txt", "world.txt")
t.run()
```
Not particularly interesting, but this is our building block. Now let us create a composite transaction that executes its sub transactions sequentially, aborting if one of them fails:

```
class Sequence(Transaction):
  def __init__(self, children: *Transaction):
    super().__init__()
    self._children = children
    self._failed: int = -1
  def _task(self):
    for i, t in enumerate(self._children):
      if not t._execute():
        self._failed = i
        raise t._error
        
  def _undo(self):
    for i in range(0, self._failed + 1):
      self._children[i]._undo()
f1 = CopyFile("a.txt", "b.txt")
f2 = CopyFile("b.txt", "c.txt")
s = Sequence(f1, f2)
s.run()
```
We can also make a task that always succeeds, even if there is an error:

```
class Optional(Transaction):
  def __init__(self, sub: Transaction):
    super().__init__()
    self._sub = sub
  def _task(self):
    # ignore the error
    if not self._sub._execute():
      self._sub._undo()
  
  def _undo(self):
    if self._sub._error is None:
      self._sub._undo()
```
We could then use these building blocks to implement more complex transactions, for example, a transaction that recursively copies files from a directory to another.

# Transaction-Oriented Language

The above would let you create and manipulate transactions in an existing programming language, but what would a transaction-oriented language look like? I’m honestly not sure, but maybe something like the following:

```
transaction copy_folder(source, dest):
  mkdir(dest)
  for p in dir(source):
    if is_directory(p):
      copy_folder(join(source, p), join(dest, p))
    else:
      copy_file(join(source, p), join(dest, p))  
transaction copy_file(source, dest):
  f1 = open(source, "r")
  f2 = open(dest, "w")
  write(f2, read(f1))
```
Each operation is a made up of smaller transactions. If any fails, undo is called accordingly, a bit like exceptions + RAII. The main difference is that operations that succeeded are *also* undone! Either the entire transactions happens, or nothing does.

Primitive transactions would define different sub-operations:

```
foreign transaction mkdir(directory):
  on execute:
    native_mkdir(directory)
  on abort:  
    native_rmdir(directory)
```
You could also support a different variant of transaction that only commits its work when everything succeeds, might be important for printing which cannot be undone:

```
foreign transaction print(text):
  on execute:
     pass
  on commit:
    native_print(text)
```
In this case, every committed operation would be placed in a FIFO queue to be executed when the parent transaction terminates. Two operators could be added to support “partial success”: the optional operator (?) and the commit operator (!).

```
# even if this transaction fails the parent can continue
my_transaction()?
# do not undo this transaction even if the parent fails
my_transaction()!
```
Compiling transactions like this into efficient code is somewhat tricky since being able to call transactions in a loop requires keeping track of which particular transactions in the loop succeeded or not, such that they can be undone.

That will usually mean heap allocation for an undo stack (a bit like how defer works in the Go language) and/or a commit queue, but one could use a temp Arena just for that purpose, which would be reasonably efficient.

# Conclusion

Transactions are good. Ted Codd was a genius and we’ve all suffered massively for not listening to him and building software using the relational model, instead chasing silly ideas like object-oriented programming and garbage like SQL6.

Reliable software implements transactions, whether the developers realize it or not. Software architectures can be evaluated in terms of how easy or how hard they make it to implement transactions. Correctness of error handling can be rephrased in terms of implementing transactions correctly.

We might not need an actual “Transaction-Oriented Programming” paradigm, just as most of the benefits of functional programming can be achieved in a procedural language with local mutation and an avoidance of side-effects, but thinking in the “native language” of a paradigm provides clarity.

The final property, Durable, may or may not be relevant to the program and/or task, however. But thinking if it does or does not matter is worth considering!

Hey, if these days we’re developing algorithms by presenting an artificial neural network with a bunch of examples, why can’t I do the same with the (for now) far superior natural neural network in your brain?

Showing the help instructions is another User-Relevant Piece of Work but the same situation applies. Running the application to completion is the size of the “task”.

They help you remember to check for the error (at least “error codes” of the monadic kind will), which is certainly valuable to avoid some nasty debugging sessions, but it’s not relevant for the task. The error will just be manually propagated to the exact same place the exception would be caught at.

I’m also 100% sure I’m not the first person to think of this.
