# Analytics Engineering with dbt Core: NYC Taxi Data

## Overview
This repository contains an Analytics Engineering project using **dbt Core** and **Google BigQuery**. It is designed to transform raw NYC Yellow Taxi trip data into clean, tested dimensional models that are ready for downstream reporting and analysis.

## Tech Stack
* **Tool:** dbt Core
* **Data Warehouse:** Google Cloud BigQuery
* **Packages:** `dbt_utils` (for surrogate key generation)

## Project Structure
* **`models/`**: Contains the core SQL transformations.
  * `staging/`: Base views that clean raw data and cast data types.
  * `marts/`: Fact and dimension tables.
* **`seeds/`**: Static CSV mapping files for dimensions like taxi zones.
* **`macros/`**: Custom SQL macros for repeated business logic.
* **`packages.yml`**: Defines external dbt dependencies.

## Prerequisites
* A Google Cloud Project with BigQuery enabled.
* A BigQuery dataset designated for this project (e.g., `zoomcamp`).
* A GCP Service Account JSON key with BigQuery Data Editor permissions.

## Setup & Execution

### 1. Configure dbt Profile
Create or update your `~/.dbt/profiles.yml` file to connect to your BigQuery project. Ensure the `keyfile` points to your securely ignored service account JSON:

```yaml
taxi_rides_ny:
  target: dev
  outputs:
    dev:
      type: bigquery
      method: service-account
      project: kestra-sandbox-511811
      dataset: zoomcamp
      threads: 4
      keyfile: /workspaces/docker/dbtcore/service-account.json
      timeout_seconds: 300
      location: europe-west1
```

### 2. Run the Pipeline
Navigate to your project directory (e.g., `dbtcore/taxi_rides_ny`) and execute the following commands in your terminal:

```bash
# Install required packages (like dbt_utils)
dbt deps

# Load the static seed CSV files into BigQuery
dbt seed

# Compile SQL, build all models, and run schema tests
dbt build
```
