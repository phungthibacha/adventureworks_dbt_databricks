# End-to-End Data Engineering with Azure, Databricks and dbt: A Deep Dive

This project demonstrates a complete data engineering pipeline leveraging Azure Databricks, Data Build Tool (DBT), and Azure cloud services. The project adheres to a **Medalion Architecture**, a data management paradigm that promotes a structured and iterative approach to data processing, ensuring data quality and reliability throughout the lifecycle.

![Medalion Architecture.png](https://github.com/phungthibacha/adventureworks_dbt_databricks/blob/master/Medalion%20Architecture.png)

## 1. Data Ingestion (Bronze Layer)

* **Objective:** Extract raw data from the source system and land it in a raw, unprocessed format.
* **Source:** Azure SQL Database (`sqldb-adventureworks-dev`) containing Adventure Works data (sale transactions, customers, products, address, ...).
* **Destination:** ADLS Gen2 `bronze` container.
* **Process:**
    1. **Data Factory Orchestration:**
        - Create an Azure Data Factory pipeline to automate the data extraction process.
        - Utilize a **Lookup** activity to dynamically retrieve a list of all tables within the specified schema (e.g., `SalesLT`) in the SQL Database.
        - Employ a **For Each** loop to iterate through each table identified by the Lookup activity.
        - Within the loop, execute a **Copy Data** activity to extract data from each SQL table and transfer it to the `bronze` container in ADLS Gen2.
        - Store data in Parquet format for efficient data processing and storage.
    2. **Databricks Integration:**
        - Integrate Databricks with the Data Factory pipeline to perform initial data validation and preparation.
        - Create a Databricks notebook that executes after the Copy Data activity within the Data Factory pipeline.
        - Within the notebook:
            - Create a dedicated database in Databricks to organize ingested data.
            - Create external tables in Databricks pointing to the Parquet files located in the `bronze` container. This enables querying and basic data exploration within the Databricks environment.

## 2. Data Transformation (Silver Layer)

* **Objective:** Transform raw data from the bronze layer into a more structured and consistent format suitable for analysis.
* **Source:** Parquet files in the ADLS Gen2 `bronze` container.
* **Destination:** ADLS Gen2 `silver` container.
* **Process:**
    1. **DBT for Data Transformation:**
        - Utilize DBT to manage and orchestrate data transformations.
        - Create snapshot files within the `dbt/snapshots` directory for each dimension table. 
            - Define snapshot configurations, including:
                - File format (Delta Lake for efficient versioning and time travel)
                - Location within the `silver` container
                - Unique key for identifying records
                - Invalidation strategy (e.g., `check` to compare with the source)
        - Create a `bronze.yml` file to define source tables within the DBT project, referencing the external tables created in Databricks.
        - Execute `dbt snapshot` to create snapshot tables in the `silver` container. This process captures the current state of the data and enables tracking changes over time (SCD type 2).

## 3. Data Mart Creation (Gold Layer)

* **Objective:** Create refined, business-oriented data marts for analytical purposes.
* **Source:** Transformed data from the `silver` layer.
* **Destination:** ADLS Gen2 `gold` container.
* **Process:**
    1. **DBT for Data Mart Creation:**
        - Create directories within the `models` folder to organize data marts (e.g., `models/marts/customer`, `models/marts/product`).
        - Define DBT models using YAML files (e.g., `dim_customer.yml`) to:
            - Specify the target table (e.g., `dim_customer`)
            - Configure table properties (e.g., file format, location, partitioning)
            - Define data quality checks (e.g., constraints, checks)
        - Create SQL files (e.g., `dim_customer.sql`) to implement data transformations:
            - Join tables from the `silver` layer.
            - Apply business logic (e.g., calculations, aggregations).
            - Handle data inconsistencies or missing values.
        - Execute `dbt run` to create the specified data marts in the `gold` container.

## 4. Data Quality & Testing

* **DBT Tests:**
    - Leverage DBT's testing framework to perform data quality checks:
        - Unique key constraints
        - Not null constraints
        - Check constraints for specific business rules
        - Data type validations
        - Referential integrity checks
    - Execute `dbt test` to run the defined tests and identify any data quality issues.
* **Documentation:**
    - Generate and serve interactive data documentation using `dbt docs generate` and `dbt docs serve`. This provides valuable insights into the data model, lineage, and data quality.

## Project Setup

* **Prerequisites:**
    - Active Azure subscription
    - Databricks CLI, DBT CLI 
* **Azure Resources:**
![Azure Resource Groups.png]((https://github.com/phungthibacha/adventureworks_dbt_databricks/blob/master/Azure%20Resource%20Groups.png))
    - Create a Resource Group, ADLS Gen2 storage account, Data Factory, Key Vault, and SQL Database.
* **Databricks Setup:**
    - Create a Databricks workspace, configure secret scope, and mount ADLS Gen2 containers.
* **DBT Setup:**
    - Initialize a DBT project, configure DBT with Databricks workspace details.

## Key Considerations:**

* **Data Lineage:** Implement data lineage tracking to understand the origin and transformations of data throughout the pipeline.
* **Data Profiling:** Conduct regular data profiling to identify data quality issues, understand data distributions, and inform data modeling decisions.
* **Scalability and Performance:** Optimize data pipelines and transformations for performance and scalability to handle growing data volumes.
* **Security:** Implement robust security measures to protect sensitive data throughout the pipeline.
