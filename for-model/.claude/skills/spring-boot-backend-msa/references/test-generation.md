# Test Code Generation Rules
## Instructions

Use this skill when:
- Generating test code for backend microservices
- Implementing JUnit tests for domain logic
- Testing event-driven communication between services
- Following TDD methodology with Given-When-Then pattern
- Verifying business logic through integration tests

When generating test code for the project, it must be generated based on the content and requirements below.

---

## Reference Metadata

When writing test code, it must be based on given, when, then data under examples data in Metadata.

---

## Test Code Components

1. **Must configure imports and code related to JUnit and Spring Boot**
2. **JPA test setup through DataJPATest annotation**
3. **Kafka configuration is strictly prohibited**. Must replace with Mock if needed
4. **Test code must be generated using Given-When-Then pattern**:
   - **Given**: Set up test data and preconditions
   - **When**: Action or method being tested
   - **Then**: Verify expected results
5. **Once test file configuration is complete, immediately execute tests and repeat execution - testing until final test passes by fixing errors**

---

## Examples

```
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
        // TODO Auto-generated catch block
        e.printStackTrace();
        assertTrue(e.getMessage(), false);
    }
}
```

---

## Successful Test Completion

The following shows an example of successful test completion:

```
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO]
[INFO] BUILD SUCCESS
[INFO]
```

---

## Test Rules

1. **A test is considered to have passed only when the completion log is output as shown above during test execution. Any Build Fail errors must be fixed.**

2. **When repeating the test-fix cycle, the '-DskipTests' option should absolutely not be used, and iterative testing and error fixing must proceed until the final test is successfully completed.**

---

## Test Structure

### Test Class Annotations
```java
@SpringBootTest
@AutoConfigureTestDatabase
@Transactional
public class ServiceNameTest {
    // Test methods
}
```

### Required Dependencies
- JUnit 5
- Spring Boot Test
- Spring Cloud Stream Test Support
- Mockito for mocking
- ObjectMapper for JSON serialization

### Test Method Pattern

#### 1. Given (Setup)
```java
//given:
Entity entity = new Entity();
entity.setField1(value1);
entity.setField2(value2);
repository.save(entity);
```

#### 2. When (Action)
```java
//when:
DomainEvent event = new DomainEvent();
event.setField(value);

// Simulate event publishing
String eventJson = objectMapper.writeValueAsString(event);
processor.inboundTopic().send(
    MessageBuilder
        .withPayload(eventJson)
        .setHeader("type", event.getEventType())
        .build()
);
```

#### 3. Then (Verification)
```java
//then:
Message<String> received = messageCollector
    .forChannel(processor.outboundTopic())
    .poll();

assertNotNull("Event must be published", received);

ResultEvent result = objectMapper.readValue(
    received.getPayload(),
    ResultEvent.class
);

assertEquals(expectedValue, result.getField());
```

---

## Best Practices

1. **Test One Thing**: Each test should verify a single behavior
2. **Descriptive Names**: Use clear test method names (e.g., `whenOrderPlaced_thenStockDecreased`)
3. **Arrange-Act-Assert**: Follow Given-When-Then structure strictly
4. **Independent Tests**: Tests should not depend on each other
5. **Clean State**: Use `@Transactional` to rollback after each test
6. **Mock External Dependencies**: Mock Kafka, external APIs, etc.
7. **Verify Events**: Always verify that expected events are published
8. **Test Edge Cases**: Include tests for error conditions and boundary values

---

## Common Test Scenarios

### 1. Repository Test
```java
@Test
public void testSaveAndFind() {
    //given:
    Order order = new Order();
    order.setCustomerId("CUST001");
    orderRepository.save(order);

    //when:
    Order found = orderRepository.findById(order.getId()).get();

    //then:
    assertEquals("CUST001", found.getCustomerId());
}
```

### 2. Event Publishing Test
```java
@Test
public void testEventPublished() {
    //given:
    Order order = new Order();
    order.setCustomerId("CUST001");

    //when:
    orderRepository.save(order);

    //then:
    Message<String> event = messageCollector
        .forChannel(processor.outboundTopic())
        .poll();
    assertNotNull(event);
}
```

### 3. Policy Handler Test
```java
@Test
public void testPolicyExecution() {
    //given:
    Delivery delivery = new Delivery();
    deliveryRepository.save(delivery);

    //when:
    OrderPlaced event = new OrderPlaced();
    event.setOrderId(123L);
    // Send event to handler

    //then:
    Delivery updated = deliveryRepository.findById(delivery.getId()).get();
    assertEquals("STARTED", updated.getStatus());
}
```

---

## Continuous Testing

1. **Run tests after each code change**
2. **Fix failing tests immediately**
3. **Do NOT skip tests with `-DskipTests`**
4. **Ensure 100% test pass rate before deployment**
5. **Use CI/CD pipeline for automated testing**
