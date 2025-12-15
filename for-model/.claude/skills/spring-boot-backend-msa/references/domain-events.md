# Domain Event Publishing Rules

### Core Components
1. **KafkaProcessor Interface**: Spring Cloud Stream channel binding definition
2. **AbstractEvent Class**: Parent class of all domain events, includes event publishing logic
3. **Domain Event Classes**: Represent specific events by inheriting from AbstractEvent (e.g., `OrderPlaced`, `DeliveryStarted`)
4. **Aggregate Root**: Defined as @Entity, publishes events from JPA lifecycle hooks
5. **PolicyHandler**: Receives and processes events from external services using @StreamListener

### Event Publishing Mechanism
1. Use JPA lifecycle hooks (@PostPersist, @PostUpdate, etc.) to create events in aggregate root
2. Pass the aggregate as constructor parameter when creating domain event instance
3. Call `publishAfterCommit()` to schedule event publishing after transaction commit
5. Send message to Kafka through KafkaProcessor's outboundTopic()
6. Automatically set event type in message header (headers['type'] = event class SimpleName)

### Event Publishing Implementation

#### 1. KafkaProcessor Interface Definition (config/kafka package)
Refer to `@fixed-generation-rules` to generate KafkaProcessor.java

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

```
public class OrderPlaced extends AbstractEvent {
    private Long id;
    private Integer qty;

    public OrderPlaced(Order aggregate) {
        super(aggregate);  // Automatically copy fields using BeanUtils
    }

    public OrderPlaced() {
        super();
    }
}
```

#### 4. Event Publishing from Aggregate Root (domain package)
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
@Service
@Transactional
public class PolicyHandler {

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
        Delivery.startDelivery(event); // Write actual business logic here
    }
}
```

#### 6. application.yml Configuration

**Configuration Rules:**
1. Destination (topic name): project name (e.g., "labshoppubsub")
2. Group (group name): set service name for event-in (e.g., "order", "delivery")
3. Broker: localhost:9092 (default), my-kafka:9092 (docker)
4. Refer to `@fixed-generation-rules` for application.yml modifications

#### 7. Spring Boot Application Class
Important: Store ApplicationContext statically to enable Bean access from AbstractEvent

```
@SpringBootApplication
@EnableBinding(KafkaProcessor.class)
public class GatewayApplication {
    public static ApplicationContext applicationContext;

    public static void main(String[] args) {
        applicationContext = SpringApplication.run(GatewayApplication.class, args);
    }
}
```