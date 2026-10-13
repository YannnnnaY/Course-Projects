# Course Projects

A collection of analytics and data-science projects — a mix of coursework and self-directed work (mostly 2024), spanning experimentation, machine learning, data engineering, and API-driven analyses.

## 🔬 Experimentation & A/B testing

- **[ota-booking-ab-test](https://github.com/YannnnnaY/ota-booking-ab-test)** (→ standalone repo) — End-to-end A/B test analysis for an OTA booking experiment: SRM check, two-proportion z-test, guardrail t-test on time-to-booking, effect size, and a rollout recommendation. Key contribution: caught a unit-of-analysis mismatch (randomized at user level, analyzed at session level) and quantified the distortion — z-statistic inflated ~20%, p-value understated ~10x.
- `Mobile_Games_AB_Testing_with_Cookie_Cats.ipynb` — A/B test analysis on the Cookie Cats mobile gaming dataset.

## 🤖 Machine learning

- **[ML_wout_packages](ML_wout_packages)** — Core ML algorithms implemented from scratch (no sklearn): logistic regression, decision-tree classification & regression, naive Bayes, and k-means clustering, plus regression/classification loss comparisons.
- **[Twitter_Fake_News](Twitter_Fake_News)** — Fake-news detection using clustering and classification on real vs. fake news articles. Note: the Kaggle dataset this used has since been made private, so the analysis isn't fully reproducible.

## 🔧 Data engineering / ETL

- **[YouTube_comments_etl](YouTube_comments_etl)** — Airflow DAG + Python ETL that pulls YouTube comments via the YouTube Data API.
- **[youtube_query_sentiment](youtube_query_sentiment)** — Scheduled YouTube data pull on GCP (Cloud Functions + Cloud Scheduler) with sentiment analysis, visualized in a Looker Studio dashboard.


## Notes

- The OTA A/B test project started in the `user_conversion_A:BTesting` folder here and has been promoted to its own repository (linked above).
- Datasets are synthetic or public; see each project's notes for details and reproducibility caveats.

