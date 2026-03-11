# 🛢️ Oracle DBA Intelligence Dataset — Morgan Stanley
### Backup / Restore Failure Prediction & Deadlock Blocking Forecaster

[![Dataset](https://img.shields.io/badge/Dataset-75%2C000%20rows-blue?style=flat-square&logo=databricks)](.)
[![Columns](https://img.shields.io/badge/Columns-57-green?style=flat-square)](.)
[![ML Ready](https://img.shields.io/badge/ML--Ready-Yes-brightgreen?style=flat-square&logo=scikit-learn)](.)
[![Oracle](https://img.shields.io/badge/Oracle-Database-red?style=flat-square&logo=oracle)](.)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](.)

---

## 📌 Overview

This is a **production-grade, synthetic Oracle DBA monitoring dataset** built to simulate real-world database incidents at an enterprise financial institution (Morgan Stanley scale). It is purpose-built for training machine learning models to predict two of the most critical Oracle DBA pain points:

1. **Backup & Restore Failure Prediction** — Predict backup job failures before they bomb your midnight restore window
2. **Deadlock & Blocking Forecaster** — Forecast deadlocks and session blocking events before they cascade into outages

The dataset mirrors the kind of telemetry a Senior Oracle DBA would pull from:
- `v$session`, `v$lock`, `v$waitstat`
- `v$rman_backup_job_details`
- `v$archived_log`, `v$log`
- `v$standby_log`, `v$dataguard_stats`
- `dba_jobs_running`, `v$undostat`

---

## 🏗️ Dataset Specifications

| Property | Value |
|---|---|
| **Rows** | 75,000 |
| **Columns** | 57 |
| **Time Range** | March 2025 – March 2026 |
| **Servers** | 56 unique Oracle DB servers |
| **Data Centers** | New York · New Jersey · London · Tokyo · Hong Kong · Frankfurt |
| **Format** | CSV (UTF-8) |
| **Size** | ~28 MB |
| **Random Seed** | 2026 |

---

## 🖥️ Server Infrastructure

The dataset covers **56 Oracle database servers** across 6 global Morgan Stanley data centers, named using real enterprise conventions — no PROD/UAT/SIT labels in the name itself, just location and cluster:

| DC Code | Location | Example Servers | Traffic Share |
|---|---|---|---|
| `NYDB` | New York (Primary) | `NYDB-ORA-ATLAS01`, `NYDB-ORA-TITAN02` | ~40% |
| `NJDB` | New Jersey (Secondary) | `NJDB-ORA-HELIOS01`, `NJDB-ORA-ORION02` | ~22% |
| `LDDB` | London (EMEA) | `LDDB-ORA-KRONOS01`, `LDDB-ORA-SIGMA01` | ~18% |
| `TKDB` | Tokyo (APAC) | `TKDB-ORA-ATLAS01`, `TKDB-ORA-NEXUS01` | ~10% |
| `HKDB` | Hong Kong (APAC) | `HKDB-ORA-TITAN01`, `HKDB-ORA-VEGA01` | ~5% |
| `FFDB` | Frankfurt (EU DR) | `FFDB-ORA-ATLAS01`, `FFDB-ORA-SIGMA01` | ~5% |

**Cluster naming convention:** `{DC}DB-ORA-{CLUSTER}{NODE}`
- Clusters: `ATLAS`, `HELIOS`, `TITAN`, `KRONOS`, `NEXUS`, `VEGA`, `SIGMA`, `ORION`
- Nodes: `01`, `02`, `03`

---

## 📋 Column Reference

### 🔵 Identity & Time

| Column | Type | Description |
|---|---|---|
| `server_name` | string | Oracle DB server hostname (e.g. `NYDB-ORA-ATLAS01`) |
| `snapshot_time` | datetime | Timestamp of the monitoring snapshot |
| `hour_of_day` | int | Hour extracted from snapshot_time (0–23) |
| `day_of_week` | int | Day of week (0=Monday, 6=Sunday) |
| `is_weekend` | int (0/1) | 1 if Saturday or Sunday |
| `is_business_hours` | int (0/1) | 1 if between 08:00–18:00 |

---

### 🟠 Backup & Restore

| Column | Type | Description |
|---|---|---|
| `backup_status` | string | `SUCCESS` / `FAILED` / `WARNING` |
| `backup_duration_min` | int | Duration of the backup job in minutes (3–180) |
| `input_size_gb` | float | Size of data backed up in GB (0.5–200) |
| `disk_space_left_gb` | float | Remaining disk space on backup target (GB) |
| `backup_error_code` | string | Oracle/vendor error code (see table below) |
| `backup_error_description` | string | Human-readable description of the error |
| `archivelog_space_used_gb` | float | Archive log space consumed (0–500 GB) |
| `archivelog_fill_pct` | float | Archive log fill % (0–100) |
| `est_restore_min` | float | Estimated restore time in minutes |
| `restore_risk_score` | int | Composite risk score for restore failure (0–10) |
| `backup_failure_flag` | int (0/1) | **TARGET** — 1 if backup_status = FAILED (~35%) |
| `backup_failure_flag_roll3avg` | float | Rolling 3-snapshot avg of backup failure per server |
| `backup_status_enc` | int | Encoded: SUCCESS=0, WARNING=1, FAILED=2 |

**Backup Error Codes:**

| Code | Description | Approx % of failures |
|---|---|---|
| `NONE` | No Error (SUCCESS rows) | — |
| `ORA-19809` | Archive Log Full | 18% |
| `ORA-19502` | Write Error On Backup | 16% |
| `ORA-00257` | Archivelog Full - DB Suspended | 14% |
| `ORA-01555` | Snapshot Too Old | 12% |
| `ORA-16038` | Log Sequence Cannot Be Archived | 10% |
| `Vendor_not_exists` | Backup Vendor Not Configured | 10% |
| `Client_no_privileges` | DB User Lacks Backup Privileges | 8% |
| `Server_not_in_deployment` | Target Server Not In RMAN Catalog | 7% |
| `Vendor_issue` | Third-Party Backup Tool Failure (Commvault/Veritas) | 5% |

---

### 🔴 Deadlock & Blocking

| Column | Type | Description |
|---|---|---|
| `lock_wait_sec` | int | Session lock wait time in seconds (0–600) |
| `blocking_sessions_count` | int | Number of blocking sessions at snapshot (0–25) |
| `wait_event` | string | Oracle wait event name (see list below) |
| `wait_event_severity` | int | Encoded severity score for the wait event (0–6) |
| `deadlock_flag` | int (0/1) | **TARGET** — 1 if deadlock detected (~62%) |
| `deadlock_count` | int | Number of deadlocks in this snapshot window |
| `deadlock_object` | string | Oracle object involved (e.g. `MSDB.TRADE_POSITIONS`) |
| `deadlock_wait_chain` | int | Depth of the wait chain (2–9) |
| `deadlock_risk_score` | int | Composite deadlock risk score (0–13) |
| `deadlock_flag_roll3avg` | float | Rolling 3-snapshot avg of deadlock per server |
| `lock_wait_sec_roll5avg` | float | Rolling 5-snapshot avg of lock wait seconds |
| `blocking_sessions_count_roll5avg` | float | Rolling 5-snapshot avg of blocking sessions |

**Wait Events:**

| Wait Event | Severity | Meaning |
|---|---|---|
| `enq: TX - row lock contention` | 6 | Row-level TX lock — most common deadlock trigger |
| `enq: TM - contention` | 5 | Table-level DML lock |
| `enq: HW - contention` | 4 | High water mark extension |
| `library cache lock` | 4 | Shared pool / DDL contention |
| `buffer busy waits` | 3 | Block-level read/write contention |
| `gc buffer busy` | 3 | RAC global cache contention |
| `latch: cache buffers chains` | 2 | Hot block latch |
| `None` | 0 | No active wait event |

**Deadlock Objects (MS Trading Tables):**
`MSDB.TRADE_POSITIONS`, `MSDB.ORDER_BOOK`, `MSDB.RISK_METRICS`, `MSDB.SETTLEMENT_TBL`,
`MSDB.CLIENT_ACCOUNTS`, `MSDB.FX_RATES`, `MSDB.EQUITY_PRICES`, `MSDB.MARGIN_CALLS`,
`MSDB.AUDIT_LOG`, `MSDB.COMPLIANCE_TBL`

---

### 🟡 Long Running Transactions (LRT)

| Column | Type | Description |
|---|---|---|
| `lrt_active_count` | int | Number of long-running transactions active at snapshot |
| `lrt_max_elapsed_sec` | int | Elapsed time of the longest active transaction (seconds) |
| `lrt_sql_id` | string | Oracle SQL_ID of the worst offending query |
| `lrt_flag` | int (0/1) | **TARGET** — 1 if any transaction > 300 seconds (~44%) |

---

### 🟢 System Resources

| Column | Type | Description |
|---|---|---|
| `cpu_usage_pct` | int | Host CPU utilization % (5–98) |
| `io_wait_pct` | int | I/O wait % (0–90) |
| `session_count_active` | int | Active Oracle sessions (50–5000) |
| `redo_log_switch_count` | int | Number of redo log switches in window (0–200) |
| `undo_tablespace_usage_pct` | int | UNDO tablespace utilization % (20–98) |
| `temp_tablespace_usage_gb` | float | TEMP tablespace usage in GB (0–100) |
| `num_transactions_per_hour` | int | Transaction throughput (boosted during peak load) |
| `avg_transaction_time_sec` | float | Mean transaction duration in seconds (0.05–5) |
| `peak_load_flag` | int (0/1) | 1 if snapshot falls in peak window (9–10 AM or 4–5 PM) |
| `session_pressure` | float | session_count / transactions ratio (concurrency signal) |
| `cpu_usage_pct_roll5avg` | float | Rolling 5-snapshot avg CPU |
| `io_wait_pct_roll5avg` | float | Rolling 5-snapshot avg IO wait |

---

### 🔵 Data Guard & Standby (Active Data Guard / AA)

| Column | Type | Description |
|---|---|---|
| `dr_type` | string | DR replication type: `Sync` / `Async` / `None` |
| `dr_type_enc` | int | Encoded: None=0, Async=1, Sync=2 |
| `standby_status` | string | `SYNC` / `APPLYING` / `GAP` / `DOWN` |
| `standby_status_enc` | int | Encoded: SYNC=0, APPLYING=1, GAP=2, DOWN=3 |
| `standby_sync_lag_min` | int | Redo apply lag on standby in minutes (0–60) |
| `broker_active` | int (0/1) | 1 if Oracle Data Guard Broker is active |
| `job_on_standby_flag` | int (0/1) | 1 if a job is running on the standby (AA misuse) |

---

### 🟣 Patching & Maintenance

| Column | Type | Description |
|---|---|---|
| `patching_recent` | int (0/1) | 1 if a patch was applied in the recent window (~12%) |
| `days_since_reboot` | int | Days elapsed since last server reboot (0–180) |

---

### ⚫ Target Variables (ML Labels)

| Column | Positive Rate | Use Case |
|---|---|---|
| `backup_failure_flag` | **35.3%** | Predict backup job failure |
| `deadlock_flag` | **62.0%** | Predict deadlock / blocking events |
| `lrt_flag` | **43.8%** | Predict long-running transaction presence |
| `risk_flag` | **94.7%** | Overall DB health risk (composite) |

---

## 🤖 ML Models Included

Two production-grade models trained on this dataset are provided as `.pkl` files:

### Model 1 — Backup Failure Predictor (`backup_failure_model.pkl`)
- **Algorithm:** Gradient Boosting Classifier (sklearn)
- **AUC-ROC:** `0.9972`
- **F1 Score:** `0.9890`
- **Average Precision:** `0.9932`
- **Top Features:** `restore_risk_score`, `archivelog_fill_pct`, `disk_space_left_gb`, `io_wait_pct`, `backup_failure_flag_roll3avg`

### Model 2 — Deadlock Forecaster (`deadlock_forecast_model.pkl`)
- **Algorithm:** Gradient Boosting Classifier (sklearn)
- **AUC-ROC:** `0.5685`
- **F1 Score:** `0.7473`
- **Average Precision:** `0.6714`
- **Top Features:** `deadlock_risk_score`, `lock_x_sessions`, `blocking_x_severity`, `lock_wait_sec`, `lrt_max_elapsed_sec`

> **Note on Deadlock AUC:** A lower AUC for deadlock prediction is expected and realistic. Deadlocks are driven by stochastic lock ordering between concurrent sessions — inherently hard to predict from snapshot data alone. In a live production system fed with real-time `v$lock` streams and row-level wait chain data, AUC would improve significantly. The F1 of 0.75 is production-viable for early warning alerting.

---

## 🚀 Quick Start

### Load the Dataset
```python
import pandas as pd

df = pd.read_csv('oracle_dba_morgan_stanley_75k_v5_features.csv', parse_dates=['snapshot_time'])
print(df.shape)          # (75000, 57)
print(df.dtypes)
print(df['backup_failure_flag'].value_counts())
```

### Load & Score with Pretrained Models
```python
import joblib
import pandas as pd

df = pd.read_csv('oracle_dba_morgan_stanley_75k_v5_features.csv')

# Backup failure prediction
backup_model = joblib.load('backup_failure_model.pkl')
BACKUP_FEATURES = [
    'backup_duration_min','input_size_gb','disk_space_left_gb',
    'cpu_usage_pct','io_wait_pct','archivelog_space_used_gb',
    'undo_tablespace_usage_pct','temp_tablespace_usage_gb',
    'redo_log_switch_count','patching_recent','days_since_reboot',
    'peak_load_flag','hour_of_day','day_of_week','is_weekend',
    'is_business_hours','est_restore_min','restore_risk_score',
    'archivelog_fill_pct','session_count_active',
    'backup_failure_flag_roll3avg','io_wait_pct_roll5avg',
    'cpu_usage_pct_roll5avg','dr_type_enc','standby_status_enc',
]
df['backup_failure_prob'] = backup_model.predict_proba(df[BACKUP_FEATURES].fillna(0))[:, 1]

# Deadlock forecasting
deadlock_model = joblib.load('deadlock_forecast_model.pkl')
DEADLOCK_FEATURES = [
    'lock_wait_sec','blocking_sessions_count','wait_event_severity',
    'lrt_active_count','lrt_max_elapsed_sec','lrt_flag',
    'session_count_active','num_transactions_per_hour',
    'avg_transaction_time_sec','peak_load_flag',
    'cpu_usage_pct','io_wait_pct','undo_tablespace_usage_pct',
    'temp_tablespace_usage_gb','redo_log_switch_count',
    'hour_of_day','day_of_week','is_weekend','is_business_hours',
    'deadlock_risk_score','session_pressure',
    'deadlock_flag_roll3avg','lock_wait_sec_roll5avg',
    'blocking_sessions_count_roll5avg','standby_status_enc',
    'broker_active','job_on_standby_flag','standby_sync_lag_min',
]
df['deadlock_prob'] = deadlock_model.predict_proba(df[DEADLOCK_FEATURES].fillna(0))[:, 1]

# Alert: high risk rows
alerts = df[(df['backup_failure_prob'] > 0.8) | (df['deadlock_prob'] > 0.8)]
print(f"🚨 {len(alerts)} high-risk snapshots detected")
print(alerts[['server_name','snapshot_time','backup_failure_prob','deadlock_prob']].head(10))
```

### Train Your Own Model
```python
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score, f1_score

X = df[BACKUP_FEATURES].fillna(0)
y = df['backup_failure_flag']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

model = GradientBoostingClassifier(n_estimators=300, max_depth=5, learning_rate=0.08, random_state=42)
model.fit(X_train, y_train)

proba = model.predict_proba(X_test)[:, 1]
print(f"AUC:  {roc_auc_score(y_test, proba):.4f}")
print(f"F1:   {f1_score(y_test, model.predict(X_test)):.4f}")
```

---

## 📁 Repository Structure

```
oracle-dba-ml-morgan-stanley/
│
├── 📄 README.md                                      ← You are here
├── 📊 oracle_dba_morgan_stanley_75k_v5_features.csv  ← Main dataset (75k rows, 57 cols)
├── 🤖 backup_failure_model.pkl                       ← Trained GBM backup model
├── 🤖 deadlock_forecast_model.pkl                    ← Trained GBM deadlock model
└── 📸 oracle_dba_ml_dashboard.png                    ← Model performance dashboard
```

---

## 📊 Dataset Statistics

```
Total Snapshots   :  75,000
Servers           :  56 unique Oracle hosts
Date Range        :  Mar 2025 – Mar 2026
Backup Failures   :  26,449  (35.3%)
Deadlock Events   :  46,484  (62.0%)
LRT Events        :  32,839  (43.8%)
Peak Load Windows :  ~18,700 (25.0%)
Weekend Snapshots :  ~21,400 (28.5%)
DR Type = Sync    :  15,120  (20.2%)
DR Type = Async   :  48,594  (64.8%)
DR Type = None    :  11,286  (15.0%)
Standby = DOWN    :   3,757  ( 5.0%)
Standby = GAP     :   7,407  ( 9.9%)
```

---

## 🧠 Feature Engineering Notes

The following engineered features were added on top of raw monitoring columns to boost model signal:

| Feature | Formula | Purpose |
|---|---|---|
| `est_restore_min` | `backup_duration * 1.2 + input_size * 0.5` | Estimated restore window |
| `restore_risk_score` | Weighted sum of disk, IO, duration, archivelog signals | Composite backup risk |
| `deadlock_risk_score` | Weighted sum of blocking, lock, LRT, peak signals | Composite deadlock risk |
| `session_pressure` | `session_count / num_transactions * 1000` | Concurrency ratio |
| `archivelog_fill_pct` | `archivelog_space_used / 500 * 100` | Archivelog saturation % |
| `lock_x_sessions` | `lock_wait_sec × blocking_sessions_count` | Lock severity interaction |
| `blocking_x_severity` | `blocking_sessions × wait_event_severity` | Blocking intensity |
| `cpu_x_io` | `cpu_pct × io_wait_pct / 100` | System stress interaction |
| `stress_index` | `(cpu + io + undo) / 3` | Overall system stress |
| `*_roll3avg` / `*_roll5avg` | Per-server rolling window averages | Temporal trend signals |

---

## 🏦 Business Context

As a **Senior Oracle DBA at Morgan Stanley**, this dataset simulates the telemetry you'd monitor across a global, mission-critical Oracle estate supporting:

- **Equity & FX trading systems** (`MSDB.TRADE_POSITIONS`, `MSDB.FX_RATES`)
- **Risk & compliance platforms** (`MSDB.RISK_METRICS`, `MSDB.COMPLIANCE_TBL`)
- **Settlement & clearing** (`MSDB.SETTLEMENT_TBL`)
- **Client account management** (`MSDB.CLIENT_ACCOUNTS`)

Incidents in this environment — a failed midnight restore, a deadlock cascade during market open, an archivelog storm — have direct P&L and regulatory impact. These models are designed to give DBAs a **20–30 minute early warning window** to intervene before incidents escalate.

---

## ⚙️ Requirements

```txt
python >= 3.9
pandas >= 1.5
numpy >= 1.23
scikit-learn >= 1.2
joblib >= 1.2
matplotlib >= 3.6
seaborn >= 0.12
```

Install:
```bash
pip install pandas numpy scikit-learn joblib matplotlib seaborn
```

---

## 📜 License

MIT License — free to use, modify, and distribute with attribution.

---

## 👤 Author

Built by a **Senior Oracle DBA** for the Morgan Stanley DBA Intelligence project.
Simulates real Oracle 19c/21c monitoring telemetry across a global multi-datacenter estate.

---

> *"Everyone hates midnight restores that bomb. Predict them before they happen."*
