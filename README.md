# 🚀 FlyRank Search Intelligence Capstone

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![DuckDB](https://img.shields.io/badge/Query_Engine-DuckDB-FFF000.svg?logo=duckdb&logoColor=black)](https://duckdb.org/)
[![Status](https://img.shields.io/badge/Status-Week%204%20Complete-success.svg)]()
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
│   ├── notebooks/
│   │   ├── w03_data_contract.ipynb    # 📓 Week 3: Data Contract & Feature Leakage (Executed)
│   │   └── w04_baseline_score.ipynb   # 📓 Week 4: Baseline Action Score & Signal Validation (Executed)
│   └── outputs/
│       └── baseline_action_score.csv  # 📊 Ranked Action Queue (Uncommitted via .gitignore)
│
├── submission/
│   └── paper_url.txt                  # 📄 Deployed research paper URL (Placeholder)
│
├── .gitignore                         # 🛡️ Git hygiene & cache exclusions
└── README.md                          # 📘 Repository documentation
```

---

## ⚡ Week 4 Deliverable: Baseline Action Score

The Week 4 deliverable is located in [`work/notebooks/w04_baseline_score.ipynb`](work/notebooks/w04_baseline_score.ipynb).

### 1. Validated Pre-Decision Signals

| Signal | Metric / Formula | Bucket Analysis ($n$) | Empirical Finding | Verdict |
| :--- | :--- | :--- | :--- | :--- |
| **Traffic Decay Momentum** | $\text{clicks}_{\text{last30}} / \text{clicks}_{\text{first30}}$ | Severe Decay ($n=8,426$), Moderate ($n=12,348$), Stable ($n=21,676$), Growing ($n=12,192$) | Pages with momentum $< 0.8$ suffer severe click dropoffs (148.6 avg clicks) despite huge impressions (42.8k). | **`CONFIRMED`** |
| **Content Age & Staleness** | $\text{anchor\_date} - \text{published\_at}$ | Fresh ($n=6,140$), Maturing ($n=11,480$), Established ($n=18,740$), Stale ($n=18,282$) | Legacy pages ($>365\text{d}$) command top impressions (72.4k) but suffer lowest CTR and momentum decay. | **`CONFIRMED`** |

---

### 2. Explainable Baseline Formulation

$$\text{score} = \ln(1 + \text{f\_gsc\_impressions\_90d}) \times \left( \frac{1}{\text{f\_gsc\_clicks\_momentum} + 0.1} \right) \times \left( 1.0 + 0.2 \times \mathbb{I}(\text{f\_content\_age\_days} \ge 180) \right)$$

### 3. Reason Code & Action Label Matrix

| Reason Code | Trigger Condition | Action Label | Strategy |
| :--- | :--- | :--- | :--- |
| `HIGH_DEMAND_DECAY` | $\text{imp}_{90d} \ge 10000$ AND $\text{momentum} < 0.8$ | `PRIORITY_REFRESH` | Urgent content overhaul on high-demand decaying assets. |
| `STRIKING_DISTANCE_STALE` | $\text{pos}_{90d} \in [4.0, 15.0]$ AND $\text{age} \ge 180\text{d}$ | `STRIKING_DISTANCE_BOOST` | On-page expansion to break page 2 / bottom page 1 into top 3. |
| `LOW_CTR_OPPORTUNITY` | $\text{ctr}_{90d} < 0.02$ AND $\text{imp}_{90d} \ge 5000$ | `METADATA_CTR_FIX` | Title tag, meta description, and snippet optimization. |
| `STABLE_PERFORMER` | Fallback | `MONITOR` | Stable performance; routine monitoring rotation. |

---

## 📜 Week 3 Deliverable: Data Contract & Feature Leakage

The Week 3 foundation is located in [`work/notebooks/w03_data_contract.ipynb`](work/notebooks/w03_data_contract.ipynb).

- **Entity Grain:** `(content_hash_id, client_hash_id, anchor_date)` verified with 0 duplicates across 54,642 March 2026 rows.
- **Availability:** Mandatory `client_has_gsc IS TRUE` and `is_published IS TRUE` filter.
- **Leakage Demonstration:** Proved point-in-time validity by contrasting honest score correlation (`0.3412`) against intentionally injected `LEAKAGE_DEMO` (`0.9984`), then removing the leaky feature.

---

## 🛠️ Quickstart & Reproduction

### Prerequisites
- Python 3.10+
- `pip install duckdb pandas numpy scipy huggingface_hub jupyter`

### Running the Notebooks
```bash
# 1. Clone repository
git clone https://github.com/Dinesh-design786/founditions-flyrank.git
cd founditions-flyrank

# 2. Run Week 3 Data Contract
jupyter notebook work/notebooks/w03_data_contract.ipynb

# 3. Run Week 4 Baseline Action Score
jupyter notebook work/notebooks/w04_baseline_score.ipynb
```

---

## 🗺️ Roadmap & Milestones

- [x] **Week 3:** Data Contract, Verification Queries & Leakage Proof of Concept
- [x] **Week 4:** Signal Validation, Explainable Baseline Rule & Ranked Queue Export
- [ ] **Week 5:** Advanced ML Model & Refresh Ranking Formulation
- [ ] **Week 6:** Offline Evaluation against Sealed June 2026 Holdout (`_sample`)
- [ ] **Final:** Research Paper Deployment & Capstone Submission

---

## 👤 Author
- **Dinesh Polumuri** ([@Dinesh-design786](https://github.com/Dinesh-design786))