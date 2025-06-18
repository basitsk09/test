Here’s a detailed and practical guide to implementing end-to-end security in a full-stack web application using Spring Boot (backend) and React (frontend) with JWT authentication, role-based access, and security best practices.


---

🔐 1. SECURITY ARCHITECTURE OVERVIEW

🔄 Flow:

[React Frontend] ⇄ [JWT Token API Login] ⇄ [Spring Boot Backend] ⇄ [Database]
                            ↓
                      [JWT Filter]
                            ↓
                  [User Access + Role Verification]


---

🧱 2. BACKEND SECURITY (SPRING BOOT)

2.1. 🔐 User Authentication with JWT

➤ Step 1: Add Dependencies

<!-- spring-boot-starter-security, web, jwt dependencies -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt</artifactId>
  <version>0.9.1</version>
</dependency>

➤ Step 2: JWT Utility Class

@Component
public class JwtUtil {
    private final String SECRET_KEY = "my-secret";

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60)) // 1 hour
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
    }

    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    public String extractUsername(String token) {
        return extractAllClaims(token).getSubject();
    }

    public boolean isTokenExpired(String token) {
        return extractAllClaims(token).getExpiration().before(new Date());
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token).getBody();
    }
}

➤ Step 3: JWT Filter

@Component
public class JwtFilter extends OncePerRequestFilter {

    @Autowired private JwtUtil jwtUtil;
    @Autowired private CustomUserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {
        String authHeader = request.getHeader("Authorization");
        String token = null;
        String username = null;

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            token = authHeader.substring(7);
            username = jwtUtil.extractUsername(token);
        }

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            if (jwtUtil.validateToken(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response);
    }
}


---

2.2. 🔒 Spring Security Config

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired private JwtFilter jwtFilter;
    @Autowired private CustomUserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(new BCryptPasswordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .antMatchers("/auth/**").permitAll()  // Allow login/register
            .antMatchers("/admin/**").hasRole("ADMIN")
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

2.3. 🛡️ Role-Based Access (RBAC)

Define roles like ROLE_ADMIN, ROLE_USER

In database or UserDetails object, store roles

Check roles using hasRole() or @PreAuthorize



---

🌐 3. FRONTEND SECURITY (React)

3.1. 🔑 Login Form → JWT

const login = async (username, password) => {
  const response = await axios.post('/auth/login', { username, password });
  const { token } = response.data;
  localStorage.setItem('token', token); // or sessionStorage
};


---

3.2. ⛓️ Sending JWT in API Calls

const axiosInstance = axios.create({
  baseURL: 'http://localhost:8080',
  headers: {
    Authorization: `Bearer ${localStorage.getItem('token')}`,
  },
});

Use interceptors to automatically attach token:

axiosInstance.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});


---

3.3. 🔒 Protect Frontend Routes

<Route path="/admin" element={
  isLoggedIn && userRole === 'ADMIN' ? <AdminPage /> : <Navigate to="/login" />
} />


---

🛡️ 4. ADDITIONAL SECURITY BEST PRACTICES

Concern	What to Do

Password storage	Use BCryptPasswordEncoder
JWT expiration	Short-lived tokens (e.g., 15min), refresh token mechanism
CSRF (in SPA)	Not needed if JWT used properly and no cookies
CORS	Enable CORS for only trusted origins in backend config
HTTPS	Use Nginx reverse proxy or enable SSL directly in Spring Boot
XSS Protection	Sanitize inputs, use dangerouslySetInnerHTML with caution in React
SQL Injection	Use ORM (JPA), PreparedStatements
Logs	Avoid logging sensitive data like passwords, JWTs



---

✅ SUMMARY

Component	Tool/Concept

Auth Protocol	JWT + HTTP Authorization Header
Backend Auth	Spring Security + JWT Filter
Password Storage	BCrypt hashing
Access Control	Role-based via hasRole() / @Secured
Frontend Auth	LocalStorage + Axios Interceptor
Transport Layer	HTTPS (TLS) with reverse proxy/Nginx
CORS	Allowed origin + methods
Security Testing	OWASP ZAP, Postman, Snyk, SonarQube



---

Would you like a complete starter boilerplate project with this full JWT security setup? I can generate it for you.

