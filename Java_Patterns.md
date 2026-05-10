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
| Observer | Behavioral | Notify subscribers automatically when state changes |
| Decorator | Structural | Wrap an object to add behavior without subclassing |
| Adapter | Structural | Make an incompatible interface work with an existing one |

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

## 6. 👀 Observer Pattern

Defines a one-to-many relationship: when the **subject** changes state,
all registered **observers** are notified automatically.
Also known as Publish/Subscribe.

```java
// Observer contract
interface Observer {
    void update(String event, Object data);
}

// Subject contract
interface Subject {
    void subscribe(Observer o);
    void unsubscribe(Observer o);
    void notify(String event, Object data);
}
```

```java
class EventBus implements Subject {
    private final List<Observer> observers = new ArrayList<>();

    @Override public void subscribe(Observer o)   { observers.add(o); }
    @Override public void unsubscribe(Observer o) { observers.remove(o); }

    @Override
    public void notify(String event, Object data) {
        for (Observer o : observers) o.update(event, data);
    }
}
```

```java
class Logger implements Observer {
    @Override public void update(String event, Object data) {
        System.out.println("[LOG] " + event + " → " + data);
    }
}

class EmailAlerter implements Observer {
    @Override public void update(String event, Object data) {
        if ("ORDER_PLACED".equals(event))
            System.out.println("[EMAIL] New order: " + data);
    }
}
```

```java
EventBus bus = new EventBus();
bus.subscribe(new Logger());
bus.subscribe(new EmailAlerter());

bus.notify("ORDER_PLACED", "order#42");
// [LOG]   ORDER_PLACED → order#42
// [EMAIL] New order: order#42

bus.notify("USER_LOGIN", "alice");
// [LOG]   USER_LOGIN → alice
//  (EmailAlerter ignores this event)
```

### Java's Built-in Support

```java
// java.util.Observable (legacy — avoid, class not interface)
// Modern: use PropertyChangeListener or reactive libraries (RxJava, Project Reactor)

// Simple lambda-based observer (when Observer is @FunctionalInterface)
@FunctionalInterface
interface Observer { void update(String event, Object data); }

bus.subscribe((event, data) -> System.out.println("lambda: " + event));
```

---

## 7. 🎁 Decorator Pattern

Attaches new behavior to an object **at runtime** by wrapping it in decorator objects.
Each decorator implements the same interface as the component it wraps.
Decorators can be stacked — the order matters.

```java
interface TextProcessor {
    String process(String text);
}

// Base implementation
class PlainText implements TextProcessor {
    @Override public String process(String text) { return text; }
}
```

```java
// Base decorator — holds a reference to the wrapped component
abstract class TextDecorator implements TextProcessor {
    protected final TextProcessor wrapped;
    TextDecorator(TextProcessor wrapped) { this.wrapped = wrapped; }
}

class TrimDecorator extends TextDecorator {
    TrimDecorator(TextProcessor w) { super(w); }
    @Override public String process(String text) {
        return wrapped.process(text).trim();         // delegate then add behavior
    }
}

class UpperCaseDecorator extends TextDecorator {
    UpperCaseDecorator(TextProcessor w) { super(w); }
    @Override public String process(String text) {
        return wrapped.process(text).toUpperCase();
    }
}

class CensorDecorator extends TextDecorator {
    private final String badWord, replacement;
    CensorDecorator(TextProcessor w, String bad, String rep) {
        super(w); this.badWord = bad; this.replacement = rep;
    }
    @Override public String process(String text) {
        return wrapped.process(text).replace(badWord, replacement);
    }
}
```

```java
// Stack decorators — each wraps the previous
TextProcessor pipeline = new CensorDecorator(
                             new UpperCaseDecorator(
                                 new TrimDecorator(
                                     new PlainText())),
                             "BAD", "***");

pipeline.process("  hello bad world  ");
// PlainText  → "  hello bad world  "
// Trim       → "hello bad world"
// UpperCase  → "HELLO BAD WORLD"
// Censor     → "HELLO *** WORLD"
```

> **Real-world example:** `java.io` streams are decorators.
> `new BufferedReader(new InputStreamReader(new FileInputStream("file.txt")))` —
> each layer wraps the previous and adds behavior (buffering, charset decoding, file reading).

---

## 8. 🔌 Adapter Pattern

Converts the interface of a class into another interface that the client expects.
Lets two incompatible interfaces work together without changing either.

```java
// What the client code expects
interface JsonParser {
    Map<String, Object> parse(String json);
}

// Existing third-party library with a different interface — can't change it
class LegacyXmlParser {
    Document parseXml(String xml) {
        System.out.println("Parsing XML: " + xml);
        return new Document(xml);          // returns a Document, not a Map
    }
}

class Document {
    String content;
    Document(String c) { this.content = c; }
    Map<String, Object> toMap() { return Map.of("content", content); }
}
```

```java
// Adapter — wraps LegacyXmlParser and exposes the JsonParser interface
class XmlToJsonAdapter implements JsonParser {
    private final LegacyXmlParser xmlParser = new LegacyXmlParser();

    @Override
    public Map<String, Object> parse(String input) {
        Document doc = xmlParser.parseXml(input);   // call the incompatible API
        return doc.toMap();                          // convert to expected type
    }
}
```

```java
// Client code only knows about JsonParser — unaware of the XML internals
JsonParser parser = new XmlToJsonAdapter();
Map<String, Object> result = parser.parse("<user><name>Alice</name></user>");
```

### Object Adapter vs Class Adapter

```java
// Object Adapter (above) — wraps an instance via composition ✅ preferred
class ObjectAdapter implements Target {
    private final Adaptee adaptee;
    ObjectAdapter(Adaptee a) { this.adaptee = a; }
    @Override public void request() { adaptee.specificRequest(); }
}

// Class Adapter — extends Adaptee AND implements Target
// Only possible in Java if Adaptee is a class you can extend
class ClassAdapter extends Adaptee implements Target {
    @Override public void request() { specificRequest(); }
}
```

| | Object Adapter | Class Adapter |
|---|---|---|
| Mechanism | Composition | Inheritance |
| Flexibility | Wrap any Adaptee subtype | Tied to one Adaptee class |
| Java support | ✅ Always works | ✅ Only if Adaptee is extendable |

---

## 9. 🏷️ Annotations

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
