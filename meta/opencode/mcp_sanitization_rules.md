---
id: 131b84f7-1e49-4a3c-b259-d3de39a6d024
aliases: []
tags:
  - opencode-config
  - security
  - design-pattern
description: Rules for removing sensitive data and refactoring code before saving to the Vault.
language: English
title: OpenCode Sanitization Rules
type: config
---

# Code Sanitization Rules

Before saving any code to the Vault via MCP, apply these transformations:

1.  **Secrets Removal**:
    * Replace API keys with `<API_KEY>`.
    * Replace passwords with `<PASSWORD>`.
    * Replace DB connection strings with `env::var(...)` or generic placeholders.

2.  **Privacy Scrubbing**:
    * Remove names of real people, internal IP addresses (10.x.x.x), and company-specific domain names.

3.  **Generalization**:
    - [!] Refactor highly specific variable names (e.g., `client_Zalando_config`) to generic ones (e.g., `client_config`) unless the pattern is specifically about that integration.
