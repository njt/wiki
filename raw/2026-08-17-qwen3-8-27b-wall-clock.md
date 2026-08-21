---
url: https://overbring.com/blog/2026-08-17-qwen3-8-27b-wall-clock/
date_fetched: 2026-08-21
---

*With Qwen3.8-27B, wall-clock time to a correct result now matters more than ever than tokens per second. A “slower” model that finishes unattended beats a faster model that keeps you in the loop. Ignore the current hot takes about  xhigh being wasteful, chat templates being suboptimal, 8-bit quants being massively better than 4-bit, and never quantizing the KV cache. This is SOTA at home, and 32 GB of VRAM has never before been more useful.*

The long-awaited Qwen3.8-27B was released on Friday, and it’s now Monday. I’ve had three days to test this thing out on actual codebases, primarily on those of Breek.gr , which is a mature SaaS, as well as on other codebases both for upcoming product launches by OVERBRING Labs and on internal business tools that serve my other brand, ISATEK , and on the upcoming redesign of isatek.gr .

Like many of us enthusiastic users of Qwen3.6-35B-A3B
 and Qwen3.6-27B
 and their respective finetunes (like KAT-Coder-V2.5-Dev
 and ThinkingCap-Qwen3.6-27B
), I spent the past week waiting for *“Qwensmas”* on Friday. In the first few hours of the open-weights drop on HuggingFace, and while tuning the `llama-server`
 invocations in the `config.yaml`
 of the `llama-swap`
 running on `supra`, I listened to and watched various early takes on the model, devoured YouTube video comments and HackerNews comments and came across *many* hot takes and opinions that do not “click” with my impressions from the last 3 months of agentic coding.


`supra`has an X570 AORUS ELITE motherboard with a Ryzen 7 5700X, 64 GB of DDR4-3200, a 1000W 80 Plus Gold PSU, and two RTX 5060 Ti 16 GB.

Mainly, I’d like to tackle one issue that trips people up, and reveals issues with how the models are being “benchmarked” (a true euphemism) on YouTube videos, and deployed locally. You see, the model was released with an extra-high (`xhigh`) reasoning effort setting being the default. *“Oh, it’s slow”*, *“it takes ages to respond”*, *“it overthinks”* are things you’ll find posted as criticisms in comments on videos, as often as you find the opposite views, i.e. that it’s incredible, it approaches cloud-hosted “State Of The Art” (“SOTA”)/frontier models, etc.

Both are correct, but only the latter is *objectively* correct when compared to what we’ve had access to until now. Subjectively, it’s not very fast in terms of `pp`/`tg`, and it *does* take long to respond after very long bouts of reasoning. Sure, it’s not the *fastest* model when you run it on budget hardware like mine with layer-splitting (`--split-mode layer`) on an AM4 motherboard (X570) with 1x PCIe Gen 4.0 x8 and 1x PCIe Gen 4.0 x4 (PCH-connected). We’re talking max. 45 tok/s for token generation, with `--spec-type draft-mtp` and `--spec-draft-n-max 2` (graphs follow below).

Objectively, it’s way, *way*, **way** more intelligent than anything else you can run locally on a budget right now. And it’s not intelligent only across the actual benchmarks, despite baffling YouTube video comments claiming that it’s “benchmaxxed”. I don’t see it (empirical evidence later); Qwen 3.5 and 3.6 weren’t benchmaxxed either.

So let us rejoice! Qwen3.8-27B delivers what the Qwen team didn’t even promise, to begin with: something that truly approaches what was SOTA/frontier back in February 2026. It runs locally, and ideally (for maximum or close-to-maximum context window) on 32 GB of VRAM (but still, without the `mmproj` / “vision tower”).

## Claims of “overthinking”, and the YouTube “benchmarking” problem

Does it “overthink”? *Jein*, as the Germans say. **Yes**, “out of the box”, it “thinks” *a lot*. Its reasoning traces end up consuming a large part of the context window, especially with the `--reasoning-preserve` flag of `llama-server` enabled.

But also **no**, it doesn’t *over*-think, even though the `xhigh` setting is neither the best default nor necessary for most tasks. Various serious YouTubers and X posters have by now analyzed this more thorougly and reached this conclusion.

However, this is ultimately not the true metric, and neither are `pp`/`tg` metrics the ultimate benchmarking metrics for agentic coding, because you can complete a task successfully in either of two extreme “modes” (i.e. at some target quality at which you consider the task “done and dusted”):

- At a high average `pp`/`tg`, across multiple rounds that are required both for the model to fix its own bugs and for human guidance of*rework*that keeps you in the loop until actual completion of the overarching task. That has been the case so far for my tasks with Qwen3.6-35B-A3B, either as a single agent, or as the main agent that orchestrates ThinkingCap subagents sequentially for coding, adversarial review, corrections, and the pre-commit use of Alibaba’s`open-code-review`on the code changes.
- At a low average `pp`/`tg`, where the model works autonomously until completion while you go do something else.

For a given target quality, the metric that really matters is **wall-clock time**. For us local-AI aficionados, this is directly linked to our monthly power bill. If you (like I do) are working on multiple things in parallel (not necessarily coding, but also running a business and needing to be away from the agent harness again and again), mode (2) is clearly preferable, even if wall-clock time is longer. In fact, mode (2) is then preferable even if it takes double the wall-clock time of mode (1). You are then context-switching less frequently.

The main benefit of mode (2) compared to the single-agent or multi-agent approach of mode (1), besides the increased intelligence of Qwen3.8-27B, is that you can walk away from it while it works, and come back to something that is pretty much close to “perfect”.

### Sidenote: quantmaxxing and illusions of equipment superiority

Let’s not start a debate here about whether Unsloth’s `UD-Q4_K_XL` quant is capable of that, whether you need `Q8_0` or even full-precision GGUFs to reach this “perfection”. My mileage may vary from that of someone who runs this across two 5090s or on a DGX Spark. So will our CapEx (GPUs) and OpEx (electricity), i.e. our break-even period on the investment. Everyone chooses what works best for their own economics and needs. What seems frivolous for me might be necessary for your use cases. No judgment there. It’s the blind cargo-culted dogma that annoys me.

For my needs, `UD-Q4_K_XL` worked truly *perfectly* on the first task I gave it, that’s why I keep that GGUF alongside AtomicChat’s `Q5_K_M` and `AD-Q6_K`. Those quants for sure don’t afford me the full 262144-token native context window, but if you take into account pursuing mode (1) with Qwen3.8-27B as a subagent, that’s totally fine. Maybe there are tasks for which a 6-bit quant cannot do what an 8-bit quant or even a full-precision GGUF can do for, but I cannot know, as I have no such case, nor the hardware to run it. As a trained engineer who runs a business, I’m deeply, truly a *satisficer*.

But here’s where we need to take a closer look at the hot takes and the wrong takes and misunderstandings and incomplete investigations that arose in the past 3 days, primarily from people rushing to post something on YouTube and get the views.

## Serving Qwen3.8-27B on 32 GB of VRAM

On Friday, August 14 at 18:00 Athens time, the Qwen3.8-27B weights were released on HuggingFace. Unsloth did their thing and uploaded their quant ladder. By midnight I was back in front of the keyboard and had downloaded `UD-Q4_K_XL`. I was too enthusiastic to run `llama-bench` or `ggrun`, so I spent 30-45 minutes tweaking `llama-server` arguments, and by 01:00 I had set up Grok Build to list the model served by `llama-swap` running on `supra` under `/model`. With a bit of trial-and-error and looking at `nvtop`’s graphs, I ended up with this, which should serve you as well as it did me, on 32 GB of total VRAM:

```
llama_server --port 8935 --host :: \
  -m Qwen3.8-27B-UD-Q4_K_XL.gguf --no-mmproj --load-mode mmap \
  -fa on -b 2048 -ub 256 -t 6 -tb 6 \
  -ctk q8_0 -ctv q4_0 \
  -ctkd q4_0 -ctvd q4_0 \
  --jinja -ngl 999 \
  --kv-offload --no-context-shift \
  --reasoning on --reasoning-preserve \
  --dry-multiplier 0.8 --dry-base 1.75 --dry-allowed-length 2 --dry-penalty-last-n 0 \
  --presence_penalty 0.0 --repeat_penalty 1.0 \
  --spec-type draft-mtp --spec-draft-n-max 2 \
  --temp 1.0 --top_p 0.95 --top_k 20 --min-p 0.0 \
  -c 262144 --parallel 1 --cont-batching
```
(Note that `--cont-batching` is not needed for `--parallel 1`, but I sometimes try out `--parallel 2` to work on more than one session at a time.)

A few things about this `llama-server` invocation go against popular dogma and hot takes online: KV cache quantization, speculative decoding drafters’ KV cache quantization, and sampling parameters.

*“Friends don’t let friends quantize the KV cache”* one YouTube comment had said. So far I’ve been running all other models with symmetric `q8_0` KV cache. However, here I wanted to see what it takes to get maximum context length here, so `-ctv q4_0` it is. Fact is: KV cache values are less sensitive to quantization than KV cache keys, but maybe this will cause issues, especially as the context window utilization grows. Trade-off and potential pitfall duly noted. If results ended up not satisfactory, I would revert to symmetric `q8_0`, and accept a shorter context window.

Another thing: plenty of videos online claim that speculative decoding is free. No, it isn’t. The MTP drafters require VRAM and compute buffers. However, since MTP is a must-have to prevent `tg` tanking as the KV cache fills up, so let’s also try quantizing the KV cache of the MTP drafters to `q4_0`.

*Oh noes!* `q4_0`?! **Sacrilege!** Whatever… How bad can it be? (Spoiler: not bad.) Live a little.

Also note the sampling settings that the Qwen team recommends. It’s “running hot” (`--temp 1.0`) compared to the default recommended sampling temperature for Qwen3.6 (`--temp 0.6`). Now that many consider that Qwen3.8 tends to “overthink”, a new idea has arisen: run it with `--temp 0.6`. Nah. Do you really know better than the model’s creators? I doubt it. So `--temp 1.0` it is, and presence and repetition penalties are also what the Qwen team recommends. The DRY arguments were copied over from what I use for Qwen3.6.

Other “witch hunts” include folk wisdom that equates to *“trust me bro, the chat template is the problem”*. Maybe; but let’s establish a baseline first. No special `froggeric` template
, or the template that the Nail/Dagger GGUFs embed

BTW, did you know? The weights of the Nail/Dagger GGUFs are the same as Qwen3.6 MoE and 27B respectively, so you can apply the template by itself on any Qwen3.5 or 3.6 without downloading GGUFs. I do so on Qwen3.6 MoE and 27B.

## Prompting with a braindump, a hope, and a prayer

With `llama-swap` up and running, I need a use case. But not just any use case. Something that will actually show me whether I can use this as a daily driver. I `cd` to the root directory of the 3 git repos of Breek v2 (more specifics on that tomorrow) and enter `grok` and select the model.

I launch Handy with Parakeet Unified 0.6B and start rambling for a few minutes into the webcam’s mic about:

- **Product scope and value propositions:**What the product is and what it does, and for whom.
- **Product development history:**Where we’re coming from (more focus on the recent v2 releases than v1.0).
- **Software architecture:**What the architecture looks like, how the overall product is deployed.
- **Entities and associations:**Which key entities/schemas are important to look into, and how they interrelate.
- **Current state and recent work:**What the current state of the 3 codebases is,- *grosso modo*, including a short braindump about recent features implemented and bugs fixed (- *“but do check the repos’ git logs to see what’s been going on”*).
- **Two REST APIs:**Why we have 2 REST APIs serving one web app, what problem the Phoenix backend solves, how it works together with the legacy PHP backend.
- **Ongoing strangler fig:**Where we are standing along the v2-internal strangler-fig pattern of some API routes from PHP to Phoenix, and what we want to do next.
- **Bug-related functionality:**Feature tiers, subscriptions, and how upgrading or renewing a Property’s “license” works both on the frontend and the backend; loosely/conceptually and in terms of the entities that are used, not in detail / with references to code.
- **The bug report:**Finally, reading verbatim what our “Champion Customer” reported via WhatsApp.

And here’s the benefit of not shipping vibecoded slop that you never ever review and can therefore never understand: you understand how the system works. Thus, I also talked about:

- Some **hunches/hypotheses**about where the bug might be lurking, with some hedging:*“but don’t take this as gospel, investigate thoroughly despite what I’m telling you, as this is all unverified on my part, so form your own conclusions after exploring”*.
- **Similar issues**we had solved before in recent months.
- **Past issues**we’ve had with Redis caching of DTOs as one potential source of bugs that we’ve been stamping out gradually.

I had brought in Redis a year ago, but it had been akin to putting a caching lipstick on a PHP/MySQL pig with some bad,

badSQL queries and slow post-fetching filtering of deeply-nested PHP associative arrays. This is a business-domain issue: property management requires the modeling of entities with many hierarchical and non-hierarchical associations, which makes it easier (and slower) to not have SQL do everything. This worked for some time, but the caching layer had been implemented, let’s say… “not very well” (i.e. it was a hasty countermeasure), and as the feature set and number of entities grew, thorough cache invalidation had become a nightmare. This was one of the reasons for adding the Phoenix REST API; we never have to cache anything, thanks to Redis PubSub and SQLite (but that’s a whole other article).

Anyway, I really braindumped into the mic for maybe 10 minutes straight, as Karpathy advises , and as I’d been doing since I discovered Handy a few weeks ago. Then I waited for Handy to insert the transcription into Grok Build’s input field, and hit Enter. To the list above, now add:

- A rich, semi-structured train-of-thought ramble on the product, the project codebase, the issue, potential root causes, and hunches.

### “Overthinking”? What overthinking? It’s doing the right amount of thinking.

Remember also that it was barely 7 hours since the GGUFs had been released, and people hadn’t yet explored the impact and (alleged?) downsides of `xhigh` reasoning effort or generated any “folk wisdom”. And if they had, I had not even checked comments or YouTube videos yet. I was itching to try out the model. Therefore, I had no awareness of this `xhigh` default setting that people later and still now deride as “wasteful”. Whatever shipped as default is what it was. First, the baseline; then, improvements. One step at a time, otherwise the alleged improvement might actually end up being worse than the baseline that you didn’t try out.

I skip setting Grok Build to “plan mode”, and set it to `always-approve` mode (YOLO). I hit Enter, the model loads, and after prompt-processing it goes into an *epic* reasoning-and-response spree.

A couple of minutes pass.

I can see that Qwen3.8 explores the codebase with `codebase-memory-mcp`
 (as using it is part of my Grok Build rules, to save on tokens, especially when I use Grok 4.6). It finds the TSX of the NextJS page on which I told it the bug appears. It figures out where the different API fetches and button TSX component are. It investigates the feature-tier upgrade button, and how the button checks permissions, how it gates its state based on the user’s current role-based and the property’s license-state permissions, and how it calls one of the two backends via an internal proxy route.

Qwen3.8 then checks to see which backend it calls. It jumps from the NextJS git repo to the PHP repo and dives deeper into the route handler that it discovered earlier in the NextJS codebase. It decides to see how the route handlers for the property information are implemented. It announces that route handlers are written in a consistent pattern (“nice!” – gee, thanks Qwen for the recognition! It took me weeks to bring them up to this state). It discovers that I’ve implemented some of `Ecto.Schema` and `Ecto.Changeset` as PHP classes. It hypothesises that the schemas of the related entities might be the issue. It checks those for any bugs and finds none, then does the same based on the hypothesis that the changeset instances might be the issue, and comes up empty-handed.

*“Damn, this thing is thorough!”*, I think, while I’m messaging my friends on Signal of what’s going on. I’ve been keeping them updated for the past 3 months on my experiments with local AI, and five of them either have ordered their own `supra` setup, or are biding their time.

Qwen3.8 then realizes that the side-concerns are not where the bug is hiding, and moves to the business logic behind the route handler, i.e. how a `%License{}` is released and a new one is “attached” to a `%Property{}` (though, in PHP, where the write path remains). It checks associations, looks at the relevant database schemas, figures out that nothing has changed there in more than 4 months after checking the git logs of the PHP backend’s git repo and the migration scripts, and moves on to more reasoning, more probing, more exhaustive exploration of every nook and cranny where the issue could be hiding.

It’s now maybe 5 minutes, tops, since I pressed Enter, and Qwen3.8 has figured out how the Phoenix backend gets notified about data changes on the PHP/MySQL side via Redis PubSub. Qwen3.8 doesn’t know what I know, i.e. that this cannot be relevant to the issue, but I let it do its thing. Tomorrow I’ll explain in another blog post why those “reasoning sidequests” are not necessarily wasted effort, if done efficiently). It decides to investigate whether the Elixir side is doing something funky with the license upgrades at all, and comes up empty. The write path is still through PHP, and Phoenix is basically a “materialized view” of the database state that makes hydration of list views way more performant. Anyway, Qwen3.8 then notices something that it finds inconsistent and marks it for later. It decides to return to the PHP repo and check out how Redis is being used for caching.

Meanwhile, I have no idea how long this is going to still run. I look at `supra`’s `llama-swap` UI and I see around 30 to 35 tok/s `tg`. Looks OK. Qwen3.8 has already “eaten up” a small chunk of its 262k context window, and I hope that it can get the job done before hitting the 85% auto-compaction threshold. Why? Because compaction is a crappy workaround that’s best avoided with the use of tightly-scoped and orchestrated subagents that never need to compact, thanks to the tight scope and no “overthinking” (which is how I’ve been using Qwen3.5 MoE with ThinkingCap for weeks now, and with great results). But more on that in another blog post.

I go to bed tired, enthusiastic, but also uncertain of what the model will ultimately achieve, and by when.

### The next morning: the verdict

First things first: I look into the situation as soon as I wake up. How did it do? As it turns out, I could have been a bit more patient and less pessimistic. You see, this is something new: `pp`/`tg` can be slow with a dense 27B model on budget NVIDIA GPUs, but these are the wrong metrics.

It did indeed finish its task, and reported its findings and that it fixed the bug. The Grok Build session shows numerous autonomous turns with *copious* reasoning traces. Crucially, there are no “doom loops”. There are not *“wait, I should also check…”* or *“but the user asked me to …”* that have sometimes appeared with stock Qwen3.6 MoE in the past 2 months.

Because I don’t anymore remember the details, a few minutes ago on another Grok Build session I switched to Nemotron 3.5 Lightning (1M context, no MTP) and asked it to investigate and report metrics from the Qwen3.8-27B session log. Key findings:

The following is the only part of this article that’s AI-generated and then redacted by me to protect the guilty, i.e. me.


- **Total autonomous coding time:**~10 min
- **Total messages generated:**104
- **Average throughput:**~10.4 messages/minute
- **Key breakthrough (3.85 min):**Model identified the root cause of the bug
- **Tools used:**- `run_terminal_command`(21),- `read_file`(17),- `use_tool`(12),- `list_dir`(2),- `search_tool`(1)
- **Compaction:**The model did NOT auto-compact during the Qwen3.8-27B part of the session; compaction was a post-session operation.
- **Model efficiency:**Qwen3.8-27B demonstrated consistent ~10 msg/min throughput during autonomous coding, making concrete progress toward identifying and fixing the bug. The model quickly grasped the architecture (3 repos), indexed the codebases, and pinpointed the exact gap before proceeding with the fix.

As for the performance metrics, I instructed Nemotron 3.5 Lightning to query `llama-swap`’s SQLite database on `supra` with `sqlite3` for performance metrics and it came up with this:

- **avg**427.94 tok/s (range 326..500)- `pp`:
- **avg**33.34 tok/s (range 31..35)- `tg`:
- **Total input tokens:**~197,000
- **Total output tokens:**~68,000
- **Total duration:**~8,741 ms avg per request
- **avg draft tokens per request:**1,132.25
- **avg accepted draft tokens per request:**723.85
- **draft acceptance rate:**63.93%

Solid performance with the `UD-Q4_K_XL` quant, across 2 budget GPUs (or what nowadays counts as “budget” given prices these days *smh*).

I can certainly live with **10 minutes** wall-clock time for such a task, even if it would have merely *investigated* the cause(s) of a bug that might be lurking in the interactions of 3 different codebases (worst case). Ten minutes is *stellar*. But did it get the job done? Or was all this “overthinking” (*sic*) for nothing?

Here’s the gist of what it reported when it allegedly finished (paraphrased/redacted):

- My hunches/hypotheses were not entirely correct, but the issue was indirectly related to them.
- It found the bug, which was caused by `[REDACTED]`, and fixed it.
- Along the way, it also identified two *indirectly related*bugs that were on its juggernaut path to a solution. It fixed these too, because they might impact the feature in other ways that we had not accounted for.

An LLM claims that it did everything perfectly means that I claim that it might be full of crap. Trust, but verify!

I switch to Qwen3.6-35B-A3B (NVFP4 “HQ” quant by `michaelw9999`). I’ll have to pay the cost of prompt re-processing the 3.8 session so far, but the 3.6 MoE also has a 262k context window. I don’t want to compact for no reason and lose *any* nuance of the juggernaut’s path to (alleged) victory. Plus, prompt processing on the 3.6 MoE is *very* fast thanks to NVFP4 quantization.

I give Qwen3.6 login credentials for a demo user on the NextJS app and ask it to use Chrome DevTools MCP and replay the UI flow that the bugfix addressed.

Isn’t it nice that you can give locally-hosted models

`.env`files and credentials without worrying about what could one day escape Amodeicide’s black box?

Qwen3.6 logs in at `http://localhost:3000` and replays the flow. It clicks around, finds a property with an expired license and assigns a new one. It finds another property, this one with a Capture license, and upgrades it to Monitor (exactly what was not working for our customer). Everything works as expected. I am impressed.

I ask it to run `ocr` (Alibaba’s `open-code-review`) on recent changes. `ocr` is still configured to use ThinkingCap-27B as the reviewer, which up until this point was the “slow but good” model in my inventory. Ideally, I’d be using a model that’s smarter than 3.8 to review 3.8’s own code changes, but a) SuperGrok means I have no access to API usage, and b) I don’t want to give DeepSeek V4 Flash or Pro access to proprietary code.

ThinkingCap cranks through the changes. The review comes up with no complaints.

I am utterly astonished. Not because I didn’t expect Qwen3.8-27B to be such a monster; I was hyped up on its hoped-for capabilities. But because (like many of us) I was still harboring a lingering worry that a) it might arrive benchmaxxed (judging by its official comparison with Opus 4.6 Max), and b) the Qwen team might *not* really have achieved such a jump in intelligence between 3.6 and 3.8, in the same model size.

## “Yeah but it’s slow!”

Is it? Is it *really* slow? Yes, `pp`/`tg` across 32 GB of VRAM comprised of two of the cheapest 16 GB Blackwell GPU model you can get aren’t the 100+ tok/s `tg` I get from Qwen3.6 MoE.

Looking at my `llama-swap` activity logs, it’s not even half that. Besides the war-story session I explained above, look at the pretty charts across 3 days’ worth of data across three different quants that I have used:

- unsloth/Qwen3.8-27B-GGUF
 `UD-Q4_K_XL`(17.9 GB)
- AtomicChat/Qwen3.8-27B-GGUF
 `AD-Q6_K`(23.1 GB)
- utautako/Qwen3.6-27B-NVIDIA-NVFP4-MTP-Q8attn-GGUF
 `NVFP4`with`Q8`attention (19.1 GB)

#### Prompt processing

| Quant | N | Mean | SD | Min | Q1 | Median | Q3 | Max | 
|---|---|---|---|---|---|---|---|---|
| `AD-Q6_K` | 143 | 460.963 | 305.656 | 13.903 | 216.751 | 406.996 | 702.661 | 1056.311 | 
| `NVFP4` | 132 | 393.752 | 370.426 | 36.659 | 143.801 | 257.665 | 496.019 | 1557.156 | 
| `UD-Q4_K_XL` | 205 | 452.233 | 281.202 | 51.067 | 238.629 | 381.945 | 652.459 | 1423.874 | 

#### Token generation

| Quant | N | Mean | SD | Min | Q1 | Median | Q3 | Max | 
|---|---|---|---|---|---|---|---|---|
| `AD-Q6_K` | 143 | 25.273 | 3.381 | 9.011 | 23.159 | 24.889 | 27.267 | 35.434 | 
| `NVFP4` | 132 | 26.580 | 6.597 | 17.484 | 21.457 | 24.924 | 29.796 | 46.223 | 
| `UD-Q4_K_XL` | 205 | 31.093 | 6.275 | 18.091 | 25.869 | 30.600 | 36.115 | 44.519 | 

Geek out over those numbers (as I did) but I ask you: *do  pp and tg matter that much, ultimately?* Does it really matter, even with the (allegedly) “wasteful” 

`xhigh` reasoning effort, if you:- Have not one, but 3 mature codebases (in 3 different programming languages, which shouldn’t matter that much or at all to a model, but compare the alternative of finding one developer who can work well on all 3…).
- Have a weird bug you need to fix, and only a hunch about potential root causes that you braindump into the prompt.
- Have your coding harness set up with useful tools like `ocr`and`codebase-memory-mcp`so that you don’t waste time and tokens having the harness list, search and`grep`source code files.
- Have some aspects of the architecture already documented in Markdown files, either by hand or generated by a competent model.

…and can give Qwen3.8-27B 10 minutes’ worth of rambling, then walk away for a similar amount of time, and return to find that it *had not only fixed that one bug, but also taken care of two more bugs that were on its path towards the solution*?

Of course it doesn’t matter! What matters is **wall-clock time to completion**, not vanity `pp`/`tg` figures. This model is clearly a *beast* for this kind of problem-solving. It’s not overthinking. It’s *thinking* about the task and its subtasks. It finds its way around open sub-tasks and investigation paths. In short, it acts very human-like, if you look at the reasoning traces. And let’s admit that there is no way, absolutely *no way* that I could have done this faster than 10 minutes–and I refactored most of the v1.0 PHP and NextJS/TypeScript-with-`any`, and wrote the Elixir/Phoenix backend from scratch. I have the lay of the land in my mental KV cache, and still there’s no way I can compete with Qwen3.8-27B’s 10 minutes of wall-clock time from a rambling prompt to an unvalidated fix.

Caveat: I have *not* yet tested it with `medium` reasoning effort. Perhaps it does equally well in terms of quality *and* is faster, too. But clearly `xhigh` gets the job done. Would `medium` have been faster than `xhigh`? Maybe. How much faster? 3 minutes? 5? 7? Does it matter? Well, yeah, if you’re working on a tight deadline or need to get something done quickly, `medium` might be faster. But for a problem like this one, `xhigh` is the way to go.

- `xhigh`here might have been wasteful compared to- `medium`, but we are already in diminishing-returns territory, and in “we have SOTA at home” territory. Note: of course if you have a massive plan with multiple sub-tasks of similar complexity, yeah,- `medium`’s likely wall-clock savings will add up.
- `UD-Q4_K_XL`did just fine- *on this type of task*.
- It’s not slow. It’s actually fast *for this type of task*, despite the 30..40+ tok/s`tg`values.
- KV’s values quantized at `q4_0`did not impact the quality,*on this type of task*.
- MTP drafters’ KV at symmetric `q4_0`quantization did not impact the quality either,*on this type of task*.

Why do I qualify the above with “on this type of task”? You’ll find out in the next piece.

For now, rejoice! This is an amazing model.

This is the first of three (or maybe four, TBD) pieces. The next one is about why most of the YouTube-style “benchmarks” circulating for the past many months are not really benchmarking or evaluating models in any way that makes sense for real-world usage, and why a mature codebase is actually part of the prompt.
