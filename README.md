I understand! You want the same microservices architecture (Auth Service, Product Service, API Gateway with JWT and two user roles) but implemented using Java Spring Boot.
This is an even more substantial request than the Node.js version, as Spring Boot projects involve more files, configuration, and a more opinionated structure. Providing "full code without omitting anything" would still be impractical for a single response.
However, I will provide a structured and detailed outline with key code snippets and explanations for each component, focusing on the core aspects:
 * Authentication Microservice (Auth Service)
 * Product Microservice (Product Service)
 * API Gateway (Spring Cloud Gateway)
For each service, I'll provide:
 * Project setup (dependencies).
 * Core application files.
 * JWT generation and validation logic.
 * Security configurations.
 * Controllers for endpoints.
 * Entity and Repository (for users).
Key Spring Boot Technologies We'll Use:
 * Spring Boot: Framework for building standalone, production-grade Spring applications.
 * Spring Security: For authentication and authorization.
 * JWT (jjwt library): For handling JSON Web Tokens.
 * Spring Data JPA / H2 Database: For user storage (in-memory for simplicity, easily switched to a persistent DB).
 * Spring Cloud Gateway: For the API Gateway.
 * Lombok: To reduce boilerplate code (optional but highly recommended).
Project Structure Overview:
parent-project/
├── auth-service/
│   ├── src/main/java/com/example/authservice/...
│   ├── src/main/resources/application.properties
│   └── pom.xml
├── product-service/
│   ├── src/main/java/com/example/productservice/...
│   ├── src/main/resources/application.properties
│   └── pom.xml
├── api-gateway/
│   ├── src/main/java/com/example/apigateway/...
│   ├── src/main/resources/application.properties
│   └── pom.xml
└── pom.xml (Parent POM - optional but good for managing dependencies)

Before you start:
 * Java Development Kit (JDK): Ensure you have JDK 17 or higher installed.
 * Maven: Ensure Maven is installed and configured.
 * IDE: IntelliJ IDEA or VS Code with Spring Boot extensions are highly recommended.
I. Authentication Microservice (Auth Service)
This service will handle user registration and login, issuing JWTs.
1. auth-service/pom.xml (Dependencies):
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.7</version> <relativePath/> </parent>
    <groupId>com.example</groupId>
    <artifactId>auth-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>auth-service</name>
    <description>Authentication Microservice</description>

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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
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

2. auth-service/src/main/resources/application.properties:
server.port=8081
spring.datasource.url=jdbc:h2:mem:authdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.jpa.hibernate.ddl-auto=update

# JWT Secret Key - MUST BE STRONG AND THE SAME ACROSS SERVICES
app.jwt.secret=yourVeryLongAndSecureSecretKeyForJWTAuthServices1234567890
app.jwt.expiration-ms=3600000 # 1 hour

IMPORTANT: Change app.jwt.secret to a much longer and more complex string in a real application. This key must be identical in the product-service as well.
3. Java Files (com.example.authservice package):
 * AuthServiceApplication.java:
   package com.example.authservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@SpringBootApplication
public class AuthServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(AuthServiceApplication.class, args);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

 * user/User.java (JPA Entity):
   package com.example.authservice.user;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.Set;
import java.util.stream.Collectors;
import java.util.Arrays;

@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User implements UserDetails {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password; // Stored as BCrypt hash
    private String roles; // Comma-separated roles, e.g., "ROLE_USER,ROLE_ADMIN"

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Arrays.stream(roles.split(","))
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());
    }

    @Override
    public boolean isAccountNonExpired() { return true; }

    @Override
    public boolean isAccountNonLocked() { return true; }

    @Override
    public boolean isCredentialsNonExpired() { return true; }

    @Override
    public boolean isEnabled() { return true; }
}

 * user/UserRepository.java:
   package com.example.authservice.user;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    boolean existsByUsername(String username);
}

 * user/UserDetailsServiceImpl.java:
   package com.example.authservice.user;

import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import lombok.RequiredArgsConstructor;

@Service
@RequiredArgsConstructor
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
    }
}

 * jwt/JwtUtil.java (JWT Utility Class):
   package com.example.authservice.jwt;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;
import com.example.authservice.user.User; // Import your User entity

@Component
public class JwtUtil {

    @Value("${app.jwt.secret}")
    private String secret;

    @Value("${app.jwt.expiration-ms}")
    private long expirationMs;

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
        return Jwts
                .parserBuilder()
                .setSigningKey(getSignKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }

    public String generateToken(User user) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", user.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(","))); // Store roles as comma-separated string
        return createToken(claims, user.getUsername());
    }

    private String createToken(Map<String, Object> claims, String userName) {
        return Jwts.builder()
                .setClaims(claims)
                .setSubject(userName)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + expirationMs))
                .signWith(getSignKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    private Key getSignKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}

 * auth/AuthController.java:
   package com.example.authservice.auth;

import com.example.authservice.jwt.JwtUtil;
import com.example.authservice.user.User;
import com.example.authservice.user.UserRepository;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtUtil jwtUtil;
    private final AuthenticationManager authenticationManager;

    @PostMapping("/register")
    public ResponseEntity<?> registerUser(@RequestBody RegisterRequest registerRequest) {
        if (userRepository.existsByUsername(registerRequest.getUsername())) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body("Username is already taken!");
        }

        User user = new User();
        user.setUsername(registerRequest.getUsername());
        user.setPassword(passwordEncoder.encode(registerRequest.getPassword()));
        user.setRoles(registerRequest.getRoles()); // e.g., "ROLE_USER", "ROLE_ADMIN,ROLE_USER"

        userRepository.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body("User registered successfully!");
    }

    @PostMapping("/login")
    public ResponseEntity<?> authenticateUser(@RequestBody LoginRequest loginRequest) {
        try {
            Authentication authentication = authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword())
            );

            // If authentication is successful, generate token
            User userDetails = (User) authentication.getPrincipal(); // Cast to your User entity
            String token = jwtUtil.generateToken(userDetails);
            return ResponseEntity.ok(new JwtResponse(token));

        } catch (AuthenticationException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid Username or Password!");
        }
    }
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class RegisterRequest {
    private String username;
    private String password;
    private String roles;
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class LoginRequest {
    private String username;
    private String password;
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class JwtResponse {
    private String token;
}

 * config/SecurityConfig.java:
   package com.example.authservice.config;

import com.example.authservice.user.UserDetailsServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import lombok.RequiredArgsConstructor;

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final UserDetailsServiceImpl userDetailsService;
    private final PasswordEncoder passwordEncoder;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // Disable CSRF for stateless API
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/auth/register", "/api/auth/login", "/h2-console/**").permitAll() // Allow public access
                .anyRequest().authenticated() // All other requests require authentication
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // Use stateless sessions for JWT
            );
        // Required for H2 Console to work with Spring Security
        http.headers(headers -> headers.frameOptions().disable());

        return http.build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailsService);
        authenticationProvider.setPasswordEncoder(passwordEncoder);
        return authenticationProvider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}

II. Product Microservice (Product Service)
This service will host protected API endpoints that require JWT authentication and role-based authorization.
1. product-service/pom.xml (Dependencies):
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.7</version> <relativePath/> </parent>
    <groupId>com.example</groupId>
    <artifactId>product-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>product-service</name>
    <description>Product Microservice</description>

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
                            <artifactId>lombok</maligngroup>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

2. product-service/src/main/resources/application.properties:
server.port=8082
app.jwt.secret=yourVeryLongAndSecureSecretKeyForJWTAuthServices1234567890

IMPORTANT: app.jwt.secret must be identical to the one in auth-service.
3. Java Files (com.example.productservice package):
 * ProductServiceApplication.java:
   package com.example.productservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProductServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}

 * product/Product.java (Simple Data Class):
   package com.example.productservice.product;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product {
    private Long id;
    private String name;
    private double price;
}

 * jwt/JwtAuthFilter.java (JWT Filter for incoming requests):
   package com.example.productservice.jwt;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;
import org.springframework.beans.factory.annotation.Value;

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
                logger.error("Error parsing JWT: " + e.getMessage());
                // Optionally, you can send an error response here
            }
        }

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            // In a real microservice, you might call auth-service to validate the token
            // For simplicity, we'll validate it locally using the same secret
            if (!jwtUtil.isTokenExpired(token)) {
                List<SimpleGrantedAuthority> authorities = Arrays.stream(roles.split(","))
                        .map(SimpleGrantedAuthority::new)
                        .collect(Collectors.toList());

                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                        username, null, authorities); // No password needed for authentication based on token
                SecurityContextHolder.getContext().setAuthentication(authToken);
            } else {
                logger.warn("Expired JWT token received: " + token);
            }
        }
        filterChain.doFilter(request, response);
    }
}

 * jwt/JwtUtil.java (Similar to Auth Service, but adapted for validation):
   package com.example.productservice.jwt;

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

    // We don't generate token here, only validate.
    // For simplicity, this service assumes token is already generated by auth-service
    // and only validates its signature and expiration.
    // If more complex validation (e.g., checking against a revoked token list) is needed,
    // this service would ideally communicate with the auth service.

    private Key getSignKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}

 * config/SecurityConfig.java:
   package com.example.productservice.config;

import com.example.productservice.jwt.JwtAuthFilter;
import com.example.productservice.jwt.JwtUtil;
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
                .requestMatchers("/api/products/public/**").permitAll() // Public products
                .anyRequest().authenticated() // All other requests require authentication
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // Use stateless sessions for JWT
            )
            .addFilterBefore(jwtAuthFilter(), UsernamePasswordAuthenticationFilter.class); // Add JWT filter

        return http.build();
    }
}

 * product/ProductController.java:
   package com.example.productservice.product;

import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    // In-memory list for demonstration
    private final List<Product> products = new ArrayList<>();
    private Long nextId = 1L;

    public ProductController() {
        products.add(new Product(nextId++, "Laptop", 1200.00));
        products.add(new Product(nextId++, "Mouse", 25.00));
        products.add(new Product(nextId++, "Keyboard", 75.00));
    }

    // Public endpoint
    @GetMapping("/public")
    public ResponseEntity<List<Product>> getPublicProducts() {
        return ResponseEntity.ok(products);
    }

    // Protected endpoint - any authenticated user
    @GetMapping
    @PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')") // Role-based access
    public ResponseEntity<List<Product>> getAllProducts() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        System.out.println("User accessing protected products: " + authentication.getName());
        System.out.println("Authorities: " + authentication.getAuthorities());
        return ResponseEntity.ok(products);
    }

    // Admin-only endpoint
    @PostMapping
    @PreAuthorize("hasRole('ROLE_ADMIN')") // Admin-only access
    public ResponseEntity<Product> addProduct(@RequestBody Product product) {
        product.setId(nextId++);
        products.add(product);
        return ResponseEntity.status(201).body(product);
    }

    // Example: User-specific information (demonstrates accessing authenticated user details)
    @GetMapping("/my-info")
    @PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')")
    public ResponseEntity<?> getMyInfo() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        return ResponseEntity.ok("Hello, " + authentication.getName() + "! Your roles: " + authentication.getAuthorities());
    }
}

III. API Gateway (Spring Cloud Gateway)
This service will act as the single entry point and route requests to the appropriate microservices.
1. api-gateway/pom.xml (Dependencies):
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.7</version>
        <relativePath/> </parent>
    <groupId>com.example</groupId>
    <artifactId>api-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>api-gateway</name>
    <description>API Gateway using Spring Cloud Gateway</description>

    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.2</spring-cloud.version> </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId> </dependency>
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

2. api-gateway/src/main/resources/application.properties:
server.port=8080

spring.cloud.gateway.routes[0].id=auth-service
spring.cloud.gateway.routes[0].uri=http://localhost:8081
spring.cloud.gateway.routes[0].predicates[0]=Path=/api/auth/**

spring.cloud.gateway.routes[1].id=product-service
spring.cloud.gateway.routes[1].uri=http://localhost:8082
spring.cloud.gateway.routes[1].predicates[0]=Path=/api/products/**

# Enable CORS for the Gateway (Important for frontend applications)
spring.webflux.cors.allow-credentials=true
spring.webflux.cors.allowed-origins=*
spring.webflux.cors.allowed-methods=GET,POST,PUT,DELETE,OPTIONS
spring.webflux.cors.allowed-headers=*
spring.webflux.cors.max-age=3600

3. Java File (com.example.apigateway package):
 * ApiGatewayApplication.java:
   package com.example.apigateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.reactive.CorsWebFilter;
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;

import java.util.Arrays;
import java.util.Collections;

@SpringBootApplication
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }

    // No need for explicit routes if configured in application.properties
    // @Bean
    // public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    //     return builder.routes()
    //             .route("auth_route", r -> r.path("/api/auth/**")
    //                     .uri("http://localhost:8081"))
    //             .route("product_route", r -> r.path("/api/products/**")
    //                     .uri("http://localhost:8082"))
    //             .build();
    // }

    // Configure CORS for Spring Cloud Gateway
    @Bean
    public CorsWebFilter corsWebFilter() {
        CorsConfiguration corsConfig = new CorsConfiguration();
        corsConfig.setAllowedOrigins(Collections.singletonList("*")); // Allow all origins (for dev)
        corsConfig.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        corsConfig.setAllowedHeaders(Collections.singletonList("*")); // Allow all headers
        corsConfig.setAllowCredentials(true);
        corsConfig.setMaxAge(3600L); // How long the CORS pre-flight request can be cached

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", corsConfig); // Apply CORS to all paths
        return new CorsWebFilter(source);
    }
}

How to Run and Test (Similar to Node.js):
 * Project Setup: Create three separate Maven projects (or a multi-module Maven project) named auth-service, product-service, and api-gateway.
 * Copy Files: Copy the pom.xml, application.properties, and Java source files into their respective projects. Ensure the package names match (com.example.authservice, com.example.productservice, com.example.apigateway).
 * Build Projects: Open a terminal in each project's root directory and run:
   mvn clean install

 * Run Applications (in separate terminal windows/tabs):
   # Terminal 1: Auth Service (Port 8081)
cd auth-service
mvn spring-boot:run

# Terminal 2: Product Service (Port 8082)
cd product-service
mvn spring-boot:run

# Terminal 3: API Gateway (Port 8080)
cd api-gateway
mvn spring-boot:run

Testing with curl or Postman/Insomnia:
All requests will now go through the API Gateway (port 8080).
1. Register Users:
 * Admin User:
   curl -X POST -H "Content-Type: application/json" -d '{"username": "admin", "password": "adminpassword", "roles": "ROLE_ADMIN,ROLE_USER"}' http://localhost:8080/api/auth/register

   * Expected Response: User registered successfully!
 * Regular User:
   curl -X POST -H "Content-Type: application/json" -d '{"username": "user1", "password": "user1password", "roles": "ROLE_USER"}' http://localhost:8080/api/auth/register

   * Expected Response: User registered successfully!
2. Login as Admin and get Token:
curl -X POST -H "Content-Type: application/json" -d '{"username": "admin", "password": "adminpassword"}' http://localhost:8080/api/auth/login

 * Expected Response: JSON with a token. Copy this token.
3. Login as User1 and get Token:
curl -X POST -H "Content-Type: application/json" -d '{"username": "user1", "password": "user1password"}' http://localhost:8080/api/auth/login

 * Expected Response: JSON with a token. Copy this token.
4. Access Public Product Endpoint (No Token Required):
curl http://localhost:8080/api/products/public

 * Expected Response: List of products.
5. Access Protected Product Endpoint (with Admin Token):
# Replace <ADMIN_TOKEN> with the token you got from admin login
curl -H "Authorization: Bearer <ADMIN_TOKEN>" http://localhost:8080/api/products

 * Expected Response: List of products. You'll also see debug output in the product service console showing the authenticated user.
6. Access Protected Product Endpoint (with User1 Token):
# Replace <USER1_TOKEN> with the token you got from user1 login
curl -H "Authorization: Bearer <USER1_TOKEN>" http://localhost:8080/api/products

 * Expected Response: List of products.
7. Add Product (Admin Only) - with Admin Token:
# Replace <ADMIN_TOKEN>
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer <ADMIN_TOKEN>" -d '{"name": "Webcam", "price": 50.00}' http://localhost:8080/api/products

 * Expected Response: JSON of the newly added product with status 201.
8. Add Product (Admin Only) - with User1 Token (Should Fail):
# Replace <USER1_TOKEN>
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer <USER1_TOKEN>" -d '{"name": "Headphones", "price": 100.00}' http://localhost:8080/api/products

 * Expected Response: Status 403 Forbidden with Spring Security's default error page (or a JSON error if you customize error handling).
9. Access User-Specific Endpoint (with User1 Token):
# Replace <USER1_TOKEN>
curl -H "Authorization: Bearer <USER1_TOKEN>" http://localhost:8080/api/products/my-info

 * Expected Response: User-specific info for user1.
Important Considerations for Production:
 * Database: Replace H2 (in-memory) with a persistent database like PostgreSQL or MySQL for auth-service.
 * JWT Secret: Generate a very strong, long, random secret key for app.jwt.secret and manage it securely (e.g., using Spring Cloud Config Server, HashiCorp Vault, Kubernetes Secrets).
 * Error Handling: Implement global exception handling in both microservices for user-friendly error responses.
 * Logging: Use SLF4J and Logback for structured logging.
 * Monitoring & Tracing: Integrate with tools like Micrometer/Prometheus/Grafana for monitoring, and Spring Cloud Sleuth/Zipkin for distributed tracing.
 * Service Discovery: For a more robust microservices setup, use a service discovery mechanism (e.g., Eureka, Consul) so services don't need to hardcode each other's URLs. Spring Cloud Gateway integrates well with this.
 * Deployment: Containerize (Docker) each service and deploy using Kubernetes or a similar orchestrator.
 * Load Balancing: Consider adding a load balancer in front of your API Gateway and potentially between services if you scale horizontally.
 * HTTPS: Always use HTTPS in production.
 * Input Validation: Use Bean Validation (@Valid) in DTOs.
This comprehensive Spring Boot implementation provides a solid foundation for your microservices architecture. Remember to build upon it with best practices for a robust and secure production application.
