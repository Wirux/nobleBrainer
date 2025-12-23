---
id: e38c92f1-6a45-4b8c-9d1f-8e2b5c7a9d3e
created: 2025-12-12
aliases: []
tags:
  - opencode-generated
  - authentication
  - cicd
description: Bash script to exchange an external OIDC token for a GCP Service Account access token using Workload Identity Federation.
language: bash
tech: gcp
title: Workload Identity Token Exchange
type: snippet
---

# Workload Identity Token Exchange

> [!INFO] Context
> **Problem**: Authenticating to Google Cloud Platform from external CI/CD pipelines (like Azure DevOps) without using insecure, long-lived Service Account keys.
> **Scope**: Covers the token exchange process using a temporary file and the `gcloud` credential configuration. Assumes an OIDC token is available in the environment.

## Conceptual Solution
The script reads a configuration file, captures the external OIDC token (e.g., from Azure DevOps), and creates a temporary `GOOGLE_APPLICATION_CREDENTIALS` file. This file tells the Google libraries how to exchange the external token for a GCP Service Account token via the STS (Security Token Service) and IAM credentials API.

## Implementation

```bash
#!/bin/bash

# Configuration: Reads target Service Account and WIF Audience from a JSON file
# Expected wif.json format:
# {
#   "sa_email": "{{SERVICE_ACCOUNT_EMAIL}}",
#   "wif_audience": "{{WIF_AUDIENCE_URI}}"
# }
variables=$(cat wif.json)
sa_email=$(echo $variables | jq -r '.sa_email')
wif_audience=$(echo $variables | jq -r '.wif_audience')

# Location for the temporary JWT token file
jwt_token=$WORKSPACE/.workload_identity.jwt
# $idToken is expected to be present in the environment (e.g., from Azure DevOps OIDC task)
echo $idToken >$jwt_token

# Create the credential configuration file
cat <<EOF >$GOOGLE_APPLICATION_CREDENTIALS
{
  "type": "external_account",
  "audience": "//iam.googleapis.com/$wif_audience",
  "subject_token_type": "urn:ietf:params:oauth:token-type:jwt",
  "token_url": "https://sts.googleapis.com/v1/token",
  "credential_source": {
    "file": "$jwt_token"
  },
  "service_account_impersonation_url": "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/$sa_email:generateAccessToken"
}
EOF

# Azure DevOps specific: Output variables for subsequent tasks
echo "##vso[task.setvariable variable=TF_INPUT]0"
echo "##vso[task.setvariable variable=ARM_CLIENT_ID]$servicePrincipalId"
echo "##vso[task.setvariable variable=ARM_OIDC_TOKEN]$idToken"
echo "##vso[task.setvariable variable=ARM_TENANT_ID]$tenantId"
echo "##vso[task.setvariable variable=ARM_USE_OIDC]true"
```

> [!TIP] Key Takeaways
> *   Avoids long-lived JSON keys.
> *   Uses `external_account` credential type.
> *   Requires the environment to provide the OIDC token (in `$idToken`).
