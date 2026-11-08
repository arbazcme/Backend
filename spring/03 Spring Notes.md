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
<img src="images/recap_of_architecture.png" width="100%" height="100%">  


### Filter Lifecycle
- init() when filter is created. starting of app, this is not manditory
- doFilter() calling next filter when one filter has done its work
- destroy() when application is closing this destroy runs

Tomcat creates HttpServletRequest but the references are of ServletRequest in filter because they are general purpose and not limited to HttpRequests this might be there in servlets as well but the underlying object is HttpServletRequest.  

### Why cant we make changes in Response object after doFilter() ?

Dispatcher Servlet internally uses streams to populate the body. once stream is used then the response may be commited state (sent or started sending the response to user) 
`response.getWriter().write()` basically this line. better to set before.   

Recap : based on ResponseEntity class's object the dispatcher servlet populates the HttpServletResponse.   


### Input Stream in Filter 

If you try to read the input stream say for logging purpose httpservletresponse.getReader() say, then pass it in doFilter() you will get empty stream inside HttpservletResponse as stream is not stored it is continously flowing and its already consumed. so be careful on that.  

<img src="images/data_as_stream.png" width="100%" height="100%">  

### Ways to declare Filters 
- OncePerRequestFilter Abstract class or You can implement Filter interface but Typecast ServletRequest to HttpServletRequest Manually

### Use of addUrlPatterns("/api/*") 
This can add some filter in chain only if the request pattern matches else the filter is not added in the chain for some request.   

The other way to achieve this is you maintain a chain and based on request you skip the filters.with help of if checks inside filter   
The difference is you enter the filter and skip through vs filter is not entered in first place.   `if(httpServletRequest.getRequestURI().startsWith("/api/")`  

## Interceptors

### Term Handler 
handler is a genric term refers to who handles the request, in rest it is controllers and in static resourses its ResourceHttpRequestHandler etc.   

Interceptors are like filters but they have addition information as we know which method of which controller is being called. because it is after dispatcher servlet. For Specilized task that is done in set of controllers we use Interceptors and genralized task for all the controllers we do use filters.   

As we Know the controller the Interceptors chain are like layers of onion peel.  
This helps in understanding The flow in the In-Depth Architecture.

<img src="images/Interceptor_chain.png" width="100%" height="100%">  

### Interceptor Methods 
- preHandle()
- postHandle()
- afterCompletion()

### postHandle() vs afterCompletion()
The difference comes when using JSP pages, in postHandle response(JSP pages) is not yet created, We can make final changes to Model Object that populates the JSP page later the response is created and sent to client later the afterCompletion runs.in REST Api's the differnece is NULL so we only use preHandle() and afterCompletion().



### Recap
handler mapping and adapter are also passed as beans (The ApplicationContext is passed) the dispatcher Servlet uses them.  
- Handler Mapping has mapping from route to class.method like, GET /users → UserController.getUsers()  
- Handler Adapter actually invokes by populating the params. it uses jackson to convert JSON <-> Java Obj and then populates.
- Both are Beans inside Application context and total application context is passed to dispatcher servlet.

<img src="images/Depth_Interceptor_chain.png" width="100%" height="100%">   

### Clarity on Recap
Who converts the JSON <-> Java Object (Obv. Jackson Library) not dispatcher not Handler Adapter but the HttpMessageConverters they are used for conversions and invoked differently
- Request: `DispatcherServlet → HandlerAdapter → Argument Resolvers → HttpMessageConverter (JSON → Java object) → Controller`.

- Response: `Controller → HandlerAdapter → Return Value Handler → HttpMessageConverter (Java object → JSON) → DispatcherServlet → postHandle() → Response Is Ready The HttpServletResponse population (as its there just popuation was pending) → afterCompletion() -> Sent to Tomcat `.


### Filters and Interceptors Practise code
```java

// AuthFilter.java
@Component
public class AuthFilter extends OncePerRequestFilter{
    @Override
    public void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        String token = request.getHeader("token");
        String key = "12345";
        if(Objects.equals(token, key)){
            System.out.println("Passed through Auth Filter");
            chain.doFilter(request, response);
            return;
        }
        response.setStatus(404);
        response.getWriter().write("Invalid key");
    }
}

// LogFilter.java
@Component
public class LogFilter extends OncePerRequestFilter {
    @Override
    public void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        System.out.println("Logging Filter");
        System.out.println("Request Method : " + request.getMethod() + "\n" + "Request route :" + request.getRequestURI());
        chain.doFilter(request, response);
    }
}

// FilterConfiguration.java
@Configuration
public class FilterConfiguration {
    public final AuthFilter authFilter;
    public  final LogFilter logFilter;
    public FilterConfiguration(AuthFilter authFilter, LogFilter logFilter){
        this.authFilter = authFilter;
        this.logFilter = logFilter;
    }
    @Bean
    public FilterRegistrationBean<AuthFilter> registerAuth(){
        FilterRegistrationBean<AuthFilter> registrationBean  = new FilterRegistrationBean<>();
        registrationBean.setFilter(authFilter);
        registrationBean.setOrder(1);
        return registrationBean;
    }
    @Bean
    public FilterRegistrationBean<LogFilter> registerLog(){
        FilterRegistrationBean<LogFilter> registerBean = new FilterRegistrationBean<>();
        registerBean.setFilter(logFilter);
        registerBean.setOrder(2);
        return registerBean;
    }
}

// AuthInterceptor.java
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler){

        if(handler instanceof HandlerMethod method){
            System.out.println("Entered Auth Interceptor");
            if(method.getBeanType() == Api.class){
                String token = request.getHeader("tokens");
                if(Objects.equals(token, "1234")){
                    System.out.println("Passed through Auth Interceptor");
                    request.setAttribute("temp", true);
                    return true;
                }else {
                    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                    return false;
                }
            }
        }
        return true;
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
                         @Nullable Exception ex) throws Exception {
        System.out.println("After completion of Interceptor");
        if((boolean)request.getAttribute("temp")){
            System.out.println("Interceptor True");
        }
    }
}

// LogInterceptor.java
@Component
public class LogInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler){
        if(handler instanceof HandlerMethod method){
            System.out.println("Entered Logging Interceptor");
            if(method.getBeanType() == Student.class){
                request.setAttribute("time", System.currentTimeMillis());
                System.out.println("HTTP Method: " + request.getMethod());
                System.out.println("Request URI: " + request.getRequestURI());
                System.out.println("Request Paramters: " + request.getQueryString());
                System.out.println("Client IP: " + request.getRemoteAddr());
                System.out.println(method.getMethod().getName() +
                        "\n" + method.getBeanType().getSimpleName());
            }
        }
        return true;
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
                                @Nullable Exception ex) throws Exception {
        System.out.println("Interceptor outer");
        Object temp = request.getAttribute("time");
        if(temp != null){
            System.out.println("Time " + (System.currentTimeMillis() - ((long) temp)) + " ms");
        }

        System.out.println("Response status: " + response.getStatus());
    }
}

// InterceptorConfiguration.java
@Configuration
public class InterceptorConfiguration implements WebMvcConfigurer {
    public final AuthInterceptor authInterceptor;
    public final LogInterceptor logInterceptor;

    public WebConfiguration(AuthInterceptor authInterceptor, LogInterceptor logInterceptor){
        this.authInterceptor = authInterceptor;
        this.logInterceptor = logInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry){
        registry.addInterceptor(logInterceptor)
                .order(2);
        registry.addInterceptor(authInterceptor)
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/auth/**")
                .order(1);
    }
}

// Student Controller
@RestController
@RequestMapping("/students")
public class Student {
    @GetMapping
    public ResponseEntity<String> greet(){
        return ResponseEntity.status(HttpStatus.OK).body("Hello");
    }
}
// Api controller
@RestController
@RequestMapping("/api")
public class Api{
    @GetMapping
    public ResponseEntity<String> greet(){
        return ResponseEntity.ok("Validation success");
    }
}
```
