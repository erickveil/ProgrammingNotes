---
title: Swift Loops
date: 024-04-04
layout: post
tags:
  - swift
  - "#loops"
---

A `for-in` loop is perhaps the most commonly used, allowing you to iterate over a sequence, such as a range of numbers, items in an array, or characters in a string. It’s very versatile and clean in syntax. For example, if you wanted to print out the numbers 1 through 5, you could use a `for-in` loop like this:

```swift
for number in 1...5 {
    print(number)
}
```

`While` loops perform a set of statements until a condition becomes false. This is useful when you don't know in advance how many times you'll need to loop. Here’s how you might use a `while` loop to iterate until a variable reaches a certain value:

```swift
var number = 1
while number <= 5 {
    print(number)
    number += 1
}
```

`Repeat-while` loops are similar to `while` loops, but they ensure that the loop's body is executed at least once, because the condition is at the end of the loop. Here’s an example:

```swift
var number = 1
repeat {
    print(number)
    number += 1
} while number <= 5
```

# Uncommon Iterations
### Iterating Backwards

To iterate backwards, you can use the `reversed()` method on your sequence or use a reversed range. For example, to print numbers 5 through 1 backwards, you can do:

```swift
for number in (1...5).reversed() {
    print(number)
}
```

### Iterating in Strides

If you want to iterate by steps of 5 (like 5, 10, 15), you can use the `stride(from:to:by:)` or `stride(from:through:by:)` function. The `to` version does not include the end value, while the `through` version does. Here's how you might use it:

```swift
for number in stride(from: 5, through: 15, by: 5) {
    print(number)
}
```

### Iterating Arrays Without Indices

Yes, you can iterate over the elements of an array directly, without using their indices. This is one of the common ways to work with arrays in Swift:

```swift
let fruits = ["Apple", "Banana", "Cherry"]
for fruit in fruits {
    print(fruit)
}
```

### Iterating Over Dictionaries and Sets

Dictionaries and sets are also collections, and you can iterate over them similarly. When you iterate over a dictionary, you get key-value pairs, which you can destructure into two variables:

```swift
let personAges = ["John": 28, "Jane": 32, "Doe": 24]
for (name, age) in personAges {
    print("\(name) is \(age) years old.")
}
```

For sets, since they store unique, unordered elements, you can iterate through the elements directly, similar to array elements:

```swift
let uniqueNumbers: Set = [5, 7, 11]
for number in uniqueNumbers {
    print(number)
}
```

Swift's design aims to make iterating over collections as intuitive and straightforward as possible, while also offering the flexibility to handle more complex iteration patterns when needed.

# Iterating Over Object Members

Swift doesn't have built-in reflection capabilities as extensive as some other languages, but you can use the `Mirror` API to inspect and iterate over the properties of an instance.

Here’s a basic example of how you might use `Mirror` to iterate over the properties of a Swift object:

```swift
struct Person {
    var name: String
    var age: Int
}

let john = Person(name: "John Doe", age: 30)
let mirror = Mirror(reflecting: john)

for child in mirror.children {
    guard let propertyName = child.label else { continue }
    print("\(propertyName): \(child.value)")
}
```

In this example, `Mirror(reflecting:)` is used to create a reflection of the `john` instance. The `children` property of the mirror contains each property (as `child`) of the instance, where `label` is the name of the property and `value` is its value. This way, you can iterate over each property and access its name and value.

 `Mirror` is generally intended for debugging or logging purposes. Swift’s strong type system and emphasis on protocol-oriented programming typically encourage other approaches for working with objects, especially when it comes to iterating over their properties.