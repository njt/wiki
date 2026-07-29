---
url: https://logicmatters.net/ifl/pdfs/IFL2_LM.pdf
title: "An Introduction to Formal Logic, 2nd Edition"
author: Peter Smith
date_fetched: 2026-07-29
date_published: 2020
description: "Partial text extract of a 420-page philosophy textbook on classical first-order quantification theory. Includes complete table of contents, front matter, preface, chapters 1–~13 (through propositional logic), and closing appendix/further reading. Middle chapters omitted."
---

NOTE FOR THE ANALYST: This is a partial text extract of a 420-page textbook,
Peter Smith, 'An Introduction to Formal Logic', 2nd edition (Logic Matters, 2020;
originally CUP 2020; free PDF at https://logicmatters.net/ifl/). The COMPLETE table
of contents is included below, followed by the front matter, preface, and the full
text of roughly the first 80 pages (chapters 1 through ~13), plus the closing
appendix and further-reading sections. Middle chapters are omitted for length —
rely on the table of contents for the shape of the omitted material.

===== TABLE OF CONTENTS AND FRONT MATTER =====
A friendly request to the reader!

As the Preface explains, this is a very slightly corrected reprint of the second
edition of my Introduction to Formal Logic, originally published in 2020 by CUP.
I am now able to make it freely available to download as a PDF. You can also
buy a print-on-demand physical copy at a price as low as I can make it from
Amazon. In these troubled times, giving students easy access to what they need
seems the least we can do.
   No introductory logic text will suit all readers. This one will be too longwinded
for some, and too brisk for others. Some readers will want more ‘informal logic’
at the outset; others with complain that I take over fifty pages before we start
formal logic proper. The tone will be too ‘mathematical’ for some, too relaxed
for others. And so it goes. I can’t please everyone. I can only hope that it works
for you!
   But if you do read the book – because it is set as a course text, or just because
you want to teach yourself some basic logic – do let me know how you find it. Are
there any particular sections that you find obscure? Are there parts that go too
fast/too slowly? Would you e.g. like more historical asides? More philosophical
asides? Oh, and did you spot any outright mistakes? Even quick notes saying ‘It
all went pretty well for me!’ are appreciated.
   I’m keen to know because I have it mind to write a third edition sooner or
later. Because the book is now self-published, it is a lot easier to keep improving
it, even if it means adding extra pages here or there. And I’d like to make the
book as readable and as useful as I can. So do please send comments to me,
peter smith at logicmatters.net. Thanks!

Peter Smith, before he retired, was Senior Lecturer in Philosophy at the Uni-
versity of Cambridge. His other books include Explaining Chaos (1998) and An
Introduction to Gödel’s Theorems (2007; 2013).
The page left intentionally blank
An Introduction to Formal Logic
Second edition



Peter Smith




Logic Matters
c Peter Smith 2003, 2020

All rights reserved. Permission is granted to distribute this PDF as a
complete whole, including this copyright page, for educational
purposes such as classroom use. Otherwise, no part of this publication
may be reproduced, distributed, or transmitted in any form or by any
means, including photocopying or other electronic or mechanical
methods, without the prior written permission of the author, except
in the case of brief quotations embodied in critical reviews and certain
other noncommercial uses permitted by copyright law. For permission
requests, write to peter smith@logicmatters.net.

First published by Cambridge University Press, 2003
Second edition, Cambridge University Press, 2020
Reprinted with corrections, Logic Matters, August 2020


A low-cost paperback of this book is available by print on demand from Amazon
ISBN 979-8-675-80394-1 Paperback

Additional resources for this publication at www.logicmatters.net,
where the latest version of this PDF will always be found.




  Since this revised edition is not produced by a publisher with a marketing depart-
  ment, your university librarian will not get to know about it in the usual way. You
  will therefore need to give them the details and ask them to order a printed copy
  for the library.
Contents

Preface                                          vii
1     What is deductive logic?                    1
2     Validity and soundness                      9
3     Forms of inference                         20
4     Proofs                                     28
5     The counterexample method                  39
6     Logical validity                           44
7     Propositions and forms                     52
Interlude: From informal to formal logic         59
8     Three connectives                          61
9     PL syntax                                  72
10    PL semantics                               81
11    ‘P’s, ‘Q’s, ‘α’s, ‘β’s – and form again    94
12    Truth functions                           104
13    Expressive adequacy                       113
14    Tautologies                               120
15    Tautological entailment                   127
16    More about tautological entailment        137
17    Explosion and absurdity                   145
18    The truth-functional conditional          148
19    ‘If’s and ‘→’s                            162
Interlude: Why natural deduction?               172
20    PL proofs: conjunction and negation       174
21    PL proofs: disjunction                    191
22    PL proofs: conditionals                   203
23    PL proofs: theorems                       211
24    PL proofs: metatheory                     216
Interlude: Formalizing general propositions     228
25    Names and predicates                      230
26    Quantifiers in ordinary language          240
                                                  v
vi                                      Contents

27    Quantifier-variable notation           249
28    QL languages                           258
29    Simple translations                    271
30    More on translations                   282
Interlude: Arguing in QL                     290
31    Informal quantifier rules              293
32    QL proofs                              299
33    More QL proofs                         315
34    Empty domains?                         329
35    Q-valuations                           333
36    Q-validity                             346
37    QL proofs: metatheory                  354
Interlude: Extending QL                      359
38    Identity                               361
39    QL= languages                          367
40    Definite descriptions                  375
41    QL= proofs                             382
42    Functions                              391
Appendix : Soundness and completeness        402
The Greek alphabet                           412
Further reading                              413
Index                                        415




vi
Preface

The first and second printed editions The world is not short of introductions to
logic aimed at philosophy students. They differ widely in pace, style, coverage,
and the ratio of formal work to philosophical commentary. Like other authors, my
initial excuse for writing yet another text was that I did not find one that offered
quite the mix that I wanted for my own students (first-year undergraduates doing
a compulsory logic course).
   Logicians are an argumentative lot and get quite heated when discussing the
best route into our subject for beginners. But, standing back from our differences,
we can all agree on this: in one form or another, the logical theory eventually
arrived at in this book – classical first-order quantification theory, to give it
its trade name – is a wonderful intellectual achievement by formally-minded
philosophers and philosophically-minded mathematicians. It is a beautiful theory
of permanent value, a starting point for all other modern work in logic. So we
care greatly about passing on this body of logical knowledge. And we write our
logic texts – like this one – in the hope that you too will come to appreciate
some of the elements of our subject, and even want to go on to learn more.
This book starts from scratch, and initially goes pretty slowly; I make no apology
for working hard at the outset to nail down some basic ideas. The pace gradually
picks up as we proceed and as the idea of a formal approach to logic becomes
more and more familiar. But almost everything should remain quite accessible
even to those who start off as symbol-phobic – especially if you make the effort
to read slowly and carefully.
   You should check your understanding by tackling at least some of the routine
exercises at the ends of chapters. There are worked answers available at the
book’s online home, www.logicmatters.net. These are often quite detailed. For
example, while the book’s chapters on formal proofs aim to make the basic
principles really clear, the online answers provide extended ‘examples classes’
exploring proof-strategies, noting mistakes to avoid, etc. So do make good use
of these further resources.
   As well as exercises which test understanding, there are also a few starred
exercises which introduce additional ideas you really ought to know about, or
which otherwise ask you to go just a little beyond what is in the main text.
The first edition of this book concentrated on logic by trees. Many looking for a
course text complained about this. This second edition, as well as significantly

                                                                                 vii
viii                                                                      Preface

revising all the other chapters, replaces the chapters on trees with chapters on
a natural deduction proof system, done Fitch-style. Which again won’t please
everyone! So the chapters on trees are still available, in a revised form. But to
keep the length of the printed book under control, you will find these chapters
– together with a lot of other relevant material including additional exercises –
at the book’s website.
This PDF version The second edition is now made freely available as a PDF
download (note, by the way, that internal cross-references are live links). Apart
from altering this Preface to note the new publication arrangements, and tak-
ing the opportunity to correct a very small number of misprints, the book is
unchanged. However, . . .
A third edition? As just noted, in adding the chapters on natural deduction
to the second edition, I had to relegate (versions of) the first edition’s chapters
on trees to the status of online supplements. But this was always a second-best
solution. Ideally, I would have liked to have covered both trees and natural
deduction (while carefully arranging things so that the reader who only wanted
to explore one of these still had a coherent path through the book). I now hope
over the coming months to be able to revert to that plan, and so eventually
produce a third edition.
Thanks Many more people have helped me at various stages in the writing and
rewriting of this book than I can now remember.
   Generations of Cambridge students more or less willingly road-tested versions
of the lecture handouts which became the first edition of this book, and I learnt
a lot from them. Then Dominic Gregory and Alexander Paseau read and com-
mented on full drafts of the book, and the late Laurence Goldstein did very
much more than it was reasonable to expect of a publisher’s reader. After pub-
lication, many people then sent further corrections which made their way into
later reprintings, in particular Joseph Jedwab. Thanks to all of them.
   While writing the second edition, I have greatly profited from comments
from, among many others, Mauro Allegranza, David Auerbach, Roman Borschel,
David Furcy, Anton Jumelet, Jan von Plato, and especially Norman Birkett and
Rowsety Moid. Exchanges on logical Twitter, a surprisingly supportive resource,
suggested some memorable examples and nice turns of phrase – particularly from
Joel D. Hamkins. Scott Weller was extremely generous in sending many pages of
corrections. Matthew Manning was equally eagle-eyed, and also made a particu-
larly helpful suggestion about how best to arrange the treatment of metatheory.
At a late stage, David Makinson rightly pressed me hard to be clearer in distin-
guishing between types of rules of inference. And my daughter Zoë gave especially
appreciated feedback in the last phases of writing. I am again very grateful to
all of them.
Most of all, I must thank my wife Patsy without whose love and support neither
version of this book would ever have been finished.


viii
1 What is deductive logic?

The core business of logic is the systematic evaluation of arguments for internal
cogency. And the kind of internal cogency that will especially concern us in this
book is logical validity.
   But these brief headlines leave everything to be explained. What do we mean
here by ‘argument’ ? What do we mean by ‘internal cogency’ ? What do we
mean, more particularly, by ‘logical validity’ ? And what kinds of ‘systematic’
evaluation of arguments are possible? This introductory chapter makes a gentle
start on answering these questions.

1.1 What is an argument?
By ‘argument’ we mean a chain of reasoning, short or long, in support of some
conclusion. So we must distinguish arguments from mere disagreements and
disputes. The children who shout at each other ‘You did’, ‘I didn’t’, ‘Oh yes,
you did’, ‘Oh no, I didn’t’ are certainly disagreeing: but they are not arguing in
our sense – they are not yet giving any reasons in support of one claim or the
other.
   Reason-giving arguments are the very stuff of all serious inquiry, whether it is
philosophy or physics, economics or experimental psychology. Of course, we also
deploy reasoned arguments in the course of everyday, street-level, inquiry into
the likely winner of next month’s election, the best place to train as a lawyer, or
what explains our team’s losing streak. We want our opinions to be true; that
means that we should aim to have good reasons backing up our opinions, so
raising the chances of getting things right. This in turn means that we have an
interest in being skilful reasoners, using arguments which really do support their
conclusions.

1.2 Kinds of evaluation
Logic, then, is concerned with evaluating stretches of reasoning. Take this really
simple example, and call it argument A. Suppose you hold
          (1) All philosophers are eccentric.
I then introduce you to Jack, telling you that he is a philosopher. So you come
to believe
                                                                                 1
1 What is deductive logic?

           (2)   Jack is a philosopher.
Putting these two thoughts together, you infer
           (3)   Jack is eccentric.
And the first point to make is that this bit of reasoning can now be evaluated
along two quite independent dimensions:
    First, we can ask whether this argument’s premisses A(1) and A(2) are
    true. Are the ‘inputs’ to your inference step correct? A(1) is in fact very
    disputable. And perhaps I have made a mistake and A(2) is false as well.
    Second, we can ask about the quality of the inference step, the move which
    takes you from the premisses A(1) and A(2) to the conclusion A(3). In
    this particular case, the inference step is surely absolutely compelling: the
    conclusion really does follow from the premisses. We have agreed that it
    may be open to question whether the premisses are actually true. However,
    if they are assumed to be true (assumed ‘for the sake of argument’, as we
    say), then we have to agree that the conclusion is true too. There’s just no
    way that A(1) and A(2) could be true and yet A(3) false. To assert that
    Jack is a philosopher and that all philosophers are eccentric, but go on to
    deny that Jack is eccentric, would be implicitly to contradict yourself.
Generalizing, it is one thing to consider whether an argument starts from true
premisses; it is another thing to consider whether it moves on by reliable inference
steps. Yes, we typically want our arguments to pass muster on both counts. We
typically want both to start from true premisses and to reason by steps which
can be relied on to take us to further truths. But it is important to emphasize
that these are distinct aims.
   The premisses of arguments can be about all sorts of topics: their truth is
usually no business of the logician. If we are arguing about historical matters,
then it is the historian who is the expert about the truth of our premisses; if we
are arguing about some question in physics, then the physicist is the one who
might know whether our premisses are true; and so on. The central concern of
logic, by contrast, is not the truth of initial premisses but the way we argue
from a given starting point – the logician wants to know when an argument’s
premisses, supposing that we accept them, do indeed provide compelling grounds
for also accepting its conclusion. It is in this sense that logic is concerned with
the ‘internal cogency’ of our reasoning.


1.3 Deduction vs. induction
(a) Argument A’s inference step is absolutely compelling: if A’s premisses are
true, then its conclusion is guaranteed to be true too. Here is a similar case:
    B      (1) Either Jill is in the library or she is in the bookshop.
           (2) Jill isn’t in the library.
        So (3) Jill is in the bookshop.

2
                                                           Deduction vs. induction

Who knows whether the initial assumptions, the two premisses, are true or not?
But we can immediately see that the inference step is again completely water-
tight. If premisses B(1) and B(2) are both true, then B(3) cannot conceivably
fail to be true.
   Now consider the following contrasting case (to illustrate that not all good
reasoning is of this type). Here you are, sitting in your favourite café. Unrealistic
philosophical scepticism apart, you are thoroughly confident that the cup of
coffee you are drinking is not going to kill you – for if you weren’t really confident,
you wouldn’t be calmly sipping as you read this, would you? What justifies your
confidence?
   Well, you believe the likes of:
  C       (1) Cups of coffee from GreatBeanz that looked and tasted just fine
              haven’t killed anyone in the past.
          (2) This present cup of GreatBeanz coffee looks and tastes just fine.
These premisses, or something rather like them, sustain your cheerful belief that
          (3)   This present cup of GreatBeanz coffee won’t kill you.
The inference that moves from the premisses C(1) and C(2) to the conclusion
C(3) is, in the circumstances, surely perfectly reasonable: other things being
equal, the facts recorded in C(1) and C(2) do give you excellent grounds for
believing that C(3) is true. However – and here is the quite crucial contrast with
the earlier ‘Jack’ and ‘Jill’ examples – it is not the case that the truth of C(1)
and C(2) absolutely guarantees C(3) to be true too.
   Perhaps someone has slipped a slow-acting tasteless poison into the coffee,
just to make the logical point that facts about how things have always been in
the past don’t guarantee that the trend will continue in the future.
   Fortunately for you, C(3) is no doubt true. The tasteless poison is a fantasy.
Still, it is a coherent fantasy. It illustrates the point that your grounds C(1) and
C(2) for the conclusion that the coffee is safe to drink are strictly speaking quite
compatible with the falsity of that conclusion. Someone who agrees to C(1) and
C(2) and yet goes on to assert the opposite of C(3) might be saying something
highly improbable, but they won’t actually be contradicting themselves. We can
make sense of the idea of C(1) and C(2) being true and yet C(3) false.
   In summary then, there is a fundamental difference between the ‘Jack’ and
‘Jill’ examples on the one hand, and the ‘coffee’ example on the other. In the
‘Jack’ and ‘Jill’ cases, the premisses absolutely guarantee the conclusion. There
is no conceivable way that A(1) and A(2) could be true and yet A(3) false:
likewise, if B(1) and B(2) are true then B(3) just has to be true too. Not so
with the ‘coffee’ case: it is conceivable that C(1) and C(2) are true while C(3) is
false. What has happened in the past is a very good guide to what will happen
next (and what else can we rely on?); but reasoning from past to future isn’t
completely watertight.

(b) We need some terminology to mark this fundamental difference. We will
introduce it informally for the moment:

                                                                                     3
1 What is deductive logic?

    If an inference step from premisses to a conclusion is completely watertight,
    i.e. if the truth of the premisses absolutely guarantees the truth of the
    conclusion, then we say that this inference step is deductively valid.
    Equivalently, when an inference step is deductively valid, we will say that
    its premisses deductively entail its conclusion.

The inferential moves in A and B count as being deductively valid. Contrast
the ‘coffee’ argument C. That argument involves reasoning from past cases to a
new case in a way which leaves room for error, however unlikely. This kind of
extrapolation from the past to the future, or more generally from some sample
cases to further cases, is standardly called inductive. The inference in C might
be inductively strong – meaning that the conclusion is highly probable, assuming
the premisses are true – but the inference is not deductively valid.
   We should stress that the deductive/inductive distinction is not the distinction
between good and bad reasoning. The ‘coffee’ argument is a perfectly decent one.
It involves the sort of usually reliable reasoning to which we have to trust our
lives, day in, day out. It is just that the inference step here doesn’t completely
guarantee that the conclusion is true, even assuming that the stated premisses
are true.

(c) What makes for reliable (or reliable enough) inductive inferences is a very
important and decidedly difficult topic. But it is not our topic in this book,
which is deductive logic. That is to say, we will here be concentrating on the
assessment of arguments which aim to use deductively valid inferences, where
the premisses are supposed to deductively entail the conclusion.
   We will give a sharper definition of the general notion of deductive validity
at the beginning of the next chapter, §2.1. Later, by the time we get to §6.2,
we will have the materials to hand to define a rather narrower notion which –
following tradition – we will call logical validity. And this narrower notion of
logical validity will then become our main focus in the remainder of the book.
But for the next few chapters, we continue to work with the wider initial notion
that we’ve called deductive validity.


1.4 Just a few more examples
The ‘Jack’ and ‘Jill’ arguments are examples where the inference steps are ob-
viously deductively valid. Compare this next argument:
    D      (1) All Republican voters support capital punishment.
           (2) Jo supports capital punishment.
        So (3) Jo is a Republican voter.
The inference step here is equally obviously invalid. Even if D(1) and D(2) are
true, D(3) doesn’t follow. Maybe lots of people in addition to Republican voters
support capital punishment and Jo is one of them.
   How about the following argument?
4

===== BODY: CHAPTERS 1 THROUGH ~13 =====
Preface

revising all the other chapters, replaces the chapters on trees with chapters on
a natural deduction proof system, done Fitch-style. Which again won’t please
everyone! So the chapters on trees are still available, in a revised form. But to
keep the length of the printed book under control, you will find these chapters
– together with a lot of other relevant material including additional exercises –
at the book’s website.
This PDF version The second edition is now made freely available as a PDF
download (note, by the way, that internal cross-references are live links). Apart
from altering this Preface to note the new publication arrangements, and taking the opportunity to correct a very small number of misprints, the book is
unchanged. However, . . .
A third edition? As just noted, in adding the chapters on natural deduction
to the second edition, I had to relegate (versions of) the first edition’s chapters
on trees to the status of online supplements. But this was always a second-best
solution. Ideally, I would have liked to have covered both trees and natural
deduction (while carefully arranging things so that the reader who only wanted
to explore one of these still had a coherent path through the book). I now hope
over the coming months to be able to revert to that plan, and so eventually
produce a third edition.
Thanks Many more people have helped me at various stages in the writing and
rewriting of this book than I can now remember.
Generations of Cambridge students more or less willingly road-tested versions
of the lecture handouts which became the first edition of this book, and I learnt
a lot from them. Then Dominic Gregory and Alexander Paseau read and commented on full drafts of the book, and the late Laurence Goldstein did very
much more than it was reasonable to expect of a publisher’s reader. After publication, many people then sent further corrections which made their way into
later reprintings, in particular Joseph Jedwab. Thanks to all of them.
While writing the second edition, I have greatly profited from comments
from, among many others, Mauro Allegranza, David Auerbach, Roman Borschel,
David Furcy, Anton Jumelet, Jan von Plato, and especially Norman Birkett and
Rowsety Moid. Exchanges on logical Twitter, a surprisingly supportive resource,
suggested some memorable examples and nice turns of phrase – particularly from
Joel D. Hamkins. Scott Weller was extremely generous in sending many pages of
corrections. Matthew Manning was equally eagle-eyed, and also made a particularly helpful suggestion about how best to arrange the treatment of metatheory.
At a late stage, David Makinson rightly pressed me hard to be clearer in distinguishing between types of rules of inference. And my daughter Zoë gave especially
appreciated feedback in the last phases of writing. I am again very grateful to
all of them.
Most of all, I must thank my wife Patsy without whose love and support neither
version of this book would ever have been finished.

viii

1 What is deductive logic?
The core business of logic is the systematic evaluation of arguments for internal
cogency. And the kind of internal cogency that will especially concern us in this
book is logical validity.
But these brief headlines leave everything to be explained. What do we mean
here by ‘argument’ ? What do we mean by ‘internal cogency’ ? What do we
mean, more particularly, by ‘logical validity’ ? And what kinds of ‘systematic’
evaluation of arguments are possible? This introductory chapter makes a gentle
start on answering these questions.

1.1 What is an argument?
By ‘argument’ we mean a chain of reasoning, short or long, in support of some
conclusion. So we must distinguish arguments from mere disagreements and
disputes. The children who shout at each other ‘You did’, ‘I didn’t’, ‘Oh yes,
you did’, ‘Oh no, I didn’t’ are certainly disagreeing: but they are not arguing in
our sense – they are not yet giving any reasons in support of one claim or the
other.
Reason-giving arguments are the very stuff of all serious inquiry, whether it is
philosophy or physics, economics or experimental psychology. Of course, we also
deploy reasoned arguments in the course of everyday, street-level, inquiry into
the likely winner of next month’s election, the best place to train as a lawyer, or
what explains our team’s losing streak. We want our opinions to be true; that
means that we should aim to have good reasons backing up our opinions, so
raising the chances of getting things right. This in turn means that we have an
interest in being skilful reasoners, using arguments which really do support their
conclusions.

1.2 Kinds of evaluation
Logic, then, is concerned with evaluating stretches of reasoning. Take this really
simple example, and call it argument A. Suppose you hold
(1) All philosophers are eccentric.
I then introduce you to Jack, telling you that he is a philosopher. So you come
to believe

1

1 What is deductive logic?
(2)

Jack is a philosopher.

Putting these two thoughts together, you infer
(3)

Jack is eccentric.

And the first point to make is that this bit of reasoning can now be evaluated
along two quite independent dimensions:
First, we can ask whether this argument’s premisses A(1) and A(2) are
true. Are the ‘inputs’ to your inference step correct? A(1) is in fact very
disputable. And perhaps I have made a mistake and A(2) is false as well.
Second, we can ask about the quality of the inference step, the move which
takes you from the premisses A(1) and A(2) to the conclusion A(3). In
this particular case, the inference step is surely absolutely compelling: the
conclusion really does follow from the premisses. We have agreed that it
may be open to question whether the premisses are actually true. However,
if they are assumed to be true (assumed ‘for the sake of argument’, as we
say), then we have to agree that the conclusion is true too. There’s just no
way that A(1) and A(2) could be true and yet A(3) false. To assert that
Jack is a philosopher and that all philosophers are eccentric, but go on to
deny that Jack is eccentric, would be implicitly to contradict yourself.
Generalizing, it is one thing to consider whether an argument starts from true
premisses; it is another thing to consider whether it moves on by reliable inference
steps. Yes, we typically want our arguments to pass muster on both counts. We
typically want both to start from true premisses and to reason by steps which
can be relied on to take us to further truths. But it is important to emphasize
that these are distinct aims.
The premisses of arguments can be about all sorts of topics: their truth is
usually no business of the logician. If we are arguing about historical matters,
then it is the historian who is the expert about the truth of our premisses; if we
are arguing about some question in physics, then the physicist is the one who
might know whether our premisses are true; and so on. The central concern of
logic, by contrast, is not the truth of initial premisses but the way we argue
from a given starting point – the logician wants to know when an argument’s
premisses, supposing that we accept them, do indeed provide compelling grounds
for also accepting its conclusion. It is in this sense that logic is concerned with
the ‘internal cogency’ of our reasoning.

1.3 Deduction vs. induction
(a) Argument A’s inference step is absolutely compelling: if A’s premisses are
true, then its conclusion is guaranteed to be true too. Here is a similar case:
B

2

(1) Either Jill is in the library or she is in the bookshop.
(2) Jill isn’t in the library.
So (3) Jill is in the bookshop.

Deduction vs. induction
Who knows whether the initial assumptions, the two premisses, are true or not?
But we can immediately see that the inference step is again completely watertight. If premisses B(1) and B(2) are both true, then B(3) cannot conceivably
fail to be true.
Now consider the following contrasting case (to illustrate that not all good
reasoning is of this type). Here you are, sitting in your favourite café. Unrealistic
philosophical scepticism apart, you are thoroughly confident that the cup of
coffee you are drinking is not going to kill you – for if you weren’t really confident,
you wouldn’t be calmly sipping as you read this, would you? What justifies your
confidence?
Well, you believe the likes of:
C

(1) Cups of coffee from GreatBeanz that looked and tasted just fine
haven’t killed anyone in the past.
(2) This present cup of GreatBeanz coffee looks and tastes just fine.

These premisses, or something rather like them, sustain your cheerful belief that
(3)

This present cup of GreatBeanz coffee won’t kill you.

The inference that moves from the premisses C(1) and C(2) to the conclusion
C(3) is, in the circumstances, surely perfectly reasonable: other things being
equal, the facts recorded in C(1) and C(2) do give you excellent grounds for
believing that C(3) is true. However – and here is the quite crucial contrast with
the earlier ‘Jack’ and ‘Jill’ examples – it is not the case that the truth of C(1)
and C(2) absolutely guarantees C(3) to be true too.
Perhaps someone has slipped a slow-acting tasteless poison into the coffee,
just to make the logical point that facts about how things have always been in
the past don’t guarantee that the trend will continue in the future.
Fortunately for you, C(3) is no doubt true. The tasteless poison is a fantasy.
Still, it is a coherent fantasy. It illustrates the point that your grounds C(1) and
C(2) for the conclusion that the coffee is safe to drink are strictly speaking quite
compatible with the falsity of that conclusion. Someone who agrees to C(1) and
C(2) and yet goes on to assert the opposite of C(3) might be saying something
highly improbable, but they won’t actually be contradicting themselves. We can
make sense of the idea of C(1) and C(2) being true and yet C(3) false.
In summary then, there is a fundamental difference between the ‘Jack’ and
‘Jill’ examples on the one hand, and the ‘coffee’ example on the other. In the
‘Jack’ and ‘Jill’ cases, the premisses absolutely guarantee the conclusion. There
is no conceivable way that A(1) and A(2) could be true and yet A(3) false:
likewise, if B(1) and B(2) are true then B(3) just has to be true too. Not so
with the ‘coffee’ case: it is conceivable that C(1) and C(2) are true while C(3) is
false. What has happened in the past is a very good guide to what will happen
next (and what else can we rely on?); but reasoning from past to future isn’t
completely watertight.
(b) We need some terminology to mark this fundamental difference. We will
introduce it informally for the moment:

3

1 What is deductive logic?
If an inference step from premisses to a conclusion is completely watertight,
i.e. if the truth of the premisses absolutely guarantees the truth of the
conclusion, then we say that this inference step is deductively valid.
Equivalently, when an inference step is deductively valid, we will say that
its premisses deductively entail its conclusion.
The inferential moves in A and B count as being deductively valid. Contrast
the ‘coffee’ argument C. That argument involves reasoning from past cases to a
new case in a way which leaves room for error, however unlikely. This kind of
extrapolation from the past to the future, or more generally from some sample
cases to further cases, is standardly called inductive. The inference in C might
be inductively strong – meaning that the conclusion is highly probable, assuming
the premisses are true – but the inference is not deductively valid.
We should stress that the deductive/inductive distinction is not the distinction
between good and bad reasoning. The ‘coffee’ argument is a perfectly decent one.
It involves the sort of usually reliable reasoning to which we have to trust our
lives, day in, day out. It is just that the inference step here doesn’t completely
guarantee that the conclusion is true, even assuming that the stated premisses
are true.
(c) What makes for reliable (or reliable enough) inductive inferences is a very
important and decidedly difficult topic. But it is not our topic in this book,
which is deductive logic. That is to say, we will here be concentrating on the
assessment of arguments which aim to use deductively valid inferences, where
the premisses are supposed to deductively entail the conclusion.
We will give a sharper definition of the general notion of deductive validity
at the beginning of the next chapter, §2.1. Later, by the time we get to §6.2,
we will have the materials to hand to define a rather narrower notion which –
following tradition – we will call logical validity. And this narrower notion of
logical validity will then become our main focus in the remainder of the book.
But for the next few chapters, we continue to work with the wider initial notion
that we’ve called deductive validity.

1.4 Just a few more examples
The ‘Jack’ and ‘Jill’ arguments are examples where the inference steps are obviously deductively valid. Compare this next argument:
D

(1) All Republican voters support capital punishment.
(2) Jo supports capital punishment.
So (3) Jo is a Republican voter.

The inference step here is equally obviously invalid. Even if D(1) and D(2) are
true, D(3) doesn’t follow. Maybe lots of people in addition to Republican voters
support capital punishment and Jo is one of them.
How about the following argument?

4

Generalizing
E

(1) Most Irish people are Catholics.
(2) Most Catholics oppose abortion on demand.
So (3) At least some Irish people oppose abortion on demand.

Leave aside the question of whether the premisses are in fact correct (that’s not
a matter for logicians: it needs sociological investigation to determine the distribution of religious affiliation among the Irish, and to find out what proportion of
Catholics support their church’s teaching about abortion). What we can ask here
– from our armchairs, so to speak – is whether the inference step is deductively
valid: if the premisses are true, then must the conclusion be true too?
Well, whatever the facts of the case, it is at least conceivable that the Irish are
a tiny minority of the Catholics in the world. And it could also be that nearly all
the other (non-Irish) Catholics oppose abortion, and hence most Catholics do,
even though none of the Irish oppose abortion. But then E(1) and E(2) would
be true, yet E(3) false. So the truth of the premisses doesn’t by itself absolutely
guarantee the truth of the conclusion (there are possible situations in which the
premisses would be true and the conclusion false). Hence the inference step is
not deductively valid.
Here’s another argument: is the inference step deductively valid this time?
F

(1) Some philosophy students admire all logicians.
(2) No philosophy student admires anyone irrational.
So (3) No logician is irrational.

With a little thought you should arrive at the right answer here too (we will
return to this example in Chapter 3).
Still, at the moment, faced with examples like our last three, all you can do
is to cast around hopefully, trying to work out somehow or other whether the
truth of the premisses does guarantee the truth of the conclusion. It would be
good to be able to proceed more systematically and to have a range of general
techniques for evaluating arguments for deductive validity. That’s what logical
theory aims to provide.
Indeed, ideally, we would like techniques that work mechanically, that can
be applied to settle questions of validity as routinely as we can settle simple
arithmetical questions by calculation. We will have to wait to see how far this is
possible. For the moment, we will just say a little more about what makes any
kind of more systematic approach possible (whether mechanical or not).

1.5 Generalizing
(a) Here again is our first sample mini-argument with its deductively valid
inference step:
A

(1) All philosophers are eccentric.
(2) Jack is a philosopher.
So (3) Jack is eccentric.

Now compare it with the following arguments:

5

1 What is deductive logic?
A0

(1) All logicians are cool.
(2) Russell is a logician.
So (3) Russell is cool.

A00

(1) All robins have red breasts.
(2) Tweety is a robin.
So (3) Tweety has a red breast.

A000

(1) All post-modernists write nonsense.
(2) Derrida is a post-modernist.
So (3) Derrida writes nonsense.

We can keep going on and on, churning out arguments to the same pattern, all
involving equally valid inference steps.
It is plainly no accident that these arguments share the property of being
internally cogent. Comparing these examples makes it clear that the deductive
validity of the inference step in the original argument A hasn’t anything specifically to do with philosophers or with the notion of being eccentric. Likewise the
validity of the inference step in argument A0 hasn’t anything specifically to do
with logicians. And so on. There is a general principle involved here:
From a pair of premisses, one saying that all things of a given kind have a
certain property, the other saying that a particular individual is indeed of
that given kind, we can validly infer the conclusion that this individual has
the property in question.
(b) However, this wordy version is not the most perspicuous way of representing
the shared inferential principle at work in the A-family. Focusing on arguments
expressed in English, we can instead say something like this:
Any inference step of the form
All F are G
n is F
So: n is G
is deductively valid.
Here the letters ‘n’, ‘F ’, ‘G’ are being used to help exhibit a skeletal pattern
of argument. We can think of ‘n’ as standing in for a name for some individual
person or thing, while ‘F ’ and ‘G’ stand in for expressions which pick out kinds
of things (like ‘philosopher’ or ‘robin’) or properties (like ‘eccentric’). A bit more
loosely, we can also use ‘is G’ to stand in for other expressions attributing properties like ‘has a red breast’ or ‘writes nonsense’. It then doesn’t matter how
we flesh out the italicized argument template or schema, as we might call it.
Any sensible enough way of substituting suitable expressions for ‘n’, ‘(is) F ’ and
‘(is) G’, and then tidying the result into reasonable English, will yield another
argument with a deductively valid inference step.
Some forms or patterns of inference are deductively reliable, then, meaning
that every inference step which is an instance of the same pattern is valid. Other
forms aren’t reliable. Consider the following pattern of inference:

6

Summary
Most G are H
Most H are K
So: At least some G are K.
(The choice of letters we use to display a pattern of inference is optional, of
course! – a point we return to in §3.2(d).) This is the type of inference involved
in the ‘Irish’ argument E, and we now know that it isn’t trustworthy.
(c) We said at the outset that logic aims to be a systematic study. We can now
begin to see how to get some generality into the story. Noting that the same
form of inference step can feature in many different particular arguments, we
can aim to examine generally reliable forms of inference. And we can then hope
to explore how such reliable forms relate to each other and to the particular
arguments which exploit them, giving us a more systematic treatment. A lot
more about this in due course.

1.6 Summary
We can evaluate a piece of reasoning along two different dimensions. We
can ask whether its premisses are actually true. And we can ask whether
the inference from the premisses is internally cogent – i.e., assuming the
premisses are true, do they really support the truth of the conclusion? Logic
is concerned with the second dimension of evaluation.
We are setting aside inductive arguments (and other kinds of non-conclusive
reasoning). We will be concentrating on arguments involving inference steps
that purport to be deductively valid. In other words, we are going to be
concerned with deductive logic, the study of inferences that aim to strictly
guarantee their conclusions, assuming the truth of their premisses.
Arguments typically come in families whose members share good or bad
types of inferential move; by looking at such general patterns or forms of
inference we can hope to make logic more systematic.

Exercises 1
By ‘conclusion’ we do not mean what concludes a passage of reasoning in the sense of
what is stated at the end. We mean what the reasoning aims to establish – and this
might in fact be stated at the outset. Likewise, ‘premiss’ does not mean (contrary to
what the Concise Oxford Dictionary says!) ‘a previous statement from which another
is inferred’. Reasons supporting a certain conclusion, i.e. the inputs to an inference,
might well be given after that target conclusion has been stated. Note too that the
move from supporting reasons to conclusions can be signalled by inference markers
other than ‘so’.
What are the premisses, inference markers, and conclusions of the following arguments? Which of these arguments do you suppose involve deductively valid reasoning?
Why? (Just improvise, and answer the best you can!)

7

1 What is deductive logic?
(1) Most politicians are corrupt. After all, most ordinary people are corrupt –
and politicians are ordinary people.
(2) Anyone who is well prepared for the exam, even if she doesn’t get an A grade,
will at least get a B. Jane is well prepared, so she will get at least a B grade.
(3) John is taller than Mary and Jane is shorter than Mary. So John is taller than
Jane.
(4) At eleven, Fred is always either in the library or in the coffee bar. And assuming he’s in the coffee bar, he’s drinking an espresso. Fred was not in the
library at eleven. So he was drinking an espresso then.
(5) The Democrats will win the election. There’s only a week to go. The polls
put them 20 points ahead, and a lead of 20 points with only a week to go to
polling day can’t be overturned.
(6) Dogs have four legs. Fido is a dog. Therefore Fido has four legs.
(7) Jekyll isn’t the same person as Hyde. The reason is that no murderers are
sane – but Hyde is a murderer, and Jekyll is certainly sane.
(8) All the slithy toves did gyre and gimble in the wabe. Some mome raths are
slithy toves. Hence some mome raths did gyre and gimble in the wabe.
(9) Some but not all philosophers are logicians. All logicians are clever. Hence
some but not all philosophers are clever.
(10) No experienced person is incompetent. Jenkins is always blundering. No competent person is always blundering. Therefore Jenkins is inexperienced.
(11) Many politicians take bribes. Most politicians have extra-marital affairs. So
many people who take bribes have extra-marital affairs.
(12) Kermit is green all over. Hence Kermit is not red all over.
(13) Every letter is in a pigeonhole. There are more letters that there are pigeonholes. So some pigeonhole contains more than one letter.
(14) There are more people than there are hairs on anyone’s head. So at least two
people have the same number of hairs on their head.
(15) Miracles cannot happen. Why? Because, by definition, a miracle is an event
incompatible with the laws of nature. And everything that happens is always
consistent with the laws of nature.
(16) (Lewis Carroll) No interesting poems are unpopular among people of real
taste. No modern poetry is free from affectation. All your poems are on the
subject of soap bubbles. No affected poetry is popular among people of real
taste. Only a modern poem would be on the subject of soap bubbles. Therefore
none of your poems are interesting.
(17) ‘If we found by chance a watch or other piece of intricate mechanism we
should infer that it had been made by someone. But all around us we do find
intricate pieces of natural mechanism, and the processes of the universe are
seen to move together in complex relations; we should therefore infer that
these too have a maker.’
(18) ‘I can doubt that the physical world exists. I can even doubt whether my
body really exists. I cannot doubt that I myself exist. So I am not my body.’

8

2 Validity and soundness
We have introduced the idea of an inference step being deductively valid and,
equivalently, the idea of some premisses deductively entailing a conclusion. This
chapter explores this notion of validity/entailment a bit further, though still in
an informal way. We also emphasize the special centrality of deductive reasoning
in serious inquiry.

2.1 Validity defined more carefully
(a) We said in §1.3 that an inference step is deductively valid if it is completely
watertight – in other words, assuming that the inference’s premisses are true, its
conclusion is absolutely guaranteed to be true as well.
But talk of an inference being ‘watertight’ and talk of a conclusion being
‘guaranteed’ to be true is too metaphorical for comfort. So – now dropping
the explicit ‘deductively’ – here is a less metaphorical definition, already hinted
at:
An inference step is valid if and only if there is no possible situation in
which its premisses would be true and its conclusion false. Equivalently, in
such a case, we will say that the inference’s premisses entail its conclusion.
Take the plural ‘premisses’, here and in similar statements, to cover the onepremiss case too. Then this definition characterizes what is often called the
classical concept of validity. It is, however, only as clear as the notion of a
‘possible situation’. So we certainly need to pause over this.
(b)
A

Consider the following bit of reasoning:
Jo jumped out of a twentieth floor window (without parachute,
safety net, etc.) and fell unimpeded onto a concrete pavement. So
she was injured.

Let’s grant that, with the laws of nature as they are, there is no way in which the
premiss could be true and the conclusion false (assume that we are talking about
here on Earth, that Jo is a human, not a beetle, etc.). In the actual world, falling
unimpeded onto concrete from twenty floors up will always produce serious – very
probably fatal – injury. Does that make the inference from A’s premiss to its
conclusion valid?

9

2 Validity and soundness
No. It isn’t, let’s agree, physically possible in our actual circumstances to jump
without being injured. But we can coherently, without self-contradiction, conceive of a situation in which the laws of nature are different or are miraculously
suspended, and someone jumping from twenty floors up can float delicately down
like a feather. We can imagine a capricious deity bringing about such a situation.
In this very weak sense, it is possible that A’s premiss could be true while its
conclusion is false. And that is enough for the inference to be deductively invalid.
(c) So our definition of validity is to be read like this: a deductively valid inference is one where it is not possible even in the most generously inclusive sense
– roughly, it is not even coherently conceivable – that the inference’s premisses
are true and its conclusion false.
Let’s elaborate. We ordinarily talk of many different kinds of possibility –
about what is physically possible (given the laws of nature), what is technologically possible (given our engineering skills), what is politically possible (given
the make-up of the electorate), what is financially possible (given the state of
your bank balance), and so on. These different kinds of possibility are related in
various ways. For example, something that is technologically possible has to be
physically possible, but may not be financially possible for anyone to engineer.
But note that if some situation is physically possible (or technologically possible, etc.) then, at the very least, a description of that situation has to be
internally coherent. In other words, if some situation is M -ly possible (for any
suitable modifier ‘M ’), then it must be possible in the thin sense that the very
idea of such a situation occurring isn’t ruled out as nonsense. The notion of possibility we need in defining validity is possibility in this weakest, most inclusive,
sense. And from now on, when we talk in an unqualified way about possibility,
it is this very weak sense that we will have in mind.
(d) Kinds of possibility go together with matching kinds of necessity – ‘necessary’ means ‘not-possibly-not’. For example, it is legally necessary for an enforceable will to be signed if and only if it is not legally possible for a will to
be enforceable yet not signed. It is physically necessary that massive objects
gravitationally attract each other if and only if it is not physically possible that
those objects not attract each other. And so on. Generalizing, we can massage
such equivalences into the standard form
It is M -ly necessary that C if and only if it is not M -ly possible that not-C,
where ‘C ’ stands in for some statement, and ‘not-C ’ stands in for the denial of
that statement (so asserting not-C is equivalent to saying it is false that C).
Likewise, our very weak inclusive notion of possibility goes with a correspondingly strong notion of necessity: it is necessary that C in this sense just in case
it is not even weakly possible that not-C. Putting it another way,
It is necessarily true that C if and only if it is true that C in every possible
situation, in our most inclusive sense of ‘possible’.

10

Consistency and equivalence
From now on, when we talk in an unqualified way about necessity, it is this very
strong sense that we will have in mind.
And yes, being necessarily true is a very strong requirement, as strong as can
be. However, it is one that can be met in some cases. For example, it is necessary
in this sense that Jill is not both married and unmarried (at the same time, in
the same jurisdiction). It is necessary in our strong sense that all triangles have
three sides (in any coherently conceivable situation, the straight-sided figures
with three internal angles are three-sided). Again, it is necessary that whenever
a mother smiles, a parent smiles.
It is equally necessary that if all philosophers are eccentric and Jack is a
philosopher, then Jack is eccentric. And this last example illustrates a general
point, which gives us an equivalent definition of deductive validity:
An inference step is valid if and only if it is necessary, in the strongest
sense, that if the inference’s premisses are true, so is its conclusion. For
short, valid inferences are necessarily truth-preserving.

2.2 Consistency and equivalence
We need a term for the kind of thing that can feature as a premiss or conclusion
in an argument – let’s use the traditional ‘proposition’.
We will return in §§7.3–7.5 to consider the tricky question of the nature of
propositions: for the moment, we just assume that propositions can sensibly be
said to be true or false in various situations (unlike commands, questions, etc.).
That’s all we need, if we are to introduce two more notions that swim in the same
conceptual stream as the notion of validity – two notions which again concern
the truth of propositions across different possible situations.
(a)

First, we will say:

One or more propositions are (jointly) inconsistent if and only if there is no
possible situation in which these propositions are all true together.
Or to put it another way, some propositions are (jointly) consistent if and only
if there is a possible situation in which they are all true together. Both times,
we mean ‘possible’ in the weakest, most inclusive, sense.
For example, taken together, the three propositions All philosophers are eccentric, Jack is a philosopher and Jack is not eccentric are jointly inconsistent.
But any two of them are consistent, as is any one of them taken by itself.
Suppose that an inference is valid, meaning that there is no possible situation
in which its premisses would be true and its conclusion false. Given our new
definition, this is equivalent to saying that the premisses and the denial of the
conclusion are jointly inconsistent. We can therefore offer another definition of
the classical conception of validity which is worth highlighting:

11

2 Validity and soundness
An inference step is valid if and only if its premisses taken together with
the denial of the conclusion are inconsistent.
This means that when we develop logical theory it matters little whether we
take the notion of validity or the notion of consistency as fundamental.
(b)

Here is the second new notion we want:

Two propositions are equivalent if and only if they are true in exactly the
same possible situations.
Again this is closely tied to the notion of a valid inference, as follows:
The propositions A and B are equivalent if and only if A entails B and B
entails A.
Why so? Suppose A and B are equivalent. Then in any situation in which A
is true, B is true – i.e. A entails B. Exactly similarly, B entails A. Conversely,
suppose A entails and is entailed by B; then in any situation in which A is true,
B is true and vice versa – i.e. A and B are equivalent.

2.3 Validity, truth, and the invalidity principle
(a) To assess whether an inference step is deductively valid, it is usually not
enough to look at what is the case in the actual world; we also need to consider
alternative possible situations, alternative ways things might conceivably have
been – or, as some say, alternative possible worlds.
Take, for example, the following argument:
B

(1) No Welshman is a great poet.
(2) Shakespeare is a Welshman.
So (3) Shakespeare is not a great poet.

The propositions here are all false about the actual world. But that doesn’t settle
whether the premisses of B entail its conclusion. In fact, the inference step here
is valid; there is no situation at all – even a merely possible one, in the weakest
sense – in which the premisses would be true and the conclusion false. In other
words, any possible situation which did make the premisses true (Shakespeare
being brought up some miles to the west, and none of the Welsh, now including
Shakespeare, being good at verse) would also make the conclusion true.
Here are two more arguments stamped out from the same mould:
C

(1) No human being is a dinosaur.
(2) Bill Clinton is a human being.
So (3) Bill Clinton is not a dinosaur.

D

(1) No one whose middle name is ‘William’ is a Democrat.
(2) George W. Bush’s middle name is ‘William’.
So (3) George W. Bush is not a Democrat.

12

Validity, truth, and the invalidity principle
These two arguments involve valid inferences of the same type – in case C taking
us from true premisses to a true conclusion, in case D taking us, as it happens,
from false premisses to a true conclusion.
You might wonder about the last case: how can a truth be validly inferred
from two falsehoods? But note again: validity is only a matter of internal cogency.
Someone who believes D(3) on the basis of the premisses D(1) and D(2) would be
using a reliable form of inference, but they would be arriving at a true conclusion
by sheer luck. Their claimed grounds D(1) and D(2) for believing D(3) would
have nothing to do with why D(3) in fact happens to be true. (Aristotle noted
the point. Discussing valid arguments with false premisses, he remarks that
the conclusion may yet be correct, but “only in respect to the fact, not to the
reason”.)
There can be mixed cases too, where valid inferences have some true and some
false premisses. So, allowing for these, we have in summary:
A valid inference can have actually true premisses and a true conclusion,
(some or all) actually false premisses and a false conclusion, or (some or all)
false premisses and a true conclusion.
The only combination ruled out by the definition of validity is a valid inference step’s having all true premisses and yet a false conclusion. Deductive
validity is about the necessary preservation of truth – and therefore a valid
inference step cannot take us from actually true premisses to an actually
false conclusion.
That last point is worth really emphasizing. Here’s an equivalent version:
The invalidity principle An inference step with actually true premisses and
an actually false conclusion must be invalid.
We will see in Chapter 5 how this principle gets to do important work when
combined with the fact that arguments come in families sharing the same kind
of inference step.
(b) So much for the valid inference steps. What about the invalid ones? What
combinations of true/false premisses and true/false conclusions can these have?
An invalid inference step can have any combination of true or false premisses
and a true or false conclusion.
Take for example the following silly argument:
E

(1) My mother’s maiden name was ‘Moretti’.
(2) Every computer I’ve ever bought is a Mac.
So (3) The Pavel Haas Quartet is my favourite string quartet.

This is invalid: there is a readily conceivable situation in which the premisses
are true and conclusion false. But what about the actual truth/falsity of the
premisses and conclusion? I’m not going to say. Any combination of actually
true or false premisses and a true or false conclusion is quite compatible with
the invalidity of the hopeless inferential move here.

13

2 Validity and soundness
So note: having true premisses and a false conclusion is enough to make an
inference invalid – in other words, it is a sufficient condition for invalidity. But
that isn’t required for being invalid – it isn’t a necessary condition.

2.4 Inferences and arguments
(a) So far, we have spoken of inference steps in arguments as being valid or
invalid. But it is more common simply to describe arguments as being valid or
invalid. In this usage, we say that an argument is valid if and only if the inference
step from its premisses to its conclusion is valid. (Or at least, that’s what we
say about one-step arguments like the toy examples we’ve been looking at: we’ll
consider multi-inference arguments later.)
Talking of valid arguments in this way is absolutely standard. But it can
mislead beginners. After all, saying that an argument is ‘valid’ can sound like an
all-in endorsement. So let’s stress the point: to say that a (one-step) argument
is valid in our sense is only to commend the cogency of the inferential move
between premisses and conclusion, is only to say that the conclusion really does
follow from the premisses. A valid argument must be internally cogent, but can
still have premisses that are quite hopelessly false.
For example, take this argument:
F

(1) Whatever Donald Trump says is true.
(2) Donald Trump says that the Flying Spaghetti Monster exists.
So (3) It is true that the Flying Spaghetti Monster exists.

Since the inference step here is evidently valid, this argument counts as valid,
in our standard usage. Which might well strike the unwary as a distinctly odd
thing to say!
You just have to learn to live with this way of speaking of valid arguments. But
then we need, of course, to have a different term for arguments that do deserve
all-in endorsement – arguments which both start from truths and proceed by
deductively cogent inference steps. The usual term is ‘sound’. So:
A (one-step) argument is valid if and only if the inference step from the
premisses to the conclusion is valid.
A (one-step) argument is sound if and only if it has all true premisses and
the inference step from those premisses to the conclusion is valid.
A few older texts use ‘sound’ to mean what we mean by ‘valid’. But everyone
agrees there is a key distinction to be made between mere deductive cogency
and the all-in virtue of making a cogent inference and having true premisses;
there’s just a divergence over how this agreed distinction should be labelled.
(b) When we are deploying deductive arguments, we typically want our arguments to be sound in the sense we just defined. But not always! For we sometimes
want to argue, and argue absolutely compellingly, from premisses that we don’t

14

‘Valid’ vs ‘true’
believe to be all true. For example, we might aim to deduce some obviously false
consequence from premisses which we don’t accept, in the hope that a disputant
will agree that this consequence has to be rejected and so come to agree with us
that the premisses aren’t all true. Sometimes we even want to argue compellingly
from premisses we believe are inconsistent, precisely in order to bring out their
inconsistency.
(c)

Note three easy consequences of our definition of soundness:

(1) any sound argument has a true conclusion;
(2) no pair of sound arguments can have conclusions inconsistent with each
other;
(3) no sound argument has inconsistent premisses.
Why do these claims hold? For the following reasons:
(10 ) A sound argument starts from true premisses and involves a necessarily truth-preserving inference move – so it must end up with a true
conclusion.
0
(2 ) Since a pair of sound arguments will have a pair of true conclusions,
this means that the conclusions are true together. If they actually are
true together, then of course they can be true together. And if they
can be true together then (by definition) the conclusions are consistent
with each other.
0
(3 ) Since inconsistent premisses cannot all be true together, an argument
starting from those premisses cannot satisfy the first of the conditions
for being sound.
Note though that if we replace ‘sound’ by ‘valid’ in (1) to (3), the claims become
false. More about valid arguments with inconsistent premisses in due course.
(d) What about arguments where there are intermediate inference steps between the initial premisses and the final conclusion (after all, real-life arguments
very often have more than one inference step)? When should we say that they
are deductively cogent?
As a first shot, we can say that such extended arguments are valid when each
inference step along the way is valid. But we will see in §4.4 that more needs to
be said. So we’ll hang fire on the question of deductive cogency for multi-step
arguments until then.

2.5 ‘Valid’ vs ‘true’
Let’s pause for a brief terminological sermon, one which is important enough to
be highlighted! The propositions that occur in arguments as premisses and conclusions are assessed for truth / falsity. Inference steps in arguments are assessed
for validity / invalidity. These dimensions of assessment, as we have stressed,
are fundamentally different. We should therefore keep the distinction carefully
marked. Hence:

15

2 Validity and soundness
Despite the common misuse of the terms, resolve to never again say that
a premiss or conclusion or other proposition is ‘valid’ when you mean it is
true. And never again say that an argument is ‘true’ when you mean that
it is valid (or sound).

2.6 What’s the use of deduction?
(a) As we noted before, deductively valid inferences are not the only acceptable
inferences. Concluding that Jo is injured from the premiss she fell twenty storeys
onto concrete is of course perfectly sensible. Reasoning of this kind is very often
reliable enough to trust your life to: the premisses may render the conclusion a
certainty for all practical purposes. But such reasoning isn’t deductively valid.
Now consider a more complex kind of inference. Take the situation of the detective, Sherlock let’s say. Sherlock assembles a series of clues and then solves the
crime by an inference to the best explanation (an old term for this is ‘abductive’
reasoning). In other words, the detective arrives at some hypothesis that best
accommodates all the strange events and bizarre happenings. In the ideally satisfying detective story, this hypothesis strikes us (once revealed) as obviously
giving the right explanation – why didn’t we think of it? Thus: why is the
bed bolted to the floor so it can’t be moved? Why is there a useless bell rope
hanging by the bed? Why is the top of the rope fixed near a ventilator grille
leading through into the next room? What is that strange music heard at the
dead of night? All the pieces fall into place when Sherlock infers a dastardly
plot to kill the sleeping heiress in her unmovable bed by means of a poisonous
snake, trained to descend the rope through the ventilator grille in response to
the snake-charmer’s music. But although this is an impressive ‘deduction’ in one
everyday sense of the term, it is not deductively valid reasoning in the logician’s
sense. We may have a number of clues, and the detective’s hypothesis H may
be the only plausible explanation we can find: but in the typical case it won’t
be a contradiction to suppose that, despite the way all the evidence stacks up,
hypothesis H is actually false. That won’t be an inconsistent supposition, only
perhaps a very unlikely one. Hence, the detective’s plausible ‘deductions’ are not
(normally) valid deductions in the logician’s sense.
Now, if our inductive reasoning about the future on the basis of the past is not
deductive, and if inference to the best explanation is not deductive either, you
might well ask: just how interesting is the idea of deductively valid reasoning? To
make the question even more worrisome, consider that paradigm of systematic
rationality, scientific reasoning. We gather data, and try to find the best theory
that fits; rather like the detective, we aim for the best explanation of the actually
observed data. But a good theory goes well beyond merely summarizing the data.
In fact, it is precisely because the theory goes beyond what is strictly given in
the data that the theory can be used to make novel predictions. Since the excess
content isn’t guaranteed by the data, however, the theory cannot be validly
deduced from observation statements.

16

An illuminating circle?
So again you well might very well be inclined to ask: if deductive inference
doesn’t feature even in the construction of scientific theories, why should it be
particularly interesting? True, it might be crucial for mathematicians – but what
about the rest of us?
(b) But that’s far too quick! We can’t simply deduce a scientific theory from
the data it is based on. However, it doesn’t at all follow that deductive reasoning
plays no essential part in scientific reasoning.
Here’s a picture of what goes on in science. Inspired by patterns in the data,
or by models of the underlying processes, or by analogies with other phenomena, etc., we conjecture that a certain theory is true. Then we use the theory
(together with assumptions about ‘initial conditions’, etc.) to deduce a range
of testable predictions. The first stage, the conjectural stage, may involve flair
and imagination, rather than brute logic, as we form our hypotheses. But at
the second stage, having made our conjectures, we need to infer testable consequences; and now this does involve deductive logic. For we need to examine
what else must be true if the hypothesized theory is true: we want to know what
our theory deductively entails. Then, once we have deduced testable predictions,
we can seek to test them. Often our predictions prove to be false. We have to
reject the theory – or else we have to revise it to accommodate the new data,
and then go on to deduce more testable consequences. The process is typically
one of repeatedly improving and revising our hypotheses, deducing consequences
which we can test, and then refining the hypotheses again in the light of test
results.
This so-called hypothetico-deductive model of science (which highlights the
role of deductive reasoning from theory to predictions) no doubt needs a lot of
development and amplification and refinement. But with science thus conceived,
we can see why deduction is absolutely central to the enterprise after all.
And what goes for science, narrowly understood, goes for rational inquiry more
widely: deductive reasoning may not be the whole story, but it is an ineliminable
core. Whether we are doing physics or metaphysics, mathematics or moral philosophy, worrying about climate change or just the next election, we need to
think through what our assumptions logically commit us to, and know when
our reasoning goes wrong. That’s why logic, which teaches us how to appraise
passages of reasoning for deductive validity, matters.

2.7 An illuminating circle?
Aristotle wrote in his Prior Analytics that “a deduction is speech (logos) in
which, certain things having been supposed, something . . . results of necessity
because of their being so”. Our own definition – an argument is valid just if there
is no possible situation in which its premisses would be true and its conclusion
false – picks up the idea that correct deductions are necessarily truth-preserving.
And we have tried to say enough to convey an initial understanding of this idea,
and to give a sense of the central importance of deductively valid reasoning.

17

2 Validity and soundness
Note, though, that – as we explained how to understand our classical definition
of validity – we in effect went round in a rather tight circle of interconnected
ideas. We first defined validity in terms of what is possible, where we are to
understand ‘possible’ in the weakest, most inclusive, sense. We then further
elucidated this relevant weak notion of possibility in terms of what is coherently
conceivable. But what is it for a situation to be coherently conceivable? A minimal condition would seem to be that a story about the conceived situation
must not involve some hidden self-contradiction. Which presumably is to be
understood, in the end, as meaning that we cannot validly deduce a contradiction
from the story. So now it seems that we have defined deductive validity in a way
that needs to be explained, at least in part, by invoking the notion of a valid
deduction. How concerning is this?
This kind of circularity is often unavoidable when we are trying to elucidate
some really fundamental web of concepts. Often, the best we can do is start
with a rough-and-ready, partial, grasp of various ideas, and then aim to draw
out their relationships and make distinctions, clarifying and sharpening the ideas
as we explore the web of interconnected notions – so going round, we hope, in
an illuminating circle. In the present case, this is what we have tried to do as we
have illustrated and explained the notion of deductive validity and interrelated
notions. We have at least said enough, let’s hope, to get us started and to guide
our investigations over the next few chapters.
Still, notions of necessity and possibility do remain genuinely puzzling. So
– before you worry that we are starting to build the house of logic on shaky
foundations – we should highlight that, looking ahead,
We will later be giving sharp technical definitions of notions of validity for
various special classes of argument, definitions which do not directly invoke
troublesome notions of necessity/possibility.
These definitions will, however, remain recognizably in the spirit of our preliminary elucidations of the classical concept.

2.8 Summary
Inductive arguments from past to future, and inferences to the best explanation, are not deductive; but the hypothetico-deductive picture shows why
there can still be a crucial role for deductive inference in scientific and other
empirical inquiry.
Our preferred definition of deductive validity is: an inference step is valid if
and only if there is no possible situation in which its premisses are true and
the conclusion false. Call this the classical conception of validity.
The notion of possibility involved in this definition is the weakest and most
inclusive.

18

Exercises 2
Equivalently, a valid inference step is necessarily truth-preserving, in the
strongest sense of necessity. NB: being ‘necessarily truth-preserving’ (this is
a useful shorthand we will make much use of) is defined by a conditional:
necessarily, if the premisses are true, so is the conclusion.
A one-step argument is valid if and only if its inference step is valid. An
argument which is valid and has true premisses is said to be sound.

Exercises 2
(a) Which of the following claims are true and which are false? Explain why the true
claims hold good, and give counterexamples to the false claims.
(1) The premisses and conclusion of an invalid argument must together be inconsistent.
(2) If an argument has false premisses and a true conclusion, then the truth of
the conclusion can’t really be owed to the premisses: so the argument cannot
really be valid.
(3) Any inference with actually true premisses and a true conclusion is truthpreserving and so valid.
(4) You can make a valid argument invalid by adding extra premisses.
(5) You can make a sound argument unsound by adding extra premisses.
(6) You can make an invalid argument valid by adding extra premisses.
(7) If some propositions are consistent with each other, then adding a further
true proposition to them can’t make them inconsistent.
(8) If some propositions are jointly inconsistent, then whatever propositions we
add to them, the resulting propositions will still be jointly inconsistent.
(9) If some propositions are jointly consistent, then their denials are jointly inconsistent.
(10) If some propositions are jointly inconsistent, then we can pick any one of them,
and validly infer that it is false from the remaining propositions as premisses.
(b*)

Show that

(1) If A entails C, and C is equivalent to C 0 , then A entails C 0 .
(2) If A entails C, and A is equivalent to A0 , then A0 entails C.
(3) If A and B entail C, and A is equivalent to A0 , then A0 and B entail C.
Can we therefore say that ‘equivalent propositions behave equivalently in arguments’ ?

19

3 Forms of inference
We saw in the first chapter how arguments can come in families which share the
same type or pattern or form of inference step. Evaluating this shareable form of
inference for reliability will then simultaneously give a verdict on a whole range
of arguments depending on the same sort of inferential move.
In this chapter, we say a little more about the idea of forms of inference and
about the schemas we use to display them.

3.1 More forms of inference
(a)
A

Consider again the argument:
(1) No Welshman is a great poet.
(2) Shakespeare is a Welshman.
So (3) Shakespeare is not a great poet.

This is deductively valid, and likewise for the parallel ‘Clinton’ and ‘Bush’ arguments which we stated in §2.3. The following is valid too:
A0

(1) No three-year old understands quantum mechanics.
(2) Daisy is three years old.
So (3) Daisy does not understand quantum mechanics.

We can improvise endless variations on this theme. And plainly, the inference
steps in these arguments aren’t validated by anything especially to do with poets,
presidents, or three-year-olds. Rather, they are all valid for the same reason,
namely the meaning of ‘no’ and ‘not’ and the way that these logical notions
distribute in the premisses and conclusion (the same way in each argument). So:
Any inference step of the form
No F is G
n is F
So: n is not G
is deductively valid.
As before (§1.5), ‘n’ in the italicized schema stands in for some name, while
‘F ’ and ‘G’ (with or without an ‘is’) stand in for expressions that sort things
into kinds or are used to attribute properties. We can call letters used in this
way schematic variables (‘schematic’ because they feature in schemas, ‘variable’

20

More forms of inference
because they can stand in for various different replacements). Substitute appropriate bits of English for the variables in our schema, smooth out the language
as necessary, and we’ll get a valid argument.
That’s rather rough. If we want to be a bit more careful, we can put the
underlying principle or rule of inference like this:
From a pair of propositions, one saying that nothing of some given kind has
a certain property, the other saying that a particular individual is of the
given kind, we can validly infer the conclusion that this individual lacks the
property in question.
But surely the schematic version is easier to understand. It is more perspicuous
to display the form of an inference by using a symbolic schema, rather than
trying to describe that form in cumbersome words.
(b)

Here is another argument which we have met before (§1.4):

B

(1) Some philosophy students admire all logicians.
(2) No philosophy student admires anyone irrational.
So (3) No logician is irrational.

Do the premisses here deductively entail the conclusion?
Consider any situation where the premisses are true. Then by B(1) there will
be some philosophy students who admire all logicians. Pick one, Jill for example.
We know from B(2) that Jill (since she is a philosophy student) doesn’t admire
anyone irrational. That is to say, people admired by Jill aren’t irrational. So
in particular, logicians – who are all admired by Jill – aren’t irrational. Which
establishes B(3), and shows that the inference step is deductively valid.
What about this next argument?
B0

(1) Some opera fans buy tickets for every new production of Tosca.
(2) No opera fan buys tickets for any merely frivolous entertainment.
So (3) No new production of Tosca is a merely frivolous entertainment.

This too is valid; and a moment’s reflection shows that it essentially involves the
same pattern of valid inference step as before.
We can again display the general principle in play using a schema, as follows:
Any inference step of the following type
Some F are R to every G
No F is R to any H
So: No G is H
is deductively valid.
Here we are using ‘is/are R to’ to stand in for a form of words expressing a
relation between things or people. So it might stand in for e.g. ‘is married to’,
‘is taller than’, ‘is to the left of’, ‘is a member of’, ‘admires’, ‘buys a ticket for’.
Abstracting from the details of the relations in play to leave just the schematic
pattern, we can see that the arguments B and B0 are deductively valid for the
same reason, by virtue of sharing the same indicated pattern of inference step.

21

3 Forms of inference
And just try describing the shared pattern of inference without using the
symbols: it can be done, to be sure, but at what a cost in ease of understanding!
(c) We have now used schemas to display three different types of valid inference
steps. We initially met the form of inference we can represent by the schema
All F are G
n is F
So: n is G.
And we have just noted the forms of inference
No F is G
n is F
So: n is not G
Some F are R to every G
No F is R to any H
So: No G is H.
Any argument instantiating one of these patterns will be deductively valid. The
way that ‘all’, ‘every’ and ‘any’, ‘some’, ‘no’ and ‘not’ distribute between the
premisses and conclusion means that inferences following these schematic forms
are necessarily truth-preserving – if the premisses are true, the conclusion has
to be true too.
For the moment, let’s just note three more examples of deductively reliable
forms of inference, involving different numbers of premisses:
No F is G
So: No G is F
All F are H
No G is H
So: No F is G
All F are either G or H
All G are K
All H are K
So: All F are K.
(Convince yourself that arguments which instantiate these schemas are indeed
valid in virtue of the meanings of the logical words ‘all’, ‘no’, and ‘or’.)

3.2 Four basic points about the use of schemas
We do not only use schemas to display patterns of reasoning in the logic classroom: it is also quite common to use schemas in a rough-and-ready way to clarify
the structure of arguments in philosophical writing.
Later, from Chapter 8 on, we will also use schemas in a significantly more
disciplined way when talking about patterns of reasoning in arguments framed
in artificial formal languages – languages that logicians love, for reasons which

22

Four basic points about the use of schemas
will soon become clear. We pause next, then, to emphasize four initial points
that apply to both the informal and the more formal uses of schemas to display
types of inference step.
(a)

Take again the now familiar form of inference we can display like this:
All F are G
n is F
So: n is G.

Does the following argument count as an instance of this schematic pattern?
C

(1) All men are mortal.
(2) Tweety is a robin.
So (3) Donald Trump is female.

Well, of course not! But let’s spell out why.
True enough, C(1) attributes a certain property to everything of a given kind;
so taken by itself, we can informally represent it as having the shape All F are
G. Likewise, C(2) and (C3) can, taken separately, be represented as having the
form n is F or n is G. However, when we describe an inference as an instance of
the three-line schema displayed above we are indicating that the same propertyascribing term F is involved in the two premisses. Likewise, we are indicating
that the name n and general term G that occur in the premisses recur in the
conclusion. This is worth highlighting:
The whole point of using recurring symbols in schemas representing forms
of inference is to indicate patterns of recurrence in the premisses and conclusion. Therefore, when we fill in a schema by substituting appropriate
expressions for the symbols, we must follow the rule: same symbols, same
substitutions.
So, in the present case, in moving back from our displayed abstract schema to
a particular instance of it, we must preserve the patterns of recurrence by being
consistent in how we substitute for the ‘F ’s and ‘G’s and ‘n’s.
(b) What about the following argument? Does this also count as an instance
of the last schema we displayed?
D

(1) All men are men.
(2) Socrates is a man.
So (3) Socrates is a man.

Instead of filling in our schema at random, we have at least this time been
consistent, substituting uniformly for both occurrences of ‘F ’, and likewise for
both occurrences of ‘G’. However, we happen to have substituted in the same
way each time.
We will allow this. So, to amplify, the rule is: same schematic variable, same
substitutions – but different schematic variables need not receive different substitutions. In the present case, if the ‘F ’s and ‘G’s are both substituted in the

23

3 Forms of inference
same way, we still get a valid argument; D cannot have true premisses and a
false conclusion!
Of course, D is no use at all as a means for persuading someone of the conclusion. An argument won’t persuade you of the truth of its conclusion if you
already have to accept the very same proposition in order to believe the argument’s premisses. Still, being valid is one thing, and being usefully persuasive is
something different. And keeping this distinction in mind, we can happily count
D as a limiting case of a deductively watertight argument. After all, an inference
step that covers no ground has no chance to go wrong.
(c)
E

And how about the following argument?
(1) Socrates is a man.
(2) All men are mortal.
So (3) Socrates is mortal.

Compared with the displayed abstract schema, the premisses here are stated ‘in
the wrong order’, with the general premiss second. But so what? An inference
step is valid, we said, just when there is no possible situation in which its premisses are true and its conclusion false. Given that definition, the validity of an
inference doesn’t depend at all on the order in which the premisses happen to
be stated. When we want to display a form of inference, the order in which the
various premisses are represented in a schema is irrelevant. Hence we can take
E as exemplifying just the same pattern of inference as before.
(d) Last but not least, what is the relation between the symbolic schema presented at the beginning of this section and its three variants below?
All F are G
All H are K
All Φ are Ψ
All ¬ are n is F
m is H
α is Φ
? is ¬
So: n is G
So: m is K
So: α is Ψ
So: ? is Plainly, these four different schemas are alternative possible ways of representing
the same form or pattern of inference (so don’t confuse schemas with forms of
inference!) Remember: what we are trying to reveal is a pattern of recurrence.
The ‘m’s, ‘n’s, ‘α’s, and ‘?’s are just different arbitrary ways of indicating how
names recur in the common pattern. Likewise the ‘F ’s or ‘Φ’s or whatever are
just different arbitrary ways of indicating where repeated property-ascribing
expressions are to be substituted. We can use whatever symbols take our fancy
to do the job. Later we will mainly be using Greek letters as schematic variables;
but for our current informal purposes we will stick to ordinary italic letters.∗

3.3 Arguments can instantiate many patterns
(a) It can be tempting to talk about ‘the’ form or pattern of inference step
exemplified by an argument. But we do need to be very careful here.
∗ If the letters of the Greek alphabet (starting alpha, beta, . . . !) are not very familiar, now

might be the time to start preparing the ground by looking at p. 412.

24

Arguments can instantiate many patterns
Consider again the hackneyed example of argument E with its two premisses
All men are mortal and Socrates is a man and the conclusion Socrates is mortal.
This is, trivially, an instance of the entirely unreliable form of inference
(1) A, B, so C
where ‘A’ etc. stand in for whole propositions (contrast here, for example, the
reliable form A, B, so A).
Exposing some structure in the individual propositions, but still ignoring
the all-important recurrences, E also instantiates the equally unreliable form
of inference
(2) All F are G, m is H, so n is K
since it has the right sort of general premiss, a second premiss ascribing some
property to a named individual, and a similar conclusion.
Next, and now filling in enough details about the structure of the inferential
step to bring out an inferentially reliable pattern of recurrence, our ‘Socrates’
argument also instantiates, as we said before, the form
(3) All F are G, n is F, so n is G.
But we can go further. E is also an instance of perfectly reliable types of inference
like
(4) All F are G, Socrates is F, so Socrates is G,
(5) All F are mortal, n is F, so n is mortal.
(6) All men are G, Socrates is a man, so Socrates is G.
And we could even, going to the extreme, take the inference to be the one and
only example of the reliable type
(7) All men are mortal, Socrates is a man, so Socrates is mortal,
which is (so to speak) a pattern with all the details filled in!
In sum, the ‘Socrates’ argument E exemplifies a number of different forms or
patterns of inference, specified at different levels of generality. Which all goes to
show that:
There is no such thing as the unique form or pattern of inference that a
given argument can be seen as instantiating.
Of course, this isn’t to deny that the form of inference (3) has a special place
in the story about E. For (3) is the most general but still reliable pattern of
inference that the argument exemplifies. In other words, the schema All F are
G; n is F; so, n is G gives just enough of the structure of E – but no more
than we need – to enable us to see that the argument is in fact valid. So it is
the reliability of this pattern of inference which someone defending argument E
will typically want to appeal to. It is might be rather tempting, then, to fall into
talking of the schema (3) as revealing ‘the’ pattern of inference in E; but, as
we’ve just seen, that would be rather loose talk.

25

3 Forms of inference
(b)

A terminological note:

It is common to refer to a universally reliable form or pattern of inference
– i.e. one whose instances are all valid – as itself being ‘valid’.
But if we fall in with this usage, we must emphasize again that, as just illustrated, an ‘invalid’ pattern of inference like (1) or (2) – meaning a pattern whose
instances are not all valid – can still have some particular instances that do happen to be valid (being valid, of course, for some other reason than exemplifying
the unreliable pattern).

3.4 Summary
Forms or patterns of inference can be conveniently represented by schemas
using letters or other symbols as variables (the specific choice of symbols
doesn’t matter). The use of these symbols is governed by the natural convention that a symbol stands in for the same expression wherever it appears
within a given argument schema.
There is strictly no such thing as the unique pattern or form exemplified by
an argument.
But when we talk of ‘the’ form of a valid argument, we typically mean the
most general reliable form of argument that it instantiates. Note, a valid
argument can also be an instance of some other, ‘too general’, unreliable
forms.
We can also say that a form of argument is valid, meaning that it is such
that all its instances are valid.

Exercises 3
(a) Which of the following patterns of inference are deductively reliable, meaning
that all their instances are valid? (Here ‘F ’, ‘G’, and ‘H’ hold the places for general
terms.) If you suspect an inference pattern is unreliable, find an instance which has to
be invalid because it has true premisses and a false conclusion.
(1)
(2)
(3)
(4)
(5)
(6)
(b)

Some F are G; no G is H; so, some F are not H.
Some F are G; some H are F ; so, some G are H.
All F are G; some F are H; so, some H are G.
No F is G; some G are H; so, some H are not F.
No F is G; no H is G; so, some F are not H.
All F are G; no G is H; so, no H is F .
What of the following patterns of argument? Are these deductively reliable?

(1) All F are G; so, nothing that is not G is F .
(2) All F are G; no G are H; some J are H; so, some J are not F .

26

Exercises 3
(3) There is an odd number of F , there is an odd number of G; so there is an
even number of things which are either F or G.
(4) All F are G; so, at least one thing is F and G.
(5) m is F ; n is F ; so, there are at least two F .
(6) Any F is G; no G are H; so, any J is J.
(c) Arguments of the kinds illustrated in (a) are so-called (categorical) syllogisms,
first systematically discussed by Aristotle in his Prior Analytics.
These syllogisms are formed from three propositions, each being of one of the following four forms, which have traditional labels:
A:
E:
I:
O:

All X are Y
No X is Y
Some X are Y
Some X are not Y.

By the way, these medieval labels supposedly come from the vowels of the Latin affirmo
(I affirm, for the positive two) and nego (I deny, for the negative two).
A syllogism then consists of two premisses and a conclusion, each having one of these
forms. The two terms in the conclusion occur in separate premisses, and then there
is a third or ‘middle’ term completing the pattern – as in our six schematic examples
above. Two questions arising:
(1) Which valid types of syllogism of this kind have a conclusion of the form A,
‘All S are P ’ ? (Use ‘M for the ‘middle’ term in a syllogism.)
(2) Which have a conclusion of the form O, ‘Some S are not P ’ ?
(d) Ancient Stoic logicians concentrated on a different family of arguments. Using
‘A’ and ‘B’ to stand in for whole propositions, and ‘not-A’ to stand in for the denial
of what ‘A’ stands in for, they endorsed the following five basic forms of arguments.
Indeed they held them to be so basic as to be ‘indemonstrable’:
(1)
(2)
(3)
(4)
(5)

If A then B; A; so B.
If A then B; not-B; so not-A.
not-(A and B); A; so not-B.
A or B; A; so not-B.
A or B; not-A; so B.

Which of these principles are acceptable, which – if any – are questionable? Give
illustrations to support your verdicts!
What about these further forms of argument? Which are correct?
(6)
(7)
(8)
(9)
(10)

If A then B; not-A; so not-B.
If A then B; B; so A.
not-(A and B); so either not-A or not-B.
A or B; so not-(not-A and not-B).
not-not-A; so A.

What about these general principles?
(11) If the inference A so B is valid, and the inference B so C is valid, then the
inference A so C is also valid.
(12) If the inference A, B so C is valid, then so is the inference A, not-C so not-B.

27

4 Proofs
In Chapter 1, we explained what it is for an inference step to be deductively
valid, and then we noted that different arguments can share a common form or
pattern of inference. Chapters 2 and 3 explored these ideas a little further.
We now move on to consider the following question: how can we establish that
a not-obviously-valid inference is in fact valid? One answer, in headline terms,
is: by giving a multi-step argument that takes us from that inference’s premisses
to its conclusion by simple, plainly valid steps – in other words, by giving a
derivation or proof. This chapter explains.

4.1 Proofs: first examples
(a)
A

Take a charmingly silly example from Lewis Carroll:
Babies are illogical; nobody is despised who can manage a crocodile;
illogical persons are despised; so babies cannot manage a crocodile.

This three-premiss, one-step, argument is in fact valid. But how can we demonstrate the argument’s validity if it isn’t immediately clear? Well, consider the
following two-step derivation:
A0

(1) Babies are illogical.
(2) Nobody is despised who can manage a crocodile.
(3) Illogical persons are despised.
(4) Babies are despised.
(5) Babies cannot manage a crocodile.

(premiss)
(premiss)
(premiss)
(from 1, 3)
(from 2, 4)

Here, we have inserted an extra step between the three premisses and the target conclusion (and we can omit writing ‘So’ before lines (4) and (5), as the
commentary on the right is already enough to indicate that these lines are the
results of inferences).
The inference from two of the premisses to the interim conclusion (4), and then
the second inference from the other original premiss and that interim conclusion
(4) to the final conclusion (5), are both evidently valid. We can therefore get
from the initial premisses to the final conclusion by a necessarily truth-preserving
route. Which shows that the inferential step in the original argument A is valid.
(b)

28

Here’s a second quick example, equally daft:

Proofs: first examples
B

Everyone loves a lover; Romeo loves Juliet; so everyone loves Juliet.

Take the first premiss to mean ‘everyone loves anyone who is a lover’ (where a
lover is, by definition, a person who loves someone). Then this argument too is
deductively valid! Here is a multi-step derivation:
B0

(1) Everyone loves a lover.
(2) Romeo loves Juliet.
(3) Romeo is a lover.
(4) Everyone loves Romeo.
(5) Juliet loves Romeo.
(6) Juliet is a lover.
(7) Everyone loves Juliet.

(premiss)
(premiss)
(from 2)
(from 1, 3)
(from 4)
(from 5)
(from 1, 6)

We have again indicated on the right the ‘provenance’ of each new statement as
the argument unfolds. And by inspection we can see that each small inference
step is valid, i.e. is necessarily truth-preserving. As the argument grows, then,
we are adding new propositions which must also be true, assuming the original
premisses are true. These new true propositions can then serve in turn as inputs
to further valid inferences, yielding more truths. Everything is chained together
so that, if the original premisses are true, each added proposition must be true
too, and therefore the final conclusion in particular must be true. Hence the original inference step in B that jumps in one leap straight across the intermediate
stages must be valid.
(c) These examples illustrate how to establish the validity of an unobviously
valid inference step, using a technique already familiar to Aristotle:
One way of demonstrating that an inferential leap from some premisses to
a given conclusion is valid is by breaking down the big leap into smaller
inference steps, each one of which is clearly valid.
There is a familiar logician’s term for a multi-step argument put together in such
a way as to form a deductively cogent derivation – it’s a proof.
(d) One-step arguments can be merely valid (deductively cogent) or they can be
sound (deductively cogent and proceeding from true premisses). Valid one-step
arguments can have false conclusions; sound arguments, however, do establish
the truth of their conclusions. There’s a similar distinction to be made about
multi-step arguments. They too can be merely valid (deductively cogent) or they
can be sound (deductively cogent and proceeding from true premisses). In the
logician’s sense, then, there can be merely valid proofs and there can be sound
proofs. It is only sound proofs that establish their conclusions as true.
So we have another potentially awkward departure from ordinary language
here, for we ordinarily think of a proof as, well, proving something, establishing
it as true! In other words, by ‘proof’ we ordinarily mean a sound proof. There
would therefore be something to be said for using one term for a cogent multistep argument, derivation (say), and reserving the term proof for an argument

29

4 Proofs
which does indeed establish its conclusion outright. (Gottlob Frege, the founding
father of modern logic, in effect adopts such a usage.) But like it or not, this
isn’t standard practice. And context should make it clear when we are talking
about a ‘proof’ in the wider logician’s sense of an internally cogent argument
(usually multi-step), and when we are using ‘proof’ more narrowly to mean a
sound argument, an outright demonstration of truth.

4.2 Fully annotated proofs
(a) In our first two examples of proofs, we have indicated at each new step
which earlier statements the inference depended on. But we can do even better
by also explicitly indicating what type or form of inference move is being invoked
at each stage.
For use in our next example, then, let’s repeat two principles that we have
met before and now briskly label them:
Any inference step of either of the following two types is valid:
(R)

No F is G
n is F
So: n is not G

(S)

No F is G
So: No G is F

So consider the following little argument:
C

No logician is bad at abstract thought. Jo is bad at abstract thought.
So Jo is not a logician.

This hardly needs a proof! – but still, we can prove it, invoking (R) and (S), as
follows:
C0

(1) No logician is bad at abstract thought.
(premiss)
(2) Jo is bad at abstract thought.
(premiss)
(3) No one bad at abstract thought is a logician. (from 1 by S)
(4) Jo is not a logician.
(from 3, 2 by R)

(Why do you think we have mentioned steps (3) and (2) in that order in the
annotation for line (4)?)
Note that in treating the move from (1) to (3) as an instance of (S) we are –
strictly speaking – cutting ourselves a bit of slack, quietly massaging the English
grammar. It’s actually rather difficult to give strict rules for doing this sort of
thing. Which is one good reason for eventually moving from considering arguments framed in messy English to considering their counterparts in more tidily
rule-governed formalized languages.
(b) Of course, the validity of C has nothing especially to do with Jo, logicians,
and abstract thought. Abstracting from its premisses and final conclusion, we
can say, more generally:

30

Fully annotated proofs
Any inference step of the following type is valid:
(T)

No F is G
n is G
So: n is not F

Further, any instance of this pattern can be shown to be valid by using a proof
parallel to C0 . In other words, the general reliability of (T) follows from the
reliability of (R) and (S).
So this little example nicely reveals another important sense in which logical
inquiry can be systematic (compare §1.5). Not only can we treat arguments
wholesale by considering together whole families relying on the same principles
of inference, but we can systematically interrelate patterns of inference like (R),
(S), and (T) by showing how the reliability of some ensures the reliability of
others.
Logical theory can therefore give us much more than an unstructured catalogue of deductively reliable principles of inference; we can and will explore how
principles of inference hang together. More about this later in the book!
(c) Let’s take another argument from Lewis Carroll, who is an inexhaustible
source of silly examples (there is no point in getting too solemn at this stage!):
D

Anyone who understands human nature is clever; every true poet
can stir the heart; Shakespeare wrote Hamlet; no one who does not
understand human nature can stir the heart; no one who is not a
true poet wrote Hamlet; so Shakespeare is clever.

The big inferential leap from those five premisses to the conclusion is in fact
valid. For note the following:
Any inference step of either of the following two types is valid:
(U)

Any/every F is G
n is F
So: n is G

(V)

No one who isn’t F is G
So: Any G is F

Then we can argue as follows, using these two forms of valid inference step:
D0

(1) Anyone who understands human nature is clever.
(premiss)
(2) Every true poet can stir the heart.
(premiss)
(3) Shakespeare wrote Hamlet.
(premiss)
(4) No one who does not understand human nature
can stir the heart.
(premiss)
(5) No one who is not a true poet wrote Hamlet. (premiss)
(6) Anyone who wrote Hamlet is a true poet.
(from 5, by V)
(7) Shakespeare is a true poet.
(from 6, 3, by U)

31

4 Proofs
(8) Shakespeare can stir the heart.
(from 2, 7, by U)
(9) Anyone who can stir the heart understands human
nature.
(from 4, by V)
(10) Shakespeare understands human nature.
(from 9, 8, by U)
(11) Shakespeare is clever.
(from 1, 10, by U)
That’s all very very laborious. But now we have the provenance of every move
fully annotated. Each of the inference steps is an instance of a clearly truthpreserving pattern, and together they get us from the initial premisses of the
argument to its final conclusion. Hence, if the initial premisses are true, the
conclusion must be true too. So our proof here really does show that the original
argument D is deductively valid.

4.3 Glimpsing an ideal
(a) Using toy examples, we have now glimpsed an ideally explicit way of setting
out one kind of regimented multi-step proof. Every premiss gets stated at the
outset and is explicitly marked as such; and then we write further propositions
one under another, each coming with a certificate which tells us what it is inferred
from and what principle of inference is being invoked.
Needless to say, everyday arguments (and not-so-everyday proofs in mathematics) normally fall a long way short of meeting these ideal standards of
explicitness! They rarely come ready-chunked into numbered statements, with
each new inference step bearing a supposed certificate of excellence. However,
faced with a puzzling multi-step argument, massaging it into something nearer
this fully documented shape will help us to assess the argument. On the one
hand, an explicitly annotated proof gives a critic a particularly clear target to
fire at. If someone wants to reject the conclusion, then they will have to rebut
one of the premisses, show an inference step doesn’t have the claimed form, or
come up with a counterexample to the reliability of one of the forms of inference
that is explicitly called on. On the other hand, looking on the bright side, if the
premisses are agreed and if the inference moves are uncontentiously valid, then
the proof will establish the conclusion beyond further dispute.
(b) The exercise of regimenting an argument into a more ideally explicit form
will reveal redundancies (premisses or inferential detours that are not needed
to get to the final conclusion) and will also expose where there are suppressed
premisses (i.e., premisses that are not stated but which are needed to get a
well-constructed proof).
Traditionally, arguments with suppressed premisses are called enthymemes.
Without worrying about that label, here’s an example:
E

The constants of nature have to take values in an extremely narrow
range (have to be ‘fine-tuned’) to permit the evolution of intelligent
life. So the universe was intelligently designed.

As it stands this is plainly gappy: there are unspoken assumptions in the back-

32

Deductively cogent multi-step arguments
ground. But it is instructive to try to turn this reasoning into a deductively
valid argument, by making explicit the currently suppressed premisses. One
such premiss will be uncontroversial, just noting that intelligent life has evolved.
The other premiss will be much more problematic: roughly, intelligent design is
needed in order for the universe to be fine-tuned enough. Only with some such
additions can we get a deductively cogent argument.

4.4 Deductively cogent multi-step arguments
When is a multi-step argument internally cogent? The headline answer is predictable: when each inference step is cogent and also the steps are chained
together in the right kind of way.
(a)
F

Consider first the following mini-argument:
(1) All philosophers are logicians.
(2) All logicians are philosophers.

(premiss)
(from 1?!)

The inference here is horribly fallacious. It just doesn’t follow from the premiss
that all philosophers are logicians that only philosophers are logicians. (You
might as well argue ‘All women are human beings, hence all human beings are
women’.)
Here’s another really bad argument:
G

(1) All existentialists are philosophers.
(2) All logicians are philosophers.
(3) All existentialists are logicians.

(premiss)
(premiss)
(from 1, 2?!)

It plainly doesn’t follow from the claims that the existentialists and logicians
are both among the philosophers that any of the existentialists are logicians,
let alone that all of them are. (You might as well argue ‘All women are human
beings, all men are human beings, hence all women are men’.)
But now imagine that someone chains this pair of rotten inferences together
into a two-step argument as follows:
H

(1) All existentialists are philosophers.
(2) All philosophers are logicians.
(3) All logicians are philosophers.
(4) All existentialists are logicians.

(premiss)
(premiss)
(from 2?!)
(from 1, 3?!)

Our reasoner first makes the same fallacious inference as in F; and then they
compound the sin by committing the same howler as in G. So they have gone
from the initial premisses H(1) and H(2) to their final conclusion H(4) by two
quite terrible moves.
Yet note that in this case – despite the howlers along the way – the final
conclusion H(4) in fact really does follow from the initial premisses H(1) and
H(2). If the existentialists are all philosophers, and all philosophers are logicians,
then the existentialists must of course be logicians!

33

4 Proofs
What is the moral of this example? If we were to say that a multi-step argument is deductively cogent just if the big leap from the initial premisses to the
final conclusion is valid, then we’d have to count the two-step argument H as a
cogent proof. Which would be a very unhappy way of describing the situation,
given that the two-step derivation involves a couple of nasty fallacies!
For a correctly formed proof – a deductively cogent multi-step argument,
where everything is in good order – we should therefore require not just that the
big leap from the initial premisses to the final conclusion is valid, but that the
individual inference steps along the way are all valid too.
Exactly as you would expect!
(b) But that is not the whole story. For deductive cogency, we also need a
proof’s valid inference steps to be chained together in the right kind of way.
To illustrate, consider the inference step here:
I

(1) Socrates is a philosopher.
(premiss)
(2) All philosophers have snub noses.
(premiss)
(3) Socrates is a philosopher and all philosophers have
snub noses.
(from 1, 2)

That’s trivially valid (since from A together with B you can infer A-and-B ).
Next, here is another equally trivial valid inference step (since from A-and-B
you can of course infer B ):
J

(1) Socrates is a philosopher and all philosophers have
snub noses.
(premiss)
(2) All philosophers have snub noses.
(from 1)

And thirdly, here is another plainly valid inference step, one of a very familiar
form which we’ve met before:
K

(1) Socrates is a philosopher.
(2) All philosophers have snub noses.
(3) Socrates has a snub nose

(premiss)
(premiss)
(from 1, 2)

(Note that I and K have the same premisses and different conclusions – but
that’s fine. We can extract more than one implication from the same premisses!)
Taken separately, then, those three little inference steps are entirely unproblematic. However, imagine now that someone chains these steps together to get
the following unholy tangle:
L

(1) Socrates is a philosopher.
(premiss)
(2) Socrates is a philosopher and all philosophers
have snub noses.
(from 1, 3, as in I)
(3) All philosophers have snub noses.
(from 2 as in J)
(4) Socrates has a snub nose.
(from 1, 3 as in K)

By separately valid steps we seem to have deduced the shape of Socrates’ nose
just from the premiss that he is a philosopher! What has gone wrong?

34

Indirect arguments
The answer is plain. In the middle of the argument we have gone round in a
circle. L(2) is derived from inputs including L(3), and then L(3) is derived from
L(2). Circular arguments like this can’t take us anywhere. Which shows that
there is another condition for being a deductively cogent multi-step argument.
As well as the individual inference steps all being valid, the steps must be chained
together in a non-circular way. In particular, each step – unless a newly made
assumption – must depend only on what’s gone before. As you would expect.

4.5 Indirect arguments
(a) So far, so straightforward. But now we need to complicate the story, introducing an important new idea, namely that we often use indirect arguments.
For example, we often try to establish a conclusion not-S by first supposing the
opposite, i.e. S, and then showing that this leads to something absurd.
Return to the silly crocodile argument A0 in §4.1. Imagine that someone has
a momentary lapse and pauses over the final step (where we inferred Babies
cannot manage a crocodile from the original premiss Nobody is despised who can
manage a crocodile together with the interim conclusion Babies are despised ).
How could we convince them that this step really is valid?
We might try amplifying the argument so we get an expanded (not quite fully
documented) proof which now runs as follows:
A00

(1) Babies are illogical.
(premiss)
(2) Nobody is despised who can manage a crocodile.
(premiss)
(3) Illogical persons are despised.
(premiss)
(4) Babies are despised.
(from 1, 3)
Suppose temporarily, for the sake of argument,
(5)
Babies can manage a crocodile.
(supposition)
(6)
Babies are not despised.
(from 2, 5)
(7)
Contradiction!
(from 4, 6)
Our supposition leads to absurdity, hence
(8) Babies cannot manage a crocodile.
(from 5–7 by RAA)

What is going on in this expanded proof? We want to establish (8). But this
time, instead of aiming directly for the conclusion, we branch off by temporarily
supposing the exact opposite is true, i.e. we suppose (5). However, this supposition immediately leads to something that flatly contradicts an earlier claim.
Hence the temporary supposition (5) has to be rejected. Therefore its denial (8)
must be true after all.
The terse justification for the final step ‘(from 5–7 by RAA)’ indicates that
we have made a reductio ad absurdum inference, to use the traditional name –
the supposition made for the sake of argument at (5) leads to absurdity at (7),
so has to be rejected.
Note by the way that, in the indented part of the proof, we are working with
our initial premisses plus the new temporary supposition all in play, assumptions

35

4 Proofs
which taken together are in fact inconsistent. By the end of the (short!) indented
bit of the proof, we have exposed this inconsistency by showing that those assumptions together entail a contradiction. As we said back in §2.4, it is very
important that we can in this way argue validly from inconsistent assumptions
to expose their inconsistency.
(b)
M

Here is another quick example. Take the argument:
No girl likes any unreconstructed sexist; Caroline is a girl who likes
anyone who likes her; Henry likes Caroline; hence Henry is not an
unreconstructed sexist.

This is valid. Here is one way to see that it is, by using another reductio argument:
M0

(1) No girl likes any unreconstructed sexist.
(premiss)
(2) Caroline is a girl who likes anyone who likes her.
(premiss)
(3) Henry likes Caroline.
(premiss)
(4) Caroline is a girl who likes Henry.
(from 2, 3)
(5) Caroline is a girl.
(from 4)
(6) Caroline likes Henry.
(from 4)
Suppose temporarily, for the sake of argument,
(7)
Henry is an unreconstructed sexist.
(supposition)
(8)
No girl likes Henry.
(from 1, 7)
(9)
Caroline does not like Henry.
(from 5, 8)
(10)
Contradiction!
(from 6, 9)
Our supposition leads to absurdity, hence
(11) Henry is not an unreconstructed sexist.
(from 7–10 by RAA)

(c) Let’s now spell out the (RAA) principle invoked in these two proofs: the
idea is surely a familiar one, even if we are now spelling it out more explicitly
than you are used to seeing. We can put it like this:
Reductio ad absurdum If A1 , A2 , . . . , An (as background premisses) plus
the temporary supposition S entail a contradiction, then A1 , A2 , . . . , An by
themselves entail not-S.
Here, the ‘A’s and ‘S’ stand in for whole propositions; and entailing a contradiction is a matter of entailing some proposition C while also entailing its denial
not-C (or equivalently, entailing the single proposition C and not-C).
It should be clear that the (RAA) principle is a good one. But it is worth
spelling out carefully why it works:
Assume that the As plus S do entail a contradiction C and not-C. Now,
whatever is entailed by truths must also be true (since entailment is truthpreserving). So suppose that the As plus S were all true together. Then C
and not-C would have to be true too. But that’s ruled out, as a contradiction
can never be true. So the As plus S cannot all be true together.

36

Summary
Of course, just knowing that at least one of As plus S must be false
doesn’t tell us where to pin the blame! However, we can conclude this: in
any situation in which the As are all true, the remaining proposition S must
be the false one. Hence those propositions A1 , A2 , . . . , An entail not-S.
(Question: What happens when n = 0 and there are no background premisses?)
(d) Reductio ad absurdum arguments are naturally called indirect arguments.
We don’t go straight from premisses to conclusion, but take a side step via some
additional supposition which we temporarily add for the sake of argument and
then eventually ‘discharge’. (We have visually signalled the side step by indenting
the column of argument to the right when we bring a new supposition into play,
and going back left when the supposition is dropped again.)
As we will see later, there are other familiar kinds of indirect arguments. And
to cover such indirect forms of reasoning, we will have to extend what we said
before about cogent multi-step arguments. In particular, we must now permit
steps which introduce new temporary suppositions ‘for the sake of argument’
(so long as they are clearly flagged). Then, as well as ordinary valid inference
steps which depend on what we have previously established and/or some new
assumption(s) currently in play, we will also allow inference steps like (RAA),
which allow us to draw consequences when we have constructed appropriate
‘subproofs’ starting with temporary assumptions.
Our discussion in this section necessarily only gives a hint of what is to come:
we will return to the topic of indirect arguments when we eventually start discussing formal ‘natural deduction’ proofs in Chapter 20. But it is worth noting
one point straightaway. In an indirect argument, we make temporary assumptions or suppositions; and to suppose some proposition is true for the sake of
argument is plainly not to assert its truth outright. Therefore propositions, the
ingredients of arguments, do need to be distinguished from outright assertions.
Propositions can be asserted, supposed, rejected, wondered about, and more.

4.6 Summary
To establish the validity of a perhaps unobvious inferential leap we can
use deductively cogent multi-step arguments, i.e. proofs, filling in the gap
between premisses and conclusion.
Simple, direct, proofs chain together inference steps that are valid – ideally,
ones which are obviously valid – building up from the initial premisses to the
desired conclusion, with each new step depending on what’s gone before.
However, some common methods of proof like reductio ad absurdum are
‘indirect’, and involve making new temporary suppositions for the sake of
argument, suppositions that are later discharged. (The availability of indirect modes of inference complicates the story about what makes for a
deductively cogent multi-step argument; we return to this.)

37

4 Proofs

Exercises 4
Which of the following arguments are valid? Where an argument is valid, sketch an
informal proof. Some of the examples are enthymemes that need repair.
(1) Only logicians are good philosophers. No existentialists are logicians. Some
existentialists are French philosophers. So, some French philosophers are not
good philosophers.
(2) No philosopher is illogical. Jones keeps making argumentative mistakes. No
logical person keeps making argumentative mistakes. All existentialists are
philosophers. So, Jones is not an existentialist.
(3) No experienced person is incompetent. Jenkins is always blundering. No competent person is always blundering. So, Jenkins is inexperienced.
(4) Jane has a first cousin. Jane’s father is an only child. So, if Jane’s mother
hasn’t a sister, she has a brother.
(5) Every event is causally determined. No action should be punished if the agent
isn’t responsible for it. Agents are only responsible for actions they can avoid
doing. Hence no action should be punished.
(6) Some chaotic attractors are not fractals. All Cantor sets are fractals. Hence
some chaotic attractors are not Cantor sets.
(7) Something is an elementary particle only if it has no parts. Nothing which has
no parts can disintegrate. An object that cannot be destroyed must continue
to exist. So an elementary particle cannot cease to exist.
(8) Either the butler or the cook committed the murder. The victim died from
poison if the cook was the murderer. The butler carried out the murder only
if the victim was stabbed. The victim didn’t die from poison. So, the victim
was stabbed.
(9) Superman is none other than Clark Kent. The Superhero from Krypton is
Superman. The Superhero from Krypton can fly. Hence Clark Kent can fly.
(10) Jack is useless at logic or he simply isn’t ready for the exam. Either Jack will
fail the exam or he is not useless at logic. Either it’s wrong that he won’t fail
the exam or he is ready for it. So Jack will fail.
(11) Any elephant weighs more than any horse. Some horses weigh more than any
donkey. Hence any elephant weighs more than any donkey.
(12) When I do an example without grumbling, it is one that I can understand.
No easy logic example ever makes my head ache. This logic example is not
arranged in regular order, like the examples I am used to. I can’t understand
these examples that are not arranged in regular order, like the examples I am
used to. I never grumble at an example, unless it gives me a headache. So,
this logic example is difficult.
Finally, an example that requires a minimal amount of arithmetical knowledge:
√
2 cannot
(13)
√ be a rational number, i.e. a fraction. We can show this as follows.
Suppose 2 = m/n, where this fraction is in lowest terms. Then (i) m2 = 2n2 ,
so m is even, and hence m = 2k. (ii) Then n2 = 2k2 , so n is even, and hence
m isn’t (or else m/n wouldn’t be in lowest term). Hence (iii) our supposition
leads to contradiction.
Set this out as a line-by-line reductio proof more in the style of §4.5.

38

5 The counterexample method
Suppose we suspect that a particular one-step argument is invalid, but are unsure. If we can construct a multi-step argument from the given premisses to the
claimed conclusion, relying on uncontentiously valid moves, then that settles it –
the argument is valid after all. But what if we have tried to find such a proof and
failed? What does that show? Maybe the argument is invalid. But maybe we
have just not spotted a proof, and the argument is in fact valid. If failure to find
a proof doesn’t settle the matter one way or the other, how can we demonstrate
that a dubious inference step really is invalid?
In this chapter we put together the invalidity principle (which we met in
Chapter 2) and the observation that different arguments can depend on the
same pattern of inference (as outlined in Chapter 3) to give us one method for
showing that various invalid arguments are indeed deductively invalid.

5.1 ‘But you might as well argue . . . ’
(a) In fact, we have already quietly used this method in passing in §4.4. We
noted that the inference step in the mini-argument
A

(1) All philosophers are logicians.
So (2) All logicians are philosophers.

is invalid. That should have been immediately clear. But, to press home the
point, we remarked that you might as well argue
A0

(1) All women are human beings.
So (2) All human beings are women.

What is the force of this brisk comparison? We can unpack it as follows:
Argument A0 has a true premiss and a false conclusion; hence by the invalidity principle of §2.3 it can’t be valid. But the inferential move in argument
A is no better. There is nothing to separate the cases. The arguments evidently share the same pattern of inference and stand or fall together as far
as validity is concerned. Hence, A is invalid too.
And that’s a simple illustration of the basic method we need. Roughly: to show
an inference step is invalid, find an argument which relies on the same form of
inference but which is clearly invalid by the invalidity principle.

39

5 The counterexample method
(b) We used the same method on a second example in §4.4. But let’s next have
a different illustration, this time revisiting an argument we met in §1.4:
B

(1) Most Irish are Catholics.
(2) Most Catholics oppose abortion on demand.
So (3) At least some Irish oppose abortion on demand.

We persuaded ourselves that the inferential step here is invalid by imagining a
situation in which its premisses would be true and conclusion false. But equally,
we could have brought out its invalidity by noting that you might as well argue
B0

(1) Most chess grandmasters are men.
(2) Most men are no good at chess.
So (3) At least some chess grandmasters are no good at chess.

What is the force of the comparison this time?
B0 ’s premisses are true. Chess is still (at least at the top levels of play) a
predominantly male activity, though one that few men are any good at. But
B0 ’s conclusion is false. So B0 is invalid by the invalidity principle. But the
inferential move in argument B is no better. The arguments evidently stand
or fall together as far as validity is concerned. Hence, B is invalid too.
(c) There is nothing mysterious or difficult or even novel going on here. The
‘But you might as well argue . . . ’ technique for showing an inference step to be
invalid is already noted and used by Aristotle, and we use it all the time in the
everyday evaluation of arguments.
For example, some gossip says that Mrs Jones must be an alcoholic, because
she has been seen going to the Cheapo Booze Emporium and everyone knows
that is where the local alcoholics are to be found. You reply, ‘But you might as
well argue that Bernie Sanders is a Republican Senator, because he’s been seen
going into the Senate, and everyone knows that that’s where the Republican
Senators are to be found’. Which shows why the gossip’s argument won’t do.

5.2 The counterexample method, more carefully
(a) Recall, however, a key point from §3.3. A pattern or form of inference F
may be unreliable; yet it can still have some special instance which does happen
to be valid (valid for some other reason than instantiating F ). Hence we can’t
say, crudely, ‘This inference I is an instance of the form F ; but here’s another
instance J of the same form F which is plainly invalid. So I is invalid too.’
We therefore need to be a bit more careful in spelling out our ‘counterexample
method’ for showing invalidity. The idea is this:
Stage 1 Given a (one-step) argument whose validity is up for assessment,
first locate a form of inference F which this argument is relying on, in the
sense that F needs to be reliably truth-preserving if the inference in the
argument is indeed to be valid.

40

A ‘quantifier shift’ fallacy
Stage 2 Show that this form of inference F is not a generally reliable one
by finding a counterexample. In other words, find another argument having
the same pattern of inference which is uncontroversially invalid, e.g. because
it has actually true premisses and a false conclusion so we can apply the
invalidity principle.
Note, by the way, that we do not here require a counterexample to be generated
by the invalidity principle, i.e. to have actually true premisses and an actually
false conclusion. Telling counterexamples don’t have to be constructed from
‘real life’ situations. A merely imaginable but uncontroversially coherent counterexample will do just as well. Why? Because that’s still enough to show that the
inferential pattern in question is unreliable: it doesn’t necessarily preserve truth
in all possible situations. However, using ‘real life’ counterexamples does often
have the great advantage that you don’t get into any disputes about what is
coherently imaginable.
(b) In using this method for showing that a given argument is invalid, everything turns on picking out at Stage 1 a form of inference F which the challenged
argument really does require to be reliable.
Often that is very easy to do. Take our initial example A: ‘All philosophers
are logicians. So all logicians are philosophers.’ What form of inference move
can this argument possibly be relying on, other than All G are H; so all H
are G? And, as we saw, it is trivial to come up with a counterexample to the
general reliability of this form of inference. Similarly for our second example B:
‘Most Irish are Catholics. Most Catholics oppose abortion. So at least some Irish
oppose abortion.’ What form of inference can that argument be relying on, other
than Most G are H; most H are K; so at least some G are K ? Again, it is trivial
to come up with a counterexample to the validity of that form of inference too.
Other cases, however, can be more problematic. Someone proposes an argument. You challenge it by replying ‘But you might as well argue . . . ’ (advancing
a seemingly parallel but patently invalid argument). The proponent of the challenged argument then seeks to show that your supposed counterexample isn’t a
fair one – the original argument didn’t actually depend on the form of inference
F you supposed. With arguments served up in ordinary prose, this might not
be easy to settle. But, at the very least, challenge-by-apparent-counterexample
will force the proponent of an argument to clarify what is supposed to be going
on, to make it plain what principle of inference is being relied on.

5.3 A ‘quantifier shift’ fallacy
(a) Let’s have an example to illustrate the last point. Consider the following
quotation (in fact, the opening words of Aristotle’s Nicomachean Ethics):
Every art and every inquiry, and similarly every action and pursuit, is
thought to aim at some good; and for this reason the good has rightly
been declared to be that at which all things aim.

41

5 The counterexample method
At first sight, there is an argument here with this initial premiss:
C

(1) Every practice aims at some good.

And then a conclusion is drawn (note the inference marker ‘for this reason’). An
unkind reader might gloss the conclusion as:
So (2) There is some good (‘the good’) at which all practices aim.
This argument then has a very embarrassing similarity to the following one:
C0

(1) Every assassin’s bullet is aimed at some victim.
So (2) There is some victim at whom every assassin’s bullet is aimed.

But every assassin’s bullet has its target (let’s suppose), without there being a
single target shared by them all. Likewise, every practice may aim at some good
end or other without there being a single good which encompasses them all.
Drawing out some logical structure, argument C relies on the inference form
Every F is R to some G
So: There is some G such that every F is R to it.
and then C0 is a counterexample to the reliability of that form of inference.
(b) Did Aristotle really use an argument relying on that disastrous form of
inference? We have ripped the quotation from the Ethics out of all context, so
there is room for debate about what his intended argument really is. We can’t
enter into such debates here. Still, anyone who wants to defend Aristotle must
at least meet the challenge of saying why the supposed counterexample fails to
make the case, and must tell us what principle of inference really is being used
in his intended argument here. If Aristotle isn’t to be interpreted as using a
fallacious form of inference as in C, then what is he up to?
The same goes, to repeat, for other challenges by the counterexample method.
Given an argument in a text apparently relying on some identified pattern of
inference, finding a counterexample to the deductive reliability of that pattern
delivers a potentially fatal blow. If you want to defend the author, then your
only hope is to show that the text has been misunderstood and/or the relevant
inference step mis-identified.
(c) Expressions like ‘every’ and ‘some’ are standardly termed quantifiers by
linguists and logicians (see Chapter 26): what we have just noted is that we
cannot always shift around the order of quantifiers in a proposition.
It is perhaps worth briskly noting in passing that a number of naive arguments
for the existence of God commit the same apparently tempting quantifier shift
fallacy, using the same bad form of inference. Consider, for example,
D

(1) Every ecological system has an intelligent designer,
So (2) There is some intelligent designer (God) who designed every
ecological system.

Premiss (1) was exploded by Darwin; but forget that. The point we want to
make now is that even if you grant (1), that does not establish (2). There may,
consistently with the premiss, be no one Master Designer of all the ecosystems

42

Summary
– rather each system could be produced by a different designer. (No respectable
philosopher of religion uses the rotten argument D as it stands; but you need to
appreciate the fallacy in the naive argument here to see why serious versions of
design arguments for God have to try a great deal harder!)

5.4 Summary
To use the counterexample strategy to prove that a target argument is invalid, (1) find a pattern of inference that the argument is depending on,
and (2) then show that the pattern is not a reliable one by finding a counterexample to its reliability, i.e. find an argument exemplifying this pattern
which has (or evidently could have) true premisses and a false conclusion.
This method is familiar in the everyday evaluation of arguments as the ‘But
you might as well argue . . . ’ gambit.
We used the counterexample strategy to show, in particular, that a certain
(occasionally tempting?) quantifier shift fallacy is indeed a fallacy.

Exercises 5
An initial group of examples. Some of the following arguments are invalid. Which?
Why?
(1) Many great pianists admire Glenn Gould. Few, if any, unmusical people admire Glenn Gould. So few, if any, great pianists are unmusical.
(2) Everyone who admires Bach loves the Goldberg Variations; some who admire
Chopin do not love the Goldberg Variations; so some admirers of Chopin do
not admire Bach.
(3) Some hikers are birdwatchers. All birdwatchers carry binoculars. Some who
carry binoculars carry cameras too. So some hikers carry cameras.
(4) Anyone who is good at logic is good at assessing philosophical arguments.
Anyone who is mathematically competent is good at logic. Anyone who is
good at assessing philosophical arguments admires Bertrand Russell. Hence
no one who admires Bertrand Russell lacks mathematical competence.
(5) Everyone who is not a fool can do logic. No fools are fit to serve on a jury.
None of your cousins can do logic. Therefore none of your cousins is fit to
serve on a jury.
(6) Most logicians are philosophers; few philosophers are unwise; so at least some
logicians are wise.
(7) All logicians are rational; no existentialists are logicians; so if Sartre is an
existentialist, he isn’t rational.
(8) If Sartre is an existentialist, he isn’t a logician. If Sartre isn’t a logician, he isn’t
good at reasoning. So if Sartre is good at reasoning, he isn’t an existentialist.

43

6 Logical validity
We now introduce a narrower notion of validity, namely (so-called) logical validity. All logically valid arguments are deductively valid in the sense of §2.1, but
not vice versa.
Our discussion here will continue to be quite informal. But it will provide a
useful stepping stone along the way to defining some more formal, precise notions
of particular kinds of logical validity that will in fact be our topic in the rest of
this book.

6.1 Topic-neutrality
Start by considering the following one-premiss arguments: are they valid?
A

(1) Jill is a mother.
So (2) Jill is a parent.

B

(1) Jack has a first cousin.
So (2) At least one of Jack’s parents is not an only child.

C

(1) Jack is a bachelor.
So (2) Jack is unmarried.

In any possible situation in which Jill is a mother, she must have a child, i.e. be
a parent. In any situation in which Jack has a first cousin (where a first cousin
is a child of one of his aunts or uncles) he must have or have had an aunt or
uncle, so his parents cannot both have been only children. Necessarily, if Jack
is a bachelor, he is unmarried. So these arguments all involve deductively valid
inference steps, according to our characterization of validity in §2.1.
In each of these cases, we can again abstract from some of the details, and
see the argument as an instance of a schema all of whose other instances are
valid too. For plainly, our arguments are not made valid by something peculiar
to Jill or Jack; rather, what matters are the concepts of a mother, a first cousin,
a bachelor. Therefore any inference of the following kinds is valid:
n is a mother
So: n is parent
n has a first cousin
So: n’s parents are not both only children

44

Logical validity, at last
n is a bachelor
So: n is unmarried.
So these are further examples of reliable principles of inference. But these principles are evidently less abstract than those we have highlighted previously.
Here is a – rather loose – definition:
Vocabulary like ‘all’ and ‘some’, ‘and’ and ‘or’, ‘not’ and ‘if’ etc. (plus the
likes of ‘is’, ‘are’) – vocabulary which doesn’t refer to specific things or
properties or relations, etc., but which is useful in discussing any topic – is
topic-neutral.
The schemas we used in earlier chapters to display some inferential patterns involved only schematic variables and topic-neutral vocabulary. By contrast, the
new schemas we have just introduced involve concepts which belong to more
specific areas of interest – these first examples in fact all concern familial relations. The new patterns of inference are none the worse for that; their instances
are perfectly good valid inferences by our informal definition in §2.1. But their
validity does not rely merely on the distribution of topic-neutral vocabulary in
the premisses and conclusions.
Let’s add a few similar examples:
D

(1) Cambridge is north of Oxford.
So (2) Oxford is south of Cambridge.

E

(1) Siena is in Tuscany,
(2) Tuscany is in Italy,
So (3) Siena is in Italy.

F

(1) Kermit is green.
So (2) Kermit is chromatically coloured.

The inference steps here are also all valid. We can again abstract to get shareable
principles of inference that can recur in other arguments. But again, the schemas
revealing these shareable principles will involve concepts which aren’t topicneutral (for example, the related concepts of north of and south of ).

6.2 Logical validity, at last
(a) Having noted that there are arguments whose deductive validity depends on
features of special-interest concepts such as those to do with family connections
or geographical relations etc., we will now put them aside. Throughout this
book, we will be concentrating on the kinds of valid arguments illustrated in our
earlier examples – i.e. arguments whose load-bearing principles of inference can
be laid out using only topic-neutral notions. Such arguments involve patterns of
reasoning that can be used when talking about any subject matter at all. And
these universally applicable patterns of reasoning are the special concern of logic
as a discipline (yet another idea that goes back to Aristotle).

45

6 Logical validity
(b) It is useful to have some terminology to mark off these core cases of valid
arguments which rely on topic-neutral principles of inference. Acknowledging the
traditional focus of logic, we will say:
An inference step is logically valid if and only if it is deductively valid in
virtue of the way that topic-neutral notions occur in the premisses and
conclusion.
We likewise say that some premisses logically entail a certain conclusion if
the inference from the premisses to the conclusion is logically valid.
Let a purely logical schema be one involving only schematic variables and topicneutral vocabulary. (For convenience, we will allow limiting cases such as ‘A,
so A’, which is a schema built just from variables. We will also allow limiting
cases like ‘There is something; so there is not nothing’ which lacks variables and
only features topic-neutral vocabulary; this can count as a purely logical schema
whose only instance is itself.) Then we can equivalently say: a logically valid
inference is an instance of a purely logical schema all of whose instances are
necessarily truth-preserving.
In this usage, the inferences in arguments A to F in this chapter, although
deductively valid in our original sense, do not count as logically valid. Contrast
our bold-labelled examples of valid arguments in earlier chapters; apart from one
possible exception to which we return in a moment, those are logically valid.
(c) How can we show that an unobviously valid inference is logically valid? By
a suitable proof, of course – meaning a proof where the pattern of inference used
at each step can be displayed using a purely logical schema. See for example
our fully annotated proof labelled D0 in §4.2: by inspection, this not only shows
that the inference from the initial premisses to the final conclusion is deductively
valid but also that it is, more specifically, logically valid.
How can we demonstrate that an argument is not logically valid? If we can
show that the argument is not deductively valid (by a counterexample, say),
then that settles the question. But this point won’t help us assess an example
like the Kermit argument which is deductively valid. However, in the case of
that argument, we can note that the most structure we can expose with a purely
logical schema is n is F, So n is G; and that is not a reliable pattern of inference!
So the Kermit argument is indeed not logically valid. Similar reasoning can be
used in many other cases.
(d) Our wider concept of deductive validity as defined in §2.1 and our new narrower concept of logical validity are both standard, and the distinction between
the two has a long history. For example, the medieval logician John Buridan
similarly distinguishes between what he would call a consequence (a necessarily truth-preserving inference) and a formal consequence (meaning an inference
which is necessarily truth-preserving just in virtue of its form – where he thinks
of form narrowly, in terms of the distribution of logical words like ‘and’ and ‘or’,

46

Logical necessity
‘some’ and ‘all’). Most logicians use logical consequence to mean what Buridan
does by formal consequence.
Note, however, that the labels attached to the wider and narrower concepts
of validity by different writers vary. In particular, those writers who foreground
just one of these two concepts will tend to call that one, whichever it is, simply
‘validity’ (unqualified). So care is needed when comparing different textbook
treatments.

6.3 Logical necessity
We say that an inference step is logically valid if and only if it is necessarily truthpreserving in virtue of how topic-neutral logical notions feature in its premisses
and conclusion. Similarly, we will say:
A proposition is a logically necessary truth (or more simply, is logically
necessary) if and only if it is necessarily true in virtue of how topic-neutral
notions feature in it.
So compare Whatever is green is coloured with Whatever is both green and square
is green. The first proposition is true of every possible situation, so is necessarily
true in the sense of §2.1(d). But it is not logically necessary in our sense, for
its truth depends on the internal connection between being green and being
coloured. But the second proposition is an example of the schematic pattern
‘Whatever is F and G is F ’, any of whose instances has to be true just in virtue
of the meanings of the topic-neutral logical notions ‘whatever’, ‘is’, and ‘and’.
For that reason, the second proposition is said to be logically necessary.
Generalizing, a logically necessary truth will be an instance of a type of proposition which can be specified by a purely logical schema using just schematic
variables and topic-neutral vocabulary, where all of the instances of that schema
are necessarily true.
As noted back in §2.1, the notions of deductive validity and necessary truth
are tied tightly together. Thus, the single-premiss inference A, so C is valid
if and only if it is necessarily true that if A, then C. We should now remark
that, exactly similarly, the notions of logical validity and logical necessity also
fit tightly together: A, so C is logically valid if and only if it is logically necessary
that if A, then C.

6.4 The boundaries of logical validity?
We can say, in a summary slogan, that an argument is logically valid just if it is
valid in virtue of its topic-neutral form.
But, on second thoughts, how secure is our grasp on the notion of topicneutrality here? We started making a list of topic-neutral vocabulary like ‘all’
and ‘some’, ‘and’ and ‘or’, etc.; but exactly how do we continue the list, and
where do we stop?

47

6 Logical validity
It in fact isn’t entirely obvious what is to count as suitably topic-neutral
vocabulary (without going into details, there are various stories on the market,
but no agreed one). Hence it is not obvious what counts as a purely logical,
topic-neutral, schema in our sense. Hence it is not obvious either just what will
count as a logically valid inference, valid in virtue of its form as captured by a
purely logical schema.
Here’s a case to think about. Take the argument
G

(1) Bill is taller than Chelsea,
(2) Chelsea is taller than Hillary,
So (3) Bill is taller than Hillary.

This is valid, and our first attempt at exposing a relevant general pattern of
inference here might be
m is taller than n
n is taller than o
So: m is taller than o.
And this isn’t a purely logical, topic-neutral, schema.
But hold on! Can’t we expose some more abstract inferential structure here?
– like this, perhaps:
m is F-er than n
n is F-er than o
So: m is F-er than o.
Here is F-er than is the comparative of F (in other words, it is equivalent to is
more F than). And this pattern of valid inference using the comparative-forming
construction arguably is topic-neutral and so purely logical.
But hold on again! We in fact can only talk of one thing being more F than
another thing, when F stands in for the kind of property that comes in degrees,
as in tall, wise, heavy, dark, flat, etc. But we can’t take comparatives of other
properties like even and prime (of numbers), dead (of people), valid (of arguments), and so on. So arguably the comparative construction is . . . -er than is
not an entirely topic-neutral device after all: it depends on what we apply it to
whether it makes sense.
So is our last schema fully topic-neutral or not? We can’t pursue this issue
further here; we will have to leave the question hanging, and so leave it unsettled
whether G counts as logically valid. We will similarly leave it unsettled whether
the deductively valid ‘Everyone loves a lover’ argument B in §4.1 also counts as
logically valid (that will depend on how you think of the link between ‘is a lover’
and ‘loves someone’).
Fortunately, that’s no problem. We just don’t need to worry about the outer
boundaries of the notion of logical validity: our concern will only be with some
clear central cases. We will be considering various ways in which validity can
depend on limited selections of quite uncontroversially topic-neutral vocabulary
like ‘all’ and ‘some’, ‘and’ and ‘or’, etc. And these cases will be absolutely clear
examples of logical validity. Likewise, we will be focussing on clear central cases

48

Definitions of validity as rational reconstructions
of logical necessity. Much more on this in due course, starting in real earnest in
Chapters 14 and 15.

6.5 Definitions of validity as rational reconstructions
How well does the definition of logical validity in §6.2(b) tally with our initial
hunches about what counts as a completely watertight, absolutely compelling
inference step that depends on topic-neutral logical notions?
There are a number of potential issues here (apart from the question about
what exactly counts as ‘topic-neutral’). Let’s focus on just one central issue. In
fact, we could have raised this same issue about our earlier definition of the
wider notion of deductive validity back in §2.1; but it would have been far too
distracting to mention it there, right at the very beginning.
To bring out the worry, consider the following argument:
H

Jack is married. Jack is not married. So the world will end tomorrow!

Plainly, just in virtue of the role of ‘not’ here, there is no possible situation
in which the premisses of this argument are true together (being married and
being not married rule each other out). Hence there is no possible situation in
which the premisses of this argument are true together and the conclusion is
false. Hence, applying our definition, the inference step in H counts as logically
valid (as is any inference of the form A, not-A, so C, for any propositions A and
C). Which might well seem a very unwelcome verdict. How can we think of this
inference as compelling, as ‘necessarily truth-preserving’ ?
There are two possible lines of response. The first runs like this:
Recall Aristotle’s appealing definition of a correct deduction as one whose
conclusion “results of necessity” from the premisses. And surely the conclusion of any compelling deduction should have something to do with the
premisses: so how could some claim about the end of the world really follow from propositions about Jack’s marital status? Argument H commits a
gross fallacy of irrelevance.
Our definition of deductive validity in §2.1(a), and our derived definition of the narrower notion of logical validity in §6.2(b), therefore both
overshoot – they count too many inferences as logically cogent. So, the definitions need to be revised: they need to be tightened up by introducing
some kind of relevance-requirement in order to rule out such daft examples
as the apocalyptic H counting as ‘valid’. Back to the drawing board!
The trouble is that when we get back to the drawing board, we find it is very
difficult to respect the hunch that H is not a cogent argument without offending
against other, equally basic, intuitions (see §20.9). And this prompts a different
response to that initially unwelcome implication of our definition of validity:
Arguably, no crisp definition of deductive validity (or of the special case of
logical validity) can be made to fit all our untutored hunches about what
is and what isn’t an absolutely compelling inference. The aim of definitions

49

6 Logical validity
of validity, then, is to give tidy ‘rational reconstructions’ of our everyday
concepts which at least smoothly capture the uncontroversial core cases.
The classical definitions of the notions of deductive and logical validity do
this very neatly and naturally.
True, we do get some odd results like the verdict on H. But this is a small
price to pay when tidying up the notion of validity. After all, H’s premisses
can never be true together; so we can’t ever use this type of officially valid
inference step to establish the irrelevant conclusion as true (it is, as we might
put it, only ‘vacuously’ truth-preserving because there can be no truth in
the premisses together to preserve). Our definition therefore sanctions an
entirely harmless extension of the intuitive idea of an absolutely compelling
inference step. Similarly (we hope!) we can live with a few other initially
odd-looking implications of our classical definitions.
So which line shall we take? Shall we revise our definitions of deductive and
logical validity, or swallow their initially unwelcome consequences?
We will take the second line (and this is very much the majority response
among modern logicians). Later we will define technical notions of validity like
Chapter 15’s ‘tautological validity’; these definitions will similarly count arguments like H as valid. Again, we will take the majority line that it is worth
paying this price to get otherwise neat and natural notions into play.
It is important to emphasize that official definitions of logical notions are quite
typically arrived at on the basis of cost-benefit assessments like this; we will meet
other examples later. The best logical theory isn’t handed down, once and for
all, on tablets of stone. We repeatedly have to balance, say, a certain natural
simplicity or smoothness of theory over here against a corresponding artificiality
over there. And the best choices for definitions, i.e. the best choices for rational
reconstructions of informally messy ideas, can depend on our particular theoretical priorities in a given context.

6.6 Summary
Vocabulary like ‘all’ and ‘some’, ‘and’ and ‘or’, etc. (plus the likes of ‘is’ and
‘are’), not about particular things or properties, but useful in discussing any
topic, is said to be topic-neutral.
A purely logical schema is one involving only schematic variables and topicneutral vocabulary.
A logically valid inference is an instance of a purely logical schema all of
whose instances are necessarily truth-preserving.
A logically necessary proposition is an instance of a purely logical schema
all of whose instances are necessarily true.
It isn’t altogether clear what counts as topic-neutral vocabulary, so it is not
clear either where to draw the boundaries of logical validity.

50

Exercises 6

Exercises 6
(a) Which of the following arguments are deductively valid? Which are logically
valid? (Defend your answers, as best you can.)
(1) Only logicians are wise. Some philosophers are not logicians. All who love
Aristotle are wise. Hence some of those who don’t love Aristotle are still
philosophers.
(2) The Battle of Hastings happened before the Battle of Waterloo. The Battle
of Marathon happened before the Battle of Hastings. Hence the Battle of
Marathon happened before the Battle of Waterloo.
(3) Jane is no taller than Jill, Jill is no taller than Jo, Jo is no taller than Jane.
So Jane, Jill, and Jo are the same height.
(4) Jane is taller than Jill, Jill is taller than Jo, Jo is taller than Jane. So Jane,
Jill, and Jo are the same height.
(5) Someone loves Alex, but Alex loves no one. The person who loves Dr Jones,
if anyone does, is Dr Jones. So Alex isn’t Dr Jones.
(6) Whoever respects Socrates respects Plato too. All who respect Euclid respect
Aristotle. No one who respects Plato respects Aristotle. Therefore Jo respects
Euclid if she doesn’t respect Socrates.
(7) Jill is a good logician only if she admires either Gödel or Gentzen. Jill admires
Gödel only if she understands his incompleteness theorem. Whoever admires
Gentzen must understand his proof of the consistency of arithmetic. No one
can understand Gentzen’s proof of the consistency of arithmetic without also
understanding Gödel’s incompleteness theorem. So if Jill is a good logician,
then she understands Gödel’s incompleteness theorem.
(8) All the Brontë sisters supported one another. The Brontë sisters were Anne,
Charlotte, and Emily. Hence Anne, Charlotte, and Emily supported one another.
(9) There are exactly two logicians at the party. There is just one literary theorist
at the party. No logician is a literary theorist. Therefore, of the party-goers,
there are exactly three who are either logicians or literary theorists.
(10) There are no unicorns. Hence the set of unicorns is the empty set.
(11) There is water in the cup. Hence there is liquid H2 O in the cup.
(12) Necessarily, water is H2 O. Hence it is not possible that water isn’t H2 O.
(13) It is possible that it is cold. It is possible that it is rainy. Hence it is possible
that it is cold and rainy.
(14) This argument is valid. Hence this argument is invalid.

(b) ‘We can treat an argument like “Jill is a mother; so, Jill is a parent” as having a
suppressed premiss: in fact, the underlying argument here is the logically valid “Jill is
a mother; all mothers are parents; so, Jill is a parent”. Similarly for the other examples
given of arguments that are supposedly deductively valid but not logically valid; they
are all enthymemes, logically valid arguments with suppressed premisses. The notion
of a logically valid argument is all we need.’ Is that right?

51

7 Propositions and forms
One last general topic before we get down to formal business. Propositions – the
ingredients of arguments – are, we said, the sort of thing that can be true or
false (as opposed to commands, questions, etc.). But what is their nature?

7.1 Types vs tokens
We begin with two sections introducing relevant distinctions. Firstly, we want
the distinction between types and tokens. This is best introduced via a simple
example.
Suppose then that you and I take a piece of paper each, and boldly write
‘Logic is fun!’ a few times in the centre. So we produce a number of different
physical inscriptions – perhaps yours are rather large and in blue ink, mine are
smaller and in black pencil. Now we key the same encouraging motto into our
laptops, and print out the results: we get more physical inscriptions, first some
formed from pixels on our screens and then some formed from printer ink.
How many different sentences are there here? We can say: many, some in ink,
some in pencil, some in pixels, etc. Equally, we can say: there is one sentence
here, multiply instantiated. Evidently, we must distinguish the many different
sentence-instances or sentence tokens – physically constituted in various ways, of
different sizes, lasting for different lengths of time, etc. – from the one sentential
form or sentence type which they are all instances of.
We can of course similarly distinguish word tokens from word types, and
distinguish book tokens – e.g. printed copies – from book types (compare the
questions ‘How many books has J. K. Rowling sold?’ and ‘How many books has
J. K. Rowling written?’).
What makes a physical sentence a token of a particular type? And what
exactly is the metaphysical status of types? Tough questions that we can’t answer
here! But it is very widely agreed that we need some type/token distinction,
however it is to be elaborated.

7.2 Sense vs tone
Next, we will say that the sense of a sentence – or at least the sense of a sentence
apt for propositional use in stating a premiss or conclusion – is that aspect or

52

Are propositions sentences?
ingredient of its meaning that is relevant to questions of truth or falsity. This
use of ‘sense’ is due to the great nineteenth-century German logician Gottlob
Frege (translating his ‘Sinn’).
In other words, the sense of a sentence fixes the condition under which it is
true. Distinguish this from the ‘colouring’ or ‘flavour’ or tone which the choice
of different words might give a sentence.
For example, if I refer to a particular woman as ‘Lizzie’ rather than ‘Elizabeth’,
the familiar tone may reflect my closeness (or my lack of respect). Similarly, to
adapt an example of Frege’s, whether I refer to her mount as a ‘horse’ or ‘steed’
or ‘nag’ or ‘gee-gee’ may reflect how I regard the beast (or depend on whether
I’m talking to a child). But these differences of tone need not affect the truth
or falsity of ‘Elizabeth’s/Lizzie’s horse/steed/nag/gee-gee is black’ – the various
permutations can have the same truth-relevant sense and be true in just the same
worldly situations. Or at least, so goes a very natural story. And we don’t have to
buy any particular theory about sense to acknowledge that we do need to make
some such distinction between core factual meaning and its embellishments.
Now, as far as logic is concerned, questions of colouring or tone aren’t going to
matter – those aspects of the overall meaning of claims don’t affect their logical
properties. What matters for validity is preservation of unvarnished truth. So it
is the sense of interpreted sentences that we will care about.

7.3 Are propositions sentences?
Return, then, to the question we posed at the beginning of the chapter. What
is a proposition? Here is one initially appealing view:
Propositions – potential premisses and conclusions and the bearers of truth
and falsity – are declarative sentences (i.e. sentences like ‘Jack kicks the ball’
as opposed to interrogatives like ‘Does Jack kick the ball?’ or imperatives
like ‘Jack, kick the ball!’).
But having made the type/token distinction, how are we now to gloss this seemingly straightforward suggestion?
(a) Harry and Hermione are taking a logic course. As they read the first chapter
of their respective copies of this book, they meet different tokens of the types
All philosophers are eccentric. Jack is a philosopher. So Jack is eccentric.
But it seems very odd to say that Harry and Hermione are thereby tangling with
different arguments. It is surely the very same argument, with the same premisses
and conclusion, which they both encounter and both get to think about.
Generalizing, we surely want to say that the same argument can appear in the
many different printed copies of this book, and in the e-copies too. So it seems
that we naturally think of arguments as types which can have many instances,
rather than as tokens. And what goes for arguments then goes for the constituent
propositions which are their premisses and conclusions.

53

7 Propositions and forms
Hence, the view that propositions are declarative sentences seems to be more
naturally glossed as claiming that propositions are sentence types.
(b) But, even with that clarification, a naive identification of propositions with
declarative sentences won’t do as it stands. Here are some problems:
(1) Don’t some grammatically acceptable declarative sentences lack sense?
Take, for example, ‘Purple prime numbers sleep furiously’ (it is arguably just nonsense to talk of numbers as either coloured or sleeping,
and also nonsense to talk of sleeping furiously). And it seems wrong to
say that a senseless sentence can be an ingredient of a real argument,
i.e. can be a contentful proposition.
(2) Other grammatical sentences have too many senses. Take, for example,
‘Visiting relatives can be boring’. This sentence can say two different
things, one of which may be true and the other false: so, same sentence,
different truth-evaluable messages conveyed, hence – it seems natural to
say – different possible ingredients of arguments, different propositions.
(3) The converse situation to (2): different sentences can surely express the
same premisses/conclusions. Adopting an example of Frege’s, consider
the arguments ‘The Greeks defeated the Persians at Plataea. So the
Persians were defeated at least once’ and ‘The Persians were defeated
by the Greeks at Plataea. So the Persians were defeated at least once.’
Aren’t these the same argument, with the active and passive versions
of the premiss just stylistic variants expressing the same thought? Or
take the arguments ‘Jack is both tall and slim. So, Jack is tall’ and
‘Jack is tall and is slim. So, he is tall.’ These strictly speaking involve
distinct pairs of sentences; yet again it seems odd to say that the minor
stylistic variation gives us different arguments.
(4) We may use a sentence like ‘He is tall’ in framing an argument, as we’ve
just done. But this sort of sentence has – as it were – an incomplete
sense, i.e. it won’t by itself express a determinate proposition which
can be a premiss or conclusion. We need context to fix who is being
referred to.
In sum, the same declarative sentences can be used to express different arguments, and different sentences can be used to express the same arguments. And
context can matter. So we can’t simply identify propositions, the ingredients of
arguments, with declarative sentences.
Still, we can perhaps work around these initial difficulties for the sentential
view of propositions by adding some qualifications (see §7.5). But there is a more
radical problem which we have saved for last. In (3) we suggested that sentences
which are stylistic variations of each other can express the same premiss or
conclusion. But can’t sentences that are entirely different also count as expressing
the same proposition?
(5) Surely the very same argument can be discussed by Aristotle, by medieval logicians, and then by Frege. And the very same argument can

54

Are propositions truth-relevant contents?
now appear both in this book and in its Italian translation, with the
same premisses and conclusions again expressed by entirely different
sentences. So what makes this the same argument all along, it now can
seem natural to claim, has to be what is said rather than how it is
said – i.e. must be the messages expressed, whether in Greek, Latin,
German, English, or Italian.

7.4 Are propositions truth-relevant contents?
Considerations like those we have just been discussing seem, then, to push us
towards a second view, along the following lines:
Propositions, potential premisses and conclusions and the bearers of truth
and falsity, are not declarative sentences but rather the messages that
declarative sentences can be used, in context, to express.
Some would talk of ‘thoughts’ rather than ‘messages’ as what are expressed by
declarative sentences – meaning possible thought-contents as opposed to acts of
thinking. And some talk just of ‘contents’. So the idea is that a proposition is
something language-independent, a content that can be shared by sentences in
different languages which have the same sense.
(An annoying terminological aside. We have from the outset been using the
word ‘proposition’ in a non-committal, theory-neutral, way – leaving it as an
open question what propositions are. Confusingly, many philosophers instead use
‘proposition’ quite specifically to refer to these putative truth-relevant thoughtcontents that can be expressed by sentences in various languages. Though just
to complicate things further, many medieval logicians and those modern writers
most influenced by them use ‘proposition’ in exactly the opposite way, specifically
to refer to declarative sentences themselves. Sorry about that! – you just have
to be alert when reading other authors.)
Whatever the terminology, however, the trouble with our second view about
the bearers of truth and falsity is that it tells us what premisses and conclusions are not – they aren’t sentences – but their positive nature is now quite
mysterious. In fact it is entirely unclear what kind of theory to offer about messages or contents when thought of as language-independent. We can’t stop to
review a hundred years of arguments about this: let’s just say that no particular
attempt to give a theory of propositions-as-contents has ever commanded very
wide support.

7.5 Why we can be indecisive
(a) So maybe we should after all try to rescue the less puzzling first view –
propositions are sentences – by fine-tuning it. Consider, then, the revised suggestion that propositions are fully interpreted declarative sentences – i.e. they
are disambiguated sentences parsed as having one determinate sense, with context supplying the references of pronouns etc. Then we may equate ‘Jack is tall’

55

7 Propositions and forms
and ‘he is tall’ when the pronoun refers to Jack; and we can perhaps also allow
minor grammatical massaging, e.g. equating active and passive versions of the
same sentence. In this way, we might hope to deal with difficulties (1) to (4). On
the other hand, we can and should perhaps just bite the bullet, and respond to
(5) by insisting that the premisses and conclusions of an argument in this book
and the corresponding premisses and conclusions in its Italian translation are
not strictly speaking the same after all, because the sentences are quite different
– rather, there will be two distinct arguments which are more or less smooth
translations of each other.
How, though, are we to nail down some version of this revised suggestion,
based on the idea that propositions are fully interpreted sentences (allowing for
minor grammatical variations)?
(b) We will have to leave that question hanging. Having flagged up two different
ways of thinking about propositions – as sentences and as contents – we aren’t
going to develop either approach any further, let alone decide between them.
Now, sitting on the fence about the nature of ordinary propositions, i.e. about
the nature of premisses and conclusions in everyday arguments, may sound irresponsible; how can we possibly leave unresolved such a very basic question as
what are arguments made of ? For our purposes in this book, however, it turns
out that we happily won’t need to adjudicate this tricky issue in ‘philosophical
logic’. Why so? Because – spoiler alert! – our key technique for assessing everyday
arguments will involve (a) rendering them into artificial formalized languages,
and then concentrating our logical efforts on (b) assessing arguments once tidily
formalized. Stage (a) won’t require us to say that the formal versions are the very
same arguments, just that they are close-enough translations. And crucially, the
sentences of our artificial formalized languages explored at stage (b) will by design be free of ambiguities and context dependence – they will come already fully
interpreted, stipulated to have a determinate sense. This means that formalized
sentence types and the messages they convey will be neatly aligned, and for our
purposes we needn’t fuss too much about distinguishing them.
Hence, in sum, it will do our project no harm to take arguments framed in
formalized languages to be simply made up of formal sentences.

7.6 Forms of inference again
(a) We have just asked how we should think of the premisses and conclusions
of everyday arguments: are they sentences or are they the messages expressed
by sentences (i.e. thought-contents)? We can now raise a related issue: when we
talk about forms of inference in everyday arguments, are we referring to patterns
to be found at the surface level of the sentences used to state the argument, or
should we primarily be thinking of patterns at the level of messages expressed
(whatever exactly that means)?
In fact, we have been cheerfully casual about this. On the one hand, we have
discussed patterns of inference to be found in arguments couched in English by

56

Forms of inference again
using schemas in which we are to systematically substitute English expressions
for the schematic variables – allowing for some grammatical tidying. Which
chimes with the policy of thinking of the ingredients of arguments as sentences,
and with thinking of patterns of inference as patterns to be found on the surface,
at sentence level.
On the other hand, when we laboriously stated in words the principles underlying our first couple of examples of reliable inference schemas (in §1.5 and
§3.1), we found ourselves talking about what premisses and conclusions say, and
this looks to be at the level of messages expressed. It is very natural to slide
into this way of speaking. After all, weren’t the great dead logicians – who were
considering arguments couched in entirely different sentences from (say) Greek,
Latin, or German – in some sense discussing the very same forms of inference
that occur in our arguments couched in English?
(b) Leaving aside arguments in other languages, the issue already arises within
a single language. Take for example the argument
All dogs have four legs. Fido is a dog. Fido has four legs.
And now consider the variants with the alternative first premisses ‘Every dog
has four legs’, ‘Any dog has four legs’, and ‘Each dog has four legs’.
Thinking at the level of sentences, these four arguments exemplify different
forms of inference, because they involve the distinct logical words ‘all’, ‘every’,
‘any’, and ‘each’. However we might well be inclined to suppose that the differences in these cases are only superficial and that the underlying inferential
structure is in some sense really the same. The respective first premisses of
these arguments are just stylistically different ways of expressing the very same
thought-content; and then the rest of the arguments are the same. It is tempting, then, to suppose that although these various versions may differ in surface
sentential form, they in some way share the same underlying ‘logical form’ –
so, thinking at the level of the messages expressed by the various sentences, the
inferences are all instances of a single form.
We might even recruit the familiar schema ‘All F are G, n is F, so n is G’ to
represent this supposed underlying shared form. And in fact schemas are very
often used in this way in philosophical writing, i.e. they are treated as fitting
the surface form of arguments only quite loosely.
(c) So which should it be, when thinking about the forms of inference in
everyday arguments? Should we be looking for patterns at sentence level, or
for underlying patterns in the messages expressed?
As we have seen, the second line can be tempting. But do we really understand
what it comes to – for what kind of theory of the nature of messages would be
needed for it to make sense? As we have already noted, there is no widely agreed
account of propositions-as-messages that we can appeal to for help.
Fortunately, at an introductory level, it doesn’t matter much whether we say
that Aristotle was discussing the very same forms of argument as we might use,
or whether we instead say that he was considering the Greek equivalents of our

57

7 Propositions and forms
forms of argument. And much more importantly – another spoiler alert! – when
we start to consider arguments regimented into formalized languages, things
become very clean and simple. As we said, formal arguments can be thought of
as made up of formal sentences; and then the inferential forms of such arguments
can be understood quite unproblematically as formal patterns at the sentential
level. More about this soon.

7.7 Summary
There are two main types of view on the nature of the propositions in
ordinary arguments: they are sentences (perhaps fully interpreted sentencetypes) or alternatively they are the messages or thought-contents that sentences can be used to express.
We need not adjudicate: our focus from now on will be on propositions
in formalized arguments, and these can unproblematically be treated as
sentences.
Likewise, there are differing views about what forms or patterns of inference
are patterns in – sentences or messages? Again we do not need to adjudicate:
our focus from now on will be on forms of inference in formalized arguments,
and these can unproblematically be treated as patterns in the surface form
of formal sentences.

Exercises 7
Consider the following exchange:
Jack: Mary took her picnic to the bank.
Jill: Mary took her picnic to the bank.
And assume that context makes it clear that Jack means bank in the sense of the river
side, and Jill means bank in the sense of a financial institution.
In this particular exchange, then, we have instances of one sentence type (as identified by its surface form). These instances have two different senses (different truthrelevant literal meanings). And these instances are used to express two different messages or thoughts (or propositions, in one sense of that overused word).
There are potentially eight different kinds of exchanges between Jack and Jill with
one utterance each as in our example; they can involve instances of one or two different
sentence types, having one or two different senses (literal meanings), expressing one or
two different messages or thoughts or propositions.
Give an example of each combination which is in fact possible.

58

Interlude: From informal to formal logic
(a)

What have we done so far? In bare headlines,

We have explored, at least in an introductory way, the (classical) notion
of a valid inference step, and the corresponding notions of a deductively
valid/sound (one-step) argument.
We have seen how to distinguish deductive validity from other virtues that
an argument might have (like being a highly reliable inductive argument).
We have noted how different arguments can share the same form of inference.
And we have seen how to exploit this fact in using the counterexample
method for demonstrating invalidity.
We have seen some simple examples of direct multi-step proofs, where we
show that a conclusion really can be validly inferred from certain premisses
by filling in the gap between premisses and conclusion with evidently valid
intermediate inference steps. We in addition briefly looked at one kind of
indirect method of proof, reductio ad absurdum.
We have also met the narrower notion of logical validity – where an inference
step is valid in this narrower sense if it is deductively valid in virtue of the
way that topic-neutral notions feature in the premisses and conclusion.
Along the way, we have had to quietly skate past a number of issues, leaving more
needing to be said. But hopefully you will have gained at least a rough-and-ready
preliminary understanding of some key logical concepts. And you need such an
understanding if you are to see the point of the more formal investigations which
follow.
(b) So what next? One good option would be to spend more time on techniques
for teasing out the arguments involved in passages of extended prose argumentation, to develop further methods of informal argument analysis, explore how to
reason with a range of key logical notions like ‘if’ and ‘all’, and then catalogue a
variety of common ways in which everyday arguments can go wrong. This kind of
study in informal logic (as it is often called) can be a highly profitable exercise.
But our focus in this book – as its title suggests! – will be rather different.
Instead of going for breadth of coverage, we aim for depth; we will develop systematic and rigorous treatments for some highly important but limited classes
of arguments. Moreover – and this is a crucial move – the arguments we focus on

59

Interlude
will be formalized arguments, framed in purpose-designed formalized languages.
These formalized languages will be written in special symbols, but that is really
just a convenience. The essential thing is that these artificial languages are governed by simple grammatical rules which fix which expressions are sentences
and which aren’t, and by simple semantic rules which fix sharp and unambiguous truth-conditions for those expressions which are sentences. This way we
don’t have to tangle with all the complexities of ordinary language. Rather, we
can evaluate arguments after they have been rendered into a tidier shape, by
translation into suitable formalized languages. More on this key policy decision
very soon.
(c) The resulting first-order quantification theory – the main branch of formal
logic which we will eventually be introducing – is (as we said in the Preface)
one of the great intellectual achievements of formally-minded philosophers and
of philosophically-minded mathematicians. It is beautiful in itself, and it opens
the door onto a very rich and fascinating field. (In this introductory book, there
will only be very occasional glimpses further through that door; but we will at
least get to the threshold.)
Quantification theory explores the logic of arguments involving quantifiers
(expressions of generality like ‘all’, ‘some’, and ‘none’ etc.) and explains how these
expressions interact with the so-called propositional or sentential connectives
(‘and’, ‘or’, ‘if’, ‘not’). It gives us a framework in which most, perhaps all, of the
deductive reasoning needed in science and mathematics can be conducted. We
will take things slowly, however. Before turning to the full theory, we are going
to be spending a lot of time on the limited fragment of this logical system which
deals just with the connectives: this is propositional logic.
Why so much effort exploring propositional logic? Because this makes for a
relatively painless introduction to our subject, by allowing us to meet a whole
range of basic ideas and strategies of formal logic in a very accessible context.
This will very considerably ease the path into full quantification theory.
Let’s get straight to work!

60

8 Three connectives
We start our more formal work by looking at length at a very restricted class
of arguments, namely those whose relevant logical structure turns simply on the
presence in premisses and/or conclusions of ‘and’, ‘or’, and ‘not’ (used as socalled sentential or propositional connectives). We begin by explaining the great
advantage of working in artificial formalized languages, even when exploring the
logic of arguments relying on just these three sentential connectives.

8.1 Two simple arguments
We met the following trivially valid argument in the first chapter:
A

(1) Either Jill is in the library or she is in the coffee bar.
(2) Jill isn’t in the library.
So (3) Jill is in the coffee bar.

We can represent this as having the form
Either A or B
Not-A
So: B.
Here, the symbols ‘A’, ‘B’ stand in for suitable sentences, and – as we have
already informally done in e.g. §2.1(d) and §4.5(c) – we are using ‘not-A’ to
indicate a sentence that expresses the denial of what ‘A’ says (so is true just
when A is false). Evidently, any inference of this form will be valid.
Here is another valid argument:
B

(1) It’s not the case that Jack played lots of football and also did
well in his exams.
(2) Jack played lots of football.
So (3) Jack did not do well in his exams.

This also instantiates a reliable pattern of inference, which we might represent:
Not-(A and B)
A
So: Not-B.
Any inference of the same type will again be valid, this time just in virtue of the
meaning of ‘and’ and the way denial works.

61

8 Three connectives
Why did we use brackets in representing the structure of B’s first premiss?
Because we plainly need to distinguish between a claim of the form
Not-(A and B)
which denies that A and B both hold together, and a claim of the form
(Not-A) and B
which denies A, but then adds that B does hold. Suppose we put
A: The President will order an invasion.
B : There will be devastation.
Then the first schema yields the hopeful thought that we won’t get an invasion with devastation; while the second schema yields the depressing claim that
although the President will not order an invasion, there will still be devastation.
Arguments A and B are trite, but do illustrate a couple of types of inference
whose validity depends on the distribution in the premisses and conclusion of
‘and’, ‘or’, and ‘not’. Our task is to explore such arguments more systematically.

8.2 ‘And’
(a) How does ‘and’ work in ordinary language? Note first that it can be used
to conjoin a pair of matched expressions belonging to almost any grammatical
category – as in ‘Jack is fit and tanned’, ‘Jill won quickly and easily’, ‘Jack
and Jill married’, or ‘Jo smoked and coughed’, where ‘and’ conjoins pairs of
adjectives, adverbs, proper names, and verbs. But leave aside all those uses, and
consider only the cases where ‘and’ operates at sentence level, i.e. it acts as a
sentential connective and joins two whole sentences to form a new one.
For example, take the sentences ‘Lyon is in France’ and ‘Turin is in Italy’; we
can conjoin them to get ‘Lyon is in France and Turin is in Italy’. This resulting
compound sentence is then true (or, if you prefer, expresses a truth) exactly
when both of the constituent sentences are true, and it is false otherwise.
Let’s use ‘’ to stand in for some expression which works as a binary connective
(i.e. one that connects two sentences to form another). Then we will say:
A  B is a conjunction of A and B just if A  B is true when both of A and
B are true, and is false when at least one of A and B is false.
(b) In our sample sentence ‘Lyon is in France and Turin is in Italy’, the connective ‘and’ serves to form a conjunction in this sense. But English has other
ways too of forming conjunctions. For note that ‘Lyon is in France but Turin is
in Italy’ is also true just when Lyon is in France and Turin is in Italy. And more
generally, pairs of claims of the form A but B and A and B do not differ in what
it takes for them to be true. Rather, in the Fregean terms of §7.2, the sentences
differ in colour or tone, not sense – the contrast between ‘but’ and the colourless
‘and’ is (very roughly) that A but B is typically used when the speaker takes it
that there is some kind of contrast between the truth or the current relevance
of A and of B.

62

‘Or’
So we can use ‘but’ to form conjunctions. And for other ways of forming
conjunctions consider, for example, ‘Lyon is in France, though Turin is in Italy’,
‘Paris is in France; moreover Lyon is in France too’.
(c) You might wonder: doesn’t A and B often express more than simply the
joint truth of A and B? For example, compare
(1) Eve became pregnant and she married Adam.
(2) Eve married Adam and she became pregnant.
Using one of these sentences rather than the other would normally be taken as
conveying a message about the temporal order of events. So does this show that
‘and’ in English sometimes signifies temporal succession as well as the truth of
the conjoined sentences? Does ‘and’ sometimes mean the same as ‘and then’ ?
It has often been claimed so. But compare the two mini-stories
(10 ) Eve became pregnant. She married Adam.
(20 ) Eve married Adam. She became pregnant.
The divergent implications of temporal succession surely still remain, because of
our default narrative practice of telling a story in the order in which the events
happened. Since the implications of temporal order remain even without the use
of ‘and’, there is no compelling need to treat temporal succession as one possible
meaning built into the connective. Perhaps even in (1) and (2) the role of ‘and’
itself is still just to form a bare conjunction.
However, we can’t expand here on the sort of issues we have just touched
on. So let’s be conciliatory. Let’s agree that there is at least a prima facie issue
about whether ‘and’, when used as a sentential connective in ordinary language,
sometimes forms more than a bare conjunction. When we turn to considering
arguments involving ‘and’, we will therefore want to make it absolutely clear
how the connective is being used.

8.3 ‘Or’
(a) Like ‘and’, we can use ‘or’ to combine a pair of matched expressions belonging to almost any grammatical category – as in ‘Jack is walking or cycling’,
‘Jack or Jill went up the hill’, etc. However, let’s just consider the cases where
‘or’ (with or without ‘either’) is used as a binary operator at sentence level, i.e.
is used as a sentential connective to disjoin two whole sentences.
Here is Jack who has failed his logic test. I say ‘Jack didn’t work or he isn’t
any good at abstract thought’. This is true if Jack didn’t work, and also true if
Jack is not good at abstract thought. But I don’t mean to rule out that he is
both a slacker and hopelessly illogical. The ‘or’ here is inclusive.
Using ‘◦’ to stand in for another binary connective, let’s say
A ◦ B is an (inclusive) disjunction of A and B just if A ◦ B is true when at
least one of A and B is true, and is false when both of A and B are false.

63

8 Three connectives
‘Or’ in English often serves to form an inclusive disjunction in this sense. Another
example: ‘We are now bound to win the cup. Surely we will beat the Lions or
we will beat the Tigers’ – again, note that I don’t rule out our beating both.
(b) You might wonder: isn’t it part of the meaning of A or B to signal that
the speaker doesn’t definitely know that A is true or know that B is true?
But why so? Of course, unless there is a special context (like giving a child
a hint in a game), it would be oddly uncooperative of me to assert the bare
disjunction A or B when I know perfectly well e.g. that A is in fact true. So for
that reason, if I merely assert A or B, you will normally expect me not to know
which. However, there is no linguistic oddity in saying ‘A or B – I know which,
but I’m not telling you!’.
(c) The main complication about the use of ‘or’ is that it seems, at least at first
sight, that claims of the type A or B can in fact have two different meanings.
Often, ‘or’ forms a disjunction in the inclusive sense we defined above (meaning
A or B or both). But can’t it equally well form an exclusive disjunction (meaning
A or B but not both)? For a possible example of an exclusive disjunction, consider
(1) A peace treaty will be signed this week or the war will drag on a year.
The thesis that ‘or’ is ambiguous between inclusive and exclusive senses used
to be popular. But maybe it is wrong (many linguists would say so); it could be
that ‘or’ in fact has a single, inclusive, literal meaning, with any implication of
exclusiveness in a particular case being due to extraneous clues of one sort or
another. For note that implications of exclusiveness can usually be coherently
cancelled. Thus, without any sense of linguistic oddity, we can surely say
(2) A peace treaty will be signed this week or the war will drag on a year;
indeed – given the fragility of the peace process – maybe both.
This would be unhappy if the initial disjunction in (2) were genuinely exclusive,
ruling out the ‘both’ case (as we’d then be saying ‘not both; maybe both’). So
it seems that the ‘or’ in (2) can’t be exclusive. But it seems odd to say that this
‘or’ changes its literal meaning if we now drop the last part of (2) and revert
to (1). Arguably, then, the presumption of exclusiveness for the original claim
(1) comes not from a special exclusive sense of ‘or’ but from our background
knowledge that peace treaties and wars don’t usually go together.
But that’s contentious and there is more to be said. And there are other
sorts of example to consider. So this is another area of debate we can’t pursue
any further. Being conciliatory again, we can at least agree that when we turn
to considering the logic of arguments involving ‘or’ we will want to make it
absolutely clear whether inclusive or exclusive disjunctions are in play.

8.4 ‘Not’
(a) To deny a claim A outright is equivalent to asserting some strict negation
of A (meaning a sentence which is true when A is false and false when A is true).

64

Scope
Earlier, we represented a negation of A very informally by ‘not-A’.
English has various ways of forming negations. For the simplest kind of case,
consider ‘Jo is married’ vs ‘Jo is not married’, ‘Jack smokes’ vs ‘Jack does not
smoke’, ‘Jill went to the party’ vs ‘Jill did not go to the party’. In these examples,
we negate a sentence by simply inserting a ‘not’ and then making any needed
grammatical changes.
By contrast, inserting a ‘not’ into the truth ‘Some students are good logicians’
at the only grammatically permissible place gives us another truth, namely ‘Some
students are not good logicians’; hence inserting ‘not’ doesn’t always produce the
negation of a sentence. Instead, we can negate ‘Some students are good logicians’
by changing ‘Some’ to ‘No’. Similarly, we can negate ‘Alexander sometimes lost
a battle’ by changing ‘sometimes’ to ‘never’. Again, we can negate an inclusive
disjunction of the form A or B by the corresponding Neither A nor B.
As well as these brisk but varied ways of forming negations, English has a
more cumbersome but more uniform way of forming negations – namely, we can
prefix ‘It is not the case that’ (or equivalently, ‘It is not true that’). Thus we
can negate ‘Some students are good logicians’ by saying ‘It is not the case that
some students are good logicians’. We can negate ‘Alexander sometimes lost a
battle’ by saying ‘It is not the case that Alexander sometimes lost a battle’. And
we can negate A or B by asserting It is not the case that A or B, so long as it
is clear that the negation-prefix applies to the whole of what follows it (a point
we will return to in just a moment).
(b) Let’s focus on the uniform construction. Now using ‘.’ to stand in for some
expression which is prefixed to a single sentence, we will say
.A is a negation of A just if .A is true when A is false, and is false when A
is true.
As we will see, although a prefixed negation operator doesn’t connect different
sentences, there are enough other similarities with the behaviour of conjunction
and disjunction signs to encourage us to treat them all together. It is therefore entirely standard to count a negation operator as an honorary sentential
‘connective’, a one-place or unary one.

8.5 Scope
(a) A prefixed ‘It is not the case that’ typically works as a negation in the sense
just defined. But not always. Consider, for example, the following pairs:
(1) Jack loves Jill, or Jill is much mistaken about Jack’s feelings.
(10 ) It is not the case that Jack loves Jill, or Jill is much mistaken about
Jack’s feelings.
(2) Jack loves Jill and it is not the case that Jill loves Jack.
(20 ) It is not the case that Jack loves Jill and it is not the case that Jill
loves Jack.

65

8 Three connectives
Here, (10 ) results from prefixing ‘It is not the case that’ to (1). But on the
natural way of reading (10 ), only the clause ‘Jack loves Jill’ is governed by – or is
in the scope of – the initial negation. Which means that both (1) and the natural
reading of (10 ) are true if Jill is much mistaken. Hence (10 ) isn’t unambiguously
a negation of (1).
We can make this clear by helping ourselves to brackets again, and using
‘Not’ as a negation prefix. Then if (1) is represented by A or B, then (10 ) is
naturally read as having the form (Not-A) or B. But a negation of (1) needs to
be equivalent to Not-(A or B).
Similarly (20 ) results from prefixing ‘It is not the case that’ to (2). But on the
natural reading of (20 ), again only the clause ‘Jack loves Jill’ is in the scope of the
initial negation. Which means that both (2) and (20 ) are false if Jill loves Jack.
Hence again one isn’t a negation of the other. In symbols, if (2) is represented by
A and Not-B then (20 ) is naturally read as having the form Not-A and Not-B.
But a negation of (2) needs to be equivalent to Not-(A and Not-B).
So, to form an unambiguous negation of some sentence in English it may not
be enough simply to prefix it with ‘It is not the case that’; we may also need to
indicate somehow the intended scope of the prefix.
(b) The implicit rules about how much of a sentence an ordinary-language
logical operator applies to are complicated, and don’t settle a unique reading for
every sentence. They permit ambiguities to arise. Consider
(3) Either Jack took Jill to the party or he took Jo and he had some fun.
No individual word in this sentence is ambiguous, we may suppose. But there are
still two possible messages here, because we can construe the sentence as being
put together in two different ways. Does this say that either Jack went to the
party with Jill or else Jack had fun going to the party with Jo? Or does it say,
differently, that he took Jill or Jo to the party and either way enjoyed himself?
There is an ambiguity in how to group together the clauses in (3) – and we
can think of this as a scope ambiguity. For what is the scope of the ‘and’ ? Does
the connective just conjoin ‘he took Jo’ with ‘he had some fun’ ? Or is its scope
wider, so that the whole of ‘either Jack took Jill to the party or he took Jo’ is
being conjoined with ‘he had some fun’ ? In speech, intonation and little pauses
can tell us what is intended. In writing, adding punctuation will help, as in:
(4) Either Jack took Jill to the party, or he took Jo and he had some fun.
(5) Either Jack took Jill to the party or he took Jo, and he had some fun.
When it comes to considering sentences with multiple connectives in logical
contexts, we evidently need some device to block scope ambiguities.

8.6 Formalization
To sum up so far: even when considering only their uses as sentential connectives, we find that natural-language ‘and’, ‘or’ and ‘not’ behave in quite complex ways. There are similar intricate complexities in the use of other logically

66

The design brief for PL languages
salient constructions. Treating the logic of arguments couched in ordinary English therefore really requires taking on two tasks simultaneously; we need both
to negotiate the many vagaries of English and to deal with the logical relations
of the premisses and conclusions once we have correctly construed them.
How are we going to handle this dual task? Let’s divide and rule, and separate
the two tasks as far as possible. We will sidestep the complexities of natural
language by reformulating the arguments we want to discuss; we will render
them into more austere and regimented formalized languages, languages which
are designed from the start to be very well-behaved, entirely clear and quite free
of ambiguities and shifting meanings – at least as far as their logical apparatus
is concerned. And then we can much more easily assess the resulting formalized
arguments. Applied to the present case, where just the sentential connectives
are in focus, this means:
We will assess an argument involving the English ‘and’, ‘or’, and ‘not’ as
sentential connectives by a two-step process:
(1) We render the given vernacular argument into a well-behaved artificial language with tidied-up versions of those three connectives.
(2) We then investigate the validity or otherwise of the now tidily
formalized version.
The first step may raise more or less tricky issues of interpretation, as we try
to reformulate the English propositions, imposing the logical straightjacket of a
sharply defined formalism. But most of our logical attention will then be on the
second step. For even given the premisses and conclusion now rendered into a
sufficiently perspicuous and unambiguous formalized language, the central question still remains: is the resulting argument deductively valid?
The divide-and-rule strategy has its roots as far back as Aristotle, whose logical works are the logician’s Old Testament. But the strategy really came of age
in the nineteenth century with Frege’s New Testament. In his Begriffsschrift
(1879), Frege presented a ‘concept-script’ designed for the perspicuous representation of a rich class of propositions. His own notational choices didn’t win much
favour; but the idea of using a formalized language to regiment arguments into a
more manageable shape quickly became central to the whole project of modern
formal logic.
In the rest of this book, we are going to follow the divide-and-rule strategy.
The justification of this approach is its richly abundant fruitfulness, though that
can only become fully clear as we go along.

8.7 The design brief for PL languages
For our immediate purposes, then, we want to design a formalized language – or
rather, a whole family of languages – for regimenting arguments involving our
three sentential connectives as their relevant logical apparatus. We will call such
a language a PL language – with the label meant to suggest ‘propositional logic’.

67

8 Three connectives
We can usefully carve up the design brief for a PL language into three parts.
But first, a new convention which we will say more about in Chapter 11:
When we want to generalize about sentences of artificial languages such as
PL languages by the use of symbols in schemas, instead of again using italic
letters like ‘A’, ‘B’, ‘C’, we will now use Greek letters like ‘α’, ‘β’, ‘γ’.
(a) We start, then, by giving our languages symbols for ‘and’ and ‘or’ as sentential connectives, and a symbol for sentence-negation. So:
We give a PL language a connective ‘∧’ (some use ‘&’) which is stipulated
to invariably form a bare conjunction, no more and no less: i.e. in all cases,
for all sentences α, β of the language, the sentence (α ∧ β) is true when α
and β are both true, and is false when one or both of α and β are false.
We add a connective ‘∨’ which is stipulated to invariably form an (inclusive)
disjunction, no more and no less. So the sentence (α ∨ β) is true when one
or both of α and β are true, and is false when α and β are both false.
Finally we add a prefixed ‘¬’ (some use ‘∼’) which is stipulated to work as
‘It is not the case that’ usually works. So the sentence ¬α forms a negation
of α, and is true just when the sentence α is false, and false when α is true.
(The inclusive disjunction sign ‘∨’ is supposed to be suggestive of the initial ‘v’ of
‘vel’, the Latin for ‘or’. Later, we will see how to deal with exclusive disjunction.)
By all means, pronounce ‘∧’, ‘∨’, and ‘¬’ as and, or, and not. But these new
symbols shouldn’t be thought of as mere equivalents of their ordinary-language
counterparts. For true equivalents would simply inherit the complexities of their
originals! The PL connectives are better thought of as cleaned-up replacements
for the vernacular connectives.
(b) We next need to avoid any structural scope ambiguities of the kind illustrated by ‘Either Jack took Jill to the party or he took Jo and he had some fun’.
Ambiguous expressions of the pattern α ∨ β ∧ γ must therefore be banned.
How do we achieve this? Compare arithmetic, where we can form potentially
ambiguous expressions like ‘1+2×3’ (is the answer 7 or 9?). To disambiguate, we
can use brackets, writing ‘1 + (2 × 3)’ or ‘(1 + 2) × 3’, which each have unique
values. We will insist on the same bracketing device in our PL languages:
Every occurrence of ‘∧’ to join two sentences α and β is to come with a pair
of brackets to yield the sentence (α ∧ β), thereby clearly demarcating the
scope of the connective and showing what it connects. Similarly for ‘∨’.
But the negation sign ‘¬’ only combines with a single sentence, and we don’t
need to use brackets to mark its scope.
This way, we are allowed sentences of the shape (α ∨ (β ∧ γ)) or ((α ∨ β) ∧ γ),
but not the unbracketed α ∨ β ∧ γ. And a negation sign binds tightly to the
sentential PL-expression that immediately follows it. So:

68

One PL language
(1) In a sentence of the form (α ∨ (¬β ∧ γ)), just β is being negated.
(2) In (α ∨ ¬(β ∧ γ)), the bracketed conjunction (β ∧ γ) is negated.
(3) And in ¬(α ∨ (β ∧ γ)), the whole of (α ∨ (β ∧ γ)) is negated.
(c) Finally, what do these three connectives ‘∧’, ‘∨’, and ‘¬’ (ultimately) connect? Simple, connective-free, sentences; we can think of these as the atoms from
which we build more complex molecular sentences using the connectives.
So we’ll need to give each PL language a base class of ‘atomic’ sentences –
and it is just here that the various PL languages will differ, in what atomic
sentences are available, and in what these atomic sentences mean. However,
since for present purposes we are not interested in further analysing these atomic
sentences, we will waste as little ink on them as possible. Therefore, for brevity,
we can start by using single letters as atoms, as in ‘P’, ‘Q’, ‘R’, ‘S’.
Note, using single letters like this does not imply that the atoms of our PL language are meaningless. Formalized languages, on our account of them, may differ
from natural languages in many ways; but they will resemble them in containing
genuine meaningful sentences, with which we can frame genuine arguments.

8.8 One PL language
(a) Take a PL language with just the four letters from ‘P’ to ‘S’ as atoms. And
suppose our meaning-giving glossary for the language looks like this:
P: Jack loves Jill.
Q: Jill loves Jack.
R: Jo loves Jill.
S: Jack is wise.
Then let’s render the following into this miniature formalized language:
(1)
(2)
(3)
(4)
(5)
(6)
(7)
(8)
(b)

Jack doesn’t love Jill.
Jack is wise and he loves Jill.
Either Jack loves Jill or Jo does.
Jack and Jill love each other.
Neither Jack loves Jill nor does Jo.
It isn’t the case that Jack loves Jill nor does Jill love Jack.
Either Jack is not wise or both he and Jo love Jill.
It isn’t the case that either Jack loves Jill or Jill loves Jack.
The first four are very easily done:

0

(1 )
(20 )
(30 )
(40 )

¬P
(S ∧ P)
(P ∨ R)
(P ∧ Q).

Just three quick comments:
(i) Note that rendering the English as best we can into our PL language
isn’t a matter of mere phrase-for-phrase transliteration or mechanical

69

8 Three connectives
coding. For example, in rendering (2) we have to assume that the ‘he’
refers to Jack; likewise we need to read (3) as saying the same as ‘either
Jack loves Jill or Jo loves Jill’ – and assume too that the disjunction
here is inclusive.
(ii) We are insisting that whenever we introduce an occurrence of ‘∧’ or
‘∨’, there needs to be a pair of matching brackets. To be sure, the
brackets in (20 ) to (40 ) are strictly speaking redundant; if there is only
one connective in a sentence, there is no possibility of a scope ambiguity.
No matter; our official bracketing policy will be strict.
(iii) Note too that it is customary not to conclude sentences of our PL
languages by full stops or periods. There is no such punctuation mark
in the languages. (However, we will add full-stops to displayed material
when it helpfully completes the surrounding English.)
To continue: We do not have a built-in ‘neither . . . , nor . . . ’ connective in our
language. But we can render (5) into our PL language in two equally good ways:
(50 ) ¬(P ∨ R)
(500 ) (¬P ∧ ¬R).
Next, the natural reading of the English (6) treats it as the conjunction of ‘It is
not the case that Jack loves Jill’ and ‘It is not the case the Jill loves Jack’:
(60 ) (¬P ∧ ¬Q).
And proposition (7) is to be rendered as follows:
(70 ) (¬S ∨ (P ∧ R)).
(Why are the brackets placed as they are?) Finally, we can translate (8) as
(80 ) ¬(P ∨ Q).
So far, so easy. But now that we’ve got going, we can translate ever more
complicated sentences. For just one more example, consider the rather laboured
claim
(9) Either Jack and Jill love each other or it isn’t the case that either Jack
loves Jill or Jill loves Jack.
This is naturally read as the disjunction of (4) and (8); so we can render it into
our PL language by disjoining the translations of (4) and (8), thus:
(90 ) ((P ∧ Q) ∨ ¬(P ∨ Q)).

8.9 Summary
Our aim is to consider arguments whose logical structure depends on the
presence of ‘and’, ‘or’, and ‘not’ in the premisses and conclusion.
There are many quirks and ambiguities in the way ordinary-language ‘and’,
‘or’, and ‘not’ behave (even when we just consider their uses as sentential connectives). To avoid the vagaries of the vernacular, we are going to

70

Exercises 8
use special-purpose languages, PL languages, to express arguments without
ambiguities or obscurities.
A PL language has a base class of ‘atomic’ symbols which can be used to
express whole messages, and which might as well be as simple as possible,
e.g. single letters. More complex, ‘molecular’, symbolic formulas are then
built up using ‘∧’, ‘∨’, and ‘¬’ (unambiguously expressing bare conjunction,
inclusive disjunction, and negation), bracketing carefully.

Exercises 8
(a) Usually, we can ambiguously negate a statement by prefixing it with ‘It is not
the case that’. Can you think of any exceptions (in addition to the kinds described in
§8.4)?
(b)

Give negations of the following in natural English:

(1)
(2)
(3)
(4)
(5)
(6)
(7)
(8)
(9)
(10)
(11)

It is not the case that both Jack and Jill went up the hill.
Neither Jack nor Jill went up the hill.
No one loves Jack.
Only tall men love Jill.
Everyone who loves Jack admires Jill.
Someone loves both Jack and Jill.
Some who love Jill are not themselves loveable.
Jill always arrives on time.
Whoever did that ought to pay for the damage.
Whenever it rains, it pours.
No one may smoke.

(c) Two propositions are contraries if they cannot be true together; they are contradictories if one is true exactly when the other is false. (Example: ‘All philosophers
are wise’ and ‘No philosophers are wise’ are contraries – they can’t both be true. But
maybe they are both false, so they are not contradictories.) Give examples of propositions which are contraries but not contradictories of the propositions in (b).
(d)
(1)
(2)
(3)
(4)
(5)
(6)
(7)
(8)

Render the following as best you can into the PL language we introduced in §8.8:
Jack is unwise and loves Jill.
Jack and Jo both love Jill.
It isn’t true that Jack doesn’t love Jill.
Jack loves Jill but Jo doesn’t.
Jack doesn’t love Jill, neither is he wise.
Either Jack loves Jill or Jill loves Jack.
Either Jack loves Jill or Jill loves Jack, but not both.
Either Jack is unwise or he loves Jill and Jo loves Jill.

71

9 PL syntax
In the previous chapter, we explained why we will be using artificial PL languages
to regiment arguments involving connectives for conjunction, disjunction, and
negation. This chapter now pins down the grammar or syntax of such languages.

9.1 Syntactic rules for PL languages
The syntax of a language tells us, which strings of symbols count as grammatical
sentences. In our case, introducing some standard terminology,
The syntax of a PL language defines what counts as a well-formed formula
or, for brevity, wff of that language.
We will explore PL syntax in three stages: (a) we fix the alphabet of symbols we
can use in a PL language; (b) we stipulate what count as the ‘atomic’ wffs of a
given language; finally, (c) we explain how other, ‘molecular’, wffs are built up
from the atoms. (I pronounce ‘wff’ either as woof or simply as formula.)
(a) First, then, we need to specify the alphabet of PL languages. We have
already met the propositional letters ‘P’, ‘Q’, ‘R’, ‘S’, the three connectives ‘∧’,
‘∨’, ‘¬’, plus brackets ‘(’, ‘)’. But we now add a few more symbols.
(1) In some cases, we will need more than a mere four basic atomic wffs.
We will use a prime ‘0 ’ for making as many additional atoms as we
want. We can then form the likes of these: P0 , P00 , Q0 , R00 , S000 , . . . .
(2) We want to be able to formalize not just individual propositions but
whole arguments. So we will want to be able to list premisses, separated
by punctuation. Let’s provide a comma to do the job.
(3) We will also want an inference marker to signal when a conclusion is
being drawn after a list of premisses. We can adopt ‘6’, the familiar
therefore sign.
So, in summary, we stipulate that:
The alphabet of a PL language is: P , Q , R , S ,0 , ∧, ∨, ¬ , ( , ) , 6 , plus the
comma.

72

Syntactic rules for PL languages
Or at least, that’s our initial alphabet. We will add two more symbols in due
course (in particular, if you have met propositional logic before, you’ll notice
that ‘→’ is missing so far: but there is a reason for delaying its introduction).
(b) Next, each PL language must have a stock of atomic wffs formed from this
alphabet, atoms which provide the basic building blocks for the language.
How many atoms we need in a particular language will depend on the complexity of the argument(s) we want to use it to formalize. Perhaps we need, say,
ten atoms. (We can safely assume in this book that we will never need more than
a finite number of atoms!) Now, at the end of §8.7, we said that ‘P’, ‘Q’, ‘R’, ‘S’
can serve as atoms; but we left it open how we are to continue that list. But we
don’t want any ambiguity about what counts as an atomic wff of a particular
language. So how do we get a determinate ten atoms? What if we want thirty
atoms (more than there are letters in the alphabet)? As we’ve just said, we will
use primes to form as many atoms as we want. So:
A PL language has a (finite) supply of one or more atomic wffs. We will
assume that each is formed from one of the four basic propositional letters
followed by zero or more primes. We specify the atoms of a particular language simply by listing them. For example, if we want exactly ten atoms,
we can choose these: P , Q , R , S , P0 , Q0 , R0 , S0 , P00 , Q00 .
(c) Having fixed the finite class of atomic wffs for a language, we now need
to give the rules for building up more complex wffs out of simpler ones using
connectives, to arrive at the language’s full class of wffs, atomic and molecular.
While the atomic wffs can vary between PL languages, the rules for constructing more complex wffs from them are constant. We outlined these rules in §8.7.
Recalling our insistence that the connectives ‘∧’ and ‘∨’ always come with a pair
of brackets to indicate their scope, we can sum things up like this:
The wffs of a particular PL language are determined as follows. Having
explicitly specified its atomic wffs, then
(W1) Any atomic wff of the language counts as a wff.
(W2) If α and β are wffs, so is (α ∧ β).
(W3) If α and β are wffs, so is (α ∨ β).
(W4) If α is a wff, so is ¬α.
(W5) Nothing else is a wff.
It should be immediately clear how to understand these rules. But, at the risk
of labouring the obvious, let’s spell out two points.
First, W2 is to be read as telling us that the result of writing a left-hand
bracket ‘(’ followed by the wff α followed by a conjunction sign ‘∧’ followed by
the wff β followed by a right-hand bracket ‘)’ is also a wff. And naturally we will
say that a wff constructed like this has the form (α ∧ β). Similarly, of course, for
W3 and W4.

73

9 PL syntax
Second, note that the rules W1 to W4, taken by themselves, don’t stop Julius
Caesar from also counting as a wff. That’s why we need to add the extremal
clause W5 to delimit the class of wffs and thereby block Caesar along with
other intruders. In W5, ‘nothing else’ means that the only wffs are the strings
of symbols that can be proved to be wffs using the preceding rules W1 to W4.
(d) For the record, in the context of discussing wffs of PL (and similar) languages, when we talk about the negation of a wff α we will from now on always
mean the wff ¬α, rather than some possible equivalent (compare §8.4, where we
talked of a negation). Similarly when we talk about the conjunction or disjunction of the wffs α and β we will mean the corresponding wffs (α ∧ β) or (α ∨ β)
rather than any equivalents.

9.2 Construction histories, parse trees
(a) Assume we are dealing with a PL language whose atoms are ‘P’, ‘Q’, ‘R’,
‘S’. Let’s use our stated W-rules to prove that the sequence of symbols
¬((P ∨ Q) ∧ ¬(¬Q ∨ R))
is a wff of this language. We can laboriously lay out an annotated derivation in
the style of §4.1. Premisses are supplied by our assumption about atoms together
with W1; and then at further steps we appeal to the other W-rules:
A

(1) ‘P’ is a wff.
(2) ‘Q’ is a wff.
(3) ‘R’ is a wff.
(4) ‘(P ∨ Q)’ is a wff.
(5) ‘¬Q’ is a wff.
(6) ‘(¬Q ∨ R)’ is a wff.
(7) ‘¬(¬Q ∨ R)’ is a wff.
(8) ‘((P ∨ Q) ∧ ¬(¬Q ∨ R))’ is a wff.
(9) ‘¬((P ∨ Q) ∧ ¬(¬Q ∨ R))’ is a wff.

(premiss)
(premiss)
(premiss)
(from 1, 2 by W3)
(from 2 by W4)
(from 5, 3 by W3)
(from 6 by W4)
(from 4, 7 by W2)
(from 8 by W4)

This reveals what we might call a construction history for our wff. However,
the vertical presentation of the proof here is perhaps not the most illuminating
way of setting things out. We can rather more perspicuously re-arrange the
derivation like this:
AT

‘Q’ is a wff
‘¬Q’ is a wff
‘R’ is a wff
‘(¬Q
∨
R)’
is
a wff
‘P’ is a wff
‘Q’ is a wff
‘(P ∨ Q)’ is a wff
‘¬(¬Q ∨ R)’ is a wff
‘((P ∨ Q) ∧ ¬(¬Q ∨ R))’ is a wff
‘¬((P ∨ Q) ∧ ¬(¬Q ∨ R))’ is a wff

This is essentially the same proof as A, but now presented in the form of a tree.
At the tips of the branches we have our premisses telling us that various atoms

74

Construction histories, parse trees
are wffs (the premiss that ‘Q’ is a wff gets repeated on different branches because
it is used twice). The horizontal lines then mark inferences. So as we move down
the tree AT, we derive further facts about wffs in accordance with the rule W4
(which keeps us on the same branch), or in accordance with the rules W2 and
W3 (when we join two branches). We impose the natural convention for building
this construction history in tree form: when we apply the joining rules, the wffs
immediately above the horizontal line appear in the same left-to-right order as
they appear inside the wff below the line.
(b) Here is another way of presenting exactly the same information as is given
by AT. First, turn everything upside down: that obviously doesn’t make a significant difference! Second, reduce clutter by omitting all those occurrences of
the repeated frame ‘. . . ’ is a wff (since the same frame surrounds every wff, it
can safely be left as understood). Then we get
¬((P ∨ Q) ∧ ¬(¬Q ∨ R))
AP
((P ∨ Q) ∧ ¬(¬Q ∨ R))
(P ∨ Q)
P

¬(¬Q ∨ R)
(¬Q ∨ R)

Q
¬Q

R

Q
Or rather this is what we get when – following convention – we change the
decorative style a bit, and replace each horizontal line with a short vertical line
(when going from one wff to another) or with a joined pair of sloping lines (when
going from one wff to two).
Reading upwards, we can still think of this as a construction tree, which
now more economically displays how our wff is constructed stage by stage in
accordance with our W-rules. Reading downwards, we have what linguists might
call a parse tree. For, read this way, our tree displays how the wff at the top can
be parsed, i.e. how it can be disassembled into its component wffs or subformulas
stage by stage until we reach its ultimate atomic components.
(c)

Let’s take another example. We will show that
(((S ∧ Q) ∧ ¬¬R) ∨ ¬(¬(P ∨ P) ∧ Q))

is also a wff of our current PL language. Here is a construction history in a neat
tree version, BT:
‘P’ is a wff ‘P’ is a wff
‘(P ∨ P)’ is a wff
‘R’ is a wff
‘¬(P
∨ P)’ is a wff
‘Q’ is a wff
‘S’ is a wff ‘Q’ is a wff ‘¬R’ is a wff
‘(S ∧ Q)’ is a wff
‘¬¬R’ is a wff
‘(¬(P ∨ P) ∧ Q)’ is a wff
‘((S ∧ Q) ∧ ¬¬R)’ is a wff
‘¬(¬(P ∨ P) ∧ Q)’ is a wff
‘(((S ∧ Q) ∧ ¬¬R) ∨ ¬(¬(P ∨ P) ∧ Q))’ is a wff

75

9 PL syntax
Pause to check which W-rule is being applied at each inference step. And note
that there is nothing in the rule W3 for constructing a disjunction of the form
(α ∨ β) which says that the schematic variables here have to be substituted for
by different expressions. So, in particular, given that ‘P’ is a wff, we can use W3
to infer that ‘(P ∨ P)’ is a wff.
Inverting and omitting the clutter of those repeated frames ‘. . . ’ is a wff
appearing in the construction history BT, we get the following parse tree, BP:
(((S ∧ Q) ∧ ¬¬R) ∨ ¬(¬(P ∨ P) ∧ Q))
((S ∧ Q) ∧ ¬¬R)
(S ∧ Q)

(¬(P ∨ P) ∧ Q)

¬¬R
Q

S

¬(¬(P ∨ P) ∧ Q)

¬R

¬(P ∨ P)

R

(P ∨ P)
P

Q

P

Again, we can read this upwards, as showing us how to construct the wff at the
top by building it up stage by stage from its atoms. Or we can read the same tree
downwards, as showing us how to parse the top wff by disassembling it stage by
stage into ever-shorter subformulas. (The idea of a parse tree is simple; but it is
worth pausing to try some of this chapter’s Exercises to test understanding.)

9.3 Wffs have unique parse trees!
(a) By rule W5, the wffs of a PL language are exactly those strings of symbols
that can be assembled from atomic wffs by using the three building rules W2–
W4. And so we must be able to disassemble wffs back into their component
atoms. Hence each wff must have at least one construction history/parse tree
showing how it can be built up from or dissembled back into atomic wffs.
Now, suppose our wff-building rules W2 and W3 had not insisted on conjunctions and disjunctions being delimited by brackets. What would have happened?
We would get expressions which have more than one parse tree, as in
P∨Q∧R
P∨Q
P

P∨Q∧R
R

Q

Q∧R

P
Q

R

This would give us a structurally ambiguous expression on a par with our
ordinary-language example from §8.5(b), ‘Either Jack took Jill to the party or
he took Jo and he had some fun’. However, our formal bracketing rules are in
place precisely to prevent such structural scope ambiguities. For example, here is
a parse tree for one properly bracketed version of that last expression:

76

Main connectives, subformulas, scope
((P ∨ Q) ∧ R)
(P ∨ Q)

R

Q
P
Read upwards, this tree shows that ‘∨’ here serves first to form ‘(P ∨ Q)’ (so this
subformula is therefore the connective’s scope in the intuitive sense which we
sharpen in the next section); ‘∧’ then forms the whole wff (so the whole wff is
the scope of that connective). And, having constructed the wff ‘((P ∨ Q) ∧ R)’,
now note there is only one way of disassembling it again – there is only one
possible parse tree for the wff.
(b) The point generalizes. Fix the convention that the wffs immediately below a
fork on a parse tree occur in the same left-to-right order as they appear inside the
wff just above the fork. Then, crucially, our bracketing rules ensure that
Every wff of a PL language has a unique parse tree.
Experimentation with a few examples will quickly convince you that this claim is
true – and such experimentation will probably be much more enlightening than
an official proof. But, for the sceptical, we do outline a proof in the Exercises.
(c) A quick observation. As we walk up a parse tree, brackets always get introduced into a wff in left-right pairs. So note that every genuine wff must be
balanced, i.e. have the same number of left-hand and right-hand brackets.

9.4 Main connectives, subformulas, scope
(a)

The following should now seem an entirely natural definition:

The main connective of a molecular wff is the first connective to be removed
as we go down the wff’s parse tree.
Equivalently, the main connective of a molecular wff is the last connective introduced in a construction history for the wff. Then:
If a wff has the form ¬α, then its main connective is that initial ‘¬’.
If a wff has the form (α ∧ β) for wffs α, β, then its main connective is that
‘∧’. We define (α ∧ β)’s conjuncts to be α and β.
Similarly, if a wff has the form (α ∨ β), then its main connective is that ‘∨’.
Its disjuncts are again α and β.
A molecular PL wff can only have one main connective. In particular a wff can’t
both be of the form (α ◦ β) for wffs α, β and ◦ a binary connective, and also of
the form (γ  δ) for wffs γ, δ and  a binary connective – unless α = γ, ◦ = ,
and β = δ. For if a wff did have distinct forms (α ◦ β) and (γ  δ), it would have
two distinct parse trees – and we have said that that’s impossible.

77

9 PL syntax
(b) The main connective of a wff determines its key logical properties – which
is why it matters. For example, given the intended meanings of ‘∨’ and ‘¬’, any
instance of the following pattern of inference is valid (compare §8.1):
(α ∨ β), ¬α 6 β.
But to apply this principle to warrant a particular argument in some PL language, we need to know whether the longer premiss in the argument really is a
disjunction, i.e. really has the form (α ∨ β) with ‘∨’ as its main connective.
(c)

Here next is the official definition for a notion we’ve already used:

A wff β is a subformula of a wff α just if β appears on the parse tree for α.
This is provably equivalent to saying that a subformula is a string of symbols
occurring in a wff which could equally stand alone as a wff in its own right. Note,
our definition allows a wff α trivially to count as a subformula of itself. But that
is a convenient convention (if you know about sets, compare how it is convenient
to allow a set X to trivially count as one of its own subsets).
(d) Our final definition in this section sharpens another intuitive idea – the
idea of the scope of a connective, which we first met in §8.5. Roughly speaking,
the scope of a given occurrence of some connective in a wff is the part of the wff
which that connective governs. So take another parse tree, CP :
((S ∧ ¬Q) ∨ ¬(P ∧ (R ∨ S)))
(S ∧ ¬Q)
S

¬(P ∧ (R ∨ S))
¬Q

(P ∧ (R ∨ S))

Q

P

(R ∨ S)

R
S
Working up the tree reveals how each particular occurrence of a connective gets
into the final wff. Thus the first conjunction ‘∧’ is introduced to form ‘(S ∧ ¬Q)’.
So we can rather naturally say that this subformula is the scope of the connective
– and then, as we intuitively want, ‘S’ and ‘¬Q’ and ‘Q’ too are in the scope of
that conjunction. Similarly, the second negation is first introduced on the tree in
the subformula ‘¬(P ∧ (R ∨ S))’: so we can say that that subformula is the scope
of this second occurrence of ‘¬’. And so on:
The scope of an occurrence of a connective in a wff is the subformula where
that occurrence first gets introduced, reading up the wff’s parse tree.
An expression is in the scope of a particular connective in the intuitive sense
if it is literally in, is part of, that connective’s scope as we have just defined it (a
connective counts as being in its own scope by our definition, but again that’s
harmless). This implies, in particular, that

78

Bracketing styles
An occurrence of a connective is in the scope of some other occurrence of
a connective in a wff if the first is introduced before the second as we go
up the relevant branch of the parse tree for the wff (i.e. go up the branch
which contains those two occurrences).
So the first negation in our example is in the scope of the first conjunction; the
second conjunction is in the scope of the second negation, and so on.
(e) Note, we have just slipped into referring to an occurrence of the conjunctive
connective as simply a ‘conjunction’ (and similarly for the other connectives).
This is handy shorthand; it should always be clear from context when we are
using e.g. ‘a conjunction’ to mean a wff formed using ‘∧’, and when we mean
an occurrence of that connective.

9.5 Bracketing styles
Finally, a quick suggestion about how we can occasionally make long wffs rather
more readable: use some way of marking pairs of left and right brackets which
belong together, to reveal more clearly how the wff is constructed.
In this spirit, we will occasionally aid the eye by making use of the different
available styles of brackets. Take, for example, the following mildly unreadable
string of symbols:
(¬((P ∨ (R ∧ ¬S)) ∨ ¬(Q ∧ ¬P)) ∧ ¬(P ∨ ¬(¬Q ∨ R))).
It is perhaps not immediately evident that this is a wff (in a language with
the right atoms, of course). But suppose we bend some of the matching pairs
of round brackets into curly ones ‘{’, ‘}’ and straighten other pairs into square
ones ‘[’, ‘]’ to get
 


¬ (P ∨ (R ∧ ¬S)) ∨ ¬(Q ∧ ¬P) ∧ ¬ P ∨ ¬(¬Q ∨ R) .
The string now looks rather more obviously a wff. Give a parse tree for it!

9.6 Summary
The syntax of a PL language, after fixing the basic alphabet, first defines
the class of atomic wffs, and then gives rules for constructing legitimate
molecular wffs out of simpler ones using the three connectives ‘∧’, ‘∨’, and
‘¬’, bracketing as we go.
Each wff has a unique construction/parse tree displaying how it is constructed ultimately from atomic wffs and equally how it can be disassembled
back down into atomic wffs.
A subformula of a wff α is a wff that appears somewhere on α’s parse tree.
The main connective of a (non-atomic) wff is the first connective to be
removed as we go down the wff’s parse tree.

79

9 PL syntax
The scope of a particular occurrence of a connective in a wff is the subformula where it is introduced going up the wff’s parse tree (this will be the
shortest subformula including that occurrence).

Exercises 9
(a) Show the following expressions are wffs of a PL language with suitable atoms by
producing parse trees. Which is the main connective of each wff? What is the scope of
each connective in (3) and of each disjunction in (4)? List all the subformulas of (5).
Use alternative styles of brackets in (5) and (6) to make them more easily readable.
(1)
(2)
(3)
(4)
(5)
(6)

((P ∧ P) ∨ R)
(¬(R ∧ S) ∨ ¬Q)
¬¬((P ∧ Q) ∨ (¬P ∨ ¬Q))
((((P ∨ P) ∧ R) ∧ Q) ∨ (¬(R ∧ S) ∨ ¬Q))
(¬(¬(P ∧ Q) ∧ ¬(P ∧ R)) ∨ ¬(P ∧ (Q ∨ R)))
(¬((R ∨ ¬Q) ∧ ¬S) ∧ (¬(¬P ∧ Q) ∧ S))

(b) Which of the following expressions are wffs of a PL language with the relevant
atoms? Repair the defective expressions by adding/removing the minimum number of
brackets needed to do the job. Show the results are now wffs by producing parse trees.
(1)
(2)
(3)
(4)
(5)
(6)
(7)
(c*)

((P ∨ Q) ∧ ¬R))
((P ∨ (Q ∧ ¬R) ∨ ((Q ∧ ¬R) ∨ P))
¬(¬P ∨ (Q ∧ (R ∨ ¬P))
(P ∧ (Q ∨ R) ∧ (Q ∨ R))
(((P ∧ (Q ∧ ¬R)) ∨ ¬¬¬(R ∧ Q)) ∨ (P ∧ R))
((P ∧ (Q ∧ ¬R)) ∨ ¬¬¬((R ∧ Q) ∨ (P ∧ R)))
(¬(P ∨ ¬(Q ∧ R)) ∨ (P ∧ Q ∧ R)
Show that parse trees for wffs are unique, in the following stages.

(1) Suppose that a wff has the form (α∧β) or (α∨β). Then show that the relevant
occurrence of the connective ‘∧’ or ‘∨’ is preceded by exactly one more lefthand bracket than right-hand bracket. And show that any other occurrence
of a binary connective in the wff is preceded by at least two more left-hand
brackets than right-hand brackets.
(2) You know that if a wff starts with a negation, it must have the form ¬α. And
if it starts with a left bracket, you now have a method of parsing it uniquely
as having either the form (α ∧ β) or (α ∨ β): what method?
(3) Now develop this into a method of disassembling a complex wff stage by stage,
building a parse tree as you go. Confirm that there are no choice points in
this process, and if we start with a wff, the result is the only possible one,
and therefore parse trees are unique.
(4) As a bonus result, show how the same basic method applied to any string of
symbols can be used to decide whether it is a wff or not (because either the
process will freeze, or will generate a parse tree).

80

10 PL semantics
Having described the syntax of PL languages, we now turn to consider their
semantics; in other words, we discuss questions of meaning and of truth. In
particular, we introduce the crucial notion of a truth-value valuation.

10.1 Interpreting wffs
(a) Recall from §8.6 our ‘divide-and-rule’ plan for tackling arguments involving
connectives. We first render a vernacular argument into a well-behaved artificial
PL language. Then we investigate the validity or otherwise of the reformulated
argument. But if the reformulated version is still to be a genuine argument whose
premisses and conclusion are contentful, the wffs of the relevant PL language
cannot be left as mere uninterpreted symbols. They must be interpreted, i.e.
must be assigned meanings.
A little more carefully – since, as we remarked in §7.2, questions of colouring
or tone don’t matter for logic – when interpreting a formal PL language, what we
need to do is assign sufficiently clear and unambiguous truth-relevant senses to
its wffs. We do this by assigning senses to the atomic wffs – these atoms form the
basic non-logical vocabulary of a given language. We have already indicated the
intended meanings of connectives, which form the topic-neutral logical vocabulary
which stays constant across PL languages. We can then work out the senses of
arbitrarily complex molecular wffs.
(b)

So, first step:

We specify the senses of a PL language’s atoms by a glossary that fixes the
atoms’ truth-conditions.
We might compare this with one of those lists at the end of a tourist guide,
giving the local equivalents of various English phrases (except that the guidebook does aim to get tone approximately right, whereas here we really care only
about truth-relevant sense).
For a mini-example, take the three-atom language with the glossary
P : Alun loves Bethan.
Q : Bethan is Welsh.
R : Alun is married to Bethan.

81

10 PL semantics
Note, atoms need not have very simple readings like these: the messages that
they are stipulated to convey can be as complex as you like. However, they do
need to have truth-conditions which are determinate enough, in a sense to be
explained shortly.
(c) We have already explained in §8.7 the meanings of the three connectives
which are common to any PL language. To repeat:
(1) ‘∧’ invariably forms a conjunction, no more and no less; it translates as ‘and’ (or ‘and also’ or ‘both . . . and . . . ’), so long as it is
clear – perhaps from contextual clues – that the English rendition
should be read as forming a bare conjunction (and not as meaning
e.g. ‘and then’).
(2) ‘∨’ invariably forms an inclusive disjunction, no more and no less;
it translates as ‘either . . . or . . . or both’, or ‘and/or’ – or indeed as plain ‘or’, so long as context again makes it clear that the
disjunction is to be understood as inclusive.
(3) ‘¬’ invariably negates the whole wff that follows; it can be translated by ‘It is not the case that . . . ’, or often by a judiciously
inserted ‘not’.
So, for trivial examples, we will have
(R ∨ ¬Q) :
(Q ∧ ¬P) :
((R ∧ P) ∧ ¬Q) :
¬(¬P ∧ R) :
¬¬Q :

Either Alun is married to Bethan or she isn’t Welsh.
Bethan is Welsh and Alun doesn’t love her.
Alun is married to Bethan and he loves her, and she
isn’t Welsh.
It’s not the case that Alun both doesn’t love Bethan
and yet also is married to her.
It isn’t the case that Bethan is not Welsh.

Note the last case. In relaxed ordinary language, multiple negatives can remain
emphatically negative (‘We don’t need no education’). But in classical logic
double ‘¬’s always cancel each other out: hence ‘¬¬Q’ tells us that Bethan is
Welsh.
(d) There’s more to do in interpreting molecular wffs than simply plugging
in the interpretations of the atoms, translating the connectives, and tidying
the English; we also need to respect the relative scopes of the connectives as
determined by the compulsory bracketing in a PL wff. For example, consider
a language with the same three atoms ‘P’, ‘Q’, ‘R’, but where these are now
interpreted as follows:
P: Jack took Jill to the party.
Q: Jack took Jo to the party.
R: Jack had some fun.
Note then that it won’t do to interpret

82

Languages and translation
(P ∨ (Q ∧ R))
as, for example,
Either Jack took Jill to the party or he took Jo and he had some fun.
For that English version has exactly the kind of potential ambiguity which the
careful bracketing in the PL language is there to eliminate – see §8.7(b). A
rendition that better expresses the correct interpretation of the wff might, for
example, be
Either Jack took Jill to the party, or he took Jo and had some fun, or
both.

10.2 Languages and translation
(a) In the last section, we described two alternative dictionaries or glossaries
for the same collection of atoms ‘P’, ‘Q’, ‘R’. Do these define, as it were, two
dialects of the same PL language? Or shall we say that we have characterized
two different languages?
Some writers in effect define a formalized language by its syntax alone, and so
they say that the same language can have many interpretations. But we’ll prefer
to take a language to be defined by its syntax and semantics: hence, to our way
of thinking, different dictionaries will give us different formal languages (which
perhaps coheres better with how we ordinarily think of languages).
So, on our approach, arguments on different topics will get translated into
different PL languages – usually these will be languages set up ad hoc, for the
special purposes at hand. But, as we will see, arguments in different formalized
languages can still have the same form: that’s how the desired generality gets
into our story.
(b) Given the glossary for the atoms of a PL language, translation to and from
this language shouldn’t be too problematic. We won’t pause for more examples
now, as there are plenty of translation Exercises at the end of this and later
chapters. However, we should perhaps say a word straight away about the notion
of ‘translation’ here.
Some prefer to talk instead of ‘paraphrasing’ or ‘rendering’ or ‘transcribing’
ordinary language into PL. But, however described, what matters is that we aim
for a wff of the relevant formalized language which captures, as closely as possible
in that language, the sense, i.e. the truth-relevant meaning, of the English claim
being translated. Tone and colour can be ignored.
Others prefer to talk of ‘symbolizing’ the English. But this way of putting it
could mislead. For ‘symbolizing’ might suggest a mere exercise of introducing
symbolic shorthand for bits of English; and such renditions would have all
the ambiguities and obscurities of their originals. PL languages are not mere
shorthand code for bits of English; their whole point is to be usefully tidied-up
replacements for the messy vernacular.

83

10 PL semantics

10.3 Atomic wffs are true or false
It is up to us what the atom ‘P’ means in a particular PL language. To take the
last mini-language we considered, we stipulated that ‘P’ there says that Jack
took Jill to the party. But having fixed the meaning by settling which Jack and
Jill are in question, etc., and therefore fixed the conditions under which ‘P’ is
true, it is then up to how things are in the world whether ‘P’ is true (or if you
prefer, whether what ‘P’ expresses is true). If Jack did take Jill to the party,
then ‘P’ is true; if not, ‘P’ is false.
And we’ll now make the idealizing and simplifying assumption that the world
will always settle the matter, one way or the other. More generally, we will
assume:
Determinacy For any PL language, once we have stipulated the senses
of its atoms – fixed their truth-conditions – each of these atoms will be
determinately true or determinately false in any possible situation.
Now, it seems that there are various ways this idealizing assumption could go
wrong and an interpreted atomic wff could fail to be either true or false. For
example:
(1) Perhaps our glossary entry for some atom ‘P’ is something like Jane is
a philosopher, but given in a context which somehow fails to pin down
which Jane we are referring to. Or the entry is ‘It is raining’, and it
isn’t clear where or when we are talking about. As we might put it,
in such cases, ‘P’ is not yet fully interpreted. In other words, the wff
doesn’t yet express a determinate truth-relevant content – there is, so
to speak, part of the message still waiting to be filled in by context –
and hence ‘P’ doesn’t yet get to the starting line for being assessable
for truth or falsehood.
(2) Perhaps, as in an example above, ‘P’ means that Alun loves Bethan.
Suppose Alun’s feelings are mixed, on the border of love. So maybe it
is neither definitely true that he loves Bethan nor on the other hand
definitely false.
(3) Perhaps ‘P’ is interpreted as one of those paradoxical ‘Liar’ sentences
which says of itself that it is false. So if ‘P’ is false, it is true; and if it
is true it is false. So again, it must be wrong to say that it is true and
wrong to say that it is false (assuming it can’t be both).
And perhaps there are other possibilities too where we will want to allow for
claims that are neither true nor false.
But we will keep things simple. Our crucial Determinacy assumption is that
such issues don’t arise for atoms of our PL languages. In particular, we can
pretend that the atoms have fully determinate, context-independent content,
so as to avoid problem (1). We suppose too that we can ignore (2) issues of
vagueness (perhaps we imagine any vague boundaries to be artificially sharpened
when we render ordinary-language propositions into our formalized language).

84

Truth values
We will also assume that we will not get entangled with (3) the likes of the Liar
Paradox.
Certainly, there are deep problems about context-dependence, about vagueness, and about liar-style paradoxes. Many whole books have been written on
each of these topics. However, just as we have to start our study of dynamics by
ignoring friction and air-resistance, we have to start our study of logic by simply
sidestepping all these problems, leaving them for another day.
So, to repeat: we henceforth assume that the basic building blocks of our PL
languages can for our purposes be taken to have complete, sharp, and unambiguous senses, and that they are determinately true or false in every possible
situation.
(Usually, whether a wff is true will depend on the situation; but not always.
An atom could express a necessary truth or necessary falsehood and so come out
true or come out false independently of the situation in the world – as would be
the case if, for example, we stipulated that ‘P’ means Anyone who is a mother
is a parent or ‘Q’ means Anyone who is a bachelor is married, making them
respectively true and false, just by definition.)

10.4 Truth values
Another bit of terminology. It is absolutely standard to talk of truth values (a
notion first explicitly introduced into logic by Frege). To explain:
If a wff (or indeed a sentence of an ordinary language) is true in a given
situation, we say it then has the truth value true; and if it is false, it then
has the truth value false. For brevity, we denote these truth values by simply
‘T’ and ‘F’.
We call an assignment of truth values to some wffs of a PL language a
valuation of those wffs.
But what are truth values? Well, we don’t really have to suppose there are such
things – we could take our talk of them simply to be a colourful idiom. However
it makes life a bit easier if we construe talk of values literally and identify truth
values with suitable objects. A standard choice is to identify the truth value T
with the number 1, and the truth value F with the number 0. We are then, in
effect, thinking of these truth-values-as-numbers as codes for the truth-status of
sentences or wffs.
With this new way of talking in play, our crucial assumption in the last section
therefore can be rephrased like this:
Determinacy The atoms of a PL language are to be interpreted so that
they have determinate truth values, exactly one of T and F, in any possible
situation.

85

10 PL semantics

10.5 Truth tables for the connectives
It is useful to have some brisk notation – added to logicians’ English – to talk
about the assignment of a truth value to a wff. So we will write
P := T, Q := F, (P ∧ Q) := F, ((P ∧ ¬Q) ∨ (¬P ∧ Q)) := T, . . .
to say that the value of ‘P’ is T, the value of ‘Q’ is F, the value of ‘(P ∧ Q)’ is
F, etc. (The symbol ‘:=’ suggests itself because it is used in some programming
languages for the assignment of values.)
Now, in §8.2, we characterized a conjunction of two claims A and B as being
true when both of A and B are true, and false when at least one of A and B
is false. Then in §8.7 we stipulated that the connective ‘∧’ always forms a bare
conjunction in a PL language. Hence the following four conditions apply to any
PL-wffs α, β:
If α := T and β := T, then (α ∧ β) := T.
If α := T and β := F, then (α ∧ β) := F.
If α := F and β := T, then (α ∧ β) := F.
If α := F and β := F, then (α ∧ β) := F.
Since we are assuming that PL wffs always have one or other truth value, one of
the four conditions on the left must apply. Whatever the situation, it will fix the
truth values of given wffs α and β; hence we can apply the relevant condition to
determine the truth value of their conjunction (α ∧ β).
Likewise, in §8.3 we defined an inclusive disjunction of two claims A and B
to be true when at least one of A and B is true, and to be false when both A
and B are false. But ‘∨’ always forms an inclusive disjunction in a PL language.
Hence, for any wffs α, β,
If α := T and β := T, then (α ∨ β) := T.
If α := T and β := F, then (α ∨ β) := T.
If α := F and β := T, then (α ∨ β) := T.
If α := F and β := F, then (α ∨ β) := F.
And thirdly, ‘negation flips truth values’; that is to say, a negation of A is true
when A is false and is false when A is true. The prefixed connective ‘¬’ serves
as a negation operator in a PL language. So in our new notation, for any wff α:
If α := T, then ¬α := F.
If α := F, then ¬α := T.
Conventionally, we sum up these three lists of conditions in tabular form:
The truth tables for the PL connectives:
α
T
T
F
F

86

β
T
F
T
F

(α ∧ β)
T
F
F
F

α
T
T
F
F

β
T
F
T
F

(α ∨ β)
T
T
T
F

α
T
F

¬α
F
T

Evaluating molecular wffs: two examples
Just read across the lines of these truth tables in the natural way, and you will
recover the lists of conditions above. So these tables can be thought of as a vivid
way of giving the senses, i.e. the truth-relevant meanings, of the PL connectives.

10.6 Evaluating molecular wffs: two examples
(a)

Consider now the following wff of some PL language:
¬((P ∨ Q) ∧ ¬(¬Q ∨ R)).

Suppose that the interpretations of the atoms in this language and the facts of
the matter conspire to generate assignments of truth values to the three atoms
involved as follows:
P := T, Q := T, R := F.
Then we can easily work out, by a plodding calculation, that the whole wff is
false in this circumstance.
The key point to bear in mind when evaluating complex wffs is that we need
to do our working ‘from the inside out’, exactly as in arithmetic. Here is a stepby-step semantic evaluation presented as a derivation in rather gruesome detail:
AS

(1) P := T.
(2) Q := T.
(3) R := F.
(4) (P ∨ Q) := T,
(5) ¬Q := F.
(6) (¬Q ∨ R) := F.
(7) ¬(¬Q ∨ R) := T.
(8) ((P ∨ Q) ∧ ¬(¬Q ∨ R)) := T.
(9) ¬((P ∨ Q) ∧ ¬(¬Q ∨ R)) := F.

(premiss)
(premiss)
(premiss)
(from 1, 2 by the table for ‘∨’)
(from 2 by the table for ‘¬’)
(from 5, 2 by the table for ‘∨’)
(from 6 the table for ‘¬’)
(from 4, 7 by the table for ‘∧’)
(from 8 by the table for ‘¬’)

However, this linear version isn’t really the most illuminating way of presenting
the derivation. As in §9.2, we can rather more perspicuously re-arrange our
reasoning as a tree, like this:
AST

Q := T
¬Q := F
R := F
:=
(¬Q
∨
R)
F
:=
:=
P
T
Q
T
(P ∨ Q) := T
¬(¬Q ∨ R) := T
((P ∨ Q) ∧ ¬(¬Q ∨ R)) := T
¬((P ∨ Q) ∧ ¬(¬Q ∨ R)) := F

Simple vertical steps correspond to applications of the rule for evaluating a negation; while steps where two branches of the tree join correspond to applications
of the rule for evaluating a conjunction or disjunction.
But now note the exact parallelism between these semantic derivations AS
and AST and the old syntactic derivations A and AT in §9.2(a) which showed
that the same wff is well-formed. This is no accident!

87

10 PL semantics
We have designed PL languages so that the syntactic structure of a wff,
i.e. the way that the symbols are put together into grammatical strings, is
a perfect guide to its semantic structure, and in particular to the way the
truth value of the whole will depend on the truth values of the atoms.
Replacing the frames ‘. . . ’ is a wff in the construction history AT by the appropriate assignments of truth values gives us the semantic derivation AST .
As with construction histories, we can also write this tree the other way up,
like a parse tree. Then the values assigned to the atoms at the bottom of branches
determine the values of wffs immediately above them on the parse tree, which in
turn determine the values of wffs above them, all the way up. Looking at things
this way, we can say that semantics chases truth up the tree of grammar (to
adapt a neat phrase of the American philosopher W. V. O. Quine).
(b)

For a second example, take the following wff:
(((S ∧ Q) ∧ ¬¬R) ∨ ¬(¬(P ∨ P) ∧ Q)).

We showed that this is a wff by the tree proof BT in §9.2(c). And consider the
following valuation of its atoms:
P := F, Q := T, R := T, S := T.
Again, we can work out the value of the wff on the given valuation by assigning
truth values like this:
BST

P := F
P := F
:=
(P ∨ P)
F
R := T
:=
¬(P
∨
P)
T
Q := T
S := T
Q := T
¬R := F
(S ∧ Q) := T
¬¬R := T
(¬(P ∨ P) ∧ Q) := T
((S ∧ Q) ∧ ¬¬R) := T
¬(¬(P ∨ P) ∧ Q) := F
(((S ∧ Q) ∧ ¬¬R) ∨ (¬(P ∨ P) ∧ Q)) := T

(c) Note that we can do these two calculations of the truth values of complex
wffs in AST and BST once we know the initial valuations of their atoms, just
using the truth tables for the connectives. We don’t at all need to know what
their constituent atoms are supposed to mean. That’s because the truth values
of wffs in PL depend only on the truth values of their atomic parts, and not
on the interpretation of those parts. More on this hugely important point in
Chapter 12.

10.7 Uniqueness and bivalence
(a) Recall from §9.3 that our strict bracketing convention prevents us from
forming structurally ambiguous expressions like ‘P ∨ Q ∧ R’.
Imagine that we did allow such expressions, and suppose P := T, Q := T, R
:= F. Then consider the following pair of calculations of truth values, reflecting
the two ways of constructing the ambiguous expression:

88

Uniqueness and bivalence
P := T Q := T
P ∨ Q := T
R := F
P ∨ Q ∧ R := F

Q := T R := F
P := T
Q ∧ R := F
P ∨ Q ∧ R := T

Here the final valuations differ because, on the left, the final expression is read
with ‘∧’ having wider scope than ‘∨’, and on the right it is read with ‘∨’ having
wider scope. The syntactic structural ambiguity in this case leads to a semantic
ambiguity: in other words, fixing the truth values of the atoms does not uniquely
fix the truth value of the whole unbracketed expression. The bracketing rules
are in place precisely to ensure that this sort of thing can’t happen. (Our strict
bracketing policy is not the only one on the market: but any policy must similarly
prevent structural ambiguities.)
(b) Now to generalize. Suppose that we have a valuation of the atoms of a
given wff α (perhaps the valuation arising from the real-world truth values of
interpreted atoms, perhaps another possible valuation). We can then work out
the consequent unique truth value taken by α on that valuation.
Look at it this way (turning the likes of AST and BST upside down). We take
the unique parse tree for α. We feed in the truth values of whichever atoms we
find at the bottom of the branches. Then we go up the branches calculating the
values of the larger and larger subformulas of α we meet, at each stage applying
the truth tables for the connectives. We must then end up with a unique truth
value for the wff α.
And note, the fact that we evaluate α by chasing values up its parse tree
means that the values of atoms that don’t appear on its parse tree can’t affect
α’s value. In other words, unsurprisingly, the truth value of a wff depends only
on the assignment of values to the atoms it actually contains.
It follows, then, that we have:
A valuation of the atomic wffs in a PL wff uniquely determines a resulting
valuation of the wff.
A valuation of all the atomic wffs of a PL language uniquely determines a
resulting valuation of every wff of the language, however complex.
(c) Put that last observation together with our assumption that the atoms of a
PL language will in any possible situation have a definite valuation, and it follows
that every wff of a PL language is determinately either true or false whatever
the situation. Hence:
Our formalized PL languages are bivalent – i.e. are two-valued, with every
wff always taking one (though not both!) of the values T and F, in any
possible situation.
In studying the logic of arguments regimented in these languages, we can therefore be said to be studying classical two-valued logic – ‘classical’ not in the sense
of ancient but in the sense of basic and mainstream. (And, regrettably, we can
only touch on one non-classical logic in this book: see §24.8.)

89

10 PL semantics

10.8 Short working
(a) The last two sections have explained the basic principles of evaluating PL
wffs given a valuation of its atoms. In this section, we will introduce a way of
presenting truth-value calculations considerably more snappily.
In practice, of course, it isn’t necessary to go through all the palaver of chasing
truth values down construction histories/up parse trees. Simple calculations of
truth values, like simple arithmetic calculations, can very easily be done in our
heads. And more complex ones can be set out in a much more economical style,
in a mini-table, as we will now show.
So compare AS /AST with the following. We begin constructing our minitable by setting down the same wff as before on the right. And on the left we
display the same initial assignment of values to its atomic subformulas:
P
T

Q
T

R
F

¬((P

∨

Q)

∧

¬(¬ Q

∨

R))

Next, we copy the values of atoms across to the right, writing the appropriate
value under each atom again:
P
T

Q
T

R
F

¬((P
T

∨

1

Q)
T

∧

2

¬(¬ Q
T

∨

2

R))
F
3

(To aid comparison with the linear derivation in AS , we have started to number
the sequence of stages at which values are calculated by the corresponding line
number in that derivation.)
We then continue by calculating the values of more and more complex subformulas of the final formula: and at each step we write the resulting value under
the main connective of the subformula being evaluated – in other words, we write
the value under the connective whose impact is being calculated at each stage.
For example, in a few steps we get to,
P
T

Q
T

R
F

¬((P
T

∨
T

Q)
T

1

4

2

∧

¬(¬ Q
FT

∨
F

R))
F

52

6

3

And in a few more steps we arrive at:
P
T

Q
T

R
F

¬((P
F T

∨
T

Q)
T

∧
T

¬(¬ Q
T FT

∨
F

R))
F

9

4

2

8

7 52

6

3

1

For clarity, we have underlined the final value we get when we evaluate the
impact of the main connective.
Let’s take a second example, revisiting the wff in BST, with the valuation of
atoms as before. We lay out the working as a mini-table again. But this time,
to speed things up, we won’t bother to copy across the values of atoms from the
left-hand part of the table to the right-hand part – after all, how much trouble

90

Short working
is it to look up the values of atoms when we need them by glancing back to the
left? So we get:
P
F

Q
T

R
T

S
T

(((S ∧ Q) ∧ ¬¬R) ∨ ¬ (¬(P ∨ P) ∧ Q))
T
T TF T F T F
T
1

4

3 2

9

8

6

5

7

Here, the numbers suggest one possible order in which to work things out. So
check the steps in that order, at each stage evaluating the subformula whose
main connective is written above the step-number.
(b) This standard way of setting out working is neat and tidy. But we can
speed things up further by noting three elementary facts:
Once one of two conjuncts α and β has been shown to be false, it is redundant to evaluate the other conjunct: the whole conjunction (α ∧ β) must be
false.
Once one of two disjuncts α and β has been shown to be true, it is redundant
to evaluate the other disjunct: the whole disjunction (α ∨ β) must be true.
We can jump straight from the truth value of a wff α to the value of its
double negation ¬¬α, since these wffs always have the same value.
For instance, in the last example, we can skip step 2. And once we have completed
step 4 and evaluated the first disjunct ‘((S ∧ Q) ∧ ¬¬R)’ as true, we can skip the
next four steps and immediately conclude that the whole disjunction is true.
Here is another quick example, showing our suggested short cuts in operation.
We take the same wff again, but with a different assignment of values to its atoms:
P
T

Q
F

R
F

S
T

(((S ∧ Q) ∧ ¬¬R) ∨ ¬ (¬(P ∨ P) ∧ Q))
T T
F
3

2

1

Since the conjunct ‘Q’ is false, the conjunction ‘(¬(P ∨ P) ∧ Q)’ must be false
(here we use the first of our shortcuts). Therefore the conjunction’s negation
‘¬(¬(P ∨ P) ∧ Q)’ is true. But that wff is the second disjunct of the complete
disjunctive wff. Therefore the whole wff must be true too (that’s the second
shortcut at work). Just three steps of working, and we are done!
One more example. Take the rather longer wff that we met in §9.5 (we used
different bracketing styles to make it more readable). And suppose we want to
work out the truth value of this wff on the valuation of atoms stated on the left:
 


P Q R S
¬ (P ∨ (R ∧ ¬S)) ∨ ¬(Q ∧ ¬P) ∧ ¬ P ∨ ¬(¬Q ∨ R)
T T F T
F
T
T
F
3

1

2

4

Since the disjunct ‘P’
 is true, the disjunction ‘(P ∨ (R ∧ ¬S))’ is true. Hence the
longer disjunction ‘ (P ∨ (R ∧ ¬S)) ∨ ¬(Q ∧ ¬P) ’ must also be true. So the

91

10 PL semantics
negation of this curly-bracketed expression has to be false. But this false wff is
the first conjunct of the whole conjunctive wff. Therefore the whole wff is false
too. That’s all the working we need.
(c) The business of evaluating wffs on various assignments of values to their
atoms, and fully setting out all the working in our brisk tabular form, is entirely
mechanical and straightforward, even though potentially very tedious. As we
have just seen, however, we can often considerably reduce the tedium. It will
only take a little practice to learn to spot when we can use our shortcuts.

10.9 Summary
The atomic wffs – the basic non-logical vocabulary – of a PL language are
interpreted by a glossary which gives the truth-relevant senses of the atoms,
and the same atoms can get different interpretations in different languages.
However, the connectives ‘∧’, ‘∨’, and ‘¬’ – forming the logical vocabulary
constant across PL languages – always get the same meanings, respectively
expressing bare conjunction, inclusive disjunction, and strict negation.
Having fixed the truth-relevant content of an atomic wff, the situation in
the world will then determine whether this atom is true or false. Or at least,
it will do so assuming that we have in fact assigned a determinate meaning
to the atom, and are setting aside cases of context-dependence, vagueness,
paradoxical sentences, etc.
The truth or falsity of each of the wffs α and β will also determine the
truth or falsity of the conjunction (α ∧ β), the disjunction (α ∨ β), and the
negation ¬α. Simple truth tables display the ways that the truth values
of the molecular wffs of those forms depend on the values of the relevant
subformulas α and β.
Applying those tables, a valuation of the atoms in a wff of a PL language (i.e.
an assignment of the value true or false to each of its atoms) will determine
a valuation for the whole wff, however complex it is; and this valuation will
be unique because of the uniqueness of the wff’s construction/parse tree.
A calculation of the truth value of a wff given a valuation of its atoms can
be set out in various styles – e.g. as a vertical linear derivation, or by a tree
decorated with truth values. Most economically, we can set out the working
in a mini-table (the basic rule is to write the value of a subformula under
its main connective).

Exercises 10
(a) Suppose we are working in a PL language where ‘P’ means Fred is a fool ; ‘Q’ means
Fred knows some logic; ‘R’ means Fred is a rocket scientist. Translate the following
sentences into this formal language as best you can. What do you think is lost in the
translations, if you can only use the ‘colourless’ connectives ‘∧’, ‘∨’ and ‘¬’ ?

92

Exercises 10
(1)
(2)
(3)
(4)
(5)
(6)
(7)
(8)
(9)

Even Fred is a rocket scientist.
Fred is a rocket scientist, but he knows no logic.
Fred is a rocket scientist, moreover he knows some logic.
Fred’s a fool, even though he knows some logic.
Although Fred’s a rocket scientist, he’s a fool and even knows no logic.
Fred’s a fool, yet he’s a rocket scientist who knows some logic.
Fred is a fool despite the fact that he knows some logic.
Fred is not a rocket scientist who knows some logic.
Fred knows some logic unless he is a fool.

(b) Confirm that the following strings are wffs by producing parse trees. Suppose that
P := T, Q := F, R := T. Evaluate the wffs first by chasing values up the trees. Then do
the working again in the short form (i.e. as a mini-table, skipping redundant working
when you can).
(1)
(2)
(3)
(4)
(5)

((R ∨ ¬Q) ∧ (Q ∨ P))
¬(P ∨ ((Q ∧ ¬P) ∨ R))
¬(¬P ∨ ¬(Q ∧ ¬R))
(¬(P ∧ ¬Q) ∧ ¬¬R)
(((P ∨ ¬Q) ∧ (Q ∨ R)) ∨ ¬¬(Q ∨ ¬R))

Work out, in short form, the truth values of the following wffs on the assignment of
values P := F, Q := F, R := T, S := F
(6)
(7)
(8)
(9)
(10)

¬((P ∨ Q) ∧ ¬(¬Q ∨ R))
¬¬((P ∧ Q) ∨ (¬S ∨ ¬R))
(((S ∧ Q) ∧ ¬¬R) ∨ ¬Q)
(((P ∧ (Q ∧ ¬R)) ∨ ¬¬¬(R ∧ Q)) ∨ (P ∧ R))
(¬((P ∨ (R ∧ ¬S)) ∨ ¬(Q ∧ ¬P)) ∧ ¬(P ∨ ¬(¬Q ∨ R)))

(c*) In this book we have taken a maximalist line about the use of brackets in PL wffs.
What conventions for dropping brackets could we have adopted (while still writing ‘∧’
and ‘∨’ between the wffs they connect) in order to reduce the numbers of brackets in a
typical wff while not reintroducing semantic ambiguities?
(d*) Polish notation for the propositional calculus – introduced by Jan Lukasiewicz
in the 1920s – is a bracket-free notation in which connectives are written before the
wffs they connect.
Traditionally, for the Negation of α we write N α; for the Konjunction of α and β we
write Kαβ; for the disjunction of the Alternatives α and β we write Aαβ. Since capital
letters are used for connectives, it is customary in Polish notation to use lower case
letters for propositional atoms. Hence ‘(¬P ∧ Q)’ becomes ‘KNpq’, ‘¬(P ∧ Q)’ becomes
‘NKpq’, ‘¬((P ∧ ¬Q) ∨ R)’ becomes ‘NAKpNqr ’, etc.
(1) Rewrite the syntactic rules of §9.1(c) for a language using Polish notation.
(2) Render the Polish wffs KNpNq, KNNpq, NKpNq, AKpqr, ApKqr, AANpNqNr,
AKNpqKpNq, ANKKpqKqrNArs into our notation.
(3) Render the wffs (1) to (5) from (b) into Polish notation.
(4) (Difficult!) Show that Polish notation, although bracket-free, introduces no
semantic ambiguities (every Polish wff can be parsed in only one way).

93

11 ‘P’s, ‘Q’s, ‘α’s, ‘β ’s – and form again
In this chapter, we pause to make it clear how our ‘P’s and ‘Q’s, and ‘α’s and
‘β’s, are being used, and why it is crucial to differentiate between them.
We also need to explain the standard convention governing the use of quotation marks (though we will very soon start avoiding their use when convenient).

===== CLOSING: APPENDIX (SOUNDNESS AND COMPLETENESS), FURTHER READING =====
κ
λ
µ
ν
ξ
o
π
ρ
σ, ς
τ
υ
φ, ϕ
χ
ψ
ω

Name
alpha
beta
gamma
delta
epsilon
zeta
eta
theta
iota
kappa
lambda
mu
nu
xi
omicron
pi
rho
sigma
tau
upsilon
phi
chi
psi
omega

English equivalent
a
b
g
d
e
z
(long) e
th
i
k
l
m
n
x
o
p
r
s
t
u (or y)
ph
ch (as in ‘loch’)
ps
(long) o

Thus, with conventional accents, Σωκράτ ης, Πλάτ ων, Aριστ oτ ´λης (Sōcratēs,
Platōn, Aristotelēs) name the Greek philosophers.
The word logic derives from λóγoς (logos), which has multiple meanings, including word, reason, and account. Thus psychology is a late coinage derived
from Greek roots, for an account of the psychē, ψυχή.
In Aristotle, a συλλoγισµóς (syllogismos) is an inference. In Euclid, a θώρηµα
(theōrēma) is a proposition to be proved.
One of the Greek words for love is φιλία (philia); and wisdom is σoφία
(sophia). Hence a lover of wisdom is a φιλóσoφoς, a philosophos.

412

Further reading
Parallel reading
If asked to recommend just one text to read alongside this book, I would choose
Nicholas J. J. Smith, Logic: The Laws of Truth (Princeton UP, 2012).
This is very clearly and engagingly written in a similar spirit.
Smith’s book also, in parts, takes an interestingly different approach since it
has chapters exploring logic by the tree method, before it later turns to a brief
treatment of natural deduction. We can argue about the pros and cons of the
two approaches, trees vs natural deduction, as an introduction to formal logic:
ideally you should know something about both. Indeed, the first edition of this
present book was tree-based. As an alternative to Smith’s chapters, then, you
will find revised versions of my earlier tree chapters at logicmatters.net.
Since Fitch’s own 1952 book, there have appeared over thirty introductory
texts using his style of proof system. No two seem to agree at all the main
choice points – for example, do we use an absurdity constant, do we use dummy
names, do we indent proofs when using the ∀-introduction rule? However, for a
friendly introduction with a similar enough system see:
Paul Teller, A Modern Formal Logic Primer (Prentice Hall, 1989).
Teller’s book also covers trees, has a particularly approachable, relaxed style, and
is freely available at the book’s website, tellerprimer.ucdavis.edu. Alternatively,
see the brisker open-source text
P. D. Magnus et al., forall x, forallx.openlogicproject.org.

Philosophical matters arising
We have inevitably touched on a number of philosophical issues which we couldn’t
pause to discuss at length – starting with worries about the nature of propositions and about the very idea of necessity. A wonderful resource which has many
entries on general topics like these, and entries too on the particular logicians
and philosophers mentioned in this book, is
The Stanford Encyclopedia of Philosophy, plato.stanford.edu.
However, some of these entries will be rather tough going for a real beginner, so
here are a few more suggestions. First, two books:

413

Further reading
A. C. Grayling, An Introduction to Philosophical Logic (Blackwell, 3rd edn.
1997),
Mark Sainsbury, Logical Forms: an Introduction to Philosophical Logic (Blackwell, 2nd edition 2000).
Read Grayling Chs. 2 and 3 on the nature of propositions, and on the ideas of
necessity and analyticity. And read Sainsbury Chs. 1, 2, 4 and 6 on logic and the
project of formalization, on truth-functionality, and on quantification. Sainsbury
discusses conditionals in his Ch. 2 (and in his Ch. 3 too). For more on troubles
with conditionals, see the introduction and some of the papers collected in
Frank Jackson, ed., Conditionals (OUP, 1991).
Jackson himself proposes the neat story about the meaning of indicative ‘ifs’
sketched in §19.5(c).
Our logical system in this book is ‘classical’, containing the rule (DN), or
equivalently (LEM). For more on constructivist doubts about the legitimacy of
these rules see
Stephen Read, Thinking About Logic (OUP, 1995), Ch. 8.
Richard Zach et al., The Open Logic Text, openlogicproject.org, first chapter
of Part XI, Intuitionism.
We have helped ourselves to a basically Fregean distinction between sense and
reference. For an approachable discussion of Frege’s own views see
Harold Noonan, Frege: A Critical Introduction (Polity, 2001), Chs. 4 and 5.

Going further in formal logic
There is a very extensive annotated Study Guide available at logicmatters.net.
But two good places to start are
David Bostock, Intermediate Logic (OUP, 1997),
Ian Chiswell and Wilfrid Hodges, Mathematical Logic (OUP, 2007).
Despite their titles, neither book goes a great deal beyond this one. But Bostock
is very good on the motivations for and interrelations between various styles
of logical system, and on explaining some key metatheorems. Also see his last
chapter for some discussion of empty names, empty domains, and free logic.
Chiswell and Hodges are also very clear and their book gives an excellent basis
for more advanced formal work.
Our final chapter hinted that there is much of interest in formal theories of
arithmetic, and in Gödel’s famed incompleteness theorems in particular. If those
hints piqued your interest, then I can’t resist suggesting that you soon dive into
Peter Smith, An Introduction to Gödel’s Theorems (CUP, 2nd edition 2013;
now freely available at logicmatters.net).

414

Index
Entries for symbols, rules of inference, and concepts give the principal location(s) where
you will find them introduced. Names are not indexed when they are merely used in
examples.
∧, 68, 82, 86–87
∨, 68, 82, 86–87
¬, 68, 82, 86–87
&, 68
∼, 68
0
, 72
6, 72, 159
:=, 86, 339
p. . .q, 99
$, 106
⊕, 114
↓, 118
↑, 119
, 122, 140, 159, 221, 356
≈, 142
, 143
>, 146
⊥, 146
→, 150, 159
↔, 156
⊃, 158
≡, 158
`, 222, 356
¬, 232
h. . .i, 234
∀, 253, 262
∃, 253, 262
≃, 276
:=q , 339
=, 363, 367
6=, 367
∃1 , 371
∃!, 371
♦, 379

(∧I), 174–175
(∧E), 174–175
(RAA), 177–179, 181
(Abs), 178–179, 181
(DN), 180, 181
(EFQ), 189
(Iter), 191
(∨I), 192, 196
(∨E), 193, 196
(MP), 203–204
(CP), 204
(LEM), 214
(∀E), 302
(∀I), 303
(∃I), 304
(∃E), 305
(=I), 382
(=E), 382, 384
absurdity, 35, 146, 157, 188, see also
reductio ad absurdum
rule, 178–179, 181
Ackermann, W., 352, 358
alphabet
of PL language, 72, 157
of QL language, 334
of QL= language, 367
analytic, 125
antecedent, 149
argument, 1
deductive, 4
indirect, 35–37
inductive, 4
multi-step, cogency of, 33–35

415

Index
one-step vs multi-step, 14, 28–30
schema, 6
sound, 14
valid, 14
Aristotle, 13, 17, 27, 29, 40–42, 45, 49,
67, 120, 211, 240
arity, of predicate, 232
atomic wff, atom
of PL language, 69
of QL language, 259–260
of QL= language, 367
‘bad’ line in truth table, 131
Barber Paradox, 313
Begriffsschrift, 67, 170, 186, 251, 269
Berkeley, G., 295
biconditional, 156
bivalence, 89
Boole, G., 320
Bostock, D., 414
Buridan, J., 46
Carroll, L., 8, 28, 31
Chiswell, I, 414
Church, A., 352
classical logic, 89, 227
cogency, internal, 2
conclusion, 2, 7
conditional, 148, see also →
antecedent vs consequent, 149
contrapositive of, 149
converse of, 149
counterfactual, 162
‘Dutchman’, 167
indicative vs subjunctive, 163
inference rules, 203–204
material, 151, 169, 208–209
‘only if’, 155
possible-world, 162–163
truth-functional, 150–151, 162–167
conjunct, 77
conjunction, 62–63, 68, 74, see also ∧
inference rules, 174–175
connective
binary, 62, 146
main, 77
nullary, 146
ternary, 106
truth-functional, 104

416

unary, 65, 146
consequent, 149
consistency, 11
PL-consistency, 210, 405
q-consistency, 347
QL-consistency, 328, 407
tautological, 135, 405
contingent, 126
contradiction, 120–121
counterexample method, 40–41
and truth-table testing, 139
countervaluation, 139
De Morgan’s Laws, 113, 117, 195, 197,
214, 269
deduction, 4
definite description, 230, 375
Russell’s theory, 375–377
scope of, 378–379
demonstrative, 230
derivation, see proof
determinacy assumption, 84–85, 260
disjunct, 77
disjunction, 74
exclusive, 64, 113–114
inclusive, 63–64, 68, see also ∨
inference rules, 192–193, 196
disjunctive normal form, 119
divide-and-rule strategy, 67, 111, 170
domain of quantification, 247–248,
252–253, 266, 392
empty, 253, 329–332
entailment, 4, 9
q-entailment, 346
tautological, 129
enthymeme, 32, 38
Entscheidungsproblem, 352
equivalence, 12
class, 362
relation, 361–363
tautological, 142
ex falso quodlibet, 146, 189
existence
‘is not a predicate’, 373
and definite descriptions, 377–378
and existential quantifier, 253
claims about, 372–373
existential import, 273

Index
existential quantifier, 263, 266
formal inference rules, 304–305
informal inference rules, 295–298
explosion, 145–146, 187–189
expressive adequacy, 116–117
extension
of function, 105
of predicate, 234–235, 333
extensional
context, 237
language, 239
extremal clause, 74
fallacy
affirming the consequent, 154
denying the antecedent, 154
of irrelevance, 49, 188
quantifier shift, 42, 254
falsum, 146
Fitch, F., 177, 413
form
logical, 57, 125, 286–287
of inference, 6, 56–58
of wff, 101–102
truth in virtue of, 125
validity in virtue of, 48, 142, 291
formalization, 67
free logic, 331, 373
Frege, G., 30, 53, 54, 67, 85, 151, 170,
186, 236, 251, 252, 269, 286, 300,
319, 337, 357
function, 105, 391–392, see also truth
function
argument, 391
extension, 105, 394
partial vs total, 392
value, 391
functional term, 230, 393–394
Gödel, K., 358, 400, 414
Gentzen, G., 186, 300
glossary
for PL language, 69, 81
for QL language, 260
‘good’ line
in proof, 224, 357, 402, 404
in truth table, 131
Grayling, G., 414
Hamkins, J., 245

Hilbert, D., 352, 358
Hodges, W., 414
hypothetico-deductive model, 17
identity, 360, 363–365, 367–368
numerical vs qualitative, 361
identity of indiscernibles, 365
iff, 157
implication, 159
indiscernibility of identicals, 365
induction, 4
inference, 2
abductive, 16
marker, 7, 127
necessarily truth-preserving, 11, 19,
49–50
to best explanation, 16–17
valid, 4, 9
inference, form of, 6, 56–58
instance of quantified wff, 255, 301
introduction vs elimination rules, 174,
179, 196, 204, 314
intuitionist logic, 226, 414
invalidity principle, 13
iteration rule, 191
Jackson, F., 414
Kreisel, G., 411
language
extensional, 239
formal, see under PL, QL, QL=
formalized, idea of, 56, 67, 69
object- vs meta-, 95
law
De Morgan’s, 113, 117, 195, 197,
214, 269
excluded middle, 120, 122, 211, 214,
225–227
non-contradiction, 120
Leibniz’s Law, 364–365
formal rule, 382–384
second order version, 389
Leibniz, G., 364
Lewis, D., 162
Liar Paradox, 84
live assumption, 223, 303, 402
logical constant, 146
Loglish, 255, 275

417

Index
Lukasiewicz, J., 93
Magnus, P., 413
main logical operator, 77, 336
Mathematical Logic (Quine), 103
metalanguage, 95
metatheory
of PL, 216–227, 402–407
of QL, 352, 354–358, 404–405, 407–
411
modal operator, 244, 379
modus ponens, 149, 203
modus tollens, 149
name
dummy, 230, 299–301
empty, 237, 372
proper, 230, 259
scopeless, 246
sense vs reference, 236
natural deduction, 173, 186
necessary truth, 11, 125
necessity
kinds of, 10–11
logical, 47
negation, 64, 74
classical conception, 180, 225, 227
double, 180, 214, 225–227
inference rules, 179–181
Nicomachean Ethics, 41
‘no’ as quantifier, 274
Noonan, H., 414
object language vs metalanguage, 95
omega-incompleteness, 400
only if, 155–156
parameter, see name, dummy
parse tree, 75, see also q-parse tree
uniqueness of, 77, 80
Peano, G., 158
Philo of Megara, 151
PL-consistent, 210, 405
PL language
interpretation, 81–83
reasons for design, 67–69
syntax, 72–74, 157
valuation, 86, 158
PL proof system, see Chs. 20–24
completeness, 224, 405–407

418

Fitch-style layout, 177
motivation, 173
soundness, 222–224, 402–404
summary of rules, 216–219
theorem, 211
Polish notation, 93
possibility, kinds of, 9–10
possible world, 12, 124
predicate, 231, 259
arity, 232, 259
expressing property or relation, 233
extension of, 234–235, 333
satisfying, 233–234
sense of, 234
premiss, 2, 7
suppressed, 32
Principia Mathematica, 113, 357
Prior Analytics, 17, 27, 240
pronoun, 230
proof
by cases, 193
formal, see PL, QL, QL= proof systems
fully annotated, 30
informal, 28–32, 293–298
schema, 221
property, 233
proposition, 11, 37, 53–56
as sentence, 53–56
as thought content, 55
contrary vs contradictory, 71
propositional logic, 60, 67
q-consistency, 347
q-entailment, 346
q-logical truth, 347
q-parse tree, 335–337
q-saturated set of wffs, 408
q-validity, 290–291, 346–348
mechanical test in simple cases,
349–350
q-valuation, 291, 333, 339–340
expanded, 337–338
isomorphic, 344
QL-consistent, 328, 408
QL language
interpretation, 260–262, 266–268
q-valuation, 291, 333, 339–340
syntax, 259–260, 262–266, 334–337

Index
QL proof system, see Chs. 32, 33
completeness, 357–358, 407–410
soundness, 356–357, 404–405
theorem, 311–312
QL= language, 367–368
QL= proof system, 382–384
sound and complete, 388
quantification theory, 60, see also QL
and QL= proof systems
first-order vs second-order, 389
quantifier, 42
binary vs unary, 241, 251–252, 271,
275
existential, 263, 266
kinds of, 240–241
‘no’, 274
numerical, 370–372
restricted, 251, 271–272
scope of, 243–247, 249, 266, 336
universal, 263, 266
quantifier shift fallacy, 42
quantifier-variable notation, 249–251,
253–255
Quine, W., 88, 99, 251, 322, 360
maxim of shallow analysis, 322
quotes, 99, 103
quotation marks, 95–100
minimizing use of, 100
rational reconstruction of idea of validity, 49, 145, 291, 348
Read, S., 414
reductio ad absurdum
formal PL rule, 179
formal rule, 178
informal, 35–37, 177
reference, 231
relation, 21, 233
equivalence, 361–363
functional, 396
reflexive, 362
symmetric, 362
transitive, 362
Russell, B., 113, 151, 158, 159, 286, 313,
357, 375, 376, 378, 379
Russell’s Paradox, 313
Sainsbury, M., 414
saturated set of wffs, 406

schema, 6, 22–24
basic, 102
proof, 221
tautological, 123
tautological argument, 141
science, role of deduction in, 17
scope, 65–66, 78
ambiguity, 66, 76
of definite description, 378–379
of quantifier, 243–247, 249, 266, 336
semantics
interpreting PL languages, 81–83,
158
interpreting QL language, 260–262
PL valuations, 85, 158
QL q-valuations, 291, 333, 339–340
for QL= language, 367–368
sense
and translation, 83
of function expression, 105
of name, 236
of PL wff, 81
of sentence, 52–53
vs tone, 52–53
sense of
predicate, 234
sentence
type vs token, 52
wff without dummy names, 300
sets, 235–236
Sheffer stroke, 119
Smith, N., 413
Smith, P., 414
soundness and completeness
for PL, 222–224, 402–407
for QL, 356–358, 404–405, 407–410
squeezing argument, 411
subformula, 78
subproof, 37, 177, 218
substitution, 23, 101, 141
syllogism
Aristotelian (categorical), 27, 320
disjunctive, 198–200, 214
syntax, 72
of PL, 72–74, 157
of QL, 259–260, 262–266, 334–337
of QL= , 367, 393
tautological entailment, 129

419

Index
generalizing, 140–141
tautology, 120–121
generalizing, 122–123
logically necessary, 124
Teller, P., 413
term, 231
functional, 230, 393–394
of QL, 300
of QL= with functions, 393
theorem
of PL, 211
of QL, 311–312
vs metatheorem, 216
topic-neutrality, 44–45, 48, 227, 331–
332, 361
Tractatus Logico-Philosophicus, 124,
364
truth function, 105, 110
expressing, 107, 114
truth table
for connective, 86–87
for wff, 90–92, 107–110
good vs bad lines, 131
test for validity, 127–131, 138
truth tree, 138, 351
truth value, 85
calculating, 87–92
gap, 377
tuple, 234
Turing, A., 352
turnstiles, 122, 140, 222, 356, 411
type vs token, 52
universal quantifier, 263, 266
formal inference rules, 302–304
informal inference rules, 293–295
use vs mention, 95–98

420

vacuous discharge, 219
vagueness, ignoring, 84, 127, 134, 227,
260
validity, see also q-validity
classical conception, 9, 11, 18, 50
deductive, 4, 130
definition as rational reconstruction,
49–50, 145, 291
logical, 4, 45–48, 111, 130, 347
of argument vs of inference step, 14
tautological, 129
vs truth, 15–16
valuation, 85, 158, see also q-valuation
c-possible vs r-possible, 111
variable
Greek letter, 100–101
metalinguistic, 95
schematic, 20, 94, 100
vel, see ∨
verum, 146
vocabulary
logical vs non-logical, 81
wff
atomic, 73
basic, 119, 406, 408
closed vs open, 300
construction history, 74
of PL, 73, 157
of QL, 335
of QL= , 367
sentence, 300
Whitehead, A., 113, 357
Wittgenstein, L., 124, 286, 364
Zach, R., 414

