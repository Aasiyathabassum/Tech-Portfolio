# 🔗 Analytics Enablement Layer: Medallion Architecture on Databricks
**Role:** Data Engineer
**Focus:** Cloud data governance, incremental ingestion, data quality automation, and analytics-ready aggregation
**Series:** Part 2 of *End-to-End Operational Data Pipeline* — see [Part 1: Shopfloor Capture & Hybrid Cloud Ingestion](#) for the upstream Power Platform / capture layer this project's analytics layer plugs into.

---

## 🧭 Overview

Part 1 of this series covers shopfloor data capture and automated ingestion into hybrid
cloud storage. This project picks up from there and builds out the **analytics
enablement layer**: a governed medallion pipeline (Bronze → Silver → Gold) that takes
raw incoming records, applies schema enforcement and automated data quality validation,
and produces business-ready aggregates — with governance, lineage, and orchestration
built in rather than bolted on afterward.

Built on public NYC taxi trip data (via Databricks' built-in sample datasets) to keep
the project fully reproducible with no proprietary or licensed data, while mirroring the
same architectural pattern used for real operational/process data.

---

## 🏗️ Architecture

```
                    ┌────────────────────────────────────────────┐
                    │              UNITY CATALOG                  │
                    │   governance · lineage · access · audit     │
                    └────────────────────────────────────────────┘
                                       │
  Landing Zone  ──▶  BRONZE  ──▶  SILVER  ──▶  GOLD  ──▶  DASHBOARD
  (raw file          (Auto        (schema      (business   (live +
   drop zone)         Loader,      enforce,     aggregates, historical
                      raw          dedup, DQ,   optimized   views,
                      ingest)      quarantine,  for BI)     Databricks
                                   constraints)              SQL)
       │                  │            │            │           │
       └──────────────────┴────────────┴────────────┴───────────┘
                    Orchestrated as one Databricks Job
                 (task dependencies, scheduled, run history)
```

---

## 🛠️ Tech Stack

- **Databricks** (Free Edition)
- **Delta Lake** — storage layer, ACID transactions, transaction history
- **Unity Catalog** — governance, access control, lineage, metadata
- **Auto Loader** — incremental raw data ingestion
- **PySpark** — data transformation
- **Databricks Workflows** — pipeline orchestration
- **Databricks SQL Dashboards** — visualization

---

## 📦 Dataset

NYC Yellow Taxi trip records, sourced from Databricks' built-in `samples.nyctaxi.trips`
table. A slice of this data is re-partitioned into daily files and written to a landing
volume to simulate a raw source-system feed — the same shape as data arriving from any
upstream capture system (including the Power Platform pipeline in Part 1) — which Auto
Loader then ingests incrementally.

---

## 🔄 Pipeline Stages

### 1️⃣ Bronze — Raw Ingestion
- Auto Loader incremental ingestion from the landing zone
- Schema-on-read with minimal transformation
- Captures ingestion metadata (source file, ingestion timestamp) for lineage/audit
- Registered as a managed Delta table in Unity Catalog

### 2️⃣ Silver — Cleaned and Validated
- Explicit schema enforcement (no reliance on type inference)
- Deduplication on a defined business key
- Null handling per documented business rules
- A data quality rule engine checks each record against business rules before
  promotion; failing records are quarantined (not discarded) for audit and reprocessing
- CHECK constraints enforced at the table level as a second, always-on guarantee

### 3️⃣ Gold — Business-Ready Aggregates
- Daily zone-level trip/revenue summary
- Hourly demand pattern (trip volume by hour-of-day / day-of-week)
- Optimized (OPTIMIZE + Z-ORDER) for downstream BI queries

---

## 🛡️ Governance & Data Quality

- Tables tagged with ownership, sensitivity classification, and domain metadata
- Full audit trail via Delta Lake transaction history
- Data quality rule engine with pass/fail reporting per rule, results logged to an
  auditable table
- End-to-end lineage visible in Unity Catalog, from the gold tables back to the raw
  landing files

---

## ⚙️ Orchestration

Bronze, Silver, and Gold stages are chained as a single Databricks Job with explicit
task dependencies, enabling scheduled, end-to-end execution with full run history —
the analytics-layer equivalent of the automated Power Automate workflows in Part 1.

---

## 📊 Visualization

A Databricks SQL Dashboard built on the Gold tables provides both a latest-snapshot view
and a historical trend view for monitoring pipeline output — closing the loop from raw
data to decision-making.

---

## 🧩 Key Design Decisions

- One catalog with three schemas (bronze/silver/gold) rather than three separate
  catalogs — simpler governance for a single-domain pipeline.
- Auto Loader runs in triggered batch mode (`availableNow`), matching a Job-orchestrated
  schedule rather than continuous streaming — the same code path scales to continuous
  ingestion if source volume justified it.
- Failed data quality checks route to a quarantine table rather than being dropped, so
  nothing is silently lost.
- CHECK constraints are enforced in addition to the rule engine, giving a guarantee that
  holds even if a future write bypasses the notebook logic.

---

## 📈 Results

*(to be filled in once the pipeline has been run end to end)*
- Rows ingested / after cleaning / quarantined:
- Data quality pass rate by rule:
- Gold table row counts:
- End-to-end job run time:

---

## 🚀 Setup & Reproduction

```
1. Clone this repository
2. Import notebooks into a Databricks workspace (Free Edition or higher)
3. Run 00_setup, then 00b_generate_landing_data
4. Run the orchestration job, or execute notebooks sequentially:
   01_bronze_ingestion → 02_silver_transform → 03_gold_aggregates
```

---

## 🔍 Skills Demonstrated

Delta Lake • Unity Catalog • Auto Loader • PySpark • Data Governance • Schema
Enforcement • Data Quality Validation • CHECK Constraints • Metadata Management •
Audit Logging • Pipeline Orchestration • Databricks SQL Dashboards

---

*Uses publicly available data. No confidential or proprietary information included.*
