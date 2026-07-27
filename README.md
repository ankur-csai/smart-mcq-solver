# Smart MCQ Solver — Top-3 Answer Ranking
 
**Deep Learning & Generative AI Project (Diploma Level, BS in Data Science & Applications, IIT Madras)**
 
| | |
|---|---|
| **Student** | Ankur `<FULL NAME>` |
| **Roll Number** | `<21fXXXXXXX>` |
| **Term** | T2-2026 |
| **Kaggle Competition** | Smart MCQ Solver Challenge |
| **Kaggle Notebook** | `DL-<rollno>-notebook-t22026` |
| **W&B Project** | [`<rollno>-t22026`](https://wandb.ai/<entity>/<rollno>-t22026) |
| **Best Public LB (MAP@3)** | `<score>` (cutoff: 0.73) |
 
---
 
## 1. Problem Statement
 
Given a question `prompt` and five candidate answers labelled `A`–`E`, predict the **three most likely correct answers in ranked order**.
 
Scored by **Mean Average Precision @ 3 (MAP@3)**: a correct answer at rank 1 scores 1.0, at rank 2 scores 0.5, at rank 3 scores 0.333, and 0 otherwise, averaged over all questions.
 
```
Ground truth: A
A B C  → 1.000
B A C  → 0.500
C D A  → 0.333
```
 
Random guessing yields (1 + 1/2 + 1/3) / 5 ≈ **0.367**, the floor any model must beat.
 
---
 
## 2. Dataset
 
| Split | Rows | Columns |
|---|---|---|
| `train.csv` | 2,000 | `id, prompt, A, B, C, D, E, answer` |
| `test.csv` | 500 | `id, prompt, A, B, C, D, E` |
 
- No missing values in either split.
- Domain skews toward physics, cosmology, and statistics.
- Prompts wrap the real question in one of five boilerplate templates (`"Pick the best possible answer: ..."`, `"Select the most accurate option: ..."`, etc.), stripped during preprocessing.
- Label distribution is mildly imbalanced: A 369 / B 490 / C 459 / D 358 / E 324.
### Length statistics (words)
 
| Field | Mean | Median | p95 | Max |
|---|---|---|---|---|
| `prompt` | 18.1 | 17 | 31 | 51 |
| options `A`–`E` | ~26 | ~23 | ~62 | 118 |
 
Combined `prompt + option` sits comfortably under 192 tokens, which sets `max_length` for all transformer models.
 
---
 
## 3. Data Quality Audit — Two Findings
 
These were discovered during EDA and materially shaped the approach. Both are documented in full in `notebooks/01_eda.ipynb` and reproduced in the report.
 
### 3.1 Train/test overlap
 
**257 of the 500 test rows (51.4%) are exact duplicates of training rows** — identical prompt, identical five options, identical ordering. A further 6 test rows share a prompt with a training row where the correct answer text still appears among the options.
 
Within the training set itself there are 183 duplicated question keys, and **zero of them carry conflicting labels** — so the lookup is unambiguous.
 
**Consequence:** a dictionary lookup alone places the true answer at rank 1 for over half the test set, contributing ≈ 0.51 MAP@3 before any model is involved. This is a property of the competition data, not of any model, and every reported score below is decomposed accordingly.
 
### 3.2 Option-length bias
 
The correct answer is systematically the longest option. Ranking the five options purely by character count descending, with no learning whatsoever:
 
- Top-1 accuracy: **40.6%** (chance = 20%)
- MAP@3: **0.578** (chance = 0.367)
This is a generation artifact of the dataset. It is used as a fallback ranker where models are unavailable, and as a **bias probe**: any model scoring near 0.578 has likely learned length rather than semantics. Model evaluation therefore reports MAP@3 both overall and on a length-controlled subset.
 
---
