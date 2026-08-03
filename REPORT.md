# REPORT — Module 3 · Assignment 1 · Supervised Learning

**Name:** Artem Lentchinski **ID:** 323324327  **Date:** 26.06.26
**Chosen task:**  C · Olist daily volume forecast)

> Keep this report in English. Be concise and honest. "I don't know, and here is why"
> is a professional answer. Empty confidence is not.

---

## 1. Problem framing
Business question (one short paragraph):
Olist, a Brazilian e-commerce platform, needs to predict the daily order volume (number of orders placed) for the following day. The forecast is generated at 08:00 BRT (UTC-3) and consumed by the warehouse manager and logistics team to make two operational decisions: whether to place an additional order with suppliers, and whether to hire temporary staff. Over-forecasting causes excess staffing costs (a fixed, recoverable loss), while under-forecasting causes delivery delays and customer dissatisfaction — a long-term, asymmetric, and largely irreversible loss, as dissatisfied customers reduce future order volume. Because the cost of under-forecasting is structurally higher and harder to recover from, the model must minimize large negative errors.

Target definition:
Target: n_orders — the total number of orders placed on the Olist platform on day D+1 (BRT midnight-to-midnight), predicted at 08:00 on day D, using only data available through the end of day D-1.

**Primary metric and why it fits the business cost** (cost of a false positive vs a
false negative; for forecasting, cost of over- vs under-forecasting):
Primary metric: RMSE (Root Mean Squared Error).
In a forecasting task, the analog of the false-positive / false-negative trade-off is over- vs under-forecasting. Over-forecasting causes excess staffing costs — a fixed, recoverable loss. Under-forecasting causes delivery delays and customer dissatisfaction — a long-term, asymmetric, and largely irreversible loss, as dissatisfied customers reduce future order volume. Because the cost of under-forecasting is structurally higher and harder to recover from, the model must above all avoid large misses, especially on high-volume days.
RMSE fits this need: it penalizes large errors quadratically rather than linearly, so a model trained to minimize RMSE prioritizes getting high-volume days right — precisely where under-forecasting is most costly.
Secondary metric: MAE — reported alongside RMSE for direct business interpretability ("on average, we miss by X orders per day").

---

## 2. Results table
Fill in every model **and the dumb baseline** on the locked test set.

| Model | Primary metric | Secondary metric | Notes |
|---|---|---|---|
| baseline | | | dumb model |
| linear | | | |
| bagging | | | |
| boosting | | | |

---

## 3. Guiding questions (graded)
Answer each in 2-5 sentences.

1. **Accuracy trap.** Why is plain accuracy misleading for your problem, and what did
   the confusion matrix (or the error distribution, for forecasting) reveal that accuracy hid?

2. **Cost of errors.** What does a false positive cost vs a false negative in your
   business context, and how did that drive your metric choice?

3. **Worth deploying?** By how much did your best model beat the dumb baseline?
   If the margin is small, is the model worth shipping — and would you ship it?

4. **What drives it.** Which features carry the predictions? Do they make business
   sense, or is the model leaning on a leak or a spurious correlation? How did you check?

5. **Worst errors.** Look at the 5 worst mistakes. What is the story — a data quality
   problem, mislabeled rows, or genuinely hard cases?

6. **Stability.** How much does the score move across CV folds? Would you hand this
   single number to a stakeholder as "the" performance? Why or why not?

The score moves across the folds very strongly. For the tree models the movement is huge: for Random Forest the spread is close to the mean - ±111 against a mean of 120. Gradient Boosting is even worse - a spread of 185, larger than its mean of 167.
This is probably rooted in the nature of trees combined with a time series. TimeSeriesSplit cuts the series by time, so each successive fold tests the model on a later stretch where order volume is higher. Trees, by design, cannot output values above the maximum seen during training. On the later folds the real order counts climb and the tree hits a ceiling.
Taking the test results into account, it becomes clear that for the tree models cross-validation is a harsher check than the test itself.
   Because of this, the CV number should be shown to a stakeholder as a measure of stability and risk, not as the model's expected performance - as a point estimate of error it overstates how badly the trees actually forecast.

7. **Leakage / time.** (Required for B too, in one line.) How did you guarantee the
   model never saw information from the future or from the test set? For a time-based
   split, what happens to the score compared with a random split, and why?

   Feature construction. Every rolling feature uses shift(1) before .rolling(), so each window ends at D−1 and never touches the target day D.
Empirical check. At row i=100 the rolling windows were recomputed by hand (iloc[i-7:i], day i excluded) and compared to the stored values with np.isclose — both matched (True). The guarantee is tested, not just declared.
Chronological split. Train and test are split by time, no shuffle. An assertion (idx_train.max() < idx_test.min()) enforces that every training date falls strictly before every test date (train 2017-01-19 → 2018-04-28, test 2018-04-29 → 2018-08-22). The test set is touched exactly once, in Part 3.

   ### Q7 — Leakage / time
<!-- DRAFT — finalize after Part 3 -->
Leakage prevention (part 1 of answer):
- shift(1) before rolling(N) → window ends at D-1
- empirical check at i=100: rolling_7/14 match manual recompute (no leak)
- dead-tail trimmed before feature build
- chronological split, no shuffle, test locked until Part 3
Score time vs random (part 2): [fill after Part 3 with real numbers]  #  Здесь моя запись заканчивается. 



8. **Monday morning.** If this went live today, what would you monitor, and what signal
   would make you retrain it?

---

## 4. Model Card
Paste the completed Model Card from the notebook here.

```
(Model Card)
```

---

## 5. Reflection
What surprised you? What would you do differently with two more weeks or more data?
How would this approach change if it were part of your mid-term project?
