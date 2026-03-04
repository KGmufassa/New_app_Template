---
description: Generate a highly technical implementation-ready PRD from current context
agent: plan
subtask: true
---

Use the entire visible conversation as source material.

Convert it into a structured technical PRD.

Additional refinement instructions:
$ARGUMENTS

---

Follow the official template below strictly.

@docs/templates/TECHNICAL_PRD_TEMPLATE.md

---

Instructions:

After generating the document:

1. Save it to:

docs/reference/prd.md

2. Overwrite if it already exists.
3. Output confirmation message:
   "PRD saved to docs/reference/prd.md"


4. Do not rename headings.
5. Populate every section.
6. All requirements must be testable.
7. Architecture decisions must be explicit.
8. Missing information must go under "Assumptions".
9. Output clean Markdown only.

**Do not output the full document in chat unless explicitly requested.

