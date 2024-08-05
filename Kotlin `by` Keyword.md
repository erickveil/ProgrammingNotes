---
title: Kotlin `by` Keyword
layout: note
tags:
  - android
  - kotlin
---

In Kotlin, the `by` keyword is used in several contexts, most notably for delegation. It provides a concise and readable way to delegate functionality from one class or interface to another. Here are the primary uses of the `by` keyword in Kotlin:

### 1. Delegation for Interfaces

You can delegate the implementation of an interface to another object using the `by` keyword. This allows you to reuse the implementation of the interface from another class or object.

```kotlin
interface Printer {
    fun print()
}

class ConsolePrinter : Printer {
    override fun print() {
        println("Printing to console")
    }
}

class PrinterDelegate(printer: Printer) : Printer by printer

fun main() {
    val printer = ConsolePrinter()
    val printerDelegate = PrinterDelegate(printer)
    printerDelegate.print()  // Output: Printing to console
}
```

In this example, `PrinterDelegate` implements the `Printer` interface by delegating the implementation to the `ConsolePrinter` instance passed to its constructor.

### 2. Property Delegation

Kotlin also allows you to delegate the implementation of a property to another object using the `by` keyword. This is useful for lazy properties, observable properties, and others.

#### Lazy Properties

A property is initialized only when it is accessed for the first time.

```kotlin
val lazyValue: String by lazy {
    println("Computed!")
    "Hello"
}

fun main() {
    println(lazyValue)  // Output: Computed! \n Hello
    println(lazyValue)  // Output: Hello
}
```

#### Observable Properties

You can delegate property changes to a handler function.

```kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main() {
    val user = User()
    user.name = "first"  // Output: <no name> -> first
    user.name = "second" // Output: first -> second
}
```

### 3. Map Delegation

You can use the `by` keyword to delegate properties to a map.

```kotlin
class User(map: Map<String, Any?>) {
    val name: String by map
    val age: Int by map
}

fun main() {
    val user = User(mapOf(
        "name" to "John Doe",
        "age" to 25
    ))

    println(user.name) // Output: John Doe
    println(user.age)  // Output: 25
}
```


# But, Why?

Delegation provides several benefits over inheritance:

1. **Composition Over Inheritance**: Delegation promotes composition over inheritance. Instead of inheriting behavior from a superclass, you can compose behavior by delegating responsibilities to other objects. This leads to more flexible and maintainable code.
    
2. **Separation of Concerns**: By delegating responsibilities, you can separate concerns more cleanly. Each class can focus on a single responsibility, adhering to the Single Responsibility Principle.
    
3. **Avoiding Inheritance Hierarchies**: Inheritance can lead to deep and complex class hierarchies, which can be hard to manage and understand. Delegation allows you to avoid such hierarchies.
    
4. **Reuse Without Inheritance**: You can reuse behavior without needing to establish a parent-child relationship between classes, making it easier to change or extend behavior without affecting unrelated parts of your code.

# Yeah, So, Why Not Just Inject Like Normal?

With delegation we help solve the "composition over inheritance" issue in a new way. But why do we have a new way? This problem is already solved in OOP, isn't it?

Indeed, you can achieve similar results by passing a class with the required functionality to the constructor. However, Kotlin's delegation using the `by` keyword simplifies this process and provides some additional benefits:

### Automatic Implementation of Interface Methods

When you use the `by` keyword, Kotlin automatically implements the interface methods by delegating them to the provided object. This reduces boilerplate code.

### Example Comparison

Let's compare both approaches to see the difference.

#### Without Delegation (`by` keyword)

```kotlin
interface Printer {
    fun print()
}

class ConsolePrinter : Printer {
    override fun print() {
        println("Printing to console")
    }
}

class PrinterDelegate(private val printer: Printer) : Printer {
    override fun print() {
        printer.print()
    }
}

fun main() {
    val consolePrinter = ConsolePrinter()
    val printerDelegate = PrinterDelegate(consolePrinter)
    printerDelegate.print()  // Output: Printing to console
}
```

Here, you need to manually forward the `print` method call to the `printer` object.

#### With Delegation (`by` keyword)

```kotlin
interface Printer {
    fun print()
}

class ConsolePrinter : Printer {
    override fun print() {
        println("Printing to console")
    }
}

class PrinterDelegate(printer: Printer) : Printer by printer

fun main() {
    val consolePrinter = ConsolePrinter()
    val printerDelegate = PrinterDelegate(consolePrinter)
    printerDelegate.print()  // Output: Printing to console
}
```

In this case, the `by` keyword automatically handles the forwarding of method calls, making the code simpler and cleaner.

### Benefits of Using `by` for Delegation

1. **Less Boilerplate**: The `by` keyword eliminates the need to write boilerplate code for forwarding method calls.
2. **Clarity**: The use of `by` clearly indicates that the class is delegating the responsibility to another object, making the intention of the code more apparent.
3. **Maintainability**: If the interface changes (e.g., new methods are added), the class using `by` delegation will automatically have those methods delegated, reducing the need for manual updates.
4. **Consistency**: Using `by` ensures a consistent pattern for delegation throughout your codebase, making it easier to understand and maintain.

### When to Use Constructor Passing vs. Delegation

- **Constructor Passing**: Use this when you need more control over how methods are called or when the delegation pattern isn't a perfect fit. It gives you the flexibility to modify the behavior if needed.
- **Delegation with `by`**: Use this when you want a straightforward delegation of responsibilities with minimal boilerplate. It is particularly useful for implementing interfaces and property delegation.
