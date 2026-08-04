# 🔗 Analytics Enablement Layer: Medallion Architecture on Databricks
**Role:** Data Engineer
**Focus:** Cloud data governance, incremental ingestion, data quality automation, and analytics-ready aggregation
**Series:** Part 2 of *End-to-End Operational Data Pipeline* — see [Part 1: Shopfloor Capture & Hybrid Cloud Ingestion](#) for the upstream capture layer this project's analytics layer plugs into.

---

## 🧭 Project Overview

Part 1 of this series covers shopfloor data capture and automated ingestion into hybrid
cloud storage. This project picks up from there and builds out the **analytics
enablement layer**: a governed medallion pipeline (Bronze → Silver → Gold) on Databricks
that takes raw incoming process records, applies schema enforcement and automated data
quality validation, and produces business-ready aggregates — with governance, lineage,
and orchestration built in rather than bolted on afterward.

The pattern mirrors how manufacturing process data (e.g. from forging or machining
operations) typically flows in practice: parameters are captured at the point of work,
pushed through a serverless ingestion layer into cloud object storage, then refined
into governed, analytics-ready tables — with downstream model development and BI
dashboarding closing the loop back to operations.

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

- **Databricks** 
- **Delta Lake** — storage layer, ACID transactions, transaction history
- **Unity Catalog** — governance, access control, lineage, metadata
- **Auto Loader** — incremental raw data ingestion
- **PySpark** — data transformation and synthetic data generation
- **Databricks Workflows** — pipeline orchestration
- **Databricks SQL Dashboards** — visualization

---

## 📦 Dataset

A synthetically generated dataset representing forging and machining process records —
part ID, process type, machine ID, process start/end timestamps, temperature, vibration,
pressure, and a pass/fail outcome — generated directly in PySpark, with no external
download or proprietary source required. A small proportion of deliberately invalid
values (out-of-range readings, negative sensor values, non-binary outcomes) is included
by design, to give the Silver layer's data quality rules and CHECK constraints something
real to detect and reject rather than passing trivially.

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
- A data quality rule engine checks each record against business rules — valid
  temperature range, non-negative vibration/pressure, binary pass/fail — before
  promotion; failing records are quarantined (not discarded) for audit and reprocessing
- CHECK constraints enforced at the table level as a second, always-on guarantee

### 3️⃣ Gold — Business-Ready Aggregates
- Daily process yield summary by machine and process type (pass/fail rate)
- Process parameter trend analysis (temperature/vibration/pressure over time, by machine)
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
task dependencies, enabling scheduled, end-to-end execution with full run history.

---

## 📊 Visualization

A Databricks SQL Dashboard built on the Gold tables provides both a latest-snapshot view
and a historical trend view of process yield and parameter stability — closing the loop
from raw data to decision-making.

---

## 🧩 Key Design Decisions

- One catalog with three schemas (bronze/silver/gold) rather than three separate
  catalogs — simpler governance for a single-domain pipeline.
- Raw data is synthetically generated rather than sourced externally, sidestepping any
  need for proprietary process data while keeping the pipeline's governance and quality
  logic representative of real manufacturing process records.
- Auto Loader runs in triggered batch mode (`availableNow`), matching a Job-orchestrated
  schedule rather than continuous streaming — the same code path scales to continuous
  ingestion if source volume justified it.
- Failed data quality checks route to a quarantine table rather than being dropped, so
  nothing is silently lost.
- CHECK constraints are enforced in addition to the rule engine, giving a guarantee that
  holds even if a future write bypasses the notebook logic.
- Stages are chained as a single Databricks Job with explicit task dependencies, rather
  than left as separate, independently-run notebooks — a common pattern in exploratory
  analytics environments where notebooks accumulate organically, but one that makes
  execution order and failure handling implicit rather than enforced.

---


## 🔍 Skills Demonstrated

Delta Lake • Unity Catalog • Auto Loader • PySpark • Synthetic Data Generation • Data
Governance • Schema Enforcement • Data Quality Validation • CHECK Constraints •
Metadata Management • Audit Logging • Pipeline Orchestration • Databricks SQL
Dashboards

---

> ⚠️ **Disclaimer:** This project uses a synthetically generated dataset representative
> of manufacturing process data. No confidential or proprietary information is included.
