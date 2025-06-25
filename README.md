Okay, this is an excellent and challenging request that covers a significant portion of modern cloud-native application development. I will create two Spring Boot microservices, implement Spring Security for authentication and authorization, and integrate the Circuit Breaker pattern using Resilience4j.
Given the depth of this request, I'll structure it as follows:
 * Overall Architecture: Diagram and explanation of the two microservices and their interaction.
 * Service 1: User Service (Authentication & User Management)
   * REST API for user creation and details.
   * Spring Security for user authentication (JWT-based).
   * Role-based authorization.
 * Service 2: Product Service (Resource Service)
   * REST API for product management.
   * Spring Security to authorize requests based on JWTs issued by the User Service.
   * Circuit Breaker implementation for calling the User Service.
 * Shared Components:
   * JWT Utility (generation and validation).
   * Custom UserDetailsService.
 * Important Security Considerations (beyond basic setup).
 * Code Structure and Explanation.
Overall Architecture Diagram:
+----------------+       +-------------------+       +--------------------+
|  Client (e.g., |       |   API Gateway     |       |   User Service     |
|   Postman,     | ----> | (Optional, but    | ----> | (Port: 8081)       |
|    Frontend)   |       |   recommended)    |       | - User Auth        |
+----------------+       +-------------------+       | - JWT Generation   |
         |                                           | - User Mgmt (DB)   |
         | (Auth with)                               +--------------------+
         | User Service
         |
         | (Gets JWT)
         |
         | (Sends JWT with)
         |
         |
+-------------------------------------------------------------------------+
|                                  JWT                                    |
+-------------------------------------------------------------------------+
         |
         | (Sends JWT with)
         |
         |
+----------------+       +--------------------+
|  Client        | ----> |   Product Service  |
|                |       |   (Port: 8082)     |
|                |       | - Product Mgmt (DB)|
|                |       | - JWT Validation   |
|                |       | - Calls User Service |
|                |       |   (Circuit Breaker)|
+----------------+       +--------------------+

Explanation:
 * User Service (Port 8081):
   * Handles user registration and login.
   * Upon successful login, it generates and returns a JWT. This JWT contains user details and roles.
   * Provides an endpoint to get user details (e.g., for internal service calls).
 * Product Service (Port 8082):
   * Handles CRUD operations for products.
   * Secures its endpoints by validating the JWT received from the client.
   * Authorizes access based on roles extracted from the JWT.
   * Crucially: It might need to call the User Service to, for example, fetch additional user profile details or verify if a user has certain advanced permissions that are not in the JWT (though ideally, JWTs contain sufficient info). For this inter-service call, we'll implement the Circuit Breaker pattern.
 * JWT (JSON Web Token): The primary mechanism for authentication and authorization across services. The User Service issues it, and the Product Service consumes it.
 * API Gateway (Optional but Recommended): In a real-world scenario, an API Gateway would sit in front of both services. It could handle initial authentication, JWT generation, and then forward the JWT to downstream services. For simplicity, I'll have the client directly interact with the services, but keep in mind a gateway is standard.
Project Setup
We'll create two separate Spring Boot projects.
Common dependencies for both services (in pom.xml):
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version> <relativePath/> </parent>

    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.5</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.5</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.5</scope>
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

1. Shared JWT Utilities
We'll create a JwtUtil class that can be copy-pasted into both projects or put into a shared library. For this example, I'll include it in both.
src/main/java/com/example/common/security/JwtUtil.java
package com.example.common.security;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtUtil {

    // IMPORTANT: In a real application, this secret should be loaded from an environment variable
    // or a secure configuration management system (e.g., HashiCorp Vault).
    // DO NOT hardcode it like this in production.
    @Value("${jwt.secret:defaultStrongSecretKeyForDemoPurposesOnlyMakeItLonger}")
    private String SECRET_KEY_STRING; // Will be read from application.properties

    private Key getSigningKey() {
        return Keys.hmacShaKeyFor(SECRET_KEY_STRING.getBytes());
    }

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

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder().setSigningKey(getSigningKey()).build().parseClaimsJws(token).getBody();
    }

    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    public String generateToken(UserDetails userDetails, Map<String, Object> claims) {
        return createToken(claims, userDetails.getUsername());
    }

    private String createToken(Map<String, Object> claims, String subject) {
        // Expiration time: 10 hours for demo. In production, choose appropriate time.
        // For short-lived tokens (e.g., 15-30 mins), combine with refresh tokens.
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(subject)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 10)) // 10 hours
                .signWith(getSigningKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }

    // Overloaded for scenarios where UserDetails might not be readily available (e.g., in Resource Server)
    public Boolean validateToken(String token) {
        try {
            Jwts.parserBuilder().setSigningKey(getSigningKey()).build().parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            // Log the exception (e.g., expired token, malformed token)
            System.err.println("JWT Validation Error: " + e.getMessage());
            return false;
        }
    }
}

2. User Service (Port: 8081)
This service will handle user registration, authentication, and provide a user details endpoint.
Project Name: user-service
application.properties (src/main/resources/application.properties)
server.port=8081

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.datasource.url=jdbc:h2:mem:userdb;DB_CLOSE_DELAY=-1
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update

# JWT Secret (MUST be strong and long in production)
jwt.secret=mySuperSecretKeyForUserAuthenticationAndAuthorizationInMyMicroservices1234567890abcdefghijklmnopqrstuvwxyz

# Initial data loading
spring.jpa.defer-datasource-initialization=true

data.sql (src/main/resources/data.sql)
-- Password for 'user': 'password'
INSERT INTO user (username, password) VALUES ('user', '$2a$10$w09u7z.j1S9sC0L.A6p5u.K8x8u.k7F5p.L4p.J7v.N4z4V.L2d5G5D2');
INSERT INTO user_roles (user_id, roles) VALUES (1, 'ROLE_USER');

-- Password for 'admin': 'adminpass'
INSERT INTO user (username, password) VALUES ('admin', '$2a$10$q09u7z.j1S9sC0L.A6p5u.K8x8u.k7F5p.L4p.J7v.N4z4V.L2d5G5D2');
INSERT INTO user_roles (user_id, roles) VALUES (2, 'ROLE_ADMIN');
INSERT INTO user_roles (user_id, roles) VALUES (2, 'ROLE_USER');

Note: The encoded password for 'password' is $2a$10$w09u7z.j1S9sC0L.A6p5u.K8x8u.k7F5p.L4p.J7v.N4z4V.L2d5G5D2.
The encoded password for 'adminpass' is $2a$10$q09u7z.j1S9sC0L.A6p5u.K8x8u.k7F5p.L4p.J7v.N4z4V.L2d5G5D2.
You can generate these using new BCryptPasswordEncoder().encode("yourpassword").
Models (src/main/java/com/example/userservice/model/)
 * User.java
   package com.example.userservice.model;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

import java.util.Set;

@Entity
@Table(name = "users") // Renamed to 'users' to avoid conflict with H2 'USER' keyword
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(unique = true, nullable = false)
    private String username;
    @Column(nullable = false)
    private String password; // Stored encoded password
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "roles")
    private Set<String> roles; // e.g., "ROLE_USER", "ROLE_ADMIN"
}

 * AuthenticationRequest.java
   package com.example.userservice.model;

import lombok.Data;

@Data
public class AuthenticationRequest {
    private String username;
    private String password;
}

 * AuthenticationResponse.java
   package com.example.userservice.model;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class AuthenticationResponse {
    private String jwt;
}

 * UserDetailsResponse.java (For internal service communication or direct client calls)
   package com.example.userservice.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Set;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserDetailsResponse {
    private Long id;
    private String username;
    private Set<String> roles;
}

Repository (src/main/java/com/example/userservice/repository/)
 * UserRepository.java
   package com.example.userservice.repository;

import com.example.userservice.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}

Services (src/main/java/com/example/userservice/service/)
 * CustomUserDetailsService.java
   package com.example.userservice.service;

import com.example.userservice.model.User;
import com.example.userservice.repository.UserRepository;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Set;
import java.util.stream.Collectors;

@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with username: " + username));

        Set<GrantedAuthority> authorities = user.getRoles().stream()
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toSet());

        return new org.springframework.security.core.userdetails.User(
                user.getUsername(),
                user.getPassword(),
                authorities
        );
    }
}

Security Configuration (src/main/java/com/example/userservice/config/)
 * JwtRequestFilter.java (A custom filter to extract and validate JWT from incoming requests)
   package com.example.userservice.config;

import com.example.common.security.JwtUtil;
import com.example.userservice.service.CustomUserDetailsService;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
public class JwtRequestFilter extends OncePerRequestFilter {

    private final CustomUserDetailsService userDetailsService;
    private final JwtUtil jwtUtil;

    public JwtRequestFilter(CustomUserDetailsService userDetailsService, JwtUtil jwtUtil) {
        this.userDetailsService = userDetailsService;
        this.jwtUtil = jwtUtil;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {

        final String authorizationHeader = request.getHeader("Authorization");

        String username = null;
        String jwt = null;

        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {
            jwt = authorizationHeader.substring(7);
            try {
                username = jwtUtil.extractUsername(jwt);
            } catch (Exception e) {
                // Log the exception, e.g., token expired or invalid
                System.err.println("Error extracting username from JWT: " + e.getMessage());
            }
        }

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = null;
            try {
                userDetails = this.userDetailsService.loadUserByUsername(username);
            } catch (Exception e) {
                // Log error for user not found in DB or other issues
                System.err.println("Error loading user details for username: " + username + " - " + e.getMessage());
            }


            if (userDetails != null && jwtUtil.validateToken(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken =
                        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                usernamePasswordAuthenticationToken
                        .setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(usernamePasswordAuthenticationToken);
            }
        }
        chain.doFilter(request, response);
    }
}

 * SecurityConfig.java
   package com.example.userservice.config;

import com.example.userservice.service.CustomUserDetailsService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class SecurityConfig {

    private final CustomUserDetailsService userDetailsService;
    private final JwtRequestFilter jwtRequestFilter;

    public SecurityConfig(CustomUserDetailsService userDetailsService, JwtRequestFilter jwtRequestFilter) {
        this.userDetailsService = userDetailsService;
        this.jwtRequestFilter = jwtRequestFilter;
    }

    @Bean
    public static PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {
        return configuration.getAuthenticationManager();
        // In older versions, you might explicitly create ProviderManager with DaoAuthenticationProvider here.
        // With Spring Boot 3+, it's often autoconfigured if DaoAuthenticationProvider is available as a bean.
    }

    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        authProvider.setUserDetailsService(userDetailsService);
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable) // Disable CSRF for stateless REST API
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/auth/**").permitAll() // Allow authentication endpoint
                .requestMatchers("/h2-console/**").permitAll() // Allow H2 console for development
                .anyRequest().authenticated() // All other requests require authentication
            )
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // No sessions
            .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class); // Add JWT filter

        // Important for H2 Console: disable frame options for h2-console
        http.headers(headers -> headers.frameOptions(frameOptions -> frameOptions.sameOrigin()));

        return http.build();
    }
}

Controllers (src/main/java/com/example/userservice/controller/)
 * AuthenticationController.java
   package com.example.userservice.controller;

import com.example.common.security.JwtUtil;
import com.example.userservice.model.AuthenticationRequest;
import com.example.userservice.model.AuthenticationResponse;
import com.example.userservice.model.User;
import com.example.userservice.model.UserDetailsResponse;
import com.example.userservice.repository.UserRepository;
import com.example.userservice.service.CustomUserDetailsService;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;

@RestController
@RequestMapping("/api/auth")
public class AuthenticationController {

    private final AuthenticationManager authenticationManager;
    private final CustomUserDetailsService userDetailsService;
    private final JwtUtil jwtUtil;
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public AuthenticationController(AuthenticationManager authenticationManager,
                                    CustomUserDetailsService userDetailsService,
                                    JwtUtil jwtUtil,
                                    UserRepository userRepository,
                                    PasswordEncoder passwordEncoder) {
        this.authenticationManager = authenticationManager;
        this.userDetailsService = userDetailsService;
        this.jwtUtil = jwtUtil;
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    @PostMapping("/authenticate")
    public ResponseEntity<?> createAuthenticationToken(@RequestBody AuthenticationRequest authenticationRequest) throws Exception {
        try {
            Authentication authenticate = authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(authenticationRequest.getUsername(), authenticationRequest.getPassword())
            );
        } catch (BadCredentialsException e) {
            throw new Exception("Incorrect username or password", e);
        }

        final UserDetails userDetails = userDetailsService.loadUserByUsername(authenticationRequest.getUsername());

        // Include roles in JWT claims
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream().map(Object::toString).toList()); // Store roles as a list of strings

        final String jwt = jwtUtil.generateToken(userDetails, claims);

        return ResponseEntity.ok(new AuthenticationResponse(jwt));
    }

    @PostMapping("/register")
    public ResponseEntity<?> registerUser(@RequestBody User user) {
        if (userRepository.findByUsername(user.getUsername()).isPresent()) {
            return ResponseEntity.badRequest().body("Username already exists!");
        }
        user.setPassword(passwordEncoder.encode(user.getPassword()));
        user.setRoles(Set.of("ROLE_USER")); // Default role for new users
        User savedUser = userRepository.save(user);
        return ResponseEntity.ok("User registered successfully with username: " + savedUser.getUsername());
    }

    @GetMapping("/user/{username}")
    // This endpoint can be used by other microservices to fetch user details, or directly by a client.
    // It requires authentication.
    public ResponseEntity<UserDetailsResponse> getUserDetails(@PathVariable String username) {
        return userRepository.findByUsername(username)
                .map(user -> new UserDetailsResponse(user.getId(), user.getUsername(), user.getRoles()))
                .map(ResponseEntity::ok)
                .orElseGet(() -> ResponseEntity.notFound().build());
    }
}

Main Application Class (src/main/java/com/example/userservice/UserServiceApplication.java)
package com.example.userservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan(basePackages = {"com.example.userservice", "com.example.common.security"}) // Scan common package
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

3. Product Service (Port: 8082)
This service will manage products and integrate with the User Service using a Circuit Breaker.
Project Name: product-service
application.properties (src/main/resources/application.properties)
server.port=8082

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.datasource.url=jdbc:h2:mem:productdb;DB_CLOSE_DELAY=-1
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update

# JWT Secret (MUST be the same as User Service for validation)
jwt.secret=mySuperSecretKeyForUserAuthenticationAndAuthorizationInMyMicroservices1234567890abcdefghijklmnopqrstuvwxyz

# Resilience4j Circuit Breaker Configuration
resilience4j.circuitbreaker.instances.userServiceBreaker.registerHealthIndicator=true
resilience4j.circuitbreaker.instances.userServiceBreaker.slidingWindowSize=10
resilience4j.circuitbreaker.instances.userServiceBreaker.failureRateThreshold=50
resilience4j.circuitbreaker.instances.userServiceBreaker.waitDurationInOpenState=5s
resilience4j.circuitbreaker.instances.userServiceBreaker.permittedNumberOfCallsInHalfOpenState=3

# Initial data loading
spring.jpa.defer-datasource-initialization=true

data.sql (src/main/resources/data.sql)
INSERT INTO product (name, description, price) VALUES ('Laptop', 'Powerful laptop for gaming and work', 1200.00);
INSERT INTO product (name, description, price) VALUES ('Smartphone', 'Latest model with great camera', 800.00);
INSERT INTO product (name, description, price) VALUES ('Keyboard', 'Mechanical gaming keyboard', 150.00);

Models (src/main/java/com/example/productservice/model/)
 * Product.java
   package com.example.productservice.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    private double price;
}

 * UserDetailsResponse.java (Copy from User Service model, or create a common shared library)
   package com.example.productservice.model; // Note: package name changed for this service

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Set;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserDetailsResponse {
    private Long id;
    private String username;
    private Set<String> roles;
}

Repository (src/main/java/com/example/productservice/repository/)
 * ProductRepository.java
   package com.example.productservice.repository;

import com.example.productservice.model.Product;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductRepository extends JpaRepository<Product, Long> {
}

Services (src/main/java/com/example/productservice/service/)
 * UserServiceClient.java (To call User Service, with Circuit Breaker)
   package com.example.productservice.service;

import com.example.productservice.model.UserDetailsResponse;
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.server.ResponseStatusException;

import java.util.Collections;
import java.util.Optional;

@Service
public class UserServiceClient {

    private final RestTemplate restTemplate;
    private static final String USER_SERVICE_BASE_URL = "http://localhost:8081/api/auth/user/";

    public UserServiceClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    // Circuit breaker named "userServiceBreaker" as configured in application.properties
    // The fallbackMethod 'getUserDetailsFallback' will be called if the circuit is open or a call fails.
    @CircuitBreaker(name = "userServiceBreaker", fallbackMethod = "getUserDetailsFallback")
    public Optional<UserDetailsResponse> getUserDetails(String username) {
        System.out.println("Attempting to call User Service for user: " + username);
        try {
            // In a real scenario, you might propagate the JWT received from the client
            // to the User Service for secure internal communication.
            // For simplicity here, we assume the user details endpoint is secured
            // with internal access or a separate service-to-service token.
            UserDetailsResponse response = restTemplate.getForObject(USER_SERVICE_BASE_URL + username, UserDetailsResponse.class);
            return Optional.ofNullable(response);
        } catch (Exception e) {
            // Log the actual exception for debugging
            System.err.println("Error calling User Service: " + e.getMessage());
            throw new ResponseStatusException(HttpStatus.INTERNAL_SERVER_ERROR, "Error fetching user details from User Service", e);
        }
    }

    // Fallback method for the circuit breaker
    public Optional<UserDetailsResponse> getUserDetailsFallback(String username, Throwable t) {
        System.err.println("Fallback activated for getUserDetails for user: " + username + ". Reason: " + t.getMessage());
        // Return a default/empty user details or handle as per business logic
        // For example, return a user with only basic info or an empty roles set.
        // In many cases, if user details are critical, this might lead to an error being returned to the client.
        UserDetailsResponse fallbackUser = new UserDetailsResponse();
        fallbackUser.setUsername(username + " (fallback)");
        fallbackUser.setRoles(Collections.singleton("ROLE_FALLBACK")); // Indicate it's a fallback response
        return Optional.of(fallbackUser);
    }
}

Security Configuration (src/main/java/com/example/productservice/config/)
 * JwtAuthEntryPoint.java (Handles unauthorized access attempts)
   package com.example.productservice.config;

import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import java.io.IOException;

// This class handles unauthenticated requests (e.g., no JWT or invalid JWT)
@Component
public class JwtAuthEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized: " + authException.getMessage());
    }
}

 * JwtRequestFilter.java (Similar to User Service, but only validates token and sets security context)
   package com.example.productservice.config;

import com.example.common.security.JwtUtil;
import io.jsonwebtoken.Claims;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;
import java.util.List;
import java.util.stream.Collectors;

@Component
public class JwtRequestFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;

    public JwtRequestFilter(JwtUtil jwtUtil) {
        this.jwtUtil = jwtUtil;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {

        final String authorizationHeader = request.getHeader("Authorization");

        String username = null;
        String jwt = null;

        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {
            jwt = authorizationHeader.substring(7);
            try {
                if (jwtUtil.validateToken(jwt)) { // Validate token validity (signature, expiration)
                    Claims claims = jwtUtil.extractAllClaims(jwt);
                    username = claims.getSubject();

                    // Extract roles from claims (assuming they were put in as a List of strings)
                    List<String> roles = (List<String>) claims.get("roles");
                    List<SimpleGrantedAuthority> authorities = roles.stream()
                            .map(SimpleGrantedAuthority::new)
                            .collect(Collectors.toList());

                    if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                        UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken =
                                new UsernamePasswordAuthenticationToken(username, null, authorities); // No password needed for authenticated JWT
                        SecurityContextHolder.getContext().setAuthentication(usernamePasswordAuthenticationToken);
                    }
                }
            } catch (Exception e) {
                System.err.println("JWT processing error: " + e.getMessage());
                // Optionally, clear security context if token is bad
                SecurityContextHolder.clearContext();
            }
        }
        chain.doFilter(request, response);
    }
}

 * SecurityConfig.java
   package com.example.productservice.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.client.RestTemplate;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class SecurityConfig {

    private final JwtRequestFilter jwtRequestFilter;
    private final JwtAuthEntryPoint jwtAuthEntryPoint;

    public SecurityConfig(JwtRequestFilter jwtRequestFilter, JwtAuthEntryPoint jwtAuthEntryPoint) {
        this.jwtRequestFilter = jwtRequestFilter;
        this.jwtAuthEntryPoint = jwtAuthEntryPoint;
    }

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable) // Disable CSRF for stateless REST API
            .exceptionHandling(exception -> exception.authenticationEntryPoint(jwtAuthEntryPoint)) // Handle unauthorized
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/h2-console/**").permitAll() // Allow H2 console for development
                .requestMatchers("/api/products/public/**").permitAll() // Example public endpoint
                .anyRequest().authenticated() // All other requests require authentication
            )
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // No sessions
            .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class); // Add JWT filter

        // Important for H2 Console: disable frame options for h2-console
        http.headers(headers -> headers.frameOptions(frameOptions -> frameOptions.sameOrigin()));

        return http.build();
    }
}

Controllers (src/main/java/com/example/productservice/controller/)
 * ProductController.java
   package com.example.productservice.controller;

import com.example.productservice.model.Product;
import com.example.productservice.model.UserDetailsResponse;
import com.example.productservice.repository.ProductRepository;
import com.example.productservice.service.UserServiceClient;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductRepository productRepository;
    private final UserServiceClient userServiceClient;

    public ProductController(ProductRepository productRepository, UserServiceClient userServiceClient) {
        this.productRepository = productRepository;
        this.userServiceClient = userServiceClient;
    }

    @GetMapping("/public/all")
    public List<Product> getAllProductsPublic() {
        return productRepository.findAll();
    }

    @GetMapping
    @PreAuthorize("hasAnyRole('USER', 'ADMIN')") // Only authenticated users (USER or ADMIN) can access
    public List<Product> getAllProducts(Authentication authentication) {
        System.out.println("Authenticated user for /api/products: " + authentication.getName());
        // Example of calling user service with circuit breaker
        Optional<UserDetailsResponse> userDetails = userServiceClient.getUserDetails(authentication.getName());
        userDetails.ifPresent(u -> System.out.println("User details from User Service (via CB): " + u.getUsername() + ", Roles: " + u.getRoles()));
        return productRepository.findAll();
    }

    @GetMapping("/{id}")
    @PreAuthorize("hasAnyRole('USER', 'ADMIN')")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        return productRepository.findById(id)
                .map(ResponseEntity::ok)
                .orElseGet(() -> ResponseEntity.notFound().build());
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')") // Only ADMIN can create products
    public Product createProduct(@RequestBody Product product) {
        return productRepository.save(product);
    }

    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')") // Only ADMIN can update products
    public ResponseEntity<Product> updateProduct(@PathVariable Long id, @RequestBody Product productDetails) {
        return productRepository.findById(id)
                .map(product -> {
                    product.setName(productDetails.getName());
                    product.setDescription(productDetails.getDescription());
                    product.setPrice(productDetails.getPrice());
                    return ResponseEntity.ok(productRepository.save(product));
                })
                .orElseGet(() -> ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')") // Only ADMIN can delete products
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        return productRepository.findById(id)
                .map(product -> {
                    productRepository.delete(product);
                    return ResponseEntity.ok().<Void>build();
                })
                .orElseGet(() -> ResponseEntity.notFound().build());
    }
}

Main Application Class (src/main/java/com/example/productservice/ProductServiceApplication.java)
package com.example.productservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan(basePackages = {"com.example.productservice", "com.example.common.security"}) // Scan common package
public class ProductServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}

Important Security Considerations for Production
 * JWT Secret: NEVER hardcode the JWT secret key. It must be a long, randomly generated string loaded from environment variables, HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, or similar secure key management systems.
 * HTTPS/TLS: All communication between clients and services, and ideally between services themselves, should be encrypted using HTTPS (TLS/SSL). This prevents eavesdropping and tampering. In Spring Boot, you can configure SSL in application.properties.
 * Cross-Origin Resource Sharing (CORS): If your frontend application is served from a different domain/port than your backend microservices, you'll need to configure CORS properly to allow browser requests.
 * Error Handling: Implement robust error handling (e.g., using @ControllerAdvice) to return meaningful error messages without exposing sensitive information.
 * Logging & Monitoring: Implement comprehensive logging (e.g., using Logback, Log4j2) and monitoring (e.g., Prometheus, Grafana, ELK stack) to track security events, authentication failures, and system health.
 * Rate Limiting: Implement rate limiting on authentication endpoints to prevent brute-force attacks.
 * Input Validation: Always validate all incoming user input on the server side to prevent injection attacks (SQL Injection, XSS, etc.). Spring validation annotations (@Valid, @NotNull, etc.) are useful.
 * Least Privilege: Ensure that each user and service only has the minimum necessary permissions to perform its functions.
 * Auditing: Log critical security events (login attempts, permission changes, data access) for auditing and forensic analysis.
 * Regular Security Audits: Conduct regular security audits and penetration testing of your applications.
 * Dependency Updates: Keep all your dependencies (Spring Boot, Spring Security, JWT libraries) up-to-date to patch known vulnerabilities.
 * Refresh Tokens: For long-lived sessions, combine short-lived JWT access tokens with longer-lived refresh tokens. The client uses the refresh token to obtain a new access token without re-authenticating with username/password every time.
How to Run and Test:
 * Save Files: Create two separate Spring Boot projects (e.g., user-service and product-service) and copy the respective code into them. Ensure the JwtUtil.java file is present in src/main/java/com/example/common/security within both projects.
 * Build: Open a terminal in each project's root directory and run mvn clean install.
 * Run Services:
   * For User Service: java -jar target/user-service-0.0.1-SNAPSHOT.jar
   * For Product Service: java -jar target/product-service-0.0.1-SNAPSHOT.jar
   * Ensure both are running on their configured ports (8081 and 8082).
 * Access H2 Consoles (for development/testing):
   * User Service: http://localhost:8081/h2-console (JDBC URL: jdbc:h2:mem:userdb)
   * Product Service: http://localhost:8082/h2-console (JDBC URL: jdbc:h2:mem:productdb)
 * Test with Postman/Insomnia:
   A. User Service - Registration:
   * Method: POST
   * URL: http://localhost:8081/api/auth/register
   * Body (raw, JSON):
     {
    "username": "testuser",
    "password": "testpassword",
    "roles": ["ROLE_USER"]
}

     (You can try registering 'user' and 'admin' as well if data.sql is not working/you want to re-register)
   B. User Service - Authentication (Get JWT):
   * Method: POST
   * URL: http://localhost:8081/api/auth/authenticate
   * Body (raw, JSON):
     {
    "username": "user",
    "password": "password"
}

     Or for admin:
     {
    "username": "admin",
    "password": "adminpass"
}

   * Response: You'll get a JWT in the response. Copy this token.
   C. Product Service - Public Endpoint:
   * Method: GET
   * URL: http://localhost:8082/api/products/public/all
   * Result: Should work without authentication.
   D. Product Service - Authenticated Endpoint (as USER):
   * Method: GET
   * URL: http://localhost:8082/api/products
   * Headers:
     * Authorization: Bearer <YOUR_JWT_TOKEN_FROM_USER_LOGIN> (use the token for 'user')
   * Result: Should return all products.
   E. Product Service - ADMIN Only Endpoint (try as USER, then as ADMIN):
   * Method: POST
   * URL: http://localhost:8082/api/products
   * Headers:
     * Authorization: Bearer <YOUR_JWT_TOKEN> (first try 'user' token, then 'admin' token)
     * Content-Type: application/json
   * Body (raw, JSON):
     {
    "name": "New Product",
    "description": "A shiny new gadget",
    "price": 99.99
}

   * Result:
     * As user: 403 Forbidden
     * As admin: 200 OK and the new product details.
   F. Test Circuit Breaker (Product Service calling User Service):
   * The GET /api/products endpoint in Product Service calls the User Service.
   * To simulate failure: Stop the User Service (Ctrl+C in its terminal).
   * Make multiple requests to http://localhost:8082/api/products (with a valid JWT from the Product Service).
   * Observe:
     * The first few requests will fail with 500 Internal Server Error (because User Service is down). You'll see "Error calling User Service" messages in Product Service console.
     * After a few failures (configured by slidingWindowSize and failureRateThreshold), the Circuit Breaker will open.
     * Subsequent requests will immediately trigger the getUserDetailsFallback method, and you'll see "Fallback activated..." in the Product Service console, likely returning a 200 OK response but with fallback user data.
     * Wait for waitDurationInOpenState (5 seconds). The circuit will move to HALF-OPEN state.
     * Make a few more requests. If the User Service is still down, the circuit will immediately go back to OPEN. If you restart the User Service now, these "test" calls will succeed, and the circuit will close.
This setup provides a solid foundation for secure microservices with essential resilience patterns.
