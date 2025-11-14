📘 README.md — Automated Weather Pipeline using Airflow, AWS Glue & Athena
# Automated Weather Pipeline using Airflow, AWS Glue & Athena

This project is a complete end-to-end data engineering pipeline that ingests weather data from a REST API, stores raw files in S3, processes them using AWS Glue (PySpark), catalogs them using Glue Crawler, and makes the final dataset queryable via Amazon Athena. Apache Airflow orchestrates the entire workflow using two DAGs.

This architecture demonstrates production-style ETL capabilities expected from a Data Engineer with **1–2 years of experience**.

---

## 🚀 Architecture Overview

![Architecture](airflow.png)

### **High-Level Workflow**
1. **Airflow (DAG 1)** makes API calls and uploads raw weather data to S3.
2. **Airflow (DAG 2)** waits for the previous run and triggers AWS Glue.
3. **AWS Glue ETL** processes & cleans the data and writes partitioned Parquet files.
4. **AWS Glue Crawler** updates the schema automatically in the Data Catalog.
5. **Amazon Athena** allows running analytical queries over optimized Parquet files.

---

## 🧱 Project Structure



Automated-Weather-Pipeline-using-Airflow/
│
├── dags/
│ ├── dag_api_extract.py # DAG 1 – API extraction → S3 raw
│ └── dag_trigger_glue.py # DAG 2 – Trigger Glue job
│
├── glue_scripts/
│ └── weather_glue_transform.py # PySpark Glue ETL script
│
├── requirements.txt
├── airflow.png # Architecture diagram
└── README.md


---

## 🛠️ Tech Stack

### **Orchestration**
- Apache Airflow (Docker + docker-compose)

### **AWS Services**
- Amazon S3 (Raw + Cleaned zones)
- AWS Glue ETL (PySpark)
- AWS Glue Crawler
- AWS Glue Data Catalog
- Amazon Athena
- IAM Roles
- CloudWatch (Monitoring)

### **Languages & Libraries**
- Python 3.x
- Boto3
- PySpark
- Requests / JSON processing

---

## ⚙️ Setup & Execution Steps

### **1. Airflow Setup using Docker**
```bash
mkdir -p dags logs plugins
docker-compose up -d


Copy the dags/ folder from this repo into your Airflow container's DAG directory.

2. Create S3 Buckets

Use two prefixes (or two buckets):

s3://<bucket>/raw/weather/

s3://<bucket>/cleaned/weather/

Raw → from API
Cleaned → Glue ETL output

3. DAG 1: API Extraction (Airflow)

Task Flow:

fetch_weather_data: Call REST API and download weather JSON/CSV.

Upload to S3 → s3://bucket/raw/weather/<date>/file.json

Retry enabled for API failures.

This ensures raw data is ingested daily/hourly.

4. Create AWS Glue Job

Your Glue job:

Name: weather-glue-job

Script: uploaded from glue_scripts/weather_glue_transform.py

Temporary dir: s3://bucket/glue-temp/

IAM role: Glue-execution-role

Glue version: 3.0 or 4.0

Worker type: Standard (for small datasets)

Glue Script Responsibilities:

Read raw JSON/CSV from S3

Flatten nested attributes

Rename & clean column names

Drop null/broken records

Convert timestamps & types

Write partitioned Parquet:

s3://bucket/cleaned/weather/observation_date=YYYY-MM-DD/

5. DAG 2: Trigger AWS Glue ETL

Steps:

Wait for previous day’s extraction results.

Start the Glue job using Boto3.

Poll job status until completion.

Optionally trigger Glue crawler.

This ensures transformation occurs only after raw data is available.

6. Glue Crawler & Athena

Glue Crawler Config:

Source: s3://bucket/cleaned/weather/

Role: Crawler role with S3 read + Glue write

Destination DB: weather_db

Running the crawler creates or updates the table schema.

7. Query Using Athena

Sample Query:

SELECT
    station_id,
    temperature,
    humidity,
    wind_speed,
    observation_time
FROM weather_db.weather_table
WHERE observation_date = current_date
ORDER BY observation_time DESC
LIMIT 50;

🎯 Key Data Engineering Concepts Demonstrated

Scheduled & automated API ingestion

Raw → Processed → Analytics data modeling

Distributed transformations using PySpark

Partition strategy for performance

Schema management with Glue Crawler

Cost-effective querying with Athena

Workflow dependencies & retries in Airflow

Infrastructure-as-code style reproducibility

🧪 Testing the Pipeline
Airflow Local Testing
airflow tasks test dag_api_extract fetch_weather_data 2024-01-01

Glue Local Testing (Sample Data)

Run PySpark locally with sample JSON.

Verify in S3

Raw data under raw/weather/YYYY-MM-DD

Processed Parquet under cleaned/weather/

Verify in Athena

Table should show new partitions after crawler run.

🔥 Resume-Ready Highlights

Use these in job applications:

Built a production-style ETL pipeline using Airflow → S3 → Glue → Athena, converting API data into query-ready partitioned Parquet datasets.

Automated ingestion, transformation, cataloging, and analytics using two Airflow DAGs and AWS Glue PySpark.

Implemented schema evolution, partition management, error handling, and retries for reliable orchestration.

Designed S3-based medallion architecture (raw + cleaned zones) for scalable analytical workflows.

👤 Author

Gnana Prakash N
Aspiring Data Engineer
GitHub: gnanaprakashn

📄 License

MIT © 2025


---

If you want:
✅ A shorter README  
✅ A version for your portfolio website  
✅ Bullet points specifically optimized for HR  
Just tell me.
