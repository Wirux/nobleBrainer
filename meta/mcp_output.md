---
id: fd1bfc53-91c9-4cf8-b84e-3b93451400b5
aliases: []
tags:
  - opencode-config
  - template
  - design-pattern
description: The official markdown structure for pattern notes, utilizing Obsidian Callouts.
language: Markdown
tech: mcp
title: OpenCode Pattern Template
type: config
---

# Pattern Note Template

**System Instruction:**
Use the structure below. Replace `{{...}}` with content. Use Obsidian Callouts for emphasis.

```markdown
---
id: {{generate_uuid_v4}}
created: {{date_YYYY-MM-DD}}
aliases: []
tags:
  - opencode-generated
  - {{keyword_tag_1}}
  - {{keyword_tag_2}}
description: {{A concise, one-sentence summary}}
language: {{programming_language_for_code_block}}
tech: {{airflow | python | sql | docker | bash}}
title: {{Action-Oriented Title}}
type: {{pattern | snippet | architecture | config | fix}}
---

# {{Title}}

> [!INFO] Context
> **Problem**: {{Specific problem this solves}}
> **Scope**: {{What is covered/excluded}}

## Conceptual Solution
{{Brief explanation of the mechanism. Keep it under 3-4 sentences.}}

## Implementation

```{{language}}
{{code_snippet_containing_ONLY_the_relevant_logic}}
// Note: Remove unrelated boilerplate.
```

> [!TIP] Key Takeaways
> * {{Bullet point 1}}
> * {{Bullet point 2}}
```
