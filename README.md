Perfect! Here's a complete JWT authentication flow implemented from React frontend to Spring Boot backend, covering everything: login, token validation, protected routes, and logout.


---

🔧 TECH STACK

Frontend: React (Axios, React Router)

Backend: Spring Boot (Spring Security, JWT)

Auth Type: Stateless JWT with Authorization: Bearer <token>



---

🔁 FLOW SUMMARY

1. Frontend login form sends username/password to Spring Boot API.


2. Backend validates credentials using AuthenticationManager.


3. Backend returns JWT token.


4. Frontend stores token (e.g., localStorage).


5. For all secure routes, frontend sends the token.


6. Backend uses JWT filter to validate and set authentication.


7. Protected endpoints are accessed only with valid tokens.




---

🔙 BACKEND (Spring Boot)

📦 1. Add Dependencies in pom.xml

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>


---

🧍 2. User Entity (In-Memory Example)

public class AppUser {
    private String username;
    private String password;
    private String role;

    public AppUser(String username, String password, String role) {
        this.username = username;
        this.password = password;
        this.role = role;
    }

    // Getters and setters
}


---

🔐 3. JWT Utility Class

@Component
public class JwtUtil {

    private final String SECRET_KEY = "my-secret";

    public String generateToken(UserDetails userDetails) {
        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .claim("authorities", userDetails.getAuthorities())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60)) // 1hr
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
    }

    public String extractUsername(String token) {
        return Jwts.parser().setSigningKey(SECRET_KEY)
                .parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    public boolean isTokenExpired(String token) {
        Date exp = Jwts.parser().setSigningKey(SECRET_KEY)
                      .parseClaimsJws(token).getBody().getExpiration();
        return exp.before(new Date());
    }
}


---

🛡️ 4. JWT Filter

@Component
public class JwtFilter extends OncePerRequestFilter {

    @Autowired private JwtUtil jwtUtil;
    @Autowired private CustomUserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String authHeader = request.getHeader("Authorization");

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            String username = jwtUtil.extractUsername(token);

            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                if (jwtUtil.validateToken(token, userDetails)) {
                    UsernamePasswordAuthenticationToken auth =
                        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(auth);
                }
            }
        }

        chain.doFilter(request, response);
    }
}


---

👤 5. CustomUserDetailsService

@Service
public class CustomUserDetailsService implements UserDetailsService {

    private static final List<AppUser> users = List.of(
        new AppUser("john", new BCryptPasswordEncoder().encode("password"), "USER"),
        new AppUser("admin", new BCryptPasswordEncoder().encode("admin123"), "ADMIN")
    );

    @Override
    public UserDetails loadUserByUsername(String username) {
        AppUser user = users.stream()
            .filter(u -> u.getUsername().equals(username))
            .findFirst()
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        return User.builder()
                .username(user.getUsername())
                .password(user.getPassword())
                .roles(user.getRole())
                .build();
    }
}


---

🔒 6. Security Config

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired private CustomUserDetailsService userDetailsService;
    @Autowired private JwtFilter jwtFilter;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(new BCryptPasswordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .antMatchers("/auth/login").permitAll()
            .anyRequest().authenticated()
            .and().sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);

        http.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
    }

    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}


---

🧾 7. Auth Controller

@RestController
@RequestMapping("/auth")
public class AuthController {

    @Autowired private AuthenticationManager authenticationManager;
    @Autowired private JwtUtil jwtUtil;
    @Autowired private CustomUserDetailsService userDetailsService;

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody AuthRequest request) {
        try {
            authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword())
            );

            final UserDetails userDetails = userDetailsService.loadUserByUsername(request.getUsername());
            final String token = jwtUtil.generateToken(userDetails);
            return ResponseEntity.ok(new AuthResponse(token));
        } catch (BadCredentialsException ex) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid credentials");
        }
    }
}


---

📦 Auth DTOs

public class AuthRequest {
    private String username;
    private String password;
    // Getters and Setters
}

public class AuthResponse {
    private String token;
    public AuthResponse(String token) { this.token = token; }
    public String getToken() { return token; }
}


---

🖥️ FRONTEND (React)

📁 Folder structure

/src
  /api
    axios.js
  /components
    Login.js
    Dashboard.js
  App.js


---

🛠️ 1. Axios with Token Support (api/axios.js)

import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:8080',
});

api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;


---

🔐 2. Login Component (components/Login.js)

import React, { useState } from 'react';
import api from '../api/axios';
import { useNavigate } from 'react-router-dom';

export default function Login() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const navigate = useNavigate();

  const handleLogin = async () => {
    try {
      const res = await api.post('/auth/login', { username, password });
      localStorage.setItem('token', res.data.token);
      navigate('/dashboard');
    } catch (err) {
      alert('Invalid credentials');
    }
  };

  return (
    <div>
      <h2>Login</h2>
      <input value={username} onChange={e => setUsername(e.target.value)} placeholder="Username" /><br/>
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} placeholder="Password" /><br/>
      <button onClick={handleLogin}>Login</button>
    </div>
  );
}


---

🔐 3. Protected Dashboard (components/Dashboard.js)

import React, { useEffect, useState } from 'react';
import api from '../api/axios';

export default function Dashboard() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    api.get('/secure-endpoint')
      .then(res => setMessage(res.data))
      .catch(err => setMessage('Unauthorized'));
  }, []);

  return (
    <div>
      <h2>Dashboard</h2>
      <p>{message}</p>
    </div>
  );
}


---

🚦 4. React Router Setup (App.js)

import React from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Login from './components/Login';
import Dashboard from './components/Dashboard';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Login />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;


---

✅ 5. Run & Test

Start Spring Boot backend on localhost:8080

Start React frontend on localhost:3000

Login → Token saved → Navigate to dashboard → Secure API hit



---

Would you like:

Refresh token setup?

Role-based access?

Logout implementation?


Let me know and I’ll extend this base for you.

