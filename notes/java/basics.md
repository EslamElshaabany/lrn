---
title: Java Basics
topic: java · fundamentals
---


> A personal notebook-style refresher covering Java fundamentals:
> memory model, variables, strings, control flow, and more.

---

## 📦 Java Memory Architecture

Java manages memory across several distinct regions. Understanding where your data lives helps you write better, more predictable code.

| Region | Scope | What lives here |
|---|---|---|
| **Heap** | Shared | All objects created with `new` |
| **String Pool** | Inside Heap | Cached string literals |
| **Stack** | Per Thread | Local primitives & object references |
| **Metaspace / Method Area** | Shared | Class blueprints, methods, static variables |
| **PC Register** | Per Thread | Tracks the current execution line |
| **Native Stack** | Per Thread | Used for JNI / C++ native code |

---

## 🗃️ Variables in Memory

### A) Stack — The 8 Primitive Types

When you declare a primitive inside a method, its **actual value** is stored directly on the stack.

```java
// ── Integer types ─────────────────────────────────────
int     x    = 10;    // 32-bit signed integer
byte    b    = 10;    // 8-bit  signed integer
short   s    = 10;    // 16-bit signed integer
long    l    = 10L;   // 64-bit signed integer  (note the L suffix)
// ── Floating-point types ──────────────────────────────
double  d    = 10.5;  // 64-bit IEEE 754
float   f    = 10.5f; // 32-bit IEEE 754         (note the f suffix)
// ── Character type ────────────────────────────────────
char    c    = 'a';   // 16-bit Unicode character (single quotes)
// ── Boolean type ──────────────────────────────────────
boolean flag = true;  // true or false
```

> **Suffixes matter:** `10` is an `int`, `10L` is a `long`, `10.5` is a `double`, `10.5f` is a `float`.

---

### B) Heap — Object References

When you create an object or array, the **object's data goes to the Heap**, but the **reference variable itself stays on the Stack**.

This applies to:
- **Classes** — `String`, `Scanner`, any custom class
- **Interfaces** — `List`, `Runnable`
- **Arrays** — `int[]`, `String[]`

```java
// ── String ────────────────────────────────────────────
String text = "Hello";                    // 'text'    → Stack | "Hello" object → Heap (String Pool)
// ── Array of references ───────────────────────────────
String[] names = new String[3];           // 'names'   → Stack | array of 3 slots → Heap
// ── Standard library class ────────────────────────────
Scanner scanner = new Scanner(System.in); // 'scanner' → Stack | Scanner object → Heap
// ── Interface reference ───────────────────────────────
List<String> list = new ArrayList<>();    // 'list'    → Stack | ArrayList object → Heap
// ── Primitive array ───────────────────────────────────
int[] numbers = new int[5];               // 'numbers' → Stack | 5 int slots → Heap
```

> **What is NEVER on the Stack?**
> - The actual data inside objects/arrays — always on the Heap
> - **Instance variables** (class-level fields) — part of the object, live on the Heap
> - **Static variables** — stored in the Metaspace

---

## 🔤 Strings

### Literal vs. Object

```java
String s1 = "Hello World";             // Uses the String Pool (preferred)
String s2 = new String("Hello World"); // Forces a brand-new Heap instance (avoid)
```

### String Pool & Reference Equality

```java
String a = "Hello";             // stored in String Pool
String b = "Hello";             // reuses the same Pool instance — same reference as 'a'
String c = new String("Hello"); // brand-new object on the Heap

System.out.println(a == b);       // true  — same Pool reference
System.out.println(a == c);       // false — different Heap address
System.out.println(a.equals(b));  // true  — same content
System.out.println(a.equals(c));  // true  — same content
```

> Always use `.equals()` to compare string **content**.
> `==` only compares **memory addresses** (references).

### Mutable Strings — StringBuilder

`String` is **immutable** — every concatenation creates a new object.
Use `StringBuilder` when building strings in a loop or with many changes.

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");
System.out.println(sb.toString()); // Hello World
```

---

## 🔒 The `final` Keyword

### Final Primitives — The value is locked

```java
final double PI       = 3.14159;
final int   MAX_USERS = 100;

// PI        = 3.15; // ❌ compile error: cannot assign a value to a final variable
// MAX_USERS = 101;  // ❌ compile error
```

### Final References — The pointer is locked, the data is not

```java
final int[] finalArray = { 1, 2, 3 };

// finalArray = new int[]{ 4, 5, 6 }; // ❌ compile error: cannot reassign the reference
finalArray[0] = 99; // ✅ allowed — the contents of the array can still be mutated

System.out.println(finalArray[0]); // 99
```

### Final Classes & Methods

```java
// final class  → cannot be extended    (e.g. String itself is declared final)
// final method → cannot be overridden in a subclass

public final class ImmutableBox {
    public final int getValue() { return 42; }
}
```

---

## 🔄 Type Casting

### Widening — Implicit (safe, no data loss)

Smaller type is automatically promoted to a larger type.

```java
int    myInt    = 9;
double myDouble = myInt; // int → double: 9 becomes 9.0 automatically

System.out.println(myDouble); // 9.0
```

### Narrowing — Explicit (manual, may lose precision)

Larger type must be cast manually — the fractional part is truncated.

```java
double pi        = 3.14;
int    truncated = (int) pi; // double → int: 3.14 becomes 3 (decimals dropped)

System.out.println(truncated); // 3
```

---

## 🔁 Loops

### Classic `for` Loop

```java
int[] arr = { 10, 20, 30, 40, 50, 60, 70, 80, 90, 100 };

for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}
```

### `break` and `continue`

```java
for (int i = 0; i < 10; i++) {
    if (i == 3) continue; // skip the rest of this iteration, jump to i++
    if (i == 7) break;    // exit the loop entirely
    System.out.println(i);
}
// Output: 0  1  2  4  5  6
```

### `while` Loop

```java
int i = 0;
while (i < 5) {
    System.out.println(i); // 0 1 2 3 4
    i++;
}
```

### `do-while` Loop

Guaranteed to execute **at least once** before the condition is checked.

```java
int j = 0;
do {
    System.out.println(j); // 0 1 2 3 4
    j++;
} while (j < 5);
```

### Enhanced `for-each` Loop

Cleaner syntax for iterating over arrays or collections — no index needed.

```java
int[] arr = { 10, 20, 30, 40, 50 };

for (int value : arr) {
    System.out.println(value);
}
```

---

## 🔀 Conditionals

### `if` / `else if` / `else`

```java
int x = 10;

if (x > 10) {
    System.out.println("x is greater than 10");
} else if (x == 10) {
    System.out.println("x is exactly 10");
} else {
    System.out.println("x is less than 10");
}
```

### Ternary Operator

Compact single-line conditional — good for simple value assignments.

```java
// syntax: condition ? valueIfTrue : valueIfFalse
int    age    = 20;
String status = (age >= 18) ? "Adult" : "Minor";
System.out.println(status); // Adult
```

### `switch` Statement

```java
int x = 10;

switch (x) {
    case 10:
        System.out.println("x is 10");
        break;
    case 20:
        System.out.println("x is 20");
        break;
    default:
        System.out.println("x is something else");
}
```

> Don't forget `break` — without it, execution **falls through** to the next case.

---

## 🧰 Methods

Methods group reusable logic. Use `static` to call a method directly on the class without instantiating an object.

```java
public class Calculator {

    // static method: accessible without creating a Calculator object
    public static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        int result = add(10, 20);
        System.out.println(result); // 30
    }
}
```

---

## 📥 User Input — Scanner

```java
import java.util.Scanner;

public class InputExample {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Enter your name: ");
        String name = scanner.nextLine(); // reads a full line of text

        System.out.print("Enter your age:  ");
        int age = scanner.nextInt();      // reads a single integer

        System.out.println("Hello, " + name + "! You are " + age + " years old.");

        scanner.close(); // always close the Scanner when done
    }
}
```

---
