---
id: 131b84f7-1e49-4a3c-b259-d3de39a6d024
aliases: []
tags:
  - security
  - privacy
  - best-practices
description: Strict rules for scrubbing secrets, specific entities, and paths before saving to the Vault.
language: Markdown
tech: mcp
title: OpenCode Sanitization Rules
type: config
---

# Code Sanitization Rules

**System Instruction:**
Before saving ANY content to the Vault (via MCP), you must pass the code and metadata through these filters.

## 1. Secrets & Credentials (High Priority)
Scan the code for potential leaks.
* **API Keys/Tokens**: Replace with `<API_KEY>` or `<TOKEN>`.
* **Passwords**: Replace with `<PASSWORD>`.
* **Connection Strings**: Replace specific URIs (e.g., `postgresql://admin:123@10.0.0.1...`) with generic ones: `postgresql://<USER>:<PASS>@<HOST>/<DB>`.
* **Cloud IDs**: Mask AWS Account IDs or GCP Project IDs if they look real (e.g., `123456789012` -> `<AWS_ACCOUNT_ID>`).

## 2. Path & Project Anonymization
If file paths, folder names, or project codes appear in the **description**, **context**, or **code comments**:
* **Redact Client Names**: Replace specific client/project names (e.g., `Orlen`, `Barclays`, `ClientX`) with `{{CLIENT}}` or `{{PROJECT}}`.
    * *Example in comment*: `# logic for Orlen migration` -> `# logic for {{CLIENT}} migration`.
* **Strip Absolute Paths**: Remove local user paths (e.g., `/Users/adamwilczek/...`) entirely.

## 3. Code Generalization
* **Refactor Names**: Change highly specific variable names to generic ones unless relevant to the pattern.
    * `def calculate_orlen_margin(...)` -> `def calculate_margin(...)`
* **Remove Noise**: Delete boilerplate imports, logging configurations, or standard comments that do not add value to the specific pattern being extracted.

## 4. Privacy Scrubbing
* **PII**: Remove names of real people (comments like `// TODO: Adam fix this`).
* **Internal Networking**: Mask internal IP addresses (e.g., `10.x.x.x` -> `<INTERNAL_IP>`).
