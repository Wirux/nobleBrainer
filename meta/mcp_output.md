---
id: fd1bfc53-91c9-4cf8-b84e-3b93451400b5
aliases: []
tags:
  - opencode-config
  - template
  - design-pattern
description: The official markdown structure that must be used for all generated pattern notes.
language: Markdown
title: OpenCode Pattern Template
type: config
---

# Pattern Note Template

Use the structure below for generating the response. Replace content inside `{{...}}` with generated content.

```markdown
---
id: {{generate_uuid_v4}}
aliases: []
tags:
  - opencode-generated
  - {{tag_1}}
  - {{tag_2}}
  - {{tag_3}}
description: {{A concise, one-sentence summary of the pattern's purpose}}
language: {{programming_language}}
title: {{Action-Oriented Title e.g:"Implementing Singleton in Python" }}
type: {{pattern | snippet | architecture | config}}
---

# {{Title: Same as in frontmatter}}

## Context & Problem
* **Problem:** {{One specific problem this pattern solves. Be concise.}}
* **Scope:** {{Explicitly state what this pattern covers (and what it does NOT cover, if necessary to avoid confusion).}}

## Conceptual Solution
{{Brief explanation of the mechanism. Keep it under 3-4 sentences.}}

## Implementation
```{{programming_language}}
{{code_snippet_containing_ONLY_the_relevant_logic}}
// Exclude unrelated code (e.g., remove unrelated TaskGroups if focusing on the Factory logic)
