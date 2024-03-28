---
title: Swift Closures
date: 24-03-27
layout: post
tags:
  - functions
  - swift
  - "#closures"
  - "#lambdas"
---

Here’s a simple example of a closure in Swift, which is similar to a lambda function:

```swift
let multiply = { (number1: Int, number2: Int) -> Int in
    return number1 * number2
}

let result = multiply(4, 2)
print(result) // Outputs: 8
```

This example defines a closure that multiplies two numbers and then executes it with the numbers 4 and 2. Swift's closures are powerful tools for creating concise and readable code, supporting features like capturing values, closure expressions, and shorthand argument names.

# Saving Closures to Use Later

Passing a closure as an argument to a function or storing it within a class as a property in Swift is a common practice that enhances flexibility and reusability of code. Let's look at both scenarios:

### Passing a Closure as an Argument to a Function

In Swift, you can pass closures to functions as you would with any other type of parameter. When defining the function, specify the closure's type with the syntax `(ParameterTypes) -> ReturnType`. Here’s an example:

```swift
func performOperation(on numbers: [Int], using operation: (Int, Int) -> Int) -> Int {
    var result = numbers.first ?? 0
    for number in numbers.dropFirst() {
        result = operation(result, number)
    }
    return result
}

let sum = performOperation(on: [1, 2, 3, 4]) { $0 + $1 }
print(sum) // Outputs: 10
```

In this example, `performOperation` takes an array of integers and a closure that defines how to combine them. The closure is then used to compute a result.

### Storing a Closure as a Property in a Class

You can also store closures as properties in a class or struct. This is useful for callbacks or customizing behavior. When declaring a closure property, you again use the closure type, and you often make it optional (`?`) so it can start as `nil`. Here’s how you might do it:

```swift
class Task {
    var completion: (() -> Void)?

    func complete() {
        completion?()
    }
}

let task = Task()
task.completion = {
    print("Task completed!")
}

task.complete() // Prints: Task completed!
```

In this example, the `Task` class has a `completion` property that is a closure with no parameters and no return value. This closure is executed when `complete()` is called.

When storing closures, especially in classes, be mindful of retain cycles. If a closure captures `self` (meaning it uses properties or methods of the class it's defined in), you may create a strong reference cycle. To avoid this, capture `self` as `weak` or `unowned` within the closure.