# Section 1: Databricks Intelligence

https://hilarious-bath-01d.notion.site/Section-1-Databricks-Intelligence-Platform-31741e81b4cb81b0b3e9d22b8dcdc88a

Weight - 10%

This section covers the fundamentals of Databricks platform, including how data is laid out for performance, the value proposition of the data intelligence platform and which compute types to choose frorm differrent use cases

## Data Layout & Query Performance Optimization

One of the biggest advantages of using Databricks is that it takes care of a lot of the heavy lifting when it comes to how your data is physically stored and queried. In traditional data warehouses and even early data lake setups, you had to manually think about things like partitioning strategies, file sizes, and indexing. With Delta Lake on Databricks, many of these concerns are handled automatically or with minimal configuration.

The exam wants you to understand which features exist to simplify data layout decisions and boost query performance. This includes things like **Liquid Clustering**, **Predictive Optimization**, **data skipping**, and **file compaction**. You do not need to memorize every internal detail, but you should know what each feature does, when it kicks in, and why it matters for performance.

At the Associate level, think of this section as understanding the "what" and "why" rather than deep implementation. Know what tools Databricks gives you so your data is laid out efficiently, and know how that translates into faster queries without you having to manually tune everything.

## Key Concepts

* Delta Lake and the Lakehouse Format

Delta Lake is the default storage format on Databricks. Every table you create in Unity Catalog is a Delta table by default. Delta Lake stores data as Parquet files plus a transaction log (the _delta_log folder). The transaction log tracks every change to the table, which enables features like ACID transactions, time travel, and schema enforcement. The Parquet files themselves store the actual data in a columnar format, which is great for analytical queries because the engine only reads the columns it needs.

* Liquid Clustering

Liquid Clustering is the modern replacement for both partitioning and Z-ordering. Instead of you deciding upfront how to partition your data (and being stuck with that choice), Liquid Clustering automatically reorganizes data files based on the clustering keys you specify. The big win is that you can change your clustering keys later without rewriting the entire table. It also works incrementally, meaning it optimizes data as new data arrives rather than requiring a full rewrite.

You enable Liquid Clustering when you create or alter a table using `CLUSTER BY`. For example: `CREATE TABLE sales CLUSTER BY (region, date)`. Databricks then handles the physical layout automatically. This is the recommended approach for new tables going forward.

* Predictive Optimization

Predictive Optimization is a Databricks feature that automatically runs maintenance operations on your Delta tables. When enabled, Databricks monitors your tables and automatically triggers OPTIMIZE (file compaction) and VACUUM (removing old files) when it determines your tables would benefit from it. You do not need to set up scheduled jobs to run these commands manually. It is enabled at the catalog or schema level in Unity Catalog, and it only works with Unity Catalog managed tables.

* Data Skipping

Data skipping is a built in optimization in Delta Lake. For every data file, Delta Lake stores min/max statistics for the first 32 columns in the transaction log. When you run a query with a filter (a WHERE clause), the engine checks these statistics and skips entire files that cannot contain matching rows. You do not need to enable this manually. It happens automatically. Liquid Clustering makes data skipping even more effective because it groups similar values together in the same files.

* File Compaction (OPTIMIZE)

Over time, as you write data to a Delta table (especially with streaming or frequent small batch writes), you can end up with many small files. This is called the "small file problem" and it hurts read performance because the engine has to open and process each file individually. The OPTIMIZE command compacts these small files into larger, more efficient files (target size is typically 1 GB for non clustered tables). With Predictive Optimization enabled, this happens automatically. You can also trigger it manually with OPTIMIZE table_name.

* Deletion Vectors

Deletion vectors are a performance optimization for DELETE, UPDATE, and MERGE operations. Instead of rewriting entire data files to remove or update a few rows, Databricks marks the affected rows in a separate deletion vector file. The actual data files remain untouched. This makes write operations much faster. On reads, the engine uses the deletion vector to filter out the marked rows. Deletion vectors are enabled by default on new Delta tables in Databricks.

## Code Examples

### Creating a Table with Liqui Clustering

```SQL
-- Create a new table with Liquid Clustering
CREATE TABLE catalog.schema.sales (
    sale_id BIGINT,
    sale_date DATE,
    region STRING,
    product_id INT,
    amount DECIMAL(10,2)
) 
CLUSTER BY (region, sale_date);

-- Change clustering keys on an existing table (no rewrite needed)
ALTER TABLE catalog.schema.sales CLUSTER BY (proudct_id, sale_date);

-- Remove clustering entirely
ALTER TABLE catalog.schema.sales CLUSTER BY NONE;

```

Question:

* Should we always Cluster? How do we identify which key to choose to cluster?
* When should remove a key from a cluster?

### Running OPTIMIZE Manually

```SQL
-- Compact small files into larger ones
OPTIMIZE catalog.schema.sales;

-- VACUUM removes old files no longer refernced by the transaction log
-- Default retention is 7 days
VACUUM catalog.schema.sales;

-- Check table details incluing file count and size
DESCRIBE DETAIL catalog.schema.sales;
```

Question:

* Typically for a data engineer, when are tell taling signs we would run OPTIMIZE manually?
* For VACUUM - when we remove old files, do we lose the data? Is archiving a safer approach if possible?

