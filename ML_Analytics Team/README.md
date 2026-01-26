
# 📙 README.md

## **ML & Analytics Team**

---

## 🧭 Team Overview

The **ML & Analytics Team** is responsible for designing, training, validating, deploying, and operationalizing machine learning models to **detect suspicious financial transactions in real time**.

This team consumes **clean Silver-layer data**, applies **anomaly detection models**, performs **model lifecycle management using MLflow**, and delivers **ML-scored outputs into the Gold layer** for downstream dashboards and business decisions.

---

## 🎯 Key Responsibilities

### 🤖 Model Development

* Select suitable fraud detection algorithms
* Train models using historical behavioral data
* Validate performance using labeled data

### 📦 Model Lifecycle Management

* Track experiments with MLflow
* Register models using Unity Catalog
* Version and deploy models safely

### ⚡ Real-Time Fraud Scoring

* Apply trained models to streaming transactions
* Generate fraud scores and anomaly flags
* Write scored outputs into Gold tables

### 📊 Analytical Outputs

* Enable fraud trend analysis
* Support dashboard visualizations
* Provide explainable metrics for stakeholders

---

## 🧠 Model Selection Strategy

### Why Anomaly Detection?

Fraud is:

* Rare (~3.5%)
* Evolving
* Often unlabeled in real-time

### Chosen Model: **Isolation Forest**

| Reason        | Explanation                    |
| ------------- | ------------------------------ |
| Unsupervised  | Does not rely solely on labels |
| Scalable      | Handles large feature sets     |
| Interpretable | Produces anomaly scores        |
| Proven        | Widely used in financial fraud |

---

## 📊 Training Data Preparation

### Input Sources

* `silver_transactions_base`
* Batch-generated behavioral features

### Feature Categories

| Type              | Examples                  |
| ----------------- | ------------------------- |
| Transaction-level | Amount, log amount        |
| Behavioral        | Txn count, avg amount     |
| Risk indicators   | High-value, international |

### Key Design Rule

> **Training is always done in BATCH.**

Streaming data is **never used directly** for training due to:

* Non-determinism
* Late-arriving data
* Reproducibility issues

---

## 🧪 Model Training Workflow

```
Silver Base Table
        ↓
Batch Feature Aggregation
        ↓
Feature Vector Construction
        ↓
Isolation Forest Training
        ↓
Model Evaluation
        ↓
MLflow Registration
```

---

## 📦 MLflow Integration (Unity Catalog)

### Why MLflow?

* Experiment tracking
* Parameter & metric logging
* Model versioning
* Deployment control

### Key Constraints Addressed

* Unity Catalog does not support model stages
* Model aliases are used instead
* Model signatures are mandatory

---

## 🧠 Model Evaluation Metrics

Although the model is unsupervised, **labels are used ONLY for evaluation**:

| Metric  | Purpose                 |
| ------- | ----------------------- |
| ROC-AUC | Ranking effectiveness   |
| PR-AUC  | Fraud detection quality |

---

## ⚡ Real-Time Scoring Architecture

### Scoring Strategy

| Aspect    | Decision       |
| --------- | -------------- |
| Training  | Batch          |
| Scoring   | Streaming      |
| Execution | `foreachBatch` |
| Trigger   | `availableNow` |

### Why `foreachBatch`?

* Allows ML inference on micro-batches
* Enables Pandas-based ML inference
* Avoids streaming state complexity

---

## 🥇 Gold Layer Outputs

### Primary Table

| Table               | Purpose                |
| ------------------- | ---------------------- |
| `fraud_predictions` | ML-scored transactions |

### Schema

| Column          | Description       |
| --------------- | ----------------- |
| TransactionID   | Unique ID         |
| event_timestamp | Event time        |
| card1           | Entity identifier |
| TransactionAmt  | Amount            |
| fraud_score     | Anomaly score     |
| is_anomaly      | Binary flag       |

---

## 🚨 Operational Challenges Solved

### 1️⃣ Checkpoint Reprocessing

* `availableNow` processes only new data
* Checkpoint reset used during development
* Stable checkpoints in production

### 2️⃣ Empty Streaming Batches

* Handled gracefully in `foreachBatch`
* Prevented table creation failures

### 3️⃣ Schema Drift & Metadata Conflicts

* Gold tables pre-created
* Ingestion metadata excluded

---

## 📊 Analytical Capabilities Enabled

The outputs of this team power:

* Fraud trend analysis
* High-risk entity detection
* Alert dashboards
* Business impact reporting

---

## 🚫 What This Team Does NOT Do

❌ Raw data ingestion
❌ Data cleaning
❌ Dashboard UI design

Those responsibilities belong to **Ingestion**, **Transformation**, and **Dashboard teams**.

---

## 🧠 How This Team Adds Value

> A model is only valuable if it can operate reliably in real time.

This team ensures:

* Trustworthy fraud detection
* Scalable ML deployment
* Business-aligned outputs
* End-to-end ML accountability

---

## 🎤 Summary

> “The ML & Analytics Team operationalizes anomaly detection models using Isolation Forest and MLflow, enabling real-time fraud scoring through structured streaming and delivering analytics-ready Gold outputs for monitoring and decision-making.”

---


