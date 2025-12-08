---
id: 4a7e9b2c-8d1f-4e3a-9c5b-7a8d9e0f1a2b
aliases: []
tags:
  - opencode-generated
  - xcom
  - mapped-tasks
description: Accessing XCom values from specific mapped task instances using Jinja templating.
language: python
source: dags/silver_generic_dag.py
tech: airflow
title: XCom Pull in Mapped Tasks
type: pattern
---

# XCom Pull in Mapped Tasks

## Context & Problem
* **Problem:** Retrieving specific XCom data for the current mapped instance when working with dynamically mapped tasks (Airflow 2.3+).
* **Scope:** Covers accessing XCom values within Jinja templates for mapped tasks, specifically using `map_indexes`.

## Conceptual Solution
Standard `xcom_pull` retrieves values from the execution date. In mapped tasks, you often need the output of the *corresponding* upstream mapped instance. By using `ti.xcom_pull(..., map_indexes=ti.map_index)`, you align the retrieval with the current task's map index, ensuring 1:1 data flow between mapped steps.

## Implementation
```python
stage_new_data = GCSToGCSOperator(
    task_id="stage_new_data",
    source_bucket=bucket,
    # Use map_indexes=ti.map_index to pull the XCom specific to this mapped instance
    source_object="{{ ti.xcom_pull(task_ids='process_group.build_paths', map_indexes=ti.map_index)['source_object'] }}",
    destination_bucket=bucket,
    destination_object="{{ ti.xcom_pull(task_ids='process_group.build_paths', map_indexes=ti.map_index)['destination_object'] }}",
    move_object=False,
    replace=True,
)
```
