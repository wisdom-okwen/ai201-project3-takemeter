# TakeMeter Planning

## Community: r/nba

**Why this community:** r/nba is one of the highest-volume sports discussion subreddits, with tens of thousands of posts and comments per week ranging from stat-driven breakdowns to impulsive game reactions to bold opinion posts. The community has an internal culture around discourse quality — regulars actively distinguish between "providing receipts" (evidence-backed claims) and posting a "naked hot take." That gap between substantive and insubstantial discourse is real, contested, and visible in the text itself, which makes it an ideal classification target.

---

## Label Taxonomy

### `analysis`
**Definition:** The post makes a structured argument using specific, verifiable evidence — statistics, historical comparisons, film observations, or tactical breakdowns. The evidence would support the claim even if the opinion framing were stripped away.

**Example 1:**
> "Jokic's playoff assist-to-turnover ratio this year is 5.1:1 — better than most actual starting PGs in the bracket. The 'he turns it over too much' critique doesn't hold up when you look at the volume of touches he's handling."

**Example 2:**
> "The Lakers' bench unit is scoring 14 fewer points per game in the playoffs vs. regular season. That delta is almost entirely explained by fewer transition opportunities — their half-court offense ranks 22nd. This isn't a personnel problem, it's a scheme problem."

---

### `hot_take`
**Definition:** A bold or contrarian opinion stated confidently without substantive supporting evidence. Claims are asserted rather than argued. May use absolute language. The post takes a stance but doesn't reason toward it.

**Example 1:**
> "Curry is the most overrated player in NBA history. He had the greatest supporting cast ever assembled and still needed LeBron to hand him a ring. His legacy is built on three-point volume, not winning."

**Example 2:**
> "KD has never been a true leader. He has never built anything — he just joins winning systems and benefits from what others constructed. The media gives him credit he hasn't earned."

---

### `reaction`
**Definition:** An immediate emotional response tied to a specific real-time event: a game moment, trade announcement, injury update, or postgame quote. The post expresses a feeling or first impression rather than making an argument. Context is present-tense and event-driven.

**Example 1:**
> "HOLY SHIT THEY ACTUALLY TRADED HIM. I did not have this on my bingo card. This league is insane."

**Example 2:**
> "That call was absolutely criminal. There is no world where that's a foul. Refs deciding playoff games again."

---

## Hard Edge Cases

**Anticipated ambiguous case:**
> "LeBron's 4th-quarter scoring this season ranks 3rd in the league — the 'no clutch gene' narrative against him is statistically illiterate."

This post could be `analysis` (cites a specific verifiable stat as evidence for a refutation) or `hot_take` (accusatory framing, uses the word "illiterate," only one cherry-picked stat).

**Decision rule:** If the evidence cited is specific, verifiable, and would genuinely support the claim if you removed the opinion framing, label it `analysis`. If the "evidence" is decorative — used for rhetorical credibility rather than as part of a reasoned argument, or if it's a single cherry-picked stat with no context — label it `hot_take`. The one-stat post above sits at the boundary; the stat is real, but the framing is accusatory and no contextual argument is made. **→ `hot_take`**

**Second edge case:** Posts that react to a specific event *and* make an argument (e.g., "That trade is a disaster — they're giving up three first-round picks for a guy who's shot under 40% from three his last two seasons."). These feel like `reaction` in tone but have analytical substance. **Decision rule:** If the post contains at least one specific, verifiable piece of evidence and that evidence is used to support a claim (not just to vent), label it `analysis` regardless of emotional framing. Emotion alone doesn't make something a `reaction`.

**Additional rules refined after label stress-testing:**
- Film observations count as verifiable evidence if they are specific and self-reported with precision (e.g., "I counted 34 possessions," "watched the fourth quarter again — 2-of-9 on defended shots"). Vague impressions ("he was everywhere on defense") do not.
- Vague quantitative claims ("at least three times," "multiple games," "always does this") do not qualify as specific evidence and push a post toward `hot_take` rather than `analysis`, even if the underlying claim might be true.
- A post that activates a general thesis using a live moment as confirmation is `hot_take`, not `reaction` — even if it uses present-tense language. The key question: is the claim specific to *this moment*, or is this moment being used as proof of a larger conviction?

---

## Data Collection Plan

**Source:** Reddit posts and top-level comments from r/nba. Specifically:
- Game threads and post-game threads (rich in `reaction`)
- "Film/stat" flaired posts and comment threads on analytics posts (rich in `analysis`)
- Flair-less opinion posts and "who wins this debate" threads (rich in `hot_take`)

**Target distribution:** ~70 examples per label (210 total), aiming for at least 65 per label before splitting. This keeps no single label above 40% of the dataset.

**If a label is underrepresented after 150 examples:** I will target-search specific post types known to produce that label (e.g., top game threads for `reaction`, season-long stat posts for `analysis`) rather than accepting imbalance.

**Split:** 70% train / 15% validation / 15% test. Stratified by label so all splits have roughly equal class distribution.

---

## Evaluation Metrics

**Why accuracy alone isn't enough:** With 3 balanced classes, accuracy is a reasonable baseline signal, but it doesn't reveal *where* the model fails. A model that gets `analysis` right 90% of the time but collapses `hot_take` into `reaction` would still show decent accuracy.

**Metrics I'll use:**
- **Overall accuracy** (both fine-tuned and zero-shot baseline, on the same held-out test set)
- **Per-class F1** for all three labels — F1 handles the precision/recall tradeoff and makes per-class failures visible
- **Confusion matrix** — to see which label pairs the model conflates most often
- **Macro-averaged F1** — weights each class equally, appropriate since I'm targeting balanced class distribution

**Why macro F1 over weighted F1:** With a roughly balanced dataset, they'll be similar. But macro F1 treats each class as equally important, which matches my goal — I care as much about getting `analysis` right as `reaction`.

---

## Definition of Success

A classifier is **genuinely useful** for a real community moderation or quality-surfacing tool if it meets all three thresholds:

1. **Overall accuracy ≥ 80%** on the held-out test set
2. **Per-class F1 ≥ 0.70** for every label (no class should be systematically failing)
3. **Outperforms the zero-shot Groq baseline** on macro F1 by at least 5 percentage points

**"Good enough" for deployment:** Criteria 1–3 above, plus the confusion matrix shows no single dominant error pattern (i.e., the model isn't systematically collapsing one label into another).

**What would disqualify the model despite good accuracy:** If the model achieves >80% accuracy primarily because one class dominates (class imbalance leak), or if test data leaked into training (suspiciously high performance on a genuinely hard task).

---

## AI Tool Plan

### Label stress-testing
Before annotating, I'll prompt Claude with my three label definitions and ask it to generate 10–15 posts that deliberately sit at the boundary between two labels — especially `hot_take`/`analysis` (the hardest pair). If it produces posts I can't classify cleanly under my current decision rules, I'll tighten the rules before touching the dataset. Done before annotation begins.

### Annotation assistance
I will use Claude (via the API or Claude.ai) to pre-label a batch of ~50 examples as a sanity check on my own labels — not as ground truth, but to surface cases where my intuition diverges from what the definitions predict. I'll track which examples were pre-labeled in a `pre_labeled` column in the CSV and review every one myself before finalizing. All AI-assisted labels are human-confirmed before use.

### Failure analysis
After generating my list of wrong predictions from the fine-tuned model, I'll give the full list (text + true label + predicted label) to Claude and ask it to identify systematic patterns — e.g., "the model conflates short hot takes with reactions" or "it mislabels sarcastic analysis as hot takes." I'll then verify each proposed pattern myself against the actual examples before writing it into the evaluation report.

---

## Stretch Features (Planned)

- [ ] **Deployed interface** — a simple Gradio app that accepts a post, runs it through the classifier, and displays the label + confidence score. Will add implementation notes to this doc if attempted.
- [ ] **Error pattern analysis** — systematic pattern identification beyond individual examples.

---

## AI Usage Disclosure

This project uses AI assistance in three bounded ways (per the AI Tool Plan above): label stress-testing, annotation assistance (pre-labeling a review batch, human-confirmed), and failure analysis (pattern identification reviewed by me). All annotated labels in the final dataset are human-confirmed. The fine-tuning pipeline and evaluation are run by me without AI automation.
