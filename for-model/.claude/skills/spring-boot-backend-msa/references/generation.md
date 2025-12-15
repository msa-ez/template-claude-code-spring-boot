# Backend Project Generation Rules

### Precautions
- All component generation must be based on metadata and must be generated without omissions. However, you must not arbitrarily create components by referencing non-existent metadata.
- Do not fall into error correction loops until all files are generated.

### Step-by-Step Component Generation Process
Projects must be created strictly following the numbered order below.

#### 1. Metadata Analysis, Validation and Package Structure Creation
- Analyze Aggregate (aggregates), Command (commands), Event (events), Policy metadata
- Refer to the "3. Project Structure" section of `@.claude/skills/spring-boot-backend-msa/references/architecture.md` to correct missing package structure

#### 2. Kafka Infrastructure Setup (Required Prerequisite)
- Generate Kafka infrastructure using the ddd-msa-infra-architect agent

#### 3. Gateway Setup
- Generate API Gateway configuration using the ddd-msa-infra-architect agent
- If gateway already exists, the agent will add routes to application.yml only

#### 4. Kafka Configuration Component Generation
- Refer to "1. KafkaProcessor Interface Definition" section of `@.claude/skills/spring-boot-backend-msa/references/domain-events.md`
- Generate `config/kafka/KafkaProcessor.java`

#### 5. Common Infrastructure Component Generation
- Refer to "2. AbstractEvent Class Implementation" section of `@.claude/skills/spring-boot-backend-msa/references/domain-events.md`
- Generate `infra/AbstractEvent.java` (includes publish(), publishAfterCommit() methods)

#### 6. Domain Component Generation

**Required**: All Aggregates in the metadata must be generated without omission.

**Generation Order (Considering Dependencies):**

##### 6.1 Repository Interface Generation
- Refer to "3.2 Layer-wise Roles and Responsibilities > Domain Layer" section of `@.claude/skills/spring-boot-backend-msa/references/architecture.md`
- File location: `domain/[Aggregate]Repository.java`
- Extend JpaRepository<[Aggregate], Long>

##### 6.2 Domain Event Class Generation
- Refer to "3. Domain Event Class Implementation" section of `@.claude/skills/spring-boot-backend-msa/references/domain-events.md`
- File location: `domain/[Event].java`
- Extend AbstractEvent, copy fields through BeanUtils.copyProperties

##### 6.3 Aggregate Root Entity Generation
- Refer to "4. Event Publishing from Aggregate Root" section of `@.claude/skills/spring-boot-backend-msa/references/domain-events.md`
- File location: `domain/[Aggregate].java`
- **Annotations**: @Entity, @Table, @Data
- **Event Publishing**: Call publishAfterCommit() in @PostPersist/@PostUpdate hooks
- **Static Repository Method**: Implement static repository() method

##### 6.4 Value Object Generation (if exists)
- Refer to "3.2 Layer-wise Roles and Responsibilities > Domain Layer" section of `@.claude/skills/spring-boot-backend-msa/references/architecture.md`
- File location: `domain/[ValueObject].java`
- Use @Embeddable annotation

**Key Points:**
- Strictly adhere to generation order to satisfy dependencies: Repository → Events → Aggregate Root → Value Objects
- All JPA entities must have appropriate annotations
- Event publishing must use publishAfterCommit() for transactional safety

#### 7. Infrastructure Adapter Component Generation

##### 7.1 PolicyHandler Generation (if policies exist)
- Refer to "5. PolicyHandler Implementation" section of `@.claude/skills/spring-boot-backend-msa/references/domain-events.md`
- File location: `infra/PolicyHandler.java`
- **Condition**: Generate only when policy metadata exists
- Filter by event type using @StreamListener + condition header
- Number of EventNames in metadata = Number of @StreamListener methods (must match)

##### 7.2 REST Controller Generation (optional)
- Refer to "3.2 Layer-wise Roles and Responsibilities > Infrastructure Layer" section of `@.claude/skills/spring-boot-backend-msa/references/architecture.md`
- File location: `infra/[Aggregate]Controller.java`
- Custom endpoints complementing Spring Data REST

##### 7.3 HATEOAS Processor Generation (optional)
- Refer to "3.2 Layer-wise Roles and Responsibilities > Infrastructure Layer" section of `@.claude/skills/spring-boot-backend-msa/references/architecture.md`
- File location: `infra/[Aggregate]HateoasProcessor.java`
- Add custom HATEOAS links by implementing ResourceProcessor

#### 8. Application Class Generation
- Refer to "7. Spring Boot Application Class" section of `@.claude/skills/spring-boot-backend-msa/references/domain-events.md`
- File location: `[ServiceName]Application.java`
- **Required Annotations**: @SpringBootApplication, @EnableBinding(KafkaProcessor.class)
- **Static ApplicationContext**: Required for Bean access from AbstractEvent

#### 9. Configuration File Generation

##### 9.1 Application Configuration
- Refer to "6. application.yml Configuration" section of `@.claude/skills/spring-boot-backend-msa/references/domain-events.md`
- Generate service application configuration using the ddd-msa-infra-architect agent
- File location: `resources/application.yml`
- **Required Configuration**: server.port (unique), spring.application.name, spring.cloud.stream.bindings
- **Profiles**: default (localhost:9092), docker (my-kafka:9092)

#### 10. Maven Configuration Generation
- Refer to "1.3 Maven Dependencies (pom.xml)" section of `@.claude/skills/spring-boot-backend-msa/references/architecture.md`
- File location: `pom.xml`
- **Required Versions**: Spring Boot 2.3.1.RELEASE, Spring Cloud Hoxton.SR12, Spring Cloud Stream Germantown.SR1
- **Required Dependencies**: spring-cloud-starter-stream-kafka, spring-boot-starter-data-jpa, H2, Lombok, HATEOAS

#### 11. Container and Deployment File Generation
- Generate Dockerfile and Kubernetes manifests using the ddd-msa-infra-architect agent
- **File Locations**: 
  - `Dockerfile`
  - `kubernetes/deployment.yaml`
  - `kubernetes/service.yaml`

#### 12. Common File Generation (Gateway, Kafka)
- Generate shared infrastructure using the ddd-msa-infra-architect agent
- The agent will check if gateway/, infra/ (kafka - docker-compose.yml) exist at root path
- If not exist, generate full configuration
- If exist, handle gateway application.yml route addition and kafka topic settings

#### 13. Test Code Generation, Testing and Execution (Final)
- Generate JUnit tests using the prd-junit-test-generator agent, which specializes in the Given-When-Then testing pattern
- Test all domain logic and event publishing
- Maven build and test execution: `mvn clean install`
- Validate local execution: `mvn spring-boot:run`
- Repeat test - error correction process until tests succeed

### Important Requirements
- All component generation must be based on metadata. Do not arbitrarily create components by referencing non-existent metadata.
- Generate components only when required metadata fields exist
- Adhere to dependency order: Repository → Events → Entities → Infrastructure
- All files must be generated according to requirements before error correction
