---
id: generic_dag_factory
aliases: []
tags:
  - core
  - dag-factory
  - template
description: Canonical generic DAG factory used as the primary scaffold for generating Airflow DAGs across domains.
mcp_id: generic_dag_factory
title: Generic DAG Factory
---

# Generic DAG Factory

## Purpose

The **Generic DAG Factory** is the canonical scaffold used to generate Airflow DAGs from declarative configuration.  
It is intentionally **provider-agnostic** and serves as the **foundation** for domain-specific task builders (BigQuery, Dataflow, dbt, batch pipelines, streaming, etc.).

This pattern ensures:

- all DAGs have a unified structure  
- environment configuration is externalized  
- pipelines are declarative and reproducible  
- domain-specific code stays modular and isolated  
- Airflow dynamically instantiates DAGs at parse time

---

# Key Properties

- **Minimal core context** that the MCP Agent always loads first  
- Fully parameterized Ð no hardcoded environment variables in code  
- Task logic is injected via a **task_builder(...)** function  
- Supports both simple tasks and complex multi-task groups  

---

# File Structure

- `DAG_DEFINITIONS` Ð declarative definition of pipelines/domains/clients  
- `ENV_CONFIGS` Ð mapping of environment names to runtime properties  
- `create_generic_dag_factory(...)` Ð central DAG factory  
- `generate_all_dags(...)` Ð instantiation loop  

---

# Example: Generic Factory Skeleton

```python
from __future__ import annotations
from datetime import datetime
from airflow.decorators import dag, task
from airflow.operators.empty import EmptyOperator

# ============================================================================
# 1. PIPELINE DEFINITIONS (Domain-level config)
# ============================================================================

DAG_DEFINITIONS = {
    "clientA": {
        "prefix": "clientA",
        "custom_params": {"source": "crm", "mode": "full"},
    },
    "clientB": {
        "prefix": "clientB",
        "custom_params": {"source": "erp", "mode": "incremental"},
    },
}

# ============================================================================
# 2. ENVIRONMENT CONFIGURATION
# ============================================================================

ENV_CONFIGS = {
    "dev": {"gcp_project_id": "project-dev"},
    "test": {"gcp_project_id": "project-test"},
    "prod": {"gcp_project_id": "project-prod"},
}

DAG_VERSION = "v1.0.0"


# ============================================================================
# 3. GENERIC DAG FACTORY
# ============================================================================

def create_generic_dag_factory(
    dag_prefix: str,
    dag_version: str,
    env_name: str,
    start_date: datetime,
    custom_params: dict,
    task_builder=None,
):

    dag_id = f"{dag_prefix}_{dag_version}"

    @dag(
        dag_id=dag_id,
        start_date=start_date,
        catchup=False,
        tags=[dag_prefix, env_name, "generic"],
    )
    def generated_dag():

        start = EmptyOperator(task_id="start")

        if task_builder:
            # Inject domain-specific logic
            dynamic_task = task_builder(custom_params)
        else:
            @task
            def noop(params):
                return params

            dynamic_task = noop(custom_params)

        end = EmptyOperator(task_id="end")

        start >> dynamic_task >> end

    return generated_dag()


# ============================================================================
# 4. DAG INSTANTIATION LOOP
# ============================================================================

from airflow.models.variable import Variable

env = Variable.get("env")

if env not in ENV_CONFIGS:
    raise ValueError(
        f"Unknown environment '{env}'. Allowed: {list(ENV_CONFIGS.keys())}"
    )

env_config = ENV_CONFIGS[env]

for name, cfg in DAG_DEFINITIONS.items():
    create_generic_dag_factory(
        dag_prefix=cfg["prefix"],
        dag_version=DAG_VERSION,
        env_name=env,
        start_date=datetime(2024, 1, 1),
        custom_params=cfg["custom_params"],
        task_builder=None,  # replaced by domain-specific patterns
    )
