package com.tcs.security;

import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.log4j.Log4j;
import lombok.extern.log4j.Log4j2;
import lombok.extern.slf4j.Slf4j;
import org.apache.log4j.Logger;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.*;
import java.io.BufferedReader;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.regex.Pattern;


/**
 * Author : Falguni Nakhwa (V1012981)
 * Date : 6 May 2025

 * A servlet filter that reads a JSON payload from the incoming HTTP request,
 * parses it into a Map<String, Object>, decrypt the request and stores the map as a request attribute.

 * Request attribute allows downstream components such as controllers to access the parsed JSON
 * without needing to re-read or re-parse the request body.

 * Key functionality:
 * - Wraps the original HttpServletRequest to enable multiple reads of the input stream.
 * - Parses JSON request body using Jackson's ObjectMapper.
 * - Stores the resulting Map in request attributes under the key "data".
 *
 */
//@Log4j
public class JsonFilter implements Filter {

    private static final List<Pattern> AUTH_ROUTES = new ArrayList<>();
    private static final List<String> NO_AUTH_ROUTES = new ArrayList<>();
    private static final List<Pattern> NO_AUTH_ROUTES_PATTERNS = new ArrayList<>();
    static Logger log = Logger.getLogger("JsonFilter");
    static {
        AUTH_ROUTES.add(Pattern.compile("/BS/*"));
        //NO_AUTH_ROUTES.add("/BS/Security/login");
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

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        System.out.println("JsonFilter  ss");
        log.info("JsonFilter log");


        try {

            HttpServletRequest httpRequest = (HttpServletRequest) request;
            CachedBodyHttpServletRequest wrappedRequest = new CachedBodyHttpServletRequest(httpRequest);

            String route = httpRequest.getRequestURI();
            log.info(route);

            if (route.endsWith(".jsp")) {
                chain.doFilter(wrappedRequest, response);
                return;
            }

            if (NO_AUTH_ROUTES.contains(route)) {
                log.info("JsonFilter 85");
                chain.doFilter(wrappedRequest, response);
                return;
            }

            for (Pattern p : NO_AUTH_ROUTES_PATTERNS) {
                if (p.matcher(route).find()) {
                    log.info("JsonFilter 92");
                    chain.doFilter(wrappedRequest, response);
                   return;
                }
            }

            String body = new BufferedReader(wrappedRequest.getReader())
                    .lines()
                    .collect(java.util.stream.Collectors.joining(System.lineSeparator()));

            ObjectMapper mapper = new ObjectMapper();

            // Convert JSON string to map
            Map<String, Object> jsonMap = mapper.readValue(body, Map.class);

            String decryptedData = AESGCM256.decrypt(
                    (String) jsonMap.get("data"),
                    (String) jsonMap.get("iv"),
                    (String) jsonMap.get("salt")
            );
            log.info("JsonFilter 112");
            log.info(decryptedData);

            // Convert JSON string to map
            Map<String, Object> dataMap = mapper.readValue(decryptedData, Map.class);

            log.info("fgvsdfgvsd >>> " + dataMap);

            // Add to request attribute
            wrappedRequest.setAttribute("data", dataMap);

            chain.doFilter(wrappedRequest, response);
        } catch (Exception e) {
            System.out.println("Exception : " + e.getMessage());
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    @Override
    public void destroy() {}
}





///////////////////////////
controller
 @RequestMapping(value = "/getSavedDataNineMig", method = RequestMethod.POST)
    public @ResponseBody List<SC9CMigration> getSavedDataNineMig(@RequestBody SC9CMigration row) {
        ////log.info("we r in AdminConller");

        return makerService.getSavedDataNineMig(row);
    }

    @InitBinder("getSavedDataNineMigTwo")
    public void initBindergetSavedDataNineMigTwo(WebDataBinder binder) {
        binder.setDisallowedFields();
    }

    ////////////// schedule 9C MigrationEnd ///////

    @RequestMapping(value = "/getSavedDataNineMigTwo", method = RequestMethod.POST)
    public @ResponseBody List<SC9CMigration> getSavedDataNineMigTwo(@RequestBody SC9CMigration row) {
        ////log.info("we r in AdminConller");

        return makerService.getSavedDataNineMigTwo(row);
    }

    ////////////// schedule 10 start ///////
    @InitBinder("submitTen")
    public void initBindersubmitTen(WebDataBinder binder) {
        binder.setDisallowedFields();
    }

    @RequestMapping(value = "/submitTen", method = RequestMethod.POST, produces = "text/plain")
    public @ResponseBody String submitTen(@RequestBody SC10 row) {
        ////log.info("we r in AdminConller");

        String updated = makerService.submitTen(row);

        return CleanPath.cleanString(updated);
    }
