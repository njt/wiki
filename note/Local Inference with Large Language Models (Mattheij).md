# Local Inference with Large Language Models (Mattheij)

Jacques Mattheij's practitioner guide to building a self-hosted LLM rig for his NDA-bound M&A advisory business: why confidentiality forces local inference, a review of production-tested inference engines and Chinese open-weight models, a detailed hardware tour (3090s, Blackwell 4000s, remanufactured 48GB 4090s, DGX Sparks), and the PCIe engineering needed to mount eight GPUs — ending at a ~$25k, 3kW rig that runs GLM 5.3 proofreading in-house.

---

## Why local at all

The trigger is legal, not ideological: his business signs NDAs with scary numbers in them, client reports can't leave the building, and the obvious fix — upload to a hosted provider before sending — is out. He's refreshingly honest about the chain of trust questions that follow ("can we trust Altman or Amodei… can we trust everybody who works there?") and dismisses them with a shrug: the answer to "is shipping the files out at all wise" was simply "No."

> There is absolutely no way that I would allow an AI control or direct input into our work process.

Interesting that he'll let an AI *proofread* but not decide. That's the same boundary a lot of careful practitioners draw — generation and judgment on separate sides of a line.

> 'Sorry Jacques, I can't let you do that' is not something that I would ever accept from a computer, computers do what I tell them to do, not what they will allow.

He also names the ideological reasons explicitly: distaste for companies vacuuming up human culture and selling it back, and refusal to let an outside party control his workflow. Note that the *first* reason (NDA) would suffice — the ideology rides along for free.

## Engines: he's used them in production, and it shows

- **llama.cpp** — the default. Simple to build, auditable, extremely versatile, not the fastest. He values that the code quality is high and updates are daily. Commercial operators avoid it because every last percent of throughput matters to them.
- **vLLM** — the commercial workhorse, more performance *if* the hardware is a good match, but fragile, expert-hungry, and best bought with the engine in mind rather than retrofitted.
- **SGLang** — middle ground; Docker packaging makes auditing harder than llama.cpp, which "wins hands down because of its simplicity."
- **DwarfStar** (antirez) — one model, unified-memory Macs, easy. **Ninfer** — one NVidia 5000-series GPU, every last drop, with a non-backwards-compatible `.ninfer` artifact format and forks for older GPUs.

His verdict on whether engine choice matters: mostly no, as long as it exposes an OpenAI-compatible endpoint, runs well, and supports your model. That endpoint convention has quietly become the ABI of local inference.

## Models: unapologetically all Chinese

His workhorses are Qwen 3.8 27B (coding, robotics), DeepSeek, and GLM 5.3 for proofreading — "so much so that it can perform better in wall-clock than smaller models that will use many more tokens and/or many more turns." He notices the obvious:

> all of the models mentioned above are Chinese. This is no accident: China is after the United States *the* AI powerhouse and because they can't or don't want to compete directly with the big US service providers they've decided to open up their models.

He calls the US response "an endless barrage of fearmongering, attempts at regulatory capture and other shenanigans." Love it or hate it, his economics are simple: "you really can't beat free." He flags one caveat — steer clear of sensitive political content.

> GLM 5.3 is also very good at cyber related tasks. To this day it has never refused an assignment.

That sentence is doing a lot of work, and not all of it reassuring. The same no-refusal property that makes a proofreader never say no applies everywhere.

## Hardware: the meat

Tiered honestly: hobby ~$10k (fat PC + 16GB GPU), small business the same starting point with more money, larger business $5–10k **per seat**. GPU opinions from running them day and night:

- **RTX 3090** — still the value pick; used prices now *exceed* original new prices, which "does not normally happen with computer hardware." He power-limits to 275W for longevity.
- **RTX PRO 4000 Blackwell** — single-slot, power-sipping, roughly 3090 performance at 2x the used-3090 price.
- **RTX 4090** — ~50% faster; Chinese remanufacturers double the VRAM to 48GB, though BAR1-window quirks can bite engines that don't compensate.
- **DGX Spark** — 128GB unified RAM but only ~275 GB/s bandwidth, a third of a 3090; "performance is lacking" but convenient.

Beyond eight GPUs in one box: PCIe lane splitting, riser cables (C-Payne), and PCIe switch boards (ADT, ~$1000, works driverless on Linux) with a homemade 2020-aluminum frame and 3D-printed mounts. This is the part of the guide almost nobody else writes down.

## The bottom line

~$25k total: 256GB RAM, 8 GPUs, 240GB VRAM, dual 1600W PSUs. Flat out: well over 3kW, "like having a very fancy space heater" — mitigated by solar panels that help "but only during the day." Performance: a few thousand tok/s across concurrent Qwen3.8 slots; GLM 5.3 at 50–100 tok/s with llama.cpp underutilising the GPUs.

## Opinionated take

This is the most useful kind of hardware writing: opinions from someone who actually runs the load, including the unglamorous parts (BAR1 windows, enumeration limits at 14 GPUs, the "GPU exclusion principle"). It's also a document with a clear point of view you should discount accordingly: he has already bought the rig, the GPU market read is self-justifying, and the anti-US-lab passages are polemic even where the underlying economics (free weights, real capability) are sound. The strongest argument in it is the boring one — NDA compliance — because it makes local inference a compliance decision rather than a hobby. The weakest is the security framing ("if OpenAI decide to hack your server they may well be your only means of defense"), which gestures at threat models without developing them. Abliterated/no-refusal models appear in his taxonomy casually; see [[What Security Teams Need to Know About Agents Using Abliterated Models]] for why that deserves more caution than he gives it.

## Related pages

- Strengthens [[Qwen3.8 27B Hardware Tests]] with an independent second rig reaching the same verdict: the 27B Qwen class is the local workhorse, and llama.cpp not yet extracting maximum GPU utilisation appears there too.
- Nuances [[GPU Self-Hosting for Coding Agents]]' cost economics with a small-business tier it doesn't cover: Mattheij's $5–10k per seat figure and "cheaper to use a hosted service, but I trust you have your own reasons" line land on the same break-even framing.
- Complicates [[Local Qwen Is Not a Worse Opus]]: Alex Ellis argues local models are a different tool, not a worse frontier model; Mattheij instead picks a giant local model (GLM 5.3) precisely to be frontier-adjacent, choosing scale over the "different tool" framing.
- Extends the hardware detail of [[Dual GPU RTX 5080 + RTX 3090 Qwen 3.6 Setup]] from two consumer GPUs to eight, with switch boards and risers as the scaling path that field report never needed.

---
*Sources: [[raw/local-inference-with-large-language-models]], [[summary/local-inference-with-large-language-models]]*
*Last updated: 2026-11-03*
