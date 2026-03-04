---
description: Classify application type and extract system constraints
agent: plan
subtask: true
---

Analyze the current clarified idea and extract structured system metadata.

Identify:

- App Type (SaaS, Internal Tool, AI-native, Marketplace, API-first, etc.)
- Tenant Model (Single / Multi)
- Scale Expectation (MVP / 1k users / 10k+ / 100k+)
- Data Sensitivity Level (Low / Moderate / High / Regulated)
- External Integrations Required
- Real-Time Requirements
- AI/LLM Involvement
- Compliance Requirements
- Monetization Model
- Deployment Expectations

---

Output structured format:

# APPLICATION CLASSIFICATION

## System Type
## Tenant Model
## Scale Profile
## Data Sensitivity
## Compliance Tier
## Real-Time Requirements
## AI Involvement
## External Integrations
## Risk Level

---

After generating:

1. Save to:

docs/reference/app-classification.md

2. Create directory if missing.
3. Overwrite existing file.
4. Output confirmation only:
"Application classification saved to docs/reference/app-classification.md"
