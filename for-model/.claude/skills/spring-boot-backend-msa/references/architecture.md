architecture.md# Spring Boot MSA Architecture Guide

## 1. Technology Stack

### 1.1 Core Frameworks
- **Application Framework**: Spring Boot 2.3.1.RELEASE
- **Development Environment**: Java 11
- **Package Namespace**: Using Javax packages (javax.persistence.*, javax.transaction.*, etc.)

### 1.2 Key Technologies

#### Data Access Layer
**Spring Data JPA**
- H2 Database (embedded DB at runtime)
- JPA Entity-based domain modeling
- Repository pattern using JpaRepository

#### RESTful API
**Spring Data REST**
- Automatic REST endpoint exposure
- HATEOAS-based hypermedia API support
- Resource processors for custom links

#### Microservice Communication
**Spring Cloud Stream** (Germantown.SR1 version)
- Channel definition based on `@Input`/`@Output` annotations
- Event reception via `@StreamListener`
- Event publishing through `MessageChannel`
- Kafka binding for event streaming

#### Lombok
**Version**: 1.18.12
- `@Data`: Auto-generation of getter, setter, toString, equals, hashCode
- `@Builder`: Builder pattern implementation
- `@NoArgsConstructor`, `@AllArgsConstructor`: Constructor generation

### 1.3 Maven Dependencies (pom.xml)

Required dependencies:
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

**Additional dependency management**:
- `spring-cloud-dependencies` (Hoxton.SR12)
- `spring-cloud-stream-dependencies` (Germantown.SR1)

---

## 2. Architecture Patterns

### 2.1 Domain-Driven Design (DDD)
- **Domain modeling** based on Event Storming
- **Aggregate Root**: Core domain entity represented as `@Entity`
- **Domain Events**: Event classes extending AbstractEvent
- **Repository**: Extends JpaRepository interface
- **Loose coupling** between services through domain events

### 2.2 Clean Architecture (Hexagonal Architecture)

#### Domain Package (Core Business Logic)
- **Aggregate Root Entity**: Implementation of business rules and logic
- **Domain Events**: Important events occurring in the domain
- **Repository Interface**: Persistence abstraction
- **Value Objects/Entities**: Domain model components

#### Infrastructure Package (External System Integration)
- **Controller** (Inbound Adapter): External request handling through REST API
- **PolicyHandler** (Inbound Adapter): External event reception and business logic triggering
- **AbstractEvent** (Outbound Adapter): Domain event publishing mechanism

### 2.3 Event-Driven Architecture (EDA)
- **Asynchronous Communication**: Based on Pub/Sub pattern
- **Event Sourcing**: Utilizing JPA lifecycle hooks (`@PostPersist`, `@PostUpdate`, etc.)
- **Transactional Consistency**: Guaranteed event publishing through `publishAfterCommit()`
- **Selective Subscription**: Event type filtering through `@StreamListener` + condition
- For detailed event publishing and reception implementation specifications, refer to `@domain-event-publishing-rules`

### 2.4 Microservice Architecture (MSA)
- Each service owns an **independent database** (H2)
- **Automatic RESTful API** exposure through Spring Data REST
- **Event-based communication** between services through Kafka
- Each service **independently deployable** (Docker + Kubernetes)

---

## 3. Project Structure

### 3.1 Overall Package Structure

Based on the provided metadata, generate the following structure and files:

```
/gateway # gateway route configuration (optional)
  /src/main
    /java/[projectname]
      /Application.java
    /resources
      /application.yml
  /pom.xml

/infra # docker-compose based Kafka infrastructure (required)
  /docker-compose.yml # Zookeeper + Kafka configuration

/[service] # Microservice directory created with service name
  /src
    /main
      /java/[projectname]
        /[ServiceName]Application.java # @SpringBootApplication + @EnableBinding(KafkaProcessor.class)

        /config
          /kafka
            /KafkaProcessor.java # Spring Cloud Stream channel interface (@Input/@Output)

        /domain
          /[Aggregate].java # Aggregate root defined as @Entity
          /[Aggregate]Repository.java # JpaRepository interface
          /[Event].java # Domain event class extending AbstractEvent
          /[ValueObject].java # Embedded value object (optional)
          /[Entity].java # Entity within aggregate (optional)

        /infra
          /AbstractEvent.java # Parent class of all domain events (publish(), publishAfterCommit())
          /[Aggregate]Controller.java # REST API controller
          /[Aggregate]HateoasProcessor.java # HATEOAS resource processor (optional)
          /PolicyHandler.java # Receives external events via @StreamListener

      /resources
        /application.yml # Spring Cloud Stream binding configuration (event-in, event-out)

    /test
      /java/[projectname]
        /[ServiceName]Test.java # JUnit test

  /kubernetes
    /deployment.yaml # Kubernetes Deployment manifest
    /service.yaml # Kubernetes Service manifest

  /pom.xml # Maven dependencies
  /Dockerfile # Container image build configuration
  /README.md # Service documentation (optional)
```

### 3.2 Layer-wise Roles and Responsibilities

#### Application Layer
**[ServiceName]Application.java**
- `@SpringBootApplication`: Spring Boot application entry point
- `@EnableBinding(KafkaProcessor.class)`: Spring Cloud Stream activation

#### Config Layer
**KafkaProcessor.java**
- Spring Cloud Stream messaging channel interface
- `@Input`: Event reception channel definition
- `@Output`: Event publishing channel definition

#### Domain Layer (Core Business Logic)

**[Aggregate].java**
- `@Entity`: Define aggregate root as JPA entity
- Implement domain business logic methods
- Event publishing through JPA lifecycle hooks (`@PostPersist`, `@PostUpdate`, etc.)

**[Aggregate]Repository.java**
- Extends `JpaRepository` interface
- Automatically provides basic CRUD methods
- Custom query methods (if needed)

**[Event].java**
- Extends `AbstractEvent`
- Define domain event fields
- Event constructor and default constructor

**[ValueObject].java** (optional)
- `@Embeddable`: Embedded value object
- Encapsulates domain concepts

**[Entity].java** (optional)
- Entity within aggregate
- Depends on aggregate root

#### Infrastructure Layer (External System Integration)

**AbstractEvent.java**
- Parent class of all domain events
- `publish()`: Immediate event publishing
- `publishAfterCommit()`: Event publishing after transaction commit
- Event transmission mechanism through Kafka

**[Aggregate]Controller.java**
- Complements auto-generated API from Spring Data REST
- API endpoints requiring custom business logic
- `@RestController` annotation

**[Aggregate]HateoasProcessor.java** (optional)
- HATEOAS resource processor
- Add custom links

**PolicyHandler.java**
- `@Service` annotation
- `@StreamListener`: Receives external domain events
- Event type filtering through condition attribute
- Aggregate root lookup and business logic execution through Repository

#### Resources Layer

**application.yml**
- Spring Cloud Stream binding configuration
  - `spring.cloud.stream.bindings.event-in`: Input channel configuration
  - `spring.cloud.stream.bindings.event-out`: Output channel configuration
- Kafka broker configuration
- Database configuration (H2)
- Server port configuration

## 4. Testing Methodology
For detailed test implementation specifications, refer to `@test-code-generation-rules`.
