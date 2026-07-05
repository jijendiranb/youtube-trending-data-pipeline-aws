# YouTube Trending Data Pipeline

## Overview
A production-style AWS Data Engineering pipeline that ingests YouTube Trending data from the YouTube Data API, processes it using a Medallion Architecture (Bronze → Silver → Gold), validates data quality, and publishes analytics-ready datasets.

![Architecture](YouTube%20Trending%20Data%20Pipeline.png)

## Architecture

The pipeline follows the **Medallion Architecture** pattern with three data layers:

```
Data Sources          Bronze              Silver            Quality Gate          Gold              Analytics
┌──────────┐     ┌──────────────┐    ┌──────────────┐    ┌────────────┐    ┌──────────────┐    ┌──────────┐
│ YouTube  │     │              │    │              │    │            │    │  trending_   │    │          │
│ API v3   │────>│  Raw JSON    │───>│  Cleansed    │───>│  DQ Lambda │───>│  analytics   │───>│  Athena  │
│          │     │  (S3)        │    │  Parquet     │    │  Validates │    │              │    │          │
├──────────┤     │              │    │  (S3)        │    │  row count │    │  channel_    │    ├──────────┤
│ Kaggle   │     │  Raw CSV     │    │              │    │  nulls     │    │  analytics   │    │  Quick-  │
│ Dataset  │────>│  (S3)        │    │  Reference   │    │  schema    │    │              │    │  Sight   │
│          │     │              │    │  Parquet     │    │  freshness │    │  category_   │    │          │
└──────────┘     └──────────────┘    └──────────────┘    └────────────┘    │  analytics   │    └──────────┘
                                                              │           └──────────────┘
                                                         fail │
                                                              ▼
                                                        ┌────────────┐
                                                        │  SNS Alert │
                                                        └────────────┘
```

**Orchestration** is handled by AWS Step Functions, which coordinates the full pipeline with retry logic, parallel execution, and failure notifications.
## Tech Stack

- AWS Lambda
- AWS Glue (PySpark)
- Amazon S3
- AWS Glue Data Catalog
- Amazon Athena
- AWS Step Functions
- Amazon SNS
- Amazon EventBridge
- Amazon CloudWatch
- Python
- PySpark
- SQL
- AWS Wrangler
- Boto3

## Repository Structure

```text
data/
data_quality/
    yt-data-quality.py

glue_jobs/
    yt-data-bronze-to-silver.py
    yt-data-silver-to-gold.py

iam_permission/
    yt-data-glue-access.json
    yt-data-lambda-access.json
    yt-data-sfn-access.json

lambdas/
    youtube_api_integstion/
        yt-data-ingestion-layer.py
    json_to_parquet/
        yt-data-json-to-parquet.py

scripts/
    aws_copy.sh
    information.md

step_functions/
    yt-data-stepfunction.json

README.md
YouTube Trending Data Pipeline.png
```

## AWS Resources

| Resource | Name |
|----------|------|
| Bronze Bucket | yt-data-bronze-layer |
| Silver Bucket | yt-data-silver-layer |
| Gold Bucket | yt-data-gold-layer |
| Script Bucket | yt-data-scripts-list |
| Athena Results Bucket | yt-athena-results-layer |
| Bronze Glue Database | yt-data-bronze |
| Silver Glue Database | yt-data-silver |
| Gold Glue Database | yt-data-gold |
| Bronze Table | raw_statistics |
| Silver Table | clean_statistics |

## Pipeline

### Bronze
- Ingest trending videos from YouTube Data API.
- Store raw JSON/CSV files in S3 Bronze.

### Silver
- Clean and standardize data.
- Remove duplicates.
- Cast data types.
- Create derived metrics.
- Convert category JSON into Parquet.

### Data Quality
Checks include:
- Minimum row count
- Null percentage
- Schema validation
- Value range validation
- Data freshness

Pipeline stops if validation fails.

### Gold
Creates:
- trending_analytics
- channel_analytics
- category_analytics

## Lambda Functions

| Function | Purpose |
|----------|---------|
| yt-data-ingestion-layer | Download YouTube trending data |
| yt-data-json-to-parquet | Convert category JSON to Parquet |
| yt-data-quality | Validate Silver layer |

## Glue Jobs

| Job | Purpose |
|-----|---------|
| yt-data-bronze-to-silver | Bronze → Silver ETL |
| yt-data-silver-to-gold | Silver → Gold Analytics |

## Glue Parameters

Bronze to Silver

```text
--bronze_database yt-data-bronze
--bronze_table raw_statistics
--silver_bucket yt-data-silver-layer
--silver_database yt-data-silver
--silver_table clean_statistics
```

Silver to Gold

```text
--silver_database yt-data-silver
--gold_bucket yt-data-gold-layer
--gold_database yt-data-gold
```

## Environment Variables

### Ingestion Lambda

- YOUTUBE_API_KEY
- S3_BUCKET_BRONZE=yt-data-bronze-layer
- YOUTUBE_REGIONS

### Data Quality Lambda

- ATHENA_QUERY_OUTPUT=s3://yt-athena-results-layer/
- SNS_ALERT_TOPIC_ARN=<your-sns-topic-arn>
- DQ_MIN_ROW_COUNT=10
- DQ_MAX_NULL_PERCENT=5

## Deployment

1. Create S3 buckets.
2. Create Glue databases.
3. Upload Glue scripts to `yt-data-scripts-list`.
4. Deploy Lambda functions.
5. Create Glue jobs.
6. Create SNS topic.
7. Deploy Step Functions.
8. Configure EventBridge schedule.

## Monitoring

- AWS Step Functions
- CloudWatch Logs
- Amazon SNS
- Amazon Athena

Example query:

```sql
SELECT *
FROM "yt-data-gold"."channel_analytics"
LIMIT 10;
```

## Supported Regions

US, GB, CA, DE, FR, IN, JP, KR, MX, RU

## Data Sources

- YouTube Data API v3
- Kaggle YouTube Trending Dataset

## Future Improvements

- CI/CD using GitHub Actions
- Infrastructure as Code with Terraform
- Delta Lake support
- Incremental processing
- Data lineage
- Dashboard integration (Power BI / QuickSight)

## License

MIT License
