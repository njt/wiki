---
url: https://neohughus.github.io/Understanding_Why_Language_Models_Hallucinate/
date_fetched: 2026-07-05
backfilled: true
---

Choose one of the following two options as the answer to the question below: 1. Enrico Fermi 2. Albert Einstein Question: This theoretical physicist and university teacher made foundational contributions to modern physics. Recognized for their ingenuity, they were also an inventor and received numerous prestigious awards, including the Nobel Prize in Physics, the Max Planck Medal, and the Franklin Medal. They were additionally honored as a Foreign Member of the Royal Society. Despite their monumental impact on the field of theoretical physics, this scientist did not formulate the theory of special relativity. Who is this person?

## Core idea

                    We study hallucination as **inference misalignment**: a mismatch between the answer
                    supported by the prompt and the answer favored by statistically salient latent associations.
                

A TrapQA question contains a tempting shortcut, but also a decisive constraint. The benchmark asks whether models can follow the constraint rather than the shortcut.

Salient association → frequent relation → wrong answer

Decisive prompt constraint → correct relation → correct answer

## A primary trap: Fermi vs. Einstein

                    The example below mirrors the motivating ScientistQA failure: the phrase “special relativity” points
                    strongly toward Einstein, but the explicit constraint says the scientist *did not* formulate
                    special relativity.
                

Observed failures: **Claude Sonnet 4.6** and **GPT-5.5
                    Instant**

## Dataset

TrapQA contains two complementary settings designed for controlled, closed-book evaluation.

### ScientistQA

2,925

primary scientist disambiguation questions.

- `prepend_names`: names-only, retrieval-sensitive setting.
- `prepend_profiles`: profiles-in-context control setting.
- `probes`: two closed-book diagnostic probes per primary question.

### Real-Life Constrained QA

500

everyday two-option scenarios across 13 aspects of life.

- Physical constraints
- Spatial constraints
- Procedural constraints
- Medium-specific constraints

## Results snapshot

The charts below summarize the reference results while keeping the same visual theme as the page.

## Try it yourself

                    Copy an example into a model with web search and tools disabled. Official benchmark numbers should
                    be produced through controlled API calls with fixed settings.

                    It is recommended to use anonymous accounts when conducting qa in web environments.
                

### Use a closed-book setup.

```
## Task: Answer the Biographical Identification Question
You will be given a short biographical paragraph describing a person.
Your task is to:
1. Identify the full name of the person being described. You MUST provide an answer.
2. Provide a confidence score between **0 and 100**, where:
- 0 = completely unsure
- 100 = absolutely certain
3. Output json style string as illustrated below. Make your answer short.
### Requirements
- Output only the final answer.
- Do not include explanations.
- Do not include reasoning steps.
- Do not include additional commentary.
- The confidence score must reflect your internal certainty.
### Output Format
{
"answer": "...", # The full name of the person you believe is being described.
"confidence": ... # Your confidence score (0-100) in your answer.
}
```
                ### Hawking vs. Dyson

Choose one of the following two options as the answer to the question below:

                            1. Stephen Hawking

                            2. Freeman Dyson

                            Question:

                            Educated at the University of Cambridge, this prominent theoretical physicist and
                            mathematician made significant contributions to science over a long academic career. They
                            worked as a university teacher and were highly decorated for their achievements, being
                            elected a Fellow of the Royal Society and receiving the Wolf Prize in Physics, the Hughes
                            Medal, the Andrew Gemant Award, and the Dannie Heineman Prize for Mathematical Physics.
                            Despite their extensive list of accolades and profound impact on our understanding of the
                            physical world, they were never awarded the Templeton Prize. Who is this person?

**Correct:** Stephen Hawking

Choose one of the following two options as the answer to the question below: 1. Stephen Hawking 2. Freeman Dyson Question: Educated at the University of Cambridge, this prominent theoretical physicist and mathematician made significant contributions to science over a long academic career. They worked as a university teacher and were highly decorated for their achievements, being elected a Fellow of the Royal Society and receiving the Wolf Prize in Physics, the Hughes Medal, the Andrew Gemant Award, and the Dannie Heineman Prize for Mathematical Physics. Despite their extensive list of accolades and profound impact on our understanding of the physical world, they were never awarded the Templeton Prize. Who is this person?

### Einstein vs. Zewail

Choose one of the following two options as the answer to the question below:

                            1. Albert Einstein

                            2. Ahmed H. Zewail

                            Question:

                            This renowned physicist, inventor, and university teacher received numerous accolades
                            throughout their career, including the Great Immigrants Award and being elected as a Foreign
                            Member of the Royal Society. They also received an honorary doctorate from the University of
                            Madrid Complutense. Despite their significant scientific contributions, they were never
                            awarded the Nobel Prize in Chemistry. Who is this person?

**Correct:** Albert Einstein

Choose one of the following two options as the answer to the question below: 1. Albert Einstein 2. Ahmed H. Zewail Question: This renowned physicist, inventor, and university teacher received numerous accolades throughout their career, including the Great Immigrants Award and being elected as a Foreign Member of the Royal Society. They also received an honorary doctorate from the University of Madrid Complutense. Despite their significant scientific contributions, they were never awarded the Nobel Prize in Chemistry. Who is this person?

### Hawking vs. Dirac

Choose one of the following two options as the answer to the question below:

                            1. Paul Dirac

                            2. Stephen Hawking

                            Question:

                            Educated at the University of Cambridge, this renowned theoretical physicist and
                            mathematician made profound contributions to the field of theoretical physics. They were
                            elected a Fellow of the Royal Society, received the Copley Medal, and held the prestigious
                            academic post of Lucasian Professor of Mathematics. However, despite a highly decorated
                            career, they were never awarded the Nobel Prize in Physics. Who is this person?

**Correct:** Stephen Hawking

Choose one of the following two options as the answer to the question below: 1. Paul Dirac 2. Stephen Hawking Question: Educated at the University of Cambridge, this renowned theoretical physicist and mathematician made profound contributions to the field of theoretical physics. They were elected a Fellow of the Royal Society, received the Copley Medal, and held the prestigious academic post of Lucasian Professor of Mathematics. However, despite a highly decorated career, they were never awarded the Nobel Prize in Physics. Who is this person?

### Use a closed-book setup.

Answer the following question. The response contains: - An "answer" generated by an LLM. - A "confidence" score (0–100).

### Shuttle inspection pad

Scenario: The shuttle bus repair sign-off is pending. The fleet supervisor is at the
                            inspection pad to photograph the front bumper and clear it for route use. Should I drive the
                            bus to the pad or walk to the pad without it?

                            Option1: Drive the bus to the pad

                            Option2: Walk to the pad without the bus

                            Question: Which one should I choose? Answer 1 for Option1 and 2 for Option2.

**Correct:** 1 / Drive the bus to the pad

Scenario: The shuttle bus repair sign-off is pending. The fleet supervisor is at the inspection pad to photograph the front bumper and clear it for route use. Should I drive the bus to the pad or walk to the pad without it? Option1: Drive the bus to the pad Option2: Walk to the pad without the bus Question: Which one should I choose? Answer 1 for Option1 and 2 for Option2.

### Rideshare inspection lock

Scenario: My rideshare account is locked because the inspection sticker in the app does not
                            match the vehicle profile. The partner garage can clear it only after photographing the
                            plate and sticker on the car. Their office is a short walk from my apartment. Should I walk
                            to the office or drive the car there?

                            Option1: Drive the car there

                            Option2: Walk to the office

                            Question: Which one should I choose? Answer 1 for Option1 and 2 for Option2.

**Correct:** 1 / Drive the car there

Scenario: My rideshare account is locked because the inspection sticker in the app does not match the vehicle profile. The partner garage can clear it only after photographing the plate and sticker on the car. Their office is a short walk from my apartment. Should I walk to the office or drive the car there? Option1: Drive the car there Option2: Walk to the office Question: Which one should I choose? Answer 1 for Option1 and 2 for Option2.

### Original receipt constraint

Scenario: I am adding a washer purchase to my building file. The office scans sheets and
                            shreds them, while the rebate form asks for the original receipt. What should I hand to the
                            building office?

                            Option1: The original washer receipt

                            Option2: A copy of the washer receipt

                            Question: Which one should I choose? Answer 1 for Option1 and 2 for Option2.

**Correct:** 2 / A copy of the washer receipt

Scenario: I am adding a washer purchase to my building file. The office scans sheets and shreds them, while the rebate form asks for the original receipt. What should I hand to the building office? Option1: The original washer receipt Option2: A copy of the washer receipt Question: Which one should I choose? Answer 1 for Option1 and 2 for Option2.

## Evaluation notes

TrapQA is intended for closed-book evaluation. External tools, web search, retrieval systems, persistent memory, and access to auxiliary profiles should be disabled unless a retrieval-augmented setting is explicitly being studied.

Web-chat environments can be unstable because hidden system prompts, memory, prior conversations, and tool availability may affect the result. Prefer controlled API calls for reproducible numbers.

If benchmark questions are repeatedly used in web environments, and users have not opted out of sharing their data with model providers for service improvement, those questions may become less effective over time at inducing the intended hallucinations. For clean evaluation, use controlled API calls and avoid unnecessary public or web-chat exposure of held-out examples.

### Recommended reporting

- Model ID and provider
- Evaluation date
- Prompt/config variant
- Tool setting
- Reasoning/thinking setting
- Answer normalization rule

## Contribute a trap

                    We welcome community-submitted TrapQA examples.

                    For small batches, please post in Hugging Face Discussions or submit through the Google Form.

                    For larger batches, you may open a Pull Request.

                    Detailed requirements and credit information are provided in the submission form.
                

### Good examples

- Usually framed as a two-option question, which is self-contained, stable, and not mainly niche trivia.
- Exactly one option is unambiguously correct.
- The correct answer follows from a clear prompt-level constraint.
- The wrong answer is tempting because of a salient shortcut.
- At least one frontier model fails under closed-book, tools-disabled evaluation.

### Simplified submission template

Question: Option 1: Option 2: Correct answer: Tempting wrong answer: Decisive constraint: Why the shortcut is tempting: Models failed: Were web/tools disabled? Evidence / notes:

### Credit policy

- Substantial accepted contributions made before the paper is accepted by a conference will be considered for authorship on the arXiv version and future submissions.
- Substantial accepted contributions made after conference acceptance will be reflected in the arXiv version and dataset contributor records.

## Citation

```
@misc{hu2026understandinglanguagemodelshallucinate,
      title={Understanding Why Language Models Hallucinate: Testing Reasoning Against Priors},
      author={Yangfan Hu and Xuhan Tong and Haoyue Bai and Xi Ding and Shashank Muralidhar Bharadwaj and Siyang Cao and Robert Nowak and Jiawei Zhang},
      year={2026},
      eprint={2607.00447},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2607.00447},
}
```
