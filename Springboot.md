## 🔄 Lifecycle of @PreAuthorize and StreamingResponseBody
✅ What happens first:
Spring's security filters (like FilterSecurityInterceptor) and @PreAuthorize evaluate before your controller method is invoked.

If the user lacks permission, the method is never entered.

⚠️ However, in your case:
Your controller returns a StreamingResponseBody, which defers actual logic execution (like reading from S3 and writing to the response) until after the method has already returned a ResponseEntity.

That means:

Security is checked → ✅ access is granted

Spring MVC calls your controller → ✅ you return a StreamingResponseBody

Spring commits headers → ❗ (this is key — headers are flushed)

Spring starts executing the lambda in StreamingResponseBody

Something goes wrong inside that lambda (e.g. exception)

Spring Security tries to handle the exception, but the response is already committed → 💥 you get:

```
Client HTTP → Servlet Container
     │
     ▼  (DispatcherType.REQUEST)
[Filter chain: JwtRequestFilter runs → SecurityContextHolder populated]
     │
     ▼
Controller returns StreamingResponseBody
     │
     └─> Container starts async, frees servlet thread
          │
          ▼  (DispatcherType.ASYNC, on background thread)
        [Filter chain: JwtRequestFilter does NOT run by default]
          │
        Controller’s StreamingResponseBody writes bytes
          │
        Headers auto‐flushed → response committed
          │
        Spring Security sees no auth → throws AccessDenied
          │
        ExceptionTranslationFilter tries to send 401 → fails (already committed)

```




## DispatcherServlet
Simply put, in the Front Controller design pattern, a single controller is responsible for directing incoming HttpRequests to all of an application’s other controllers and handlers.
Spring’s DispatcherServlet implements this pattern and is, therefore, responsible for correctly coordinating the HttpRequests to their right handlers.

In Spring MVC, the DispatcherServlet is the “front-controller” — the single entry point for all incoming HTTP requests to your web application. Its job is not to actually execute your business logic, but rather to orchestrate and route each request on to the right place (your controllers, handlers, views, etc.).

Here’s the high-level flow when a request comes in:

Incoming request → DispatcherServlet
The servlet container (Tomcat, Jetty, etc.) sees that the URL matches your DispatcherServlet’s mapping (e.g. /* or /app/*), and hands the HttpServletRequest (and HttpServletResponse) over to it.

Find the right handler → HandlerMapping
The DispatcherServlet asks one or more HandlerMapping beans, “Hey, who can handle a GET to /orders/123?” Under the covers, Spring has registered a RequestMappingHandlerMapping that has scanned all your @Controller (or @RestController) beans and recorded which methods are annotated with @RequestMapping, @GetMapping, etc., and what URL path they handle.

“Delegate” the request → HandlerAdapter
Once a handler (e.g. a particular controller method) is chosen, the DispatcherServlet needs a way to invoke it without knowing the details of your method signature. That’s where HandlerAdapter comes in. You can think of a HandlerAdapter as a little bridge or adapter that knows:

How to take the raw HttpServletRequest and map path-variables, query parameters, request bodies, etc., into the Java method’s parameters.

How to call the method (via reflection).

How to interpret its return value (e.g. a ModelAndView, or an @ResponseBody object).

In most modern Spring MVC apps you’ll use the built-in RequestMappingHandlerAdapter, which is wired up automatically when you use the @EnableWebMvc or Spring Boot’s MVC auto-configuration.

Handler execution → Controller method runs
Your controller method runs, does whatever business logic it needs (fetching data, updating state, etc.), and returns something like:

A ModelAndView (view name + model data), or

A Java object (e.g. an Order), which Spring serializes to JSON/XML if you’ve annotated it with @ResponseBody or you’re in a @RestController.

Render the response → ViewResolver & View
If you returned a view name, the DispatcherServlet asks a ViewResolver (e.g. a Thymeleaf or JSP resolver) to turn that name into an actual View object, then it renders it with the model data into HTML. If you returned an @ResponseBody result, Spring skips straight to the HttpMessageConverter machinery to write JSON/XML to the response.

What does “delegate the request” mean?
When we say DispatcherServlet delegates the request, we mean:

It hands off responsibility for processing to another component (a handler/controller via a HandlerAdapter), rather than trying to process everything itself.

DispatcherServlet: “I’ve got an incoming HTTP call, but I’m not the one who knows about your business logic. Let me find something that does.”

HandlerMapping: “Here’s the controller method best suited for this URL + HTTP method.”

HandlerAdapter: “Here’s how to take the servlet request, invoke that controller method, and turn its return into a response.”

By separating concerns this way, Spring MVC keeps your controllers super-focused on application logic (annotated methods), while the DispatcherServlet and its collaborators handle all the plumbing of parameter binding, method invocation, view resolution, and so on.

Quick analogy
Imagine a busy office reception desk:

The receptionist (DispatcherServlet) greets every visitor.

They look at the visitor’s name tag (URL + HTTP method) and consult a directory (HandlerMapping) to find the right department.

They pick up the phone (HandlerAdapter) and transfer the caller to the department extension (your controller method).

The department does its work and reports back.

Finally, the receptionist delivers the response (renders the view or writes out the JSON).

That hand-off from receptionist → department is exactly what we mean by “delegating the request.”







