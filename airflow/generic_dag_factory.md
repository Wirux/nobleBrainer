---
id: facade00-fact-0000-0000-000000000001
created: 2025-12-08
aliases: []
tags:
  - opencode-generated
  - dag-factory
  - environment-config
description: A pattern for generating multiple Airflow DAGs dynamically using a factory function and centralized configuration to avoid late binding issues.
language: python
tech: airflow
title: Generic DAG Factory
type: pattern
---

# Generic DAG Factory

> [!INFO] Context
> **Problem**: Creating multiple similar Airflow DAGs (e.g., for different tenants) leads to code duplication. Using loops can cause "late binding" issues where all DAGs use values from the last iteration.
> **Scope**: Covers DAG factory functions, centralized configuration, and generation loops.

## Conceptual Solution
Use a **Factory Function** to encapsulate the DAG definition, passing dynamic values (company names, project IDs) as arguments. This "freezes" variables for each instance, preventing late binding. Centralize configuration in maps (`DAG_DEFINITIONS` and `ENV_CONFIGS`) to separate logic from data.

## Implementation

```python
from airflow.decorators import dag, task
from airflow.models.variable import Variable
from airflow.datasets import Dataset
import pendulum

# =============================================================================
# 1. CENTRAL CONFIGURATION
# =============================================================================
DAG_DEFINITIONS = {
    "tenant_a": {
        "prefix": "tenant_a",
        "table_name": "source_table_a",
    },
    "tenant_b": {
        "prefix": "tenant_b",
        "table_name": "source_table_b",
    },
}

ENV_CONFIGS = {
    "dev": {
        "project_id": "my-dev-project",
        "region": "europe-west1",
        "service_account": "sa-dev@my-project.iam.gserviceaccount.com",
    },
    "prod": {
        "project_id": "my-prod-project",
        "region": "europe-west1",
        "service_account": "sa-prod@my-project.iam.gserviceaccount.com",
    }
}

# =============================================================================
# 2. DAG FACTORY FUNCTION
# =============================================================================
def create_dag_factory(
    prefix: str,
    table_name: str,
    project_id: str,
    region: str,
    service_account: str,
):
    """
    Factory function to generate a DAG.
    Arguments are passed explicitly to avoid late binding in loops.
    """
    dag_id = f"{prefix}_processing_dag"
    dataset_name = f"{prefix}_dataset"
    target_table = f"{project_id}.{prefix}.{table_name}"

    @dag(
        dag_id=dag_id,
        start_date=pendulum.datetime(2023, 1, 1, tz="UTC"),
        schedule=[Dataset(dataset_name)],
        catchup=False,
        tags=[prefix, "generated"],
    )
    def generated_dag():
        
        @task(task_id="process_data")
        def process_task():
            print(f"Processing {table_name} for {prefix} in {region}")
            # Logic using target_table...

        process_task()

    return generated_dag()

# =============================================================================
# 3. GENERATION LOOP
# =============================================================================
env = Variable.get("env", default_var="dev") 
if not (env_vars := ENV_CONFIGS.get(env)):
    raise ValueError(f"Unknown environment: '{env}'")

for key, config in DAG_DEFINITIONS.items():
    # Call the factory with explicit arguments
    create_dag_factory(
        prefix=config["prefix"],
        table_name=config["table_name"],
        project_id=env_vars["project_id"],
        region=env_vars["region"],
        service_account=env_vars["service_account"],
    )
```

> [!TIP] Key Takeaways
> * **Explicit Arguments**: Passing values into the factory function breaks the closure scope issue common in Python loops.
> * **Config Separation**: Moving configuration out of the code allows for cleaner environment management.
