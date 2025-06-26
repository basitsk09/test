Building a complete microservices example with all these components from scratch involves a significant amount of code. Instead of providing the entire codebase here, I'll outline the architecture, key concepts, and provide code snippets for the core parts using Spring Boot and Spring Cloud. This will give you a solid foundation for learning and implementing it yourself.
Microservices Architecture Overview
We'll create the following services:
 * Eureka Server (Service Registry and Discovery): Central server for microservices to register themselves and discover others.
 * API Gateway (Spring Cloud Gateway): Single entry point for all client requests, handling routing, security (JWT), and potentially circuit breaking.
 * Auth Service (Microservice 1): Handles user authentication and generates JWT tokens.
 * Product Service (Microservice 2): Provides product-related data.
 * Order Service (Microservice 3): Handles order-related operations and interacts with Product Service.
Key Technologies:
 * Spring Boot: For building standalone, production-ready Spring applications.
 * Spring Cloud Netflix Eureka: For Service Registry and Discovery.
 * Spring Cloud Gateway: For API Gateway.
 * Spring Security & JWT: For Authentication and Authorization.
 * Spring Cloud OpenFeign: For declarative REST client (inter-service communication).
 * Spring Cloud Circuit Breaker (Resilience4j): For fault tolerance.
 * Maven: For build automation.
1. Eureka Server (Service Registry and Discovery)
This is the central hub where all your microservices will register themselves.
Project Setup:
 * Create a new Spring Boot project.
 * Dependencies: Spring Web, Eureka Server.
pom.xml (Eureka Server):
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.7</version> <relativePath/> </parent>
    <groupId>com.example</groupId>
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-server</name>
    <description>Eureka Server for Microservices</description>

    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.2</spring-cloud.version> </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

src/main/resources/application.yml (Eureka Server):
server:
  port: 8761 # Default Eureka server port

eureka:
  client:
    register-with-eureka: false # Eureka server itself doesn't register with itself
    fetch-registry: false       # Eureka server doesn't need to fetch registry
    serviceUrl:
      defaultZone: http://localhost:${server.port}/eureka/
  instance:
    hostname: localhost

src/main/java/com/example/eurekaserver/EurekaServerApplication.java:
package com.example.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}

2. API Gateway (Spring Cloud Gateway)
This acts as the single entry point for all external requests, routing them to the appropriate microservice. It will also handle JWT validation.
Project Setup:
 * Create a new Spring Boot project.
 * Dependencies: Spring WebFlux (Spring Cloud Gateway uses Reactor/WebFlux), Spring Cloud Gateway, Eureka Client, Spring Security, JJWT API, JJWT Impl, JJWT Jackson.
pom.xml (API Gateway):
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.7</version>
        <relativePath/>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>api-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>api-gateway</name>
    <description>API Gateway for Microservices</description>

    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.2</spring-cloud.version>
        <jjwt.version>0.12.3</jjwt.version> </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

src/main/resources/application.yml (API Gateway):
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        - id: auth-service
          uri: lb://AUTH-SERVICE # lb:// for load balancing with Eureka
          predicates:
            - Path=/auth/**
        - id: product-service
          uri: lb://PRODUCT-SERVICE
          predicates:
            - Path=/products/**
          filters:
            - JwtAuthFilter # Custom filter for JWT validation
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/orders/**
          filters:
            - JwtAuthFilter
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

jwt:
  secret: yourSecretKeyForJWTTokenGenerationValidation # Replace with a strong secret key

src/main/java/com/example/apigateway/ApiGatewayApplication.java:
package com.example.apigateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient // Enables this service to register with Eureka
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}

JWT Authentication Filter (src/main/java/com/example/apigateway/filter/JwtAuthFilter.java):
This is a custom GatewayFilter that intercepts requests to validate JWT tokens.
package com.example.apigateway.filter;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.security.Key;
import java.util.Date;
import java.util.List;
import java.util.function.Predicate;

@Component
public class JwtAuthFilter extends AbstractGatewayFilterFactory<JwtAuthFilter.Config> {

    @Value("${jwt.secret}")
    private String secret;

    // Endpoints that do not require authentication
    public static final List<String> openApiEndpoints = List.of(
            "/auth/register",
            "/auth/login"
    );

    public JwtAuthFilter() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();

            Predicate<ServerHttpRequest> isSecured = r -> openApiEndpoints.stream()
                    .noneMatch(uri -> r.getURI().getPath().contains(uri));

            if (isSecured.test(request)) {
                if (!request.getHeaders().containsKey(HttpHeaders.AUTHORIZATION)) {
                    return onError(exchange, "Authorization header is missing", HttpStatus.UNAUTHORIZED);
                }

                String authHeader = request.getHeaders().get(HttpHeaders.AUTHORIZATION).get(0);
                if (authHeader != null && authHeader.startsWith("Bearer ")) {
                    authHeader = authHeader.substring(7);
                } else {
                    return onError(exchange, "Invalid Authorization header", HttpStatus.UNAUTHORIZED);
                }

                try {
                    validateToken(authHeader);
                    // You can extract claims and add them to request headers if needed
                    // Claims claims = extractAllClaims(authHeader);
                    // exchange.getRequest().mutate().header("username", claims.getSubject()).build();

                } catch (Exception e) {
                    return onError(exchange, "Unauthorized access to application", HttpStatus.UNAUTHORIZED);
                }
            }
            return chain.filter(exchange);
        };
    }

    private Mono<Void> onError(ServerWebExchange exchange, String err, HttpStatus httpStatus) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(httpStatus);
        return response.setComplete();
    }

    private Key getSignKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    public void validateToken(final String token) {
        Jwts.parserBuilder().setSigningKey(getSignKey()).build().parseClaimsJws(token);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSignKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    public static class Config {
        // Put the configuration properties for your filter here
    }
}

3. Microservices (Auth, Product, Order)
Each microservice will follow a similar structure.
Common Dependencies for Microservices:
 * Spring Web
 * Eureka Client
 * Spring Boot DevTools
 * Lombok (optional, for less boilerplate code)
 * Spring Security (for Auth Service)
 * JJWT (for Auth Service)
 * Spring Cloud OpenFeign (for Order Service to call Product Service)
 * Spring Cloud Starter Circuit Breaker Resilience4j
pom.xml (Example for Auth Service - similar for others, adjust name/dependencies):
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.7</version>
        <relativePath/>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>auth-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>auth-service</name>
    <description>Authentication Microservice</description>

    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.2</spring-cloud.version>
        <jjwt.version>0.12.3</jjwt.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>${jjwt.version}</version>
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
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

src/main/resources/application.yml (Example for Auth Service - adjust port and name):
server:
  port: 8081 # Unique port for each microservice

spring:
  application:
    name: auth-service # Unique name for each microservice

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true # Recommended for Docker/containerized environments

jwt:
  secret: yourSecretKeyForJWTTokenGenerationValidation # Must be same as API Gateway

Microservice Main Application Class:
package com.example.authservice; // Adjust package name

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class AuthServiceApplication { // Adjust class name

    public static void main(String[] args) {
        SpringApplication.run(AuthServiceApplication.class, args);
    }
}

Auth Service Implementation Details:
This service will handle user registration and login, generating a JWT token upon successful authentication.
src/main/java/com/example/authservice/config/SecurityConfig.java:
package com.example.authservice.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        // For simplicity, we'll use an in-memory user store.
        // In a real application, you'd use a database (e.g., Spring Data JPA with UserDetailsService).
        // This is a placeholder for demonstration.
        return username -> {
            if ("user".equals(username)) {
                return org.springframework.security.core.userdetails.User.withUsername("user")
                        .password(passwordEncoder().encode("password"))
                        .roles("USER")
                        .build();
            }
            throw new org.springframework.security.core.userdetails.UsernameNotFoundException("User not found: " + username);
        };
    }

    @Bean
    public AuthenticationManager authenticationManager(UserDetailsService userDetailsService, PasswordEncoder passwordEncoder) {
        DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailsService);
        authenticationProvider.setPasswordEncoder(passwordEncoder);
        return new ProviderManager(authenticationProvider);
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/auth/register", "/auth/login").permitAll()
                .anyRequest().authenticated()
            )
            .httpBasic(withDefaults()); // Basic authentication for simplicity, but we'll use JWT for actual auth

        return http.build();
    }
}

src/main/java/com/example/authservice/util/JwtUtil.java:
package com.example.authservice.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtUtil {

    @Value("${jwt.secret}")
    private String secret;

    public String generateToken(String userName) {
        Map<String, Object> claims = new HashMap<>();
        return createToken(claims, userName);
    }

    private String createToken(Map<String, Object> claims, String userName) {
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(userName)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 30)) // 30 minutes expiration
                .signWith(getSignKey(), io.jsonwebtoken.SignatureAlgorithm.HS256)
                .compact();
    }

    private Key getSignKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSignKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public Boolean validateToken(String token, String username) {
        final String extractedUsername = extractUsername(token);
        return (extractedUsername.equals(username) && !isTokenExpired(token));
    }
}

src/main/java/com/example/authservice/controller/AuthController.java:
package com.example.authservice.controller;

import com.example.authservice.util.AuthRequest;
import com.example.authservice.util.AuthResponse;
import com.example.authservice.util.JwtUtil;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
public class AuthController {

    private final JwtUtil jwtUtil;
    private final AuthenticationManager authenticationManager;

    @PostMapping("/register")
    public String registerUser(@RequestBody AuthRequest authRequest) {
        // In a real application, you'd save the user to a database after encoding the password
        // For this example, we're just simulating registration.
        // You can add logic to check if user already exists etc.
        System.out.println("User registered: " + authRequest.getUsername());
        return "User registered successfully!";
    }

    @PostMapping("/login")
    public AuthResponse authenticateAndGetToken(@RequestBody AuthRequest authRequest) {
        Authentication authentication = authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(authRequest.getUsername(), authRequest.getPassword()));
        if (authentication.isAuthenticated()) {
            String token = jwtUtil.generateToken(authRequest.getUsername());
            return new AuthResponse(token);
        } else {
            throw new UsernameNotFoundException("Invalid user request!");
        }
    }
}

src/main/java/com/example/authservice/util/AuthRequest.java (POJO for login/register):
package com.example.authservice.util;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class AuthRequest {
    private String username;
    private String password;
}

src/main/java/com/example/authservice/util/AuthResponse.java (POJO for JWT response):
package com.example.authservice.util;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class AuthResponse {
    private String jwtToken;
}

Product Service Implementation Details:
A simple service to provide product information.
src/main/java/com/example/productservice/controller/ProductController.java:
package com.example.productservice.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/products")
public class ProductController {

    private final Map<String, String> products = new HashMap<>();

    public ProductController() {
        products.put("1", "Laptop");
        products.put("2", "Mouse");
        products.put("3", "Keyboard");
    }

    @GetMapping("/{id}")
    public String getProductById(@PathVariable String id) {
        return "Product: " + products.getOrDefault(id, "Product Not Found");
    }

    @GetMapping
    public String getAllProducts() {
        return "All Products: " + products.values();
    }
}

Order Service Implementation Details (with Feign Client and Circuit Breaker):
This service will interact with the Product Service using Feign Client and will implement a Circuit Breaker pattern.
pom.xml (Order Service - add Feign and Circuit Breaker dependencies):
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.7</version>
        <relativePath/>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>order-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>order-service</name>
    <description>Order Microservice</description>

    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId> </dependency>
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
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

src/main/resources/application.yml (Order Service - adjust port and name):
server:
  port: 8083

spring:
  application:
    name: order-service
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
    circuitbreaker:
      resilience4j:
        circuitbreaker:
          configs:
            default:
              slidingWindowSize: 10
              failureRateThreshold: 50
              waitDurationInOpenState: 5s
              permittedNumberOfCallsInHalfOpenState: 3
              recordExceptions:
                - org.springframework.web.client.HttpServerErrorException
                - java.io.IOException
          instances:
            productServiceBreaker: # Name of the circuit breaker instance
              baseConfig: default
# For Actuator endpoints to monitor circuit breaker
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

src/main/java/com/example/orderservice/OrderServiceApplication.java:
package com.example.orderservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker; // Deprecated, but still works for some versions
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients // Enable Feign clients in this application
//@EnableCircuitBreaker // Consider using @EnableSpringCircuitBreaker if you're on a newer Spring Cloud version
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

src/main/java/com/example/orderservice/client/ProductClient.java (Feign Client):
package com.example.orderservice.client;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "PRODUCT-SERVICE") // 'PRODUCT-SERVICE' is the application name of Product Service in Eureka
public interface ProductClient {

    @GetMapping("/products/{id}")
    String getProductById(@PathVariable("id") String id);
}

src/main/java/com/example/orderservice/controller/OrderController.java (with Circuit Breaker):
package com.example.orderservice.controller;

import com.example.orderservice.client.ProductClient;
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/orders")
@RequiredArgsConstructor
public class OrderController {

    private final ProductClient productClient;

    @GetMapping("/create/{productId}")
    @CircuitBreaker(name = "productServiceBreaker", fallbackMethod = "createOrderFallback")
    public String createOrder(@PathVariable String productId) {
        String productDetails = productClient.getProductById(productId);
        // In a real application, you'd save order details to a database
        return "Order created for " + productDetails;
    }

    // Fallback method for Circuit Breaker
    public String createOrderFallback(String productId, Throwable t) {
        System.out.println("Fallback method executed because of: " + t.getMessage());
        return "Order creation failed for product " + productId + ". Please try again later. (Fallback)";
    }
}

src/main/java/com/example/productservice/config/WebSecurityConfig.java (Empty/Disabled security for simplicity in internal microservices, or configure as needed):
For Product and Order Services, you might want to disable Spring Security by default if you're relying solely on the API Gateway for authentication/authorization.
package com.example.productservice.config; // Adjust package for Order Service too

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().permitAll() // Allow all requests for internal services after gateway authentication
            );
        return http.build();
    }
}

How to Run and Test:
 * Start Eureka Server: Run EurekaServerApplication.java (port 8761).
 * Start Auth Service: Run AuthServiceApplication.java (port 8081).
 * Start Product Service: Run ProductServiceApplication.java (port 8082).
 * Start Order Service: Run OrderServiceApplication.java (port 8083).
 * Start API Gateway: Run ApiGatewayApplication.java (port 8080).
Testing Flow:
 * Register User (via Gateway):
   * POST to http://localhost:8080/auth/register
   * Body (JSON):
     {
    "username": "testuser",
    "password": "testpassword"
}

   * You should get "User registered successfully!"
 * Login and Get JWT Token (via Gateway):
   * POST to http://localhost:8080/auth/login
   * Body (JSON):
     {
    "username": "testuser",
    "password": "testpassword"
}

   * You should get a JSON response like {"jwtToken": "eyJ..."}. Copy this token.
 * Access Product Service (without Token - should fail):
   * GET to http://localhost:8080/products/1
   * Should return 401 Unauthorized.
 * Access Product Service (with Token - should succeed):
   * GET to http://localhost:8080/products/1
   * Add Authorization header: Bearer YOUR_JWT_TOKEN_HERE
   * Should return "Product: Laptop".
 * Access Order Service (with Token):
   * GET to http://localhost:8080/orders/create/1
   * Add Authorization header: Bearer YOUR_JWT_TOKEN_HERE
   * Should return "Order created for Product: Laptop".
Testing Circuit Breaker:
 * Stop Product Service.
 * Access Order Service multiple times:
   * GET to http://localhost:8080/orders/create/1 (with JWT token)
   * Initially, you might see connection refused errors (or a timeout).
   * After a few failures (configured by slidingWindowSize and failureRateThreshold), the circuit breaker will open, and you'll immediately get the fallback message: "Order creation failed for product 1. Please try again later. (Fallback)".
   * Wait for the waitDurationInOpenState (5 seconds in this config).
   * After the wait duration, the circuit will go into HALF_OPEN state. The next few requests will attempt to call the Product Service (permittedNumberOfCallsInHalfOpenState). If they succeed, the circuit closes. If they fail, it goes back to OPEN.
This setup provides a comprehensive example of microservices patterns with Spring Boot and Spring Cloud. Remember to adjust ports, names, and security configurations as per your actual requirements.
