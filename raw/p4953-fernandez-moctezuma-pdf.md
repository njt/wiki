---
url: https://www.vldb.org/pvldb/vol19/p4953-fernandez-moctezuma.pdf
date_fetched: 2026-09-08
---

                                              The Dataflow Model Revisited
       Or: That Feeling When You Realize Every Problem You’ve Been Solving Is a Database Problem

                                  Tyler Akidau                                                            Rafael J. Fernández-Moctezuma
                                   Redpanda                                                                                 Google
                            takidau@redpanda.com                                                                     rfernand@google.com

                                   Reuven Lax                                                                             Daniel Mills
                                    Google                                                                                  Cambra
                               relax@google.com                                                                       daniel@cambra.dev

ABSTRACT                                                                                      1    INTRODUCTION
Eleven years ago, the Dataflow Model paper argued that unbounded,                                                                 Atul
out-of-order data was the new normal, and that we must stop                                                         (tossing the paper on the table)
waiting for data to ever become complete. It proposed a unified                               It’s not good.
model (windowing, triggers, watermarks, and retractions) for freely                                                               Tyler
trading off correctness, latency, and cost across batch and streaming                         I’m sorry, what?
engines. On the occasion of its VLDB Test of Time award, we grade
                                                                                                                               Atul
our own work—a paper about streaming analytics, in truth if not in
                                                                                              It’s well written, but it doesn’t matter. I don’t want to read this.
name— on what aged well, what aged badly, and what we missed.
                                                                                              Why would I waste my time on it?
   We find the paper’s core foundations largely sound: the primacy
of event time, the futility of waiting for completeness, and the                                                              Tyler
insistence on strong consistency aged well. But we got impor-                                                          (dies a little inside)
tant parts of the analytical interface wrong: (1) we let windowing                               Thus began the reviews for the Dataflow Model paper.1 We’ve
and triggering, whose semantics were tangled with operational                                 all gotten blunt feedback on papers. Sometimes it hits hard, at
concerns, dominate the exposition beyond their due, (2) triggers                              the right time, in the right way. With elaboration, Atul Adya’s
were an over-engineered answer to a question users should never                               tough love landed thusly: it’s not always enough to have good
have faced, and (3) the stream-centric worldview missed a deeper                              ideas; sometimes you need to hit the reader over the head with why
truth: streams and tables are two representations of the same object                          those ideas matter, and in language that resonates across disparate
with different access semantics. The mechanisms that delivered on                             communities.
the paper’s analytical goals ultimately evolved out of the database                              Our reaction was a manifesto aimed at everyone. The revised
playbook: SQL, incremental view maintenance, and materialized                                 paper did not merely describe a model and a system; it planted a
views with explicit freshness contracts. We focused too much on                               flag: “We as a field must stop trying to groom unbounded datasets into
the mechanics of streaming instead of finishing what the database                             finite pools of information that eventually become complete.” [8] That
community started but never completed: making the complexity                                  anthemic register, equal parts technical contribution and call to
of analytical streaming disappear almost entirely.                                            arms, was a key driver in the paper’s eventual impact. The vocabu-
   Still, the verdict is not all confession. We explore how the com-                          lary it consolidated (event time versus processing time, watermarks,
pleteness principle split into two successful forms: watermarks                               triggers, unaligned windows) spread well beyond the systems we
(where streams stay visible) and snapshot-consistent refresh                                  built, and a decade later the paper is generously remembered as a
(where they do not); we trace why the latter reached far more                                 paradigm shift in stream processing [63].
users by asking far less of them, and generalize the former into                                 A retrospective is by definition an exercise in self-indulgence,
declared constraints on change. We also (1) find the batch-versus-                            but if you make it equal parts self-critique, it at least smells a little
streaming debate was mostly semantic, (2) watch low-latency de-                               better in the end. The honest answer to “did we get anything right?”
mand bifurcate along the old OLTP/OLAP line, leaving analytics                                is complicated in ways we think are instructive and illuminating
happily at gentler freshness, (3) adopt the framing we wish we had                            about the overall arc of streaming (particularly analytics) and its
started with (leave in, leave out, push harder), and (4) ponder the                           long and storied history with the world of databases. This paper is
eventual disappearance of streaming beyond analytics.                                         therefore a self-assessment, organized as follows: if we were writing
                                                                                              the Dataflow Model paper again today, what would we leave in,
PVLDB Reference Format:                                                                       what would we leave out, and what do we wish we had pushed on
Tyler Akidau, Rafael J. Fernández-Moctezuma, Reuven Lax, and Daniel
                                                                                              harder? Which of the things we built turned out to matter? And
Mills. The Dataflow Model Revisited. PVLDB, 19(12): 4953 - 4964, 2026.
doi:10.14778/3827998.3838710                                                                  licensed to the VLDB Endowment.
                                                                                              Proceedings of the VLDB Endowment, Vol. 19, No. 12 ISSN 2150-8097.
This work is licensed under the Creative Commons BY-NC-ND 4.0 International                   doi:10.14778/3827998.3838710
License. Visit https://creativecommons.org/licenses/by-nc-nd/4.0/ to view a copy of
                                                                                              1 Dialogue paraphrased; memory is imperfect. But some experiences sear themselves
this license. For any use beyond those covered by this license, obtain permission by
emailing info@vldb.org. Copyright is held by the owner/author(s). Publication rights          in more strongly than others.




                                                                                       4953
which of the things we worried about simply stopped mattering on                               2015 and MillWheel [5]3 papers made it unavoidable. Per reviewer
their own?                                                                                     feedback, we hit the community over the head with the urgency of it,
   One observation worth stating up front: though much of the                                  and industry responded in turn. Today it is load-bearing vocabulary
original paper’s rhetoric was framed in general terms, the Dataflow                            in every major system [e.g., 16, 22, 55] and in the documentation
Model itself was never a truly general streaming model; it was                                 of every product with a streaming story. If the paper contributed
always focused on streaming analytics. Streaming as a broad dis-                               one permanent thing, that shift is it.
cipline extends beyond the confines of analytics, and it should be
assumed that the lessons gleaned from the successes and failures                                  Consistency is non-negotiable. In 2015, streaming systems were
of the Dataflow Model extend primarily to the realm of analyt-                                 widely assumed to be fast but wrong, an assumption institutional-
ics unless otherwise indicated. We address the broader evolution                               ized by the Lambda Architecture [46]: a weakly consistent speed
of streaming across applications, infrastructure, and pipelines in                             layer, forever apologizing to a batch layer that produced the real
Section 7. With that caveat in mind, we proceed.                                               answers. The paper insisted that properly built streaming systems
   Our verdict, in brief: we got the physics right, but the inter-                             could match batch correctness, full stop. The field agreed; indeed,
face wrong. The primacy of event time, the futility of waiting for                             the industry’s own practitioners had begun calling time on Lambda
unbounded data to become complete, and the insistence that con-                                well before we went to print [40]. Exactly-once processing became
sistency was non-negotiable have all aged well (Section 2). What                               table stakes [26, 53, 69], and the dual-pipeline pattern has since
the field did with the completeness principle is a richer story than                           retired to the museum of workarounds. We are also happy to report
the one we told, and it split down two roads, only one of which the                            that the paper’s underlying sociological claim, that users quickly
paper foresaw (Section 3). The interface is where the regrets live:                            stop trusting weakly consistent results, required no revision at all.
windowing proved simpler than we made it out to be (just another                                  Never rely on completeness. The paper asked the field to “live
grouping dimension), and should not have been the foundation of                                and breathe under the assumption that we will never know if or
our consistency model. Triggers, the user-facing retraction proto-                             when we have seen all of our data” [8]. In replacing the grooming of
col, and the stream-only worldview they were embedded in aged                                  datasets toward a completeness that never comes with continuous
poorly, each an elaborate answer, in hindsight, to a question the                              adaptation as data evolves, we anticipated a world with a richer
user should never have been asked (Section 4). And the biggest                                 set of interesting inputs: not just append-only event logs but also
miss: the database literature already held decades of the right tools                          changelogs of mutable state, arriving late, out of order, and subject
(relational operators and algebras, incremental view maintenance,                              to revision. The principle stands. Trouble was, in the same breath,
materialized views, and punctuation semantics), largely unrecog-                               we handed the field a replacement: the watermark, a completeness
nized or unacknowledged by us,2 and never fully developed to their                             estimate with the model’s canonical trigger built around it. So
logical conclusion by the community that invented them (Section 5).                            what the field ultimately heard, as we discuss next, was something
We close with what we would do differently, what remains open,                                 narrower.
and where we think this line of work goes next (Sections 6–8). The
aggregate lesson of the decade, and the through-line of this paper:                            3     TWO ROADS FROM COMPLETENESS
we were all focused on the details of streaming, and although many
                                                                                               If you refuse to wait for completeness, you must still decide when
of those details did and still do matter, the real solution for analytics
                                                                                               to say something. The 2015 paper’s first answer was to reject the
was making streaming disappear almost entirely.
                                                                                               premise: completeness is a thing to estimate, never to await. Water-
                                                                                               marks were the estimate, a heuristic signal of event-time progress;
2    WHAT WE GOT RIGHT                                                                         windows gave the estimate something to attach to, carving un-
For starters, the model saw broad adoption: Google Cloud Dataflow                              bounded data into finite regions that can individually be declared
and Apache Flink built on it directly and remain under active devel-                           done; and triggers made the pairing a language for acting before,
opment, and its fingerprints are on Apache Kafka Streams, Apache                               at, and after that signal. The decade that followed took the com-
Spark Structured Streaming, Hazelcast Jet, Apache Samza’s High                                 pleteness question down two very different roads, and the paper
Level Streams API, RisingWave, and BigQuery Continuous Queries.                                overtly anticipated only one of them.
But adoption is a coarse measure of success. The ideas won mind-
share, but did they deliver on their promises?                                                 3.1     The Stream-Centric Road:
   Three of the paper’s bets aged well, essentially without caveat.                                    Watermarks Über Alles
Interestingly, all three apply generally across all of streaming, not
                                                                                               Within stream-centric systems, watermarks won outright. Flink
just analytics. The things we got right, we got really right.
                                                                                               adopted them faithfully, Spark and Kafka Streams in deliberately
   Event time versus processing time. The distinction between when                             simplified form, and the concept proved robust enough that three
an event happened and when a system observes it most certainly                                 substantially different architectures could implement recognizably
did not originate with us. The distinction runs through the time-                              the same idea. By 2021, several of us had distilled a decade of pro-
management and out-of-order-processing literature [45, 64], but our                            duction experience into a paper of our own, establishing formal

                                                                                               3 Retrospectives also offer an opportunity to right some wrongs, and one worth high-
2 In his defense, Rafael has always been a SQL true believer; he was the one who gave          lighting here is that Tyler is often misattributed as lead author on the MillWheel paper
us our academic street cred in the original paper. The rest of us took longer to come          due to the alphabetical power of his last name; Sam McVeety was the true lead author,
around.                                                                                        but we made the misguidedly egalitarian decision to alphabetize everyone.




                                                                                        4954
watermark semantics and comparing two of the leading implemen-                                  in a table in a database somewhere; the destination was never in
tations, Flink and Cloud Dataflow, in depth [7].                                                question. Those same engines have themselves been converging
   That paper also recorded a lesson the 2015 paper did not yet                                 on declarative surfaces for describing that table—SQL of course,
know, one that had taken years of production use to come into                                   alongside Flink’s Table API, Beam’s schema-aware transforms, and
focus: not all situations truly need such a completeness signal [7].                            its YAML API. But hand-assembling that table with a pipeline is
The ones that do fall into two related camps:4                                                  not the same product as declaring it, and the difference is what this
                                                                                                road is about.
   Single conclusive answers. Some computations demand one fi-
                                                                                                    In this table-centric formulation, the table represents an incre-
nal result rather than a stream of refinements. Sometimes the ac-
                                                                                                mentally refined output state that usually lags the state of its inputs.
tion is hard to amend: the notification or alert that must fire once,
                                                                                                This formulation is eventual consistency. In fairness to our younger
correctly. Sometimes the intermediates are actively misleading: a
                                                                                                selves, the 2015 paper was never opposed to it; rather, it explicitly
click-through rate joined from two unsynchronized inputs will hap-
                                                                                                admired the Lambda Architecture’s move of offering a best low-
pily report values north of 1.0 until both sides converge, and its
                                                                                                latency estimate with correctness to follow, and set out to reproduce
consumers prefer no answer to a nonsensical one.
                                                                                                that pattern within a single streaming pipeline; panes (the paper’s
   Reasoning about absence. Without a completeness signal, there                                term for each successive triggered rendering of a window), refine-
is no way to distinguish events that are missing from events that                               ments, and retractions were eventually consistent output in all but
are merely delayed. Outer joins and anomalous dip detection are                                 name. What the paper distrusted, rightly, was weak consistency:
the canonical examples.                                                                         data lost or duplicated along the way, the speedy layer whose num-
                                                                                                bers users learned to stop believing. What the paper never did was
    Retraction frameworks, such as in DBSP [19], can express both
                                                                                                name the eventual-consistency posture it had adopted and follow
without any completeness signal, but only in the limit, as every
                                                                                                through on its concrete implications. The reason, in hindsight, is
result remains provisional and correctable. Yet provisional is exactly
                                                                                                simple: the paper was stream-centric to its core.
what these use cases cannot accept: a retraction can amend an an-
                                                                                                    Viewed through a stream-centric lens, the pitch holds together:
swer; it cannot un-fire an alert. Revision and finality are orthogonal
                                                                                                the output is a stream of ever-better answers, and refinement is
concerns; these use cases need the latter, and watermarks remain a
                                                                                                something that happens to the results as they flow past. But the
practical tool for providing it.
                                                                                                lens is incomplete in the way that counts: a stream of refinements
    The need for completeness is real. But the larger truth cuts the
                                                                                                is, inescapably, building a table. And every question that actually
other way: many analytical applications are well served by incre-
                                                                                                matters about eventual consistency is a question about that table:
mental refinement alone, where an answer that updates continu-
                                                                                                What state is the result in right now? Is what I am reading inter-
ously provides more utility than a static answer you have to wait
                                                                                                nally consistent? Will it ever stop changing? The model had no
for (modulo the consistency trade-offs of Section 3.2). And even
                                                                                                vocabulary to even ask them (a gap we return to in Section 5).
the applications that demand completeness often tolerate delay:
                                                                                                    Those questions deserve clear eyes, because eventual consistency
billing must be complete, but rarely within seconds; workloads
                                                                                                does not automatically deserve trust. As we have memorably heard
sold as “low latency” rarely mean anything less. Watermarks solve
                                                                                                David Maier observe: eventual consistency is entirely compatible
completeness at low latency. Put together, the honest accounting
                                                                                                with perpetual inconsistency. Precision helps here, because the term
is telling: needing completeness is common, and needing speed is
                                                                                                means more than one thing.
common; needing both at once is far more rare—and yet is precisely
what we designed for.                                                                              Lag. The common usage refers to output that lags input: of-
                                                                                                ten true of NoSQL key-value stores such as Apache Cassandra,
3.2     The Table-Centric Road:                                                                 generally true of asynchronously replicated systems, and true by
        Ease of Use for the Win                                                                 definition of continuously maintained tables and views. This lag
The larger impact of the don’t-wait principle arrived by a road the                             dimension is the tamer of the two. Completion information tells
paper did not anticipate: materialized views. A user declares, usu-                             you which rows are complete and how far behind you are, and
ally in SQL, what the output should be, and the system keeps a table                            it already has well-worn forms: replication high watermarks in
continuously materialized as its inputs change. The past decade                                 replicated systems, data watermarks or punctuations in ours.
filled in this road from every direction: Materialize [48], Delta Live                             Incoherence. The second dimension is the one that bites: internal
Tables [68], Snowflake Dynamic Tables [59], Feldera [19], BigQuery                              snapshot consistency. A table should be self-consistent at every
Continuous Queries [34], and Flink Materialized Tables [12], all rest-                          moment, even while it lags its inputs; if a query references the
ing on the long history of materialized view maintenance [35]. The                              same rows from two subqueries, both should agree on the set of
systems differ in trade-offs (append-only outputs versus generalized                            rows processed. Obvious as that sounds, streaming pipelines have
view maintenance, and the consistency spectrum we return to be-                                 routinely failed to provide it.
low), but they share the quality that matters: a declarative interface                             The 2015 paper’s implicit answer was windowing: delay mate-
used to materialize a table, with the streaming left to the engine.                             rialization until the window closes, and consistency follows. But
    Tellingly, even pipelines built on stream-centric engines such                              a user should not have to modify their query’s semantics, adding
as Google Cloud Dataflow and Apache Flink most often terminate                                  a grouping they never wanted, to obtain a guarantee the system
4 A third concern belongs to the engine: obsolescence, knowing when buffered state can          should have provided. Windowing as a consistency crutch can even
safely be reclaimed. This one is internal machinery, not user semantics.                        break the query outright: add a tumbling window to a join, and




                                                                                         4955
rows falling on opposite sides of a window boundary silently stop                              The industry has been converging on this generalization (final-
joining. A windowed aggregation belongs in a query to produce                               ization) from more than one direction. Delta Live Tables made early
windowed results, full stop.                                                                strides in this space: its streaming tables treat append-only input as
                                                                                            processed-once and never revisited, with APPLY CHANGES layering
   Fortunately we see systems now being built that provide snap-                            change data capture on top [27]. A more imperative approach, but
shot isolation or stronger [3]. The distinction from mere freshness                         effective nonetheless. Four years later Snowflake delivered frozen
is worth spelling out. Under snapshot isolation, a result is still only                     regions on Dynamic Tables [58]: a declarative predicate that marks
eventually consistent with respect to the final answer, but it is per-                      rows immutable, allowing engines to skip re-verification of frozen
petually consistent as a view of the world: at any given moment,                            data, downstream consumers to treat it as final, and backfilled his-
everything you read reflects one coherent state of the inputs. This                         tory to remain stable even when upstream sources diverge.
holistic coherence is what makes the easy, table-centric interface                             Generalizing the second axis: there is no reason progress must
safe for the vast class of correlation-shaped use cases (joining and                        be measured only in time, or even in time at all. The ladder of gen-
comparing across sources) that quietly fall apart when each table                           erality was visible all along, as several of us later surveyed in detail
reflects a different moment. Lacking it, eventual consistency decays                        in the watermarks paper [7]. Watermarks track a single time do-
into Maier’s perpetual chaos.                                                               main. Timely Dataflow’s timestamp frontiers [52] let multiple time
   How much coherence, spanning how many tables, is where                                   domains advance independently and are the machinery beneath
systems differ. Per-view atomicity is the floor, and increasingly                           Differential Dataflow’s simultaneous incremental-and-iterative pro-
table stakes. Delta Live Tables extends refresh consistency across                          cessing. Punctuations [64] subsume them all: completeness signaled
a pipeline [68]. Snowflake Dynamic Tables discards the pipeline                             for arbitrary predicates over the data, a canonical notion of progress
boundary entirely: refresh timestamps are guaranteed to align                               acknowledged in both our 2015 Dataflow and 2013 MillWheel pa-
across any set of tables in the account, regardless of differing target                     pers [5, 8] as long anticipating our notion of watermark. The lineage
lags, but reads are consistent only within a single table (product                          keeps resurfacing: sequence-based progress tokens in modern table
decision, not architectural limit) [59].                                                    systems, and most recently watches in the Pub/Sub setting [2], pro-
   Materialize and Feldera hold the strongest line: every read, at                          posed in a symmetry we most certainly did not engineer, by two
any moment, observes all views at a single timestamp [19, 47]. A                            reviewers acknowledged in our 2015 paper.
coherent view across everything you read at once remains the differ-                           It is also no accident that the NEXMark queries [65] remain
entiator, not the floor; coherence across everything you maintain,                          the field’s benchmark staple: they were written by people who
meanwhile, has quietly become the expectation.                                              understood progress as something more general than a clock. So
   This formulation delivers everything analytical users wanted                             why did the doubly special case win? Because generality, in those
from streaming, minus everything that made streaming hard. The                              formulations, billed by the operator: punctuations require every
road rests on classical incremental view maintenance, modernized                            operator to correctly update and propagate arbitrary predicates,
by Timely and Differential Dataflow [49, 52], change queries [6],                           frontiers make sophisticated operators intimidating to write, and
DBSP [19], and Enzyme [68]. All of this work did much to put the                            most use cases need neither. Watermarks were a deliberate sweet-
2015 paper’s stated goals (correctness, latency, and cost, tunable per                      spot choice, not an oversight [7].
use case) into the hands of ordinary users uninterested in authoring                           Put the two generalizations together and a vocabulary emerges
or maintaining streaming pipelines. It achieved this chiefly by never                       that we think is the right way to have framed the whole space:
asking users to think about streams at all.                                                 declared constraints on change. What can no longer change (final-
                                                                                            ization); in what order changes may arrive (ordering); in which
3.3     The Road Not Taken:                                                                 direction values may move (monotonicity); how change is scoped
        Declared Constraints on Change                                                      (partitioning). Time still appears, but only as one dimension a con-
Both roads were grasping toward the same underlying concept:                                straint might reference, not as the substrate of the vocabulary itself;
completeness. Our model gave it exactly one embodiment, windows                             the watermark does not necessarily go away, but a table is free to
finalized by watermarks. But as Section 3.2 argued, windowing is                            decide whether to use it for finalization. A constraint can still refer-
an aggregation dimension, not a consistency mechanism, and the                              ence watermark-based window finalization, but is no longer limited
watermark is only one kind of completion signal.5 Completeness                              to only that approach. These are properties engines can verify and
in tables generalizes along two axes.                                                       exploit. Crucially, they are also easy for the engine to manage and
   Generalizing the first axis: what window finalization really as-                         propagate. Were we to write the paper again, this would be our
serts is that a region of data has become immutable. There is no                            central completeness narrative.
reason the boundaries of that region must be monotonically advanc-
ing points in event time; it can be an arbitrary declared predicate,
provided the immutable region never shrinks. The predicate may                              4 WHAT WE GOT WRONG
be defined in terms of a watermark or punctuation, but business
logic is free to dictate others. For example, an account being closed                       4.1 Windowing and Triggering, Entangled
may mean that all associated rows are now immutable.                                        The 2015 paper’s most elaborate machinery is the windowing for-
                                                                                            malism. With concepts such as assignment and merging as sepa-
5 That windowing delivered completeness and consistency was never the problem. The          rable operations, and sessions as the motivating case, the formal-
quibble is that the model pitched it as the only way of obtaining them.                     ism was novel in its merging half, while assignment we borrowed




                                                                                     4956
openly from Li et al. [44].6 Even on its own terms the formalism                               generalized the contract: a Dynamic Table’s target lag may be de-
was incomplete: merging has an inverse, splitting, and we missed                               clared at any node in the dependency graph, not merely at the out-
it; windows in our model could grow but never shrink.7 Temporal                                puts [57, 59]. Interior tables that exist purely as plumbing declare
validity windows—in which each value in a relation is valid un-                                their lag as DOWNSTREAM, deriving their cadence from whichever
til superseded, so that a late-arriving successor quietly truncates                            tables consume them (precisely the sink-driven derivation just de-
the reach of its predecessor, splitting whatever data the shrunken                             scribed), while the scheduler reconciles competing contracts by hon-
window once contained—require exactly the operation we never                                   oring the strictest downstream demand and refreshing the graph
provided. The use case is as practical as they come: it is how you                             in dependency order. The generalization is the table-centric world-
join against a table of currency-conversion rates in event time [36].                          view asserting itself once more: in a pipeline, only the outputs speak
One of us later spent a book chapter working around its absence [9],                           for freshness; in a database, every node is potentially somebody’s
and the temporal-database literature had, naturally, been modeling                             output, and so every node must be allowed to speak.
validity intervals for decades: valid time dates to the field’s found-                            Meanwhile, BigQuery moved the contract across the divide en-
ing taxonomy [56], and had reached the SQL standard [41] years                                 tirely. Classic materialized views, such as in BigQuery, Snowflake,8
before our model failed to express a window that shrinks.                                      and many other databases, had always held the read side at its
    The trigger language, meanwhile, was expressive enough to                                  extreme: results current as of the query, whether by transparently
reconstruct every emission pattern we had ever encountered: com-                               merging the view with base-table changes at read time or by main-
posites, sequences, repetitions, data-driven firings, custom signals.                          taining the view synchronously with each write, at whatever cost
Yet almost nobody used either windowing or triggers at anything                                that freshness implies. max_staleness made that guarantee nego-
close to full generality. Said differently, it was a nightmare of un-                          tiable [33]: within the declared bound, a query is served from the
necessary complexity. A decade of production distilled the trigger                             view as-is; beyond it, the merge-on-read kicks in—freshness as a
menagerie to two members: fire when the watermark passes, and                                  contract the reader declares and the engine prices, rather than a
fire periodically. The windowing taxonomy, whose full depths later                             fixed property of the view.
required a survey of its own to catalog [67], collapsed for most                                  The diagnosis, we think, is not that the machinery was wrong
users into “group by a time bucket.”                                                           in its details; it is that both pieces were promoted far above their
    The clunkiness of triggers was more than cosmetic; it was struc-                           station. Windowing is a time-flavored GROUP BY, “just a slight
tural. Triggers attached wherever windowing was declared, deep in                              modification of a thing everyone already innately understands:
the interior of the plan, and propagated outward from there. When                              grouping” [9]. In other words, one grouping construct among many,
a grouping lived inside a library transform there was frequently no                            not the center of a model. And what users wanted from triggers
sane place to specify its triggering at all. The design also smuggled                          was never a language for when; it was a contract for how fresh. The
in an obligation the model never honored: a sequence of triggered                              construct that survived contact with real users is that of target
refinements is only meaningful if consumed in order, final pane                                lag, the emission contract in its purest form: a single declared
last. This obligation is really an ordering requirement we return to                           bound on staleness, from which the engine derives every scheduling
in the next subsection.                                                                        decision the trigger language would have made the user specify. The
    The fix, from the model’s perspective, is to invert the arrange-                           engine can optimize against this construct, batching and amortizing
ment entirely: the purpose of triggering is to control emit frequen-                           work in ways an imperative firing rule forbids. The construct now
cies, so the emit policy belongs at the root of the plan (the sink) and                        ships across the industry under different names, such as refresh
should be pushed down by the engine toward whichever interior                                  triggers in Delta Live Tables (2021), max_staleness in BigQuery
groupings require triggering, precisely the way database optimizers                            (2022), TARGET_LAG in Snowflake (2023), and FRESHNESS on Flink’s
have pushed predicates down their plans for decades. That inver-                               Materialized Tables (2024) [12]. One idea, four spellings. Declarative
sion matches what users actually intend, dissolves the ordering                                freshness turned out to be to triggers what the relational model was
obligation into the engine where it belongs, and maps naturally                                to CODASYL’s navigational databases: once users could declare the
onto SQL, where nobody can set a trigger on every grouping, but                                result they wanted, nobody missed telling the system how to get
everyone already attaches export policies at the outputs (EXPORT                               there.
DATA, COPY INTO). Stated as a design principle: separate the se-                                  The expressive benefits of a declarative approach were not un-
mantics of progress from the mechanics of processing, and let the                              known to us at the time: one of us had previously explored letting
schedule of result production be declared from outside the query                               consumers request results from a continuous query via punctu-
rather than programmed within it.                                                              ations flowing against the stream, expressing both the intent to
    Production systems have since validated the inversion, and fur-                            produce (or suppress) results and the subset of the stream refer-
ther generalized it. Delta Live Tables validated it first, with declared                       enced, in terms of the output schema [31].
refresh schedules at the scope of a pipeline [28]. Snowflake then                                 The general lesson is the one this paper keeps returning to: op-
                                                                                               erating on the stream is often too hard for mere mortals. Users
                                                                                               should declare the result they want. Deriving the incremental
6 And while we are redistributing windowing credit, an erratum: the 2015 paper stated          computation—the actual streaming—is the engine’s job, and the
the windowing semantics of CEDR [17] and Trill [23] were insufficient to express               province of experts who enjoy that sort of thing (e.g., [6, 19, 49, 52,
sessions. CEDR could express them via left anti-semijoins, leveraging the temporal             68]).
construct of the data model.
7We don’t claim assign-plus-merge-plus-split completes the algebra; it merely patches
the hole we happened to fall into.                                                             8 Not to be confused with Dynamic Tables.




                                                                                        4957
4.2    The Ordering We Never Promised                                                Second, the paper never said where along the dial the demand
The model declared every collection unordered. The choice came                    actually lives, and it was written from the vantage point of an in-
from two sides: functional programming and an obsession with                      dustry that had already decided the answer was “as low as possible.”
scale-out parallelism. This choice was strictly speaking a falsehood              But as we found in the years that followed, everyone needs low
no one could realistically live by. Real pipelines routinely require              latency until you tell them how much it costs... then for a surpris-
at least weak ordering: any upsert-shaped output where the last                   ing number of use cases, that requirement just melts away. The
write wins is implicitly ordered, and the model’s own triggering                  demand, it turns out, bifurcates, and along a line databases had
semantics—early panes followed by an on-time pane, refinements                    already drawn decades earlier: OLTP versus OLAP.
followed by retractions—are meaningless unless those outputs are                     Operational and transactional uses of streaming (the applications
observed in order. The saving grace, also unnamed in the model,                   and infrastructure we take up in Section 7) skew genuinely toward
is that global order is almost never the actual requirement: an                   millisecond latency, by definition: an application responding to
upsert stream into a keyed store needs only per-key ordering, and                 the world must keep pace with it, often within human reaction
sinks today recover even that ad hoc, with sequence numbers here                  time. Analytics, the 2015 paper’s real subject, returned the opposite
and last-writer-wins there. In practice, users leaned on ordering                 verdict: seconds-to-minutes freshness, delivered cheaply, reliably,
guarantees that specific engines happened to provide and the model                and without operational heroics, serves the wide middle of analytic
never promised. Our miss: we pitched the model without getting                    use cases—and a long tail is content with far less. The systems that
into sufficient detail to formulate ordering semantics, even though               met users there flourished. Ultra-low-latency analytics remains
we were painfully aware of them in practice. It is one more exhibit               vital for a narrow tier of workloads, and genuinely impressive
for the previous subsection’s lesson: stream-level contracts are                  engineering serves that tier, but it is a tier, not the market. Nor was
treacherous even for the people who write them.                                   the engineering wasted: the incremental machinery built in pursuit
                                                                                  of milliseconds is much of what now serves the broader market at
                                                                                  gentler freshness.
4.3    The Latency Realities We Left Unstated                                        Still, much of the industry (and few more enthusiastically than
What did the 2015 paper actually claim about latency? Less than its               us) spent years optimizing the corner of the analytical latency–
reputation might suggest. It said that one can “never fully optimize              cost–correctness space with the fewest customers in it. A paper
along all dimensions of correctness, latency, and cost,” and that                 about balancing correctness, latency, and cost was the natural place
systems must provide tools for balancing the three “appropriate for               to say where the customers were, and it stayed silent. The framing
the specific use case at hand” [8]. That text has aged shockingly well;           was not wrong; our collective aim was, and the text never tried to
it applies generally across streaming even today, and in analytics, a             correct it. Once again, what the field took away was not what the
declared target lag is its thesis shipped as product. We were not even            paper said.
living at the low end of the dial ourselves: our operational concerns
ran closer to single-digit seconds than single-digit milliseconds. On             4.4     The Peace Nobody Heard
its face, there is nothing here to confess. But this subsection earns             On batch versus streaming, we will partially defend our younger
its place among the regrets twice over.                                           selves: the claim was right. The 2015 paper declared that execution
    First, the paper oversold the dial. It promised “the ability to dial          engines should not dictate semantics, that batch, micro-batch, and
in precisely the amount of latency and correctness for any specific               streaming systems could all provide equal correctness, and that
problem domain” [8]; physics declines to honor that promise at                    engine choice should reduce to “the practical underlying differences
the dial’s low-latency end. When data volumes are high, state-of-                 between them: those of latency and resource cost” [8]. A decade
the-art systems can muster seconds of latency at high percentiles                 later, that is simply how the winning systems work: streaming
for their target use cases, but we do not expect generalized large-               semantics computed on whatever machinery the freshness contract
data systems to reach milliseconds: the optimizations that make                   will pay for. Change queries over table snapshots [6] are streaming
queries efficient (batching for vectorized execution among them)                  with batch technology; a declared view refreshing every minute is
evaporate at very low latency, and taming environmental noise                     micro-batch wearing a declarative suit.
such as noisy neighbors means under-utilized machines and more                       What we got wrong was the articulation, and the articulation
expensive queries.                                                                mattered. Since the word “streaming” means two different things,
    Nor is even gentle freshness universal. A MIN aggregation under               the war fed on the ambiguity. There is streaming the semantics:
modifications and deletions requires a full rescan to maintain incre-             computation over data that changes over time, the conceptual model
mentally; the Reactive Aggregator [61] lowers that to 𝑂 (log |table|),            that strictly subsumes batch (a bounded dataset is a stream that
still impractical every ten seconds on a large table; and the ap-                 happens to end), and there is streaming the engine: continuous,
proximate variants that can keep up surrender exactness to do it.                 record-at-a-time9 execution, which is merely one point in a space
Operators resist incrementalization in ways rarely obvious to users,              of execution strategies (batch, micro-batch, true streaming) with
so the system must make the trade-offs on their behalf.                           genuine strengths and materially different trade-offs (a Flink is not a
    Importantly, notice what this example embodies: the paper’s                   Spark, in both directions). Batch as an engine is superb technology:
first quote above, made concrete. Pin latency at scale and cost or
correctness must give. “Never fully optimize along all dimensions”                9 Nominally. No production engine is truly record-at-a-time; there is always batching
was the truer sentence; the dial promised a freedom the paper’s                   under the covers. The important questions are how much, where, and what limitations
own caveat had already revoked.                                                   are imposed as a result.




                                                                           4958
efficient, throughput-optimized, operationally simple, and very hard                             (what happened), a table is a stream of snapshots (what was true
to beat on cost. The 2015 paper saw the distinction; it even tried                               when), and each is recoverable from the other [36].11
to legislate it, reserving “batch” and “streaming” exclusively for                                   One of us later spent a book chapter exhuming the tables from the
execution engines and pushing “bounded” and “unbounded” for                                      Dataflow Model along exactly these lines [9]: every operation in the
data [8]. The field kept half the memo. Bounded and unbounded                                    model classifies as stream→stream (element-wise), stream→table
were then in the vocabulary, but “streaming” swallowed the concept                               (grouping), or table→stream (ungrouping, which is all a trigger
anyway and inevitably some discussions over a decade have gone                                   ever was), and even a MapReduce job comes apart into streams
to defending engines as though the semantics were at stake. With                                 and tables from top to bottom. Then came the formalization: the
two words for the two things, the peace is self-evident: adopt the                               time-varying relation [18], of which a table is a point-in-time view
semantics everywhere, and choose the engine by the freshness                                     and a stream is the changelog. One object, two representations,
contract. With one word for both, we got an engine war our own                                   with the entire relational algebra remaining meaningful over it.
claim said was unnecessary.                                                                      Meijer had even explained the same duality for .NET’s enumerable
   The war ended only when the categories themselves went away:                                  and observable collections years earlier [50].
in the table-centric systems, whether a given refresh executes as a                                  Had we understood this in 2015, most of the paper’s machinery
full recomputation or an incremental delta is the engine’s decision,                             would have simplified: retractions stop being an exotic protocol
made per table and per moment, invisible from above. One sign of                                 and become ordinary changelog rows; triggers stop being a firing
the peace is that Apache Flink itself now ships it: a Materialized                               language and become a materialization policy; and the “unified
Table declares its FRESHNESS, and the engine converts that one                                   model” we claimed arrives not by bridging batch and streaming
number into either a continuous streaming job or a scheduled batch                               APIs but by recognizing they are different views of the same thing.
refresh [12]. When viewed from this angle, there is nothing left                                     Yet even once we had the vocabulary, we still did not internalize
to fight about and no need for a treaty; the war’s subject simply                                all of its consequences. Consider the mechanisms for in-place evo-
dissolves.                                                                                       lution of a running stateful query, which Apache Flink and Google
                                                                                                 Cloud Dataflow both model as an out-of-band operation: upgrade
5 WHAT WE MISSED                                                                                 from a savepoint in Flink, job replacement in Dataflow. Viewed
                                                                                                 through the duality, an update is just more stream: a structural-
5.1 Streams and Tables
                                                                                                 change marker in the changelog, which is how many databases
The most consequential omission is easy to state: tables. The word                               already record schema modifications. The machinery sat outside
barely appears in the 2015 paper, yet tables are everywhere in                                   the model only because nobody asked the model to describe it.
it, unnamed. GroupByKeyAndWindow takes a stream and produces
evolving keyed state (a table). A trigger observes evolving state                                5.2      SQL
and emits its changes (a stream). The paper’s figures drew both
                                                                                                 We bet on a high-level programming model in a general-purpose
halves of this duality repeatedly; its authors missed the nouns. In
                                                                                                 language, largely ignoring SQL. The value of SQL here is threefold
the candid words of the lead author: “I just didn’t fully understand
                                                                                                 and we underestimated all three: reach, because every analyst on
what I was talking about yet. Streams and tables were an epiphany
                                                                                                 earth can write it and none of them will learn a streaming API;
when I finally understood them.”
                                                                                                 declarativeness, because a query the user did not procedurally spec-
    In fairness, the database playbook already had entries moving in
                                                                                                 ify is a query the engine is free to incrementalize, optimize, and
that direction: CQL’s conversion operators shuttled data between
                                                                                                 re-plan, hence enabling the entire database optimization toolkit to
streams and relations [15], and TelegraphCQ ran continuous queries
                                                                                                 become freely available; and its quietly time-varying nature, because
over both side by side [25]. But the duality itself went unstated
                                                                                                 a SQL query over time-varying relations is already meaningful as
(streams and tables remained two things to convert between, not
                                                                                                 both a point-in-time question and a continuously maintained one.
one object seen two ways),10 and the complex packaging did the
                                                                                                 SQL never had a batch/streaming split to heal. Almost.
ideas no favors. So despite knowledge of these works, the epiphany
                                                                                                    Table-native SQL did need one genuinely new primitive: a way
awaited.
                                                                                                 to extract the stream from the table, specifically, the changes be-
    The vocabulary that finally took hold came, true to this paper’s
                                                                                                 tween two states of a time-varying relation, whether surfaced as
thesis, from the database world, and it was already in the air as we
                                                                                                 a CHANGES construct [6, 32] or an EMIT STREAM clause [18] (CQL’s
wrote: as popularized by Kreps, Kleppmann, and the community
                                                                                                 Istream and Dstream had come tantalizingly close two decades
around Apache Kafka [38, 39, 55], the duality had been hiding in
                                                                                                 earlier, outside the standard [15], but as two streams to juggle rather
the architecture of every database all along. The raw materials were
                                                                                                 than one changelog). In the narrow accounting of language sur-
never secret: recovery had long treated the log as the ground truth
                                                                                                 face, the standard already held nearly all of the answers, and the
from which tables are rebuilt [51], warehouses had shipped deltas
                                                                                                 streaming decade’s contribution was a single missing primitive:
between systems for years as change data capture [42], and every
                                                                                                 the reverse of the ratio we would have guessed in 2015. But that
materialized view demonstrated the round trip; it was all simply
                                                                                                 accounting is narrow indeed, and we return to the fuller ledger
filed under mechanism, not model. Read together, the mechanisms
                                                                                                 below.
spell out the model: a stream is an ordered sequence of changes
                                                                                                 11 Hyde’s rendering of the duality is the most physical: a table’s value over time is the
10 CQL came the closest: Istream and Dstream are true change streams out of a relation.          Heaviside step function, and its stream of changes is the Dirac delta, the derivative
But the only road back was a window (excerpts, not integration), so the round trip               of the step. Replaying a log and snapshotting state are the two directions of the
never closed.                                                                                    fundamental theorem of calculus, wearing database clothes.




                                                                                          4959
   Ignoring SQL completely was a mistake; claiming SQL for ev-                    It took both communities, and more than three decades, to finish
erything would be the opposite mistake. Expressiveness is not                     the thought.
actually the issue (recursive common table expressions make SQL                       It is perhaps no coincidence that the database half of this story
Turing-complete, so any computation can in principle be written),                 was written largely in research venues and the streaming half
ergonomics is: an important population of users prefers a general-                largely in production systems. The two halves complemented each
purpose language because some logic is genuinely awkward to                       other in ways neither could have planned. The research community
state declaratively, and closing that gap is a challenge for the SQL              worked out the principles at a time when the workloads that would
community to take up or decline.                                                  demand them at planetary scale did not yet exist: theory ready
   Whichever surface one prefers, though, our data model missed                   and waiting decades ahead of its demand curve. The practitioners,
another opportunity. The 2015 model made no assumptions about                     when those workloads arrived, chased them with more urgency
the structure of elements: to the model they were opaque values,                  than scholarship, and kept rediscovering principles whose proper
their interpretation left entirely to user code (in Apache Beam’s                 names we learned only later. Our title concedes the concepts, and it
embodiment, user-registered Coders decoding blobs into whatever                   should: the concepts were theirs. But it is only half serious, because
types the pipeline author fancied). A decade of experience says                   priority is only half the story; finishing the thought took both the
nearly all real-world data fits a relational schema, and the sliver that          ideas and their delivery. And if the streaming decade earned any-
does not (encoded video, say) fits a single binary column. Building               thing in this telling, we hope it is a seat at the table it took us a
on schemas and a relational algebra from the start would have                     decade to see.
bought a semantic layer that users could reason about and engines                     Regardless, the result was worth the time and effort it took on
could plan and optimize. This is a lesson demonstrated since by                   both sides. A user declares a table (any SQL query) and a freshness
Beam’s own schema API [11] and Flink’s Table API [14]. Blobs                      contract. The engine streams. The user never sees it. That is what
gained us generality precisely where nobody needed it, at the cost                the 2015 paper was reaching for, and it is instructive that reaching
of everything an optimizer could have done with the structure we                  it required deleting nearly everything the paper spent its pages on
threw away.                                                                       from the analytical user surface.
                                                                                      There is a final irony here, best stated in the vocabulary of the
                                                                                  Beam Model’s take on stream-and-table theory itself, summarized
5.3    The Answer Neither Community Finished                                      in Figure 1. The operation taxonomy in that theory has a deliberate
       Alone                                                                      hole: there is no table→table operation, because data cannot pass
Incremental view maintenance is the user-friendly answer to stream-               from rest back to rest without moving in between [9]. Yet a declared
ing analytics, and is thirty-something years old [35]. This is the                table maintained over other tables is precisely a table→table oper-
keystone “That Feeling When” of our title, and it deserves to be                  ation at the interface. The physics have not changed; the streams
stated plainly: the problem the 2015 paper attacked with windows,                 are all still in there, doing the moving. They have simply been in-
triggers, watermarks, and retractions (keep a derived result con-                 ternalized by the engine. The one operation our own theory said
tinuously correct as its inputs change) is the materialized view                  could not exist turned out to be the product everyone wanted, and
maintenance problem, posed and substantially theorized by the                     manufacturing the illusion of it is, quite literally, what it means to
database community while most of us were still in school.                         make streaming disappear.
    But the title’s feeling cuts both ways, and here is the second
edge: if every problem we worked on was a database problem,
these problems belonged to the database community all along, and
the community that invented the answers did not finish them either.                                 → stream                       → table
Classical materialized views arrived with restrictive query support,

                                                                                    stream →
opaque staleness, and ergonomics that assumed a DBA in the loop;                                element-wise ops                   grouping
they remained a niche feature for decades, and while the theory kept                            (ParDo, filters, joins)    (GroupByKey, windowing)
advancing (higher-order view maintenance [4] would eventually
feed DBSP [19]), the database world largely stopped pushing the
product.                                                                                                                    impossible, and yet. . .

                                                                                     table →
    Getting from that foundation to systems ordinary users adopt                                   ungrouping             the product everyone wanted:
                                                                                               (all a trigger ever was)         declarative views,
took the streaming decade’s contributions. The theory hardened
                                                                                                                            with streams hidden inside
into general incremental-update frameworks: first Timely and Dif-
ferential Dataflow [49, 52], then DBSP [19], whose elegant reduction
turns the incrementalization of any query into a mechanical, prov-                Figure 1: The stream/table operation taxonomy [9]. The
ably correct circuit transformation. Onto that base came event-time               fourth cell cannot physically exist as data cannot pass from
semantics grafted where they matter [18], change derivation over                  rest to rest without moving in between. The winning inter-
arbitrary query plans [6], and above all the ease-of-use discipline of            face manufactures the illusion of it: the engine does the
a declared query plus a declared freshness bound, with everything                 moving, invisibly, making streaming disappear, as a picture.
else automated [33, 59, 68]. Nobody was fully right. They had the
answer in their grasp and did not see it; we reinvented fragments of
it under new names and repeatedly failed to make the connection.




                                                                           4960
6     IF WE COULD TURN BACK TIME                                               history from a raw log because a node died is economically absurd,
If we could redo the work all over again with what we know now,                which is why serious systems still persist intermediate state.
we would leave some things in, remove others, and maybe relax a                   What changed is that this machinery became commodity, in
bit regarding a few things that otherwise resolved themselves.                 both senses of the word: implemented everywhere, and assem-
                                                                               bled from parts everything else already shared. Popular stream-
                                                                               centric systems accomplish this whether via fine-grained check-
6.1    Leave In                                                                point protocols in MillWheel [5] and its successor Google Cloud
Everything in Section 2: event time, consistency-or-bust, the com-             Dataflow [43], deterministic micro-batch replay in Spark Struc-
pleteness principle. We would also keep unaligned windows. Ses-                tured Streaming [16, 69], or Chandy–Lamport snapshots in Apache
sions were and remain a genuinely important use case, and merge-               Flink [22]. For databases, durable logs provided a floor to rebuild
based window assignment proved the right mechanism for them.                   from, while transactional table storage gave intermediate state a
However, we would keep them on the terms of Section 4: as group-               home that was already snapshot-isolated and durable. In the view-
ing semantics an engine implements (complete with the splitting                maintenance world, intermediate state is tables all the way down,
operation we forgot), not as a centerpiece users program against.              and recovery is just the next refresh reading committed snapshots.
And we would keep the ambition, if not the framing, of engine-                 Fault tolerance stopped being a bespoke protocol each engine had
independence: the claim that semantics must not be dictated by                 to invent and became one more thing from the database toolbox,
the execution engine was correct and foundational, even in light               so it, too, disappeared.
of the overlooked connection that relational models had already                   Notably, this disappearance should be scored as a victory, not
anticipated this idea.                                                         an anticlimax. Exactly-once was only ever a boast because the
                                                                               industry had spent years decommoditizing correctness: no 1980s
6.2    Leave Out                                                               database would have dreamed of advertising that its aggregations
                                                                               didn’t miscount. That we had to say it out loud for so long was the
Most of the trigger language, for the reasons of Section 4. And
                                                                               anomaly. That nobody needs to say it anymore is the ship righted.
user-facing retractions, with heavy emphasis on the qualifier. The
mechanism itself turned out to be triumphantly correct. In the sys-
tems descended from the 2015 model, retractions as a user-visible
protocol were barely implemented and almost never used as de-                  6.4    What We Wish We Had Pushed
signed. We were not even the first to try: Borealis had made revision
                                                                               Remarkably little, it turns out. We expected to fill this bucket gen-
messages (insertions, deletions, replacements) first-class citizens
                                                                               erously, but found our regrets were of other kinds: mechanisms we
of its data model a decade earlier [1], complete with the prolifera-
                                                                               pushed too hard (Section 4) and truths we failed to see (Section 5).
tion problems and opt-out applications that foreshadowed our own
                                                                               What fills it instead is demand: new expectations the world has
protocol’s fate.
                                                                               since placed on the model, each deserving an honest answer rather
   But hidden beneath the declarative surface of the view-main-
                                                                               than a retroactive wish. There are two, and under this paper’s scope
tenance world, retractions are not merely used; they are critical.
                                                                               they turn out to share an answer: both are, chiefly, demands from
Incremental view maintenance is the propagation of deltas and their
                                                                               beyond analytics.
negations through a plan. We find this mechanism in the diffs of
                                                                                  First, the model committed to DAGs and did not include iteration.
Differential Dataflow [49], the retract streams Flink’s SQL planner
                                                                               Back edges were awkward guests in the formalism, so we left them
threads between operators [14], change queries in Snowflake [6],
                                                                               out. The clearest demand since comes from workflow automation,
the deltas of Delta Live Tables’ Enzyme engine [68], and the Z-sets
                                                                               agentic systems most recently: stateful, event-driven, long-running
of DBSP [19]. The idea was entirely right. What was wrong was the
                                                                               programs that react to the world and feed results back into them-
audience: we handed an engine’s mechanism to users and asked
                                                                               selves. That feedback loop is a stream-processing workload with
them to reason about it in the raw. When consistency is the engine’s
                                                                               precisely the back edge we never modeled, and serving it with the
job, the user never needs to see a retraction at all.
                                                                               model today means simulating cycles through external message
   The one place retractions rightly stay user-facing is the stream
                                                                               queues. So, honestly: we missed cycles.
boundary itself: when the stream is the product, as in change data
                                                                                  But cycles are complex for analytics on both sides of the in-
capture, deletes are retractions deliberately surfaced, and nobody
                                                                               terface: the engine needs richer progress tracking than singleton
asks their CDC feed to hide them. In table form, retractions are the
                                                                               watermarks and new semantics for state lifetime, and the author
engine’s business; in stream form, they are the contract. It is the
                                                                               needs to reason recursively, something the average business analyst
paper’s pattern in miniature: right mechanism, wrong altitude.
                                                                               does not reach for on a whim. The analytic appetite is correspond-
                                                                               ingly real but modest (SQL’s recursive common table expressions,
6.3    The Worry That Became Commodity                                         Differential Dataflow’s iterative computations [49]), and the use
We worried a great deal in 2015 about pipelines owning their fault             cases that justify the complexity skew heavily operational. That
tolerance. Strongly consistent state, exactly-once processing, and             demand belongs to the story of Section 7, and while meeting it will
checkpointing protocols were simultaneously headline features,                 take real invention, tracking progress through cycles will not: the
research topics, and competitive differentiators for every engine [5,          flying fixed-point operator was doing so for cyclic stream queries
22]. The core worry remains legitimate: state must survive failure.            in 2009 [24], and Timely Dataflow’s frontiers solved it again in
Recomputable inputs don’t change this math: recomputing a year of              2013 [52]. That much of the territory is charted.




                                                                        4961
   Second is a primitive regularly nominated for our “should have                                     callbacks, retries, dual writes, hand-plumbed consistency between
talked about it” list, one we possessed from the start but declined to                                systems that share no model at all. The reason is structural: each
push: state and timers. MillWheel was state and timers everywhere—                                    component has a coherent internal model, but components interact
manually managed per-key state, explicitly set timers—and the 2015                                    at a lower level (bytes, addresses, packets, connections). The overall
model was largely an attempt to abstract that machinery away.                                         system is only as coherent as the model its parts share, yet the
The primitive was implemented when we wrote the paper, but we                                         modern stack is a mess of incompatible models. Streaming analyt-
lacked consensus to expose it. Looking back, we remain only half                                      ics disappeared into the relational database because the relational
repentant.                                                                                            model is expressive enough to cover its whole domain. For the
   The repentant half: in analytics, the most common use of state                                     entirety of streaming to disappear, and not just analytics, it needs a
and watermark-triggered timers was to access the completion pred-                                     sufficiently general model to disappear into.
icate. This goes back to our earlier premise that windows were pro-                                       What the database supplied was a collapse: the changelog and
moted above their station: many use cases needed to know when                                         the point-in-time relation, recognized as one object, freeing the
data was complete but did not fit cleanly inside the windowing                                        engine to choose the physics beneath a fixed meaning. Nothing
model, so users manually buffered state and relied on watermark                                       about that requires the object to be a relation. A program with a
timers to learn when it was safe to act. The correct answer is the                                    declarative meaning and a streaming-dataflow operational behavior
one from Section 3.3: windowing is for aggregation, not consis-                                       is the same duality over arbitrary computation: what it means is
tency, and generalized finalization predicates are the correct way                                    fixed, how it runs is the engine’s to decide.
to expose completeness semantics in the model.                                                            Realizing this requires generalizing along two dimensions: (1)
   The unrepentant half: most of what state and timers serve is                                       the result: the language must express arbitrary application logic,
operational rather than analytic. The model later incorporated a                                      not just relational queries over tables; and (2) the operations: the
public state and timers API, which has proven enduringly popular;                                     single variable of freshness must expand into a richer envelope of
Flink offers a near-identical API and has built an entire framework                                   latency, durability, priority, and cost.
atop it [13]. Look at what those pipelines do, and two jobs dominate:                                     The payoff is verification and branching: properties an engine
large-scale state machines that are not analytic queries at all, and                                  can establish rather than test at the seams, and system state that is
sinks into other systems.12 Neither job is necessarily beyond declara-                                a value (snapshot-isolated, forkable, replayable). Nor do AI agents
tive analytics (SQL’s MATCH_RECOGNIZE, another type of partitioned                                    make any of this moot: abstraction is what keeps complexity trac-
state machine, comes to mind), but we still feel these constructs are                                 table as a system grows, and the plumbing agents hand-build today
an escape hatch—the assembly language of streaming—rather than                                        is exactly the fragmentation a coherent model erases.
a core part of the analytically minded Dataflow Model; tellingly,                                         This is the direction the work of one of us now points [20, 21];
the view-maintenance generation that delivered on the model’s                                         whether it can be pulled off beyond analytics, we do not yet know.
analytical goals has so far felt no need for an equivalent.                                           But the transformation that overtook analytics looks ready to hap-
   The market, for its part, has since confirmed the diagnosis. As                                    pen wherever software is still assembled by hand from mismatched
the MillWheel era’s workloads dispersed onto newer generations                                        parts. We are curious to see where things stand in another eleven
of systems, analytics settled into view maintenance, while state-                                     years.
ful workflows landed in purpose-built homes of their own: the
durable-execution lineage running from AWS’s Simple Workflow
Service [10] through Uber’s Cadence [66] to Temporal [62], joined                                     8   CONCLUSIONS
from the database side by DBOS [29] and from the streaming side by                                    Did we get anything right? Yes: the physics. Event time, the refusal
Restate [54]. Notably, those systems kept state and timers, wrapped                                   to wait for completeness, and the insistence on consistency were
up in friendlier trappings. What amounted to an escape hatch in                                       correct. These concepts were worth planting a flag over, and the
analytics is a foundational construct in workflow automation, and                                     field is better for having adopted them. No: the interface. Windows
this is where the abstractions have been rising to meet the use case.                                 as a consistency mechanism, triggers, retractions, and the stream
                                                                                                      itself were the wrong things to hand a user. Nearly everything
7     THE LARGER DISAPPEARANCE                                                                        the paper elaborated most lovingly has been abstracted away by
We have argued extensively that streaming analytics is finding a                                      systems that ask for a query and a freshness bound. The part we
happy ending as its complexity disappears into the database, but                                      can neither fully claim nor entirely concede: the destination was
we dare not imply one size fits all [60]. The honest coda is that it                                  in the database literature all along, invented decades earlier, left
disappeared only there. Within analytics, the taming is real: gone are                                materially unfinished by its inventors, materially underappreciated
the days of a small army of developers crafting streaming pipelines,                                  by us, and completed at last by both communities together.
and an expert data analyst now gets their work done with the same                                        One final confession, long owed and long suffered: “Dataflow
ease as any other interaction with the database. But this claim falls                                 Model” was an unfortunate name, borrowed from the product. We
short when applied to streaming generally.                                                            were well aware the name collided with not one but two genera-
   Step outside of analytics and into applications, microservices,                                    tions of prior art on the word: Kildall’s dataflow analysis [37] and
and APIs, and streaming’s complexity reappears at once: queues,                                       DeMarco’s dataflow diagrams [30], among others.
                                                                                                         On another note, had this paper come a little later, it would have
12 In the database world, sinks are typically provided as built-ins; here, users appreciated          ended up the “Beam Model” instead, which is how the commu-
being able to build their own rather than wait for someone else.                                      nity that still fosters the original model now refers to it. No more




                                                                                               4962
illuminating, far less hackle-raising. While preparing this retro-                                MillWheel: Fault-Tolerant Stream Processing at Internet Scale. Proc. VLDB Endow.
spective, we admit, eleven years later we still don’t have a good                                 6, 11 (2013), 1033–1044. https://doi.org/10.14778/2536222.2536229
                                                                                              [6] Tyler Akidau, Paul Barbier, Istvan Cseri, Fabian Hueske, Tyler Jones, Sasha
product-agnostic name for the model.                                                              Lionheart, Daniel Mills, Dzmitry Pauliukevich, Lukas Probst, Niklas Semmler,
    “That Feeling When,” indeed. In the database community, that                                  Dan Sotolongo, and Boyuan Zhang. 2023. What’s the Difference? Incremental
                                                                                                  Processing with Change Queries in Snowflake. Proc. ACM Manag. Data 1, 2,
phrase is a running joke, but as many of you know, there is nonethe-                              Article 196 (2023), 196:1–196:27 pages. https://doi.org/10.1145/3589776
less a certain ache to discovering your hard-earned “invention” is                            [7] Tyler Akidau, Edmon Begoli, Slava Chernyak, Fabian Hueske, Kathryn Knight,
the punchline. With time, however, the ache mellows into respect                                  Kenneth Knowles, Daniel Mills, and Dan Sotolongo. 2021. Watermarks in Stream
                                                                                                  Processing Systems: Semantics and Comparative Analysis of Apache Flink and
and gratitude, and we could not be happier to see these ideas at                                  Google Cloud Dataflow. Proc. VLDB Endow. 14, 12 (2021), 3135–3147. https:
work in so many places, regardless of their provenance.                                           //doi.org/10.14778/3476311.3476389
    May the next decade’s builders read from as many communities                              [8] Tyler Akidau, Robert Bradshaw, Craig Chambers, Slava Chernyak, Rafael J.
                                                                                                  Fernández-Moctezuma, Reuven Lax, Sam McVeety, Daniel Mills, Frances Perry,
as possible. May their reviewers be as merciless as ours. And may                                 Eric Schmidt, and Sam Whittle. 2015. The Dataflow Model: A Practical Approach
they feel the joy and pride of seeing their creation adopted as                                   to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-
                                                                                                  of-Order Data Processing. Proc. VLDB Endow. 8, 12 (2015), 1792–1803. https:
broadly as ours was, minus the discovery that something simpler                                   //doi.org/10.14778/2824032.2824076
was there all along.                                                                          [9] Tyler Akidau, Slava Chernyak, and Reuven Lax. 2018. Streaming Systems: The
                                                                                                  What, Where, When, and How of Large-Scale Data Processing. O’Reilly Media.
                                                                                             [10] Amazon Web Services. 2012. Amazon Simple Workflow Service. https://aws.
ACKNOWLEDGMENTS                                                                                   amazon.com/swf/.
Our deepest thanks to Atul Adya, for the 2015 review that gave                               [11] Apache Beam. 2026. Schemas — Beam Programming Guide. https://beam.apache.
                                                                                                  org/documentation/programming-guide/#what-is-a-schema.
the paper its heart and for embodying, entirely unwittingly, this                            [12] Apache Flink. 2024. Materialized Tables (FLIP-435). https://nightlies.apache.org/
paper’s thesis: that the database community had the foundational                                  flink/flink-docs-release-1.20/docs/dev/table/materialized-table/overview/.
answers all along, and we failed to recognize them even when they                            [13] Apache Flink. 2026. Stateful Functions. https://nightlies.apache.org/flink/flink-
                                                                                                  statefun-docs-master/.
were standing right next to us. (Tyler literally knew Atul for 16                            [14] Apache Flink. 2026. Table API. https://nightlies.apache.org/flink/flink-docs-
years before finally making the connection he was that transaction                                stable/docs/dev/table/tableapi/.
                                                                                             [15] Arvind Arasu, Shivnath Babu, and Jennifer Widom. 2006. The CQL Continuous
isolation guy. The shame!) To Colin Meek for lending his deep and                                 Query Language: Semantic Foundations and Query Execution. The VLDB Journal
thoughtful review to the more technical corners of the original pa-                               15, 2 (2006), 121–142. https://doi.org/10.1007/s00778-004-0147-z
per, and for his early feedback on this retrospective. To David Maier,                       [16] Michael Armbrust, Tathagata Das, Joseph Torres, Burak Yavuz, Shixiong Zhu,
                                                                                                  Reynold Xin, Ali Ghodsi, Ion Stoica, and Matei Zaharia. 2018. Structured Stream-
on whose work we stood more than we realized, and whose insights                                  ing: A Declarative API for Real-Time Applications in Apache Spark. In Proceedings
sharpened this piece. To Saurabh Bansal, Kory Kraft, Yaroslav Litus,                              of the 2018 International Conference on Management of Data (SIGMOD). 601–613.
and Bill Neubauer for their observations and suggestions. To Dan                                  https://doi.org/10.1145/3183713.3190664
                                                                                             [17] Roger S. Barga, Jonathan Goldstein, Mohamed H. Ali, and Mingsheng Hong. 2007.
Sotolongo, fellow traveler from triggers to tables, now bearing the                               Consistent Streaming Through Time: A Vision for Event Stream Processing. In
torch with Daniel toward One Model to Rule Them All, for his                                      Proceedings of the Third Biennial Conference on Innovative Data Systems Research
                                                                                                  (CIDR).
gracious feedback on this paper. To Grzegorz Czajkowski, who                                 [18] Edmon Begoli, Tyler Akidau, Fabian Hueske, Julian Hyde, Kathryn Knight, and
backed the original work when it mattered most, costing us only                                   Kenneth Knowles. 2019. One SQL to Rule Them All: An Efficient and Syntactically
our pride in branded naming rights. To Paris Carbone for a generous                               Idiomatic Approach to Management of Streams and Tables. In Proceedings of
                                                                                                  the 2019 International Conference on Management of Data (SIGMOD). 1757–1772.
retrospective; your footnote on the VLDB slide deck made Tyler’s                                  https://doi.org/10.1145/3299869.3314040
year, and your shout-out to our what/where/when/how schtick                                  [19] Mihai Budiu, Tej Chajed, Frank McSherry, Leonid Ryzhyk, and Val Tannen. 2023.
was pure gold. Thanks to our original co-authors: Robert Bradshaw,                                DBSP: Automatic Incremental View Maintenance for Rich Query Languages. Proc.
                                                                                                  VLDB Endow. 16, 7 (2023), 1601–1614. https://doi.org/10.14778/3587136.3587137
Craig Chambers, Slava Chernyak, Sam McVeety, Frances Perry, Eric                             [20] Cambra. 2026. Composition Shouldn’t Be This Hard. https://cambra.dev/blog/
Schmidt, and Sam Whittle. Thanks to the MillWheel, FlumeJava,                                     announcement.
                                                                                             [21] Cambra. 2026. The System as a Program. https://cambra.dev/blog/the-system-
and Cloud Dataflow teams, and to the Apache Beam community                                        as-a-program.
who carried this work into the world. And a sincere thank you to                             [22] Paris Carbone, Asterios Katsifodimos, Stephan Ewen, Volker Markl, Seif Haridi,
the VLDB community for over a decade of engagement and for the                                    and Kostas Tzoumas. 2015. Apache Flink: Stream and Batch Processing in a
                                                                                                  Single Engine. IEEE Data Engineering Bulletin 38, 4 (2015), 28–38.
honor.                                                                                       [23] Badrish Chandramouli, Jonathan Goldstein, Mike Barnett, Robert DeLine, Danyel
                                                                                                  Fisher, John C. Platt, James F. Terwilliger, and John Wernsing. 2014. Trill: A
REFERENCES                                                                                        High-Performance Incremental Query Processor for Diverse Analytics. Proc.
                                                                                                  VLDB Endow. 8, 4 (2014), 401–412. https://doi.org/10.14778/2735496.2735503
 [1] Daniel J. Abadi, Yanif Ahmad, Magdalena Balazinska, Uğur Çetintemel, Mitch              [24] Badrish Chandramouli, Jonathan Goldstein, and David Maier. 2009. On-the-Fly
     Cherniack, Jeong-Hyon Hwang, Wolfgang Lindner, Anurag S. Maskey, Alexander                   Progress Detection in Iterative Stream Queries. Proc. VLDB Endow. 2, 1 (2009),
     Rasin, Esther Ryvkina, Nesime Tatbul, Ying Xing, and Stan Zdonik. 2005. The                  241–252. https://doi.org/10.14778/1687627.1687655
     Design of the Borealis Stream Processing Engine. In Proceedings of the Second           [25] Sirish Chandrasekaran, Owen Cooper, Amol Deshpande, Michael J. Franklin,
     Biennial Conference on Innovative Data Systems Research (CIDR). 277–289.                     Joseph M. Hellerstein, Wei Hong, Sailesh Krishnamurthy, Samuel R. Madden,
 [2] Atul Adya, Phil Bogle, and Colin Meek. 2025. Understanding the Limitations of                Vijayshankar Raman, Frederick Reiss, and Mehul A. Shah. 2003. TelegraphCQ:
     Pubsub Systems. In Proceedings of the 2025 Workshop on Hot Topics in Operating               Continuous Dataflow Processing for an Uncertain World. In Proceedings of the
     Systems (HotOS). 165–171. https://doi.org/10.1145/3713082.3730397                            First Biennial Conference on Innovative Data Systems Research (CIDR). https:
 [3] Atul Adya, Barbara Liskov, and Patrick E. O’Neil. 2000. Generalized Isolation                //cidrdb.org
     Level Definitions. In Proceedings of the 16th International Conference on Data          [26] Guoqiang Jerry Chen, Janet L. Wiener, Shridhar Iyer, Anshul Jaiswal, Ran
     Engineering (ICDE). 67–78. https://doi.org/10.1109/ICDE.2000.839388                          Lei, Nikhil Simha, Wei Wang, Kevin Wilfong, Tim Williamson, and Serhat
 [4] Yanif Ahmad, Oliver Kennedy, Christoph Koch, and Milos Nikolic. 2012.                        Yilmaz. 2016. Realtime Data Processing at Facebook. In Proceedings of the
     DBToaster: Higher-Order Delta Processing for Dynamic, Frequently Fresh Views.                2016 International Conference on Management of Data (SIGMOD). 1087–1098.
     Proc. VLDB Endow. 5, 10 (2012), 968–979. https://doi.org/10.14778/2336664.                   https://doi.org/10.1145/2882903.2904441
     2336670                                                                                 [27] Databricks Inc. 2022. Simplifying Change Data Capture with Databricks Delta
 [5] Tyler Akidau, Alex Balikov, Kaya Bekiroğlu, Slava Chernyak, Josh Haberman,                   Live Tables. https://www.databricks.com/blog/2022/04/25/simplifying-change-
     Reuven Lax, Sam McVeety, Daniel Mills, Paul Nordstrom, and Sam Whittle. 2013.                data-capture-with-databricks-delta-live-tables.html.




                                                                                      4963
[28] Databricks Inc. 2026. Triggered vs. Continuous Pipeline Mode. https://docs.                        Locking and Partial Rollbacks Using Write-Ahead Logging. ACM Transactions
     databricks.com/aws/en/ldp/pipeline-mode. Documentation for Delta Live Tables,                      on Database Systems 17, 1 (1992), 94–162. https://doi.org/10.1145/128765.128770
     since renamed Lakeflow Declarative Pipelines.                                                 [52] Derek G. Murray, Frank McSherry, Rebecca Isaacs, Michael Isard, Paul Barham,
[29] DBOS Inc. 2026. DBOS. https://www.dbos.dev.                                                        and Martín Abadi. 2013. Naiad: A Timely Dataflow System. In Proceedings
[30] Tom DeMarco. 1979. Structured Analysis and System Specification. Yourdon                           of the 24th ACM Symposium on Operating Systems Principles (SOSP). 439–455.
     Press.                                                                                             https://doi.org/10.1145/2517349.2522738
[31] Rafael J. Fernández-Moctezuma, Kristin Tufte, and Jin Li. 2009. Inter-Operator                [53] Shadi A. Noghabi, Kartik Paramasivam, Yi Pan, Navina Ramesh, Jon Bringhurst,
     Feedback in Data Stream Management Systems via Punctuation. In Proceedings                         Indranil Gupta, and Roy H. Campbell. 2017. Samza: Stateful Scalable Stream
     of the Fourth Biennial Conference on Innovative Data Systems Research (CIDR).                      Processing at LinkedIn. Proc. VLDB Endow. 10, 12 (2017), 1634–1645. https:
[32] Google Cloud. 2025. Time Series Functions. https://docs.cloud.google.com/                          //doi.org/10.14778/3137765.3137770
     bigquery/docs/reference/standard-sql/time-series-functions#changes.                           [54] Restate. 2026. Restate. https://restate.dev.
[33] Google Cloud. 2026.          BigQuery: Manage Materialized View Staleness                     [55] Matthias J. Sax, Guozhang Wang, Matthias Weidlich, and Johann-Christoph
     (max_staleness). https://cloud.google.com/bigquery/docs/materialized-views-                        Freytag. 2018. Streams and Tables: Two Sides of the Same Coin. In Proceedings
     create.                                                                                            of the International Workshop on Real-Time Business Intelligence and Analytics
[34] Google Cloud. 2026. Introduction to Continuous Queries. https://docs.cloud.                        (BIRTE). 1–10. https://doi.org/10.1145/3242153.3242155
     google.com/bigquery/docs/continuous-queries-introduction.                                     [56] Richard T. Snodgrass and Ilsoo Ahn. 1985. A Taxonomy of Time in Databases. In
[35] Ashish Gupta, Inderpal Singh Mumick, and V. S. Subrahmanian. 1993. Maintain-                       Proceedings of the 1985 ACM SIGMOD International Conference on Management
     ing Views Incrementally. In Proceedings of the 1993 ACM SIGMOD International                       of Data. 236–246. https://doi.org/10.1145/318898.318921
     Conference on Management of Data. 157–166. https://doi.org/10.1145/170035.                    [57] Snowflake Inc. 2026. Dynamic Tables. https://docs.snowflake.com/en/user-
     170066                                                                                             guide/dynamic-tables-about.
[36] Julian Hyde. 2016. Streams, Joins, and Temporal Tables. https://s.apache.org/                 [58] Snowflake Inc. 2026. Frozen Regions and Backfill. https://docs.snowflake.com/
     streams-joins-and-temporal-tables.                                                                 en/user-guide/dynamic-tables/frozen-regions.
[37] Gary A. Kildall. 1973. A Unified Approach to Global Program Optimization. In                  [59] Daniel Sotolongo, Daniel Mills, Tyler Akidau, Anirudh Santhiar, Attila-Péter Tóth,
     Proceedings of the 1st Annual ACM SIGACT-SIGPLAN Symposium on Principles of                        Ilaria Battiston, Ankur Sharma, Botong Huang, Boyuan Zhang, Dzmitry Pauliuke-
     Programming Languages (POPL). 194–206. https://doi.org/10.1145/512927.512945                       vich, Enrico Sartorello, Igor Belianski, Ivan Kalev, Lawrence Benson, Leon Papke,
[38] Martin Kleppmann. 2016. Making Sense of Stream Processing. O’Reilly Media.                         Ling Geng, Matt Uhlar, Nikhil Shah, Niklas Semmler, Olivia Zhou, Saras Nowak,
[39] Jay Kreps. 2013. The Log: What Every Software Engineer Should Know about                           Sasha Lionheart, Till Merker, Vlad Lifliand, Wendy Grus, Yi Huang, and Yiwen
     Real-Time Data’s Unifying Abstraction. https://engineering.linkedin.com/                           Zhu. 2025. Streaming Democratized: Ease Across the Latency Spectrum with
     distributed-systems/log-what-every-software-engineer-should-know-about-                            Delayed View Semantics and Snowflake Dynamic Tables. In Companion of the
     real-time-datas-unifying.                                                                          2025 International Conference on Management of Data (SIGMOD-Companion).
[40] Jay Kreps. 2014. Questioning the Lambda Architecture. https://www.oreilly.com/                     https://doi.org/10.1145/3722212.3724455
     radar/questioning-the-lambda-architecture/.                                                   [60] Michael Stonebraker and Uğur Çetintemel. 2005. “One Size Fits All”: An Idea
[41] Krishna Kulkarni and Jan-Eike Michels. 2012. Temporal Features in SQL:2011.                        Whose Time Has Come and Gone. In Proceedings of the 21st International Confer-
     ACM SIGMOD Record 41, 3 (2012), 34–43. https://doi.org/10.1145/2380776.                            ence on Data Engineering (ICDE). IEEE, 2–11. https://doi.org/10.1109/ICDE.2005.1
     2380786                                                                                       [61] Kanat Tangwongsan, Martin Hirzel, Scott Schneider, and Kun-Lung Wu. 2015.
[42] Wilburt Labio and Hector Garcia-Molina. 1996. Efficient Snapshot Differen-                         General Incremental Sliding-Window Aggregation. Proc. VLDB Endow. 8, 7 (2015),
     tial Algorithms for Data Warehousing. In Proceedings of the 22nd International                     702–713. https://doi.org/10.14778/2752939.2752940
     Conference on Very Large Data Bases (VLDB). 63–74.                                            [62] Temporal Technologies. 2019. Temporal. https://temporal.io.
[43] Reuven Lax. 2017. After Lambda: Exactly-Once Processing in Google Cloud                       [63] Pınar Tözün, Viktor Leis, Anja Gruenheid, Paris Carbone, and Eleni Tzirita
     Dataflow. https://cloud.google.com/blog/products/data-analytics/after-lambda-                      Zacharatou. 2025. Reminiscences on Influential Papers. ACM SIGMOD Record
     exactly-once-processing-in-google-cloud-dataflow-part-1.                                           54, 3 (2025), 22–27. https://doi.org/10.1145/3774303.3774307
[44] Jin Li, David Maier, Kristin Tufte, Vassilis Papadimos, and Peter A. Tucker. 2005.            [64] Peter A. Tucker, David Maier, Tim Sheard, and Leonidas Fegaras. 2003. Ex-
     Semantics and Evaluation Techniques for Window Aggregates in Data Streams.                         ploiting Punctuation Semantics in Continuous Data Streams. IEEE Trans-
     In Proceedings of the 2005 ACM SIGMOD International Conference on Management                       actions on Knowledge and Data Engineering 15, 3 (2003), 555–568. https:
     of Data. 311–322. https://doi.org/10.1145/1066157.1066193                                          //doi.org/10.1109/TKDE.2003.1198390
[45] Jin Li, Kristin Tufte, Vladislav Shkapenyuk, Vassilis Papadimos, Theodore John-               [65] Peter A. Tucker, Kristin Tufte, Vassilis Papadimos, and David Maier. 2002.
     son, and David Maier. 2008. Out-of-Order Processing: A New Architecture for                        NEXMark—A Benchmark for Queries over Data Streams (Draft). Technical Report.
     High-Performance Stream Systems. Proc. VLDB Endow. 1, 1 (2008), 274–288.                           OGI School of Science & Engineering at OHSU.
     https://doi.org/10.14778/1453856.1453890                                                      [66] Uber Technologies. 2017. Cadence Workflow. https://cadenceworkflow.io.
[46] Nathan Marz. 2011. How to Beat the CAP Theorem. http://nathanmarz.com/                        [67] Juliane Verwiebe, Philipp M. Grulich, Jonas Traub, and Volker Markl. 2023. Survey
     blog/how-to-beat-the-cap-theorem.html.                                                             of Window Types for Aggregation in Stream Processing Systems. The VLDB
[47] Materialize Inc. 2026. Isolation Levels. https://materialize.com/docs/reference/                   Journal 32, 5 (2023), 985–1011. https://doi.org/10.1007/s00778-022-00778-6
     isolation-level/.                                                                             [68] Ritwik Yadav, Supun Abeysinghe, Min Yang, Jeffrey Helt, Manuel Ung, Yuhong
[48] Frank McSherry, Andrea Lattuada, Malte Schwarzkopf, and Timothy Roscoe. 2020.                      Chen, Melody Hu, William Wei, Yiming Yang, Tom van Bussel, Sourav Chat-
     Shared Arrangements: Practical Inter-Query Sharing for Streaming Dataflows.                        terji, Indrajit Roy, Paul Lappas, Yannis Papakonstantinou, Tahir Fayyaz, Bilal
     Proc. VLDB Endow. 13, 10 (2020), 1793–1806. https://doi.org/10.14778/3401960.                      Aslam, Ross Bunker, Michael Armbrust, and Shrikanth Shankar. 2026. Enzyme:
     3401974                                                                                            Incremental View Maintenance for Data Engineering. In Companion of the
[49] Frank McSherry, Derek G. Murray, Rebecca Isaacs, and Michael Isard. 2013.                          2026 International Conference on Management of Data (SIGMOD-Companion).
     Differential Dataflow. In Proceedings of the Sixth Biennial Conference on Innovative               https://doi.org/10.1145/3788853.3803098
     Data Systems Research (CIDR).                                                                 [69] Matei Zaharia, Tathagata Das, Haoyuan Li, Timothy Hunter, Scott Shenker, and
[50] Erik Meijer. 2012. Your Mouse Is a Database. Commun. ACM 55, 5 (May 2012),                         Ion Stoica. 2013. Discretized Streams: Fault-Tolerant Streaming Computation at
     66–73. https://doi.org/10.1145/2160718.2160735                                                     Scale. In Proceedings of the 24th ACM Symposium on Operating Systems Principles
[51] C. Mohan, Don Haderle, Bruce Lindsay, Hamid Pirahesh, and Peter Schwarz.                           (SOSP). 423–438. https://doi.org/10.1145/2517349.2522737
     1992. ARIES: A Transaction Recovery Method Supporting Fine-Granularity




                                                                                            4964

