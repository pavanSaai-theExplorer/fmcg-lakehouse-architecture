# FMCG Lakehouse Architecture | Databricks & AWS

![Python](https://img.shields.io/badge/Python-3.9-blue)
![Spark](https://img.shields.io/badge/Apache%20Spark-3.3-orange)
![Databricks](https://img.shields.io/badge/Databricks-Runtime%2012.2-red)
![AWS](https://img.shields.io/badge/AWS-S3-yellow)
![Status](https://img.shields.io/badge/Pipeline-Production%20Ready-success)

## 📖 Executive Summary
This project implements a scalable **Data Lakehouse Architecture** to solve a complex data integration challenge in the FMCG sector. Designed to support a Merger & Acquisition (M&A) scenario, the pipeline ingests, cleans, and merges disconnected legacy data streams into a unified **Single Source of Truth** for analytics.

Built on **Databricks** and **AWS S3**, the solution leverages the **Medallion Architecture** to ensure data quality and utilizes **Delta Lake** for ACID transactions and efficient incremental processing.

---

## 🏢 The Business Challenge
**Scenario:** A leading FMCG giant, *AtliQ*, acquired a niche startup, *Sports Bar*.
**Problem:** The startup's sales, customer, and product data existed in isolated legacy formats (CSVs) with inconsistent schemas and data quality issues (typos, duplicates, nulls).
**Objective:** Integrate the startup's data into AtliQ's master data warehouse without disrupting existing analytics, enabling a unified view of the post-merger revenue.

---

## 🏗 System Architecture
The pipeline follows the industry-standard **Medallion Architecture**:

| Layer | Type | Responsibility | Storage Format |
| :--- | :--- | :--- | :--- |
| **Bronze** | Raw | Ingest raw CSVs from AWS S3 "as-is". | Delta Table |
| **Silver** | Clean | Schema enforcement, deduplication, null handling, and standardization. | Delta Table |
| **Gold** | Aggregated | Business-level aggregates and Star Schema for BI. | Delta Table |

*(Note: Add your architecture diagram image here if you have one, e.g., `![Architecture Diagram](path/to/image.png)`)_

---

## 🛠 Tech Stack & Decisions

* **Compute:** **Apache Spark (PySpark)** on Databricks Community Edition.
    * *Why?* Distributed processing capabilities required for future scaling of FMCG transaction volumes.
* **Storage:** **AWS S3** (Bronze landing zone) & **Delta Lake**.
    * *Why?* S3 provides cost-effective object storage; Delta Lake brings ACID compliance and time-travel capabilities to the data lake.
* **Orchestration:** **Databricks Workflows**.
    * *Why?* To automate daily incremental loads (CDC).
* **Visualization:** **Databricks SQL**.
    * *Why?* Direct integration with the Gold layer for low-latency reporting.

---

## ⚙️ Key Engineering Implementations

### 1. Robust Data Ingestion (Bronze)
* Established secure mounting between Databricks and AWS S3 using access keys.
* Implemented a parameterized reader to handle both historical (full load) and daily (incremental) files dynamically.

### 2. Advanced Transformations (Silver)
* **Schema Enforcement:** Converted loose CSV string types to strict Spark `DateType` and `IntegerType`.
* **Data Quality Guardrails:** * Regex-based cleaning (e.g., standardizing `protien` -> `protein`).
    * Null handling strategies (Mean imputation for metrics, default values for categorical).
* **Deduplication:** Window functions (`row_number()`) used to identify and remove duplicate transaction IDs.

### 3. Upsert Strategy & Data Merging (Gold)
* Implemented **SCD Type 1 (Slowly Changing Dimensions)** using Delta Lake's `MERGE INTO` command.
* **Logic:** * *Match Found:* Update existing customer records.
    * *No Match:* Insert new customer records.
* This ensures the Master Data remains current without creating duplicate entries during daily loads.

```python
# Snippet: Delta Lake Upsert Logic
deltaTable.alias("t").merge(
    source_df.alias("s"),
    "t.customer_id = s.customer_id"
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()
