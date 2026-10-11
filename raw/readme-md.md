---
url: https://huggingface.co/cross-encoder/ms-marco-MiniLM-L6-v2/blob/main/README.md
date_fetched: 2026-10-11
---

### Instructions to use cross-encoder/ms-marco-MiniLM-L6-v2 with libraries, inference providers, notebooks, and local apps. Follow these links to get started.

- Libraries
-  sentence-transformers How to use cross-encoder/ms-marco-MiniLM-L6-v2 with sentence-transformers: `from sentence_transformers import CrossEncoder model = CrossEncoder("cross-encoder/ms-marco-MiniLM-L6-v2") query = "Which planet is known as the Red Planet?" passages = [ "Venus is often called Earth's twin because of its similar size and proximity.", "Mars, known for its reddish appearance, is often referred to as the Red Planet.", "Jupiter, the largest planet in our solar system, has a prominent red spot.", "Saturn, famous for its rings, is sometimes mistaken for the Red Planet." ] scores = model.predict([(query, passage) for passage in passages]) print(scores)`
-  Transformers How to use cross-encoder/ms-marco-MiniLM-L6-v2 with Transformers: `# pip install -U transformers accelerate # Load model directly from transformers import AutoTokenizer, AutoModelForSequenceClassification tokenizer = AutoTokenizer.from_pretrained("cross-encoder/ms-marco-MiniLM-L6-v2") model = AutoModelForSequenceClassification.from_pretrained("cross-encoder/ms-marco-MiniLM-L6-v2", device_map="auto")`
- Notebooks
- Google Colab
- Kaggle

### Download README.md from cross-encoder/ms-marco-MiniLM-L6-v2: direct link, hf CLI and curl.

- Browser
- Download file 3.67 kB
-  https://huggingface.co/cross-encoder/ms-marco-MiniLM-L6-v2/resolve/main/README.md 
- Command line
-  hf download hf://cross-encoder/ms-marco-MiniLM-L6-v2/README.md 
-  curl -L -o README.md https://huggingface.co/cross-encoder/ms-marco-MiniLM-L6-v2/resolve/main/README.md 

```
license: apache-2.0
datasets:
  - sentence-transformers/msmarco
language:
  - en
base_model:
  - cross-encoder/ms-marco-MiniLM-L12-v2
pipeline_tag: text-ranking
library_name: sentence-transformers
tags:
  - transformers
```
# Cross-Encoder for MS Marco

This model was trained on the MS Marco Passage Ranking task.

The model can be used for Information Retrieval: Given a query, encode the query with all possible passages (e.g. retrieved with ElasticSearch). Then sort the passages in a decreasing order. See SBERT.net Retrieve & Re-rank for more details. The training code is available here: SBERT.net Training MS Marco

## Usage with SentenceTransformers

The usage is easy when you have SentenceTransformers installed. Then you can use the pre-trained models like this:

```
from sentence_transformers import CrossEncoder
model = CrossEncoder('cross-encoder/ms-marco-MiniLM-L6-v2')
scores = model.predict([
    ("How many people live in Berlin?", "Berlin had a population of 3,520,031 registered inhabitants in an area of 891.82 square kilometers."),
    ("How many people live in Berlin?", "Berlin is well known for its museums."),
])
print(scores)
# [ 8.607138 -4.320078]
```
## Usage with Transformers

```
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch
model = AutoModelForSequenceClassification.from_pretrained('cross-encoder/ms-marco-MiniLM-L6-v2')
tokenizer = AutoTokenizer.from_pretrained('cross-encoder/ms-marco-MiniLM-L6-v2')
features = tokenizer(['How many people live in Berlin?', 'How many people live in Berlin?'], ['Berlin has a population of 3,520,031 registered inhabitants in an area of 891.82 square kilometers.', 'New York City is famous for the Metropolitan Museum of Art.'],  padding=True, truncation=True, return_tensors="pt")
model.eval()
with torch.no_grad():
    scores = model(**features).logits
    print(scores)
```
## Performance

In the following table, we provide various pre-trained Cross-Encoders together with their performance on the TREC Deep Learning 2019 and the MS Marco Passage Reranking dataset.

| Model-Name | NDCG@10 (TREC DL 19) | MRR@10 (MS Marco Dev) | Docs / Sec | 
|---|---|---|---|
| Version 2 models | |||
| cross-encoder/ms-marco-TinyBERT-L2-v2 | 69.84 | 32.56 | 9000 | 
| cross-encoder/ms-marco-MiniLM-L2-v2 | 71.01 | 34.85 | 4100 | 
| cross-encoder/ms-marco-MiniLM-L4-v2 | 73.04 | 37.70 | 2500 | 
| cross-encoder/ms-marco-MiniLM-L6-v2 | 74.30 | 39.01 | 1800 | 
| cross-encoder/ms-marco-MiniLM-L12-v2 | 74.31 | 39.02 | 960 | 
| Version 1 models | |||
| cross-encoder/ms-marco-TinyBERT-L2 | 67.43 | 30.15 | 9000 | 
| cross-encoder/ms-marco-TinyBERT-L4 | 68.09 | 34.50 | 2900 | 
| cross-encoder/ms-marco-TinyBERT-L6 | 69.57 | 36.13 | 680 | 
| cross-encoder/ms-marco-electra-base | 71.99 | 36.41 | 340 | 
| Other models | |||
| nboost/pt-tinybert-msmarco | 63.63 | 28.80 | 2900 | 
| nboost/pt-bert-base-uncased-msmarco | 70.94 | 34.75 | 340 | 
| nboost/pt-bert-large-msmarco | 73.36 | 36.48 | 100 | 
| Capreolus/electra-base-msmarco | 71.23 | 36.89 | 340 | 
| amberoad/bert-multilingual-passage-reranking-msmarco | 68.40 | 35.54 | 330 | 
| sebastian-hofstaetter/distilbert-cat-margin_mse-T2-msmarco | 72.82 | 37.88 | 720 | 

Note: Runtime was computed on a V100 GPU.
