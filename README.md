Great follow-up. Here's how to implement the Login Controller in Spring Boot to handle JWT-based authentication.


---

🎯 Goal

We need a controller that:

Accepts username and password via POST /auth/login

Authenticates the user using AuthenticationManager

Generates a JWT if credentials are valid

Returns the token in the response



---

✅ Step-by-Step: LoginController.java

@RestController
@RequestMapping("/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private JwtUtil jwtUtil;

    @Autowired
    private CustomUserDetailsService userDetailsService;

    // Endpoint for login
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody AuthRequest authRequest) {
        try {
            // 1. Authenticate credentials
            authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    authRequest.getUsername(), authRequest.getPassword()
                )
            );

            // 2. Load user details
            UserDetails userDetails = userDetailsService.loadUserByUsername(authRequest.getUsername());

            // 3. Generate JWT
            String token = jwtUtil.generateToken(userDetails);

            // 4. Return token
            return ResponseEntity.ok(new AuthResponse(token));

        } catch (BadCredentialsException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid credentials");
        }
    }
}


---

📦 DTOs: AuthRequest & AuthResponse

public class AuthRequest {
    private String username;
    private String password;

    // Getters and setters
}

public class AuthResponse {
    private String token;

    public AuthResponse(String token) {
        this.token = token;
    }

    public String getToken() {
        return token;
    }
}


---

🔐 CustomUserDetailsService

This should load users from DB or in-memory and implement UserDetailsService.

@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository; // Optional, if using DB

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // In-memory or DB lookup
        User user = userRepository.findByUsername(username)
                        .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        return new org.springframework.security.core.userdetails.User(
            user.getUsername(), user.getPassword(), getAuthorities(user)
        );
    }

    private Collection<? extends GrantedAuthority> getAuthorities(User user) {
        return Collections.singleton(new SimpleGrantedAuthority("ROLE_" + user.getRole()));
    }
}


---

🧪 Sample Request from Postman/Frontend

POST /auth/login

{
  "username": "john",
  "password": "pass123"
}

Response:

{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI..."
}


---

Let me know if you'd like the User entity, repository, or database integration for this login setup too.

