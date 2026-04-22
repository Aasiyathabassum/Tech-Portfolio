# Delta Lake Data Governance Pipeline
**Role:** Data Engineer  
**Focus:** Data ingestion, schema enforcement, Delta Lake governance, Unity Catalog, data quality

---

## Project Overview

This project demonstrates the design and implementation of an **end-to-end data governance pipeline** using **Databricks and Unity Catalog**. Raw manufacturing sensor data is ingested, validated, and written to a governed Delta table with governance controls embedded at every stage — from schema enforcement at ingestion to constraint-based validation at write time.

The pipeline addresses a common real-world challenge: raw data that is unowned, unvalidated, and undiscoverable. By applying governance principles directly in the technical implementation, the solution ensures data is **trustworthy, traceable, and accountable** from the moment it enters the platform.

---

##  Problem Context

Organisations managing large volumes of operational data often face:

- Raw data with inconsistent types and no enforced structure
- No ownership or accountability when pipelines fail
- Bad data silently entering tables and corrupting downstream analytics
- No visibility into who changed what and when
- Datasets that are undiscoverable and poorly documented

This project addresses these challenges by building governance directly into the data pipeline rather than treating it as an afterthought.

---

##  Key Contributions

- Designed and implemented an **end-to-end governed data pipeline** on Databricks
- Defined an **explicit schema** with nullable constraints based on business rules — replacing unreliable type inference
- Registered the dataset as a **managed Delta table in Unity Catalog**, making it discoverable and governed across the platform
- Applied **CHECK constraints** to enforce data validity rules at write time, actively blocking invalid records
- Tagged the table with **governance metadata** including owner, sensitivity classification, domain, and review date
- Queried the **Delta transaction history** to surface a full audit trail of every operation performed on the table
- Built a **data quality checker** that validates the dataset against business rules and reports pass/fail results

---

## 🔄 Pipeline Overview

### 1️⃣ Data Ingestion
- Raw CSV uploaded to a **Unity Catalog Volume** — a governed storage location
- Extracted and loaded into a Databricks notebook for processing

### 2️⃣ Schema Enforcement
- Explicit schema defined with correct data types per column
- Nullability rules applied based on business logic:
  - Identifiers and critical flags set to non-nullable
  - Sensor readings allowed to be nullable to handle real-world gaps

### 3️⃣ Delta Table Registration
- Data written as a **managed Delta table** registered in Unity Catalog
- Immediately discoverable, ownable, and permissionable across the organisation

### 4️⃣ Constraint-Based Validation
- CHECK constraints added directly on the table:
  - Temperature readings must fall within a physically valid range
  - Failure flags must be binary values only
- Constraints tested by attempting to insert invalid records — violations caught and rejected at write time

### 5️⃣ Governance Metadata Tagging
- Table tagged with ownership, sensitivity level, domain, and last reviewed date
- Ensures accountability and discoverability without relying on external documentation

### 6️⃣ Audit Trail
- Delta transaction history queried to surface every operation performed on the table
- Each entry records the operation type, user, timestamp, and parameters — providing full traceability

### 7️⃣ Data Quality Checks
- Quality checker built to validate three business rules:
  - No null values in key identifier columns
  - All failure flags contain valid values only
  - No physically impossible negative values in sensor readings
- Results reported as pass/fail per check

---

## 🛠️ Tools & Technologies

- Databricks (Serverless)
- Unity Catalog
- Delta Lake
- PySpark
- Python

---

## 📈 Business & Technical Impact

- Established **schema enforcement at ingestion** — preventing type mismatches from propagating downstream
- Enabled **constraint-based data rejection** — bad data never reaches the table
- Improved **data discoverability and ownership** through Unity Catalog registration and metadata tagging
- Created a **full audit trail** enabling traceability of every change made to the dataset
- Delivered a **reusable governance pattern** applicable to any structured dataset ingested into a Databricks lakehouse

---

## 🔍 Skills Demonstrated

Delta Lake • Unity Catalog • PySpark • Data Governance • Schema Enforcement • Data Quality • Constraint Validation • Metadata Management • Audit Logging • Databricks • Python

---

> ⚠️ **Disclaimer:** This project uses a publicly available dataset from the UCI Machine Learning Repository. No confidential or proprietary data is included.
