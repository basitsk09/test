Okay, let's consolidate and explicitly show the code for the JWT filter and its integration in a new, hypothetical microservice called order-service. This will demonstrate how you'd replicate the pattern from product-service if you were to add another protected microservice.
We'll assume the auth-service and api-gateway remain the same as previously defined.
New Microservice: Order Service
This service will manage customer orders and will have protected endpoints that require authentication and role-based authorization.
1. order-service/pom.xml (Dependencies):
(This will be very similar to product-service/pom.xml)
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.7</version> <relativePath/> </parent>
    <groupId>com.example</groupId>
    <artifactId>order-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>order-service</name>
    <description>Order Microservice</description>

    <properties>
        <java.version>17</java.version>
    </properties>

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
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.5</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
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
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

2. order-service/src/main/resources/application.properties:
server.port=8083 # Different port for this new service
app.jwt.secret=yourVeryLongAndSecureSecretKeyForJWTAuthServices1234567890 # MUST BE THE SAME AS AUTH-SERVICE!

Crucial: The app.jwt.secret must be identical to the one in auth-service and product-service.
3. Java Files (com.example.orderservice package):
 * OrderServiceApplication.java:
   package com.example.orderservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

 * order/Order.java (Simple Data Class):
   package com.example.orderservice.order;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Order {
    private Long id;
    private String customerUsername;
    private String productName;
    private int quantity;
    private double totalAmount;
}

 * jwt/JwtAuthFilter.java (The JWT Filter - Identical to Product Service's!):
   package com.example.orderservice.jwt; // Note the package name changes

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

import io.jsonwebtoken.Claims;

@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");
        String token = null;
        String username = null;
        String roles = null;

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            token = authHeader.substring(7);
            try {
                username = jwtUtil.extractUsername(token);
                Claims claims = jwtUtil.extractAllClaims(token);
                roles = claims.get("roles", String.class); // Retrieve roles from claims
            } catch (Exception e) {
                logger.error("Error parsing JWT in Order Service: " + e.getMessage());
                // Don't throw here directly, let Spring Security's mechanisms handle it
            }
        }

        // If username extracted AND no authentication is currently set in the context
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            // Here, we're assuming the token is valid if it's not expired and signed correctly
            // In a real system, you might add more checks (e.g., against a token blacklist)
            if (!jwtUtil.isTokenExpired(token)) {
                List<SimpleGrantedAuthority> authorities = Arrays.stream(roles.split(","))
                        .map(SimpleGrantedAuthority::new)
                        .collect(Collectors.toList());

                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                        username, null, authorities);
                SecurityContextHolder.getContext().setAuthentication(authToken);
            } else {
                logger.warn("Expired JWT token received in Order Service: " + token);
            }
        }
        filterChain.doFilter(request, response);
    }
}

 * jwt/JwtUtil.java (The JWT Utility - Identical to Product Service's!):
   package com.example.orderservice.jwt; // Note the package name changes

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.function.Function;

@Component
public class JwtUtil {

    @Value("${app.jwt.secret}")
    private String secret;

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    public Claims extractAllClaims(String token) {
        return Jwts
                .parserBuilder()
                .setSigningKey(getSignKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    public Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Key getSignKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}

 * config/SecurityConfig.java (Security Configuration - Identical to Product Service's structure):
   package com.example.orderservice.config; // Note the package name changes

import com.example.orderservice.jwt.JwtAuthFilter;
import com.example.orderservice.jwt.JwtUtil;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true) // Enable @PreAuthorize
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtUtil jwtUtil;

    @Bean
    public JwtAuthFilter jwtAuthFilter() {
        return new JwtAuthFilter(jwtUtil);
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // Disable CSRF for stateless API
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/orders/public/**").permitAll() // Public orders
                .anyRequest().authenticated() // All other requests require authentication
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // Use stateless sessions for JWT
            )
            .addFilterBefore(jwtAuthFilter(), UsernamePasswordAuthenticationFilter.class); // Add JWT filter

        return http.build();
    }
}

 * order/OrderController.java:
   package com.example.orderservice.order;

import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final List<Order> orders = new ArrayList<>();
    private Long nextOrderId = 1L;

    public OrderController() {
        orders.add(new Order(nextOrderId++, "user1", "Laptop", 1, 1200.00));
        orders.add(new Order(nextOrderId++, "admin", "Mouse", 2, 50.00));
        orders.add(new Order(nextOrderId++, "user1", "Keyboard", 1, 75.00));
    }

    // Public endpoint (not protected)
    @GetMapping("/public")
    public ResponseEntity<List<Order>> getPublicOrders() {
        return ResponseEntity.ok(orders);
    }

    // Protected endpoint - accessible by ROLE_USER or ROLE_ADMIN
    @GetMapping("/my-orders")
    @PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')")
    public ResponseEntity<List<Order>> getMyOrders() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String currentUsername = authentication.getName();
        // Filter orders specific to the authenticated user
        List<Order> userOrders = orders.stream()
                .filter(order -> order.getCustomerUsername().equals(currentUsername))
                .collect(Collectors.toList());
        return ResponseEntity.ok(userOrders);
    }

    // Admin-only endpoint - to view all orders
    @GetMapping
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    public ResponseEntity<List<Order>> getAllOrders() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        System.out.println("Admin user viewing all orders: " + authentication.getName());
        return ResponseEntity.ok(orders);
    }

    // Endpoint to create an order - accessible by ROLE_USER or ROLE_ADMIN
    @PostMapping
    @PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')")
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        // Ensure the order is for the authenticated user, or if admin, they can set it.
        if (!authentication.getAuthorities().contains(new SimpleGrantedAuthority("ROLE_ADMIN")) &&
            !order.getCustomerUsername().equals(authentication.getName())) {
            return ResponseEntity.status(403).build(); // Not allowed to create order for another user
        }

        order.setId(nextOrderId++);
        orders.add(order);
        return ResponseEntity.status(201).body(order);
    }
}

Update API Gateway Configuration:
You'll need to add a new route for the order-service in your api-gateway/src/main/resources/application.properties:
# api-gateway/src/main/resources/application.properties

server.port=8080

spring.cloud.gateway.routes[0].id=auth-service
spring.cloud.gateway.routes[0].uri=http://localhost:8081
spring.cloud.gateway.routes[0].predicates[0]=Path=/api/auth/**

spring.cloud.gateway.routes[1].id=product-service
spring.cloud.gateway.routes[1].uri=http://localhost:8082
spring.cloud.gateway.routes[1].predicates[0]=Path=/api/products/**

# NEW ROUTE FOR ORDER SERVICE
spring.cloud.gateway.routes[2].id=order-service
spring.cloud.gateway.routes[2].uri=http://localhost:8083
spring.cloud.gateway.routes[2].predicates[0]=Path=/api/orders/**

spring.webflux.cors.allow-credentials=true
spring.webflux.cors.allowed-origins=*
spring.webflux.cors.allowed-methods=GET,POST,PUT,DELETE,OPTIONS
spring.webflux.cors.allowed-headers=*
spring.webflux.cors.max-age=3600

Explanation with Code Flow for Order Service:
 * Client Request: A client sends a request like GET http://localhost:8080/api/orders/my-orders with an Authorization: Bearer <JWT_TOKEN> header.
 * API Gateway:
   * Receives the request on 8080.
   * Sees the path /api/orders/**.
   * Routes the request to http://localhost:8083 (the order-service). The Authorization header is forwarded along.
 * Order Service Receives Request:
   * The Spring Boot application running on 8083 receives the request.
   * Spring Security Filter Chain Activation: Because EnableWebSecurity is enabled and JwtAuthFilter is configured:
     * The request enters the JwtAuthFilter.
     * doFilterInternal is called.
 * Inside JwtAuthFilter.java (Order Service):
   // 1. Extract Header:
String authHeader = request.getHeader("Authorization"); // Gets "Bearer <JWT_TOKEN>"
String token = authHeader.substring(7); // Extracts "<JWT_TOKEN>"

// 2. Extract Username and Claims using JwtUtil:
// This uses the JwtUtil (which has the shared JWT_SECRET from application.properties)
// to verify the token's signature and expiration, and then extract the username and roles.
username = jwtUtil.extractUsername(token);
Claims claims = jwtUtil.extractAllClaims(token);
roles = claims.get("roles", String.class); // e.g., "ROLE_USER,ROLE_ADMIN"

// 3. Set Authentication Context (if token is valid):
if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
    if (!jwtUtil.isTokenExpired(token)) {
        // Convert comma-separated roles to Spring Security GrantedAuthority objects
        List<SimpleGrantedAuthority> authorities = Arrays.stream(roles.split(","))
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());

        // Create an Authentication object
        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                username, null, authorities); // `null` for password as we're authenticating by token

        // Set the authentication object in Spring's SecurityContextHolder.
        // THIS IS CRUCIAL. Now, Spring Security knows who the user is and what roles they have.
        SecurityContextHolder.getContext().setAuthentication(authToken);
    } else {
        logger.warn("Expired JWT token received...");
    }
}

// 4. Continue Filter Chain:
filterChain.doFilter(request, response); // Passes the request to the next filter (or controller)

 * Order Controller (OrderController.java):
   * The request now arrives at the OrderController.
   * Before executing a method like getMyOrders(), Spring Security (because of @EnableMethodSecurity) checks the @PreAuthorize annotation:
     @GetMapping("/my-orders")
@PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')")
public ResponseEntity<List<Order>> getMyOrders() {
    // ...
}

   * Spring Security looks at the SecurityContextHolder (which was populated by JwtAuthFilter). If the authenticated user (obtained from the JWT) has either ROLE_USER or ROLE_ADMIN, the method execution is allowed. Otherwise, a 403 Forbidden error is returned before the method's logic even runs.
   * Inside the controller method, SecurityContextHolder.getContext().getAuthentication() can be used to retrieve the authenticated user's details (username, roles) that were extracted from the JWT.
This detailed code walkthrough for order-service demonstrates that yes, for every microservice that needs to enforce security using JWTs, you'll replicate the JwtUtil and JwtAuthFilter pattern, along with the SecurityConfig to integrate them into Spring Security's flow.
