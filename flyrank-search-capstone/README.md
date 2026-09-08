# 🚀 FlyRank Search Intelligence Capstone

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![DuckDB](https://img.shields.io/badge/Query_Engine-DuckDB-FFF000.svg?logo=duckdb&logoColor=black)](https://duckdb.org/)
[![Status](https://img.shields.io/badge/Status-Week%203%20Complete-success.svg)]()
[![Lane](https://img.shields.io/badge/Lane-Refresh%20%2F%20Content%20Opportunity%20Scoring-purple.svg)]()

> **Research Question:**  
> *"Which pages should be prioritized for content review or refresh based on observable search-performance signals?"*

---

## 📌 Executive Summary

This repository houses the end-to-end research and engineering artifacts for the **FlyRank Search Intelligence Capstone** (*Refresh / Content Opportunity Scoring* lane). 

The goal of this project is to develop an objective, algorithmic scoring mechanism that diagnoses search performance decay (traffic decay, click-through dropoff, rank slippage) against total addressable search demand to recommend high-ROI pages for editorial refresh.

```text
┌───────────────────────────────┐     ┌───────────────────────────────┐     ┌───────────────────────────────┐
│     Warehouse Artifacts       │     │     Point-in-Time Signals     │     │      Opportunity Scoring      │
│  alienalien/internship-bucket │ ──► │ • 90d Search Impressions      │ ──► │  High Market Demand           │
│   (54,642 March 2026 rows)    │     │ • 90d CTR & Avg Position      │     │            +                  │
│                               │     │ • Click Decay Momentum (<1.0) │     │  Decaying Momentum (<1.0)     │
└───────────────────────────────┘     └───────────────────────────────┘     └───────────────────────────────┘
```

---

## 📂 Repository Architecture

```text
flyrank-search-capstone/
│
├── work/
│   └── notebooks/
│       └── w03_data_contract.ipynb    # 📓 Core Week 3 Deliverable (Executed)
│
├── submission/
│   └── paper_url.txt                  # 📄 Deployed research paper URL (Placeholder)
│
├── .gitignore                         # 🛡️ Git hygiene & cache exclusions
└── README.md                          # 📘 Repository documentation
```

---

## 📜 Week 3 Deliverable: Data Contract & Feature Leakage

The primary deliverable for Week 3 is located in [`work/notebooks/w03_data_contract.ipynb`](work/notebooks/w03_data_contract.ipynb). It establishes the formal boundaries, point-in-time validity, and empirical proof of concept on real data.

### 1. Formal Data Contract

| Contract Dimension | Specification |
| :--- | :--- |
| **Row Meaning (Grain)** | One published content asset (`content_hash_id`) for a client domain (`client_hash_id`) at a decision date (`anchor_date`). |
| **Source Table** | `alienalien/internship-warehouse-bucket` (`windows.parquet` materialized feature store). |
| **Development Window** | **March 2026** (`2026-03-01` to `2026-03-31`) with 90-day/30-day pre-anchor lookbacks. |
| **Ranking Objective** | **Refresh / Opportunity Score** prioritizing decaying assets with high latent search demand. |
| **Deliberate Exclusions** | Unpublished drafts (`is_published IS FALSE`), missing GSC tracking (`client_has_gsc IS FALSE`), sealed holdout (`_sample` / June 2026), post-anchor telemetry. |

---

### 2. Empirical Verification Queries (DuckDB)

Executed against the March 2026 development slice (54,642 total rows):

* **Query 1 (Grain Uniqueness):** Confirms `(content_hash_id, client_hash_id, anchor_date)` has **0 duplicates** across 54,642 rows.
* **Query 2 (Date Span):** Confirms continuous temporal span from `2026-03-01` to `2026-03-31` (all 31 calendar days).
* **Query 3 (Availability `IS TRUE`):** Confirms `client_has_gsc IS TRUE` and `is_published IS TRUE` filter retains 54,642 verified rows.

---

### 3. Point-in-Time Feature Frame ($\le 5$ Features)

All features are strictly knowable prior to the decision moment $t$:

```text
┌─────────────────────────┬──────────────────────────────────┬────────────────────────────────────────────────────────┐
│ Feature Name            │ Source / Calculation             │ Knowability at Decision Moment                         │
├─────────────────────────┼──────────────────────────────────┼────────────────────────────────────────────────────────┤
│ f_gsc_impressions_90d   │ 90-day pre-anchor impression sum │ Aggregates search logs up to day t-1.                  │
│ f_gsc_ctr_90d           │ clicks_90d / impressions_90d     │ Historical CTR within [t-90, t).                       │
│ f_gsc_avg_position_90d  │ 90-day mean SERP position        │ Past SERP ranking logs logged prior to t.              │
│ f_gsc_clicks_momentum   │ clicks_last30 / clicks_first30   │ Both recent and baseline windows belong to the past.   │
│ f_content_age_days      │ anchor_date - published_at       │ Content publication date is immutable in the past.     │
└─────────────────────────┴──────────────────────────────────┴────────────────────────────────────────────────────────┘
```

---

### 4. Controlled Leakage Demonstration

To audit and prove leakage resilience, we compared honest point-in-time scoring against an injected future feature:

$$\text{Honest Refresh Score} = \ln(1 + \text{impressions}_{90d}) \times \frac{1}{\text{momentum} + 0.1}$$

$$\text{Leaky Score} = \text{Honest Score} + \underbrace{\text{LEAKAGE\_DEMO}}_{1.5 \times \text{target\_gsc\_clicks\_30d} + 10}$$

```text
📊 Evaluation Results (Spearman Rank Correlation with Future 30d Clicks):
  • Honest score: 0.3412  (Realistic baseline)
  • Leaky score:  0.9984  (Artificial inflation — removed immediately)
```

> **Why `LEAKAGE_DEMO` is Leakage:**  
> It incorporates `target_gsc_clicks_30d`, capturing user interactions during $[t, t+30d)$. At decision date $t$, future traffic is unobservable. Dropping `LEAKAGE_DEMO` ensures 100% production validity.

---

### 5. Identified Data Limitation

> **Content Velocity & Seasonality Confounding:**  
> Multi-week rolling aggregations do not normalize for macroeconomic or seasonal search volume fluctuations. Pages in seasonal verticals may exhibit temporary momentum drops without true content decay. Future iterations will cross-reference rank stability alongside momentum.

---

## 🛠️ Quickstart & Reproduction

### Prerequisites
- Python 3.10+
- `pip install duckdb pandas numpy scipy huggingface_hub jupyter`

### Running the Notebook
```bash
# 1. Clone the repository
git clone https://github.com/Dinesh-design786/founditions-flyrank.git
cd founditions-flyrank

# 2. Launch Jupyter Notebook
jupyter notebook work/notebooks/w03_data_contract.ipynb
```

---

## 🗺️ Roadmap & Milestones

- [x] **Week 3:** Data Contract, Verification Queries & Leakage Proof of Concept
- [ ] **Week 4:** Feature Engineering & Historical Signal Transformation
- [ ] **Week 5:** Baseline Model & Opportunity Ranking Formulation
- [ ] **Week 6:** Offline Evaluation against Sealed June 2026 Holdout (`_sample`)
- [ ] **Final:** Research Paper Deployment & Capstone Submission

---

## 👤 Author
- **Dinesh Polumuri** ([@Dinesh-design786](https://github.com/Dinesh-design786))
