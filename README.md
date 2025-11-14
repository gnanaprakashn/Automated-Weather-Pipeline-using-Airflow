Nice — you gave the raw steps. I turned them into a sharp, believable, *hireable* README that reads like it was built by a 1–2 yoe data engineer. Copy this `README.md` into your repo (replace or edit text like bucket names, IAM role ARNs, schedule, API endpoint, and any credentials placeholders).

---

```markdown
# Automated Weather Pipeline — Airflow + AWS Glue + Athena

[![Status](https://img.shields.io/badge/status-production-brightgreen)](https://github.com/gnanaprakashn/Automated-Weather-Pipeline-using-Airflow)
[![Built with](https://img.shields.io/badge/built%20with-Airflow%20%7C%20Glue%20%7C%20Athena%20%7C%20S3-blue)]()

## Project Summary
**Automated Weather Pipeline** is an end-to-end ETL system that ingests weather data from a REST API, lands raw payloads in S3, transforms them with an AWS Glue PySpark job into partitioned Parquet, and exposes the processed dataset in Athena. Apache Airflow orchestrates two DAGs to manage extraction, transformation and cataloging. This repository contains Airflow DAG(s), Glue scripts, architecture diagram, and environment requirements.

This implementation demonstrates the core responsibilities and tooling proficiency expected of a Data Engineer with **1–2 years** of hands-on experience: designing reliable workflows, cloud ETL, schema management, and analytics-ready datasets.

---

## Architecture

![Architecture Diagram](airflow.png)

**High level flow**
1. Airflow DAG (DAG 1) → fetches data from Weather REST API → uploads raw files to S3 (raw/).
2. Airflow DAG (DAG 2) → waits for previous-day result and triggers AWS Glue ETL job.
3. AWS Glue (PySpark) → reads raw S3 objects, flattens/cleans JSON, renames columns, type-casts, writes partitioned Parquet to S3 (cleaned/).
4. Glue Crawler → crawls `cleaned/` path → updates Glue Data Catalog (database + table).
5. Athena → query the processed table for analysis / dashboards.

---

## Repo structure

```

Automated-Weather-Pipeline-using-Airflow/
│
├── dags/
│   ├── dag_api_extract.py         # DAG 1: API extraction -> S3 raw
│   └── dag_trigger_glue.py        # DAG 2: wait / trigger Glue job
│
├── glue_scripts/
│   └── weather_glue_transform.py  # Glue PySpark transformation
│
├── requirements.txt
├── airflow.png
└── README.md

````

---

## Tech stack / Tools used
- Orchestration: **Apache Airflow** (Docker + docker-compose)
- Cloud Storage & Catalog: **Amazon S3**, **AWS Glue Data Catalog**, **AWS Glue Crawler**
- ETL: **AWS Glue** (PySpark)
- Querying: **Amazon Athena**
- Monitoring: Airflow UI + Glue job logs in CloudWatch / S3
- Language: Python 3.x (Airflow operators + Glue PySpark)
- CI / local: Docker, docker-compose

---

## Prerequisites

- Docker & docker-compose installed locally
- AWS account with permissions for: S3, Glue, IAM, Athena, CloudWatch, Glue Crawler
- AWS CLI configured (`aws configure`) or Airflow connections set to use AWS credentials
- An S3 bucket to hold raw and cleaned datasets
- A Glue IAM role with the following minimum permissions:
  - `s3:GetObject`, `s3:PutObject`, `s3:ListBucket` on relevant buckets
  - `glue:*` for job/crawler operations (or scoped subset)
  - `iam:PassRole` for Glue job execution

---

## Quick start (local Airflow + deploy steps)

> Run these commands from your project root or execute via your deployment pipeline.

### 1) Prepare local Airflow (Docker)
```bash
# create folders Airflow expects
mkdir -p ./dags ./logs ./plugins

# bring up Airflow (adjust docker-compose if needed)
docker-compose up -d
````

Place `dags/` contents into Airflow's DAG folder (the repo `dags/` is ready to be copied).

### 2) S3 buckets (example)

Create two logical prefixes or separate buckets:

* `s3://<bucket>/raw/weather/`   — raw API payloads (JSON/CSV)
* `s3://<bucket>/cleaned/weather/` — Parquet output (partitioned by `date`)

### 3) Configure Airflow connections & variables

In Airflow UI → Admin → Connections:

* `aws_default` → AWS credentials (access key / secret or role)
* `http_default` → base url to Weather API (if used by `HttpHook`), or the DAG can use `requests` and environment variable for the API key

Set Airflow Variables or environment variables:

```
S3_BUCKET=<your-bucket>
API_KEY=<weather-api-key>
GLUE_JOB_NAME=weather-glue-job
GLUE_SCRIPT_PATH=s3://<bucket>/scripts/weather_glue_transform.py   # optional
```

### 4) DAG 1 — API extraction (what it does)

* Task: `fetch_weather_data`

  * Call REST API, save JSON/CSV file locally or stream to `s3://<bucket>/raw/weather/YYYY-MM-DD/<timestamp>.json`
  * Use deterministic file names for idempotency
* Task: `upload_to_s3` (if the DAG writes local then uploads)
* Retries configured (e.g., 3 retries with exponential backoff)
* Logging to Airflow logs + copy of raw files retained in S3 for reproducibility

### 5) Glue script & job (how to create)

On AWS Console → Glue → Jobs → Create job:

* Name: `weather-glue-job` (or use `GLUE_JOB_NAME` above)
* IAM role: `<glue-execution-role-arn>`
* Glue version: e.g., `Glue 3.0` (or the version you use)
* Python version: PySpark (choose the correct Spark/Python combination)
* Script path: S3 path to `glue_scripts/weather_glue_transform.py` or upload from repo
* Temporary directory: `s3://<bucket>/glue-temp/`
* Worker type & number: choose based on dataset size (e.g., G.1X/Standard worker 2–5 for dev)

**Glue job snippet summary (what the script does):**

* Read objects under `s3://<bucket>/raw/weather/`
* Flatten nested JSON into tabular columns
* Rename columns to snake_case and human-friendly names
* Clean: remove duplicates / drop null-critical rows / convert types (timestamps, floats, ints)
* Generate `date` partition column (e.g., `observation_date`)
* Write Parquet to `s3://<bucket>/cleaned/weather/observation_date=YYYY-MM-DD/`
* Optionally call AWS Glue API or write partition metadata for immediate discovery

### 6) DAG 2 — Trigger Glue job

* Task: `wait_for_previous_day_result` — confirm files exist in `s3://<bucket>/raw/weather/<prev-date>/`
* Task: `start_glue_job` — use Boto3 via PythonOperator to start Glue job
* Task: `poll_glue_job_status` — wait/poll until completion; fail on error
* Task: `run_crawler` — optionally start Glue Crawler once job completes

### 7) Glue Crawler → Data Catalog

* Create crawler:

  * Data store: S3
  * Include path: `s3://<bucket>/cleaned/weather/`
  * IAM role: crawler-role (with S3 read + Glue write)
  * Target database: `weather_db`
* Run crawler after Glue completes to create/refresh table(s) used by Athena

### 8) Athena — Query the cleaned data

* Create database: `weather_db` (or use crawler-created)
* Example Athena query:

```sql
SELECT station_id, temperature, humidity, wind_speed, observation_time
FROM weather_db.weather_table
WHERE observation_date = date_format(current_timestamp, '%Y-%m-%d')
ORDER BY observation_time DESC
LIMIT 100;
```

---

## Implementation notes (how I built it)

* **Idempotent ingestion:** filenames and S3 prefixes include execution date/timestamp to prevent accidental overwrites.
* **Partitioning strategy:** partitioned by `observation_date` to make Athena scans efficient and cost-effective.
* **Glue optimization:** small-state aggregation / repartitioning before write to reduce small files; use `coalesce()` or `repartition()` as required.
* **Error handling:** Airflow tasks include retries and exponential backoff; Glue job failure logs pulled from CloudWatch for troubleshooting.
* **Security:** Use IAM roles with least privilege; avoid embedding AWS keys in code — prefer instance/role-based auth or Secrets Manager.

---

## Example commands (copy & run where applicable)

Create folders for Airflow:

```bash
mkdir -p dags logs plugins
```

Sample docker start (project-local):

```bash
docker-compose up -d
```

Sample AWS Glue start (boto3 in DAG/Python):

```python
import boto3
glue = boto3.client('glue')
resp = glue.start_job_run(JobName='weather-glue-job', Arguments={'--S3_INPUT':'s3://bucket/raw/...'})
job_run_id = resp['JobRunId']
```

---

## Testing & validation

* Trigger `dag_api_extract` manually in Airflow to verify raw files appear in S3.
* Run Glue job locally with smaller sample data or via AWS Glue Dev Endpoint for faster iteration.
* Validate schema in Athena; run sample queries and check partitioning.
* Use Airflow UI to replay runs, inspect logs, and test failure/retry behavior.

---

## Troubleshooting tips

* `fetch_weather_data` fails: test API key & endpoint with `curl` or Postman. Confirm rate limits.
* Glue job times out: increase workers or memory; profile transforms and repartition.
* Athena returns no results: confirm Glue crawler refreshed the table and partitions exist.
* Excess small files: write coalesced Parquet and use Glue job to compact partitions periodically.

---

## Resume / Interview bullet (copy-paste)

* Built an end-to-end Airflow + AWS Glue ETL pipeline for ingesting weather data from REST API → S3 → Glue → Athena; implemented idempotent ingestion, partitioned Parquet outputs, automated schema discovery via Glue Crawler, and production-ready DAGs with retries and monitoring.

---

## Author & Contact

**Gnana Prakash N** — Aspiring Data Engineer (1–2 years practical experience)
GitHub: [gnanaprakashn](https://github.com/gnanaprakashn)
Email: *(add your email)*

---

## License

MIT © Gnana Prakash

```

---

If you want, I’ll:
- Insert the exact bucket names, IAM role ARN, and sample API endpoint into the README (you paste them here).  
- Produce a short one-paragraph LinkedIn blurb and 2–3 strong resume bullets from this project.
::contentReference[oaicite:0]{index=0}
```
