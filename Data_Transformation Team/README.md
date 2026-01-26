
# 📗 README.md

## **Data Transformation Team**

---

## 🧭 Team Overview

The **Data Transformation Team** is responsible for converting **raw streaming data from the Bronze layer** into **clean, structured, analytics-ready datasets** in the **Silver layer**. This team applies **data quality rules, standardization, enrichment, and feature preparation** while preserving streaming semantics and ensuring downstream compatibility with **machine learning and real-time scoring**.

This team plays a **critical role** in stabilizing the pipeline by separating **raw ingestion concerns** from **business and ML-ready data modeling**.

---

## 🎯 Key Responsibilities

### 🧹 Data Cleaning & Standardization

* Type-casting raw string fields into correct data types
* Handling nulls, invalid records, and corrupt data
* Normalizing timestamps and numeric fields

### 🧪 Feature Preparation

* Create **row-level behavioral indicators**
* Preserve transactional granularity
* Ensure consistency across batch and streaming usage

### 🥈 Silver Layer Construction

* Build streaming-safe Silver tables
* Maintain schema stability
* Prepare datasets for both **ML training (batch)** and **real-time scoring (streaming)**

---

## 🏗️ Architectural Position

```
Bronze Layer (Raw Streaming)
        ↓
Silver Base (Clean Streaming Data)
        ↓
Batch Feature Engineering (for ML training)
        ↓
Real-Time Scoring & Gold Metrics
```

⚠️ **Important Design Rule:**
The Data Transformation Team does **NOT** train ML models or compute Gold metrics.

---

## 🥈 Silver Layer Strategy

### Why Silver Layer Is Critical

The Silver layer ensures:

* Clean and trusted data
* Reusability across teams
* Stability against upstream schema drift
* Safe interaction with ML pipelines

---

## 📂 Silver Tables Owned by This Team

| Table Name                 | Type      | Purpose                             |
| -------------------------- | --------- | ----------------------------------- |
| `silver_transactions_base` | Streaming | Clean, row-level transaction data   |
| `feature_matrix_table_5min`| Batch     | Feature preparation for ML training |

---

## 🧠 Core Design Decisions (Very Important)

### ❗ Why No Streaming Window Aggregations in Silver?

**Streaming window aggregations emit results only when windows close**, which caused:

* Empty tables under `availableNow`
* Unstable ML training datasets
* Difficult debugging

✅ **Decision:**
Silver layer contains **only row-level transformations**, not aggregations.

Windowed aggregations are computed **in batch** for ML training and **separately for analytics**.

---

## 🔄 Silver Base Streaming Table

### Purpose

Provide **clean, structured, streaming-safe transactional data**.

---

### Data Transformations Applied

#### 1️⃣ Type Casting

All raw string fields are cast to their correct types:

| Field             | Type      |
| ----------------- | --------- |
| `TransactionID`   | BIGINT    |
| `TransactionDT`   | BIGINT    |
| `TransactionAmt`  | DOUBLE    |
| `isFraud`         | INT       |
| `card1`           | INT       |
| `event_timestamp` | TIMESTAMP |

---

#### 2️⃣ Data Quality Rules

* Remove rows with null or invalid transaction amounts
* Ensure numeric values are positive
* Fill optional categorical fields with `"UNKNOWN"`

---

#### 3️⃣ Row-Level Feature Engineering

| Feature                  | Description                       |
| ------------------------ | --------------------------------- |
| `is_high_value_txn`      | Flag for high-amount transactions |
| `log_transaction_amount` | Log-scaled transaction value      |
| `is_international_txn`   | Address mismatch indicator        |

These features are **stateless** and **streaming-safe**.

---

### Silver Base Creation Logic (Conceptual)

```python
silver_base_df = (
    bronze_df
      .cast_and_clean_fields()
      .apply_row_level_features()
      .withWatermark("event_timestamp", "10 minutes")
)
```

---

## 🕒 Event-Time Handling

### Why Watermarks Are Used

* Handle late-arriving data
* Enable safe downstream joins
* Prevent unbounded state growth

⚠️ **Rule Enforced:**
Only **one event-time column** per streaming table.

---

## 🧪 Batch Feature Engineering for ML

### Why Batch Instead of Streaming?

Streaming windows:

* Require long-running queries
* Do not emit results in finite triggers
* Complicate ML reproducibility

### Batch Features Generated

* Transaction count per card
* Average transaction amount
* Standard deviation
* Merchant diversity

These features are:

* Deterministic
* Reproducible
* Suitable for ML training

---

## 🔐 Data Governance & Stability

* Schema explicitly defined
* Ingestion metadata preserved (but not propagated to Gold)
* No breaking schema changes
* Checkpoint isolation between pipelines

---

## 🚫 What This Team Does NOT Do

❌ ML model training
❌ Fraud scoring
❌ Alert generation
❌ Dashboard creation

Those responsibilities belong to the **ML Analytics** and **Dashboard teams**.

---

## 🧠 How This Team Adds Value

> Data quality issues propagate exponentially downstream.

This team:

* Prevents cascading failures
* Simplifies ML pipelines
* Enables stable real-time scoring
* Enforces architectural discipline

---

## 🎤 Summary

> “The Data Transformation Team converts raw streaming data into clean, structured Silver tables using streaming-safe transformations and batch-based feature engineering, ensuring stability, reproducibility, and ML compatibility.”

---

