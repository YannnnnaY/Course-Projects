# OTA Booking A/B Test: Conversion & Guardrail Analysis

An end-to-end A/B test analysis for an online travel agency (OTA) booking experiment, with a focus on **catching and correcting a unit-of-analysis mismatch** — a subtle but common bug where the level of randomization and the level of statistical analysis don't match.

## Project overview

The experiment tests a `variant` treatment against `control`, randomized at the **user** level, with:
- **Primary metric:** booking conversion rate
- **Guardrail metric:** time-to-booking (among users who convert)

The analysis walks through the standard A/B testing pipeline — SRM check, primary metric test, guardrail check, effect size, decision — and then adds a **robustness check** that re-runs the key tests at the correct unit of analysis and quantifies how much the mis-specified version distorted the results.

## Why this project exists

Most A/B test walkthroughs stop at "run a z-test, get a p-value." This one instead asks: *is the test even valid as specified?*

While reviewing the analysis, I noticed that although users were randomized 1:1 into `control`/`variant`, each user could generate up to 3 sessions in the data — and the primary metric test was run on session-level rows rather than aggregating to one observation per user first. That means repeated sessions from the same user were being treated as independent draws, which violates the independence assumption behind the z-test and chi-squared test and can distort the test statistic.

I re-ran the SRM check and the primary metric test at the correct (user) level and compared both versions side by side.

## Key finding

| Unit of analysis | z-stat | p-value | Effect size (ATE) |
|---|---|---|---|
| Session-level (mis-specified) | 3.72 | 0.0001 | 14.2% |
| User-level (correct) | 3.10 | 0.0010 | 11.6% |

The decision doesn't flip either way in this dataset, but the session-level version **inflates the z-statistic by ~20%, understates the p-value by almost an order of magnitude, and overstates the effect size by ~23%** — purely as an artifact of treating clustered (non-independent) observations as independent. In a lower-powered experiment, this kind of mis-specification could easily flip a null result into a "significant" one.

**Takeaway:** always check whether your unit of analysis matches your unit of randomization before trusting a test's p-value.

## Data

Synthetic OTA booking experiment data (January 2025), two files:

- **`users_data.csv`** (10,000 rows) — one row per user: `user_id`, `experiment_group` (`control`/`variant`)
- **`sessions_data.csv`** (16,982 rows) — one row per session: `session_id`, `user_id`, `session_start_timestamp`, `booking_timestamp` (null if no booking occurred), `time_to_booking`

Users can have multiple sessions (1–3 in this dataset); a small number of sessions (1,698) have no matching `user_id` in the users table (guest/unauthenticated traffic) and are dropped via inner join, since they can't be attributed to an experiment arm.

## Methodology

1. **Sample Ratio Mismatch (SRM) check** — chi-squared goodness-of-fit test on group sizes, run at both the session level and the (correct) user level
2. **Primary metric test** — two-proportion z-test on conversion rate, run at both levels for comparison
3. **Guardrail metric test** — two-sample t-test (via `pingouin`) on time-to-booking, conditional on conversion
4. **Effect size** — relative average treatment effect (ATE) on conversion
5. **Robustness check** — user-level re-aggregation and side-by-side comparison against the session-level results
6. **Decision** — full rollout recommendation based on primary metric, guardrail, and MDE considerations

## Tech stack

- `pandas`, `numpy` — data wrangling and aggregation
- `scipy.stats` — chi-squared SRM test
- `statsmodels` — two-proportion z-test
- `pingouin` — t-test with confidence intervals and effect sizes
- `matplotlib` — visualizing the SRM chi-squared distribution

## How to run

```bash
pip install pandas numpy scipy statsmodels pingouin matplotlib jupyter
jupyter notebook notebook.ipynb
```

Requires `sessions_data.csv` and `users_data.csv` in the same directory as the notebook.

## Files

```
.
├── notebook.ipynb        # full analysis
├── sessions_data.csv      # session-level event data
├── users_data.csv         # user-level experiment assignment
└── README.md
```

## Notes / known issues

- The guardrail metric (`time_to_booking`) is only observed for users who convert, which makes the t-test a comparison *conditional on conversion* — a potential source of selection bias if the treatment changes who converts, not just how fast. This is flagged as a limitation rather than resolved in the current version.
- `pingouin`'s `ttest()` output column name changed across versions (`p-val` vs `p_val`) — if you hit a `KeyError` on that column, check `guardrail_result.columns` for the exact name in your installed version.
