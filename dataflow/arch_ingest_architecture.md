# High Level Architecture: Ingest Dataflow

## Overview
This pipeline implements a parallel Oracle to GCS Parquet extractor using Apache Beam. It is designed to handle large datasets efficiently by leveraging partitioning and intermediate stream buffering.

## Architecture Components

### 1. Schema Discovery (One-time)
Before processing data, the pipeline establishes a contract for the data structure:
*   **Mechanism**: The `GetSchema` DoFn executes a single query to `ALL_TAB_COLUMNS` to fetch metadata (column names, data types, precision, scale).
*   **Normalization**:
    *   Column names are sanitized (special characters like `$`, `#`, ` ` are replaced with `_`).
    *   The `PART` column is explicitly excluded from the final output schema.
    *   Oracle types are mapped to Polars types (e.g., `NUMBER` -> `pl.Int64`/`pl.Decimal`, `VARCHAR2` -> `pl.String`).
*   **Result**: A strict Polars schema is generated and passed as a side input to workers, ensuring consistency across all parallel operations.

### 2. Partition Discovery
To maximize parallelism, the pipeline dynamically determines how to split the workload:
*   **Logic**: The `DiscoverPartitions` DoFn executes `SELECT DISTINCT PART FROM source ORDER BY PART`.
*   **Handling**:
    *   **Partitioned Source**: Each distinct partition ID is yielded as a separate work item, triggering a separate Beam worker.
    *   **Non-Partitioned Source**: If no partitions are found, a single `None` work item is yielded, processing the table as a monolithic block.
*   **Purpose**: Distributes the heavy lifting of data extraction across multiple workers based on the natural partitioning of the source data.

### 3. Data Extraction (Oracle -> CSV Stream Buffer)
Data is not written directly to Parquet during the extraction phase. Instead, it flows through a CSV intermediate stage.

#### Why CSV before Parquet?
1.  **Memory Pressure & Buffering**: Writing Parquet files typically requires buffering row groups in memory to calculate page statistics and encodings. For large Oracle extracts, holding these buffers on workers while maintaining an open database cursor can lead to OOM (Out of Memory) errors.
2.  **Decoupling IO**: The CSV approach allows the pipeline to "drain" the Oracle cursor as fast as the network allows, streaming raw text bytes to GCS (`io.TextIOWrapper` wrapping the GCS file stream). This keeps the database connection active for the shortest necessary time.
3.  **Checkpointing**: Provides a tangible raw data state on GCS. If the conversion step fails, the raw extraction doesn't need to be re-run against the database.

*   **Process**:
    *   Queries `SELECT * FROM source WHERE PART = :id`.
    *   Fetches in chunks (cursor size: 80,000 rows).
    *   Streams directly to a GCS object using `csv.writer`.
    *   The `PART` column is stripped out on-the-fly during this write process.

### 4. Format Conversion (CSV -> Parquet)
Once the raw data is safely in GCS, it is converted to the final analytical format.
*   **Engine**: Uses `polars` for high-performance, vectorized processing.
*   **Method**: 
    *   `pl.scan_csv`: Lazily scans the temporary CSV file using the **pre-calculated schema** (from Step 1). This ensures types are enforced correctly without inferring them from the CSV text.
    *   `sink_parquet`: Streams the execution plan to a Parquet file with Snappy compression.
*   **Cleanup**: The temporary CSV file is deleted immediately after a successful conversion.

### 5. Operational Hooks
*   **Stored Procedures**: The pipeline wraps the execution with `START` and `END` procedure calls to the database, logging the pipeline's progress and the final row counts for auditability.
