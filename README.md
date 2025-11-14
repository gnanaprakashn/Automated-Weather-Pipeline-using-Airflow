

# **README.md — Automated Weather Pipeline (Airflow + AWS Glue)**

````markdown
# Automated Weather Pipeline (Airflow + AWS Glue)

This project is a real data engineering workflow I built to automate weather data ingestion, cleaning, and analytics using **Airflow**, **AWS Glue**, **S3**, **Glue Crawler**, and **Athena**.  
The pipeline runs in two stages:  
**(1) fetch & store raw data**,  
**(2) clean & prepare data for analytics**.

---

## 🔥 What This Pipeline Does (In Simple Words)

1. **Airflow DAG #1** calls a weather API → stores raw JSON in S3.  
2. **Airflow DAG #2** waits for that raw file → triggers a Glue ETL job.  
3. **Glue PySpark script** cleans/normalizes the JSON → writes partitioned Parquet to S3.  
4. **Glue Crawler** picks up new files → updates the Data Catalog.  
5. **Athena** queries the final dataset in seconds.

This is the exact workflow used in most production ETL setups.

---

## 📌 Why I Built It

I wanted a project that shows **actual skills** a data engineer uses daily:

- scheduling / dependencies (Airflow)  
- API ingestion  
- S3 raw layer → cleaned layer  
- distributed Spark transformation (Glue)  
- schema automation (Crawler)  
- SQL analytics (Athena)

This pipeline covers the full lifecycle from ingestion → transformation → analytics.

---

## 🧩 How I Designed the Pipeline (My Steps)

### **1. Airflow setup (Docker)**
- Installed Airflow using Docker  
- Created `dags/`, `logs/`, `plugins/` folders  
- Launched Airflow:
  ```bash
  docker-compose up -d
````

### **2. Raw & Clean Buckets**

Created two prefixes:

* `s3://bucket/raw/`
* `s3://bucket/cleaned/`

### **3. DAG 1 — API Extraction**

* Called the weather REST API
* Saved response locally
* Uploaded file to `raw/DATE/` folder in S3
* Added retries for API failures
* Logged file name + S3 key for traceability

### **4. Glue ETL Script**

* Read raw JSON files from S3
* Flattened nested attributes
* Renamed columns
* Cleaned inconsistent rows
* Converted timestamp formats
* Wrote **partitioned Parquet** to:

  ```
  s3://bucket/cleaned/date=YYYY-MM-DD/
  ```

### **5. DAG 2 — Trigger Glue**

* Waited for yesterday’s raw data
* Started Glue job
* Polled job status
* Optional: triggered crawler after success

### **6. Glue Crawler**

* Pointed to `cleaned/` folder
* Created Athena table automatically
* Managed schema updates (new columns, partitions)

### **7. Athena**

Queried everything using SQL.

Example:

```sql
SELECT temperature, humidity, observation_time 
FROM weather_db.weather_data
ORDER BY observation_time DESC;
```

---

## 🗂️ Repository Structure

```
/dags
    ├── dag_api_extract.py
    ├── dag_trigger_glue.py

/glue_scripts
    └── weather_glue_transform.py

requirements.txt
airflow.png
README.md
```

---

## 🧠 Key Things I Learned / Implemented

* **Idempotent ingestion** → each DAG run creates a unique raw file
* **Partition strategy** → optimized Athena scan cost
* **PySpark transformations** → flattened JSON cleanly
* **Airflow → Glue integration** using Boto3
* **Crawler automation** → no manual schema updates
* **Two-DAG design** → clear separation of ingestion vs transformation

---

## 🎯 What This Pipeline Proves About My Skills

* I can design cloud ETL with real orchestration.
* I know how to handle raw → processed → query layers.
* I can work with AWS Glue, S3, Athena, and Airflow in a practical setup.
* I understand schema evolution, reliability, retries, and monitoring.
* I can build pipelines that follow real-world patterns.

---

## 📎 Architecture Diagram

![Pipeline Architecture](airflow.png)

---

## 👤 Author

**Gnana Prakash N**
Data Engineering Enthusiast
GitHub: gnanaprakashn

```

---

If you want the README even **shorter**, **more aggressive**, or **more senior-looking**, tell me the style:

- 🔵 **Short & Clean**  
- 🔴 **Bold & Confident**  
- 🟢 **More technical**  
- 🟣 **More storytelling**  
- 🟠 **Resume-oriented**  

Pick a style, and I’ll rewrite the whole README again.
```
