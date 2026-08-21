---
url: https://www.hardware-corner.net/qwen3-8-27b-hardware-tests/
date_fetched: 2026-08-21
---

Qwen3.8 27B is another model in the 27B class that looks particularly interesting for local inference. We wanted to find out what it actually takes to run it on consumer hardware, especially at longer context lengths.

This article is based on our own llama.cpp benchmark results. We are not evaluating model intelligence, coding quality, or reasoning accuracy here. The focus is hardware: VRAM usage, context scaling, prompt processing speed, and token generation speed.

We tested the model on several NVIDIA GPUs, including the RTX 3090, RTX 4090, RTX 5090, and a dual RTX 5060 Ti setup. We also have results from an Apple M5 Max system with 128 GB of unified memory.

The testing environment used Ubuntu 24.04, NVIDIA driver 595.58.03, CUDA 12.8, and llama.cpp (MTP disabled) build 153d324bc (10364). All GPU tests used CUDA with full model offload and Flash Attention enabled.

One important detail is that llama.cpp reports the model architecture internally as `qwen35` in the benchmark output. The benchmark data below is labeled as Qwen3.8 27B because that is the model we tested.

## What Qwen3.8 27B Requires

The Q4_K Small build used in our tests is 16.68 GiB and contains 27.32 billion parameters.

That makes VRAM the first practical limit. The model itself fits on a 24 GB GPU, but the usable context length depends on the additional memory required by the KV cache and runtime overhead.

Our VRAM measurements show a fairly predictable increase as context grows.

| Context Length | Measured VRAM Usage | 
|---|---|
| 4k | 18 GB | 
| 8k | 18 GB | 
| 16k | 19 GB | 
| 32k | 20 GB | 
| 64k | 22 GB | 
| 128k | 26 GB | 
| 256k | 34 GB | 

The important number for a single 24 GB card is 64k. At that context length, our measured usage was about 22 GB, leaving limited headroom for the operating system, CUDA runtime, and other allocations.

At 128k, the requirement rises to about 26 GB, so a 24 GB GPU is no longer enough for a fully GPU-resident configuration.

A 32 GB card such as the RTX 5090 can handle 128k in our testing. At 256k, the measured requirement rises to around 34 GB, which moves the model beyond the capacity of a 32 GB consumer GPU.

This gives Qwen3.8 27B a relatively simple hardware profile: 24 GB is the practical starting point for long-context use, while 32 GB makes 128k context possible without offloading.

## What GPU Do You Need for Qwen3.8 27B?

For basic 4k to 32k local inference, a 24 GB GPU is the most straightforward option. The RTX 3090 remains particularly interesting because its large VRAM pool allows the model to run without compromises at these context sizes.

For longer agentic workloads, 64k is the practical target on a 24 GB card. Our measurements show 22 GB at 64k, which leaves some operating margin.

The RTX 5090 is the more capable single-GPU option. Its 32 GB of VRAM is enough for the measured 128k configuration, although the model is not especially fast at that context length compared with shorter contexts.

The dual RTX 5060 Ti configuration is another interesting route. Two 16 GB cards provide enough aggregate VRAM to run the model at larger context lengths, although multi-GPU inference introduces its own scaling and interconnect considerations.

The key difference between these systems is not simply whether the model loads. It is how much context you can run while keeping the entire workload in GPU memory.

## Dual RTX 5060 Ti Performance Results

We tested two RTX 5060 Ti cards with 15,849 MiB of VRAM each. Combined, the setup provides roughly 32 GB of physical GPU memory, which is enough to reach the larger context sizes in this test.

The RTX 5060 Ti setup is substantially slower than the RTX 3090 and RTX 4090 for token generation, but it demonstrates that a lower-cost multi-GPU configuration can handle the model.

| Context | Prompt Processing (t/s) | Token Generation (t/s) | 
|---|---|---|
| 4k | 1167.70 | 22.47 | 
| 8k | 1116.94 | 22.03 | 
| 16k | 1032.66 | 21.57 | 
| 32k | 904.12 | 20.43 | 
| 64k | 721.46 | 18.50 | 
| 128k | 512.63 | 15.55 | 

At 4k context, generation is around 22.5 tokens per second. That falls to 15.6 tokens per second at 128k.

The setup is therefore more interesting as a capacity-per-dollar solution than as a high-speed inference platform. If the goal is simply getting a 27B model and a large context window into a relatively affordable system, two 16 GB cards can make sense.

## RTX 3090 Performance Results

The RTX 3090 is still one of the most interesting cards for local LLM inference because of its 24 GB VRAM capacity and relatively low second-hand pricing.

For Qwen3.8 27B, that 24 GB capacity is enough to run the model up to 64k context in our testing.

| Context | Prompt Processing (t/s) | Token Generation (t/s) | 
|---|---|---|
| 4k | 1308.05 | 40.31 | 
| 8k | 1235.01 | 39.76 | 
| 16k | 1136.84 | 38.77 | 
| 32k | 977.25 | 37.02 | 
| 64k | 766.59 | 33.95 | 

The RTX 3090 starts at just over 40 tokens per second and remains at nearly 34 tokens per second at 64k context.

That is a good result for a card that is several generations old. The main limitation is VRAM rather than raw compute. At 64k we measured about 22 GB of VRAM usage, leaving relatively little room before reaching the card’s 24 GB capacity.

For a price-conscious local inference build, this makes the RTX 3090 one of the more practical choices for Qwen3.8 27B.

## RTX 4090 Performance Results

The RTX 4090 has the same nominal 24 GB VRAM capacity as the RTX 3090, so VRAM remains the limiting factor for Qwen3.8 27B.

The difference is compute performance. The 4090 is considerably faster, particularly for prompt processing.

| Context | Prompt Processing (t/s) | Token Generation (t/s) | 
|---|---|---|
| 4k | 2962.59 | 46.16 | 
| 8k | 2854.39 | 45.52 | 
| 16k | 2679.11 | 44.32 | 
| 32k | 2367.35 | 42.17 | 
| 64k | 1918.13 | 38.41 | 

At 4k context the RTX 4090 reaches 46.16 tokens per second, compared with 40.31 tokens per second on the RTX 3090.

The difference is larger for prompt processing. The RTX 4090 processes the 4k test at almost 3,000 tokens per second, while the RTX 3090 is around 1,300 tokens per second.

The 4090 therefore makes sense when you want higher throughput from a 24 GB card, but it does not solve the long-context capacity problem. We tested up to 64k, and that is the context range where the 24 GB memory limit remains practical.

## RTX 5090 Performance Results

The RTX 5090 changes the equation because it combines significantly higher GPU performance with 32 GB of VRAM.

That additional VRAM is important for Qwen3.8 27B. Our measurements reached 128k context while staying inside the card’s 32 GB capacity.

| Context | Prompt Processing (t/s) | Token Generation (t/s) | 
|---|---|---|
| 4k | 3749.51 | 74.83 | 
| 8k | 3592.89 | 52.23 | 
| 16k | 1825.86 | 38.02 | 
| 32k | 1146.03 | 28.96 | 
| 64k | 717.76 | 26.22 | 
| 128k | 461.17 | 22.79 | 

The 4k result is 74.83 tokens per second, but performance falls significantly as context increases.

At 32k the result is 28.96 tokens per second, while at 128k it falls to 22.79 tokens per second.

This is an important point when looking only at GPU specifications. The RTX 5090 gives Qwen3.8 27B the VRAM needed for 128k context, but longer context still carries a substantial performance cost.

For interactive agentic workloads, 32k to 64k is therefore a more balanced operating range if response speed matters.

## M5 Max with 128 GB Unified Memory

The M5 Max system provides a completely different approach.

Instead of discrete GPU VRAM, the machine uses 128 GB of unified memory shared between the CPU and GPU. That gives it far more memory capacity than a 24 GB or 32 GB consumer GPU.

We tested Qwen3.8 27B at contexts up to 64k.

| Context | Prompt Processing (t/s) | Token Generation (t/s) | 
|---|---|---|
| 4k | 834.1 | 31.4 | 
| 8k | 719.9 | 27.7 | 
| 16k | 543.5 | 23.6 | 
| 32k | 516.8 | 21.6 | 
| 64k | 424.7 | 18.2 | 

The M5 Max does not match the RTX 3090, RTX 4090, or RTX 5090 for token generation, but the 128 GB memory pool gives it a large capacity advantage.

For users who value a quiet, compact system with a large memory pool, unified memory remains an interesting alternative to discrete GPU hardware. The trade-off is throughput.

## NVIDIA GB10 Performance Results

We also tested Qwen3.8 27B on a Dell Pro Max system powered by NVIDIA’s GB10 platform. This is a very different setup from the discrete GeForce GPUs above. The GB10 uses a large unified memory pool rather than a conventional 24 GB or 32 GB graphics card, giving it enough memory to run Qwen3.8 27B across the full context range tested.

This benchmark was run separately from the GeForce tests, using CUDA 13.2 and NVIDIA driver 595.71.05.

| Context | Prompt Processing (t/s) | Token Generation (t/s) | 
|---|---|---|
| 4k | 812.60 | 12.20 | 
| 8k | 782.60 | 11.90 | 
| 16k | 752.68 | 11.57 | 
| 32k | 678.66 | 10.88 | 
| 64k | 576.25 | 9.72 | 
| 128k | 441.66 | 8.03 | 
| 256k | 228.24 | 5.94 | 

The main advantage of the GB10 is memory capacity rather than inference speed. Qwen3.8 27B can be run at 256k context without approaching the memory limits that would prevent the same configuration from fitting on a 24 GB or 32 GB consumer GPU.

The performance profile is considerably different from the RTX 3090, RTX 4090, and RTX 5090. At 4k context, token generation is 12.20 tokens per second, compared with 40.31 on the RTX 3090 and 74.83 on the RTX 5090. At 64k context, the GB10 produces 9.72 tokens per second, and at 256k it falls to 5.94 tokens per second.

Prompt processing also declines as context grows, going from 812.60 tokens per second at 4k to 228.24 tokens per second at 256k.

For Qwen3.8 27B, the GB10 is therefore best understood as a high-memory local inference platform rather than a high-throughput GPU. Its large unified memory pool makes very long contexts possible on a single system, while the lower token generation rate makes it less suitable when fast interactive responses are the priority.

## Qwen3.8 27B vs Qwen3.6 27B

Qwen3.6 27B is the most useful comparison because it is close in model size and can be tested on the same hardware.

We ran both models on the RTX 5090 using the same general llama.cpp benchmark approach. The Qwen3.8 result uses Q4_K Small at 16.68 GiB, while the Qwen3.6 result uses Q4_K Medium at 16.39 GiB.

The two models are therefore close in memory footprint, but their benchmark behavior is different.

| Context | Qwen3.8 27B PP / TG | Qwen3.6 27B PP / TG | 
|---|---|---|
| 4k | 3749.51 / 74.83 | 3873.80 / 75.57 | 
| 8k | 3592.89 / 52.23 | 3720.65 / 73.32 | 
| 16k | 1825.86 / 38.02 | 3485.89 / 72.81 | 
| 32k | 1146.03 / 28.96 | 2941.09 / 69.45 | 
| 64k | 717.76 / 26.22 | 1957.43 / 63.66 | 
| 128k | 461.17 / 22.79 | 1179.94 / 54.58 | 

At 4k, the two models are relatively close in token generation. Qwen3.8 reaches 74.83 tokens per second, while Qwen3.6 reaches 75.57.

The difference becomes much larger as context increases. At 64k, Qwen3.8 generates at 26.22 tokens per second compared with 63.66 for Qwen3.6 in our RTX 5090 tests.

Prompt processing shows the same pattern. At 64k, Qwen3.8 measured 717.76 tokens per second, while Qwen3.6 reached 1957.43 tokens per second.

This means the Qwen3.8 result should not be interpreted as a straightforward speed improvement over Qwen3.6. Based on our measurements, Qwen3.8 is closer to Qwen3.6 in general behavior than to the much faster Muse Glimmer 30B result we previously measured.

## Qwen3.8 27B vs Meta Muse Glimmer 30B on the RTX 3090

For RTX 3090 buyers, Meta’s Muse Glimmer 30B is an especially useful comparison.

Our previous Muse Glimmer 30B test used a Q4_K Medium build measuring 14.78 GiB, while the Qwen3.8 27B Q4_K Small build used here is 16.68 GiB.

We tested both models on the RTX 3090.

| Context | Qwen3.8 27B PP / TG | Muse Glimmer 30B PP / TG | 
|---|---|---|
| 4k | 1308.05 / 40.31 | 1476.98 / 45.30 | 
| 16k | 1136.84 / 38.77 | 1364.44 / 44.49 | 
| 32k | 977.25 / 37.02 | 1270.14 / 43.84 | 
| 64k | 766.59 / 33.95 | 1098.35 / 42.45 | 
| 128k | Not practical on 24 GB | 852.85 / 38.13 | 
| 256k | Not practical on 24 GB | 584.38 / 35.46 | 

Muse Glimmer 30B is faster in our RTX 3090 tests at every directly comparable context length.

At 4k, Muse Glimmer reaches 45.30 tokens per second compared with 40.31 for Qwen3.8. At 64k, the gap grows to 42.45 versus 33.95 tokens per second.

The VRAM difference is also important. Muse Glimmer 30B used about 16 GB at the shorter context sizes and reached only around 20 GB at 256k in our testing. Qwen3.8 reached around 22 GB at 64k and approximately 26 GB at 128k.

For someone buying a used RTX 3090 specifically for agentic or long-context inference, that makes Muse Glimmer the easier model to run at very large context lengths.

Qwen3.8 remains perfectly usable on the same card, but 64k is the more realistic upper limit for a fully GPU-resident configuration.

## Hardware Comparison for Qwen3.8 27B

The benchmark results show a fairly clear division between the available hardware.

| Hardware | VRAM / Memory | 4k TG | 64k TG | 128k TG | Practical Target | 
|---|---|---|---|---|---|
| 2x RTX 5060 Ti | ~32 GB | 22.47 | 18.50 | 15.55 | 128k | 
| RTX 3090 | 24 GB | 40.31 | 33.95 | N/A | 64k | 
| RTX 4090 | 24 GB | 46.16 | 38.41 | N/A | 64k tested | 
| RTX 5090 | 32 GB | 74.83 | 26.22 | 22.79 | 128k | 
| M5 Max | 128 GB unified | 31.4 | 18.2 | Not tested | 64k tested | 

The RTX 3090 is still the value option if you can find one cheaply on the used market. It gives you 24 GB of VRAM and enough capacity for 64k context.

The RTX 4090 is the faster 24 GB option, but it does not provide additional context capacity.

The RTX 5090 is the cleanest single-GPU solution if you want 128k context and higher short-context throughput. Its main limitation is that Qwen3.8 loses a significant amount of generation speed as context grows.

A dual RTX 5060 Ti setup is an interesting lower-cost capacity solution, especially for users comfortable with multi-GPU systems. It is much slower than the higher-end cards, but it provides enough aggregate VRAM for 128k in our test.

The M5 Max offers a very different balance. Its 128 GB unified memory gives it substantial capacity, while inference speed is closer to the RTX 3090 at short context and below it at longer context.

## What Hardware Makes Sense for Agentic Use?

For agentic workloads, context length matters because an agent can accumulate tool results, source material, previous actions, and intermediate reasoning over a long session.

For Qwen3.8 27B, our 24 GB GPU results make 64k the practical target. The model uses around 22 GB at that context length, leaving limited but usable headroom.

A 32 GB RTX 5090 gives more room for 128k context. At that size, however, our measured generation rate was 22.79 tokens per second.

That makes 128k possible, but not necessarily optimal for every workload. A shorter context can provide much better interactive performance, especially when the application frequently generates many responses rather than performing occasional long-context analysis.

This is also where the comparison with Muse Glimmer 30B becomes important. On the RTX 3090, Muse Glimmer maintained substantially higher generation speed at 64k and could continue to 128k and 256k while staying within 24 GB.

## Conclusion

Qwen3.8 27B is not an especially demanding model in terms of parameter count, but its context memory requirements quickly become the limiting factor.

A 24 GB GPU is enough for the model and makes 64k context practical. The RTX 3090 is particularly attractive for this because of its second-hand value, while the RTX 4090 delivers higher throughput without adding VRAM.

The RTX 5090 is the better single-GPU choice for users who want 128k context. Its 32 GB VRAM is enough for the measured 128k configuration, although generation speed drops considerably compared with short context.

For users focused on price and local inference, the RTX 3090 remains a sensible Qwen3.8 27B platform. For maximum single-GPU performance and 128k context, the RTX 5090 is the stronger option.

The main takeaway from our testing is that Qwen3.8 27B behaves much more like Qwen3.6 in overall hardware requirements and speed than the faster Meta Muse Glimmer 30B we tested on the RTX 3090. The model is easy enough to run on 24 GB hardware, but long context comes with a real performance cost.

Our results are from llama.cpp build 153d324bc (10364), Ubuntu 24.04, NVIDIA driver 595.58.03, and CUDA 12.8. Different llama.cpp builds, quantizations, KV cache settings, and system configurations can produce different results, so these numbers should be treated as measurements for this specific software and hardware setup rather than universal performance figures.
