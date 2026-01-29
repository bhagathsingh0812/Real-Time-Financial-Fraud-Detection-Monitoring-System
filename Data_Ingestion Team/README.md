
# ✅ Data Ingestion Team

## (Bronze Layer + Streaming Ingestion)

---

# 📌 Role: Data Ingestion Engineer

### Real-Time Financial Fraud Detection & Monitoring System

**Responsibility:** Build the real-time ingestion pipeline that brings raw transaction events into Databricks reliably.

---

## 1. Objective of Ingestion Layer

Banks generate millions of transactions per day.
Fraud detection systems must ingest this data **continuously** with:

* Low latency
* Fault tolerance
* Scalability
* Schema safety

The ingestion engineer ensures:

✅ Raw events arrive safely
✅ No transaction is lost
✅ Streaming pipeline is production-ready

---

## 2. Data Source (Free & Open)

### Dataset Used: IEEE-CIS Fraud Detection (Kaggle)

This dataset simulates real-world fraud behavior and contains:

### Transaction Table

* `TransactionAmt`
* `ProductCD`
* `card1–card6`
* `addr1, addr2`
* Fraud label: `isFraud`

### Identity Table

* `DeviceType`
* `DeviceInfo`

Joined by:

```
TransactionID
```

---

## 3. Why Streaming Simulation is Required

Real banking streams are not publicly available.
So we simulate real-time transactions using:

* Historical dataset replay
* Python streaming generator
* Incremental file arrival

This mimics:

* Kafka-style event streams
* Live banking transaction feeds

---

## 4. Streaming Data Generator (Phase 2)

### Purpose

Convert historical CSV rows into live streaming batches.

### Output Format

* JSON micro-batches
* Written every few seconds

Example:

```
batch_001.json
batch_002.json
batch_003.json
```

### Generator Folder

```
/Volumes/.../fraud_stream/input/
```

### Why JSON?

* Flexible schema
* Auto Loader friendly
* Supports nested metadata

---

## 5. Bronze Layer Design

### Bronze = Raw Streaming Landing Zone

Bronze stores:

* Raw transaction fields
* No transformations
* Append-only ingestion
* Source metadata

### Bronze Table

```
stream_bronze_data
```

Location:

```
angad_kumar91.fraud_detection_bronzelayer.stream_bronze_data
```

---

## 6. Auto Loader Ingestion (Structured Streaming)

### Why Auto Loader?

Auto Loader provides:

✅ Incremental ingestion
✅ Schema evolution
✅ Fault tolerance
✅ Exactly-once processing

### Core Streaming Code

```python
bronze_df = (
    spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .option("cloudFiles.schemaLocation", schema_path)
        .load(streaming_input_path)
)
```

---

## 7. Metadata Captured in Bronze

Auto Loader automatically adds:

| Column                 | Purpose                   |
| ---------------------- | ------------------------- |
| `_ingestion_timestamp` | When data arrived         |
| `_source_file`         | File origin               |
| `_file_size`           | File size                 |
| `_rescued_data`        | Corrupt/unexpected fields |

This is critical for:

* Audit compliance
* Debugging
* Replay capability

---

## 8. Streaming Write into Delta Bronze Table

```python
(
  bronze_df.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", checkpoint_path)
    .trigger(availableNow=True)
    .table("stream_bronze_data")
)
```

### Key Concepts

#### Checkpointing

Stores progress so Spark can recover after failure.

#### availableNow Trigger

Processes all available files once (best for POC).

---

## 9. Bronze Layer Guarantees

Bronze ensures:

✅ Raw truth preserved
✅ Data never overwritten
✅ Replay possible
✅ Foundation for Silver transformations

---

## 10. Common Interview Questions (Ingestion)

### Q1: Why Bronze Layer?

Because raw data must be stored unchanged for audit and reprocessing.

### Q2: Why Auto Loader instead of manual streaming?

Auto Loader handles:

* schema drift
* incremental discovery
* scalability

### Q3: What happens if ingestion fails?

Checkpoint ensures Spark resumes exactly where it stopped.

---

## 11. Output of Ingestion Engineer

By the end of this role, we deliver:

* Streaming transaction ingestion
* Bronze Delta table
* Fault-tolerant pipeline foundation

---
