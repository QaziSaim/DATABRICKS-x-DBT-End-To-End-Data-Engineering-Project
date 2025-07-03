# DATABRICKS-x-DBT-End-To-End-Data-Engineering-Project
!['Project WorkFlow'](https://github.com/QaziSaim/DATABRICKS-x-DBT-End-To-End-Data-Engineering-Project/blob/main/Screenshot%202025-06-29%20212439.png)

Sure! Here's the **updated and enhanced `README.md`** for your **Databricks End-to-End Flights Project**, now including:

* Use of **serverless compute**
* Runtime duration (\~5 minutes)
* End-to-end automation details

---

# ✈️ Databricks End-to-End Flights Project – Bronze Layer (Medallion Architecture)

## 📌 Project Overview

This project implements the **Bronze Layer** of a **Medallion Architecture** (Bronze → Silver → Gold) using **Databricks**. It focuses on scalable, automated ingestion of raw flight data into Delta Lake, setting the foundation for clean and reliable data engineering pipelines.

> ✅ The entire pipeline runs on **serverless compute** and completes in **\~5 minutes** per execution.

---

## 🌐 Technologies & Features

* 🔁 **Delta Lake + Auto Loader** for incremental ingestion
* ⚙️ **Databricks Jobs & Tasks** for orchestration
* 💻 **Serverless compute** for cost-effective performance
* 📁 **Volumes + Workspaces** for structured data management
* ✅ **Dynamic task parameterization**
* 🔄 **Schema evolution mode = rescue** (handles changing schemas)
* 🧪 Supports re-ingestion when new data is added

---

## 🗂️ Data Lake Layout

**Volumes:**

```
/Volumes/
├── raw/
│   └── rawvolume/
│       └── rawdata/
│           ├── airports/
│           │   └── airports.csv
│           ├── bookings/
│           ├── customers/
│           └── flights/
├── bronze/
    └── bronzevolume/
        ├── airports/
        ├── bookings/
        ├── customers/
        └── flights/
```

**Databases:**

* `rawdb`
* `bronzedb`
* `silverdb`
* `golddb`

---

## 📓 Notebooks Summary

### 1. `1_setup`

* Sets up required databases and volume paths.
* Initializes workspace structure.

---

### 2. `2_bronze_layer`

* Ingests raw data using **Auto Loader**:

```python
dbutils.widgets.text("src", "")
src_value = dbutils.widgets.get("src")

df = spark.readStream.format("cloudFiles") \
    .option('cloudFiles.format', 'csv') \
    .option("cloudFiles.schemaLocation", f"/Volumes/workspace/bronze/bronzevolume/{src_value}/checkpoint") \
    .option("cloudFiles.schemaEvolutionMode", "rescue") \
    .load(f"/Volumes/workspace/raw/rawvolume/rawdata/{src_value}/")

df.writeStream.format("delta") \
    .outputMode("append") \
    .trigger(once=True) \
    .option("checkpointLocation", f"/Volumes/workspace/bronze/bronzevolume/{src_value}/checkpoint") \
    .option("path", f"/Volumes/workspace/bronze/bronzevolume/{src_value}/data") \
    .start()
```

---

### 3. `3_src_parameters`

* Dynamically generates source list for looping:

```python
src_array = ['bookings', 'airports', 'customers', 'flights']
src_array = [{'src': n} for n in src_array]
dbutils.jobs.taskValues.set(key="output_key", value=src_array)
```

---

## 🔁 Job Pipeline: `bronzeingestion`

### 🎯 Structure

| Task Name              | Function                                   |
| ---------------------- | ------------------------------------------ |
| `src_parameters`       | Generates list of sources (`src_array`)    |
| `IncrementalIngestion` | Iterates over sources and ingests each one |

### 🔄 Task Flow

* `src_parameters` → loops `IncrementalIngestion` over:

  * `airports`
  * `bookings`
  * `customers`
  * `flights`

### ✅ Re-ingestion Support

* Adding new CSVs to `/rawvolume/rawdata/{src}` and re-running the job ingests only new data (due to **Auto Loader checkpointing**).

---

## ⚡ Performance

* **Runtime:** \~5 minutes for full bronze ingestion
* **Cluster Type:** **Databricks Serverless** (no manual cluster management)
* **Trigger Type:** `once` (batch-style streaming ingestion)

---

## 🔜 Next Steps

* [ ] Implement **Silver Layer** for data cleaning and filtering
* [ ] Add **Gold Layer** for aggregations and BI readiness
* [ ] Integrate with **Power BI** / **Excel** / **Tableau**
* [ ] Add **unit tests** for pipeline components

---

Let me know if you’d like this saved as a `.md` file or pushed directly to a GitHub repo structure.

