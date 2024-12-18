# End-to-End Data Engineering with Azure, Databricks and dbt

This project demonstrates a complete data engineering pipeline using Azure Databricks, Data Build Tool (DBT), and Azure as the cloud provider. The project showcases the process of data ingestion into a lakehouse, data integration with Azure Data Factory (ADF), and data transformation with Databricks and DBT.

**Project Architecture:**

- **Medalion Architecture:** We utilize a medallion architecture for data storage, consisting of bronze, silver, and gold layers.
- **Azure Services:**

    - Resource Group: `rg-adventureworks-dev` 
    - ADLS Gen2 Storage: `adlsadventureworksdev` (hierarchical namespace) with containers: bronze, silver, gold
    - Azure Data Factory: `adf-adventureworks-dev1`
    - Azure Key Vault: `kv-adventureworks-dev` for storing secrets
    - Azure SQL Database: `sqldb-adventureworks-dev` with automatic data population
    - Azure Databricks Workspace: `db-adventureworks-dev`

**1. Project Setup:**

- **Create Azure Resources:**
    - Use Azure CLI or the Azure portal to create the following resources within the specified resource group:
        - ADLS Gen2 storage account with hierarchical namespace and the three containers (bronze, silver, gold).
        - Azure Data Factory.
        - Azure Key Vault.
        - Azure SQL Database with automatic data population.
- **Configure Azure Data Factory:**
    - Create linked services for Azure SQL Database and ADLS Gen2 storage.
    - Create datasets for Azure SQL tables and ADLS Gen2 Parquet files.
    - Create a pipeline to orchestrate the data ingestion process:
        - Use a Lookup activity to fetch all tables from the SQL Database.
        - Use a For Each loop to iterate through each table.
        - Use a Copy Data activity within the loop to copy each table from SQL Database to the bronze container in ADLS Gen2 as Parquet files.

**2. Configure Databricks Workspace:**
    - Create a new Databricks workspace.
    - Create a secret scope in Databricks and store the ADLS Gen2 access key from the Key Vault.
    - Mount the bronze, silver, and gold containers from ADLS Gen2 to Databricks using the secret scope.
- **Configure Databricks Notebook (Bronze Layer):**
    - Create a Databricks notebook with the following logic:
        - Create a database in Databricks.
        - Create external tables in Databricks pointing to the Parquet files in the bronze container.
- **Configure DBT:**
    - Initialize a new DBT project: `dbt init adventureworks_dbt_databricks`
    - Configure DBT with your Databricks workspace details (host, HTTP path, token).
    - Configure DBT to use the Databricks database and schema.

**3. DBT Model Development:**

- **Snapshots (Silver Layer):**
    - Create snapshot files in the `dbt/snapshots` folder for each dimension table.
    - Define snapshot configurations (file format, location, unique key, strategy, etc.).
    - Create a `bronze.yml` file in the `models` folder to define source tables in the bronze layer.
- **Data Marts (Gold Layer):**
    - Create directories for dimension tables (e.g., `models/marts/customer`) and fact tables.
    - Create YAML files (e.g., `dim_customer.yml`) to configure materialized tables, file format, location, and data quality checks.
    - Create SQL files (e.g., `dim_customer.sql`) to join tables and perform transformations.

**4. Run DBT:**

- Run `dbt snapshot` to create snapshots of dimension tables.
- Run `dbt run` to execute the SQL transformations and create data marts.
- Run `dbt test` to perform data quality checks on the created tables.
- Generate and serve documentation: `dbt docs generate` and `dbt docs serve`
