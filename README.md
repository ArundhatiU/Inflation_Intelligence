# Inflation Intelligence: History Meets Real-Time

A serverless, event-driven AWS data pipeline that automates ingestion, transformation,
and analysis of 100+ years of U.S. inflation and unemployment data — reducing analysis
time from days to minutes through automated workflows and interactive Tableau dashboards.

---

## Team

Arundhati Ubhad, Muthu Chellappa, Senthilkumaran Ramanathan

---

## Problem Statement

Traditional economic reporting relies on batch-loaded warehouses and static dashboards,
resulting in delayed and inflexible analysis. This limits timely decision-making for
researchers and policymakers.

This project replaces that approach with an event-driven, schema-on-read pipeline that
automatically processes new data the moment it lands in S3 — enabling live ad-hoc
queries and dynamic dashboards instead of scheduled batch reports.

---

## Objective

Build a cloud-based, serverless pipeline that:
- Automatically detects and processes new monthly BLS data drops
- Transforms raw CPI and unemployment CSVs into analysis-ready Parquet files
- Enables instant SQL queries via Athena
- Powers interactive Tableau dashboards for macroeconomic analysis

---

## Tech Stack

| Layer | Tool |
|---|---|
| Cloud Provider | AWS |
| Ingestion | S3 (Raw Zone) |
| Orchestration | AWS Lambda + Glue Workflow |
| Transformation | AWS Glue (PySpark ETL Notebook) |
| Storage | S3 (Processed Zone — Parquet) |
| Cataloging | AWS Glue Crawler + Data Catalog |
| Querying | Amazon Athena |
| Monitoring | Amazon CloudWatch |
| Access Control | AWS IAM |
| Visualization | Tableau |

---

## Datasets

| Dataset | Source | Format |
|---|---|---|
| US Consumer Price Index (CPI) 1913–2023 | [Kaggle](https://www.kaggle.com/datasets/tunguz/us-consumer-price-index-and-inflation-cpi) | CSV |
| US Unemployment Rates by Age | [Kaggle](https://www.kaggle.com/datasets/guillemservera/us-unemployment-rates) | JSON Lines |
| US Unemployment Rates by Sex | Same source | CSV |

---

## Pipeline Architecture

```plaintext
Kaggle CSV/JSON Files
        │
        ▼
   S3 Raw Zone  (s3://cloudproblem-project-b1/raw-data)
        │
        ▼ (S3 event trigger)
   AWS Lambda   ← Detects new file drop, triggers Glue Workflow
        │
        ▼
   Glue Crawler #1  ← Infers schema from raw data → registers in Data Catalog
        │
        ▼
   Glue ETL Notebook (PySpark)
   ├── Data cleansing & validation
   ├── Year-over-year inflation calculations
   ├── Rolling averages
   └── CPI + Unemployment join
        │
        ▼
   S3 Processed Zone  (Parquet, partitioned)
        │
        ▼
   Glue Crawler #2  ← Infers transformed schema → updates Data Catalog
        │
        ▼
   Amazon Athena  ← Ad-hoc SQL queries
        │
        ▼
   Tableau Dashboards
```

---

## Key Transformations (PySpark)

- Year-over-year inflation rate calculation
- Rolling average smoothing for trend analysis
- Join of CPI and unemployment datasets on date key
- Filtering and aggregation by year, month, and gender
- Output written as partitioned Parquet for fast Athena queries

---

## Athena Query Examples

```sql
-- Latest annual inflation rate
SELECT year, AVG(inflation_rate) AS avg_inflation
FROM transformed_cpi
GROUP BY year
ORDER BY year DESC;

-- Average unemployment by sex for 2014
SELECT sex, AVG(unemployment_rate) AS avg_rate
FROM transformed_unemployment
WHERE year = 2014
GROUP BY sex;

-- Average unemployment rate per year
SELECT year, AVG(unemployment_rate) AS avg_rate
FROM transformed_unemployment
GROUP BY year
ORDER BY year;
```

---

## Tableau Dashboards

### 1. Cost of Living Over Time (1913–2023)
- CPI index trend over 101 years
- Peak reached in 2014 at index value 233.9 (2.3× higher than 1913 baseline)

### 2. Annual Inflation Rate — Extremes & Drill-Down
- Highest inflation: **17.80% in 1917** (April alone contributed 4.61%)
- Deepest deflation: **-10.85% in 1921** (February at -3.16% was the worst month)
- Drill-down from annual → monthly view; roll-up back to summary

### 3. Unemployment by Gender (1948–2015)
- Men consistently experience higher unemployment during downturns
- 2010 Great Recession represented the worst modern labor crisis

### 4. Price Volatility Analysis (2007–2013)
- Monthly price fluctuation during the Great Recession era
- 2008 financial crisis produced the most extreme swings including -1.99% deflation

---

## Key Insights

| Dimension | Insight |
|---|---|
| Temporal Trends | 1970s stagflation hit 13.5%; 2022 post-COVID peak reached 9.1% |
| Labor Correlation | Phillips Curve relationship broke down post-2020 |
| Volatility Regimes | Inflation >5% with unemployment spike signals recession risk |
| Predictive Proxy | Unemployment >4% historically precedes ~1.5% inflation drop next quarter |

---

## Infrastructure Notes

- **IAM:** Least-privilege roles defined per service (Lambda, Glue, Athena, S3)
- **S3 Structure:**
```
  s3://cloudproblem-project-b1/
  ├── raw-data/         ← CSV and JSON source files
  └── processed-data/   ← Parquet output, partitioned by year
```
- **Lambda:** Python-based trigger on S3 `ObjectCreated` event
- **Glue Workflow:** Sequential — Crawler #1 → ETL Notebook → Crawler #2
- **CloudWatch:** Logs captured for Lambda invocations and Glue job runs

---

## Limitations & Future Improvements

- Data ingestion is manual (Kaggle download + S3 upload); direct BLS API integration
  would fully automate ingestion
- No live data connection in Tableau (static extract); Athena JDBC connector
  would enable real-time refresh
- EventBridge scheduled triggers could replace manual uploads for monthly automation
- ML forecasting layer (SageMaker) planned for inflation trend prediction

---

## Learnings

- Serverless, event-driven pipeline design on AWS
- PySpark transformations in AWS Glue Notebooks
- Schema-on-read architecture using Glue Crawler + Athena
- IAM least-privilege role design across multiple AWS services
- Parquet partitioning strategies for query performance
- Connecting Athena to Tableau for BI on S3 data
