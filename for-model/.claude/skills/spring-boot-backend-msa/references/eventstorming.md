# Event Storming Sticker-Based Rules
## Instructions

Use this skill when:
- Generating backend code from Event Storming metadata
- Creating Event, Command, Policy, and Aggregate components
- Implementing DDD-based microservice structure
- Following metadata-driven code generation patterns

### Important Notes
**Files must be generated strictly based on metadata, and no arbitrary data not in metadata should be generated.**

---


## Event

### 1. Reference Metadata
Events use metadata under 'events' to publish events when business logic processing is complete.

### 2. Generated Files

**Event.java**
- **Filename**: Event sticker name.java
- **Structure**:
  - Declare fieldDescriptors
  - Constructor that receives Aggregate as parameter
  - Declare default constructor

---


## Command

### 1. Reference Metadata
Commands use metadata under 'commands', and isRestRepository determines whether to use CRUD API or Extended API. When using Extended API, command files corresponding to DTOs must be generated through fieldDescriptors.

### 2. Generated Files

**Command.java**
- **Filename**: Command sticker name + Command.java
- **Generation Condition**: When isRestRepository is false
- **Structure**:
  - Declare fieldDescriptors

**Controller.java**
- **Filename**: Aggregate sticker name + Controller.java
- **Structure**:
  - When commands using extended API exist, generate extended API code based on controllerInfo and generated Command files
  - For default API, generate files and handle CRUD with Repository pattern
  - All above requirements must be generated in a single Controller.java file

---


## Policy

### 1. Reference Metadata and Role

Handles domain events defined in policies.incomingRelations.source resources of the model metadata. Refer to @domain-event-publishing-rules for event reception and handling implementation.

Declare port methods within the Aggregate Root Entity with Policy name, passing received events to implement business logic.

### 2. Generated Files

**PolicyHandler.java**
- **Filename**: PolicyHandler.java
- **Structure**:
  - Declare listening events
  - Declare connected Policies according to event declarations
  - Call port methods of Aggregates generated based on Policies
  - If policies do not exist, generate PolicyHandler.java with default configuration

---


## Aggregate

### 1. Reference Metadata
Aggregates should be generated based on metadata in the 'aggregates' section, using fieldDescriptors representing attributes and class diagram relationships (Enumeration, ValueObject, Entity) existing in aggregateRoot.entities.relations.

### 2. Generated Files

**AggregateRoot.java**
- **Filename**: AggregateName.java
- **Structure**:
  - Declare attributes based on fieldDescriptors
  - lifeCycle (only generated when command metadata's isRestRepository is false)
  - Declare business logic methods for inbound adapters that publish events (extended API controller, eventListener policyHandler)
  - Refer to @domain-event-publishing-rules for event publication implementation

**Repository.java**
- **Filename**: AggregateName + Repository.java
- **Structure**:
  - Extend JPARepository
  - Write queries referring to queryOption if parameters of views or ReadModels exist

---


## Generation Workflow

1. **Analyze Metadata**: Parse Event Storming metadata (events, commands, policies, aggregates)
2. **Generate Events**: Create event classes extending AbstractEvent
3. **Generate Commands**: Create command DTOs when isRestRepository is false
4. **Generate Aggregates**: Create aggregate roots with JPA entities
5. **Generate Repositories**: Create repository interfaces extending JpaRepository
6. **Generate Controllers**: Create REST controllers for extended APIs
7. **Generate PolicyHandlers**: Create event listeners for inter-service communication

## Key Principles

- **Metadata-Driven**: All code generation must strictly follow metadata
- **DDD Patterns**: Implement Domain-Driven Design patterns (Aggregate, Entity, Value Object)
- **Event-Driven**: Use domain events for state changes and inter-service communication
- **RESTful APIs**: Generate REST endpoints based on aggregate names
- **Type Safety**: Use proper Java types based on fieldDescriptors
