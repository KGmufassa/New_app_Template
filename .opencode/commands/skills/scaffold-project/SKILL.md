---
name: scaffold-project
description: Initialize a project repository scaffold based on framework, stack, and architecture outputs.
---

# Scaffold Project Skill

This skill generates the base project structure required before code generation.

It converts planning artifacts into a working repository scaffold.

---

# Required Inputs

The following files must exist:

- docs/reference/framework.md
- docs/reference/stack.md
- docs/reference/architecture.md
- docs/reference/plan.md

If any are missing, stop execution and report the missing artifact.

---

# Step 1: Determine Framework

Read framework.md and determine the primary application framework.

Examples:

Next.js  
FastAPI  
Django  
Node Express  
Spring Boot

---

# Step 2: Initialize Repository

If the current directory is not a git repository:

Initialize one.

Create `.gitignore` with common exclusions:

node_modules  
.env  
dist  
build  
.cache

---

# Step 3: Initialize Framework

Initialize the project based on the framework.

Examples:

Next.js

npx create-next-app@latest .

FastAPI

Create structure:

app/
main.py
requirements.txt

Node API

npm init -y

---

# Step 4: Install Dependencies

Read stack.md and install required packages.

Example:

npm install express drizzle-orm postgres zod

or

pip install fastapi sqlalchemy pydantic

---

# Step 5: Create Directory Structure

Use the architecture definition to create directories.

Default structure:

/api
/components
/services
/db
/lib
/config
/scripts
/tests

---

# Step 6: Generate Base Files

Create the following files if they do not exist:

.env.example

README.md

config/app.config

Entry point examples:

app/main.ts
server/index.ts

---

# Step 7: Create Project README

Generate a README.md including:

Project name  
Technology stack  
Architecture summary  
Setup instructions

---

# Step 8: Validation

Verify:

- directories exist
- dependencies installed
- configuration files created
- framework initializes successfully

If validation fails, report the issue and halt execution.

---

# Output

The final scaffold must contain:

package.json or requirements.txt  
.env.example  
README.md  

/api  
/components  
/services  
/db  
/config  
/scripts  
/tests
