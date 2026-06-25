# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in r/nba — distinguishing analytical posts from hot takes and in-the-moment reactions.

---

## Community

**r/nba** is one of the highest-volume sports discussion subreddits, with hundreds of thousands of posts and comments per week. Discourse ranges from stat-driven breakdowns to impulsive game reactions to bold opinion posts with no supporting evidence. The community has a strong internal culture around discourse quality — regulars actively distinguish between "providing receipts" (evidence-backed claims) and posting a "naked hot take." That gap between substantive and insubstantial discourse is real, contested, and visible in the text itself, making it a strong fit for a classification task.

---

## Label Taxonomy

| Label | Definition |
|-------|------------|
| `analysis` | A structured argument backed by specific, verifiable evidence (stats, historical comparisons, film observations, tactical breakdowns). The evidence would support the claim even if the opinion framing were removed. |
| `hot_take` | A bold or contrarian opinion asserted confidently without substantive supporting evidence. Claims are stated, not reasoned. May use absolute language ("never," "always," "GOAT," "overrated"). |
| `reaction` | An immediate emotional response to a specific real-time event (game moment, trade, injury, postgame quote). Tied to present-tense context; expresses a feeling or first impression rather than making an argument. |

**Key decision rules:**
- If emotional framing but the evidence would stand on its own → `analysis`
- One cherry-picked stat as decoration for a large inferential leap → `hot_take`
- Post using a live moment to confirm a general career-long thesis → `hot_take`, not `reaction`
- Self-reported film observations with specific counts ("I counted 34 possessions") → qualifies as evidence → `analysis`
- Vague quantitative claims ("always," "many times," "at least a few") → `hot_take`

---

## Dataset

**Source:** Public posts and top-level comments from r/nba collected via Reddit's public JSON API and web search. Collection targeted three post types:
- Game threads and post-game threads → `reaction`
- Stat/film/analytics discussion posts → `analysis`
- Opinion posts, GOAT debates, "unpopular opinion" threads → `hot_take`

**Labeling process:** Posts were pre-labeled using Claude (AI-assisted) against the taxonomy definitions in [planning.md](planning.md), then reviewed and confirmed by the human annotator. All final labels are human-confirmed. Pre-labeled examples are flagged with `pre_labeled=true` in the CSV.

**Label distribution:**

| Label | Count |
|-------|-------|
| `analysis` | — |
| `hot_take` | — |
| `reaction` | — |
| **Total** | **—** |

*(Fill in after running Section 1 of the notebook — the `df["label"].value_counts()` output)*

**Three difficult-to-label examples:**

| Example (truncated) | Ambiguity | Decision |
|---------------------|-----------|----------|
| "LeBron's 4th-quarter scoring ranks 3rd in the league — the 'no clutch gene' narrative is statistically illiterate." | Cites one stat (real, verifiable) but accusatory framing and no contextual argument. Is it `analysis` or `hot_take`? | `hot_take` — the stat is decorative, not the foundation of a reasoned argument. The inferential leap from one split stat to a career-wide claim is too large. |
| "Draymond just got ejected and I hate it but the guy has cost them at least three postseason games over his career. It's a pattern. At some point the pattern is the story." | Triggered by a live event (reaction trigger) but asserts a career-wide pattern claim (hot_take). The count "at least three" is vague. | `hot_take` — the pattern claim is career-wide and the evidence is vague ("at least three" is not a specific count). The live event triggers the thought but doesn't make it a reaction. |
| "I can't believe that collapse. But look — they played their bench for 6 straight minutes against a closing unit and their net rating in those stretches is minus-14 on the season. This wasn't bad luck, this was coaching." | Opens as pure emotional reaction but pivots to a season-long net rating stat used to support a causal claim. | `analysis` — one specific, verifiable piece of evidence used to support a claim clears the analysis bar, even with emotional framing. |

---

## Model

**Base model:** `distilbert-base-uncased` (HuggingFace)

**Training approach:** Fine-tuned using HuggingFace `transformers` Trainer API on a Google Colab T4 GPU. A sequence classification head was added on top of the pre-trained encoder. Training ran for 3 epochs with early stopping based on validation accuracy.

**Hyperparameter decisions:**

| Hyperparameter | Value | Reasoning |
|----------------|-------|-----------|
| Learning rate | 2e-5 | Standard starting point for BERT-family fine-tuning; lower than typical classification to avoid catastrophic forgetting of pre-trained representations |
| Epochs | 3 | Small dataset (200 examples) risks overfitting quickly; 3 epochs balances learning with regularization |
| Batch size | 16 | Fits T4 GPU memory comfortably; smaller than 32 to give the optimizer more frequent updates on a small dataset |
| Weight decay | 0.01 | Light L2 regularization to reduce overfitting risk |
| Warmup steps | 50 | ~10% of training steps; helps the learning rate ramp up gently before the main training phase |

---

## Baseline Comparison

**Baseline model:** `llama-3.3-70b-versatile` via Groq API, prompted with the full label taxonomy definitions and one example per label, zero-shot (no task-specific training).

**Baseline prompt summary:** The system prompt defined all three labels with one-sentence definitions and one concrete example each. Decision rules for the three hard boundary cases (hot_take vs. analysis, reaction vs. analysis, hot_take vs. reaction) were embedded directly in the prompt. The model was instructed to output only the label name.

| Model | Overall Accuracy | Macro F1 |
|-------|-----------------|----------|
| Zero-shot Groq baseline | — | — |
| Fine-tuned DistilBERT | — | — |

*(Fill in after running Sections 4–6 of the notebook)*

---

## Evaluation Report

### Overall metrics (test set)

| Model | Accuracy | Macro F1 |
|-------|----------|----------|
| Zero-shot baseline | — | — |
| Fine-tuned DistilBERT | — | — |

### Per-class metrics — Fine-tuned model

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| `analysis` | — | — | — | — |
| `hot_take` | — | — | — | — |
| `reaction` | — | — | — | — |
| **Macro avg** | — | — | — | — |

### Per-class metrics — Baseline

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| `analysis` | — | — | — | — |
| `hot_take` | — | — | — | — |
| `reaction` | — | — | — | — |
| **Macro avg** | — | — | — | — |

### Confusion matrix — Fine-tuned model

*(Replace dashes with actual numbers from `confusion_matrix.png` after running Section 4)*

| | Predicted: analysis | Predicted: hot\_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | — | — | — |
| **True: hot\_take** | — | — | — |
| **True: reaction** | — | — | — |

See [outputs/confusion_matrix.png](outputs/confusion_matrix.png) for the visual version.

### Wrong predictions — Analysis

*(After running Section 4, review the "wrong predictions" cell output and pick 3 for deep analysis. Use the questions below as a guide.)*

**Error 1:**
- **Text:** —
- **True label:** — | **Predicted:** —
- **Analysis:** *(Which labels are being confused? Why is that boundary hard? Is this a labeling problem or a data distribution problem? What would need to change to fix it?)*

**Error 2:**
- **Text:** —
- **True label:** — | **Predicted:** —
- **Analysis:** —

**Error 3:**
- **Text:** —
- **True label:** — | **Predicted:** —
- **Analysis:** —

**Dominant error pattern:** *(After reviewing all wrong predictions, identify if there's a systematic pattern — e.g., "the model consistently conflates short hot_take posts with reactions" or "it mislabels analysis posts that use emotional framing.")*

### Sample classifications

*(Run 3–5 posts through your fine-tuned model after training, capture the predicted label and softmax confidence, and fill this in.)*

| Post (truncated to 150 chars) | True label | Predicted | Confidence | Correct? |
|-------------------------------|------------|-----------|------------|----------|
| — | — | — | — | — |
| — | — | — | — | — |
| — | — | — | — | — |
| — | — | — | — | — |
| — | — | — | — | — |

**On the correctly predicted example:** *(Explain in 1–2 sentences why the prediction is reasonable — what signal in the text likely drove the correct classification.)*

---

## Reflection: What the Model Learned vs. What I Intended

*(Write after seeing the evaluation results. Address: What did the model overfit to? What surface-level patterns did it learn instead of the structural distinctions you defined? What does the confusion matrix tell you about which boundary it never fully learned?)*

---

## Spec Reflection

**One way the spec helped:** The spec's framing of "what makes labels too vague" — specifically the example of `good` vs. `bad` labels failing because "quality is in the eye of the beholder" — pushed me toward defining labels in terms of structural features of the text (evidence present/absent, event-tethered/not) rather than quality judgments. That made the taxonomy considerably more consistent to apply.

**One way implementation diverged:** The spec suggested collecting data "manually (copy-paste into a spreadsheet)" over 1–2 hours. I used AI-assisted collection (Reddit JSON API fetches + pre-labeling via Claude, with human review) both because it enabled better coverage across all three label types simultaneously and because the spec explicitly permits pre-labeling with disclosure. This produced a more balanced dataset than manual spot-collection would have, but meant I spent more time reviewing labels than generating them.

---

## AI Usage

**Instance 1 — Label stress-testing:** Before annotating, I prompted Claude with all three label definitions and asked it to generate 15 posts deliberately sitting at the two hardest boundaries (hot_take/analysis and hot_take/reaction). Several of those cases couldn't be classified cleanly under my original rules, which led to two specific additions: (a) film observations with specific counts qualify as verifiable evidence, and (b) vague quantitative claims ("at least three," "many times") don't. I added these to planning.md's decision rules before annotating a single example.

**Instance 2 — Data collection and pre-labeling:** I used a multi-agent workflow to fetch real r/nba posts from Reddit's public JSON API and search results, with agents pre-labeling each post against the taxonomy definitions. I reviewed every pre-labeled example before finalizing the dataset and corrected labels where I disagreed with the agent's classification. All pre-labeled examples are flagged in the CSV (`pre_labeled=true`). See [planning.md](planning.md) for the full AI Tool Plan.

**Instance 3 — Failure pattern analysis:** After running the fine-tuned model, I pasted the full wrong-predictions list into Claude and asked it to identify systematic patterns before writing the evaluation report. I then verified each proposed pattern myself by re-reading the specific examples before including it in the analysis.

---

## Files

| File | Description |
|------|-------------|
| [planning.md](planning.md) | Label taxonomy, decision rules, data plan, evaluation criteria, AI tool plan |
| [data/takemeter_labeled.csv](data/takemeter_labeled.csv) | Annotated dataset (train/val/test split handled by notebook) |
| [outputs/evaluation_results.json](outputs/evaluation_results.json) | Full evaluation metrics from Colab |
| [outputs/confusion_matrix.png](outputs/confusion_matrix.png) | Confusion matrix visualization |
| [ai201_project3_takemeter_starter_clean.ipynb](ai201_project3_takemeter_starter_clean.ipynb) | Training notebook (run in Google Colab with T4 GPU) |
