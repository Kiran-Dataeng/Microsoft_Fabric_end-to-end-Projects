# Microsoft_Fabric_end-to-end-Projects


# Fabric Data Pipeline — Retail Sales Ingestion

## Overview
A Microsoft Fabric pipeline (`ingets_data_pipe`) that ingests `project1_retail_sales_clean.csv`
from Databricks storage into a Lakehouse, landing it in the Bronze layer as raw,
unprocessed data. The pipeline is triggered both by a Databricks event and an
every-minute schedule, and is monitored through Fabric's run history.

## Architecture
<img width="1440" height="600" alt="image" src="https://github.com/user-attachments/assets/68514308-b440-4141-877f-ca03c95f3744" />

**Flow:**
project1_retail_sales_clean.csv → ingets_data_pipe (Copy data activity) → Lakehouse (Bronze table)
Triggered by: Databricks event trigger + schedule trigger (every minute)

## What "Bronze" Means in This Context
The Bronze layer is the raw ingestion zone. Data lands here exactly as it arrives
from the source — no cleaning, no transformation, no validation. Its purpose is to:
- Preserve an unaltered copy of the source data for traceability
- Act as the single source of truth if downstream layers (Silver/Gold) ever need to be rebuilt
- Catch schema drift or bad data early, before it spreads downstream

In this project, Bronze = the raw table inside the Lakehouse, storing
`project1_retail_sales_clean.csv`'s columns exactly as ingested.

## Pipeline Details
- **Source:** `project1_retail_sales_clean.csv` (Databricks file storage)
- **Destination:** Lakehouse → Bronze table
- **Copy method:** `Copy data1` activity inside `ingets_data_pipe`
- **Triggers:**
  - Databricks event trigger (`newtrigger1`) — fires on every value
  - Schedule trigger — every minute

## Pipeline Screenshot
<img width="1002" height="593" alt="image" src="https://github.com/user-attachments/assets/36eb30ba-3320-4a27-89e0-b411155aad3b" />

## Monitoring & Failure Handling
- Checked run history in Fabric's Monitor Hub to confirm successful/failed runs
- Simulated a failure by pointing the source to a bad file/column type mismatch
- Observed error: `TypeConversionFailure` — a string value like "2410.72" couldn't
  be converted to the destination column's type
- Resolution: corrected the destination column type to Double/Decimal and
  re-ran the pipeline against a freshly created Bronze table

## Next Steps
- Add a Silver layer with cleaning/validation logic
- Add a Gold layer with aggregated business metrics
- Add alerting on pipeline failure
