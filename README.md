You're asking for a rate limiter implementation, which is a crucial component for microservices to prevent abuse, manage traffic, and ensure fair resource usage.
I'll provide an example of a rate limiter implemented at the API Gateway level (Spring Cloud Gateway), as this is the most common and effective place to apply rate limiting in a microservices architecture. It acts as the first line of defense before requests even hit your backend services.
Key Concepts of Rate Limiting:
 * Token Bucket Algorithm: A common algorithm where requests "take a token" from a bucket. If the bucket is empty, the request is rejected or queued. Tokens are added to the bucket at a fixed rate.
 * Leaky Bucket Algorithm: Similar to token bucket, but requests are processed at a steady rate, like water leaking from a bucket. Excess requests overflow.
 * Fixed Window Counter: Requests within a time window (e.g., 60 seconds) are counted. Once the limit is reached, further requests are rejected until the next window.
 * Sliding Window Log: More accurate than fixed window, keeps a log of timestamps for each request and evicts old ones.
 * Sliding Window Counter: Combines fixed and sliding window for better accuracy with less memory than sliding window log.
Spring Cloud Gateway's Built-in Rate Limiter:
Spring Cloud Gateway provides a powerful and flexible RequestRateLimiter filter, which supports the Token Bucket algorithm out-of-the-box using Redis for distributed rate limiting.
Why Redis?
For rate limiting across multiple instances of your API Gateway (or across different microservices), you need a shared, fast, and persistent store for rate limit counters. Redis is ideal for this due to its in-memory data structures and speed.
Implementation Steps for Rate Limiter (Spring Cloud Gateway with Redis)
1. Set up Redis:
If you don't have Redis running, the easiest way is with Docker:
docker run --name my-redis -p 6379:6379 -d redis/redis-stack-server:latest

(This uses redis/redis-stack-server which includes RedisGraph, RedisJSON etc. If you just need core Redis, redis:latest is fine)
2. Update api-gateway/pom.xml:
Add the Spring Boot Starter Data Redis Reactive dependency.
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-jwt-shared</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <artifactId>api-gateway</artifactId>
    <name>api-gateway</name>
    <description>API Gateway using Spring Cloud Gateway</description>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

3. Update api-gateway/src/main/resources/application.properties:
Add Redis connection properties and configure the RequestRateLimiter filter for your routes.
server.port=8080

# Redis Configuration
spring.data.redis.host=localhost
spring.data.redis.port=6379

# --- Route Definitions with Rate Limiting ---

spring.cloud.gateway.routes[0].id=auth-service
spring.cloud.gateway.routes[0].uri=http://localhost:8081
spring.cloud.gateway.routes[0].predicates[0]=Path=/api/auth/**
spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter
spring.cloud.gateway.routes[0].filters[0].redis-rate-limiter.replenishRate=1 # Tokens added per second
spring.cloud.gateway.routes[0].filters[0].redis-rate-limiter.burstCapacity=2 # Max tokens in bucket (initial burst)
spring.cloud.gateway.routes[0].filters[0].redis-rate-limiter.requestedTokens=1 # Tokens required per request (default 1)
spring.cloud.gateway.routes[0].filters[0].key-resolver=#{@ipKeyResolver} # Use IP address for rate limiting

spring.cloud.gateway.routes[1].id=product-service
spring.cloud.gateway.routes[1].uri=http://localhost:8082
spring.cloud.gateway.routes[1].predicates[0]=Path=/api/products/**
spring.cloud.gateway.routes[1].filters[0]=RequestRateLimiter
spring.cloud.gateway.routes[1].filters[0].redis-rate-limiter.replenishRate=5 # Allow 5 requests/sec
spring.cloud.gateway.routes[1].filters[0].redis-rate-limiter.burstCapacity=10 # Allow 10 initial requests
spring.cloud.gateway.routes[1].filters[0].key-resolver=#{@userKeyResolver} # Use authenticated user for rate limiting

spring.cloud.gateway.routes[2].id=order-service
spring.cloud.gateway.routes[2].uri=http://localhost:8083
spring.cloud.gateway.routes[2].predicates[0]=Path=/api/orders/**
spring.cloud.gateway.routes[2].filters[0]=RequestRateLimiter
spring.cloud.gateway.routes[2].filters[0].redis-rate-limiter.replenishRate=3 # Allow 3 requests/sec
spring.cloud.gateway.routes[2].filters[0].redis-rate-limiter.burstCapacity=5 # Allow 5 initial requests
spring.cloud.gateway.routes[2].filters[0].key-resolver=#{@ipKeyResolver} # Use IP address for rate limiting

spring.webflux.cors.allow-credentials=true
spring.webflux.cors.allowed-origins=*
spring.webflux.cors.allowed-methods=GET,POST,PUT,DELETE,OPTIONS
spring.webflux.cors.allowed-headers=*
spring.webflux.cors.max-age=3600

Rate Limiter Parameters Explanation:
 * replenishRate: How many tokens are added to the bucket per second. This is the sustained rate. (e.g., 5 means 5 requests per second).
 * burstCapacity: The maximum number of tokens the bucket can hold. This defines the maximum "burst" of requests allowed at once. (e.g., 10 means 10 requests can be handled quickly if the bucket is full).
 * requestedTokens: How many tokens each request consumes (defaults to 1).
4. Create api-gateway/src/main/java/com/example/apigateway/config/RateLimiterConfig.java:
This file defines the KeyResolver beans, which tell the rate limiter what to apply the limit to (e.g., by IP address, by authenticated user, by API key, etc.).
package com.example.apigateway.config;

import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.ReactiveSecurityContextHolder;
import org.springframework.security.core.context.SecurityContext;
import reactor.core.publisher.Mono;

@Configuration
public class RateLimiterConfig {

    // Rate limit based on IP address
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getAddress().getHostAddress());
    }

    // Rate limit based on authenticated user (username)
    // This requires Spring Security to be set up and able to extract user from JWT
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> ReactiveSecurityContextHolder.getContext() // Get reactive security context
                .map(SecurityContext::getAuthentication)             // Get authentication object
                .filter(Authentication::isAuthenticated)             // Ensure user is authenticated
                .map(Authentication::getName)                        // Get username
                .defaultIfEmpty("anonymous");                        // Fallback for unauthenticated users
    }

    // You could also create other KeyResolvers, e.g., for an API Key from a custom header:
    /*
    @Bean
    public KeyResolver apiKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getHeaders().getFirst("X-API-KEY"));
    }
    */
}

Explanation of KeyResolvers:
 * ipKeyResolver(): This is simple and effective for general traffic control. It limits requests from a specific client IP address.
 * userKeyResolver(): This is more sophisticated. It tries to get the authenticated user's name from Spring Security's reactive context.
   * If a user is logged in (and the JWT is validated by a downstream service that sets the context, or if the Gateway itself was doing some security processing), their requests will be limited by their username.
   * If no user is authenticated (e.g., a request to /api/auth/login or a public endpoint), it falls back to "anonymous", which means all anonymous requests share a single rate limit. You might want to use ipKeyResolver for anonymous traffic and userKeyResolver for authenticated.
Integrating userKeyResolver's with JWT (Important Consideration):
For userKeyResolver to work, the Authentication object must be present in the ReactiveSecurityContextHolder. In our current architecture, the API Gateway itself doesn't validate the JWT and set the SecurityContext. The JWT validation happens downstream in the product-service and order-service.
Therefore, userKeyResolver as implemented above will only work if:
 * You implement JWT validation within the API Gateway itself (more complex, typically involves a custom filter in Gateway that parses JWT and sets context). This is a common pattern for "centralized authentication" at the Gateway.
 * You use a different KeyResolver for authenticated users that doesn't rely on Spring Security's context at the Gateway (e.g., extract userId directly from JWT in a custom gateway filter before the rate limiter).
For simplicity and to directly leverage Spring Cloud Gateway's RequestRateLimiter with userKeyResolver, we would need to add a JWT validation filter to the API Gateway. This makes the Gateway more intelligent about security, but also adds complexity.
Let's choose a practical approach for now:
 * For Auth Service, use ipKeyResolver (login/register are often public and limited by IP).
 * For Product Service and Order Service, we can still use userKeyResolver. When an authenticated request comes, it will have the Authorization header. Although the Gateway itself might not set the SecurityContextHolder, the userKeyResolver will likely return anonymous for all requests because the Authentication object isn't available at the Gateway level in this specific setup.
Corrected approach for userKeyResolver in a Gateway that doesn't validate JWT:
If the Gateway doesn't do JWT validation itself, userKeyResolver as written won't differentiate between authenticated users. A simpler and more practical KeyResolver for authenticated users at the Gateway level (without central JWT validation) would involve extracting a user ID from the Authorization header (if present) before the rate limiter:
This would require a custom filter to extract the user ID and pass it along. But for a direct RequestRateLimiter configuration, ipKeyResolver is the most straightforward for all routes if the Gateway isn't parsing JWTs.
For this example, I'll stick to ipKeyResolver for all routes to keep it simple and functional without making the API Gateway a security processor. If you need user-specific rate limits, you'd need a custom Gateway filter that extracts userId from the JWT (and logs/caches it if needed) and uses that as the key, or use a framework like Spring Cloud Security which can integrate JWT validation directly into the Gateway.
Revised application.properties to use ipKeyResolver for all for simplicity:
server.port=8080

# Redis Configuration
spring.data.redis.host=localhost
spring.data.redis.port=6379

# --- Route Definitions with Rate Limiting ---

spring.cloud.gateway.routes[0].id=auth-service
spring.cloud.gateway.routes[0].uri=http://localhost:8081
spring.cloud.gateway.routes[0].predicates[0]=Path=/api/auth/**
spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter
spring.cloud.gateway.routes[0].filters[0].redis-rate-limiter.replenishRate=1 # Tokens added per second
spring.cloud.gateway.routes[0].filters[0].redis-rate-limiter.burstCapacity=2 # Max tokens in bucket (initial burst)
spring.cloud.gateway.routes[0].filters[0].key-resolver=#{@ipKeyResolver} # Use IP address for rate limiting

spring.cloud.gateway.routes[1].id=product-service
spring.cloud.gateway.routes[1].uri=http://localhost:8082
spring.cloud.gateway.routes[1].predicates[0]=Path=/api/products/**
spring.cloud.gateway.routes[1].filters[0]=RequestRateLimiter
spring.cloud.gateway.routes[1].filters[0].redis-rate-limiter.replenishRate=5 # Allow 5 requests/sec
spring.cloud.gateway.routes[1].filters[0].redis-rate-limiter.burstCapacity=10 # Allow 10 initial requests
spring.cloud.gateway.routes[1].filters[0].key-resolver=#{@ipKeyResolver} # Use IP address for rate limiting

spring.cloud.gateway.routes[2].id=order-service
spring.cloud.gateway.routes[2].uri=http://localhost:8083
spring.cloud.gateway.routes[2].predicates[0]=Path=/api/orders/**
spring.cloud.gateway.routes[2].filters[0]=RequestRateLimiter
spring.cloud.gateway.routes[2].filters[0].redis-rate-limiter.replenishRate=3 # Allow 3 requests/sec
spring.cloud.gateway.routes[2].filters[0].redis-rate-limiter.burstCapacity=5 # Allow 5 initial requests
spring.cloud.gateway.routes[2].filters[0].key-resolver=#{@ipKeyResolver} # Use IP address for rate limiting

spring.webflux.cors.allow-credentials=true
spring.webflux.cors.allowed-origins=*
spring.webflux.cors.allowed-methods=GET,POST,PUT,DELETE,OPTIONS
spring.webflux.cors.allowed-headers=*
spring.webflux.cors.max-age=3600

How to Test the Rate Limiter:
 * Start Redis (as shown above).
 * Build all projects from the parent directory: mvn clean install
 * Start all services: auth-service, product-service, order-service, and api-gateway.
   * Ensure api-gateway starts successfully and connects to Redis (check its logs).
 * Test Auth Service Rate Limiting (replenishRate=1, burstCapacity=2):
   * Open your terminal.
   * Rapidly send multiple login requests (more than 2 within a second):
     curl -X POST -H "Content-Type: application/json" -d '{"username": "admin", "password": "adminpassword"}' http://localhost:8080/api/auth/login
# Press Up arrow and Enter very quickly multiple times

   * Expected Behavior:
     * The first 1-2 requests (depending on timing) should succeed and return a JWT.
     * Subsequent requests sent too quickly will return a 429 Too Many Requests HTTP status code.
     * Wait a second, and you should be able to send another request successfully (as tokens replenish).
 * Test Product Service Rate Limiting (replenishRate=5, burstCapacity=10):
   * You'll need an Admin or User token first. Login via /api/auth/login.
   * Rapidly send multiple product requests (more than 10 quickly, then more than 5 per second):
     # Replace <YOUR_TOKEN>
curl -H "Authorization: Bearer <YOUR_TOKEN>" http://localhost:8080/api/products
# Press Up arrow and Enter very quickly multiple times

   * Expected Behavior: Similar to auth service, you'll initially get responses, then 429 Too Many Requests when limits are hit.
This implementation provides a robust and distributed rate-limiting mechanism at the API Gateway using Spring Cloud Gateway and Redis. Remember to adjust replenishRate and burstCapacity based on your actual traffic requirements and desired limits for different API endpoints.
