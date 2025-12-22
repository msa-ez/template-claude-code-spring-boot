# JUnit Test Code Generator Requirements

## Overview
JUnit test code examples to be used in PRD-driven test development document (`.claude/agents/prd-junit-test-generator.md`)

## Core Requirements

### Test Type
- **integrationTest**: Event-driven integration test using Given-When-Then pattern

### Placeholder Substitution
Replace the following placeholders with actual values:
- `[ServiceName]`: Service/Bounded Context name (e.g., Order, Delivery, Inventory)
- `[EntityName]`: Entity/Aggregate name (e.g., Order, Inventory, Delivery)
- `[RepositoryName]`: Repository interface name (e.g., OrderRepository, InventoryRepository)
- `[EventName]`: Event class name (e.g., OrderPlaced, StockDecreased)

## Test Code Structure

### Test Class Structure
```
src/
  └── test/
      └── java/
          └── [package]/
              └── [ServiceName]ApplicationTests.java
```

## Actual Test Code Generation Example

### Event-Driven Integration Test Pattern
Location: `src/test/java/[package]/[ServiceName]ApplicationTests.java`

```java
package com.example.inventory;

import static org.junit.Assert.*;

import com.example.inventory.config.kafka.KafkaProcessor;
import com.example.inventory.domain.*;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.cloud.stream.test.binder.MessageCollector;
import org.springframework.context.ApplicationContext;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.util.MimeTypeUtils;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class InventoryApplicationTests {

    private static final Logger LOGGER = LoggerFactory.getLogger(
        InventoryApplicationTests.class
    );

    @Autowired
    private KafkaProcessor processor;

    @Autowired
    private InventoryRepository repository;

    @Autowired
    private ApplicationContext applicationContext;

    @Autowired
    private MessageCollector messageCollector;

    @Test
    @SuppressWarnings("unchecked")
    public void test0() {
        //given:
        Inventory entity = new Inventory();
        entity.setId(1L);
        entity.setStock(10);
        entity.setProductName("TV");
        repository.save(entity);

        //when:
        OrderPlaced event = new OrderPlaced();
        event.setId(1L);
        event.setProductId("1");
        event.setQty(5);
        event.setCustomerId("CUST001");
        event.setProductName("TV");

        InventoryApplication.applicationContext = applicationContext;

        ObjectMapper objectMapper = new ObjectMapper()
            .configure(
                DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,
                false
            );
        try {
            String msg = objectMapper.writeValueAsString(event);

            processor
                .inboundTopic()
                .send(
                    MessageBuilder
                        .withPayload(msg)
                        .setHeader(
                            MessageHeaders.CONTENT_TYPE,
                            MimeTypeUtils.APPLICATION_JSON
                        )
                        .setHeader("type", event.getEventType())
                        .build()
                );

            //then:
            Message<String> received = (Message<String>) messageCollector
                .forChannel(processor.outboundTopic())
                .poll();

            assertNotNull("Resulted event must be published", received);

            StockDecreased outputEvent = objectMapper.readValue(
                (String) received.getPayload(),
                StockDecreased.class
            );

            LOGGER.info("Response received: {}", received.getPayload());

            assertEquals(String.valueOf(outputEvent.getId()), "1");
            assertEquals(String.valueOf(outputEvent.getStock()), "5");
            assertEquals(String.valueOf(outputEvent.getProductName()), "TV");
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            assertTrue(e.getMessage(), false);
        }
    }
}
```
