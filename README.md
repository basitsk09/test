Alright, let's dive into another real-world, more complex example that showcases additional Kafka features and common patterns in a Spring Boot application.
Industry Use Case: User Activity Tracking and Notification System
Imagine a large online platform (e.g., a social media site, a content platform, or an e-learning system) where user actions generate a vast stream of events. We want to process these events for various purposes, including:
 * Real-time Analytics: Understanding user behavior.
 * Personalized Notifications: Sending relevant notifications to users based on their activity or the activity of others they interact with.
Why Kafka is Perfect for this:
 * High Volume/Velocity: User activity can generate millions of events per second. Kafka is built to handle this scale.
 * Decoupling & Fan-out: A single user activity event can trigger multiple downstream processes (e.g., analytics, notification generation, recommendation engine updates). Kafka allows multiple independent consumers to subscribe to the same topic.
 * Event Sourcing & Auditability: Every user action is an immutable event, providing a comprehensive audit trail.
 * Real-time Processing: Enables immediate reactions to user activity (e.g., sending a notification right after a like).
 * Resilience: If the notification service goes down, activity events are buffered in Kafka and processed once it recovers.
Scenario Breakdown:
We'll build two Spring Boot microservices:
 * activity-tracker-service (Producer):
   * Simulates a user performing an action (e.g., "LIKE_POST", "COMMENT_POST", "FOLLOW_USER").
   * Publishes these UserActivityEvent objects to a Kafka topic.
 * notification-service (Consumer & Producer - demonstrating request-reply pattern):
   * Consumer: Listens for UserActivityEvents.
   * Logic: When a "LIKE_POST" event occurs, it might determine if the post owner should be notified. When a "COMMENT_POST" event occurs, it might notify the post owner and other commenters.
   * Producer (Internal): Once a notification is generated, it might publish a NotificationEvent to a separate Kafka topic for delivery (e.g., by a dedicated email/push notification sender service). This adds another layer of decoupling.
   * Advanced Feature (Request-Reply): We'll also demonstrate a basic request-reply pattern where a specific request (e.g., "get last 10 activities for a user") can be sent to Kafka, and a dedicated listener in activity-tracker-service can respond to a reply topic. (This is slightly more complex, and might not be used for notifications themselves but for querying event streams).
High-Level Architecture for this Example:
+--------------------+       +---------------------------+       +-------------------------+
| User Interface     |------>| Activity Tracker Service  |------>| Kafka Cluster           |
| (Simulated)        |       | (Spring Boot App)         |       | (user-activities Topic) |
+--------------------+       +---------------------------+       +-------------------------+
                                      |                                ^           ^
                                      |                                |           |
                                      | (Publishes User Activity Events)           |
                                      V                                |           |
                                                                       |           |
                                                                       |           |
+------------------------------------------------------------------------------------+
|                                  KAFKA CLUSTER                                     |
+------------------------------------------------------------------------------------+
|                                                                                    |
|  Topic: `user-activities` (for raw user activity events)                          |
|  Topic: `notifications` (for generated notification events)                       |
|  Topic: `activity-query-requests` (for querying activity stream)                  |
|  Topic: `activity-query-replies` (for responses to activity queries)              |
|                                                                                    |
+------------------------------------------------------------------------------------+
                                      ^           |
                                      |           |
+---------------------------+         |           |
| Notification Service      |<--------|           |
| (Spring Boot App)         |         |           |
|                           | (Consumes user-activities) |
| (Generates notifications) |         |           |
|                           |-------->| (Publishes notifications)
|                           |         |
| (Handles query requests)  |<--------| (Consumes activity-query-requests)
|                           |-------->| (Publishes replies to activity-query-replies)
+---------------------------+         |
                                      |
                                      V
+---------------------------+
| Notification Sender       |
| (Email/Push Service)      |
| (Simulated)               |<--------| (Consumes notifications)
+---------------------------+

Step 0: Kafka Infrastructure (Docker Compose)
Use the same docker-compose.yml from the previous example to spin up your Kafka cluster.
docker compose up -d

Shared Components: Event Models
We'll define two event models, both of which will be used by multiple services.
1. com.example.common.UserActivityEvent.java
package com.example.common;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;
import java.util.Map;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserActivityEvent {
    private String eventId; // Unique ID for this specific event
    private String userId;  // User who performed the action
    private String activityType; // e.g., "LIKE_POST", "COMMENT_POST", "FOLLOW_USER"
    private LocalDateTime timestamp;
    private String targetId; // ID of the post, user, product, etc. that was acted upon
    private String relatedUserId; // User related to the target (e.g., post owner, followed user)
    private Map<String, String> details; // Any additional key-value details
}

2. com.example.common.NotificationEvent.java
package com.example.common;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;
import java.util.Map;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class NotificationEvent {
    private String notificationId;
    private String recipientUserId;
    private String notificationType; // e.g., "POST_LIKED", "NEW_COMMENT", "NEW_FOLLOWER"
    private String message;
    private LocalDateTime timestamp;
    private Map<String, String> data; // Additional data for the notification
}

Explanation of Models:
 * UserActivityEvent: Captures granular user actions. Includes userId (actor), activityType, targetId (what was acted on), and relatedUserId (who owns the target).
 * NotificationEvent: Represents a generated notification ready to be delivered. Includes recipientUserId, notificationType, and a message.
Project 1: activity-tracker-service (Kafka Producer & Query Handler)
Purpose: Simulates user actions and publishes them. Also demonstrates a Kafka-based request-reply pattern for querying historical data.
1. pom.xml (Dependencies):
Ensure you have spring-boot-starter-web, spring-kafka, and lombok. Jackson will be implicitly added.
2. src/main/resources/application.properties:
# Server port for the Activity Tracker Service
server.port=8080

# Kafka Producer Configuration
spring.kafka.producer.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

# Kafka Consumer Configuration (for request-reply pattern)
spring.kafka.consumer.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=activity-query-group # Specific group for query handlers
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.auto-offset-reset=latest # For queries, typically start from latest messages

# Custom Kafka topic configurations
app.kafka.topic.user-activities=user-activities
app.kafka.topic.activity-query-requests=activity-query-requests
app.kafka.topic.activity-query-replies=activity-query-replies # Topic to send replies to

Explanation:
 * We're configuring both producer (for UserActivityEvent) and consumer (for activity-query-requests) in this service.
 * value-serializer=org.springframework.kafka.support.serializer.JsonSerializer: For serializing UserActivityEvent objects.
 * app.kafka.topic.*: Define our topic names.
3. Copy Event Models:
Place UserActivityEvent.java and NotificationEvent.java into activity-tracker-service/src/main/java/com/example/common/.
4. com.example.activitytracker.service.UserActivityService.java (Producer & Data Store):
package com.example.activitytracker.service;

import com.example.common.UserActivityEvent;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
@Slf4j
public class UserActivityService {

    private final KafkaTemplate<String, UserActivityEvent> kafkaTemplate;

    @Value("${app.kafka.topic.user-activities}")
    private String userActivitiesTopic;

    // Simulate an in-memory storage of recent activities for query demonstration
    // In a real system, this would be a persistent database (e.g., Cassandra, MongoDB)
    private final ConcurrentHashMap<String, List<UserActivityEvent>> userActivitiesStore = new ConcurrentHashMap<>();
    private final int MAX_ACTIVITIES_PER_USER = 100; // Limit for in-memory store

    /**
     * Records a user activity event and publishes it to Kafka.
     * @param event The UserActivityEvent to record.
     */
    public void recordActivity(UserActivityEvent event) {
        if (event.getEventId() == null || event.getEventId().isEmpty()) {
            event.setEventId(UUID.randomUUID().toString());
        }
        event.setTimestamp(LocalDateTime.now());

        log.info("Recording and sending user activity: {}", event);

        // Store activity in memory for quick query demonstration
        userActivitiesStore.compute(event.getUserId(), (userId, activities) -> {
            if (activities == null) {
                activities = Collections.synchronizedList(new ArrayList<>());
            }
            activities.add(0, event); // Add to the beginning to keep latest first
            if (activities.size() > MAX_ACTIVITIES_PER_USER) {
                return activities.subList(0, MAX_ACTIVITIES_PER_USER); // Trim old activities
            }
            return activities;
        });

        // Publish to Kafka using userId as the key for partitioning
        kafkaTemplate.send(userActivitiesTopic, event.getUserId(), event)
                .whenComplete((result, ex) -> {
                    if (ex == null) {
                        log.info("Activity published for user: {} to topic: {}, partition: {}, offset: {}",
                                event.getUserId(),
                                result.getRecordMetadata().topic(),
                                result.getRecordMetadata().partition(),
                                result.getRecordMetadata().offset());
                    } else {
                        log.error("Failed to publish activity for user: {}. Error: {}", event.getUserId(), ex.getMessage(), ex);
                    }
                });
    }

    /**
     * Retrieves recent activities for a given user from the in-memory store.
     * In a real scenario, this would query a database.
     * @param userId The ID of the user.
     * @param limit The maximum number of activities to return.
     * @return A list of recent UserActivityEvent objects.
     */
    public List<UserActivityEvent> getRecentActivities(String userId, int limit) {
        return userActivitiesStore.getOrDefault(userId, Collections.emptyList())
                .stream()
                .limit(limit)
                .collect(Collectors.toList());
    }
}

Explanation:
 * KafkaTemplate<String, UserActivityEvent>: Producer is set up to send UserActivityEvent objects.
 * recordActivity():
   * Generates eventId and timestamp.
   * In-Memory Store (userActivitiesStore): A ConcurrentHashMap is used to simulate a database storing recent user activities. This allows the getRecentActivities method (used for the request-reply) to quickly serve data. In a real application, this would be a persistent database.
   * kafkaTemplate.send(userActivitiesTopic, event.getUserId(), event): Sends the event. Using userId as the key ensures all activities for the same user go to the same partition, which can be useful for stateful stream processing later.
5. com.example.activitytracker.listener.ActivityQueryListener.java (Consumer for Request-Reply):
package com.example.activitytracker.listener;

import com.example.activitytracker.service.UserActivityService;
import com.example.common.UserActivityEvent;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
@RequiredArgsConstructor
@Slf4j
public class ActivityQueryListener {

    private final UserActivityService userActivityService;
    private final KafkaTemplate<String, String> replyKafkaTemplate; // For sending plain JSON replies
    private final ObjectMapper objectMapper; // For JSON serialization

    @KafkaListener(topics = "${app.kafka.topic.activity-query-requests}",
                   groupId = "${spring.kafka.consumer.group-id}")
    public void listenForActivityQueries(
            String userId, // The message value (userId to query)
            @Header(KafkaHeaders.REPLY_TOPIC) String replyTopic, // Header tells us where to reply
            @Header(KafkaHeaders.CORRELATION_ID) byte[] correlationId // Optional: for matching requests/replies
    ) {
        log.info("Received activity query for user: {} to reply to topic: {}", userId, replyTopic);

        List<UserActivityEvent> recentActivities = userActivityService.getRecentActivities(userId, 10); // Get top 10

        try {
            String replyMessage = objectMapper.writeValueAsString(recentActivities);
            log.info("Sending reply for user {}: {}", userId, replyMessage);

            // Send reply to the specified replyTopic
            replyKafkaTemplate.send(replyTopic, new String(correlationId), replyMessage)
                    .whenComplete((result, ex) -> {
                        if (ex == null) {
                            log.info("Reply sent successfully to topic: {}, partition: {}, offset: {}",
                                     result.getRecordMetadata().topic(),
                                     result.getRecordMetadata().partition(),
                                     result.getRecordMetadata().offset());
                        } else {
                            log.error("Failed to send reply for user: {}. Error: {}", userId, ex.getMessage(), ex);
                        }
                    });
        } catch (JsonProcessingException e) {
            log.error("Error serializing activities for user {}: {}", userId, e.getMessage(), e);
        }
    }
}

Explanation (Request-Reply):
 * @KafkaListener(topics = "${app.kafka.topic.activity-query-requests}", ...): This listener specifically picks up messages from the activity-query-requests topic.
 * @Header(KafkaHeaders.REPLY_TOPIC) String replyTopic: This crucial header, automatically populated by Spring Kafka when a request-reply is initiated, tells our listener where to send the response.
 * @Header(KafkaHeaders.CORRELATION_ID) byte[] correlationId: Another standard header for matching a reply to its original request.
 * userActivityService.getRecentActivities(): Calls the service to fetch the simulated data.
 * replyKafkaTemplate.send(replyTopic, new String(correlationId), replyMessage): Sends the JSON string of activities back to the replyTopic. The correlationId is also sent back to the requester can match it.
 * NOTE: For a full request-reply in Spring Kafka, the requester also needs specific setup (e.g., ReplyingKafkaTemplate). We'll just show the responder part here.
6. com.example.activitytracker.controller.UserActivityController.java (REST Endpoint):
package com.example.activitytracker.controller;

import com.example.activitytracker.service.UserActivityService;
import com.example.common.UserActivityEvent;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@RequestMapping("/api/activities")
@RequiredArgsConstructor
@Slf4j
public class UserActivityController {

    private final UserActivityService userActivityService;

    /**
     * Endpoint to simulate a user activity.
     * Example:
     * POST /api/activities
     * {
     * "userId": "user-A",
     * "activityType": "LIKE_POST",
     * "targetId": "post-123",
     * "relatedUserId": "user-B",
     * "details": {"source": "mobile_app"}
     * }
     */
    @PostMapping
    public ResponseEntity<String> recordUserActivity(@RequestBody UserActivityEvent activityRequest) {
        log.info("Received request to record user activity: {}", activityRequest);
        userActivityService.recordActivity(activityRequest);
        return new ResponseEntity<>("User activity recorded: " + activityRequest.getActivityType(), HttpStatus.ACCEPTED);
    }
}

Explanation:
 * A simple POST endpoint to receive UserActivityEvents from a simulated UI or backend system.
 * It calls userActivityService.recordActivity() to process and publish the event.
Project 2: notification-service (Kafka Consumer & Notification Producer)
Purpose: Consumes UserActivityEvents, applies business logic to decide if a notification is needed, and publishes NotificationEvents.
1. pom.xml (Dependencies):
Same as activity-tracker-service's pom.xml.
2. src/main/resources/application.properties:
# Server port for the Notification Service
server.port=8081

# Kafka Consumer Configuration (for user-activities topic)
spring.kafka.consumer.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=notification-processor-group # Unique consumer group ID for notifications
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.auto-offset-reset=earliest # Start from beginning to catch all activities

# Specify the type that JsonDeserializer should deserialize to for user-activities
spring.kafka.consumer.properties.spring.json.value.default.type=com.example.common.UserActivityEvent

# Kafka Producer Configuration (for notifications topic)
spring.kafka.producer.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer

# Custom Kafka topic configurations
app.kafka.topic.user-activities=user-activities
app.kafka.topic.notifications=notifications

Explanation:
 * server.port=8081: Different port.
 * group-id=notification-processor-group: This consumer group is distinct from the activity-tracker-service's query group, meaning it will independently process all user-activities events.
 * spring.kafka.consumer.properties.spring.json.value.default.type=com.example.common.UserActivityEvent: Essential for deserializing UserActivityEvent.
 * Producer configuration (spring.kafka.producer.*) for publishing NotificationEvents.
 * app.kafka.topic.notifications: New topic for generated notifications.
3. Copy Event Models:
Place UserActivityEvent.java and NotificationEvent.java into notification-service/src/main/java/com/example/common/.
4. com.example.notificationservice.service.NotificationGenerator.java (Consumer Logic & Producer of Notifications):
package com.example.notificationservice.service;

import com.example.common.NotificationEvent;
import com.example.common.UserActivityEvent;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

@Service
@RequiredArgsConstructor
@Slf4j
public class NotificationGenerator {

    private final KafkaTemplate<String, NotificationEvent> kafkaTemplate;

    @Value("${app.kafka.topic.user-activities}")
    private String userActivitiesTopic; // Listen to this topic

    @Value("${app.kafka.topic.notifications}")
    private String notificationsTopic; // Publish to this topic

    private static final String NOTIFICATION_GROUP_ID = "${spring.kafka.consumer.group-id}";


    /**
     * Listens for UserActivityEvent messages and generates notifications.
     * @param event The received UserActivityEvent object.
     */
    @KafkaListener(topics = "${app.kafka.topic.user-activities}", groupId = NOTIFICATION_GROUP_ID)
    public void handleUserActivityEvent(UserActivityEvent event) {
        log.info("Notification Service: Received UserActivityEvent: {}", event);

        NotificationEvent notification = null;

        // Business logic to decide when to generate a notification
        switch (event.getActivityType()) {
            case "LIKE_POST":
                // Notify the owner of the post
                if (event.getRelatedUserId() != null && !event.getRelatedUserId().equals(event.getUserId())) { // Don't notify self
                    notification = createNotification(
                            event.getRelatedUserId(), // Recipient: post owner
                            "POST_LIKED",
                            String.format("%s liked your post '%s'", event.getUserId(), event.getTargetId()),
                            Map.of("postId", event.getTargetId(), "likerId", event.getUserId())
                    );
                }
                break;
            case "COMMENT_POST":
                // Notify the owner of the post and potentially other commenters (simplified for example)
                if (event.getRelatedUserId() != null && !event.getRelatedUserId().equals(event.getUserId())) {
                    notification = createNotification(
                            event.getRelatedUserId(), // Recipient: post owner
                            "NEW_COMMENT",
                            String.format("%s commented on your post '%s': '%s'", event.getUserId(), event.getTargetId(), event.getDetails().getOrDefault("commentText", "...")),
                            Map.of("postId", event.getTargetId(), "commenterId", event.getUserId())
                    );
                }
                // In a real system, you'd also fetch and notify other relevant users
                break;
            case "FOLLOW_USER":
                // Notify the followed user
                if (event.getTargetId() != null && !event.getTargetId().equals(event.getUserId())) {
                    notification = createNotification(
                            event.getTargetId(), // Recipient: followed user
                            "NEW_FOLLOWER",
                            String.format("%s is now following you!", event.getUserId()),
                            Map.of("followerId", event.getUserId())
                    );
                }
                break;
            // Add more cases for other activity types
            default:
                log.info("No notification rule for activity type: {}", event.getActivityType());
                break;
        }

        if (notification != null) {
            publishNotification(notification);
        }
    }

    private NotificationEvent createNotification(String recipientUserId, String type, String message, Map<String, String> data) {
        return new NotificationEvent(
                UUID.randomUUID().toString(),
                recipientUserId,
                type,
                message,
                LocalDateTime.now(),
                new HashMap<>(data)
        );
    }

    /**
     * Publishes a generated NotificationEvent to the notifications topic.
     * @param notification The NotificationEvent to publish.
     */
    private void publishNotification(NotificationEvent notification) {
        log.info("Notification Service: Publishing notification: {}", notification);
        kafkaTemplate.send(notificationsTopic, notification.getRecipientUserId(), notification)
                .whenComplete((result, ex) -> {
                    if (ex == null) {
                        log.info("Notification published for recipient: {} to topic: {}, partition: {}, offset: {}",
                                notification.getRecipientUserId(),
                                result.getRecordMetadata().topic(),
                                result.getRecordMetadata().partition(),
                                result.getRecordMetadata().offset());
                    } else {
                        log.error("Failed to publish notification for recipient: {}. Error: {}", notification.getRecipientUserId(), ex.getMessage(), ex);
                    }
                });
    }
}

Explanation:
 * @KafkaListener(topics = "${app.kafka.topic.user-activities}", ...): Listens for user activity events.
 * handleUserActivityEvent(UserActivityEvent event): Takes UserActivityEvent as input.
 * Business Logic: The switch statement contains simplified logic for generating notifications based on activityType. This is where complex business rules would reside.
 * kafkaTemplate.send(notificationsTopic, notification.getRecipientUserId(), notification): If a notification is generated, it's then published to the notificationsTopic. The recipientUserId is used as the key to ensure notifications for the same user go to the same partition.
How to Run and Test
 * Start Kafka:
   In your terminal, navigate to the directory with docker-compose.yml and run:
   docker compose up -d

 * Start activity-tracker-service:
   Open your IDE, navigate to the activity-tracker-service project, and run its main method (e.g., ActivityTrackerServiceApplication).
   Alternatively, from the project root:
   ./mvnw spring-boot:run

 * Start notification-service:
   Open a new terminal, navigate to the notification-service project, and run its main method (e.g., NotificationServiceApplication).
   Alternatively, from the project root:
   ./mvnw spring-boot:run

   You should see log messages from both applications indicating they are starting up and Kafka listeners are initialized.
 * Test User Activities (using activity-tracker-service):
   Scenario 1: User A likes Post 123 (owned by User B)
   * Method: POST
   * URL: http://localhost:8080/api/activities
   * Headers: Content-Type: application/json
   * Body:
     {
  "userId": "user-A",
  "activityType": "LIKE_POST",
  "targetId": "post-123",
  "relatedUserId": "user-B",
  "details": {"source": "mobile_app"}
}

   * curl command:
     curl -X POST \
  http://localhost:8080/api/activities \
  -H 'Content-Type: application/json' \
  -d '{
    "userId": "user-A",
    "activityType": "LIKE_POST",
    "targetId": "post-123",
    "relatedUserId": "user-B",
    "details": {"source": "mobile_app"}
  }'

   Observe Logs:
   * activity-tracker-service console: Logs about recording and publishing the LIKE_POST event.
   * notification-service console: Logs about receiving the LIKE_POST event, generating a POST_LIKED notification for user-B, and publishing it to the notifications topic.
   Scenario 2: User C comments on Post 456 (owned by User D)
   * Method: POST
   * URL: http://localhost:8080/api/activities
   * Headers: Content-Type: application/json
   * Body:
     {
  "userId": "user-C",
  "activityType": "COMMENT_POST",
  "targetId": "post-456",
  "relatedUserId": "user-D",
  "details": {"commentText": "Great post!", "source": "web_browser"}
}

   * curl command:
     curl -X POST \
  http://localhost:8080/api/activities \
  -H 'Content-Type: application/json' \
  -d '{
    "userId": "user-C",
    "activityType": "COMMENT_POST",
    "targetId": "post-456",
    "relatedUserId": "user-D",
    "details": {"commentText": "Great post!", "source": "web_browser"}
  }'

   Observe Logs:
   * activity-tracker-service console: Logs about recording and publishing the COMMENT_POST event.
   * notification-service console: Logs about receiving the COMMENT_POST event, generating a NEW_COMMENT notification for user-D, and publishing it.
   Scenario 3: User E follows User F
   * Method: POST
   * URL: http://localhost:8080/api/activities
   * Headers: Content-Type: application/json
   * Body:
     {
  "userId": "user-E",
  "activityType": "FOLLOW_USER",
  "targetId": "user-F",
  "relatedUserId": "user-F"
}

   * curl command:
     curl -X POST \
  http://localhost:8080/api/activities \
  -H 'Content-Type: application/json' \
  -d '{
    "userId": "user-E",
    "activityType": "FOLLOW_USER",
    "targetId": "user-F",
    "relatedUserId": "user-F"
  }'

   Observe Logs:
   * activity-tracker-service console: Logs about recording and publishing the FOLLOW_USER event.
   * notification-service console: Logs about receiving the FOLLOW_USER event, generating a NEW_FOLLOWER notification for user-F, and publishing it.
Advanced Testing (Manual Request-Reply):
To fully test the request-reply, you'd typically need another Spring Boot service that sends the request and waits for the reply using ReplyingKafkaTemplate. Since we don't have a third service, we can simulate sending a request using Kafka's command-line tools and observe the activity-tracker-service's response.
1. Open a new terminal and navigate to your Kafka installation's bin directory (or use docker exec):
If using Docker Compose:
docker exec -it broker bash

2. Produce a request message to activity-query-requests:
kafka-console-producer.sh --broker-list broker:29092 --topic activity-query-requests \
  --property "parse.key=true" \
  --property "key.separator=:" \
  --property "value.serializer=org.apache.kafka.common.serialization.StringSerializer" \
  --property "acks=all" \
  --property "headers=KafkaReplyTopic:activity-query-replies,KafkaCorrelationId:my-request-id-123"

 * --broker-list broker:29092: Kafka broker address (using internal Docker hostname and port).
 * --topic activity-query-requests: The request topic.
 * --property "headers=...": This is how we inject the crucial KafkaReplyTopic and KafkaCorrelationId headers.
 * --property "parse.key=true" --property "key.separator=:": Allows us to enter "key:value" for the message.
After running the above command, type your request, then press Enter:
user-A:user-A-query-request

(Here, user-A is the key, and user-A-query-request is the value. Our listener only cares about the key for the user ID, but both are sent).
3. Observe activity-tracker-service logs:
You should see activity-tracker-service logging that it received the query, fetched activities for user-A (from its in-memory store), and then sent a reply to activity-query-replies.
4. Consume the reply from activity-query-replies:
In another terminal (or the same docker exec session):
kafka-console-consumer.sh --bootstrap-server broker:29092 --topic activity-query-replies --from-beginning --property print.headers=true

You should see the JSON list of UserActivityEvents, along with the KafkaReplyTopic and KafkaCorrelationId headers in the output.
This example illustrates a more intricate event-driven system using Kafka, showing how different services can communicate asynchronously and react to real-time events, and even handle requests and replies over Kafka topics.
