---
id: 9a3c7b8d-2e1f-4a5b-9c8d-7e6f5a4b3c2d
created: 2025-12-18
aliases: []
tags:
  - opencode-generated
  - loops
  - transformation
  - locals
description: Patterns for transforming data using single and nested for loops within locals.
language: hcl
tech: terraform
title: Locals Loop Transformation
type: pattern
---

# Locals Loop Transformation

> [!INFO] Context
> **Problem**: Transforming raw data (files, complex objects) into flat maps or lists suitable for `for_each` resource iteration.
> **Scope**: Covers `fileset` iteration, nested loops for map flattening, and creating unique lists.

## Conceptual Solution
Terraform `locals` are ideal for pre-processing data. Use `merge([...])` with a `for` loop to build maps from lists or files. Use nested `for` loops within `merge` or `flatten` to normalize deep structures (like projects containing tags) into flat maps keyed by composite unique IDs.

## Implementation

```hcl
locals {
  # 1. Single Loop: iterate files and build a map
  # Uses the spread operator (...) to merge the list of maps
  projects = merge([
    for key, value in fileset(path.module, "data/*.json") : {
      "${split(".", key)[0]}" = {
        name    = split(".", key)[0]
        content = jsondecode(file("${path.module}/data/${key}"))
      }
    }
  ]...)

  # 2. Nested Loop: flatten a map of maps
  # Iterates over the result of the first loop
  project_tags = merge([
    for project_key, project_data in local.projects : {
      for tag_key, tag_value in project_data.content["tags"] :
      "${project_key}-${tag_key}" => {
        project = project_key
        tag     = tag_key
        value   = tag_value
      }
    }
  ]...)

  # 3. Unique List: extract unique values
  organizations = distinct(flatten([
    for key, value in local.projects :
    value.content.organization
  ]))
}
```

> [!TIP] Key Takeaways
> * Use `merge([... ]...)` (spread operator) to turn a list of maps into a single map.
> * Use composite keys (e.g., `"${key}-${v_key}"`) to ensure uniqueness when flattening nested structures.
> * Flattening complex structures in `locals` simplifies `for_each` logic in resources.
