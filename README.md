Great question — let’s break it down clearly.


---

🔄 When You First Login — What Happens?

❓ No Token Yet — So Why Does the JWT Filter Run First?

✅ Yes, the JWT filter still runs, but it does not validate or block the request if it's for a public endpoint like /auth/login.


---

🔎 Here's How It Works:

1. 🔁 All requests go through the Security Filter Chain — that includes your custom JwtFilter or any Spring Security filters.


2. ⚖️ Your JwtFilter checks:

Is there a token in the Authorization header?

If yes, validate it and set authentication.

If no, just pass the request along unmodified.



3. 🛑 The filter must not block public endpoints like /auth/login.




---

✅ Practical Implementation in JwtFilter:

@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {

    String authHeader = request.getHeader("Authorization");

    if (authHeader != null && authHeader.startsWith("Bearer ")) {
        String token = authHeader.substring(7);
        String username = jwtUtil.extractUsername(token);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            if (jwtUtil.validateToken(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
    }

    // No token? Not our job. Pass request to next filter/controller
    filterChain.doFilter(request, response);
}


---

👇 OR use shouldNotFilter() to skip login path

@Override
protected boolean shouldNotFilter(HttpServletRequest request) {
    String path = request.getServletPath();
    return path.equals("/auth/login") || path.equals("/auth/register");
}


---

✅ Summary

Step	What Happens

Login Request (/auth/login)	Hits JwtFilter first, but filter skips token check
Token not required	Because it's a public URL (permitAll() in SecurityConfig)
Controller Handles Login	Controller validates credentials, generates token
Next Requests	Must carry the JWT in Authorization header



---

Let me know if you want to see how to generate a refresh token or how token expiry is handled!

