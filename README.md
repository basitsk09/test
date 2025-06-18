Great — let’s break down the SecurityConfig class in Spring Boot so you understand when it is called and why it exists.


---

🔐 What is SecurityConfig?

It’s a Java configuration class that customizes Spring Security’s default behavior.
It extends WebSecurityConfigurerAdapter (or implements SecurityFilterChain in Spring Security 6+).


---

🎯 Purpose of SecurityConfig

Responsibility	What it does

🔐 Define secured vs public endpoints	Tells Spring which URLs need authentication (.authorizeRequests())
🧍 Set authentication rules	How to authenticate (via JWT, form login, etc.)
🛡️ Register filters	Adds JwtFilter before Spring's default filters
🔑 Provide user details & password encoding	Inject UserDetailsService and password encoder
🚫 Disable unwanted defaults	Disable CSRF, form login, etc., for REST API



---

🕰️ When is SecurityConfig Called?

✅ It is automatically loaded at app startup due to the @Configuration and @EnableWebSecurity annotations.

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // called automatically on startup
}


---

🔍 Key Sections in SecurityConfig

1. configure(AuthenticationManagerBuilder auth)

🔐 Sets the authentication method (e.g., in-memory or DB via UserDetailsService).

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService)
        .passwordEncoder(new BCryptPasswordEncoder());
}

This tells Spring:

> “Use my custom logic (CustomUserDetailsService) to authenticate users, and hash passwords using BCrypt.”




---

2. configure(HttpSecurity http)

🛡️ Configures what is protected, and what filters to apply.

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.csrf().disable()
        .authorizeRequests()
        .antMatchers("/auth/login").permitAll() // no token needed here
        .anyRequest().authenticated()           // all other requests need token
        .and().sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

    http.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
}

This tells Spring:

Don’t use CSRF protection (not needed for REST APIs)

Allow /auth/login without authentication

All other URLs need JWT

Don’t use session cookies

Add our JwtFilter before Spring's UsernamePasswordAuthenticationFilter



---

3. authenticationManagerBean()

🔑 Needed when we do manual login like:

authenticationManager.authenticate(
  new UsernamePasswordAuthenticationToken(username, password)
);

@Bean
public AuthenticationManager authenticationManagerBean() throws Exception {
    return super.authenticationManagerBean();
}

This lets you inject AuthenticationManager into your controller or service.


---

✅ Summary of What SecurityConfig Does

Step	Purpose

1	Configure how users are authenticated
2	Define which endpoints require login
3	Attach custom filters like JWT
4	Customize security policies (no session, no CSRF, etc.)
5	Automatically picked up at startup by Spring



---

Would you like this rewritten using Spring Security 6 style (with SecurityFilterChain bean) too?

