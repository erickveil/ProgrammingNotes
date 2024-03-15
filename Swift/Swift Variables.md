---
layout: post
title: Swift Variables
date: 24-03-14
tags:
  - swift
  - types
  - variables
---

In Swift, you can define **variables** using the `var` keyword and **constants** using the `let` keyword. Here's how you can declare and define variables and constants:

1. Declare and define a variable:
```Swift
var myVariable: Int = 10
```

2. Declare a variable without defining it (initialize it later):
```Swift
var myVariable: Int 
// You can assign a value to 'myVariable' later in your code
```

3. Declare and define a constant:
```Swift
let myConstant: String = "Hello, Swift!"
```

4. Declare a constant without defining it (initialize it later):
```Swift
let myConstant: String 
// You can assign a value to 'myConstant' later in your code, but you can't change it once it's set
```

You cannot define a variable first and then declare it later separately in Swift. Once you declare a variable or constant, you either define it at the same time or leave it undefined if you plan to assign a value to it later.

# Default Types

Several default data types are available for storing different kinds of values. Here are some of the most commonly used default data types:

1. Int
2. Double
3. Float
4. Bool
5. String
6. Character
7. Array
8. Dictionary
9. Set

# Arrays Dictionaries and Sets

## Defining:

Here are examples of how to declare and define arrays, dictionaries, and sets in Swift, both separately and in one step:

1. Arrays:
```Swift
// Declare and define an array of Integers
var numbers: [Int] = [1, 2, 3, 4, 5]

// Declare an array without defining it (initialize it later)
var names: [String]

// Define the array later
names = ["Alice", "Bob", "Charlie"]

```

2. Dictionaries:
```Swift
// Declare and define a dictionary with String keys and Int values
var ages: [String: Int] = ["Alice": 30, "Bob": 25, "Charlie": 35]

// Declare a dictionary without defining it (initialize it later)
var scores: [String: Int]

// Define the dictionary later
scores = ["Alice": 95, "Bob": 87, "Charlie": 92]

```


3. Sets:
```Swift
// Declare and define a set of Integers
var uniqueNumbers: Set<Int> = [1, 2, 3, 4, 5]

// Declare a set without defining it (initialize it later)
var uniqueLetters: Set<Character>

// Define the set later
uniqueLetters = ["a", "b", "c"]

```

## Referencing:

To reference values from arrays, dictionaries, and sets in Swift, you use subscript notation for arrays and dictionaries, and you can directly access elements in a set. Here's how you can reference values from each of these collections:

1. Arrays:
```Swift
// Given an array
var numbers: [Int] = [1, 2, 3, 4, 5]

// Accessing elements by index
let firstNumber = numbers[0] // Access the first element (1)
let thirdNumber = numbers[2] // Access the third element (3)

```

2. Dictionaries:
```Swift
// Given a dictionary
var ages: [String: Int] = ["Alice": 30, "Bob": 25, "Charlie": 35]

// Accessing values by key
let aliceAge = ages["Alice"] // Access Alice's age (30)
let bobAge = ages["Bob"]     // Access Bob's age (25)

```

3. Sets:
```Swift
// Given a set
var uniqueNumbers: Set<Int> = [1, 2, 3, 4, 5]

// Accessing elements directly
let containsThree = uniqueNumbers.contains(3) // Check if the set contains 3

```

In each example, you use the appropriate syntax to access values from the collection. For arrays and dictionaries, you use subscript notation (`[]`) with the index or key respectively, while for sets, you can directly access elements using methods like `contains(_:)`.

### Sets
Sets are unordered collections, meaning they don't have a defined order for their elements, and therefore, they do not support subscript notation for accessing elements by index like arrays and dictionaries.

If you need to access a specific value in a set, you would typically iterate over the set's elements or use set operations to check for the presence of a value. Here's an example of iterating over a set to find a specific value:

```Swift
var uniqueNumbers: Set<Int> = [1, 2, 3, 4, 5]

// Iterate over the set to find a specific value
let desiredNumber = 3
var found = false

for number in uniqueNumbers {
    if number == desiredNumber {
        found = true
        break
    }
}

if found {
    print("The set contains \(desiredNumber).")
} else {
    print("The set does not contain \(desiredNumber).")
}

```

Alternatively, you can use the `contains(_:)` method.

### Push / Pop
You can perform similar operations to push and pop on arrays, but not on dictionaries or sets. Here's how it works:

1. Arrays: You can use `append(_:)` to push an element to the end of an array and `removeLast()` to pop the last element from the array.

```Swift
var numbers: [Int] = [1, 2, 3, 4, 5]

// Push an element to the end of the array
numbers.append(6) // numbers becomes [1, 2, 3, 4, 5, 6]

// Pop the last element from the array
let poppedElement = numbers.removeLast() // poppedElement = 6, numbers becomes [1, 2, 3, 4, 5]

```

2. Dictionaries and Sets: Dictionaries and sets are unordered collections, so they don't have a concept of pushing or popping elements like arrays. Instead, you interact with dictionaries by adding or removing key-value pairs, and with sets by adding or removing individual elements.

```Swift
var ages: [String: Int] = ["Alice": 30, "Bob": 25, "Charlie": 35]
var uniqueNumbers: Set<Int> = [1, 2, 3, 4, 5]

// Example of adding a key-value pair to a dictionary
ages["David"] = 40 // ages becomes ["Alice": 30, "Bob": 25, "Charlie": 35, "David": 40]

// Example of removing a key-value pair from a dictionary
ages["Bob"] = nil // ages becomes ["Alice": 30, "Charlie": 35]

// Example of adding an element to a set
uniqueNumbers.insert(6) // uniqueNumbers becomes [1, 2, 3, 4, 5, 6]

// Example of removing an element from a set
uniqueNumbers.remove(3) // uniqueNumbers becomes [1, 2, 4, 5, 6]

```

### nil
`nil` in Swift is similar to `null` in other programming languages, such as Java or C#. It represents the absence of a value. However, in Swift, `nil` is more powerful and safer due to Swift's optionals.

In Swift, optionals allow variables to have a value or be `nil`, indicating the absence of a value. Optionals are used to handle situations where a value may be missing, preventing runtime errors caused by accessing uninitialized or null variables.

Swift does not have a concept of null for non-optional variables. Instead, optionals are used to represent both a value and the absence of a value. You declare an optional variable by appending a question mark (`?`) to the type.

Here's an example:

```Swift
var optionalNumber: Int? = nil // Declaring an optional variable with a value of nil
```

# Representing Variables in a String

To include a variable value in a string, you can use string interpolation or concatenation. String interpolation allows you to embed variables directly within a string literal, making your code more concise and readable.

Here's how you can include a variable value in a string using string interpolation:

```Swift
let name = "John"
let age = 30

let message = "Hello, my name is \(name) and I am \(age) years old."
print(message)
```

In this example, the variables `name` and `age` are embedded within the string `message` using `\()` syntax. When the string is printed, the values of the variables are interpolated into the string.

Alternatively, you can use string concatenation to achieve the same result:

```Swift
let name = "John"
let age = 30

let message = "Hello, my name is \(name) and I am \(age) years old."
print(message)
```

In this approach, the variables are concatenated with the rest of the string using the `+` operator. However, string interpolation is generally preferred as it is more concise and readable.

# Determining Type

You can get the type of a variable using the `type(of:)` function.

```Swift
let someVariable = 42
let variableType = type(of: someVariable)
print("The type of someVariable is: \(variableType)")
```
