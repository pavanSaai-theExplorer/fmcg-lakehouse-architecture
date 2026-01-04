# FMCG Post-Merger Data Integration Pipeline

## 📋 Project Overview
**Scenario:** A large FMCG company ("AtliQ") has acquired a smaller startup ("Sports Bar").  
**The Problem:** The startup stores data in disparate, unmanaged CSV files with different schema standards (e.g., different column names, ID formats) compared to the parent company.  
**The Goal:** Build an ELT pipeline to ingest, clean, and merge the startup's data into AtliQ's existing enterprise Data Lakehouse to enable unified reporting.

This project implements a **Medallion Architecture** using **Databricks** and **Delta Lake** to ensure data quality and consistency during the migration.

---

## 🏗️ Architecture

```mermaid
graph TD
    subgraph Source [AWS S3]
        CSV[Raw CSV Files]
    end

    subgraph Databricks [Databricks Lakehouse]
        Bronze[("Bronze Layer<br>(Raw Delta)")]
        Silver[("Silver Layer<br>(Cleaned & Hashed)")]
        Gold[("Gold Layer<br>(Aggregated Facts)")]
        
        CSV -->|Ingest w/ Schema Inference| Bronze
        Bronze -->|PySpark Cleaning| Silver
        Silver -->|MERGE / Upsert| Gold
    end

    subgraph BI [Analytics]
        Dash[Databricks Dashboard]
        Gold --> Dash
    end
