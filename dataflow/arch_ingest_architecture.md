---
id: d4e5f6a7-b8c9-0123-4567-89abcdef0123
created: 2025-12-08
aliases: []
tags:
  - opencode-generated
  - beam
  - oracle
  - parquet
description: Parallel Oracle to GCS Parquet extractor pipeline using Apache Beam and Polars for memory-efficient processing.
language: python
tech: dataflow
title: Parallel Oracle Ingest Architecture
type: arch
---

# Parallel Oracle Ingest Architecture

> [!INFO] Context
> **Problem**: Extracting large datasets from Oracle to Parquet often causes OOM errors due to row-group buffering and long-held database cursors.
> **Scope**: Covers schema discovery, partition-based splitting, intermediate CSV buffering, and final Parquet conversion.

## Conceptual Solution
This pipeline decouples extraction from formatting. It uses **Schema Discovery** to enforce strict types, **Partition Discovery** to parallelize reading, and an intermediate **CSV Buffer** on GCS to "drain" the database quickly. Final conversion to Parquet is handled by **Polars** (vectorized) using the pre-fetched schema, ensuring type safety and memory efficiency.

## Architecture Components

### 1. Schema Discovery (One-time)
*   **Mechanism**: Queries `ALL_TAB_COLUMNS` to fetch metadata (names, types, precision).
*   **Normalization**: Sanitizes column names (removes `$`, `#`) and maps Oracle types to Polars types (e.g., `NUMBER` -> `pl.Int64`).
*   **Result**: A strict Polars schema passed as a side input to workers.

### 2. Partition Discovery
*   **Logic**: `SELECT DISTINCT PART FROM source` to determine splits.
*   **Handling**: Yields partition IDs for parallel workers. If no partitions, yields a single `None` item.

### 3. Data Extraction (Oracle -> CSV Stream)
*   **Why CSV?**:
    1.  **Memory**: Avoids Parquet row-group buffering in RAM.
    2.  **Speed**: Streams raw text to GCS, releasing the DB cursor faster.
    3.  **Checkpoint**: Provides a raw state on GCS for recovery.
*   **Process**: Chunks of 80k rows are streamed to GCS via `csv.writer`. The `PART` column is stripped.

### 4. Format Conversion (CSV -> Parquet)
*   **Engine**: Polars (`pl.scan_csv`, `sink_parquet`).
*   **Method**: Lazily scans CSV using the **pre-calculated schema** from Step 1.
*   **Cleanup**: Deletes temporary CSVs after successful conversion.

### 5. Operational Hooks
*   Wraps execution with `START`/`END` stored procedures for auditing.

> [!TIP] Key Takeaways
> * **Decoupling IO**: Writing to CSV first prevents OOMs by avoiding Parquet buffering during the DB fetch.
> * **Schema Contract**: Fetching schema upfront ensures type consistency across distributed workers.
> * **Polars Integration**: Using Polars for the final conversion provides high-performance, vectorized writing.
