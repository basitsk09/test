You're asking an excellent question that gets to the heart of microservices design and best practices for shared code.
Can every microservice use one JwtUtil and one JwtAuthFilter?
Short Answer: Yes, conceptually, but not directly by sharing a single compiled JwtUtil.java or JwtAuthFilter.java file across separate Spring Boot applications.
The correct approach is to create a shared library (a separate Maven module/JAR) that contains these common security components (JwtUtil, JwtAuthFilter, SecurityConfig setup), and then each microservice that needs JWT validation can depend on and include this library.
Why a Shared Library?
 * DRY (Don't Repeat Yourself): Avoids copying and pasting the exact same code into every microservice, reducing redundancy.
 * Consistency: Ensures that all microservices use the exact same JWT validation logic. If you fix a bug or enhance security in the shared library, all dependent microservices benefit by simply updating the library version.
 * Maintainability: Easier to manage and update common code in one place.
 * Version Control: You can version your shared security library independently.
Implementation Steps (Spring Boot Multi-Module Project):
We'll create a parent Maven project and three child modules:
 * common-security (the shared library)
 * auth-service
 * product-service (will depend on common-security)
 * order-service (will depend on common-security)
 * api-gateway
I. Parent pom.xml (Root of the Project)
Create a directory (e.g., microservices-jwt-shared) and put this pom.xml inside it.
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging> <groupId>com.example</groupId>
    <artifactId>microservices-jwt-shared</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <name>Microservices with Shared JWT Security</name>
    <description>A multi-module project demonstrating shared JWT security components.</description>

    <properties>
        <java.version>17</java.version>
        <spring-boot.version>3.2.7</spring-boot.version>
        <spring-cloud.version>2023.0.2</spring-cloud.version>
        <jjwt.version>0.12.5</jjwt.version>
        <lombok.version>1.18.30</lombok.version>
    </properties>

    <modules>
        <module>common-security</module>
        <module>auth-service</module>
        <module>product-service</module>
        <module>order-service</module>
        <module>api-gateway</module>
    </modules>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
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
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <version>${spring-boot.version}</version>
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
        </pluginManagement>
    </build>

</project>

II. common-security Module (The Shared Library)
Create a subdirectory common-security under your parent project.
1. common-security/pom.xml:
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-jwt-shared</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>common-security</artifactId>
    <name>Common Security Library</name>
    <description>Shared JWT utilities and filters for microservices</description>
    <packaging>jar</packaging> <dependencies>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

</project>

2. common-security/src/main/java/com/example/common/security/jwt/JwtUtil.java:
package com.example.common.security.jwt;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

@Component // Make it a Spring component so it can be injected
public class JwtUtil {

    // These values will be picked up from the consuming service's application.properties
    @Value("${app.jwt.secret}")
    private String secret;

    @Value("${app.jwt.expiration-ms:3600000}") // Default to 1 hour if not specified
    private long expirationMs;

    // --- Token Generation (typically for Auth Service) ---
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        // Add roles to claims
        claims.put("roles", userDetails.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(",")));
        return createToken(claims, userDetails.getUsername());
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

    // --- Token Validation and Extraction (for Protected Services) ---
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    public Claims extractAllClaims(String token) {
        return Jwts
                .parserBuilder()
                .setSigningKey(getSignKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    public Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    // This method is for services that need to do a full userDetails match (Auth Service)
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }

    private Key getSignKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}

3. common-security/src/main/java/com/example/common/security/jwt/JwtAuthFilter.java:
package com.example.common.security.jwt;

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
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.MalformedJwtException;
import io.jsonwebtoken.SignatureException;
import io.jsonwebtoken.UnsupportedJwtException;

@RequiredArgsConstructor // For injecting JwtUtil
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
            } catch (ExpiredJwtException e) {
                logger.warn("JWT Token has expired: {}", e.getMessage());
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // 401 Unauthorized
                response.getWriter().write("JWT Token has expired");
                return; // Stop further processing
            } catch (SignatureException e) {
                logger.error("Invalid JWT Signature: {}", e.getMessage());
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // 401 Unauthorized
                response.getWriter().write("Invalid JWT Signature");
                return;
            } catch (MalformedJwtException e) {
                logger.error("Invalid JWT Token: {}", e.getMessage());
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // 401 Unauthorized
                response.getWriter().write("Invalid JWT Token");
                return;
            } catch (UnsupportedJwtException e) {
                logger.error("JWT Token is unsupported: {}", e.getMessage());
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // 401 Unauthorized
                response.getWriter().write("JWT Token is unsupported");
                return;
            } catch (IllegalArgumentException e) {
                logger.error("JWT claims string is empty: {}", e.getMessage());
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // 401 Unauthorized
                response.getWriter().write("JWT claims string is empty");
                return;
            } catch (Exception e) {
                logger.error("Error parsing or validating JWT: {}", e.getMessage(), e);
                response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR); // 500 Internal Server Error
                response.getWriter().write("Error processing JWT");
                return;
            }
        }

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            // Assuming the token is valid at this point as exceptions would have been caught
            // Re-validate expiration just in case, though ExpiredJwtException should catch it
            if (!jwtUtil.isTokenExpired(token)) {
                List<SimpleGrantedAuthority> authorities = Arrays.stream(roles.split(","))
                        .map(SimpleGrantedAuthority::new)
                        .collect(Collectors.toList());

                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                        username, null, authorities);
                SecurityContextHolder.getContext().setAuthentication(authToken);
            } else {
                logger.warn("JWT Token has expired during re-check: " + token);
                // This case should ideally be handled by ExpiredJwtException above,
                // but kept as a safeguard.
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // 401 Unauthorized
                response.getWriter().write("JWT Token expired during re-check");
                return;
            }
        }
        filterChain.doFilter(request, response);
    }
}

4. common-security/src/main/java/com/example/common/security/config/JwtSecurityConfig.java:
This class encapsulates the common Spring Security configuration for services that need JWT authentication.
package com.example.common.security.config;

import com.example.common.security.jwt.JwtAuthFilter;
import com.example.common.security.jwt.JwtUtil;
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
@EnableWebSecurity // Enable Spring Security's web security support
@EnableMethodSecurity(prePostEnabled = true) // Enables @PreAuthorize, @PostAuthorize etc.
@RequiredArgsConstructor
public class JwtSecurityConfig {

    private final JwtUtil jwtUtil;

    @Bean
    public JwtAuthFilter jwtAuthFilter() {
        return new JwtAuthFilter(jwtUtil);
    }

    // This Bean will be created in any service that imports common-security and enables
    // this configuration (e.g., via @Import or component scanning)
    @Bean
    public SecurityFilterChain jwtFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // Disable CSRF for stateless API
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // Use stateless sessions for JWT
            )
            // Add your custom JWT filter before Spring's default UsernamePasswordAuthenticationFilter
            .addFilterBefore(jwtAuthFilter(), UsernamePasswordAuthenticationFilter.class);

        // NOTE: Specific authorization rules (.authorizeHttpRequests) are NOT defined here.
        // Each consuming microservice will define its own specific path authorizations
        // in its own SecurityConfig. This base config only sets up the filter chain.

        return http.build();
    }
}

III. auth-service Module
Create auth-service subdirectory.
(This service will still generate tokens, but it won't use the shared JwtAuthFilter for its own incoming requests, as its endpoints are public for login/register)
1. auth-service/pom.xml:
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>microservices-jwt-shared</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <artifactId>auth-service</artifactId>
    <name>auth-service</name>
    <description>Authentication Microservice</description>

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
    </dependencies>

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

3. auth-service/src/main/java/com/example/authservice/AuthServiceApplication.java:
package com.example.authservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan; // Important for common-security components
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@SpringBootApplication
@ComponentScan(basePackages = {"com.example.authservice", "com.example.common.security"}) // Scan common-security
public class AuthServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(AuthServiceApplication.class, args);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

4. auth-service/src/main/java/com/example/authservice/user/User.java: (No change from before, keeping it in auth-service as it's specific to user management here)
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

5. auth-service/src/main/java/com/example/authservice/user/UserRepository.java: (No change)
package com.example.authservice.user;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    boolean existsByUsername(String username);
}

6. auth-service/src/main/java/com/example/authservice/user/UserDetailsServiceImpl.java: (No change)
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

7. auth-service/src/main/java/com/example/authservice/auth/AuthController.java: (Slight change to use common-security.jwt.JwtUtil)
package com.example.authservice.auth;

import com.example.common.security.jwt.JwtUtil; // NOW FROM COMMON LIBRARY
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
    private final JwtUtil jwtUtil; // Injected from common-security
    private final AuthenticationManager authenticationManager;

    @PostMapping("/register")
    public ResponseEntity<?> registerUser(@RequestBody RegisterRequest registerRequest) {
        if (userRepository.existsByUsername(registerRequest.getUsername())) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body("Username is already taken!");
        }

        User user = new User();
        user.setUsername(registerRequest.getUsername());
        user.setPassword(passwordEncoder.encode(registerRequest.getPassword()));
        user.setRoles(registerRequest.getRoles());

        userRepository.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body("User registered successfully!");
    }

    @PostMapping("/login")
    public ResponseEntity<?> authenticateUser(@RequestBody LoginRequest loginRequest) {
        try {
            Authentication authentication = authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword())
            );

            User userDetails = (User) authentication.getPrincipal();
            String token = jwtUtil.generateToken(userDetails); // Use generateToken from shared JwtUtil
            return ResponseEntity.ok(new JwtResponse(token));

        } catch (AuthenticationException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid Username or Password!");
        }
    }
}

@Data @NoArgsConstructor @AllArgsConstructor class RegisterRequest { private String username; private String password; private String roles; }
@Data @NoArgsConstructor @AllArgsConstructor class LoginRequest { private String username; private String password; }
@Data @NoArgsConstructor @AllArgsConstructor class JwtResponse { private String token; }

8. auth-service/src/main/java/com/example/authservice/config/SecurityConfig.java: (No change from before, since this service doesn't need to validate incoming JWTs)
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
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/auth/register", "/api/auth/login", "/h2-console/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );
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

IV. product-service Module
Create product-service subdirectory.
1. product-service/pom.xml:
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
    </dependencies>

</project>

2. product-service/src/main/resources/application.properties:
server.port=8082
app.jwt.secret=yourVeryLongAndSecureSecretKeyForJWTAuthServices1234567890 # MUST BE THE SAME

3. product-service/src/main/java/com/example/productservice/ProductServiceApplication.java:
package com.example.productservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan; // Important for common-security components
import org.springframework.context.annotation.Import; // To explicitly import JwtSecurityConfig
import com.example.common.security.config.JwtSecurityConfig; // Import the shared config

@SpringBootApplication
@ComponentScan(basePackages = {"com.example.productservice", "com.example.common.security"}) // Scan common-security
@Import(JwtSecurityConfig.class) // Explicitly import the shared security config
public class ProductServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}

4. product-service/src/main/java/com/example/productservice/product/Product.java: (No change)
package com.example.productservice.product;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data @NoArgsConstructor @AllArgsConstructor
public class Product { private Long id; private String name; private double price; }

5. product-service/src/main/java/com/example/productservice/config/SecurityConfig.java:
(This SecurityConfig only defines specific authorization rules for this service, relying on JwtSecurityConfig for the core filter chain setup)
package com.example.productservice.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
// Removed @EnableWebSecurity and @EnableMethodSecurity here as they are in JwtSecurityConfig
public class SecurityConfig {

    // This bean name is distinct from jwtFilterChain in common-security
    @Bean
    public SecurityFilterChain productSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            // These are applied AFTER the common-security's JwtAuthFilter
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/products/public/**").permitAll() // Public products
                .anyRequest().authenticated() // All other requests require authentication
            );
            // csrf and sessionManagement are already handled by JwtSecurityConfig

        return http.build();
    }
}

6. product-service/src/main/java/com/example/productservice/product/ProductController.java: (No change)
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

    private final List<Product> products = new ArrayList<>();
    private Long nextId = 1L;

    public ProductController() {
        products.add(new Product(nextId++, "Laptop", 1200.00));
        products.add(new Product(nextId++, "Mouse", 25.00));
        products.add(new Product(nextId++, "Keyboard", 75.00));
    }

    @GetMapping("/public")
    public ResponseEntity<List<Product>> getPublicProducts() {
        return ResponseEntity.ok(products);
    }

    @GetMapping
    @PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')")
    public ResponseEntity<List<Product>> getAllProducts() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        System.out.println("User accessing protected products: " + authentication.getName());
        System.out.println("Authorities: " + authentication.getAuthorities());
        return ResponseEntity.ok(products);
    }

    @PostMapping
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    public ResponseEntity<Product> addProduct(@RequestBody Product product) {
        final Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        System.out.println("Admin " + authentication.getName() + " adding product.");
        product.setId(nextId++);
        products.add(product);
        return ResponseEntity.status(201).body(product);
    }

    @GetMapping("/my-info")
    @PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')")
    public ResponseEntity<?> getMyInfo() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        return ResponseEntity.ok("Hello, " + authentication.getName() + "! Your roles: " + authentication.getAuthorities());
    }
}

V. order-service Module
Create order-service subdirectory.
(This will be almost identical to product-service in terms of structure and dependency)
1. order-service/pom.xml:
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
    </dependencies>

</project>

2. order-service/src/main/resources/application.properties:
server.port=8083
app.jwt.secret=yourVeryLongAndSecureSecretKeyForJWTAuthServices1234567890 # MUST BE THE SAME

3. order-service/src/main/java/com/example/orderservice/OrderServiceApplication.java:
package com.example.orderservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Import;
import com.example.common.security.config.JwtSecurityConfig;

@SpringBootApplication
@ComponentScan(basePackages = {"com.example.orderservice", "com.example.common.security"})
@Import(JwtSecurityConfig.class)
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

4. order-service/src/main/java/com/example/orderservice/order/Order.java: (No change)
package com.example.orderservice.order;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data @NoArgsConstructor @AllArgsConstructor
public class Order { private Long id; private String customerUsername; private String productName; private int quantity; private double totalAmount; }

5. order-service/src/main/java/com/example/orderservice/config/SecurityConfig.java:
package com.example.orderservice.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain orderSecurityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/orders/public/**").permitAll()
                .anyRequest().authenticated()
            );
        return http.build();
    }
}

6. order-service/src/main/java/com/example/orderservice/order/OrderController.java: (No change)
package com.example.orderservice.order;

import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
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

    @GetMapping("/public")
    public ResponseEntity<List<Order>> getPublicOrders() {
        return ResponseEntity.ok(orders);
    }

    @GetMapping("/my-orders")
    @PreAuthorize("hasAnyRole('ROLE_USER', 'ROLE_ADMIN')")
    public ResponseEntity<List<Order>> getMyOrders() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String currentUsername = authentication.getName();
        List<Order> userOrders = orders.stream()
                .filter(order -> order.getCustomerUsername().equals(currentUsername))
                .collect(Collectors.toList());
        return ResponseEntity.ok(userOrders);
    }

    @GetMapping
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    public ResponseEntity<List<Order>> getAllOrders() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        System.out.println("Admin user viewing all orders: " + authentication.getName());
        return ResponseEntity.ok(orders);
    }

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
        return ResponseEntity.status(201).body(order);
    }
}

VI. api-gateway Module
Create api-gateway subdirectory.
1. api-gateway/pom.xml:
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
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

2. api-gateway/src/main/resources/application.properties:
server.port=8080

spring.cloud.gateway.routes[0].id=auth-service
spring.cloud.gateway.routes[0].uri=http://localhost:8081
spring.cloud.gateway.routes[0].predicates[0]=Path=/api/auth/**

spring.cloud.gateway.routes[1].id=product-service
spring.cloud.gateway.routes[1].uri=http://localhost:8082
spring.cloud.gateway.routes[1].predicates[0]=Path=/api/products/**

spring.cloud.gateway.routes[2].id=order-service
spring.cloud.gateway.routes[2].uri=http://localhost:8083
spring.cloud.gateway.routes[2].predicates[0]=Path=/api/orders/**

spring.webflux.cors.allow-credentials=true
spring.webflux.cors.allowed-origins=*
spring.webflux.cors.allowed-methods=GET,POST,PUT,DELETE,OPTIONS
spring.webflux.cors.allowed-headers=*
spring.webflux.cors.max-age=3600

3. api-gateway/src/main/java/com/example/apigateway/ApiGatewayApplication.java: (No change)
package com.example.apigateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
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

    @Bean
    public CorsWebFilter corsWebFilter() {
        CorsConfiguration corsConfig = new CorsConfiguration();
        corsConfig.setAllowedOrigins(Collections.singletonList("*"));
        corsConfig.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        corsConfig.setAllowedHeaders(Collections.singletonList("*"));
        corsConfig.setAllowCredentials(true);
        corsConfig.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", corsConfig);
        return new CorsWebFilter(source);
    }
}

How to Run and Test:
 * Project Setup: Create the parent directory (microservices-jwt-shared) and the five subdirectories (common-security, auth-service, product-service, order-service, api-gateway).
 * Copy Files: Place all the respective pom.xml, application.properties, and Java source files in their correct locations. Ensure package names match (com.example.common.security, com.example.authservice, etc.).
 * Build All Projects (from the microservices-jwt-shared parent directory):
   mvn clean install

   This command will build common-security first, then the other modules which depend on it.
 * Run Applications (in separate terminal windows/tabs):
   # Terminal 1: Auth Service (Port 8081)
cd auth-service
mvn spring-boot:run

# Terminal 2: Product Service (Port 8082)
cd product-service
mvn spring-boot:run

# Terminal 3: Order Service (Port 8083)
cd order-service
mvn spring-boot:run

# Terminal 4: API Gateway (Port 8080)
cd api-gateway
mvn spring-boot:run

Testing:
The testing steps using curl or Postman/Insomnia are largely the same as before.
 * Register users via /api/auth/register.
 * Login to get tokens via /api/auth/login.
 * Access product endpoints via /api/products (protected) and /api/products/public (public).
 * Now, also test order endpoints:
   * GET http://localhost:8080/api/orders/public (public)
   * GET http://localhost:8080/api/orders/my-orders (protected, requires token, should show orders for the logged-in user only)
   * GET http://localhost:8080/api/orders (protected, admin only)
   * POST http://localhost:8080/api/orders (protected, user/admin to create, requires token)
This multi-module structure correctly implements the concept of "one JwtUtil and one JwtAuthFilter" by sharing them as a common library. Each service that needs JWT validation simply includes this library and enables its configuration, leveraging the consistent, centrally managed security logic.
