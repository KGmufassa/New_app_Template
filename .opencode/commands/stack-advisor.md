---
description: Retrieve stack-specific best practices using Context7 MCP and generate structured stack guidance
agent: plan
subtask: true
---

Reference:

@docs/reference/app-classification.md

---

# Objective

Determine the application's primary tech stack and retrieve official best practices using Context7 MCP.

If tech stack is not defined in classification:
- Abort and instruct user to define stack in PRD or classification.

---

# Step 1: Detect Stack

Extract from classification or PRD:

- Frontend framework
- Backend framework
- Database
- ORM
- Deployment target
- Testing framework (if defined)

---

# Step 2: Use Context7 MCP

For each detected technology:

Query Context7 MCP for:

- Recommended folder structure
- Service layer patterns
- Dependency injection practices
- Environment configuration patterns
- Error handling patterns
- Testing best practices
- Production deployment best practices
- Performance recommendations
- Security considerations

Important:
- Retrieve only official and stable patterns.
- Ignore experimental features.
- Focus on current major version best practices.
- Summarize findings concisely.

Do NOT store raw MCP output.
Summarize into structured guidance.

---

# Step 3: Produce Structured Output

Output format:

# STACK BEST PRACTICES

## Stack Overview

- Frontend:
- Backend:
- Database:
- ORM:
- Deployment Target:

---

## Recommended Folder Structure

---

## Service & Layering Patterns

---

## Configuration Management

---

## Database Best Practices

---

## API Patterns

---

## Testing Strategy

---

## Deployment & DevOps

---

## Performance Considerations

---

## Security Considerations

---

# Step 4: Save Artifact

After generating:

1. Save file to:

docs/reference/stack-best-practices.md

2. Create directory if missing.
3. Overwrite existing file.
4. Output confirmation only:

"Stack best practices saved to docs/reference/stack-best-practices.md"
