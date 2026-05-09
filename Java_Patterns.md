# ☕ Java Design Patterns & Annotations

> Quick-reference for the most common GoF patterns and Java annotation mechanics.
> OOP fundamentals (inheritance, interfaces, etc.) → see `Java_OOP.md`.

---

## 📋 Quick Reference

| Pattern | Category | One-liner |
|---|---|---|
| Singleton | Creational | One global instance, ever |
| Factory | Creational | Delegate object creation to a method/class |
| Strategy | Behavioral | Swap algorithms at runtime |
| Visitor | Behavioral | Add operations to objects without changing their classes |
| Facade | Structural | Simplified interface over a complex subsystem |

---

## 1. 🏭 Factory Pattern

Delegates object creation to a factory method or class.
Callers don't need to know the concrete type — just ask for the right flavor.

```java
interface Notification {
    void send(String message);
}

class EmailNotification implements Notification {
    @Override public void send(String msg) { System.out.println("Email: " + msg); }
}

class SMSNotification implements Notification {
    @Override public void send(String msg) { System.out.println("SMS: " + msg); }
}

class PushNotification implements Notification {
    @Override public void send(String msg) { System.out.println("Push: " + msg); }
}
```

### Simple Factory (static method)

```java
class NotificationFactory {
    static Notification create(String type) {
        return switch (type) {
            case "email" -> new EmailNotification();
            case "sms"   -> new SMSNotification();
            case "push"  -> new PushNotification();
            default      -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}

Notification n = NotificationFactory.create("email");
n.send("Hello!");   // Email: Hello!
```

### Factory Method Pattern (via inheritance)

Each subclass decides what to create — follows Open/Closed Principle.

```java
abstract class Dialog {
    abstract Notification createNotification();   // factory method

    void notifyUser(String msg) {
        Notification n = createNotification();    // delegates to subclass
        n.send(msg);
    }
}

class MobileDialog extends Dialog {
    @Override Notification createNotification() { return new PushNotification(); }
}

class WebDialog extends Dialog {
    @Override Notification createNotification() { return new EmailNotification(); }
}

Dialog dialog = new MobileDialog();
dialog.notifyUser("Welcome!");   // Push: Welcome!
```

---

## 2. 🔒 Singleton Pattern

Ensures a class has **exactly one instance** and provides a global access point.

```java
class Config {
    private static Config instance;

    private String dbUrl = "jdbc:postgresql://localhost/mydb";

    private Config() { }                  // private constructor — no external new

    static Config getInstance() {
        if (instance == null) {
            instance = new Config();
        }
        return instance;
    }

    String getDbUrl() { return dbUrl; }
}

Config c1 = Config.getInstance();
Config c2 = Config.getInstance();
c1 == c2;   // true — same object
```

### Thread-Safe Singleton (double-checked locking)

```java
class ThreadSafeConfig {
    private static volatile ThreadSafeConfig instance;   // volatile prevents cache issues

    private ThreadSafeConfig() { }

    static ThreadSafeConfig getInstance() {
        if (instance == null) {
            synchronized (ThreadSafeConfig.class) {       // lock only when needed
                if (instance == null) {                   // re-check inside lock
                    instance = new ThreadSafeConfig();
                }
            }
        }
        return instance;
    }
}
```

### Enum Singleton (simplest, thread-safe by design)

```java
enum AppConfig {
    INSTANCE;

    private String env = "production";
    String getEnv() { return env; }
}

AppConfig.INSTANCE.getEnv();   // "production"
```

---

## 3. 🎯 Strategy Pattern

Defines a family of algorithms, encapsulates each one, and makes them interchangeable.
Lets behavior change at runtime without modifying the host class.

```java
@FunctionalInterface
interface SortStrategy {
    void sort(int[] data);
}
```

```java
class BubbleSort implements SortStrategy {
    @Override public void sort(int[] data) {
        // bubble sort logic
        System.out.println("Sorting with bubble sort");
    }
}

class QuickSort implements SortStrategy {
    @Override public void sort(int[] data) {
        Arrays.sort(data);   // shortcut for demo
        System.out.println("Sorting with quick sort");
    }
}
```

```java
class DataProcessor {
    private SortStrategy strategy;

    DataProcessor(SortStrategy strategy) { this.strategy = strategy; }

    void setStrategy(SortStrategy strategy) { this.strategy = strategy; }  // swap at runtime

    void process(int[] data) {
        strategy.sort(data);   // delegates to whichever strategy is active
    }
}

DataProcessor dp = new DataProcessor(new QuickSort());
dp.process(new int[]{3, 1, 2});      // Sorting with quick sort

dp.setStrategy(new BubbleSort());
dp.process(new int[]{3, 1, 2});      // Sorting with bubble sort

// With lambda (because SortStrategy is @FunctionalInterface)
dp.setStrategy(data -> System.out.println("Custom sort"));
```

---

## 4. 👁️ Visitor Pattern

Lets you add new operations to an object hierarchy **without modifying those classes**.
The visitor "visits" each element and performs its operation.

```java
interface Shape {
    void accept(ShapeVisitor visitor);   // dispatch hook
}

class Circle implements Shape {
    double radius;
    Circle(double r) { this.radius = r; }
    @Override public void accept(ShapeVisitor v) { v.visit(this); }
}

class Rectangle implements Shape {
    double w, h;
    Rectangle(double w, double h) { this.w = w; this.h = h; }
    @Override public void accept(ShapeVisitor v) { v.visit(this); }
}
```

```java
interface ShapeVisitor {
    void visit(Circle c);
    void visit(Rectangle r);
}
```

```java
// New operation: area calculation — added WITHOUT touching Shape classes
class AreaVisitor implements ShapeVisitor {
    @Override public void visit(Circle c)    { System.out.println("Area: " + Math.PI * c.radius * c.radius); }
    @Override public void visit(Rectangle r) { System.out.println("Area: " + r.w * r.h); }
}

// New operation: export to SVG
class SvgVisitor implements ShapeVisitor {
    @Override public void visit(Circle c)    { System.out.println("<circle r='" + c.radius + "'/>"); }
    @Override public void visit(Rectangle r) { System.out.println("<rect w='" + r.w + "' h='" + r.h + "'/>"); }
}
```

```java
List<Shape> shapes = List.of(new Circle(5), new Rectangle(3, 4));

ShapeVisitor area = new AreaVisitor();
for (Shape s : shapes) s.accept(area);
// Area: 78.53...
// Area: 12.0

ShapeVisitor svg = new SvgVisitor();
for (Shape s : shapes) s.accept(svg);
// <circle r='5.0'/>
// <rect w='3.0' h='4.0'/>
```

> **Key insight:** `accept()` calls the right `visit()` overload via double dispatch —
> the concrete type of the shape selects which `visit` method runs.

---

## 5. 🏠 Facade Pattern

Provides a **simple interface** to a complex subsystem.
Callers don't need to know about the subsystem's internal components.

```java
// Complex subsystem — each class handles one thing
class VideoDecoder {
    void decode(String file)  { System.out.println("Decoding " + file); }
}

class AudioMixer {
    void mix()                { System.out.println("Mixing audio"); }
}

class BufferManager {
    void allocate(int mb)     { System.out.println("Allocating " + mb + "MB buffer"); }
    void release()            { System.out.println("Releasing buffer"); }
}

class SubtitleRenderer {
    void load(String lang)    { System.out.println("Loading " + lang + " subtitles"); }
}
```

```java
// Facade — one simple method hides all the complexity
class VideoPlayer {
    private final VideoDecoder   decoder  = new VideoDecoder();
    private final AudioMixer     mixer    = new AudioMixer();
    private final BufferManager  buffer   = new BufferManager();
    private final SubtitleRenderer subs   = new SubtitleRenderer();

    void play(String file, String lang) {
        buffer.allocate(256);
        decoder.decode(file);
        mixer.mix();
        subs.load(lang);
        System.out.println("Playing...");
    }

    void stop() {
        buffer.release();
        System.out.println("Stopped.");
    }
}
```

```java
// Caller only sees the facade — subsystem is invisible
VideoPlayer player = new VideoPlayer();
player.play("movie.mp4", "en");
player.stop();
```

---

## 6. 🏷️ Annotations

Annotations are **metadata** attached to code elements (classes, methods, fields, params).
They don't affect logic directly — but processors, frameworks, and the JVM read them.

### Built-in Annotations

```java
class Base {
    void method() { }
    @Deprecated void oldMethod() { }     // mark as obsolete — IDE warns on use
}

class Child extends Base {
    @Override                            // tell compiler: this must override a parent method
    void method() { }

    @SuppressWarnings("unchecked")       // silence a specific compiler warning
    void rawTypes() {
        List list = new ArrayList();     // raw type warning suppressed
    }
}
```

### Custom Annotation

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)     // available via reflection at runtime
@Target(ElementType.METHOD)             // can only be placed on methods
public @interface Timed {
    String label() default "";           // annotation element with default
}
```

**`@Retention` values:**

| Value | Meaning |
|---|---|
| `SOURCE` | Compile-time only — stripped from .class (e.g. `@Override`) |
| `CLASS` | In .class but not loaded at runtime (default) |
| `RUNTIME` | Available via reflection at runtime |

**`@Target` values:** `TYPE`, `METHOD`, `FIELD`, `PARAMETER`, `CONSTRUCTOR`, `LOCAL_VARIABLE`, `PACKAGE`

### Process Annotations via Reflection

```java
class OrderService {
    @Timed(label = "create-order")
    void createOrder() { System.out.println("Creating order"); }

    @Timed
    void cancelOrder() { System.out.println("Cancelling order"); }
}
```

```java
for (Method method : OrderService.class.getDeclaredMethods()) {
    if (method.isAnnotationPresent(Timed.class)) {
        Timed t = method.getAnnotation(Timed.class);
        System.out.println("Timed method: " + method.getName()
                           + " | label: '" + t.label() + "'");
    }
}
// Timed method: createOrder | label: 'create-order'
// Timed method: cancelOrder | label: ''
```

### Marker Annotation (no elements)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Singleton { }      // presence alone carries the meaning

@Singleton
class AppConfig { }

boolean isSingleton = AppConfig.class.isAnnotationPresent(Singleton.class); // true
```

### Repeatable Annotation (Java 8+)

```java
@Repeatable(Roles.class)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Role { String value(); }

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Roles { Role[] value(); }    // container annotation

@Role("admin")
@Role("editor")
class UserController { }

Role[] roles = UserController.class.getAnnotationsByType(Role.class);
// roles[0].value() = "admin"
// roles[1].value() = "editor"
```
