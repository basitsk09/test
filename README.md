package com.tcs.security;

import com.tcs.services.LoginService;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.Jwts;
import org.apache.bcel.generic.DADD;
import org.apache.commons.lang.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.web.filter.OncePerRequestFilter;
import com.tcs.services.LoginService;
import com.tcs.services.LoginServiceImpl;
import javax.activation.DataSource;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;
import java.util.Date;
import java.util.concurrent.locks.ReentrantLock;
import java.util.regex.Pattern;

public class JWTTokenAuthFilter extends OncePerRequestFilter {
    public static final String JWT_KEY = "JWT-TOKEN-SECRET";
    private static final List<Pattern> AUTH_ROUTES = new ArrayList<>();
    private static final List<String> NO_AUTH_ROUTES = new ArrayList<>();
    private static final List<Pattern> NO_AUTH_ROUTES_PATTERNS = new ArrayList<>();
    private static ReentrantLock lock = null;

    static {
        AUTH_ROUTES.add(Pattern.compile("/BS/*"));
        NO_AUTH_ROUTES.add("/BS/Security/login");
        NO_AUTH_ROUTES.add("/BS/Security/logout");
        NO_AUTH_ROUTES.add("/BS/Security/reNewSession");
        NO_AUTH_ROUTES.add("/BS/index.jsp");
        NO_AUTH_ROUTES.add("/BS/views/login.jsp");
        NO_AUTH_ROUTES.add("/BS/pdfStream.jsp");
        NO_AUTH_ROUTES.add("/BS/displaySignedPDF.jsp");
        NO_AUTH_ROUTES.add("/BS/signPDF.jsp");
        NO_AUTH_ROUTES.add("/BS/favicon.ico");
        NO_AUTH_ROUTES.add("/BS/signapplet.jar.pack.gz");
        NO_AUTH_ROUTES.add("/BS/Admin/downloadSignedReport");
        NO_AUTH_ROUTES.add("/BS/displayPDF.jsp");
        NO_AUTH_ROUTES.add("/BS/acceptReport.jsp");
        NO_AUTH_ROUTES_PATTERNS.add(Pattern.compile("/BS/resources/*"));
        NO_AUTH_ROUTES_PATTERNS.add(Pattern.compile("/BS/assets/*"));
        NO_AUTH_ROUTES_PATTERNS.add(Pattern.compile("/BS/Security/downloadHelp"));
        NO_AUTH_ROUTES_PATTERNS.add(Pattern.compile("/BS/Security/downloadDocs"));
    }

    /*private Logger LOG = LoggerFactory.getLogger(JWTTokenAuthFilter.class);*/

    /*@Autowired
    private UserService userService;*/

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        if (null == lock) {
            lock = new ReentrantLock();
        }
        /* This is to bypass OPTIONS For BSA nextGen not for Production*/
        if("OPTIONS".equals(request.getMethod())) {
            response.setHeader("Access-Control-Allow-Origin", "*"); // or specific domain
            response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE, PUT");
            response.setHeader("Access-Control-Allow-Headers", "Authorization, Content-Type");
            response.setHeader("Access-Control-Allow-Credentials", "true");
            response.setStatus(HttpServletResponse.SC_OK);
            return;
        }

        String authorizationHeader = request.getHeader("authorization");
        String authenticationHeader = request.getHeader("authentication");
        String route = request.getRequestURI();
        // no auth route matching
        boolean needsAuthentication = false;
        for (Pattern p : AUTH_ROUTES) {
            if (p.matcher(route).matches()) {
                needsAuthentication = true;
                break;
            }
        }
//        logger.info("route: " + route);
        if (route.startsWith("/BS/")) {
            needsAuthentication = true;
        }

        if (NO_AUTH_ROUTES.contains(route)) {
            needsAuthentication = false;
        }

        for (Pattern p : NO_AUTH_ROUTES_PATTERNS) {
            if (p.matcher(route).find()) {
                needsAuthentication = false;
                break;
            }
        }
        // Checking whether the current route needs to be authenticated
        if (needsAuthentication) {
            // Check for authorization header presence
            String authHeader = null;
            if (authorizationHeader == null || authorizationHeader.equalsIgnoreCase("")) {
                if (authenticationHeader == null || authenticationHeader.equalsIgnoreCase("")) {
                    authHeader = null;
                } else {
                    authHeader = authenticationHeader;
                }
            } else {
                authHeader = authorizationHeader;
            }
//            logger.info(authHeader.length());
            if (StringUtils.isBlank(authHeader) || !authHeader.startsWith("Bearer ")) {
                throw new AuthCredentialsMissingException("Missing or invalid Authorization header.");
            }

            final String token = authHeader.substring(7); // The part after "Bearer "

            try {
                final Claims claims = Jwts.parser().setSigningKey(JWT_KEY)
                        .parseClaimsJws(token).getBody();


                // @@@ Validate the user session (using `userId` key from frontend)
                boolean validation = validateSession(request, claims, response,token);
                if (!validation) {
                    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED, "0");
                    //response.sendRedirect("/BS/Security/logout");
                    return ;

                }
//                logger.info("Returned setAttribute");
                    request.setAttribute("claims", claims);
                    // Now since the authentication process if finished
                    // move the request forward

//                logger.info("Returned setAttribute successfully");
                    filterChain.doFilter(request, response);


            } catch (final Exception e) {
                if(e.getClass() == ExpiredJwtException.class) {
                    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED,"1");
                    return ;
                }
                e.printStackTrace();
                throw new AuthenticationFailedException("Invalid token. Cause:" + e.getMessage());
            }
        } else {
            filterChain.doFilter(request, response);
        }
    }

    // Validate the session using front end token and backend stored token
    private boolean validateSession(HttpServletRequest request, Claims claims, HttpServletResponse response,String token) throws Exception {
        String userIdFromToken = claims.get("userId", String.class);
        String validToken = getUserToken(userIdFromToken,lock);
        //String userIdFromSession = (String) request.getSession().getAttribute("userId");
        if (validToken == null || !validToken.equals(token))  {
            System.err.println("User mismatch detected. Possible token manipulation.");
            return false;
        }
        return true;
    }

    private String getUserToken(String userID, ReentrantLock lock) throws SQLException {
        Connection con = new DBConn().getConnectionFromJNDI();
        String sessionId = null;
        try {
            String query = "select USER_SESSION from bs_user where user_id = ? ";
            lock.lock();
            PreparedStatement ps = con.prepareStatement(query);
            ps.setString(1, userID);
            ResultSet rs = ps.executeQuery();
            if (rs.next()) {
                sessionId = rs.getString("USER_SESSION");
            }
            rs.close();
            con.close();

        } catch (SQLException e) {
            logger.error("SQL Exception Occurred" + " : " + e.getMessage());
        } finally {
            if (null != con) {
                try {
                    con.close();
                } catch (SQLException e) {
                   logger.error("SQL Exception Occurred" + " : " + e.getMessage());
                }
            }
            lock.unlock();
        }
        return sessionId;
    }

}



