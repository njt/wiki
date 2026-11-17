---
url: https://memory.stanford.edu/
date_fetched: 2026-11-17
---

## Memory beyond the context window

An LLM’s knowledge is fixed when training ends, yet the world it serves keeps changing. Users reveal their preferences over months, new findings overturn old ones, and agents make mistakes they should not repeat. Retraining for every change is impractical, and a frozen model forgets whatever it learns as soon as the text leaves its context window.

What a question needs is rarely known in advance. A request for energy-boosting foods may hinge on an iron deficiency mentioned months earlier in unrelated words. Placing everything in context is costly and often buries what matters, while retrieval misses information that resembles the question only weakly.

MEMORILLA instead learns to read the entire collection and distill what each question needs into a few memory tokens. Four requirements guide its design.

- Collection-wide access - Each query reads the whole collection, not a few passages chosen in advance. 
- Query-dependent recall - What memory returns depends on the question, so one collection serves many requests. 
- A fixed token budget - The model receives the same few memory tokens however large the collection grows. 
- Reusable document embeddings - Each document is encoded once, and the store grows without re-encoding the past. 

## How MEMORILLA works

MEMORILLA separates remembering from recalling. A frozen encoder stores each incoming document, such as a conversation or a book chapter, as an embedding. For each question, learned memory slots read the question and the entire store, and become memory tokens that the frozen LLM reads alongside it.

**Remember** · Session 1 · March 3

My doctor says I have iron-deficiency anemia.

I’ll keep that context in mind for future conversations.

Memory storeOne embedding per session

MEMORILLA16 learned slots

Growing the memory is cheap. Adding a document costs one encoder pass, as with a retrieval index, and the model always receives a fixed number of memory tokens, sixteen in our supervised experiments. Only the memory module is trained, in stages that move from paraphrasing Wikipedia to general question answering and summarization to the benchmarks’ training splits.

### How MEMORILLA differs from RAG

Retrieval-augmented generation (RAG) and MEMORILLA can share the same document embeddings and the same frozen LLM. They differ in how they read the store and in what reaches the model.

Question What should I eat to keep my energy up during the day?

RAG

Starts from the stored document embeddings

“Complex carbohydrates such as brown rice, quinoa and oats provide steady energy through the day.”

MEMORILLA

Starts from the same stored embeddings

“Given your iron-deficiency anemia and half-marathon training, choose iron-rich foods such as lentils and spinach.”

Up to 4,212× fewer tokens than full context

Document tokens per query on NarrativeQA

**67,385**

**3,612**

**16**

27× lower latency than full context

End-to-end latency with 512 documents

**19.9 s**

**0.74 s**

RAG gives the model the exact text of the documents it selects, which suits questions whose answer is a name it can find verbatim. MEMORILLA instead reads the whole collection, so it can connect information that resembles the question only weakly, but its fixed number of memory tokens can lose fine details such as exact names.

## Results

We test MEMORILLA on eight benchmarks spanning personalization, knowledge-intensive QA and long-document QA, with every method answering through the same frozen Qwen3-8B. MEMORILLA has the highest mean score (49.68) and leads on five benchmarks, while giving the model 394 times fewer document tokens than full context on average.

## Results on all eight benchmarks View scores ↗

| Benchmark / metric | Full context | Best baseline | MEMORILLA | 
|---|---|---|---|
| PersonaMem-v2Accuracy (SA) | 27.92 | 30.90xRAG | 44.60Best | 
| PersonalizationV4Accuracy (SA) | 59.12 | 67.88MemoRAG | 66.65 | 
| PubMedQAAccuracy | 67.00 | 69.00LLMLingua-2 | 59.00 | 
| FactKGAccuracy | 80.43 | 80.43Full context | 84.33Best | 
| NarrativeQAROUGE-L | 17.99 | 20.51MemoRAG | 19.31 | 
| TriviaQAExact match | 52.60 | 56.60MemoRAG | 57.00Best | 
| LaMP-4ROUGE-L | 14.21 | 16.10xRAG | 18.15Best | 
| LaMP-7ROUGE-L | 43.71 | 46.84xRAG | 48.37Best | 

Scores use each benchmark’s primary metric on a 0–100 scale. Higher is better. SA denotes short-answer evaluation. The reported mean combines different metrics. PersonaMem-v2 and PersonalizationV4 receive additional benchmark-specific fine-tuning.

### Personalization

A good assistant knows its user without being told twice. In PersonaMem-v2 and our synthetic PersonalizationV4, preferences are rarely stated outright and lie scattered across more than a hundred past conversations. LaMP-4 and LaMP-7 ask the model to write a headline or rewrite a tweet in the user’s own style.

The largest margin appears on PersonaMem-v2, where MEMORILLA reaches 44.60% short-answer accuracy against 30.90% for the strongest baseline, xRAG. Full context reaches only 27.92% while using 1,974 times more document tokens. On LaMP-7, MEMORILLA lifts ROUGE-L from 43.13 without memory to 48.37.

### Knowledge-intensive question answering

Some claims hold only when facts connect. Verifying the FactKG claim “AIDAstella was built by a company in Papenburg” requires linking the ship to its builder and the builder to the town.

Claim, labeled true AGM-88 HARM is produced by a company that manufactures the ALV X-1.

The triples in this claim’s collection

- AGM-88 HARM → weight → 355000.0
- AGM-88 HARM → length → 4.1
- AGM-88 HARM → diameter → 0.254
- AGM-88 HARM → type → Air-to-surface missile
- AGM-88 HARM → type → Anti-radiation missile
- AGM-88 HARM → origin → United States
- AGM-88 HARM → service → 1985
- AGM-88 HARM → designer → Texas Instruments
- AGM-88 HARM → manufacturer → Texas Instruments, then Raytheon Corporation
- AGM-88 HARM → engine → Thiokol SR113-TC-1 dual-thrust rocket engine
- Alliant Techsystems → product → AGM-88 HARM
- ALV X-1 → height → 54.0
- ALV X-1 → height → 16.4592
- ALV X-1 → diameter → 1.524
- ALV X-1 → diameter → 5.0
- ALV X-1 → rocket stages → 2
- ALV X-1 → status → Inactive
- ALV X-1 → total launches → 1
- ALV X-1 → failed launches → 1
- ALV X-1 → country origin → United States
- ALV X-1 → launch site → Mid-Atlantic Regional Spaceport
- ALV X-1 → manufacturer → Alliant Techsystems
- ALV X-1 → function → Sub-Orbital Test Vehicle
- ALV X-1 → mass → Unknown
- ALV X-1 → sites → Mid-Atlantic Regional Spaceport
- ALV X-1 → launches → 1
- ALV X-1 → mean of transportation/diameter → 1.524

and 24 further triples

RAG-50

**False**incorrect

Reads all 27 triples as text

**78.4%**accuracy on all claims

MEMORILLA

**True**correct

Reads all 27 triples into 16 memory tokens

**84.3%**accuracy on all claims

MEMORILLA verifies 84.3% of claims, ahead of the 80.4% reached with every fact in context, while using 108 times fewer document tokens. Since full context sees every fact, the gap does not come from missing facts. As claims compose more facts, RAG-50 falls from 78.7% to 51.8%, while MEMORILLA stays above 80%.

### Long-document question answering

Long documents bury answers in thousands of tokens, often phrased unlike the question. NarrativeQA asks about entire books and screenplays, and TriviaQA pairs trivia questions with Wikipedia and web documents.

MEMORILLA reaches the best TriviaQA exact match (57.0) with 743 times fewer document tokens than full context. On NarrativeQA, it gains most over RAG-50 on “why” and “how” questions, while RAG-50 wins on “who” and “which” questions, whose answers appear verbatim in the text.

## Memory for agents

Memory becomes essential when a model must learn from its own experience. In TextWorld, a frozen Qwen3-4B agent explores up to 65 rooms to complete a quest within 50 turns, while MEMORILLA condenses its growing history into eight memory tokens. Only the memory module is trained, using reinforcement learning on the reward the game provides.

Full-history prompting

Full history in context

Move 0

MEMORILLA

Full history plus 8 memory tokens

Move 0

Without memory, the agent shuttles between the start, laundromat, studio, salon and vault and runs out of turns. With MEMORILLA, it backtracks briefly between the salon, vault, study and canteen, then reaches the playroom goal through the restroom.

Both agents read the same full history, so the comparison isolates what eight memory tokens add. On held-out games, average success rises from 18.6% to 24.2% and games completed within three attempts from 25.0% to 37.5%. Untrained memory falls below the baseline, so the gain comes from learning, not extra tokens.

- Games completed within three attempts
- Full-history prompting: 25.0%MEMORILLA: 37.5%
- Average success rate
- Full-history prompting: 18.6%MEMORILLA: 24.2%
- Episodes that end before the turn limit
- Full-history prompting: 29%MEMORILLA: 42%

Held-out games with a frozen Qwen3-4B agent. Bars share a scale from 0 to 100%.

## Memory and retrieval

Learned memory complements retrieval rather than replacing it. MEMORILLA tends to win when an answer depends on how a collection fits together, while retrieval tends to win when the answer is a name it can find verbatim. On short sources such as PubMedQA, reading the text directly remains the better choice.

Each method answers questions the other misses. On FactKG, 15.9% are solved by MEMORILLA alone and 10.0% by retrieval alone. The union is an oracle bound rather than a measured hybrid, but it makes combining the two a promising next step.

## Future directions

We see memory as a path toward language models that keep improving after deployment, carrying what they learn across conversations, tasks and successive models. MEMORILLA is an early step in that direction.

Toward continually learning systems

- Assistants with continuity
- An assistant could grow with a person over years, keeping each user’s memory in a separate store that can stay on their device and be deleted on request.
- Experience that improves action
- Agents could carry lessons across tasks and avoid repeating mistakes. In TextWorld, learned memory already helps an agent escape loops within a game.
- Memory beyond one model
- Because the store holds only embeddings, it could extend to images, audio and video, and portable memory modules could let one store serve many models.
- Cumulative scientific reasoning
- Research systems could connect findings across papers, code and experiments, pairing learned memory for synthesis with retrieval for verifying exact claims.

Getting there requires memory that keeps fine details and improves with use. The nearest steps are pairing memory with retrieval, training encoders for compression, and testing transfer across model families.

**Current limits.** MEMORILLA can lose fine-grained details such as exact names. Because it writes memory directly into the LLM’s input embeddings, it requires open model weights and a memory module trained for each LLM. All experiments pair a Qwen encoder with a Qwen decoder on text alone, so transfer to other model families and modalities is untested, as are domains such as code and science beyond biomedicine. The TextWorld agent is trained and evaluated on a single game suite with episodes of at most 50 turns, so learning across environments and over long deployments remains to be shown.

## Citation

`@article{fallahpour2026memorilla,  title   = {Memorilla: Latent Semantic Memory for LLMs},  author  = {Fallahpour, Adibvafa and Idehpour, Parsa and Kothapalli, Vignesh and Mounier, Nikita and Wu, Shirley and Pardis, Shayan and Leskovec, Jure},  journal = {arXiv preprint arXiv:2610.21375},  year    = {2026}}`## References

- Côté, M.-A., et al. (2019). TextWorld: A learning environment for text-based games. Computer Games, CGW 2018 revised selected papers. Springer. 41–75. Preprint.
- Jiang, B., et al. (2025). PersonaMem-v2: Towards personalized intelligence via learning implicit user personas and agentic memory. arXiv:2512.06688.
- Salemi, A., et al. (2024). LaMP: When large language models meet personalization. ACL.
- Kim, J., et al. (2023). FactKG: Fact verification via reasoning on knowledge graphs. ACL.
- Kočiský, T., et al. (2018). The NarrativeQA reading comprehension challenge. TACL.
- Joshi, M., et al. (2017). TriviaQA: A large scale distantly supervised challenge dataset for reading comprehension. ACL.
- Jin, Q., et al. (2019). PubMedQA: A dataset for biomedical research question answering. EMNLP-IJCNLP.
- Chhikara, P., et al. (2025). Mem0: Building production-ready AI agents with scalable long-term memory. arXiv:2504.19413.
- Qian, H., et al. (2025). MemoRAG: Boosting long context processing with global memory-enhanced retrieval augmentation. ACM Web Conference. 2366–2377. Published version.
- Cheng, X., et al. (2024). xRAG: Extreme context compression for retrieval-augmented generation with one token. NeurIPS.
- Pan, Z., et al. (2024). LLMLingua-2: Data distillation for efficient and faithful task-agnostic prompt compression. Findings of ACL.
- Yang, A., et al. (2025). Qwen3 technical report. arXiv:2505.09388.
- Peng, B., et al. (2024). YaRN: Efficient context window extension of large language models. ICLR. Preprint.
- Shao, Z., et al. (2024). DeepSeekMath: Pushing the limits of mathematical reasoning in open language models. arXiv:2402.03300. Introduces GRPO.
