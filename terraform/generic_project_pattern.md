---
id: e3b0c442-98fc-1c14-9abc-4422b5e28a9b
aliases: []
tags:
  - terraform
  - architecture
  - design-pattern
  - iac
description: A pattern decoupling infrastructure logic (Terraform Modules) from configuration data (JSON), orchestrated by a Root Module.
language: HCL
tech: terraform
title: Generic Terraform Project Pattern
type: pattern
---

# Generic Terraform Project Pattern

**System Instruction:**
This pattern demonstrates how to separate **Infrastructure Logic** from **Configuration Data** using Terraform Modules and JSON variable files.

## Core Concept
Instead of hardcoding values in Terraform resources or using complex `.tfvars` files, this pattern uses:
1.  **Root Module (`main.tf`)**: The orchestrator that declares variables and calls child modules.
2.  **Child Modules**: Reusable components containing the actual resource definitions (e.g., `google_project`, `google_service_account`).
3.  **JSON Configuration (`*.auto.tfvars.json`)**: A user-friendly data file that Terraform automatically loads to populate variables.

> [!TIP] Benefits
> *   **Separation of Concerns**: DevOps Engineers maintain the Modules (Logic). Developers/Operators maintain the JSON (Data).
> *   **Automation Friendly**: JSON is easier to generate/parse programmatically than HCL.
> *   **Simplicity**: New environments or projects can be added by simply updating the JSON file.

## Implementation Details

### 1. Configuration (Data Layer)
Define project attributes in a JSON file. Terraform automatically loads files ending in `.auto.tfvars.json`.

**File**: `IaC/101_main.auto.tfvars.json`
```json
{
  "name": "admin",
  "folder_number": "<FOLDER_ID>",
  "billing_account": "<BILLING_ACCOUNT_ID>",
  "workload": {
    "issuer_url": "https://vstoken.dev.azure.com/<TENANT_ID>",
    "subject": "sc://<ORG>/<PROJECT>/<SERVICE_CONNECTION>"
  },
  "notification_emails": [
    "<EMAIL>"
  ]
}
```

### 2. Root Module (Orchestration Layer)
The root module defines the variables matching the JSON structure and passes them to child modules.

**File**: `IaC/01_main.tf`
```hcl
# Call the admin module (OIDC configuration)
module "admin" {
  source = "./modules/oidc"
  providers = {
    google = google.admin
  }
  
  # Pass variables from JSON
  name            = var.name
  folder_number   = var.folder_number
  billing_account = var.billing_account
  
  workload = {
    config = {
      issuer_url = var.workload.issuer_url
    }
    subject = var.workload.subject
  }
}

# Call the project module (Project factory)
module "project" {
  source = "./modules/projects"
  providers = {
    google = google.project
  }
  
  # Pass outputs from 'admin' module to 'project' module
  admin_project_id = module.admin.admin_project_id
  
  # Pass variables from JSON
  folder_number    = var.folder_number
  billing_account  = var.billing_account
  workload = {
    audience_iam = module.admin.wif_audience_iam
    subject      = var.workload.subject
  }
}
```

### 3. Child Modules (Logic Layer)
Modules like `projects` or `oidc` consume these inputs to create resources.

**File**: `IaC/modules/projects/main.tf` (Conceptual)
```hcl
resource "google_project" "project" {
  name       = var.name
  project_id = var.project_id
  folder_id  = var.folder_number
  billing_account = var.billing_account
}
```

## Workflow
1.  **Define**: Update `101_main.auto.tfvars.json` with new project details.
2.  **Plan**: Run `terraform plan`. Terraform loads the JSON, populates `var.*`, and the Root Module configures the Child Modules.
3.  **Apply**: Run `terraform apply` to provision resources.
