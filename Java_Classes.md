# ☕ Java Classes — Pre-OOP Fundamentals

> Everything about how a class is built before inheritance and polymorphism enter the picture.
> Covers: access modifiers, constructors, initializer blocks, `this`, `static`, `final`,
> getters/setters, `toString`/`equals`/`hashCode`, nested classes, and enums.

---

## 📋 Quick Reference

| Topic | Keyword / Concept | Key Rule |
|---|---|---|
| Access modifiers | `public` `protected` `private` (none) | Control visibility across packages/classes |
| Constructor | same name as class, no return type | Called once at object creation |
| Constructor chaining | `this(...)` | Must be the first statement |
| Instance initializer | `{ }` block in class body | Runs before constructor body |
| Static initializer | `static { }` block | Runs once when class is loaded |
| `this` | refers to current instance | Access fields, chain constructors |
| `static` | class-level member | Shared across all instances |
| `final` field | `final` | Must be assigned once — in declaration, initializer, or constructor |
| Immutable class | `final` class + `final` fields + no setters | Safe to share without defensive copies |

---

## 1. 🔐 Access Modifiers

Control who can see a class, field, method, or constructor.

| Modifier | Same Class | Same Package | Subclass | Everywhere |
|---|---|---|---|---|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| *(package-private)* | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

```java
public class BankAccount {
    public    String  owner;      // visible everywhere
    protected double  balance;    // visible to subclasses and same package
              int     txCount;    // package-private (no keyword) — same package only
    private   String  pin;        // this class only

    private double internalFee() { return 1.5; }   // helper — not part of public API
    public  void   deposit(double amount) { balance += amount; }
}
```

> **Rule of thumb:** default to `private` for fields, expose only what callers need.
> Widen access only when there's a reason.

---

## 2. 🏗️ Constructors

A constructor initializes a new object. It has **no return type** and the same name as the class.

```java
class Point {
    int x;
    int y;

    // ── No-arg constructor ────────────────────────────────────
    Point() {
        this.x = 0;
        this.y = 0;
    }

    // ── Parameterized constructor ─────────────────────────────
    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // ── Copy constructor ──────────────────────────────────────
    Point(Point other) {
        this.x = other.x;
        this.y = other.y;
    }
}

Point origin = new Point();          // (0, 0)
Point p      = new Point(3, 4);      // (3, 4)
Point copy   = new Point(p);         // (3, 4) — independent copy
```

### Default Constructor

If you write **no constructors at all**, the compiler inserts a no-arg constructor silently.
The moment you add any constructor, the implicit one disappears.

```java
class Empty { }                      // compiler adds: Empty() { super(); }

class HasParam {
    HasParam(int x) { }
    // new HasParam() would now be a compile error — no no-arg constructor
}
```

### Constructor Chaining with `this(...)`

Delegate to another constructor in the **same class** to avoid duplicating initialization logic.
`this(...)` must be the **first statement** in the constructor body.

```java
class Rectangle {
    int width, height, border;

    Rectangle(int width, int height, int border) {
        this.width  = width;
        this.height = height;
        this.border = border;
    }

    Rectangle(int width, int height) {
        this(width, height, 1);      // delegate — no border defaults to 1
    }

    Rectangle(int side) {
        this(side, side);            // square — delegates up the chain
    }
}
```

---

## 3. ⚡ Initializer Blocks

### Instance Initializer Block

Runs **every time an object is created**, just before the constructor body.
Useful when multiple constructors share the same setup.

```java
class Logger {
    String id;
    long   createdAt;

    {   // instance initializer — runs before every constructor
        createdAt = System.currentTimeMillis();
        System.out.println("Logger created");
    }

    Logger()           { this.id = "default"; }
    Logger(String id)  { this.id = id; }
}

// Both constructors automatically get the initializer block's logic.
```

### Static Initializer Block

Runs **once** when the class is first loaded by the JVM.
Used for complex static field setup that can't be done in a one-liner.

```java
class Config {
    static final Map<String, String> DEFAULTS;

    static {                         // runs once when Config.class is loaded
        DEFAULTS = new HashMap<>();
        DEFAULTS.put("timeout",  "30");
        DEFAULTS.put("retries",  "3");
        DEFAULTS.put("logLevel", "INFO");
    }
}
```

### Execution Order

```java
class Demo {
    static int staticField = 1;

    static { System.out.println("1. static initializer"); }  // ① class load

    int instanceField = 2;

    { System.out.println("2. instance initializer"); }       // ② each new

    Demo() { System.out.println("3. constructor"); }         // ③ each new

    // Output for: new Demo(); new Demo();
    // 1. static initializer
    // 2. instance initializer
    // 3. constructor
    // 2. instance initializer
    // 3. constructor
}
```

---

## 4. 🪄 The `this` Keyword

`this` refers to the current instance. Three uses:

```java
class Circle {
    double radius;
    String color;

    Circle(double radius, String color) {
        this.radius = radius;        // ① disambiguate field vs param with same name
        this.color  = color;
    }

    Circle scale(double factor) {
        this.radius *= factor;
        return this;                 // ② return current instance (fluent/builder style)
    }

    Circle(double radius) {
        this(radius, "black");       // ③ call another constructor in this class
    }
}

// Fluent chaining via return this
new Circle(5).scale(2).scale(0.5);
```

---

## 5. ⚙️ The `static` Keyword

`static` members belong to the **class itself**, not to any instance.
All instances share the same static field; static methods have no `this`.

### Static Fields

```java
class Counter {
    static int total = 0;   // shared across all Counter instances
    int id;

    Counter() {
        total++;
        this.id = total;
    }
}

Counter a = new Counter();  // total = 1, a.id = 1
Counter b = new Counter();  // total = 2, b.id = 2

Counter.total;              // ✅ access via class name (preferred)
a.total;                    // works but misleading — looks instance-specific
```

### Static Methods

```java
class MathUtils {
    static int square(int x) { return x * x; }

    // static methods CANNOT access instance fields or call instance methods
    // static methods CANNOT use `this`
}

MathUtils.square(5);   // 25 — no object needed
```

### Static Nested Class

A nested class marked `static` has no reference to the outer instance.

```java
class Outer {
    static class Inner {
        void hello() { System.out.println("static nested"); }
    }
}

Outer.Inner i = new Outer.Inner();   // no Outer instance needed
i.hello();
```

---

## 6. 🔒 The `final` Keyword

### `final` Field — Assigned Once

```java
class Config {
    final int MAX_RETRIES = 3;          // assigned at declaration

    final String host;
    Config(String host) {
        this.host = host;               // also valid: assign in constructor
    }

    // this.host = "other";             // ❌ compile error — already assigned
}
```

### `final` + `static` = Constant

```java
class Units {
    static final double PI      = 3.14159;
    static final int    TIMEOUT = 30;
}

Units.PI;     // access without an instance
```

### `final` Method — Cannot Override

```java
class Base {
    final void lock() { System.out.println("locked behavior"); }
}

class Child extends Base {
    // void lock() { }   // ❌ compile error
}
```

### `final` Class — Cannot Extend

```java
final class Immutable { }
// class Sub extends Immutable { }   // ❌ compile error
// String, Integer, and all wrapper types are declared final in the JDK
```

### Immutable Class Recipe

```java
public final class Money {                     // ① final class
    private final double amount;               // ② final fields
    private final String currency;             // ③ no setters

    public Money(double amount, String currency) {
        this.amount   = amount;
        this.currency = currency;
    }

    public double getAmount()   { return amount; }
    public String getCurrency() { return currency; }

    public Money add(Money other) {            // ④ return new instance instead of mutating
        if (!this.currency.equals(other.currency)) throw new IllegalArgumentException("Currency mismatch");
        return new Money(this.amount + other.amount, this.currency);
    }
}
```

---

## 7. 🔑 Getters & Setters

Encapsulate private fields behind controlled read/write access.

```java
class Person {
    private String name;
    private int    age;

    public String getName() { return name; }
    public void   setName(String name) {
        if (name == null || name.isBlank()) throw new IllegalArgumentException("name required");
        this.name = name;
    }

    public int  getAge() { return age; }
    public void setAge(int age) {
        if (age < 0 || age > 150) throw new IllegalArgumentException("invalid age");
        this.age = age;
    }
}
```

> For `boolean` fields, use `is` instead of `get`: `isActive()`, `isEmpty()`.

### Record — Auto-generated Accessors (Java 16+)

```java
record Point(int x, int y) { }         // fields, constructor, accessors, equals, hashCode, toString — all free

Point p = new Point(3, 4);
p.x();                                  // 3  (no "get" prefix in records)
p.y();                                  // 4
```

---

## 8. 🖨️ `toString`, `equals`, `hashCode`

Every class inherits these from `Object`. Override them to get meaningful behavior.

### `toString` — Readable Representation

```java
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }

    @Override
    public String toString() {
        return "Point(" + x + ", " + y + ")";
    }
}

System.out.println(new Point(3, 4));   // Point(3, 4)  — println calls toString automatically
```

### `equals` — Value Equality

```java
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;                  // same reference
        if (!(obj instanceof Point other)) return false;
        return this.x == other.x && this.y == other.y;
    }
}

new Point(1, 2).equals(new Point(1, 2));   // true  (content)
new Point(1, 2) == new Point(1, 2);        // false (references)
```

### `hashCode` — Must Be Consistent with `equals`

**Contract:** if `a.equals(b)` is `true`, then `a.hashCode() == b.hashCode()` must also be `true`.
Violating this breaks `HashMap`, `HashSet`, and any hash-based collection.

```java
@Override
public int hashCode() {
    return Objects.hash(x, y);   // combines fields into a well-distributed hash
}
```

### All Three Together

```java
class Point {
    final int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }

    @Override public String  toString()         { return "Point(" + x + ", " + y + ")"; }
    @Override public boolean equals(Object obj) {
        if (!(obj instanceof Point o)) return false;
        return x == o.x && y == o.y;
    }
    @Override public int     hashCode()         { return Objects.hash(x, y); }
}
```

---

## 9. 🪆 Nested & Inner Classes

### Static Nested Class

No access to outer instance. Useful for grouping helper types.

```java
class LinkedList {
    static class Node {                  // accessed as LinkedList.Node
        int  val;
        Node next;
        Node(int val) { this.val = val; }
    }

    Node head;
}
```

### Inner Class (Non-static)

Has an implicit reference to the enclosing instance.

```java
class Outer {
    int value = 10;

    class Inner {
        void print() {
            System.out.println(Outer.this.value);  // accesses outer instance's field
        }
    }
}

Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();   // requires an Outer instance
inner.print();   // 10
```

### Anonymous Class

One-off implementation of an interface or abstract class, defined inline.

```java
Runnable r = new Runnable() {
    @Override
    public void run() { System.out.println("anonymous class"); }
};
r.run();

// Modern equivalent with lambda (when interface is @FunctionalInterface)
Runnable r2 = () -> System.out.println("lambda");
```

### Local Class

Defined inside a method — scoped to that method only.

```java
void process() {
    class LocalHelper {
        void help() { System.out.println("local class"); }
    }
    new LocalHelper().help();
}
```

---

## 10. 📦 Enums

An enum is a class with a fixed set of named constants. Each constant is a singleton instance.

```java
enum Day {
    MON, TUE, WED, THU, FRI, SAT, SUN;

    boolean isWeekend() {
        return this == SAT || this == SUN;
    }
}

Day d = Day.FRI;
d.isWeekend();         // false
d.name();              // "FRI"
d.ordinal();           // 4  (zero-indexed)
Day.valueOf("MON");    // Day.MON
Day.values();          // Day[] — all constants in declaration order
```

### Enum with Fields & Constructor

```java
enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    EARTH  (5.976e+24, 6.37814e6),
    MARS   (6.421e+23, 3.3972e6);

    final double mass;
    final double radius;

    Planet(double mass, double radius) {       // constructor is private implicitly
        this.mass   = mass;
        this.radius = radius;
    }

    double surfaceGravity() {
        final double G = 6.67300E-11;
        return G * mass / (radius * radius);
    }
}

Planet.EARTH.surfaceGravity();   // ~9.8
```

### Enum in Switch

```java
Day today = Day.WED;

String type = switch (today) {
    case MON, TUE, WED, THU, FRI -> "Weekday";
    case SAT, SUN                 -> "Weekend";
};
```

### Enum Implementing an Interface

```java
interface Describable { String describe(); }

enum Status implements Describable {
    PENDING  { @Override public String describe() { return "Waiting to start"; } },
    RUNNING  { @Override public String describe() { return "In progress";      } },
    DONE     { @Override public String describe() { return "Completed";        } };
}

Status.RUNNING.describe();   // "In progress"
```
