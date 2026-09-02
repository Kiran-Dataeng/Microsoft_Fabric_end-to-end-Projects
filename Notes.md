Lakehouse
-----------
What it is: A combination of a data lake (files) and a database (tables), unified. Under the hood, it stores data as Delta Parquet files, but lets you query it with SQL or process it with Spark (PySpark, Scala, etc.) — exactly what you've been doing with spark.read.csv(...), unionByName, etc.

Who uses it: Data engineers, data scientists — anyone doing heavy transformation, cleaning, or ML work.

Key trait:  Schema-flexible and code-first. You can dump raw CSV/JSON/XML files into the "Files" area (unstructured), then turn them into governed Delta "Tables" (structured) via Spark notebooks — which is exactly the CSV/TSV/JSON/XML/Parquet workflow you've been building.


Analogy: A big flexible warehouse room where raw boxes (files) sit next to labeled, shelved inventory (tables) — and you, the engineer, decide when to move something from a box into a shelf.

================================================================================================================================================================================================================================================================================



Warehouse
-----------
What it is: A traditional, SQL-only relational data warehouse. No Spark, no notebooks — just structured tables and T-SQL, like a modern version of SQL Server or Synapse SQL pools.

Who uses it: Analysts and BI developers who think in SQL, not code. Also good for enforcing strict schemas, transactions, and stored procedures.

Key trait: Fully structured and governed from the start — every table has a fixed, enforced schema. No "raw files" area like Lakehouse has.

Analogy: A finished, organized storefront — everything already labeled, priced, and shelved. Customers (report builders) just come in and buy (query) — no rummaging through boxes.


================================================================================================================================================================================================================================================================================


Data Factory (within Fabric)
-----------------------------------

What it is: The orchestration and ingestion layer — pipelines that move data from A to B on a schedule or trigger, and can call other things (like your notebook) as a step.

Who uses it: Data engineers setting up automated workflows — "every night at 2am, pull new files from an SFTP server / API / on-prem database, land them in the Lakehouse, then run the transformation notebook, then refresh the Warehouse."

Key trait: It doesn't store or transform data itself in a deep way (though it has some light transform activities) — it connects and sequences other things. It's the glue, not the destination.

Analogy: The delivery trucks and dispatch schedule — deciding what gets picked up, from where, when, and dropped off at which building (Lakehouse or Warehouse).

================================================================================================================================================================================================================================================================================


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



## Key Learning: Schema-Enabled vs Default Lakehouse Behavior

While loading the Bronze table into a Spark DataFrame inside a Fabric notebook,
I discovered an important nuance in how Fabric resolves table references depending
on which Lakehouse is set as the notebook's default:

**Observation:**
- When a **schema-enabled Lakehouse** (e.g., `Bronze_lh`, with tables organized under
  a schema like `Bronze.retail_sale_messy`) is attached but **not set as the default**
  Lakehouse, Spark cannot resolve `schema.table` or even fully qualified 3-part names
  (`lakehouse.schema.table`) reliably — it throws errors like
  `TABLE_OR_VIEW_NOT_FOUND` or `REQUIRES_SINGLE_PART_NAMESPACE`.
- When a **non-schema-enabled Lakehouse** (flat table structure, no schema folder)
  is set as the default, tables can be read directly using just the table name:
```python
  df = spark.read.table("table_name")
```
- However, once a **non-schema Lakehouse is set as default**, attempting to read
  from a **different, schema-enabled Lakehouse** using a fully qualified 3-part
  name still fails — Spark's default `spark_catalog` only supports a single-part
  namespace, and does not correctly traverse cross-Lakehouse schema paths.

**Root cause:**
Fabric's Spark SQL catalog resolution is tightly coupled to whichever Lakehouse
is set as default for the notebook session. Schema-enabled Lakehouses require
either:
1. Being set as the default Lakehouse (allowing `schema.table` syntax), or
2. Being queried through a fully qualified path that matches Fabric's internal
   catalog structure exactly (which can require 3- or 4-part naming depending
   on how the Lakehouse is registered, e.g. via a linked/mirrored source).

**Resolution / Best Practice:**
- Keep one Lakehouse attached per notebook where possible, and explicitly set it
  as default before running queries.
- For schema-enabled Lakehouses, always reference tables as `schema.table_name`.
- If cross-Lakehouse access is required and catalog resolution keeps failing,
  bypass the SQL catalog entirely and read the Delta table directly via its
  OneLake ABFS path:
```python
  df = spark.read.format("delta").load(
      "abfss://<workspace-id>@onelake.dfs.fabric.microsoft.com/<Lakehouse>.Lakehouse/Tables/<schema>/<table_name>"
  )
```
  This method is independent of default Lakehouse settings or catalog depth,
  and reliably works across schema-enabled and non-schema-enabled Lakehouses alike.



## Next Steps
- Add a Silver layer with cleaning/validation logic
- Add a Gold layer with aggregated business metrics
- Add alerting on pipeline failure
