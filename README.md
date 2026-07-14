# Toxic Comment Classification using LLM Prompt Engineering

**MSc Business Analytics | Large Language Models | University of Edinburgh Business School (2025–26)**
**Group project — my role: Project Manager + EDA & Data Sampling**

---

## What this project is about

Online platforms need tools to automatically detect harmful comments — but toxic content doesn't come in one flavour. A comment can be obscene, threatening, identity-based hate speech, or several of these at once. This makes it a **multi-label classification problem**, where each comment can carry up to six labels simultaneously: `toxic`, `severe_toxic`, `obscene`, `threat`, `insult`, `identity_hate`.

We used the [Jigsaw Toxic Comment Classification dataset](https://www.kaggle.com/competitions/jigsaw-toxic-comment-classification-challenge) to test whether GPT-4.1, guided by prompt engineering alone (no fine-tuning), could match or outperform BERT models that were trained specifically on this task.

**The short answer: yes — and prompt design mattered more than prompt complexity.**

---

## My contribution

This was a group project. I led the project as **Project Manager** and personally owned the **EDA and data sampling** work, which formed the foundation every other part of the pipeline depended on.

**What that actually meant in practice:**

- I identified early on that our initial results were poor not because of the model, but because of how we sampled the data. Stratified sampling was underrepresenting minority label combinations (e.g. `threat`, `identity_hate`), so the models were never seeing enough of these to classify them reliably. I redesigned the sampling strategy into a custom **stress test evaluation set** — see `notebooks/EDA_and_data_sampling.ipynb`.
- I also ensured the few-shot pool (used for RAG retrieval) had zero overlap with the evaluation set, which would otherwise have caused data leakage and inflated results.
- Throughout the project I acted as the bridge between the team members working on prompt engineering and those less familiar with LLM concepts. When RAG pipelines got complex and the group got lost, I mapped out the full pipeline in plain terms so everyone could contribute meaningfully to final decisions.

The notebooks for methods M1–M4 and the BERT baselines were developed by other team members. I reviewed and was involved in all key decisions, but those notebooks are their primary work.

---

## Key finding

> Prompt quality matters more than prompt complexity.

Zero-shot GPT-4.1 (M1, simplest method) achieved a **micro-F1 of 0.821** — outperforming Toxic-BERT (0.490) by 67.5% despite no task-specific training. Adding 41 static few-shot examples (M2) actually *hurt* performance (micro-F1 dropped to 0.786) due to prompt overload and imbalanced example selection. Dynamic RAG retrieval (M3) recovered this by retrieving semantically relevant examples per query (micro-F1: 0.827). The most engineered method (M4: HA-RAG) achieved the best overall score (micro-F1: 0.829) — but only marginally better than M1, at significantly higher cost and complexity.

| Method | Approach | Micro F1 | Cost/sample |
|---|---|---|---|
| BERT-Base Cased | Zero-shot transfer | 0.295 | — |
| Toxic-BERT | Fine-tuned on Jigsaw | 0.490 | — |
| M1: Zero-Shot | GPT-4.1, no examples | 0.821 | $0.0006 |
| M2: Few-Shot | GPT-4.1, 41 static examples | 0.786 | $24.09 total |
| M3: RAG Dynamic | GPT-4.1, 7 retrieved examples | 0.827 | $11.59 total |
| M4: HA-RAG | GPT-4.1, hybrid augmentation | 0.829 | $7.82 total |

---

## Repo structure

```
toxic-comment-classification/
│
├── README.md                               ← you are here
├── reflection.md                           ← personal takeaways from the project
├── prompt_templates.md                     ← all prompt templates with design rationale
├── VIBE_CODING_PROMPTS.md                  ← AI use statement and structured prompt log
│
├── notebooks/
│   ├── EDA_and_data_sampling.ipynb         ← MY WORK: class imbalance analysis + stress test sampling
│   ├── BERT_baseline.ipynb                 ← BERT-Base Cased and Toxic-BERT baselines
│   ├── METHOD1_zero_shot.ipynb             ← GPT-4.1 zero-shot
│   ├── METHOD2_few_shot.ipynb              ← GPT-4.1 with 41 static examples
│   ├── METHOD3_RAG_dynamic.ipynb           ← GPT-4.1 with cosine similarity retrieval
│   └── METHOD4_RAG_hybrid.ipynb            ← GPT-4.1 with hybrid augmentation pipeline
│
└── data/
    └── README.md                           ← how to download the Kaggle data
```

---

## How to run this

### 1. Get the data

The Jigsaw dataset is too large to include in this repo. Download `train.csv` from Kaggle:
[https://www.kaggle.com/competitions/jigsaw-toxic-comment-classification-challenge/data](https://www.kaggle.com/competitions/jigsaw-toxic-comment-classification-challenge/data)

Place it in the `data/` folder, then run `notebooks/EDA_and_data_sampling.ipynb` first — this generates the two files every other notebook depends on:
- `stress_test_eval_set.csv` — the 8,000-row evaluation set
- `few_shot_pool_fixed.csv` — the 151k-row RAG retrieval pool (no leakage)

### 2. Install dependencies

```bash
pip install openai tiktoken pandas numpy scikit-learn transformers torch seaborn matplotlib
```

### 3. Set your OpenAI API key

```bash
# macOS / Linux
export OPENAI_API_KEY="sk-..."

# Windows
set OPENAI_API_KEY=sk-...
```

> Never hardcode API keys in notebooks. Each notebook reads the key using `os.getenv("OPENAI_API_KEY")`.

### 4. Update data paths

The notebooks contain absolute paths from the original development environment. Before running, update these to point to your local `data/` folder.

---

## Dataset

**Source:** [Jigsaw Toxic Comment Classification Challenge](https://www.kaggle.com/competitions/jigsaw-toxic-comment-classification-challenge) via Kaggle

The raw dataset contains ~160k Wikipedia comments labelled across six toxicity categories. The dataset is heavily imbalanced — `threat` and `identity_hate` represent the rarest categories, which drove our sampling design decisions.

---

## Tools and models used

- **GPT-4.1** via OpenAI API (temperature=0, seed=42 for reproducibility)
- **BERT-Base-Cased** and **Toxic-BERT** (unitary/toxic-bert) via HuggingFace
- Python: pandas, numpy, scikit-learn, matplotlib, seaborn, openai, tiktoken, transformers, torch

---

*This repository reflects my individual portfolio entry for a group project. Code in M1–M4 notebooks was developed primarily by other team members; the EDA and sampling notebook is my own work.*
