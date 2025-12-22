---


name: Backend Generation Guide
description: Integrated guidelines for Spring Boot-based MSA backend code generation
---



# Backend Project Generation Guideline
## Instructions
This document provides skill sets and guidelines to apply when processing backend code generation requests based on PRD files.

## Request Type Identification

When receiving a code generation request, first check if it is a **PRD** request.

## Skill Application Rules

### PRD Requests (Backend)

Use all skills in the `.claude/skills/spring-boot-backend-msa` directory for backend code generation.

#### Mandatory Priority: Read generation first
Before starting any code generation work, you must **read the following file first**:
- @.claude/skills/spring-boot-backend-msa/references/generation.md

#### Backend References to Apply

1. **Generation Reference** (Highest Priority)
   - Description: Basic rules and procedures for backend code generation
   - Path: @.claude/skills/spring-boot-backend-msa/references/generation.md

2. **Domain Events Reference**
   - Description: Domain event publishing and handling rules
   - Path: @.claude/skills/spring-boot-backend-msa/references/domain-events.md

3. **Architecture Reference**
   - Description: Event Storming sticker-based design rules, package structure organization, and technical stack guidelines
   - Path: @.claude/skills/spring-boot-backend-msa/references/architecture.md

4. **Fixed Generation Reference**
   - Description: Infrastructure configuration for DDD-based microservices
   - Instructions: Use the infra-architect agent to create Dockerfile, Kubernetes manifests, API Gateway, and Kafka infrastructure

5. **Test Generation Reference**
   - Description: Backend test code generation rules
   - Instructions: Use the test-generator agent to create JUnit tests following the Given-When-Then pattern

## Important Notes
- **Generation Reference Always Has Priority**: Must be read before all other references
- **All References Must Be Applied**: All reference files in the relevant directory must be applied without exception
- **Prevent Metadata Omission and Misreference**: Files must be generated according to the metadata defined in PRD.txt, and unrelated files should not be generated using arbitrary metadata not defined in the documents
- **Response Language**: All responses, explanations, and communications must be provided in Korean
