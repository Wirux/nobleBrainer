---
id: 131b84f7-1e49-4a3c-b259-d3de39a6d024
aliases: []
tags:
  - security
  - privacy
  - best-practices
description: Strict rules for scrubbing secrets, specific entities, and paths.
language: Markdown
tech: mcp
title: OpenCode Sanitization Rules
type: config
---

# Code Sanitization Rules

**System Instruction:**
Pass all content through these filters before saving.

## 1. Secrets & Credentials (CRITICAL)
* **API Keys/Tokens**: Replace with `<API_KEY>` or `<TOKEN>`.
* **Passwords**: Replace with `<PASSWORD>`.
* **URIs**: `postgres://admin:123@10.0.0.1/db` -> `postgres://<USER>:<PASS>@<HOST>/<DB>`.
* **Cloud IDs**: `123456789012` -> `<AWS_ACCOUNT_ID>`.

## 2. Path & Project Anonymization
* **Client Names**: Replace specific client names (e.g., `Orlen`, `Barclays`) with `{{CLIENT}}`.
* **Absolute Paths**: Remove `/Users/adamwilczek/...`. Use relative paths or generic placeholders.

## 3. Atomic Cleaning (Noise Reduction)
* **License Headers**: DELETE all Copyright/License headers (e.g., "Copyright 2023 Company X...").
* **Boilerplate**: Remove standard imports or logging setups unless they are the *core* of the pattern.
* **Large Comment Blocks**: Remove large commented-out code blocks.

## 4. Privacy Scrubbing
* **PII**: Remove developer names (`// TODO: Adam fix this` -> `// TODO: fix this`).
* **IPs**: Mask internal IPs (`10.x.x.x` -> `<INTERNAL_IP>`).
