You're asking for a real-world, industry-level example of Kafka with Spring Boot, complete with a full code implementation and detailed explanations. This is an excellent way to understand Kafka's power beyond simple messaging.
Let's use a common and very relevant industry use case: Real-time Order Processing and Inventory Updates in an E-commerce System.
Industry Use Case Scenario:
Imagine an e-commerce platform where:
 * Customers place orders.
 * Each order needs to be processed (e.g., payment, fulfillment).
 * Simultaneously, the inventory of products needs to be updated immediately to prevent overselling.
 * There might be downstream systems (e.g., analytics, shipping) that also need to be aware of new orders.
Why Kafka is perfect for this:
 * Decoupling: The order placement service, inventory service, payment service, and shipping service can operate independently. If the inventory service goes down temporarily, order placement can still continue, and inventory updates will be processed once it's back online.
 * Scalability: As order volume increases, you can scale consumers (e.g., multiple inventory update consumers) to handle the load.
 * Durability: Orders (events) are persisted in Kafka, so even if a service crashes, no order data is lost.
 * Real-time: Inventory updates happen almost instantly after an order is placed.
 * Event Sourcing/Audit Trail: Kafka provides an immutable log of all order-related events, which can be invaluable for auditing, debugging, and rebuilding system state.
 * Multiple Consumers: Different services (inventory, analytics, shipping) can subscribe to the same order event stream without interfering with each other.
High-Level Architecture:
+----------------+       +-------------------+       +--------------------+
| Customer / Web |------>| Order Service     |       | Kafka Cluster      |
| Application    |       | (Spring Boot App) |------>| (Order Events Topic)|
+----------------+       +-------------------+       +--------------------+
                                      |                       ^
                                      |                       |
                                      | (Publishes New Order  |
                                      | Events to Kafka)      |
                                      V                       |
+-------------------+           +-----------------------+     |
| Payment Gateway   |           | Inventory Service     |<----|
| (Simulated)       |<----------| (Spring Boot App)     |     | (Consumes Order Events)
+-------------------+           +-----------------------+     |
                                      |                       |
                                      V                       |
+-------------------+           +-----------------------+     |
| Fulfillment System|<----------| Shipping Service      |<----|
| (Simulated)       |           | (Spring Boot App)     |     | (Consumes Order Events)
+-------------------+           +-----------------------+     |
                                                                |
                                                                V
                                                        +-------------------+
                                                        | Analytics Service |
                                                        | (Spring Boot App) |
                                                        +-------------------+

For this example, we'll focus on the Order Service (Producer) and the Inventory Service (Consumer).
Project Setup
We'll create two separate Spring Boot projects for clarity, mimicking independent microservices:
 * order-service: Acts as the Kafka Producer.
 * inventory-service: Acts as the Kafka Consumer.
Common Dependencies (for both projects via Spring Initializr):
 * Project: Maven Project
 * Language: Java
 * Spring Boot: 3.3.0 (or latest stable)
 * Java: 17 (or higher)
 * Dependencies:
   * Spring Web (for REST endpoints)
   * Spring for Apache Kafka
   * Lombok (optional, but highly recommended for less boilerplate)
   * Jackson Databind (will be implicitly added, but important for JSON serialization/deserialization)
Step 0: Kafka Infrastructure (Docker Compose)
Make sure your Kafka cluster is running. Use the docker-compose.yml from the previous example:
# docker-compose.yml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  broker:
    image: confluentinc/cp-kafka:7.5.0
    hostname: broker
    container_name: broker
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9093:9093"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1

Run: docker compose up -d
Shared Component: The Order Event Model
Since both services will deal with the same type of "Order Created" event, let's define a common Java class for it. In a real project, this would likely be in a shared library or a schema registry. For this example, you'll copy this class to both projects.
com.example.common.OrderCreatedEvent.java
package com.example.common;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;
import java.util.List;

@Data // Lombok: Generates getters, setters, toString, equals, hashCode
@NoArgsConstructor // Lombok: Generates no-argument constructor
@AllArgsConstructor // Lombok: Generates constructor with all arguments
public class OrderCreatedEvent {
    private String orderId;
    private String customerId;
    private LocalDateTime orderTimestamp;
    private List<OrderItem> items;
    private double totalAmount;

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class OrderItem {
        private String productId;
        private int quantity;
        private double pricePerUnit;
    }
}

Explanation:
 * This POJO (Plain Old Java Object) represents the structure of an order event.
 * @Data, @NoArgsConstructor, @AllArgsConstructor are Lombok annotations that reduce boilerplate by automatically generating constructors, getters, setters, equals(), hashCode(), and toString() methods.
 * OrderItem is an inner class representing items within the order.
 * We'll serialize instances of this class to JSON before sending them to Kafka.
Project 1: order-service (Kafka Producer)
Purpose: Simulates a customer placing an order. It generates an OrderCreatedEvent and publishes it to a Kafka topic.
1. pom.xml (Verify Dependencies):
Make sure you have:
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.kafka</groupId>
			<artifactId>spring-kafka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
		</dependency>

2. src/main/resources/application.properties:
# Server port for the Order Service
server.port=8080

# Kafka Producer Configuration
spring.kafka.producer.bootstrap-servers=localhost:9092
# Key will be String (e.g., orderId)
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
# Value will be JSON representation of OrderCreatedEvent
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

# Custom Kafka topic for order events
app.kafka.topic.order-events=order-events

Explanation:
 * server.port=8080: Default port for the Order Service.
 * value-serializer=org.springframework.kafka.support.serializer.JsonSerializer: This is crucial! Spring Kafka provides a JsonSerializer that automatically converts your Java objects (like OrderCreatedEvent) into JSON strings for Kafka messages.
 * app.kafka.topic.order-events: A custom property to define our Kafka topic name. Good practice to externalize.
3. Copy OrderCreatedEvent.java:
Place the OrderCreatedEvent.java file (and its inner OrderItem) into order-service/src/main/java/com/example/common/.
4. com.example.orderservice.service.OrderService.java (Producer Logic):
package com.example.orderservice.service;

import com.example.common.OrderCreatedEvent;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

import java.util.UUID;
import java.time.LocalDateTime;

@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {

    private final KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate;

    @Value("${app.kafka.topic.order-events}")
    private String orderEventsTopic;

    /**
     * Simulates placing an order and publishes an OrderCreatedEvent to Kafka.
     * @param event The OrderCreatedEvent to publish.
     */
    public void placeOrder(OrderCreatedEvent event) {
        // Assign a unique order ID if not already set
        if (event.getOrderId() == null || event.getOrderId().isEmpty()) {
            event.setOrderId(UUID.randomUUID().toString());
        }
        event.setOrderTimestamp(LocalDateTime.now()); // Set current timestamp

        log.info("Attempting to send order event: {}", event);

        // Send the event to Kafka using the orderId as the key for partitioning
        kafkaTemplate.send(orderEventsTopic, event.getOrderId(), event)
                .whenComplete((result, ex) -> {
                    if (ex == null) {
                        log.info("Successfully published order event for order ID: {} to topic: {}, partition: {}, offset: {}",
                                event.getOrderId(),
                                result.getRecordMetadata().topic(),
                                result.getRecordMetadata().partition(),
                                result.getRecordMetadata().offset());
                    } else {
                        log.error("Failed to publish order event for order ID: {}. Error: {}", event.getOrderId(), ex.getMessage(), ex);
                    }
                });
    }
}

Explanation:
 * KafkaTemplate<String, OrderCreatedEvent>: Now parameterized with String for the key and OrderCreatedEvent for the value, as we're sending complex objects.
 * @Value("${app.kafka.topic.order-events}"): Injects the topic name from application.properties.
 * placeOrder(OrderCreatedEvent event): This method simulates placing an order. It assigns a UUID as orderId and sets the timestamp.
 * kafkaTemplate.send(orderEventsTopic, event.getOrderId(), event): This is the core line.
   * orderEventsTopic: The Kafka topic name.
   * event.getOrderId(): We use the orderId as the message key. This is important! Kafka guarantees that all messages with the same key will go to the same partition. This is vital for maintaining order-specific event sequence (e.g., all events for Order-123 are processed in order).
   * event: The OrderCreatedEvent object, which JsonSerializer will convert to JSON.
 * .whenComplete(): Handles the asynchronous result of sending.
5. com.example.orderservice.controller.OrderController.java (REST Endpoint):
package com.example.orderservice.controller;

import com.example.common.OrderCreatedEvent;
import com.example.orderservice.service.OrderService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
@Slf4j
public class OrderController {

    private final OrderService orderService;

    /**
     * Endpoint to simulate placing a new order.
     * Receives an OrderCreatedEvent (or a partial order) in the request body.
     */
    @PostMapping
    public ResponseEntity<String> createOrder(@RequestBody OrderCreatedEvent orderRequest) {
        log.info("Received request to create order: {}", orderRequest);
        orderService.placeOrder(orderRequest);
        return new ResponseEntity<>("Order processing initiated for order ID: " + orderRequest.getOrderId(), HttpStatus.ACCEPTED);
    }
}

Explanation:
 * @PostMapping: Maps HTTP POST requests to /api/orders.
 * @RequestBody OrderCreatedEvent orderRequest: Spring automatically deserializes the incoming JSON request body into an OrderCreatedEvent object thanks to Jackson and spring-boot-starter-web.
 * Calls the orderService to publish the event.
Project 2: inventory-service (Kafka Consumer)
Purpose: Listens for OrderCreatedEvent messages from Kafka and simulates updating inventory.
1. pom.xml (Verify Dependencies):
Same as order-service's pom.xml in terms of core dependencies.
2. src/main/resources/application.properties:
# Server port for the Inventory Service (different from Order Service)
server.port=8081

# Kafka Consumer Configuration
spring.kafka.consumer.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=inventory-update-group # Unique consumer group ID
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
# Value will be JSON representation of OrderCreatedEvent, so use JsonDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.auto-offset-reset=earliest

# Specify the type that JsonDeserializer should deserialize to
spring.kafka.consumer.properties.spring.json.value.default.type=com.example.common.OrderCreatedEvent

# Kafka Listener Properties (for @KafkaListener)
spring.kafka.listener.ack-mode=container

# Custom Kafka topic for order events (must match producer)
app.kafka.topic.order-events=order-events

Explanation:
 * server.port=8081: Different port to avoid conflicts with order-service.
 * group-id=inventory-update-group: Each service acting as a consumer should have its own unique group-id if it wants to process all messages independently from other services.
 * value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer: This deserializes the incoming JSON Kafka message back into a Java object.
 * spring.kafka.consumer.properties.spring.json.value.default.type=com.example.common.OrderCreatedEvent: This is critical for JsonDeserializer. It tells Spring Kafka which concrete Java class it should try to deserialize the JSON message into.
 * app.kafka.topic.order-events: Same topic name as the producer.
3. Copy OrderCreatedEvent.java:
Place the OrderCreatedEvent.java file (and its inner OrderItem) into inventory-service/src/main/java/com/example/common/.
4. com.example.inventoryservice.listener.OrderEventListener.java (Consumer Logic):
package com.example.inventoryservice.listener;

import com.example.common.OrderCreatedEvent;
import lombok.extern.slf4j.Slf4j;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

@Component
@Slf4j
public class OrderEventListener {

    private static final String TOPIC_NAME = "${app.kafka.topic.order-events}";
    private static final String GROUP_ID = "inventory-update-group";

    // Simulate an in-memory inventory for demonstration purposes
    // In a real application, this would interact with a database
    private final ConcurrentHashMap<String, AtomicInteger> productInventory = new ConcurrentHashMap<>();

    // Initialize some dummy inventory
    public OrderEventListener() {
        productInventory.put("PROD-001", new AtomicInteger(100));
        productInventory.put("PROD-002", new AtomicInteger(50));
        productInventory.put("PROD-003", new AtomicInteger(200));
        log.info("Initialized inventory: {}", productInventory);
    }


    /**
     * Listens for OrderCreatedEvent messages from Kafka and updates inventory.
     * @param event The received OrderCreatedEvent object.
     */
    @KafkaListener(topics = TOPIC_NAME, groupId = GROUP_ID)
    public void handleOrderCreatedEvent(OrderCreatedEvent event) {
        log.info("Received OrderCreatedEvent for Order ID: {} from customer: {}",
                 event.getOrderId(), event.getCustomerId());

        boolean inventoryUpdatedSuccessfully = true;

        for (OrderCreatedEvent.OrderItem item : event.getItems()) {
            String productId = item.getProductId();
            int quantityOrdered = item.getQuantity();

            productInventory.compute(productId, (key, currentCount) -> {
                if (currentCount == null) {
                    log.warn("Product '{}' not found in inventory. Cannot update.", productId);
                    inventoryUpdatedSuccessfully = false; // Mark as failed
                    return null; // Don't add if not found
                }
                if (currentCount.get() < quantityOrdered) {
                    log.warn("Insufficient stock for product '{}'. Available: {}, Ordered: {}",
                             productId, currentCount.get(), quantityOrdered);
                    inventoryUpdatedSuccessfully = false; // Mark as failed
                    return currentCount; // Keep current count
                }
                int newCount = currentCount.addAndGet(-quantityOrdered);
                log.info("Updated inventory for product '{}': {} -> {}", productId, currentCount.get() + quantityOrdered, newCount);
                return new AtomicInteger(newCount); // Return new AtomicInteger
            });

            if (!inventoryUpdatedSuccessfully) {
                // In a real system, you'd log this, potentially send a compensation event,
                // or put the order in a 'pending review' state.
                log.error("Inventory update failed for order {}. Aborting further processing for this order's items.", event.getOrderId());
                break; // Stop processing other items for this order
            }
        }

        if (inventoryUpdatedSuccessfully) {
            log.info("Successfully processed order {} for inventory update. Current inventory: {}", event.getOrderId(), productInventory);
            // In a real system, after successful inventory update, you might:
            // - Publish an "InventoryReservedEvent" or "OrderProcessedEvent"
            // - Trigger the shipping process
        } else {
            log.error("Failed to fully process order {} for inventory update due to stock issues. Further action needed.", event.getOrderId());
        }
    }

    // Optional: An endpoint to view current inventory (for debugging/testing)
    // You would typically not expose this in a production microservice directly.
    // However, for demo purposes, it's useful.
    // In a real app, you might have a dedicated inventory lookup service.
    @KafkaListener(id = "inventoryStatus", topics = "inventory-status-request", containerFactory = "kafkaListenerContainerFactory", autoStartup = "false")
    public void listenForInventoryStatusRequests(String productId, @org.springframework.messaging.handler.annotation.Header(KafkaHeaders.REPLY_TOPIC) String replyTopic) {
        log.info("Received inventory status request for product: {} to reply to topic: {}", productId, replyTopic);
        Integer stock = productInventory.getOrDefault(productId, new AtomicInteger(0)).get();
        // In a real app, you might use KafkaTemplate to send a reply
        // kafkaTemplate.send(replyTopic, "Current stock for " + productId + ": " + stock);
        log.info("Simulating sending reply: Current stock for {}: {}", productId, stock);
    }
}

Explanation:
 * @KafkaListener(topics = TOPIC_NAME, groupId = GROUP_ID): This method will be invoked whenever an OrderCreatedEvent is published to the order-events topic by any producer, as long as it's part of the inventory-update-group.
 * handleOrderCreatedEvent(OrderCreatedEvent event): Spring Kafka automatically deserializes the incoming JSON Kafka message into an OrderCreatedEvent object thanks to the JsonDeserializer and the default.type property.
 * Simulated Inventory: productInventory is a simple ConcurrentHashMap acting as a mock inventory. In a real system, this would interact with a database (e.g., calling an InventoryRepository to update actual product counts).
 * Inventory Logic: The code iterates through order items, checks stock, and updates the simulated inventory. It also includes basic error logging for insufficient stock.
 * @KafkaListener(id = "inventoryStatus", ...): This is an example of a request-reply pattern or another type of listener. It's commented out because it requires more setup (like a dedicated reply topic and KafkaTemplate injection in the consumer), but I've included it to show that a single consumer can listen to multiple topics or have different KafkaListener methods. The autoStartup = "false" means it won't start automatically without explicit configuration.
How to Run and Test
 * Start Kafka:
   Open your terminal in the directory containing docker-compose.yml and run:
   docker compose up -d

   Verify containers are running: docker compose ps
 * Start order-service:
   Navigate to the order-service project root.
   ./mvnw spring-boot:run

   (Or run from your IDE's main class OrderServiceApplication)
 * Start inventory-service:
   Open a new terminal, navigate to the inventory-service project root.
   ./mvnw spring-boot:run

   (Or run from your IDE's main class InventoryServiceApplication)
   You should see log messages from both applications. The inventory-service will log its initial inventory.
 * Test Order Placement:
   Use curl or Postman to send a POST request to the order-service.
   Request:
   * Method: POST
   * URL: http://localhost:8080/api/orders
   * Headers:
     * Content-Type: application/json
   * Body (JSON):
   {
  "customerId": "CUST-001",
  "items": [
    {
      "productId": "PROD-001",
      "quantity": 5,
      "pricePerUnit": 10.50
    },
    {
      "productId": "PROD-002",
      "quantity": 2,
      "pricePerUnit": 25.00
    }
  ],
  "totalAmount": 102.50
}

   Example curl command:
   curl -X POST \
  http://localhost:8080/api/orders \
  -H 'Content-Type: application/json' \
  -d '{
    "customerId": "CUST-001",
    "items": [
      {
        "productId": "PROD-001",
        "quantity": 5,
        "pricePerUnit": 10.50
      },
      {
        "productId": "PROD-002",
        "quantity": 2,
        "pricePerUnit": 25.00
      }
    ],
    "totalAmount": 102.50
  }'

 * Observe Logs:
   * order-service console: You'll see logs indicating the order event was received and successfully published to Kafka, along with its partition and offset.
   * inventory-service console: Almost immediately, you'll see logs indicating that the OrderCreatedEvent was received and the inventory was updated for PROD-001 and PROD-002.
   Try a second order for the same customer:
   {
  "customerId": "CUST-001",
  "items": [
    {
      "productId": "PROD-001",
      "quantity": 10,
      "pricePerUnit": 10.50
    }
  ],
  "totalAmount": 105.00
}

   You'll see PROD-001 stock further reduced.
   Try to oversell (e.g., PROD-002 quantity 100):
   {
  "customerId": "CUST-002",
  "items": [
    {
      "productId": "PROD-002",
      "quantity": 100,
      "pricePerUnit": 25.00
    }
  ],
  "totalAmount": 2500.00
}

   The inventory-service will log a WARN about insufficient stock and an ERROR about failure to process.
Key Takeaways from this Example:
 * Asynchronous Communication: The Order Service doesn't wait for the Inventory Service to finish updating before responding to the customer. It just publishes an event and moves on. This improves responsiveness and resilience.
 * Decoupling: order-service and inventory-service are completely independent microservices. They only share the OrderCreatedEvent contract and the Kafka topic. You could deploy and scale them separately.
 * Reliability: Even if inventory-service was down when an order was placed, Kafka would store the OrderCreatedEvent. When inventory-service comes back online, it would pick up the unprocessed event and update the inventory.
 * Scalability (Consumer Groups): If order volume grew significantly, you could run multiple instances of inventory-service (all with group-id=inventory-update-group). Kafka would automatically distribute partitions among these instances, allowing them to process orders in parallel.
 * JSON Serialization/Deserialization: Spring Kafka, with Jackson, simplifies sending and receiving complex Java objects as JSON messages over Kafka.
 * Message Keys for Ordering/Partitioning: Using orderId as the key ensures that all events related to a specific order go to the same partition, which can be crucial for maintaining processing order for that specific entity.
This example demonstrates a fundamental pattern for building event-driven microservices architectures using Spring Boot and Kafka.
