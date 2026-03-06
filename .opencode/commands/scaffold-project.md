---
description: Initialize the project scaffold using the architecture and stack outputs
agent: build
subtask: true
---

# Scaffold Project Command

Use the `scaffold-project` skill to initialize the project structure.

The scaffold must be generated using the planning artifacts produced earlier in the pipeline.

Required context files:

- docs/reference/framework.md
- docs/reference/stack.md
- docs/reference/architecture.md
- docs/reference/plan.md

Invoke the `scaffold-project` skill.

Ensure the scaffold includes:

- project directory structure
- base configuration files
- dependency initialization
- environment configuration
- README documentation

After completion verify:

- directories exist
- config files exist
- dependency manager initialized

Output only a confirmation message:

"Project scaffold successfully created."
