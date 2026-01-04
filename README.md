# FMCG Post-Merger Data Integration Pipeline

## 📋 Project Overview
**Scenario:** A large FMCG company ("Velocity Gear") has acquired a smaller startup ("Vitality Nutrition").  

**The Problem:** The startup stores data in disparate, unmanaged CSV files with different schema standards (e.g., different column names, ID formats) compared to the parent company.  

**The Goal:** Build an ELT pipeline to ingest, clean, and merge the startup's data into Velocity Gear's existing enterprise Data Lakehouse to enable unified reporting.

This project implements a **Medallion Architecture** using **Databricks** and **Delta Lake** to ensure data quality and consistency during the migration.

---

## 🏗️ Architecture

<img width="1182" height="637" alt="image" src="https://github.com/user-attachments/assets/dd009b0e-fc04-4efb-b41e-ae7f6ddf6cdf" />

---

## 🛠️ Tech Stack

### ☁️ Cloud Storage
- **AWS S3**  
  Acts as the **Source Data Lake**, storing raw and intermediate datasets.

### ⚙️ Compute
- **Apache Spark (Databricks Community Edition)**  
  Used for large-scale distributed data processing and analytics.

### 🔄 Data Transformation
- **PySpark**  
  For scalable ETL and data transformations.
- **Spark SQL**  
  Enables SQL-based querying and transformations on big data.

### 🗄️ Storage Format
- **Delta Lake**  
  - Provides **ACID transactions**
  - Ensures **schema enforcement & evolution**
  - Supports **time travel** and data versioning

### 🧭 Orchestration
- **Databricks Notebook Workflows**  
  Used to schedule, manage, and automate end-to-end data pipelines.

---

## 🧩 Implementation Details

### 1️⃣ Ingestion — Bronze Layer
- Connected **Databricks** to **AWS S3**
- Ingested raw CSV files:
  - Orders
  - Products
  - Customers
  - Gross_Price
- Stored data as **Delta Tables**

**Why Delta Lake?**
- Preserves raw historical data
- Supports **data versioning (time travel)**
- Provides higher reliability compared to raw CSV files

---

### 2️⃣ Transformation & Data Quality — Silver Layer

#### 🔧 Schema Standardization
- Renamed columns to align with the **parent company’s naming conventions**

#### 🧹 Data Cleaning
- Standardized inconsistent location names  
  - Example: `Blr`, `Bangalore` → `Bengaluru`
- Removed duplicate records
- Handled missing and null values

#### 🔑 Handling Key Collisions
**Challenge**  
- The startup used simple integer-based IDs (e.g., `101`)
- These conflicted with existing IDs in the parent company’s database

**Solution**  
- Generated **surrogate keys** (`product_code`)
- Used **SHA-256 hashing** on unique business attributes:
  - `Product Name + Category`
- Ensured **global uniqueness** across systems

---

### 3️⃣ Merging & Aggregation — Gold Layer

#### 📊 Granularity Adjustment
- Aggregated **daily transactional data** into **monthly sales totals**
- Aligned with the parent company’s reporting standards

#### 🔁 Incremental Loading
- Implemented using **Delta Lake `MERGE INTO`**

**Merge Logic**
- If record exists → **Update**
- If record does not exist → **Insert**

**Benefits**
- Ensures **idempotent pipeline execution**
- Allows safe re-runs without creating duplicate records
- Supports scalable and reliable reporting datasets

---

## 📊 Results

The final data pipeline feeds a **Databricks Dashboard** that provides a unified and trusted view of the business.

### Key Insights
- **Total Revenue**  
  Consolidated revenue across both the parent company and the acquired startup.
- **Monthly Sales Trend**  
  Visualized month-over-month growth patterns, identifying seasonality spikes (e.g., increased sales in Q4).
- **Sales by Category**
  Provided a breakdown of top-performing product lines to assist inventory planning.

  <img width="1402" height="661" alt="image" src="https://github.com/user-attachments/assets/a50371b6-f50f-47a1-9831-d33af7f9be06" />


---

## 🔮 Future Improvements (Production Considerations)

To make this pipeline production-ready for a large-scale enterprise environment, the following enhancements can be implemented:

### 🧭 Orchestration
- Use **Databricks Workflows** or **Apache Airflow** to schedule and monitor daily pipeline executions.

### ✅ Data Quality
- Integrate **Great Expectations** to enforce data quality rules.
- Fail the pipeline on critical checks (e.g., negative revenue, null primary keys).

### 🚀 CI/CD
- Implement **GitHub Actions** to:
  - Run automated tests
  - Validate notebooks
  - Deploy changes across environments

### 🏗️ Infrastructure as Code
- Use **Terraform** to provision and manage:
  - AWS S3 buckets
  - Databricks clusters
- Ensures secure, repeatable, and version-controlled infrastructure.
