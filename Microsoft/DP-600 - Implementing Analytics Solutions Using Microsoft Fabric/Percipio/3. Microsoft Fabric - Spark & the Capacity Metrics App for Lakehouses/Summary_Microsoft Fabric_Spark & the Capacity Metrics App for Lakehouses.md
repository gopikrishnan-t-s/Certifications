## 1. Apache Spark on Fabric
*   **Spark Pool:** The Fabric equivalent of a Spark cluster. Contains a head node (runs the driver program) and worker nodes (run executor processes).
*   **Starter Pools:** Pre-configured, pre-allocated Spark clusters designed for fast setup and lightweight workloads. They eliminate cluster startup delays.
*   **High Concurrency Mode:** Allows dynamic resource sharing across multiple simultaneous users and workloads, vastly improving initial session startup speeds in shared environments.
*   **Spark History Server:** UI used for debugging and optimizing jobs. It tracks execution logs, stages, tasks, memory/CPU usage, and helps identify performance bottlenecks or failed jobs.
*   **Lakehouse Association:** To run Spark SQL queries or read Delta tables, a PySpark notebook **must** have a Lakehouse explicitly attached/associated with it.
---
## 2. T-SQL Endpoint vs. Spark
*   **T-SQL Endpoint (Lakehouse):** Provides a SQL-based interface to query data. It is **strictly read-only** for lakehouses (it cannot be used for INSERTS, UPDATES, or DELETES). It only queries the "Tables" section of the Lakehouse (structured data).
*   **T-SQL Endpoint (Warehouse):** Unlike the Lakehouse, the T-SQL endpoint in a Data Warehouse is read-write.
*   **Spark:** Used for read and write operations. It can query both the "Tables" (managed Delta tables) and "Files" (unstructured/semi-structured data) sections.
---
## 3. Fabric Shortcuts
Shortcuts are symbolic links that reference data in external sources or other lakehouses without moving or duplicating the data. They dynamically reflect changes made at the source.
*   **ADLS Gen2 Shortcuts:** 
    *   **Requirement:** The Azure Storage Account **must** have "Hierarchical namespaces" enabled.
    *   **Authentication:** Commonly uses a Shared Access Signature (SAS) token. 
    *   **Principle of Least Privilege:** When generating the SAS token, grant **only "Read" and "List"** permissions.
*   **Amazon S3 Shortcuts:** Requires an IAM policy (with Read/List permissions) and an IAM User with an Access Key ID and Secret Access Key.
*   **Dynamic Nature:** If a file on ADLS/S3 is updated, the shortcut in Fabric reflects the change immediately. However, if a *managed Delta table* was built from that shortcut, the table must be manually refreshed/overwritten to reflect new row counts or data.
---
## 4. PySpark & Spark SQL Code Snippets
Expect code-completion questions on the exam. Memorize these syntaxes:

*   **Loading Data:**
    *   `df = spark.table("table_name")`
    *   `df = spark.read.format("csv").option("header","true").load("Files/path/file.csv")`
*   **Filtering (Logical AND/OR):**
    *   `df.where((col("Profit") > 2000) & (col("Segment") == 'Corporate'))` (Use single `&` for AND, `|` for OR).
*   **Adding/Renaming Columns:**
    *   Rename: `df.withColumnRenamed("oldName", "newName")`
    *   Calculated: `df.withColumn("ProfitMargin", round((col("Profit") / col("Sales")) * 100))`
    *   Conditional: `df.withColumn("Size", when(col("Sales") < 300, "Small").otherwise("Large"))`
*   **Grouping and Aggregation:**
    *   `df.groupBy("Segment").agg(sum("Sales").alias("TotalSales"), avg("Profit").alias("AvgProfit"))`
*   **Writing Data:**
    *   To Parquet/JSON: `df.write.mode("overwrite").parquet("Files/path")`
    *   To Managed Delta Table: `df.write.format("delta").saveAsTable("table_name")`
*   **Cell Magics:**
    *   `%%sql` converts a notebook cell to run pure Spark SQL.
---
## 5. Fabric Capacity Metrics App & Throttling
The Capacity Metrics App (installed via Microsoft AppSource) monitors Capacity Unit (CU) usage, storage, and overages. You must supply your CapacityID (found in the Admin Portal URL) to configure it.
*   **SKUs (Stock Keeping Units):** Defines your capacity size (e.g., F64 means 64 maximum Capacity Units).
*   **Overages:** When workloads consume more CUs than allocated, Fabric "borrows" from future capacity. 
*   **Throttling Sequence (Exam Highly Testable):** If overages persist, Fabric penalizes the tenant in specific escalating stages:
    1.  **Overage Protection (< 10 minutes):** Jobs can borrow future capacity without penalty.
    2.  **Interactive Delay (10 - 60 minutes):** Fabric introduces delays (e.g., ~20 seconds) to interactive operations (like report rendering).
    3.  **Interactive Rejection (1 - 24 hours):** All interactive user queries and report interactions are denied/rejected outright.
    4.  **Background Rejection (> 24 hours):** The most extreme measure. Even scheduled background jobs and data refreshes are rejected until CU usage normalizes.