# **Microsoft Fabric: Getting Started and the DP-600 Exam**
---
### 1. Microsoft Fabric Core Components
Microsoft Fabric unifies previously separate Azure data services into a single ecosystem. You need to understand the distinct role of each component:
*   **OneLake:** The centralized, unified storage foundation for all data types across the organization (structured, semi-structured, and unstructured). 
*   **Lakehouses:** A hybrid architecture that combines the flexibility of data lakes (for diverse data types) with the structure and management of traditional data warehouses.
*   **Warehouses:** Optimized specifically for structured, relational datasets and traditional business analytics using **T-SQL**.
*   **Data Factory:** Your toolset for data ingestion, transformation, and orchestration, handling both ETL/ELT pipelines and real-time streaming.
*   **Semantic Models:** User-friendly data structures that organize complex datasets, making them ready for interactive reporting in Power BI.
*   **Delta Lake:** An open-source storage format running on OneLake that provides **ACID transactions, time travel, and schema enforcement**.
*   **KQL Databases:** Uses Kusto Query Language (KQL) and is optimized for handling log and time-series data for real-time analytics.

---

### 2. DP-600 Exam Breakdown (Updated Nov 2024)
The exam evaluates your ability to design, deploy, and manage enterprise-scale analytics. 

| Exam Domain | Weight | Key Focus Areas |
|---|---|---|
| **Preparing Data** | 45-50% | Data ingestion, transformation (filtering, merging, joining), resolving data quality issues, implementing star schemas, and querying via SQL/KQL. |
| **Maintaining Solutions** | 25-30% | Applying row/column/workspace security, lifecycle management (deployment pipelines, version control), optimization, and dependency management. |
| **Semantic Models** | 25-30% | Designing star schemas, DAX optimizations, advanced DAX calculations (iterators, windowing), incremental refresh, and XMLA endpoint deployment. |

---

### 3. Exam Logistics & Testing Strategies
*   **Format & Scoring:** You have roughly **120 minutes** to complete multiple-choice and multiple-response questions. You need a score of **700/1000** to pass.
*   **Open Book Resource:** You are permitted to access **Microsoft Learn (MS Learn)** during the test. However, accessing personal notes or other websites will result in immediate revocation.
*   **Testing Tips:** Don't get stuck on one question; use the **"mark for review"** feature. Eliminate obviously wrong answers, read carefully before deciding, and generally trust your first instinct rather than second-guessing.

---

### 4. Hands-on Practice Requirement
To pass, you need practical experience. You can sign up for a **60-day Fabric free trial** at `app.fabric.microsoft.com`. 
*   **Key Constraint:** The trial requires an **organizational email address**; personal Microsoft accounts are not supported.