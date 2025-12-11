# Test Code Generation Rules
When generating test code for projects, it should be created based on the content and requirements below.

## Reference Metadata
When writing test code, it should be written based on the given, when, then data under examples data in the metadata.

## Test Code Components
1. **Must configure imports and code related to JUnit and Spring Boot**
2. **JPA test setup via DataJPATest annotation**
3. **Kafka configuration is strictly prohibited**. Use Mock as replacement if needed
4. **Must generate test code using Given-When-Then pattern**:
   - **Given**: Set up test data and preconditions
   - **When**: Action or method being tested
   - **Then**: Verify expected results
5. **Once test file configuration is complete, immediately execute tests and repeat execution - testing until final tests pass by fixing errors**

## Example

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
## Test Rules
1. **Only when the completion log as shown above is output during test execution, the test is considered passed. Build Fail errors must be fixed.**
2. **When repeating test-fix cycles, never use the '-DskipTests' option, and iterative testing and error fixing must continue until the final test completes successfully.**

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

## Best Practices
1. **Test One Thing**: Each test should verify a single behavior
2. **Descriptive Names**: Use clear test method names (e.g., `whenOrderPlaced_thenStockDecreased`)
3. **Arrange-Act-Assert**: Strictly follow Given-When-Then structure
4. **Independent Tests**: Tests should not depend on each other
5. **Clean State**: Use `@Transactional` for rollback after each test
6. **Mock External Dependencies**: Mock Kafka, external APIs, etc.
7. **Verify Events**: Always verify that expected events were published
8. **Test Edge Cases**: Include tests for error conditions and boundary values

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

## Continuous Testing
1. **Run tests after each code change**
2. **Fix failed tests immediately**
3. **Never skip tests with `-DskipTests`**
4. **Ensure 100% test pass rate before deployment**
5. **Use CI/CD pipeline for automated testing**
