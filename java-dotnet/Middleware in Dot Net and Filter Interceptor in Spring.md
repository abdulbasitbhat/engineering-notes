# Middleware — ASP.NET Core vs Spring Boot (Deep Dive)
--------------------------------------------------------------------

# 1. What is Middleware?

**Middleware = code that runs between request and response**

It sits in the application pipeline and can:
- Read request
- Modify request
- Stop request
- Call next component
- Modify response

Think of middleware as **checkpoints** every request passes through.

Flow:

Client → Middleware → Middleware → Controller → Middleware → Response

Middleware runs:
- On request way in
- On response way out

--------------------------------------------------------------------

# 2. Real-Life Analogy

Airport security.

You enter airport:
1. ID check
2. Security scan
3. Boarding check
4. Gate

Each step:
- Can stop you
- Can modify something
- Can allow forward

That is middleware pipeline.

--------------------------------------------------------------------

# 3. Why Middleware Exists

Without middleware, controllers would handle everything:

- Logging
- Authentication
- Authorization
- Exception handling
- Headers
- CORS
- Routing

Controller becomes messy.

Middleware separates:
**cross-cutting concerns** from business logic.

Controller → business logic  
Middleware → system-level logic  

--------------------------------------------------------------------

# 4. ASP.NET Core Middleware (Most Important)

ASP.NET Core is built around middleware.

Entire request pipeline = middleware chain.

Configured in:

Program.cs

Example:

    var app = builder.Build();

    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();

    app.MapControllers();

    app.Run();

Each Use...() is middleware.

--------------------------------------------------------------------

# 5. How Middleware Works Internally

Each middleware:

1. Receives HttpContext
2. Does some work
3. Calls next middleware
4. Gets response back
5. Can modify response

Flow:

Request →
Middleware A →
Middleware B →
Controller →
Response →
Middleware B →
Middleware A →
Client

So middleware runs both directions.

--------------------------------------------------------------------

# 6. Custom Middleware Example (ASP.NET)

Create logging middleware.

    public class LoggingMiddleware
    {
        private readonly RequestDelegate _next;

        public LoggingMiddleware(RequestDelegate next)
        {
            _next = next;
        }

        public async Task Invoke(HttpContext context)
        {
            Console.WriteLine("Request started: " + context.Request.Path);

            await _next(context); // call next middleware

            Console.WriteLine("Response sent");
        }
    }

Register it in Program.cs:

    app.UseMiddleware<LoggingMiddleware>();

Now every request passes through it.

--------------------------------------------------------------------

# 7. Short-Circuiting Middleware

Middleware can stop pipeline.

Example: authentication check.

    public async Task Invoke(HttpContext context)
    {
        if (!context.Request.Headers.ContainsKey("Authorization"))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Unauthorized");
            return; // STOP pipeline
        }

        await _next(context);
    }

Controller will never execute if auth fails.

This is called:
**short-circuiting middleware**

--------------------------------------------------------------------

# 8. Order Matters (Very Important)

Middleware order = execution order.

Wrong order causes bugs.

Correct:

    app.UseAuthentication();
    app.UseAuthorization();

Wrong:

    app.UseAuthorization();
    app.UseAuthentication();

Authorization runs before user exists → fails.

Always remember:
Middleware pipeline is sequential.

--------------------------------------------------------------------

# 9. Built-in ASP.NET Middleware

Common middleware:

    app.UseExceptionHandler();
    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseRouting();
    app.UseCors();
    app.UseAuthentication();
    app.UseAuthorization();
    app.MapControllers();

These power entire ASP.NET request flow.

--------------------------------------------------------------------

# 10. Middleware vs Controller

| Middleware | Controller |
|-----------|-----------|
| Runs for every request | Runs for specific route |
| Handles cross-cutting concerns | Handles business logic |
| Early in pipeline | Late in pipeline |
| Can stop request | Usually returns response |

Middleware handles:
logging, auth, errors

Controller handles:
business logic

--------------------------------------------------------------------

# 11. Spring Boot Equivalent of Middleware

Spring does not call it middleware directly.

But same concept exists.

Three main equivalents:

1. Filters (closest to middleware)
2. Interceptors
3. AOP (aspects)

--------------------------------------------------------------------

# 12. Spring Boot Filter Example

Filter runs before controller.

    import jakarta.servlet.*;
    import jakarta.servlet.http.HttpServletRequest;
    import java.io.IOException;

    public class LoggingFilter implements Filter {

        @Override
        public void doFilter(ServletRequest request,
                             ServletResponse response,
                             FilterChain chain)
                             throws IOException, ServletException {

            HttpServletRequest req = (HttpServletRequest) request;
            System.out.println("Request: " + req.getRequestURI());

            chain.doFilter(request, response); // call next

            System.out.println("Response sent");
        }
    }

Register filter:

    @Configuration
    public class FilterConfig {

        @Bean
        public FilterRegistrationBean<LoggingFilter> loggingFilter(){
            FilterRegistrationBean<LoggingFilter> bean =
                new FilterRegistrationBean<>();

            bean.setFilter(new LoggingFilter());
            bean.addUrlPatterns("/*");
            return bean;
        }
    }

Now every request passes through filter.

--------------------------------------------------------------------

# 13. Spring Interceptor Example

Runs before and after controller.

    public class AuthInterceptor implements HandlerInterceptor {

        @Override
        public boolean preHandle(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler) throws Exception {

            String token = request.getHeader("Authorization");

            if(token == null){
                response.setStatus(401);
                return false; // stop request
            }

            return true;
        }
    }

Register interceptor:

    @Configuration
    public class WebConfig implements WebMvcConfigurer {

        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new AuthInterceptor());
        }
    }

--------------------------------------------------------------------

# 14. How Middleware Interacts with Controllers & Services

ASP.NET Flow:

Client request  
→ Middleware (logging)  
→ Middleware (auth)  
→ Middleware (routing)  
→ Controller  
→ Service  
→ Repository  
→ Response  
→ Middleware modifies response  
→ Client  

Spring Flow:

Client request  
→ Filter  
→ DispatcherServlet  
→ Interceptor  
→ Controller  
→ Service  
→ Repository  
→ Response  
→ Interceptor  
→ Filter  
→ Client  

Middleware/filter runs BEFORE controller.  
Service layer runs INSIDE controller execution.

--------------------------------------------------------------------

# 15. Interaction Example (ASP.NET)

Logging middleware logs request.

Auth middleware validates user.

Then controller runs.

    app.UseMiddleware<LoggingMiddleware>();
    app.UseMiddleware<AuthMiddleware>();
    app.MapControllers();

Controller:

    [ApiController]
    [Route("orders")]
    public class OrderController : ControllerBase
    {
        private readonly OrderService _service;

        public OrderController(OrderService service)
        {
            _service = service;
        }

        [HttpGet]
        public string Get()
        {
            return _service.GetOrders();
        }
    }

Flow:
Request → Logging → Auth → Controller → Service → Response

--------------------------------------------------------------------

# 16. Mapping Spring vs ASP.NET Middleware

| Concept | Spring Boot | ASP.NET Core |
|--------|-------------|--------------|
| Main pipeline | DispatcherServlet | Middleware pipeline |
| Pre-processing | Filter | Middleware |
| Pre/post controller | Interceptor | Middleware |
| Cross-cutting logic | AOP | Middleware |
| Config location | Config classes | Program.cs |

ASP.NET:
Middleware-first architecture.

Spring:
Servlet + filter + interceptor chain.

--------------------------------------------------------------------

# 17. When to Use Middleware

Use middleware for:

- Logging
- Authentication
- Authorization
- Exception handling
- Request timing
- Headers modification
- Rate limiting
- CORS
- Global validation

Do NOT use middleware for:
Business logic.

That belongs in services/controllers.

--------------------------------------------------------------------

# 18. One-Line Mental Model

Middleware = pipeline components that process request and response before controller and after controller.

They sit between:
Client ↔ Controller ↔ Response

And control everything in between.
