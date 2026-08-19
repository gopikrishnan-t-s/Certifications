### 1. Spark Batch Jobs & Python Scripting

#### The Development-to-Production Pipeline
During the development phase of a data engineering pipeline within Microsoft Fabric, engineers typically use interactive **Spark Notebooks** (`.ipynb` files). This environment allows for immediate execution, visualization, and rapid debugging of code. 

Once the ETL (Extract, Transform, Load) logic is stabilized, the notebook is migrated to a production-grade batch workflow. This requires converting the notebook into a flat Python script (`.py` file) and executing it via a **Spark Job Definition (SJD)**. This shift from interactive to batch mode introduces critical changes in session initialization, code structures, and logging destinations.

#### Explicit Session Instantiation
In an interactive notebook, the Spark infrastructure is pre-warmed, and a global `SparkSession` instance named `spark` is automatically injected into the runtime environment. 

In a batch Python script, however, no such implicit injection occurs. Because the script is executed directly by the cluster’s driver via the command line, you must explicitly construct the `SparkSession` and set up the driver's runtime environment. 

```python
# Mandatory import statements for batch execution
from pyspark.sql import SparkSession
from pyspark.conf import SparkConf

if __name__ == "__main__":
    # Programmatic construction of the SparkSession
    spark = SparkSession.builder \
        .appName("Compute India Sales by City") \
        .getOrCreate()
    
    # Accessing the Spark Context for low-level configuration
    spark_context = spark.sparkContext
    
    # Explicitly configuring log output verbosity
    spark_context.setLogLevel("DEBUG")
```

#### The `if __name__ == "__main__":` Guard
This Python mechanism acts as a gatekeeper. It ensures that the execution block only runs when the script is invoked directly as a program (e.g., by the cluster manager via Apache Livy). If the file is imported as a module by another Python script, this block is bypassed, preventing unintended execution.

#### Display vs. Print Commands in Batch Runs
*   **`display(df)`:** This command is optimized for interactive environments. It compiles a rich, paginated graphical grid on the client side. In a batch Python script, `display()` has no graphical front-end to render to and will throw errors or behave as a no-op. It must be stripped out of production `.py` files.
*   **`print()`:** Regular Python `print()` statements are safe for batch scripts. They write directly to the standard output stream of the Spark driver process.

#### Logging Streams in Spark Job Definitions
When troubleshooting batch jobs inside Fabric, logs are split across three primary streams:
1.  **Driver (`stdout`):** This stream captures standard programmatic outputs. Any plain Python `print()` statements executed in your code will write directly to `Driver stdout` under the `Latest stdout` or `All stdout` views.
2.  **Driver (`stderr`):** Contrary to what its name suggests, `stderr` is not reserved exclusively for fatal errors. In Spark, the driver writes verbose diagnostic and runtime information to `stderr` (such as transaction states, task distributions, and configuration confirmations). If a job fails, the detailed stack trace and specific Java/Scala exceptions (e.g., `AnalysisException` indicating `TABLE_OR_VIEW_ALREADY_EXISTS`) will be recorded here.
3.  **Prelaunch (`stdout`):** This stream records events *before* the Python driver script actually runs. It documents the initialization of the Spark context, virtual machine allocation, Livy session handshakes, and library resolution.

---

### 2. Custom Spark Pools & Environments

#### Spark Pools: Compute Architecture
In Microsoft Fabric, the term **Spark Pool** represents the underlying physical or virtual cluster. These pools consist of:
*   **Head Node:** Runs the Driver program, which orchestrates the execution plan, splits Spark jobs into physical stages, schedules tasks, and coordinates with the resource manager (YARN).
*   **Worker Nodes:** Host the Executor processes, which are responsible for executing individual tasks (computations, joins, filtering) and storing data partitions in memory or on disk.

```
                  +-----------------------------------+
                  |             HEAD NODE             |
                  |  +------------------------------+  |
                  |  |        Driver Program        |  |
                  |  +------------------------------+  |
                  +-----------------+-----------------+
                                    |
            +-----------------------+-----------------------+
            |                                               |
+-----------v-----------------------+           +-----------v-----------------------+
|            WORKER NODE            |           |            WORKER NODE            |
|  +------------------------------+  |           |  +------------------------------+  |
|  |       Executor Process       |  |           |  |       Executor Process       |  |
|  |  [Task 1] [Task 2] [Task 3]  |  |           |  |  [Task 1] [Task 2] [Task 3]  |  |
|  +------------------------------+  |           |  +------------------------------+  |
+-----------------------------------+           +-----------------------------------+
```

#### Starter Pools vs. Custom Pools
*   **Starter Pools:** Fabric provides pre-configured, instantly available compute clusters called Starter Pools. Because these VMs are kept on warm standby, session startup times are minimal (typically under 10 seconds). However, you cannot customize the hardware family or size of starter pools. They automatically configure with standard memory-optimized medium nodes and default to scaling between 1 and 10 nodes.
*   **Custom Pools:** For highly resource-intensive or predictable production workloads, you can define custom Spark pools. Custom pools allow you to configure:
    *   **Node Family:** Such as *Memory Optimized* (designed for high-performance in-memory processing, joins, caching, and ML), *Compute Optimized*, *General Purpose*, or *Storage Optimized*.
    *   **Node Size:** Scaled from Small to Extra Large, depending on the memory and CPU cores required.
    *   **Compute Configuration Customization:** Allocates dedicated node sizes specifically for certain workspaces, lakehouses, or notebooks.

#### Scaling Mechanics: Autoscaling vs. Dynamic Executor Allocation
*   **Autoscale:** Scales compute capacity at the **infrastructure (VM) layer**. Based on the overall volume of tasks, the pool will spin up additional worker nodes (up to your specified maximum) or spin them down during idle periods.
*   **Dynamically Allocate Executors:** Scales compute capacity at the **process (Spark runtime) layer**. It does not provision more VMs. Instead, it dynamically adjusts the number of Executor processes running *within* the currently active nodes based on task queue pressure. This improves internal node efficiency and prevents idle executors from wasting memory.

#### High Concurrency Session Sharing
Normally, each Spark notebook initiates its own isolated, dedicated Spark session on the pool, consuming discrete compute resources and requiring a warm-up period. 

When **High Concurrency Mode** is enabled, multiple distinct notebooks can attach to the **same active Spark session** (up to 5 notebooks per session). 

*   **Sharing:** The notebooks share the warm driver and executors, allowing subsequent notebooks to bypass session initialization entirely.
*   **Fast Spin-up:** A new notebook joining an active high-concurrency session can spin up in under 5 seconds, rather than requiring the typical 10-to-30 second standard startup time.

#### Environments
An **Environment** in Fabric is a logical boundary that wraps your Spark configurations and library dependencies together:
*   **Spark Compute Settings:** Restricts and defines maximum driver cores (4, 8, 16), driver memory (28GB, 56GB, 112GB), executor cores, and executor memory allocated to notebooks using the environment.
*   **Public Libraries:** Lets you declare PyPI or Conda packages (e.g., `pandas`, `numpy`) at specific versions to be deployed cluster-wide when the environment is loaded.
*   **Custom Libraries:** Lets you upload custom library packages directly to the cluster nodes. Supported formats are Java/Scala packages (`.jar`), packaged Python wheels (`.whl`), or compressed tarballs (`.tar.gz`).
*   **Spark Properties:** Customizes system-level Spark variables (e.g., execution timeouts, shuffle partition sizes).
*   **Resources:** A dedicated space to upload static configuration files, keys, or reference assets, making them accessible to any notebook attached to the environment without having to mount external lakehouses.

---

### 3. Delta Table Internals & Time Travel

#### Underlying Storage Architecture
A Delta table is not a traditional relational database table. It is an abstract, transactional database layer built on top of raw file storage within **OneLake**. A Delta table consists of:
*   **The Directory:** The folder named after the table (e.g., `/Tables/employees`).
*   **The Delta Log (`_delta_log`):** A mandatory subfolder that functions as the transaction log. It records every change (write, update, delete, schema modification) as a sequential, zero-padded JSON commit file (starting with `00000000000000000000.json`, then `000001.json`, etc.). These JSON logs are the source of truth for ACID transactions, keeping a chronological record of files added or removed.
*   **The Data Partitions:** Compressed, columnar **Apache Parquet** files (e.g., `part-0000...snappy.parquet`). These files store the actual records. When you write data, new Parquet files are generated rather than editing old ones.

```
/Tables/employees/
├── _delta_log/
│   ├── 00000000000000000000.json  <-- Version 0 (CREATE TABLE)
│   ├── 00000000000000000001.json  <-- Version 1 (INSERT)
│   └── 00000000000000000002.json  <-- Version 2 (UPDATE)
├── part-00000-abcd-c000.snappy.parquet  <-- Data for Version 1
└── part-00001-efgh-c000.snappy.parquet  <-- Data for Version 2
```

#### Auditing with `DESCRIBE HISTORY`
Running `DESCRIBE HISTORY <table_name>` parses the `_delta_log` and outputs the table’s audit trail. Each row in the output represents a distinct commit:
*   **`version`:** An autoincrementing integer starting at `0`.
*   **`timestamp`:** The exact UTC date and time the commit was finalized.
*   **`operation`:** The engine command that executed (e.g., `CREATE TABLE AS SELECT`, `WRITE`, `UPDATE`, `DELETE`, `RESTORE`).
*   **`operationParameters`:** The internal variables of the commit (such as predicates used in a `DELETE` or `UPDATE` query).
*   **`isolationLevel`:** The transactional security level (by default, Delta Lake utilizes `Serializable` isolation—the most rigorous standard, ensuring zero dirty or non-repeatable reads in concurrent environments).

#### Time Travel Operations
Because Delta Lake never mutates data in place (it writes new Parquet files and updates the active file list in the transaction log), you can query older states of your data by targeting specific versions or timestamps.

##### Spark SQL Time Travel
*   **By Timestamp:** Returns data as it existed at a specific point in time. 
    ```sql
    SELECT * FROM employees TIMESTAMP AS OF '2024-09-15T11:59:18Z'
    ```
*   **By Version:** Returns data associated with a specific transaction index.
    ```sql
    SELECT * FROM employees VERSION AS OF 2
    ```

##### PySpark Time Travel
*   **By Timestamp:**
    ```python
    delta_path = "Tables/employees"
    df = spark.read.format("delta") \
        .option("timestampAsOf", "2024-09-16T07:58:40Z") \
        .load(delta_path)
    ```
*   **By Version:**
    ```python
    df = spark.read.format("delta") \
        .option("versionAsOf", 2) \
        .load(delta_path)
    ```

#### Restoring Tables via Time Travel
To permanently rollback the active state of a table to an earlier point in its history, use the `RESTORE` command:

```sql
RESTORE employees TO VERSION AS OF 2
```

When this command is run, the engine executes a specific set of operations:
*   **New Transaction Log:** It does not erase the history between version 2 and the present. Instead, it writes a **new transaction log entry** (e.g., `000005.json`) with the operation labeled as `RESTORE`.
*   **No Data Duplication:** It does not physically copy or duplicate the old Parquet files of version 2. Instead, the newly generated JSON transaction log references the existing Parquet files associated with version 2 as the active data set, ensuring the operation is metadata-only and highly efficient.
*   **Auditability:** Because history is strictly additive, you retain full time-travel capability. Even after restoring to version 2, you can still query version 4 (the state of the table immediately before the restore occurred).

---

### 4. Managed vs. External Delta Tables

The structural differences between managed and external tables are defined by where their files live and how they handle deletions. Both run on the same Delta Lake technology, but they have distinct ownership boundaries:

#### Managed Delta Tables
*   **Data Control:** Fabric takes complete ownership of both the table’s metadata and its physical storage.
*   **Location:** The physical Parquet files and `_delta_log` folders are stored in the default workspace tables directory (always inside the `/Tables` system directory of the Lakehouse).
*   **Syntax:**
    ```python
    df.write.format("delta").saveAsTable("phone_sales")
    ```
*   **Deletion Behavior (`DROP TABLE`):** When you run a `DROP TABLE phone_sales` query, **Fabric deletes both the table schema registration in the metastore and all physical data files** in the `/Tables/phone_sales` directory on OneLake. The data is unrecoverable.

#### External Delta Tables
*   **Data Control:** Fabric manages only the schema registration and pointer definition in the metastore. The user retains complete ownership of the physical storage location.
*   **Location:** The user explicitly defines the path. This can point to the `/Files` section of the local lakehouse, or to outside cloud providers like ADLS Gen2, AWS S3, or Google Cloud Storage.
*   **Syntax:**
    ```python
    # Defining a specific path converts this automatically to an External Table
    df.write.format("delta").saveAsTable("phone_sales_external", path="Files/external")
    ```
*   **Deletion Behavior (`DROP TABLE`):** When you run `DROP TABLE phone_sales_external`, **Fabric deletes only the schema metadata from the metastore**. The actual physical Parquet data files, the directories, and the internal `_delta_log` folder at `Files/external` remain untouched and intact. You can easily re-register this data as a table later.

#### Saving as Delta Format (No Table Registration)
You can write DataFrames out in Delta format without registering them as tables in the Fabric catalog at all:

```python
df.write.format("delta").save("Files/delta/phone_sales")
```

This saves the snappy Parquet files and the transaction log folder (`_delta_log`) directly to OneLake. However, because it is not registered in the metastore, the SQL analytics endpoint, visual queries, and Power BI cannot discover or query it until it is registered as an external table.

#### Summary Matrix for Exam Review

| Metric / Feature | Managed Delta Tables | External Delta Tables |
| :--- | :--- | :--- |
| **Write Syntax** | `.saveAsTable("name")` | `.saveAsTable("name", path="...")` |
| **Physical Storage Path** | Always inside `/Tables` directory | User-defined (`/Files`, ADLS, S3, GCS) |
| **Schema Metastore Owner** | Fabric | Fabric |
| **Data File Owner** | Fabric | User / External System |
| **Drop Table Result** | Deletes schema **AND** physical data | Deletes schema **ONLY** (data persists) |
| **View Files in Lakehouse UI** | Supported directly via table menu | Grayed out (must browse folder path manually) |