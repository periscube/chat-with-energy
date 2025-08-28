-----

# Conversational AI Platform for ENTSO-E Energy Data

This repository contains the complete data pipeline and configuration for building a conversational AI platform on European energy market data from the ENTSO-E Transparency Platform. The goal is to automate the ingestion, validation, transformation, and modeling of this data to enable intuitive, natural language querying via a Generative BI interface.

## 📋 Project Management

This project is managed using **GitHub Projects**. You can view our Kanban board, track current tasks, and see the project's progress here:

➡️ https://github.com/orgs/periscube/projects/2

We use GitHub Issues to track user stories and tasks. Each issue is categorized under an Epic using labels for clear organization. GitHub's free plan provides a full feature set for public repositories, including unlimited collaborators and project boards, making it ideal for our team.

## 🏛️ Architecture

The data pipeline is designed as a modern, multi-stage ELT (Extract, Load, Transform) process orchestrated by Apache Airflow.

\!([https://user-images.githubusercontent.com/](https://www.google.com/search?q=https://user-images.githubusercontent.com/)…/architecture.png)  
*(You can create and upload a diagram image to your repo and link it here)*

The data flow is as follows:

1.  **Unload (Airflow):** An Airflow DAG runs on a schedule to trigger a Python script.
2.  **Extract & Validate:** The script fetches the latest data from the ENTSO-E RESTful API.
3.  **In-Flight Quality Checks:**
      * **Schema Validation:** `Pydantic` models perform strict type and schema checking on the raw API response.
      * **Data Quality:** `Great Expectations` runs data quality checks against the validated data to ensure its integrity and correctness.
4.  **Stage:** The validated, clean data is converted to the efficient **Parquet** format and staged in a local **MinIO** object storage bucket.
5.  **Load (StarRocks):** An Airflow task initiates a `Broker Load` job in StarRocks, which efficiently ingests the Parquet files from MinIO into the raw data tables.
6.  **Transform (dbt):** Once loading is complete, Airflow triggers a `dbt run` command. dbt executes a series of SQL models that transform the raw data into a clean, AI-friendly star schema (facts and dimensions).
7.  **Semantic Layer (Wren AI):** Wren AI connects to the final analytical tables in StarRocks. It hosts the semantic layer, which includes business logic, taxonomy, and aliases, enabling the Text-to-SQL engine.
8.  **Chat with Data:** Users interact with the data by asking natural language questions through the Wren AI interface, which leverages Google's Gemini API for its conversational capabilities.

## 🛠️ Tech Stack

This project utilizes a modern, open-source data stack:

  * **Orchestration:** Apache Airflow
  * **Programming Language:** Python
  * **Dependency Management:** Poetry
  * **Data Warehouse:** StarRocks (running on Kubernetes)
  * **Staging Layer:** MinIO (S3-compatible object storage)
  * **Data Transformation:** dbt (Data Build Tool) & dbt-expectations
  * **Data Validation:** Pydantic, Great Expectations
  * **Semantic & BI Layer:** Wren AI
  * **Generative AI:** Google Gemini API
  * **Infrastructure:** Docker Desktop, Kubernetes (k8s), WSL 2

## 🚀 Getting Started

Follow these steps to set up the development environment locally.

### Prerequisites

  * Docker Desktop with WSL 2 enabled
  * Kubernetes enabled within Docker Desktop
  * Python 3.9+
  * **Poetry** for Python package management. (See [installation guide](https://www.google.com/search?q=https://python-poetry.org/docs/%23installation))
  * An approved **ENTSO-E API Security Token**.
  * A **Google Gemini API Key**.

### 1\. Clone the Repository

```bash
git clone [your-repository-url]
cd [your-repository-name]
```

### 2\. Set Up Infrastructure

Deploy the necessary infrastructure components using Docker and Kubernetes.

  * **StarRocks Cluster:** Follow the instructions in `/infrastructure/starrocks/` to deploy a StarRocks cluster on k8s.
  * **MinIO Object Storage:** Follow the instructions in `/infrastructure/minio/` to deploy a MinIO instance.

### 3\. Configure the Environment

  * **Python Dependencies:** This project uses Poetry for dependency management. Install the required packages from the `pyproject.toml` file:
    ```bash
    poetry install
    ```
  * **Airflow Setup:** Initialize Airflow and configure the connections for StarRocks and MinIO in the Airflow UI.
  * **dbt Setup:** Configure your `profiles.yml` file in the `dbt_project` directory to connect to your local StarRocks instance.
  * **Wren AI Setup:** Launch Wren AI and configure its connection to StarRocks. Add your Google Gemini API key to the Wren AI settings.

### 4\. Run the Pipeline

You can trigger the main DAG from the Airflow UI to run the end-to-end pipeline. There are two primary DAGs:

  * `entsoe_daily_load`: For incremental daily data ingestion.
  * `entsoe_historical_backfill`: For the initial bulk load of the last two years of data.

## 📂 Project Structure

```
.
├── airflow/
│   ├── dags/                 # Airflow DAG definitions
│   └── scripts/              # Python scripts for extraction and loading
├── dbt_project/
│   ├── models/               # dbt models (staging, marts)
│   └── dbt_project.yml       # dbt project configuration
├── infrastructure/
│   ├── starrocks/            # Kubernetes configs for StarRocks
│   └── minio/                # Docker-compose for MinIO
├── great_expectations/
│   └── expectations/         # Expectation Suites for data quality
├──.gitignore
├── README.md
└── pyproject.toml            # Poetry configuration and dependencies
```

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
