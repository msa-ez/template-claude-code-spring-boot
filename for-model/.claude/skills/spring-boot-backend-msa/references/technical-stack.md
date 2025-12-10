# Technical Stack Rules
## Instructions

Use this skill when:
- Setting up a new Spring Boot microservice project
- Understanding the technology stack requirements
- Implementing DDD, Clean Architecture, and Event-Driven Architecture patterns
- Configuring Maven dependencies and frameworks

When generating code, strictly adhere to the following frameworks, technologies, databases and utilities, architecture patterns and design methodologies.

---


## Core Frameworks

1. **Application Framework**: Spring Boot 2.3.1.RELEASE
2. **Development Environment**: Java 11
3. **Package Namespace**: Use Javax packages (javax.persistence.*, javax.transaction.*, etc.)

---


## Main Technologies

### 1. Data Access Layer
**Spring Data JPA**
- H2 Database (runtime embedded DB)
- Domain modeling based on JPA Entity
- Repository pattern with JpaRepository

### 2. RESTful API
**Spring Data REST**
- Automatic REST endpoint exposure
- Hypermedia-based API with HATEOAS support
- Resource processors for custom links

### 3. Microservice Communication
**Spring Cloud Stream** (Germantown.SR1 version)
- Channel definition based on `@Input`/`@Output` annotations
- Event reception through `@StreamListener`
- Event publication through `MessageChannel`
- Kafka binding for event streaming

### 4. DDD Pattern Implementation
- Aggregate Root definition through `@Entity`
- Repository pattern based on JpaRepository
- Domain event-based communication
- Value objects with `@Embeddable`

### 5. Lombok
**Version**: 1.18.12
- `@Data`: Auto-generate getter, setter, toString, equals, hashCode
- `@Builder`: Builder pattern implementation
- `@NoArgsConstructor`, `@AllArgsConstructor`: Constructor generation

---


## Test Methodology

Refer to `@test-code-generation-rules` for detailed test implementation specifications.

1. **Test-Driven Development (TDD)**
   - Unit testing using JUnit
   - Integration testing using Spring Test
   - Given-When-Then pattern
   - `@DataJpaTest` for repository testing

---


## Messaging and Event Processing

Refer to `@domain-event-publishing-rules` for messaging and event processing configuration.

**Maven Dependencies**:
- `spring-cloud-starter-stream-kafka`: Kafka binding
- `spring-cloud-dependencies` (Hoxton.SR12)
- `spring-cloud-stream-dependencies` (Germantown.SR1)

---


## Architecture Patterns and Design Methodologies

### 1. Domain-Driven Design (DDD)
- **Domain modeling** based on Event Storming
- **Aggregate Root**: Core domain entity expressed with `@Entity`
- **Domain Event**: Event class extending AbstractEvent
- **Repository**: Extend JpaRepository interface
- **Loose coupling** between services through domain events

### 2. Clean Architecture (Hexagonal Architecture)

#### Domain Package (Core Business Logic)
- Aggregate Root Entity
- Domain Event
- Repository interface
- Business methods (implemented as static methods)

#### Infrastructure Package (External System Integration)
- **Controller**: Inbound adapter (REST API)
- **PolicyHandler**: Inbound adapter (event reception)
- **AbstractEvent**: Outbound adapter (event publication)

### 3. Event-Driven Architecture (EDA)
- **Asynchronous communication** based on Pub/Sub pattern
- **Event sourcing**: Utilize JPA Lifecycle Hooks (`@PostPersist`, `@PostUpdate`, etc.)
- **Event publication**: Guarantee transaction consistency through `publishAfterCommit()`
- **Event reception**: Selective subscription through `@StreamListener` + condition

### 4. Microservices Architecture (MSA)
- Each service owns **independent database** (H2)
- **Automatic RESTful API** exposure through Spring Data REST
- **Event-based communication** between services through Kafka
- Each service **independently deployable** (Docker + Kubernetes)

---


## Key Technologies Summary

| Category | Technology | Version |
|---
---
---
-|---
---
---
--|---
---
---
|
| Framework | Spring Boot | 2.3.1.RELEASE |
| Language | Java | 11 |
| Data Access | Spring Data JPA | - |
| Database | H2 | Runtime |
| REST API | Spring Data REST | - |
| Messaging | Spring Cloud Stream | Germantown.SR1 |
| Event Broker | Apache Kafka | - |
| Testing | JUnit | - |
| Code Generation | Lombok | 1.18.12 |
| Containerization | Docker | - |
| Orchestration | Kubernetes | - |

---


## Maven Dependencies (pom.xml)

Key dependencies required:
```xml
<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>

    <!-- H2 Database -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Spring Cloud Stream -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-kafka</artifactId>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.12</version>
        <scope>provided</scope>
    </dependency>

    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---


## Development Principles

1. **Convention over Configuration**: Leverage Spring Boot auto-configuration
2. **Metadata-Driven**: Generate code strictly based on Event Storming metadata
3. **Transaction Management**: Use `@Transactional` for consistency
4. **Event-Driven**: Publish domain events for state changes
5. **RESTful**: Follow REST principles with proper HTTP methods
6. **Testable**: Write tests before implementation (TDD)
