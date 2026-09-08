# FlyRank Search Intelligence Capstone

## Project Overview
This repository contains the FlyRank Search Intelligence Capstone project focused on the **Refresh / Content Opportunity Scoring** lane. 

### Research Question
> *Which pages should be prioritized for content review or refresh based on observable search-performance signals?*

## Project Structure
```text
flyrank-search-capstone/
│
├── work/
│   └── notebooks/
│       └── w03_data_contract.ipynb
│
├── submission/
│   └── paper_url.txt
│
└── README.md
```

## Week 3 Deliverable — Data Contract & Feature Leakage
The current deliverable (`work/notebooks/w03_data_contract.ipynb`) establishes:
1. **Data Contract**: Defining the entity grain, table sources, time window (March 2026 slice), ranking target, and explicit exclusions.
2. **Three Verification Queries**:
   - Query 1: Grain uniqueness test (`content_hash_id`, `client_hash_id`, `anchor_date`).
   - Query 2: Row count and date span verification for March 2026 (`2026-03-01` to `2026-03-31`).
   - Query 3: Availability filter check using `IS TRUE` (`client_has_gsc IS TRUE`).
3. **Five Feature Definitions**: A compact feature frame (<=5 features) with explicit point-in-time knowability justifications.
4. **Leakage Demonstration**: Honest vs. leaky model evaluation using `LEAKAGE_DEMO` (target-derived feature), followed by immediate removal of the leaky feature.
5. **Limitation & Self-Check**: Dataset limitations based on real data characteristics and compliance checklist.

## Dataset
- **Data Source**: `alienalien/internship-warehouse-bucket` (`FlyRank/internship-warehouse` star schema on HuggingFace).
