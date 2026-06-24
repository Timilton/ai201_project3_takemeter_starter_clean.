# TakeMeter — Classifying Discourse on r/HBCU

A fine-tuned text classifier that reads a post from [r/HBCU](https://www.reddit.com/r/HBCU/) and labels **what the author is doing with it** — asking for help, sharing news, celebrating a win, or starting a discussion. It fine-tunes `distilbert-base-uncased` and compares against a zero-shot Groq (`llama-3.3-70b-versatile`) baseline on the same held-out test set.

The full design rationale — label boundaries, edge-case rules, metric choices — lives in [planning.md](planning.md). This README is the standalone report.

---

## What I'm measuring

The community organizes its discourse by **authorial intent**, not topic. The same subject (e.g. FAMU) appears as a personal "which school should I pick?" question, a neutral "free summer program, deadline extended" news link, and a celebratory "won 5-1, back to the title game" sports post. The classifier has to read intent and framing, not just keywords.

### Labels

| Label | Definition |
|---|---|
| `advice_seeking` | Asks the community for input on the author's **own** decision or situation. |
| `news_share` | Shares an external item about HBCUs (program, opportunity, article, culture feature) with no "we won" framing and no question about the author's own decision. |
| `achievement` | Announces or celebrates a concrete, completed accomplishment by an HBCU, its teams, or members (championship, ranking, research milestone, record) — usually third-person about the institution. |
| `discussion` | Poses a general opinion or open question meant to start a conversation; not about the author's own pending decision and not just relaying news. |

**Example posts** (real, from the subreddit):
- `advice_seeking` — "Coppin State vs Towson — I got into both… is the extra cost worth it, in y'all's opinion?"
- `news_share` — "FAMU offers free summer Ag program for teens, deadline extended to June 9."
- `achievement` — "FAMU overpowers Southern 5-1 to get back to the SWAC title game."
- `discussion` — "How are HBCUs able to give so many scholarships compared to T50s and other flagships?"

### How the boundaries are decided

Labels are mutually exclusive; ties are broken by this priority order:
**`advice_seeking`** (input on own situation?) → **`achievement`** (a concrete win is the point?) → **`discussion`** (a substantive opinion/general question?) → **`news_share`** (default external relay).

The hardest boundary is **`achievement` vs. `news_share`** — both are often third-person link posts about an institution. The rule: if it *celebrates a win/ranking/milestone* → `achievement`; if it *relays neutral info or an opportunity* → `news_share`. A competitive award framed as a win is `achievement`; a routine program launch is `news_share`. Full edge-case table in [planning.md §3](planning.md).

---

## Data

**Source:** public posts from [r/HBCU](https://www.reddit.com/r/HBCU/), collected with [scrape.py](scrape.py) (hits Reddit's public JSON endpoints; no login required, run from a residential IP). Post flair is captured as an annotation hint.

**Labeling process:**
1. `scrape.py` → `hbcu_dataset_raw.csv` (raw text, empty labels, flair in notes).
2. `prelabel.py` → an LLM pre-labels each row using the definitions above (speed-up only).
3. **Every row is reviewed and corrected by hand** against the definitions; difficult cases are logged in the `notes` column and in [planning.md §3](planning.md).
4. Final file: a single `text,label,notes` CSV uploaded to the notebook, which does the 70/15/15 train/val/test split.

**Label distribution:** _to be filled in after final collection — target is ~40+ examples per label with no label above 70%, per the Milestone 3 checkpoint. See [planning.md §4](planning.md) for the per-label collection plan and the underrepresentation fallback (most likely needed for `discussion`)._

**Three difficult annotation cases** (documented in [planning.md §3](planning.md)):
1. A "who's dorming next year? hmu" transfer post — a social/connection request with no clean home; folded into `advice_seeking`.
2. A scholarship post with a personal frame ("schools I applied to") but a *general* ask ("how can HBCUs fund so much?") → `discussion`, not `advice_seeking`.
3. A sports result posted under a "News" flair — flair says news, framing says win → `achievement`.

---

## Models

- **Baseline:** zero-shot Groq `llama-3.3-70b-versatile`, prompted with the label definitions and instructed to output only a label name. Run on the test set *before* any fine-tuning.
- **Fine-tuned:** `distilbert-base-uncased`, fine-tuned on the training split. Starting hyperparameters: 3 epochs, learning rate 2e-5, batch size 16. _Any change from these defaults, and the reason, will be noted here after training._

---

## Evaluation Report

> **Status: pending the Colab fine-tuning run.** The sections below are the report structure; numbers, the confusion matrix, and the failure analysis will be filled in from `evaluation_results.json` and `confusion_matrix.png` once the model is trained. They are intentionally left blank rather than estimated.

### Overall accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq) | _TBD_ |
| Fine-tuned DistilBERT | _TBD_ |

### Per-class metrics (fine-tuned)

| Label | Precision | Recall | F1 |
|---|---|---|---|
| advice_seeking | _TBD_ | _TBD_ | _TBD_ |
| news_share | _TBD_ | _TBD_ | _TBD_ |
| achievement | _TBD_ | _TBD_ | _TBD_ |
| discussion | _TBD_ | _TBD_ | _TBD_ |

**Headline metric is macro-F1** (not accuracy), because the classes are imbalanced and an always-majority predictor would score deceptively well. Success target: macro-F1 ≥ 0.70 with no per-class F1 below 0.55 (see [planning.md §6](planning.md)).

### Confusion matrix (fine-tuned)

_TBD — written out here as a markdown table once available; `confusion_matrix.png` committed alongside._ The taxonomy predicts errors should concentrate on the `achievement` ↔ `news_share` boundary; the matrix will test that.

### Three analyzed failures

_TBD — three specific misclassified test posts, each with: which label pair was confused, why that boundary is hard, whether it's a labeling vs. data/prompt problem, and what would fix it._

### Sample classifications

_TBD — 3–5 posts run through the fine-tuned model with predicted label and confidence, including one correct prediction with a sentence on why it's reasonable._

### Reflection: intended vs. learned

_TBD — the higher-level gap between the label definitions and the model's actual decision boundary (what it overfit to, what it missed)._

### How the evaluation was generated

Confusion matrix (saved as `confusion_matrix.png` and committed to the repo):

```python
# Confusion matrix
cm = confusion_matrix(ft_true_ids, ft_pred_ids)
disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=label_names)
fig, ax = plt.subplots(figsize=(7, 5))
disp.plot(ax=ax, cmap="Blues", colorbar=False)
ax.set_title("Fine-Tuned Model — Confusion Matrix (Test Set)")
plt.tight_layout()
plt.savefig("confusion_matrix.png", dpi=150)
plt.show()
print("✅ Saved: confusion_matrix.png  →  commit this to your repo and include in README")
```

Wrong-prediction dump used for the failure analysis (review these, pick 3 to analyze in depth above):

```python
# Print wrong predictions for your error analysis
wrong_idx = np.where(ft_pred_ids != ft_true_ids)[0]
print(f"Wrong predictions: {len(wrong_idx)} / {len(ft_true_ids)}\n")

for i, idx in enumerate(wrong_idx[:15]):
    text = test_df.iloc[idx]["text"]
    true_label = ID_TO_LABEL[ft_true_ids[idx]]
    pred_label = ID_TO_LABEL[ft_pred_ids[idx]]
    confidence = ft_probs[idx][ft_pred_ids[idx]]
    print(f"--- #{i+1} ---")
    print(f"Text:      {text[:200]}{'...' if len(text) > 200 else ''}")
    print(f"True:      {true_label}")
    print(f"Predicted: {pred_label}  (confidence: {confidence:.2f})")
    print()
```

---

## Repository

| File | Purpose |
|---|---|
| [planning.md](planning.md) | Design doc: labels, edge-case rules, data/metric/AI plans. |
| `scrape.py` | No-credentials scraper for r/HBCU public JSON. |
| `prelabel.py` | LLM pre-labeling helper (reviewed by hand afterward). |
| `hbcu_seed_labeled.csv` | 8 hand-labeled seed posts read directly from the sub. |
| `README.md` | This report. |
| `evaluation_results.json`, `confusion_matrix.png` | Colab outputs (committed after the run). |

### Reproduce
```bash
python3 scrape.py        # -> hbcu_dataset_raw.csv  (run from a residential IP)
python3 prelabel.py      # -> pre-labeled CSV; then review every row by hand
# upload the final text,label,notes CSV to the starter Colab notebook:
# Section 1 (label map + upload) -> 2 (split/tokenize) -> 5 (Groq baseline)
# -> 3 (fine-tune) -> 4 (eval + confusion matrix) -> 6 (comparison + export)
```
Label map for the notebook:
```python
label_map = {"advice_seeking": 0, "news_share": 1, "achievement": 2, "discussion": 3}
```

---

## AI Usage

This project used AI assistance at several points; per the assignment, it is disclosed here.

1. **Label-taxonomy design (Claude).** I directed Claude to evaluate an initial topic-based label idea (sports/expansion/achievements/culture) against the assignment's intent and real posts. It pushed back that those were topic categories rather than intent labels and that they failed mutual-exclusivity/coverage. After reading actual posts, the taxonomy was revised to the four intent labels here. I made the final call on every label and definition.
2. **Label stress-testing (Claude).** I gave Claude the definitions and asked for boundary posts across all label pairs. Cases it couldn't classify cleanly led me to tighten two rules: `argument`-style claims must cite *verifiable* evidence, and a throwaway "thoughts?" does not make a link post a `discussion`. (See [planning.md §7a](planning.md).)
3. **Annotation assistance (`prelabel.py`).** An LLM pre-labels each scraped row; **I review and correct every label** before training. The `pre_label` column is retained so the human-correction rate can be reported. Pre-labels are suggestions, never final.
4. **Failure analysis (planned).** Misclassified test examples will be given to an LLM to surface error patterns, each of which I verify by re-reading the cited posts before including it.

---

## Spec Reflection

**Where the spec helped:** the milestone's insistence on reading real posts before committing to labels directly caused the taxonomy to change — an initial discourse-quality scheme (analysis/hot_take/reaction, borrowed from sports subs) didn't survive contact with r/HBCU, where news-sharing and advice dominate. Following that step saved annotating 200 examples against labels that don't fit.

**Where the implementation diverged:** the spec frames data collection as primarily manual copy-paste and warns against turning it into a coding project. I used a scraper instead, because Reddit's volume and flair metadata made programmatic collection more reliable and kept the flair signal attached to each post. I kept the spirit of "stay close to the data" by hand-reviewing every label rather than trusting the scraper or the pre-labeler.
