# Backend Project Generation Rules
## Instructions

Use this skill when:
- Generating complete backend microservice code from metadata
- Creating DDD-based Spring Boot project structure
- Implementing Kafka-based event-driven microservices
- Ensuring all required components are generated in systematic order
- Building complete projects including Gateway and infrastructure setup
- Generating and executing test code for complete project construction

Follow these requirements when generating projects:

## Project Generation Order

Projects must be generated in the following order according to the prompt:

Package structure creation → Domain and infra files generation → pom.xml generation → Application.java and application.yml generation → Fixed files generation (Dockerfile, Manifests, gateway, kafka) → Test code generation → Test & Execution (mandatory)

## Essential Rules

All files must be generated according to requirements before error correction. Do not fall into error correction loops during file generation.

## Project Generation Requirements

### Basic Principles

1. **Metadata-Based Generation**: All files are generated based on metadata and must not be generated with data not included in metadata.

2. **No Reference to Existing Root Files**: When generating projects, do not reference existing files in Root, and package structure follows metadata definitions.

3. **Rule Compliance**: Strictly adhere to @package-structure/SKILL.md, @technical-stack/SKILL.md, @domain-events/SKILL.md.

### Kafka Infrastructure Setup (Mandatory Prerequisite)

**Important**: Check and create Kafka infrastructure before generating any service

- If kafka/docker-compose.yml does not exist in Root, create it
  - Zookeeper (port 2181, image: confluentinc/cp-zookeeper:7.9.1)
  - Kafka (port 9092, image: confluentinc/cp-kafka:7.9.1)
- If it already exists, reuse it

### Gateway Setup

- If gateway directory does not exist in Root, create it according to @fixed-generation-rules
- If it already exists, add new service routes to gateway/src/main/resources/application.yml

### Detailed File Generation Order

#### Step 1: Package Structure Creation

```
/[service]
  /src/main/java/[projectname]
    /config/kafka
    /domain
    /infra
  /src/main/resources
  /kubernetes
```

#### Step 2: Kafka Configuration Files

- `config/kafka/KafkaProcessor.java`
  - INPUT = "event-in", OUTPUT = "event-out" constants
  - Methods with @Input/@Output annotations

#### Step 3: Common Infrastructure Files

- `infra/AbstractEvent.java`
  - publish(), publishAfterCommit() methods
  - BeanUtils.copyProperties logic
  - TransactionSynchronizationManager utilization

#### Step 4: Domain Files Generation

**Generation Order (considering dependencies):**

1. Repository interface (extends JpaRepository)
2. Domain event classes (extends AbstractEvent)
3. Aggregate Root entity
   - @Entity, @Table, @Data annotations
   - Publish events in @PostPersist/@PostUpdate etc.
   - static repository() method
4. Value Objects (if exist)

#### Step 5: Infrastructure Adapters

- `infra/PolicyHandler.java` (if policies exist)
  - @StreamListener(KafkaProcessor.INPUT)
  - Filter with condition = "headers['type']=='EventName'"
- `infra/[Aggregate]Controller.java` (optional)
- `infra/[Aggregate]HateoasProcessor.java` (optional)

#### Step 6: Application Class

- `[ServiceName]Application.java`
  - @SpringBootApplication
  - @EnableBinding(KafkaProcessor.class)
  - public static ApplicationContext applicationContext

#### Step 7: Configuration Files

- `resources/application.yml`
  - server.port (unique port per service)
  - spring.application.name
  - spring.cloud.stream binding configuration
  - default profile (brokers: localhost:9092)
  - docker profile (brokers: my-kafka:9092)

#### Step 8: Maven Configuration

- `pom.xml`
  - Spring Boot 2.3.1.RELEASE
  - Spring Cloud Hoxton.SR12
  - Spring Cloud Stream Germantown.SR1
  - Required dependencies: spring-cloud-starter-stream-kafka, H2, Lombok, HATEOAS

#### Step 9: Container and Deployment Files

- `Dockerfile`
- `kubernetes/deployment.yaml`
- `kubernetes/service.yaml`

#### Step 10: Common Files Generation (Gateway, Kafka)

- Check if gateway/, infra/ (kafka - docker-compose.yml) exist in Root path
- If not exist, configure files through `@fixed-generation/SKILL.md`
- If exist, modify gateway application.yml routes and handle kafka processing and topic settings in application.yml referring to `@fixed-generation/SKILL.md`

#### Step 11: Test Code Generation (Final)

- Refer to `@test-generation/SKILL.md`
- JUnit-based Given-When-Then pattern

#### Step 12: Test and Execution (Mandatory)

- Execute Maven build and tests
- Verify local execution
- Must repeat test - error correction process until tests succeed referring to @test-generation/SKILL.md

## Error Correction Rules

- Do not fix errors until all files are generated
- Be careful not to fall into error correction loops during generation
