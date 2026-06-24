# TakeMeter — Project Planning

A classifier that reads a post from r/HBCU and labels **what the author is doing with it** — asking for help, sharing news, celebrating a win, or starting a discussion.

> **Grounding note:** Labels and examples below are drawn from posts actually read on [r/HBCU](https://www.reddit.com/r/HBCU/), not from assumptions about how the community "should" talk. An earlier draft used discourse-quality labels (argument / lived_experience) borrowed from sports-debate subs; reading real posts showed those barely occur here, while news-sharing dominates. The taxonomy was revised to fit the data.

---

## 1. Community

**Subreddit:** [r/HBCU](https://www.reddit.com/r/HBCU/) — "Your Digital Yard, where HBCU culture meets community."

A text-heavy community of students, prospective students, alumni, and supporters of Historically Black Colleges and Universities. Reading the feed, the posts organize cleanly around a few flairs that the community itself uses: **Advice Needed 🗣️**, **News 📰**, **HBCU Sports 🏈**, **Transfer 🔄**, and **Academics**.

**Why it's a good classification fit:** the discourse varies by *what the author wants from the post*, not by topic. The same subject (say, FAMU) shows up as a personal "which school should I pick?" question, a neutral "free summer program, deadline extended" news link, and a celebratory "won 5-1, back to the title game" sports post. A model has to read **intent and framing**, not just keywords — three FAMU posts, three different labels. That's the signal TakeMeter is meant to measure.

---

## 2. Labels

Four labels, each defined by **authorial intent**. Examples marked *(real)* are taken verbatim/near-verbatim from the subreddit; *(illustrative)* are constructed in the same mold and will be confirmed against real posts during collection.

### `advice_seeking`
The post asks the community for input on the author's **own** decision or situation.
- *(real)* "Coppin State vs Towson — I got into both, want to be an archaeologist… is the extra cost worth it or not, in y'all's opinion?"
- *(real)* "Spelman vs American — I'm a senior, committed to American but just got into Spelman and I'm so torn, pls help."

### `news_share`
The post shares an external item about HBCUs — a program, opportunity, article, or culture feature — without a "we won" framing and without asking for the author's own decision. Usually a link post.
- *(real)* "FAMU offers free summer Ag program for teens, deadline extended to June 9." *(News, link)*
- *(real)* "The Man That Put Grambling's Culture Into the Spotlight." *(News, link — culture feature)*

### `achievement`
The post announces or celebrates a concrete, completed accomplishment by an HBCU, its teams, or its members — a championship, ranking, research milestone, or record. These are **overwhelmingly third-person about the institution**; that framing is the model's main signal.
- *(real)* "FAMU overpowers Southern 5-1 to get back to the SWAC title game." *(HBCU Sports, link)*
- *(illustrative)* "Spelman ranked #1 HBCU by US News for the 19th straight year."

### `discussion`
The post poses a general opinion or open question meant to start a conversation — not about the author's own pending decision, and not just relaying news.
- *(real)* "How are HBCUs able to give so many scholarships compared to T50s and other flagships? The endowments at larger schools are obviously much bigger, so how can these smaller HBCUs provide so much?" *(Academics)*
- *(illustrative)* "Does the HBCU-vs-PWI debate actually matter for STEM careers, or is it overblown?"

---

## 3. Hard Edge Cases

Three boundary zones, each with an explicit rule. Apply in the §3 priority order when more than one seems to fit.

| Boundary | The trap | Decision rule |
|---|---|---|
| **achievement vs. news_share** *(the hardest — see below)* | Both are often third-person link posts about an institution. In the real feed, posts 2 and 8 came from the **same author and the same blog**: one a program announcement, one a game result. | If the post **celebrates a win/ranking/milestone** ("won 5-1", "ranked #1", "record funding") → `achievement`. If it relays **neutral info, an opportunity, or a feature** with no win framing → `news_share`. |
| **advice_seeking vs. discussion** | Both can be phrased as questions. | If the author wants input on **their own** decision/situation → `advice_seeking`. If it's a **general** question posed to the community for debate → `discussion`. ("Should *I* pick Coppin?" = advice; "How *can HBCUs* fund so many scholarships?" = discussion.) |
| **news_share vs. discussion** | A link post with the author's own take added. | If the author mostly **relays** the item → `news_share`. If they add a **substantive opinion or question** that's the real point of the post → `discussion`. |

**Hardest case (checkpoint answer):** `achievement` vs. `news_share`. They are the closest pair because both are frequently third-person link posts about an HBCU — the real feed even has two from the *same author/blog* that split across the two labels. The boundary is **whether a win/ranking/milestone is being celebrated** (achievement) versus neutral info or an opportunity being relayed (news_share). During annotation, a grant or award is the trickiest sub-case: a *competitive award framed as a win* → `achievement`; *routine funding or a program launch* → `news_share`.

**Priority when still tied:** `advice_seeking` (does the author want input on their own situation?) → `achievement` (is a concrete win the point?) → `discussion` (is there a substantive opinion/general question?) → `news_share` (default for a plain external relay).

---

## 4. Data Collection Plan

**Source:** posts and top-level comments from [r/HBCU](https://www.reddit.com/r/HBCU/), pulled via the Reddit API (PRAW) across `hot`, `new`, and `top` (month + year) and across flairs, so one sort order's bias doesn't dominate.

**Target:** 200 labeled examples, floor of **~40 per label** (mild imbalance is fine — handled in evaluation via macro-F1, not by forcing an even split).

| Label | Target | Difficulty | Where it lives |
|---|---|---|---|
| advice_seeking | ~55 | Easy | Advice Needed, Transfer flairs — most common |
| news_share | ~50 | Easy | News flair, link posts |
| achievement | ~45 | Moderate | HBCU Sports, ranking/grant news (seasonal) |
| discussion | ~40 | Hardest | Academics + open-question posts; least common |

**If a label is underrepresented after 200:** (1) targeted collection first — for `discussion`, search open-question phrasings ("why do", "how are", "is it worth", "thoughts on") and sort by comment count, since discussion posts draw replies; for `achievement`, widen the date range to capture full sports seasons and ranking-release windows. (2) Extend the time range further back. (3) Only if a label still can't clear ~25, reconsider merging it (e.g. fold `discussion` into `advice_seeking` as "any question") — but log that as an explicit taxonomy change, not a silent drop. Final per-class counts go in the writeup so the imbalance is visible.

---

## 5. Evaluation Metrics

This is a **4-class problem with class imbalance and known confusable pairs**, so accuracy alone misleads: if `advice_seeking` is ~30% of the data, an always-`advice_seeking` model scores ~30% while being useless. The metric set is built to expose that.

**Primary — Macro-F1.** Averages per-class F1 with equal weight per class, so neglecting the smaller class (`discussion`) tanks the score. This is the headline number I optimize and report.

**Per-class precision, recall, F1.** TakeMeter exists to *distinguish* post types, so a model can have decent macro-F1 while failing one class. Per-class numbers say *which* distinction it hasn't learned — e.g. low `achievement` recall means wins are being absorbed into `news_share`.

**Confusion matrix.** The most diagnostic artifact. My taxonomy *predicts* where errors should cluster (the §3 zones, especially achievement↔news_share). If confusion concentrates there, the model fails where humans do; if it confuses unrelated pairs (e.g. advice_seeking↔achievement), something is more deeply wrong. It ties errors straight back to the label design.

**Weighted-F1 (secondary).** Reported alongside macro-F1 to show performance on the real, skewed distribution a deployed tool would see.

**Baselines for context.** (1) Majority-class baseline and (2) a TF-IDF + logistic-regression bag-of-words model. The real model must clearly beat both, or it's just keying on topic words rather than intent.

---

## 6. Definition of Success

Concrete, checkable thresholds — not "it works well":

- **Headline:** Macro-F1 **≥ 0.70** on a held-out test set.
- **No weak class:** every per-class F1 **≥ 0.55** (no label effectively unlearned — `discussion` is the one to watch).
- **Errors are explainable:** **≥ 70%** of misclassifications fall in the §3 edge zones (mostly achievement↔news_share) — i.e. the model fails where humans find it hard, not randomly.
- **Beats baselines:** macro-F1 at least **+0.15** over majority-class and **+0.10** over the TF-IDF baseline.

**Good enough for deployment** as an assistive community tool — e.g. auto-tagging the feed so mods can route `advice_seeking` posts for answers and surface `news_share`/`achievement` into a weekly digest: macro-F1 in the **0.70–0.80** range with no class below 0.55. At that level a human still reviews edge cases, but the tool removes most of the sorting work — the realistic bar for an assistive (not autonomous) classifier. Below 0.70 it makes more cleanup than it saves; above ~0.85 on data this subjective would make me suspect a leak or annotation artifact.

---

## 7. AI Tool Plan

No code to generate, so AI tools help at three specific points.

### 7a. Label stress-testing (before annotating)

I give an LLM the §2 definitions and §3 rules and ask for posts on each boundary. If I can't classify a generated post cleanly, the definition needs tightening. First pass, across **all** pairs:

1. *achievement vs. news_share* — "Howard lands a $5M federal research grant." → a grant is the trickiest sub-case. Rule: a *competitive award framed as a win* → `achievement`; a *routine funding/program announcement* → `news_share`. Added this explicitly to §3.
2. *advice_seeking vs. discussion* — "Is the HBCU experience really worth the extra cost?" → rephrase-dependent. About the author's own choice → `advice_seeking`; general → `discussion`. The §3 "own situation vs. general" rule handles it; flagged as the riskiest annotation call.
3. *news_share vs. discussion* — link post + "thoughts?" → minimal commentary → `news_share`; a real opinion/question that is the point → `discussion`. Tightened: a throwaway "thoughts?" does **not** make it `discussion`.
4. *achievement vs. discussion* — "FAMU #1 again — but are these rankings even meaningful?" → leads with an opinion/question, so the celebration is incidental → `discussion` per priority order. Clean.
5. *advice_seeking vs. news_share* — "Saw this scholarship deadline (link) — should I apply?" → relays news *and* asks about own decision → `advice_seeking` wins per priority. Kept as a canonical training example.

**Outcome:** rules survived with two tightenings (grant = win-framing test; throwaway "thoughts?" ≠ discussion). I'll re-run on the final definitions before annotating.

### 7b. Annotation assistance

**Decision: yes, with 100% human review.** An LLM (Claude) pre-labels each example; I review and correct every one. The model's label is a suggestion, never final.
- **Tracking for disclosure:** each row carries `pre_label` (LLM), `final_label` (me), `was_corrected` (bool). I report LLM↔me agreement in the AI-usage section — low agreement is itself a signal my definitions are fuzzy.
- **Bias guard:** I hand-label the first ~30 examples *blind* (before seeing any LLM suggestion) to calibrate my own judgment and avoid anchoring.

### 7c. Failure analysis

After evaluation I export every misclassification (text, true label, predicted label) and ask an LLM to **find patterns** — which label pairs dominate errors, whether wrong predictions share surface features (link-only posts, length, "thoughts?" phrasing, single-stat posts), and whether any "errors" are actually my annotation mistakes.
- **What I look for:** is confusion concentrated in the §3 zones (expected, esp. achievement↔news_share) or elsewhere (a real problem)? Does `discussion` recall lag? Are bare link posts systematically misread?
- **Verification:** every AI-proposed pattern is a hypothesis, not a finding. I re-read a sample of the cited examples myself and confirm the pattern holds before it enters the writeup.

---

*Update this document before starting any stretch features later.*
