---
url: https://web.stanford.edu/class/cs345d-01/rl/aries.pdf
date_fetched: 2026-08-09
---

ARIES: A Transaction Recovery Method
Supporting Fine-Granularity Locking
and Partial Rollbacks Using
Write-Ahead Logging
C. MOHAN
IBM Almaden

Research

Center

and
DON HADERLE
IBM Santa Teresa

Laboratory

and
BRUCE

LINDSAY,

IBM Almaden

HAMID

Research

PIRAHESH

and PETER SCHWARZ

Center

In this paper we present

a simple

and

Semantics),

and efficient method, called ARIES ( Algorithm
for Recouery
which supports partial
rollbacks
of transactions,
finegranularity
(e. g., record) locking and recovery using write-ahead logging (WAL). We introduce
history
to redo all missing updates before performing
the rollbacks of
the paradigm of repeating
the loser transactions
during restart after a system failure. ARIES uses a log sequence number
in each page to correlate the state of a page with respect to logged updates of that page. All
updates of a transaction
are logged, including those performed during rollbacks. By appropriate
chaining of the log records written during rollbacks to those written during forward progress, a
bounded amount of logging is ensured during rollbacks even in the face of repeated failures
during restart or of nested rollbacks
We deal with a variety of features that are very Important
transaction processing system ARIES supports
in building and operating an industrial-strength
fuzzy checkpoints, selective and deferred restart, fuzzy image copies, media recovery, and high
concurrency lock modes (e. g., increment /decrement) which exploit the semantics of the operations and require the ability
to perform operation
logging. ARIES is flexible
with respect
to the kinds of buffer management
policies that can be implemented.
It supports objects of
varying
length efficiently.
By enabling
parallelism
during restart, page-oriented
redo, and
logical undo, it enhances concurrency and performance.
We show why some of the System R
paradigms for logging and recovery, which were based on the shadow page technique, need to be
changed in the context of WAL. We compare ARIES to the WAL-based
recovery methods of
Isolation

Exploiting

Authors’ addresses: C Mohan, Data Base Technology Institute,
IBM Almaden Research Center,
San Jose, CA 95120; D. Haderle, Data Base Technology Institute,
IBM Santa Teresa Laboratory, San Jose, CA 95150; B. Lindsay, H. Pirahesh, and P. Schwarz, IBM Almaden Research
Center, San Jose, CA 95120.
Permission to copy without fee all or part of this material is granted provided that the copies are
not made or distributed
for direct commercial advantage, the ACM copyright notice and the title
of the publication
and its date appear, and notice is given that copying is by permission of the
Association for Computing Machinery.
To copy otherwise, or to republish, requires a fee and/or
specific permission.
@ 1992 0362-5915/92/0300-0094
$1.50
ACM Transactions on Database Systems, Vol

17, No. 1, March 1992, Pages 94-162

ARIES: A Transaction Recovery Method

.

95

DB2TM, IMS, and TandemTM systems. ARIES is applicable not only to database management
systems but also to persistent
object-oriented
languages,
recoverable
file systems and
transaction-based
operating
systems. ARIES has been implemented,
to varying
degrees, in
IBM’s OS/2TM Extended Edition Database Manager, DB2, Workstation
Data Save Facility/VM,
Starburst and QuickSilver,
and in the University
of Wisconsin’s EXODUS and Gamma database
machine.
Categories
dures,

and Subject

checkpoint/

Management]:

fault

Physical

tems—concurrency,
and

D.4.5

[Operating

Systems]:

Reliability–backup

proce-

[Data]: Files– backup/ recouery; H.2.2 [Database
and restart;
H.2.4
[Database Management]: Sys-

tolerance;

E.5.

Design–reco~ery

transaction

tration—logging
General

Descriptors:

restart,

H.2.7 [Database

processing;

Management]:

Database

Adminis-

recovery

Terms: Algorithms,

Designj

Performance,

Additional
Key Words and Phrases: Buffer
write-ahead logging

Reliability

management,

latching,

locking,

space management,

1. INTRODUCTION
In

this

section,

first

we

introduce

some

ery, concurrency
control,
and buffer
organization
of the rest of the paper.
1.1

Logging,

Failures,

and Recovery

The transaction

concept,

for a long

It encapsulates

time.

which

and Durability)
properties
not limited
to the database
Guaranteeing

the

concurrent
important

execution
problem

been

developed

performance
methods
judged

have
using

in

concepts

relating

and then

to

recov-

we outline

the

Methods

is well
the

understood

ACID

by now,

(Atomicity,

has been

Consistency,

around
Isolation

[361. The application
of the transaction
concept is
area [6, 17, 22, 23, 30, 39, 40, 51, 74, 88, 90, 1011.

atomicity

and

durability

of transactions,

in

the

face

of

of multiple
transactions
and various
failures,
is a very
in transaction
processing.
While
many
methods
have
the

past

characteristics,
not always
several

basic

management,

to

deal

with

and

the

complexity

been

metrics:

acceptable.
degree

this

problem,
and

Solutions

of concurrency

the

assumptions,

ad hoc nature

of such

to this

may

supported

problem
within

be

a page

and across pages, complexity
of the resulting
logic, space overhead
on nonvolatile
storage and in memory for data and the log, overhead
in terms of the
number
of synchronous
and asynchronous
1/0s required
during restart
recovery and normal
processing,
kinds of functionality
supported
tion rollbacks,
etc.), amount
of processing
performed
during
degree of concurrent
processing
supported
during
restart
system-induced
transaction
rollbacks
caused by deadlocks,

(partial
restart

transacrecovery,

recovery,
extent of
restrictions
placed

‘M AS/400, DB2, IBM, and 0S/2 are trademarks
of the International
Business Machines Corp.
Encompass, NonStop SQL and Tandem are trademarks
of Tandem Computers, Inc. DEC, VAX
DBMS, VAX and Rdb/VMS are trademarks of Digital Equipment
Corp. Informix is a registered
trademark of Informix Software, Inc.

ACM Transactions on Database Systems, Vol. 17, No 1, March 1992.

96

C. Mohan et al

.

on stored data (e. g., requiring
unique
keys for all records,
mum size of objects to the page size, etc.), ability
to support
which

allow

the

concurrent

execution,

based

restricting
maxinovel lock modes

on commutativity

and

other

properties
[2, 26, 38, 45, 88, 891, of operations
like increment/decrement
on
the same data by different
transactions,
and so on.
In this
paper
we introduce
a new recovery
method,
called
ARL?LSl
(Algorithm
very well
flexibility

for Recovery
and Isolation
Exploiting
Semantics),
which
fares
with respect to all these metrics.
It also provides
a great deal of
to take
advantage
of some special
characteristics
of a class
of applications
that
of applications
for better
performance
(e. g., the kinds
IMS Fast Path [28, 421 supports
efficiently).
To meet transaction
and data recovery
guarantees,
ARIES records in a log

the progress
able

data

of a transaction,
objects.

transaction’s
types
back).
records

and its actions

log becomes

committed

of failures,
When the
also

The

actions

the

which

source

are reflected

for

cause changes
ensuring

in the database

or that its uncommitted
actions
logged actions
reflect
data object

become

the

source

for

reconstruction

to recover-

either

that

despite

various

the

are undone
(i.e., rolled
content,
then those log
of damaged

or lost

data

(i.e., media recovery).
Conceptually,
the log can be thought
of as an ever
growing
sequential
file. In the actual implementation,
multiple
physical
files
may be used in a serial fashion
to ease the job of archiving
log records [151.
Every
record

log record is assigned a unique
log sequence number
(LSN)
is appended to the log. The LSNS are assigned in ascending

when that
sequence.

Typically,
they are the logical addresses of the corresponding
log records. At
[6’71. If more
times, version
numbers
or timestamps
are also used as LSNS
than one log is used for storing
the log records relating
to different
pieces of
data, then a form of two-phase
commit
protocol
(e. g., the current
industrystandard
Presumed
Abort protocol
[63, 641) must be used.
The nonvolatile
version
of the log is stored on what is generally
called
stable storage. Stable storage means nonvolatile
storage which remains
intact
Disk is an example
of nonvolatile
and available
across system
failures.
storage and its stability
is generally
improved
by maintaining
synchronously
two identical
copies of the log on different
devices.
We would
expect
online log records stored on direct access storage devices to be archived
cheaper and slower medium
like tape at regular
intervals.
The archived
records

may

be discarded

once the appropriate

image

copies

(archive

the
to a
log

dumps)

of the database
have been produced
and those log records
are no longer
needed for media recovery.
Whenever
log records are written,
they are placed first only in the volatile
storage
(i.e., virtual
storage)
buffers
of the log file. Only at certain
times
(e.g., at commit time) are the log records up to a certain point (LSN) written,
in log page sequence, to stable storage.
This is called
forcing
the log up to
that LSN. Besides forces caused by transaction
and buffer
manager
activi -

1 The choice of the name ARIES, besides its use as an acronym that describes certain features of
our recovery method, is also supposed to convey the relationship
of our work to the Starburst
project at IBM, since Aries is the name of a constellation.
ACM TransactIons on Database Systems, Vol. 17, No 1, March 1992

ARIES: A Transaction Recovery Method
ties, a system
buffers as they

process
fill up.

may,

For ease of exposition,

in

the

we assume

background,
that

periodically

each log record

force

describes

performed
to only a single page. This is not a requirement
in the Starburst
[87] implementation
of ARIES,
sometimes

.

97

the

log

the update

of ARIES.
In fact,
a single log record

might
be written
to describe updates to two pages. The undo (respectively,
redo) portion
of a log record provides
information
on how to undo (respectively,
redo) changes
performed
by the transaction.
A log record
which
contains

both

record.

Sometimes,

information

the

undo

or only

log record

and the

a log

the undo

or an undo-only

that

is performed,

(e.g.,
fields

before
within

subtract
3 from
high concurrency
performed

the

the update
the object)

redo

record

information

may

information.
log record,

undo-redo

Such a record

information

and after the
or operationally

For

example,

with

property

of the model

of [3], which

be locked

exclusively

(X mode)

uses the widely

of the commercial

accepted

log
redo

a redo-only
on the action

be recorded

physically

logging
permits
semantics
of the

certain

operations,

the

the use of
operations
same field

updates
of many transactions.
These
is permitted
by the strict executions

essentially

for commit

and prototype

is called

the

images
or values
of specific
add 5 to field 3 of record 15,

field 4 of record 10). Operation
lock modes, which exploit
the

on the data.

undo-redo
only

Depending

may

update
(e.g.,

an

to contain

respectively.

of a record could have uncommitted
permit
more concurrency
than what

ARIES

is called

be written

write

says that

ahead

systems

modified

objects

must

duration.
logging

(WAL)

based on WAL

protocol.

are IBM’s

Some

AS/400TM

[9, 211, CMU’S Camelot
961, Unisys’s
DMS/1100

[23, 901, IBM’s DB2TM [1, 10,11,12,13,14,15,19,
35,
[271, Tandem’s
EncompassTM
[4, 371, IBM’s IMS [42,
m [161, Honeywell’s
MRDS [911,
43, 53, 76, 80, 941, Informix’s
Informix-Turbo
[29], IBM’s
0S/2
Extended
Tandem’s
NonStop
SQL ‘M [95], MCC’S ORION
EditionTM

Database

Manager

[71, IBM’s

QuickSilver

[40],

IBM’s

Starburst

[871, SYNAPSE
[781, IBM’s
System/38
[99], and DEC’S VAX DBMSTM
and
VAX Rdb/VMSTM
[811. In WAL-based
systems, an updated
page is written
back to the same nonvolatile
storage location
from where it was read. That
is, in-place
what

updating

happens

is performed

in the shadow

on nonvolatile

page technique

which

storage.

Contrast

is used in systems

this

with

such as

System R [311 and SQL/DS
[51 and which is illustrated
in Figure
1. There the
updated
version
of the page is written
to a different
location
on nonvolatile
storage and the previous
version of the page is used for performing
database
recovery
if the system were to fail before the next checkpoint.
The WAL protocol
asserts that the
some data must already
be on stable
allowed
to replace the previous
version
That

is, the system

storage
records
storage.

is not allowed

version
of the
which describe
To enable the

method
of recovery
describes
the most

log records representing
changes
to
storage
before the changed
data is
of that data on nonvolatile
storage.

to write

an updated

page to the nonvolatile

database
until
at least the undo portions
of the log
the updates to the page have been written
to stable
enforcement
of this protocol,
systems using the WAL

store
recent

in every page the LSN
of the log record
that
update
performed
on that
page. The reader
is
ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

98

.

C Mohan et al.

Map

Page

~

Fig. 1.

Shadow page technique.
Logical page LPI IS read from physical page PI and after
modlflcat!on IS wr!tten to physical page PI’ P1’ IS the current
vers!on and PI IS the shadow version
During
a checkpoint,
the

shadow

referred

than

which

shadowing

is performed

of the

problems

of the

IS performed

us!ng

data

why

the

the

and

the

On

a failure,

and

the

log

WAL

page technique.

using

also

current

version
data

shadow

base

version

base

about

the shadow

some of the important

verson

recovety

to [31, 971 for discussions

ered to be better

IS

shadow

the

of the

d]scarded

version

becomes

technique

is consid-

[16, 781 discuss

a separate

methods

in

log. While

these

avoid

some

approach,

they

still

retain

original

shadow

page

drawbacks

and they

introduce

some new ones. Similar

comments
apply to the methods suggested in [82, 881. Later, in Section 10, we
show why some of the recovery
paradigms
of System R, which were based on
the shadow page technique,
are inappropriate
in the WAL context,
when we
need support
are described

for high levels
in Section 2.

Transaction

status

of concurrency

and various

stored

log

is also

in

the

and

other

features

no transaction

considered
complete
until its committed
status and all its log data
recorded
on stable storage by forcing
the log up to the transaction’s
log record’s

LSN.

This

allows

a restart

recovery

procedure

that
can

be

are safely
commit

to recover

any

transactions
that completed
successfully
but whose updated
pages were not
physically
written
to nonvolatile
storage before the failure
of the system.
This means that
a transaction
is not permitted
to complete
its commit
processing
(see [63, 64]) until
the redo portions
of all log records
of that
transaction
have been written
to stable storage.
We deal with three types of failures:
transaction
or process, system, and
media or device. When a transaction
or process failure
occurs, typically
the
transaction
would
be in such a state that
its updates
would
have to be
undone.

It is possible

that

the

transaction

had

corrupted

some pages

in the

buffer
pool if it was
the process disappeared.

in the
When

middle
of performing
some updates
when
the virtual
a system failure
occurs, typically

storage

contents

be lost

and

restarted

and

the

database

and

contents

of

recovered

using

the

would
recovery

that

the

performed
log.

media
an

image

the

transaction

using

the

When

a media

or device

would

be

and

copy

lost

(archive

system

nonvolatile
the

dump)

failure
lost

would

have

storage
data

version

occurs,
would
of the

to be

versions

of

typically
have
lost

data

the
to

be
and

log.

Forward
processing
refers to the updates performed
when the system is in
normal
(i. e., not restart
recovery)
processing
and the transaction
is updating
ACM TransactIons on Database Systems, Vol

17, No. 1, March 1992.

ARIES: A Transaction Recovery Method
the database

because

of the data

user or the application

program.

and using

the log to generate

to the ability
later

manipulation
That

the (undo)

to set up savepoints

in the transaction

the transaction

request

(e.g.,

update

calls.

the

execution

during

the rolling

since the establishment

back

concept

is exposed

only

with

place

if a partial

another

partial

at the application

database

level

recovery.

rollback

were

rollback

whose

A

to be later
point

Partial

issued

by the
back

rollback

refers

of a transaction

and

savepoint

performed

by

[1, 31]. This

is

all updates
of the transaction
Whether
or not the savepoint

is immaterial

nested

calls

99

is not rolling

of the changes

of a previous

to be contrasted
with
total rollback
in which
are undone and the transaction
is terminated.
deals

SQL)

is, the transaction

.

rollback
followed

to us since this

paper

is said to have

taken

by a total

of termination

rollback

is an earlier

point

or

in the

transaction
than the point of termination
of the first rollback.
Normal
undo
refers to total or partial
transaction
rollback
when the system is in normal
operation.

A normal

undo

or it may
constraint

be system
violations).

initiated
because of deadlocks
or errors (e. g., integrity
Restart
undo refers to transaction
rollback
during

restart

recovery

after

may be caused

a system

by a transaction

failure.

To make

request

partial

to rollback

or total

rollback

efficient
and also to make debugging
easier, all the log records written
by a
transaction
are linked
via the PreuLSN
field of the log records in reverse
chronological

order.

That

transaction
would
point
that transaction,
if there
the

updates

performed

is,

the

most

recently

written

log

record

of the

to the previous
most recent log record written
by
is such a log record.2 In many WAL-based
systems,
during

a rollback

are logged

using

what

are

called

compensation
log records (CLRS)
[151. Whether
a CLR’S update
is undone,
should that CLR be encountered
during
a rollback,
depends on the particular
system.

As we will

see later,

in ARIES,

a CLR’S

update

is never

undone

and

hence CLRS are viewed as redo-only
log records.
Page-oriented
redo is said to occur if the log record whose update is being
redone describes which page of the database
was originally
modified
during
normal
processing
and if the same page is modified
during
the redo processing. No internal
descriptors
of tables or indexes need to be accessed to redo
the update.

That

is to be contrasted

is, no other
with

page of the database

logical

redo

which

and AS/400
for indexes
[21, 621. In those
not logged separately
but are redone using

needs to be examined.

is required

in System

systems, since
the log records

This

R, SQL/DS

index changes are
for the data pages,

performing
a redo requires
accessing
several
descriptors
and pages of the
database.
The index
tree
would
have
to be retraversed
to determine
the page(s) to be modified
and, sometimes,
the index page(s) modified
because
of this redo operation
may be different
from the index page(s) originally
modified
during
normal
processing.
Being able to perform
page-oriented
redo
allows
the

the system

recovery

to provide

of one page’s

recovery
contents

independence
does not

require

amongst

objects.

accesses

That

to any

is,

other

2 The AS/400, Encompass and NonStop SQL do not explicitly
link all the log records written by
backward scan of the log must be
a transaction.
This makes undo inefficient
since a sequential
performed to retrieve all the desired log records of a transaction.
ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992

100

.

C. Mohan et al

(data or catalog) pages of the database.
media recovery very simple.
In a similar
Being
levels

fashion,

we can define

As we will

describe

page-oriented

undo

and

able to perform
logical
undos allows
the system
of concurrency
than what would be possible if the

restricted

only

to

page-oriented

undos.

appropriate
concurrency
control
of one transaction
to be moved
one were

restricted

to only

This

later,

this

makes

logical

undo.

to provide
higher
system were to be

is because

the

former,

with

protocols,
would permit uncommitted
updates
to a different
page by another
transaction.
If

page-oriented

undos,

then

the

latter

transaction

would have had to wait for the former
to commit.
Page-oriented
redo and
page-oriented
undo permit
faster recovery
since pages of the database
other
than the pages mentioned
in the log records are not accessed. In the interest
of efficiency,
interest
of

ARIES
supports
high
concurrency,

ARIES/IM

method

and

recovery

and show the advantages
of being able to perform
ARIES/IM
with other index methods.

logical

1.2

Latches

for

page-oriented
redo and its supports,
in
logical
undos.
In [62], we introduce

concurrency

control

in B ‘-tree

the
the

indexes

undos

by comparing

and Locks

Normally

latches

and locks

are used to control

access to shared

information.

Locking

has been

discussed

to a great

in the

Latches,

have

not

been

latches

are

the

other

hand,

semaphores.

Usually,

data,

locks

while

physical
Latches

locks.

the deadlock

Also,

are requested
alone,

in such

or involving

Acquiring

and

discussed
used

are used to assure

worry about
environment.

consistency
are usually

extent
to

that

much.

guarantee

logical

literature.

Latches

physical

consistency

since we need to support
held for a much shorter

detector

is not informed

about

so as to avoid

deadlocks

latches

and locks.

releasing

a latch

is

much

of

We need

to

a multiprocessor
period than are

latch

cheaper

on
like

consistency

of data.

a manner

are

waits.

Latches

involving

latches

than

acquiring

and

releasing
a lock. In the no-conflict
case, the overhead
amounts
to 10s of
instructions
for the former
versus 100s of instructions
for the latter.
Latches
are cheaper because the latch control information
is always in virtual
memory in a fixed place, and direct
addressability
to the latch information
is
possible given the latch name. As the protocols
presented
later in this paper
and those in [57, 621 show, each transaction
holds at most two or three
latches simultaneously.
As a result,
the latch request blocks can be permanently allocated
to each transaction
and initialized
with transaction
ID, etc.
right at the start of that transaction.
On the other hand, typically,
storage for
individual
locks has to be acquired,
formatted
and released
dynamically,
causing more instructions
to be executed to acquire and release locks. This is
advisable
because, in most systems, the number
of lockable
objects is many
orders of magnitude
greater
than the number
of latchable
objects. Typically,
all information
relating
to locks currently
held or requested
by all the
transactions
is stored in a single,
central
hash table.
Addressability
to a
particular
lock’s information
is gained
the address
of the hash anchor
and
pointers.

Usually,

ACM Transactions

in the

process

on Database Systems, Vol

by first hashing
then,
possibly,

of trying

to locate

17, No 1, March 1992

the lock
following
the

lock

name to get
a chain
of
control

block,

ARIES: A Transaction Recovery
because multiple
transactions
may be simultaneously
the contents
of the lock table,
one or more latches
released—one

latch

on the

hash

anchor

lock’s chain of holders and waiters.
Locks may be obtained
in different
IX

(Intention

exclusive),

and,

Method

reading
and modifying
will
be acquired
and

possibly,

one on the

modes such as S (Shared),

IS (Intention

Shared)

and

101

.

SIX

specific

X (exclusive),

(Shared

Intention

exclusive),
and at different
granularities
such as record (tuple),
table
tion), and file (tablespace)
[321. The S and X locks are the most common

(relaones.

S provides
the read privilege
and X provides
the read and write privileges.
Locks on a given object can be held simultaneously
by different
transactions
only if those locks’ modes are compatible.
The compatibility
relationships
amongst

the

above

modes

of locking

are shown

in Figure

2. A check

mark

(’<) indicates
that the corresponding
modes are compatible.
With
hierarchical locking,
the intention
locks (IX, IS, and SIX) are generally
obtained
on
the higher
levels of the hierarchy
(e.g., table),
and the S and X locks are
obtained
and X),

on the lower levels (e. g., record).
The nonintention
mode locks (S
when obtained
on an object at a certain
level of the hierarchy,

implicitly
grant locks of the corresponding
mode on the lower level objects of
that higher
level object. The intention
mode locks, on the other hand, only
give the privilege
of requesting
the corresponding
mode locks on the lower level objects. For example,
grants

S on all

the

records

explicitly
on the records.
defined in the literature

of that

table,

and

it

Additional,
semantically
[2, 38, 45, 551 and ARIES

intention
or nonintention
SIX on a table implicitly
allows

X to be requested

rich lock modes have been
can accommodate
them.

Lock requests
may be made with
the conditional
or the unconditional
option. A conditional
request means that the requestor
is not willing
to wait
if, when the request
is processed, the lock is not grantable
immediately.
An
unconditional
lock becomes
unconditional

request
means that the requestor
is willing
to wait until
the
grantable.
Locks
may be held for different
durations.
An
request for an instant
duration
lock means that the lock is not

to be actually
granted,
call with
the success
duration

locks

but the lock manager
has to delay returning
status
until
the lock becomes
grantable.

are released

some time

after

they

are acquired

the lock
Manual

and, typically,

long before transaction
when the transaction

termination.
terminates,

Commit
duration
locks are released only
i.e., after commit
or rollback
is completed.

The above

discussions

concerning

conditional

durations,

except

for commit

Fine-Granularity

Locking

1.3

Fine-granularity
database systems

duration,

apply

requests,

different

to latches

also.

modes,

and

(e.g., record) locking
has been supported
by nonrelational
(e.g., IMS [53, 76, 801) for a long time. Surprisingly,
only

few of the commercially
locking,
even though

a

available
relational
systems provide fine-granularity
IBM’s
System R [321, S/38 [991 and SQL/DS
[51, and

Tandem’s
Encompass
[37]
supported
record
and/or
key
the beginning.
3 Although
many interesting
problems
relating

locking
from
to providing

3 Encompass and S/38 had only X locks for records and no locks were acquired
these systems for reads.

automatically

ACM Transactions

by

on Database SyStanS, Vol. 17, No 1, March 1992

102

C. Mohan

.

Fig. 2.
matrix

Lock

et al.

m

mode comparability

lx

+

Slx

4

.’

fine-granularity
locking
in the context
of WAL
remain
to be solved, the
research community
has not been paying enough attention
to this area [3, 75,
88]. Some of the System R solutions
worked
only because of the use of the
shadow page recovery
technique
in combination
with
10). Supporting
fine-granularity
locking
and variable
flexible
fashion
requires
addressing
some interesting

locking
length
storage

issues

database

which

have

never

really

been

discussed

in

the

(see Section
records
in a
management
literature.

Unfortunately,
some of the interesting
techniques
that were developed
for
System R and which are now part of SQL/DS
did not get documented
in the
literature.
here

the

expense

some of those

At

problems

As supporting

high

of making

this

and their

concurrency

tion of an application
requiring
systems
gain in popularity,

paper

long,

we will

be discussing

solutions.

gains

very high
it becomes

importance

(see [79] for the descrip-

concurrency)
necessary

to

and as object-oriented
invent
concurrency

control
and recovery
methods
that take advantage
of the semantics
of the
operations
on the data [2, 26, 38, 88, 891, and that support
fine-granularity
locking
efficiently.
Object-oriented
systems may tend to encourage
users to
define

a large

to be the
view

of the

number

appropriate
database,

the container
locking
during
users

may

tend

of small

objects

and users

granularity

of locking.

In

the

of a page,

with

concept

may
the

many

terminal

object

instances

object-oriented

logical

its physical

of objects, becomes unnatural
to think
object accesses and modifications.
Also,
to have

expect

interactions

orientation

about as the
object-oriented

as

unit
of
system

during

the

course

transaction,
thereby
increasing
the lock hold times.
If the
were to be a page, lock wait times and deadlock
possibilities

unit
will

of locking
be aggra-

vated.
Other discussions
concerning
transaction
management
oriented
environment
can be found in [22, 29].
As more and more customers
adopt
relational
systems

in

an object-

applications,
it becomes ever more important
77, 79, 83] and storage management
without
the system
users or administrators.
Since

to handle
requiring
relational

for

of a

production

hot-spots [28, 34, 68,
too much tuning
by
systems
have been

welcomed
to a great extent because of their ease of use, it is important
that
we pay greater
attention
to this area than what has been done in the context
of the nonrelational
systems.
Apart
from the need for high concurrency
for
user data, the ease with
which
online
data definition
operations
can be
performed
in relational
systems by even ordinary
users requires
the support
for high concurrency
of access to, at least, the catalog data. Since a leaf page
in an index typically
describes
data in hundreds
of data pages, page-level
locking
of index data is just not acceptable.
A flexible
recovery
method that
ACM TransactIons on Database Systems, Vol

17, No. 1, March 1992.

ARIES: A Transaction Recovery Method
allows the
needed.
The

support

above

facts

of high

levels

argue

supporting

for

such as increment/decrement
rently
modify
even the same
increment

and decrement

of concurrency

during

semantically

rich

.

103

index

accesses

is

modes

of locking

which
allow multiple
transactions
to concurpiece of data. In funds-transfer
applications,

operations

are frequently

performed

on the branch

and teller balances by numerous
transactions.
If those transactions
to use only X locks, then they will be serialized,
even though their

are forced
operations

commute.
1.4

Buffer

The

buffer

manages

Management
manager

(BM)

is the

buffer

pool

and

storage

version

the

nonvolatile

component

does

1/0s

of the
to

of the database.

transaction

read/write

The

fix

pages

primitive

system

that

from/to

the

of the BM may

be used to request
the buffer
address of a logical
page in the database.
If
the requested
page is not in the buffer pool, BM allocates
a buffer
slot and
reads
when

the p~ge in. There may be instances
(e. g., during
a B ‘-tree page split,
the new page is allocated)
where the current
contents
of a page on

nonvolatile

storage

are not of interest.

In such a case, the

fix– new primitive

may be used to make the BM allocate
a ji-ee slot and return
the address of
that slot, if BM does not find the page in the buffer pool. The fix-new
invoker
will then format
the page as desired.
Once a page is fixed in the buffer pool,
the corresponding
buffer slot is not available
for page replacement
until
the
unfix primitive
is issued by the data manipulative
component.
Actually,
for
each page, BM keeps a fix count which is incremented
by one during
every
fix operation
and which is decremented
by one during
every unfix operation.
A page in the buffer pool is said to be dirty if the buffer version of the page
has

some

updates

which

are

not

yet

reflected

in

the

nonvolatile

storage

version of the same page. The fix primitive
is also used to communicate
the
intention
to modify the page. Dirty pages can be written
back to nonvolatile
the

modification

read accesses to the page while

storage

it is being

of BM

when

no fix

in writing

with

in the

background,

to reduce

intention
written

is held,

out.

on a continuous

nonvolatile

storage

the amount

of redo work

if a system

failure

were

to occur

buffer

pool

pages

in the

nondirty

state

so that

other

pages

without

synchronous

write

1/0s

having

allowing
the role

basis,

pages

that

and also to keep a certain
they

thus

[96] discusses
dirty

would

percentage

may

to

be needed

be replaced

to be performed

of the
with
at the

time of replacement.
While
performing
those writes,
BM ensures that the
WAL protocol
is obeyed. As a consequence,
BM may have to force the log up
to the LSN of the dirty page before writing
the page to nonvolatile
storage.
Given the large
of this nature
transactions

buffer pools that
to be very rare

committing

are common today, we would expect a force
and most log forces to occur because
of

or entering

the prepare

state.

BM also implements
the support
for latching
pages. To provide
direct
addressability
to page latches and to reduce the storage associated with those
latches, the latch on a logical page is actually
the latch on the corresponding
buffer

slot. This

means

that

a logical

page can be latched

only

after

it is fixed

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

104

C. Mohan et al

.

in the buffer

pool and the latch

These

are

stored

in the buffer

BCB
dirty

highly

has to be released

acceptable

conditions.

control

(BCB)

block

also contains
the identity
status of the page, etc.

The

before

latch

the page is unfixed.

control

information

for the corresponding

of the logical

page,

what

buffer

the fix

is

slot.

count

The

is, the

Buffer
management
policies
differ
among the many systems in existence
WAL-Based
Methods”).
If a page modified
by a
(see Section
11, “Other
transaction
is allowed to be written
to the permanent
database on nonvolatile
storage before that transaction
commits,
then the steal policy is said to be
followed
no-steal

Otherwise,
by the buffer manager
(see [361 for such terminologies).
policy is said to be in effect. Steal implies
that during
normal

restart

rollback,

some

volatile

storage

version

commit until
the database,

undo

work

of the

might

have

database.

to be performed

If a transaction

policy, during
transactions.

said to occur

database

if, even in the virtual

not performed
database
calls.

storage

in-place
when
the transaction
issues
The updates
are kept in a pending
list

performed

in-place,

mined that
to be rolled

the transaction
is definitely
back, then the pending
list
policy

using

has

the pending

implications

list

information,

on whether

Organization

The

rest

of the

paper

is organized

no

the updates

are

after

a transaction

as follows.

is

it is deter-

If the transaction
needs
or ignored.
The deferred

own updates or not, and on whether
partial
rollbacks
For more discussions
concerning
buffer management,
1.5

recovery,
updating

the corresponding
elsewhere
and are

only

committing.
is discarded

to

version of
a no-force

restart
Deferred

buffers,

non-

allowed

all pages modified
by it are written
to the permanent
then a force policy is said to be in effect. Otherwise,

policy is said to be in effect. With a force
redo work will be necessary for committed

updating

on the

is not

a
or

can

“see”

its

are possible or not.
see [8, 15, 24, 961.

After

stating

our

goals

in

Section
2 and giving
an overview
of the new recovery
method
ARIES
in Section 3, we present,
in Section 4, the important
data structures
used by
ARIES during normal
and restart
recovery processing.
Next, in Section 5, the
protocols followed
during normal processing
are presented
followed,
in Section
6, by the description
of the processing
performed
during
latter
section also presents
ways to exploit
parallelism
methods

for

performing

recovery

selectively

restart
during

or postponing

recovery.
recovery
the

The
and

recovery

of

some of the data.
checkpoints
during
impact
of failures

Then, in Section
7, algorithms
are described
for taking
the different
log passes of restart
recovery
to reduce the
during
recovery.
This is followed,
in Section
8, by the

description
of how
Section 9 introduces

fuzzy image copying
and media
the significant
notion of nested

implementing

them

tiques

a method

some

for

of

the

existing

recovery

context

of the

shadow

page

caused

by using

detail

the

characteristics

different

systems

in

ACM Transactions

those

efficiently.

technique

paradigms
as

System

in the

WAL

of the

WAL-based

of many
such

Section

paradigms
and

IMS,

on Database Systems, Vol

DB2,

recovery
are supported.
top actions and presents
10

which
R. We

context.
Encompass

17, No. 1, March 1992

describes

and

cri-

originated

in

the

discuss

Section

the

problems

11 describes

in

recovery

methods

in use

and

NonStop

SQL.

ARIES: A Transaction Recovery Method

.

105

Section 12 outlines
the many different
properties
of ARIES.
We conclude by
summarizing,
in Section 13, the features
of ARIES which provide
flexibility
and efficiency,
and by describing
the extensions
and the current
status of the
implementations
of ARIES.
Besides presenting
a new recovery
method,
by way of motivation
for our
work,

we also

describe

some

previously

unpublished

aspects

of recovery

in

System R. For comparison
purposes,
we also do a survey
of the recovery
methods used by other WAL-based
systems and collect information
appearing
in several

publications,

aims in
resulting

many

of which

are not widely

this paper
is to show the intricate
from the different
choices made for

available.

One of our

and unobvious
interactions
the recovery
technique,
the

granularity
of locking
and the storage
management
scheme.
One cannot
make arbitrarily
independent
choices for these and still expect the combination to function
together
correctly
and efficiently.
This point
needs to be
emphasized
books
cover,

as it

is not

always

dealt

with

adequately

on concurrency
control
and recovery.
as much as possible, all the interesting

one encounters
processing

in building

and operating

in

most

papers

and

In this paper, we have tried to
recovery-related
problems
that
an

industrial-strength

transaction

system.

2. GOALS
This

section

lists

the goals

of our work

in designing
a recovery
method
The goals relate to the metrics
discussed

earlier,

Simplicity.
and program
algorithms

in Section

Concurrency
for, compared
are

bound

strived

for

a simple,

paper

is long

because

that

are mostly

simple.
feeling.

and outlines

the difficulties

involved

that supports the features
that we aimed for.
for comparison
of recovery
methods
that we

1.1.
and recovery
with
other

to

be error-prone,

yet

powerful

and

of its comprehensive

are complex subjects to think
aspects of data management.
if

they

flexible,

are

complex.

algorithm.

discussion

about
The

Hence,

Although

of numerous

problems

ignored

in the

literature,

the

main

algorithm

itself

the

overview

presented

in Section

3 gives

the reader

Hopefully,

we
this

is quite
that

Operation
logging.
The recovery
method had to permit
operation
logging
(and value logging)
so that semantically
rich lock modes could be supported.
This would
let one transaction
modify
the same data that
was modified
earlier
by another
transaction
which
transaction:’
actions are semantically

has not yet committed,
when the
compatible
(e.g., increment/decrement

operations;
see [2, 26, 45, 881). As should be clear,
always perform
value or state logging
(i. e., logging
images

of modified

systems

that

data),

do very

cannot

physical

support

recovery
methods
which
before-images
and after-

operation

—byte-oriented—

two

logging.

logging

of all

This

includes

changes

to a

page [6, 76, 811. The difficulty
in supporting
operation
logging
is that we need
to track precisely,
using a concept like the LSN, the exact state of a page
with respect to logged actions relating
to that page. An undo or a redo of an
update

should

not be performed

without

being

sure that

the original

update

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992

.

C. Mohan et al

is present

or is not present,

106

transactions

that

need to know

respectively.

had previously

precisely

how

This

modified

the page

also means

a page

has been

that,

if one or more

start

rolling

back,

affected

during

the rollbacks

then

we

and how much of each of the rollbacks
had been accomplished
so far. This
requires
that
updates
performed
during
rollbacks
also be logged
via the
so-called
compensation
log records (CLRS).
The LSN concept lets us avoid
attempting
to redo
present
in the page.

an operation
when
the operation’s
effect
is already
It also lets us avoid attempting
to undo an operation

when

effect

the operation’s

is not present

in the page.

Operation

logging

lets

us perform,
thing
that

if found desirable,
logical
logging,
which means that not everywas changed
on a page needs to be logged
explicitly,
thereby

saving

log

space.

amount

of free space on the page,

For

example,

operations
can be performed
value logging,
see [881.

changes

of control

information,

need not be logged.

logically.

like

the

The redo and the undo

For a good discussion

of operation

and

Efficient
support for the storage and manipFlexible storage management.
ulation
of varying
length
data is important.
In contrast
to systems
like
IMS, the intent
here is to be able to avoid the need for off-line reorganization
of the data to garbage
collect
any space that
might
have been freed up
because of deletions
and updates
that caused data shrinkage.
It is desirable

that

the

that
the

the
data

logging
within

moved
this
that

data

recovery

method

and locking
a page for

to be locked

and the

concurrency

control

method

be such

is logical
in nature
so that
movements
garbage
collection
reasons
do not cause

or the

movements

also means that one transaction
must
page currently
has some uncommitted

to be logged.

For

an

of
the

index,

be able to split a leaf page even if
data inserted
by another
transac-

tion. This may lead to problems
in performing
page-oriented
undos using the
log; logical undos may be necessary.
Further,
we would like to be able to let
a transaction
that has freed up some space be able to use, if necessary,
that
space during
its later insert
activity
[50]. System R, for example,
does not
permit

this

Partial

in data

pages.

rollbacks.

It

was

essential

that

the

new

recovery

method

sup-

port the concept
of savepoints
and rollbacks
to savepoints
(i.e.,
partial
rollbacks).
This
is crucial
for handling,
in a user-friendly
fashion
(i. e.,
without
requiring
a total
rollback
of the transaction),
integrity
constraint
violations
information
Flexible

(see [1, 311), and
(see [49]).
buffer

problems

management.

arising

The recovery

from

using

obsolete

method

should

make

cached

the

least

number
of restrictive
assumptions
about the buffer
management
policies
(steal,
force, etc.) in effect. At the same time, the method
must be able to
take advantage
of the characteristics
of any specific policy that is in effect
(e.g., with a force policy there is no need to perform
any redos for committed
transactions.)
This flexibility
could result in increased
concurrency,
decreased
1/0s and efficient
usage of buffer
storage.
Depending
on the policies,
the
work

that

needs

ACM Transactions

to be performed

during

restart

on Database Systems, Vol. 17, No. 1, March 1992

recovery

after

a system

ARIES: A Transaction Recovery Method
failure
large

or during
media recovery
maybe
main
memories,
it must be noted

more
that

.

107

or less complex.
Even
a steal policy
is still

with
very

desirable.
This is because,
with
a no-steal
policy,
a page may never get
written
to nonvolatile
storage
if the page always
contains
uncommitted
updates due to fine-~anularity
locking
and overlapping
transactions’
updates
to that

page.

running

transactions.

to frequently
by locking

The

situation

reduce
all

would

be further

aggravated

those

conditions,

either

Under

concurrency
objects

are

long-

would

have

by quiescing

all activities

on the page (i.e.,

page)

then

the

page

to non-

volatile
storage, or by doing nothing
special and then paying
a huge
redo recovery
cost if the system were to fail. Also, a no-steal
policy

restart
incurs

additional

the

if there

the system

bookkeeping

uncommitted
updates.
cally rich lock modes,
in the general
Hence,
general

11 with

reference
It should

independence.

perform

to

and

track

writing

whether

a page

contains

any

given our goal of supporting
semantiand varying
length objects efficiently,
undo

logging

and in-place

updating.

like the transaction
workspace
model of AIM [46] are not
for our purposes.
Other
problems
relating
to no-steal
are

in Section

Recovery
and

overhead

We believe that,
partial
rollbacks

case, we need to perform

methods
enough

discussed

on the

media

recovery

to IMS

Fast

be possible

or restart

Path.

to image

recovery

copy (archive

at different

dump),

granularities,

rather
than only at the entire
database
level. The recovery
of one object
should
not force the concurrent
or lock-step
recovery
of another
object.
Contrast
this with what happens
in the shadow page technique
as implemented
in System R, where index and space management
information
are
recovered
lock-step
with user and catalog
table (relation)
data by starting
from

an internally

to all the
processing.
some

consistent

state

related
objects
of the
Recovery independence

object,

catalog

descriptors
of that
may be undergoing

information

of the whole

in

object and its related
recovery
in parallel

the two may be out of synchronization
be possible
later point
devices.

database

and redoing

changes

database
simultaneously,
as in normal
means that, during the restart
recovery of
the

database

cannot

be

accessed

for

objects, since that information
itself
with the object being recovered
and
[141. During

restart

recovery,

it should

to do selective
recovery
and defer recovery
of some objects to a
in time to speed up restart
and also to accommodate
some offline

Page-oriented

recovery

means

that

even

if one page

in the

database

is corrupted
because of a process failure
or a media problem,
it should be
possible to recover that page alone. To be able to do this efficiently,
we need
to log

every

spans

multiple

conjunction

page’s
with

change

pages

and

the writing

will make media recovery
image
copying
of different
different
frequencies.
Logical
undo.
that
is different

This
from

individually,
the

update

even
affects

if the

more

of CLRS for updates

object

being

updated

than

one page.

This,

performed

during

rollbacks,

in

very simple (see Section 8). This will also permit
objects to be performed
independently
and at

relates
to the ability,
during
undo, to affect
the one modified
during
forward
processing,

a page
as is

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

108

.

C. Mohan et al.

needed in the earlier-mentioned
context of the split
index page containing
uncommitted
data of another
to perform

logical

undos

especially
in search
rollback
processing,

allows

higher

levels

by one transaction
of an
transaction.
Being able

of concurrency

to be supported,

structures
[57, 59, 621. If logging
is not performed
during
logical
undos would be very difficult
to support,
if we

also desired
recovery
independence
and page-oriented
recovery.
but
at the expense
of
R and SQL/DS
support
logical
undos,
independence.
Parallelism

and

fast

With

recovery.

multiprocessors

becoming

System
recovery

very

com-

mon and greater
recovery
method

data availability
becoming
increasingly
important,
the
has to be able to exploit
parallelism
during
the different

stages

of restart

recovery

and

the recovery

method

be such that

that

during

media

recovery.

recovery

It

is also

important

fast,

if in fact

can be very

hot-standby
approach
is going to be used (a la IBM’s IMS/VS
Tandem’s
NonStop
[4, 371). This means that redo processing
possible,

undo

processing

should

be page-oriented

(cf.

always

a

XRF [431 and
and, whenever
logical

redos

and undos in System R and SQL/DS
for indexes and space management).
It
should also be possible to let the backup system start processing
new transactions, even before the undo processing
for the interrupted
transactions
completes.

This

there

were

is necessary
long

Minimal

update

because

overhead.

Our

restart

recovery

normal

and

storage

consumption,

undo

processing

may

take

a long

time

if

transactions.

etc.)

goal

is to have

processing.
imposed

by the

good

The

performance

overhead

recovery

(log

method

both

during

data

volume,

in virtual

and

nonvolatile
storages for accomplishing
the above goals should be minimal.
Contrast
this with the space overhead
caused by the shadow page technique.
This goal also implied
that we should minimize
the number
of pages that are
modified
(dirtied)
during
restart.
The idea is to reduce the number
of pages
that have to be written
back to nonvolatile
storage and also to reduce CPU
overhead.
This rules out methods
which,
during
restart
recovery,
first undo
some committed
changes that had already
reached
the nonvolatile
storage
before the failure
and then redo them (see, e.g., [16, 21, 72, 78, 881). It also
rules
out
nonvolatile

methods
storage

in which
updates
that
are not present
in a page on
are undone
unnecessarily
(see, e.g., [41, 71, 881). The

method
should not cause deadlocks
involving
transactions
that are already
rolling
back. Further,
the writing
of CLRS should not result in an unbounded
number
of log records having
to be written
for a transaction
because of the
undoing
of CLRS, if there were nested rollbacks
or repeated
system failures
during
rollbacks.
It should also be possible
to take checkpoints
and image
copies without
quiescing
significant
activities
in the system. The impact
of
these operations
on other activities
should be minimal.
To contrast,
checkpointing
and image copying
in System R cause major perturbations
in the
rest of the system [31].
As the reader will have realized
by now, some of these goals are contradictory.
Based on our
features,
experiences
ACM Transactions

knowledge
with IBM’s

on Database Systems, Vol

of different
developers’
existing
systems’
existing
transaction
systems and contacts
17, No 1, March 1992

ARIES: A TransactIon Recovery Method

.

109

with customers,
we made the necessary tradeoffs.
We were keen on learning
from the past successes and mistakes
involving
many prototypes
and products.

3. OVERVIEW
The

aim

method

OF ARIES

of this
ARIES,

section
which

is to provide
satisfies

a brief

quite

overview

reasonably

of the

the goals that

Section
2. Issues like
deferred
and selective
restart,
restart recovery,
and so on will be discussed in the later
ARIES

guarantees

the

atomicity

and durability

new

recovery

we set forth

in

parallelism
during
sections of the paper.

properties

of transactions

in the fact of process,
transaction,
system
and media
failures.
For this
purpose, ARIES keeps track of the changes made to the database by using a
log and it does write-ahead
logging
(WAL).
Besides
logging,
on a peraffected-page
transactions,

basis, update
ARIES
also

(CLRS),

updates

during

both

partial

rollback

activities
performed
during
forward
logs, typically
using
compensation

performed

normal

and

in which

back two of them

during
restart

partial

or total

rollbacks

processing.

Figure

3 gives

a transaction,

and then

starts

going

after

performing

forward

again.

the two updates,
two CLRS are written.
In ARIES,
that they are redo-only
log records. By appropriate
log records

written

during

forward

processing,

processing
of
log records
of transactions

an example

three
Because

of a

updates,

rolls

of the undo

of

CLRS have the property
chaining
of the CLRS to

a bounded

amount

of logging

is ensured
during
rollbacks,
even in the face of repeated
failures
during
restart
or of nested rollbacks.
This is to be contrasted
with what happens
in
IMS, which may undo the same non-CLR
multiple
times, and in AS/400,
DB2
and NonStop
SQL, which, besides undoing
may also undo CLRS one or more times
severe

problems

in real-life

customer

as Figure

5 shows,

when

to be written,

the CLR,

besides

containing

for redo purposes,

is made

times,
caused

situations.

In ARIES,
action

the same non-CLR
multiple
(see Figure
4). These have

the undo

to contain

points
to the predecessor
of the just
information
is readily
available
since

of a log record

a description
the

causes

a CLR

of the compensating

UndoNxtLSN

pointer

which

undone
log record.
The predecessor
every log record,
including
a CLR,

contains
the PreuLSN
pointer
which points to the most recent preceding
log
record written
by the same transaction.
The UndoNxtLSN
pointer
allows us
to determine
precisely
how much of the transaction
has not been undone so
far. In Figure
5, log record 3’, which is the CLR for log record 3, points to log
record 2, which is the predecessor
of log record 3. Thus, during
rollback,
the
UndoNxtLSN
field
of the most recently
written
CLR keeps track
of the
progress

of rollback.

It tells

the system

from

whereto

continue

the rollback

the transaction,
rollback
or if

if a system failure
were to interrupt
the completion
a nested rollback
were to be performed.
It lets the

bypass

log

those

records

that

had

already

been

undone.

Since

of

of the
system

CLRS

are

available
to describe what actions are actually
~erformed
during
the undo of
an original
action, the undo action need not be, in terms of which page(s) is
affected, the exact inverse of the original
action. That is, logical undo which
allows

very

high

concurrency

to be supported

is made

possible.

ACM Transactions on Database Systems, Vol

For example,

17, No. 1, March 1992.

110

C. Mohan et al.

.

w
Fig. 3.

Partial

rollback

example.

12

Log

After

performing

rollback

I

33’2’4

3 actions,

by undoing

log

records

and

performs

3

!3j

the

transaction

performs

actions

3 and

2, wrlt!ng

2,

starts

and

and

then

4 and

5

2’”

3“

3’

3’

2’

1’

act~ons

>
a patilal

the compensation

go[ng

forward

aga!n

Before Failure
1

Log

During
DB2, s/38,
Encompass --------------------------AS/400

Restart

,
~

1;

lMS

>

)

I’ is the CLR for I and I“ is the CLR for I’
Fig. 4

Problem

of compensating

compensations

or duplicate

compensations,

or both

a key inserted
on page 10 of a B ‘-tree by one transaction
may be moved to
page 20 by another
transaction
before the key insertion
is committed.
Later,
if the first transaction
were to roll back, then the key will be located on page
20 by retraversing
the tree and deleted from there. A CLR will be written
to
describe the key deletion
on page 20. This permits
page-oriented
redo which
is very efficient.
[59, 621 describe
this logical undo feature.
ARIES

uses a single

a page is updated
and
placed in the page-LSN

LSN

ARIES/LHS

and ARIES/IM

on each page to track

the page’s

a log record is written,
the LSN
field of the updated
page. This

which
state.

exploit

Whenever

of the log record is
tagging
of the page

with
the LSN
allows
ARIES
to precisely
track,
for restartand mediarecovery
purposes,
the state of the page with respect to logged updates
for
that page. It allows ARIES to support
novel lock modes! using which, before
an update
performed
on a record’s
field by one transaction
is committed,
another
transaction
may be permitted
to modify
the same data for specified
operations.
Periodically
during
checkpoint
log records
and the
modified

normal
identify

processing,
ARIES
takes
checkpoints.
the transactions
that are active, their

LSNS of their
most recently
written
log records,
data (dirty data) that is in the buffer pool. The latter

needed

to determine

begin

its processing.

ACM Transactions

from

where

the

redo

pass

of restart

on Database Systems, Vol. 17, No. 1, March 1992.

The
states,

and also
information
recovery

the
is

should

ARIES: A Transaction Recovery Method

Before

12
‘,;
\\

Log

-%

I

.

111

Failure

3
3’ 2’ 1!
)
F i-. ?%
/
/
-=---/

------

---

During

Restart

/

,,

----------------------------------------------+1

I’ is the Compensation Log Record for I
I’ points to the predecessor, if any, of I
Fig. 5.

During

ARIES’

restart

technique

recovery

from

the first

record

this

analysis

pass,

for avoiding compensating
compensations.

compensation

(see Figure

first

of the last
information

6), ARIES

and duplicate

scans the log, starting

checkpoint,

up to the end of the

about

pages

dirty

log.

and transactions

During

that

were

in progress at the time of the checkpoint
is brought
up to date as of the end of
the log. The analysis
pass uses the dirty pages information
to determine
the
starting

point

( li!edoLSIV)

for the log scan of the immediately

pass. The analysis
pass also determines
the list of transactions
rolled back in the undo pass. For each in-progress
transaction,
most recently
written
log record
will
also be determined.

following

redo

that are to be
the LSN of the
Then,
during

the redo pass, ARIES
repeats history, with respect to those updates logged on
stable storage, but whose effects on the database pages did not get reflected
on nonvolatile

storage

before

the

failure

of the

system.

This

is done for the

updates of all transactions,
including
the updates of those transactions
that
had neither
committed
nor reached the in-doubt
state of two-phase
commit by
the time
loser

of the system

transactions

failure

are

(i.e.,

redone).

even the missing

This

essentially

updates

reestablishes

of the so-called
the

state

of

the database
as of the time of the system failure.
A log record’s
update
is
redone if the affected page’s page-LSN
is less than the log record’s LSN. No
logging
is performed
when updates
are redone.
The redo pass obtains
the
locks needed to protect the uncommitted
updates of those distributed
transactions that will remain
in the in-doubt
(prepared)
state [63, 64] at the end of
restart
The

recovery.
next log pass

updates

are rolled

is the

back,

undo

in reverse

pass

during

chronological

which
order,

all

loser

transactions’

in a single

sweep

of

the log. This is done by continually
taking
the maximum
of the LSNS of the
next log record to be processed for each of the yet-to-be-completely-undone
loser transactions,
until
no transaction
remains
to be undone.
Unlike
during
the redo pass, performing
undos is not a conditional
operation
during
the
undo pass (and during
normal
undo).
That
is, ARIES
does not compare
the page.LSN
of the affected
page to the LSN of the log record to decide
ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

112

C. Mohan et al

.

m

Log
@

Checkpoint

r’

Follure

i

DB2

System

Analysis

I

Undo Losers
/
*—————
——.

R

Redo Nonlosers

—— — ————,&

Redo Nonlosers
. ------

IMS

“––-––––––X*

..:--------

(FP Updates)

1 -------

ARIES

Redo ALL
Undo Losers

.-:”---------

Fig. 6,

whether

or not

transaction

to undo

during

the

Restart

processing

the

update.

undo

pass,

& Analysis

Undo Losers (NonFP Updates)

in different

When
if

it

methods.

a non-CLR
is an

I

is encountered

undo-redo

for

or undo-only

a
log

record, then its update is undone.
In any case, the next record to process for
that transaction
is determined
by looking
at the PrevLSN
of that non-CLR.
Since

CLRS

are never

undone

(i.e.,

CLRS

are not

compensated–

see Figure

5), when a CLR is encountered
during
undo, it is used just to determine
the
next log record to process by looking
at the UndoNxtLSN
field of the CLR.
For those transactions
which were already
rolling
back at the time of the
system failure,
ARIES
will rollback
only those actions
been undone.
This is possible since history
is repeated
and since the last CLR written
for each transaction
indirectly)

to the next

non-CLR

record

that

that had not already
for such transactions
points
(directly
or

is to be undone,

The net result

is

that, if only page-oriented
undos are involved
or logical undos generate
only
CLRS, then, for rolled back transactions,
the number
of CLRS written
will be
exactly equal to the number
of undoable)
log records
processing
of those transactions.
This will
be the

written
during
forward
case even if there
are

repeated

failures

rollbacks.

4. DATA

STRUCTURES

This

section

4.1

Log Records

during

describes

Below,

we describe

types

of log records.

ACM Transactions

restart

the major

the

important

or if there

data

are nested

structures

fields

that

that

are used by ARIES.

may

be present

on Database Systems, Vol. 17, No. 1, March 1992,

in

different

ARIES: A Transaction Recovery Method

.

113

LSN.
Address
of the first byte of the log record in the ever-growing
log
address space. This is a monotonically
increasing
value. This is shown here
as a field only to make it easier to describe
ARIES.
The LSN need not
actually

be stored

in the record.

Indicates

whether

Type.

record

this

regular

update

pare’),

or a nontransaction-related

TransID.

Identifier

PrevLSN.

LSN

is a compensation

(’update’),

a commit
record

(e.g.,

of the transaction,

of the preceding

record

(’compensation’),

protocol-related

record

‘OSfile_return’).

if any, that

log record

wrote

written

the log record.

by the

tion. This field has a value of zero in nontransaction-related
the first log record of a transaction,
thus avoiding
the need
begin

transaction

PageID.
identifier
PageID

same transacrecords and in
for an explicit

log record.

Present
only in records of type ‘update’
or ‘compensation’.
of the page to which the updates of this record were applied.

will

normally

consist

of two

parts:

an objectID

(e.g.,

and a page number
within
that object. ARIES can deal with
contains
updates for multiple
pages. For ease of exposition,
only

a

(e. g., ‘pre-

The
This

tablespaceID),

a log record
we assume

that
that

one page is involved.

UndoNxtLSN.
Present
of this
transaction
that
UndoNxtLSN
is the value

only in CLRS. It is the LSN of the next log record
is to be processed
during
rollback.
That
is,
of PrevLSN
of the log record that the current
log

record is compensating.
If there
this field contains
a zero.
Data.

This

is the

redo

are no more

and/or

undo

data

log records

that

to be undone,

describes

was performed.
CLRS contain
only redo information
undone.
Updates
can be logged in a logical fashion.

the

then

update

that

since they are never
Changes
to some fields

(e.g., amount
of free space) of that page need not be logged since they can be
easily derived.
The undo information
and the redo information
for the entire
object need not be logged. It suffices if the changed fields alone are logged.
For increment
or decrement
types of operations,
before and after-images
of
the field are not needed.
Information
about the type of operation
and the
decrement
or increment
amount
is enough.
The information
here would also
be used to determine
redo and/or

undo

4.2

Page Structure

One

of the

fields

the appropriate

of this

action

routine

to be used to perform

the

log record.

in every

page

of the

database

is the

page-LSN

field.

It

contains
the LSN of the log record that describes
the latest update
to the
page. This record may be a regular
update record or a CLR. ARIES
expects
the buffer manager
to enforce the WAL protocol.
Except for this, ARIES does
not place any restrictions
on the buffer
page replacement
policy.
The steal
buffer
management
policy may be used. In-place
updating
is performed
on
nonvolatile
storage.
Updates
are applied
immediately
and directly
to the
ACM Transactions on Database Systems, Vol. 17, No, 1, March 1992.

114

.

buffer
as in
ing

C. Mohan et al.

version of the page containing
INGRES
[861 is performed.
and,

flexible

4.3

consequently,
enough

A table

deferred

not to preclude

Transaction

If

the object. That is, no deferred updating
it is found
desirable,
deferred
updat-

logging

can

be

those

policies

from

table

is used during

implemented.
being

ARIES

is

implemented.

Table

called

the

transaction

restart

recovery

to track

the state of active transactions.
The table is initialized
during
the analysis
pass from the most recent checkpoint’s
record(s)
and is modified
during
the
analysis

of the

log records

written

after

the

During
the undo pass, the entries
of the
checkpoint
is taken
during
restart
recovery,
will

be included

in

the

checkpoint

during
normal
processing
by the
important
fields of the transaction
TransID.
State.

Transaction
Commit

or unprepared
LastLSN.

record(s).

The

same

transaction
manager.
table follows:

checkpoint.

table

If a
table

is also

A description

used
of the

ID.

state of the transaction:

prepared

The LSN
The

If the most

of the latest
LSN

recent

of the

log record

log record
next
written

record

(’P’ –also

then this field’s
is a CLR, then

UndoNxtLSN

CLR.

value

from

that

written

called

in-doubt)

by the transaction.

to be processed

or seen for this

undoable
non-CLR
log record,
If that most recent log record

4.4

of that

are also modified.
the contents
of the

(’U’).

UndoNxtLSN.
back.

beginning
table
then

value will
this field’s

during

transaction

rollis an

be set to LastLSN.
value is set to the

Dirty_ Pages Table

A table called the dirty .pages table is used to represent
information
about
dirty buffer pages during
normal
processing.
This table is also used during
restart
recovery.
The actual implementation
of this table may be done using
hashing
or via the deferred-writes
queue mechanism
the table consists of two fields: PageID and RecLSN
normal
processing,
when a nondirty
the intention
to modify,
the buffer

of [961. Each entry in
(recovery
LSN). During

page is being fixed in the buffers
manager
records in the buffer pool

with
(BP)

dirty .pages table, as RecLSN,
the current
end-of-log
LSN, which will be the
LSN of the next log record to be written.
The value of RecLSN indicates
from
what point in the log there may be updates which are, possibly,
not yet in the
nonvolatile
storage version
of the page. Whenever
pages are written
back
to nonvolatile
storage, the corresponding
entries in the BP dirty _pages table
are removed.
record(s) that

The contents
of this table
are included
is written
during
normal
processing.
The

in the checkpoint
restart
dirty –pages

table
is initialized
from the latest
checkpoint’s
record(s)
and
during
the analysis
of the other
records
during
the analysis
ACM Transactions

on Database Systems, Vol

17, No 1, March 1992

is modified
pass. The

ARIES: A Transaction Recovery Method
minimum
RecLSN
pass during
restart

5. NORMAL
This

discusses

the

processing.

part

of recovering

5.1

Updates

During

table

gives

the

starting

point

for

115

the

redo

PROCESSING

section

transaction

value in the
recovery.

.

normal

from

actions

that

are

Section

6 discusses

a system

failure.

processing,

transactions

performed

the actions

as part
that

may be in forward

of normal

are performed

processing,

as

partial

rollback
or total rollback.
The rollbacks
may be system- or application-initiated.
The causes of rollbacks
may be deadlocks,
error conditions,
integrity
constraint
violations,
unexpected
database
state, etc.
If the granularity
of locking
is a record, then, when an update
is to be
performed
on a record in a page, after the record is locked, that
in the buffer and latched in the X mode, the update is performed,

page is fixed
a log record

is appended
to the log, the LSN of the log record is placed in the page .LSN
field of the page and in the transaction
table, and the page is unlatched
and
unfixed.

The page latch

is held

during

the call to the logger.

This

is done to

ensure that the order of logging
of updates of a page is the same as the order
in which those updates are performed
on the page. This is very important
if
some

of the

redo

information

is going

to be logged

amount
of free space in the page) and
guaranteed
for the physical
redo to work
be held during
read and update operations
the page contents.
This is necessary
might
move records around
within
such garbage

collection

is going

look at the page since they

repetition
correctly.
to ensure

transaction

get confused.

Readers

S mode and modifiers
latch in the X mode.
The data page latch is not held while any

necessary

performed.

held

At

most

two

page

(e.g.,

the

because inserters
and updaters
of records
a page to do garbage
collection.
When

on, no other

might

physically

of history
has to be
The page latch must
physical
consistency
of

latches

are

should

be allowed

of pages latch
index

operations

simultaneously

to

in the

(also

are
see

[57, 621). This means
that
two transactions,
T1 and T2, that
are modifying different
pieces of data may modify a particular
data page in one order
(Tl, T2) and a particular
index page in another
order (T2, T1).4 This scenario
is impossible
in System R and SQL/DS
since in those systems, locks, instead
of latches
are used for providing
physical
consistency.
Typically,
all the
(physical)
page locks are released only at the end of the RSS (data manager)
call. A single
RSS call deals with
modifying
the data and all relevant
indexes.

This

deadlocks

may

involve

waiting

(physical)

page

for many
locks

1/0s

alone

and locks.
or (physical)

This

means

page

that

locks

and

gets very complicated if operations like increment/decrement
are supported
high concurrency lock modes and indexes are allowed to be defined on fields on which
operations are supported. We are currently studying those situations.

with
such

4 The situation

involving

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

116

.

C. Mohan et al

(logical)

record/key

System

R and SQL/DS.

Figure

7 depicts

locks

are possible.

a situation

They

at the time

have

been

of a system

a major

failure

problem

which

in

followed

the commit
of two transactions.
The dotted lines show how up to date the
states of pages PI and P2 are on nonvolatile
storage with respect to logged
updates of those pages. During
restart
recovery,
it must be realized
that the
most recent log record written
for PI, which was written
by a transaction
which later committed,
needs to be redone, and that there is nothing
to be
redone for P2. This situation
points to the need for having
the LSN to relate
the state of a page on nonvolatile
and the need for knowing
where
some information

storage
restart

in the checkpoint

record

to a particular
position
redo pass should begin
(see Section

5.4).

in the log
by noting

For the example

scenario,
the restart
redo log scan should begin at least from the log record
representing
the most recent update of PI by T2, since that update needs to
be redone.
It is not assumed that a single log record can always accommodate
information
needed to redo or undo the update
operation.
There

all the
may be

instances

purpose.

when

more

than

one record

needs

to be written

for this

For example,
one record
may be written
with
the undo information
and
another
one with the redo information.
In such cases, (1) the undo-only
log
record should be written
before the redo-only
log record is written,
and (2) it
is the LSN of the redo-only
log record
field.
The first condition
is enforced
situation

in which

the

written
to stable storage
the redo of that redo-only
history

feature)

only

redo-only

that should be placed in the page.LSN
to make sure that we do not have

record

and

not

the

undo-only

before a failure,
and that during
log record is performed
(because

to realize

later

that

there

isn’t

restart
of the

an undo-only

record

a

gets

recovery,
repeating
record

to

undo the effect of that operation.
Given that the undo-only
record is written
before the redo-only
record, the second condition
ensures that we do not have
a situation
in which
even though
the page in nonvolatile
storage
already
contains
the
unnecessarily
the undo-only
redo could

update
during
record

cause

of the redo-only
record, that same update
gets redone
restart
recovery because the page contained
the L SN of
instead of that of the redo-only
record. This unnecessary

integrity

problems

if operation

There may be some log records written
cannot or should not be undone
(prepare,

logging

is being

performed.

during
forward
processing
free space inventory
update,

that
etc.

records). These are identified
as redo-only
log records. See Section 10.3 for a
discussion
of this kind of situation
for free space inventory
updates.
Sometimes,
the identity
of the (data) record to be modified
or read may not
be known before a (data) page is examined.
For example,
during
an insert,
the record ID is not determined
until the page is examined
to find an empty
slot. In such cases, the record lock must be obtained
after the page is latched.
To avoid waiting
for a lock while
holding
a latch, which
could lead to an
undetected
deadlock,
the lock is requested
conditionally,
and if it is not
granted,
then the latch is released and the lock is requested
unconditionally.
Once the unconditionally
requested
lock is granted,
the page is latched again,
and any previously
verified
conditions
are rechecked.
This rechecking
is
ACM Transactions on Database Systems, Vol 17, No. 1, March 1992.

ARIES: A Transaction Recovery Method

/’
/’
j;:’

Log
PI

o

T1

a

T2

because,

changed.

The

page_LSN

bered

detect

quickly,

If

conditions

the

after

update,

it

is performed

taken.

If

the

update
If

can
the

page,

be sufficient
actions

to support
tion

if they

hold

performed
amount

an

rency

control

be used

conditions

if

any

could

have

for

performing

Otherwise,
granted

have

remem-

satisfied

be

is

be

changes

above.
lock

could

could

possibly
the

corrective

actions

are

immediately,

then

the

locking

a

page

or

the

page

since

the

coarser

lock

will

change,

the

transaction.

Except

for

this

case.

But,

if the

then,

even with

should

be made
locks

while

reading

the

page.

utility

in

the

copy
to normal

transaction

is

not

restricted

concurrency

control

that

are

similar

to hold
are

the

assured

locking,

is

a transac-

X latch

on the

physical

consistency

Unlocked
interest

system

reads
of

may

page

also

be

causing

the

least

systems

in

which

processing.
to

to

page

a

page

record-locking

reads,

than

on the

as in the

acquiring

ARIES

something

executing

not

image

as the

is

to latch

only

those

mechanism.
locking,

Even

like

the

other

ones

in

concur[2],

could

rollbacks,

the

ARIES.

Total or Partial Rollbacks

To provide

flexibility

of a sauepoint

notion

of a transaction,
could

be

in

limiting

the

is supported

a savepoint

outstanding

savepoint

at

is established

can

updates

to the

data.

After

executing

for

request

the

undoing

outstanding

of all

savepoint.

be

number

of savepoints

every

perform

the

established.

time.
SQL
This

such

Any

Typically,

in

data

manipulation

is needed

a while,

updates

After

transaction

the execution

before

atomicity.

of

during

in

level

extent

[1, 31]. At any point

a point

might

still

the

of unlatching

as described

a page

schemes

with

time

rematching,

dirty

S latch

of
used

the

to

same

of interference
is

unlatched,

was
at

found

are

the

Applicability
locking

page

need

or

who

by

state as a failure.

still

the

the

is updating
readers

5.2

is no

unlocked

that

Database

as before.

are

so that

Commit

P2

@ Checkpoint

requested

to isolate

taken

‘:\,;

w

Failure

are

of

there

‘“O

Commit

value
on

granularity

then

the

conditionally

proceed

‘!
‘!

/

required
to

/’

PI

Fig. 7.

occurred.

P

#“ PI

LZN’”S

pi

117

El

/
,’

.

the

a system

to support

transaction

performed

after

a partial

rollback,

like

I)B2,

command
SQL
or the

the

a

that

statementsystem

can

establishment

of a

the

can

ACM Transactions on Database Systems, Vol

transaction

17, No. 1, March 1992.

118

.

C. Mohan et al.

continue
lar

execution

savepoint

and

is

that

savepoint

LSN

of the

no
or

in

a preceding

log

record

virtual

of the

transaction

SaveLSN

is

set

to

zero.

savepoint,

it

supplies

the

to be exposed

expose

the

numbers

and

INGRES

[181.

Figure
locks

are

undo
get

activity

deadlock,
During

the

order

in

System

as

rollback,

and,

for

information

written.

when

PrevLSN
Since
tion
When

is

will

is

the

log

never

up to determine
helps

nested

rollback

during

the

UndoNxtLSN

be undone,

they

don’t

have

log
after

by

looking

rollback,

the

next

log

over

were

to

rollback

then,

would

be processed

describe

partial

rollback

scenarios

various

recovery

methods,

handled

efficiently

by

Being

able

the

flexibility

of

of the

original

us

inverses
page

which

situations

are

management
ARIES’
deal

safely

not

with

small

in

a

multiple
is

As

records

some

to contain
undo

informa-

the

rollback.

next
When

of that

record
the

This

record

to

a

is

CLR
is looked

UndoNxtLSN
means

that

UndoNxtLSN
were

in
undone

conjunction

with

restart

undos

see how

nested

rollbacks

CLRS,

the

to

a

during

though

be easy

Figures

if

CLRS,

Even

should

the

to be written.

field.

that

need

mentioned

during

Thus,

of the

CLRS

CLR

ignored

records.

in

performed,

62].

actions

4, 5, and

force

actions.

In

particular,

undo

action

involved

in

the

original

action.

Such

example,

index

management

for

undo

during

to

in,

the

performed

having

Section

guarantee

ACM Transactions

via

not

possible
(see

log

again.

of

fit

in

13
the
are

ARIES.

to describe,

was

because

of the

rollback

ease

processed,

log

a

[1001.

will

to contain

to be processed.

in

of

chronological

is made

field

do not

involved

For

this

PrevLSN

undone

first

it

is

its

latches

action

[59,

are

UndoNxtLSN

record

none

it

up

already

occur,

records

No
during

undo

field
caused

and

back

acquired

reverse

undo
in

undo

skip

second

a logical
described

to

is written.

where

whose

encountered,

the
us

its

Redo-only

is

during

as

the

case

[42]

algorithms

in

a CLR

to the

when

the

undone

record

determined

pointer

that,
written,

before-images).

encountered

the

possible

in

about

ARIES

is written,

in

a non-CLR

process

to extend

a CLR

CLRS
(e.g.,

the

sometimes

value

are

is undone,

not

TransID.

get

a

sequence

IMS

the

cannot

and

to

is used for rolling

that

641

records

or

ensured

[31,

all

to

before,

log

system

is

always

that

It is easy
It

R*

the

a latch

transaction

record)
concept

values

and

the

back

savepoint

though

have

that

CLR.

are

and

roll

in

is

at

a log

to

expect

SaveLSN

back

SaueLSN,

written

symbolic

which

the

established

the

to

established,

as is done

is the

record

single

would

internally,

log

assume

be

we

yet
If

some

even

a rolling

the

each

exposition,

non-CLRs

Since
R

not

A particu-

performed

called

desires

we

use

routine

rollback,

a page.

deadlocks,

has

ROLLBACK

to the

during

then

but

is

is being

SaveLSN.

to LSNS

routine

in

it

3).

been

transaction,

transaction

level,
user

the

on

involved

user

The input

acquired

the

when
the

Figure

has

a savepoint

savepoint

remembered

mapping

8 describes

to a savepoint.

(i.e.,

to the

do the

the

(see

a rollback

When
by

If

again

if

written

When

at the

SaveLSNs

forward
one.

storage.

beginning

were

going
outstanding

to

latest

remembered

start

longer

the

actions

to

undo

gives

the

exact

be
could

affect

logical
[621

and

a

undo
space

10.3).

of a bounded
computer

amount
systems

of logging
situations

during
in which

on Database Systems, Vol. 17, No. 1, March 1992

undo

allows

a circular

us to
online

ARIES: A Transaction Recovery Method

\\\
***

.

119

\

u

,0

w

m

dFm
m

0
<0

m
c
m
L

..

~
v
al
sQ

..
..

z

x

m“.

-J

nc.1

WE

>

..!

!’. :

0 %’
:
0 CIA. . . .
Fl

‘n

..!

I

.

5

n“

-_l

WI-’-l

al

!!

..
w
M.-s
mztn
CL.
ulc
-am
UWL
aJ-.J
Crfu
u!
It
0
.-l
=%
ql-

z

&

l..-

..2
!!

;E
%’2

al-

ACM Transactions on Database Systems, Vol. 17, No 1, March 1992.

120

C. Mohan

.

log might

et al

be used and log space is at a premium.

keep in reserve

enough

log

transactions

under

critical

mentation

of ARIES

in

advantage

of this.

When

a transaction

of the

savepoint

partial

or

cannot

any

release,

conditions

Extended

rolls

back,

the

is the

locks

rollback

again,

thereby

causing

data

after

a partial

rollback

completes.

nor

ever

chaining

undoes

a

particular

of the

CLRS

using

when
a CLR

is written

transaction’s
for

This

makes

it possible

to consider

rather

than

5.3

Transaction

form

some
Commit

prepare

record

includes

transaction.
to

could

be

updates
read

occur

the

during
S

actions

(such

as the

sake

of

Once
any

the

OSfile.

this

log

action

the

does

release

locks

undoes

CLRS

because
a (partial)
object

the

lock

of

the
roll-

is undone

on that

using

partial

(e. g.,

Presumed

object.

rollbacks

erasing

return
not

(IX,

X,

that

the

in-doubt

state,

recovery,

to

the

the

same

record
new

files

to be

committing

[191.

cause

complete

the

would

To

may

locks

is written,
locks

site).

objects’

erasing

those

or a different

which

the

uncommitted

other

such

of

by

failure

some

of objects)

in

that
of the

held

then
the

if

state

as part

etc.)

protect
no

and

if a system

prepare

prepare
site

log

SIX,

to ensure

released,

Abort

transactions
to the

logging
like

be

part

deal

of

with

erased,

contents,

we

files

until

we

are

sure

that

the

We

need

to

log

these

pending

it is committed

by

record.
enters

actions,

the

dropping

definitely

locks

be

into

(at

and releasing

record
does

a

never
once,

protocol

5 When

of getting

prepare

involves

an

such

during

written

enters

could

actions

a transaction

pending

which

after

to be undone

than

to terminate

is done

restart

IS)

avoiding

is

end record

because,

field,

release

commit
is used

transaction.
and

performing
in

the
and

rollbacks.

locks

a transaction

for

actions

after
do not

updates
R

deadlocks

of update-type

transaction

transaction

establishment

DB2

to a particular

can

resolving

64]))

of the

as part

distributed

postpone

update

synchronously

is

list

in-doubt

(e.g.,
later

the

first

of two-phase

logging

after

of the

acquired

more

system

same

ARIES

UndoNxtLSN

to total

which

reacquired,

locks

non-CLR

takes

the

like

System

because

the

(see [63,

the

The

were

the

imple-

be released

rollback

cause

The
Manager

Termination

that

protocol

a partial

very

resorting

or Presumed
the

it,

after
may

we can
running

shortage).

Database

systems

But,

the

and

always

fact,

all

space

obtained

In

still

currently

rollback

inconsistencies.

back,

Assume

the

log

locks

after

may

the bound,

back

Edition

of the

is completed.

of the

a later

target

Knowing

to roll

(e. g.,

0S/2

rollback

release

lock

to be able

the

which

total

space

the

they

in-doubt

locks.

its

they

must

or returning

redo-only

state,

Once the end record
be performed.
a file

log record.

to the

For

each

operating

For ease of exposition,

is not

associated

with

take

place

a checkpoint

when

is written,

any

particular
is in

transaction

writing

an

if there

are

pending

system,

action
we

write

we assume

that

and

this

that

progress.

5Another possibility
is not to log the locks, but to regenerate the lock names during restart
recovery by examining
all the log records written by the in-doubt transaction— see Sections 6.1
and 64, and item 18 (Section 12) for further ramifications
of this approach
ACM Transactions

on Database Systems, Vol. 17, No. 1, March 1992.

ARIES: A Transaction Recovery Method
A transaction
record, rolling
actions

list,

in-doubt
state is rolled
back by writing
the transaction
to its beginning,
discarding

in

releasing

its locks,

and then

writing

the end record.

not the rollback
and end records are synchronously
written
will depend on the type of two-phase
commit
protocol used.
record

may

be avoided

if the

121

a rollback
the pending

the

back

of the prepare

.

Whether

or

to stable storage
Also, the writing

transaction

is not

a distributed

the amount

of work

one or is read-only.

5.4

Checkpoints

Periodically,

checkpoints

are taken

to reduce

that

needs

to be performed
during
restart
recovery.
The work may relate to the extent of
the log that needs to be examined,
the number
of data pages that have to be
read from nonvolatile
storage, etc. Checkpoints
can be taken asynchronously
(i.e.,

while

fuzzy

checkpoint

transaction

processing,

is initiated

including

by

writing

updates,
a

end– chkpt

record

is constructed

transaction

table,

the

BP dirty-pages

table,

(like

tablespace,

indexspace,

tion

for the objects

which

BP dirty–pages

table

by including

has entries).

and

any

log this

end-chkpt

record

Such

Then

a

the

of the normal

mapping

informa-

are “open”

(i.e.,

for simplicity

can be accommodated
the case where multiple

Once the

file

etc.) that

Only

on).

record.

in it the contents

assume that all the information
record. It is easy to deal with
information.

is going

begin-chkpt

for

of exposition,

we

in a single end- chkpt
records are needed to

is constructed,

it is written

to the log. Once that record reaches stable storage, the LSN of the begin-chkpt
record is stored in the master record which is in a well-known
place on stable
storage.
If a failure
were to occur before the end–chkpt
record migrates
to
stable storage, but after the begin _chkpt
record migrates
to stable storage,
then that checkpoint
is considered
an incomplete
checkpoint.
Between
the
begin--chkpt

and

end. chkpt

log

records,

transactions

might

have

written

other log records.
If one or more transactions
are likely
to remain
in the
in-doubt
state for a long time because
of prolonged
loss of contact
with
the

commit

coordinator,

then

it is a good idea

to include

record

information

about

the update-type

locks

(e.g.,

those

transactions.

This

way,

were

to occur,

recovery,

those

locks

could

if a failure
be

reacquired

in the

end-chkpt

X, IX and SIX)

without

then,

having

during
to

held

by

restart

access

the

prepare records of those transactions.
Since latches
may need to be acquired
to read the dirty _pages table
correctly
while gathering
the needed information,
it is a good idea to gather
the information
a little
at a time to reduce contention
on the tables.
For
example,
tion

if the

dirty

100 entries

before
Figure

_pages

table

If the

already

during

written

important

by
because

transactions
the

entries

acquisichange

remain
correct (see
redo point,
besides

of the RecLSNs of the dirty pages included
also takes into account the log records that

since

effect

each latch

examined

the end of the checkpoint,
the recovery
algorithms
10). This is because,
in computing
the restart

taking
into account the minimum
in the end_chkpt
record, ARIES
were

has 1000 rows,

can be examined.

of some

the

beginning

of the

updates

of the
that

checkpoint.
were

performed

This

is

since

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

122

C. Mohan et al.

.

the

that

initiation
of the checkpoint
might
not
is recorded
as part of the checkpoint.

be reflected

require

that

pages

a checkpoint.

The

ARIES

does

storage
on

not

during

a continuous

system
ple

processes.

pages

buffer

in

in

this

in case

make

a copy

minimizes

When

the

to

before

site

dirty

were

pages

data

unavailability

system
the

properties

to

at the

to this

routine

is the

the

begin

failure

writes

and

write

multi

how

DB2

manages

its

pages

which

are

those

pages

hot-spot

ensure

for

that

reduce

manager

restart

the

prevention

the

buffer

the

1/0

is,

redo

-

are
work,

of updates

manager

could

the

This

from

copy.

writes.

.chkpt

or

the

pass

is updated

the

ensure

and

needs
the

9 describes

of the

master

record

which

of

last

complete

routine

undo

appropriately.

At

the

checkpoint

invokes

the

routines

order.

The

the

of restart

end

system.

contains

in that

pass,

be

RESTART

the

of a failed

the

to

atomicity

restart

This

and

recovery

state
of the

LSN

record

a failure,

Figure

beginning

shutdown.

redo

table

after

a consistent

taken
for

buffer

the
pool

recovery,

a

be as short

as

is taken.

high

availability,

duration

the

redo

and

undo

passes.

Only

if

necessary

to

latch

pages

before

they

improving

data

availability

for

way

the

One

recovery

6.1

Analysis

The

first

Figure

ments

the

are

contains

explored

of the

log

that
the

analysis

pass

actions.

the

The
list

outputs

the
failed

or was

the

log

from

which

the

records

that

may

be written

missing.

must

exploiting

parallelism

are
by

is going
modified

to

be

during

allowing

new

during

employed

restart

is

it

recovery.

transaction

processing

[601.

list

totally

ACM ‘llansactlons

of pages

The

that

rolled

back

were

routine

are the

transaction

by

were

in

potentially
the

must
this

before

routine
system

dirty

are

is the

in

the

which

processing
failure,

that

imple-

LSN

of the

table,

which

the
in-doubt
or unprepared
the dirty–pages
table, which

RedoLSN,

start

analysis

is the

routine

routine

and

pass

recovery

to this

or shutdown;

down;

redo

restart

ANALYSIS

input

which

failure

shut

during

RESTART_

of transactions

system

are

processing

is by

parallelism

of this

of system

contains

had

of restart
this

is made

describes

at the time

that

in

10

record.

master

of accomplishing

Pass

pass

pass.

state

the

to

perform

possible.

Ideas

using

To avoid

of transactions.

invoked

_pages

during

background

some

and

restarts

data

gets

checkpoint

the

to

operation,

nonvolatile

in

are

1/0

to

buffer

about

often

time

forced

list

page

the

has

to occur.

of those

bring

pass,

For

reasonably

failure

batch
details

dirty

the

PROCESSING

that

analysis

storage

be

is that
pages

if there
manager

of each

durability

pointer

can

[961 gives

buffer

transaction

input

manager

an

to

routine

dirty

Even

the

pages

the

performed

out

during

6. RESTART

The

buffer

fashion.

a system

hot-spot

assumption

writing

operation.

to nonvolatile

such

and

1/0

modified,

written
to

The

one

pools

frequently
just

basis,

dirty

any

in

end
but

on Database Systems, Vol. 17, No. 1, March 1992.

the
records
for

buffers

when

is the

location

log.
for
whom

The

only

the
on
log

transactions
end

records

ARIES: ATransaction

Recovery Method

.

123

RE.STAR7(Master Addr);
Restart_Analys~
Restart_

s(Master_Addr,

Redo(RedoLSN,

buffer

pool

Dirty_Pages

remove

entries

for

Restart_

Undo (Trans_Tabl

reacquire

locks

Trans_Table,

Trans_Table,
table

Dlrty_Pages,

:=

Dirty_

Pages;

non-buffer-resident

for

RedoLSN);

Dlrty_Pages);
pages

from

the

buffer

pool

Dirty_

Pages

table;

e);

transactions;

prepared

checkpoint;
RETURN;
Fig.9.

During

this

does not

already

the table
transaction

back.

if a log record

appear

in the

with
the current
table is modified

also to note
undone

pass,

the

LSN

if it were
file

which

is encountered

dirty

_pages

of the

most

recent

then

log record

ultimately

log record

that

whose

an entry

is encountered,

are in the dirty-pages

table

sure that
the redo

no page belonging
pass. The same file

later,

original

operation

causing

that

identity

is made

would

the transaction

order to make
accessed during
once the

for a page

table,

log record’s
LSN as the page’s RecLSN.
to track the state changes of transactions

determined

If an OSfile.return

to that

Pseudocode for restart.

then

in
The
and

need

to be

had to be rolled

any pages belonging

are removed

from

the latter

in

to that version
of that file is
may be recreated
and updated

the

file

erasure

is committed.

In

that case, some pages of the recreated
file will reappear
in the dirty-pages
table later with RecLSN
values greater
than the end-of-log
LSN when the
file was erased. The RedoLSN
is the minimum
RecLSN from the dirty-pages
table at the end of the analysis
are no pages in the dirty _pages
It is not necessary
ARIES

that

implementation

there

is no analysis

pass.

pass.
table.

The

redo pass can be skipped

there

be a separate

in

0S/2

the

This

analysis

Extended

is especially

Section

6.2),

in the

redo

pass,

ARIES

missing

updates.

That

is, it redoes

them

irrespective

logged

by loser or nonloser
redo

tion.
This

transactions,

does not need to know

unlike

the loser

unconditionally

their

update

locks

System

R, SQL/DS
status

are reacquired

computation

to consider

the

Begin _LSNs

in the

Manager
before

redoes

all

they

were

and DB2.

of a transac-

only for the undo pass.
in which
for in-doubt

by inferring

from the log records of the in-doubt
transactions,
during the redo pass. This technique
for reacquiring
turn requires
that we know, before the start
the in-doubt
transactions.
Without
the analysis
pass, the transaction

of whether

or nonloser

That information
is, strictly
speaking,
needed
would
not be true
for a system
(like
DB2)

transactions

Database

as we mentioned

(see also

Hence,

pass and, in fact,

Edition

because,

if there

the

lock

names

as they are encountered
locks forces the RedoLSN

of in-doubt

which

in

of the redo pass, the identities

of

table

transactions

could

be constructed

from

the checkpoint
record and the log records encountered
during
the redo pass.
The RedoLSN
would have to be the minimum(minimum(
RecLSN
from the
dirty-pages
table in the end.chkpt
record), LSN(begin-chkpt
record)).
Suppression of the analysis
pass would also require
that other methods be used to
ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992

124

C. Mohan

0

et al.

#~START_ANALYSIS(Mast er_Addr,
ln]tiallze

the

Trans_’able,

Trans_Table

tables

D1rty_pages,

arm D1rty_Pages

to

RedoLSN) ;
empty;

Master_Rec := Read_Dl sk(Master_Addr)
;
Open_ Log_ Scan (Master_Rec .Chkpt LSN) ;
LogRec := Next_ Logo;

/’
/*

LogRec := Next_ Logo;
WHILE NOT(End_of_Log)

open log scan at Beg)n_Chkpt
/* read )n the Begln_Chkpt
read

log

record

followlng

record
record

‘/
‘/

Begln_Chkpt

*/

00;

ret Urn*/
IF trans related
record & LogRec.7ransi3
‘/C- ;n Trans Table THEN /* not chkpt/OSflle
/* log ~ecord */
Insert
(Log Rec. Trans ID, ’U’ ,Log Rec. LSN, Log Rec. Frev LSN) l!,:o Trans Table;
SELECT(LogRec. Type)
WHEN(’update’
I ‘compensation’)
DO;
Trans_Tabl

e[LogRec. Trans ID] .Last LSN := LogRt-:. LSN;

IF LogRec. Type = ‘update’
IF LogRec 1s undoable

THEN
THEN Trans_Tahl

e[.ogRec.

TransIO]

.UndoNxt LSN := LogRec. LSN;

ELSE Trans_Tabl e[LogRec. Trans IDU.UndoNxt LSN := LogRec. UndoNxt LSN;
/’ next record to undo 1s the one pointed
IF LogRec is redoable
& LogRec. ~age ID NOT IN DTrty_Pages THEN
insert
(LogRec. Page ID, Log Rec. LSN) Into Llrty_Pages;
END; /’ WHEN(‘update’
I ‘compensation’)
*/
WHEN(‘Begln_Chkpt ‘) ; /* found an Incomplete

record.

ignore

CLR */

*/

ENO; /* SELECT ‘/
LogRec := Next_ Logo;
ENO; /’ WHILE ‘/
FOR EACH Trans Table entry with (State = ‘U’) & (Undo Nxt LSN = O) 00; /* rolled
back trans
write
end re~ord and remove entry from Trans Table;
I* w)th mlsslng
end record

*/
*[

DO;
in LogRec. Tran_Table

IF Trans ID NOT IN Trans_Table
Insert
entry (Trans ID, State,
ENO;
END; /*

Begln_Chkpt

by this

It

WHEN(‘End_ Chkpt’)
FOR each entry

checkpoint’s

to

00;

THEN 00;
Last LSN,UndoNxt LSN) In Trans

FOR ‘/

FOR each entry in LogRec.Dirty
PagLst 00;
IF Pagel Ll NOT IN Olrty_Pages-THEN
lrsert
ELSE set RecLSN of Dlrty_Pages
END; /’ FOR ‘/
END; /’ WHEN(’End Chkpt’)
*/
WhEN( ‘prepare’
\ ‘rollback’)
DO;

entry

entry

(Page IO, RecLSN) In Olrty_Pages;

to Rec LSN In Olrty_PagLst;

IF LogRec. Type = ‘prepare’
THEk Trans_Tabl e[Log Rec. Transit].
ELSE Trans Table [LogRec .Trans ID]. State
:= ‘U’;
Trans_Tabl~[LogRec

.TransID]

bac<’)
entry

WHEN(‘OSfile_return’)

from Olrty_?ages

delete

ENO; /* FOR */
RedoLSN := minimum(Di rty_Pages.
RE-URN;

processing

redo

updates

Another

pass

be

chkpt

6.2

Redo Pass

The

second

pass

of the

pass.

Figure

11

describes

TransID
all

:= ‘P’ ;

= LogRec. Trans ID;

pages of

/*

to files

used

begin_

which

Pseudocode for restart

consequence

cannot

*/
for

Rec LSN) ;

Fig. 10.

avoid

State

.Last LSN := LogRec. LSN;

ENO; /’ WHEN(’prepare’
I ‘roll
WHEN(‘end’)
delete Trans_Table

system.

Table;

which

is that

have

returned

return

file;

start

been

returned

dirty

.pages

table

update

log

records

which

that

is made

during

the

RESTART.REDO

filter

for

~edo *I

analysis.

the

to

posltlon

to the
used
occur

operating
during

the

after

the

is the

redo

record.

log

restart
routine

ACM 11-ansact,ons on Database Systems, Vol. 17, No. 1, March 1992

recovery
that

implements

ARIES: A Transaction Recovery Method

.

125

Di rty_Pages);

RESTART-REDO(RedoLSN,

/* open log scan and :;s]tlon
at restart
pt *J
/* read log record a: restart
redo point
*/
/* look at all records
till
end of log */

Open_ Log_Scan(RedoLSN);
LojRec := Next_ Logo;
WHILE NOT(End_of_Log) 00;
IF LogRec. Type = (’update’

I ‘compensation’)

& LogRec is

redoable

&

LogRec. PageIO IN Oirty-Pages
& LogRec. LSN >= Oi rty_Pages[LogRec
.~ageID]
THEN 00;
/’ a redoable
page update.
updated page mg-t
not
/* disk before
sys failure.
Page := fix&l atch(LogRec. PageIO, ‘X’);
IF Page. LSN < LogRec. LSN THEN 00
Redo_Update(Page,

need to access
/*

update

not

.Rec LSN

have made It to */
cage and check Its LSN */

or cage.

need to

redo

It

*I

/’

redo

update

*/

[*

redid

update

*I

LogRec);

Pag.?. LSN := LogRec. LSN;
END;
ELSE Dlrty_Pages

[LogRec. PageIO] .Rec LSN := Page. LSN+l;

.~date already
on page *I
update dirty
page list
with correct
info.
tr-s w1ll happen if this
*/
~~gewas written
to disk after
:Re checkpt b.t before sYs failure
*/

/’
I*
unfix&unlatch

/’

(Page);

ENO;
LogRec : = Next_ Log ();

/“

LSN on ~age has to
/a read next
/*

ENO;
RETURN;

Fig. 11.

the

redo

pass

actions.

the

dirty-pages

table

records
log

are

records

from

a check

table.

If

does

and

for

the

page

might

be
this

than

such

log

point.

When

to see if the
the

log

record’s

in

the

table,

the

log

record’s

the

record’s

page
LSN,

the

reestablishes

the

database

performed

by

loser

updates

behind

this

repeating

some

of that

redo

[691 we

have

Since
table

reduce
redo

may

get

the

the

number

modified

table

pages

that

are

read

were

dirty

at

the

have

Because

further

been

be

may

that

were

written

to

and

such

log

records

update

is redone.
which

as of the
are

of restricting

the

RecLSN

to be examined.

time

of system

redone.

The

rationale

10.1.

It turns

out

in Section
records

Thus,

have

To
to be

may

be

the

repeating

failure.
that

unnecessary.

In

of history

during

this

in

dirty-pages

the

redo

and

examined

redo.

Only

log

is because

nonvolatile

storage,

can

to

and

records

the

pages

listed

in

pass.

Not

all

some

of the

pages

that

dirty

later

which

became
the

saving

some

that
the

pass.

this

before

although

eliminate

ACM Transactions

or

storage

volume
log

the

during

checkpoint

write

be used

pass.

This

last

to

redone.

is found

entries

nonvolatile

reducing

be

the

state

dirtied

to

like

to
LSN

to

get

of the

systems

page

have

equal

with

time

expect

the

which

the

written

of reasons

or

pages

require

do not

is encoundirty-pages

of pages

read

we

the

only

during

will

record
in the

page’s

log

scanning

that

might

log

idea

log

of pages

is explained

and
No

than

If the

state

RedoLSN

starts

appears

transactions

transactions’

is page-oriented,

dirty-pages

might

of history
of loser

explored

to possibly

number

the

suspected

update

then

the

*/

log

is greater
is

is accessed.

This

routine

to limit

pass

page
it

end of

routine.

a redoable

LSN

then

serves

are

redo

referenced

information
Even

routine

RedoLSN

till

*/
*/

redo,

restart-analysis
The

suspicion,

the

this

the

routine.

if

that

to
by

this

is made

it

resolve
less

by

the

inputs

supplied

written

tered,
RecLSN

The

Pseudocode for restart

reading

be checked
1og record

identify
that

system
CPU
the

option

corresponding

the
the

failure.
overhead,

dirty
is

pages

available

pages

from

on Database Systems, Vol. 17, No. 1, March 1992,

126

C. Mohan et al.

0

the

dirty

.pages

analysis

table

pass.

complete,
being

Even

when

those

such

records

if

a system

failure

written.

The

brevity,

we

log

records

were

in

a narrow

corresponding

pages

are

always

encountered

during

the

to

after

1/0s

be

written

window

could

prevent

them

from

will

not

get

modified

during

this

as to

how,

pass.
For
after

logging

of the

the

pending

actions

redone

during

of all
are

For

the

exploiting

dirty

..-pages

parallel

table

in

dirty

the

log.

via

6.3

Undo Pass

The

third

pass
pass

backups

[731.

input
is

page

is

consulted

performed

or

to

Contrast
that

like

DB2

The

restart

-undo

logical

order,

in

do not

routine

a single

the

maximum

of the

LSNS

the

yet-to-be-completely-undone
to be undone.

The

in

routine

writes

protocol

undo

pass.

while

writing

pass.

the

LSN

initiated,

operation

perform

selective

transactions,

This

log

record

5.2.

CLRS.

the
the

buffer
to

for

redo.
chronotaking

to be processed

for

until

transaction

for

encountered
In

be

10.1

reverse

no loser
each
log

process
manager

nonvolatile

ACM TransactIons on Database Systems, Vol. 17, No. 1, March 1992

each

transaction
table
records

of

the

should

continually

transaction

The

pages

by

Also,
on

Section

in

is done

transactions,

of the

dirty

transaction

undo

but

next

Section

restart

this

history

in

before

implements

in

log.

undo

is the

describe

to process

described

the

These
disaster

that

we

entry

as

recovery

routine

undo

a given

supporting

what

of the

processing

order
of

is

losers

to

for

undo

back

Updates

as before.

is

an

pool,

processes.

since

routine

whether

as the

buffer

properties

restart

pass

and,

the

same

UNDO

record

The

WAL

during

basis

represented

the

an

log

information

order

context

next

transactions.
this

the

by

those
we

in the

this

loser

back

can

of

process.

the

redo

queues
by the

one

from

during

of the

determined

only

consulted

repeat

rolled

transactions,

is

by

the
we

multiple

orders

buffers

logged,

using

queues

the
in

not

into

to

with

are

come

not

rolls
sweep

encountered

of pages

made

this

in

in

in-memory

correctness

determine

not.

systems

remains

before

the

(as dictated

with

to

in
1/0s

pages

is

table

is repeated

occur

information

group

RESTART_

The

history

to

asynchronous

or

the

_pages

since

actions

and

that

describes

actions.

dirty

building

reapplied

applicable

The

pass

different
any

are

log

are

redo

record

also

table.

not

violate

updates

of the

12

log
in

are

pending

be available

records

page

be dealt

not

execution

the

the

complete

queue

remote

Figure

a per

1/0s

applied

does

ideas

undo

on

may

to be reapplied

get

missing

the

remaining
of

they

like

each

may

This

its

parallelism

the

table)

that

pages

recovery

need

corresponding

requires

different

were

before

of initiating

during
things

initiated

the

availability

log

performed

.pages

processing

pass.

so that

potentially

asynchronously

all

pages

sophisticated

which

the

possibility

corresponding

records

page

us the

the

perform

This

the

gives

before

also

transaction,

a failure
but

pass.

parallelism,

updates

if

of a transaction,

of that

redo

these

Since

here

record

all

pass.

the

discuss
end

to read

possibly

in

do not

the

of

to be

for

each

of

is exactly

rolling

back

follows

the

usual

during

the

storage

the

ARIES: A Transaction Recovery Method

.

127

.
REST,.4//T-UMM(T rans-Tabl

e);

WHILE EXISTS (Trans with

State

= ‘U’

UndoLSN := maxlmum(UndoNxtLSN)
/’

Trans_Table)

DO;

from Trans_Tab7e

in

entries

pick

UP

LogRec := Log-Read (UndoLSN);
SELECT(LogRec. Type)
WHEN(‘update’)
DO;

with

State

= ‘u’ ;

UndoNxtLSN of unprepared
trans with maximum UndoNxt LSN */
J* read log record to be undone or a CLR *J

IF LogRec is undoable THEN 00;
f’ record needs undoing (not
Page := flx&latch(LogRec
.Page IO, ‘X’);
Undo_Update(Page, LogRec);
Log_Wri te(’compensati
on’ ,LogRec .Trans ID, Trans_Tabl e[LogRec. TransID]
LogRec. Page ID, LogRec. PrevLSN,
Page. LSN := LgLSN;

/’

store

I* write
CLR */
CLR in page */

LSN of

*/

Log_Wrlte( ’end’ ,LogRec .Trans IO, Trans_Tabl e[LogRec. Transit].
delete Trans_Table entry where TransID . LogRec. TransIO;
ENO;

table

*/

undone

*/

/* pick UP addr of next record to examine
e[LogRec. TransIO] .UndoNxtLSN := LogRec. PrevLSN;

*/

[ ‘ prepare’)

Trans_Tabl

.UndoNxtLSN

I*
ENO; /*
/*
END;
RETURN;

To exploit
processes.
single

parallelism,

the

It is important

that

process

leaves

open

undos

to

because
the

the

may

require

parallel,

as

actually

applying

a single
Figure

possibility
(see

that

explained

trans

from
fully

*/
*I

:= LogRec. UndoNxt LSN;

UP addr of

next

record

to examine

*I

be performed

using

be dealt

with

completely

by

the

CLRS.

This

still

applying

the

chaining

of writing

CLRS

first,

problems

in

logical

undos),

and

then

Section

6.2.

this

fashion,

can

be performed

changes

the

in

for

6.4

to the

In

pages

an example

recovery

updates

to the

written

to

after

second

was

transaction

went
updates

missing

and
one

ARIES,

without

multiple

accomplishing

a

this

for
in

redoing

the

CLRS

the

undo

work

(undo

update.

of log

in parallel,
using

Before
After

records

of
even

6).

During

first

redone

performed.

update

log

have

and

the

savepoint

Each
of how

the

many

option

recovery
concept,

times

of

allowing

is completed.
we

restart

could,

in the

write,

3)

and

then

restart

will

the

a
the

recovery,

then

the

undos

be matched

with

is performed.

continuation
ARIES

undo

the

disk

recovery

Since

Here,

failure,

that

and

record

ARIES.
the

4 and

and

restart

5

page.

6) are

after

(updates

scenario

same

(3, 4, 4’, 3’, 5 and

regardless
we

the

performed

forward

1) are

CLR,

disk

transactions
supports

restart

describe

the

With

also

UndoNxtLSN

records

rollback

(of 6, 5,2

can

transaction

transaction.

was

partial

pass

each

Section
in

the

13 depicts
log

at most

I*

*I
*I

Pseudocode for estart undo.

undo

of the

pages

objects

the

pick

LastLSN, . . .) ;
/* delete
trans

‘/

SELECT “/
WHILE */

Fig. 12.

page

*I

Trans_Tabl e[LogRec. TransID] .LastLSN := LgLSN;
/’ store LSN of CLR in table
unfix&unl atch(Page);
ENO;
I* undoable
record case
ELSE;
/* record cannot be undone - ignore it
Trans_Tabl e[LogRec. Trans IO] .UndoNxt LSN := LogRec. PrevLSN; /x next record to process is
J* the one preceding
this record in its backward chain
IF LogRec. PrevLSN = O THEN DO;
/* have undone completely
- write
end

WHEN(‘rollback’

all

record)

.LastLSN,

. . . ,LgLSN, Data);

ENO; /* WHEN(‘update’)
*/
WHEN(‘compensation’)
Trans_Tabl e[LogRec. TransID]

for

redo-only

pass,

repeats
roll

of

loser

history
back

each

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

128

C. Mohan et al.

●

u

Wrl te !bdated

m

*
12344’3’5

REDO

344356

UNDO

6521

Fig. 13.

loser

only

to

its

transactions.
tion

at

latest

Later,

a

savepoint

could

entry

execution
to

(1)

the

ability

records

for

its

uncommitted,

ever

completing

application

are

new
some

recovery

the

amount

accomplished
the

failure,
as

undo

work

of

undo

offline

objects,

new

then

DB2
This

solely

on the

information

of

of the

smallest
the

table)

that

offline

objects

made
with

to protect
until

need

are

is usually

critical

data

and

then

DB2,

some

for

of the

is

done

example,

it

for

is

which

the

CLRS

alone

because

the

non-CLR

[151.

That

is,
log

undo

will

no
the

finish

logical

for
be

in

in virtual

storage,

the

they

are

brought

online,

[141.

The

LSN

Unless
to those

accesses

there

are

objects,
to those

those

ranges
some

no locks
objects

objects

are

on Database Systems, Vol. 17, No 1, March 1992

DB2.

DB2

not

brought

(DBA)
that

those

before
of log

in-doubt

need

will

fact

is

inverses

allocation

and

remembered.

based
forward

indexes)

exact

undos

database

up.

handling

the

when

transactions

When

minipage,

are

to

and/or
on those

during

(or

(called

possible

be generated

actions

is
for

is brought

and

can

written

page

the
there

CLRS

records

Because

table

in the

completed.

the

It

system
redo

is able

to write

to reduce

the

transactions

is possible

of
doing

unavailable.

opening

objects

defer

system

in

offline

processing
to

the

since

ACM Transactions

This

the
wish

loser

those

objects

time.

may

when

updates

is

whenpositions,

some

uncommitted

recovery

locks

cursor

for

to other
also

log

those

information

restore

to restart
we

In

to be recovered

accessible

wish

first

are

exceptions

is maintained

to be applied
tions

an

the
would

transaction’s

enough
can

Hence,
in

of locking,

actions.
in

may

data

when

transactions

original

the

to be performed

granularity

remembers,

are

such
even

needs

about
correctly

(2) reacquiring

system

some

to be performed

work

transactions.

the

point
which

transactions.

recovery

the

processing

we

possible.

a later

recovering

needs

If some

soon
during

restart

this

from

(3) logging
the

information

loser

applica-

so on.

as

to

processing

perform

and

the
its

Restart

time

by

names

back

invoking

Doing

updates,

so that

rolling
by

enough

lock

undone

and

work
of

passing

a system

transactions

totally

is to be resumed.

recovery,

state,

after

and

not

or Deferred

Sometimes,

transaction

established

program

Selective

6.4

of

the

generate

restart

savepoints

instead

resume

point

which

require

before

recovery example with ARIES.

savepoint,

we

special
from

Restart

they

records
transac-

to be acquired
be permitted
online,

then

ARIES:

recovery
the

is performed

remembered

for

offline
In

transactions

also,

we

has

modified

undos.

This

the

object.

Redos

For

logical

undos

we can

take

example,
for

the

forward

normal

Method

using

rollbacks,

the

.

log

CLRS

129

records

maybe

in

written

similar
logical

at all

undo

management
and

insert

stating

undo

that

page

maybe

page-oriented

the

state

10.3),

is O% full.

of

page-oriented.
generally

appropriate
we

loser

require

current

Section

the

of [62]

may

always

operation,

the

of

that

on the

are

(see

retraversing

page

when

based
they

record

none

objects

generate

methods
(e.g.,

of which

predict

are

since

space

of an

provided

offline

undos

a problem,

approach

update

logical

actions,
of the

CLRS.

can

write

But

for

For

a CLR
the

high

this

is not

possible,

since

the

the

index

tree

do

a

key

in fact,

we

affected,

is unpredictable;

undo

not

will

work

to

and

hence

logical

is necessary.

It is not
during
the

rolling

or more

management

in terms
even

undo

not

the

index
the

deletion),

take
one

involving

during

of

can

is because

are

space-related

cannot

during

a conservative

concurrency,
effect

by

Even

Recovery

objects.

ARIES

logical

efficiently

ranges.

A Transaction

possible

restart

records

to handle

recovery

at a later
that

in

chronological

reverse

each

transaction,

that

record,

other

records

Even

in

all

the

the

the

point

Remember

of some

of the

handle

the

undos

(possibly,

in time,

if the

two

and

the

undos

recovery
order.

next

PrevLSN

methods,
Hence,

record

to

and/or

sets

the

be

is

of a transaction

logical)

of records

undo
it

records

of the

are

of a transaction

enough

processed

to

during

rest

is

done

remember,

for

undo;

from

leads

us to all

the

of the

loser

transactions

offline

objects,

UndoNxtLSN

chain

the

of

interspersed.

to be processed.

under

the

circumstances

where

have

to perform,

potentially

logical,

restart

needs

to be supported,

then

one

undos
we

or more
on some

suggest

the

following

if deferred

algorithm:

it for
1. Perform
the repeating
of history
for the online objects, as usual; postpone
the log ranges.
the off/ine objects and remember
2. Proceed
with
the undo pass as usual,
but stop undoing
a loser transaction
when
one of its log records
is encountered
for which
a CLR
cannot
be
generated
for the above reasons. Call such a transaction
a stopped transaction.
But continue
undoing
the other, unstopped
transactions.
3. For the stopped transactions,
acquire locks to protect their updates which have
not yet been undone.
This could be done as part of the undo pass by continuing
to follow the pointers,
as usual, even for the stopped transactions
and acquiring locks based on the encountered
non-CLRs
that were written by the stopped
transactions.
4. When restart
recovery
is completed
and later the previously
offline
objects are
made online, fkst repeat history
based on the remembered
log ranges and then
continue
with
the undoing
of the stopped
transactions.
After
each of the
stopped transactions
is totally
rolled back, release its still held locks.
5. Whenever
an offline
object becomes online,
when the repeating
of history
is
completed
for that object, new transactions
can be allowed
to access that object
in parallel
with the further
undoing
of all of the stopped transactions
that can
make progress.
The

above

tion

in

in-doubt

requires

the

update

the

ability

(non-GLR)

to generate
log

records.

lock

names

DB2

is

based
doing

on the
that

informa-

already

for

transactions.
ACM Transactions on Database Systems, Vol

17, No, 1, March 1992.

130

C. Mohan

.

Even
the

if none

of the

processing

of

transactions

et al.

are

objects

new

completed,

ing:

(1)

first

repeat

locks

for

the

uncommitted

(2) then

start

are

new

as each

loser

requires

that

restart

the

log

records

pass.

If a loser

failure,

then,

are

during

the

the
locks

updates
back

as soon

as possible,

then

we

the

first

update

if record

as

the

we

do not

undo

once;

hence,

CLRS
it

and

will

AS/400,

DB2)

(e.g.,

This

release

early

normal

transaction

partial

rollbacks.

to

of restart

recovery

Analysis
can

save

of the

By
work

transaction

the

transaction

of

the

on

do not

undo

work

in

systems

that

the

been

undone.
records

corresponding

the

that

object’s

works

same

undo

CLRS

a

non-CLR

more

than

of locks

can

be

performed

in

the

optionally,

impact
taking

of

only

non-CLR

undo

resolution

the

some

log

This

(e. g.,
once

ARIES

during

deadlocks

using

of failures

on CPU

processing

checkpoints

during

different

taking

at the

of the

a checkpoint

if

a failure

were
checkpoint

table

of this

table

at
of

the
this

end

end

and
stages

analysis

to

occur

during

recovery.

will

be the

same

of

the

analysis

list

restart

dirty-pages

table

checkpoint

This

is

different

from

what

happens

during

.pages

list

is obtained

from

the

buffer

pool

redo

pass,

the

buffer

contains

pass,
The

as the

pass.

dirty–pages

dirty

obtained

release

the
the

to

be

those

that
latter,

equal

to release

specially

transaction

such

processing.

pass.
some

mark

yet

like

we

how

by,

for

to

not

would

because

describe

we

be reduced

have

we

or

need

undone.

In this

section,

that

than

is

RESTART

redo

system

pass

record

DURING

can

the

of the

to be undone.

less

permit

that

remain

log

possibly

(1)

ensure

analysis

then

7. CHECKPOINTS

1/0

time

are

step

that

or

undo

to

loser

(1)

during

and

not

step

the

Locks

that

of the
in

records

are

can

the
and

Performing

is in effect)

corresponding

Encompass,
IMS).

by

locking

rollbacks

at the

CLR.
and

follow-

transactions,

encountered
back

log

LSNS
last

rolled

soon

than

whose

records,

acquired

during

those

represent

more

obtained
as to which

transaction’s

the

log

appropriately

rolling

loser

the

doing

as the

are

already

that

of

their

completes.

transactions

was

is desired

it by
in-doubt

locks

adjusted

for

(e. g., record,

because

be

is being

object

as

loser

only

even
The

it

rollbacks

on

and

rollback

RedoLSN

records

pass

loser

but

the

based

transaction

which
lock

accommodate

of the

information

the

redo

can

reacquire,

parallel.

be known

log
of

If a long
of its

the

it will

UndoNxtLSN

before

transaction’s

of the

with

a transaction,
These

in

transaction

is offline,

start

transactions

performed

the

we

and
updates

released
all

then

history

processing

transactions

to be recovered

transactions

entries

of

The

entries

the

entries

will

be

the

same

as

at

the

end

of the

analysis

a normal

we

entries

checkpoint.
(BP)

pass.
For

the

dirty-pages

table.
Redo

pass.

At

the

beginning

notified

so that,

whenever

during

the

pass,

that

page

redo
by

ACM Transactions

making

it
the

of the

it writes
will

out

change

RecLSN

a modified
the

be equal

restart
to the

manager

page

to nonvolatile

dirty

_pages

table

LSN

of that

log

on Database Systems, Vol. 17, No. 1, March 1992.

(BM)

is

storage
entry
record

for
such

ARIES: A Transaction Recovery Method
that

all

BM

manipulates

log

records

up

to that

the

restart
own

have

to maintain

its

ing.

Of course,

it should

the

buffers.

redo

pass

The
to

a failure
the

reduce
to

dirty-pages

of
the

to

the

amount

of the

before

the

of

this

dirty–pages

entries

of

the

table

transaction
not

log

end
at

table

affected

by

The

be

the

same

as

time

of

checkpoint
the

whether

or

not

parallelism

of the

undo

pass,

table.

At

point,

then

does

during

etc.

During

during
then

If a checkpoint

of the
BP

logic

or redo

since

a restart

work

view

complex

date

checkpoints

case,

they

may

will

such

called

a

is cleaned

are
this

table

are

written

pages
to

become
are

modified

the

undo

checkpoint

are

the

of the

checkpoint.

will

be the

same

checkpoint

to
dirty,

table

time

in
as it

during

of that

up

no longer

any

recovery,
up

sometimes

some

physical

to be performed.

depicted

in

This

Figure
The

a system

failure
in

as
pass,

same

as the

The

entries

as the

entries

17

would

of

While

these

to take

place

in

that

consequence

complicates
no

a

pages)

for

of the

the

restart

longer

be

true

after

checkpoint

logic

and

its

effect

an earlier

restart

ARIES

restart.

shadow

is another

during

during

be required

(the

R. This

restart
[31].

it may

pages

in System

completes.

is able

checkpoints

System

were

to easily
are

consid-

accommo-

optional

in

our

or

R.

RECOVERY
that

(like

media

DBspace,

tions.

With

might

contain

archive

such
if

will

entity.

A

operation

involving

with

modifications

to

concurrency

uncommitted

image

updates,

desired,

we

could

also

easily

uncommitted

updates.

Let

us

assume

that

directly

the

from

be required

etc.)

dump)

a high

some

recovery
tablespace,

concurrently

course,

dirty-pages

table

transaction

be forced

fuzzy

performed

restart

of the

at the

to be describable

assume

some

in

is taken
list

be repeated

following

too

free

employed

about

time

as

is

manipulates
when

same

This

pages

are

The

the

pass.

the

pages

entries

analysis

the

entries

the

time.

restart
to

cannot

the

on a restart

8. MEDIA

of this

at that

checkpoint

ered

We

table

taken

when

entries

.pages

table

history

the

dirty

R, during

undo

entries

dirty–pages

table
be

that

manager

pass,

transaction

more

BP

undo

transaction

fact

the

undo.

entries

System

onward,

the

of the
In

corresponding

adding

of the

checkpoint

the

processing–removing

entries

the

which

normal

normal
the

for

storage,

nonvolatile

this

if
of

be

of

beginning

redone
entries

checkpoint.

end

dirty-pages

be

the

the

the

process-

will

at

BP

From

the

to

At

buffers.

during

need

the

the

time

pass.

pass.

entries

in

any

redo

becomes

those

currently

would

Undo

by removing

normal
are

the

pass.

not

pages

that

redo

if

does

during

taken

the

this

131

is enough
BM

be

will

of

It

fashion.

of what

of

checkpoint

table

processed.
this

as it does

checkpoints

transaction

table

Of

table

allow

the

is

in

track

restart

checkpointing
the

been

table

be keeping

occur
list

had

still

of

the

entries

record

dirty--pages

above

were

log

dirty-pages

.

nonvolatile

storage

version

in

at the

level

of a file

fuzzy

image

copy (also

such

an

entity

can

the

entity

by

other

transac-

copy

method,

contrast

produce
the

image

of the

an

the

image

be

copy

to the

method

image

copy

with

copying

is

performed

entity.

This

of [52].

means

no
that

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

C. Mohan et al.

132

.

more

recent

versions

transaction

of

system’s

version

of the

geometry

can

manager

overheads

have

be

up

via

to

copying

some

of

buffers.

object

would

usually

be exploited

copied

pages

may

be

directly

from

the

nonvolatile

be

during

will

for

the

Copying

such

be eliminated.

the

direct

much

more

a copy

operation

Since

copying,

it

the

may

efficient

be

the

transaction

system’s

buffers.

If the

support

incremental

image

copying,

as described

easy

to

modify

case,

some

latching

minimal

at the

When
begin.

amount

page

to

of synchronization

level,

but

no locking

fuzzy

image

copy

operation

is

record

of

most

recent

complete

the

image

copy

The

assertion

along

with

copy checkpoint.

point

information

with

LSNS

is

less

image-copied

that

than

externalized

to

in

the

record

of

began.

Hence,

up

to date

as of that

redo point.

The

the

in

computing

given

in

Section

the

image

image-copied
point

in

for
the

5.4

log.

We

into

taking
media

while

the

course,

in

be needed.

For

example,

the

location

us call

is

the
and

this

checkpoint
on this

check-

in

records

been

logged

SNs

of

dirt

y

the

log

pages

of

end.chkpt

the

record),

would

have

been

image

copy

opera-

of the

fuzzy
entity

would

be at least

media

point

the

account

the

LSN

of the

redo

point

the

of

noted

based

that

discussing

is

the

call

recovery

it
that

checkpoint))

time

version

the

Of

checkpoint’s

copy

by

then

be made

had

copy

storage

reason

record

Let

can

that

image

nonvolatile

tion

updates

desirable

[131),

checkpoint

minimum(minimum(RecL

entity

LSN(begin_chkpt

all

not
than

is found

initiated,

data.

that

buffer

does

be needed.

the

the

device

in
it.

will
will

the

convenient

latter

accommodate

chkpt

remembered

image

method

the

system

more

to

presented

the

since

transaction

also

in

storage

since

and

(e.g.,

the

present

is the

computation

begin.

same

of

as

recovery
chkpt

as the

the

one

restart

redo

point.
When

media

reloaded

and

redo

point.

being

recovery
then

During

recovered

is required,

a redo
the
are

redo

image-copied

scan,

all

version

starting

the

log

records

the

corresponding

in the

image

copy

checkpoint

the

information

LSN

on the

page

a log

record

refers

to

makes
a page

that

is not

the

recovery

relating

to

updates

are

record’s

Unlike

in

entity

media

dirt

than

the

LSN

of the

begin–chkpt

checkpoint,

then

that

page

must

be

accessed

pared

to the

log

LSN

to check

update

must

of the

log

transactions
pass

is reached,
that

of restart

transactions
such

as
the

DBA

table

end

analysis
of the

an
page

ARIES,

arbitrary
needs

ACM Transactions

any

in-progress

to the

separately
in

DB2—see
from

last

if

the

log

record
its

of the

LSN

be redone.

transactions,

com-

Once
then

are

undone,

as in

the

the

identities,

etc.

of

(e.g.,

in

6.4)

or

complete

an

exceptions

may

be

the

those

about
Section

the

log

and

list

redo,

entity

somewhere

pass

undo
such
table

obtained

by

in

log

checkpoint

the

log.
provides

every

database

recovery,

if the

information

logging

database

are

changes

The

be kept

an

in

made

may

Page-oriented
Since,

had

if there

recovery.

the

performing
until

record’s

entity

.pages

list
and

is

applied,

restart

y_pages

copy

the

dirty

during

image

end

is greater

it unnecessary.

of the

the

from

and

or the

LSN

the

is initiated

processed

unless

record’s

scan

page
the

is

recovery
page’s
can

amongst

objects.

is logged

separately,

in

nonvolatile

storage

and

by

extracting

damaged

recovery

independence

update
be

the

accomplished

on Database Systems, Vol. 17, NO 1, March 1992

easily

even

if
the

ARIES: A Transaction Recovery Method
an

earlier

copy

version

of the

with

systems

index

and

from

damage

log

as described

like

System

R

in

which,

management
such

from

the

image

determine
rolled

back

would

be

would

actions,

partially

if
or

undone.

work

being

performed,

if it

not

made

any

log

records,

changes

(see

Individual
media

Section

10.2

pages

of

problems

but

and

the

process

is actively

before

the

process

gets

database

code

page

System

making

executed

by

terminations

may

occur

because

of the

key)

or due

to the

operating

process

had

operation

to

update.

Given

rupted

page

volatile

put

the

all
is

[151.

The

The

bit

in

because

termination

in the

buffer

record

describing

the

process
implement,

user’s

interruption

system’s
limit.

action

It

is

pool

and

changes.

itself,

which

such

abnormal

(e.g.,

by

on noting

generally

is

hitting
that

an

the

expensive

cor-

version

of the

page

from

the

non-

uncorrupted

bring

log

it

records

up

to

for

that

from

the

RecLSN

does

this

kind

’1’

forward

roll-forward
for

the

recovery

is detected

the

page

is fixed

and

(i. e.,

page

updated,

update

is tested

bit

‘O’.

recovery

to bring

situation

were

in the

version

of the

page

on nonvolatile
that

were

letting

restart

page
storage.
left

in

ACM Transactions

but

an

availability

A related
the

system

recovery
were
fixed

state
of the

by

the

buffer

the

page

header.

Once

the

update

and

page

a page
is equal

From

corrupted

in

value

transaction

page
scan

automatically

logged

whenever

to see if its
entire

a bit

redo

missing

by

LSN

is latched,
to ‘l’,

for

in which

viewpoint,
to recover
all

in the

problem
state

page

the

buffer

X-latched.

the
by

that

this,

using

is initiated.

down

updates

Given

by

every

redo

operation

of a page

this

pages

rolling
The

after

first

page

by

page,

internal

to

page

date

efficient

remembered
of

is reset

those

only

not
process

to a page

DB2

rolled

of restart

the

bit

for

corrupted

abnormal

over

pass

read

the

that

analysis

to

modified),

a broken

alternative

the

complete

such

be

An
to skip

recover

is

automatic

recovered.

in

before

operation

is unacceptable

transac-

to

to

case

back

state

set

or write,

rolled

some

way

corruption

read

that

an

and

an

being

result

uninterruptable

DB2
is

process

page

may

application

time

transactions

the

circumstances,

is

started

CPU

such
to

these

relevant

manager.

its

of

to
had

all

storage

using
log

exhausted

records

rollback)

scans

a log

the

log

to the
total

backward

an

like

starting

scans

the

the

systems

written

by

transactions

being

changes

to write

not

any

R during

of

when
logging

If

pointers

may

because

which
are

or

forward

18).

of reconeven

to date

attention

changes

out

place

up

partial

any

Figure

performance-conscious

attention

if CLRS

turns

a chance
is

R),
state

database

also

while

System

a page’s

These

to the

in

the

for

made

and

as it is done

pages

backward

are

log

for

(e. g.,

recovery

operation

even

undone.

that

updates

index

paying

forward

written,

expensive

(commit,

they

pages’
not

complete

in

133

is to be contrasted

the

then

they

the

are

be

totally,
if

some

should

that

so

be to preprocess

the

any,

rolling

This

for

the

require
state

see

recovery

If

pages

and

records

Also,

bringing

to

had

what

data

state

required

recovered

back

require
rebuilding

transaction’

what

since

may

then

copy
above.

log

(e. g.,

(e.g.,

copy
the

image

is damaged).

is performed,

representing

useless

a page

explicitly

undo

an

pages’)

object

of an index

is performed
when

of

the

entire

one page

tion

from

using

to

the

would

of that

space

structing
only

page

page

.

those

logged

uncorrupted

is to make
the

it
from

sure

abnormally

on Database Systems, Vol. 17, No. 1, March 1992.

134

C. Mohan et al.

.

terminating

process,

leaving
and

latch,

unfix

calls

footprints

enough
the

user

are

around

process

issued

before

aids

by

the

transaction

performing

system

processes

system.

By

operations

like

fix,

unfix

in performing

the

necessary

clean-ups.
For

the

CLRS

variety

This

good

only

page

9. NESTED

TOP

committed,
not.

with

when

do need

illustrated

in

the

may

the

be allowed

transaction.

If the

would

like

of

whether

some
the

of file

the

data

extended

area

the

effects

if the

of the

performed

by

transaction

initiating

starting

is,
and

In

ARIES,

the

of course,
the

transactions

poses,

is taken

an

concept
very

to mean

should

not

which

is

dependent

undone

irrespective

of the

once
on

execution

nested

top

consists

action

ascertaining

the

position

the

redo

and

nested

top

action;

on

completion

step
We

of

the

sequence

nested

top

transactions.

traditionally

data

until

the
be

initiating

we are able
to initiate

to support
indepen-

top

action,

of actions

of

a transaction

is

and

complete

for

some

is logged

inde-

unacceptable.

nested

action

[511. A

transaction

between
would

it

that

independent

having

in the

top actions
waits

conflicts

enclosing

to

our

purwhich

later

action

stable

storage,

which

define

transaction.

a sequence

following

of the

undo

very

been

have

top action,

subsequence

might

completion,

The

A

not

undo

their

of a nested
actions.

it would

before
called

without

extending

then

an

transaction

to lock

transactions

of the

system

actions

is

a file

extends

committed

a failure

or

This

to the

which

of the

logging

Such

transaction,

of the

(1)

of actions

a

steps:

current

transaction’s

information

last

associated

with

log

the

record;

actions

of the

and

of

UndoNxtLSN

back,

proceeding.

performing

(2)

(3)

to roll
other

to be

commits

other

were

transactions,

the

the

outcome

transaction

database,

updates

efficiently,

any

A

in the

commit

kinds

before

to perform

be

[521, which

themselves.

to the

by

vulnerable

the

a transaction

independent

independent

requirement

dent

in

a transaction

of

After

by the

independent

commits

using

above

page

ultimately

extension.

performed

such

transaction

mechanism

locking.

updates

extension-related

were themselves
interrupted
to undo them,
These
is necessary

updates

prior

transaction

of updates

hand,

transaction

writing

only

these

extension.

database

pendent

elsewhere,

suggested

transaction
for

to some system

to undo

other

approach,

y property

extending

to a loss

lead
the

we

context
to use

be acceptable
well

no-CLRs

atomicit

causes updates

which

the

and

is supporting

locking.

irrespective

We

section

in this
system

ACTIONS

are times

There

mentioned

even if the

idea

is to be contrasted

supports

On

of reasons

is a very

the

nested

top

points

to the

log

the

effects

of

action,

record

writing

whose

dummy
CLR whose
was remembered
in

a

position

(l).

assume

associated

that

updates

to system

any

data

actions

normally

like

creating

a file

resident

outside

the

externalized,

before

the

dummy

CLR

is written.

are

to only

the

system

data

that

referring

ACM Transactions

on Database Systems, Vol

When

is resident

17, No 1, March 1992,

we
in

the

and

their

database

are

discuss
database

redo,

we

itself.

ARIES: A Transaction Recovery Method

.

135

*
Fig. 14.

Using
roll

this
back

nested
after

will

ensure

that

not

undone.

If

written,

then

nested

top

redo-only)

top

the

action

top

the
the

log

action.

records

Unlike

CLR

is

sense

be thought

for

of as the

advantage

of our

approach

to

forced

record

quent

be

actions.

Nor

do we

costly
Figure

we

do not
conflict

14 gives
5. Log

transaction’s
back,

It

should

then

we

writing

an

example
6’ acts
is

6’ ensures
be

on repeating
can

log

the

that

of a hash-based
62].

[59,

10.

RECOVERY

This

section

(e.g.,

additional
our

include
of

existing

goals

and

in ARIES.
R,

record

undo-redo

(as

opposed

atomicity

property

is nothing

pass.

The

for

enclosing

the

price

to redo

top

action.

need

proceeding

of starting
Contrast

not

with

in

a

The

wait

its

a new

this

to
the

when

CLR

nested

is
the

for

dummy

transaction

before

the

since

there

redo

problems.
top

dummy
by

nested

that

the

nested

for

subse-

transaction.

approach

action
CLR.

consisting

with

of the

Even

though

the

a failure

and

hence

top

action

is not

undone.

nested

top

top

action

action

it

the

of only

needs

a single

redo-only

log

of the

nested

action

and

index

top

record

be

update,
and

concept

management

to

relies

a single

using

method

actions

enclosing

implementation

consists

Applications

some
be

In particular,
were

problems
and

found

recovery

to motivate

which

of the

locking

can

of the

System

commit

pay

storage

record)

discussion

features

CLRS,
the

are

CLR

undone

as

desired

be

action

can

avoid
in the

be found

PARADIGMS

describes

granularity

will

normal

the

update

in

action

during

interrupted

CLR.

context

top

of a nested

If the

dummy

dummy

the

as the

that

emphasized

history.

top

the

to

CLR

approach.

record
activity

rolled

nested

before

storage

lock

were

dummy

of the

the

into

the

occur

stable

6 Also,

run

then

as part

is that
to

independent-transaction

3, 4 and

ing

the

transaction

action,

written

provides

encountered

this

are

enclosing

top

to

were
nested

This

the

nested

performed

failure

incomplete

action’s

if

of the

updates

a system

a dummy
can

approach,

completion

10g records.

nested

Nested top action example.

in

[97].

methods
the

need

for

Our

aim

in

the

is to

providing

fine-

rollbacks.

Some

show

us difficulties

certain

why

with

transaction

caused

we show
developed

associated

handling

some

features
of the

context

how

which
recovery

of

certain

in accomplish-

the

we

had

to

paradigms
shadow

page

6 The dummy CLR may have to be forced if some urdogged updates may be performed
other transactions which depended on the nested top action having completed.

later by

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

136

C. Mohan et al.

.

are inappropriate

technique,
high

levels

of

in the

have

been

algorithms

with

limitations

The

R paradigms

— selective

redo

— undo

work

— no

logging

WAL

In

paradigms
System

when

concurrency.
adopted

of

context

or

and

of WAL,

errors
of interest

[3,

15,

is a need

there

more

of

those

for

System

R

leading

to the

design

of

52,

72,

82,

881.

(i.e.,

no

16,

71,

78,

are:

recovery.

redo

updates

one

are

restart

preceding

is to be used

past,

and/or
that

during

the

work

during

performed

restart

recovery.

during

transaction

rollback

CLRS).
— no logging
—no

of index

and

of page

state

tracking

no LSNS

10.1

goal

of this

has

been

implemented

introduces

in

subsection
in

supporting

is to motivate

database

recovery

undo

pass

(see Figure

6).

redo

pass.

As

show

redo

is incorrect

DB2,

on the

will

We

call

this

discuss

WAL

technique

During

the

describing

if

record’s

the

log

redo

is less

of the

page.

undo

updates

that

is always

written,

to

way.

ACM Transactions

the

undo

prepared

While

the

the

and

an

and

then

the

undo

redo

preceding

WAL-based
pass,

System

in-doubt)

selective

approach

perform

pass

The

(i.e.,

it

recovery.

pass
of

During

and

Writing

DB2,

redo

transac-

paradigm

to take,

implemented.

page

contains

an

it

of

has

many

locking

and

inconsistencies

in

is compared

to the

LSN

to

whether

determine

page.
is

the

performed

the

page
not

the

CLR

when
also

page’s

on

then

not

it

an undo
when

is not

handling

on Database Systems, Vol. 17, No. 1, March 1992,

log

record
record’s

less

than

LSN

is

if the

page

the
set

undo

action

page.

Whether

page,

a CLR

are

to

LSN

no

contain

to handle

a

before.

on the
as part

actions

does

force

is

pass,

the

performed

transaction’s

and

LSN

is performed

consider

of a log
the

the
undo

us

described

page

and

to be undone,

been

simpler

to be necessary

the

During

undo

the

when

If

redone

record

have

Let
as

15).

actually

page

LSN

the

Figure

be

to data

be

update

would

only

lead

to

LSN

log

support

will

were

Otherwise,

even

out

redo

locking.

efficient
as

page

when

recovery

turns

a

that

that

generally

log:

R paradigm

opposite.

approach

to

the

L SN

page

such

each

than

in a special

System

redo

problems

they

the

performs

redo.

This

(see

is written,

of

fine-granularity
the

locking

then

the

the

[151.

LSN

media

the

to be the

record’s

CLR

R first

selectiue

be reapplied

not

The

failures,

of committed

page

needs

history.

after
passes

and

the

the

2

later,

to

or

make

in

the

on the

repeats

restart

pass,

to

locking

ARIES

which

ing
ation

WAL-based

update

LSN,

performed

with

System

record
in

needs

log

(i.e.,

below,

redo

an

the

systems,

selective
systems,

show

seems

WAL-based

such

of selective

to

does just

hand,
actions

as we

concept

and

WAL

the

pitfalls,

the

systems

with

other

R intuitively

update

updates

introduce

updates

only

[311.

perform

to logged

is to

systems

we

System
Some

changes.

it

many

why

transaction

tions

to relate

fine-granularity

When

R redoes

information

itself

Redo

The

aim

management

on page

on pages).

Selective

The

space

is

describ-

of the

undo

oper-

being

rolled

back.

the

update,

just

rolled

back

updates

actually

performed

a failure

of the

to
on

system

ARIES: A Transaction Recovery Method

T1 Is a Nonloser

Update

30
20

Fig. 15.

during

restart

Selective

recovery.

which

did

not

PI

which

had

to be undone,

Pi’s

LSN

being

changed

were

to be written

the

have

of this
the

update

the

if

U2’

page

It

should

locking

Given

these

by

the

subsequently
LSN

30

the

LSN

time

by

comes

to

or

not.

which

undo

the

Figures

and

fine-granularity
with

LSN

20

the

update

with

LSN

30 since

to

perform

the

in

the

page.

This

value

to

history,

since

whether
than

page-LSN

under

to a losing

the

page

20

by

(say,

modi-

T2)

was

update

with

would

have

the

loser.

So, when

not

if

its

update

needs

know
this

problem

pushed

with

the
to

be

selective

In

the

latter

scenario,

not

redoing

belongs

to

a loser

transaction,

but

redoing

of the

or

only

by

it belongs

or

On
any

method
respect

LSN

latter

it.
be

when

where

with

The

appear

not

even

update

illustrate

is because

is

greater

it

would

established

would
16

undo

determine

the

we

situation

if PI

interrupts

would

with

for

that,

to undo

WAL-based

update

redone.

locking.

pass

page_LSN

be

it

be made

page

U1

and

After

arises

transaction’s

value

and

update

the

for

written

failure

there

redo

in

being

restart,

of a page

(say,

to

15

redo

present

[15].

U2
update

of U2).

next

problem

state

Ul)

would

DB2

the

loser,

the

selective

earlier

a system

this

of the

had

beyond

LSN

before

with

a nonloser

page

for

that

transaction

by

(>

an update

an

(CLll

then

transaction

the
undo

the

track

was

written,

case

of

lose

losing

Tl)

been

scenario.

was

an attempt

emphasized

modified

of the

undone

had

or in-rollback)

first

U1’

during

and

as is the

would

in

of l.Jl’

then,
U2

properties

we

(in-progress
fied

be

there

storage

restart,

is used,

discussion,

LSN

if there

but

resulting

nonvolatile

completion

problem.

happen,

to the

to

hand,

will

to be undone,

as if P1 contains
other

redo with WAL—problem-free

This

PI

Redoes

137

Loser

T2 is a

UNDO Undoes Update

REDO

.

transaction,

former

even

the

not

an

equal

to

is no longer

to a nonloser
update

undo

logic

update
log

a true

relies

should

records

causes

the

it

not

though
on the

be

page_LSN

undone

LSN).

By

indicator

of the

current

is

not

present

for

example,

is

(undo

not

if

repeating
state

of the

page.
Undoing
harmless
oriented
DBMS

an

action

even

only

under

certain

and

logging,

locking
and

VAX

Rdb/VMS

space,

when

its
as

[81],

of freed

and

unique

data

inconsistencies

will

be caused

effect

is not

in

the

they

and

reuse

present

effect

conditions;
are

other

keys

for
by

implemented
systems

all

records.

undoing

an

in
with
in

[6],

there

With
original

a page

will

be

physical/byteIMS

[76],

VAX

is no automatic
operation
operation

logging,
whose

page.
ACM Transactions

on Database Systems, Vol. 17, No. 1, March 1992.

138

C. Mohan et al.

.

0T1

Vr! fe !Mated

~,

F“,2

10

20

I’Jq

,,

30

Commit

.
.

i

LSN

T2 is a Loser

T1 is a Nonloser

REDO Redoes Update
UNDO Will

Try
Update

Though

f

30

to Undo
Is NOT

20 Even
on Page

ERROR?!
Fig. 16.

Reversing

the

order

Selective

of the

the

problem

either.

This

pass

were

to precede

the

need

to be redone.

become

greater

of that

CLR’S

update

is redone

would

not

redoing

LSN

30

selective

redo

incorrect
redo

pass,

Figure

15,

then
the

to

Since,

the

page.

even

update

we

undo

of the

if the

would

that

violate

the

undo

passes

suggested

might

lose

of 20

in

would

make
and

the

redo

than

the

is not

present

durability

and

If

not

solve

the

undo

of which

of a CLR

during

will
[3].

track

is less
update

the

scenario

is

writing

page-LSN

though

and

approach

30, because

only

redo

that

In

than

redo with WAL—problem

the
the

pass,
log

actions

page

LSN

assignment

a log

record’s

record’s

LSN,

we

on the

page.

Not

properties

of

atomicity

transactions.
The

use

to have
be

of the

the

undone

shadow

concept
and

what

during

a checkpoint,

shadow

uersion,
create

points

version

needs
an

technique

to

be

by

that

With

the

version
storage.

updated

Figure

1).

R makes

to determine

consistent

of the

(see

System

system

redone.

on nonvolatile

version

database

in

action

is saved

a new

of the

page

of page.LSN

page,
During

called

between

two check-

constituting
recovery

is performed

during

restart

version,

and

shadowing

is done

even

there

is

no

ambiguity

about

which

which

are

not.

All

updates

database,

and

all

one

reason

database.7
correct]

This

y even

management

is
with

selective

changes

are

updates
the

redo.
not

logged

logged
System
The

logged,

but

current

thus

shadow

after

the

the

restart,

a result,

the

to

database,

the

in

needs
technique,

of the

As

not

page

Updates

ery.

and

unnecessary

what

shadow

from

database

it

updates
the

before

the

R

recovery

last

recov-

are

in

the

in

the

checkpoint

checkpoint

are

method

other

reason

is that

index

are

redone

or undone

are

functions
and

logically.

space
8

7 This simple view, as it is depicted in Figure 17, is not completely accurate–see
Section 10.2.
s In fact, if index changes had been logged, then selective redo would not have worked. The
problem would have come from structure modifications
(like page split) which were performed
which were taken advantage of later by transacafter the last checkpoint by loser transactions
tions which ultimately
committed. Even if logical undo were performed (if necessary), if redo was
page oriented, selective redo would have caused problems. To make it work, the structure
modifications
could have been performed using separate transactions.
Of course, this would have
been very expensive. For an alternate, efficient solution, see [62].
ACM Transactions

on Database Systems, Vol.

17,

No.

1,

March 1992.

ARIES: A Transaction Recovery Method
was described
history.
Apart

As

repeats

repeating

history

commit

some

ultimately

10.2
The

for

during

a long

them.
not

time,

actions

could

were

left

paper,

in the

writing

as

transaction

back

of

and

may
For

what

3

may

updates

to nonvolatile

to

we

care

a checkpoint

about
is

the

a way

of the

next

record

to be

which

may

already

be rolling

the

failure

is unimportant
are

not

uisible

That

is,

restart

recovery

starts

from

the

before

the

system

failure

the

time

of

committed

or

rollbacks

after

transactions

the

checkpoint.

last

passes

to avoid

redoing

backward

scan,

over

some
when

the

log

actions
the

in

since

the
in

—this
failure.

which
The
during

the

the

time

keeps

track

since

processing
and

handling
pass.

restart.

version

this,

initiated

performed

as of the

shadow

to handle
completed

only

to have

to undo

them

about

a partial

rollback

a little

the

partial
need

wanted

later

having

are
those

the

designers

information

last

of

CLRS

is to avoid
The

of

at the

during

database

only

some

changes

database

of

at
R

transactions,

the

of the
written

The

of a transaction

special

redo

is

a

R.

state

Despite

special

R

database

is

some

of progress

System

of the

is
Since

been

active

the

state

to do some

in-doubt

System

rollback

checkpoint

R needs

since

state

last

partial
level,

have

System

the

System

and

might

in

rollentire

systems.

in

of a system

written,

occurs

of the

the

application

record

num-

the

Supporting

the

of the

after

system

of

track

each

any

not

do this

The

of

for

to keep
to

this

only

and

rollback

in

advantages

cause

easy

time

at

actions

transaction

state

for
back.

present

13.

back.

a failure

checkpoint

undone

would

elsewhere

will

at

undone

known

its

roll
also

the

transaction

So,

back

present-day
when

Section

have

whether

these

in

of writing

in recovery

In fact,

violation

not

during

need

play

violation

partial
if

is relatively

taken.

roll
key

a

back

they

and

of

around

a significant

advantages

the

roll-

concept

been

literature,

all

by

updates

the

has

the

section

advantages

internally,

It

that

to note

the

for

the
and

problems

causing

performed

storage,

in

this

partially

illustrates

be rolling

rollback.

While
and

we try

or

requirement

of the

multiple

ability

transaction

describe

systems

In

a unique

that

problems.

additional

these

totally

least

a transaction

for

the

introduced

CLRS

community.

[56].

statement

effects

never

but

locking,

9.

difficulties

been,

research

in

example,

at

important

really

contexts,

Figure

database

redo,

us the

many

role

summarize

31],

checkpoint

gives

of the

in

fundamental

questions

[1,

we

It

Section

writing

to them

and

undone

We

transaction.

time

the

the

how

relating

be

update

transaction

selective

fine-granularity

of whether

in

discuss

some

not

by the

the

rollback
very

has

recognized

CLRS.

of reasons.

to

solves

appropriate

A

perform

effect.

described

and

problems

open

ber

side

implemented

there

utility

well

is

progress

been

of CLRS,

been

support

irrespective

as was

rollbacks

has

Their

not

to

beneficial

subsection
their

CLRS

discussion

does
us

139

State

of this

writing

another

or not,

in tracking

performed

allowing

of a transaction

commits

goal

backs

ARIES

from

has

actions

Rollback

before,

.

with

a

occurred

is encountered.
Figure
All

log

18 depicts
records

are

an
written

example
by the

of a restart
same

recovery

transaction,

scenario
say T1.

ACM Transactions on Database Systems, Vol

for

In the

System

R.

checkpoint

17, No. 1, March 1992.

140

.

C. Mohan et al

Last

‘“g~
Uncommitted
Changes
Need
Undo

Fig. 17.

Committed
Changes
Redo

Or In-Doubt
Need

Simple view of recovery processing in System R

~..----_- . .
12

3

4

5,,.’-6

7

Partial

rollback

handling

8 ::jg

Log

@

Checkpoint

Fig. 18.

record,

the

information

checkpoint

was

taken

for

T1

log

record

partial

rollback.

System

write

a separate

log

record

be

inferred

information

must

records

of

points

to the

the

record

PrevLSN

after

the

we

examine,

that

pointer
3, we conclude
with

recovery

During
record

5 and

of

will

partial

hence

it

6,

7,

during

the

redo

and

during

the

analysis

8.
pass,

pass

in

9 is a commit
and

during

Here,

the

same

transaction

pass.

To

see why

the

undo

To

pass

has

log

transaction
record

this

protocol.

4 and

notice

via
written
When
that

its

preceding

started

with

the

database

the

log

undo
state

of 3
from

state

of the

database

as of the

to be undone.

Whether

1 needs

transaction

or not.

T1

is a losing

that

a partial
that

is involved

Such

of the

is the

is patched

redo

a

needs

log
record

not

does

a transaction

immediately

that

also

log

follow

record

restart,

ensure

record
the

not

rollback

that

the
log

by

determined

it is concluded

records

written

of the

on whether
is

by

log

during

2 definitely

pass

written
that

the
of

place.

record

does
pass,

it

time

because

took

processing

1, instead

Since,

depend

analysis

be undone

the

but

chaining

forward

rollback

the

the

recently

first

to

write

a log

to be performed

log

will

in

analysis

2.

record

or not

record

rollback

breakage

needs

redone
If log

a partial

the

of

log

the

that

undo

checkpoint,

to be undone

that

CLRS,

say

most

is pointing

ended

not

by

undone

from

the

of the

2 since
been

does

of a partial

record
which

was

record

already

only

to

But

as part

the

3 had

Ordinarily,

pointer.

completion

Prev-LSN

last

R not

a transaction.

to log

points

in System R,

the

log

record

rollback

had

rolled

back

by

5 to make
then,

during

pass

log

both

putting

records
in the

to precede

to
the

records

log

undo

are

not

pointer

to log

record

9.

undo

pass,

log

record

4 and

5 will

undo

the

caused
a forward

it point
the

9 points

redo

pass
pass

and
in

2

be redone.
the

redo

System

in

R,g

g In the other systems, because of the fact that CLRS are written
and that, sometimes, page
LSNS are compared with log record’s LSNS to determine whether redo needs to be performed or
not, the redo pass precedes the undo pass— see the Section “10. 1. Selectlve Redo” and Figure 6.
ACM Transactions

on Database Systems, Vol

17, No. 1, March 1992

ARIES: A Transaction Recovery Method
consider

the

allowed

to

following
reuse

that

transaction,

in

the

rollback,

partial

record’s
dealt

ID
with

sequence
redo

the

scenario:

Since

a transaction

record’s

ID

a record

above

case,

which

for

a record

had

might

have

been

reused

in

redo

pass.

To

the

of actions

be fore

the

have

dealt

with

the

portion

in

repeat

the

undo

the

by

the

deleted
undo

of the

141

a record

later

been

in

history

failure,

deleted

inserted

might

to be

that

.

because

pass,

and

transaction

with

respect

must

be performed

is

same

to

of
that

that

the

is

original
before

the

is performed.

If 9 is neither
will

be

and

1 will

Since
one

a commit

record

nor

to

a loser

and

during

the

none

of the

determined
be undone.

CLRS

are

as well

from

what

in

System

R and

operations

were

interspersed

to this

Not

logging

footnote

being

done

example:

the
also

physically

(i.e.,

piece

of

data

1, T2

adds

had

logged

the

after-image

for

will

be a data

value

3

integrity

instead

fancy

by

different

by
lock

dumb

using

depend

Section
high

not

mode

concurrency

the

processing
of

the

back,

and

redo

the

operation

for

after

recovery

and

in
Of

redo

the

same

object.

let

redo

does

not

necessarily

or

not

flexible

logging

to be supported

object

has
Then,

If T1

undo,

and

then
will

have

for

T1

is

System

R did

not

2 concurrent
the

support

logging

of

very

efficiently

performed

logging;

management

is

logically

examples).

the

being

updates

byte-oriented

621 for

T2

these

data

mean

[59,

to
an

the

information

also

us consider

storage

of undo
(see

be

(see

checkpoint.

Allowing

recovery

space
normal

information

commits.

to support

is

further

F?, undo

course,

be needed

last
T2

System

different

some

or undo

O after

a given

during

logging

value

which

R also

occur

2

history

cause

not

2, T1 rolls

update.

the

potentially

redo

quite

Let

its

Allowing

be

repeating

on an

would

This

may

in

for

performed

redoing

whether

processing,

operation).

which

on

transactions’

by

case,

logic.

way

other

operation

because

to

exact

in System

did

records

be redone.

the

this

will

the
with

(i.e.,

prevents

ln

physically

10.3).

resiart

problem
2.

transactions

information
will

of

will

the

log

created
has

adds

the

that

CLRS

T1

accomplished

could

split

transaction

records

changes

These

the

pass

restart

a

during

transaction

during

8).
as

after-image

known,

index

writing

the

A

such

then

undo

hence

processing

required

Not

be logged—not

pages
normal

(see

being
5.4).

is not

during

problems

processing

actions

different

contributes

from

written

as across

management

Section

pass,

happened

to guarantee).

record,

redo

or undo

impossible

a prepare

the

undo

processing

page

In

not

transaction’s

forward

be

redo
that

used
will

ARIES

(see

permit
supports

these.
WAL-based
during
the

systems

handle

using

CLRS.

So, as far
forward,

rollbacks
data

is

always

“marching”

being

rolled

back.

Gontrast

which

the

rollbacks.

state

of the

That

method

this

problem

works

only

to be rolled

back,

then

some

once

and,

worse

still,

the

compensating

more

than

once.

This
back

consequence

is illustrated
even

the

before

by

with

were

rolling

even

with

The

by

some

approach,

is “pushed”

page

level

(or

CLRS

is that,
are

are

also

in

Figure

4, in

failure

of

the

in

which

in

during

granularity)

if a transaction

undone

more

undone,
a transaction

system.

of
are

[521,

back

coarser

actions

actions

state

actions

suggested

LSN,

original

performed
the

original

the

the

ACM Transactions

actions

is concerned,

if

of writing
of its

logging

as recovery

as denoted

locking.

started

immediate

data,

this

Then,

than

possibly
had
during

on Database Systems, Vol. 17, No. 1, March 1992.

142

C. Mohan

.

recovery,
CLRS

the
are

previously

undone

the

idea

lock

management
Section

the

next

CLRS

are

ARIES

avoids

such

Not

undoing

CLRS

CLRS.
and

12, and
section

early

release

Section

and

Unfortunately,
support

written

again.

of writing

22,

et al.

in

6.4).

[691.

Additional

We

that

is

point

has

already

like

feel

benefits

still

in

also

to

dead-

(see

item

are

discussed

the

Section

in

[921

suggested

is an important

non-

retaining

relating

objects

of CLRS

one

undone

while

discussed

the

this

already

on undone
benefits

were

methods

rollbacks.

and

a situation,

of locks

Some

recovery

partial

undone

in
8.

do

drawback

not

of such

methods.

10.3

Space

The

goal

Management

of this

management
length

records

A

management

is

reservation
updates,
vent
before

doing

is to make

sure

that

the

or

update

the

on

[761.

problem

here,

a

data
do

interest

of increasing

released

by

commit

one

not

first

using

a logical

is

with
by

not

with

reader

solutions

is referred

concurrency,

we

from

transaction.

do

being

The

undo

way

approach

under

desirable

a goal,

it

of data

do (see

811).

locking

have

something
page

to

like

which

describes

how

is that

garbage
or log

us the

flexibility

and

modify

have

to

be

#,

points

the

e.g.,

and

attempting

problems
in

insert

requiring
space

collects

of being

able

a scenario

storing

the

to

perform

flexible
19
200
in

it.

in

LSN

redo

involve

bytes
This

and

name

looks

#

identifies

a location

on the

The

record

got

unused

space

around

with

an

same
the

is
and

for

log

consequence
does

page.

within

systems

not

have

This

gives

a page
like

IMS,

to store
utilities

fragmentation.
track

These

of the

actual

page

version

of the

page)

point

in

log

used.

Assuming

storage

on a page
need

the

around

earlier

The

on a page

within

keeping

page

record.

changed.

nonvolatile

from

want

lock

record

not

not

record’s

records

which

is attempted
shows

logging

page.

address

The

management
the

did

The

storage

in the

storage

We

deal

to

to users.

as

the

record.

In

frequently

do

a page,

the

efficiently.

y of data

to

within
to use

of the

to move

records

not

[62].

want

on the

slot

moved

was

pre-

not

location

data

are

quite

the

actual

of the

length

page.

# ) where
the

that

Figure
left

slot
to

a

that

when

updates

changed

records

19 shows

(by,

name

were

collection
the

run

lock

that
within

contents

availability

state

free

(page

variable

the

Figure

logical

for

as the

bytes

be

then

to lock

reduce

of a record
specific

to

in

logging

byte

index

is dealt

was

the

For

with

undo

and

first

space

to [50].

is described

locking

to identify

problem
this

another

management

of the

This
to

by

storage

did

another

want

byte-oriented)

is, we

during

by

not

(i.e.,

That

storage

consumed

flexible

[6, 76,

flexible

consumed

Since

systems

space

varying

a transaction

physical
some

in
and

is committed.

deal

transaction

of the

locking

released

page

interested

the

circumstances

involved

of locking

transaction

We

The

problems

record
space

space-releasing
in

the

granularity

efficiently.
in

space

the

such

out

level

to be supported

briefly

in
the

page

with

until

discussed

to

than

to be dealt

deletion

transaction

finer

are

problem

record

subsection

when

the

same

which

has

exact

ACM TransactIons on Database Systems, Vol. 17, No. 1, March 1992

tracking

the

leads

to

that

all

transaction,
only

an

100 bytes
of

page

of

state

ARIES: A Transaction Recovery Method

Redo Attempted
From Here.
It Fails Due to
Lack of Space

Page Full
As of Here

143

.

Page State
On Disk

.Og
Oelete
RI
Free 200
Bytes

Fig. 19.

using

an

avoid

Typically,

each

pages

map

file

called

pages

space

(SMPS)

in

to

possibly

based

on

location

of other

records

the

record,

one

new

enough

many

free

is full,

leasing

or

space

/’

with space for insert.

operations

which

the

same

or more

FSIPS

are

in

inserting

at

least

in

of the

independence,

T1

full,

are

already

more

relations

has

They

are

space

full.

Later,

not

require

an

update

space

would

change

FSIP.

If

had

then

T1’s

T1

need

for

need

to do logical

logging

That

is,

while

whether

that

does

cause

the

change

transaction
but

to

the

FSIP.
full
FSIP

cause

the

state

of the

changes

to

with

undoing

a data

causes

the
not

then

ing

the

We

perform

to perform

an example

if

T1

were

to

this

should

not

cause

record

as

entry

to say

O% full,

FSIP

update,

free
the

can

space
FSIP

easily

update

an update

space

the

system

construct
to the

during

which

a CLR

and

to the

FSIP

update

performed

during

of the

update

during

rollback.

then

the

to

the
be

to the
for

the

to determine

which

and

if it

describes
in

forward

rollback.

O%
does

updates.

example

during

full

would

points

has

of

record,

inventory

the

the

update

to change

an

FSIP

which

changes

free

write

from

scenario

information
and

23%

it

a redoiundo

redo-only

as

page
the

This

from

back,
an

the

recovery

full,

roll

log

page.

to

handling

change

Now,

to the

an

in which
inverse

the

to

with

space-re-

update

to change

FSIP

to go to 35%

data

every

to provide

space

FSIP

of the

not

also

page

the

respect

update

FSIP.

the

change

would

undos

exact

and

current

construct
is not

to

cause

operation

does
needs

to

its

a change,

be logged.

update

only

25%

special

also

an

keeps

least

avoid

and

on the

FSIP

an

undo,

of

page

at

that

the

as that

a data

requires

To

space

written

the

sure

FSIP.
must

keys)

The

as that

page

the

3 l%

the

make

and

related

record.

FSIPS

might

about

such

cause

T2

operation,

index

to identify

to the

to

rollback

given

to

a data

information

a clustering

new

to

requiring

to 25%

the

space

a

insert

consulted

redo

the

called

a record

(or closely

etc.)

during

might

full

key

corresponding

FSIPS

thereby

from

is full,

the

updates

Transaction

describes

information

operation

or

During

(e.g.,
5090

-consuming

recovery

wrong,

for

one
(FSIPS).

FSIP
pages.

obtained

with
it

of

pages

Each
index

information

information

27%

or

information

page

ing,

redo

records

inventory

DB2.

data

space

approximate

to

problem

to

containing

free

relating

the

attempting

Insert
Commit
R3
Consume
100 Bytes

to the page.

applied
few

Oelete
R2
Free 200
Bytes

Wrong redo point-causing

to

LSN

Insert
R2
Consume
200 Bytes

We

forward

which

a

processcan

also

process-

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

144

@

C. Mohan

10.4

Multiple

LSNS

Noticing

the

support

record

et al

problems

caused

locking,

may

by

assigning

object’s

state

precisely

explain

why

it is not a good

already

supports

DB2
This

happens

requiring

does

do locking

an

during

LSN

for

the

corresponding
is set

the

minipage

redo

pass,
an

leaf

page

and

not

to determine

the

minipage.

for

storing

the

LSNS,

tends

able

for

storing

keys.

Further,

of record

and

if that
This

log

locking,

to

efficiently.

We

desired

when

minipage

locking

very

efficient.

The

before

performing

recovery,
restart

recovery

to be sufficient,
page

into

handle

the

a fixed

number

space

reservation

for

fine-granularity

the

terminology

11.

OTHER

In

following,
methods

on the

shadow

because

of

(see

sions).

First,

which

we

Siemens.

we

are

summarize
also

technique

introduce

be examining

ACM Transactions

is cumbersome
for

each

media

history

during

repeating

technique

is

the

varying

one

length

even

especially

divides

like

at

page,

transactions

because

properties
WAL

that

turns

out
up a

needed

proposed

to

in

[61]

objects

(atoms

in

other

significant

in

the
this

has

been

for

the

1/0s

paper

and

[31]

systems

have

Next,
been

of information

we

based

considered

here

costly

17, No. 1, March 1992

copies

for

additional

and

recovery

compare
that

significant
about

checkpoints,

involving

informed

with

it here.

on Database Systems, Vol

methods

shadow

extra

different

not

very

and

implemented

of lack

R) are

e.g.,

section.
We

some
Recovery

of System

of data,
of this

of

protocol.

overhead

dimensions.

of [25]
to include

the

space

we briefly

But,

to be

physically

support

(like

sections

unable

case

have

objects

recovery,

special

avail-

length

DB2

disadvantages,

storage

various

space
to the

objects

no

on

overhead

of loser

the

use

previous

method

by

we

the

along

undone

space

waste)

(LSN)

Methods

is

METHODS

clustering

methods

it

record’s

conveniently

deleted

Since

minipages,

physical

recovery

undo,
log

rollback

problem.

the

will

page

of

in ARIES.

do not

much

the

The

technique
the

seen

well-known

nonvolatile

blocks

done,

which
page

their

disturbing

being

having

paper).

WAL-BASED

the

extra

for

locking

recovery

varying

variable
to make

of

of that

over

state

minipage’s

to be actually

therefore

way

of loser

field.

to the

(and

The

is updated,
During

carry

of

2 to 16

besides

LSNS.

LSNS

as we have

12].

each

LSN

too

when

option

actions

tracks

needs

a page.

into

[10,

redoing

incurring
not

simple

the

index

is compared

a single
is

has

a minipage

that

fragment

Maintaining
have

than

of the

minipage

update

especially

supported

to

to

trying

less

minipage,

minipage

LSN

it does

best.

each

in the

besides

user

DB2

with

of the

record’s

is

not

Whenever

page

technique,

key

LSN

is stored

the

LSN

despite

is as follows.

maximum

to the

LSN

when

of a minipage

pages,

as a whole.
LSN

the

leaf page

granularity

associating

equal

page

that

where

up each

on such

record’s

per

of locking

indexes

by
log

LSN

of

at the

the

LSN

to suggest
that
we track
each
a separate
LSN
to each object.
Next
we

divide

properly

separately

one

tempting

idea.

case

and

transactions
state

the

to physically

recovery

having

be

a granularity

DB2

minipages
DB2

in

by

it

the

of

data,

page

map

discusmethods

the

different

the

DB-cache

modifications
implementation,

ARIES: A Transaction Recovery Method
IBM’s

IMS/VS

[41,

database

system,

consists

of

relatively

flexible,

and

IMS

Fast

has

many

42,

43,

48,

53,

two

76,

parts:
Path

42,

93],

[28,

restrictions

(e.g.,

no support

can

both

FF

and

buffering

methods

used

by

the

two

parts

depending

on the

database

types

and

the

locked

objects

databases
fixed

length

make

the

page

FP

locking

hold

times

is

supported

provides

ports

data

pOOk

[80,

for

DEDBs.

granularities

databases:
MSDBS

mechanisms

(i.e.,
for

storage

support

field

MSDB

only

calls)

Only

many

high-

hot-standby

support

[431.

IMS,

via

locking,

also

sharing

across

two

different

systems,

each

with

to

records.
IMS,

global

FF,

of the

main

(DEDBs).
possible

In

have

its

with

own

supbuffer

941.
relational

database
access

granularities

been

and

repeatable

read)

[10, 11, 12]. DB2

for

and

indexes

tables

reorganizing

for

data.

with

atomicity.

has

been

A

The

single

provides

NonStop
a

support

single
of

NonStop
record)

(file,

key

prefix

and

able

read,

and

unlocked

or permanently

even

Schwarz

[881
(a

la

differences,

for

IMS)

two

and

as

will

be

less

complex

been

implemented

in

management.

adopted

and

OLM

the
write

storage

and

written

back

an

steal
a

fetch

record

alone.

These

pages

that

might

have

has

a sophisticated

been

in
buffer

commit

granularities

stability,

repeat-

off temporarily

based

logging

SQL,

a page
are
help

the

two-phase

methods

logging

time

These

records

and
updates

methods
two

During

whenever

storage.

OLM

Tan-

multisite

locking

value

NonStop

every

changes

on

have
method

method

value
several
(VLM),

(OLM),

has

901.

policies.

record

in
DB2

operation

no-force

end-write

to nonvolatile

The

the

Encompass,
and

some

data

on files.
The

[23,

and

IMS

Encompass

be turned

recovery

below.
Camelot

and

NonStop,

(cursor

can

logging.

than

Abort

and

stability,

loading

DB2

allow

levels

data,

off temporarily

With

different

operations

outlined

CMU’S

They

Logging

different

for

DB2

supports

(cursor

Both

Presumed

consistency

operation

is much

access.

It

like

[95].

products.

supports

nonutility

presents

page

The

19].

[4, 37] with

SQL

its

the

read).

and

both

algorithm

for

and

15,

access

system.

DB2.

14,

operations

NonStop

SQL

or dirty

which

have

recovery

using

64].

13,

to be turned

can

data

operating
in

logging

transaction

transaction

[63,

[1,

table

utility

distributed

MVS

levels

allows

support

the

available

consistency

during

Tandem’s

hot-standby

SQL

and

Encompass
in

in

(tablespace,

indexes)
only

incorporated

for
are

presented

locking

page

system

functions

different

failure.

the

of

and

differences.

support.

minipage

dirty

many

DEDBs

has

also

have

database

algorithm

ing

recovery

But,

recovery

Buffer

A single

The

large

data

logging

IMS

indexes).

(FP)

is
but

and

distributed

protocol

which

efficient

features

Limited

within

is more

parallelism

is IBM’s

dem

which

data.

the

minimum

a hierarchical
(FF),

secondary

kinds

be the

is

145

Path

databases

provides

which
Function

operations,

two

entry

FP

for
Fast

supports
data

but

and

XRF,

and

records,
lock

availability

DB2

vary.

(MSDBS)

941,
Full

transaction

access

80,

IMS

.

a

OLM,

and

DB2

processing,

is

from

nonvolatile

is

successfully

read

dirty

written
in

VLM

normal

page
during

identifying

buffer

pool

manager

[10,

restart
the

at

the

961,

and

process-

super

time

VLM

set

of

of system

writes

a log

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

146

.

record

C. Mohan

whenever

whenever
after

the

storage.

DB2’s

For

not

writes,

to the

of the
pass

FP

MSDB

log

released.
held

processes.

to stable

the

storage

ultimately

logging

is

completed

all

the

of the

to

nonvolatile

the

pages

1/0s,

release

updates

being

policy

is used

pages

to nonvolatile

the

next

lelism
that

modified

nonvolatile
section

FF,

this

storage.
force

Normal

the

the

system

all

activity

the

checkpoint

record

SQL,

and

Encompass

to what

of

writing

group

This

follows

uncommitted

version.

Also,

tion

present.
yet

it

that
needed

is written.
been

changes

Care
written

is

writes

the

for

a checkpoint.

is ensured

major

no

Since

deferred

will

be

partial

For

DEDBs,

to

nonvolatile

placed

on

in

this

any

ones

are

System

R,

an

DB2,

contents
NonStop

committed
are

on Database Systems, Vol. 17, No. 1, March 1992

and

actions

are

instead

(table

spaces,

For

MSDBS

of two

files

updating

updates

update

is that,

objects

one

operation

IMS,

when

[961.

in

taken
quiesce

The

checkpoint

present

storage

that
VLM

committed
the

considered

dirty
on

since

is

being

difference

alternately

pages

data

even

object

the

locking

checkpoint.

each

Before

all

page

of ARIES.

The

paral-

policies.

and

DB2’s

with

gain

OLM

checkpoints

ARIES.

no

to

DEDB

than

are the

similar

a no-steal
the

to

storage

mode.

to those

contents

force

finer

consistent)

for

MSDBS,

and

of the

go ahead

also

algorithms

recovery

described

during

steal

forced

uncommitted

writing

and

uncommitted

we

storage

process

Since

concurrently.

volatile

the

to nonvolatile

(fuzzy)

their

for

user

are

with

processes

recovery

it

transaction

as possible

on

IMS

not

locking

going

table,

ACM Transactions

page

take,

a RecLSN

have

since

checkpoints

with

record

any

Normal

_pages

commit

in

processing.

dirty

After

result

some

similar

[28]).

committed),

not

commit

and

been

does

transaction.

restart

has

records

completion

the

the

is used—see

to

log

on

forces

all

the

which,

to let

in

transferred

it force

the

record
of time

by

as soon

FF

log

modified

storage

FF

(not
locks

processes

is intended

take

logic

buffers

amount

are

FP

a single

record

commit

locks

transaction

of separate

IMS

log

the

to let

commit

were

use

are

complete

DEDB
time

the

the

minimizes

The

after

etc. ) list

are

FP

is given

indexspaces,
writes

how

transaction
do

are

the

released
is

records.

system

of the

similar

before

in the

necessarily

activities

even

course,

(not

logging

MSDB

system

during

is not

in

consistent

the

the

result

checkpointing.

when

in

and

locks.

Of

log

in

records

are

is used.

transaction

log

that

may

objects

a transaction

policy

applied

that

dirty

that

the

(i.e.,

by

means

a no-steal

are

The

nonvolatile

the

placing

This

IMS

to bring

updates

processing

1/0s.

by

a given

storage

a transaction,

supported

for

log

to nonvolatile

committing
were

these

manager

DEDBs.

the

records

the

DEDB

transaction’s
for

This

DEDBs,

using

forced
for

updating.

DEDBs

the

to

For

(i.e.,

storage

back

been

deferred

MSDB
log

only

written

have

updates.

locks

The

record

performed

records

storage.

on

system

another

is

log

MSDB

MSDB

stable

and

operation

close

failure.

After

the

The

on

are

all

manager.

is opened,

The

space

uses

does

time,

indexspace

closed.

as of the

own

storage),

placed

locks

pages

IMS

at commit

on stable
are

is

analysis

see its

or an

space

up to date

MSDBS,

does

is

a

dirty

information

call

a tablespace

such

all

et al.

alone,
on

non-

is performed

for

their

checkpointed

changes

of a transac-

are

applied

updated
included

after

the

pages

which

in

check-

the

ARIES: A Transaction Recovery Method
point

records.

recovery,

These

any

Encompass
storage

log

and
during

page

together
written

NonStop

SQL

dirtied

tion

of the

second

this

policy,

the

completion

completion

of the

writing

Partial

rollbacks.

partial

partial

rollbacks.
level.

access

FP

data.

undo

data

in

the

FF

IMS

FP

does

for

changes

FP

log

records

and

supports

because

FP

needs

to get

MSDBS,

write

is always

the

coordinator

into

prepared

the

updates

kept
policy

time.

Since

a no-steal

the

modified

pages

time.

CLRS

restart
log

This

transaction

mit,

with

storage—
policy,

of

must

some

of its

when
none

simply
SQL,

have

been

commit

system

went

down.

the

corresponding

FP

storage

hence

writes

CLRS
contain

undo

information

is

volatile

storage

is accessed

during

even

a no-steal

reader

rollbacks,
often,
has

that
there

people
many

VLM

such

records

only

redo

there

records

the

for

and

are

needed,
with

still

assume

the

some
that

problems

not

write

CLRS

during

of logging

will

occur

for

repeated

failures

during

media

is done

rollback

for

DEDBs,

the

buffer

IMS

(FF

and

FP)

write

IMS

FP

might

in-progress

pool

about

to

of the
been

[931.

Since

CLRS,

unmodified
This

to

IMS

FP

with

many

the

FP

log

for

which

the

data

on

non-

illustrate

to

should

without

no-steal

written

to be undone,

these

com-

to nonvolatile

because
have

at

transaction.

written

recovery

to be dealt

for

at

from

recovery,

recovery.

is

is performed

discarded

would

eliminates

records

supporting
at restart

partial
for

problems.

FP.

Too

Actually,

it

shortcomings.

amount
rollbacks.

are

been

and

log

it never

corresponding

no-steal

any

hence

to write

policy

OLM

rollback,

and

though,

media

restart

VLM,

to

commit

be nothing

just

DB2,

processing—i.e.,

would

system

This

one

updates

to simplify

for

the

made.

and

having
Even

information,

write

a normal

locking

most)

do not

is performed

updating

page

application

is

restart

in

the

sup-

supports

not

written

purged

the

already

do not

rollback

lists

During

records

the

does

by

SQL,

have
to

DB2

(at

log

of

nonvolatile

are

NonStop
also.

use

deferred

and

by

updating

in two-phase

is followed

written

of

for

that

FP

During

not

decision

Since

at the

is because

internal

it would

(to-do)

rollbacks

Because

1, IMS

applications

NonStop

pending

DEDBs

records

for

VLM

is exposed

rollbacks.

the

state.
in

Encompass,

during
some

since

and

a

comple-

waiting

2 Release

deferred

normal

until

OLM

to those

Encompass,

during

data

the

rollback

[1].

CLRS

such

SQL,

because

atomicity

delayed

that

the

page.

be

Version

is excluded

rollbacks

before

of the

may

only

partial

CLRS

not

dirtying

concept

data

log records.

storage

nonvolatile

pages.

savepoint

reason

to

to

requires

From

The

write

pages
that

NonStop

is available

statement-level

IMS

find

fact,

recovery.

dirty

nonvolatile

restart

data

policy

dirty

rollback.

FP

the

the

support

Compensation
and

Encompass,

during

for

some

following
old

examining,

force

of a checkpoint
of the

for

checkpoint

enforce
to

This

its

DB2

provide

checkpoint

In

need
the

might
They

be written

transaction

program

MSDBS.

must

the

before

a checkpoint.

once

port

avoid

records

147

.

does

Of
recovery.

course,
OLM

restart.
this

has

writes

restart

a rolled
In
some

CLRS

rollbacks.
back

fact,

As

transaction,

CLRS

are

written

negative

implications

for

and

undos

redos

a result,
even

a bounded
in

the

face

only

for

normal

with

respect

performed

of
to

during

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

148

.

restart

C. Mohan

undomodify

(called

done

to

modify
rupt

deal

with

and

redomodify

restart
restart

causing

the

CLRS

for

a given

worst

case,

the

grows

exponentially.

The

net

result

forward

need

to redo

will

and

DB2

undo

changes

the

writing

Log record

the

state)

logging

and

logs

both

the

information

log

objects.

records

for

IMS

page.

This

FP

the

also

complete

log

NonStop
OLM
and

logs
DB2

logs

number

the

track

address

of the
a backup’s

buffer

undo

and

redo

information

of updated

only

the

before-

and

shot

of each

object.

redo

or

information

periodically

and

only

and
the

redomodify
the

the

does

not

The
undo

and

set of pages

of

by

a modified
recovery

DB2

the

updated

CLRS

and
fields.

since

their

consistent
records

records
of the

snap-

contain

corresponding

parts

VLM

of Encompass

information

the

in

updated

records.

undomodify
where

of

and

redomodify

L SNS

For

information

Encompass

logs an operation

undomodify
but

modify,

redo

IMS

names

of

operation.

OLM

OLM’S

specifies

update

FF

Since

or restart

updates.

the

Ih’ls

occupied

of DEDBs’

value

[761).

information.

takeover

after-images

does

(see

enough
lock

log

policy,

after-image

IMS

redo

the

of

force

(i.e.,

before,

includes

record

recovery.

information

only

IMS

given

of its

media

them.

others,

the

locking

during

of the

the

information.

have

both

like

IMS

for

case,

work

be undone.

OLM’S

to

CLRS
a

of redo

might

which

the

is used

CLRS

But

need

redo

problem.

write
for

mentioned

byte-range)
the

this

the

failures

CLR

during

In

restart

Because

redo

As

to

to contain

map

only

system

description

a page

only

and

backup

the

records.

updates

the

need

undo

worst

linearly.

IMS

also

log

the

support,

amount

SQL

In
grows

hot-standby

information

to reduce

same

(i.e.,

CLRS

failures,

the

policy.

not

thus

identical

processing.

avoids

does

multiple

of CLRS,

repeated

ARIES

times

writes

physical

updates,
XRF

FP

no-steal

or restart

hence

themselves.

of multiple,

during

how
and

of

OLM

IMS

(or

its

pass

CLR’S

of its

the

forward
written

5 shows

and

contents.

CLRS’

during
records

undo

because

undo

and

processing.

IMS

undo

CLRS

written

of records)

providing

for

multiple

by

IMS

CLRS

because

during

inter-

if

for

that,

written

failures

record

generated

writing

records

undo-

are

Encompass

update

is

This

multiple

a given

of log

written

write

for

Figure

up

respectively).

might

CLRS

the

is

wind

OLM

No

record

during

records,

restart.

records

of CLRS

number

CLRS

might

during

recovery,
writing

redomodify

and

failures

processing.

During

ignores

et al

no

modify
also

contain

modified

object

reside.

Encompass
and NonStop
SQL use one LSN on each page
Page overhead.
uses no LSNS, but OLM uses one
to keep track of the state of the page. VLM
LSN.
DB2
uses one LSN
and IMS
FF no LSN.
Not having
the LSN
in IMS
FF and VLM

to know

the exact

state

of a page does not cause

any problems

because of IMS’ and VLM’S value logging
and physical
locking
attributes.
It
is acceptable
to redo an already
present
update
or undo an absent update.
IMS FP uses a field in the pages of DEDBs as a version number
to correctly
handle redos after all the data sharing
systems have failed [671. When DB2
divides
an index
minipage,
besides
ACM Transactions

leaf page into minipages
then it
one LSN for the page as a whole.

on Database Systems, Vol

17, No. 1, March 1992.

uses

one LSN

for

each

ARIES: A Transaction Recovery Method

.

149

Log passes during
restart
recovery.
Encompass
and NonStop
SQL
two passes (redo and then undo), and DB2 makes three passes (analysis,

make
redo,
their

and

then

redo

passes

undo— see Figure

This

is sufficient

dirty

page

from

the

because

within

6).

Encompass

beginning

two

of the

of the buffer
checkpoints

and

NonStop

penultimate

management
after

the

SQL

start

successful

page

checkpoint.

policy

of writing

became

dirty.

to disk
They

a

also

seem to repeat history
before performing
the undo pass. They do not seem to
repeat history
if a backup system takes over when a primary
system fails [41.
In the case of a takeover
by a hot-standby,
locks are first reacquired
for the
losers’ updates and then the rollbacks
with the processing
of new transactions.
using

a separate

that

point,

process

which

is

to gain

of the losers are performed
in parallel
Each loser transaction
is rolled back

parallelism.

determined

using

DB2

successful checkpoint,
as modified
by the analysis
DB2 does selective redo (see Section 10.1).
VLM
makes one backward
undo, and then redo). Many

starts

information

its redo

recorded

scan from
in

the

pass. As mentioned

last

before,

pass and OLM makes three passes (analysis,
lists are maintained
during
OLM’S and VLM’S

passes. The undomodify
and redomodify
log records of OLM are used only to
modify
these lists, unlike
in the case of the CLRS written
in the other
systems.
In VLM,
the one backward
pass is used to undo uncommitted
changes on nonvolatile
storage and also to redo missing
committed
changes.
No log records are written
during
these operations.
In OLM, during
the undo
pass, for each object to be recovered,
if an operation
consistent
version
of
the object

does not

exist

on nonvolatile

storage,

of the object from the snapshot
log record
version of the object, (1) in the remainder
updates

that

precede

the snapshot

then

it restores

a snapshot

so that, starting
from a consistent
of the undo pass any to-be-undone

log record

can be undone

in the redo pass any committed
or in-doubt
updates (modify
follow
the snapshot
record can be redone
logically.
This

logically,

and (2)

records only) that
is similar
to the

shadowing
performed
in [16, 781 using a separate
log—the
difference
is that
the database-wide
checkpointing
is replaced by object-level
checkpointing
and
the use of a single log instead of two logs.
IMS first reloads MSDBS from the file that received their contents
during
the

latest

successful

that

were

included

checkpoint

before

the

DEDB

buffers

are also reloaded

into

the same

during

the

a failure,

the

Then,

it makes

one forward

pass

in the checkpoint

records

that,

be altered.

buffers

as before.

This

number

of buffers

cannot

means

failure.

The
restart

dirty
after

just

over the log (see Figure
6). During
that pass, it accumulates
log records in
memory
on a per-transaction
basis and redoes, if necessary,
completed
transactions’
FP updates.
Multiple
processes
are used in parallel
to redo the
DEDB updates.
As far as FP is concerned,
only the updates
starting
from
the last checkpoint
before the failure
are of interest.
At the end of that one
pass, in-progress
transactions’
FF updates are undone (using the log records
in memory),
in parallel,
using
one process per transaction.
If the space
allocated
in memory
for a transaction’s
log records is not enough,
then a
backward
scan of the log will be performed
to fetch the needed records during
that transaction’s
rollback.
In the XRF context,
when a hot-standby
IMS
ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

150

C. Mohan

.

et al.

takes over, the handling
of the loser transactions
Tandem
does it. That
is, rollbacks
are performed
transaction
processing.
Page forces
the

end

during

of restart.

restart.

OLM,

Information

VLM

on

and

is similar
in parallel

DB2

Encompass

force

and

all

to

the
with

dirty

NonStop

way
new

pages

SQL

at

is

not

a checkpoint

only

at

and NonStop

SQL

is

available.
Restart

checkpoints.

the end of restart
not available.
Restrictions
record

have

IMS,

DB2,

recovery.

on data.
a unique

OLM

and VLM

Information

Encompass
key.

This

take

on Encompass

and

unique

NonStop
key

require

that

every

is used to guarantee

SQL

that

if an

attempt
is made to undo a logged action which
was never applied
to the
nonvolatile
storage
version
of the data, then
the latter
is realized
and
the undo
fails.
In other
words,
idempotence
of operations
is achieved
using

the unique

key.

IMS

in effect

hence does not allow records
results
in the fragmentation
imposes
that

some additional

an object’s

does byte-range

locking

and logging

and

to be moved around freely within
a page. This
and the less efficient
usage of free space. IMS

constraints

representation

with

respect

be divided

into

to FP data.
fixed

length

VLM

requires

(less

than

one

page sized), unrelocatable
quanta.
The consequences
of these restrictions
are
similar
to those for IMS.
[2, 26, 56] do not discuss recovery
from system failures,
while the theory of
[33] does not include
semantically
logging).
In other sections of this

rich
paper,

with

some of the other

that

12.

ATTRIBUTES

ARIES

makes

approaches

modes of locking
(i.e., operation
we have pointed
out the problems

have

been proposed

in the literature.

OF ARIES

few assumptions

about

the

data

or its model

and has several

advantages
over other recovery
methods.
While ARIES is simple, it possesses
several interesting
and useful
properties.
Each of most of these properties
has been demonstrated
in one or more existing
or proposed
systems,
as
summarized
in the last section.
However,
we
proposed or real, which has all of these properties.
ARIES are:

know
of no single
system,
Some of these properties
of

(1) Support for finer
larities
of locking.

concurrency

control

page-level

and

a uniform
locking

than page-level
ARIES

supports

fashion.

Recovery

is not

is. Depending

on the

expected

affected

by

contention

what

(2) Flexible
buffer management
long as the write-ahead
logging

schemes

the

for the data,

ate level of locking
can be chosen. It also allows
locking
(e.g., record, table, and tablespace-level)
tablespace).
Concurrency
control
schemes of [2]) can also be used.

and multiple

record-level

other

granu-

locking

in

granularity

of

the appropri-

multiple
granularities
of
for the same object (e. g.,
(e.g.,

the

during
restart and normal
processing.
protocol
is followed,
the buffer manager

As
is

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992

than

locking

ARIES: A Transaction Recovery Method
free to use any page
incomplete
transactions
transactions

commit

replacement
policy.
In particular,
dirty
pages of
can be written
to nonvolatile
storage before those
(steal

dirtied
by a transaction
transaction
is allowed
lead

to

reduced

151

.

policy).

Also,

be written
to commit

demands

for

it is not

required

that

back to nonvolatile
storage
(i.e., no-force policy).
These

buffer

storage

and

fewer

all

pages

before the
properties

1/0s

involving

frequently
updated
(hot-spot)
pages. ARIES does not preclude
the possibilities of using deferred-updating
and force-at-commit
policies and benefiting
from them. ARIES is quite flexible
in these respects.
(3) Minimal
(excluding
required

space overhead–only
log) space overhead

one
of this

LSN
per page.
scheme is limited

The permanent
to the storage

on each page to store the LSN

of the last logged

action

on the page.
(4)

No

The LSN

of a page is a monotonically

constraints

on

data

actions.

There

are

logged
unique

keys,

around

within

ensured

since

operation

should

be redone

(5)

taken

during

Actions

etc,

exact inverses

Records

to guarantee

collection.

the

page

on each

are being

written

during

original

actions

and

recorded

in

former.

data

length.

undo

of

respect

to

can be moved

Idempotence

is used

or

with

Data

performed
value.

of redo

on the

can be of variable

of operations

is

whether

an

to determine

or not.

the undo

of the actions

the

idempotence

no restrictions

a page for garbage
LSN

increasing

of an update

taken

undos,

what

during

any differences

actually
An

had

example

need not necessarily

the original

update.

between

to be done

of when

the

be the

Since

CLRS

the inverses

of the

during

undo

can

be

inverse

might

not

be

correct is the one that relates
to the free space information
10% free, 20% free) about data pages that are maintained

(like at least
in space map

pages.

while

Because

of finer

than

page-level

granularity

locking,

no free

space information
change takes place during the initial
update of a page by
a transaction,
a free space information
change might occur during the undo
(from 20% free to 10% free) of that original
change because of intervening
update activities
of other transactions
(see Section 10.3).
Other
benefits
of this attribute
in the context
of hash-based
storage
methods

and index

management

(6) Support for operation
to a page can be logged
redo information

can be found

in [59, 621.

logging and novel lock modes.
in a logical fashion.
The undo

for the entire

object

The changes made
information
and the

need not be logged.

It suffices

if the

changed fields alone are logged. Since history
is repeated,
for increment
or
decrement
kinds of operations
before- and after-images
of the field are not
needed.
Information
about the type of operation
and the decrement
or
increment
amount
is enough.
Garbage
collection
actions
and changes to
some fields (e.g., amount
of free space) of that page need not be logged.
Novel lock modes based on commutativity
and other properties
of operations can be supported
[2, 26, 881.
(7) Even redo-only
and undo-only
(single
call
to the
be efficient

records are accommodated.
log component)
sometimes

undo and redo information

an update

about

While
it may
to include
the

in the same log record,

at other

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

152

C. Mohan et al.

.

times it may be efficient
(from the original
data, the undo record
constructed
and, after the update is performed
in-place in the data
from

the

sary

(because

updated

different
tions,

data,

of log

records.
the

undo

the

record

ARIES
record

redo

record

size

restrictions)

can

must

can

be

condi-

Under

be logged

before

the

record.

savepoints

and

partial

rollback

system)

redo

to

even logically
cached catalog

require

these

two

rollback.
Besides
allowing
allows the establishment
of

of transactions

will

necesin

situations.

Without
the support for partial
rollbacks,
(e.g., unique
key violation,
out-of-date
distributed
database
wasted work.

and/or

information

both

for partial
and total transaction
to be rolled back totally,
ARIES
the

the

handle

Support
transactions

(8)

constructed)

to log

can be
record,

total

such

savepoints.

recoverable
information

rollbacks

and

errors
in a
result

in

(9) Support for objects spanning
multiple
pages.
Objects
pages (e.g., an IMS “record”
which consists of multiple
scattered
over many pages). When an object is modified,

can span multiple
segments
may be
if log records are

written

for every

works

itself

does not treat

page

affected

by that

multipage

objects

update,

(10) Allows files to be acquired
or returned,
system.
ARIES
provides
the flexibility
namically

and

permanently

to

ARIES

in any special

the

fine.

ARIES

way.

any time, from or to the operating
of being able to return
files dy-

operating

system

(see

[19]

for

the

detailed
description
of a technique
to accomplish
this). Such an action is
considered
to be one that cannot be undone.
It does not prevent
the same
file from being
reallocated
to the database
system.
Mappings
between
objects (table spaces,
as in System R.
(11)

Some actions

etc.) and files

of a transaction

are not required

maybe

to be defined

committed

statically

even if the transaction

as

a whole is rolled back.
This
a dummy
CLR to implement

refers to the technique
of using the concept of
nested top actions.
File extension
has been

given

which

as an example

situation

could benefit

from

tions of this technique,
in the context
of hash-based
index management,
can be found in [59, 621.

this.

Other

storage

applica-

methods

and

(12) Efficient
checkpoints
(including
during
restart recovery).
By supporting
fuzzy checkpointing,
ARIES makes taking
a checkpoint
an efficient
operation. Checkpoints
can be taken
even when update
activities
and logging
are

going

on concurrently.

Permitting

processing
will help reduce
The dirty .pages
information

the

the number
redo pass.

are read

of pages

which

impact
written

checkpoints

even

during

restart

of failures
during
restart
recovery.
during
checkpointing
helps reduce
from

nonvolatile

storage

during

the

(13) Simultaneous
processing
of multiple
transactions
in forward
processing
and /or in rollback
accessing same page.
Since many transactions
could
simultaneously
be going forward
or rolling
back on a given page, the level
of concurrent
access supported
could be quite high.
Except for the short
duration
latching
which
has to be performed
any time
a page is being
ACM Transactions

on Database Systems, Vol. 17, No. 1, March 1992.

ARIES: A Transaction Recovery Method
physically
rollback,

modified
or examined,
rolling
back transactions

.

153

be it during
forward
processing
or during
do not affect one another
in any unusual

fashion.
(14) No locking or deadlocks
during
transaction
rollback.
is required
during
transaction
rollback,
no deadlocks
will

Since no locking
involve
transac-

tions that are rolling
back. Avoiding
locking
during
rollbacks
simplifies
not only the rollback
logic,
but also the deadlock
detector
logic.
The
deadlock
detector
need not worry about making
the mistake
of choosing
a
rolling
back transaction
as a victim
in the event of a deadlock (cf. System R
and R* [31, 49, 64]).
(15)

Bounded

logging

rollbacks.

Even

CLRS written
The number
time

during

restart

if repeated

is unaffected.
of log records

of transaction

in spite of repeated

failures

occur

during

failures

restart,

or of nested

the

number

of

This is also true if partial
rollbacks
are nested.
written
will be the same as that written
at the

rollback

during

normal

processing.

The latter

again

is

a fixed number
and is, usually,
equal to the number
of undoable
records
written
during
the forward
processing
of the transaction.
No log records
are written
during
the redo pass of restart.
(16)

Permits

faster

exploitation

restart.

Restart

of parallelism

and

can be made

faster

selective/deferred
by not doing

processing

for

all the needed

1/0s

synchronously
ARIES permits

one at a time while processing
the corresponding
log record.
the early identification
of the pages needing
recovery
and

the

of asynchronous

initiation

pages.

The

memory

pages

during

parallel

can be processed
the

redo

pass.

dling
of a given
transaction
processing
can be postponed

Undo

Fuzzy

image

copying

the

reading

as they

parallelism

transactions

(archive

data
the

performed

system

forward

traversal

of those
into

complete

han-

requires

the transaction

for

media

restart
offline

in parallel

recovery.

Media

are supported
very efficiently.
To
actual act of copying
can even be
(i.e.,

without

going

buffer
pool).
This can happen
even while
the latter
modifying
the information
being copied. During
media
(18) Continuation
repeats history

in

are brought

can be performed

dumping)

recovery
and image copying
of the
take advantage
of device geometry,
outside

for

by a single
process.
Some of the
to speed up restart
or to accommodate

devices. If desired, undo of loser
with new transaction
processing.
(17)

1/0s

concurrently

through

the

is accessing
recovery
only

and
one

of the log is made.
of loser transactions
after
and supports
the savepoint

a system
concept,

restart.
Since ARIES
we could, in the undo

pass, instead
of totally
rolling
back the loser transactions,
roll back each
loser only to its latest savepoint.
Locks must be acquired
to protect
the
transaction’s
uncommitted,
not undone
updates.
Later,
we could resume
the transaction
by invoking
its application
at a special entry
point
and
passing enough
be resumed.
(19)

Only

information

one backward

about

traversal

the savepoint

from

of log during

restart

which

execution

or media

is to

recovery.

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

154

C. Mohan

.

Both

during

media

Need

only

compensation

recovery
and restart
This is especially
important
in a slow medium
like tape.

redo

information

records

information.

are

in

never

So, on the average,

Support

for distributed

transactions.

Whether

does not affect
(22)

Early

compensation

undone
the

site

log

need

traversal
of
of the log is

records.

to contain

ARIES

Since

only

of log space consumed

space consumed

transactions.
a given

they

the amount

a transaction
rollback
will be half
processing
of that transaction.
(21)

one backward
if any portion

recovery

the log is sufficient.
likely to be stored
(20)

et al

redo

during

during

the forward

accommodates

distributed

is a coordinator

or a subordinate

site

ARIES.

release

of locks

during

transaction

rollback

and

deadlock

resolu-

tion using partial
rollbacks.
Because
ARIES
because it never undoes a particular
non-CLR

never
undoes
CLRS and
more than once, during
a

(partial)

first

rollback,

when

the transaction’s

object is undone and a CLR is written
on that object. This makes it possible
partial
rollbacks.
It should
from being

or both

dealing
with
long
Database
Manager.

to a particular

undo

and

redo

information.

This

may

be useful

for

fields,
as is the case in the 0S/2
Extended
Edition
In such instances,
for such data, the modified
pages

have to be forced

to nonvolatile

storage

before

commit.

media recovery and partial
rollbacks
can be supported
logged and for which updates shadowing
is done.

13.

update

be noted that ARIES does not prevent
the shadow page technique
used for selected portions
of the data to avoid logging
of only undo

information

would

very

for it, the system can release the lock
to consider resolving
deadlocks
using

will

Whether
depend

or not

on what

is

SUMMARY

In this

paper,

some of the

we presented

recovery

the

paradigms

ARIES
of System

recovery

method

and

R are inappropriate

showed

why

in the

WAL

context.
We dealt with
a variety
of features
that
are very important
in
building
and operating
an industrial-strength
transaction
processing
system.
Several
issues regarding
operation
logging,
fine-granularity
locking,
space
management,
and flexible
recovery
were discussed.
In brief, ARIES
accomplishes the goals that we set out with by logging
all updates
on a per-page
basis, using an LSN on every page for tracking
page state, repeating
history
during
restart
recovery
before undoing
the loser transactions,
and chaining
the CLRS to the predecessors
of the log records that they compensated.
Use of
ARIES

is not

restricted

to the

database

area

alone.

implementing
persistent
object-oriented
languages,
and transaction-based
operating
systems.
In fact,
QuickSilver
distributed
operating
system [401 and

It can also be used
recoverable
it is being
in a system

aid the backing
up of workstation
In this section, we summarize

data on a host [441.
as to which specific features

to which

give

specific

ACM Transactions

attributes

that

us flexibility

of ARIES

and efficiency.

on Database Systems, Vol. 17, No. 1, March 1992

for

file systems
used in the
designed
to
lead

ARIES: A Transaction Recovery Method
Repeating

history

exactly,

which

CLRS during

undos,

permits

the following,

chained

the UndoNxtLSN

using

(1) Record
within
records
logged.

in turn

field

implies

using

irrespective

155

.

LSNS

and writing

of whether

CLRS

are

or not:

level locking
to be supported
and records to be moved around
a page to avoid
storage
fragmentation
without
the moved
having
to be locked and without
the movements
having
to be

(2) Use only

one state

variable,

a log sequence

number,

per page.

(3) Reuse of storage released by one transaction
for the same transaction’s
later actions or for other transactions’
actions once the former
commits,
thereby

leading

to the

efficient

usage

of storage.

preservation

of clustering

of records

(4) The inverse of an action origianlly
performed
during
forward
of a transaction
to be different
from the action(s)
performed
undo
That

of that original
is, logical undo

undo

on the

(6) Recovery
of each page independently
relating
to transaction
state, especially

same

with

of other pages or of log
during
media recovery.

records

of transactions

(8) Selective

and undo

or deferred

restart,

processing

rollback

processing
during
the

concurrently

(7) If necessary,
the continuation
the time of system failure.

(9) Partial

the

action (e. g., class changes in the space map pages).
with recovery
independence
is made possible.

(5) Multiple
transactions
may
transactions
going forward.

transaction

and

to improve

data

page

which

of losers

were

in progress

concurrently

with

at
new

availability.

of transactions.

(10) Operation
logging
and logical
logging
of changes within
a page. For
example,
decrement
and increment
operations
may be logged,
rather
than the before- and after-images
of modified
data.
Chaining,
using the UndoNxtLSN
field,
forward
processing
permits
the following,
history

CLRS to log records written
during
provided
the protocol
of repeating

is also followed:

(1) The avoidance
CLRS.

This

of undoing

also makes

CLRS’

actions,

it unnecessary

thus

avoiding

to store undo

(2) The avoidance
of the undo of the same log record
processing
more than once.
(3) As a transaction

is being

rolled

back,

the ability

writing

information

written
to release

by partially

(4) Handling
partial
log, as in System
(5) Making

permanent,

rolling

rollbacks
R.
if

back
without

necessary

for

in CLRS.

during

object when all the updates to that object had been undone.
important
while
rolling
back a long transaction
or while
deadlock

CLRS

forward

the lock on an
This may
resolving

be
a

patching

the

the victim.
any special

actions

via

top

nested

like

actions,

some

of the

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

156

C. Mohan

.

et al.

changes made by a transaction,
irrespective
itself subsequently
rolls back or commits.
Performing

the analysis

(1) Checkpoints

pass before

to be taken

any

of whether

repeating

time

history

during

the

the

permits

redo

and

transaction

the following:
undo

passes

of

recovery.
(2) Files to be returned
ing dynamic
binding
(3) Recovery

of file-related

user data,
(4) Identifying
1/0s

to the operating
system dynamically,
between
database
objects and files.

could

information

concurrently

without

requiring

special

pages

possibly

requiring

be initiated

for them

treatment
redo,

with

volatile

storage
when

of

asynchronous

parallel

pages by eliminating
e.g., that some empty

been freed.

(6) Exploiting
opportunities
to avoid
writing
end. write
records
after
table

recovery

the redo pass starts.

(5) Exploiting
opportunities
to avoid redos on some
those pages from the dirty .pages table on noticing,
pages have

the

allow-

for the former.

so that

even before

thereby

and

reading
some pages during
redo, e.g., by
dirt y pages have been written
to non-

by

eliminating

those

the end. write

records

are encountered.

(7) Identifying
the transactions
locks could be reacquired

pages

from

the

dirty

.pages

in the in-doubt
and in-progress
states so that
for them
during
the redo pass to support

selective
or deferred
restart,
the continuation
of loser transactions
after
restart,
and undo of loser transactions
in parallel
with new transaction
processing.
13.1

Implementations

ARIES

forms

and Extensions

the basis

of the recovery

algorithms

used in the IBM

Research

prototype
systems Starburst
[871 and QuickSilver
[401, in the University
of
Wisconsin’s
EXODUS
and Gamma
database
machine
[201, and in the IBM
program
products
0S/2
Extended
Edition
Database
Manager
[71 and Workstation
history,

Data Save Facility/VM
has been implemented

[441. One feature
of ARIES, namely
repeating
in DB2 Version
2 Release 1 to use the concept

of nested top action
for supporting
segmented
tablespaces.
A simulation
study of the performance
of ARIES
is reported
in [981. The following
conclu“Simulation
results
indicate
the
sions from that
study are worth
noting:
success of the ARIES
recovery
method
in providing
fast recovery
from
failures,
caused by long intercheckpoint
intervals,
efficient
use of page LSNS,
log LSNS, and RecLSNs avoids redoing updates unnecessarily,
and the actual
recovery

skillfully.

Besides,

concurrency
control
and recovery
indicated
by the negligibly
small

load

is reduced

algorithms
difference

the

overhead

incurred

by

the

on transactions
is very low, as
between
the mean transaction

response time and the average
duration
of a transaction
if it ran alone in a
never failing
system.
This observation
also emerges
as evidence
that the
recovery method goes well with concurrency
control through
fine-granularity
locking,
an important
virtue. ”
ACM Transactions

on Database Systems, Vol. 17, No. 1, March 1992

.

157

of the

nested

ARIES: A Transaction Recovery Method
We have

extended

transaction
methods,

model
called

ARIES
(see [70,

ARIES

to make
85]).

/KVL,

it work

in the

Based

on ARIES,

ARIES/IM

and

context

we have

ARIES

developed

/LHS,

to

new

efficiently

provide
high concurrency
and recovery
for B ‘-tree
indexes
[57, 62] and for
hash-based
storage structures
[59]. We have also extended
ARIES to restrict
the amount

of repeating

of history

that

takes

place

for the loser

transactions

[691. We have designed
concurrency
control
and recovery
algorithms,
on ARIES,
for the N-way data sharing
(i. e., shared disks) environment
66,67,

68]. Commit.LSN,

that
exists
reevaluation
in

[54,

a method

which

takes

in every
page to reduce
the
overheads,
and also to improve

58,

processing,

60].

Although

we did not

messages

discuss

are

message

advantage

based
[65,

of the page.LSN

locking,
latching
and predicate
concurrency,
has been presented
an

important

logging

part

and recovery

of transaction
in this

paper.

ACKNOWLEDGMENTS

We have benefited
immensely
from the work that was
System R project and in the DB2 and IMS product
groups.

performed
We have

valuable
lessons by looking
at the experiences
with those
the source code and internal
documents
of those systems

systems. Access to
was very helpful.

The

from

Starburst

project

gave

us the

opportunity

to begin

design some of the fundamental
algorithms
of a transaction
into account experiences
with the prior systems. We would
edge

the

contributions

also like to thank
have adopted
our
Brian

Oki,

and Irv

Traiger

Erhard

of the

designers

of the

other

in the
learned

scratch

and

system, taking
like to acknowl-

systems.

We

would

our colleagues
in the research
and product
groups that
research
results.
Our thanks
also go to Klaus
Kuespert,
Rahm,

for their

Andreas

detailed

Reuter,

comments

Pat

Selinger,

Dennis

Shasha,

on the paper.

REFERENCES
1. BAKER, J., CRUS, R., AND HADERLE, D. Method for assuring atomicity of multi-row
update
operations in a database system. U.S. Patent 4,498,145, IBM, Feb. 19S5.
2. BADRINATH, B. R., AND RAMAMRITHAM, K.
Semantics-based
concurrency control: Beyond
3rd IEEE
International
Conference
on Data Engineering
commutativity.
In Proceedings
(Feb. 1987).
Concurrency
Control
and Recovery
in
3. BERNSTEIN, P., HADZILACOS, V., AND GOODMAN, N.
Database
Systems. Addison-Wesley,
Reading, Mass., 1987.
4. BORR, A. Robustness to crash in a distributed
database: A non-shared-memory
multi10th International
Conference
on Very Large Data Bases
processor approach. In Proceedings
(Singapore, Aug. 1984).
5. CHAMBERLAIN,D., GILBERT, A., AND YOST, R. A history of System R and SQL)Data System.
7th International
Conference
on Very Large
Data Bases (Cannes, Sept.
In Proceedings
1981).
ACM Trans.
6. CHANG, A., AND MERGEN, M. 801 storage: Architecture
and programming.
Comput. Syst., 6, 1 (Feb. 1988), 28-50.
7. CHANG, P. Y., AND MYRE, W. W.
0S/2 EE database manager: Overview and technical
ZBM Syst. J. 27, 2 (198S).
highlights.
schemes
8. COPELAND, G., KHOSHAFIAN, S., SMITH, M., AND VALDURIEZ, P. Buffering
International
Conference
on Data
Engineering
for permanent
data. In Proceedings
(Los Angeles, Feb. 1986).
ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

158

.

C. Mohan

et al.

9. CLARK, B. E., AND CORRTGAN,M. J.

Application

System/400

performance

characteristics.

IBM S@. J. 28, 3 (1989).
10. CHENG, J., LOOSELY, C., SHIBAMIYA, A., AND WORTHINGTON, P. IBM Database 2 perforIBM Sy.st. J. 23, 2 (1984).
mance: Design, implementation,
and tuning.
11. CRUS, R , HADERLE, D., AND HERRON, H. Method for managing
lock escalation
in a
multiprocessing,
multiprogramming
environment.
U.S. Patent 4,716,528, IBM, Dec. 1987.
IBM
Tech. Disclosure
12. CRUS, R., MALKEMUS, T., AND PUTZOLU, G. R. Index mini-pages
Bull. 26, 4 (April 1983), 5460-5463.
13. CRUS, R., PUTZOLU, F., AND MORTENSON, J. A
Incremental
data base log image copy IBM
!l’ec~. Disclosure Bull. 25, 7B (Dec. 1982), 3730-3732.
Bull. 25, 7B
14. CRUS, R., AND PUTZOLU, F. Data base allocation table. IBM Tech. Disclosure
(Dec. 1982), 3722-2724.
15. CRUS, R. Data recovery in IBM Database2.
IBM Syst. J. 23,2(1984).
Informix-Turbo,
In Proceedings LZEECornpcon
Sprmg’88(Feb.
-March l988),
16. CURTIS, R.
operating
17. DASGUPTA, P., LEBLANC, R., JR., AND APPELBE, W. The Clouds distributed
8th International
Conference
on Distributed
Computing
Systems
system. In Proceedings
(San Jose, Calif., June 1988).
AGuideto
INGRES. Addison-Wesley,
Reading, Mass., l987.
18. DATE, C.
data sets. IBM Tech. Disclosure
19. DEY, R., SHAN, M., AND TRAIGER, 1. Method fordropping
Bull. 25, 11A (April 1983), 5453-5455.
AND
20. DEWITT, D., GHANDEHARIZADEH, S., SCHNEIDER, D., BRICKER, A., HSIAO, H.-I.,
Data Eng.
RASMUSSEN,R. The Gamma database machine project. IEEE Trans. Knowledge
2, 1 (March 1990).
21. DELORME, D., HOLM, M., LEE, W., PASSE, P., RICARD, G., TIMMS, G., JR., AND YOUNGREN, L.
Database index journaling
for enhanced recovery. U.S. Patent 4,819,156, IBM, April 1989
The treatment
of
22. DIXON, G. N., BARRINGTON, G. D., SHRIVASTAVA, S., AND WHEATER, S. M.
persistent objects in Arjuna. Comput. J. 32, 4 (1989).
management.
Ph.D. dissertation,
Tech. Rep. CMU-CS-88-192,
23. DUCHAMP, D. Transaction
Carnegie-Mellon
Univ., Dec. 1988,
ACM
of database buffer management,
24. EFFEUSBERG, W., AND HAERDER, T. Principles
Trans. Database
Syst. 9, 4 (Dec. 1984).
25. ELHARDT, K , AND BAYER, R. A database cache for high performance and fast restart in
database systems. ACM Tram Database Syst. 9, 4 (Dec. 1984).
locking for
26. FEKETE, A., LYNCH, N., MERRITT, M., AND WEIHL, W. Commutativity-based
nested transactions.
Tech. Rep. MIT/LCS/TM-370.b,
MIT, July 1989,
Data base integrity
as provided for by a particular
data base management
27. FOSSUM, B
J. W. Klimbie and K. L. Koffeman, Eds., North-Holland,
system. In Data Base Management,
Amsterdam,
1974.
of concurrency control in IMS/VS
Fast Path.
28. GAWLICK, D., AND KINKADE, D. Varieties
IEEE Database
Eng. 8, 2 (June 1985).
management
in an object-oriented
database system.
29. GARZA, J., AND KIM, W. Transaction
ACM-SIGMOD
International
Conference
on Management
of Data (Chicago,
In Proceedings
June 1988).
CHAOS’%
Support for real-time
atomic transactions.
In
30. GHEITH, A., AND SCHWAN, K.
Proceedings
19th International
Symposium
on Fault-Tolerant
Computing
(Chicago, June
1989).
31. GRAY, J., MCJONES, P., BLASGEN, M., LINDSAY, B., LORIE, R., PRICE, T., PUTZOLU, F., AND
ACM
Comput.
TRAIGER, I. The recovery manager of the System R database manager.
Suru. 13, 2 (June 1981).
Systems–An
Aduanced
systems. In Operating
32. GRAY, J. Notes on data base operating
Course, R. Bayer, R. Graham, and G. Seegmuller,
Eds., LNCS Vol. 60, Springer-Verlag,
New York, 1978.
m database systems. J. ACM 35, 1 (Jan. 1988),
33. HADZILACOS, V, A theory of reliability
121-145.
S.yst. 13, 2 (1988),
hot spot data in DB-sharing
systems. Inf
34. HAERDER, T. Handling
155-166.
ACM Transactions

on Database Systems, Vol. 17, No. 1, March 1992

ARIES: A Transaction Recovery Method

.

159

35. HADERLE, D., AND JACKSON, R.

IBM Database 2 overview. IBM Syst. J. 23, 2 (1984).
Principles
of transaction
oriented database recovery–A
taxonomy. ACM CornPUt. Sure. 15, 4 (Dec. 1983).
37. HELLAND, P. The TMF application programming
interface: Program to program communication, transactions,
and concurrency in the Tandem NonStop system. Tandem Tech. Rep.
TR89.3, Tandem Computers, Feb. 1989.
36. HAERDER, T., AND REUTER, A.

38. HERLIHY, M.,
Proceedings

AND WEIHL, W.

7th

ACM

Hybrid

concurrency

SIGAC’T-SIGMOD-SIGART

Systems (Austin, Tex., March 1988).
39. HERLIHY, M., AND WING, J. M. Avalon:
17th
International
systems. In Proceedings
(Pittsburgh,
Pa., July 1987).

control

for abstract

Symposium

Language
Symposium

support
on

data

on Principles

for

reliable

Fault-Tolerant

types.

In

of Database

distributed
Computing

40. HASKIN, R., MALACHI, Y., SAWDON, W., AND CHAN, G. Recovery management
in QuickSilver. ACM !/’runs. Comput. Syst. 6, 1 (Feb. 1988), 82-108.
Dec. GG24-1652, IBM, April 1984.
41. IMS/ VS Version 1 Release 3 Recovery/Restart.
Programming.
Dec. SC26-4178, IBM, March 1986.
42. IMS/ VS Version 2 Application
43. IMS/ VS Extended
April 1987.

Recovery

44. IBM Workstation Data
1990.

Facility

(XRF):

Save Facility

/ VM:

Technical
General

Reference.
Information.

Dec. GG24-3153,

IBM,

Dec. GH24-5232,

IBM,

45. KORTH, H. Locking primitives
in a database system. JACM 30, 1 (Jan. 1983), 55-79.
46. LUM, V., DADAM, P., ERBE, R., GUENAUER, J., PISTOR, P., WALCH, G., WERNER, H., AND
WOODFILL, J. Design of an integrated
DBMS to support advanced applications.
In Proceedings International
Conference
on Foundations
of Data Organization
(Kyoto, May 1985).
47. LEVINE, F., AND MOHAN, C. Method for concurrent record access, insertion,
deletion and
alteration using an index tree. U.S. Patent 4,914,569, IBM, April 1990.
Isolation
Locking.
Dec. GG66-3193, IBM Dallas Systems
48. LEWIS, R. Z. ZMS Program
Center, Dec. 1990.
49. LINDSAY, B., HAAS, L., MOHAN, C., WILMS, P., AND YOST, R. Computation
and communication in R*: A distributed
database manager. ACM Trans. Comput. Syst. 2, 1 (Feb. 1984).
9th ACM Symposium
on Operating
Systems Principles
(Bretton Woods,
Also in Proceedings
Oct. 1983). Also available as IBM Res. Rep. RJ3740, San Jose, Calif., Jan. 1983.
50. LINDSAY, B., MOHAN, C., AND PIRAHESH, H. Method for reserving space needed for “rollBull. 29, 6 (Nov. 1986).
back” actions. IBM Tech. Disclosure
51. LISKOV, B.,

AND SCHEIFLER, R. Guardians
and actions: Linguistic
support for robust,
distributed
programs. ACM Trans. Program. Lang. Syst. 5, 3 (July 1983).
52. LINDSAY, B., SELINGER, P., GALTIERL C., GRAY, J., LORIE, R., PUTZOLU, F., TRAIGER, I., AND
WADE, B. Notes on distributed
databases. IBM Res. Rep. RJ2571, San Jose, Calif., July
1979.
53. MCGEE, W. C. The information
management
syste]m IMS/VS—Part
II: Data base faciliIBM Syst. J. 16, 2 (1977).
ties; Part V: Transaction processing facilities.
54. MOHAN, C., HADERLE, D., WANG, Y., AND CHENG, J.
Single table access using multiple
indexes: Optimization,
execution,
and concurrency
control techniques.
In Proceedings
International
Conference
on Extending
Data Base Technology
(Venice, March 1990). An
expanded version of this paper is available
as IBM Res. Rep. RJ7341, IBM Almaden
Research Center, March 1990.
55. MOHAN, C., FUSSELL, D., AND SILBERSCHATZ, A. Compatibility
and commutativity
of lock
modes. Znf Control 61, 1 (April 1984). Also available as IBM Res. Rep. RJ3948, San Jose,
Calif., July 1983.
56. MOSS, E., GRIFFETH, N., AND GRAHAM, M. Abstraction
in recovery management.
In
Proceedings
ACM SIGMOD
International
Conference
on Management
of Data (Washington,
D. C., May 1986).
57. MOHAN, C. ARIES /KVL: A key-value locking method for concurrency control of multiac16th International
Conference
tion transactions operating on B-tree indexes. In Proceedings
on Very Large Data Bases (Brisbane, Aug. 1990). Another version of this paper is available
as IBM Res. Rep. RJ7008, IBM Almaden Research Center, Sept. 1989.
ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992.

160

.

C. Mohan et al

58. MOHAN, C.

Commit -LSN: A novel and simple method for reducing locking and latching in
16th International
Conference
on Very Large
processing systems In Proceedings
Data l?ases (Brisbane, Aug. 1990). Also available as IBM Res. Rep. RJ7344, IBM Almaden
Research Center, Feb. 1990.
59 MOHAN, C. ARIES/LHS:
A concurrency control and recovery method using write-ahead
logging for linear hashing with separators. IBM Res. Rep., IBM Almaden Research Center,
Nov. 1990.
60. MOHAN, C. A cost-effective method for providing improved data avadability
during DBMS
of the 4th International
Workshop
on HLgh
restart recovery after a failure
In Proceedings
Performance
Transachon
Systems (Asilomar,
Calif., Sept. 1991). Also available as IBM Res.
Rep. RJ81 14, IBM Almaden Research Center, April 1991.
transaction

61. Moss, E., LEBAN, B., AND CHRYSANTHIS, P. Fine grained concurrency for the database
3rd IEEE International
Conference
on Data Engineering
(Los Angeles,
cache. In Proceedings
Feb. 1987),
62. MOHAN, C., AND LEVINE, F. ARIES/IM:
An efficient and high concurrency index management method using write-ahead
logging. IBM Res. Rep. RJ6846, IBM Almaden Research
Center, Aug. 1989.
63. MOHAN, C., AND LINDSAY, B. Efficient commit protocols for the tree of processes model of
2nd ACM SIGACT/
SIGOPS
Sympos~um
on Pridistributed
transactions.
In Proceedings
nciples of Distributed
Computing
(Montreal,
Aug. 1983). Also available
as IBM Res. Rep.
RJ3881, IBM San Jose Research Laboratory,
June 1983.
64. MOHAN, C., LINDSAY, B., AND OBERMARCK, R. Transaction
management
in the R* dktributed database management
system. ACM Trans. Database Syst. 11, 4 (Dec. 1986).
65. MOHAN, C., ANn NARANG, I. Recovery and coherency-control
protocols for fast intersystem
page transfer and tine-granularity
locking in a shared disks transaction
environment.
In
Proceedings
17th International
Conference
on Very Large
Data Bases (Barcelona,
Sept.
1991). A longer version is available
as IBM Res. Rep. RJ8017, IBM Almaden Research
Center, March 1991.
66. MOHAN, C., AND NARANG, I. Efficient
locking and caching of data in the multisystem
of the International
Conference
on
shared disks transaction
environment.
In proceedings
Extending
Database
Technology
(Vienna, Mar. 1992). Also available
as IBM Res. Rep.
RJ8301, IBM Almaden Research Center, Aug. 1991.
67. MOHAN, C., NARANG, I., AND PALMER, J. A case study of problems in migrating
to
distributed
computing: Page recovery using multiple logs in the shared disks environment.
IBM Res. Rep. RJ7343, IBM Almaden Research Center, March 1990.
68. MOHAN, C., NARANG, I., SILEN, S. Solutions
to hot spot problems in a shared disks
of the 4th International
Workshop
on High Perfortransaction environment.
In proceedings
mance Transaction
Systems (Asilomar,
Calif., Sept. 1991). Also available as IBM Res Rep.
8281, IBM Almaden Research Center, Aug. 1991.
69. MOHAN, C., AND PIRAHESH, H. ARIES-RRH:
Restricted repeating of history in the ARIES
7th International
Conference
on Data Engitransaction
recovery method. In Proceedings
neering
(Kobe, April
1991). Also available
as IBM Res. Rep. RJ7342, IBM Almaden
Research Center, Feb. 1990
70. MOHAN, C , AND ROTHERMEL, K.
Recovery protocol for nested transactions
using writeBull. 31, 4 (Sept 1988).
ahead logging. IBM Tech. Dwclosure
3rd
71. Moss, E. Checkpoint and restart in distributed
transaction
systems. In Proceedings
Symposium
on Reliability
in Dwtributed
Software
and Database
Systems
(Clearwater
Beach, Oct. 1983).
13th International
72. Moss, E
Log-based recovery for nested transactions.
In Proceedings
Conference
on Very Large Data Bases (Brighton,
Sept. 1987).
73. MOHAN, C., TIUEBER, K., AND OBERMARCK, R. Algorithms
for the management
of remote
backup databases for disaster recovery. IBM Res. Rep. RJ7885, IBM Almaden Research
Center, Nov. 1990.
74. NETT, E., KAISER, J., AND KROGER, R. Providing recoverability
in a transaction
oriented
6th International
Conference
on Distributed
distributed
operating system. In Proceedings
Computing
Systems (Cambridge,
May 1986).
ACM Transactions

on Database Systems, Vol. 17, No, 1, March 1992

ARIES: A Transaction Recovery Method
75. NOE, J., KAISER, J., KROGER, R., AND NETT, E.
locking.

The commit/abort
problem
GMD Tech. Rep. 267, GMD mbH, Sankt Augustin, Sept. 1987.

76. OBERMARCK, R. IMS/VS
Calif., July 1980.
77. O’NEILL, P.
(Dec. 1986).

The

program

Escrow

isolation

transaction

78. ONG, K.

SYNAPSE

approach

SIGMOD

Symposium

on Principles

feature.

ACM

method.

to database

IBM

.

161

in type-specific

Res. Rep. RJ2879,

San Jose,

Trans. Database Syst. 11, 4

recovery.

of Database

In Proceedings
3rd ACM
SIGACT(Waterloo, April 1984).
contention in a stock trading database: A

Systems

79. PEINL, P., REUTER, A., AND SAMMER, H. High
ACM SIGMOD
International
Conference
on Management
of Data
case study. In Proceedings
(Chicago, June 1988).
80. PETERSON,R. J., AND STRICKLAND, J. P. Log write-ahead protocols and IMS/VS logging. In

ACM SIGACT-SIGMOD

Proceedings

2nd

(Atlanta,

Ga., March

Symposium on Principles of Database Systems

1983).

81. RENGARAJAN, T. K., SPIRO, P., AND WRIGHT, W.
DBMS software. Digital Tech. J. 8 (Feb. 1989).
82. REUTER, A.

Softw. Eng.

A fast transaction-oriented
4 (July 1980).

scheme for UNDO

mechanisms
recovery.

of VAX

IEEE Trans.

SE-6,

83. REUTER, A.
SIGMOD

logging

“High availability

Concurrency

Symposium

on high-traffic

on Principles

84. REUTER, A. Performance
(Dec. 1984), 526-559.

analysis

data elements.

of Database

Systems

of recovery techniques.

ACM
SIGACTIn Proceedings
(Los Angeles, March 1982).

ACM Trans. Database Syst. 9,4

85. ROTHERMEL, K., AND MOHAN, C. ARIES/NT:
A recovery method based on write-ahead
15th International
Conference
on Very Large
logging fornested transactions.
In Proceedings
Data Bases (Amsterdam,
Aug. 1989). Alonger
version ofthis
paper is available as IBM
Res. Rep. RJ6650, lBMAlmaden
Research Center, Jan. 1989.
86. ROWE, L., AND STONEBRAKER, M.
The commercial INGRES epilogue. Ch. 3 in The ZNGRES Papers, Stonebraker,
M., Ed., Addson-Wesley, Reading, Mass., 1986.
87. SCHWARZ, P., CHANG, W., FREYTAG, J., LOHMAN, G., MCPHERSON, J., MOHAN, C., AND
Workshop
on
PIRAHESH, H. Extensibility
in the Starburst database system. In Proceedings
Object-Oriented
Data Base Systems (Asilomar,
Sept. 1986). Also available as IBM Res. Rep.
RJ5311, San Jose, Calif., Sept. 1986.
88. SCHWARZ,P. Transactions on typed objects. Ph.D. dissertation,
Carnegie Mellon Univ., Dec. 1984.

Tech. Rep. CMU-CS-84-166,

ACM
Trans.
89. SHASHA, D., AND GOODMAN, N. Concurrent
search structure
algorithms.
Database
Syst. 13, 1 (March 1988).
90. SPECTOR, A., PAUSCH, R., AND BRUELL, G. Came Lot: A flexible, distributed
transaction
IEEE
Compcon Spring
’88 (San Francisco, Calif., March
processing system. In Proceedings
1988).

91. SPRATT, L.

ACM
The transaction
resolution journal: Extending the before journal.
1985).
92. STONEBRAKER, M. The design of the POSTGRES storage system. In Proceedings
International
Conference
on Very Large Data Bases (Brighton,
Sept. 1987).
Syst.

Oper.

Rev. 19, 3 (July

IMSj VS Version 1 Release 3 Fast Path
93. STILLWELL, J. W., AND RADER, P. M.
Dec. G320-0149-0, IBM, Sept. 1984.
94. STRICKLAND, J., UHROWCZIK, P., AND WATTS, V. IMS/VS:
An evolving system.

13th

Notebook.
IBM

Syst.

J. 21, 4 (1982).
95.

high-performance,
THE TANDEM DATABASE GROUP. NonStop
SQL: A distributed,
Science Vol. 359,
high-availability
implementation
of SQL. In Lecture Notes in Computer
D. Gawlick, M. Haynie, and A. Reuter, Eds., Springer-Verlag,
New York, 1989.

96. TENG, J., AND GUMAER, R.
IBM

Syst.

97. TRAIGER, I.
Virtual
4 (Oct. 1982), 26-48.
98. VURAL, S.

Managing

IBM

Database

2 buffers

to maximize

performance.

ACM

Syst.

J. 23, 2 (1984).

memory

management

for database systems.

A simulation
study for the performance
recovery method. M. SC. thesis, Middle East Technical

Oper.

Rev.

16,

analysis of the ARIES transaction
Univ., Ankara, Feb. 1990.

ACM Transactions on Database Systems, Vol. 17, No. 1, March 1992,

162

.

C. Mohan et al.

WATSON, C. T., AND ABERLE, G. F
System/38 machine database support. In IBM Syst,
38/ Tech. Deu., Dec. G580-0237, IBM July 1980.
100. WEIKUM, G. Principles and realization
strategies of multi-level
transaction
management.
ACM Trans. Database
Syst. 16, 1 (Mar. 1991).
101. WEINSTEIN, M., PAGE, T., JR , LNEZEY, B., AND POPEK, G. Transactions
and synchroniza10th ACM Symposium
on Operating
tion in a distributed
operating system. In Proceedings
Systems Principles
(Orcas Island, Dec. 1985).
99

Received January

1989; revised November

1990; accepted April

1991

ACM TransactIons on Database Systems, Vol. 17, No. 1, March 1992


