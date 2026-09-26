Here's a 7-subject example that cleanly separates the two indices.

**Data** (status: 0 = censored, 1 = cause 1 [main event], 2 = cause 2 [competing risk]; higher risk score = predicted higher risk of cause 1):

| ID | Time | Status | Risk score |
|----|------|--------|-----------|
| 1 | 2 | cause 1 | 9 |
| 2 | 3 | cause 2 | 8 |
| 3 | 4 | cause 1 | 7 |
| 4 | 5 | censored | 3 |
| 5 | 6 | cause 1 | 6 |
| 6 | 7 | cause 2 | 2 |
| 7 | 8 | censored | 1 |

**Why this is designed the way it is:** subject 2 has a high risk score (8) but dies of the competing cause early, before subjects 3 and 5 (who have lower scores) ever have their cause-1 events. That's the scenario where naive and Wolbers C-index disagree.

**1. Naive C-index (cause 2 treated as censoring)**
Comparable pair rule: the earlier-time member of the pair must be a cause-1 event.

- Subject 1 (t=2, cause1) vs everyone (6 pairs) → all concordant (risk 9 is highest)
- Subject 3 (t=4, cause1) vs 4,5,6,7 (4 pairs) → all concordant
- Subject 5 (t=6, cause1) vs 6,7 (2 pairs) → all concordant
- Subject 2 and 6 (cause 2) never qualify as the earlier member, so pairs like (2,3) or (2,5) are excluded entirely.

Total: 12/12 concordant → **naive C-index = 1.00**

**2. Wolbers C-index (cause 1 main, cause 2 as competing risk)**
Same 12 pairs, *plus* pairs where a cause-1 event occurs after a competing event — these are now treated as valid/comparable, since a subject who already died of cause 2 can never have the cause-1 event:

- (3 vs 2): risk 7 < 8 → **discordant**
- (5 vs 2): risk 6 < 8 → **discordant**

Total: 12 concordant + 2 discordant = 14 pairs → **Wolbers C-index = 12/14 ≈ 0.857**

**Takeaway:** the naive index says the model discriminates perfectly, because it silently drops subject 2 from consideration once they die of cause 2. Wolbers' index correctly penalizes the model for having ranked subject 2 as high-risk for cause 1 when they never got the chance to have it — giving a more honest (lower) picture of discrimination in the presence of competing risks.

If useful, I can give you R code (`riskRegression::Score` implements Wolbers) or Python code that reproduces both numbers on this toy dataset.

All 21 pairs (7 choose 2), grouped by comparability:

**Comparable under both naive and Wolbers (12 pairs, all concordant)**

| Pair | Times | Statuses | Risk (earlier vs later) | Result |
|------|-------|----------|--------------------------|--------|
| 1–2 | 2,3 | cause1, cause2 | 9 vs 8 | concordant |
| 1–3 | 2,4 | cause1, cause1 | 9 vs 7 | concordant |
| 1–4 | 2,5 | cause1, cens | 9 vs 3 | concordant |
| 1–5 | 2,6 | cause1, cause1 | 9 vs 6 | concordant |
| 1–6 | 2,7 | cause1, cause2 | 9 vs 2 | concordant |
| 1–7 | 2,8 | cause1, cens | 9 vs 1 | concordant |
| 3–4 | 4,5 | cause1, cens | 7 vs 3 | concordant |
| 3–5 | 4,6 | cause1, cause1 | 7 vs 6 | concordant |
| 3–6 | 4,7 | cause1, cause2 | 7 vs 2 | concordant |
| 3–7 | 4,8 | cause1, cens | 7 vs 1 | concordant |
| 5–6 | 6,7 | cause1, cause2 | 6 vs 2 | concordant |
| 5–7 | 6,8 | cause1, cens | 6 vs 1 | concordant |

**Comparable only under Wolbers (2 pairs — the extra ones, both discordant)**

| Pair | Times | Statuses | Risk (cause1 vs cause2) | Result |
|------|-------|----------|---------------------------|--------|
| 2–3 | 3,4 | cause2, cause1 | 7 vs 8 | discordant |
| 2–5 | 3,6 | cause2, cause1 | 6 vs 8 | discordant |

These are excluded under the naive rule because subject 2's cause-2 time (t=3) is earlier than subjects 3 and 5's cause-1 times, and the naive rule only allows the *earlier* member of a pair to "count" if it's a cause-1 event.

**Not comparable under either method (7 pairs)**

| Pair | Times | Statuses | Why excluded |
|------|-------|----------|--------------|
| 2–4 | 3,5 | cause2, cens | earlier is cause2, later isn't cause1 |
| 2–6 | 3,7 | cause2, cause2 | earlier is cause2, later isn't cause1 |
| 2–7 | 3,8 | cause2, cens | earlier is cause2, later isn't cause1 |
| 4–5 | 5,6 | cens, cause1 | earlier is censored (not cause1 or cause2) |
| 4–6 | 5,7 | cens, cause2 | earlier is censored |
| 4–7 | 5,8 | cens, cens | earlier is censored |
| 6–7 | 7,8 | cause2, cens | earlier is cause2, later isn't cause1 |

Totals: 12 + 0 = 12 naive pairs (12 concordant → **1.00**); 12 + 2 = 14 Wolbers pairs (12 concordant, 2 discordant → **12/14 ≈ 0.857**).
