---
id: 550e8400-e29b-41d4-a716-446655440000
aliases: []
tags:
  - opencode-generated
  - airflow
  - python
  - dag
description: Explains how to use @task_group for hierarchical task organization and dynamic mapping in Airflow.
language: Python
title: Dynamic Task Groups in Airflow
type: pattern
---

# Dynamic Task Groups in Airflow

## Context & Problem
* **Problem:** Complex DAGs with repeating patterns often become cluttered and hard to read. Furthermore, processing a dynamic list of inputs requires creating tasks at runtime without hardcoding them.
* **Scope:** Covers using `@task_group` for organization and `.expand_kwargs()` for dynamic mapping. It also addresses resource management using Airflow Pools.

## Conceptual Solution
The `@task_group` decorator allows developers to treat a collection of tasks as a single hierarchical unit in the UI. By combining this with Dynamic Task Mapping (`.expand_kwargs()`), a DAG can fan-out this entire group for each item in a list of inputs. Crucially, when dynamically expanding heavy tasks (like Dataflow jobs), assigning them to an Airflow `pool` prevents cluster overload by limiting concurrent executions.

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
        pool=DATAFLOW_POOL  # <--- Important: Using a pool to limit concurrency
    )
    
    # Task 2: Log Success
    log_to_bq = PythonOperator(
        task_id="log_successful_job",
        python_callable=_log_callable, # Assume this callable is defined elsewhere
        op_kwargs={"log_info": log_info},
        pool=DATAFLOW_POOL
    )
    
    run_dataflow_job >> log_to_bq

# Usage in DAG:
# job_specs is a list of dictionaries prepared earlier containing 'body' and 'log_info'
# Dynamically create task groups for each job spec.
run_groups = process_jobs_group.expand_kwargs(job_specs)
```
