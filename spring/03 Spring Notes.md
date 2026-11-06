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


### Filter Lifecycle
- init() when filter is created. starting of app, this is not manditory
- doFilter() calling next filter when one filter has done its work
- destroy() when application is closing this destroy runs

Tomcat creates HttpServletRequest but the references are of ServletRequest in filter because they are general purpose and not limited to HttpRequests this might be there in servlets as well but the underlying object is HttpServletRequest.  
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

### Why cant we make changes in Response object after doFilter() ?

Dispatcher Servlet internally uses streams to populate the body. once stream is used then the response may be commited state (sent or started sending the response to user) 
`response.getWriter().write()` basically this line. better to set before.  

Tip :  based on ResponseEntity class's object the dispatcher servlet populates the HttpServletResponse.   


### Input Stream in Filter 
If you try to read the input stream say for logging purpose httpservletresponse.getReader() say, then pass it in doFilter() you will get empty stream inside HttpservletResponse as stream is not stored it is continously flowing and its already consumed. so be careful on that.  

### Ways to declare Filters 
- One Way of implementing Filter is above mentioned
- OncePerRequestFilter Abstract class or You can implement Filter but Typecast ServletRequest to HttpServletRequest Manually
```java
@Component
@Order(1)
public class LogFilter extends OncePerRequestFilter {

    @Override
    public void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String id = UUID.randomUUID().toString();
        System.out.println(request.getMethod() + " " + request.getRequestURI() + " " + id);

        response.setHeader("X-Request-Id", id);

        System.out.println("Order 1");
        filterChain.doFilter(request, response);
    }
}

// Or Dont Set @Component and Order and This way

public class AuthFilter extends OncePerRequestFilter {
    @Override
    public void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        String key = "12345";
        String token = request.getHeader("x-api-token");
        if(!Objects.equals(token, key)){
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.setContentType("application/json");
            response.getWriter().write("Invalid Credentials");
            return;
        }
        System.out.println("Order 2");

        System.out.println("Valid Request of /api/");
        chain.doFilter(request, response);
    }
}

// In some Configuration class
@Configuration
public class FilterConfiguration {

    @Bean
    public FilterRegistrationBean<AuthFilter> authFilterBean() {
        FilterRegistrationBean<AuthFilter> registrationBean = new FilterRegistrationBean<>();

        registrationBean.setFilter(new AuthFilter());
        registrationBean.setOrder(2);
        registrationBean.addUrlPatterns("/api/*");
        return registrationBean;
    }
}

```

### addUrlPatterns("/api/*") 
This can add this filter in chain only if the pattern matches. You can simulate this with help of if checks inside filter the difference is you enter the filter and skip through vs filter is not entered in first place. `if(httpServletRequest.getRequestURI().startsWith("/api/")`  

### Recap
handler mapping and adapter are also passed as beans (The ApplicationContext is passed) the dispatcher Servlet uses them.  
- Handler Mapping has mapping from route to class.method like, GET /users → UserController.getUsers()  
- Handler Adapter actually invokes by populating the params. it uses jackson to convert JSON <-> Java Obj and then populates.
- Both are Beans inside Application context and total application context is passed to dispatcher servlet.

## Interceptors

### Term Handler 
handler is a genric term refers to who handles the request, in rest it is controllers and in static resourses its ResourceHttpRequestHandler etc.   

Interceptors are like filters but they have addition information as we know which method of which controller is being called. because it is after dispatcher servlet. For Specilized task that is done in set of controllers we use Interceptors and genralized task for all the controllers we do use filters.  
<img src="images/interceptors.png" width="75%" height="75%">

As we Know the controller the Interceptors chain are like layers of onion peel. 
<img src="images/interceptors_onoin.png" width="100%" height="100%">

This helps in understanding The flow in the In-Depth Architecture
<img src="images/complete_flow_interceptors.png" width="100%" height="100%">



