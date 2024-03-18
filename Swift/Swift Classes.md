---
title: Swift Classes
date: 24-03-18
layout: post
tags:
  - swift
  - "#types"
  - "#class"
---
# Declaring a Class

In Swift, defining a class involves specifying its properties and methods within a class declaration. Here's how you can define a simple class in Swift:

```Swift
class Person {
    var name: String
    var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }

    func introduce() -> String {
        return "Hello, my name is \(name) and I am \(age) years old."
    }
}
```

**Initializer**: The `init` method is a special method called an initializer. It's used to create an instance of the class with its properties initialized to their starting values. Replace `propertyName` with the name of your property and `PropertyType` with the type of your property.

# Instantiation

To instantiate the class you've defined, you create a new instance by calling its initializer with the required parameters. Here's how you do it:

```Swift
let person = Person(name: "John Doe", age: 30)
```

This line of code creates a new `Person` object with the name "John Doe" and age 30. You use the `let` keyword to declare a constant named `person` that refers to the instance. If you plan to modify the properties of the `Person` instance later, you should use `var` instead of `let` to declare it.

After instantiation, you can access its properties and methods like this:

```Swift
// Accessing properties
print(person.name) // Output: John Doe
print(person.age)  // Output: 30

// Calling a method
print(person.introduce()) // Output: Hello, my name is John Doe and I am 30 years old.
```

