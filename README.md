You're looking to integrate Kafka for asynchronous communication between your Spring Boot microservices. This is a common and powerful pattern in microservices architectures, enabling loose coupling, scalability, and resilience.
Instead of direct HTTP calls between, say, the Order Service and the Product Service (which would be synchronous and blocking), we'll use Kafka as a message broker.
Scenario for Example:
Let's enhance the Order Service and Product Service interaction:
 * When an order is created in the Order Service, it will publish a message to a Kafka topic (e.g., order-events).
 * The Product Service will consume messages from this order-events topic.
 * Upon receiving an order creation event, the Product Service will deduct the ordered product's quantity (or log it for simplicity, as we don't have a database for products).
Key Kafka Concepts:
 * Producer: An application that publishes messages to Kafka topics.
 * Consumer: An application that subscribes to Kafka topics and processes messages.
 * Topic: A category or feed name to which messages are published.
 * Broker: A Kafka server that stores and manages topics.
Prerequisites:
 * Kafka Installation: You need a running Kafka broker. The easiest way is using Docker:
   docker run -d --name zookeeper -p 2181:2181 confluentinc/cp-zookeeper:7.6.0
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=1 -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 --link zookeeper confluentinc/cp-kafka:7.6.0

   (Give it a minute or two to fully start up.)
 * Existing Microservices Setup: We'll build upon the multi-module Spring Boot project with shared JWT security.
Implementation Steps
I. Update Parent pom.xml
No changes are strictly needed here if you already have the spring-boot.version and spring-cloud.version defined, as Kafka dependencies will be added directly to the service POMs.
II. order-service (Producer)
The Order Service will produce OrderCreatedEvent messages to Kafka.
1. order-service/pom.xml:
Add the spring-kafka dependency.
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-jwt-shared</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <artifactId>order-service</artifactId>
    <name>order-service</name>
    <description>Order Microservice</description>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common-security</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

2. order-service/src/main/resources/application.properties:
Add Kafka producer configuration.
server.port=8083
app.jwt.secret=yourVeryLongAndSecureSecretKeyForJWTAuthServices1234567890

# Kafka Producer Configuration
spring.kafka.producer.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer # For JSON messages
spring.kafka.producer.properties.spring.json.add.type.headers=false # Avoid adding __TypeId__ header

# Topic names
app.kafka.topic.order-events=order-events

3. order-service/src/main/java/com/example/orderservice/event/OrderCreatedEvent.java:
This is the DTO (Data Transfer Object) for the Kafka message.
package com.example.orderservice.event;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderCreatedEvent {
    private String orderId;
    private String customerUsername;
    private String productName;
    private int quantity;
    private double totalAmount;
    private LocalDateTime createdAt;
}

4. order-service/src/main/java/com/example/orderservice/kafka/OrderProducer.java:
This class will send messages to Kafka.
package com.example.orderservice.kafka;

import com.example.orderservice.event.OrderCreatedEvent;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
@RequiredArgsConstructor
public class OrderProducer {

    private static final Logger log = LoggerFactory.getLogger(OrderProducer.class);
    private final KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate;

    @Value("${app.kafka.topic.order-events}")
    private String orderEventsTopic;

    public void sendOrderCreatedEvent(OrderCreatedEvent event) {
        log.info("Sending OrderCreatedEvent to Kafka topic {}: {}", orderEventsTopic, event);
        kafkaTemplate.send(orderEventsTopic, event.getOrderId(), event)
                .whenComplete((result, ex) -> {
                    if (ex == null) {
                        log.info("OrderCreatedEvent sent successfully to topic {} with offset {}: {}",
                                result.getRecordMetadata().topic(),
                                result.getRecordMetadata().offset(),
                                event);
                    } else {
                        log.error("Failed to send OrderCreatedEvent to topic {}: {}", orderEventsTopic, event, ex);
                    }
                });
    }
}

5. order-service/src/main/java/com/example/orderservice/order/OrderController.java:
Modify the createOrder method to publish the Kafka event.
package com.example.orderservice.order;

import com.example.orderservice.event.OrderCreatedEvent; // Import the event DTO
import com.example.orderservice.kafka.OrderProducer; // Import the producer
import lombok.RequiredArgsConstructor; // Add this for @RequiredArgsConstructor
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDateTime; // Import LocalDateTime
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor // Use Lombok's RequiredArgsConstructor
public class OrderController {

    private final List<Order> orders = new ArrayList<>();
    private Long nextOrderId = 1L;
    private final OrderProducer orderProducer; // Inject OrderProducer

    // Constructor modified to initialize products (if still needed)
    public OrderController(OrderProducer orderProducer) {
        this.orderProducer = orderProducer;
        orders.add(new Order(nextOrderId++, "user1", "Laptop", 1, 1200.00));
        orders.add(new Order(nextOrderId++, "admin", "Mouse", 2, 50.00));
        orders.add(new Order(nextOrderId++, "user1", "Keyboard", 1, 75.00));
    }


    // ... (other methods are unchanged) ...

    @PostMapping
    @PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')")
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (!authentication.getAuthorities().contains(new SimpleGrantedAuthority("ROLE_ADMIN")) &&
            !order.getCustomerUsername().equals(authentication.getName())) {
            return ResponseEntity.status(403).build();
        }

        order.setId(nextOrderId++);
        orders.add(order);

        // NEW: Publish OrderCreatedEvent to Kafka
        OrderCreatedEvent event = new OrderCreatedEvent(
            String.valueOf(order.getId()),
            order.getCustomerUsername(),
            order.getProductName(),
            order.getQuantity(),
            order.getTotalAmount(),
            LocalDateTime.now()
        );
        orderProducer.sendOrderCreatedEvent(event);

        return ResponseEntity.status(201).body(order);
    }
}

III. product-service (Consumer)
The Product Service will consume OrderCreatedEvent messages from Kafka.
1. product-service/pom.xml:
Add the spring-kafka dependency.
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-jwt-shared</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <artifactId>product-service</artifactId>
    <name>product-service</name>
    <description>Product Microservice</description>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common-security</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

2. product-service/src/main/resources/application.properties:
Add Kafka consumer configuration.
server.port=8082
app.jwt.secret=yourVeryLongAndSecureSecretKeyForJWTAuthServices1234567890

# Kafka Consumer Configuration
spring.kafka.consumer.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=product-service-group
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer # For JSON messages
spring.kafka.consumer.properties.spring.json.trusted.packages=* # Trust all packages for deserialization
spring.kafka.consumer.properties.spring.json.value.default.type=com.example.productservice.event.OrderCreatedEvent # Default type if header is not present

# Topic names
app.kafka.topic.order-events=order-events

Important: spring.kafka.consumer.properties.spring.json.trusted.packages=* and spring.kafka.consumer.properties.spring.json.value.default.type are crucial for Spring Kafka's JsonDeserializer to correctly deserialize JSON messages into your OrderCreatedEvent DTO without the producer explicitly adding type headers.
3. product-service/src/main/java/com/example/productservice/event/OrderCreatedEvent.java:
The consumer needs the same DTO definition as the producer to deserialize the message correctly. It must have the exact same package and class name relative to the value-deserializer configuration.
package com.example.productservice.event; // Must match the package in order-service

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderCreatedEvent {
    private String orderId;
    private String customerUsername;
    private String productName;
    private int quantity;
    private double totalAmount;
    private LocalDateTime createdAt;
}

4. product-service/src/main/java/com/example/productservice/kafka/OrderConsumer.java:
This class will listen for and process Kafka messages.
package com.example.productservice.kafka;

import com.example.productservice.event.OrderCreatedEvent;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component
public class OrderConsumer {

    private static final Logger log = LoggerFactory.getLogger(OrderConsumer.class);

    @KafkaListener(topics = "${app.kafka.topic.order-events}", groupId = "${spring.kafka.consumer.group-id}")
    public void listenOrderCreated(OrderCreatedEvent event) {
        log.info("Received OrderCreatedEvent from Kafka: {}", event);

        // Here, you would implement your business logic for the Product Service
        // For example:
        // 1. Deduct inventory for event.getProductName() by event.getQuantity()
        // 2. Update product status
        // 3. Log the order for analytical purposes
        // 4. Send an internal notification

        log.info("Product Service: Processing order for product '{}', quantity '{}' by user '{}'",
                event.getProductName(), event.getQuantity(), event.getCustomerUsername());

        // In a real application, if you had a database for products:
        // Product product = productRepository.findByName(event.getProductName());
        // if (product != null && product.getQuantity() >= event.getQuantity()) {
        //    product.setQuantity(product.getQuantity() - event.getQuantity());
        //    productRepository.save(product);
        //    log.info("Inventory updated for product: {}", event.getProductName());
        // } else {
        //    log.warn("Insufficient stock or product not found for order: {}", event.getOrderId());
        //    // Handle out-of-stock scenario (e.g., compensating transaction, notify order service)
        // }
    }
}

III. Update api-gateway (No Changes)
The API Gateway remains unchanged, as it's unaware of the internal Kafka communication. It still routes HTTP requests to the order-service as before.
How to Run and Test
 * Start Kafka and Zookeeper (using the Docker commands provided at the beginning).
 * Build all projects from the parent directory: mvn clean install
 * Start all services in separate terminals:
   * auth-service (Port 8081)
   * product-service (Port 8082)
   * order-service (Port 8083)
   * api-gateway (Port 8080)
   Verify that order-service and product-service logs show successful Kafka connection.
 * Test the Flow:
   * Login as a user (e.g., user1 or admin) to get a JWT token:
     curl -X POST -H "Content-Type: application/json" -d '{"username": "user1", "password": "user1password"}' http://localhost:8080/api/auth/login
# Copy the token

   * Create an Order using the order-service via the API Gateway:
     # Replace <USER_TOKEN> with your actual JWT token
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer <USER_TOKEN>" -d '{"customerUsername": "user1", "productName": "Keyboard", "quantity": 1, "totalAmount": 75.00}' http://localhost:8080/api/orders

   * Observe the Logs:
     * order-service logs: You should see messages indicating that OrderCreatedEvent is being sent to Kafka.
       INFO ... OrderProducer - Sending OrderCreatedEvent to Kafka topic order-events: OrderCreatedEvent(...)
INFO ... OrderProducer - OrderCreatedEvent sent successfully to topic order-events with offset ...

     * product-service logs: You should see messages indicating that OrderCreatedEvent is being received and processed from Kafka.
       INFO ... OrderConsumer - Received OrderCreatedEvent from Kafka: OrderCreatedEvent(...)
INFO ... OrderConsumer - Product Service: Processing order for product 'Keyboard', quantity '1' by user 'user1'

This demonstrates asynchronous, event-driven communication using Kafka between your order-service (as a producer) and your product-service (as a consumer). This pattern makes your services more resilient (if product service is down, order service can still produce, and product service will process when it comes back up) and scalable.
