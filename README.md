# End-to-End Data Engineering Architecture

A centralized portfolio repository demonstrating end-to-end data engineering infrastructure developed through the DataTalksClub Data Engineering Zoomcamp. This repository bridges local containerized development, AI-assisted data quality audits, cloud infrastructure-as-code (IaC), declarative workflow orchestration, and cloud-scale analytics engineering with **dbt Core** across Google Cloud Platform (GCP)[cite: 1].

---

## 🏗️ System Architecture

The repository is organized into four decoupled, production-grade components that form a complete data ingestion, validation, orchestration, and transformation pipeline:

```
├── docker/        # Containerized development, streaming ingest & LLM-driven anomaly detection
├── terraform/     # Infrastructure as Code (IaC) provisioning GCP cloud resources
├── kestra/        # Declarative ELT workflow orchestration, backfilling & BigQuery upserts
└── dbt/           # Analytics engineering & dimensional data modeling in BigQuery via dbt Core
```

```
       +-------------------------------------------------------+
       |                  Terraform (IaC)                      |
       |  - Provisions Google Cloud Storage (GCS) Buckets      |
       |  - Provisions Google BigQuery Analytical Datasets     |
       +---------------------------+---------------------------+
                                   |
                                   v
+------------------------------------+   +-------------------------------------------------------+
|            Docker Core             |   |                     Kestra Engine                     |
| - Isolated Runtimes (Compose)      |-->| - Declarative YAML Workflows                          |
| - PostgreSQL Staging               |   | - Parameterized Execution & Triggers                  |
| - Service Networking               |   | - Cloud Extraction -> GCS Ingestion -> BigQuery MERGE |
| - OpenAI & Anthropic Anomaly Audits|   +---------------------------+---------------------------+
+------------------------------------+                               |
                                                                     v
                                         +-------------------------------------------------------+
                                         |           dbt Core (Analytics Engineering)            |
                                         | - Direct BigQuery Adapter Integration (dbt-bigquery)  |
                                         | - Staging Views, Fact/Dimensional Models & Seeds      |
                                         | - Jinja Macros, Documentation & Automated Data Tests  |
                                         +-------------------------------------------------------+
```

---

## 📁 Repository Modules

### 1. [Docker Infrastructure (`/docker`)](./docker)
* **Local Containerization:** Manages isolated development environments and backend services via `docker-compose`[cite: 1].
* **Database Staging:** Provisions PostgreSQL containers with persistent volume mounts for local pipeline staging[cite: 1].
* **Network Isolation:** Configures custom Docker bridge networks to enable communication between data ingest scripts and databases.
* **LLM-Powered Anomaly Insights:** Integrates OpenAI and Anthropic APIs to inspect ingestion telemetry, detect data quality anomalies, and generate automated diagnostic summaries on irregular records.(https://github.com/dhirajtripathi93/docker/blob/main/docker/pipeline/AI_INSIGHTS.md)

### 2. [Terraform Cloud Infrastructure (`/terraform`)](./terraform)
* **Infrastructure as Code (IaC):** Declarative GCP infrastructure management using HashiCorp Configuration Language (HCL)[cite: 1].
* **Storage Layer:** Provisions Google Cloud Storage (GCS) data lake buckets configured with automated lifecycle transition rules and uniform bucket-level access[cite: 1].
* **Analytical Warehouse:** Provisions and manages BigQuery analytical datasets with location-targeted configurations[cite: 1].
* **Lifecycle Automation:** Utilizes structured `variables.tf` parameterization to support repeatable `init`, `plan`, `apply`, and `destroy` workflows across environments[cite: 1].

### 3. [Kestra Orchestration Engine (`/kestra`)](./kestra)
* **Declarative Pipeline Management:** Directs end-to-end ELT workflows via declarative YAML definitions with built-in retries and execution logs.
* **Dynamic Parameterization:** Accepts runtime variables (target execution dates, datasets, table IDs) to enable workflow reusability across datasets.
* **Idempotent Data Ingestion:** Enforces data integrity during ad-hoc backfills by running pre-load staging table truncations to prevent duplicate records.
* **Production Upsert Operations:** Executes SQL `MERGE` statements (`WHEN NOT MATCHED THEN INSERT`) to load analytical data from GCS into Google BigQuery without primary key collisions.
* **Secret Management:** Securely connects to GCP using containerized Google Cloud Service Account credentials (`gcp-key.json`).

### 4. [dbt Core Analytics Engineering (`/dbt`)](./dbt)
* **Cloud-Scale Warehouse Integration:** Enhances the standard local DuckDB curriculum by configuring **dbt Core** (`dbt-bigquery`) to operate directly on the Google BigQuery tables populated by the upstream Kestra ELT pipelines[cite: 1].
* **CLI-Driven Profile Management:** Authenticates local/Codespace environments to GCP via a customized `profiles.yml` and Service Account JSON key configuration.
* **Dimensional Data Modeling:** Structures transformation layers into modular **staging** views (type casting, deduplication, renaming) and **core/mart** dimensional models (`fact` and `dim` tables) for downstream BI consumption[cite: 1].
* **Reusable Logic & Reference Data:** Leverages dynamic **Jinja macros** to abstract repetitive SQL case logic and incorporates static lookup CSVs via **dbt Seeds**.
* **Automated Data Governance & Testing:** Enforces data reliability across raw sources (`sources.yml`) and transformed models (`schema.yml`) using built-in and package-based (`dbt_utils`) assertions for uniqueness, non-null constraints, and accepted values during `dbt build`.

---

## 🚀 Execution & Verification

### Prerequisites
* **Docker & Docker Compose** installed locally or in a cloud workspace[cite: 1].
* **API Keys:** Active `OPENAI_API_KEY` and `ANTHROPIC_API_KEY` configured in `.env` for automated audit evaluations[cite: 1].
* **Terraform CLI** (v1.0+) installed.
* **dbt Core & BigQuery Adapter** (`pip install dbt-core dbt-bigquery`).
* **Google Cloud SDK (`gcloud`)** and an active GCP Project with a Service Account key (`Storage Admin` and `BigQuery Admin` roles).

### Pipeline Deployment Steps

1. **Clone the Monorepo:**
   ```bash
   git clone [https://github.com/dhirajtripathi93/docker.git](https://github.com/dhirajtripathi93/docker.git) data-engineering-portfolio
   cd data-engineering-portfolio
   ```

2. **Run Local Ingestion & LLM Audits:**
   ```bash
   cd docker
   # Configure environment variables (.env) with DB credentials and LLM API keys
   docker-compose up -d
   cd ..
   ```

3. **Provision GCP Infrastructure:**
   ```bash
   cd terraform
   terraform init
   terraform plan -out=tfplan
   terraform apply tfplan
   cd ..
   ```

4. **Deploy Containerized Orchestration (Ingest to BigQuery):**
   ```bash
   cd kestra
   # Place your gcp-key.json and set up your .env configuration
   docker-compose up -d
   # Access http://localhost:8080 to trigger flows and load raw BigQuery tables
   cd ..
   ```

5. **Execute dbt Core Transformations & Tests:**
   ```bash
   cd dbt
   # Verify connection to BigQuery via ~/.dbt/profiles.yml
   dbt debug
   # Install package dependencies, load seeds, run models, and execute tests
   dbt deps
   dbt build
   ```
