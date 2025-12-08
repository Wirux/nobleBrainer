---
id: fd1bfc53-91c9-4cf8-b84e-3b93451400b5
aliases: []
tags:
  - opencode-config
  - template
  - design-pattern
description: The official markdown structure that must be used for all generated pattern notes.
language: Markdown
tech: mcp
title: OpenCode Pattern Template
type: config
---

# Pattern Note Template

**System Instruction:**
Use the Markdown structure below strictly. Replace content inside `{{...}}` with generated content.
ensure the YAML Frontmatter is valid.

```markdown
---
id: {{generate_uuid_v4}}
aliases: []
tags:
  - opencode-generated
  - {{keyword_tag_1}}
  - {{keyword_tag_2}}
description: {{A concise, one-sentence summary}}
language: {{programming_language_for_code_block}}
source: {{sanitized_relative_path}}
tech: {{airflow | python | sql | docker | bash}}
title: {{Action-Oriented Title e.g., "Implementing Singleton in DAG Factory"}}
type: {{pattern | snippet | architecture | config | fix}}
---

# {{Title: Same as in frontmatter}}

## Context & Problem
* **Problem:** {{One specific problem this pattern solves. Be concise.}}
* **Scope:** {{What does this cover? What does it explicitely exclude?}}

## Conceptual Solution
{{Brief explanation of the *why* and the mechanism. Keep it under 3-4 sentences.}}

## Implementation
```{{language}}
{{code_snippet_containing_ONLY_the_relevant_logic}}
// Note: Remove unrelated boilerplate. Focus on the pattern logic.
