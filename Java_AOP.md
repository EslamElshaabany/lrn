# ☕ AOP in Spring Boot — Aspect-Oriented Programming

> Cross-cutting concerns explained: what AOP is, why it exists, how Spring implements it,
> and how `@RestControllerAdvice` fits in as a specialized AOP feature.

---

## 📋 Quick Reference

| Term | One-liner |
|---|---|
| Cross-cutting concern | Logic that spans many classes (logging, auth, validation, error handling) |
| Aspect | The class that holds the cross-cutting logic |
| Advice | The method inside an Aspect — the actual code that runs |
| Join Point | A point in execution where advice *could* run (method call, exception, etc.) |
| Pointcut | An expression that *selects* which join points to intercept |
| Weaving | The process of applying advice to target code |
| Proxy | What Spring actually creates to intercept method calls |
| `@Aspect` | Marks a class as an Aspect |
| `@RestControllerAdvice` | Specialized Aspect that intercepts controller exceptions |

---

## 1. 🤔 The Problem AOP Solves

Imagine you have 50 service methods and you need to:
- Log every method call
- Check authentication before every method
- Handle exceptions consistently

Without AOP you add the same boilerplate to every method:

```java
// BAD — repeated in every method
public Order createOrder(OrderRequest req) {
    log.info("createOrder called");            // logging
    authService.checkAuth();                   // auth check
    try {
        // ... actual business logic ...
    } catch (Exception e) {
        log.error("createOrder failed", e);    // error handling
        throw new AppException(e);
    }
}
```

AOP moves that boilerplate **out** of the methods entirely.
Your business code stays clean; the cross-cutting logic lives in one place.

---

## 2. 🧠 Core Concepts

### Join Point
A moment during program execution where you *could* insert extra behavior.
In Spring AOP, join points are always **method executions**.

```
UserService.createUser(...)  ← join point
OrderService.placeOrder(...) ← join point
```

### Pointcut
A predicate / expression that picks *which* join points to intercept.
Written using AspectJ expression language.

```java
// "intercept any method in any class inside com.example.service"
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceLayer() {}
```

### Advice
The actual code that runs at a join point. Types:

| Advice type | When it runs |
|---|---|
| `@Before` | Before the method executes |
| `@After` | After the method (always, success or exception) |
| `@AfterReturning` | After successful return |
| `@AfterThrowing` | After an exception is thrown |
| `@Around` | Wraps the method — you control when it runs |

### Aspect
A class annotated with `@Aspect` that groups pointcuts + advice together.

---

## 3. ⚙️ How Spring Implements AOP (Proxies)

Spring does **not** modify your bytecode directly.
Instead it wraps your bean in a **proxy** object at startup.

```
Your code calls:  userService.createUser(req)
                         ↓
              Spring Proxy intercepts call
                         ↓
              Advice runs (@Before / @Around)
                         ↓
              Actual UserService.createUser(req)
                         ↓
              Advice runs (@After / @AfterReturning)
                         ↓
              Result returned to your code
```

> **Implication:** AOP only works on Spring-managed beans called from *outside* the bean.
> If a method calls another method on `this` (self-call), the proxy is bypassed and advice won't run.

---

## 4. 🛠️ Minimal Working Example

**Add the dependency** (already included if you use `spring-boot-starter`):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**Write the Aspect:**

```java
@Aspect
@Component
public class LoggingAspect {

    private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);

    // Pointcut: any method in any class inside the service package
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}

    // Advice: runs before every matched method
    @Before("serviceLayer()")
    public void logBefore(JoinPoint jp) {
        log.info("Calling: {}", jp.getSignature().getName());
    }

    // Advice: runs after successful return
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void logAfter(JoinPoint jp, Object result) {
        log.info("Returned from: {} → {}", jp.getSignature().getName(), result);
    }
}
```

---

## 5. 🔀 @Around — The Most Powerful Advice

`@Around` wraps the method completely. You decide when (and whether) to call it.
Use it for timing, transaction-like behavior, or conditional execution.

```java
@Around("serviceLayer()")
public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.currentTimeMillis();

    Object result = pjp.proceed();          // ← actually calls the method

    long duration = System.currentTimeMillis() - start;
    log.info("{} took {}ms", pjp.getSignature().getName(), duration);

    return result;
}
```

> `pjp.proceed()` is the call to the real method. If you forget it, the method never runs.

---

## 6. 🎯 Pointcut Expression Cheat Sheet

```java
// Any method in UserService
execution(* com.example.service.UserService.*(..))

// Any method in any class of the service package
execution(* com.example.service.*.*(..))

// Only public methods, any package
execution(public * *(..))

// Methods whose name starts with "get"
execution(* get*(..))

// Methods annotated with @Transactional
@annotation(org.springframework.transaction.annotation.Transactional)

// Combine with && || !
execution(* com.example.service.*.*(..)) && !execution(* com.example.service.*.find*(..))
```

---

## 7. 🚨 @RestControllerAdvice — AOP for Exception Handling

`@RestControllerAdvice` is a **specialized, pre-built Aspect** that Spring provides.
It intercepts exceptions thrown by `@RestController` methods — you don't write a pointcut yourself.

```
@RestControllerAdvice
      = @ControllerAdvice   (targets all controllers — the pointcut)
      + @ResponseBody       (serialize the return value as JSON)
```

### Why it's AOP

Your controller code throws an exception and stops. Spring intercepts it via proxy,
finds a matching `@ExceptionHandler` method in your advice class, and calls it instead.
Your controller never needs a try-catch.

### Basic Structure

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Handles a specific exception type
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }

    // Handles validation errors (@Valid failures)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult()
                           .getFieldErrors()
                           .stream()
                           .map(e -> e.getField() + ": " + e.getDefaultMessage())
                           .collect(Collectors.joining(", "));
        return new ErrorResponse("VALIDATION_ERROR", message);
    }

    // Catch-all — always add this
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleAll(Exception ex) {
        return new ErrorResponse("INTERNAL_ERROR", "Something went wrong");
    }
}
```

### ErrorResponse DTO

```java
public record ErrorResponse(String code, String message) {}
```

### How exception matching works

Spring checks handlers from **most specific to least specific**.
`ResourceNotFoundException` is matched before `Exception`.
Put your specific handlers first, `Exception.class` last.

---

## 8. 📌 Scope — @RestControllerAdvice vs Custom @Aspect

| Feature | `@RestControllerAdvice` | Custom `@Aspect` |
|---|---|---|
| Target | Controllers only | Any Spring bean |
| Setup | Just write `@ExceptionHandler` | Write pointcut + advice |
| Use for | Global exception handling, model attributes | Logging, timing, auth, caching |
| Exception handling | Yes — its main job | Only via `@AfterThrowing` |

---

## 9. 🗺️ When to Use What

| Scenario | Use |
|---|---|
| Consistent error responses across all endpoints | `@RestControllerAdvice` + `@ExceptionHandler` |
| Log every service method call | `@Aspect` + `@Before` / `@AfterReturning` |
| Measure execution time | `@Aspect` + `@Around` |
| Check a custom annotation before methods run | `@Aspect` + `@Before` + `@annotation()` pointcut |
| Validate input on controller | `@RestControllerAdvice` + `MethodArgumentNotValidException` |
| Retry logic around external calls | `@Aspect` + `@Around` (or use `@Retryable`) |

---

## 10. 💡 Common Pitfalls

### Self-invocation bypass
```java
@Service
public class OrderService {
    public void placeOrder() {
        this.validateOrder();   // ❌ proxy bypassed — @Before on validateOrder won't fire
    }

    @SomeAspectAnnotation
    public void validateOrder() { ... }
}
```
Fix: inject the service into itself, or restructure to avoid internal calls.

### @RestControllerAdvice doesn't catch Filter exceptions
Filters run before Spring MVC — exceptions there bypass `@RestControllerAdvice`.
Handle those with a custom `OncePerRequestFilter` that catches and writes the response.

### Aspect on non-Spring beans
AOP only works on beans managed by Spring. `new MyService()` gets no proxy.

---

## 11. 🔗 Relation to Other Files

| Topic | File |
|---|---|
| Design patterns (Strategy, Decorator, etc.) | `Java_Patterns.md` |
| Spring Boot jargon (IoC, DI, Bean) | `Java_Spring_Glossary.md` |
| OOP fundamentals | `Java_OOP.md` |
