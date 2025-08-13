# Azure Cloud Architecture Walkthrough

This walkthrough explains how Azure resources were provisioned and integrated for the **LendingClub Loan Data ETL Pipeline**.  
All resources were set up using **Azure credits**, enabling a production-like environment for development and testing without additional costs.

---

## 1. Azure Data Lake Storage (ADLS) – Staging Area
![ADLS Staging](../img/azure/01_adls_staging.png)  

The raw **LendingClub 2018Q4 dataset** was uploaded to **Azure Data Lake Storage Gen2** in the container `stagingadls`.

**Configuration Highlights:**
- **Hot access tier** for low-latency reads during frequent ETL operations  
- **Geo-redundant storage (GRS)** to maintain a backup copy in another Azure region for disaster recovery  
- Acts as the **first step in the ETL pipeline**, serving as a secure and highly available landing zone before transformation

---

## 2. Azure Databricks – Mounting ADLS
![Databricks Mount](../img/azure/02_databricks_mount.png)  

Using **Azure Databricks**, the ADLS container (`stagingblob`) was mounted into the Databricks File System (DBFS).  
This allowed PySpark to directly access raw CSV files stored in ADLS without manual download/upload steps.

**Key Benefits of Mounting:**
- Direct read/write between Databricks and ADLS  
- Simplified file path references in PySpark code  
- Centralized cloud storage integration for scalable data processing

---

## 3. Writing Processed Data to Azure SQL Database
![Write to SQL](../img/azure/03_write_to_sql.png)  

Once cleaned and transformed in Databricks, the dataset was written to **Azure SQL Database** via the Spark JDBC connector.

**Configuration Details:**
- JDBC URL with database name, server host, and port  
- SQL Server JDBC driver: `com.microsoft.sqlserver.jdbc.SQLServerDriver`  
- Target table: `loan_data`  
- Write mode: `overwrite` to refresh data with each pipeline run

**Outcome:**
- Persistent storage of curated, analytics-ready data  
- Fast SQL queries for downstream analysis  
- Easy integration with BI/reporting tools like Power BI

---

## 4. Querying Final Data in Azure SQL Database
![SQL Query Results](../img/azure/04_sql_query_results.png)  

After loading, the processed `loan_data` table was queried in **Azure SQL Database Query Editor** to verify successful ingestion.

**Verification:**
- Schema confirmed with cleaned and properly typed columns (`term`, `int_rate`, `home_ownership`, `grade`, `purpose`)  
- Sample records validated that transformations (e.g., removing `%`, stripping text, converting types) were correctly applied  
- Data is ready for advanced analytics, machine learning, or BI dashboards

---

## End-to-End Flow Summary

1. **Extract** – Upload raw LendingClub CSV to ADLS staging container (`stagingadls`)  
2. **Transform** – Use Azure Databricks + PySpark to clean, format, and derive features  
3. **Load** – Write transformed data into Azure SQL Database via JDBC  
4. **Validate** – Query processed table in Azure SQL to ensure accuracy

---

## Notes
- All resources were provisioned using **Azure credits**, simulating a real-world cloud deployment without cost.  
