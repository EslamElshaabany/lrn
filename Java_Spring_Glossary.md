# ☕ Java & Spring Boot — Terms & Buzzwords

> A plain-English glossary of the jargon you'll hear constantly.
> Grouped by theme so related terms build on each other.

---

## 📋 Quick Lookup

| Term | One-liner |
|---|---|
| POJO | Plain class — no framework dependencies |
| JavaBean | POJO + no-arg constructor + getters/setters + `Serializable` |
| DTO | Object used to carry data between layers |
| Entity | Class mapped to a database table |
| DAO | Object that talks directly to the database |
| Repository | DAO abstraction — Spring Data style |
| Spring Bean | Object whose lifecycle is managed by the Spring container |
| IoC | Framework controls object creation, not your code |
| DI | Dependencies are injected in, not looked up |
| AOP | Cross-cutting concerns (logging, auth) woven in without touching business code |
| Auto-configuration | Spring Boot wires common setup for you based on what's on the classpath |

---

## 1. 🧱 Core Java Object Types

### POJO — Plain Old Java Object

A class with no mandatory superclass, no required interfaces, no framework annotations.
The term exists to contrast with older patterns (EJBs) that forced heavy inheritance.

```java
public class User {
    private String name;
    private int    age;
    // any fields, any methods — no rules imposed
}
```

### JavaBean

A POJO that follows three additional conventions:
1. Public no-arg constructor
2. Private fields exposed via `getX()` / `setX()` accessors
3. Implements `Serializable`

```java
public class UserBean implements Serializable {
    private String name;
    private int    age;

    public UserBean() { }                      // ① no-arg constructor

    public String getName()        { return name; }   // ② getter
    public void   setName(String n){ this.name = n; } // ② setter
    public int    getAge()         { return age; }
    public void   setAge(int a)    { this.age = a; }
}
```

> Frameworks like JSF, Jackson, and Spring MVC rely on the JavaBean convention
> to map JSON, form data, or XML to objects automatically.

### DTO — Data Transfer Object

An object whose only job is to carry data between layers (e.g., controller → service, or API response).
No business logic — just fields, constructor, and accessors.

```java
// What the API receives
public record CreateUserRequest(String name, String email) { }

// What the API returns
public record UserResponse(Long id, String name) { }
```

> DTOs decouple your API shape from your internal domain/entity model.
> If the DB schema changes, the DTO stays stable and vice versa.

### VO — Value Object

An object defined entirely by its value, not its identity.
Two VOs with the same data are considered equal. Usually immutable.

```java
record Money(double amount, String currency) { }

new Money(10, "USD").equals(new Money(10, "USD"));  // true — same value, equal objects
```

### Entity

A class that maps directly to a database table row.
In JPA/Hibernate, annotated with `@Entity`.

```java
@Entity
@Table(name = "users")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long   id;
    private String name;
    private String email;
}
```

### DAO — Data Access Object

A class that encapsulates all database operations for a specific entity.
Separates persistence logic from business logic.

```java
public class UserDao {
    public User findById(Long id) { /* raw JDBC or SQL */ }
    public void save(User user)   { /* insert/update */ }
    public void delete(Long id)   { /* delete */ }
}
```

> Spring Data's `Repository` is the modern replacement — it generates the DAO for you.

---

## 2. 🌱 Spring Core Concepts

### IoC — Inversion of Control

Instead of your code creating its own dependencies (`new Service()`),
the **framework** creates and manages them.
You describe what you need; the container figures out how to build it.

```java
// Without IoC — you control creation
public class OrderController {
    private OrderService service = new OrderService(new EmailSender(), new PaymentGateway());
}

// With IoC — the container controls creation
public class OrderController {
    private final OrderService service;  // container injects this for you
    ...
}
```

### DI — Dependency Injection

The mechanism IoC uses to deliver dependencies.
Spring supports three styles:

```java
@Service
public class OrderService {

    // ── Constructor injection (preferred) ─────────────────────
    private final PaymentGateway payment;
    private final EmailSender    email;

    public OrderService(PaymentGateway payment, EmailSender email) {
        this.payment = payment;
        this.email   = email;
    }

    // ── Setter injection ──────────────────────────────────────
    private NotificationService notifier;

    @Autowired
    public void setNotifier(NotificationService n) { this.notifier = n; }

    // ── Field injection (convenient but avoid — hard to test) ─
    @Autowired
    private AuditLogger logger;
}
```

> Constructor injection is the best practice: dependencies are explicit,
> the class is always in a valid state, and it's easy to test without Spring.

### Spring Bean

Any object whose lifecycle (creation, wiring, destruction) is managed by the Spring **ApplicationContext**.
Beans are singletons by default — one shared instance per context.

```java
@Component          // registers the class as a bean
public class EmailSender { ... }

@Service            // alias for @Component — signals "business logic layer"
public class UserService { ... }

@Repository         // alias for @Component — signals "data access layer"
public class UserRepository { ... }

@RestController     // alias for @Component + @ResponseBody — signals "web layer"
public class UserController { ... }
```

### ApplicationContext vs BeanFactory

| | `BeanFactory` | `ApplicationContext` |
|---|---|---|
| Bean creation | Lazy (on first request) | Eager (at startup) |
| Features | Basic DI only | + Events, AOP, i18n, resource loading |
| Use | Rarely directly | Always — it's what Spring Boot gives you |

```java
// You rarely interact with the context directly, but you can:
@Autowired ApplicationContext ctx;
UserService svc = ctx.getBean(UserService.class);
```

### Bean Scopes

```java
@Component @Scope("singleton")  // default — one instance for the whole app
class SharedCache { }

@Component @Scope("prototype")  // new instance every time it's injected or requested
class RequestParser { }

@Component @Scope("request")    // one instance per HTTP request (web apps)
class RequestContext { }

@Component @Scope("session")    // one instance per HTTP session (web apps)
class UserSession { }
```

### AOP — Aspect-Oriented Programming

Separates **cross-cutting concerns** (logging, security, transactions, timing)
from business logic by "weaving" extra behavior around method calls.

```
      ┌─────────────────────────────────────────┐
      │  @Around / @Before / @After (the aspect) │
      │  ┌─────────────────────────────────────┐ │
      │  │   actual method (the join point)    │ │
      │  └─────────────────────────────────────┘ │
      └─────────────────────────────────────────┘
```

```java
@Aspect @Component
public class TimingAspect {

    @Around("@annotation(Timed)")               // pointcut — which methods to intercept
    public Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start  = System.currentTimeMillis();
        Object result = pjp.proceed();           // call the real method
        long elapsed = System.currentTimeMillis() - start;
        System.out.println(pjp.getSignature() + " took " + elapsed + "ms");
        return result;
    }
}
```

| Term | Meaning |
|---|---|
| **Aspect** | The class containing cross-cutting logic |
| **Advice** | The action (`@Before`, `@After`, `@Around`, `@AfterThrowing`) |
| **Pointcut** | Expression that selects which methods to intercept |
| **Join point** | The actual method execution being intercepted |
| **Weaving** | The process of applying aspects to target objects |

---

## 3. 🚀 Spring Boot Specifics

### Auto-configuration

Spring Boot reads the classpath and automatically wires common infrastructure.
Add `spring-boot-starter-data-jpa` → it configures a `DataSource`, `EntityManagerFactory`,
and transaction manager without any XML or `@Bean` definitions.

```java
// This single annotation triggers component scan + auto-config + @Configuration
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
// Equivalent to:
// @Configuration + @EnableAutoConfiguration + @ComponentScan
```

### Starter Dependencies

Curated dependency bundles — one import pulls in everything needed for a feature.

| Starter | What it brings |
|---|---|
| `spring-boot-starter-web` | Spring MVC, Tomcat, Jackson |
| `spring-boot-starter-data-jpa` | Spring Data JPA, Hibernate, HikariCP |
| `spring-boot-starter-security` | Spring Security |
| `spring-boot-starter-test` | JUnit 5, Mockito, AssertJ |
| `spring-boot-starter-actuator` | Health, metrics, info endpoints |

### Profiles

Named configurations for different environments.
Only beans and properties matching the active profile are loaded.

```java
@Profile("prod")
@Service
class ProdEmailSender implements EmailSender { ... }   // only active in prod

@Profile("dev")
@Service
class FakeEmailSender implements EmailSender { ... }   // only active in dev
```

```yaml
# application.yml
spring:
  profiles:
    active: dev          # set active profile

---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://prod-db/app
```

### Actuator

Exposes runtime information about your app over HTTP endpoints.

```
GET /actuator/health    → { "status": "UP" }
GET /actuator/metrics   → JVM memory, HTTP request counts, etc.
GET /actuator/env       → all configuration properties
GET /actuator/beans     → every Spring bean in the context
GET /actuator/mappings  → all @RequestMapping routes
```

---

## 4. 🗄️ Spring Data & JPA Terms

### Repository Hierarchy

```
Repository<T, ID>                      (marker interface)
  └── CrudRepository<T, ID>            (save, findById, findAll, delete)
        └── PagingAndSortingRepository  (+ pagination & sorting)
              └── JpaRepository<T, ID>  (+ flush, saveAll, batch ops) ← use this
```

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Spring Data generates the SQL from the method name
    List<User> findByEmail(String email);
    List<User> findByAgeGreaterThan(int age);
    Optional<User> findByNameAndEmail(String name, String email);

    // Custom query when method name becomes unwieldy
    @Query("SELECT u FROM User u WHERE u.createdAt > :date")
    List<User> findRecentUsers(@Param("date") LocalDate date);
}
```

### JPA / Hibernate Terms

| Term | Meaning |
|---|---|
| **ORM** | Maps Java objects ↔ database tables |
| **Entity** | A class annotated `@Entity` — maps to a table |
| **EntityManager** | JPA API for CRUD + queries (Hibernate implements it) |
| **Session** | Hibernate's name for EntityManager |
| **Persistence Context** | The in-memory cache of managed entities for a transaction |
| **Lazy loading** | Related data is fetched only when accessed |
| **Eager loading** | Related data is fetched immediately with the parent query |
| **N+1 problem** | Lazy loading triggers N extra queries for N related records |
| **`@Transactional`** | Wraps a method in a DB transaction — rollback on exception |
| **JPQL** | JPA's object-oriented query language (works on entities, not tables) |

```java
@Service
public class OrderService {

    @Transactional                          // begin tx before, commit after, rollback on exception
    public void placeOrder(Order order) {
        orderRepo.save(order);
        inventoryService.deduct(order);     // if this throws, the save above is rolled back
    }

    @Transactional(readOnly = true)         // hint for optimizations — no dirty checking
    public List<Order> getOrders(Long userId) {
        return orderRepo.findByUserId(userId);
    }
}
```

---

## 5. 🌐 Spring MVC Terms

```java
@RestController                             // = @Controller + @ResponseBody (returns JSON)
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")                    // GET  /api/users/{id}
    public UserResponse getUser(@PathVariable Long id) { ... }

    @PostMapping                            // POST /api/users
    public UserResponse create(@RequestBody @Valid CreateUserRequest req) { ... }

    @PutMapping("/{id}")                    // PUT  /api/users/{id}
    public UserResponse update(@PathVariable Long id, @RequestBody UpdateUserRequest req) { ... }

    @DeleteMapping("/{id}")                 // DELETE /api/users/{id}
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id) { ... }
}
```

| Annotation | Role |
|---|---|
| `@PathVariable` | Extract `{id}` from the URL path |
| `@RequestParam` | Extract `?page=2` from query string |
| `@RequestBody` | Deserialize JSON request body into an object |
| `@ResponseBody` | Serialize return value to JSON (included in `@RestController`) |
| `@Valid` | Trigger Bean Validation on the argument |
| `@ResponseStatus` | Set the HTTP status code for the response |
| `@ExceptionHandler` | Handle a specific exception within the controller |

---

## 6. 🔧 Miscellaneous Terms

| Term | Meaning |
|---|---|
| **Classpath** | The list of directories/JARs the JVM searches for `.class` files |
| **JAR** | Java Archive — a zipped bundle of compiled `.class` files + metadata |
| **WAR** | Web Application Archive — like a JAR but for servlet containers (Tomcat, Jetty) |
| **Fat JAR / Uber JAR** | A single JAR containing the app + all its dependencies — what `mvn package` produces in Spring Boot |
| **Serialization** | Converting an object to bytes (for storage or network transfer) |
| **Deserialization** | Converting bytes back to an object |
| **Proxy** | A generated wrapper around a bean that intercepts method calls (how AOP and `@Transactional` work) |
| **Classpath scanning** | Spring scanning packages for `@Component`-annotated classes to register as beans |
| **Convention over configuration** | Sensible defaults so you only configure when you deviate |
| **Hot reload** | Restarting only changed classes without a full JVM restart (spring-boot-devtools) |
| **`@Value`** | Inject a property value from `application.properties` into a field |
| **`@ConfigurationProperties`** | Bind a whole block of properties to a typed class |

```java
@Value("${app.timeout:30}")                  // inject — default 30 if not set
private int timeout;

@ConfigurationProperties(prefix = "app.mail")
@Component
public class MailConfig {
    private String host;
    private int    port;
    private String username;
    // getters + setters — bound to app.mail.host, app.mail.port, etc.
}
```
