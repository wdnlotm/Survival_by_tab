## C‑index Calculation (Cause 1 as event, Cause 2 treated as censored)

| ID | Time | Status | Risk score |
|----|------|--------|-----------|
| 1 | 2 | cause 1 | 9 |
| 2 | 3 | cause 2 | 7 |
| 3 | 4 | cause 1 | 8 |
| 4 | 5 | censored | 3 |
| 5 | 6 | cause 1 | 6 |
| 6 | 7 | cause 2 | 2 |
| 7 | 8 | censored | 1 |

### Calculate c-index with cause 1 as event cause 2 as censored.

**Setup:** Only subjects with `cause 1` status serve as valid "event" anchors for comparison. For each such subject, I compare against every other subject with a *strictly longer* observed time (any status — cause 2 or censored — is eligible as the comparator, since only the anchor needs to be an actual cause‑1 event).

A pair is **concordant** if the subject with the shorter time has the higher risk score.

### Event anchors: ID1 (T=2, R=9), ID3 (T=4, R=8), ID5 (T=6, R=6)

**Anchor ID1 (T=2, R=9)** — comparable to all 6 others (all have T>2):

| Compare to | T | R | R9 > R? |
|---|---|---|---|
| ID2 | 3 | 7 | ✅ |
| ID3 | 4 | 8 | ✅ |
| ID4 | 5 | 3 | ✅ |
| ID5 | 6 | 6 | ✅ |
| ID6 | 7 | 2 | ✅ |
| ID7 | 8 | 1 | ✅ |

→ 6/6 concordant

**Anchor ID3 (T=4, R=8)** — comparable to those with T>4: ID4, ID5, ID6, ID7

| Compare to | T | R | R8 > R? |
|---|---|---|---|
| ID4 | 5 | 3 | ✅ |
| ID5 | 6 | 6 | ✅ |
| ID6 | 7 | 2 | ✅ |
| ID7 | 8 | 1 | ✅ |

→ 4/4 concordant

**Anchor ID5 (T=6, R=6)** — comparable to those with T>6: ID6, ID7

| Compare to | T | R | R6 > R? |
|---|---|---|---|
| ID6 | 7 | 2 | ✅ |
| ID7 | 8 | 1 | ✅ |

→ 2/2 concordant

### Totals

- Concordant pairs = 6 + 4 + 2 = **12**
- Comparable pairs = 6 + 4 + 2 = **12**

$$
C\text{-index} = \frac{12}{12} = \boxed{1.00}
$$

The risk scores perfectly rank the cause‑1 event times in this dataset (every higher‑risk cause‑1 case occurred no later than every lower‑risk comparator).
