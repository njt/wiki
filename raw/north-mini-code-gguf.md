---
url: https://huggingface.co/heimann/North-Mini-Code-1.0-GGUF
date_fetched: 2026-07-05
backfilled: true
---

### Instructions to use heimann/North-Mini-Code-1.0-GGUF with libraries, inference providers, notebooks, and local apps. Follow these links to get started.

- Libraries
-  llama-cpp-python How to use heimann/North-Mini-Code-1.0-GGUF with llama-cpp-python: # !pip install llama-cpp-python from llama_cpp import Llama llm = Llama.from_pretrained( repo_id="heimann/North-Mini-Code-1.0-GGUF", filename="north-mini-code-Q4_K_M.gguf", ) llm.create_chat_completion( messages = "No input example has been defined for this model task." ) 
- Notebooks
- Google Colab
- Kaggle
- Local Apps Settings
-  llama.cpp  How to use heimann/North-Mini-Code-1.0-GGUF with llama.cpp: ##### Install (macOS, Linux)curl -LsSf https://llama.app/install.sh | sh # Start a local OpenAI-compatible server with a web UI: llama serve -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M # Run inference directly in the terminal: llama cli -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Install from WinGet (Windows)winget install llama.cpp # Start a local OpenAI-compatible server with a web UI: llama serve -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M # Run inference directly in the terminal: llama cli -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Use pre-built binary# Download pre-built binary from: # https://github.com/ggerganov/llama.cpp/releases # Start a local OpenAI-compatible server with a web UI: ./llama-server -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M # Run inference directly in the terminal: ./llama-cli -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Build from source codegit clone https://github.com/ggerganov/llama.cpp.git cd llama.cpp cmake -B build cmake --build build -j --target llama-server llama-cli # Start a local OpenAI-compatible server with a web UI: ./build/bin/llama-server -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M # Run inference directly in the terminal: ./build/bin/llama-cli -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Use Dockerdocker model run hf.co/heimann/North-Mini-Code-1.0-GGUF:Q4_K_M 
- LM Studio
- Jan
-  Ollama  How to use heimann/North-Mini-Code-1.0-GGUF with Ollama: ollama run hf.co/heimann/North-Mini-Code-1.0-GGUF:Q4_K_M 
-  Unsloth Studio  How to use heimann/North-Mini-Code-1.0-GGUF with Unsloth Studio: ##### Install Unsloth Studio (macOS, Linux, WSL)curl -fsSL https://unsloth.ai/install.sh | sh # Run unsloth studio unsloth studio -H 0.0.0.0 -p 8888 # Then open http://localhost:8888 in your browser # Search for heimann/North-Mini-Code-1.0-GGUF to start chatting ##### Install Unsloth Studio (Windows)irm https://unsloth.ai/install.ps1 | iex # Run unsloth studio unsloth studio -H 0.0.0.0 -p 8888 # Then open http://localhost:8888 in your browser # Search for heimann/North-Mini-Code-1.0-GGUF to start chatting ##### Using HuggingFace Spaces for Unsloth# No setup required # Open https://huggingface.co/spaces/unsloth/studio in your browser # Search for heimann/North-Mini-Code-1.0-GGUF to start chatting 
-  Pi  How to use heimann/North-Mini-Code-1.0-GGUF with Pi: ##### Start the llama.cpp server# Install llama.cpp: brew install llama.cpp # Start a local OpenAI-compatible server: llama serve -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Configure the model in Pi`# Install Pi: npm install -g @mariozechner/pi-coding-agent # Add to ~/.pi/agent/models.json: { "providers": { "llama-cpp": { "baseUrl": "http://localhost:8080/v1", "api": "openai-completions", "apiKey": "none", "models": [ { "id": "heimann/North-Mini-Code-1.0-GGUF:Q4_K_M" } ] } } }`##### Run Pi# Start Pi in your project directory: pi 
-  Hermes Agent new How to use heimann/North-Mini-Code-1.0-GGUF with Hermes Agent: ##### Start the llama.cpp server# Install llama.cpp: brew install llama.cpp # Start a local OpenAI-compatible server: llama serve -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Configure Hermes# Install Hermes: curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash hermes setup # Point Hermes at the local server: hermes config set model.provider custom hermes config set model.base_url http://127.0.0.1:8080/v1 hermes config set model.default heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Run Hermeshermes 
- Atomic Chat new
-  OpenClaw new How to use heimann/North-Mini-Code-1.0-GGUF with OpenClaw: ##### Start the llama.cpp server# Install llama.cpp: brew install llama.cpp # Start a local OpenAI-compatible server: llama serve -hf heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Configure OpenClaw# Install OpenClaw: npm install -g openclaw@latest # Register the local server and set it as the default model: openclaw onboard --non-interactive --mode local \ --auth-choice custom-api-key \ --custom-base-url http://127.0.0.1:8080/v1 \ --custom-model-id "heimann/North-Mini-Code-1.0-GGUF:Q4_K_M" \ --custom-provider-id llama-cpp \ --custom-compatibility openai \ --custom-text-input \ --accept-risk \ --skip-health ##### Run OpenClawopenclaw agent --local --agent main --message "Hello from Hugging Face" 
-  Docker Model Runner  How to use heimann/North-Mini-Code-1.0-GGUF with Docker Model Runner: docker model run hf.co/heimann/North-Mini-Code-1.0-GGUF:Q4_K_M 
-  Lemonade  How to use heimann/North-Mini-Code-1.0-GGUF with Lemonade: ##### Pull the model# Download Lemonade from https://lemonade-server.ai/ lemonade pull heimann/North-Mini-Code-1.0-GGUF:Q4_K_M ##### Run and chat with the modellemonade run user.North-Mini-Code-1.0-GGUF-Q4_K_M ##### List all available modelslemonade list 

# North-Mini-Code-1.0 GGUF

GGUF quantizations of CohereLabs/North-Mini-Code-1.0 (30B-A3B MoE, `cohere2_moe` architecture).

**Requires speedy-llama** until cohere2_moe support merges into upstream llama.cpp (PR #24260). These files use the standard GGUF keys and are expected to load on upstream once that PR lands.

| File | Size | Wikitext-2 PPL | Same top token as bf16 | 24GB card | 
|---|---|---|---|---|
| Q4_K_M | 18.6 GB | 8.34 (+3.2% vs bf16) | 90.4% (mean KLD 0.049) | ~230 tok/s, fully offloaded | 
| Q5_K_M | 21.7 GB | 8.19 (+1.3% vs bf16) | 93.4% (mean KLD 0.023) | ~211 tok/s, fully offloaded (8K ctx) | 

bf16 baseline PPL 8.09; measured over 64x512-token chunks of wikitext-2-raw test, KL-divergence computed against bf16 logits on the same tokens.

```
llama-cli -m north-mini-code-Q4_K_M.gguf --jinja -ngl 99 --temp 1.0 --top-p 0.95
```
Sampling params per the model card. Converted from the bf16 safetensors release; chat template embedded.

- Downloads last month
- 771

4-bit

5-bit

## Model tree for heimann/North-Mini-Code-1.0-GGUF

Base model

CohereLabs/North-Mini-Code-1.0
