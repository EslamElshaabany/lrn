# ☕ Java OOP Cheat Sheet

> Core object-oriented concepts: inheritance, interfaces, overriding, and more.
> Design patterns and annotations → see `Java_Patterns.md`.

---

## 📋 Quick Reference

| Concept | Keyword | Key Rule |
|---|---|---|
| Inheritance | `extends` | Single parent only |
| Abstract class | `abstract` | Can't be instantiated; mix of abstract + concrete |
| Interface | `implements` | Multiple allowed; all methods `public` by default |
| Override | `@Override` | Same signature, different body |
| Type check | `instanceof` | Returns `boolean`; pattern match in Java 16+ |
| Iterator | `Iterator<T>` | `hasNext()` / `next()` / `remove()` |
| Exception | `try/catch/finally` | Checked must be declared; unchecked needn't be |
| Reflection | `Class<?>` | Inspect and invoke at runtime |

---

## 1. 🧬 Inheritance

A subclass **extends** a parent class to inherit its fields and methods.
Java allows only **single inheritance** for classes (but multiple via interfaces).

```java
class Animal {
    String name;

    Animal(String name) { this.name = name; }

    void speak() { System.out.println(name + " makes a sound"); }
}

class Dog extends Animal {
    String breed;

    Dog(String name, String breed) {
        super(name);        // call parent constructor — must be first line
        this.breed = breed;
    }

    @Override
    void speak() { System.out.println(name + " barks"); }  // overrides parent
}

Dog d = new Dog("Rex", "Husky");
d.speak();          // Rex barks
d.name;             // inherited field from Animal
```

### `super` — Access Parent Members

```java
class Cat extends Animal {
    Cat(String name) { super(name); }

    @Override
    void speak() {
        super.speak();                          // Animal makes a sound
        System.out.println(name + " also meows");
    }
}
```

### Constructor Chain

```java
// If parent has no no-arg constructor, subclass MUST call super(...) explicitly.
// Java implicitly inserts super() if you don't — compile error if parent lacks no-arg.
class Base {
    Base(int x) { }          // no no-arg constructor
}
class Child extends Base {
    Child() { super(42); }   // ✅ required
}
```

---

## 2. 🏛️ Abstract Class

An **abstract class** cannot be instantiated. It defines a contract (abstract methods)
while also providing shared implementation (concrete methods).

Use when: subclasses share common behavior but must implement their own version of some methods.

```java
abstract class Shape {
    String color;

    Shape(String color) { this.color = color; }

    abstract double area();                      // subclasses MUST implement

    void describe() {                            // shared concrete method
        System.out.println(color + " shape, area = " + area());
    }
}

class Circle extends Shape {
    double radius;

    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    double area() { return Math.PI * radius * radius; }
}

class Rectangle extends Shape {
    double w, h;

    Rectangle(String color, double w, double h) {
        super(color);
        this.w = w; this.h = h;
    }

    @Override
    double area() { return w * h; }
}

Shape s = new Circle("red", 5);
s.describe();       // red shape, area = 78.53...
// new Shape("x");  // ❌ compile error — cannot instantiate abstract class
```

---

## 3. 🔌 Interfaces

An interface defines a **pure contract** — what a class can do, not how.
A class can `implement` **multiple** interfaces.

```java
interface Flyable {
    int MAX_ALT = 10000;     // implicitly public static final

    void fly();              // implicitly public abstract
    void land();

    default void status() {  // default method — optional override (Java 8+)
        System.out.println("In the air");
    }
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {
    @Override public void fly()  { System.out.println("Duck flying"); }
    @Override public void land() { System.out.println("Duck landing"); }
    @Override public void swim() { System.out.println("Duck swimming"); }
}
```

### Interface vs Abstract Class

| | Abstract Class | Interface |
|---|---|---|
| Instantiate | ❌ | ❌ |
| Constructors | ✅ | ❌ |
| Fields | Any visibility | `public static final` only |
| Methods | Any (abstract + concrete) | `abstract` + `default` + `static` |
| Inheritance | Single | Multiple |
| Use when | Shared state + partial impl | Pure contract / multiple types |

### Functional Interface (Java 8+)

An interface with exactly **one abstract method** — can be used as a lambda target.

```java
@FunctionalInterface
interface Transformer {
    int transform(int x);
}

Transformer doubler = x -> x * 2;
System.out.println(doubler.transform(5)); // 10
```

---

## 4. 🔄 Method Overriding

Overriding replaces a parent method with a subclass-specific implementation.
The method signature (name + params) must match exactly.

```java
class Vehicle {
    String fuelType() { return "gasoline"; }
}

class Tesla extends Vehicle {
    @Override
    String fuelType() { return "electric"; }    // replaces parent's version
}

Vehicle v = new Tesla();  // polymorphism: reference is Vehicle, object is Tesla
v.fuelType();             // "electric" — runtime dispatch picks Tesla's version
```

### Rules

```java
// ✅ Can widen return type (covariant return)
class Base    { Base    getInstance() { return new Base(); } }
class Child extends Base { @Override Child getInstance() { return new Child(); } }

// ✅ Can reduce access? No — must be same or MORE visible
// class Base  { public void foo() {} }
// class Child { @Override void foo() {} }  ❌ narrowing public → package-private

// ❌ Cannot override final methods
class Locked { final void secret() {} }
// class Hacker extends Locked { void secret() {} }  ❌ compile error

// ❌ Cannot override static methods (hiding, not overriding)
```

### Overloading vs Overriding

```java
class Calc {
    int add(int a, int b)          { return a + b; }        // original
    double add(double a, double b) { return a + b; }        // OVERLOAD — different params
}

class AdvCalc extends Calc {
    @Override
    int add(int a, int b) { return a + b + 1; }             // OVERRIDE — same signature
}
```

---

## 5. 🔍 `instanceof`

Checks if an object is an instance of a class or interface at runtime.

```java
Animal a = new Dog("Buddy", "Lab");

a instanceof Animal  // true  — Dog IS-A Animal
a instanceof Dog     // true  — actual runtime type
a instanceof Cat     // false

// Safe cast pattern (classic)
if (a instanceof Dog) {
    Dog d = (Dog) a;
    System.out.println(d.breed);
}
```

### Pattern Matching (Java 16+)

Combines the check and cast into one line.

```java
if (a instanceof Dog d) {          // d is scoped to this block
    System.out.println(d.breed);   // no explicit cast needed
}

// Works in switch too (Java 21 — switch pattern matching)
String result = switch (a) {
    case Dog d  -> d.breed + " dog";
    case Cat c  -> c.name + " cat";
    default     -> "unknown animal";
};
```

---

## 6. 🔁 Iterator

`Iterator<T>` provides a standard way to traverse a collection,
including safe removal during iteration.

```java
List<String> names = new ArrayList<>(Arrays.asList("Alice", "Bob", "Charlie"));

Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    if (name.startsWith("B")) {
        it.remove();            // ✅ safe removal — no ConcurrentModificationException
    }
}
// names = ["Alice", "Charlie"]
```

### Implement `Iterable` on a Custom Class

Implementing `Iterable<T>` lets your class work in a for-each loop.

```java
class NumberRange implements Iterable<Integer> {
    private final int start, end;

    NumberRange(int start, int end) { this.start = start; this.end = end; }

    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<>() {
            int current = start;

            @Override public boolean hasNext() { return current <= end; }
            @Override public Integer next()    { return current++; }
        };
    }
}

for (int n : new NumberRange(1, 5)) {
    System.out.print(n + " ");   // 1 2 3 4 5
}
```

---

## 7. ⚠️ Exception Handling

### Hierarchy

```
Throwable
├── Error          (JVM-level, don't catch: OutOfMemoryError, StackOverflowError)
└── Exception
    ├── RuntimeException     (UNCHECKED — compiler doesn't force you to handle)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   └── IllegalArgumentException
    └── IOException          (CHECKED — must catch or declare throws)
        ├── FileNotFoundException
        └── ...
```

### try / catch / finally

```java
try {
    int result = 10 / 0;        // throws ArithmeticException
} catch (ArithmeticException e) {
    System.out.println("Caught: " + e.getMessage()); // / by zero
} catch (Exception e) {         // catch-all — put more specific ones first
    e.printStackTrace();
} finally {
    System.out.println("Always runs — cleanup, close resources here");
}
```

### try-with-resources (Java 7+)

Auto-closes any `AutoCloseable` resource — no need for a `finally` block.

```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = br.readLine()) != null) System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
}
// br.close() is called automatically even if an exception is thrown
```

### Custom Exception

```java
class InsufficientFundsException extends RuntimeException {
    private final double amount;

    InsufficientFundsException(double amount) {
        super("Insufficient funds: need " + amount + " more");
        this.amount = amount;
    }

    double getAmount() { return amount; }
}

class BankAccount {
    private double balance;

    void withdraw(double amount) {
        if (amount > balance) throw new InsufficientFundsException(amount - balance);
        balance -= amount;
    }
}
```

### Checked vs Unchecked

```java
// CHECKED — caller must handle or propagate
void readFile(String path) throws IOException {    // must declare
    Files.readAllBytes(Path.of(path));
}

// UNCHECKED (extends RuntimeException) — no declaration needed
void validate(int x) {
    if (x < 0) throw new IllegalArgumentException("x must be >= 0");
}
```

---

## 8. 🪞 Reflection

Reflection lets you inspect and interact with classes, fields, and methods **at runtime**,
without knowing their types at compile time.

> Useful for frameworks, serialization, and testing. Avoid in hot paths — it's slow.

```java
import java.lang.reflect.*;

class Person {
    private String name = "Alice";
    public int age = 30;

    public String greet(String msg) { return msg + ", I'm " + name; }
    private void secret()          { System.out.println("private method"); }
}
```

### Inspect a Class

```java
Class<?> clazz = Person.class;           // or: obj.getClass()

clazz.getName();                          // "Person"
clazz.getDeclaredFields();                // all fields (public + private)
clazz.getMethods();                       // public methods (inherited too)
clazz.getDeclaredMethods();               // all methods declared in this class
```

### Read / Write Fields

```java
Person p = new Person();

Field nameField = Person.class.getDeclaredField("name");
nameField.setAccessible(true);                  // bypass private access

String val = (String) nameField.get(p);         // read  → "Alice"
nameField.set(p, "Bob");                        // write → name is now "Bob"
```

### Invoke Methods

```java
Method greet = Person.class.getMethod("greet", String.class);
String result = (String) greet.invoke(p, "Hello");   // "Hello, I'm Alice"

Method secret = Person.class.getDeclaredMethod("secret");
secret.setAccessible(true);
secret.invoke(p);                                     // prints "private method"
```

### Create Instances

```java
Class<?> clazz = Class.forName("Person");               // load by name
Constructor<?> ctor = clazz.getDeclaredConstructor();   // no-arg constructor
Person p = (Person) ctor.newInstance();
```

### Check Annotations at Runtime

```java
Method m = Person.class.getMethod("greet", String.class);
if (m.isAnnotationPresent(Override.class)) { ... }
Annotation[] all = m.getAnnotations();
```
