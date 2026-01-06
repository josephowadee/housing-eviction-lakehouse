# NYC Housing & Eviction Lakehouse (Databricks)

## Overview
This project implements a Databricks-style **lakehouse data pipeline** for NYC eviction filings using a **Bronze / Silver / Gold (medallion) architecture**. The pipeline ingests public housing data, applies standardization and governance rules, enforces data quality checks, and produces analytics-ready tables for reporting and dashboards.

The design mirrors how a public-sector data warehouse or lakehouse would be built to support housing programs, eviction prevention efforts, and operational reporting.

---

## Data Source

**NYC Open Data – Evictions Dataset (Socrata)**

- Dataset page: https://data.cityofnewyork.us/Housing-Development/Evictions/6z8x-wfk4  
- SODA2 CSV API (snapshot export used for ingestion): https://data.cityofnewyork.us/resource/6z8x-wfk4.csv?$limit=200000  
- OData v4 endpoint (alternate interface): https://data.cityofnewyork.us/api/odata/v4/6z8x-wfk4  
- Socrata API docs: https://dev.socrata.com/

**Ingestion Strategy:** Point-in-time snapshot for auditability.  
In a production environment, this pipeline would be scheduled and extended to incremental ingestion using authenticated Socrata APIs and/or OData for change-based sync patterns.

---

## Architecture
NYC Open Data (Socrata)
|
v
+––––––––––+
|  Bronze Layer      |
|  Raw Snapshot      |
|  Delta Table       |
+––––––––––+
|
v
+––––––––––+
|  Silver Layer      |
|  Standardized      |
|  Deduplicated      |
|  Governed          |
+––––––––––+
|
v
+––––––––––+
|  Gold Layer        |
|  Analytics Tables  |
|  Data Quality      |
+––––––––––+
|
v
Databricks SQL / BI
Dashboards & Reports

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/ee5d27ad-198a-4987-9177-7fd38e080f93" />

---

## Medallion Layers

### Bronze Layer
**Purpose:** Preserve the raw source state for traceability and auditability.

- Table: `bronze_nyc_evictions`
- Characteristics:
  - Immutable snapshot
  - Minimal parsing
  - No business logic applied

Bronze is intentionally permissive and retains all source records.

---

### Silver Layer
**Purpose:** Enforce minimum analytical viability and governance.

- Table: `silver_nyc_evictions`
- Transformations applied:
  - String normalization (trimmed fields)
  - Date standardization (`executed_date`)
  - Required-field completeness checks
  - Deduplication using a **business key**

**Business Key:** `court_index_number`  
This uniquely identifies an eviction case assigned by Housing Court. Silver enforces one row per case to prevent inflated analytics.

Invalid or incomplete records remain available in Bronze for audit and reprocessing.

---

### Gold Layer
**Purpose:** Provide analytics-ready, consumption-focused datasets.

Gold tables are designed to be queried directly by dashboards, reports, or downstream systems.

#### Analytics Tables
- `gold_evictions_by_zip`
  - Eviction counts by ZIP code
- `gold_evictions_by_borough_month`
  - Monthly eviction trends by borough

These tables function as **fact-style aggregations** optimized for reporting.

---

## Data Quality Strategy

Data quality checks are executed **after Silver transformations** and persisted as a Gold table to provide monitoring and auditability.

### Data Quality Table
- Table: `gold_dq_nyc_evictions`
- One row per pipeline run
- Metrics captured:
  - Total row count
  - Null counts for key fields
  - Date coverage (min / max executed date)
  - Run timestamp

This pattern supports:
- Historical quality trend analysis
- Early detection of ingestion or schema issues
- Transparency for regulated environments

<img width="986" height="468" alt="data quality" src="https://github.com/user-attachments/assets/02729b22-bf1a-4813-a498-6e025bd14305" />
 

---

## Dashboards
A lightweight Databricks SQL dashboard is built on **Gold tables only**, ensuring separation of transformation logic and analytics.

Included views:
- Evictions over time (monthly trend)
- Share of evictions by ZIP code (top ZIPs)
- Latest data quality check

Dashboards consume curated tables and do not embed transformation logic.

---

## Jobs & Workflow Readiness
The pipeline is structured as a deterministic, idempotent workflow:

1. Bronze ingestion
2. Silver standardization and deduplication
3. Gold analytics table creation
4. Gold data quality metrics persistence

Each stage can be promoted into a **Databricks Job** with task dependencies and scheduling. The current implementation focuses on pipeline logic and governance rather than orchestration UI.

---

## Repository Structure
housing-eviction-lakehouse/
├── README.md
├── notebooks/
│   └── Housing_Eviction_Lakehouse.ipynb
├── sql/
│   ├── gold_evictions_by_zip.sql
│   ├── gold_evictions_by_borough_month.sql
│   └── gold_dq_nyc_evictions.sql
├── docs/
│   └── dashboard.png

> Note: Raw source data is not stored in this repository.

---

## Key Takeaways
- Implements a Databricks lakehouse using medallion architecture
- Demonstrates ingestion, transformation, data modeling, and governance
- Includes explicit data quality monitoring
- Designed for public-sector, audit-sensitive environments
- Analytics-ready and dashboard-enabled

---

## Production Considerations
In a production environment, this pipeline would be extended with:
- Scheduled Databricks Jobs
- Incremental ingestion via authenticated APIs
- CI/CD for notebook promotion
- Role-based access controls
- Performance tuning and cost optimization

---

## Author
Joseph Owadee
