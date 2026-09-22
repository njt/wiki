---
url: https://www.youtube.com/watch?v=osnxm52Kghw
date_fetched: 2026-09-22
---

# Computers are bad, actually | Kyle Kingsbury, Jepsen

Channel: Antithesis
Uploaded: 20260818
Duration: 35:49
URL: https://www.youtube.com/watch?v=osnxm52Kghw

## Description

Kyle Kingsbury discusses the complexities of building and testing distributed systems, drawing from his years of experience as a consultant and creator of the Jepsen testing library. 

He outlines five core strategies for effective testing: 

- Performing fault injection to simulate partial failures 
- Establishing a closed world to ensure total knowledge of inputs and outputs
- Accounting for indefinite error outcomes (success, failure, or unknown)
- Managing concurrency through strict consistency models like linearizability
- Utilizing generative or property-based testing to uncover unexpected invariants

The presentation concludes with a critique of over-reliance on AI-generated code and an argument for maintaining deep engineering intuition through rigorous design, proofs, and simulation environments.

http://antithesis.com/
https://jepsen.io

## Transcript (auto-generated captions)

Hello.
My name is Kyle Kingsbury. You might
know me as Afer online, and I've worn a
lot of hats in my career.
I used to be an intern who did systems
and networking. I
fixed people's desktop computers. I took
apart laptops, rack servers, pulled
cables, diagnosed faulty punch down
blocks, did hardware security badges. I
became a Rails developer for a little
bit. I used Cacti, RRD tool, and Nagios.
That explains the greys in this beard.
I wound up being a back-end engineer at
a series of web startups and was
responsible for the care and feeding of
databases and queues and caches. I wrote
API services. I did search.
I wrote my own monitoring system because
I needed more greys and
wound up immersing myself in a sea of
dashboards and graphs and getting paged
by angry Germans at 3:00 in the morning.
And all of this started to make me
wonder
if maybe computers weren't all they're
cracked up to be.
At the time, back around 2010, 2015,
distributed systems were actually kind
of wild west situation. We had MongoDB
where the client would send your
transaction to the server and just say,
"Yeah, it's great." without even waiting
to see what happened.
We had Riak where if you asked it to
list all the keys in the system, it
would
do that dutifully and take itself
offline because it was so expensive.
We had people inventing their own like
yellow consensus models.
And I started to think maybe I could
demonstrate this. And maybe this is the
cause of some of those weird errors and
crashes and like corrupt data I'm seeing
in our system.
So I wrote Jepsen, which is a library
for testing distributed systems or
single-node systems.
And with this library, I started writing
blog posts and giving talks about uh
different databases and queues and
caches, some single node, some multi
node. Uh and
eventually I quit my job to do it full
time. So, now I'm a consultant. People
hire me to come and test a system for a
few weeks and write about it. I've
reported on about 50 systems in the last
13 years, and in that time I found about
243 things I can talk about publicly. Uh
of those, 36 of them involved
availability or latency issues. We had
42 different crashes, panics, that sort
of the bug. Uh eight 11 cases where you
could read data from an aborted or
intermediate transaction.
35 cases of ordering violations or
places where you could get transactions
that were interleaved improperly. 12
cases of split brain where different
nodes believe different truths about the
network. Uh 63 places where you'd have
lost update or other forms of data loss,
and eight cases of garbage or corrupted
reads where you would put in the number
five and it would come back as a tuna
fish sandwich.
In short, databases were bad and they
continue to be bad. Uh but they're bad
in different ways. I I'm pleased to
report that modern distributed systems
take advantage of better consensus
algorithms. Uh often times they talk
about a fault model up front and they're
willing to uh
consider the possibility of partitions
and pauses and and clock skew and those
sorts of things. So, systems are more
robust than they were before.
However, I'm still finding bugs in
pretty much every system I test. Uh
which raises the question like what's so
hard about this in the first place?
So, I want to talk today about why I
think it is that building systems is
hard and more specifically why testing
distributed systems is difficult. And
maybe give some tactics that we can use
to do it better.
The first of those tactics is fault
injection. And this needs no
introduction. We're all familiar with
Chaos Monkey, but I I think it bears
repeating that distributed systems are
characterized by recurrent partial
failure. The failures happen all the
time because there's lots of nodes.
There's many chances to roll the die
each day and have a terrifying fire. And
the failures are partial because there's
lots of nodes. And so, when one falls,
the others continue. That doesn't mean
they continue towards a good goal.
Sometimes they go off and do terrible
things.
What kinds of failures do I mean? Well,
of course, you could have a crash, a
segfault, a panic, a node who physically
catches on fire.
When this happens, sometimes the machine
splits like a piñata and leaves some
data on the floor. Doesn't always do
that in a way you can pick up.
Sometimes it crashes and another node
takes over. And when it takes over, it
might make incorrect assumptions or be
missing some critical information.
Another cause of data corruption or
safety issues can be a pause.
And this is a little strange cuz pauses
happen all the time in our programs.
Most of us don't write real-time
systems. We use asynchronous runtimes
and operating systems that can pause.
When you flush to EBS, it takes 20
millis some of the time, but
occasionally it takes 200 seconds. And
in that moment, your program might be
perceived as dead by some of its
neighbors. Maybe they go on to take some
action thinking that you're dead and
you're never going to come back, but
you're just fine. You're just busy
waiting for the disk. And then you come
back and suddenly people have acted in
your absence.
It doesn't go well.
Another place where pauses can cause
safety issues is if there's some unsafe
conditions in the program normally. Like
maybe there's a bug which allows
transactions to interleave improperly.
But because the windows of concurrency
are short, you mostly never see it.
But when one of them pauses, that window
becomes large and you have a chance to
see the two transactions interleaving
improperly.
Another big failure mode to consider is
clock skew. This is, of course, normal
in all systems, but when it becomes too
large, whatever that means for you,
maybe milliseconds, maybe years, you can
end up with interesting problems if your
system relies on those clocks for
safety.
Two of the big cases where this happens
are using leases to make sure that
you're still leader. You can end up with
stale reads because your machine thinks
it's still a leader on account of the
clock not being accurate.
You can also with systems that uh
determine the order of rights based on
the clocks, like Cassandra without like
transactions, you can end up with a node
in the future writing something which
cannot be overwritten because its rights
always happen later.
So, if a node has a cosmic ray hit it
and its clock flips to the year 2040,
uh
is that possible? 2035?
Um
then if it deletes a record, nobody else
can ever write to it again because for
the next, you know, 15 or 20 years, all
of your rights happen in the logical
past of that future deletion tombstone.
Cassandra calls this a tombstone record.
Network partitions are of course a
problem. Uh you can have nodes that uh
drop, delay, duplicate, or reorder
packets. That can happen in any
asynchronous network. And in practice,
you can see delays of up to 5 minutes in
some networks and interruptions anywhere
from milliseconds to hours, sometimes
days depending on the backhoes.
When this happens, nodes can come to
different conclusions.
And then there's disk corruption. Uh
disks, as any hardware engineer will
tell you, are a portal to the
underworld. They can misdirect your
rights and put them somewhere you didn't
ask for. They can misdirect a read and
give you data from a part of the disk
you didn't ask about. If you're on a VM,
maybe they give you data from other VMs
that you don't even own.
Sometimes that happens with memory
subsystems, too.
A lot of databases aren't built to
handle corrupt data, or they aren't
built to do the dance of fsyncing quite
right in all cases. And so, this can
lead to corruption and loss as well.
In general, these faults are where most
of the interesting stuff happens in
distributed systems. And so, it's
critically important that when we test a
system, we inject those faults during
the test.
Second big idea is of a closed world.
Let's say I were asked to make sure that
a system is safe in production.
Well, I might start with some easy
metrics like I want to see 200 requests
per second succeeding a good put. And
I'd like to see reasonable latencies.
Let's say less than 50 milliseconds P95
and less than 1 second P99.
And maybe I want only like a 5% error
rate tops. These are the kind of the
traditional observability metrics that
we are used to make sure our service is
at least alive and doing things.
Uh but then a client comes to you and
says, "Hey, um somehow my bank balance
is negative." And you're like, "Ooh,
that should never happen.
Uh let me see." And so, you put an
assertion in the application code that
when it loads the data from the
database, it checks to see if it has a
negative balance. And if it does, it
screams at you and it tells the um the
ops people.
And then you can at least detect this
safety violation in production.
And then someone says, "Hey, when I log
in, sometimes I see a different set of
projects associated with my account."
And you go, "Gosh, how's that
happening?" And you discover that they
have two separate accounts with the same
email address. And this would have never
happened when you had a single database
with a single unique index, but now the
database is sharded and so those unique
indexes aren't globally unique.
And so, you write this program that
scans over the whole database every
night. It looks for those duplicates
across the shards.
And you start to think to yourself like,
"Huh, maybe there's a spectrum here of
observability. Some of the things I look
at in production are liveness and
availability performance metrics, and
some of them are like correctness and
safety properties. And maybe I can
verify all kinds of things in the live
system."
And someone says, "Great, can you tell
me if transactions can atomically update
in multiple records in a transaction?"
And you go, "Huh."
Well, I happen to know that the writers
of this database always update two keys,
X and Y, in the same transaction. And
they do the same kind of thing in both.
Like,
increment X by three, increment Y by
three.
Decrement X by one, decrement Y by one.
And if they did that, then I could
periodically read X and Y in a
transaction. And if I saw the same
number in both,
I would know that the system was
correct.
But if I ever read a value like X3 Y2, I
would have detected an error, right?
Like, maybe I had a read skew where I
saw part of a transaction, but not the
other part. Or maybe only part of a
write was committed.
But, a month into this, I start getting
all these errors, and I realize that I
forgot about that one service in the
corner that periodically writes just X,
not Y.
And so, of course, the correct thing for
the database to do was to return X3 Y2.
My assumptions were wrong, even though
the test is telling me there's an error.
And so, this tells me that correctness
of this property hinges on seeing all of
the rights in the system. I can't verify
it if I don't know everything.
And it turns out a lot of properties
work this way.
If I want to know if a counter is equal
to the number of increments, I need to
know all of the increments that
happened.
If I want to make sure that every
shipping record comes from a
corresponding purchase, I need to know
what all of the purchases are. And so on
and so forth.
So, many tests require total knowledge
of all the inputs and outputs of a
system.
Which tells us that we should establish
some sort of closed universe in which we
know all the stuff that happens to test
safely.
There's two big ways you can do this.
One is to run your test in an isolated
version of the system. Like, I I set up
a fresh cluster in most Jepsen tests. I
know I'm running on clean state. Nothing
leaks from run to run. And this also
frees me to do terrible things to
database. I can hammer it with
transactions that would uh knock a
regular database offline, or at least
make other people really unhappy with
me. Doesn't matter. It's my database,
not yours.
But sometimes you're testing a system
you don't control or one whose
dependencies would be complicated or
impossible to set up. It runs on custom
hardware. You don't have a sun system in
your back. So, instead, you can run your
test in isolated subset of the records,
like having your own key space or table
or database. And then, as long as the
database guarantees that's isolated,
then you can use that closed world
hypothesis again.
The third thing I want to talk about
involves indefinite errors.
Let's say I want to make sure that that
counter example is correct, that a
counter's value is equal to the number
of increments performed.
So, great. I'm going to have a test.
It's going to submit an increment
operation. It all gets back okay. I
submit a read operation. I get back the
number one. Who thinks this is correct?
What is Are we software engineers? Where
Am I in the right room?
What would you expect to see if not one?
One, right? One increment? Okay, cool.
Thank you.
Sorry?
Yeah, yeah, yeah. We're This is a closed
world. We've established that part.
Thank you. But yes, good call.
Okay, now what would happen if the
increment were lost or if the increment
times out? I get some error I didn't
anticipate. What should I read now?
Zero or one. Yeah, because maybe the
increment is lost before it reaches the
system. Or maybe the increment arrives
the system, it's executed, and I don't
get the acknowledgement back because it
crashed or the wrong node was there.
So, I could read either zero or one in
this scenario, right?
Is there another option?
Three?
You don't get anything back?
Yeah, the read could also return
nothing, in which case, trivially we
have to accept an error condition, yeah.
Another possibility is that this is a a
network that is partitioned or delayed,
and so that increment is in flight
and it doesn't take place now. I see
zero. But in two or three seconds it
jumps from zero to one.
Or maybe five minutes from now it jumps
from seven to eight.
And this points to something really
uncomfortable about the nature of
distributed systems, that any indefinite
operation, any timeout, any unknown
error is logically concurrent with
everything else the system does for the
rest of the system's life.
And that means that every test, every
system we ever build that does a
distributed call of any kind has to
track three distinct outcomes.
You want to know if it was definitely
okay,
if it definitely failed, or if it was
indefinite, if it maybe happened, maybe
will happen later, maybe never happens.
And if you don't categorize these things
rigorously, then your tests aren't going
to be correct.
And it turns out that a lot of databases
and a lot of clients don't make this
easy to figure out. There's a lot of
times you look at the error code and
you're like, "No leader available." Is
that definite or indefinite? Depends on
the system. People often have
unconventional expectations about that.
So, your checkers need to reason about
all three of these outcomes.
You want to say not just that a counter
is equal to the number of increments,
but that its value should be at least as
much as the acknowledged increments and
at most as much as the attempted
increments.
And that captures the range of possible
outcomes from indeterminacy.
Part four, concurrency.
Okay, so I'm reading a requirements
document and I get back that frozen
accounts can't make transfers.
Like, okay, sounds reasonable on the
surface. How do I test it?
Well, I have some different clients A
and B and they submit freeze and
transfer operations to a single account,
and I get the following history.
Client A tries to freeze the account.
Client B tries to transfer from the
account. Client A receives
acknowledgement that the freeze
completed, and client B receives
acknowledgement that the transfer
completed. Both of them happened okay.
Who thinks this is all right?
&gt;&gt; [clears throat]
&gt;&gt; Nobody. Who thinks this is bad?
A lot of you think this is bad. Okay.
And there is a world in which this is
bad, right? The freeze happens first,
then the transfer, and we violated that
state machine constraint that frozen
accounts can't make transfers. So, we
found a bug.
And I'm not so sure, right? Because
maybe the transfer took place just after
the transfer message arrived, but the
freeze, because of a queue or a delayed
network message or a something, the
freeze took place a little bit later,
just before the freeze acknowledgement
went back to the client.
And in this timeline, the transfer took
place before the freeze, and the
invariant was preserved.
So, this history is what we call
linearizable.
It obeyed the rules of the state machine
along some order of events, such that
the events appear to execute between
invocation and completion.
So, the transfer looks like it happened
after the transfer was initiated and
before the transfer was acknowledged.
And likewise, the freeze appears to
happen after the freeze begins and
before the freeze [snorts] was
acknowledged.
This is legal
under the particular consistency model
linearizability.
But if we change that concurrency
structure a little bit, same outcome,
both succeed, but now the freeze
completes okay before the transfer
begins, well, then the latest possible
time that the freeze could execute is
right before the freeze is acknowledged.
And the earliest time the transfer could
execute is right after the transfer
begins. There's no universe in which
they
uh in which the freeze happens after the
transfer. And therefore, this history is
not linearizable, and we have found a
bug in the system.
So, because these two differ only in
timing information, that tells me
something important about designing a
test. When I get a requirement like
accounts, frozen accounts can't make
transfers,
I need to ask under what consistency
model. And a lot of times that's not
formally articulated in the design
document.
So, a big part of testing systems is
trying to tease out like what do you
really mean? Are they sequential,
serializable, linearizable? Try to
figure out those semantics. And then
when you design the test, you have to
record the necessary timing or ordering
information in addition to just what
happened.
Because some models will allow certain
types of orders and others won't.
If I want to verify a property like
strict serializability or
linearizability, I need to know the
actual real-time sequence, or at least
greater bounds on that sequence, uh for
the invocation and completion times. If
you want to verify a session property
like strong session snapshot isolation,
I need to know what each session did in
order, but not necessarily the orders
between sessions.
Finally, any checker I design has to be
built to think about those multiple
execution orders, trying different
sequences, and seeing if there's some
path along which the execution is
correct.
&gt;&gt; Which brings us to part five,
generative testing.
So,
let's say I'm asked to verify the
correctness of a simple list, and I can
append things to it and I can read from
it. Let's say the list acts as
linearizable again.
So, I start off by appending the number
one to X, and then I read X. I expect to
see the list one.
So far, so good.
But, I have indefinite errors and
concurrency because this is a
distributed systems test. And so, I need
to think about multiple outcomes from
that append.
If the append were to return okay,
then I'd know that the re-observed value
should be the list one.
But if it failed, I should see the empty
list, right?
And if it's indefinite,
both.
Either would be okay.
What if I did a second append?
Well, let's call A1 the result of
appending one to X, and A2 the result of
appending two to X, when he does it in
sequence in a single thread on a single
machine. And then I'll read X.
If both succeed, I'll get one {comma}
two.
If both fail, I'll see nothing. And if
one fails, I'll see just one or just
two.
If the append of one fails, the append
of two is unknown, I could see either
the empty list or just two.
And if the append of one is unknown, and
the append of two fails, I could see
either the empty list or just one.
These are symmetric.
If append of one succeeds, the append of
two is unknown, I could have either the
list two or one two, depending on
whether or not the append of one took
place.
And by symmetry, I might expect that if
the append of one is unknown, the append
of two is okay, I would see two or one
two.
&gt;&gt; Or if it's unknown, you don't you don't
see anything.
&gt;&gt; I could.
&gt;&gt; Well, the contents of the list is going
to be all the permutations of both
appends.
&gt;&gt; Uh under linearizability, not if they
both succeed. But in this case, yes.
Because the append of one could have
been indefinite. It might have been
concurrent with the append of two, but
arrived before the read. So, I could
read one two, two one, or two alone.
There's an asymmetry here, which is
induced by that failure in conjunction
with the ordered execution on the local
thread.
And this is really bizarre. I want to
emphasize, we took a single-threaded
program,
and somehow we got concurrent execution
out of it.
And that should tell you something
really uncomfortable about building
distributed systems and testing them.
Of course, if you have uh
both of them unknown, you could have the
empty list, just one, just two, one,
two, two, one. Everything's on the
table. So, from just two appends and one
read, I got this two-dimensional 3 by 3
matrix of outcomes with anywhere from
one to five legal states for the system
to be in.
And if I were to add a third append,
it'd be 27 different, you know, worlds,
right? Down this path lies madness. You
don't want to enumerate every possible
thing that could happen. It's not going
to work.
So, what do we do instead?
Well, let's back up. Let's say that I
did a bunch of unique appends, like one
and two, and then I did some read. Is
there anything I could say about the
read without knowing specifically what
it's going to be?
Well, I want to make sure that I never
lose data that's committed. So, maybe
I'll say that every successfully
appended entry should be present in the
read.
And conversely, I don't want to see
failed
field appends.
And because I chose unique elements and
append and read shouldn't change the
state or duplicate elements, I should
see no duplicates in the read values.
And I don't want to see garbage, so I
never want to see something that I
didn't try to append. Again, that closed
world hypothesis helped me out.
So, these are easy to write properties
that nonetheless give me very strong
correctness checks on the system without
saying precisely what the outcome's
going to be.
And the narrower I make these classes,
the more precise I get in my test. I
could capture some of that timing
invariant saying that if an append of
element A completes successfully, then
for any element B I try to add later,
that B has to appear, if ever, after A
does in any read.
That's not 100% of linearizability, but
it's part of it, and it's part that you
could probably build a little state
machine to enforce.
I could go more general and check
serializability. Say that there has to
exist some total order of operations,
which if you did the stuff in the
history in that order, you'd get exactly
the same results.
That sounds simple, but it can be kind
of weird. Like, if I had a history where
I appended one to X, and then I read X
and it went to one,
and then I appended two to X,
this would be nonsense if you saw this
in a single node system or in a single
thread.
Is it serializable?
It is.
I start off with the empty list. I
append two. I append one. I read the
list two one. There is some order in
which these make sense.
Satisfies serializability via time
travel, but it does.
&gt;&gt; [laughter]
&gt;&gt; And this suggests the way to check
serializability might be to write down
all the possible orders of events in the
system and just try them with little
state machine and see if any of them
pass and give you the same results.
So, these are properties that have to
always hold in the system, regardless
the specific execution I do. And that
means I don't have to just write write
one write two. I could write one two
three. I could write the first 7 million
integers. I could write -7 cat foo bar.
Uh and this frees me to generate lots of
possible inputs, which maybe increases
the state space I cover
and maybe lets me do tests that would
change as I get to very large inputs and
still have measures of correctness. Some
systems only do interesting things in
their like code path branching once you
get to a big enough buffer or big enough
input.
Now, you can do all kinds of fancy
things on top of this, but
the core is you generate randomized
inputs to the system, you apply them to
the system using some network client,
you record the results categorizing them
as okay, indefinite, or failure.
You record the concurrency structure and
you produce some history of events. And
then you write some function that looks
at that history and tells you, was it
valid or not? Did I find an invariant
violation?
And this is a powerful approach. This
property-based or generative testing
allows you to find stuff that you never
would have thought to guess on your own.
For example, in Postgres, there's a
serializable isolation level. And in
that level,
uh there are inside of Postgres, there's
a test suite that like writes down a
whole bunch of transactions you're
supposed to do in order. And they've
written example-based tests that showed
these transactions are correctly
isolated even when you run them
concurrently.
But when I hit it with Jepsen, it almost
immediately found a violation of
serializability.
And the reason is that there's this trio
of transactions over two keys doing just
a few reads and writes in each one,
which caused the uh causality tracking
mechanism in the concurrency control
system to just misplace some
information. It it it like doesn't
correctly isolate the views. And nobody
thought to write down that particular
tree of transactions. I didn't think to
write it down because I'm lazy and not
that smart. But by having the computer
make up a bunch of random examples, I
was able to get that case. The Postgres
team could fix the bug and now we all
enjoy a safer database.
To conclude, I want to argue that now
more than ever is the time for us to
care about testing systems well. And it
has to do with the way that some people
and companies are choosing to produce
software today. It has to do with the
architecture of some modern systems. I
think you know where this is going.
At the heart of every software system,
at the base of the entire architecture,
are people. Uh the QA engineers who
verify the system, the operation staff
who are experts in in kernels and
networking, uh the people who wrote the
code in the first place, the product
managers who help specify what the
system does, the researchers and subject
matter experts and
and all those people
well, some of them don't work at your
company, do they?
Instead, they work at an intermediary, a
subcontractor like Merkle, where they do
sort of Uber for thinking, right? You
get like a few dollars an hour in this
irregular like gig work system where you
you do small programming tasks or other
kinds of knowledge work
uh and you produce a bunch of outputs
which are fed into some bigger system
you never really see.
Uh those
those particles of work, those little
uh like gravel chunks are aggregated
together by companies like OpenAI and
Anthropic into large statistical
corpuses and used to build reinforcement
learning systems, atop which the throne
of ChatGPT Codex or Anthropic's Claude
rests.
And if you are a software engineer who
uses one of these systems, you may find
yourself here
in the shadow of the Shoggoth
looking at the $50,000 a month your
company is spending on your personal
Claude token use and wondering how long
this game of musical chairs can possibly
go on. You may be looking at the people
uh over at Merkle and uh wondering if
the promised efficiencies of AI
materialize, if you will be next in the
round of layoffs that seem to be so
popular this year. You may be wondering
if it's time to start a union or perhaps
if there's space for you under the new
bigger lens that OpenAI is a quadrillion
dollars to build next year.
But for the time being, you're in the
shadow of the Shoggoth, holding your
beautiful software object up and asking
it to make some improvements. Find a
bug, add a feature, give me
internationalization. I would like uh a
proof system embedded in my My Little
Pony Speak &amp; Spell. And the Shoggoth
considers your request, all of your
tokens which say you are an expert and
make no mistakes, and the poll cats fire
up and there is a whole musical theater
number. Linear algebra happens, tokens
fly every which way, and after, you
know, 6 hours and who knows how many
thousands of dollars,
a new
bigger software artifact emerges.
And And you look at this beautiful code
base. I mean, look at this PR that
Claude made me. It's fantastic. Uh
Oh my gosh, is there now a dependency on
clocks in there?
And is that a Ferris wheel? Is that
important?
And is that fish tank load bearing?
These are questions that you might ask
if you were to see a pull request from
your colleagues, but
you might not have time to investigate
them too deeply because this isn't just
once every 6 months, this is three times
a day.
And the point of AI as management
understands it is to go fast. That's why
they're willing to throw $50,000 a month
at a single person's pocket
expenditures.
And so, you don't have the time to go
and look at all of this in detail. You
simply ship.
You give it a cursory once over, maybe
you ask another large language model to
review it, and then the product goes to
production.
What do we lose when we ship software
without looking at it? What do we lose
when we ship software without building
it directly?
Well, of course, we lose work. We lose
time. We didn't have to do all of the
labor involved, and that's the huge
benefit of the system, right?
But code is not the only outcome of the
act of programming.
Another important outcome of doing the
work of sitting there bashing your head
against the API and trying to puzzle
through the right types and the right
invariants and the, you know, ways in
which you're going to transform things,
part of the outcome of that is that you
develop a deep intuition and knowledge
for the context of the problem. You
understand the domain and the way it's
modeled. You you
where the abstractions are and where
they don't represent reality correctly?
And you gain intuition for the ways in
which other people on your team are
solving the problem.
And this is work that just happens
intuitively by by writing code. We do it
automatically.
This is straight from Bainbridge's 1983
paper Ironies of Automation.
When we don't do that work, we lose the
context of the system itself.
The other thing we lose is what Detienne
and Vernant might have called metis,
which is the cunning, the uh skilled
craftsperson wisdom, the polymorphic
adaptable intuition which allows us to
look at a complex problem like sailing a
boat or planting a different landrace
for each part of my field, and develop a
solution which balances lots of complex
uh requirements. And as engineers,
that's what we're asked to do, right?
We're given underspecified, constantly
shifting, complex requirements, and
we're asked to build some software which
uh is roughly correct and somewhat
cost-efficient and maybe runs on time
most of the time.
And we balance those things.
But that's a skill. That's an intuition
which we develop in part by doing the
work.
You learn like, oh yeah, sometimes I
can't use the like Java util concurrent,
you know, linked transfer list here
because it has this weird memory barrier
effect that ruins concurrency on this
thing. And you got that intuition by
making that mistake one time. But if you
didn't make the mistake, you never
develop the intuition.
So we lose not only specific knowledge
about the system, but also general
wisdom.
Maybe we acquire these at a higher
level. I don't have to think about
register allocation now, that's great. I
The compiler can do that for me. Maybe
that's all of programming now is we
think about high-level domain constructs
and the LLMs do the rest. They are after
all phenomenal at building intuition for
themselves, whatever that means for the
shoggoths.
But it's always been easy for us to
write subtly incorrect software, right?
Who here has missed a bug in review?
Yeah.
So, it's easy to get systems which look
fine, but then when you push them to
production, suddenly explode in some
terrible way.
Uh and I'm finding that when I test
systems that were developed with large
language model assistance, this seems
much more common.
And I can't say if this is universal or
not, I only have a few samples, but from
speaking with my peers who are, you
know, on very heavily LM using teams,
even the ones who think that they
personally are really good at it, they
are deeply frustrated with the quality
of the software that's coming out of the
system as a whole.
So, I think that now we need to focus
more on correctness, and that doesn't
just mean testing, it means every part
of the software correctness, complete
breakfast. We need designs which allow
us to reason about and intuitively
apprehend the system.
We need proofs which let us,
uh you know, either through hand-waving
or through rigorous machine-checked
theorems,
show that the system has some
correctness properties. We can use types
and other static methods to enforce
correctness of execution before we run
it. We can use tests, both in the small
scale and in the large integration tests
like Jepsen, to verify different scopes
of the software and all the combinations
in which they run. We use simulation
environments, things like uh Flow from
the FoundationDB people. Antithesis is a
simulation environment. Erlang's pulse
scheduler allows you to get pathological
thread interleavings.
These systems let us run time faster and
reach uh race conditions we wouldn't
have seen if we just ran the software
normally.
We need to inject faults, both in
testing and in production, so that's
where chaos engineering comes from,
creating not only knowledge about what
the system will do under faults, but
also a cultural incentive to build
systems which are robust to them because
production is failing often.
And of course, we need observability to
see what's happening in prod.
Now, we've versioned almost every part
of this breakfast, and it's all
delicious. I'm happy to share it with
you.
If you are focusing on tests, I want to
return to these five core ideas. Because
distributed systems can fail and fail
often, we need to do fault injection in
our tests. Because you can only show
some properties to be correct if you
have a closed universe, we want to
establish total knowledge of the inputs
and outputs.
Because every operation in a distributed
system can be indefinite, we need to
track, is it okay? Did it fail? Or was
it unknown?
We need to track the concurrency
structure of the history and design a
test which reasons about concurrent
outcomes. And this leads us naturally to
generative or property-based testing. We
produce randomized inputs, apply them to
the system, and then verify that the
results hold.
&gt;&gt; [sighs]
&gt;&gt; These are the techniques I use in
Jepsen, but you don't have to use Jepsen
or hire me as a consultant to get these
benefits. You can do them yourself. You
can do them with a Pearl script. Uh I
think that putting even a little bit of
work into doing testing this way will
help give you significant advantages.
I'm so glad to share it.
Thank you to AWS and Tethys for having
me here today. Thank you all for your
kind attention. If you'd like to learn
more, you'll find all the reports on
jepsen.io.
&gt;&gt; Okay. So, back to I want to talk about
building software with agents.
What
