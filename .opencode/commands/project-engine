---
description: Controlled lifecycle orchestration from classification through plan generation
agent: build
subtask: false
---

# PROJECT ENGINE

Objective:
Orchestrate the structured lifecycle beginning at classification and progressing through plan generation.

Reference:
@docs/reference/project-state.md

---

# Phase Validation

1. Read project-state.md.
2. Identify Current Phase.

If Current Phase != Brainstorm:
- Inform user of current phase.
- Ask whether to resume from current phase or restart.
- Do NOT overwrite artifacts automatically.

---

# Lifecycle Sequence

## Step 1 — Classification

Run:
/classify-app

Pause.
Ask user to review classification.

If approved:
Run:
/advance-phase
Else:
Stop.

---

## Step 2 — Framework Generation

Run:
/generate-framework

Pause for review.

If approved:
Run:
/advance-phase
Else:
Stop.

---

## Step 3 — PRD Generation

Run:
/generate-prd

Pause for review.

If approved:
Run:
/advance-phase
Else:
Stop.

---

## Step 4 — Stack Advisory

Run:
/stack-advisor

(No phase advancement required.)

---

## Step 5 — Architecture Generation

Run:
/generate-architecture

Pause for review.

If approved:
Run:
/advance-phase
Else:
Stop.

---

## Step 6 — Plan Generation

Run:
/generate-plan

Pause for review.

If approved:
Run:
/advance-phase
Else:
Stop.

---

# Completion

Output:

"Project Engine complete. System is ready for implementation phase."
