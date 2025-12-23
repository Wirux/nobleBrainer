---
id: 5f8a1b2c-3d4e-5f6a-7b8c-9d0e1f2a3b4c
created: 2025-12-12
aliases: []
tags:
  - opencode-generated
  - azure-devops
  - terraform
  - authentication
description: Azure DevOps pipeline configuration for authenticating to GCP using Workload Identity Federation before running Terraform.
language: yaml
tech: cicd
title: Azure DevOps GCP Workload Identity Pipeline
type: pattern
---

# Azure DevOps GCP Workload Identity Pipeline

> [!INFO] Context
> **Problem**: Integrating the GCP Workload Identity token exchange process into an Azure DevOps pipeline structure to enable Terraform to provision GCP resources securely.
> **Scope**: Covers the pipeline YAML configuration, specifically the `AzureCLI@2` task usage for credential preparation and the subsequent Terraform tasks.

## Conceptual Solution
The pipeline uses the `AzureCLI@2` task to execute the token exchange script. This task is chosen because it provides access to the Azure DevOps OIDC token (via the Service Connection) and allows exporting environment variables (like `GOOGLE_APPLICATION_CREDENTIALS` and `ARM_*` variables) to subsequent tasks in the same job.

## Implementation

```yaml
trigger:
- main

variables:
- name: Azure.WorkloadIdentity.Connection
  value: {{SERVICE_CONNECTION_NAME}} # e.g., gcp_serviceconnector
- name: GOOGLE_APPLICATION_CREDENTIALS
  value: $(Pipeline.Workspace)/.workload_identity.wlconfig

jobs:
- job: plan
  displayName: 'Terraform plan'
  steps:
    # Key Step: Prepare Credentials using the token exchange script
    - task: AzureCLI@2
      displayName: 'Prepare Credentials'
      inputs:
        connectedServiceNameARM: $(Azure.WorkloadIdentity.Connection)
        addSpnToEnvironment: true
        scriptType: 'bash'
        scriptLocation: 'scriptPath'
        # References the script created in the complementary pattern
        scriptPath: '$(System.DefaultWorkingDirectory)/token.sh'
      env:
        WORKSPACE: $(Pipeline.Workspace)
        
    # Run terraform with the authenticated session
    - script: |
        cd IaC
        terraform init
        terraform plan
      displayName: 'Plan'

- job: check
  displayName: 'Manual validation'
  dependsOn: plan
  pool: server
  steps:
  - task: ManualValidation@0
    inputs:
      # notifyUsers: '{{NOTIFY_EMAIL}}'
      instructions: 'continue?'

- job: apply
  displayName: 'Terraform apply'
  dependsOn: check
  steps:
    # Credentials must be re-initialized in a new job/agent
    - task: AzureCLI@2
      displayName: 'Prepare Credentials'
      inputs:
        connectedServiceNameARM: $(Azure.WorkloadIdentity.Connection)
        addSpnToEnvironment: true
        scriptType: 'bash'
        scriptLocation: 'scriptPath'
        scriptPath: '$(System.DefaultWorkingDirectory)/token.sh'
      env:
        WORKSPACE: $(Pipeline.Workspace)
        
    - script: |
        cd IaC
        terraform init
        terraform apply -auto-approve
      displayName: 'Apply'
```

> [!TIP] Key Takeaways
> *   **Re-authentication**: The `AzureCLI@2` task (and the token exchange) must run in *each* job (`plan` and `apply`) because jobs run on fresh agents.
> *   **Service Connection**: The `connectedServiceNameARM` must be an Azure Resource Manager connection with Workload Identity Federation enabled on the Azure side (to generate the OIDC token).
> *   **Environment Variables**: The `token.sh` script (see [[gcp/workload_identity_token_exchange]]) exports variables using `##vso[task.setvariable...]` which persist for the duration of the job.
