# Package Structure Rules
## Instructions

Use this skill when:
- Setting up the package structure for a new microservice
- Understanding the layered architecture organization
- Organizing domain, infrastructure, and configuration code
- Following DDD and Clean Architecture principles

### Package Structure Requirements

Generate the following files according to the structure and file descriptions based on provided metadata:

```
/gateway # gateway route settings @fixed-generation-rules (optional)
  /src/main
    /java/[projectname]
      /Application.java
    /resources
      /application.yml
  /pom.xml

/infra # Kafka infrastructure based on docker-compose (required)
  /docker-compose.yml # Zookeeper + Kafka configuration

/[service] # Microservice directory generated with service name
  /src
    /main
      /java/[projectname]
        /[ServiceName]Application.java # @SpringBootApplication + @EnableBinding(KafkaProcessor.class)

        /config
          /kafka
            /KafkaProcessor.java # Spring Cloud Stream channel interface (@Input/@Output)

        /domain
          /[Aggregate].java # Aggregate Root defined with @Entity
          /[Aggregate]Repository.java # JpaRepository interface
          /[Event].java # Domain event class extending AbstractEvent
          /[ValueObject].java # Embedded value object (optional)
          /[Entity].java # Entity within Aggregate (optional)

        /infra
          /AbstractEvent.java # Parent class for all domain events (publish(), publishAfterCommit())
          /[Aggregate]Controller.java # REST API Controller (complements Spring Data REST auto-generation, optional)
          /[Aggregate]HateoasProcessor.java # HATEOAS resource processor (optional)
          /PolicyHandler.java # Receive external events with @StreamListener (if policies exist)

      /resources
        /application.yml # Spring Cloud Stream binding configuration (event-in, event-out)

    /test
      /java/[projectname]
        /[ServiceName]Test.java # Cucumber/JUnit tests (optional)

  /kubernetes
    /deployment.yaml # Kubernetes Deployment manifest
    /service.yaml # Kubernetes Service manifest

  /pom.xml # Maven dependencies (Spring Boot 2.3.1.RELEASE)
  /Dockerfile # Container image build configuration
  /README.md # Service documentation (optional)
```

---

## Package Descriptions

### Application Root (`/[ServiceName]Application.java`)
- Main Spring Boot application class
- Annotated with `@SpringBootApplication` and `@EnableBinding(KafkaProcessor.class)`
- Stores static ApplicationContext for event publishing

### Config Package (`/config`)
- **kafka/KafkaProcessor.java**: Defines Spring Cloud Stream channels
  - `@Input` for consuming events
  - `@Output` for publishing events

### Domain Package (`/domain`)
Core business logic and domain model:
- **Aggregate Root**: Main entity with `@Entity`, business methods, and event publishing
- **Repository**: Interface extending `JpaRepository` for data access
- **Domain Events**: Event classes extending `AbstractEvent`
- **Value Objects**: Embedded objects with `@Embeddable` (optional)
- **Entities**: Child entities within aggregate (optional)

### Infrastructure Package (`/infra`)
Adapters for external systems:
- **AbstractEvent**: Base class for all events with publish/publishAfterCommit methods
- **Controller**: REST API endpoints (optional, complements Spring Data REST)
- **PolicyHandler**: Event listener with `@StreamListener` for inter-service communication
- **HateoasProcessor**: Resource processor for HATEOAS links (optional)

### Resources (`/resources`)
- **application.yml**: Configuration for JPA, Kafka, and Spring Cloud Stream
  - Two profiles: `default` (local), `docker` (containerized)

### Test Package (`/test`)
- **[ServiceName]Test.java**: JUnit/Cucumber tests following TDD
- Uses `@DataJpaTest` for JPA testing
- Given-When-Then pattern

### Kubernetes (`/kubernetes`)
- **deployment.yaml**: Kubernetes Deployment manifest with health probes
- **service.yaml**: Kubernetes Service manifest for network access

### Root Files
- **pom.xml**: Maven dependencies and build configuration
- **Dockerfile**: Container image build instructions
- **README.md**: Service documentation (optional)

---

## Architecture Layers

### 1. Domain Layer (`/domain`)
- Core business logic
- Technology-agnostic
- Contains aggregates, entities, value objects, repositories, domain events

### 2. Infrastructure Layer (`/infra`)
- External system integration
- REST API controllers (inbound adapter)
- Event listeners (inbound adapter)
- Event publishers (outbound adapter)

### 3. Configuration Layer (`/config`)
- Framework configuration
- Kafka channel bindings
- Application settings

---

## Key Principles

1. **Clean Architecture**: Clear separation between domain and infrastructure
2. **DDD**: Domain-driven design with aggregates and repositories
3. **Event-Driven**: Domain events for state changes and communication
4. **Microservices**: Independent deployable services with own database
5. **Spring Boot**: Framework-based with convention over configuration
