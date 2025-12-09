---
name: Frontend Generation Guide
description: Integrated guidelines for React-based frontend code generation
---

# Frontend Project Generation Guidelines

## Instruction

This document provides the skillset and guidelines to apply when handling frontend code generation requests based on Frontend-PRD files.

## Request Type Identification

When receiving a code generation request, first check if it is a **Frontend-PRD** request.

## Skill Application Rules

### Frontend-PRD Request (Frontend)

For frontend code generation, refer to all references in the `.claude/skills/react-frontend` directory.

#### Frontend Generation Order

Frontend generation proceeds in the following order:

**Step 1: Generate Frontend Base Structure**
- First read and execute @frontend-setup-prd.md
- Create base structure including package structure, sidebar, routing, etc.
- If frontend structure already exists, only adjust sidebar and main cards

**Step 2: Generate Metadata-based Frontend Code**
- Read and execute @.claude/skills/react-frontend/references/generation.md
- Generate components based on Frontend-PRD.txt metadata
- Complete build and error fixes

**Step 3: Generate and Connect ERP Common Components**
- Read and execute @erp-component-prd.md
- Generate ERP essential feature components (Export/Import, Search, Filter, Chatter, Activities, etc.)
- Connect with existing components
- Complete build and final error fixes

#### Frontend Reference Materials to Apply

1. **Frontend Setup** (Step 1)
   - Description: Frontend base structure and sidebar generation
   - Path: @@.claude/skills/react-frontend/references/frontend-setup-prd.md

2. **Generation Reference** (Step 2 - Highest Priority)
   - Description: Metadata-based frontend code generation rules and procedures
   - Path: @.claude/skills/react-frontend/references/generation.md

3. **Architecture Reference**
   - Description: Frontend architecture, technology stack, project structure and organization principles
   - Path: @.claude/skills/react-frontend/references/architecture.md

4. **Design System Reference**
   - Description: UI design system and component design rules
   - Path: @.claude/skills/react-frontend/references/design-system.md

5. **ERP Component** (Step 3)
   - Description: ERP common component generation and connection
   - Path: @.claude/skills/react-frontend/references/erp-component-prd.md

6. **Troubleshooting Reference** (Final)
   - Description: Solutions for common frontend development issues
   - Path: @.claude/skills/react-frontend/references/troubleshooting.md

## Important Notes
- **Generation Reference**: After Step 1 frontend base structure generation, this must be checked first when generating metadata-based frontend code
- **All Reference Materials Must Be Applied**: All reference files in the directory must be applied without exception
- **Prevent Metadata Omissions and Misreferences**: Files must be generated according to the metadata defined in Frontend-PRD.txt, and unrelated files should not be generated using arbitrary metadata not defined in the document
- **Response Language**: All responses, explanations and communications must be provided in Korean
