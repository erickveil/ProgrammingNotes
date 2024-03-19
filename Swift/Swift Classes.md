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

# Subclassing

Here's an example to illustrate how to create a subclass. Let's say we have a `Person` class, as defined previously:

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

Now, suppose we want to create an `Employee` class that inherits from `Person` but also includes an additional property for the employee's job title:

```Swift
class Employee: Person {
    var jobTitle: String

    init(name: String, age: Int, jobTitle: String) {
        self.jobTitle = jobTitle
        // Call the superclass's initializer
        super.init(name: name, age: age)
    }

    // Example of method overriding
    override func introduce() -> String {
        return super.introduce() + " I work as a \(jobTitle)."
    }
}
```

In this `Employee` class:

- It inherits from `Person` by specifying `: Person` after its name.
- It adds a new property, `jobTitle`.
- It overrides the `init` method to include the initialization of `jobTitle`. Inside the `init`, it calls `super.init(name: age:)` to ensure the properties inherited from `Person` are properly initialized.
- It overrides the `introduce` method to extend the behavior of the method inherited from `Person`. The `override` keyword is required to override a method. `super.introduce()` is called to include the original introduction message, and then it adds information about the job title.

To instantiate an `Employee` and use its properties and methods:

```Swift
let employee = Employee(name: "Jane Doe", age: 28, jobTitle: "Software Engineer")
print(employee.introduce())
// Output: Hello, my name is Jane Doe and I am 28 years old. I work as a Software Engineer.
```


