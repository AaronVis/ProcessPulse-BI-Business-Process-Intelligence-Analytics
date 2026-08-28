# ProcessPulse BI: Loan Application Process Mining & Performance Diagnostics

A real-world process mining and BI pipeline dissecting 31,509 loan origination cases from the BPI Challenge 2017 log. Built to identify where applications stall, why 44.6% of them drop off, and where automated steps give way to severe human validation backlogs.

---

## What the System Does

Loan processes look sequential in operational documentation, but event logs reveal rework loops and dead time. This project extracts the ground-truth workflow directly from timestamped XES logs:

- Maps actual case execution paths against standard happy paths using PM4Py.
- Isolates handoff delays between system-driven steps and manual credit checks.
- Models clean, query-ready analytics tables inside PostgreSQL.
- Surfaces cycle times, bottleneck distributions, and variant distributions in a two-page Power BI dashboard.

---

## Data Pipeline Architecture

```text
[BPI 2017 Event Log (XES)]
       │
       ▼  (Python / PM4Py / Pandas)
[ETL & DFG Bottleneck Mining]
       │
       ▼  (SQLAlchemy Engine)
[PostgreSQL Warehouse (`analytics.case_summary`)]
       │
       ▼  (Direct SQL / Aggregated Tables)
[Power BI Executive & Flow Dashboards]
```

---

## Key Operational Findings

- **The 27-Day Inaction Cliff**: Applications reaching `A_Complete` stall for a median of ~27 days before terminating in `A_Cancelled` / `O_Cancelled`. This points directly to customer hesitation after receiving the formal offer, rather than upstream system delays.
- **The 8-Day Verification Backlog**: Transitioning a valid file from `A_Complete` into `A_Validating` takes roughly 8 days on average, indicating a severe resource bottleneck right before credit approval.
- **System Actor Dominance**: `User_1` acts as the primary background service account firing high-volume initial transitions, while subsequent loan modifications fall entirely on manual human queues with steep variance.

---

## Dashboard Breakdown

### Page 1: Executive Overview
- High-level KPIs: 31,509 total applications, 21.9-day mean cycle duration, 19.1-day median duration, and a 44.6% cancellation rate.
- Volume cuts across loan purpose (Car, Home Improvement, Debt Consolidation) and application type (New Credit vs. Limit Raise).
- Cycle duration distribution highlighting long-tail outlier cases past the 50-day mark.

### Page 2: Process Discovery & Flow
- Directly-Follows Graph (DFG) filtered to the top 85% activity and 75% path volume to eliminate graph noise while keeping case math exact.
- Clustered breakdown of median vs. 90th percentile wait hours across all key transitions.
- Variant breakdown matrix detailing step paths alongside exact case counts and share percentages.

---

## Quickstart & Local Setup

### 1. Requirements
- Python 3.10+
- PostgreSQL 14+
- Graphviz (`winget install Graphviz.Graphviz` on Windows or `brew install graphviz` on macOS)
- Power BI Desktop

### 2. Environment Setup
```bash
# Clone repository
git clone [https://github.com/](https://github.com/)<your-username>/ProcessPulseBI.git
cd ProcessPulseBI

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 3. Run Pipeline
1. Ensure your PostgreSQL server is active on `localhost:5432` with a database named `processpulse_bi`.
2. Run `notebooks/01_dataset_exploration.ipynb` end-to-end to parse the XES log, compute process metrics, and write tables to the database.
3. Open `powerbi/ProcessPulse_Overview.pbix` in Power BI Desktop to inspect the visualizations.