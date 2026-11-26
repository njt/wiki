---
url: https://engram.com/blog/lora-size
date_fetched: 2026-11-26
---

TLDR:

- We develop a predictive theory of the capacity of a LoRA adapter as a function of its rank.
- Unlike full-parameter fine-tuning, LoRA learns a
decreasingamount per-parameter.- Given an SFT dataset, we can predict the LoRA size required to fit it.

At Engram, we train lots of models. We train models for individual users and update them when information changes. A major focus of our research is how to customize and adapt models efficiently at scale.

This post focuses on LoRA1. LoRA has become the predominant approach for customizing models in industry. While there exists a large body of knowledge on how to set the hyperparameters and train LoRA effectively2, 3, it’s still unclear how to set LoRA’s rank. Although it’s agreed-upon that LoRA underperforms full-finetuning on larger datasets, it’s not clear when this is the case. How big does a dataset have to be to require a larger LoRA?

In practice, smaller LoRAs enjoy a number of systems advantages, being lighter to load4, 5, faster to generate text from6, and easier to store on disk. For larger models, rank-1 can store quite a lot; some datasets, like RL data, can be represented in a small number of bits and often fit within a rank-one LoRA adapter (or smaller!7, 8, 9).

In this post, we develop a predictive theory and practical approach for sizing LoRA adapters to a new dataset. The idea is simple: the size of the LoRA required depends on how many bits it takes to encode the dataset under the base model. We use this idea to derive an exact formula for how many bits a single LoRA adapter can learn as a function of its rank.

We then extend our idea to measure the capacity of LoRA on full transformers. Since theory can’t predict the nonlinear behavior of deep transformers, we turn to an empirical solution: we generate synthetic datasets of exactly bits under the base model and train on them to measure LoRA’s capacity. Our capacity predictions hold up on real datasets: across a variety of settings, we can accurately determine the necessary rank of the adapter.

## Information theory and LLMs

We begin this post with a basic primer on the connection between information theory and language modeling. Remember that our goal is to find the smallest adapter that fits a given dataset. To do that, we need to measure adapters and datasets in the same unit: bits.

**Language modeling and Shannon**

Language models assign probabilities to strings:

A language model is, at its core, a text compressor10: a model that assigns high probability to data can encode it efficiently. And we’ve known this since the 1950s! By Shannon’s source coding theorem11, a model’s quality (its perplexity, or cross-entropy) corresponds directly to its effectiveness as a compressor. The number of bits needed to encode a string  under a model  is its negative log-likelihood:

Here’s an example of what it’s like to view training loss as *bits*. Given some fixed-size training dataset, we can directly convert loss to bits learned by 12:

**Generating  exactly N bits using an LLM**

To measure the capacity of LLMs being trained with adapters, we need to generate data with a controllable number of bits. Prior work in this space creates data of fixed size with random generative processes, like randomly generated tables of facts13 or random strings of tokens.

To generate truly random data from an LLM, we leverage a tool called arithmetic coding. Arithmetic coding gives us the ability to transform probability distributions into codes; it represents the entire code using a single floating point number in that we can turn into a sequence using .

We take this one step further. A pretrained open-source model like Qwen or Inkling already provides likelihoods for text, so we can run arithmetic coding in reverse: start from  random bits and decode them into text. The result is a sample from  whose code length under the model is  bits. One can think of this as an implementation of Neural Linguistic Steganography14 using randomly selected codes.

Here’s an example of arithmetic coding in use. The following panel allows you to generate strings with exact bit counts under SmolLM2:

Code lengths come out slightly longer than the number of encoded bits because tokens are indivisible. In this example, `queen` is worth eleven bits (surprising!), while `little` carries just two bits. Later in this post we will generate many strings like this to build a dataset of exactly known size, then train an adapter on it to measure how many bits it can learn.

## How much can LoRA store?

Given a matrix to adapt, LoRA works by training two matrices whose product is a low-rank update to :

Here is an matrix being adapted and and are the matrices trained via LoRA. Since is often much smaller than and , the total number of parameters trained by the LoRA is typically much smaller than training all of .

The parameter is a rank-independent scaling factor; in this work, we fix to avoid conflating rank with learning rate.

### Sizing LoRA, in theory

Prior work has estimated that the capacity of full transformer training scales roughly linearly with parameter count15, 16, 17. We don’t intuitively expect this to be true with LoRA, since some dimensions influence the model outputs more than others. Instead, we expect diminishing returns to capacity as rank increases. This is exactly what we see in practice, and it turns out we can predict the capacity curve quite cleanly using a simple statistical model.

Suppose we sample data from a model and then adapt one of its weight matrices to memorize that data. How much can we expect to increase the log-likelihood (and hence decrease the code length) with a rank- adapter? To second-order, looks like an inverted bowl in parameter space. How high we can climb on that bowl can be determined from the gradient and Hessian at the point , and these are both random functions of the training data . Since the data is sampled from , the statistics of the gradient and Hessian are determined by the Fisher information matrix, which greatly simplifies the calculations.

After some algebra, we find that the capacity of a rank- adapter is approximately,

where is the -th singular value of the random matrix . The capacity grows roughly linearly at low rank,

but it plateaus as the rank increases and the singular values decrease. At full rank, the capacity is  bits, recovering a classical result from Wilks’ theorem in statistics18.

For an 800M parameter Qwen3.5 model, which has linear layers in its attention value projections (v_proj), the theory predicts about 378k bits, or 0.48 bits-per-parameter at full rank and 1.3-1.4 bits-per-parameter at low rank.

When we measure capacity using actual training, this prediction turns out to be quite accurate; , the blue curve, is accurate at every rank:

### Sizing LoRA, in practice

Theory predicts around 0.5-1.4 bpp when perturbing individual weight matrices. What about full transformers? When we adapt several matrices *jointly*, the capacity is more than the sum of its parts. We see super-additive scaling of capacity thanks to complementary interactions from adapting multiple weights at once19.

While the theory above cannot explain these super-additive effects yet, we do have a way to generate a dataset of a fixed size (measured above) and a way to measure how much a model learns from data (by checking its loss).

**Extrapolating to the infinite-training regime.** We fit a function to estimate the asymptote of the loss curve under infinite training, similar to20. This modeling allows us to train for a fixed number of steps and predict the asymptote of the loss under high compute, rather than perform the full training. It also allows us to avoid constantly sweeping learning rates, since different learning rates converge to the same capacity under infinite training.

We find a simple log-logistic curve fits our training runs well:

Here is the number of training steps, is the step where the adapter has stored half its eventual bits, and sets how sharp the rise is; we fit this on runs of 1k steps and up, and report as the adapter’s capacity.

Naively, one might expect the total capacity to be the sum of each adapter’s capacity, and hence the bits stored per parameter to remain roughly constant as we adapt more and more layers. However, as we increase the number of layers, total capacity actually rises from 1.5 bits-per-parameter to 3.3:

For a fixed-depth model, we see that increasing the LoRA rank also increases total capacity. Our technique for generating data for across sizes allows us to easily measure the capacity of LoRAs across ranks:

We can also easily study how capacity interacts with training. Although the majority of our results assume capacity in the infinite-training regime, we can quickly model how even in the case of LoRA, capacity rises with the number of training steps21. This finding is intuitive, since training for longer stores more bits:

To test how much models forget about what they already know, we can measure likelihood on samples from the base model; this gives us a measurement of the total number of bits the model forgets during the training process. Across all the models trained for this post, we observe a remarkably predictable trend where models forget roughly as much as they learn:

This finding implies that training is quite efficient: as we add new information to a model, it forgets a comparable amount. A model that learns 10 KB of new information might forget 12 KB of old information. This setting provides a clean testbed for future research to investigate the connection between learning & forgetting.

## Sizing LoRA capacity with real datasets

Remember that our goal is to estimate the rank of LoRA required to fit some arbitrary SFT dataset. To do this, we need a method for measuring the size of datasets. In the previous section, we used log-likelihood under as a way to generate synthetic data of a precise size. Following the same logic, we can measure the log-likelihood of real data to compute its size under .

### Measuring the size of full datasets

We first extend the model’s context window to 2M tokens via YaRN22. If the data still doesn’t fit into context, we estimate its size in bits under the model by cutting it into windows of 2M tokens and summing the log likelihood:

where are consecutive 2M-token windows of the dataset .

Although this gives us a slight overestimate for the total information content of a dataset, in practice we find the error to be small and the estimates useful enough to predict rank within 5% error.

### Testing on real workspaces

We now test our ability to properly allocate adapters and train. We test on datasets of varying sizes scattered across the public internet as well as our and our partners’ workspaces.

For each new dataset, we first measure its size in bits under the model and read the predicted rank off the synthetic capacity curve. We then train LoRAs on the real dataset itself at ranks from to , and report the measured rank as the smallest rank that stores 95% of the dataset.

Even though our capacity estimates assume the training data is drawn from the base model, our predictions are extremely accurate on real datasets: the orange dots, the smallest LoRAs required to fit each dataset in practice, fall almost on the blue line, our estimate of a given LoRA rank's capacity.

## Conclusion

This post described our work to automatically provision adapters of a fixed size by using information theory to estimate adapter capacity and relate it to the size of a given dataset. This gives a way to mint PEFT modules on-the-fly of the minimum size required for a given task.

In practice, we don’t always use LoRA. We’re exploring other options for parameter-efficient learning such as trainable prefixes23, sparse memory layers24, steering vectors25, GDN state, and trainable experts26. We find the landscape of PeFT methods contains a number of tradeoffs, and different modules work better for different tasks. We’re also working on other parts of this stack, such as more accurately estimating loss without training and understanding capacity in the finite-compute regime.

On a broader scale, examining how model capacity interacts with data size provides us a clean playground to investigate fundamental questions around how models learn.

## Acknowledgements

We thank Henry Smith and Jacob Portes for their useful feedback on this post.

## Citation

Engram. “Predicting LoRA size for any dataset.” 2026.

```
@misc{engram2026loramemorize,
  author = {{Engram}},
  title = {Predicting LoRA size for any dataset},
  year = {2026},
  howpublished = {Engram blog},
  url = {https://engram.com/blog/lora-size}
}
```
## Appendix

### A: Theory

Intuitively, the capacity of the adapter is the number of bits it can store about the data it is trained on. We formalize this concept as the decrease in code length of training data, on average over the training distribution. When the training distribution differs from the base model, the decrease in code length comes from a combination of learning the *signal* — the change in distribution — and *noise* — the particulars of the individual training samples. We can simplify our study of capacity by sampling from the base model, eliminating the signal so that any reduction in code length comes from memorizing the noise in the training samples. While this is unrealistic, it gives conservative worst-case estimates of capacity per parameter, and as we will empirically demonstrate, the theory is predictive in more realistic settings as well.

Let denote the adapted model after a perturbation to the model parameters. Here, the perturbation is trained to maximize the log-likelihood of the training data. To second order, the objective is approximately,

where is the gradient (i.e., the score function) and is the Hessian. With enough samples , the Hessian is well-approximated by the negative Fisher, .

Here, is a vector obtained by raveling a matrix-valued adapter . If we optimize to find the best rank- adapter, the approximate increase in log-likelihood is,

To obtain a closed-form expression for the approximate increase in log-likelihood, we need one further assumption. As written above, the Fisher information is a matrix . If the Fisher information matrix admits a Kronecker-factorization27, , where  and  are invertible matrices, then in terms of the matrix-valued perturbation:

where is the gradient reshaped into a matrix. Now reparameterize in terms of . Then,

where and is constant in .

Maximizing subject to the rank- constraint amounts to setting to the truncated SVD of with the top singular values. With this optimal choice, the approximate increase in log-likelihood is,

The capacity of a rank- adapter is the expected increase, measured in bits. Recall that the entries of are gradients of the log-likelihood — that is, the score functions. Thus, in expectation, is mean zero and has covariance equal to the Fisher information matrix. When we multiplied by the inverse square roots to obtain the matrix , we effectively whitened the covariance so that has mean-zero, uncorrelated, unit-variance entries, and by a martingale CLT argument, they are approximately i.i.d. Gaussian. Thus, the capacity is approximately,

The singular values of  follow a predictable distribution from random matrix theory. In the limit  with , assuming w.l.o.g. that , the distribution of the  rescaled squared singular values  (equivalently, the eigenvalues of ) converges to the **Marchenko–Pastur law**29 with density

with support . Let be the upper quantile of , i.e., a fraction of the rescaled squared singular values lie above it. The first equation below is this definition. Summing the captured values, each worth , gives the second:

The approximation becomes exact as the matrix grows with fixed. At 512×1024 it is within 0.4% of the exact .

The integrals are still a bit unwieldy, but substituting leads to a closed-form expression for the curve , with :

To get at a given rank , solve the first equation for numerically (it is monotone in ) and substitute into the second.

**Low-rank limit.** For , the tail above  shrinks to the edge , so each direction captures about , and

Normalizing by the number of LoRA parameters gives

For Qwen3.5-0.8B this gives 1.40 bits per LoRA parameter for v_proj (512×1024) and 1.32 for up_proj (1024×3584 with the smaller dimension first).

### B: Experimental setup

**Optimization**. All training used the AdamW optimizer with a short linear warmup, cosine decay to zero, gradient-norm clipping, and no weight decay. All experiments are conducted on Qwen3.5-0.8B, although we expect our findings to transfer cleanly to other base models.

We use a combination of internal and external datasets:

- **Engram (XS):**internal emails (22k tokens, 6.8 KB)
- **Engram (MD):**internal research notes (2.2M tokens, 628 KB)
- **Harvey C&H (XS, SM):**documents from a synthetic law firm’s workspace (451k tokens, 94 KB; 4.0M tokens, 821 KB)
- **Developer Docs:**a public developer-documentation site (751k tokens, 131 KB)
- **swebench-django:**the Django repository’s source code from SWEBench (2.5M tokens, 550 KB)
- **MATH-500:**competition math problems with solutions (141k tokens, 20 KB)
- **GSM8K:**grade-school math word problems with solutions, train split (1.3M tokens, 236 KB)
- **Tulu 3:**2,000 samples from the Tulu 3 SFT mixture (1.8M tokens, 666 KB)
- **No Robots:**human-written instructions and responses (2.6M tokens, 1.1 MB)
- **OpenThoughts3:**250 long reasoning traces (3.9M tokens, 1.2 MB)
- **MMMLU:**multilingual MMLU questions in 14 languages, 70% subset (20.5M tokens, 8.6 MB)
- **enwik8:**the first 100 MB of English Wikipedia (27M tokens, 9.5 MB)

It can be useful to examine dataset “size” along two dimensions, *bits* and *tokens*.

- We might consider the total entropy of a dataset under – in some sense, the amount of information the dataset “has” to contribute to the model.
- We can also simply count the cumulative number of tokens.

Both metrics are useful for different cases. We visualize these datasets along both axes:
