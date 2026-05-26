# HoneyGen — AI-Based Deceptive Data Generation for Honeypot Systems

> Feeding attackers synthetic data so convincing, they'll never know it's fake.

COMSATS University Islamabad ( AI Semester Project)
**Saman Kalsoom** (FA23-BCE-099) 

---

## What Is This?

When attackers breach a system and steal a database, they walk away with immediately usable data — real passwords, real emails, real user records. Traditional honeypots are meant to trap them, but skilled attackers see through fake data in minutes.

**HoneyGen** solves this by using AI to generate synthetic data that is statistically and linguistically indistinguishable from real data. When an attacker steals it, they get nothing but convincing garbage.

The system proves its own effectiveness: a **Random Forest discriminator** tries to tell real data from fake. When it reaches ~50% accuracy — essentially random guessing — the fakes are indistinguishable.

---

## Demo

```
$ python -m streamlit run app.py
```

Three tabs. One pipeline.

| Tab | What it does |
|-----|-------------|
| Email Generator | Generates fake corporate emails using fine-tuned GPT-Neo |
| Credential Generator | Generates fake user credentials using GaussianCopula |
| Evaluation Results | Runs the discriminator and reports accuracy |

---

## Architecture

```
Real Data (Enron Emails + Credential CSV)
                  |
                  v
      +--- Preprocessing & Cleaning ---+
      |                                 |
      v                                 v
 Email Track                    Credential Track
 ------------                   ----------------
 GPT-Neo (HuggingFace)          GaussianCopula (SDV)
 Fine-tuned on 18K emails       Learns column distributions
 T4 GPU · ~20-30 min            + cross-column correlations
      |                                 |
      +-----------------+---------------+
                        |
                        v
          Random Forest Discriminator
          (Real vs. Fake classifier)
                        |
                        v
              Streamlit GUI (app.py)
```

---

## How It Works

### Email Generation — GPT-Neo

GPT-Neo is pre-trained on billions of words. Fine-tuning it on **18,000 real Enron emails** teaches it the structure, tone, and vocabulary of corporate email specifically.

- **Training data:** Enron corpus (~30K emails, filtered to ~18K under 1,000 chars)
- **Training environment:** Google Colab T4 GPU (~20–30 min vs. 40+ hours on CPU)
- **Quality metric:** Perplexity — lower = model is more confident and accurate
- **Discriminator result:** ~50% accuracy *(varies by prompt — well-crafted prompts produce emails the classifier cannot distinguish from real ones)*

### Credential Generation — GaussianCopula

GaussianCopula (SDV) fits a statistical distribution to each column of the real credential table and learns correlations between columns — e.g. if `username = john_doe`, the email field likely contains `john`. It then samples from those learned distributions to produce synthetic rows.

This is meaningfully better than Faker, which generates each column in isolation with no learned patterns.

### Discriminator — Random Forest

The discriminator mixes real and fake data, extracts features, and tries to classify each sample. It is the ground truth for how good the fakes are.

| Accuracy | Interpretation |
|----------|---------------|
| ~100% | Fakes are obviously fake |
| ~83% | Still distinguishable — needs work |
| ~50% | Indistinguishable — target achieved |

---

## Results

| Data Type | Discriminator Accuracy | Status |
|-----------|----------------------|--------|
| Emails | ~50% (prompt-dependent) | Target achieved |
| Credentials | In progress | Improving |

> **Note on email accuracy:** Results vary based on the generation prompt. With a well-crafted prompt that matches real Enron email style, the discriminator reaches ~50% — essentially random guessing, meaning the fakes are indistinguishable.

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Language Models | HuggingFace Transformers, GPT-Neo |
| Synthetic Tabular Data | SDV (GaussianCopula), Faker |
| Discriminator | scikit-learn (Random Forest) |
| Frontend | Streamlit |
| Training | Google Colab T4 GPU |
| Language | Python 3.14 |
| Platform | Windows, VS Code |

---

## Project Structure

```
ai_proj/
├── app.py                   # Streamlit application (3 tabs)
├── Untitled1 (1).ipynb      # GPT-Neo training notebook (Colab)
├── gpt2-enron/              # Fine-tuned GPT-Neo model files
│   ├── config.json
│   ├── pytorch_model.bin
│   └── tokenizer files...
├── ctgan_credentials.csv    # GaussianCopula-generated fake credentials
└── README.md
```

---

## Getting Started

### Prerequisites

```bash
pip install streamlit transformers torch sdv scikit-learn faker pandas
```

### Run the App

```bash
python -m streamlit run app.py
```

> Use `python -m streamlit` — direct `streamlit run` may fail on Windows due to PATH issues.

### Retrain the Model (optional)

Open `Untitled1 (1).ipynb` in Google Colab, set runtime to **T4 GPU**, then run all cells.

---

## Dataset

The **Enron Email Dataset** is a collection of real internal corporate emails released publicly during the US government's investigation into Enron Corporation's 2001 accounting fraud. It contains 600,000+ emails and has been a standard NLP/ML research benchmark for over 20 years.

Available at: https://www.kaggle.com/datasets/wcukierski/enron-email-dataset

---

## Key Design Decisions

- **GaussianCopula over CTGAN** — CTGAN consistently crashed on free-tier Colab due to RAM limits. GaussianCopula is lighter and produces comparable results for tabular credential data.
- **Session state caching** — the evaluation tab reads from `st.session_state["generated_emails"]` populated by the Email Generator tab, avoiding regenerating 200 emails on every evaluation click.
- **Perplexity as training metric** — standard for LLM quality evaluation, separate from discriminator accuracy.

---

*Made with way too much Colab GPU time.*
