# AI Job Hunter — Project Plan

An assistant that analyzes AI/ML job postings and compares them with a CV:
it classifies the role and seniority, extracts required skills, measures CV–job fit,
finds skill gaps, and summarizes long postings.

**Estimated duration:** 7–8 weeks
**Course:** Hugging Face LLM Course (each phase is linked to the chapters it applies)

---

## Rules (read before every work session)

1. **One phase at a time.** Only one `phase-N-*` branch is open. No new branch before the current one is merged and tagged.
2. **Definition of Done first.** A phase is finished only when every box in its checklist is ticked. Not "almost". Not "works on my notebook".
3. **New ideas go to `IDEAS.md`.** One line, then back to work. No implementing ideas mid-phase.
4. **Gate review.** Before opening the next phase, send the supervisor: the filled checklist, the metrics, and one known weakness. Next phase opens only after the review passes.
5. **Stuck rule.** If a phase takes more than 2× its estimated time, do not skip ahead. Discuss: shrink the goal or change the approach.
6. **Small daily commits.** At least one meaningful commit per working day.

---

## Phase 0 — Setup (1–2 days)

**Goal:** a clean working environment.

- [ ] GitHub repo `ai-job-hunter` created (public), with README, `.gitignore` (Python), MIT license
- [ ] Folder structure created (see bottom of this file)
- [ ] `PLAN.md` and `IDEAS.md` committed
- [ ] Hugging Face account ready + access token saved in Colab secrets (never in code)
- [ ] Colab notebook can `pip install transformers datasets` and load a model from the Hub
- [ ] Tagged `v0.0`

---

## Phase 1 — Data (≈1 week)
**Course:** Ch1, Ch2 (overview), Ch5 (Datasets library)
**Branch:** `phase-1-data`

**Goal:** a clean, labeled dataset of AI/ML job postings.

**Definition of Done**
- [ ] ≥ 300 AI/ML job postings collected (public dataset filtered to AI roles, and/or manual collection)
- [ ] Source and license of the data written in README
- [ ] No duplicates, no empty or very short texts (documented filtering rules)
- [ ] Each posting has: `title`, `description`, `seniority` (Junior / Mid / Senior), `role` (e.g. ML Engineer / Data Scientist / LLM-NLP Engineer / Other)
- [ ] Label distribution printed and saved (know if it is imbalanced)
- [ ] 30 random examples checked by eye; error rate of labels written down
- [ ] Split into train / validation / test (e.g. 70/15/15), stratified, no leakage (same posting not in two splits)
- [ ] Saved as a Hugging Face `Dataset` (local + pushed to Hub, private is fine)
- [ ] Tagged `v0.1`

**Interview questions to be able to answer**
- How did you decide the labels? What would you do if labels were noisy?
- What is data leakage and how did you prevent it?
- Why stratified splitting?

---

## Phase 2 — Seniority / Role Classifier (≈1 week)
**Course:** Ch2, Ch3 (fine-tuning), Ch4 (sharing models)
**Branch:** `phase-2-classifier`

**Goal:** a fine-tuned encoder that predicts seniority (and optionally role) from a posting.

**Definition of Done**
- [ ] Baseline: TF-IDF + Logistic Regression, macro-F1 recorded
- [ ] DistilBERT (or similar) fine-tuned with `Trainer`
- [ ] Fine-tuned model beats the baseline on the **test** set
- [ ] Macro-F1 ≥ 0.75 on test (target may be revised at gate review after seeing data)
- [ ] Confusion matrix + 10 misclassified examples analyzed (why did it fail?)
- [ ] Handled long texts (truncation strategy explained)
- [ ] Model + model card pushed to the Hub; loads and predicts from a fresh notebook
- [ ] Tagged `v0.2`

**Interview questions**
- Encoder vs decoder vs encoder-decoder: why an encoder here?
- How does self-attention work? What does `[CLS]` represent?
- Why macro-F1 and not accuracy?
- Fine-tuning vs prompting an LLM for this task: trade-offs?

---

## Phase 3 — Skill Extraction (≈1–1.5 weeks)
**Course:** Ch6 (tokenizers), Ch7 (token classification)
**Branch:** `phase-3-skills`

**Goal:** extract skills (e.g. PyTorch, Docker, RAG, SQL) from a posting.

**Definition of Done**
- [ ] Skill list (≥ 100 skills) written as a dictionary
- [ ] Baseline: dictionary/regex matcher, precision & recall measured on a hand-labeled set
- [ ] Training data: weak labels from the dictionary, BIO format, and ≥ 100 examples corrected by hand
- [ ] Token classification model fine-tuned; labels aligned correctly with subword tokens
- [ ] Model finds at least some skills that the dictionary misses (examples shown)
- [ ] Entity-level F1 (seqeval) reported on the hand-labeled test set
- [ ] Short tokenizer experiment: how 2–3 tokenizers split skill names (e.g. "scikit-learn", "LangChain")
- [ ] Tagged `v0.3`

**Interview questions**
- BPE vs WordPiece vs SentencePiece?
- How do you align word-level labels with subword tokens?
- What is weak supervision and what are its risks?

---

## Phase 4 — CV ↔ Job Matching (≈1 week)
**Course:** Ch5 (semantic search with FAISS)
**Branch:** `phase-4-matching`

**Goal:** given a CV, rank postings by fit and list missing skills.

**Definition of Done**
- [ ] CV parsed into text (PDF → text)
- [ ] Embeddings computed with a sentence-embedding model; FAISS index built
- [ ] Chunking strategy chosen and explained (whole posting vs sections)
- [ ] Small evaluation set: 3–5 CVs × 20 postings judged by hand as good/bad fit; report Precision@5 or nDCG
- [ ] Skill-gap output: skills in the posting (from Phase 3) minus skills in the CV
- [ ] Compared at least 2 embedding models, choice justified with numbers
- [ ] Tagged `v0.4`

**Interview questions**
- What are embeddings? Why cosine similarity?
- How does a vector index (FAISS) find neighbors fast?
- How do you evaluate retrieval?
- What is RAG and where would it fit here?

---

## Phase 5 — Agent (≈1–1.5 weeks)
**Course:** — (builds on everything before)
**Branch:** `phase-5-agent`

**Goal:** an agent that answers questions like "Which junior LLM roles fit my CV, and what should I learn first?"

**Definition of Done**
- [ ] Agent loop written **by hand** (thought → tool call → observation → repeat), no framework
- [ ] Tools implemented as Python functions: `search_jobs`, `classify_posting`, `extract_skills`, `compare_cv`, `suggest_learning_plan`
- [ ] Uses structured tool calling (JSON / function-calling format of the chosen model)
- [ ] Error handling: unknown tool, invalid arguments, empty results, max-steps limit (no infinite loops)
- [ ] 10 test questions; logs show each step; success rate recorded
- [ ] (Optional, after the above) same agent rebuilt with a library, differences written down
- [ ] Tagged `v0.5`

**Interview questions**
- How does an agent work internally? Who actually runs the tool?
- ReAct vs plain function calling?
- How do you stop loops and hallucinated tool calls?

---

## Phase 6 — Summarization with LoRA (≈1 week)
**Course:** Ch7 (summarization), Ch11 (fine-tuning LLMs, chat templates, LoRA)
**Branch:** `phase-6-lora`

**Goal:** a small LLM that summarizes a long posting into 3 lines.

**Definition of Done**
- [ ] Reference summaries for ≥ 200 postings (generated by a stronger model, ≥ 30 checked by hand)
- [ ] Base small model (e.g. Qwen2.5-0.5B-Instruct) evaluated **before** fine-tuning
- [ ] SFT with LoRA (QLoRA if memory requires), using the correct chat template
- [ ] ROUGE before vs after reported + 10 side-by-side examples
- [ ] Training settings documented (rank, alpha, lr, epochs, GPU memory used)
- [ ] LoRA adapter pushed to the Hub
- [ ] Tagged `v0.6`

**Interview questions**
- How does LoRA work? Why does it save memory?
- What is quantization (8-bit / 4-bit)?
- SFT vs RLHF vs DPO?
- Why is ROUGE not enough to judge a summary?

---

## Phase 7 — Evaluation & Demo (≈0.5–1 week)
**Course:** Ch9 (Gradio, Spaces)
**Branch:** `phase-7-demo`

**Goal:** a public demo and a strong README.

**Definition of Done**
- [ ] Gradio app: upload CV + paste posting → seniority, skills, fit score, gaps, summary
- [ ] Deployed on Hugging Face Spaces and works for a stranger
- [ ] Latency per request measured
- [ ] README: problem, architecture diagram, results table (all phases), limitations, how to run
- [ ] 2-minute demo video or GIF
- [ ] Final results reviewed in a mock interview with the supervisor
- [ ] Tagged `v1.0`

---

## Repo structure

```
ai-job-hunter/
├── README.md
├── PLAN.md
├── IDEAS.md
├── data/            # small samples only; full data on the Hub
├── notebooks/       # one notebook per phase: 01_data.ipynb, 02_classifier.ipynb ...
├── src/
│   ├── data/
│   ├── classifier/
│   ├── skills/
│   ├── matching/
│   ├── agent/
│   └── summarizer/
├── app/             # Gradio app
└── reports/         # metrics, confusion matrices, gate review notes
```

---

## Gate review template (send this at the end of each phase)

```
Phase: N
Checklist: all boxes ticked? (paste it)
Main metrics:
Known weakness:
Time spent vs estimate:
New ideas added to IDEAS.md:
```
