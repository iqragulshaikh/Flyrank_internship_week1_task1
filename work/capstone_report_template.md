# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** IQRA GUL
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/iqragulshaikh/Flyrank_internship_week1_task1/blob/main/work/notebooks/01_refresh_opportunity_scoring%20(1).ipynb
- **Date:** 19-10-2026

## 0. Abstract

*(Write last. Five sentences, in this order.)*

1. **Question:** Can recent trends in clicks, impressions, CTR and average position be used to flag pages likely to lose clicks next month, so they can be reviewed for refresh first?
2. **Data:** TODO (release, tables, date window, number of pages/months after filtering, all public-safe).
3. **Method:** Page x month panel; features from the last three months only; label = next-month clicks fall by more than 20%; a time-aware split compared against a persistence baseline.
4. **Result:** TODO (model vs baseline: ROC AUC, PR AUC, precision@K, each next to the base rate).
5. **Use:** A ranked review list with reason codes (decision support, not a causal claim) that an editor can work through top-down.

## 1. Problem framing

- **Decision supported:** Which existing pages should an editor review for refresh first, given limited editing time?
- **Unit of analysis:** page x month.
- **Output:** a risk score per page, a reason code, and a suggested review action; ranked top-N.
- **Human action:** open the top-ranked pages, check the reason code, then refresh/rewrite, review title/meta, protect, or monitor.
- **Cost of a wrong call:** a false positive wastes editor time on a healthy page; a false negative lets a decline go unnoticed for a month. The list is a prioritization aid, so ranking quality (precision@K, lift) matters more than a hard yes/no.
- **Why ML helps:** many pages and several interacting signals (clicks, position, CTR, volatility). A fixed rule only looks at one of them.

## 2. Data safety

- **Used:** TODO (tables and columns: page identifier, month, clicks, impressions, position).
- **Deliberately excluded:** TODO (e.g. `trend_direction`, `trend_pct` and any other label-derived fields; raw URLs, domains, queries, client names).
- **Leakage considered:**
  - Features use only months t, t-1, t-2. The label uses t+1.
  - Label-derived fields are never used as features.
  - Page / client IDs are pseudonymous and used for grouping only, never as features.
  - A one-month gap separates train and test, so no train label overlaps the test period.
- **Confirmation:** TODO (confirm nothing client-identifying appears anywhere in `work/`; published rankings use an anonymous rank, not URLs).

## 3. Baseline

- **Rule:** persistence. Score = -(month-over-month click change). Pages that fell last month are assumed to keep falling.
- **Why fair:** transparent, needs no training, and is what an editor would do by eye.
- **Numbers (same split, same metrics as the model):** TODO
- **Base rate (share of pages that decline in the test months):** TODO

## 4. Model / analysis

- **Methods:** Logistic Regression (scaled, class-balanced) and Gradient Boosting (sklearn `HistGradientBoostingClassifier`), chosen to compare a simple linear model with a nonlinear one.
- **Features (exact list):**
  `clicks`, `chg_clicks_1`, `chg_clicks_2`, `chg_impr_1`, `ctr`, `chg_ctr_1`, `position`, `chg_pos_1`, `volatility`.
- **Left out on purpose:** page/client IDs, any label-derived trend fields, URLs and text. TODO (add anything else).
- **Target (one sentence):** a page is "declining" if its clicks next month are more than 20% lower than this month's clicks (pages under 10 clicks are excluded as too noisy).

## 5. Evaluation

- **Split:** time-aware. Train on the older months, test on the newest 2, with a 1-month gap. TODO (optionally add a grouped-by-client check if client IDs exist, to show it generalizes to unseen clients).
- **Metrics (model vs baseline, same split):** ROC AUC, PR AUC, precision@100, each next to the base rate.

| Method | ROC AUC | PR AUC | Precision@100 | Base rate |
|---|---|---|---|---|
| Baseline (persistence) | TODO | TODO | TODO | TODO |
| Logistic Regression | TODO | TODO | TODO | TODO |
| Gradient Boosting | TODO | TODO | TODO | TODO |

- **Error analysis:** TODO (what kind of pages does the model miss or over-flag? e.g. seasonal pages, very low-traffic pages, recovering pages).

## 6. Interpretation

- **What the model found:** TODO (feature importances / permutation importance in plain words).
- **Surprises and negative results:** TODO (if the model barely beats the baseline, say so. That is a valid result).

## 7. Recommendation

- **Ranked actions:** TODO (top-N anonymized list; reason code counts: `DECLINING_FAST`, `RANK_SLIPPING`, `HIGH_IMPR_LOW_CTR`, `RECOVERING`, `STABLE_MONITOR`).
- **How an editor uses it tomorrow:** sort by risk score, start with `DECLINING_FAST` and `RANK_SLIPPING`, review `HIGH_IMPR_LOW_CTR` pages for title/meta, leave `RECOVERING` alone.
- **Confidence and limits:** observational data; scores are predictive of, and associated with, later declines. They do not show that any refresh causes recovery. Seasonality and site-wide changes can look like page-level declines. TODO (add data-window limits).

## 8. Reproducibility

- **Run from a fresh clone:**
  1. `pip install -r requirements.txt`
  2. Set `HF_TOKEN` (read token) as an environment variable or Colab secret.
  3. Run `work/01_refresh_opportunity_scoring.ipynb` top to bottom.
- **Seeds:** `random_state=42`.
- **Environment:** TODO (`pip freeze` highlights: duckdb, pandas, scikit-learn, matplotlib).
- **Outputs committed:** `outputs/results_table.csv`, `outputs/top_recommendations_anonymized.csv`, `results_model_vs_baseline.png`.
- **Sealed holdout (only if claimed):** commit the cell/script that builds the sealed frame and the metrics file it produced. Otherwise do not claim one.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset ([https://flyrank.ai](https://flyrank.ai)).

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support language everywhere · no causal claims · no "predicted Google's algorithm" · no client-identifying details · base rate reported next to every precision@K / accuracy · numbers in this report match a fresh re-run.
