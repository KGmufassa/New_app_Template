---
description: Advance project to next lifecycle phase if requirements are met
agent: build
subtask: true
---

Read:

@docs/reference/project-state.md

Determine current phase.

Validate required artifact exists:

If Current Phase = Brainstorm
→ Require docs/reference/app-classification.md

If Classified
→ Require docs/reference/framework.md

If Framework Generated
→ Require docs/reference/prd.md

If PRD Generated
→ Require docs/reference/architecture-v1.md

If Architecture Generated
→ Require docs/reference/plan.md

If validation fails:
Output which artifact is missing.

If validation succeeds:
Update project-state.md:

- Move to next phase
- Lock artifact version
- Update Last Updated timestamp

Save changes.

Output confirmation only.
