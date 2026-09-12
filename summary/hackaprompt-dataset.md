---
title: "hackaprompt dataset"
url: https://huggingface.co/datasets/hackaprompt/hackaprompt-dataset
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - security-and-sandboxing
---

# HackAPrompt Dataset

Contains submissions from a global prompt hacking competition designed to expose systemic vulnerabilities in LLMs. Accepted at EMNLP 2023.

## Competition Setup

Evaluated Models: GPT-3 (text-davinci-003), FlanT5-XXL, ChatGPT (gpt-3.5-turbo).

## Dataset Structure

| Column | Description |
|--------|-------------|
| level | Numerical difficulty/complexity indicator |
| user_input | User's submission in response to challenge |
| prompt | Full prompt sent to model (includes user input) |
| completion | Model's generated output |
| model | Model type |
| expected_completion | Ideal/expected model output |
| token_count | Token count of user input |
| correct | Boolean: whether completion matches expected output |
| error | Boolean: processing error occurred |
| score | Numerical score (submissions platform only) |
| dataset | Source: playground_data or submission_data |
| timestamp | Submission datetime (playground data only) |

## Specifications

- Modalities: Tabular, Text
- Format: Parquet
- Language: English
- Size: 100K-1M entries, 150 MB
- License: MIT
- ArXiv: 2311.16119

## Research Applications

Understanding "in the wild" adversarial attacks on LLMs; prompt injection vulnerability research; analyzing failure modes and attack patterns; developing defenses against prompt hacking.

## Downstream Models (14+)

- protectai/deberta-v3-base-prompt-injection
- Verm1ion/injection-sentry-deberta
- llm-semantic-router/toolcall-verifier

## Citation

Schulhoff et al. "Ignore This Title and HackAPrompt: Exposing Systemic Vulnerabilities of LLMs Through a Global Prompt Hacking Competition." EMNLP 2023, Singapore.
