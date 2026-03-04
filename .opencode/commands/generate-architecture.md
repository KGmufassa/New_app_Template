---
description: Generate enforced architecture document and save to docs/reference
agent: plan
subtask: true
---
Before executing:

Check project-state.md

If Current Phase < PRD Generated:
Abort and instruct user to generate PRD first.

---

Convert the current context or referenced PRD into a structured Architecture Document.

Additional refinement instructions:
$ARGUMENTS

Follow this template strictly:

@docs/templates/ARCHITECTURE_TEMPLATE.md

Ensure:
- Folder structure is explicitly defined
- Layer responsibilities are enforced
- No vague descriptions
- Trade-offs are documented
- Scaling strategy is explicit

---

After generating the document:

1. Save the file to:

docs/reference/architecture.md

2. If the docs/reference directory does not exist, create it.
3. Overwrite the file if it already exists.
4. Do NOT output the full document in chat.
5. Output only this confirmation message:

"Architecture document saved to docs/reference/architecture.md"
