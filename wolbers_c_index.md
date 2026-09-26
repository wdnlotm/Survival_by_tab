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




## Wolbers C‑index (Cause 1 = event, Cause 2 = competing risk)
| ID | Time | Status | Risk score |
|----|------|--------|-----------|
| 1 | 2 | cause 1 | 9 |
| 2 | 3 | cause 2 | 7 |
| 3 | 4 | cause 1 | 8 |
| 4 | 5 | censored | 3 |
| 5 | 6 | cause 1 | 6 |
| 6 | 7 | cause 2 | 2 |
| 7 | 8 | censored | 1 |

### Calculate Wolbers c-index with cause 1 as event cause 2 as competing risk event.

**Key difference from standard C‑index:** Under the Wolbers definition, a subject who experiences the **competing event (cause 2)** is treated as having *effectively infinite* time for the event of interest (cause 1) — since they can no longer ever have a cause‑1 event. This makes them a valid comparator for **any** cause‑1 anchor, *even if their observed time occurred earlier*. Censored subjects, by contrast, are only usable as comparators when their time is strictly greater than the anchor's time (as before).

**Event anchors:** ID1 (T=2, R=9), ID3 (T=4, R=8), ID5 (T=6, R=6)

### Anchor ID1 (T=2, R=9)
All others have T > 2, so all 6 are usable as before.

| j | T | R | Concordant (9>R)? |
|---|---|---|---|
|ID2|3|7|✅|
|ID3|4|8|✅|
|ID4|5|3|✅|
|ID5|6|6|✅|
|ID6|7|2|✅|
|ID7|8|1|✅|

→ **6/6**

### Anchor ID3 (T=4, R=8)
Usual comparators (T>4): ID4, ID5, ID6, ID7 — **plus** ID2 (cause 2, T=3 < 4), now eligible.

| j | T | R | Concordant (8>R)? |
|---|---|---|---|
|ID2|3|7|✅ (extra pair — competing event)|
|ID4|5|3|✅|
|ID5|6|6|✅|
|ID6|7|2|✅|
|ID7|8|1|✅|

→ **5/5**

### Anchor ID5 (T=6, R=6)
Usual comparators (T>6): ID6, ID7 — **plus** ID2 (cause 2, T=3 < 6), now eligible.

| j | T | R | Concordant (6>R)? |
|---|---|---|---|
|ID2|3|7|❌ (6 < 7 → discordant)|
|ID6|7|2|✅|
|ID7|8|1|✅|

→ **2/3**

### Totals

- Concordant pairs = 6 + 5 + 2 = **13**
- Usable (comparable) pairs = 6 + 5 + 3 = **14**

$$
C_{\text{Wolbers}} = \frac{13}{14} \approx \boxed{0.929}
$$

**Interpretation:** Including the two subjects with competing (cause‑2) events as comparators regardless of time order added 3 extra pairs (ID2 vs. ID3, ID2 vs. ID5, and ID2's original pair with ID1 was already counted). One of these — ID5 vs. ID2 — is discordant (risk 6 < 7), pulling the index down slightly from the perfect 1.00 obtained when cause‑2 subjects were simply treated as censored.
