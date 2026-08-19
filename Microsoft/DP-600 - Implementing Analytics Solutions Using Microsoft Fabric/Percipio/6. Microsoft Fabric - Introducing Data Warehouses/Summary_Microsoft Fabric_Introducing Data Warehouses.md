# SECTION 1: Strategic Architectural Placement

```
+-------------------------------------------------------------------------------------------------+
|                                        MICROSOFT OneLake                                        |
+-------------------------------------------------------------------------------------------------+
|                                                |                                                |
|             [ FABRIC LAKEHOUSE ]               |              [ FABRIC WAREHOUSE ]              |
|                                                |                                                |
|  - Data Types: Unstructured, Semi-             |  - Data Types: Strictly Structured             |
|    structured, and Structured.                 |    (relational table architectures).           |
|  - Primary Engine: Apache Spark                |  - Primary Engine: Polaris SQL Distributed     |
|    (Python, Scala, Spark SQL, R).              |    Query Engine.                               |
|  - SQL Capabilities: Read-Only (T-SQL)         |  - SQL Capabilities: Full Read & Write         |
|    via the SQL Analytics Endpoint.             |    (DQL, DML, DDL).                            |
|  - Transactions: ACID at single-table layer    |  - Transactions: ACID multi-table support      |
|    using Delta transaction logs.               |    (explicit BEGIN/COMMIT TRANSACTION).        |
|  - Storage Format: Snappy Parquet files        |  - Storage Format: Delta Snappy Parquet        |
|    wrapped in Delta Lake open-source schemas.  |    optimized in OneLake.                        |
+-------------------------------------------------------------------------------------------------+
```

### 1. Unified Architectural Matrix

| Parameter / Feature | Fabric Data Warehouse | Fabric Data Lakehouse |
| :--- | :--- | :--- |
| **Data Modality** | Structured Relational Tables | Structured, Semi-Structured, Unstructured |
| **Primary Query Engine** | Polaris Distributed SQL Engine | Apache Spark / Polaris (for Read Endpoint) |
| **Data Modification (DML)** | Fully Supported (`INSERT`, `UPDATE`, `DELETE`) | Blocked on SQL Endpoint (Requires Spark) |
| **Schema Definition (DDL)** | Fully Supported (`CREATE`, `ALTER`, `DROP`) | Blocked on SQL Endpoint (Requires Spark) |
| **Transaction Control** | Explicit (`BEGIN`, `COMMIT`, `ROLLBACK`) | Implicit only via single-table Delta commits |
| **Stored Procedures / Triggers** | Fully Supported | Not Supported |
| **Primary Development Tool** | T-SQL Query Editor / SSMS / ADS | Spark Notebooks / PySpark / Data Wrangler |
| **Primary Audience** | SQL Engineers, Warehouse Devs, BI Creators | Data Engineers, Data Scientists, ML Engineers |

### 2. Alternative Specialized Storage Architectures
*   **Power BI Datamarts:**
    *   **Target Audience:** Citizen Developers and Business Analysts.
    *   **Development Modality:** No-code/low-code graphical user interface utilizing **Power Query** under the hood.
    *   **Data Limit:** Designed specifically for database sizes up to **100 GB**.
    *   **Language Support:** T-SQL read/write capabilities alongside graphical M-expression builders.
*   **KQL Databases (Kusto Query Language):**
    *   **Target Audience:** Real-time stream engineers, systems operators, security analysts.
    *   **Data Modality:** High-velocity telemetry, time-series metrics, system logs, sensor data, and IoT streams.
    *   **Scalability:** Optimized to ingest and run sub-second searches across terabytes to petabytes of event records.
    *   **Language Support:** Kusto Query Language (KQL) explicitly, featuring strong native support for geospatial functions and temporal aggregation.

---

# SECTION 2: Ingestion & In-Place Processing

### 1. Ingestion Methods: Core Capabilities

```
                       +---------------------------------------+
                       |        DATA INGESTION PIPELINES       |
                       +-------------------+-------------------+
                                           |
            +------------------------------+------------------------------+
            |                              |                              |
+-----------v-----------+      +-----------v-----------+      +-----------v-----------+
|    COPY INTO (SQL)    |      |    DATA PIPELINES     |      |    DATAFLOWS GEN2     |
|                       |      |                       |      |                       |
|  - Direct, high-      |      |  - End-to-end multi-  |      |  - Low-code visual    |
|    throughput SQL.    |      |    step pipelines.    |      |    ETL modeling.      |
|  - Fast, parallel     |      |  - Complex multi-     |      |  - Built on Power     |
|    bulk loading.      |      |    source scheduling. |      |    Query engine.      |
+-----------------------+      +-----------------------+      +-----------------------+
```

*   **COPY INTO (SQL Command):** Direct, high-throughput SQL engine mechanism. It is optimal for fast, parallel bulk-loading of structured CSV or Parquet files from external cloud directories (ADLS Gen2, Azure Blob Storage) straight into database schemas.
*   **Data Pipelines:** Robust, end-to-end orchestration workflows inside Azure Data Factory. Ideal for complex scheduling, multi-system coordination, loop tasks, variable evaluation, and alerting (retry policies, failure paths).
*   **Dataflows Gen2:** Low-code graphical ETL environments using **Power Query**. Best suited for business analysts who need to clean, combine, and reshape source tables visually before loading them into target tables. Integrates with the Power Query **M formula language**.

---

### 2. Bulk Loading Data via `COPY INTO`

#### Establishing Shared Access Signature (SAS) Authorization
To allow Microsoft Fabric’s internal Polaris query engine to communicate directly with Azure Blob Storage or ADLS Gen2, you must configure a secure connection. The best practice for delegating this access without exposing master keys is generating a **Shared Access Signature (SAS)** token.

*   **Target Modality:** Restrict services strictly to `Blob` storage.
*   **Resource Boundaries:** Enable `Service` (entire storage level), `Container` (directory level), and `Object` (specific CSV/Parquet files) scopes.
*   **Permission Sets (Least Privilege):** 
    *   *Read-Only / Ingestion:* Enable only `Read` and `List` permissions.
    *   *Read-Write / Error Logging:* Enable `Read`, `Write`, `Delete`, `List`, `Add`, and `Create` permissions.

#### Core T-SQL Schema Construction
```sql
-- Explicit DDL executed in the Data Warehouse
CREATE TABLE [dbo].[SupermarketSales] (
    [InvoiceID] VARCHAR(20) NOT NULL,
    [Branch] CHAR(1) NOT NULL,
    [City] VARCHAR(50) NOT NULL,
    [CustomerType] VARCHAR(20) NOT NULL,
    [Gender] VARCHAR(10) NOT NULL,
    [ProductLine] VARCHAR(50) NOT NULL,
    [UnitPrice] DECIMAL(10, 2) NOT NULL,
    [Quantity] INT NOT NULL,
    [Total] DECIMAL(10, 4) NOT NULL,
    [SaleDate] DATE NOT NULL,
    [Payment] VARCHAR(20) NOT NULL
);
```

#### Executing the Bulk `COPY INTO` Command
```sql
-- High-performance bulk load syntax
COPY INTO [dbo].[SupermarketSales]
FROM 'https://loonyfabricstorage.blob.core.windows.net/loonyfabriccontainer/supermarket_sales_no_header.csv'
WITH (
    FILE_TYPE = 'CSV',
    CREDENTIAL = (
        IDENTITY = 'Shared Access Signature', 
        SECRET = '?sv=2022-11-02&ss=b&srt=sco&sp=rl&se=2026-12-31T23:59:59Z&st=2026-08-19T12:00:00Z&spr=https&sig=SampleSignatureKeyHash='
    )
);
```

---

### 3. Bulletproofing Ingestion: Fault Tolerance & Error Auditing

By default, the `COPY INTO` transaction operates on a **zero-fault policy** (`MAXERRORS = 0`). If a single row fails conversion (such as matching a column's data type, string truncation, or encountering a file header row), the transaction is immediately aborted, and all changes are rolled back.

To build a fault-tolerant load pipeline that logs errors for post-mortem analysis:
*   **`FIRSTROW` Parameter:** Setting `FIRSTROW = 2` tells the parsing engine to skip the first record of a flat file. Use this specifically to bypass a **header row** containing column names.
*   **`MAXERRORS` Parameter:** Sets the conversion fault threshold. Setting `MAXERRORS = 10` allows up to 10 corrupt records to be skipped without failing the load. If an 11th bad row is hit, the load terminates.
*   **`ERRORFILE` Parameter:** Tells the engine to dynamically capture and output raw rejected lines and detailed error logs into a specified sub-folder in your storage container.

```sql
-- Advanced fault-tolerant bulk load with error logging
COPY INTO [dbo].[SupermarketSales]
FROM 'https://loonyfabricstorage.blob.core.windows.net/loonyfabriccontainer/supermarket_sales.csv'
WITH (
    FILE_TYPE = 'CSV',
    FIRSTROW = 2,           -- Skips header row
    MAXERRORS = 5,          -- Tolerates up to 5 individual row conversion errors
    ERRORFILE = 'https://loonyfabricstorage.blob.core.windows.net/loonyfabriccontainer/rejected_records_dir/',
    CREDENTIAL = (
        IDENTITY = 'Shared Access Signature', 
        SECRET = '?sv=2022-11-02&ss=b&srt=sco&sp=rwdlacyx&se=2026-12-31T23:59:59Z...'
    ),
    ERRORFILE_CREDENTIAL = (
        IDENTITY = 'Shared Access Signature', 
        SECRET = '?sv=2022-11-02&ss=b&srt=sco&sp=rwdlacyx&se=2026-12-31T23:59:59Z...'
    )
);
```

#### Diagnostic Output Structure
When a row is rejected, two output files are generated in the `ERRORFILE` subdirectory under a unique job run folder:
1.  **`error.json`:** Logs the system error metadata.
    ```json
    [
      {
        "Error": "Bulk load data conversion error (truncation)",
        "File": "https://loonyfabricstorage.blob.core.windows.net/loonyfabriccontainer/supermarket_sales.csv",
        "RowOffset": 0,
        "Column": 2,
        "ColumnName": "Branch",
        "Value": "Branch",
        "IsOutputted": true
      }
    ]
    ```
2.  **`row.csv`:** Captures the raw record payload that failed processing.
    ```csv
    InvoiceID,Branch,CustomerType,Gender,ProductLine,UnitPrice,Quantity,Total,Date,Payment
    ```

---

# SECTION 3: Advanced T-SQL Operations in the Warehouse

### 1. CREATE TABLE AS SELECT (CTAS)

The `CTAS` statement is an optimized DDL/DML operation. It creates a target table structure and populates it with data returned from a `SELECT` statement in a single, parallel execution.

#### CTAS with Real-time Data Transformation
```sql
-- Creates table 'MandalaySales_with_Year_Month_Day' and populates it in parallel
CREATE TABLE [dbo].[MandalaySales_with_Year_Month_Day] AS
SELECT 
    DATEPART(YEAR, [SaleDate]) AS [Year],
    DATEPART(MONTH, [SaleDate]) AS [Month],
    DATEPART(DAY, [SaleDate]) AS [DayOfMonth],
    *
FROM [dbo].[SupermarketSales]
WHERE [City] = 'Mandalay';
```

#### CTAS for Schema Cloning Only (Fast Table Copying)
To duplicate a table's exact schema, data types, and nullability properties without copying any of its data, use a universally false filter:
```sql
-- Creates an empty schema clone of the SupermarketSales table
CREATE TABLE [dbo].[SupermarketSales_Clone] AS
SELECT * 
FROM [dbo].[SupermarketSales]
WHERE 1 = 0; -- The predicate '1 = 0' is false; creates schema only with 0 records
```

---

### 2. Transaction Handling & Error Recovery

Unlike Data Lakehouse SQL analytics endpoints, Fabric Warehouses provide full transaction support. You can combine multiple `INSERT`, `UPDATE`, and `DELETE` commands within `BEGIN TRANSACTION` and `COMMIT TRANSACTION` statements. Using standard `TRY...CATCH` blocks ensures that if any operation fails, the entire transaction is rolled back.

```sql
-- Explicit multi-statement transaction with try-catch block
BEGIN TRANSACTION;

BEGIN TRY
    -- SQL Operation 1: Insert valid record
    INSERT INTO [dbo].[SupermarketSales] (InvoiceID, Branch, City, CustomerType, Gender, ProductLine, UnitPrice, Quantity, Total, SaleDate, Payment)
    VALUES ('INV99990', 'A', 'Mandalay', 'Member', 'Female', 'Health & Beauty', 25.50, 5, 127.50, '2026-08-19', 'Credit Card');

    -- SQL Operation 2: Intentionally triggers a conversion error
    -- UnitPrice is a DECIMAL(10,2) but receives a VARCHAR 'invalid_text_fault'
    INSERT INTO [dbo].[SupermarketSales] (InvoiceID, Branch, City, CustomerType, Gender, ProductLine, UnitPrice, Quantity, Total, SaleDate, Payment)
    VALUES ('INV99991', 'B', 'Mandalay', 'Normal', 'Male', 'Electronics', 'invalid_text_fault', 2, 199.98, '2026-08-19', 'Credit Card');

    -- Commit transaction if both operations succeed
    COMMIT TRANSACTION;
    PRINT 'Transaction committed successfully.';
END TRY
BEGIN CATCH
    -- Rollback all operations if any error is encountered
    ROLLBACK TRANSACTION;
    PRINT 'Transaction rolled back due to an error.';
END CATCH;
```

---

### 3. Complex Relational Querying (Pivoting Data)

The Polaris engine supports advanced T-SQL functions, such as dynamic pivoting, string manipulation, and calendar date parsing. Below is an analytical pivot query that aggregates transaction totals across months into distinct columns:

```sql
-- Transposes month rows into columns
SELECT 
    ProductLine,
    [January],
    [February],
    [March]
FROM (
    -- Source Query: Prepares data, extracting month name as string
    SELECT 
        ProductLine,
        DATENAME(MONTH, SaleDate) AS Month,
        Total
    FROM [dbo].[SupermarketSales]
) AS SourceTable
PIVOT (
    -- Aggregates numerical values for each month
    SUM(Total) FOR Month IN ([January], [February], [March])
) AS PivotTable
ORDER BY ProductLine ASC;
```

---

# SECTION 4: Database Mirroring & Replication

Fabric **Database Mirroring** is a zero-ETL data replication feature. It allows organizations to bring data from external transactional databases into Fabric in near-real-time without building or managing complex ingestion pipelines.

```
+------------------------------------+          Near-Real-Time          +------------------------------------+
|     EXTERNAL TRANSACTIONAL DB      |          Replication             |          MICROSOFT OneLake         |
|  (Azure SQL, Cosmos, Snowflake)    |==================================> (Automatically converted to Delta/ |
|                                    |       (Fully Managed, No-ETL)    |   Parquet, instant Power BI access) |
+------------------------------------+                                  +------------------------------------+
```

### 1. Near-Real-Time Synchronization
Mirroring replicates changes from source systems directly into OneLake in near-real-time. This provides analytical workloads with immediate access to operational changes.

### 2. Automatic Delta Conversion
As data is replicated, it is automatically converted into the open **Delta Lake (Parquet)** format. This makes it instantly queryable by other Fabric engines (Spark, SQL analytics endpoints) and optimized for low-latency Direct Lake reporting in Power BI.

### 3. Mirrored Data Sources
At the time of this writing, Fabric supports direct mirroring from the following external database platforms:
*   Azure SQL Database
*   Azure Cosmos DB
*   Azure Databricks
*   Snowflake (Generally Available)

---

# SECTION 5: T-SQL Identifier Delimiters (Square Brackets)

In T-SQL, square brackets `[]` serve as database object name delimiters. You must know when to use them for the exam:

*   **Handling Spaces or Special Characters:** If a table name or column contains spaces or special characters, you must enclose the identifier in square brackets to ensure correct parsing.
    ```sql
    -- Space in column name requires brackets
    SELECT [Average Revenue] FROM [dbo].[Q3 Performance Metrics];
    ```
*   **Preventing Reserved Keyword Conflicts:** If an attribute shares a name with a SQL reserved keyword (such as `Date`, `User`, or `Table`), brackets tell the parser to treat the term as an identifier rather than a command.
    ```sql
    -- 'Date' is a system keyword; must be wrapped in brackets
    SELECT [InvoiceID], [Date] FROM [dbo].[SupermarketSales];
    ```

---

# SECTION 6: Ultimate DP-600 Mastery Cheat Sheet

This master cheat sheet combines all sections (Core Capacities, Lakehouses, Spark, Medallion Architecture, and Data Warehousing) into a single unified prep tool.

### 1. Fabric Compute, Capacities, & Throttling
*   **SKUs (F-SKUs):** Ranging from **F2 to F2048**. F64 is the critical boundary (64 Capacity Units). SKUs below F64 require a Power BI PPU license for users to view reports; F64 and above allow free users to consume reports.
*   **Throttling:** Fabric uses a **smoothing** algorithm to average resource usage over a 24-hour window. If a capacity consistently overuses resources, the system enforces a strict, escalating penalty sequence:
    1.  **Overage Protection (<10 mins):** No penalty; borrowing from future capacity is allowed.
    2.  **Interactive Delay (10 mins - 1 hour):** Interactive requests (e.g., dashboard filtering) are delayed by ~20 seconds.
    3.  **Interactive Rejection (1 hour - 24 hours):** Rejects all interactive operations and dashboard rendering.
    4.  **Background Rejection (>24 hours):** Rejects all background processes, including scheduled refreshes, pipelines, and Spark runs.

### 2. Lakehouse Tables: Managed vs. External
*   **Managed Tables:** Stored in the default `/Tables` system folder of the Lakehouse as Delta tables. Dropping the table via Spark **permanently deletes both the schema metadata AND the physical data files** in OneLake.
*   **External Tables:** Stored in a custom location, such as the `/Files` folder or external cloud directories (S3, ADLS Gen2). Dropping the table **deletes only the schema metadata from the metastore**; the physical Parquet data files remain intact.
*   **No Table (Format-Only):** Running `.save("path")` writes data as Delta/Parquet files but does not register the table in the metastore. These files cannot be queried via SQL or Power BI until they are registered.

### 3. PySpark Data Processing Commands
*   **Ingesting CSV:** `df = spark.read.format("csv").option("header", "true").schema(custom_schema).load("Files/bronze/*.csv")`
*   **Column Manipulation:** `df.withColumnRenamed("OldCol", "NewCol").withColumn("Ratio", (col("Profit") / col("Sales")) * 100)`
*   **Aggregation:** `df.groupBy("ProductLine").agg(sum("Sales").alias("TotalSales"), avg("Profit").alias("AvgProfit"))`
*   **Conditional Formatting:** `df.withColumn("Size", when(col("Qty") > 10, "Large").otherwise("Small"))`
*   **Delta Write:** `df.write.format("delta").mode("overwrite").saveAsTable("my_table")`

### 4. Delta Time Travel & Historical Queries
*   **Spark SQL / T-SQL Syntax:**
    *   `SELECT * FROM employees VERSION AS OF 3;`
    *   `SELECT * FROM employees TIMESTAMP AS OF '2026-08-19T22:05:00Z';`
*   **PySpark Option Syntax:**
    *   `spark.read.format("delta").option("versionAsOf", 3).load("Tables/employees")`
    *   `spark.read.format("delta").option("timestampAsOf", "2026-08-19T22:05:00Z").load("Tables/employees")`
*   **Table Restore:** Permanently rolls the table state back to an earlier version. This is a metadata-only commit (V+1) that references existing Parquet files without duplicating them.
    *   `RESTORE TABLE employees TO VERSION AS OF 2;`

### 5. Medallion & Star Schema Architecture
*   **Bronze Layer:** Preserves unprocessed raw files in their native formats.
*   **Silver Layer:** Standardizes data, casting data types, removing duplicates, and eliminating statistical outliers.
*   **Gold Layer:** Models clean, analytical data in a **Star Schema** (one central Fact table surrounded by multiple Dimension tables).
*   **One-to-Many Relationships:** Star schema relationships must be defined as **one-to-many (`1:*`)** flowing from the Dimension table (`1`) to the Fact table (`*`).
*   **Assume Referential Integrity:** Tells Power BI it can use high-performance `INNER JOIN` operations instead of outer joins. The ETL pipeline must guarantee this integrity during data processing since Fabric does not enforce these constraints during table writes.
*   **Optimizations:**
    *   `OPTIMIZE`: Compacts small Parquet files (128 MB to 1 GB) to improve query scanning speeds.
    *   `V-Order`: Applies Microsoft's proprietary compression, sorting, and encoding algorithm to optimize files for all Fabric engines.
    *   `VACUUM`: Deletes outdated, unreferenced data files older than a retention threshold (7 days by default). This reduces storage costs but limits your time travel window.

### 6. Warehouse T-SQL Operations
*   **Capabilities:** Data warehouses support full T-SQL read/write capabilities (DQL, DML, DDL).
*   **`COPY INTO`:** The primary high-performance SQL command used for bulk loading. Use `FIRSTROW = 2` to skip header records, and use `ERRORFILE` to capture rejected rows.
*   **`CTAS` (Create Table As Select):** An optimized query that compiles a table schema and inserts data in parallel. Use a false filter `WHERE 1 = 0` to copy a table's structure as an empty clone.
*   **Explicit Transactions:** Data warehouses support transactional rollback handling using `BEGIN TRANSACTION` and standard `TRY...CATCH` blocks.

---

### 7. Structured Follow-up
This JSON block must be enclosed in a fenced code block with the language tag `suggested_questions`.
