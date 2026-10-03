---
url: https://huggingface.co/allenai/AstaBrief_8B
date_fetched: 2026-10-03
---

## AstaBrief-8B

AstaBrief-8B is a model trained to turn a research question and retrieved scientific literature excerpts into a cited report using only supervised fine-tuning and offline direct preference optimization.

The model is initialized from AstaBrief-8B-SFT and fine-tuned on AstaBrief_DPO_Mix, a collection of real user queries with two reports per query and preference judgements over those report pairs. Reports are generated from both multi-step Asta ScholarQA and single-step report generation pipelines, using a variety of backing models: Claude 3.5 Sonnet, Claude 3.7 Sonnet, o3, o4-mini, GPT-4.1, DeepSeek-V3 and DeepSeek-R1. Two judge models – GPT-4.1 and DeepSeek-R1 – compared each pair and picked a winner. We ensured that LLM judges were aligned with human preferences (95% agreement) and only kept pairs where both judges agreed.

For a detailed overview of the project, see our blog.

## Inference and Usage


Recommended prompt: This checkpoint was fine-tuned using this prompt. For best results, we recommend using the same prompt format at inference time with your input query and section references. Using a different prompt or interaction format may lead to degraded or inconsistent behavior.

Following is an example inference code snippet:

```
from transformers import AutoModelForCausalLM, AutoTokenizer
from vllm import LLM, SamplingParams
model_name = "allenai/AstaBrief_8B_SFT"
# load the tokenizer and the model
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype="auto",
    device_map="auto"
)
# prepare the model input by providing your query and retrieved literature excerpts in our recommended prompt format
messages = [
    {"role": "user", "content": formatted_sft_prompt}
]
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True,
)
# instantiate sampling parameters
llm = LLM(model=model_name)
sampling_params = SamplingParams(
    temperature=0.7,
    top_p=0.95,
    max_tokens=4096,
    stop_token_ids=[tokenizer.eos_token_id],
)
# conduct text completion
outputs = llm(text, sampling_params)
output_text = outputs[0].outputs[0].text.strip()
output_ids = len(outputs[0].outputs[0].token_ids)
print("content:", output_text)
```
## Evaluation Results

Our DPO training process improves performance over the SFT checkpoint as well as the base Qwen3-8B model on the ScholarQA-CS2 test set, a set of 100 user-written computer science research questions.

| Model | Average | Ingredient Recall | Answer Precision | Citation Precision | Citation Recall | 
|---|---|---|---|---|---|
| Qwen3-8B | 77.3 | 77.8 | 90.6 | 76.2 | 64.6 | 
| AstaBrief-8B-SFT | 83.7 | 85.2 | 90.4 | 87.7 | 71.3 | 
| AstaBrief-8B | 87 | 90.2 | 89 | 90.5 | 78.2 | 

Overall, AstaBrief-8B is competitive with Asta ScholarQA and DR-Tulu on a variety of benchmarks.

| Model | SQA-CS2 (Dev) | SQA-CS2 (Test) | DeepScholarBench | Win Rate vs Asta SQA (SQA-CS2 Dev) | Win Rate vs Asta SQA (SQA-CS2 Test) | 
|---|---|---|---|---|---|
| Asta ScholarQA | 87.6 | 86.2 | 60.25 | N/A | N/A | 
| DR-Tulu-8B | 86.5 | 88.8 | 56.26 | 36% | 54% | 
| AstaBrief-8B | 86.3 | 87.0 | 53.50 | 55% | 72% | 

## Intended Uses and Limitations

This model is licensed under Apache 2.0 and is based on Qwen 3-8B. It is intended for research and educational use in accordance with Ai2's Responsible Use Guidelines. Please refer to the details in our DPO dataset for more information about the data sources used in DPO training.

## Training

DPO training was conducted in open-instruct on 8xH100s. Hyperparameter settings:

| Hyperparameter | Value | 
|---|---|
| #Epochs | 7 | 
| LR | 5e-06 | 
| LR Scheduler | Linear | 
| Warmup Ratio | 0.1 | 
| DPO Loss Type | DPO Norm | 
| DPO Beta | 10 | 
| Per-device Batch Size | 1 | 
| Gradient Accumulation Steps | 8 | 
| Max sequence length | 16000 | 
| Precision | BF16 | 

Links

📝 Project blog: https://allenai.org/blog/astabrief

🤗 DPO dataset: AstaBrief_DPO_Mix

🤖 Base model: Qwen3-8B

🤖 SFT checkpoint: AstaBrief_8B_SFT

- Downloads last month
- 173
