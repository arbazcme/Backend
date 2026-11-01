## Filters 

Filter is a component that intercepts HTTP request/response before dispatcher servlet.It acts as middleware between web server Tomcat and Dispatcher servlet.  
This Filter component is introduced in servlet and servlet-container period.  
What Filters are used for  
1. Inspect Request (Headers, Body, Cookies)
2. Check Authentication information
3. Add Response Headers
4. Modify request or response
5. Block/Allow the request
6. Log Request Details
7. Measure total request duration (Total time to serve)

### Position of Filter in Spring flow 
<img src="images/filterpos.png" width="100%" height="100%">

### Filter code
```java
@Component
@Order(1)
public class AuthenticationFilter implements Filter  {
    @Override
    public void doFilter(ServletRequest servletRequest,ServletResponse servletResponse,FilterChain filterChain)throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) servletRequest;
        HttpServletResponse httpResponse = (HttpServletResponse)  servletResponse;

        String token =  httpRequest.getHeader("token");
        String apiKey = httpRequest.getHeader("x-api-key");

        if(token == null || !token.equals("12345")) {
            httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return;
        }
        if(apiKey == null || !apiKey.equals("secret123")) {
            httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            httpResponse.setContentType("application/json");
            httpResponse.getWriter().write("{\n" + " \"message\" : \"Invalid or missing Api key\"\n" + "}");
            return;
        }
        filterChain.doFilter(servletRequest, servletResponse);
    }
}

@Component
@Order(2)
public class LoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest,ServletResponse servletResponse,FilterChain filterChain)throws IOException, ServletException {

        long startTime = System.currentTimeMillis();
        HttpServletRequest httpRequest = (HttpServletRequest) servletRequest;
        HttpServletResponse httpResponse =(HttpServletResponse)  servletResponse;

        String requestId = UUID.randomUUID().toString();
        httpResponse.setHeader("X-Request-ID", requestId);

        // Request log
        System.out.println("Incoming Request : "+ httpRequest.getMethod() + " "+ httpRequest.getRequestURI());
      try {
          filterChain.doFilter(servletRequest, servletResponse);
      }
      finally {
          long duration = System.currentTimeMillis() - startTime;
          // Response status log
          System.out.println("Response status: "+  httpResponse.getStatus());
          System.out.println("API Response time : " + duration);
      }
    }
}

```
