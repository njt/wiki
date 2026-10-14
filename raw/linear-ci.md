---
url: https://starsling.dev/blog/linear-ci
date_fetched: 2026-10-14
---

A few weeks ago Linear published a blog post about how they reworked their CI. Their optimizations led to a 15% faster test suite and 50% reduction in runner-time spent per test, while their test suite grew 4x in size. The post was popular on X and Hacker News, and has a lot of technical depth (with pretty graphics).

We built a skill called ci-speedup that now has 70+ CI optimizations, including Linear's, that your agent can use to make your GitHub Actions faster. This post is a remix of Linear's post, for agents. We tied each optimization they detailed to a pattern in the skill so you can copy a prompt for each optimization and have your agent implement it on your own repo.

Just send this post to your agent and they'll work through all 15 of Linear's optimizations, then suggest installing ci-speedup to check for 60+ more. You can find ci-speedup on GitHub and on skills.sh.

StarSling is agent-native CI: fast GitHub Actions runners with AI agents that keep finding and shipping optimizations like the ones in this post, while you sleep. It's trusted by AI-first teams like Mastra and Better Auth (Vercel). Install it for free and check out our docs.

In our pursuit to make CI more performant at Linear, we optimized for how long a PR waits on CI and how much runner time it consumes. Despite our test suites almost quadrupling since the start of the year, we brought pull request wait time down from more than 6 minutes to just over 5, while cutting runner time per test roughly in half.

Broadly, we improved CI in four ways:

Upgraded infrastructure and tooling

Optimized the jobs that gate other work

Reduced repeated setup

Made test execution more efficient

Linear’s codebase is primarily TypeScript, but many of these optimizations apply across languages and toolchains.

Some of our earliest gains required almost no optimization of CI itself. Moving our workloads off GitHub Actions to third-party runners with faster CPUs, higher-performance storage, and better cache infrastructure gave us faster machines to run the same pipeline on. In a like-for-like comparison of the two days either side of the switch, jobs ran 34% faster on average, with some workloads like tsc dropping 52%.

Disclosure: We make StarSling Runners, one of the two options OPT81 can suggest benchmarking (the other is a larger GitHub-hosted runner). If you want faster runners without changing anything else, try them on the job this finding names.

Separately, modernizing our toolchain also paid off. Switching to tsgo, the native TypeScript compiler, cut the weekly median of the tsc check by 73%, large enough to move the bottleneck off of typechecking entirely.

Linting was another early target. A handful of our custom lint rules depended on TypeScript type information, either to enforce a restriction or apply an autofix. That meant every lint run had to build the full type graph before evaluating those rules, making linting one of our most memory-intensive CI jobs.

We rewrote the rules to use static analysis over the abstract syntax tree, identifying function-like constructs and guard patterns without type information. That let ESLint drop TypeScript entirely, reducing API lint time by 68%, and full-repository lint time by 55%. Memory usage dropped substantially as well.

Removing the dependency on type information also made our later move to Oxlint much easier because rules that operate purely on syntax are straightforward to port. Oxlint itself reduced the CI runner-minutes spent on linting.

With the underlying infrastructure and individual checks running faster, we zoomed out to look at CI as a system. That drew our attention to the small jobs that sat in front of everything else. Every run starts by checking which paths a PR touched and whether these tests have already passed for the same inputs. We gate on those checks at the job level so skipped work never reserves a runner, but that also puts them directly on the critical path. None of the eight API test shards can start until they finish, making even small delays disproportionately important.

Several of our workflows start with a change-detection job that decides what runs next; for instance, it checks whether a diff contains a database migration and outputs a signal used to schedule the relevant database CI checks. These jobs were checking out the full working tree even though they needed only a small subset of it. We capped the fetch depth, which took the slowest of these gates from 94 seconds to 20, and removed checkout entirely from the jobs that never needed a working tree, reducing time spent on those from 27 seconds to 7. For commit push and merge-queue events, where we do have to diff paths, we found that a sparse, blobless checkout with limited history was enough, saving another 11 odd seconds.

After we swapped the underlying runner infrastructure, we noticed that our checkout times (with actions/checkout) in our jobs had gotten longer and would sometimes hang. Because the third-party runners sit outside GitHub’s network, they rely on a direct IP link to reach GitHub. The provider traced the hangs to intermittent degradation on that link. Several of our workflows begin with a checkout, so a stalled fetch could delay the entire CI run.

To be resilient to the network instability, we replaced actions/checkout with a composite action of our own that retried with backoff, and sets GIT_HTTP_LOW_SPEED_LIMIT and GIT_HTTP_LOW_SPEED_TIME so a stalled connection aborts after about 30 seconds instead of hanging and also uses the checkout cache, which keeps a persistent git mirror on a sticky disk. The result was far fewer runs where a critical-path job sat idle waiting for checkout to finish.

Not every job on the critical path needed to be there. We were writing cache markers as part of the final check before merging, which meant a pull request could sit in the merge queue even after its tests had passed. We moved that write into a job that runs once the test shards finish but gates nothing, shaving 42 seconds from the merge path for every API pull request and merge-queue entry.

Together, these changes took roughly a minute off the required check for API pull requests on cache misses, while also reducing runner starts.

From there, we turned to the setup cost repeated across every job, like booting a runner, installing packages, and provisioning build dependencies. That overhead means a job that does only seconds of useful work can end up consuming whole minutes of infrastructure time. Here are a few steps we took to work around that issue:

Our API test shards each spent 7 to 8 seconds installing the same Postgres client with apt on every run. We moved it into a small CI base image containing Node and the client, so each shard could start from an environment that was ready to run. We later added the required native build headers to the image after discovering that downloading them during setup could occasionally hang, shortening the tail.

Linear’s codebase is a monorepo managed as a pnpm workspace. Our API test workflow was installing the entire workspace even though it only needed the API package and its dependencies. Restricting the install to our API package cut pnpm install from 44-73 seconds to 16-18 seconds. We applied the same pattern to API-adjacent jobs, which were each installing the full repository and uploading a dependency cache that later runs almost never hit.

We also tested caching node_modules and found it was faster to rebuild. The cache key depended on a frequently changing lockfile, and even a cache hit took about 28 seconds to restore, compared with roughly 7.5 seconds for a filtered install. The cache was adding save time and variability without giving us any discernible advantage.

Together, these three changes reduced per-shard setup time by roughly 44%, from 110-140 seconds to 67-73 seconds.

Beyond this, there were other forms of repeated setup we could avoid altogether.

Some setup work only needs to be repeated when its inputs change. Our API containers, for example, were replaying the full database migration history on every run, even when a PR hadn’t changed the schema. For those cases, we switched to loading a generated schema snapshot and bootstrap file instead, cutting database setup from roughly 12 seconds to 1-2 seconds per container.

Seven independent checks were each starting a runner, checking out the repository, and installing dependencies before doing only seconds of useful work. We consolidated them into two jobs, and then ran the seven tasks concurrently inside them. That reduced the number of times we paid the same setup overhead from seven to two. Based on June usage, the change saved roughly 87,000 runner-minutes per month, equivalent to 11.8% of our total CI usage.

With the fixed cost of each test shard down, we could afford to parallelize the API suite more aggressively. It was the largest and one of the most frequently executed parts of our workflow, so improvements there had an outsized effect on merge time.

Vitest, the test runner we use for our TypeScript test suites, distributes work by file rather than by the duration of individual tests. That meant a few unusually large test files could dominate a shard and effectively hold up completion of the entire suite, even when the other shards finished much earlier.

We split those large files into smaller, more focused files while preserving the structure of the tests, then evaluated different shard and runner configurations. We had already gone from three to four shards earlier in the year; moving to eight made the critical job roughly 19% faster and 19% cheaper in our initial benchmark. A week after the change, the slowest shard dropped from 5.25 minutes to 4.33 minutes.

Share module state only with strict isolation rules

Vitest normally isolates every test file, which for us meant rebuilding the entity, GraphQL, and decorator graph in each test shard. We introduced an opt-in vitest project with isolate: false, allowing safe files to share a module registry within each worker.

This was our largest single performance improvement, worth roughly 17% in monthly savings at our volume. The slowest shard fell from roughly 300-379 seconds to about 195 seconds, while total API-shard runner time dropped from about 32.8 to 22 minutes per run.

It was also the optimization with the highest correctness risk. We made eligibility explicit with an opt-in comment on every file, and added the necessary teardown for shared state. A handful of files used fake timers or shared state in ways we couldn’t untangle safely, so we left them in the isolated project. And because agents now write the majority of our tests, we updated our respective agent skills to account for this performance opt-in as well, so generated tests follow the same constraints by default.

Further sharding only pays off when the fixed cost per shard is low, since doubling the shard count also doubles the workflow time spent on setup. The setup optimizations we referred to earlier are what made eight shards practical. At 110-140 seconds per shard, eight shards would have spent 15-19 minutes of runner time on setup alone, more than the tests themselves. Setup is now around 40 seconds, so eight shards spend less total setup time than four did before, while parallelizing the tests twice as far.

This change follows from three earlier ones, “Preinstall shared dependencies in the CI image”, “Install only the dependencies each job needs” and “Don’t cache when it’s faster to rebuild”: once each shard’s setup shrinks, more shards pay.

Prompt for your agent

Improvements that compound across a system

Had we not made a deliberate effort to improve CI earlier this year, today’s test suite would take roughly 11 minutes, close to double what developers wait now. And the work doesn’t end here. It’s clear that our codebase will continue to grow; we’re currently adding roughly 2,000 tests a week. Keeping CI fast as that happens will be a continued effort, much of it using what we learned through this process to new bottlenecks.

Linear ends where every team doing this work ends: it is never finished. Their test suite keeps growing, and they expect to keep applying what they learned to whatever the next bottleneck turns out to be. That is the job ci-speedup is built for. Run it again whenever CI slows down, and it checks your latest runs against its whole catalog, Linear's included, and names the next bottleneck with numbers from your own history. If you would rather not run that loop yourself, StarSling Runners come with agents that keep opening PRs to make your runs faster.
