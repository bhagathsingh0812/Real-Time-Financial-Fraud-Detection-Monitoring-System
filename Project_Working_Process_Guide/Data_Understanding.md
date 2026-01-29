

# 📘 README — Transaction Data Understanding for Fraud Detection 

## 📌 Purpose of This Document

This README explains:

✅ What transaction fields mean

✅ Which fields help detect fraud

✅ How fraud patterns appear in data

✅ How ML (Isolation Forest) uses these fields

✅ What parameters/features are most important

This is focused only on:

* **Data understanding**
* **Fraud detection relevance**
* **ML usage**

---

# 🎯 Why Data Fields Matter in Fraud Detection

Fraud is not detected using a single column.

Fraud is detected when a transaction looks **unusual** compared to normal behavior.

This means we look at:

* Amount anomalies
* Time bursts
* Card behavior
* Location mismatch
* Email identity risk
* Product irregularities

Machine learning learns these patterns automatically.

---

# ✅ Real Transaction Example (Live Data)

Below is a real transaction record:

```
TransactionID = 2987012
isFraud       = 0
TransactionDT = 86564
TransactionAmt= 50.00
ProductCD     = W
card1         = 3786
card2         = 418
card3         = 150
card4         = visa
card5         = 226
card6         = debit
addr1         = 204
addr2         = 87
dist1         = null
dist2         = null
P_emaildomain = verizon.net
R_emaildomain = null
```

This means:

> A normal debit card transaction of $50, using Visa, for product category W, with some missing distance and recipient email info.

---

# 🧾 Key Fields Used to Detect Fraud (Most Important)

---

# 1️⃣ Transaction Amount Fields

## **TransactionAmt**

### What it represents

Amount spent in the transaction.

### Why fraud detection uses it

Fraudsters often make:

* unusually high purchases
* abnormal spending compared to typical behavior

### Example

| TransactionAmt | Risk Level |
| -------------- | ---------- |
| 10 – 100       | Normal     |
| 2000+          | High Risk  |

### Feature derived

```python
is_high_value_txn = TransactionAmt > 1000
log_transaction_amount = log(TransactionAmt)
```

### Fraud intuition

> A debit card purchase of $5000 is rare → anomaly.

---

---

# 2️⃣ Time Behavior Fields

## TransactionDT + event_timestamp

### What it represents

Time of transaction (in seconds offset).

### Why important

Fraud often happens in bursts:

* many transactions in a few minutes
* unusual time-of-day activity

### Example

| Scenario         | Suspicious? |
| ---------------- | ----------- |
| 1 txn in 5 min   | Normal      |
| 25 txns in 5 min | Fraud burst |

---

### Feature created using window

```python
txn_count_5min = count(*) in last 5 minutes
```

Fraud pattern:

> Too many transactions too quickly → bot attack.

---

---

# 3️⃣ Card Identity Fields (Who is paying?)

Fraud repeats on specific cards.

---

## card1 (Primary Card ID)

### Example

```
card1 = 3786
```

### Why important

> Fraudsters often reuse stolen cards multiple times.

### Gold analysis

* Fraud rate per card
* High-risk cards list

---

## card4 (Network Type)

### Example

```
visa
mastercard
```

Different networks have different fraud patterns.

---

## card6 (Debit/Credit)

### Example

```
debit
credit
```

Fraud behavior differs:

| Type   | Fraud Style          |
| ------ | -------------------- |
| Debit  | smaller, quick fraud |
| Credit | large online fraud   |

Isolation Forest flags rare combinations:

> Debit card + huge transaction amount

---

---

# 4️⃣ Location & Address Fields

## addr1 and addr2

### Meaning

Billing and shipping region codes.

### Fraud relevance

Mismatch indicates suspicious activity.

### Example

| addr1 | addr2 | Risk       |
| ----- | ----- | ---------- |
| 204   | 204   | Normal     |
| 204   | 87    | Suspicious |

Feature derived

```python
is_international_txn = addr1 != addr2
```

Fraud intuition:

> Billing in one region, shipping elsewhere → possible fraud.

---

---

# 5️⃣ Distance Fields

## dist1, dist2

### Meaning

Distance between customer location and delivery.

### Fraud relevance

Fraudsters often ship goods far away.

Example:

| dist1  | Risk       |
| ------ | ---------- |
| 2 km   | Normal     |
| 600 km | Suspicious |

Null values mean unknown → uncertainty increases.

---

---

# 6️⃣ Email Domain Identity Fields

## P_emaildomain (Purchaser Email)

### Example

```
gmail.com
hotmail.com
verizon.net
```

Fraudsters often use:

* disposable domains
* fake temporary emails

Example:

| Domain         | Risk        |
| -------------- | ----------- |
| gmail.com      | Normal      |
| randommail.xyz | Fraud-prone |

---

## R_emaildomain (Recipient Email)

Mismatch is suspicious:

| Purchaser | Recipient   | Risk   |
| --------- | ----------- | ------ |
| gmail.com | gmail.com   | Normal |
| gmail.com | unknown.xyz | Fraud  |

---

---

# 7️⃣ Product Field

## ProductCD

### Example

```
W, C, H
```

Certain products have higher fraud risk.

Fraud pattern:

> Many different products purchased quickly.

Feature:

```python
product_diversity_5min
```

---

# 🔥 Most Important Fraud Detection Features in This Project

| Feature                | Detects               |
| ---------------------- | --------------------- |
| TransactionAmt         | High-value fraud      |
| txn_count_5min         | Transaction bursts    |
| avg_amount_5min        | Spending spikes       |
| stddev_amount_5min     | Erratic behavior      |
| product_diversity_5min | Bot shopping patterns |
| is_international_txn   | Location mismatch     |
| Email domain mismatch  | Fake identity fraud   |
| card1 risk profiling   | Repeat fraud attempts |

---

# 🤖 How Machine Learning Detects Fraud (Isolation Forest)

---

## Why Isolation Forest?

Fraud is rare.

Isolation Forest is good because:

* It does not need balanced labels
* It detects anomalies automatically

---

## What does it learn?

Isolation Forest learns:

> Normal transactions form dense clusters
> Fraud transactions are rare outliers

---

## Training Features Used

```python
feature_cols = [
    "TransactionAmt",
    "log_transaction_amount",
    "is_high_value_txn",
    "is_international_txn",
    "txn_count_5min",
    "avg_amount_5min",
    "stddev_amount_5min",
    "product_diversity_5min"
]
```

These represent:

* Amount behavior
* Location mismatch
* Velocity fraud patterns
* Product irregularity

---

## Key Model Parameters

```python
IsolationForest(
    n_estimators=200,
    contamination=0.02,
    random_state=42
)
```

### Explanation

| Parameter     | Meaning                                         |
| ------------- | ----------------------------------------------- |
| n_estimators  | Number of trees (more trees = better stability) |
| contamination | Expected fraud percentage (~2%)                 |
| random_state  | Reproducibility                                 |

---

## Model Output

Isolation Forest produces:

* anomaly_score
* anomaly label

```python
pdf["anomaly"] = model.predict(X)
```

| Output | Meaning            |
| ------ | ------------------ |
| -1     | Fraud-like anomaly |
| +1     | Normal transaction |

---

# ✅ Fraud Example Detected by Model

A transaction like:

```
TransactionAmt = 5000
txn_count_5min = 30
addr1 != addr2
P_emaildomain = disposable.xyz
product_diversity_5min = 10
```

Model conclusion:

> Extremely rare behavior → anomaly → fraud candidate

---

# 🎯 Final Beginner Summary

Fraud detection in this project works because:

* Transaction amount shows abnormal spending
* Time windows detect burst attacks
* Card identity detects repeat fraud patterns
* Address mismatch detects location fraud
* Email domains detect fake identities
* Isolation Forest flags rare combinations automatically

---


