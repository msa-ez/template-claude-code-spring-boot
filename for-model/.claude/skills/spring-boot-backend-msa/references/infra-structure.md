# Infrastructure Code Generator Requirements

## Overview
Infrastructure example code to be used in DDD MSA architecture document (`.claude/agents/msa-infra-architect.md`)

## Core Requirements

### 1. Template Type Classification
1. dockerfile: Dockerfile template
2. k8s: Kubernetes manifests (deployment.yaml, service.yaml)
3. gateway: API Gateway configuration (pom.xml, GatewayApplication.java, application.yml)
4. kafka: Kafka infrastructure (docker-compose.yml, KafkaProcessor.java)
5. appConfig: Service application configuration (application.yml)

### 2. Placeholder Substitution
Replace the following placeholders with actual values:
- `[serviceName]`: Bounded Context name (e.g., order, delivery, inventory)
- `[projectName]`: Project name (e.g., ecommerce)
- `[servicePort]`: Service port (e.g., 8081, 8082, 8083)
- `[packageName]`: Package name (e.g., com.ecommerce.order)
- `[resourcePath]`: Resource path (e.g., orders, deliveries, inventories)
- `[username]`: Docker username

## Generated File Structure

```
Bounded Context Name/
  ├── Dockerfile
  ├── kubernetes/
  │   ├── deployment.yaml
  │   └── service.yaml
  └── src/main/
      ├── java/com/ecommerce/order/config/kafka/
      │   └── KafkaProcessor.java
      └── resources/
          └── application.yml

gateway/
  ├── pom.xml
  └── src/main/
      ├── java/gateway/
      │   └── GatewayApplication.java
      └── resources/
          └── application.yml

infra/
  └── docker-compose.yml
```

## Actual Code Generation Examples

### Example 1: order Bounded Context - Dockerfile
Location: `[serviceName]/Dockerfile`

```dockerfile
FROM openjdk:11-jdk-slim
COPY target/*SNAPSHOT.jar app.jar
EXPOSE 8080
ENV TZ=Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
ENTRYPOINT ["java","-Xmx400M","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar","--spring.profiles.active=docker"]
```

#### Usage Example
```bash
node infra-code-generate.js -bc order -p ecommerce -t dockerfile
```

### Example 2: order Bounded Context - Kubernetes Deployment
Location: `[serviceName]/kubernetes/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: [serviceName]
  labels:
    app: [serviceName]
spec:
  replicas: 1
  selector:
    matchLabels:
      app: [serviceName]
  template:
    metadata:
      labels:
        app: [serviceName]
    spec:
      containers:
        - name: [serviceName]
          image: "[username]/[serviceName]:latest"
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "docker"
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
```

#### Usage Example
```bash
node infra-code-generate.js -bc order -p ecommerce -u myusername -t k8s
```

### Example 3: order Bounded Context - Application Configuration
Location: `[serviceName]/src/main/resources/application.yml`

```yaml
server:
  port: [servicePort]
---

spring:
  profiles: default
  jpa:
    properties:
      hibernate:
        show_sql: true
        format_sql: true
  cloud:
    stream:
      kafka:
        binder:
          brokers: localhost:9092
      bindings:
        event-in:
          group: [serviceName]
          destination: [projectName]
          contentType: application/json
        event-out:
          destination: [projectName]
          contentType: application/json
```

#### Usage Example
```bash
node infra-code-generate.js -bc order -p ecommerce --port 8081 -t appConfig
```

### Example 4: API Gateway Configuration
Location: `gateway/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.[projectName]</groupId>
    <artifactId>gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
    </parent>
    <properties>
        <java.version>11</java.version>
        <spring-cloud.version>2021.0.8</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

Location: `gateway/src/main/java/gateway/GatewayApplication.java`

```java
package gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

Location: `gateway/src/main/resources/application.yml`

```yaml
server:
  port: 8088
---
spring:
  profiles: default
  cloud:
    gateway:
      routes:
        - id: [serviceName]
          uri: http://localhost:[servicePort]
          predicates:
            - Path=/[resourcePath]/**
        - id: frontend
          uri: http://localhost:8080
          predicates:
            - Path=/**
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOriginPatterns: ["*"]
            allowedMethods: ["*"]
            allowedHeaders: ["*"]
            allowCredentials: true
---
spring:
  profiles: docker
  cloud:
    gateway:
      routes:
        - id: [serviceName]
          uri: http://[serviceName]:8080
          predicates:
            - Path=/[resourcePath]/**
        - id: frontend
          uri: http://frontend:8080
          predicates:
            - Path=/**
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: ["*"]
            allowedMethods: ["*"]
            allowedHeaders: ["*"]
            allowCredentials: true
server:
  port: 8080
```

#### Usage Example
```bash
node infra-code-generate.js -bc order -p ecommerce --port 8081 -t gateway
```

### Example 5: Kafka Infrastructure
Location: `infra/docker-compose.yml`

```yaml
services:
  kafka:
    image: confluentinc/cp-kafka:latest
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      CLUSTER_ID: "kraft-cluster-01"
      KAFKA_KRAFT_MODE: "true"
      KAFKA_PROCESS_ROLES: "broker,controller"
      KAFKA_NODE_ID: "1"
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka:9093"
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
      KAFKA_LOG_RETENTION_HOURS: 168
      KAFKA_LOG_DIRS: "/var/lib/kafka/data"
    volumes:
      - kafka-data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD-SHELL", "kafka-topics --list --bootstrap-server localhost:9092 || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
volumes:
  kafka-data:
```

#### Usage Example
```bash
node infra-code-generate.js -bc order -p ecommerce -t kafka
```
