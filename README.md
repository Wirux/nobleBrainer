# nobleBrainer Vault

## Introduction

The **nobleBrainer Vault** is a specialized Obsidian knowledge base designed to serve as a dynamic data source for the Model Context Protocol (MCP). It functions specifically as the backend repository for the `obsidian-mcp-server`, bridging the gap between static knowledge management and active AI agent interaction.

By structuring information within this vault, we provide AI agents with a persistent context layer, enabling them to retrieve defined workflows, adhere to specific operational patterns, and utilize stored logic during execution.

## Purpose

The primary purpose of this vault is to store structured knowledge, patterns, and workflows in a format accessible to AI agents. It acts as the "long-term memory" and "rule set" for the MCP server.

Key objectives include:
*   **Context Provisioning:** Supplying AI models with necessary background information and logic flows without requiring manual context injection for every session.
*   **Standardization:** Enforcing consistent behavior in AI outputs by defining rigorous patterns and sanitization rules.
*   **Workflow Definition:** Mapping out complex processes using Directed Acyclic Graphs (DAGs) to guide AI reasoning and task execution.

## Content Overview

The vault is organized into specific semantic domains to separate logic flows from interaction rules.

### Workflows and Logic (`dag`)
This section contains Directed Acyclic Graphs (DAGs) and logic factories. These documents represent structured processes where tasks or concepts have defined dependencies and directions. The MCP server utilizes these definitions to understand the order of operations for complex tasks, ensuring that prerequisites are met before subsequent steps are attempted.

### MCP Patterns and Rules (`patterns`)
This section houses the operational "constitution" for the MCP server. It includes:
*   **Entry Points:** Definitions for how the server should initialize interactions.
*   **Output formatting:** Templates and rules that dictate how the AI should structure its responses.
*   **Sanitization:** Security and cleanliness protocols to ensure data integrity during input/output operations.

These patterns ensure that the AI agent interacts with the vault in a safe, predictable, and highly structured manner.
