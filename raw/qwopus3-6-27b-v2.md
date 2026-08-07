---
url: https://huggingface.co/Jackrong/Qwopus3.6-27B-v2
date_fetched: 2026-08-07
---

### Instructions to use Jackrong/Qwopus3.6-27B-v2 with libraries, inference providers, notebooks, and local apps. Follow these links to get started.

- Libraries
-  Transformers How to use Jackrong/Qwopus3.6-27B-v2 with Transformers: `# Use a pipeline as a high-level helper from transformers import pipeline pipe = pipeline("image-text-to-text", model="Jackrong/Qwopus3.6-27B-v2") messages = [ { "role": "user", "content": [ {"type": "image", "url": "https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/p-blog/candy.JPG"}, {"type": "text", "text": "What animal is on the candy?"} ] }, ] pipe(text=messages)``# Load model directly from transformers import AutoProcessor, AutoModelForMultimodalLM processor = AutoProcessor.from_pretrained("Jackrong/Qwopus3.6-27B-v2") model = AutoModelForMultimodalLM.from_pretrained("Jackrong/Qwopus3.6-27B-v2", device_map="auto") messages = [ { "role": "user", "content": [ {"type": "image", "url": "https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/p-blog/candy.JPG"}, {"type": "text", "text": "What animal is on the candy?"} ] }, ] inputs = processor.apply_chat_template( messages, add_generation_prompt=True, tokenize=True, return_dict=True, return_tensors="pt", ).to(model.device) outputs = model.generate(**inputs, max_new_tokens=40) print(processor.decode(outputs[0][inputs["input_ids"].shape[-1]:]))`
- Notebooks
- Google Colab
- Kaggle
- Local Apps Settings
-  vLLM  How to use Jackrong/Qwopus3.6-27B-v2 with vLLM: ##### Install from pip and serve model`# Install vLLM from pip: pip install vllm # Start the vLLM server: vllm serve "Jackrong/Qwopus3.6-27B-v2" # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:8000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "Jackrong/Qwopus3.6-27B-v2", "messages": [ { "role": "user", "content": [ { "type": "text", "text": "Describe this image in one sentence." }, { "type": "image_url", "image_url": { "url": "https://cdn.britannica.com/61/93061-050-99147DCE/Statue-of-Liberty-Island-New-York-Bay.jpg" } } ] } ] }'`##### Use Dockerdocker model run hf.co/Jackrong/Qwopus3.6-27B-v2 
-  SGLang  How to use Jackrong/Qwopus3.6-27B-v2 with SGLang: ##### Install from pip and serve model`# Install SGLang from pip: pip install sglang # Start the SGLang server: python3 -m sglang.launch_server \ --model-path "Jackrong/Qwopus3.6-27B-v2" \ --host 0.0.0.0 \ --port 30000 # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:30000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "Jackrong/Qwopus3.6-27B-v2", "messages": [ { "role": "user", "content": [ { "type": "text", "text": "Describe this image in one sentence." }, { "type": "image_url", "image_url": { "url": "https://cdn.britannica.com/61/93061-050-99147DCE/Statue-of-Liberty-Island-New-York-Bay.jpg" } } ] } ] }'`##### Use Docker images`docker run --gpus all \ --shm-size 32g \ -p 30000:30000 \ -v ~/.cache/huggingface:/root/.cache/huggingface \ --env "HF_TOKEN=<secret>" \ --ipc=host \ lmsysorg/sglang:latest \ python3 -m sglang.launch_server \ --model-path "Jackrong/Qwopus3.6-27B-v2" \ --host 0.0.0.0 \ --port 30000 # Call the server using curl (OpenAI-compatible API): curl -X POST "http://localhost:30000/v1/chat/completions" \ -H "Content-Type: application/json" \ --data '{ "model": "Jackrong/Qwopus3.6-27B-v2", "messages": [ { "role": "user", "content": [ { "type": "text", "text": "Describe this image in one sentence." }, { "type": "image_url", "image_url": { "url": "https://cdn.britannica.com/61/93061-050-99147DCE/Statue-of-Liberty-Island-New-York-Bay.jpg" } } ] } ] }'`
-  Unsloth Studio  How to use Jackrong/Qwopus3.6-27B-v2 with Unsloth Studio: ##### Install Unsloth Studio (macOS, Linux, WSL)curl -fsSL https://unsloth.ai/install.sh | sh # Run unsloth studio unsloth studio -H 0.0.0.0 -p 8888 # Then open http://localhost:8888 in your browser # Search for Jackrong/Qwopus3.6-27B-v2 to start chatting ##### Install Unsloth Studio (Windows)irm https://unsloth.ai/install.ps1 | iex # Run unsloth studio unsloth studio -H 0.0.0.0 -p 8888 # Then open http://localhost:8888 in your browser # Search for Jackrong/Qwopus3.6-27B-v2 to start chatting ##### Using HuggingFace Spaces for Unsloth# No setup required # Open https://huggingface.co/spaces/unsloth/studio in your browser # Search for Jackrong/Qwopus3.6-27B-v2 to start chatting ##### Load model with FastModel`pip install unsloth from unsloth import FastModel model, tokenizer = FastModel.from_pretrained( model_name="Jackrong/Qwopus3.6-27B-v2", max_seq_length=2048, )`
-  Docker Model Runner  How to use Jackrong/Qwopus3.6-27B-v2 with Docker Model Runner: docker model run hf.co/Jackrong/Qwopus3.6-27B-v2 

- 💡 1. Base Model, Training Library & Cooperation 
- 📖 2. Background & Motivation 
- ⚡ 3. Reasoning Efficiency & MTP Speedup 
- 📊 4. Evaluation & Benchmarks 
- 🗺️ 5. Training & Data Pipeline Overview 
- 🎯 6. Three-Stage Curriculum Learning 
- 🎨 7. Trace Inversion Case Studies (5 Key Domains Showcase)
- 🤝 8. Collaboration & Training Details 
- ⚠️ 9. Known Training & Deployment Issues (IMPORTANT) 
- 📚 10. Resources & Guides 
- 🙏 11. Acknowledgements 
- 📖 12. Citation 

## 💡 1. Base Model, Training Library & Cooperation


Vision & Tool Calling Support: Qwopus3.6-27B-v2 natively supports vision and tool-use capabilities. To enable vision functionality, download`mmproj.gguf`from the GGUF Repository and place it in the same directory as the main`.gguf`file.


Community Release Notice: Qwopus3.6-27B-v2 is anexperimental community releaseand has not undergone complete safety evaluations or standard benchmarking. It is intended solely for research and exploration.

## 📖 2. Background & Motivation

## ⚡ 3. Reasoning Efficiency & MTP Speedup

## 📊 4. Evaluation & Benchmarks

## 🗺️ 5. Training & Data Pipeline Overview

The training process fuses **Trace Inversion** data augmentation with a **Three-Stage Curriculum Learning** pipeline. The core engineering focuses on expanding context length gradually while training on reconstructed reasoning traces to guarantee format stability.

```
       [ 🗺️ Trace Inversion: Reconstructing Distillation Workflow ]
  A. Surrogate Model Training (Trace Inverter)
     Open-source Model (GLM-5.1 / DS-V4) ──► Complete Reasoning Chain ──► [ Qwen3-235B Compression ] ──► Reasoning Bubbles
                                              │                                   │
                                              └──────────► [ Training ] ◄─────────┘
                                                   (Base: Qwen3-4B-Instruct)
                                                   (Result: Trace-Inverter-4B)
  B. Inversion Phase: Reconstructing Claude-4.7-Max
     _______________________________________________________
    |                                                       |
    |  Claude-4.7-Max API ──► Compressed Bubbles + Answer   |
    |_______________________________________________________|
                      │
                      ▼
    [ 🧠 Trace-Inverter-4B (Logic Reconstructor) ] ──► Synthetic Deep Reasoning Trace (Learnable CoT)
                      │
                      ▼
    [ 🧩 Data Splicing ] ◄────────── (Original Prompt + Response)
    (Embed reconstructed CoT in <think> tags, splicing with original prompt/response)
                      │
                      ▼
             (Result: claude-opus-4.6/4.7 inverted sets)
  C. Final SFT Curriculum Pipeline
     ___________________________________________
    |                                           |
    |          Base Model (Qwen3.6-27B)         |
    |___________________________________________|
                      │
                      ▼
    [ 📦 Phase 1: Format Inception ] ──► [ 🛠️ Phase 2: Complexity Expansion ] ──► [ 🚀 Phase 3: Long-Context SFT ]
      ( < 4096 tokens )                     ( 4096 - 8192 tokens )                 ( 8192 - 32K tokens )
      (Short-context stable format)         (Medium-complexity reasoning)          (Long/Multi-turn / 10% replay)
                      │                                                                       │
                      └─────────────────────────────┬─────────────────────────────────────────┘
                                                    ▼
                                   _____________________________________________
                                  |                                             |
                                  |   🌟 Final Model: Qwopus3.6-27B-v2          |
                                  |_____________________________________________|
```
## 🎯 6. Three-Stage Curriculum Learning

To steadily scale up the reasoning quality under long-context inference, **Qwopus3.6-27B-v2** adopts a Curriculum Learning strategy, progressively mixing longer and more complex reasoning templates:

| Curriculum Stage | Focus & Sample Characteristics | Strategy Details | 
|---|---|---|
| 📦 Stage 1: Format Inception | • Limit context within 4,096 tokens • Emphasize stable reasoning templates | Focuses on short-to-medium length, cleanly formatted reasoning samples. The primary goal is to establish a reliable, structured reasoning output format (such as auto-closing `<think>`tags), preventing premature exposure to complex chains from causing format collapse. | 
| 🛠️ Stage 2: Complexity Expansion | • Extend length to 4,096 - 8,192 tokens • Introduce high-difficulty logic samples | Gradually increases the ratio of complex reasoning chains. By aligned distillation with "teacher models" whose reasoning style distributions closely match the Qwen3.6 base, the capacity gap is controlled to achieve highly efficient knowledge transfer. | 
| 🚀 Stage 3: Long-Context SFT | • Progressively scale window up to 32K tokens • 10% high-quality short sample replay | In this stage, the model is pushed to deep reasoning scenarios under ultra-long context and multi-turn dialogues. To prevent capacity drift or degradation of short-instruction comprehension during long-text training, a 10% replay of high-quality short samples is strictly enforced. | 

## 🎨 7. Trace Inversion Case Studies (5 Key Domains Showcase)

To demonstrate how **Trace Inversion** reconstructs logical continuity and eliminates negative entropy, the following interactive panels show the contrast between raw compressed "Reasoning Bubbles" and the fully step-by-step reconstructed chain-of-thought (Learnable CoT) under 5 typical scenarios:

### 📐 Domain 1: Mathematics (Probability Calculation)

### 🚀 Domain 2: Physics (Kinematics)

### 💻 Domain 3: Coding (Algorithm Logic)

### 🧠 Domain 4: Logical Reasoning (Syllogism)

### 💡 Domain 5: Core Theory (Reasoning Bubble vs. Learnable CoT)

## 🤝 8. Collaboration & Training Details

This model is a collaborative milestone achieved with hardware engineer **Kyle Hessling**. You can follow him on X / Twitter: @KyleHessling1 to keep up with the latest hardware infrastructure and distributed training updates. 🙏

| Dimension | Details & Infrastructure | 
|---|---|
| 🖥️ Training Hardware | NVIDIA DGX Cluster / H100 / RTX 6000 Pro | 
| ⚙️ Fine-tuning Framework | Unsloth (used for highly efficient SFT of dense models and memory optimization) | 

## ⚠️ 9. Known Training & Deployment Issues (IMPORTANT)

While the 27B dense model architecture is relatively stable, certain low-level framework compatibility issues may still surface during large-scale parameter updates and complex long-context training. **It is highly recommended to monitor the following technical risk points during secondary fine-tuning and deployment:**

| Module / Component | Issue & Troubleshooting Diagnostics | 
|---|---|
| 🔀 Weight Merge (LoRA Merger) | When merging LoRA adapters back into the base model, it is highly susceptible to peak memory out-of-memory (OOM) errors. Ensure the merging host has sufficient virtual memory or perform the low-precision merge on the CPU. | 
| 🛠️ Dependency Compatibility | PEFT, Transformers 5.x fusion mode, and Unsloth patches may occasionally cause module import failures (ImportError) or weight mapping conflicts. Please align your dependency versions with those provided in our finetuning-guide repository. | 


Local Fine-Tuning & Deployment Warning: If you attempt to run secondary fine-tuning or merge adapter weights locally, please proceed with caution and be prepared to manually patch model definition files or pin dependency versions strictly.

## 📚 10. Resources & Guides

👉 **GitHub Repository: Jackrong-llm-finetuning-guide**
Access the repository to dive into the codebase and reproduce our results locally or on Google Colab.

## 🙏 11. Acknowledgements

Special thanks to:

- The Qwen team for providing the powerful Qwen3.6 base model.
- Unsloth for providing the highly efficient fine-tuning framework.
- Open-source datasets and community contributors.
- **Kyle Hessling**for the close collaboration on this project.

## 📖 12. Citation

```
@misc{jackrong_qwopus36_27b_v2,
  title        = {Qwopus3.6-27B-v2},
  author       = {Jackrong},
  year         = {2026},
  publisher    = {Hugging Face}
}
```
- Downloads last month
- 5,301
