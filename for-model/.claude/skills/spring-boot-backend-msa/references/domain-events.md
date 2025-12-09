# Domain Event Publishing Rules
## Instructions

Use this skill when:
- Implementing domain event publishing in aggregate roots
- Setting up event consumption between microservices
- Configuring Kafka-based event streaming
- Ensuring transactional consistency in event publication
- Implementing PolicyHandlers for inter-service communication

### Core Components

1. **KafkaProcessor Interface**: Defines Spring Cloud Stream channel bindings
2. **AbstractEvent Class**: Parent class for all domain events, containing event publishing logic
3. **Domain Event Classes**: Inherit from AbstractEvent to represent specific events (e.g., `OrderPlaced`, `DeliveryStarted`)
4. **Aggregate Root**: Defined with @Entity, publishes events from JPA Lifecycle Hooks
5. **PolicyHandler**: Receives and handles events from external services using @StreamListener

### Event Publishing Mechanism

1. Generate events from Aggregate Root using JPA Lifecycle Hooks (@PostPersist, @PostUpdate, etc.)
2. Pass Aggregate as constructor parameter when creating Domain Event instances
3. Call `publishAfterCommit()` to schedule event publication after transaction commit
4. Actual publication occurs at transaction commit time through TransactionSynchronizationManager
5. Send messages to Kafka through KafkaProcessor's outboundTopic()
6. Automatically set event type in message header (headers['type'] = event class SimpleName)

### Event Publishing Implementation

#### 1. KafkaProcessor Interface Definition (config/kafka package)
Generate KafkaProcessor.java referring to `@fixed-generation-rules`

#### 2. AbstractEvent Class Implementation (infra package)
```
package [projectname].infra;

import [projectname].[ServiceName]Application;
import [projectname].config.kafka.KafkaProcessor;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.BeanUtils;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.transaction.support.TransactionSynchronizationAdapter;
import org.springframework.transaction.support.TransactionSynchronizationManager;
import org.springframework.util.MimeTypeUtils;

public class AbstractEvent {
    String eventType;
    Long timestamp;

    public AbstractEvent(Object aggregate) {
        this();
        BeanUtils.copyProperties(aggregate, this);
    }

    public AbstractEvent() {
        this.setEventType(this.getClass().getSimpleName());
        this.timestamp = System.currentTimeMillis();
    }

    public void publish() {
        KafkaProcessor processor = [ServiceName]Application.applicationContext.getBean(
            KafkaProcessor.class
        );
        MessageChannel outputChannel = processor.outboundTopic();

        outputChannel.send(
            MessageBuilder
                .withPayload(this)
                .setHeader(
                    MessageHeaders.CONTENT_TYPE,
                    MimeTypeUtils.APPLICATION_JSON
                )
                .setHeader("type", getEventType())
                .build()
        );
    }

    public void publishAfterCommit() {
        TransactionSynchronizationManager.registerSynchronization(
            new TransactionSynchronizationAdapter() {
                @Override
                public void afterCompletion(int status) {
                    AbstractEvent.this.publish();
                }
            }
        );
    }

    // getters and setters
    public String getEventType() {
        return eventType;
    }

    public void setEventType(String eventType) {
        this.eventType = eventType;
    }

    public Long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Long timestamp) {
        this.timestamp = timestamp;
    }

    public boolean validate() {
        return getEventType().equals(getClass().getSimpleName());
    }
}
```

#### 3. Domain Event Class Implementation (domain package)

Example:
```
package [projectname].domain;

import [projectname].infra.AbstractEvent;
import lombok.Data;
import lombok.ToString;

@Data
@ToString
public class OrderPlaced extends AbstractEvent {
    private Long id;
    private String productId;
    private Integer qty;
    private String customerId;

    public OrderPlaced(Order aggregate) {
        super(aggregate);  // Auto-copy fields using BeanUtils
    }

    public OrderPlaced() {
        super();
    }
}
```

#### 4. Publishing Events from Aggregate Root (domain package)

Example:
```
@PostPersist
public void onPostPersist() {
    OrderPlaced orderPlaced = new OrderPlaced(this);
    orderPlaced.publishAfterCommit();  // Publish after transaction commit
}
```

### Event Consumption Implementation

#### 5. PolicyHandler Implementation (infra package)
```
package [projectname].infra;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import javax.transaction.Transactional;
import [projectname].config.kafka.KafkaProcessor;
import [projectname].domain.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Service;

@Service
@Transactional
public class PolicyHandler {

    @Autowired
    DeliveryRepository deliveryRepository;

    @StreamListener(KafkaProcessor.INPUT)
    public void whatever(@Payload String eventString) {}

    @StreamListener(
        value = KafkaProcessor.INPUT,
        condition = "headers['type']=='OrderPlaced'"
    )
    public void wheneverOrderPlaced_StartDelivery(
        @Payload OrderPlaced orderPlaced
    ) {
        OrderPlaced event = orderPlaced;
        System.out.println(
            "\n\n##### listener StartDelivery : " + orderPlaced + "\n\n"
        );

        // Execute business logic
        Delivery.startDelivery(event);
    }
}
```

#### 6. application.yml Configuration

**Configuration Rules:**
1. Destination (Topic Name): ProjectName (e.g., "labshoppubsub")
2. Group (Group Name): Set service name in event-in (e.g., "order", "delivery")
3. Broker: localhost:9092 (default), my-kafka:9092 (docker)
4. Refer to `@fixed-generation-rules` for application.yml modifications

#### 7. Spring Boot Application Class
Important: Store ApplicationContext statically to enable Bean access from AbstractEvent

```
package [projectname];

import [projectname].config.kafka.KafkaProcessor;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
@EnableBinding(KafkaProcessor.class)
public class OrderApplication {
    public static ApplicationContext applicationContext;

    public static void main(String[] args) {
        applicationContext = SpringApplication.run(OrderApplication.class, args);
    }
}
```

### Key Features

#### Transaction Consistency Guarantee
- `publishAfterCommit()`: Publish events only after transaction commit
- Utilize `TransactionSynchronizationManager`
- Maintain data consistency by not publishing events if DB save fails

#### Event Filtering
- Distinguish event types by 'type' field in message header
- Selective subscription using `@StreamListener`'s condition attribute
- Each PolicyHandler processes only events of interest

#### Kafka Infrastructure Integration
- Integrate with Kafka settings in docker-compose.yml defined in @fixed-generation-rules
- All services pub/sub to the same topic (project name)
- Each service instance is distinguished by Consumer Group

### ✅ Correct Implementation Example

```
@Service
@Transactional
public class PolicyHandler {

    @Autowired
    DeliveryRepository deliveryRepository;

    // Empty handler (Required): Consumes unprocessed events to clean up logs
    @StreamListener(KafkaProcessor.INPUT)
    public void whatever(@Payload String eventString) {}

    // Process specific events only (filter by condition)
    @StreamListener(
        value = KafkaProcessor.INPUT,
        condition = "headers['type']=='OrderPlaced'"
    )
    public void wheneverOrderPlaced_StartDelivery(
        @Payload OrderPlaced orderPlaced
    ) {
        // Receive events only from external service (Order)
        Delivery.startDelivery(orderPlaced);
    }
}
```

### ❌ Anti-patterns to Avoid

```
// Bad example: Using Object parameter without condition
@StreamListener(KafkaProcessor.INPUT)
public void whenAnyEvent(@Payload Object event) {
    // Attempting to process all events → Process own events → Infinite loop risk
}

// Bad example: Publishing new events during event processing without transaction consideration
@StreamListener(value = KafkaProcessor.INPUT, condition = "headers['type']=='OrderPlaced'")
public void handleOrder(@Payload OrderPlaced event) {
    Delivery delivery = new Delivery();
    repository.save(delivery);  // Publishes event in @PostPersist
    // → DeliveryStarted event can come back to its own PolicyHandler
}
```
