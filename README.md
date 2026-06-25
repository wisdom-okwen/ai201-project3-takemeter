# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in r/nba — distinguishing analytical posts from hot takes and in-the-moment reactions.

---

## Community

**r/nba** — one of the highest-volume sports discussion subreddits. Discourse ranges from stat-driven breakdowns to impulsive game reactions to bold opinion posts. The community has an internal culture around discourse quality ("providing receipts" vs. "posting a naked hot take"), making it a strong fit for a classification task.

---

## Label Taxonomy

| Label | Definition |
|-------|------------|
| `analysis` | A structured argument backed by specific, verifiable evidence (stats, historical comparisons, film observations). Evidence would support the claim even without opinion framing. |
| `hot_take` | A bold or contrarian opinion asserted confidently without substantive supporting evidence. Claims are stated, not reasoned. |
| `reaction` | An immediate emotional response to a specific real-time event (game moment, trade, injury). Tied to present-tense context; expresses feeling rather than argument. |

---

## Dataset

**Source:** Public posts and top-level comments from r/nba.

**Collection approach:**
- Game threads and post-game threads → `reaction`
- Film/stat flaired posts and analytics discussion threads → `analysis`
- Opinion posts and "who wins this debate" threads → `hot_take`

**Label distribution:**

| Label | Count |
|-------|-------|
| `analysis` | TBD |
| `hot_take` | TBD |
| `reaction` | TBD |
| **Total** | **≥ 200** |

**Labeling process:** Examples collected manually from Reddit, annotated by primary annotator (me) using the decision rules in [planning.md](planning.md). Approximately 50 examples were pre-labeled by Claude (AI-assisted) as a sanity check, then confirmed or corrected by me before inclusion. All final labels are human-confirmed.

**Difficult cases:**

| Example | Difficulty | Decision |
|---------|------------|----------|
| TBD after annotation | TBD | TBD |
| TBD | TBD | TBD |
| TBD | TBD | TBD |

---

## Model

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training approach:** Fine-tuned on labeled dataset using HuggingFace `transformers` + `Trainer` API on a Colab T4 GPU. Sequence classification head added on top of the pre-trained encoder.

**Key hyperparameter decisions:**
- TBD (learning rate, epochs, batch size — to be filled after training)

---

## Baseline Comparison

Zero-shot baseline: `llama-3.3-70b-versatile` via Groq API, prompted with label definitions and asked to classify each test example without task-specific training.

| Model | Overall Accuracy | Macro F1 |
|-------|-----------------|----------|
| Zero-shot Groq baseline | TBD | TBD |
| Fine-tuned DistilBERT | TBD | TBD |

---

## Evaluation Report

*To be filled after training and evaluation.*

### Overall metrics (test set)

| Model | Accuracy | Macro F1 |
|-------|----------|----------|
| Baseline | TBD | TBD |
| Fine-tuned | TBD | TBD |

### Per-class metrics (fine-tuned model)

| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| `analysis` | TBD | TBD | TBD |
| `hot_take` | TBD | TBD | TBD |
| `reaction` | TBD | TBD | TBD |

### Confusion matrix

*To be added: confusion_matrix.png from Colab output.*

### Wrong predictions (analysis)

| Example | True label | Predicted | Why the model got it wrong |
|---------|------------|-----------|---------------------------|
| TBD | | | |
| TBD | | | |
| TBD | | | |

### Reflection

*What the model learned vs. what I intended it to learn — to be written after evaluation.*

---

## Running the Interface

*To be filled if the deployed interface stretch feature is completed.*

---

## Files

| File | Description |
|------|-------------|
| `planning.md` | Label taxonomy, data plan, evaluation criteria, AI tool plan |
| `data/takemeter_labeled.csv` | Annotated dataset (train/val/test split included) |
| `outputs/evaluation_results.json` | Full evaluation metrics from Colab |
| `outputs/confusion_matrix.png` | Confusion matrix visualization |

---

## AI Usage

This project uses AI assistance in three bounded ways: (1) label stress-testing before annotation, (2) annotation assistance for ~50 pre-labeled examples (all human-confirmed), and (3) failure pattern analysis after evaluation. The fine-tuning pipeline and all final labeling decisions are human-driven. See [planning.md](planning.md) for details.
