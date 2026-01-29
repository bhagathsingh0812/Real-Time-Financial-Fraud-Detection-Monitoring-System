
# ✅ ML Engineer

## Isolation Forest + MLflow + Real-Time Fraud Scoring

---

# 📌 Role: Machine Learning Engineer

### Real-Time Financial Fraud Detection & Monitoring System

**Responsibility:** Train anomaly detection model, register with MLflow, and deploy it for real-time fraud scoring.

---

## 1. Objective of ML in Fraud Detection

Fraud detection is fundamentally an **anomaly detection problem**:

* Fraud transactions are rare (~3.5%)
* Fraud patterns evolve continuously
* Labels are limited or delayed

So instead of supervised classification, we use:

✅ Unsupervised anomaly detection
→ Isolation Forest

Goal:

> Assign each transaction a fraud_score in real time

---

## 2. Why Isolation Forest?

Isolation Forest is ideal because:

* Works without needing many fraud labels
* Handles high-dimensional features
* Efficient for large-scale streaming scoring

### Core Idea

Fraud transactions are:

* rare
* different
* easier to isolate in decision trees

Isolation Forest outputs:

* anomaly score
* prediction (-1 anomaly, +1 normal)

---

## 3. ML Training Dataset Source

We train using Silver layer output:

```
silver_transactions_base
```

This contains:

* clean schema
* row-level fraud features
* ML-ready columns

---

## 4. Feature Set Used in Model Training

Model expects exactly these 8 features:

| Feature                | Type   | Meaning             |
| ---------------------- | ------ | ------------------- |
| TransactionAmt         | double | Transaction amount  |
| log_transaction_amount | double | Normalized amount   |
| is_high_value_txn      | int    | High-value flag     |
| is_international_txn   | int    | Location mismatch   |
| txn_count_5min         | long   | Burst frequency     |
| avg_amount_5min        | double | Spending average    |
| stddev_amount_5min     | double | Spending volatility |
| product_diversity_5min | long   | Merchant diversity  |

These features capture both:

* transaction-level risk
* behavioral anomalies

---

## 5. Model Training Pipeline (Industry Standard)

We build a scikit-learn pipeline:

```python
pipeline = Pipeline(steps=[
    ("scaler", StandardScaler()),
    ("iforest", IsolationForest(
        n_estimators=200,
        contamination=0.035,
        random_state=42
    ))
])
```

### Why scaling?

Isolation Forest performs better when features are normalized.

---

## 6. Training Process

We convert Spark → Pandas (POC scale):

```python
pdf = training_df.select(feature_cols + ["isFraud"]).toPandas()

X = pdf[feature_cols]
y = pdf["isFraud"]
```

Then train:

```python
pipeline.fit(X)
```

---

## 7. Fraud Scoring Logic

Isolation Forest outputs decision scores:

```python
raw_scores = pipeline.named_steps["iforest"].decision_function(X)
fraud_scores = -raw_scores
```

Meaning:

* Higher fraud_score → more suspicious

We define anomaly flag:

```python
is_anomaly = fraud_score > threshold
```

---

## 8. Model Evaluation (POC Metrics)

Even though model is unsupervised, we use labels only for evaluation:

### ROC-AUC

```python
roc_auc = roc_auc_score(y, fraud_scores)
```

### PR-AUC

```python
precision, recall, _ = precision_recall_curve(y, fraud_scores)
pr_auc = auc(recall, precision)
```

These metrics help validate detection performance.

---

# ✅ MLflow Integration (Enterprise Requirement)

---

## 9. Why MLflow?

MLflow provides:

* experiment tracking
* model versioning
* registry deployment
* reproducibility

Fraud models must be:

* auditable
* controlled
* version-managed

---

## 10. Logging Model with Signature (Unity Catalog)

Unity Catalog requires:

* input schema signature
* input example

```python
from mlflow.models.signature import infer_signature

signature = infer_signature(X, pipeline.predict(X))

mlflow.sklearn.log_model(
    pipeline,
    artifact_path="model",
    signature=signature,
    input_example=X.iloc[:5]
)
```

### Why signature matters?

Prevents inference errors by enforcing feature schema.

---

## 11. Model Registration

```python
mlflow.sklearn.log_model(
    pipeline,
    artifact_path="model",
    registered_model_name="fraud_isolation_forest"
)
```

Now model appears in Databricks Model Registry.

---

## 12. Unity Catalog Deployment via Alias

Stages like Production are not supported in UC.

Instead we assign alias:

```
@prod
```

Model load:

```python
model_uri = "models:/fraud_isolation_forest@prod"
fraud_model = mlflow.pyfunc.load_model(model_uri)
```

This is industry deployment standard.

---

# ✅ Real-Time Fraud Scoring (Streaming Deployment)

---

## 13. Streaming Scoring Architecture

```
Silver Base Stream
     ↓
Batch behavioral snapshot
     ↓
MLflow model inference
     ↓
Gold fraud_predictions table
```

---

## 14. Why foreachBatch?

Spark ML models cannot directly run inside streaming transformations.

So we use:

```python
.writeStream.foreachBatch(score_batch)
```

This allows:

* micro-batch Pandas inference
* safe Gold writes

---

## 15. Production Scoring Function

Key steps:

1. Convert batch → Pandas
2. Cast types strictly
3. Apply model
4. Write Gold results

```python
def score_batch(batch_df, batch_id):

    pdf = batch_df.toPandas()

    X = pdf[feature_cols].astype(expected_types)

    anomaly_scores = fraud_model.predict(X)

    pdf["fraud_score"] = -anomaly_scores
    pdf["is_anomaly"] = (pdf["fraud_score"] > 0.6)

    spark.createDataFrame(pdf_out).write.insertInto("fraud_predictions")
```

---

## 16. Gold Output Table

Final fraud scoring table:

```
fraud_predictions
```

Contains:

| Column          | Purpose          |
| --------------- | ---------------- |
| TransactionID   | Transaction key  |
| event_timestamp | Event time       |
| TransactionAmt  | Amount           |
| fraud_score     | Suspicion score  |
| is_anomaly      | Fraud alert flag |

This table powers dashboards.

---

## 17. Key Streaming Lesson: Checkpoints

Streaming writes depend on checkpoint state.

If rows read = 0:

* no new data since checkpoint
* reset checkpoint during development

Production:

* checkpoint stays stable

---

# 🎤 Interview Questions (ML Engineer)

---

### Q1: Why Isolation Forest?

Fraud is rare, unsupervised anomaly detection is practical.

### Q2: Why MLflow?

Enterprise requires model versioning, governance, reproducibility.

### Q3: Why model signature?

Ensures inference schema consistency and prevents silent errors.

### Q4: Why foreachBatch?

Allows applying Python ML models safely in streaming pipelines.

### Q5: What does fraud_score mean?

Higher score = transaction is more anomalous and suspicious.

---

# ✅ Output of ML Engineer

ML Engineer delivers:

* Isolation Forest anomaly model
* MLflow registered deployment
* Real-time scoring pipeline
* Gold fraud prediction dataset

---
