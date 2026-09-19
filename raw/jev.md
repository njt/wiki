---
url: https://sgnt.ai/p/jev/
date_fetched: 2026-09-19
---

# You could have built Jev

*(a human wrote every word of this article, but an LLM did a picture and the pseudocode. This article is probably massively out of date if you’re reading it any time after Sep 2026)*

There’s a lot of excitement about Jev, but also a great deal of questions being asked that suggest that many people don’t really understand what Jev is and is not.

At the end of this (short, simple, illustrated) article, you should be able to reason about Jev from first principles.

## Some bullshit backstory

This article was going to be “build your own Jev with gpt-oss-120b”, but this being The Age of LLMs, in the time between thinking about that and putting some time aside to actually do it, at least four projects doing that have been released. In my defense, I have a real job, enjoy sleeping, and I’m old.

And that’s fine, it would have been a hassle to do the benchmarks myself when I can just steal them from other people.

## What is actually input and output

You already know that an LLM accepts tokens in, and produces tokens out, and that a token is usually a word fragment.

You may or may not know that LLMs use a fixed, ordered token dictionary. Some LLMs (especially in the same family) share these dictionaries. They’re created before training by a statistical analysis of the corpus, rather than being hand-built. They differ in size too, for example, gpt-oss-120b has a dictionary of 201,088 tokens, Gemma 3 has 262,208.

LLMs read and write arrays of numbers between 0 and whatever the token dictionary size (-1) is, each number representing an item in that token dictionary. Something is converting those to and from readable characters for you:

You may or may not also know that the majority of LLMs you use are *auto-regressive*, which means they produce a single token at a time. Once a token is generated, that token is then added to the end of the input, and the next token is generated using the same process.

But you don’t need to keep generating tokens. You could just generate one, then stop. A single run through the network.

Finally, you may already know that the above was a white lie. The LLM doesn’t generate a single token, it gives you back an array that gives you the probability — for every token in the dictionary — that that token is the next one in the sequence. (technically it gives you *logits*, but you can turn those into probabilities easily)

The harness the LLM is running in picks a token for you from those probabilities (how randomly is what *temperature* controls), and then converts from that token ID into characters (before adding that token to the input, and doing the next one).

## How to make a Jev-like

So you’ve got some time and tokens to burn, and you want to make your own Jev. Here’s one way to do it. You will need:

### A prompt-writer

You need to take the user’s specially crafted prompt (this taken from the Jev site):

```
Questions:
{
  "is_sandwich": {
    "type": "noul",
    "instructions": "Is `food` a sandwich?",
    "criteria": {
      "true": "A sandwich is a food dish where a filling, such as meat, cheese, vegetables, or spread, is placed between structural starch",
      "false": "The food has no bread enclosing a filling or uses only a single slice of bread, or uses a non-bread wrapper such as a tortilla, wafer, or cookie."
    }
  }
}
State:
{
  "food": "Ice cream sandwich",
  "definition": "An ice cream sandwich is a frozen dessert with a layer of ice cream between two cookies, wafers, or thin pieces of cake."
}
```
and turn it into a prompt that an LLM will understand. We use prompt-caching in this household, so we’ll write a constant prompt-start based on *Questions*:

```
Please answer the question ["Is `food` a sandwich?"] from the options below, where the criteria is:
"0": A sandwich is a food dish where a filling, such as meat, cheese, vegetables, or spread, is placed between structural starch
"1": "The food has no bread enclosing a filling or uses only a single slice of bread, or uses a non-bread wrapper such as a tortilla, wafer, or cookie."
WE WILL ONLY LOOK AT THE FIRST CHARACTER OUTPUT. YOU MUST ONLY OUTPUT A SINGLE CHARACTER CORRESPONDING TO YOUR CHOICE.
Here is the data:
```
and then we can push the state in as a dynamic part:

```
Food: "Ice cream sandwich"
Definition: "An ice cream sandwich is a frozen dessert with a layer of ice cream between two cookies, wafers, or thin pieces of cake."
```
You can *fairly* reliably push this into your favourite model, and get back just a zero or a one. Not always! Models like to chat and they like to tell you how smart they are while not following instructions.

Here’s OpenAI’s Astra being a good boi and getting it right first time:

### A parser

We *could* now get the string response back from LLM, take the first character, and complain loudly if it decides to give us some exposition first. But Jev returns probabilities, and also promises “NO HALLUCINATIONS”, so that approach won’t quite cut it.

Instead, we’ll take the returned *logit array* (kinda “token probabilities”) we described above, and pull out just the probabilities for the tokens we care about, in this case “0” and “1”:

```
# Use the model's own tokenizer, without adding special tokens.
zero_tokens = tokenizer.encode("0", add_special_tokens=false)
one_tokens  = tokenizer.encode("1", add_special_tokens=false)
assert length(zero_tokens) == 1
assert length(one_tokens) == 1
id_0 = zero_tokens[0]  # The token ID for the text "0", not ID 0!
id_1 = one_tokens[0]  # Likewise, this isn't necessarily ID 1.
logits = model.next_token_logits(prompt)  # One score per dictionary entry.
answer_logits = [logits[id_0], logits[id_1]]
```
Now these aren’t probabilities directly, but we can blindly do what everyone else does and use the *softmax function* to turn them into a probability distribution over the answers we do care about:

```
# Assume we got a _logit_ of 1.2 for the character "0", and 3.2 for "1".
                         "0"          "1"
Logits                   1.2          3.2
Subtract max (3.2)      −2.0          0.0
Exponentiate             0.135335     1.000000
Total = 0.135335 + 1.000000 = 1.135335
P("0") = 0.135335 / 1.135335 ≈ 0.1192 ≈ 11.92%
P("1") = 1.000000 / 1.135335 ≈ 0.8808 ≈ 88.08%
# Model says 88% probability that it is _NOT_ a sandwich (option "1").
```
And voila: we have a system that looks like Jev. You give it classifier tasks, and you get back simple answers with “no hallucinations”, and your output token count stays lean (so you can call them free).

(no hallucinations here doesn’t mean it can’t be wrong, it just means any answer we get back is definitely answer-shaped. This is TypeSafe’s marketing term, not mine, please don’t @ me)

## How to make a Jev competitor

None of this is all that clever, original, or secret. TheoLeeCJ/openjev does essentially exactly this, as does the unrelated ekzhang/openjev-sglang but with some more cleverness.

There are some slightly different approaches too: daseinlabs/open-jev does almost the same thing, except instead of mapping each answer to a single token like 0 or 1, it uses the whole potential response token text and combines the probability of each successive token in that answer. vinnylarouge/jevlike trains its own scoring model.

TypeSafe have released their *own* adaptor too for turning an LLM into a Jev-like. Because they want to benchmark against existing commercial models — for which they don’t have access to the raw output logit vector — they instead just ask the underlying models really nicely to choose an option as string, and ask them again if they got back something else. It’s a little more sophisticated than that (native structured outputs are used when available) but still. This doesn’t (in my opinion, or in theirs) create a particularly fair test, but I also don’t see how else they could have done it, so this also seems reasonable.

How do these compare to Jev in practice?

**TheoLeeCJ/openjev**: almost exactly the technique I suggested above. Reconstructed 102 decisions from Typesafe’s published evaluation material, and used Qwen3.5-4B as the fronting model. 84.5% agreement with the reference answers, vs 88.3% that Jev got on that subset. Speed compared against itself by reading just the single token output asking the model for JSON, it got a 5x speedup. Tells us Qwen3.5-4B can be competitive in accuracy for some questions, and there’s definitely some speedup just from asking for a single token back. results

**ekzhang/openjev-sglang**: also essentially the same single-token technique, and published *while I was writing this article* (of course). 1,000 sampled MMLU-Pro questions run against live Jev, which got 83%: Qwen3.6-35B-A3B got 59%, Qwen3.8-27B got 60.0% MMLU-Pro results, so Jev apparently much more accurate on more complicated questions. Then 3,270 BoolQ validation questions gave a much smaller gap: Qwen3.6-35B-A3B got 89% vs Jev’s 91.56% BoolQ results

**S Anand**: also published *while I was writing the article*: asked the frontier models and Jev to classify intents from BANKING77, everything via OpenRouter (using OpenRouter’s new Decisions endpoint for Jev, regular one for everything else). Just asked for raw JSON outputs from the models, didn’t use the technique above, so this is more analogous to TypeSafe’s own benchmarks. Found Jev to be a little faster and a little cheaper than DeepSeek v4.1 Flash and GPT-5.6 Luna, at the cost of a little accuracy. On the surface a bad result for Jev, although only 77 samples, and each sample is tiny (like 8 words) which is liable to response-time differences between network costs. results. I am a little skeptical of this result purely because (at the time I read it) the author says ” Jev is low-frontier not pareto optimal ” before showing it is absolutely on the pareto-frontier, but he may have fixed it by the time you read it.

## The secret Jev sauce

These results suggest Jev is pretty good at this (esp as they’re a new lab), which isn’t surprising, because Jev has been optimized to be good at this. The claimed optimizations are:

- 
A top-secret LLM architecture that makes Jev better at performing this kind of task than a general model; 
- 
A new training method “Reinforcement Learning for Calibrated Decisions (RLCD)” that makes Jev better at performing this kind of task than a general model; 
- 
“Parallel sampling”, which means they can answer several questions in one sweep, which should make it fast and cheap. 

As an outsider it is of course impossible to determine how much advantage the first two provide in quality and speed.

The really interesting point is going to be in a few days / weeks, when one of the frontier labs releases their inevitable Jev clone. The mean-girl/neck-beard/peanut-gallery take here of course is that the only clever thing Jev really does is derive a single token without generating any visible reasoning, an approach that’s so obvious four open-source projects have already done it with open models. I hope that take is wrong, but we will have to wait to find out.

(genuinely, not a hater: I am excited to see them vindicated that they’ve created something really cool)

We don’t know — and are unable to tell from the benchmarks — what sticking TheoLeeCJ/openjev or ekzhang/openjev-sglang in front of a frontier model gives you in terms of speed, accuracy, and cost. We’ll either find that this reduces Jev’s claimed advantages substantially (perhaps to zero), or we’ll find that the architecture and training differences TypeSafe have done provide a durable optimization that the frontier labs don’t currently have.

## In conclusion

You should now be able to reason about Jev.
