---
title: Swift Functions
date: 20-03-26
layout: post
tags:
  - swift
  - "#functions"
  - "#methods"
---

# Defining a Function

Here's a basic structure:

```swift
func functionName(parameterName: ParameterType) -> ReturnType {
    // function body goes here
    return valueOfReturnType
}
```

If your function doesn't return a value, you can omit the return type:

```swift
func sayHello() {
    print("Hello, world!")
}
```

You can call this function by simply using its name followed by parentheses:

```swift
sayHello() // prints "Hello, world!"
```

If your function has parameters, you include them in the parentheses when you call the function. Here's an example with parameters:

```swift
func greet(person: String, day: String) -> String {
    return "Hello \(person), today is \(day)."
}

let message = greet(person: "Bob", day: "Tuesday")
print(message) // prints "Hello Bob, today is Tuesday."
```

Swift functions can also return multiple values using tuples:

```swift
func getUser() -> (name: String, age: Int) {
    return ("Alice", 30)
}

let user = getUser()
print("Name: \(user.name), Age: \(user.age)")
```

# Separating Declaration and Definition

You can declare a function's signature (its name, parameters, and return type) in one place, typically within a protocol or as part of a class or struct declaration, and then provide its definition—its actual body—somewhere else. This is commonly used in protocols to define a set of functions that conforming types must implement, or in classes and structs to separate the declaration of a function from its implementation for readability or organizational purposes.

However, it's important to clarify that unlike some other languages like C or C++, Swift doesn't have a separate declaration and implementation file concept for functions. Instead, when you're working within a single Swift file, you define the function where you declare it. The separation typically comes into play with protocols, or with inheritance in classes.

### Using Protocols
When using protocols, you declare a function in the protocol, and any type that conforms to the protocol must implement the function.

```swift
protocol Greeter {
    func greet(name: String) -> String
}

class EnglishGreeter: Greeter {
    func greet(name: String) -> String {
        return "Hello, \(name)!"
    }
}
```

### Inheritance in Classes
In classes, you can declare a function in a parent class and override it in a child class to provide a specific implementation.

```swift
class Animal {
    func makeSound() {
        // Default implementation
        print("Some generic animal sound")
    }
}

class Dog: Animal {
    override func makeSound() {
        // Specific implementation for Dog
        print("Woof!")
    }
}
```

# Interesting Return Types

### `nil`
- In Swift, `nil` is used to represent the absence of a value for any optional type. An optional type is a type that can hold either a value or `nil` to indicate that a value is missing. You define an optional by adding a question mark (`?`) after the type of a property, variable, or return value.
- Functions can return an optional type, which means they are allowed to return `nil` if there's no value to return.

```swift
func findIndexOf(string: String, in array: [String]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == string {
            return index
        }
    }
    return nil // No matching string found
}
```

### `Void`
- In Swift, a function that doesn't explicitly return a value actually returns `Void`, which is an empty tuple `()`. This is similar to returning `void` in languages like C or Java. You don't usually need to specify `Void` as the return type; you can simply omit the return type and return arrow (`->`) in the function declaration.
- You can explicitly state that a function returns `Void`, but it's more common to see functions without a return type for this purpose.

```swift
func doNothing() -> Void {
    // This function intentionally left blank
}

// More commonly written as:
func doNothing() {
    // This function intentionally left blank
}
```

### Tuples
- Tuples are a way to return multiple values from a function as a single compound value. You can use tuples to return several values grouped together without creating a struct or class.

```swift
func minMax(array: [Int]) -> (min: Int, max: Int)? {
    guard let min = array.min(), let max = array.max() else {
        return nil // Array might be empty
    }
    return (min, max)
}

if let result = minMax(array: [1, 2, 3, 4, 5]) {
    print("Min: \(result.min), Max: \(result.max)")
}
```

### `Never`
- `Never` is an interesting return type in Swift. It indicates that a function will not return because it either throws an error that must be handled or causes the program to exit. It's used in functions that do not complete normally, such as a function that triggers an assertion failure or an infinite loop.

```swift
func alwaysThrows() throws -> Never {
    throw SomeError()
}

func infiniteLoop() -> Never {
    while true {
    }
}
```

These examples showcase some of the flexible ways Swift handles return types, making it powerful and expressive, especially with features like optional types and tuples.

# Functions as Arguments

Functions are first-class citizens, which means you can pass a function as an argument to another function. This feature is very powerful and allows for more flexible and reusable code, especially when it comes to callbacks, higher-order functions, and functional programming concepts.

To pass a function as an argument, you need to specify the type of the function as the parameter type. The function type includes the types of its parameters and its return type. Here's a basic example to illustrate how this works:

### Example: Passing a Function as an Argument

```swift
// Define a function that takes two Ints and returns an Int
func add(a: Int, b: Int) -> Int {
    return a + b
}

// Define a function that takes two Ints and returns an Int
func multiply(a: Int, b: Int) -> Int {
    return a * b
}

// A function that takes another function as a parameter
// The parameter 'operation' is of type '(Int, Int) -> Int'
func performOperation(_ a: Int, _ b: Int, operation: (Int, Int) -> Int) -> Int {
    return operation(a, b)
}

// Call 'performOperation' with 'add' as an argument
let sum = performOperation(4, 5, operation: add)
print("Sum: \(sum)") // Output: Sum: 9

// Call 'performOperation' with 'multiply' as an argument
let product = performOperation(4, 5, operation: multiply)
print("Product: \(product)") // Output: Product: 20
```

In this example, `performOperation` is a higher-order function because it takes another function (`operation`) as an argument. The `operation` parameter is a function that takes two `Int` values and returns an `Int`. You can then call `performOperation` with either the `add` or `multiply` function as the `operation` argument.

This capability allows for highly flexible and reusable code designs, such as callbacks, function chaining, and more sophisticated patterns like map-reduce operations on collections.

# Functions as Class Members

Here's how you can store and use a function as a class member:

### Step 1: Defining the Class and Function Property

First, you define a class with a property to store the function. The type of the property will be a function type, specifying the parameters and return type of the function that can be stored.

```swift
class Calculator {
    var operation: (Int, Int) -> Int

    init(operation: @escaping (Int, Int) -> Int) {
        self.operation = operation
    }

    func performOperation(a: Int, b: Int) -> Int {
        return operation(a, b)
    }
}
```

In this example, the `Calculator` class has an `operation` property that can store any function matching the type `(Int, Int) -> Int`, which is a function that takes two `Int` parameters and returns an `Int`. The `@escaping` annotation is used because the function is stored for later use, beyond the scope of the function that receives it.

The `@escaping` attribute is used to indicate that a closure (or a function, since functions are a special case of closures in Swift) is allowed to "escape" from the scope in which it is defined. This means the closure can be stored and executed after the function in which it is passed as an argument returns. Without the `@escaping` attribute, closures are considered non-escaping by default, meaning they must be executed within the function's body before it returns.

### Step 2: Storing and Using the Function

Next, you create instances of the `Calculator` class, providing different functions to be stored as the `operation` property. You can then use the `performOperation` method to invoke the stored function.

```swift
// Define two simple arithmetic functions
func add(a: Int, b: Int) -> Int {
    return a + b
}

func multiply(a: Int, b: Int) -> Int {
    return a * b
}

// Create Calculator instances with different operations
let additionCalculator = Calculator(operation: add)
let multiplicationCalculator = Calculator(operation: multiply)

// Perform operations
let sum = additionCalculator.performOperation(a: 3, b: 4)
print("Sum: \(sum)")  // Output: Sum: 7

let product = multiplicationCalculator.performOperation(a: 3, b: 4)
print("Product: \(product)")  // Output: Product: 12
```

# Overloading

### By Number of Parameters
You can overload functions by having a different number of parameters.

```swift
func displayMessage(_ message: String) {
    print(message)
}

func displayMessage(_ message: String, withTitle title: String) {
    print("\(title): \(message)")
}
```

### By Type of Parameters
Functions can be overloaded by varying the type of one or more parameters.

```swift
func add(_ a: Int, _ b: Int) -> Int {
    return a + b
}

func add(_ a: Double, _ b: Double) -> Double {
    return a + b
}
```

### By Argument Labels
You can also overload functions by using different argument labels.

```swift
func multiply(value a: Int, by b: Int) -> Int {
    return a * b
}

func multiply(_ a: Int, with b: Int) -> Int {
    return a * b
}
```

# Operator Overloading

Swift allows you to overload operators, similar to C++. Operator overloading enables you to provide custom implementations for existing operators (like `+`, `-`, `*`, `/`, etc.) for your own custom types. This can make your custom types more intuitive to use and integrate seamlessly with Swift's syntax.

To overload an operator in Swift, you define a global function with the `operator` keyword followed by the operator you want to overload. Inside the function, you specify the logic for how the operator should work with your custom types.

### Example: Overloading the `+` Operator for a Custom Type

Let's say you have a simple `Vector2D` struct that represents a two-dimensional vector, and you want to add the capability to add two vectors together using the `+` operator.

```swift
struct Vector2D {
    var x: Double
    var y: Double
}

// Overload the '+' operator for the Vector2D type
func +(left: Vector2D, right: Vector2D) -> Vector2D {
    return Vector2D(x: left.x + right.x, y: left.y + right.y)
}

// Example usage
let vector1 = Vector2D(x: 1.0, y: 2.0)
let vector2 = Vector2D(x: 3.0, y: 4.0)
let resultVector = vector1 + vector2
print("Resulting Vector: (\(resultVector.x), \(resultVector.y))")
```

In this example, the `+` function takes two `Vector2D` instances (`left` and `right`) and returns a new `Vector2D` instance that represents the vector sum of the inputs. You can then use the `+` operator to add two `Vector2D` instances in a natural and readable way.

### Custom Operators
In addition to overloading existing operators, Swift also allows you to define custom operators. You can declare a custom operator with the `operator` keyword and specify whether it's a prefix, infix, or postfix operator. After declaring it, you can implement the operator by defining a global function with the same name as your operator and the logic you want it to perform.
