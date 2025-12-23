---
id: 550e8400-e29b-41d4-a716-446655440000
created: 2025-12-08
aliases: []
tags:
  - opencode-generated
  - task-group
  - dynamic-mapping
description: Explains how to use @task_group for hierarchical task organization and dynamic mapping in Airflow.
language: python
tech: airflow
title: Dynamic Task Groups
type: pattern
---

# Dynamic Task Groups

> [!INFO] Context
> **Problem**: Complex DAGs with repeating patterns become cluttered. Processing a dynamic list of inputs requires creating tasks at runtime.
> **Scope**: Covers `@task_group`, `.expand_kwargs()` for dynamic mapping, and resource management with Airflow Pools.

## Conceptual Solution
The `@task_group` decorator treats a collection of tasks as a hierarchical unit. Combining this with Dynamic Task Mapping (`.expand_kwargs()`) fans out the entire group for each input item. Assigning heavy tasks to an Airflow `pool` prevents cluster overload during high concurrency.

## Implementation

```python
import logging
from typing import List, Dict, Any
from airflow.decorators import task, task_group
from airflow.providers.google.cloud.operators.dataflow import DataflowStartFlexTemplateOperator
from airflow.operators.python import PythonOperator

# Generalized constants
DATAFLOW_POOL = "my_custom_pool"
GCP_PROJECT_ID = "my-gcp-project-id"
GCP_REGION = "europe-west1"

@task_group(group_id="process_jobs")
def process_jobs_group(body: Dict[str, Any], log_info: Dict[str, Any]):
    """TaskGroup for running and logging a single Dataflow job."""
    
    # Task 1: Run Dataflow
    run_dataflow_job = DataflowStartFlexTemplateOperator(
        task_id="run_dataflow_job",
        project_id=GCP_PROJECT_ID,
        location=GCP_REGION,
        gcp_conn_id="google_cloud_default",
        body=body,
        deferrable=True,
        pool=DATAFLOW_POOL  # <--- Important: Limit concurrency
    )
    
    # Task 2: Log Success
    log_to_bq = PythonOperator(
        task_id="log_successful_job",
        python_callable=_log_callable, # Assume defined elsewhere
        op_kwargs={"log_info": log_info},
        pool=DATAFLOW_POOL
    )
    
    run_dataflow_job >> log_to_bq

# Usage in DAG:
# job_specs is a list of dictionaries containing 'body' and 'log_info'
# run_groups = process_jobs_group.expand_kwargs(job_specs)
```

> [!TIP] Key Takeaways
> * **Visual Organization**: Task Groups keep the Airflow UI clean.
> * **Concurrency Control**: Always use Pools when dynamically expanding resource-intensive tasks.
