---
id: c4e5f6a7-b8c9-4d0e-1f2a-3b4c5d6e7f8a
created: 2025-12-18
aliases: []
tags:
  - opencode-generated
  - nvim
  - troubleshooting
  - treesitter
description: Fixes Neovim crashes when opening files with embedded Bash scripts by disabling the corrupted Bash parser.
language: lua
tech: nvim
title: Treesitter Bash Injection Fix
type: fix
---

# Treesitter Bash Injection Fix

> [!INFO] Context
> **Problem**: Neovim crashes immediately when opening YAML or Markdown files containing embedded Bash code blocks (e.g., CI/CD pipelines).
> **Scope**: Affects `nvim-treesitter` configurations where the `bash` parser is corrupted or incompatible with injected language regions.

## Conceptual Solution
The crash occurs when the `yaml` or `markdown` parser attempts to hand off syntax highlighting to the `bash` parser for embedded scripts. The solution is to explicitly disable the `bash` parser installation to prevent this handoff, while keeping `yaml` and `markdown` parsers active for structural highlighting.

## Implementation

```lua
return {
  {
    "nvim-treesitter/nvim-treesitter",
    opts = function(_, opts)
      if type(opts.ensure_installed) == "table" then
        vim.list_extend(opts.ensure_installed, {
          "markdown",
          "markdown_inline",
          "yaml",
        })
      end
      -- We ignore 'bash' to prevent crashes caused by injections in YAML/Markdown files
      opts.ignore_install = { "bash" }
    end,
  },
}
```

> [!TIP] Key Takeaways
> * Disabling `bash` prevents the crash but removes highlighting for bash blocks.
> * Run `:TSInstall yaml markdown markdown_inline` to ensure structural parsers are present.
> * Run `:TSUninstall bash` to remove any existing corrupted parser files.
