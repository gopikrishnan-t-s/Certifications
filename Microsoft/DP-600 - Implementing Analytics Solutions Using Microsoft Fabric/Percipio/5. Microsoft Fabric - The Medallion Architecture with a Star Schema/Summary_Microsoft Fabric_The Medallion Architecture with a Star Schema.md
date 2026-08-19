As you consolidate your knowledge for the DP-600 exam, this guide provides a highly structured, scannable study resource covering the **Medallion Architecture**, **Star Schema implementation in Fabric**, and **Delta Table performance tuning**.

---

### 1. The Medallion Architecture in Fabric
The Medallion Architecture is a progressive data-refinement pattern that structures data as it moves through raw, validated, and business-ready states inside OneLake.

```
+-------------------+      Data Cleaning      +------------------+      Dimensional      +----------------+
|   BRONZE LAYER    |      Deduplication      |   SILVER LAYER   |       Modeling        |   GOLD LAYER   |
| (Raw Landing/Files)=====> Outlier Removal =====>(Cleaned/Structured)=====> Fact/Dims Schema =====>(Business-Ready)
|   CSV/JSON/Native |      Enrichments        |   Delta Tables   |      Aggregations      |  Star Schema   |
+-------------------+                         +------------------+                        +----------------+
```

#### The Bronze (Raw) Layer
*   **Purpose:** Serves as the landing zone for all ingestion types (structured, semi-structured, or unstructured).
*   **Format:** Preserves original native formats (e.g., CSV, JSON, XML) to act as a historical and auditable source of record.
*   **Fabric Implementation:** Typically stored within the `/Files` section of a Lakehouse or surfaced dynamically using OneLake shortcuts to ADLS Gen2, AWS S3, or Google Cloud Storage.

#### The Silver (Validated) Layer
*   **Purpose:** Houses cleaned, standardized, structured, and enriched data.
*   **Common Transformations:** 
    *   Structural type casting (e.g., parsing string dates into uniform `DateType`).
    *   De-duplication and dropping entirely null records.
    *   Statistical outlier removal (e.g., using interquartile range `IQR = Q3 - Q1`).
*   **Fabric Implementation:** Stored as **managed Delta tables** within OneLake to leverage ACID transactions and transactional version history.

#### The Gold (Enriched/Curated) Layer
*   **Purpose:** Deliver business-ready, aggregated data optimized for executive dashboards, operational reporting, and self-service analytics.
*   **Design Pattern:** Structured around a **Star Schema** (Fact and Dimension tables).
*   **Fabric Implementation:** Managed tables that back Direct Lake semantic models for real-time, low-latency Power BI reporting.

---

### 2. Implementing Upserts & Incremental Loads (Bronze to Silver)
To prevent processing duplicate data, the Medallion architecture relies on incremental loads using Delta Lake's **merge and upsert** capabilities.

#### Ingesting with Wildcards (Bronze)
To ensure the pipeline dynamically captures new files without changing the code, load data using folder-level wildcard matches:
```python
# Loads all current and newly added CSVs in the bronze folder
superstore_df = spark.read.format("csv") \
    .option("header", "true") \
    .schema(order_schema) \
    .load("Files/bronze/*.csv")
```

#### Merging and Upserting into Silver
Rather than overwriting the Silver table, use the `merge` API to update matching records or insert new ones:
```python
from delta.tables import *

# Initialize the target Silver table
deltaTable = DeltaTable.forPath(spark, 'Tables/global_superstore_silver')
dfUpdates = superstore_df  # DataFrame with new batch data

# Align target (silver) and source (updates) using a multi-key business predicate
deltaTable.alias('silver') \
    .merge(
        dfUpdates.alias('updates'),
        '''
        silver.OrderID = updates.OrderID AND 
        silver.OrderDate = updates.OrderDate AND 
        silver.CustomerID = updates.CustomerID AND 
        silver.ProductID = updates.ProductID
        '''
    ) \
    .whenMatchedUpdate(set={
        # Map values if updates are needed on match, or leave blank to skip
    }) \
    .whenNotMatchedInsert(values={
        "OrderID": "updates.OrderID",
        "OrderDate": "updates.OrderDate",
        "CustomerID": "updates.CustomerID",
        "ProductID": "updates.ProductID",
        "Sales": "updates.Sales",
        "Profit": "updates.Profit"
    }) \
    .execute()
```

---

### 3. Dimensional Modeling in the Gold Layer

#### Facts vs. Dimensions
*   **Fact Tables:** Contain numerical metrics/aggregates (e.g., `Sales`, `Quantity`, `Profit`, `Discount`) and foreign keys that map to dimensions.
*   **Dimension Tables:** Contain descriptive, categorical context (e.g., customers, geography, product lines). They **must** be deduplicated on their primary key to ensure clean joins:
    ```python
    # Ensure customer dimensions contain unique primary keys
    dim_customers_df = dim_customers_df.drop_duplicates(subset=["CustomerID"])
    ```

#### Semantic Model Cardinality & Relationships
*   **Cardinality:** Relationships in a Star Schema are strictly **one-to-many (`1:*`)** from the Dimension table to the Fact table.
*   **Filter Propagation:** Typically set to **Single Direction** (filters flow from the Dimension table to slice the Fact table, but not vice-versa).
*   **Active vs. Inactive:** Active relationships are used by default in DAX queries and Power BI visuals.
*   **Referential Integrity ("Assume Referential Integrity"):**
    *   *Warning:* Microsoft Fabric (like Snowflake) **does not enforce** foreign key constraints during table writes.
    *   *Syllabus Concept:* Checking "Assume Referential Integrity" tells the Power BI engine it can safely perform highly optimized `INNER JOIN` operations instead of outer joins, speeding up report queries. However, you must guarantee this integrity beforehand in your ETL/Spark processing code.

---

### 4. Partitioning & Table Maintenance

Over time, tables accumulate small files or historical remnants that slow down reads. Fabric offers specialized optimizations to maintain peak performance:

#### Table Partitioning
Partitioning physically separates table data into sub-directories in storage based on a low-cardinality column (e.g., Year, Region, or State).
```python
# Partitioning code structure
df.write.format("delta") \
    .partitionBy("State") \
    .mode("overwrite") \
    .saveAsTable("CustomersPartitioned")
```
*   *Optimization Mechanism:* This enables **partition pruning**. If a query specifies `WHERE State = 'NY'`, the engine bypasses other directories entirely and only scans the `State=NY` folder, reducing overall I/O.

#### The Table Maintenance Suite

- **`OPTIMIZE` (Compaction):**
  - **What it does:** Merges numerous fragmented, small Parquet files (often generated by streaming or frequent batch inserts) into uniform, large Parquet files (typically target sizes between 128 MB and 1 GB).
  - **Benefit:** Significantly improves file scan performance and metadata handling.

- **`V-Order` (Microsoft Proprietary):**
  - **What it does:** Applies an advanced sorting, encoding, and compression algorithm to Delta Parquet files during an `OPTIMIZE` or write execution.
  - **Benefit:** Maximizes read speeds across all Fabric query engines (Power BI Direct Lake, SQL Analytics Endpoint, and Spark).

- **`VACUUM` (Cleanup):**
  - **What it does:** Permanently deletes raw data files (Parquet) that are no longer referenced in the active transaction log (e.g., files remaining from older time-travel versions).
  - **Rule of Thumb:** By default, files are only deleted if they are older than the retention threshold (**7 days**). Reducing this threshold saves storage costs but limits your maximum time-travel window.